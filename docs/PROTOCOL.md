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
{"project":"OnePlayer","title":"播放器 Seek 与缓存诊断","body":"可以，当前这轮已经把 Seek 日志与 Transport 状态重新对齐。\n\n从现有证据看，时间基准更新是主要问题……","group":"ChatGPT","url":"https://chatgpt.com/"}
```

## 字段

- `project`：必需，项目稳定短名称。
- `title`：可选，优先使用当前真实 ChatGPT 会话标题；无法可靠取得时使用自然生成的任务标题。
- `body`：必需，来自已经准备好的最终回答开头的正文预览，最多 500 字符。
- `group`：可选，默认 `ChatGPT`。
- `url`：可选，点击 Bark 通知后的跳转 URL。

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

`[BARK_NOTIFY_V1]` 评论只作为 GitHub Actions 的临时传输载体。

工作流会在 Bark 处理结束后立即尝试删除触发评论；清理步骤使用 GitHub `GITHUB_TOKEN` 的 `issues: write` 权限。即使 Bark 处理失败，清理步骤仍会通过 `always()` 尽量执行。

因此正常情况下，通知评论只会短暂存在数秒。但仓库是 Public 时，短暂公开仍不等于绝对私密：第三方抓取、缓存或平台内部留存仍理论上可能发生，所以不得依赖“自动删除”来承载秘密信息。
