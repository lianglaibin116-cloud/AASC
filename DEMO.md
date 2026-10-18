# AASC 测试演示

本文演示 AASC V1 的内置静态规则如何检查 Skills、脚本和 MCP 配置。样例只作为文本读取；恶意示例中的命令请勿运行。

## 运行自动化测试

下载并解压仓库中的 `ai-agent-security-center-v1.0.zip`，按 README 的开发者步骤安装测试依赖后，在项目根目录运行：

```bash
python -m pytest -q
```

测试把 `tests/fixtures/malicious/` 和 `tests/fixtures/normal/` 交给静态分析函数，不执行其中的 Python 脚本，也不发起网络请求。

## 恶意样例

- `tests/fixtures/malicious/SKILL.md`：含有忽略先前指令、读取 `~/.ssh/id_rsa` 并上传令牌的文本。预期发现 `PROMPT_OVERRIDE`、`PROMPT_SECRET`、`SENSITIVE_PATH` 等线索。
- `tests/fixtures/malicious/runner.py`：文本中包含使用 curl 读取 `~/.aws/credentials` 并发送到保留示例域名 `example.invalid` 的代码。只扫描，不要执行。
- `tests/fixtures/malicious/mcp.json`：MCP 配置包含占位符环境令牌，预期发现 `MCP_INLINE_ENV` 配置线索。

## 正常样例

`tests/fixtures/normal/` 中的 Skill 仅读取用户明确选择的 CSV 并统计行数。预期当前内置规则没有风险线索；这不代表安全认证。

## 在桌面界面演示

1. 启动 AASC 桌面开发模式或浏览器开发模式。
2. 打开“本地资产审核”，自定义路径填写解压后的 `tests/fixtures/malicious`。
3. 勾选只读授权并开始扫描。先阅读总结报告，再在详细列表查看文件、行号、证据和修复建议。
4. 将路径换成 `tests/fixtures/normal` 再扫描，对比结果。

演示不要求安装 Cisco Skill Scanner、Semgrep、Gitleaks 或 OSV-Scanner。结果来自 AASC 内置规则，外部扫描器会如实显示为未运行。静态规则可能误报或漏报，请复核原文上下文；不要把报告当成漏洞判定或安全背书。
