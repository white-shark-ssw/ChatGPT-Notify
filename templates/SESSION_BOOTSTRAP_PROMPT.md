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

执行要求：

1. 先读取当前项目真实的 `AGENTS.md`、启动入口、项目状态文档和相关规则；不得猜测项目结构。
2. 读取通知中心的 `BOOTSTRAP.md`、`templates/CHATGPT_NOTIFY_RULES.md` 和 `templates/PROJECT_RULE_SNIPPET.md`。
3. 根据当前项目真实结构做最小接入：
   - 推荐新增 `docs/automation/CHATGPT_NOTIFY_RULES.md`；
   - 在当前项目真实启动规则中增加每次新开发会话必须读取该通知规则的要求。
4. `white-shark-ssw/ChatGPT-Notify` 与 PR `#1` 是固定值，不要改成业务项目仓库，也不要让用户重新提供。
5. 根据当前项目真实名称替换 `{{PROJECT_NAME}}`。
6. Bark Key 不属于业务项目，不要索取、读取、记录或写入 Bark Key。
7. 不修改与通知接入无关的源码、架构、构建配置或项目规则。
8. 如果当前项目规定重要规则变更必须同步项目文档，则按该项目自身规则同步；否则不要额外制造文档。
9. 完成后重新读取实际修改内容，确认新会话启动链路会自动加载通知规则。
10. 从接入完成后的后续重要任务开始，在完整最终回答准备好之后、输出最终回答之前，向通知中心 PR #1 添加一条顶层 Conversation 评论：

```text
[BARK_NOTIFY_V1]
{"project":"<PROJECT_NAME>","title":"ChatGPT · <PROJECT_NAME>","body":"<本轮完成内容的一句话摘要>","group":"ChatGPT","url":"https://chatgpt.com/"}
```

通知只表示“本轮完整回答已经准备输出”；不得把它错误描述成 CI、构建产物、真机测试或稳定性证据。

如果当前会话没有 GitHub 顶层 PR 评论写能力，不能伪造发送成功，也不能因此阻塞正常回答。
