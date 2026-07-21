# 服务器环境报告

文件名沿用教程仓库结构；实际采样日期为 2026-07-21。

| 项目 | 实际值 |
|---|---|
| GPU | NVIDIA GeForce RTX 4090，24564 MiB |
| 驱动 | 550.144.03，NVIDIA-SMI 报告 CUDA 12.4 |
| Conda 环境 | `so101-train` |
| Python | 3.12.13 |
| PyTorch / TorchVision | 2.7.1+cu126 / 0.22.1+cu126 |
| LeRobot | 0.5.1，包内源码 editable 安装 |
| Transformers / PyAV | 5.3.0 / 15.1.0 |
| TorchCodec | 0.3.0（训练数据不使用它解码） |

CUDA 张量乘法在 RTX 4090 上成功执行。训练完成后 `/data/coding` 仍有约 119GB 可用空间。
