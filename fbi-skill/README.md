# fbi-skill

FBI 数据分析综合技能，通过 ChatBI API 提供报表搜索、报表解读、数据问答分

## 安装

使用 GitHub CLI 安装：

```bash
gh skill install xuluoforcainiao/fbi-skill fbi-skill
```

## 使用

安装后，在 QoderWork 中提及或使用此 skill 时会自动激活。

你也可以通过以下方式手动触发：

```
请使用 fbi-skill skill 来帮我...
```

## 文件结构

```
fbi-skill/
├── SKILL.md          # Skill 定义（核心）
├── config.yaml
├── daily_reports.yaml
├── package.json
├── requirements.txt
└── scripts/
    └── chat_bi_session.py
    └── create_chat.py
    └── credential.py
    └── find_my_report.py
    └── generate_daily_report.py
    └── init_daily_reports_config.py
    └── query_chat_result.py
    └── search_indicator.py
    └── search_report.py
    └── utils.py
    └── workspace.py
```

## 更新日志

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0.0 | 2026-05-21 | 初始发布 |

## 许可证

MIT

## 作者

[xuluoforcainiao](https://github.com/xuluoforcainiao)
