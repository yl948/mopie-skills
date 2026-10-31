# 配置步骤

命令行装在用户自己的电脑上，不装进 Mopie 容器。

## 1. 准备命令行

```bash
npx @larksuite/cli@latest install
lark-cli --version
```

已经安装就不要重复安装。

## 2. 新建应用

先用 `lark-cli profile list` 看这台电脑是否已经创建过飞书应用。已有应用不能复用，也看不出它们连着哪一台 Mopie。

```bash
mkdir -p /tmp/mopie-feishu
lark-cli config init --new --name mopie --lang zh_cn > /tmp/mopie-feishu/init.log 2>&1
```

应用名称固定是 `mopie`。`--name` 必须带上，否则会替换这台电脑当前的飞书配置。

把日志里的 `https://open.feishu.cn/page/cli?user_code=...` 发给用户。确认码只用一次。命令中断后重新执行，并使用新链接。

用户确认后，日志出现 `OK: 应用配置成功` 和新的 App ID。再用：

```bash
lark-cli --profile mopie auth status
```

机器人身份应为 `ready`。不要打印 App Secret。

## 3. 写入这台 Mopie

只发送新应用的 App ID 和 App Secret：

```bash
curl -sS -X POST "$MOPIE_URL/api/notifications/channels/feishu/setup" \
  -H "Authorization: Bearer $MOPIE_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"飞书","app_id":"APP_ID","app_secret":"APP_SECRET"}'
```

成功后渠道已保存，交互机器人已打开，绑定窗口为 10 分钟。密钥和 App Secret 不要写进回复、文档或日志。

App Secret 存在本机钥匙串加密文件里。取出时只核对长度，不打印内容。解密失败时看 [接口与排错](api.md)。

## 4. 绑定接收人

让用户在 10 分钟内私聊刚创建的机器人，只发送「绑定」。机器人回复「已绑定」后，再点「测试」。

窗口过期就再调用一次上面的写入接口，或让用户在页面点「检查并保存」，然后马上发送「绑定」。
