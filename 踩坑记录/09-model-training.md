# 9. 模型训练

## 9.1 训练概述

本次使用离线包中的 `lerobot/smolvla_base` 对 SO-101 双相机 v3.0 数据微调。有效训练矩阵只有 CUDA 冒烟和一个无增强基线，不混入旧数据集历史实验。

| 实验 | 步数 | batch | save_freq | 结果 |
|---|---:|---:|---:|---|
| CUDA 冒烟 | 200 | 8 | 200 | 成功，`000200` 已保存 |
| 基线 | 25,000 | 8 | 2,000 | 成功，13 个 checkpoint |

## 9.2 Python 与环境

LeRobot 0.5.1 需要 Python 3.12。本次创建 `so101-train`，实际验证版本为 Python 3.12.13、LeRobot 0.5.1、PyTorch 2.7.1+cu126、PyAV 15.1.0。

## 9.3 不执行旧 migration

包内模型已经使用新 processor 格式。旧 `migrate_policy_normalization.py` 不适用，也没有执行。离线模型加载必须设置：

```bash
--policy.path="$SO101_BUNDLE_ROOT/models/smolvla_base"
--policy.load_vlm_weights=false
```

原始 config 若保留 `load_vlm_weights=true`，会在离线模式下查找未包含的基础 VLM 权重；通过训练 CLI 的该覆盖项加载成功。

## 9.4 相机 feature 映射

数据键为 `front/top`，策略 feature 为 `camera1/camera2`。必须保留：

```bash
--rename_map='{"observation.images.front":"observation.images.camera1","observation.images.top":"observation.images.camera2"}'
```

## 9.5 PyAV 不是可选项

v3.0 AV1 拼接视频在 TorchCodec 下可能出现视频流识别错误。训练必须指定：

```bash
--dataset.video_backend=pyav
```

全量验证实际以 PyAV 解码了 90 个 episode 的首尾样本。

## 9.6 输出目录必须不存在

首次冒烟启动时预先创建了 `02_smoke_200` 目录，LeRobot 因 `resume=false` 抛出 `FileExistsError`，发生在模型和数据初始化之前。保留预启动监控后删除空目录并重跑，成功。结论：让 `lerobot-train` 自己创建 output directory。

## 9.7 冒烟命令

```bash
lerobot-train \
  --policy.path="$SO101_BUNDLE_ROOT/models/smolvla_base" \
  --policy.load_vlm_weights=false --policy.device=cuda --policy.use_amp=true \
  --policy.n_obs_steps=1 --policy.push_to_hub=false \
  --dataset.repo_id=local/smolvla90_2color_v1_v30 \
  --dataset.root="$SO101_BUNDLE_ROOT/data/smolvla90_2color_v1_v30" \
  --dataset.streaming=false --dataset.video_backend=pyav \
  --steps=200 --batch_size=8 --num_workers=4 --save_freq=200 --log_freq=10
```

step 200 记录 loss 0.274，`000200/pretrained_model/` 存在；无 OOM、无 TorchCodec 错误、无 Hugging Face 下载。

## 9.8 25,000-step 基线

基线通过 tmux 会话 `so101-smolvla-baseline` 执行。固定参数：batch 8、AMP、`n_obs_steps=1`、`image_transforms.enable=false`、PyAV、2k 保存。训练从 12:55:41 开始，在 14:47:58 记录 `End of training`，共 1:51:51，约 3.73 step/s。

## 9.9 checkpoint

| 步数 | 路径 |
|---:|---|
| 2k-24k | `checkpoints/002000` 至 `checkpoints/024000`，每 2k 一份 |
| 25k | `checkpoints/025000` |

所有 13 份都保留。服务器不对它们排名；本机需要做固定场景实机比较。
