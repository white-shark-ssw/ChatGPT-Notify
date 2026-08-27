# ChatGPT-Notify

项目级 ChatGPT 完成通知中心：项目会话完成重要工作后，向本仓库固定 PR 投递结构化评论；GitHub Actions 再通过 Bark 推送到 iPhone。

```text
ChatGPT 项目会话
    ↓ PR 顶层评论
ChatGPT-Notify 固定通知 PR
    ↓ issue_comment: created
GitHub Actions
    ↓ Bark API
Bark / APNs
    ↓
iPhone
```

## 通知中心地址

- Repository: `white-shark-ssw/ChatGPT-Notify`
- Git URL: `https://github.com/white-shark-ssw/ChatGPT-Notify.git`
- Protocol: `BARK_NOTIFY_V1`
- Notification PR: 初始化后固定为 `#1`

## 初始化

1. 在仓库 `Settings → Secrets and variables → Actions` 新建 Repository Secret：`BARK_KEY`。
2. `BARK_KEY` 的值只填写 Bark device key，不要把完整推送 URL 或 key 写进源码、聊天或项目规则。
3. 保持 `.github/workflows/bark-notify.yml` 位于默认分支 `main`。
4. 建立并永久保留 Draft PR `#1`，标题建议为 `ChatGPT Notification Channel — KEEP OPEN`。
5. 用 `docs/PROTOCOL.md` 中的测试评论验证 GitHub Actions → Bark → iPhone 真机链路。
6. 需要启用通知的项目读取 `BOOTSTRAP.md`，由该项目会话按规则完成接入。

## 安全原则

- Bark Key 只存在 GitHub Actions Secret `BARK_KEY`。
- ChatGPT 会话只向通知 PR 写结构化事件，不接触 Bark Key。
- 通知正文禁止携带 Token、Cookie、密码、私有下载直链等秘密信息。
- 通知失败不允许阻塞项目本身的最终回答，也不允许伪造“已通知”。

详细说明：

- `BOOTSTRAP.md`：给需要启用通知的项目会话使用的一键接入规则。
- `docs/PROTOCOL.md`：通知评论协议。
- `docs/SETUP.md`：通知中心初始化与测试。
- `templates/CHATGPT_NOTIFY_RULES.md`：项目内通知规则模板。
- `templates/PROJECT_RULE_SNIPPET.md`：项目启动入口接入片段。
