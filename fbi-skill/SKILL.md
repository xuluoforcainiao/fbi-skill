---
name: fbi-skill
version: 0.11.0
description: FBI 数据分析综合技能，通过 ChatBI API 提供报表搜索、报表解读、数据问答分析、T+1 日报生成五大能力。适用：搜索报表、解读报表、分析指标(UV/PV/GMV等)、数据趋势分析、异动归因、快速问数、基于 FBI 真实数据生成日报。根据用户意图自动路由到对应功能。
install_source: aone-kit
install_method: cli
name_zh: FBI数据分析洞察
---

# FBI 数据分析综合技能

## 执行约束

> **重要**：所有脚本均为黑盒工具，严格按照本文档和对应 reference 中定义的命令格式执行。

### 全局硬规则

- 禁止读取 `scripts/` 目录下的 Python 源码
- 禁止基于源码理解尝试重新实现或模拟脚本逻辑
- 禁止直接构造 HTTP 请求替代脚本执行
- 禁止修改脚本文件内容
- **禁止单独执行非入口脚本**（如 `credential.py`、`utils.py`、`workspace.py`），这些是内部模块，不是独立工具
- **禁止自行添加文档中未列出的命令行参数**（如 `--emp-id`、`--user-id`、`--token` 等）
- `/path/to/skill/` 替换为实际安装路径，**保持工作目录在 workspace root**

### 失败处理

脚本执行失败时：

1. 检查命令参数是否正确
2. 检查前置步骤是否完成
3. 将完整的错误输出（`stdout + stderr`）报告给用户
4. 等待用户指示，不要自行重试超过 1 次

### Python 环境

> 本技能所有脚本均为 Python 编写，执行前必须确保当前环境已安装 Python 3。
> 可通过 `python3 --version` 或 `python --version` 检查。
> 若未安装，请先安装 Python 3，并通过 `pip install -r requirements.txt` 安装依赖。

### 用户身份与权限

> 当前用户的工号、花名等信息由脚本内部通过凭证服务自动获取。
> 所有数据查询**仅限当前登录用户自身**。
> 若用户要求查询其他人的数据，直接告知“仅支持查询自己的数据”，然后结束流程。

### 链接处理

- 脚本输出中的 URL（报表链接、在线报告、ChatBI 执行过程等）均为**参考链接**
- **禁止使用浏览器或任何自动化工具打开这些链接**
- 仅在回复末尾以 Markdown 参考链接形式展示给用户

## 长轮询硬规则

以下规则属于**不可退化主协议**，即使细节下沉到 `references/`，这里仍然始终生效：

- 对日报 / 长轮询查询任务，启动脚本后的默认动作只有一个：**持续等待脚本自然结束**
- 只要涉及日报 / 长轮询查询脚本，默认都应显式按 **1800 秒（30 分钟）** 的等待窗口执行
- 只要已经拿到 `answerMessageId`，默认就应立即进入：`query_chat_result.py "<answerMessageId>" --wait --timeout-seconds 1800`
- 拿到 `answerMessageId` 后，**禁止** agent 自己在外层 `sleep` 30/60/90 秒后，再多次重复调用查询脚本来模拟轮询
- 只要脚本仍在持续输出轮询进度，就继续等待，**不要**自行终止、`kill`、`stop`、`cleanup`
- 默认只在明确终态（成功 / 失败 / 本次等待窗口结束）汇报
- 失败处理只包括记录错误原因和发送提醒，不包括额外清理动作
- 默认只允许**一条前台主线**，不要并行派生第二条查询链路，不要重复创建任务
- 非显式开启 `accept_partial_result: true` 时，`reportUrl`、`finalAnswer` 等中间可用结果只表示“已有进展”，**不等价于应立即结束等待或对外汇报**
- 在脚本仍持续输出等待进度时，**禁止**切换到浏览器、网页工具或其他 fallback 路径

## 意图路由

根据用户问题判断使用哪种功能模式：

| 功能模式 | 触发条件 | 下一步 |
|---|---|---|
| **报表搜索** | 搜索/查找/列出报表、我的报表、有权限的报表 | 读取 `references/report_search.md` |
| **报表解读** | 有报表 ID 或报表名称 + 整体解读/生成报告/ppt/文档 | 读取 `references/chatbi_analysis.md` |
| **数据问答分析** | 无特定报表，通用数据问题/趋势/异动/问数 | 读取 `references/chatbi_analysis.md` |
| **日报生成** | 固定报表清单、每天执行一次、生成 T+1 日报 | **先读取** `references/daily_report.md`，如涉及等待再读取 `references/chatbi_polling.md` |

**优先级规则（当条件重叠时）：**

1. 报表 ID + 具体指标名称 -> 指标巡检 / 报表解读
2. 报表 ID + 整体解读 / 生成报告 -> 报表解读
3. 有报表 ID 但无其他明确意图 -> 报表解读
4. 无报表 ID -> 通用数据分析

## 按需读取规则

为避免上下文腐烂，**不要一次性把所有 reference 全部读入上下文**。按命中模式加载：

- 报表搜索：只读取 [report_search.md](references/report_search.md)
- 报表解读 / 通用数据分析：先读取 [chatbi_analysis.md](references/chatbi_analysis.md)
- 只要进入 `answerMessageId` 轮询、长等待、pending 判断等环节：补充读取 [chatbi_polling.md](references/chatbi_polling.md)
- 只要进入日报模式：**执行前必须读取** [daily_report.md](references/daily_report.md)
- 环境错误、凭证错误、版本问题：按需读取 [troubleshooting.md](references/troubleshooting.md)

## 入口脚本索引

仅以下脚本可作为入口使用：

- `python /path/to/skill/scripts/search_report.py --keyword "<关键词>"`
- `python /path/to/skill/scripts/search_report.py --url "<URL>"`
- `python /path/to/skill/scripts/find_my_report.py permission`
- `python /path/to/skill/scripts/find_my_report.py favorite`
- `python /path/to/skill/scripts/find_my_report.py visit`
- `python /path/to/skill/scripts/chat_bi_session.py [会话名称]`
- `python /path/to/skill/scripts/chat_bi_session.py --clear`
- `python /path/to/skill/scripts/create_chat.py "<问题>"`
- `python /path/to/skill/scripts/query_chat_result.py "<answerMessageId>"`
- `python /path/to/skill/scripts/generate_daily_report.py`
- `python /path/to/skill/scripts/init_daily_reports_config.py`

## 模式入口说明

### 报表搜索

- 只做“找报表”，不做数据分析
- 返回报表 ID 和元信息
- 详细流程、输出规范与注意事项见 [report_search.md](references/report_search.md)

### 报表解读 / 数据问答分析

- 通过 ChatBI 会话 + `create_chat.py` 提交问题
- 通过 `answerMessageId` 获取结果
- 若需要轮询或长等待，补充读取 [chatbi_polling.md](references/chatbi_polling.md)
- 详细流程见 [chatbi_analysis.md](references/chatbi_analysis.md)

### 日报生成

- 日报模式不是“拿到请求就直接跑脚本”，而是“先确认 workspace 配置，再执行日报主线”
- **执行前必须先读取** [daily_report.md](references/daily_report.md)
- 只要涉及 `answerMessageId`、长等待、pending 判断、禁止动作，**还必须读取** [chatbi_polling.md](references/chatbi_polling.md)

## 输出规范

脚本返回中可能包含在线报告 URL 和 ChatBI 执行过程 URL，默认按以下格式展示：

```markdown
（正文：分析结论 / 巡检结论 / 日报结果）

---

**参考链接**

- [在线报告](https://fbi.alibaba-inc.com/...)
- [ChatBI 执行过程](https://fbi.alibaba-inc.com/fbi/chatbi/chat.htm?...)
```

## 环境与故障排查

- 凭证错误、会话异常等问题，按需读取 [troubleshooting.md](references/troubleshooting.md)
- 若 reference 中已有明确处理方式，以 reference 为准
