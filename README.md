# myGPT-SoVITS

基于 [GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS) 的个人二次开发与训练实践项目。

这个仓库主要用于整理我自己的 **数据准备、特征提取、模型微调与命令行推理** 流程，目标不是还原官方完整 GUI，而是沉淀一套更适合自己复现、调试和继续开发的最小可运行版本。

如果你想要的是：

- 从零整理一套 GPT-SoVITS 的训练/推理流程
- 在 Windows 上尽量少踩一点依赖坑
- 用命令行完成数据准备、训练和推理
- 在原始项目基础上做一些兼容性和工程化调整

那这个仓库会比较适合参考。

## 项目特点

- 聚焦命令行工作流：围绕数据准备、特征提取、训练、推理组织脚本，而不是依赖完整 WebUI。
- 保留 GPT-SoVITS 核心链路：文本特征、HuBERT 特征、语义 token、SoVITS/GPT 两阶段训练都可独立运行。
- 补充了实际使用中的兼容处理：
  - `LangSegment` 导入兼容与回退
  - `G2PW` 不可用时的降级方案
  - Windows / 单卡环境下绕开 `libuv` 的单进程训练路径
  - 推理阶段的文本编码探测、语言检测与本地 NLTK 资源处理
- 配套了中文操作手册，适合作为个人学习记录和复现笔记。

## 仓库定位

这不是官方 GPT-SoVITS 的镜像，也不是一个追求功能最全的发行版。

更准确地说，它是一个：

- 适合自己持续维护的实验仓库
- 能够沉淀踩坑经验的训练项目
- 面向“最小可运行流程”的工程化整理版本

## 工作流概览

完整流程大致如下：

1. 准备并清洗音频数据
2. 编写标注文本
3. 提取文本侧特征（音素、BERT）
4. 提取音频侧特征（CN-HuBERT、32k wav）
5. 生成语义 token
6. 训练 SoVITS（语义 token -> 波形）
7. 训练 GPT（文本 -> 语义 token）
8. 使用参考音频 + 目标文本完成推理

核心脚本如下：

- `1-dp-get-text.py`：提取文本侧特征
- `2-dp-get-hubert-wav32k.py`：提取 HuBERT 特征与 32k 音频
- `3-dp-get-semantic.py`：生成语义 token
- `s2_train.py`：训练 SoVITS
- `s1_train.py`：训练 GPT
- `inference_cli.py`：命令行推理

## 目录结构

```text
myGPT-SoVITS/
├─ configs/               # 训练配置
├─ GPT_weights/           # GPT 权重输出
├─ SoVITS_weights/        # SoVITS 权重输出
├─ pretrained_models/     # 预训练模型
├─ logs/                  # 特征与训练日志
├─ tools/                 # 工具脚本与辅助资源
├─ text/                  # 文本处理相关模块
├─ test_text/             # 推理测试文本
├─ inference_output/      # 推理输出
├─ 1-dp-get-text.py
├─ 2-dp-get-hubert-wav32k.py
├─ 3-dp-get-semantic.py
├─ s1_train.py
├─ s2_train.py
└─ inference_cli.py
```

## 快速开始

### 1. 安装依赖

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

### 2. 放置预训练模型

将 GPT-SoVITS 依赖的 BERT、CN-HuBERT 和基础权重放到 `pretrained_models/` 目录下。

推荐直接参考：

- `README_CN.md`
- `操作手册.md`

## 最小推理示例

```bash
python inference_cli.py ^
  --gpt_model ./GPT_weights/v1_trial-e40.ckpt ^
  --sovits_model ./SoVITS_weights/v1_trial_e20_s480.pth ^
  --target_text ./test_text/text.txt ^
  --output_path ./inference_output ^
  --lang ja
```

说明：

- `--gpt_model`：GPT 权重路径
- `--sovits_model`：SoVITS 权重路径
- `--target_text`：待合成文本
- `--output_path`：输出目录
- `--lang`：可选 `auto|zh|en|ja|ko|yue`

当前版本的 `inference_cli.py` 默认读取 `tools/denoised/output/reference.wav` 作为参考音频，请先准备好对应文件。

## 文档入口

- [README_CN.md](./README_CN.md)：项目中文说明
- [README_EN.md](./README_EN.md)：项目英文说明
- [操作手册.md](./操作手册.md)：最小可运行流程与常见问题

## 相关文章

- [GPT-SoVITS学习笔记](https://akineko.netlify.app/2026/04/19/gpt-sovits/)

这篇文章更偏原理理解和实践心得，适合搭配仓库一起看。

## 致谢

- [RVC-Boss/GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS)
- [Ayaka-Yuki/GPT-SoVITS-fine-tuning](https://github.com/Ayaka-Yuki/GPT-SoVITS-fine-tuning)

## License

本仓库使用 [MIT License](./LICENSE)。
