# 1. Ubuntu 系统与持久化路径

## 现象

训练脚本、模型缓存和输出散落在 home、`/tmp` 与数据盘时，关机、清理或迁移后无法重建完整实验。

## 已采用规则

服务器训练和交接产物统一放在 `/data/coding`。该路径保存数据集副本、离线模型缓存、日志、checkpoint、打包产物和校验文件；`/tmp` 只用于可丢弃的中间文件。

## 最小检查

```bash
df -h /data/coding
find /data/coding/so101-smolvla-experiment-2026-07-20 -maxdepth 2 -type f | sort
```

训练开始前记录可用空间；训练结束后先生成 SHA256，再复制或传输。磁盘有余量不代表归档完成，只有哈希验证一致才代表文件副本可用。
