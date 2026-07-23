# SO-101 数据采集测试全流程

## 1. 当前基础状态

依据《2026-07-15 SO-101硬件测试报告》，当前已经具备数据采集条件：

- 主臂：`/dev/ttySO101_LEADER`
- 从臂：`/dev/ttySO101_FOLLOWER`
- 环境相机：`/dev/videoSO101_ENV`，使用 `YUYV`
- 夹爪相机：`/dev/videoSO101_GRIPPER`，使用 `MJPG`
- 主臂校准 ID：`my_leader`
- 从臂校准 ID：`my_follower`
- Conda 环境：`lerobot`
- LeRobot 版本：`0.3.4`
- 项目目录：`/home/quixoteh/coding/so101-real/lerobot`

本流程先录制 2 条短 episode，确认数据落盘、视频、关节状态和动作均正常，再开始批量采集。

## 2. 成功标准

最小采集测试必须同时满足：

1. 主臂、从臂和两路相机均成功连接。
2. 录制过程中从臂稳定跟随主臂，无突跳、反向或掉线。
3. 两条 episode 均成功保存，没有编码报错。
4. `front` 和 `top` 两路视频清晰、连续、方向正确，无黑屏或冻结。
5. 可视化检查中关节状态、动作和图像时间同步。
6. 在清空工作区后，episode 0 可以安全回放。

任一项不满足时，不进入批量采集。

## 3. 采集前准备

### 3.1 明确本批任务

正式启动前固定以下内容，不要在同一数据集中混用不同定义：

- 任务指令，例如：`put small green block in plate`
- 成功条件：目标物完整落入盘中
- 失败条件：未抓起、掉落、放错物体、未放入盘中、发生碰撞
- 数据集 ID，例如：`local/so101_green_plate_smoke_v1`

若任务不是“绿色方块放入盘中”，必须同时替换命令中的 `dataset.repo_id` 和 `dataset.single_task`。

### 3.2 物理安全检查

- 固定主臂、从臂和两台相机，确认底座无松动。
- 整理 USB 线和电源线，确保机械臂全行程内不会拉扯线缆。
- 清空非任务物体，确认从臂不会撞到桌沿、相机或人体。
- 将从臂置于舒适的中间姿态，避免从关节极限附近开始。
- 操作者应随时能够断开机械臂电源。
- 第一轮只做慢速、短距离动作。

### 3.3 软件和设备检查

```bash
conda activate lerobot
cd /home/quixoteh/coding/so101-real/lerobot

ls -l \
  /dev/ttySO101_LEADER \
  /dev/ttySO101_FOLLOWER \
  /dev/videoSO101_ENV \
  /dev/videoSO101_GRIPPER

test -r /home/quixoteh/.cache/huggingface/lerobot/calibration/teleoperators/so101_leader/my_leader.json
test -r /home/quixoteh/.cache/huggingface/lerobot/calibration/robots/so101_follower/my_follower.json

df -h /home/quixoteh
```

预期四个设备别名均存在，两个校准文件检查无输出且返回成功，磁盘空间充足。

如设备别名缺失，先重新插拔对应设备并执行：

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

不要改回不稳定的 `/dev/ttyACM0`、`/dev/video2` 等编号。

## 4. 录制前视觉预检

先复用已通过的带视觉遥操作命令，运行 30 至 60 秒：

```bash
python -m lerobot.teleoperate \
  --robot.type=so101_follower \
  --robot.port=/dev/ttySO101_FOLLOWER \
  --robot.id=my_follower \
  --robot.cameras='{front: {type: opencv, index_or_path: "/dev/videoSO101_GRIPPER", width: 640, height: 480, fps: 30, fourcc: MJPG}, top: {type: opencv, index_or_path: "/dev/videoSO101_ENV", width: 640, height: 480, fps: 30, fourcc: YUYV}}' \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttySO101_LEADER \
  --teleop.id=my_leader \
  --display_data=true
```

检查：

- `front` 必须是夹爪相机，能看清夹爪和目标物。
- `top` 必须是环境相机，能覆盖完整操作区域。
- 图像无黑屏、明显卡顿、过曝或严重模糊。
- 主从臂跟随和夹爪开合正常。

通过后退出遥操作，将任务物体、盘子和机械臂摆到第一条 episode 的初始状态。

## 5. 最小数据采集测试

当前本地 `record.py` 会在连接完成后直接开始第一条 episode，而不是先进入复位阶段。因此，必须在执行命令前完成第一条场景布置，并准备立即操作主臂。

```bash
conda activate lerobot
cd /home/quixoteh/coding/so101-real/lerobot

lerobot-record \
  --robot.type=so101_follower \
  --robot.port=/dev/ttySO101_FOLLOWER \
  --robot.id=my_follower \
  --robot.cameras='{front: {type: opencv, index_or_path: "/dev/videoSO101_GRIPPER", width: 640, height: 480, fps: 30, fourcc: MJPG}, top: {type: opencv, index_or_path: "/dev/videoSO101_ENV", width: 640, height: 480, fps: 30, fourcc: YUYV}}' \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttySO101_LEADER \
  --teleop.id=my_leader \
  --display_data=true \
  --dataset.repo_id=local/so101_green_plate_smoke_v1 \
  --dataset.push_to_hub=false \
  --dataset.num_episodes=2 \
  --dataset.single_task='put small green block in plate' \
  --dataset.fps=30 \
  --dataset.episode_time_s=20 \
  --dataset.reset_time_s=9999
```

本地 LeRobot 0.3.4 不支持项目旧文档中的 `--robot.fixed_joints` 参数，因此本次测试不要添加该参数。

### 5.1 键盘控制

| 按键 | 功能 |
|---|---|
| 右箭头 | 提前结束当前录制或复位阶段 |
| 左箭头 | 放弃当前 episode，并重新录制该条 |
| ESC | 停止整个采集过程 |

### 5.2 两条 episode 的操作顺序

1. 命令连接完成后，第一条 episode 立即开始。
2. 平稳完成抓取和放置；动作完成后立刻按右箭头，避免录入长时间静止帧。
3. 程序进入复位阶段。重新摆放目标物和盘子，并调整主从臂初始姿态。
4. 准备完成后按右箭头，开始第二条 episode。
5. 完成动作后立刻按右箭头，等待视频编码和数据保存结束。
6. 若一条出现碰撞、误抓、遮挡或不流畅，按左箭头重录，不保留失败演示。

## 6. 数据落盘检查

默认数据目录为：

```text
/home/quixoteh/.cache/huggingface/lerobot/local/so101_green_plate_smoke_v1
```

检查目录和元数据：

```bash
DATASET_ROOT=/home/quixoteh/.cache/huggingface/lerobot/local/so101_green_plate_smoke_v1

find "$DATASET_ROOT" -maxdepth 5 -type f | sort
jq '{total_episodes, total_frames, fps, robot_type, features}' "$DATASET_ROOT/meta/info.json"
```

预期结果：

- `total_episodes` 为 `2`。
- `fps` 为 `30`。
- 存在两条 episode 的 Parquet 数据。
- 存在 `observation.images.front` 和 `observation.images.top` 两路视频。
- `features` 中同时包含 `observation.state`、两路图像和 `action`。

## 7. 可视化验收

先只看数据，不驱动机械臂：

```bash
python -m lerobot.scripts.visualize_dataset \
  --repo-id local/so101_green_plate_smoke_v1 \
  --episode-index 0 \
  --mode local
```

再将 `--episode-index` 改为 `1` 检查第二条。

逐条确认：

- 两路图像身份没有反转。
- 视频从动作开始前很短的位置起步，动作结束后立即停止。
- 图像连续，无冻结、花屏和明显掉帧。
- 目标物、夹爪和盘子在关键阶段均可见。
- 动作轨迹平滑，没有无意义停顿、反复试探或突然加速。
- `single_task` 与实际动作完全一致。

## 8. 实机回放验收

回放会真实驱动从臂。先移除任务物和障碍物，将操作者置于可立即断电的位置，然后只回放 episode 0：

```bash
lerobot-replay \
  --robot.type=so101_follower \
  --robot.port=/dev/ttySO101_FOLLOWER \
  --robot.id=my_follower \
  --dataset.repo_id=local/so101_green_plate_smoke_v1 \
  --dataset.episode=0
```

通过标准：从臂按录制轨迹平稳运动，无关节突跳、超限趋势、异常噪声或串口掉线。发现异常立即断电，不继续回放第二条。

## 9. 进入正式批量采集

最小测试全部通过后，新建数据集版本，不要在 smoke 数据集上直接续录。例如：

```text
local/so101_green_plate_train_v1
```

建议第一批先录 10 条，立即验收，再扩大到 20 至 50 条。正式采集遵循：

1. 每个数据集只对应一个清晰的语言任务。
2. 每条都是成功、流畅、无碰撞的专家演示。
3. 目标物、容器和初始机械臂位置要有覆盖，但先保证基础场景质量，再增加干扰和光照变化。
4. 不要在同一小批次同时改变过多变量，否则出现问题时难以定位原因。
5. 每 10 条暂停检查视频、轨迹、失败率和磁盘空间。
6. 数据集名称使用 `任务_场景_批次_版本`，发现系统性问题时升级版本，不覆盖旧数据。

若继续已有数据集，必须确认任务定义和相机配置完全一致，再使用：

```bash
--resume=true
```

否则应创建新的 `repo_id`。同名目录已经存在但不使用 `--resume=true` 时会报 `FileExistsError`。

## 10. 正式数据质量准入标准

单条 episode 出现以下任一情况应重录：

- 抓错物体、未抓稳、掉落或放置失败。
- 手臂或夹爪与桌面、相机、容器发生非任务接触。
- 动作明显抖动、长时间停顿、无意义往返或速度过快。
- 图像黑屏、冻结、遮挡、严重模糊或曝光异常。
- 动作完成后保留了较长静止片段。
- 任务语言与场景或动作不一致。
- 相机位置、焦距、机械臂底座或工作台在同一数据集中发生未记录的改变。

每批采集结束后记录：日期、操作者、数据集 ID、任务指令、episode 数量、成功重录数、相机位置、光照条件、异常情况和验收结论。
