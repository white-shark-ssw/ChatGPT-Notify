# ChatGPT Completion Notification Rules

本项目启用了项目级 Bark 完成通知。

## 固定配置

- Notification hub repository: `white-shark-ssw/ChatGPT-Notify`
- Notification channel PR: `#1`
- Project name: `{{PROJECT_NAME}}`
- Protocol: `BARK_NOTIFY_V1`
- Bark group: `ChatGPT-Notify`（由通知中心 workflow 强制设置）
- Default notification policy: **every final reply**

## 启动要求

每次新的开发会话开始时，在执行项目任务前读取并遵守本文件。

本文件只定义“完成通知”行为，不改变项目本身的源码、构建、测试、发布或文档规则。若本文件与项目安全规则冲突，项目安全规则优先；但不得虚假声称通知已经发送。

如果项目有 `AGENTS.md` 或等价的第一启动规则，必须在该文件中直接写明：每个面向用户的最终回答都必须触发一次通知，并要求读取本文件。不要只依赖间接引用。

## 何时发送

**每一个准备发送给用户的最终回答都必须发送一次完成通知。**

不再根据任务重要性、耗时、是否修改代码、是否调用 GitHub/CI/IPA、回答长短或问答类型过滤。普通寒暄、简单事实问答和短澄清，只要是最终回答，同样发送。

以下内容不单独发送：commentary、中间进度、工具过程消息、尚未完成的“正在处理/继续检查”等状态。

每个用户轮次最多发送一次最终回答通知。若用户在当前轮明确要求“本轮不要通知”，该明确当前指令优先。

## 执行顺序

每轮最终回答必须按这个顺序：

1. 完成本轮所有必要分析、代码、工具调用、验证和项目文档同步。
2. 形成准备发送给用户的最终回答。
3. 从最终回答生成通知标题与正文预览。
4. 向 `white-shark-ssw/ChatGPT-Notify` PR `#1` 添加一条顶层 PR Conversation 评论。
5. 记录创建结果返回的 comment id，并立即把同一条评论更新为：

```text
<!-- ChatGPT-Notify transient payload redacted -->
```

6. 确认即时擦除调用返回成功后，再输出最终回答。
7. 不得在通知之后继续进行会改变最终结论的操作。

通知中心 workflow 只监听 `issue_comment: created`，因此创建后立即编辑不会触发第二次推送。已验证 Action 会保留创建事件中的原始 payload 并完成 Bark 推送。

通知中心随后会用 GitHub Actions 自带的 `GITHUB_TOKEN` 删除该临时评论。当前 workflow 同时授予 `issues: write` 与 `pull-requests: write`；已真测确认 Bark 推送与删除评论均成功。因此正常情况下 PR 不会持续积累占位评论。

如果创建通知评论成功但立即擦除失败，最终回答中必须明确注明：

`Bark 通知已触发，但 GitHub 临时通知正文擦除失败，请检查通知 PR #1。`

## 通知标题与正文

目标是尽量接近 ChatGPT 官方 iOS 推送的阅读体验。

### title

- 如果当前环境能够可靠取得当前 ChatGPT 会话标题，优先使用该标题。
- 如果无法可靠取得会话标题，则根据本轮会话主题生成一个自然短标题，建议 6–22 个中文字符。
- 不固定添加 `ChatGPT · {{PROJECT_NAME}}` 前缀。
- 不得声称拿到了真实会话标题，除非当前环境确实提供了它。

### body

- 使用已经准备好的最终回答开头作为通知正文预览。
- 建议约 150–500 个字符；内容较短时可直接使用完整短答。
- 尽量保留自然换行，使展开通知后能看到更多实际回答内容。
- 可以压缩 Markdown 标题、列表符号或代码围栏，但不得改变原意和证据级别。
- 禁止包含密码、Token、Cookie、Bark Key、下载直链、私有媒体地址或其他秘密。

## Bark 分组

业务项目不负责选择 Bark 分组。通知中心 workflow 会统一覆盖为：

```text
ChatGPT-Notify
```

## GitHub 工具纪律

调用前先使用当前环境真实提供的 GitHub 工具定义，不得猜测工具名、参数名或 API。

本协议要求项目会话具备：

1. 向指定仓库的指定 PR 添加一条顶层 Conversation 评论；
2. 使用创建结果返回的 comment id，立即更新同一条顶层评论为固定隐藏占位内容。

不要创建 inline review comment、PR review，也不要修改业务项目自己的 issue。

如果当前会话没有可用的 GitHub 顶层 PR 评论写能力，通知失败不得阻塞正常最终回答。最终回答末尾只需简短注明：

`Bark 完成通知未发送：当前会话没有可用的 GitHub PR 评论写能力。`

不得伪造“已通知”。

## 评论协议

创建时的临时评论必须采用：

```text
[BARK_NOTIFY_V1]
{"project":"{{PROJECT_NAME}}","title":"<当前会话标题或自然生成的任务标题>","body":"<最终回答开头的正文预览，建议 150–500 字符>","url":"https://chatgpt.com/"}
```

为兼容已接入的旧项目，workflow 仍接受输入中的 `group` 字段，但会忽略其值并强制使用 `ChatGPT-Notify`。

要求：

- 第一行必须精确为 `[BARK_NOTIFY_V1]`。
- 第二行开始必须是合法 JSON。
- `project` 固定为 `{{PROJECT_NAME}}`。
- `body` 最多 500 字符。
- 每个用户轮次最多发送一次最终回答通知。
- 不发送“开始处理”“处理中”“即将完成”等过程通知。

创建成功后必须立即把同一条评论更新为隐藏占位；Action 推送后会再彻底删除该评论。

## 已打开会话的限制

仓库规则更新不会自动注入已经打开且不再读取启动规则的旧会话。旧会话若是在接入或规则更新前启动，需要一次性重新读取项目 `AGENTS.md` / 启动规则与本文件，之后的回复才能按新规则执行。新会话应由项目启动链自动加载。

## 隐私边界

通知中心仓库可以保持 Public，以避免 Private GitHub-hosted Actions 分钟额度限制。

完整通知正文只在“创建评论 → 项目会话立即擦除”之间短暂公开，之后只剩隐藏占位；Action 完成后占位评论也会被删除。该机制显著降低暴露时间，但不是绝对隐私保证：极短时间内仍理论上可能被抓取或缓存，GitHub 平台内部也可能保留事件数据。因此不得依赖本机制传输秘密或高敏感内容。

## 证据级别

Bark 通知只代表 ChatGPT 当前这一轮已经形成完整回答并准备输出，不自动代表 CI passed、artifact produced、real-device tested、bug fixed 或 stable/frozen。通知正文必须沿用项目本身的真实证据级别，不得夸大。
