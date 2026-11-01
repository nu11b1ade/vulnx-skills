# vulnx-skills

给 AI Agent 使用的轻量级 [ProjectDiscovery VulnX](https://github.com/projectdiscovery/vulnx) CLI Skill。

直接调用现有 `vulnx`，覆盖 CVE 查询、产品/厂商漏洞检索、KEV / EPSS / CVSS 筛选、PoC 与 Nuclei 模板可用性、时间范围、批量补全、聚合和 JSON 导出。只查漏洞情报，不把检索命中当作目标已存在漏洞的证明。

**版本基线：VulnX v2.0.2**（2026-11-01 核对的最新稳定版，发布于 2026-07-09）。Skill 使用英文指令以便不同 Agent 读取，要求用用户的语言回答。

## 轻量设计

- 安装内容只有 `SKILL.md`、按需加载的 `references/cli.md` 和可选的 `agents/openai.yaml`。
- 无 Python/Node 运行时依赖、CLI 包装器、常驻服务、MCP 配置或内置二进制。
- 核心流程直接用 CLI；额外语法和安装说明仅在需要时读取。
- 不保存 API Key，不自动更新已安装的 CLI，不自动执行 PoC 或 Nuclei 模板。

## 安装

先安装官方 VulnX CLI 并确保 Agent 的执行环境能在 PATH 找到它。可以下载[官方 Release](https://github.com/projectdiscovery/vulnx/releases/latest)中对应平台的二进制并核对校验和，或者使用 Go 1.24+：

```sh
go install github.com/projectdiscovery/vulnx/v2/cmd/vulnx@v2.0.2
vulnx version --disable-update-check
```

然后获取本仓库：

```sh
git clone https://github.com/nu11b1ade/vulnx-skills.git
```

将 **`skills/vulnx/` 整个目录**复制到所用 Agent 支持的 skills 目录，保留内部相对路径。具体安装目录和触发方式以该 Agent 文档为准；支持 `SKILL.md` 且具备终端执行能力的 Agent 可以使用。`agents/openai.yaml` 是可选的 UI 元数据，其他 Agent 可以忽略。

如果 Agent 运行在 Docker 或远程服务器中，CLI 和 Skill 都要安装在 **Agent 实际执行命令的环境**里。

VulnX 支持免 Key 查询，但受服务限流约束。需要认证时，在本机终端执行 `vulnx auth`，或通过 Agent 的密钥管理注入 `PDCP_API_KEY`；不要把 Key 写进 Skill 或提交到仓库。

## 使用示例

安装后可直接对 Agent 说：

- “用 vulnx 查 CVE-2021-44228，给出受影响版本、修复建议和来源。”
- “查 Tomcat 最近 30 天的高危漏洞，标注 KEV、EPSS 和公开 PoC 情况。”
- “按 ids.txt 补全所有 CVE，列出未查到的 ID，导出 JSON。”
- “查 Log4j 的全部漏洞，分页导出，不要只给第一页。”
- “统计已进入 KEV 的漏洞的严重程度分布。”

也可以让支持显式 Skill 调用的 Agent 使用 `$vulnx`。

## 处理的 CLI 注意点

- `--output` 只写 JSON；search/analyze 要求 `.json` 后缀，id/filters 换后缀也不会转 CSV，且均拒绝覆盖已有文件。
- 搜索默认仅返回 10 条；完整导出需要分页和去重。
- 单次 `id` 最多处理 100 个去重后的 ID；部分失败仍可能以成功状态退出。
- 单 ID 输出对象，多 ID 输出数组；搜索输出带 `results/count/total` 的对象。
- 零命中的搜索可能以状态 0 返回空 stdout，需要诊断确认。
- 字段投影可能让未返回的布尔字段显示为 false，不能据此断言不存在相关情报。

具体命令见 [Skill](skills/vulnx/SKILL.md) 和[按需参考](skills/vulnx/references/cli.md)。

## 验证

2026-11-01 已通过 Skill 格式、元数据和相对链接检查；使用校验和匹配官方发布资产的 v2.0.2 Linux amd64 CLI，完成真实 API 的字段发现、产品搜索、单 CVE 查询，以及本地模拟 API 的 14 项边界检查（分页参数、空结果、部分失败、100 ID 上限、输出格式和防覆盖等）。模拟检查验证的是 CLI 行为，不代表线上 API 的配额或数据完整性保证。

## 维护

升级基线时，先检查 [latest release](https://github.com/projectdiscovery/vulnx/releases/latest)，核对发布版 `--help` 和源码中的参数、JSON 格式、分页及错误行为，再更新 Skill 中的版本和核对日期。不要只修改版本号，也不要把主分支中尚未发布的能力写成稳定版能力。

## License

本仓库内容采用 [MIT License](LICENSE)。这是独立维护的社区 Skill，与 ProjectDiscovery 无官方隶属关系；VulnX 的代码和二进制由其上游项目独立分发。
