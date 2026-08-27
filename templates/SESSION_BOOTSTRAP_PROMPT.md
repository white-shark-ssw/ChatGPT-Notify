# ChatGPT-Notify 项目接入指令

把本文件直接提供给需要启用 Bark 完成通知的项目会话，并告诉它：**按本文件为当前项目完成通知接入。**

---

你现在需要为“当前正在开发的 GitHub 项目”接入统一 ChatGPT 完成通知。

通知中心已经固定存在，不需要向用户询问通知仓库地址或 Bark Key：

- Notification hub: `white-shark-ssw/ChatGPT-Notify`
- Hub URL: `https://github.com/white-shark-ssw/ChatGPT-Notify`
- Bootstrap: `https://github.com/white-shark-ssw/ChatGPT-Notify/blob/main/BOOTSTRAP.md`
- Notification PR: `#1`
- Protocol: `BARK_NOTIFY_V1`
- Bark group: `ChatGPT-Notify`（由通知中心强制设置）
- Default policy: **every final reply**

执行要求：

1. 先读取当前项目真实的 `AGENTS.md`、启动入口、项目状态文档和相关规则；不得猜测项目结构。
2. 读取通知中心的 `BOOTSTRAP.md`、`templates/CHATGPT_NOTIFY_RULES.md` 和 `templates/PROJECT_RULE_SNIPPET.md`。
3. 根据当前项目真实结构做最小接入：
   - 推荐新增 `docs/automation/CHATGPT_NOTIFY_RULES.md`；
   - 在当前项目**第一启动规则**（优先 `AGENTS.md`）中直接增加“每个面向用户的最终回答都必须通知”的强制条款，并要求读取详细通知规则；
   - 不能只依赖二级/间接引用。
4. `white-shark-ssw/ChatGPT-Notify` 与 PR `#1` 是固定值，不要改成业务项目仓库，也不要让用户重新提供。
5. 根据当前项目真实名称替换 `{{PROJECT_NAME}}`。
6. Bark Key 不属于业务项目，不要索取、读取、记录或写入 Bark Key。
7. 不修改与通知接入无关的源码、架构、构建配置或项目规则。
8. 如果当前项目规定重要规则变更必须同步项目文档，则按该项目自身规则同步；否则不要额外制造文档。
9. 完成后重新读取实际修改内容，确认新会话启动链路会自动加载通知规则。
10. 从接入完成后的后续**每一个最终回答**开始，在完整最终回答准备好之后、输出最终回答之前，向通知中心 PR #1 添加一条顶层 Conversation 评论：

```text
[BARK_NOTIFY_V1]
{"project":"<PROJECT_NAME>","title":"<当前会话标题或自然生成的任务标题>","body":"<最终回答开头的正文预览，建议 150–500 字符>","url":"https://chatgpt.com/"}
```

11. 创建评论成功后，必须读取工具返回的 comment id，并立即把同一条顶层评论更新为：

```text
<!-- ChatGPT-Notify transient payload redacted -->
```

12. 只有创建通知评论与立即擦除调用都完成后，才输出最终回答。若创建成功但擦除失败，最终回答中必须明确提醒用户检查通知 PR #1。

不要按任务重要性、耗时、是否改代码、回答长短或问答类型过滤。普通寒暄、简单事实问答、短澄清只要是最终回答，也必须通知。commentary / 中间进度 / 工具过程不是最终回答，不单独通知；每个用户轮次最多一次。

通知标题优先使用当前真实 ChatGPT 会话标题；如果当前环境无法可靠取得会话标题，则根据当前会话主题生成自然短标题，不得假装已读取 UI 标题。

通知正文应来自已经准备好的最终回答开头，保留必要换行，尽量让 Bark 展开后的阅读体验接近 ChatGPT 官方 iOS 推送，而不是只写一句“任务完成”。

业务项目不需要传 Bark 分组。通知中心会统一把所有 ChatGPT 项目通知设置为 `ChatGPT-Notify` 分组。为兼容旧规则，即使 payload 中仍包含 `group`，通知中心也会忽略其值。

通知中心 workflow 只监听 `issue_comment: created`。真实测试已经确认，创建后立即编辑评论不会破坏本次 Bark 推送；Action 会继续使用创建事件中的原始 payload。

Action 在 Bark 处理结束后会删除这条临时评论。因此正常情况下公开 PR 不会积累占位留言，但 GitHub Timeline 可能保留“Bot deleted a comment”的删除事件记录。

仓库规则更新不会自动注入已经打开的旧会话。若某会话在接入或规则更新前已经打开，需要一次性重新读取项目第一启动规则和 `docs/automation/CHATGPT_NOTIFY_RULES.md`；新会话应自动读取。

即时擦除仍不是绝对隐私保证，因此通知 payload 中禁止放入任何秘密或高敏感信息。

通知只表示“本轮完整回答已经准备输出”；不得把它错误描述成 CI、构建产物、真机测试或稳定性证据。

如果当前会话没有 GitHub 顶层 PR 评论写能力，不能伪造发送成功，也不能因此阻塞正常回答。
