# 导入记录

每次把代码导入 GitHub 仓库后，在这里追加一条。表格记录概要，下面记详情。

> 本文件与 `skills/github-import/SKILL.md` 第五、六节保持同步。

## 记录表

| # | 导入时间（本地 / UTC） | 目标仓库 | 分支 | 提交号 | 文件数 | 结果 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 2026-09-15 12:04 / 04:04Z | `c888c991/github-test` | main | `ddc91cc` | 1 | 成功 |
| 2 | 2026-09-15 12:14 / 04:14Z | `c888c991/workbuddy-skills` | main | `7da0431` | 3 | 成功 |

## 字段说明

| 字段 | 取值方式 |
| --- | --- |
| 导入时间 | `date "+%Y-%m-%d %H:%M:%S"`，UTC 用 `date -u "+%Y-%m-%dT%H:%M:%SZ"` |
| 目标仓库 | `用户名/仓库名` |
| 分支 | 通常是 `main` |
| 提交号 | 前 7 位短哈希，`git rev-parse --short HEAD` |
| 文件数 | `git show --stat HEAD` 汇总行 |
| 结果 | 成功 / 失败 + 失败原因 |

---

## 详情

### 第 1 次 · 2026-09-15 12:04（UTC 04:04）

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

### 第 2 次 · 2026-09-15 12:14（UTC 04:14）

- **仓库**：`c888c991/workbuddy-skills`（新建，公开）
- **地址**：https://github.com/c888c991/workbuddy-skills
- **分支**：`main`
- **提交号**：`7da0431`
- **提交信息**：初始化技能仓库，录入 github-import 技能
- **导入内容**：
  - `README.md` —— 仓库说明
  - `IMPORT-LOG.md` —— 本文件
  - `skills/github-import/SKILL.md` —— 把代码导入 GitHub 的标准流程
  - 共 3 个文件 423 行
- **本地路径**：`E:\workbudy\2026-09-15-11-55-02\workbuddy-skills`
- **用途**：建立技能仓库，供后续在 Codex 里写好的代码按 `github-import` 技能流程导入
- **踩的坑**：`gh repo create --push` 仍走 SSH 报 22 端口被拒。原因是
  `gh config get git_protocol -h github.com` 返回 `ssh`，**按主机设置覆盖了全局设置**。
  用 `gh config set git_protocol https -h github.com` 修正后正常。
- **备注**：本次同时把 `github-import` 技能装到本机
  `C:\Users\19106\.workbuddy\skills\github-import\`，并删除了内容重复的旧技能
  `configure-github-repo`（其内容已全部并入 `github-import`）
- **结果**：成功。远端 3 个条目（含 `skills` 目录）、1 条提交。
