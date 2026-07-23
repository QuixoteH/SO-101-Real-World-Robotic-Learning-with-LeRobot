# 2. 网络、代理与离线边界

训练阶段禁止重新从 Hugging Face 下载模型、tokenizer 或数据。所有依赖应随交接包提供，并通过本地缓存运行。

## 规则

```bash
export HF_HUB_OFFLINE=1
export TRANSFORMERS_OFFLINE=1
export HF_HUB_CACHE=/实际/本地/hf_cache
```

若离线加载失败，先检查缓存目录是否完整、路径是否被 shell 展开为预期值、LeRobot 版本是否一致。不要用“关掉离线模式重新下载”掩盖缺失文件；这会让训练输入与记录不再一致。

本机曾出现 `socks://` 代理格式与当前 `httpx` 不兼容的次要错误。它不能解释本地数据集结构缺失；应先修复首要的 `tasks.jsonl`、metadata 或路径问题。
