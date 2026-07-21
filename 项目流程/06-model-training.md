# 6. 模型训练

SmolVLA 基于包内 `models/smolvla_base` 继续训练。模型使用新 processor 格式，不执行旧 normalization migration。

## 冒烟

- 200 steps，batch 8，AMP，PyAV，无增强。
- 最终记录 loss 0.274，约 3.47 step/s。
- `checkpoints/000200/pretrained_model/` 已保存。

## 25,000-step 基线

- 开始：2026-07-21 12:55:41 +08:00。
- 结束：2026-07-21 14:47:58 +08:00，`End of training`。
- 实际 batch：8；总步数：25,000；日志报告训练循环 1:51:51，约 3.73 step/s。
- 保存：每 2,000 steps，另保存最终 25,000；共 13 份 checkpoint。

详见 [踩坑记录/09-model-training.md](../踩坑记录/09-model-training.md)。
