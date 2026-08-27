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

默认不发送：

- 普通寒暄；
- 简单事实问答；
- 很短的澄清；
- 中间进度更新；
- 尚未完成的部分结果；
- 仅告知“正在处理”。

如果用户在当前会话明确要求“本项目每个最终回答都通知”或“本轮不要通知”，用户当前指令优先。

## 执行顺序

必须按这个顺序：

1. 完成本轮所有必要分析、代码、工具调用和验证。
2. 形成准备发送给用户的最终结论。
3. 判断本轮是否满足通知条件。
4. 如满足，使用当前可用的 GitHub 写工具，向 `white-shark-ssw/ChatGPT-Notify` 的 PR `#1` 添加一条顶层 PR Conversation 评论。
5. GitHub 评论操作返回成功后，再输出最终回答。
6. 不得在通知之后继续进行会改变最终结论的开发操作。

不能把通知安排成“最终回答发出以后再后台执行”；不存在这种后台阶段。

## GitHub 工具纪律

修改或调用前先使用当前环境真实提供的 GitHub 工具定义，不得猜测工具名、参数名或 API。

目标能力必须是：

> 向指定仓库的指定 PR 添加一条顶层 Conversation 评论。

不要：

- 创建 inline review comment；
- 创建 PR review；
- 修改业务项目的 issue；
- 把 Bark Key 放进评论；
- 因通知失败去修改业务源码。

如果当前会话没有可用的 GitHub 写权限或对应工具，通知失败不得阻塞正常最终回答。最终回答末尾只需简短注明：

`Bark 完成通知未发送：当前会话没有可用的 GitHub PR 评论写能力。`

不要伪造“已通知”。

## 评论协议

评论必须严格采用：

```text
[BARK_NOTIFY_V1]
{"project":"{{PROJECT_NAME}}","title":"ChatGPT · {{PROJECT_NAME}}","body":"<本轮完成内容的一句话摘要>","group":"ChatGPT","url":"https://chatgpt.com/"}
```

要求：

- 第一行必须精确为 `[BARK_NOTIFY_V1]`。
- 第二行开始必须是合法 JSON。
- `project` 固定为 `{{PROJECT_NAME}}`。
- `title` 默认使用 `ChatGPT · {{PROJECT_NAME}}`。
- `body` 使用简洁、可在锁屏直接辨识的一句话。
- 不把完整回答复制到 `body`。
- 不包含密码、Token、Cookie、Bark Key、下载直链、私有媒体地址或其他秘密。
- 每个符合条件的用户轮次最多发送一次完成通知。
- 不发送“开始处理”“处理中”“即将完成”等通知。

## 证据级别

Bark 通知只代表：

> ChatGPT 当前这一轮已经形成完整回答并准备输出。

它不自动代表：

- CI passed；
- artifact produced；
- real-device tested；
- bug fixed；
- stable / frozen。

通知摘要必须沿用项目本身的证据级别，不得夸大。
