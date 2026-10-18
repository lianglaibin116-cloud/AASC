# AI Agent Security Center V1.0

**仓库简称：`AASC`** · 本地优先的 AI Agent Skills 与 MCP 静态审计桌面工具。

[![Desktop installers](https://github.com/lianglaibin116-cloud/AASC/actions/workflows/desktop-release.yml/badge.svg)](https://github.com/lianglaibin116-cloud/AASC/actions/workflows/desktop-release.yml) · [MIT License](LICENSE) · [安全报告方式](SECURITY.md)

AI Agent Security Center（仓库简称 **AASC**）用于在安装或使用 AI Agent 工具前检查 Skills、MCP 配置及相关脚本。它帮助开发者和普通用户回答几个实际问题：这个 Skill 有没有要求 AI 忽略既有规则？脚本会不会读取 SSH 密钥、云凭据或环境变量？是否存在把敏感数据发往外部地址的代码？本机已经配置了哪些 Skills 和 MCP Server？

V1 是可运行的本地优先静态审计桌面应用。你可以输入公开 GitHub 仓库地址与完整 commit SHA，在代码进入本机前审查指定版本；也可以在明确授权后，只读扫描本机 Claude Code、Codex、Cursor 的 Skills 与 MCP 配置。程序不会执行待审计仓库中的脚本，也不会自动安装仓库内容。

扫描结果分成总结报告和详细列表：总结报告指出建议先复核的文件与扫描器覆盖情况；详细列表给出每条线索的证据、文件、行号、级别、排序依据和修复建议。项目还会记录资产、SHA-256 文件哈希及扫描历史，方便复查版本变化。静态规则提供的是待核实线索，不是恶意结论或安全认证。

桌面界面由 Vue 3 与 Tauri 2 提供，扫描 API 使用 FastAPI，历史记录存入 SQLite。安装包构建工作流支持 macOS Apple silicon、macOS Intel 和 Windows 11 x64。

V1 **仅做静态与配置审计**。没有动态沙箱、Sysmon、MCP 网关、运行时监控或自动阻断。未发现风险不等于安全认证。

## 桌面发行版

正式桌面版由 Tauri 2 承载 Vue 界面，并将 FastAPI 后端打包为随应用启动的本机 sidecar。使用者打开桌面程序即可工作，不需要安装 Python、Node.js，也不需要手动打开浏览器或启动后端。应用关闭时会结束后端进程。它是一个桌面审计应用，会检查 MCP 配置；它本身不是 MCP Server。

GitHub Actions 已配置为构建以下安装文件：

- macOS Apple silicon：`.dmg`
- macOS Intel：`.dmg`
- Windows 11 x64：NSIS `.exe` 安装程序

向 GitHub 仓库推送 `v*` 格式的版本标签后，Actions 会构建三个平台，并创建 GitHub Release 附件。手动运行 `Desktop installers` workflow 会生成可下载的 Actions 构建产物，不创建 Release。源码仓库中没有提交各平台二进制；这些文件由各平台的 CI runner 分别编译。

当前 workflow 未配置代码签名证书，生成的安装文件为未签名版本。Windows 可能显示 SmartScreen 提示；macOS 会显示 Gatekeeper 首次打开提示。正式面向公众发布时，应配置 Windows 代码签名证书，以及 Apple Developer ID 的签名与公证凭据。

### 从源码构建桌面安装包

需要 Python 3.12、Node.js 22、Rust stable。Windows 11 还需要 [Tauri 的 Windows 前置条件](https://v2.tauri.app/start/prerequisites/)，包括 Microsoft C++ Build Tools 和 WebView2；macOS 需要 Xcode Command Line Tools。

在项目根目录执行：

```bash
python -m pip install -r backend/requirements-desktop.txt
python scripts/build_backend_sidecar.py
cd frontend
npm ci
npm run tauri:icon
npm run tauri:build
```

Windows PowerShell 将路径分隔符改成反斜杠，例如 `backend\requirements-desktop.txt`、`scripts\build_backend_sidecar.py`。Windows 安装包位于 `frontend/src-tauri/target/release/bundle/nsis/`；macOS DMG 位于 `frontend/src-tauri/target/release/bundle/dmg/`。sidecar 必须在目标操作系统和芯片架构上构建；不要把 macOS 构建出的 sidecar 用于 Windows，反之亦然。

### 桌面开发模式

`npm run tauri:dev` 用于开发 UI，仍连接本机 `127.0.0.1:8000` 上手动启动的 FastAPI 服务。按下面的浏览器开发步骤启动后端，再从 `frontend` 目录运行 `npm run tauri:dev`。发行版则会自行启动打包的后端。

## 浏览器开发模式（仅供本地开发预览）

先安装 Python 3.11 或更新版本、Node.js 20.19+、Git。将本项目放在本机任意工作目录，在 PowerShell 中进入项目根目录：

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r backend\requirements.txt
cd frontend
npm ci
npm run build
cd ..
.\.venv\Scripts\python.exe -m uvicorn security_center.app:app --app-dir backend --host 127.0.0.1 --port 8000
```

浏览器打开 `http://127.0.0.1:8000`。服务只监听本机回环地址。前端已构建后，由 FastAPI 同端口提供页面；不必再开第二个开发服务器。第一次安装依赖需要网络，日常扫描本地目录不需要网络。

如果 Windows 的 Python 命令是 `py -3.12` 或 `python`，相应替换第一行。安装失败时先检查 Python、Node 与网络代理。请勿对待审计的仓库运行 `pip install`、`npm install`、PowerShell 安装脚本或 `git submodule update`。

### 使用方式

1. **GitHub 安装前审核**：输入 `https://github.com/owner/repo` 和 40 位 commit SHA。可在 GitHub 提交页面复制完整 SHA。扫描器经 GitHub API 核对 SHA，再从 `codeload.github.com` 下载该提交 ZIP；临时网络错误最多重试 3 次。限制下载及解压大小，拒绝路径穿越和链接，结束后删除临时副本。只支持公开仓库，不接收访问令牌；GitHub 未登录 API 的速率限制仍适用。
2. **本地审核**：打开“本地资产审核”，逐行填写要审计的自定义目录或文件；默认只扫描这些路径。若还要扫描自动发现的常见路径，勾选“同时扫描自动发现的常见路径”。勾选本次只读授权后才会开始扫描；结果页会列出实际扫描的根路径。常见路径包括 `~/.claude/skills`、`~/.claude/settings*.json`、`~/.claude.json`、`~/.codex/skills`、`~/.codex/config.toml`、`~/.cursor/skills`、`~/.cursor/mcp.json` 和 Claude Desktop 的 macOS/Windows MCP 配置。重叠目录只扫描一次，符号链接不跟随。
3. **结果**：默认先看“总结报告”，再按需要打开“详细列表”。总结页给出建议先看的文件、优先级数量、线索类别和扫描器覆盖；点击文件或优先级可进入对应明细。详细页可按优先级筛选、搜索，并查看逐条证据、行号、排序依据和修复建议。资产清单与文件哈希也在详细页。可分别下载总结 Markdown 与完整明细 CSV。

### 如何理解报告

| 分组 | 含义 | 建议动作 |
| --- | --- | --- |
| 优先复核 | 命中较强规则，或线索出现在有效的 Skill/智能体指令、可执行文件中；同一文件可能同时涉及敏感路径与对外通信 | 先阅读原文和调用链，核实是否真的读取或发送敏感数据 |
| 常规复核 | 有风险线索，但需要更多上下文才能判断 | 结合文件用途、参数来源和配置权限核实 |
| 参考信息 | 多数是文档、示例或测试文件中的静态命中 | 后续检查；仍需确认是否被实际执行或复制到生产环境 |

这个顺序由文件位置、规则和同文件线索自动生成，**不是漏洞已确认的结论**，也不是“参考信息可忽略”的保证。规则严重级别与复核优先级分别显示；外部扫描器未安装、未选择或运行失败仍会明确列在覆盖情况中。打开旧的历史记录也会按当前规则生成双层报告，无须重新扫描。

**报告来源**：每次扫描都会自动运行程序内置的静态规则，因此即使 Cisco Skill Scanner、Semgrep、Gitleaks、OSV-Scanner 都未运行，也可能产生报告。总结报告会列出内置规则与外部扫描器各贡献了多少条线索；“扫描器覆盖”先显示内置规则已运行，再显示四个可选外部工具的状态。外部工具标为“未运行/未安装”时，它的专用检测能力没有覆盖本次扫描，也没有贡献这些线索。“未安装”是指后端进程未在 `PATH` 中找到对应命令。

扫描历史存于 Windows `%LOCALAPPDATA%\AI Agent Security Center\history.sqlite3`；macOS 存于 `~/Library/Application Support/AI Agent Security Center/history.sqlite3`。原始仓库内容不写入数据库；证据先脱敏再保存。发现项仍可能包含不完整脱敏的信息，请按本机敏感数据管理数据库。

## 可选外部扫描器及许可证

两个扫描页面均可逐个勾选已安装的外部 CLI。未选择或未安装都会明确显示 **“未运行”**；解析失败、超时显示 **“运行失败”**，不会算成通过。本地目录的外部扫描先复制已审计的常规文件到临时目录，不对原文件运行 CLI。所有外部命令均以参数数组启动，没有 Shell 拼接。

| 扫描器 | 上游许可证 | 可执行文件 | V1 调用方式 | 联网规则 |
| --- | --- | --- | --- |
| Cisco Skill Scanner | Apache-2.0 | `skill-scanner` | 对每个 `SKILL.md` 所在目录执行 `scan --format json` | 不启用 LLM、VirusTotal、AI Defense；清除相关 API Key 环境变量 |
| Semgrep CLI | LGPL-2.1（具体版本可能含其他文件级声明） | `semgrep` | 内置本地规则 `semgrep-rules.yml`、JSON 输出、关闭指标上报 | 不下载远程规则 |
| Gitleaks | MIT | `gitleaks` | `dir` 模式、JSON 输出、秘密值脱敏 | 无需访问 Git 历史 |
| OSV-Scanner | Apache-2.0 | `osv-scanner` | `scan source -r --format json` | **必须单独勾选同意联网查询依赖** |

这些工具由用户自行安装，不随 AASC 应用打包。AASC 只调用本机 CLI；工具本身及其使用仍受各自许可证约束。应用依赖的许可证与分发提示见 [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md)。

安装示例：Cisco 可用 `pipx install cisco-ai-skill-scanner`；Semgrep 可用 `pipx install semgrep`；Gitleaks 与 OSV-Scanner 请从各自官方发布页安装与系统架构匹配的版本，并把可执行文件加入 `PATH`。安装任何安全工具前仍需核验发布来源和版本。外部 CLI 是本机执行的程序，可能受其自身版本与配置影响；仅对你信任的安装版本启用。

## 桌面版接口与日志

发行版首次启动会由 Tauri 启动 sidecar，并在本次应用会话内分配随机 loopback 端口和 API 令牌。后端只绑定 `127.0.0.1`，所有 API 请求都需要令牌；应用退出时结束 sidecar 进程组。后端日志写入数据库所在应用数据目录的 `backend.log`，可用于排查启动问题。WebView 与后端的本机通信不会把扫描内容发往第三方。

## macOS / 开发者运行

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r backend/requirements-dev.txt
cd frontend && npm ci && npm run build && cd ..
.venv/bin/python -m pytest -q
.venv/bin/python -m uvicorn security_center.app:app --app-dir backend --host 127.0.0.1 --port 8000
```

`GET /api/health` 返回运行状态。前端 `npm run dev` 可在 `http://127.0.0.1:5173` 运行，但仍需先启动后端。API 文档在 `/docs`。

## 测试演示

仓库附有一组正常样例和一组刻意构造的风险样例，位于源码包的 `tests/fixtures/`。它们用于演示静态规则能发现什么，也作为自动化测试输入。你可以下载[源码包](ai-agent-security-center-v1.0.zip)，解压后按上面的开发者步骤安装测试依赖，再运行 `python -m pytest -q`。逐步演示、样例说明和预期结果见 [DEMO.md](DEMO.md)。

演示用例包括：

| 样例 | 内容 | 预期结果 |
| --- | --- | --- |
| `tests/fixtures/normal/` | 只读取用户明确选定的 CSV，并统计行数 | 内置规则不应报出风险线索 |
| `tests/fixtures/malicious/SKILL.md` | 指令要求忽略先前规则、读取 SSH 私钥并上传令牌 | 检出提示词越权、秘密读取/传送及敏感路径线索 |
| `tests/fixtures/malicious/runner.py` | 示例命令尝试用 `curl` 读取云凭据并向保留示例域名发送 | 检出危险命令、敏感文件访问和潜在外传线索 |
| `tests/fixtures/malicious/mcp.json` | MCP 环境配置包含占位符令牌 | 检出内联环境变量配置线索 |

也可以在桌面 UI 的“本地资产审核”里填入解压后的 `tests/fixtures/malicious`，勾选只读授权后扫描；再扫描 `tests/fixtures/normal` 对比结果。外部扫描器无需安装，内置规则就能完成这个演示；覆盖面会如实标明各外部工具未安装或未运行。

风险样例中的命令是静态分析用的测试文本，**请勿运行 `runner.py`**。自动化测试只读取并分析这些文件，不执行样例内容。不同规则版本可能调整线索名称或分级；误报和漏报均有可能，仍需阅读原文上下文。

## 安全边界与检测范围

- 仅允许标准 GitHub HTTPS 地址与完整 commit SHA；拒绝 URL 中的凭据、端口、查询参数和非 GitHub 域名。不会检出 submodule，也不会执行仓库脚本。ZIP 下载上限 50 MiB，解压总量上限 100 MiB，文件数量上限 5000，单文件上限 20 MiB；静态分析单文件上限 2 MiB。
- 内置规则检查提示词越权、秘密读取与传送、敏感路径、网络发送、危险命令和脚本执行入口。MCP JSON 会盘点服务器名称，并提示内联环境变量。规则是线索，不做恶意意图的最终判定。
- 远程内容只作为数据处理。页面以 Vue 文本插值显示证据，不渲染仓库 Markdown/HTML。数据库只存脱敏发现、路径、哈希和元数据，不存原始文件。扫描服务没有任何“安装”或“执行仓库代码”的 API。
- 外部扫描器作为本机程序运行；V1 没有操作系统级网络隔离，故不能证明它们绝对不会联网。默认不勾选外部扫描器，OSV 另需联网同意。需要严格零出站环境时，请只用内置规则并在系统防火墙中限制进程出站。
- 本机 API 绑定回环地址，写入扫描请求需自定义请求头，且只接受本机页面或 Tauri 来源。不要把服务改绑到公网地址。

## 许可证与第三方工具

本项目自有代码和文档采用 MIT 许可证，具体文本见 [LICENSE](LICENSE)。外部扫描器是用户单独安装的程序，不随应用打包；各工具许可证、依赖说明及分发边界见 [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md)。使用或分发这些外部程序时，仍需遵守它们各自的许可证。

安全问题请通过 GitHub 仓库的 **Security → Report a vulnerability** 私下报告，详见 [SECURITY.md](SECURITY.md)。

## 项目结构

```text
backend/security_center/  FastAPI、拉取/静态扫描、SQLite、外部扫描器接入
backend/launcher.py      桌面版 FastAPI 入口
frontend/src/             Vue 3 页面
frontend/src-tauri/       Tauri 2 桌面窗口、sidecar 和图标
scripts/build_backend_sidecar.py  当前平台的 Python 后端打包脚本
.github/workflows/        macOS/Windows 安装包与后端测试工作流
tests/fixtures/           正常与恶意样例（仅供静态测试）
tests/test_core.py        单元及 API 授权测试
tests/test_reporting.py   报告分组、导出和历史兼容测试
```

## V2 预留

动态沙箱、Sysmon 事件采集、MCP 网关与阻断需独立设计权限模型、事件存储与审批流程。V1 数据模型保留项目、扫描、版本、文件哈希和扫描器状态，可以作为后续关联基础；目前没有采集或拦截运行时行为。

## 参考

- [Tauri 2 Windows 前置条件](https://v2.tauri.app/start/prerequisites/)；[Tauri sidecar 文档](https://v2.tauri.app/develop/sidecar/)
- [Cisco Skill Scanner CLI](https://github.com/cisco-ai-defense/skill-scanner/blob/main/docs/reference/cli-command-reference.md)
- [Gitleaks `dir` 模式](https://github.com/gitleaks/gitleaks/blob/master/README.md)
- [OSV-Scanner 源码扫描](https://github.com/google/osv-scanner/blob/main/docs/scan-source.md)
