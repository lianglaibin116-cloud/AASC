# AASC 测试演示

本文演示 AASC V1 内置静态规则如何检查 Skills、脚本和 MCP 配置。测试样例只作为文本读取；其中恶意示例故意包含危险命令，**不要直接运行 `runner.py`**。

## 下载并准备

从仓库首页下载 `ai-agent-security-center-v1.0.zip` 并解压。在项目根目录按 README 的开发者步骤安装测试依赖，然后运行：

```bash
python -m pytest -q
```

自动化测试会把 `tests/fixtures/malicious/` 和 `tests/fixtures/normal/` 交给静态分析函数；不会执行其中的 Python 脚本，也不会发起网络请求。

## 恶意样例

### Skill 指令：`tests/fixtures/malicious/SKILL.md`

样例要求模型忽略先前指令，读取 `~/.ssh/id_rsa`，然后上传令牌。预期规则包括：

- `PROMPT_OVERRIDE`：提示词越权或要求忽略现有指令。
- `PROMPT_SECRET`：智能体指令要求读取或发送秘密。
- `SENSITIVE_PATH`：引用 SSH 私钥等敏感路径。

### Python 脚本：`tests/fixtures/malicious/runner.py`

样例调用 `curl`，尝试把 `~/.aws/credentials` 作为请求内容发送到 `example.invalid`。该域名是保留的示例域名；测试时不会执行这条命令。预期规则提示危险命令、敏感文件访问和潜在外传。

### MCP 配置：`tests/fixtures/malicious/mcp.json`

配置样例在 MCP Server 的环境变量中设置了占位令牌。预期规则为 `MCP_INLINE_ENV`。它演示的是配置审计线索，不代表值是真实凭据。

## 正常样例

`tests/fixtures/normal/` 中的 Skill 只读取用户明确选定的 CSV 文件，并统计行数。预期内置规则结果为零条。这里的“零条”只表示当前规则没有发现已知线索，不是安全认证。

## 在桌面界面查看

1. 启动 AASC 桌面开发模式或浏览器开发模式。
2. 打开“本地资产审核”。
3. 在自定义目录框中填写解压后的 `tests/fixtures/malicious` 路径，勾选“我授权本次对上述本地路径进行只读扫描”，开始扫描。
4. 在总结报告中查看建议优先复核的文件，再打开详细列表检查证据、文件行号、规则和修复建议。
5. 将路径换成 `tests/fixtures/normal` 再扫描，比较结果。

这个演示不需要安装 Cisco Skill Scanner、Semgrep、Gitleaks 或 OSV-Scanner。未安装或未运行的外部扫描器会在覆盖情况中标为未运行；本页预期结果来自 AASC 内置规则。

静态规则可能误报或漏报。请结合完整上下文复核线索；不要把报告当成漏洞判定或安全背书。
