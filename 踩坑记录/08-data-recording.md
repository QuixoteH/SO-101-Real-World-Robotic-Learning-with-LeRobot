# 8. 录制、resume 与无效 episode

## 首条和 reset

当前采集端的录制流程可能在设备连接后直接开始第一条，而不是先进入复位阶段。因此命令启动前必须摆好第一个场景；不要默认可以慢慢准备。

## 空目录不是可 resume 数据集

以 `num_episodes=0` 初始化只生成部分 metadata，可能没有 `tasks.jsonl`；随后使用 `--resume=true` 会失败。正确流程是首次不带 resume 录至少一条有效 episode，确认视频和 metadata 完整后才追加。

## 无效条处理

短误录、全程静止、起始时方块已在盘内、视频缺失或任务失败的 episode 不得进入训练。删除时必须同步 Parquet、两路视频、`episodes.jsonl`、统计文件和 `info.json`，随后用 LeRobot 重新加载验证编号连续。详见 [采集实操与质检](../项目流程/05-data-collection-practice.md)。
