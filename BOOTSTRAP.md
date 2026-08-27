# ChatGPT Notify — Project Bootstrap

本文件用于把项目级 Bark 完成通知接入任意 GitHub 项目。

通知中心固定配置：

- Hub repository: `white-shark-ssw/ChatGPT-Notify`
- Hub URL: `https://github.com/white-shark-ssw/ChatGPT-Notify`
- Channel PR: `#1`
- Protocol: `BARK_NOTIFY_V1`
- Bark group: `ChatGPT-Notify`

## 给项目会话的执行要求

当用户要求“为当前项目接入 ChatGPT-Notify”时：

1. 先读取当前项目已有的 `AGENTS.md`、项目启动规则和文档规则，确认真实启动入口。
2. 不猜测目录结构；只在确认后修改。
3. 将本仓库 `templates/CHATGPT_NOTIFY_RULES.md` 的规则内容复制到当前项目合适的位置，推荐：`docs/automation/CHATGPT_NOTIFY_RULES.md`。
4. 将 `templates/PROJECT_RULE_SNIPPET.md` 的启动片段合并到当前项目真实启动入口，例如 `AGENTS.md`。
5. 将模板中的 `{{PROJECT_NAME}}` 替换为当前项目的稳定短名称。
6. `NOTIFY_REPO` 固定使用 `white-shark-ssw/ChatGPT-Notify`，`CHANNEL_PR` 固定使用 `1`，不要要求用户再次提供。
7. 不把 Bark Key 写入业务项目；Bark Key 只存在通知中心仓库的 GitHub Actions Secret `BARK_KEY`。
8. 只做接入通知所需的最小修改，不顺手重构项目其他代码或规则。
9. 修改后读取实际文件确认规则已落盘，并在最终回答中明确列出修改位置。

## 接入后的运行语义

项目会话在重要任务真正完成、最终回答已经准备好之后，应在输出最终回答之前向 `white-shark-ssw/ChatGPT-Notify` PR `#1` 添加一条顶层 PR Conversation 评论：

```text
[BARK_NOTIFY_V1]
{"project":"<PROJECT_NAME>","title":"<当前会话标题或自然生成的任务标题>","body":"<最终回答开头的正文预览，建议 150–500 字符>","url":"https://chatgpt.com/"}
```

通知中心 workflow 会统一把 Bark `group` 设置为 `ChatGPT-Notify`，业务项目无需传分组值。为兼容旧项目，输入中的旧 `group` 字段仍可存在，但会被忽略。

通知标题尽量采用当前真实 ChatGPT 会话标题；如果当前环境无法可靠取得会话标题，则根据会话主题生成自然短标题，不得假装已读取到真实 UI 标题。

通知正文应来自已经准备好的最终回答开头，保留必要换行，让 Bark 展开后能看到更多实际回答内容，而不是只发一句“任务完成”。

### 创建后立即擦除公开正文

创建评论成功后，项目会话必须保存返回的 comment id，并立刻调用 GitHub 顶层评论更新能力，把同一条评论改成：

```text
<!-- ChatGPT-Notify transient payload redacted -->
```

`issue_comment: created` workflow 会继续使用创建事件中的原始 payload 推 Bark；立即编辑评论不会触发第二次推送，因为通知中心只监听 `created`。

### Action 推送后彻底删除

通知中心 workflow 在 Bark 处理结束后会使用 GitHub Actions 自带的 `GITHUB_TOKEN` 删除该临时评论。当前 workflow 授予 `issues: write` 与 `pull-requests: write`，run #10 已真测确认：Bark 推送成功后，删除步骤也成功，测试 comment 已从 PR 评论列表消失。

因此正常生命周期是：

```text
创建完整 payload → 立即擦除正文 → Action 推 Bark → Action 删除评论
```

这样既尽量缩短 Public 仓库中的正文暴露时间，也避免 PR 长期堆积占位评论。

这种机制仍不是绝对隐私保证。评论在“创建 → 擦除”两次 API 调用之间可能短暂公开，因此禁止在通知 payload 中放入密码、Token、Cookie、Bark Key、私有下载地址或其他秘密信息。

通知成功只代表“本轮完整回答已准备输出”，不能夸大为 CI passed、IPA produced、真机验证或问题已稳定解决。

如果当前会话没有 GitHub PR 顶层评论写能力，通知失败不能阻塞正常最终回答；必须如实注明未发送，不得伪造成功。
