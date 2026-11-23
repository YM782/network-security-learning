# Network Security Learning（网络安全学习）

> A bilingual, source-linked curriculum from computer fundamentals to authorized security testing.
> 从计算机底层到授权安全测试的中英双语系统学习路线，保留每份资料的原始来源。

## Learning objective（学习目标）

Build the ability to explain how computers and networks work, analyze systems and traffic, understand vulnerabilities, test authorized lab targets, troubleshoot failures, and write professional reports.
目标是能够解释计算机和网络的工作原理，分析系统与流量，理解漏洞，在授权靶场中验证问题、排查故障并编写专业报告。

This is a defensive and educational learning repository. All scanning and exploitation exercises must stay within systems you own or have explicit permission to test.
本仓库用于防御性学习和安全教育。所有扫描与漏洞验证必须限定在自己拥有或明确获得授权的系统中。

## Repository map（仓库目录）

| Folder / file（英文名称） | 中文说明 |
|---|---|
| `01-Computer-Fundamentals/` | 计算机组成、CPU、内存、进程、线程和操作系统原理 |
| `02-Linux-Windows/` | Linux、Windows、Shell、权限、服务和日志 |
| `03-Computer-Networking/` | OSI、TCP/IP、DNS、ARP、路由、交换和 TLS |
| `04-Programming/` | Python、Bash、PowerShell、SQL 和 C 语言基础 |
| `05-Cybersecurity-Fundamentals/` | 安全原则、密码学、认证、风险和漏洞管理 |
| `06-Network-Analysis/` | Wireshark、Nmap、流量分析和资产识别 |
| `07-Web-Security/` | HTTP、API、身份认证、SQL 注入、XSS、SSRF 等 |
| `08-Penetration-Testing/` | 授权测试流程、范围管理、验证和报告 |
| `09-Active-Directory/` | Windows 域、Kerberos、LDAP、SMB 和域安全 |
| `10-Security-Labs/` | WebGoat、Juice Shop、GOAD 等隔离靶场 |
| `11-Troubleshooting/` | 系统、网络、虚拟机、Docker 和工具故障排查 |
| `SOURCES.md` | 原始 GitHub 项目、官方文档和课程来源 |
| `STUDY-ROADMAP.md` | 12 个月学习路线和阶段验收 |

The folders above are the planned structure. Add original notes, lab reports, and troubleshooting records as each stage is completed.
以上为规划目录；随着学习进度逐步添加原创笔记、实验报告和故障记录。

## Learning principles（学习原则）

1. Understand the mechanism before memorizing commands（先理解原理，再记忆命令）.
2. Pair each theory topic with a reproducible lab（每个理论主题都配一个可复现实验）.
3. Record expected output, actual output, and troubleshooting steps（记录预期输出、实际输出和排错过程）.
4. Prefer official documentation and original repositories（优先官方文档和原始仓库）.
5. Never test third-party systems without explicit authorization（未经明确授权，不测试第三方系统）.

## Lab safety（实验安全）

- Use VMware host-only or another isolated virtual network for vulnerable machines（脆弱靶机使用 VMware Host-only 等隔离网络）.
- Take snapshots before risky changes（高风险操作前创建快照）.
- Do not expose WebGoat, Juice Shop, GOAD, or other intentionally vulnerable labs to the public internet（不要把故意存在漏洞的靶场暴露到公网）.
- Use only test accounts and synthetic data（只使用测试账户和虚构数据）.
- Keep credentials, tokens, subscription links, and private keys out of Git（不要把密码、令牌、订阅链接和私钥提交到 Git）.

## Start here（从这里开始）

1. Read [STUDY-ROADMAP.md](STUDY-ROADMAP.md)（学习路线）.
2. Review [SOURCES.md](SOURCES.md)（资料来源）.
3. Begin with computer fundamentals and Linux, not exploitation tools（从计算机基础和 Linux 开始，不要从漏洞利用工具开始）.
