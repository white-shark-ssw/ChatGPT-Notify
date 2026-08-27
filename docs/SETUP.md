# Setup

## 1. 设置 Bark Secret

进入：

`Settings → Secrets and variables → Actions → New repository secret`

创建：

```text
Name: BARK_KEY
Value: <你的 Bark device key>
```

只保存 key，不要保存完整推送 URL。

## 2. 固定通知 PR

通知中心使用永久保留的 Draft PR `#1` 作为事件入口。

推荐标题：

```text
ChatGPT Notification Channel — KEEP OPEN
```

该 PR 不应合并或关闭。

## 3. 手动测试

在 PR `#1` 的 Conversation 页面发送：

```text
[BARK_NOTIFY_V1]
{"project":"NotifyTest","title":"ChatGPT · NotifyTest","body":"通知中心测试成功。","group":"ChatGPT","url":"https://chatgpt.com/"}
```

然后查看 `Actions → Bark notification`。

成功标准必须分开判断：

1. PR 评论已创建；
2. GitHub Action 已触发；
3. Bark API 返回成功；
4. iPhone 真机实际收到 Bark。

不要把“Action 成功”描述成“iPhone 已经收到”。

## 4. 接入其他项目

让需要启用通知的项目会话读取：

`https://github.com/white-shark-ssw/ChatGPT-Notify/blob/main/BOOTSTRAP.md`

随后由项目会话根据真实项目启动入口完成最小接入。

## 5. 点击通知

默认协议使用：

```json
{"url":"https://chatgpt.com/"}
```

这只表示点击 Bark 后打开 ChatGPT 网站/由系统处理 Universal Link；在没有官方稳定的私人 conversation deep link 证据前，不声称能够精确回到产生通知的具体会话。
