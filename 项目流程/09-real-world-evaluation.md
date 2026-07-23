# SmolVLA 绿色/紫色 Checkpoint 评估总报告与最终命令

## 一页结论

本报告汇总 6 组真实机械臂连续推理记录，每组 10 条，共 60 条。终态成功判据统一为：方块在环境相机末帧中**完整位于盘内**。

| 颜色 | Checkpoint | 成功 | 成功率 | 结论 |
|---|---:|---:|---:|---|
| 绿色 | 002000 | 2 / 10 | 20% | 早期基线，不可部署。 |
| 绿色 | 010000 | 9 / 10 | 90% | 当前连续展示最佳；应优先做独立复位复验。 |
| 绿色 | 020000 | 7 / 10 | 70% | 可完成任务，但现有证据不超过 010000。 |
| 紫色 | 002000 | 1 / 10 | 10% | 早期基线，不可部署。 |
| 紫色 | 010000 | 5 / 10 | 50% | 中等稳定性，尚不足以部署。 |
| 紫色 | 020000 | 5 / 10 | 50% | 与 010000 同水平，尚不足以部署。 |

按颜色合并时，绿色为 `18/30 = 60.0%`，紫色为 `11/30 = 36.7%`；按 checkpoint 合并时，002000 为 `3/20 = 15.0%`、010000 为 `14/20 = 70.0%`、020000 为 `12/20 = 60.0%`。

## 评估协议与边界

| 项目 | 实际设置 |
|---|---|
| 任务 | `put small green block in plate` 或 `put small purple block in plate` |
| 控制/数据频率 | 20 Hz |
| 相机 | `camera1`: 夹爪视角，480x640、MJPG、30 FPS、旋转 90 度；`camera2`: 环境视角，640x480、YUYV、20 FPS。 |
| 机器人 | SO101 follower，端口 `/dev/ttySO101_FOLLOWER`，ID `my_follower`。 |
| 主臂 | SO101 leader，端口 `/dev/ttySO101_LEADER`，ID `my_leader`。 |
| 每条最大时长 / reset | 30 s / 20 s |
| 记录格式 | LeRobot v3、双路 AV1 视频、Parquet 关节记录；不上传 Hub。 |
| 成功判定 | 人工复核环境相机终帧；方块完整在盘内为成功。 |

终态成功率不能回答“中间是否空抓”“是否掉落”“哪个阶段首次失败”。`|policy action - observation.state|` 的关节误差也不是 GPU 推理耗时、相机帧率或端到端延迟。要给出这些结论，下一轮必须记录阶段标签与逐阶段时间戳。

## 数据与统计总表

| 颜色 | Checkpoint | 活跃数据目录 | 帧数 | 总时长 | 单条时长均值 ± 样本标准差 | 全关节 action-state MAE |
|---|---:|---|---:|---:|---:|---:|
| 绿色 | 002000 | `eval_so101_green_ckpt_002000_recal_v2` | 944 | 47.20 s | 4.72 ± 1.44 s | 4.61 deg |
| 绿色 | 010000 | `eval_so101_green_ckpt_002000_recal_v1` | 1,431 | 71.55 s | 7.15 ± 1.04 s | 3.61 deg |
| 绿色 | 020000 | `eval_so101_green_ckpt_020000_recal_v1` | 1,201 | 60.05 s | 6.01 ± 1.14 s | 3.78 deg |
| 紫色 | 002000 | `eval_so101_purple_ckpt_002000_recal_v1` | 814 | 40.70 s | 4.07 ± 1.06 s | 4.64 deg |
| 紫色 | 010000 | `eval_so101_purple_ckpt_010000_recal_v1` | 1,002 | 50.10 s | 5.01 ± 0.93 s | 3.97 deg |
| 紫色 | 020000 | `eval_so101_purple_ckpt_020000_recal_v1` | 970 | 48.50 s | 4.85 ± 1.14 s | 3.85 deg |



## 分颜色解读

### 绿色

绿色任务显示清晰的训练早期到中期提升：002000 为 2/10，010000 提升到 9/10，020000 为 7/10。010000 是当前最值得做固定起点复验的候选，但其 9/10 来自连续展示而非独立复位；95% Wilson 区间为 59.6%-98.2%，样本仍不足以给出稳定部署承诺。

### 紫色

紫色任务整体更难：002000 为 1/10，010000 和 020000 均为 5/10。紫色的低成功率不能仅归咎于颜色本身，可能同时受初始位置、方块外观、连续运行累积偏差和策略能力影响。当前没有任何紫色 checkpoint 达到可接受的独立复位部署门槛。

## 实际记录所用命令：公共固定部分

六组运行均使用以下环境、设备、相机与记录参数；每组仅替换下一节表中的四项差异参数。为保持与已记录实验相同，环境变量也保留离线模式与本地 Hugging Face 缓存。

```bash
conda activate so101-real-py312

cd /home/quixoteh/SO-ARM-101

export HF_HUB_OFFLINE=1
export TRANSFORMERS_OFFLINE=1
export HF_HUB_CACHE=/home/quixoteh/SO-ARM-101/artifacts/so101-smolvla-training-bundle-2026-07-20/models/hf_cache
export TOKENIZERS_PARALLELISM=false

lerobot-record \
  --robot.type=so101_follower \
  --robot.port=/dev/ttySO101_FOLLOWER \
  --robot.id=my_follower \
  --robot.disable_torque_on_disconnect=true \
  --robot.cameras='{
camera1: {
type: opencv,
index_or_path: "/dev/videoSO101_GRIPPER",
width: 480,
height: 640,
fps: 30,
fourcc: MJPG,
rotation: 90
},
camera2: {
type: opencv,
index_or_path: "/dev/videoSO101_ENV",
width: 640,
height: 480,
fps: 20,
fourcc: YUYV
}
}' \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttySO101_LEADER \
  --teleop.id=my_leader \
  --dataset.repo_id=<见下表> \
  --dataset.root=<见下表> \
  --dataset.single_task=<见下表> \
  --dataset.fps=20 \
  --dataset.episode_time_s=30 \
  --dataset.reset_time_s=20 \
  --dataset.num_episodes=10 \
  --dataset.video=true \
  --dataset.push_to_hub=false \
  --dataset.num_image_writer_processes=0 \
  --policy.path=<见下表> \
  --policy.device=cuda \
  --policy.load_vlm_weights=false \
  --play_sounds=false
```



## 归档位置

所有 ZIP 在当时均通过 `unzip -t` 完整性校验。为避免公开仓库链接到仅存在于本机的路径，下表只记录归档文件名；原始 ZIP 不随本仓库发布。

| 组别 | 归档文件名 |
|---|---|
| 绿 002000 | `eval_so101_green_ckpt_002000_episodes_0_9.zip` |
| 绿 010000 | `eval_so101_green_ckpt_010000_episodes_0_9.zip` |
| 绿 020000 | `eval_so101_green_ckpt_020000_episodes_0_9.zip` |
| 紫 002000 | `eval_so101_purple_ckpt_002000_episodes_0_9.zip` |
| 紫 010000 | `eval_so101_purple_ckpt_010000_episodes_0_9.zip` |
| 紫 020000 | `eval_so101_purple_ckpt_020000_episodes_0_9.zip` |
