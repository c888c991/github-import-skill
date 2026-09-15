# workbuddy-skills

我的技能仓库。存放可复用的操作流程，供 WorkBuddy / Codex 等工具调用。

## 目录结构

```
workbuddy-skills/
├── README.md              本文件，仓库说明
├── IMPORT-LOG.md          导入记录表（每次导入追加一条）
└── skills/                技能目录，一个技能一个文件夹
    └── github-import/
        └── SKILL.md       把本地代码导入 GitHub 仓库的标准流程
```

## 现有技能

| 技能 | 用途 |
| --- | --- |
| `github-import` | 把本地写好的代码导入 GitHub 仓库。含环境配置、要导入哪些文件的清单、逐步教程、报错排查、导入记录 |

## 怎么用

### 在本机使用

技能同时保存在本机 `C:\Users\19106\.workbuddy\skills\` 下，可直接被调用。
本仓库这份是备份与跨设备同步用的。

### 在 Codex 里使用

把 `skills/<技能名>/SKILL.md` 复制到 Codex 对应的技能目录下即可。

### 改完记得同步

仓库里和本机各有一份，两边内容要一致。改完任意一份，把另一份覆盖过去：

```bash
# 本机 → 仓库
cp -r "C:/Users/19106/.workbuddy/skills/github-import" skills/

# 仓库 → 本机
cp -r skills/github-import "C:/Users/19106/.workbuddy/skills/"
```

## 导入记录

每次往任意仓库导入代码，都记到 `IMPORT-LOG.md`，方便回溯"什么时候导了什么"。
最近的记录：

| # | 时间 | 仓库 | 提交号 | 结果 |
| --- | --- | --- | --- | --- |
| 1 | 2026-09-15 12:04 | `c888c991/github-test` | `ddc91cc` | 成功 |
| 2 | 2026-09-15 12:14 | `c888c991/workbuddy-skills` | `7da0431` | 成功 |

## 本机环境摘要

| 项目 | 值 |
| --- | --- |
| GitHub 账号 | `c888c991` |
| 推送方式 | HTTPS + gh 凭据助手，无需 SSH 密钥 |
| 提交署名 | `c888c991` |
| 提交邮箱 | `211490266+c888c991@users.noreply.github.com` |

**重要**：本机装有 Steam++（瓦特工具箱），会劫持 GitHub 域名并重签证书。
Git 必须保持 `http.sslBackend=schannel` + `http.schannelCheckRevoke=false`，
否则无法推送。详见 `skills/github-import/SKILL.md`。
