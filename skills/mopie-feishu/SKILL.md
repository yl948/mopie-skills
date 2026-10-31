---
name: mopie-feishu
description: 为 Mopie 新建并接入飞书通知。用户说「配置飞书」「给 Mopie 加飞书」「飞书通知」时使用。每台 Mopie 都新建自己的飞书应用。
---

# Mopie 飞书配置

面向第一次配置的用户。不要提其他机器、内网地址、已有应用或之前的配置。

每台 Mopie 新建一个飞书应用。一个应用只能保持一条长连接。`open_id` 只属于创建它的应用，别的应用的号不能填进来。

## 先拿到这两项

1. Mopie 的完整地址，必须包含协议、主机和端口，例如 `http://主机:5174`。没给端口就问，不要猜。
2. 一把用途为「配置飞书」的密钥。用户在「设置 → API 密钥管理」创建。不要要登录密码，也不要用「搜索并添加订阅」的密钥。

这台 Mopie 还没有「配置飞书」这个用途时，改为让用户在通知页面填写 App ID 和 App Secret，再点「检查并保存」。

## 配置

### 1. 准备命令行

命令行装在用户自己的电脑上：

```bash
npx @larksuite/cli@latest install
lark-cli --version
```

已经安装就不要重复安装。

### 2. 新建应用

先用 `lark-cli profile list` 看这台电脑是否已经创建过飞书应用。已有应用不能复用，看不出它们连着哪一台 Mopie。

```bash
mkdir -p /tmp/mopie-feishu
lark-cli config init --new --name mopie --lang zh_cn > /tmp/mopie-feishu/init.log 2>&1
```

应用名称固定是 `mopie`。`--name` 必须带上，否则会替换这台电脑当前的飞书配置。

把日志里的 `https://open.feishu.cn/page/cli?user_code=...` 发给用户。确认码只用一次，命令中断后重新执行并使用新链接。

用户确认后，日志出现 `OK: 应用配置成功` 和新的 App ID。用 `lark-cli --profile mopie auth status` 核对，机器人身份应为 `ready`。不要打印 App Secret。

### 3. 写入这台 Mopie

只发送新应用的 App ID 和 App Secret：

```bash
curl -sS -X POST "$MOPIE_URL/api/notifications/channels/feishu/setup" \
  -H "Authorization: Bearer $MOPIE_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"飞书","app_id":"APP_ID","app_secret":"APP_SECRET"}'
```

成功后渠道已保存，交互机器人已打开，绑定窗口为 10 分钟。密钥和 App Secret 不要写进回复、文档或日志。

### 4. 绑定接收人

让用户在 10 分钟内私聊刚创建的机器人，只发送「绑定」。成功后再点「测试」。窗口过期就再点一次「检查并保存」，然后马上发送「绑定」。

## 不要做

- 不复用已有飞书应用，也不填别人的 App ID、App Secret 或 `open_id`。
- 不让用户去飞书后台手动开通权限、添加事件或发布版本。命令行新建的应用已经带好。
- 不运行 `lark-cli auth login --recommend`，也不在 Mopie 连接时长连接时执行 `lark-cli event consume`。
- 不把 App Secret、完整 `open_id`、钥匙串内容写进回复、文档或日志。
