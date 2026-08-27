# ChatGPT Completion Notification Rules

本项目启用了项目级 Bark 完成通知。

## 固定配置

- Notification hub repository: `white-shark-ssw/ChatGPT-Notify`
- Notification channel PR: `#1`
- Project name: `{{PROJECT_NAME}}`
- Protocol: `BARK_NOTIFY_V1`
- Default notification policy: important completion only

## 启动要求

每次新的开发会话开始时，在执行项目任务前读取并遵守本文件。

本文件只定义“完成通知”行为，不改变项目本身的源码、构建、测试、发布或文档规则。若本文件与项目安全规则冲突，项目安全规则优先；但不得因此虚假声称通知已经发送。

## 何时发送

默认仅在本轮属于明显需要处理时间的重要任务，并且本轮工作已经真正完成、最终回答已经准备好时发送一次通知。

典型需要发送：

- 代码实现或代码修改完成；
- Bug / 日志 / 崩溃分析完成；
- CI、构建、IPA、发布或测试证据分析完成；
- GitHub PR / issue / workflow 的实质性处理完成；
- 多步骤技术调查或研究完成；
- 用户要求生成的文件/产物已经完成；
- 一次较长的项目决策分析已经形成完整结论。

默认不发送普通寒暄、简单事实问答、很短澄清、中间进度、尚未完成的部分结果或“正在处理”通知。

如果用户在当前会话明确要求“本项目每个最终回答都通知”或“本轮不要通知”，用户当前指令优先。

## 执行顺序

必须按这个顺序：

1. 完成本轮所有必要分析、代码、工具调用和验证。
2. 形成准备发送给用户的最终回答。
3. 从最终回答中生成通知标题与正文预览。
4. 判断本轮是否满足通知条件。
5. 如满足，使用当前可用的 GitHub 写工具，向 `white-shark-ssw/ChatGPT-Notify` 的 PR `#1` 添加一条顶层 PR Conversation 评论。
6. **记录创建评论返回的 comment id，并立即调用 GitHub 顶层评论更新能力，把同一条评论正文替换为：**

```text
<!-- ChatGPT-Notify transient payload redacted -->
```

7. 确认更新调用返回成功后，再输出最终回答。
8. 不得在通知之后继续进行会改变最终结论的开发操作。

这里的“立即擦除”是协议的一部分，不是可选清理。`issue_comment: created` 的 GitHub Actions 事件会保留创建时的原始 payload，因此通知中心仍可使用原始内容推 Bark；评论随后被编辑不会触发第二次推送，因为 workflow 只监听 `created`。

如果“创建通知评论”成功但“立即擦除评论”失败，正常最终回答仍可继续，但必须在最终回答中明确注明：

`Bark 通知已触发，但 GitHub 临时通知正文擦除失败，请检查通知 PR #1。`

不能把通知安排成“最终回答发出以后再后台执行”；不存在这种后台阶段。

## 通知标题与正文

目标是尽量接近 ChatGPT 官方 iOS 推送的阅读体验，而不是只发一句“任务完成”。

### title

- 如果当前环境能够可靠取得当前 ChatGPT 会话标题，优先使用该标题。
- 如果无法可靠取得会话标题，则根据本轮会话主题生成一个自然的短标题，建议 6–22 个中文字符。
- 标题应类似 `多会话驻留与快速切换`、`播放器 Seek 与缓存诊断`，而不是固定写 `ChatGPT · {{PROJECT_NAME}}`。
- 不得声称拿到了真实会话标题，除非当前环境确实提供了它。

### body

- 使用“已经准备好的最终回答”的开头内容作为通知正文预览，而不是另写一句空泛摘要。
- 建议约 150–500 个字符；内容较短时可直接使用完整短答。
- 尽量保留自然换行，使展开通知后可以像 ChatGPT 官方推送一样看到更多实际回答内容。
- 可以压缩 Markdown 标题、列表符号或代码围栏，但不得改变原意和证据级别。
- 不得为了通知而泄露密码、Token、Cookie、Bark Key、下载直链、私有媒体地址或其他秘密。

## GitHub 工具纪律

修改或调用前先使用当前环境真实提供的 GitHub 工具定义，不得猜测工具名、参数名或 API。

本协议要求两种能力：

1. 向指定仓库的指定 PR 添加一条顶层 Conversation 评论；
2. 使用创建结果返回的 comment id，立即更新同一条顶层评论为固定的隐藏占位内容。

不要创建 inline review comment、PR review，也不要修改业务项目自己的 issue。

不要尝试让 GitHub Actions 使用默认 `GITHUB_TOKEN` 删除这条用户评论。当前真实测试中，即使 workflow 获得 `Issues: write`，删除仍返回 HTTP 403；该方案已被否定。

如果当前会话没有可用的 GitHub 顶层 PR 评论写能力，通知失败不得阻塞正常最终回答。最终回答末尾只需简短注明：

`Bark 完成通知未发送：当前会话没有可用的 GitHub PR 评论写能力。`

不要伪造“已通知”。

## 评论协议

创建时的临时评论必须严格采用：

```text
[BARK_NOTIFY_V1]
{"project":"{{PROJECT_NAME}}","title":"<当前会话标题或自然生成的任务标题>","body":"<最终回答开头的正文预览，建议 150–500 字符>","group":"ChatGPT","url":"https://chatgpt.com/"}
```

要求：

- 第一行必须精确为 `[BARK_NOTIFY_V1]`。
- 第二行开始必须是合法 JSON。
- `project` 固定为 `{{PROJECT_NAME}}`。
- `title` 不再固定添加 `ChatGPT ·` 前缀。
- `body` 来自最终回答预览，最多 500 字符。
- 每个符合条件的用户轮次最多发送一次完成通知。
- 不发送“开始处理”“处理中”“即将完成”等通知。

创建成功后，必须立即把同一条评论更新为：

```text
<!-- ChatGPT-Notify transient payload redacted -->
```

## 临时评论与隐私

通知中心仓库当前可以保持 Public，以避免 Private GitHub-hosted Actions 分钟额度限制。

`[BARK_NOTIFY_V1]` 评论只作为极短暂的事件触发载体。真实测试已经确认：创建评论后立即由 ChatGPT GitHub 连接擦除正文，不会影响 `issue_comment: created` workflow 使用原始 payload 推送 Bark。

这会把公开正文暴露窗口压缩到创建与更新两次 GitHub API 调用之间，通常远短于等待 GitHub Actions 执行完成。但它仍不构成绝对隐私保证：极短时间内理论上仍可能被第三方抓取或缓存，GitHub 平台内部也可能保留事件数据。因此通知正文仍然禁止包含任何秘密或高敏感信息。

如果未来需要“连占位评论本身也完全删除”，应额外配置一个仅限 `white-shark-ssw/ChatGPT-Notify` 仓库、具备 Issues write 的独立凭证，再由 Action 删除；不要把广泛权限 PAT 写入业务项目。

## 证据级别

Bark 通知只代表：

> ChatGPT 当前这一轮已经形成完整回答并准备输出。

它不自动代表 CI passed、artifact produced、real-device tested、bug fixed 或 stable / frozen。通知正文必须沿用项目本身的真实证据级别，不得夸大。
