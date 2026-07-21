# 7. 源码与配置边界

训练源码固定为离线包中的 `code/lerobot-v0.5.1`，没有从 GitHub 拉取替代版本，也没有修改训练源码。模型已经是新 processor 格式，不运行旧 normalization migration。

本次唯一配置覆盖是训练要求的 `--policy.load_vlm_weights=false`、`--policy.n_obs_steps=1`、PyAV 后端与相机 rename map。
