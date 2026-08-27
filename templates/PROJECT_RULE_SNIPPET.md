# 项目启动规则接入片段

把下面内容合并进目标项目自己的 `AGENTS.md` 或等价启动规则，并将 `{{PROJECT_NAME}}` 替换为真实项目短名称。

```text
## ChatGPT Completion Notification

本项目启用项目级完成通知。

每次新开发会话开始时，除项目原有启动资料外，还必须读取：

`docs/automation/CHATGPT_NOTIFY_RULES.md`

通知配置：

- Hub repository: white-shark-ssw/ChatGPT-Notify
- Channel PR: #1
- Project name: {{PROJECT_NAME}}

完成通知规则不得替代或降低本项目原有开发、验证、文档和安全要求。
```

然后把 `templates/CHATGPT_NOTIFY_RULES.md` 复制到目标项目：

```text
docs/automation/CHATGPT_NOTIFY_RULES.md
```

并把其中全部 `{{PROJECT_NAME}}` 替换成真实项目名称。
