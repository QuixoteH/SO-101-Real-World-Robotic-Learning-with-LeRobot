# 踩坑记录索引

本目录记录已实际遇到的问题、直接原因和最小修复边界。它不替代上游文档；遇到未记录的错误，应先保留命令、版本、日志和复现条件，再归类。

| 编号 | 主题 | 先检查什么 |
|---|---|---|
| [01](./01-ubuntu-system.md) | 系统与持久化 | 数据是否写到持久数据盘。 |
| [02](./02-network.md) | 网络与离线 | 代理格式、缓存完整性、是否意外联网。 |
| [03](./03-nvidia-driver.md) | GPU/CUDA | `nvidia-smi`、PyTorch CUDA 可用性。 |
| [04](./04-lerobot-env.md) | 环境版本 | 采集端与训练端是否混用。 |
| [05](./05-so101-hardware.md) | 串口与校准 | 稳定设备名、校准 ID、关节方向。 |
| [06](./06-camera.md) | 相机 | 设备身份、fourcc、连续读帧和 USB 供电。 |
| [07](./07-source-modifications.md) | 源码边界 | 是否把临时修补误当作通用配置。 |
| [08](./08-data-recording.md) | 录制 | 首条启动、resume 与无效 episode。 |
| [09](./09-model-training.md) | 训练 | feature 映射、PyAV、输出目录与 checkpoint。 |
| [10](./10-model-inference.md) | 推理 | 真实硬件安全、版本与评估协议。 |
| [11](./11-appendix.md) | 交接与校验 | 归档、哈希与证据边界。 |
