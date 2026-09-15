# github-import-skill

**用途：把本地写好的代码导入 GitHub 仓库。**

这是一个技能仓库，存放可复用的操作手册。当前只有一个技能，仓库名就直接说明了它是干什么的。

## 仓库里有什么

```
github-import-skill/
├── AGENTS.md                              给 Codex / AI 助手的入口说明（AI 先读这份）
├── README.md                              本文件，给人看的
├── IMPORT-LOG.md                          导入记录表（每次导入追加一条）
└── skills/
    └── github-import-skill/
        └── SKILL.md                       技能本体：完整操作手册
```

## 技能清单

| 技能 | 用途 |
| --- | --- |
| `github-import-skill` | 把本地写好的代码导入 GitHub 仓库 |

## 这个技能解决什么问题

代码在本地（包括在 Codex 里）写好了，要传到 GitHub。听起来简单，实际会卡在这些地方：

- `gh` 没装、没登录、权限范围不对
- 提交身份（署名、邮箱）没配，提交记录不算在自己头上
- SSH 22 端口被拒
- 本机装了 Steam++（瓦特工具箱）劫持 GitHub 域名，Git 报证书错误
- 误把 `.env`、密钥文件推了上去
- 推完之后不知道推了什么、什么时候推的

技能里对每条都写了原因和处理办法。

## 怎么用

### 用 Codex / AI 助手

直接读 `AGENTS.md`，那是入口。`AGENTS.md` 里写了技能索引、使用约束和禁止事项。

### 用 WorkBuddy

在本仓库目录下打开对话，说"把某个项目传到 GitHub"，助手会读 `AGENTS.md` 和技能文件。

### 自己在命令行操作

打开 `skills/github-import-skill/SKILL.md`，照着第三节的五步做。命令可以直接复制。

## 导入记录

每次往任意仓库导入代码，都记到 `IMPORT-LOG.md`。最近的记录：

| # | 时间 | 仓库 | 提交号 | 结果 |
| --- | --- | --- | --- | --- |
| 1 | 2026-09-15 12:04 | `c888c991/github-test` | `ddc91cc` | 成功 |
| 2 | 2026-09-15 12:14 | `c888c991/workbuddy-skills` → 本仓库 | `7da0431` | 成功 |

## 本机环境摘要

| 项目 | 值 |
| --- | --- |
| GitHub 账号 | `c888c991` |
| 推送方式 | HTTPS + gh 凭据助手，无需 SSH 密钥 |
| 提交署名 | `c888c991` |
| 提交邮箱 | `211490266+c888c991@users.noreply.github.com` |

**重要**：本机装有 Steam++（瓦特工具箱），会劫持 GitHub 域名并重签证书。
Git 必须保持 `http.sslBackend=schannel` + `http.schannelCheckRevoke=false`，
否则无法推送。详见 `skills/github-import-skill/SKILL.md` 第一节。

## 维护约定

- 技能**只在仓库里维护一份**，本机不再保留副本。改完必须提交推送，否则 Codex 检索不到。
- 新增技能时：在 `skills/` 下建同名文件夹，并同步更新 `AGENTS.md` 的技能索引和本文件的技能清单。
- 技能的 `description` 字段决定检索能不能命中，要写清用途和触发词。
