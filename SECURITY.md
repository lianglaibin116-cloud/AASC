# Security Policy

## Reporting a vulnerability

Please report suspected vulnerabilities privately through GitHub's **Report a vulnerability** feature on this repository's Security tab. Do not include credentials, private scan reports, or customer data in a public issue.

Include the affected version or commit, platform, steps to reproduce, and observed impact. Please allow maintainers reasonable time to investigate before public disclosure.

## Scope

AASC V1 performs static and configuration auditing. It does not provide runtime monitoring, a dynamic sandbox, an MCP gateway, or automatic blocking. Findings can be false positives or miss issues; they are not a security certification.

The application is designed for local processing. Optional scanner CLIs are separate programs installed by the user; their network behavior depends on their versions and configuration. OSV-Scanner requires explicit opt-in to online vulnerability queries.
