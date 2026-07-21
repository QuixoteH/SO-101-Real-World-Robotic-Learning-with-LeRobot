# 3. LeRobot 开发环境

旧 `torch` 环境为 Python 3.10，缺少 LeRobot、PyAV 和 Transformers，不能复用。创建 `so101-train` 后，使用包内 `code/lerobot-v0.5.1` 安装 `.[smolvla]`。

训练环境的关键验证：`torch.cuda.is_available()` 为真；GPU 名称为 RTX 4090；CUDA 矩阵运算成功；`lerobot-train --help` 成功。

完整版本见根目录环境报告。模型和 tokenizer 均通过 `HF_HUB_OFFLINE=1` 与包内 `models/hf_cache` 使用，不重新下载。
