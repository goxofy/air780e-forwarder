# Air700E / Air780E / Air780EP / Air780EPV 短信转发 来电通知

## 保姆级教程：https://kdocs.cn/l/coe1ozIlSX70

## :sparkles: Feature

- [x] 多种通知方式
    - [x] [Telegram](https://github.com/0wQ/telegram-notify)
    - [x] [PushDeer](https://www.pushdeer.com/)
    - [x] [Bark](https://github.com/Finb/Bark)
    - [x] [钉钉群机器人 DingTalk](https://open.dingtalk.com/document/robots/custom-robot-access)
    - [x] [飞书群机器人 Feishu](https://open.feishu.cn/document/ukTMukTMukTM/ucTM5YjL3ETO24yNxkjN)
    - [x] [企业微信群机器人 WeCom](https://developer.work.weixin.qq.com/document/path/91770)
    - [x] [Pushover](https://pushover.net/api)
    - [x] [邮件 next-smtp-proxy](https://github.com/0wQ/next-smtp-proxy)
    - [x] [Gotify](https://gotify.net)
    - [x] [Inotify](https://github.com/xpnas/Inotify) / [合宙官方的推送服务](https://push.luatos.org)
    - [x] 邮件 (SMTP协议)
- [x] 通过短信控制设备
    - [x] 发短信, 格式: `SMS,10010,余额查询`
- [x] 定时基站定位
- [x] 定时查询流量
- [x] 定时上报存活
- [x] 开机通知
- [x] POW 按键长按短按操作
- [x] 使用消息队列, 测试添加几百条通知, 不会卡死
- [x] 通知发送失败, 自动重发, 断电后再次开机可以恢复重发
- [x] 从机模式：通过串口转发收到的短信，并接受串口指令发送短信

## :satellite: 从机模式与串口发短信

本仓库在上游基础上做了精简，**仅保留从机模式**，并新增**串口发短信**能力，用于配合 ESP32-C3 等上位机（上位机配置见下方 [`esphome/`](esphome/)）。

- 已删除主机 (MASTER) 角色，`script/config.lua` 中不再有 `ROLE` 配置；串口接收回调固定按从机逻辑处理（`script/main.lua`）。
- **收短信**：模块收到短信后，经 `NOTIFY_TYPE = "serial"` 通道，由 UART1 (115200, 8N1) 推送到上位机。
- **发短信**：上位机通过 UART1 写入一行指令 `SMS,号码,内容\n`，模块解析后调用 `sms.send` 发送。
    - 以换行符 `\n` 作为指令结束符，已做缓冲，支持分包/粘包。
    - 号码须为纯数字（可带前导 `+`），长度 5–20；内容中请勿包含换行符。
    - 示例：`SMS,10010,余额查询\n`

## :electric_plug: ESP32-C3 上位机 (ESPHome)

[`esphome/`](esphome/) 目录是配套的 ESP32-C3 上位机配置（基于 [ESPHome](https://esphome.io/) + Home Assistant）：

- `esphome/config.yaml` — 设备配置：
    - UART(GPIO0 TX / GPIO1 RX, 115200) 对接 Air780E。
    - 读取从机推送的短信，按 UTF-8 边界拼包，POST 转发到 Pushover（`#ALIVE_REPORT` 心跳不转发；内容做 JSON 转义）。
    - 暴露三个 Home Assistant 实体用于发短信：文本 `SMS 收件号码`、文本 `SMS 内容`、按钮 `发送短信`；按钮按下后向 UART 写入 `SMS,号码,内容\n`。
    - 所有敏感信息(WiFi、API key、OTA 密码、Pushover token/user)均通过 `!secret` / `substitutions` 引用。
- `esphome/secrets.yaml.example` — secrets 模板。使用时复制为 `esphome/secrets.yaml` 并填入真实值；`secrets.yaml` 已被 `.gitignore` 忽略，请勿提交。

```bash
cp esphome/secrets.yaml.example esphome/secrets.yaml
# 编辑 esphome/secrets.yaml 填入真实值后，用 ESPHome 编译烧录 esphome/config.yaml
```

## :hammer: Usage

https://mizore.notion.site/Air780E-e750efe0d6cc40c3baa276eeb811d534


