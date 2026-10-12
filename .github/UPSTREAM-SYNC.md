# 上游自动同步说明

本仓库是 **[LGQLIFE/agnes-ai-studio](https://github.com/LGQLIFE/agnes-ai-studio)** 的镜像，
由 GitHub Actions 每天自动检查上游是否有新提交，有则同步过来。

- 上游：`https://github.com/LGQLIFE/agnes-ai-studio.git`（MIT 协议）
- 同步分支：`main`、`master`（两个都跟随上游）
- 同步模式：`merge`（保留本仓库改动并与上游合并）
- 触发时间：每天 UTC 02:17，即北京时间 10:17
- 手动触发：`Actions` → `Sync Upstream` → `Run workflow`

## 这个仓库的特殊之处

上游的默认分支 `main` 是个**空占位**（只有 `LICENSE`、`.gitignore` 和一句描述的 `README.md`），
**真正的项目代码全在 `master` 分支**。所以本镜像的默认分支被设为 `master`，打开即见代码。

`master` 分支里约 125 MB 是提交进仓库的 PyInstaller 构建产物：

| 路径 | 大小 |
| :--- | :--- |
| `dist/Agnes-AI-Studio.exe` | 62.6 MB |
| `build/Agnes-AI-Studio/Agnes-AI-Studio.pkg` | 62.3 MB |
| `build/` 其余文件 | 约 11 MB |

这部分是**原样镜像**的（与上游 100% 一致），源码本身只有约 1.5 MB。首次 clone 会比较慢。

## 改配置

所有行为由 `.github/upstream-sync.json` 决定，改这一个文件即可：

```json
{
  "upstream": "https://github.com/LGQLIFE/agnes-ai-studio.git",
  "branches": ["main", "master"],
  "primary": "master",
  "mode": "merge",
  "syncTags": true
}
```

- `branches`：要跟随上游同步的分支列表。上游新增分支时，把分支名加进来即可。
- `mode`：`merge` 合并保留改动；`mirror` 每次硬覆盖为上游最新（丢弃本地改动）。
- `syncTags`：是否同步 tag。

## 已知限制

1. **仓库连续 60 天没有任何提交活动时，GitHub 会自动停用定时任务。**
   进 `Actions` → 选 `Sync Upstream` → 点 `Enable workflow` 即可恢复。
2. 本仓库是 **Public**，Actions 分钟数免费不限量。若改成 Private，会消耗账号每月免费额度
   （本任务单次约 1 分钟）。
3. 若上游做了 force-push 重写历史，`merge` 模式可能报冲突而不是静默覆盖。
   届时把 `mode` 改成 `mirror` 再手动跑一次即可对齐。
4. `mirror` 模式下每个分支会保留一个额外的提交（`chore(ci): 安装上游自动同步工作流`），
   用于把同步工作流本身装回分支——上游没有 `.github/` 目录，不会被覆盖。

## 本地手动同步

```bash
git fetch upstream --prune --tags --force
git checkout master && git merge upstream/master
git checkout main   && git merge upstream/main
git push origin --all --tags
```
