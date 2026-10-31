# 接口与排错

## 写入渠道

`POST /api/notifications/channels/feishu/setup`

请求体只有三项：

```json
{"name":"飞书","app_id":"cli_xxx","app_secret":"..."}
```

调用时带 `Authorization: Bearer mk_...`。这把密钥的用途必须是「配置飞书」。它只能调用这一个接口。

成功时返回「凭证可用，交互机器人已打开」，并包含已保存的渠道。渠道里的 App Secret 会显示为掩码。`allowed_open_ids` 这时仍是空的，要等用户发送「绑定」后才会写入。

## 常见失败

| 现象 | 处理 |
|---|---|
| 密钥被拒绝 | 这把密钥不是「配置飞书」。不要改用登录密码。 |
| `没有可接收通知的 open_id` | 还没绑定。重新打开 10 分钟窗口，让用户私聊发送「绑定」。 |
| `open_id cross app` | 填了别的应用的 `open_id`。清空后重新绑定，不要从另一台 Mopie 复制。 |
| 确认链接打开后没有生成应用 | 确认码已失效。重新执行新建命令，发送新链接。 |
| 解密 App Secret 失败 | 加密文件在 `~/Library/Application Support/lark-cli/appsecret_<App ID>.enc`，主密钥在同目录的 `master.key.file`。不要把解密结果打印出来。 |

## 不需要的操作

命令行新建的应用已经带有机器人、收发消息和上传图片的权限。不要再打开飞书后台的权限、事件或发布页面。

不要运行 `lark-cli auth login --recommend`。那是个人身份登录，不是通知渠道需要的授权。

不要在 Mopie 已连接时长连接时执行 `lark-cli event consume`。一个应用只能保持一条长连接。
