# 6. 服务器离线训练

## 已执行配置

- 数据：`smolvla90_2color_v1`，90 条、17,778 帧、两条语言任务。
- 模型：`lerobot/smolvla_base` 的本地离线缓存。
- 训练：先 200 step CUDA 冒烟，再 25,000 step 基线；batch size 8，AMP，未启用图像增强。
- checkpoint：2k 至 24k 每 2k 保存，另有 25k 最终 checkpoint。

## 必须先通过的检查

1. 数据集能静态加载，episode 与帧数正确。
2. 图像字段映射 `front -> camera1`、`top -> camera2` 已显式确认。
3. GPU 可用，loss 有限，无 NaN/Inf 和 OOM。
4. 保存的 checkpoint 能在独立进程重新加载。

200 step 冒烟只能证明训练链路通，不代表模型有效。25k 训练完成也不能替代真实评估。实际环境选择、磁盘与版本理由见 [服务器训练准备评估](./06-server-training-readiness.md)，命令细节和问题边界见 [训练踩坑记录](../踩坑记录/09-model-training.md)。
