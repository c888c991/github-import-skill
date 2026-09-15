---
name: github-import-skill
description: 把本地写好的代码导入 GitHub 仓库的完整操作手册与记录。用于"代码写完了要传到 GitHub""新建一个仓库""推代码上去""上传项目到 GitHub""git push 报错"这类场景。内容包含本机环境档案（账号、gh 登录、免密推送、加速工具导致的证书设置）、要导入哪些文件的清单与 .gitignore 模板、五步操作教程、9 条常见报错的原因与处理、以及每次导入的时间与数据记录表。触发词：导入 GitHub、传到 GitHub、推代码上去、上传代码到仓库、新建仓库、git push 失败、证书报错、查导入记录。
agent_created: true
---

# 把本地代码导入 GitHub 仓库（github-import-skill）

## 零、给 Codex 的专用说明

**Codex 读到本文件时，先只读这一节，再按需往下翻。**

### 这个仓库是什么

存放可复用技能的仓库。当前技能只有一个：`github-import-skill`，用途就是本文件标题——**把本地代码导入 GitHub 仓库**。

### 什么时候该用

用户说了下面任意一句，就直接按第三节的教程执行，不要自己另发明一套流程：

- 把代码传到 GitHub / 推到 GitHub / 上传到仓库
- 帮我建个仓库 / 新建仓库
- git push 报错了 / 推不上去
- 上次传到哪了 / 查一下导入记录

### 执行前自检（不通过先修，别硬来）

```bash
gh --version                                        # 没有就装：winget install --id GitHub.cli -e
gh auth status                                      # 必须显示 Logged in to github.com account c888c991
gh config get git_protocol -h github.com            # 必须返回 https
git config --global --get http.sslBackend           # 必须返回 schannel
git config --global --get http.schannelCheckRevoke  # 必须返回 false
```

后两项是本机特有的：这台机器装了 Steam++（瓦特工具箱），会劫持 GitHub 域名并重签证书，
少了这两项会直接报 `CRYPT_E_NO_REVOCATION_CHECK`。详见第一节。

### 四个硬性要求

1. **推送前必须跑 `git status`**，亲眼确认没有密钥文件混进去。
2. **必须检查 `.gitignore`**，没有就按第二节模板建一个。
3. 首次推送用 `git push -u origin main`。
4. **推完必须验证**：跑第三节第 5 步的三条验证命令，别看推送没报错就完事。

### 禁止事项

- **不要**把 `http.sslBackend` 改成 `openssl`，会报证书链缺失，更难排查。
- **不要**把 `.env`、`*.pem`、`id_rsa`、任何密钥或令牌推上去。真推了要立刻作废重发。
- **不要**去申请 gh 的 `user`、`admin:public_key` 权限。本方案用 noreply 邮箱 + HTTPS 绕开，不需要它们。

### 做完之后的强制动作

在 `IMPORT-LOG.md` 的记录表追加一行，并在详情区补一段。要记录的字段见第二节末尾的
"每次导入要记录的元数据"表。**这一步不能省**，用户就是靠这张表回溯"什么时候导了什么"。

---

## 一、这台机器的环境档案

**已配置完成，正常情况下不需要重复设置。** 换电脑时才看本节末尾的"从零配置"。

| 项目 | 值 |
| --- | --- |
| GitHub 账号 | `c888c991`（账号数字 id `211490266`） |
| gh 命令行工具 | `C:\Program Files\GitHub CLI\gh.exe`（2.100.0） |
| 登录状态 | 已登录，凭据存在系统密钥库 |
| 推送方式 | HTTPS + gh 凭据助手，**不需要 SSH 密钥**。`gh config get git_protocol -h github.com` 必须返回 `https` |
| 提交署名 | `c888c991` |
| 提交邮箱 | `211490266+c888c991@users.noreply.github.com` |

### 已写入 `C:\Users\19106\.gitconfig` 的全局配置

```
user.name=c888c991
user.email=211490266+c888c991@users.noreply.github.com
credential.https://github.com.helper=!'C:\Program Files\GitHub CLI\gh.exe' auth git-credential
http.sslBackend=schannel
http.schannelCheckRevoke=false
```

### 关键前提：本机装有 Steam++（瓦特工具箱）

它会把 `github.com`、`api.github.com` 等域名在 hosts 里劫持到 `127.0.0.1`，用自己的证书重新签名来加速访问。

**影响**：Git 默认的证书组件会做吊销检查，遇到这种自签证书会直接报错。上面那两行
`sslBackend=schannel` + `schannelCheckRevoke=false` 就是为了绕过它，**不要删**。

**注意**：不要改成 `http.sslBackend=openssl`，那样会报
`unable to get local issuer certificate (20)`，更难排查。

**顺带提醒用户**：这类工具能看到解密的 GitHub 流量，是它加速的代价。建议保留（否则可能连不上），但心里要有数。

### 从零配置（换电脑时用）

```bash
# 1. 装 gh
winget install --id GitHub.cli -e --accept-source-agreements --accept-package-agreements --disable-interactivity

# 2. 网页授权（交互式，必须用户亲自点）
"/c/Program Files/GitHub CLI/gh.exe" auth login --hostname github.com --git-protocol https --web --skip-ssh-key
#    用后台方式跑，读出一次性验证码转告用户：
#    打开 https://github.com/login/device 输入验证码，点 Authorize github

# 3. 配身份
git config --global user.name "c888c991"
git config --global user.email "211490266+c888c991@users.noreply.github.com"
git config --global init.defaultBranch main

# 4. 配免密推送
gh auth setup-git
gh config set git_protocol https -h github.com   # 必须带 -h，否则按主机设置仍是 ssh

# 5. 证书坑（仅当有加速工具时）
git config --global http.sslBackend schannel
git config --global http.schannelCheckRevoke false
```

`gh auth login` 默认只给 `gist / read:org / repo` 三项权限。读邮箱要 `user`，
上传 SSH 密钥要 `admin:public_key`——都得再走一次交互授权，很麻烦。
**所以本方案用 noreply 邮箱 + HTTPS 绕开，不折腾这两项权限。**

---

## 二、要导入什么（先看这一节，别急着推）

### 必须带

| 文件 | 为什么 |
| --- | --- |
| `README.md` | 仓库门面，写清楚这是什么、怎么跑起来 |
| `.gitignore` | 防止把临时文件和密钥推上去，**没有它最容易出事** |
| 源码 / 内容文件 | 主体 |
| 依赖清单 | `requirements.txt`（Python）、`package.json`（Node）等，别人拿到能复现环境 |

### 建议带

- `LICENSE` —— 公开仓库建议加，没有许可证等于默认保留所有权利
- `.env.example` —— 配置模板，只放字段名不放真实值
- 使用说明、截图（放 `docs/` 或 `assets/`）

### 绝对不要推（重要）

| 内容 | 原因 |
| --- | --- |
| 密钥、令牌、密码 | `.env`、`*.pem`、`id_rsa`、`token.txt`。**一旦推上去，即使删掉也留在历史里，必须立刻作废重新生成** |
| 依赖目录 | `node_modules/`、`venv/`、`.venv/` |
| 编译产物 | `dist/`、`build/`、`__pycache__/`、`*.pyc` |
| 数据集 / 视频素材 | 单文件超 100 MB 会被 GitHub 直接拒收；这类内容放网盘或 Git LFS |
| 系统文件 | `.DS_Store`、`Thumbs.db`、`.idea/`、`.vscode/`（个人配置部分） |

### 推荐的 `.gitignore` 模板

```
# 密钥与配置
.env
.env.local
*.pem
*.key

# Python
__pycache__/
*.pyc
venv/
.venv/

# Node
node_modules/
dist/
build/

# 编辑器与系统
.vscode/
.idea/
.DS_Store
Thumbs.db

# 大文件
*.zip
*.tar.gz
```

### 每次导入要记录的元数据

| 字段 | 取值方式 |
| --- | --- |
| 导入时间 | 本地时间 + UTC，用 `date "+%Y-%m-%d %H:%M:%S"` 和 `date -u` |
| 仓库 | `用户名/仓库名` |
| 分支 | 通常是 `main` |
| 提交号 | 前 7 位短哈希，`git rev-parse --short HEAD` |
| 提交信息 | 本次 `-m` 的内容 |
| 文件清单 | `git show --stat --oneline HEAD` 或 `gh api repos/.../contents` |
| 文件数 / 行数 | `git show --stat` 的汇总行 |
| 结果 | 成功 / 失败 + 失败原因 |
| 备注 | 踩到的坑、特殊处理 |

---

## 三、导入教程（五步）

### 第 1 步：进项目目录，检查要推什么

```bash
cd <项目目录>
ls -a                    # 确认没有不该推的文件
```

如果目录里还没有 `.gitignore`，**先按第二节的模板建一个**，再往下走。

### 第 2 步：初始化

```bash
git init -b main
git add .
git status               # 关键：看一眼将要提交的文件清单，确认没有密钥
```

`git status` 这一步不要省。发现多余文件先补进 `.gitignore`，再 `git rm --cached <文件>`。

### 第 3 步：提交

```bash
git commit -m "说明这次做了什么"
```

### 第 4 步：建立远程仓库

**新建仓库**（`gh` 一条命令搞定建仓 + 关联 + 推送）：

```bash
gh repo create <仓库名> --public --source=. --remote=origin --push --description "<一句话描述>"
```

把 `--public` 换成 `--private` 就是私有仓库。

**远程仓库已存在**（只关联，不推送）：

```bash
git remote add origin https://github.com/c888c991/<仓库名>.git
```

### 第 5 步：推送并验证

```bash
git push -u origin main          # 首次带 -u，之后直接 git push
```

验证：

```bash
gh repo view c888c991/<仓库名> --json name,url,visibility,defaultBranchRef
gh api repos/c888c991/<仓库名>/contents --jq '.[].name'
gh api repos/c888c991/<仓库名>/commits --jq '.[] | {sha: .sha[0:7], author: .commit.author.name, message: .commit.message}'
```

### 日常改动（项目已建好之后）

```bash
git add .
git commit -m "说明这次改了什么"
git push
```

---

## 四、常见报错对照表

| 报错关键字 | 原因 | 处理 |
| --- | --- | --- |
| `CRYPT_E_NO_REVOCATION_CHECK (0x80092012)` | Steam++ 自签证书 + 吊销检查 | 确认 `http.sslBackend=schannel`、`http.schannelCheckRevoke=false` |
| `unable to get local issuer certificate (20)` | 误把后端改成了 openssl | 改回 `git config --global http.sslBackend schannel` |
| `ssh: connect to host github.com port 22: Connection refused` | 22 端口被墙/被拦 | 改用 HTTPS；remote 换掉：`git remote set-url origin https://github.com/...` |
| 同上，但出现在 `gh repo create --push` 时 | gh 的按主机协议设置仍是 ssh，全局设置被它覆盖 | `gh config set git_protocol https -h github.com`。**必须带 `-h github.com`**，用 `gh config get git_protocol -h github.com` 核对 |
| `remote: Support for password authentication was removed` | 用了密码认证 | `gh auth setup-git` 配凭据助手 |
| `HTTP 404 ... needs the "user" scope` | gh 权限不足 | 非必要别去申请，用 noreply 邮箱代替 |
| `file is 123.45 MB; this exceeds GitHub's file size limit` | 大文件 | 从仓库移除，改用 Git LFS 或网盘 |
| `failed to push some refs ... non-fast-forward` | 远端有新提交 | `git pull --rebase origin main` 再推 |
| `fatal: not a git repository` | 不在仓库目录，或没 `git init` | 先 `git init -b main` |

**通用排查起点**（别一上来就怀疑网络）：

```bash
nslookup github.com                                              # 解析到 127.0.0.1 说明被加速工具劫持
grep -i github /c/Windows/System32/drivers/etc/hosts
tasklist | grep -iE "steam|watt|accel"
```

---

## 五、导入记录

**每次导完，在下面的表格追加一行，并在"六、导入详情"里补一段。**
本节的表格与仓库根目录 `IMPORT-LOG.md` 的第一节保持一致，记录完记得两边都更新。

| # | 导入时间（本地 / UTC） | 仓库 | 分支 | 提交号 | 文件数 | 结果 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 2026-09-15 12:04 / 04:04Z | `c888c991/github-test` | main | `ddc91cc` | 1 | 成功 |
| 2 | 2026-09-15 12:14 / 04:14Z | `c888c991/workbuddy-skills`（已改名，见七） | main | `7da0431` | 3 | 成功 |

---

## 六、导入详情

### 第 1 次 · 2026-09-15 12:04（UTC 04:04）

- **仓库**：`c888c991/github-test`（公开）
- **分支**：`main`
- **提交号**：`ddc91cc`
- **提交信息**：第一次提交：初始化测试仓库
- **导入内容**：`README.md`（1 个文件，29 行）
- **本地路径**：`E:\workbudy\2026-09-15-11-55-02\github-test`
- **用途**：验证 GitHub 配置是否打通
- **踩的坑**：见第四节第 1、3 行。SSH 22 端口被拒 → 换 HTTPS；随后报
  `CRYPT_E_NO_REVOCATION_CHECK` → 定位到本机 Steam++ 加速工具劫持域名 →
  设置 `http.sslBackend=schannel` + `http.schannelCheckRevoke=false` 解决。
  中途误试 `http.sslBackend=openssl`，报证书链缺失，已改回。
- **结果**：成功。远端 1 个文件，1 条提交。

### 第 2 次 · 2026-09-15 12:14（UTC 04:14）

- **仓库**：`c888c991/workbuddy-skills`（当时叫这个名字，12:26 改名为 `github-import-skill`）
- **分支**：`main`
- **提交号**：`7da0431`
- **提交信息**：初始化技能仓库，录入 github-import 技能
- **导入内容**：`README.md`、`IMPORT-LOG.md`、`skills/github-import/SKILL.md`（本技能本体），
  共 3 个文件 423 行
- **本地路径**：`E:\workbudy\2026-09-15-11-55-02\workbuddy-skills`
- **用途**：建立技能仓库，供后续在 Codex 里写好的代码按本技能流程导入
- **踩的坑**：`gh repo create --push` 仍然走 SSH 报 22 端口被拒。原因是
  `gh config get git_protocol -h github.com` 返回 `ssh`——**按主机设置覆盖了全局设置**。
  用 `gh config set git_protocol https -h github.com` 修正后正常。已补进第四节对照表。
- **备注**：本次同时把本技能装到本机 `C:\Users\19106\.workbuddy\skills\github-import\`，
  并删除了内容重复的旧技能 `configure-github-repo`（其内容已全部并入本技能）
- **结果**：成功。远端 3 个条目（含 `skills` 目录）、1 条提交。

---

## 七、变更记录

| 时间 | 变更 |
| --- | --- |
| 2026-09-15 12:26 | 仓库名 `workbuddy-skills` → `github-import-skill`；技能目录 `skills/github-import/` → `skills/github-import-skill/`。原因：原名看不出技能用途，Codex 检索时无法判断。旧地址会自动 301 跳转。 |
| 2026-09-15 12:26 | 仓库描述改为"把本地代码导入 GitHub 仓库的技能（github-import-skill）。含操作手册、报错排查与导入记录。" |
| 2026-09-15 12:26 | 加强技能 `description`，并在正文最前面加了"给 Codex 的专用说明"一节。 |
| 2026-09-15 12:26 | 删除本机技能副本 `C:\Users\19106\.workbuddy\skills\github-import\`，改为只维护仓库这一份（单一来源）。 |
| 2026-09-15 12:26 | 新增仓库根目录 `AGENTS.md`，供 Codex 自动加载。 |

对应提交号：`a8aa0fd`（改名 + 新增 AGENTS.md）、`07df0bf`（重写技能文件）。

### 这次踩的坑

重命名 `skills/github-import/` 时被系统占用，报 `Permission denied`，连子目录都改不动；
随后新建的 `skills/github-import-skill/` 一度落盘丢失，最后靠重写文件解决。

**结论：这台机器上不要用 `mv` 重命名已存在的目录**（尤其 GitHub 相关的目录，
Steam++ 可能持有文件句柄）。改用 `git mv`，或"新建目录 + 复制内容 + 删除旧目录"。

## 八、维护提醒

- 本技能**只在仓库里维护一份**，本机不再保留副本。所以：**改完记得提交推送**，否则改动只留在本地。
- 仓库根目录的 `AGENTS.md` 是给 Codex 自动读取的入口，内容与本节"零、给 Codex 的专用说明"保持一致。**改了一处要同步另一处。**
- 若以后要新增技能，在 `skills/` 下建同名文件夹，并同步更新根目录 `README.md` 的技能清单和 `AGENTS.md` 的技能索引。
