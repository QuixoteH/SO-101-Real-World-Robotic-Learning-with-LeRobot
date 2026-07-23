# 3. 开发环境与版本边界

本项目存在两套具有不同职责的环境，不能交叉复制命令。

| 环境 | 用途 | 关键版本 |
|---|---|---|
| `lerobot` | 原始真机采集、校准和早期回放 | Python 3.10.18、LeRobot 0.3.4、PyTorch 2.7.1+cu126。 |
| `so101-real-py312` | 本机 checkpoint 推理与评估 | Python 3.12、LeRobot 0.5.1。 |
| 服务器训练环境 | 离线 SmolVLA 微调 | Python 3.12.13、LeRobot 0.5.1、PyTorch 2.7.1+cu126。 |

采集端不支持的参数不能因为训练端存在就添加；训练端的 `camera1`/`camera2` 也不能假设采集端自动生成。模型、tokenizer 和 Hugging Face 缓存必须完整随服务器交接包保存，离线模式下禁止临时联网补齐依赖。

环境报告中的 GPU、驱动与 CUDA 验证见 [环境报告](../ENVIRONMENT_REPORT_2026-07-07.md)。
