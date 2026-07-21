# 服务器训练流程

1. 校验上传离线包外层 SHA256 和包内 `SHA256SUMS`。
2. 记录 GPU、磁盘、Conda 与 Python 环境；创建 Python 3.12 的 `so101-train`。
3. 以离线变量加载数据和模型，使用 PyAV 验证 90 个 episode 的首尾样本。
4. 运行 200-step CUDA 冒烟，确认 loss、显存、checkpoint 和离线行为。
5. 在 tmux 中运行 25,000-step 基线，每 2,000 steps 保存一次。
6. 保留全部 checkpoint，生成交接文档、目录校验清单和 zstd 归档。

| 阶段 | 状态 | 证据 |
|---|---|---|
| 离线包校验 | 完成 | 外层与包内 SHA256 通过 |
| 数据/模型验证 | 完成 | 180 样本、360 图像解码；本地模型与 processor 加载 |
| CUDA 冒烟 | 完成 | step 200，checkpoint `000200` |
| 基线训练 | 完成 | step 25,000，13 个 checkpoint |
| 服务器交接 | 完成 | `SERVER_HANDOFF.md`、`SHA256SUMS`、zstd 归档 |
