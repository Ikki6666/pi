---
name: pi-fork-sync
description: pi 仓库的 git 双远程工作流。官方 origin（earendil-works/pi）保持只读用于拉取上游更新，个人修改全部放在 personal 分支并推送到自己的 fork（Ikki6666/pi）。在 pi 仓库中提交或推送自己的修改、同步官方最新代码、切换 main/personal 分支、处理 rebase 和强推，或 git 访问 github.com 超时失败时使用。
---

# pi Fork 同步工作流

本仓库是官方 pi 仓库的克隆，用户在上面维护自己的修改。git 布局：

| 对象 | 指向 | 用途 |
|---|---|---|
| 远程 `origin` | https://github.com/earendil-works/pi.git | 官方，只读，永不推送 |
| 远程 `fork` | https://github.com/Ikki6666/pi.git | 用户自己的仓库 |
| 分支 `main` | 跟踪 `origin/main` | 官方镜像，永不提交 |
| 分支 `personal` | 跟踪 `fork/personal` | 用户所有修改都提交到这里 |

## 日常修改

在 `personal` 分支上工作。提交遵守仓库 AGENTS.md 规则（显式路径 `git add <path>`，禁止 `git add -A`）：

```bash
git switch personal          # 若不在该分支
git add <显式路径> && git commit -m "fix: ..."
git push                     # 已跟踪 fork/personal，直接推到用户仓库
```

## 同步官方更新

```bash
git switch main && git pull  # main 只做拉取
git switch personal
git rebase main              # 官方更新垫到用户提交之下
git push -f fork personal    # rebase 改写历史，需强推
```

强推只允许 `fork`（用户自己的仓库）。冲突只可能出现在用户改过的文件；解决后 `git rebase --continue`。

## 网络注意

直连 github.com 会超时（HTTP2 framing / 443 连接失败）。`~/.gitconfig` 已按域名配置代理：

```
http.https://github.com.proxy = http://127.0.0.1:7897
```

- git 对 github.com 的 push/pull 已自动走代理，无需额外参数。
- 若仍然超时：先确认本地代理客户端（127.0.0.1:7897，HTTP/SOCKS 同端口）在运行，再用 `curl -x http://127.0.0.1:7897 -sI https://github.com` 验证连通。
- `gh` 命令行不吃 git 配置，需要时加环境变量：`HTTPS_PROXY=http://127.0.0.1:7897 gh ...`。
- 该代理仅对 github.com 生效，公司内网 GitLab（gitlab.origin-power.com）不受影响。

## 禁止事项

- 永不向 `origin` 推送（用户无官方写权限，推送必然失败）。
- 永不在 `main` 上提交任何东西。
- 永不强推除 `fork personal` 之外的任何分支。
- AGENTS.md 列出的破坏性 git 命令（`reset --hard`、`checkout .`、`stash` 等）在个人分支同样禁止。
