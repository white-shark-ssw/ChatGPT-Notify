# ChatGPT Notify — Project Bootstrap

本文件用于把项目级 Bark 完成通知接入任意 GitHub 项目。

通知中心固定配置：

- Hub repository: `white-shark-ssw/ChatGPT-Notify`
- Hub URL: `https://github.com/white-shark-ssw/ChatGPT-Notify`
- Channel PR: `#1`
- Protocol: `BARK_NOTIFY_V1`

## 给项目会话的执行要求

当用户要求“为当前项目接入 ChatGPT-Notify”时：

1. 先读取当前项目已有的 `AGENTS.md`、项目启动规则和文档规则，确认真实启动入口。
2. 不猜测目录结构；只在确认后修改。
3. 将本仓库 `templates/CHATGPT_NOTIFY_RULES.md` 的规则内容复制到当前项目合适的位置，推荐：
   `docs/automation/CHATGPT_NOTIFY_RULES.md`。
4. 将 `templates/PROJECT_RULE_SNIPPET.md` 的启动片段合并到当前项目真实启动入口，例如 `AGENTS.md`。
5. 将模板中的 `{{PROJECT_NAME}}` 替换为当前项目的稳定短名称。
6. `NOTIFY_REPO` 固定使用 `white-shark-ssw/ChatGPT-Notify`，`CHANNEL_PR` 固定使用 `1`，不要要求用户再次提供。
7. 不把 Bark Key 写入业务项目；Bark Key 只存在通知中心仓库的 GitHub Actions Secret `BARK_KEY`。
8. 只做接入通知所需的最小修改，不顺手重构项目其他代码或规则。
9. 修改后读取实际文件确认规则已落盘，并在最终回答中明确列出修改位置。

## 接入后的运行语义

项目会话在重要任务真正完成、最终回答已经准备好之后，应在输出最终回答之前向：

`white-shark-ssw/ChatGPT-Notify` PR `#1`

添加一条顶层 PR Conversation 评论：

```text
[BARK_NOTIFY_V1]
{"project":"<PROJECT_NAME>","title":"ChatGPT · <PROJECT_NAME>","body":"<本轮完成内容的一句话摘要>","group":"ChatGPT","url":"https://chatgpt.com/"}
```

通知成功只代表“本轮完整回答已准备输出”，不能夸大为 CI passed、IPA produced、真机验证或问题已稳定解决。

如果当前会话没有 GitHub PR 顶层评论写能力，通知失败不能阻塞正常最终回答；必须如实注明未发送，不得伪造成功。
