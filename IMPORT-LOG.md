# 记录

本文件有两张表：

- **导入记录** —— 用技能把代码导入某个 GitHub 仓库后，在这里追加。
- **变更记录** —— 本技能仓库自身的维护改动（改名、结构、规则调整等）。

> 内容与 `skills/github-import-skill/SKILL.md` 第五、六、七节保持同步。

---

## 一、导入记录

| # | 导入时间（本地 / UTC） | 目标仓库 | 分支 | 提交号 | 文件数 | 结果 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 2026-09-15 12:04 / 04:04Z | `c888c991/github-test` | main | `ddc91cc` | 1 | 成功 |
| 2 | 2026-09-15 12:14 / 04:14Z | `c888c991/workbuddy-skills`（后改名为本仓库） | main | `7da0431` | 3 | 成功 |

### 字段说明

| 字段 | 取值方式 |
| --- | --- |
| 导入时间 | `date "+%Y-%m-%d %H:%M:%S"`，UTC 用 `date -u "+%Y-%m-%dT%H:%M:%SZ"` |
| 目标仓库 | `用户名/仓库名` |
| 分支 | 通常是 `main` |
| 提交号 | 前 7 位短哈希，`git rev-parse --short HEAD` |
| 文件数 | `git show --stat HEAD` 汇总行 |
| 结果 | 成功 / 失败 + 失败原因 |

### 导入详情

#### 第 1 次 · 2026-09-15 12:04（UTC 04:04）

- **仓库**：`c888c991/github-test`（公开）
- **地址**：https://github.com/c888c991/github-test
- **分支**：`main`
- **提交号**：`ddc91cc`
- **提交信息**：第一次提交：初始化测试仓库
- **导入内容**：`README.md`（1 个文件，29 行）
- **本地路径**：`E:\workbudy\2026-09-15-11-55-02\github-test`
- **用途**：验证 GitHub 配置是否打通
- **踩的坑**：
  1. `ssh: connect to host github.com port 22: Connection refused` —— SSH 22 端口被拒，改用 HTTPS。
  2. `schannel: CRYPT_E_NO_REVOCATION_CHECK (0x80092012)` —— 定位到本机 Steam++（瓦特工具箱）
     把 GitHub 域名劫持到 `127.0.0.1` 并重签证书。设置 `http.sslBackend=schannel` +
     `http.schannelCheckRevoke=false` 解决。
  3. 中途误试 `http.sslBackend=openssl`，报 `unable to get local issuer certificate (20)`，已改回。
- **结果**：成功。远端 1 个文件、1 条提交。

#### 第 2 次 · 2026-09-15 12:14（UTC 04:14）

- **仓库**：`c888c991/workbuddy-skills`（新建，公开；12:26 改名为 `github-import-skill`）
- **分支**：`main`
- **提交号**：`7da0431`
- **提交信息**：初始化技能仓库，录入 github-import 技能
- **导入内容**：`README.md`、`IMPORT-LOG.md`、`skills/github-import/SKILL.md`，共 3 个文件 423 行
- **本地路径**：`E:\workbudy\2026-09-15-11-55-02\workbuddy-skills`
- **用途**：建立技能仓库，供后续在 Codex 里写好的代码按技能流程导入
- **踩的坑**：`gh repo create --push` 仍走 SSH 报 22 端口被拒。原因是
  `gh config get git_protocol -h github.com` 返回 `ssh`，**按主机设置覆盖了全局设置**。
  用 `gh config set git_protocol https -h github.com` 修正后正常。
- **备注**：同时把技能装到本机 `C:\Users\19106\.workbuddy\skills\github-import\`，
  并删除了内容重复的旧技能 `configure-github-repo`
- **结果**：成功。远端 3 个条目、1 条提交。

---

## 二、变更记录

| 时间 | 变更 | 说明 |
| --- | --- | --- |
| 2026-09-15 12:26 | 仓库改名：`workbuddy-skills` → `github-import-skill` | 原名看不出里面是什么技能，Codex 检索时无法判断用途。旧地址会自动 301 跳转 |
| 2026-09-15 12:26 | 仓库描述改为"把本地代码导入 GitHub 仓库的技能…" | 同上，帮助检索命中 |
| 2026-09-15 12:26 | 技能目录改名：`skills/github-import/` → `skills/github-import-skill/` | 与仓库名保持一致 |
| 2026-09-15 12:26 | 加强技能 `description` 字段 | 原描述没点明用途，现加入触发词，提高检索命中率 |
| 2026-09-15 12:26 | 技能正文新增"零、给 Codex 的专用说明"一节 | 放在最前面，Codex 打开文件即可看到怎么用、禁止什么 |
| 2026-09-15 12:26 | 新增仓库根目录 `AGENTS.md` | Codex 自动读取的入口文件 |
| 2026-09-15 12:26 | 删除本机技能副本 | 改为只在仓库维护一份，避免两边不一致 |

### 提交号

| 提交 | 提交号 | 内容 |
| --- | --- | --- |
| 第 1 次改名 | `a8aa0fd` | 仓库与技能改名为 github-import-skill，新增 AGENTS.md、强化描述 |
| 第 2 次 | `07df0bf` | 重写技能文件到 `skills/github-import-skill/SKILL.md` |

### 过程中的问题

`skills/github-import/` 重命名时被系统占用（权限拒绝，连子目录都改不动），
随后新建的 `skills/github-import-skill/` 一度落盘丢失，已重写后提交。
**结论：这台机器上不要用 `mv` 重命名已存在的目录，改用 `git mv` 或"新建 + 复制 + 删除"。**
