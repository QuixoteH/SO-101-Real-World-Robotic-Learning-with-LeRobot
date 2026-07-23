# 服务器环境报告

## 用途与证据等级

本文件记录用于 2026-07-20 离线训练的服务器环境，不是任意机器的安装配方。只有下表中的检测结果可视为已验证；其他机器必须重新执行同类检查。

| 项目 | 已验证值 |
|---|---|
| GPU | NVIDIA RTX 4090，24 GB。 |
| Python | 3.12.13。 |
| PyTorch | 2.7.1+cu126。 |
| LeRobot | 0.5.1。 |
| 数据盘 | `/data/coding`；训练结束后约有 119 GB 可用。 |
| CUDA 冒烟 | CUDA 张量乘法和 200 step 训练均成功。 |

## 训练前最小检查

```bash
nvidia-smi
python -c 'import torch; print(torch.__version__, torch.cuda.is_available(), torch.cuda.get_device_name(0))'
python -c 'import lerobot; print(lerobot.__version__)'
df -h /data/coding
```

预期是 GPU 可见、`torch.cuda.is_available()` 为 `True`、磁盘有容纳模型缓存、数据副本、checkpoint 和日志的余量。任何依赖下载、数据解压和训练输出都应放入持久数据盘；不要依赖 `/tmp` 保存实验唯一副本。

## 离线边界

训练使用本地 Hugging Face 缓存，开启 `HF_HUB_OFFLINE=1` 与 `TRANSFORMERS_OFFLINE=1`。离线模式下出现缺少 tokenizer、processor 或模型文件时，应补齐交接包，而不是临时改为联网并把不可复现依赖混入实验。
