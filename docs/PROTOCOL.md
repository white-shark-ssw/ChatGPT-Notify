# Notification Protocol v1

通知入口是本仓库固定 PR `#1` 的顶层 Conversation 评论。

## 评论格式

第一行必须精确为：

```text
[BARK_NOTIFY_V1]
```

第二行开始必须是合法 JSON 对象：

```text
[BARK_NOTIFY_V1]
{"project":"OnePlayer","title":"播放器 Seek 与缓存诊断","body":"可以，当前这轮已经把 Seek 日志与 Transport 状态重新对齐。\n\n从现有证据看，时间基准更新是主要问题……","url":"https://chatgpt.com/"}
```

## 字段

- `project`：必需，项目稳定短名称。
- `title`：可选，优先使用当前真实 ChatGPT 会话标题；无法可靠取得时使用自然生成的任务标题。
- `body`：必需，来自已经准备好的最终回答开头的正文预览，最多 500 字符。
- `url`：可选，点击 Bark 通知后的跳转 URL。
- `group`：兼容字段。旧项目可以继续传，但通知中心会忽略其值。

## 固定 Bark 分组

所有通过本通知中心发送的 Bark 通知统一使用：

```text
ChatGPT-Notify
```

该值由 `.github/workflows/bark-notify.yml` 强制设置，业务项目不能覆盖。这样所有 ChatGPT 项目通知会在 Bark 中进入一个独立分组，不与其他 Bark 来源混杂。

## 工作流拒绝条件

- 评论不在固定 PR `#1`；
- 第一行不是 `[BARK_NOTIFY_V1]`；
- JSON 无法解析；
- 缺少 `project` 或 `body`；
- 字段超过长度限制；
- `url` 不是绝对 `http://` 或 `https://` URL。

当前模板有意只接受 HTTP(S) URL。若未来确认某个 App 的自定义 URL Scheme 需要开放，再按真实需求放宽，不预先允许任意 Scheme。

## 内容要求

通知应尽量接近 ChatGPT 官方 iOS 推送的阅读体验：标题像会话标题，正文展示实际回答开头并保留必要换行。

禁止放入 Bark Key、GitHub Token、Cookie、密码、下载直链、私有媒体地址、大段源码或其他秘密信息。

## 临时评论生命周期

`[BARK_NOTIFY_V1]` 评论只作为极短暂的 GitHub Actions 创建事件载体。

项目会话创建通知评论后，必须使用创建结果返回的 comment id，立刻把同一条评论更新为：

```text
<!-- ChatGPT-Notify transient payload redacted -->
```

通知中心 workflow 只监听 `issue_comment: created`，因此编辑评论不会触发第二次推送。真实测试 run #9 已确认：即使公开评论已经立刻被改成隐藏占位符，GitHub Actions 仍会使用创建事件中的原始正文完成 Bark 推送。

随后 workflow 会在 Bark 处理结束后删除该临时评论。当前 workflow 同时授予 `issues: write` 与 `pull-requests: write`；run #10 已确认删除步骤成功，测试 comment 已从 PR 评论列表消失。

最终生命周期：

```text
创建完整 payload
→ 项目会话立即擦除正文
→ Action 使用 created 事件原始 payload 推 Bark
→ Action 删除临时评论
```

即时擦除保护短时公开内容，最终删除保持 PR 整洁。

## 隐私边界

该机制会把公开正文暴露窗口压缩到“创建评论 → 更新评论”两次 GitHub API 调用之间，并在 Action 完成后连占位评论一起删除。

但它仍不构成绝对私密：在被擦除前，第三方理论上仍可能抓取或缓存；GitHub 平台内部也可能保留事件数据。因此不得依赖本机制传输秘密或高敏感内容。
