# 项目启动规则接入片段

把下面内容合并进目标项目自己的 `AGENTS.md` 或等价**第一启动规则**，并将 `{{PROJECT_NAME}}` 替换为真实项目短名称。

```text
## ChatGPT Completion Notification

本项目启用项目级完成通知。

每次新会话开始时，除项目原有启动资料外，还必须读取：

`docs/automation/CHATGPT_NOTIFY_RULES.md`

**每一个面向用户的最终回答都必须触发一次完成通知。** 不得按任务重要性、耗时、是否修改代码、回答长度或问答类型过滤。普通寒暄、简单事实问答和短澄清只要是最终回答也通知；中间进度/工具过程不单独通知；每个用户轮次最多一次。

通知配置：

- Hub repository: white-shark-ssw/ChatGPT-Notify
- Channel PR: #1
- Project name: {{PROJECT_NAME}}
- Bark group: ChatGPT-Notify（通知中心强制设置）

最终回答的执行顺序必须是：完成工作与验证 → 准备最终回答 → 按详细规则创建通知评论并立即擦除 → 再输出最终回答。

完成通知规则不得替代或降低本项目原有开发、验证、文档和安全要求。
```

然后把 `templates/CHATGPT_NOTIFY_RULES.md` 复制到目标项目：

```text
docs/automation/CHATGPT_NOTIFY_RULES.md
```

并把其中全部 `{{PROJECT_NAME}}` 替换成真实项目名称。

已经打开且早于该规则的旧会话不会自动获得仓库更新，需要一次性重新读取当前项目第一启动规则和 `docs/automation/CHATGPT_NOTIFY_RULES.md`。
