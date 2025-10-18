# 基于 GPT-SoVITS 管道的 Vtuber 语音生成器微调

本项目使用 Vtuber 相关数据，基于 **GPT-SoVITS 管道**微调语音生成器模型。以下是详细的流程步骤：

---
## 0. 安装依赖：
  ```bash
    pip install -r requirements.txt
  ```
## 1. 训练数据生成

- **音频下载：**  
  - 使用 [yt-dlp](https://github.com/yt-dlp/yt-dlp) 下载 Vtuber 音频文件，同时下载字幕和时间戳：

    ```bash
    yt-dlp --write-subs --all-subs -f bestaudio --extract-audio --audio-format wav --sub-format srt -o "%(title)s.%(ext)s" --cookies-from-browser chrome url
    ```

  - 对于没有字幕的视频，使用 [Whisper](https://github.com/openai/whisper) 生成 SRT 格式的字幕：

    ```bash
    whisper sample.wav --model small --output_format srt --language Chinese
    ```

  - （可选）为了减少时间戳误差，通过运行 tools/audio_text_folder 中的 split_audio 脚本执行初步切片，以减小大型音频文件的大小：

    ```bash
    python split_audio.py sample.wav output_folder
    ```

---

## 2. 数据集准备

- **音频切片：**  
  使用自定义切片器根据字幕时间戳切割音频文件。但是，某些切片可能在开头或结尾包含较长的静音。请在脚本中重新定义您的输入文件夹和输出文件夹：
  ```bash
    python slice.py
  ```
  
- **降噪：**  
  对切片后的音频文件进行降噪处理以提高质量：
   ```bash
    python denoise.py -i input_folder -o output_folder  
  ```

- **转录：**  
  为每个音频切片生成相应的文本文件，包含各自的转录内容。这也在切片步骤中完成。

---

## 3. 特征生成

### 3.1 文本特征
- 将转录文本输入到预训练的 BERT 模型 ([Chinese-Roberta-WWM-Ext-Large](https://huggingface.co/lj1995/GPT-SoVITS/tree/main)) 中以生成 BERT 标记。需要将模型下载到 pretrained_model 文件夹
- 文本还使用 [pypinyin-g2pW](https://github.com/mozillazg/pypinyin-g2pW) 转换为音素。需要将 G2PWModel 下载到 tools 文件夹中的 G2PWModel 文件夹中。
- 提取步骤可通过运行以下命令完成：
   ```bash
    python 1-dp-get-text.py
  ```

### 3.2 音频特征
- 将输入音频转换为 CN-HuBERT 特征。我们使用预训练的 CN-HuBERT 模型 ([chinese-hubert-base](https://huggingface.co/lj1995/GPT-SoVITS/tree/main))
- CN-HuBERT 特征被传递到预训练的 **SynthesizerTrn** 模型 (S2G488K.pth) 以生成语义标记。
- 提取步骤可按步骤运行：
  ```bash
    python 2-dp-get-hubert-wav32k.py
  ```
  ```bash
    python 3-dp-get-semantic.py
  ```

---

## 4. 微调

### 4.1 SoVITS 训练
- 训练模型从语义标记预测 WAV 音频输出：
- 训练步骤通过以下命令完成：
  ```bash
     python s2_train.py --config "./configs/tmp_s2.json" --exp_dir "./logs/v1_trial"
  ```
### 4.2 GPT 训练
- GPT 组件被训练使用以下内容预测下一个语义标记：
  - 当前语义标记
  - 音素
  - BERT 特征
- 训练步骤通过以下命令完成：
  ```bash
    python s1_train.py --config "configs/tmp_s1.yaml"
  ```

---

## 5. 推理

- **inference_cli**：
  - 从 GPT-weights 文件夹和 SoVITS-weights 文件夹加载微调后的权重。
  - 使用参考音频和目标文本生成目标音频。我们在脚本中预定义了一段参考音频，请替换为您自己的参考音频路径。
    ```bash
      python inference_cli.py --gpt_model ./GPT_weights/gpt_model.ckpt --sovits_model ./SoVITS_weights/sovits_model.pth --target_text ./test.txt  --output_path ./output_folder
    ```

- **观察：**  
  移除参考音频会显著降低生成的目标音频的质量。

---

## 6. 评估
- 我们使用 [speechmetrics](https://github.com/aliutkus/speechmetrics/tree/master) 和 [mel_cepstral_distance](https://github.com/jasminsternkopf/mel_cepstral_distance) 进行模型的评估和比较。

---

## 相关项目

有关更多详细信息，请访问官方 [GPT-SoVITS GitHub 仓库](https://github.com/RVC-Boss/GPT-SoVITS/tree/main?tab=readme-ov-file)。


