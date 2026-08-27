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
{"project":"OnePlayer","title":"ChatGPT · OnePlayer","body":"本轮 Seek / Transport 分析已经完成。","group":"ChatGPT","url":"https://chatgpt.com/"}
```

## 字段

- `project`：必需，项目稳定短名称。
- `title`：可选，省略时由工作流生成。
- `body`：必需，一句话完成摘要。
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

`body` 应让人在锁屏上立即知道哪个任务完成了，例如：

```text
播放器 Seek 日志分析完成，已定位到时间基准更新问题。
```

禁止放入：Bark Key、GitHub Token、Cookie、密码、下载直链、私有媒体地址、大段源码或其他秘密信息。
