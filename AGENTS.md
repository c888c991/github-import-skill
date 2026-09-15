# AGENTS.md —— 给 Codex / AI 助手的仓库说明

> 本文件是本仓库的入口。AI 助手在检索本仓库时，**先读这份**，再按需打开具体技能文件。
> 人也可以读，只是措辞偏指令式。

## 这个仓库是什么

存放**可复用操作技能**的仓库。每个技能是一份 Markdown 操作手册，放在 `skills/<技能名>/SKILL.md`。

不是代码项目，没有构建、没有依赖、不需要运行任何东西。

## 技能索引

| 技能 | 目录 | 用途一句话 |
| --- | --- | --- |
| `github-import-skill` | `skills/github-import-skill/SKILL.md` | 把本地写好的代码导入 GitHub 仓库 |

以后新增技能，必须同步更新这张表和根目录 `README.md`。

## 怎么用这些技能

### 第一步：判断该用哪个技能

看用户的话里有没有这些意思：

| 用户说的话 | 用哪个技能 |
| --- | --- |
| 把代码传到 GitHub、推到 GitHub、上传到仓库、帮我建个仓库、git push 报错了、上次传到哪了 | `github-import-skill` |

拿不准就先看技能的 `description` 字段——里面写了触发词。

### 第二步：打开技能文件，按里面的步骤做

**不要自己重新发明流程。** 技能里写的命令、检查项、禁止事项都是踩过坑之后定下来的，
照着做就行。特别是 `github-import-skill` 里的"四个硬性要求"和"禁止事项"，
每一条都对应一次真实故障。

### 第三步：技能要求记录结果时，必须记录

`github-import-skill` 要求每次导入后在 `IMPORT-LOG.md` 追加记录。
**这是硬性要求，不能省。** 用户可以靠这张表回溯"什么时候导了什么、提交号是多少"。

## 使用 `github-import-skill` 的快速约束

执行前自检（全通过再动手）：

```bash
gh --version                                        # 没有就装：winget install --id GitHub.cli -e
gh auth status                                      # 必须显示 Logged in to github.com account c888c991
gh config get git_protocol -h github.com            # 必须返回 https，返回 ssh 会推送失败
git config --global --get http.sslBackend           # 必须返回 schannel
git config --global --get http.schannelCheckRevoke  # 必须返回 false
```

三条禁令：

1. **不要**把 `http.sslBackend` 改成 `openssl`。这台机器装了 Steam++（瓦特工具箱），
   会劫持 GitHub 域名并重签证书，改成 openssl 必报证书链缺失。
2. **不要**把 `.env`、`*.pem`、`id_rsa` 等密钥或令牌推上去。推送前必须跑 `git status` 亲眼确认。
3. **不要**去申请 gh 的 `user`、`admin:public_key` 权限。本仓库的方案用 noreply 邮箱 + HTTPS 绕开。

完整流程见 `skills/github-import-skill/SKILL.md` 第三节。

## 维护约定

- 技能只有仓库这一份，本机不再保留副本。**改完必须 `git add` + `git commit` + `git push`**，
  否则改动只留在本地，Codex 检索不到。
- 技能的 `description` 字段是检索命中的关键，写清用途和触发词，别写成一句话概括就完事。
- 根目录 `README.md` 面向人，本 `AGENTS.md` 面向 AI，两者都要和技能内容保持一致。
