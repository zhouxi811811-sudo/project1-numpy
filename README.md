# Project 1 NumPy 图像编码器

使用课程提供的 OpenCLIP ViT-B-32 图像权重完成 NumPy 前向计算。输入为 float32 [B,3,224,224]，输出为逐图归一化的 float32 [B,512]。

## 文件与复现

本仓库提供完成的 image_encoder.py、运行说明和验证记录，实验报告随课程提交材料提供。运行依赖课程原始 project1-学生版.zip。先解压课程附件，将本仓库的 image_encoder.py 复制到与 check.py 同一目录，替换学生版中的同名文件。保留原包中的 helpers.py、runtime.py、run.py、check.py、requirements.txt、config.json、weights/ 和 assets/。

需要 Python 3.9 或以上。在含 check.py 的目录打开终端，运行以下命令。macOS/Linux 使用 source 激活；Windows 使用 .venv\\Scripts\\activate。

    python3 -m venv .venv
    source .venv/bin/activate
    python -m pip install -r requirements.txt
    python check.py --stage all
    python run.py --output output_tokens.npz

自定义图片的使用示例：

    python run.py --images path/to/a.jpg path/to/b.png --output my_tokens.npz

输出 NPZ 文件中的 ids 与 output_tokens 按输入顺序对应。默认十张图片得到 (10,512) 数组。计算使用 CPU 和固定参数。

## 实现

- TODO-01：先拆分图块网格，再交换轴，按块行、块列以及块内 C/H/W 顺序展平。
- TODO-02：把特征分成 12 个头，每头 64 维，保留批次与 token 顺序。
- TODO-03：计算缩放 QK 分数、注意力加权、头合并和输出投影。
- TODO-04：两次残差相加，以及使用给定 GELU 的 MLP。
- TODO-05：图块投影、CLS 前置拼接和位置相加。
- TODO-06：选择 CLS、post-Norm、输出投影和单位长度缩放。

## 实测环境与结果

2026-11-20，macOS arm64，Python 3.12.14、NumPy 2.3.5、Pillow 12.3.0。全部八个原始检查阶段通过，详见 validation.txt。最终输出最大绝对误差 6.780e-07。额外使用新生成的 291×173 灰度图通过 run.py 检验单图入口，结果为有限 float32 (1,512)，L2 范数为 1。

逐阶段排错顺序：

    python check.py --stage patches
    python check.py --stage tokens
    python check.py --stage block0
    python check.py --stage last_tokens
    python check.py --stage output
    python check.py --stage batch

diagnostic.txt 记录主动构造的错误切块实现及其修复对照。直接 reshape 与先重排轴的结果分别和课程参考比较，用于说明形状正确仍可能发生像素顺序错误。

## 贡献与辅助说明

课程提供模型参数、预处理、数学辅助函数、加载与运行框架、参考数据和检查脚本。本次六组 TODO、代码注释、运行验证、受控故障复现及报告初稿由 Codex AI 辅助生成与执行。报告保留这一辅助范围说明。姓名、学号、班级由提交者填写。

大型参数文件、参考数据与虚拟环境使用课程附件及本地环境。
