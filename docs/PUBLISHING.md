# 下载站、发布包与客户端更新指南

> 适用范围：这是公开下载站的跨电脑接手指南。它只说明公开网站基础设施、发布顺序和验证边界；具体产品的源码、业务数据、账号配置和产品专属验收，以该产品随仓库交付的 README、HANDOFF 与测试为准。

## 先分清三件事

1. 上传完整 ZIP，只让新用户可以下载；它不会让已安装客户自动更新。
2. 修改产品的客户端更新清单，才可能让**该产品中已实现且已验证的更新器**发现新版本；不是每个产品都有自动更新，也不是每个旧版都兼容同一清单。
3. 修改下载站卡片，只改变网页展示和新用户下载入口；`git push` 本身不等于客户端已更新。

任何一项未验证，都只能报告对应项完成，不能合并表述为“所有用户已升级”。

<a id="website"></a>

## 网站接手与 Pages 边界

### 新 Windows 电脑准备

至少安装 Git。若要查询 Pages、Release 或远端提交，可额外安装 GitHub CLI 并使用浏览器交互登录；不要把令牌、浏览器 Cookie、AccessKey 或用户配置放入仓库。

```powershell
git clone https://github.com/nmrsz645-bit/software-download-center.git
Set-Location software-download-center
git fetch origin
git switch main
git pull --ff-only origin main
git status --short
```

截至本指南核对时，Pages 使用 `main` 分支根目录，域名为 `https://download.luotuoqiluotuozhaoma.com/`，并启用 HTTPS。仓库没有受控的 `.github/workflows` 文件；GitHub 的 Pages 构建由平台管理。只有合并或推送到 Pages 实际来源 `main` 才可能发布网页。

本指南所在的 `docs-cross-pc-handoff` 分支仅用于文档：

```powershell
git fetch origin main docs-cross-pc-handoff
git switch docs-cross-pc-handoff
git rebase origin/main
git push origin docs-cross-pc-handoff
```

不要强推。若远端 HEAD 已变化，先 `git fetch`，阅读差异并正常整合；不要覆盖他人的站点或清单修改。未经单独授权，不把文档分支合入 `main`，不改 Pages 设置。

### 站点文件职责

| 文件 | 职责 | 不应在普通发布时做什么 |
| --- | --- | --- |
| `index.html` | 静态卡片、静态下载兜底链接和浏览器端运行时刷新逻辑 | 不因为新版本直接猜测或批量替换全部产品版本 |
| `CNAME` | 下载站自定义域名 | 不改域名或 HTTPS/Pages 设置 |
| `updates/fire-matrix-local/latest.json` | 火花矩阵应用更新载荷清单 | 不把它当作全站 catalog 或其他产品清单 |
| `updates/fire-matrix-local/site-latest.json` | 火花矩阵网站卡片读取的站内 JSON | 不与应用更新清单混用 |

网页加载失败或运行时清单不符合页面校验时，页面会保留 `index.html` 中的静态下载链接。这是兜底，不是新版本已发布的证明。

<a id="catalog"></a>

## 卡片、运行时清单与 catalog

### 当前机制

下载站没有统一、仓库内生效的 `catalog.json`，`index.html` 也不读取公共 catalog。曾存在的公开历史对象 `catalog.json` 和 `downloads/catalog.json` 位于旧 OSS 域名，当前仍可访问，但在本次核对的网页运行时代码中**没有被加载**；不要仅因它们存在就修改或宣称网页依赖它们。

当前 `index.html` 同时包含：

- 静态产品卡片和完整 ZIP 兜底链接；
- 通过 `<script>` 加载的产品专属 `latest.js`，用于刷新部分卡片；
- 火花矩阵的同仓库 `updates/fire-matrix-local/site-latest.json`；
- 知乎出书和 DNF 对表工具各自的内联加载/字段校验逻辑。

因此，更新网页前先在 `index.html` 找到该产品的卡片 `id` 和加载器，确认它属于下列哪一种：

| 机制 | 已核对的例子 | 页面要求 |
| --- | --- | --- |
| 外部 `latest.js` | 根 `updates/latest.js`、`updates/novel/latest.js`、`updates/zhangxin-book/latest.js`、`updates/novel-amount-monitor/latest.js`，以及旧 OSS 下的部分产品路径 | 脚本须定义页面实际读取的 `window.*_LATEST` 对象；页面至少要求有效版本和 HTTPS 完整包 URL |
| 站内 JSON | `updates/fire-matrix-local/site-latest.json` | 页面校验 `productId`、四段版本号、64 位 SHA-256、国内完整包 URL 和 GitHub 备用包 URL 的固定格式 |
| 内联专属加载器 | 知乎出书、DNF 对表工具 | 页面会校验 `productId`、三段版本号、SHA-256 和由版本推导出的完整包 URL |
| 静态卡片 | 没有对应加载器的卡片 | 需要在 `index.html` 同时更新显示版本、下载链接和文字；不要假设会自动刷新 |

`latest.js` 只服务网页展示时，通常含 `version` 与 `fullPackageUrl`，有些对象另含 `fullPackageSha256`。它不是所有产品客户端更新器都读取的清单。客户端通常读取产品专属 JSON，字段、旧版兼容 URL、包布局和是否需用户确认均由该产品决定。

### 修改网站卡片的最小核对

1. 先从公网重新下载完整 ZIP，重新计算 SHA-256。
2. 确认该卡片是静态、外部 `latest.js`、站内 JSON 还是内联专属加载器。
3. 仅更新该机制实际读取的字段；不要把其他产品清单、旧 catalog 或下载链接混入。
4. 用带随机查询参数的 HTTPS 页面重新打开，检查显示版本和链接；再直接下载链接核对哈希。
5. 网站展示和客户端更新清单分别记录结果。

<a id="client-update"></a>

## 版本包与客户端更新发布

### 通用且不可颠倒的顺序

1. 在项目外的新建输出目录构建，避免带入用户配置、媒体、日志、浏览器资料、数据库、Token 或历史包。
2. 运行该产品规定的离线测试、包布局检查和密钥扫描；记录版本、构建提交、字节数和 SHA-256。
3. 上传不可变版本路径的**完整 ZIP**；如果产品有 app-only 更新 ZIP，也上传它。不要覆盖旧版本包。
4. 从公网重新下载每个 ZIP，比较字节数和 SHA-256。任一不符即停止，不改清单。
5. 只在产品专属兼容规则确认后，最后更新客户端 JSON / `latest.js`。旧客户端可能读取固定旧地址；没有该产品的隔离旧版升级证据时，不要改写旧入口。
6. 再更新适用的网站卡片或站内 JSON，并完成页面读回。
7. 在独立旧版本目录运行该产品的正常启动入口，验证升级、回滚和受保护数据。直接运行内部 EXE 可能绕过启动器和更新检查。

“用户可选择下载、下次启动安装；失败继续使用旧版”的产品，必须保留该产品自己的更新器标识、状态目录、清单 URL、互斥规则和启动入口。不要复制另一个产品的 EXE 作为替代品。

### 已核对到的构建/发布工具

没有一份已经验证可发布所有产品的统一脚本。下列工具只能在其所属项目、已读该项目 HANDOFF 且获得发布授权后使用：

| 工具 | 已核对用途 | 安全使用方式 |
| --- | --- | --- |
| `tools/build_github_release.py` | 小说处理中心：构建本地 Release 输出 | `python tools/build_github_release.py --version <版本> --output <全新输出目录>`；它不是 OSS 上传命令 |
| `tools/prepare_domestic_release.py` | 小说处理中心：将已构建输出整理为待上传目录 | `python tools/prepare_domestic_release.py --release <构建输出> --output <全新待上传目录>`；仍需单独获授权上传 |
| `scripts/publish_oss_release.py` | 抖音作品提取发布管线：校验后上传 ZIP，最后写清单 | 必须先运行 `python scripts/publish_oss_release.py --directory <发布目录> --dry-run`；移除 `--dry-run` 会真实上传，只有在该产品发布授权、临时凭据和隔离验收均具备时才可执行 |
| `Prepare-DomesticRelease.ps1`、产品内构建脚本 | 部分产品的本地组包/清单准备 | 参数、输出和是否触网不一致；先阅读该项目文档和脚本 `--help`/源码，不要按名称推断功能 |

若产品没有已验证上传脚本，采用手工上传：由获授权的 OSS 操作者先上传新版本的完整 ZIP 与更新 ZIP，公网回下载验哈希，最后才上传该产品的客户端清单和适用的网站元数据。不要在未确认 bucket、prefix、兼容地址、缓存策略和回滚入口时执行手工上传。

<a id="verify-rollback"></a>

## 发布验证与回滚

### 最小验收记录

每次发布至少保存以下非敏感证据：

- 构建提交、产品版本、完整 ZIP / 更新 ZIP 文件名、字节数和 SHA-256；
- 公网 HTTPS URL、回下载哈希和清单内容；
- 下载站页面显示的版本与实际链接；
- 隔离旧版升级、`app.previous` 或该产品等价回滚证据；
- 用户数据、配置、日志和媒体未被包或替换动作触碰的证据；
- 未验证项，例如“未在另一台真实客户机跑完整业务”。

### 回滚原则

发布失败时，优先停止在尚未改变清单的阶段：保留旧清单和旧版，让旧版继续可用。若已经切换清单，只能在确认旧包仍可下载、旧清单内容可恢复且产品专属回滚规则已验证后，将**该产品的**清单恢复到此前已验证版本；不要删除不可变包、清空用户状态、覆盖用户安装目录或把其他产品的清单用于回滚。

网页回滚只恢复对应卡片/运行时元数据到已验证的下载 URL；它不能替代客户端清单回滚。严重风险也不应通过禁用旧版来解决，除非用户另有明确、产品专属的决定。

<a id="credentials"></a>

## 凭据、最小权限与安全获取

### GitHub

- 使用浏览器交互登录的 Git 凭据或 `gh auth login`；只授予推送该网站仓库分支、查询 Pages/Release 所需权限。
- 不提交个人访问令牌、Git Credential Manager 文件、SSH 私钥、浏览器 Cookie 或会话导出。
- 文档分支推送不等于网站发布；发布网页前再次确认 Pages 源仍是 `main`。

### OSS

- 优先使用操作者在 OSS 控制台的受控登录，或产品已配置的短时 STS/OIDC 凭据；不要把长期 AccessKey 写进脚本、命令历史、ZIP、仓库或聊天。
- 已核对的抖音发布脚本从当前进程读取 `ALIBABA_CLOUD_ACCESS_KEY_ID`、`ALIBABA_CLOUD_ACCESS_KEY_SECRET` 和 `ALIBABA_CLOUD_SECURITY_TOKEN`。这些变量名不是凭据本身；仅在经授权、最小范围的临时会话中设置。
- 权限应限制到目标 bucket 和产品 prefix 的上传、读取/HEAD 校验及必要清单写入；不授予无关 bucket 的删除、全局列举或权限管理。

### 禁止进入发布物的内容

不得读取、复制、提交、上传或打进安装包：真实 API Key、AccessKey、Token、Cookie、浏览器 profile、业务数据库、用户视频/文本、队列、日志、客户配置及内部会话资料。发现疑似敏感内容时停止该次发布，先在隔离构建目录处理，不要用删除用户数据的方式“清理”。

## 接手前的最后检查清单

- [ ] 已确认正在操作的是新建隔离构建/测试目录，而非用户安装目录。
- [ ] 已确认产品是否有自动更新、旧客户端兼容地址和用户选择逻辑。
- [ ] 已验证不可变包的公网下载与 SHA-256，再改任何清单。
- [ ] 已确认该网站卡片的真实运行时机制，而不是依据旧 catalog 或卡片文字猜测。
- [ ] 已分别验证客户端清单、下载站链接和隔离升级；没有把它们合并成一项结论。
- [ ] 已记录可恢复的旧清单/旧包 URL；未删除历史发布物或用户数据。
