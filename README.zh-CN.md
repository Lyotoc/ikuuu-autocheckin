# Ikuuu 自动签到 (GitHub Actions)

本说明为中文版本，仅使用半角标点避免乱码风险。若编辑器或终端出现中文乱码，请按“终端乱码快速修复”操作，并确保以 UTF-8 打开/保存文件。

- 工作流文件: .github/workflows/ikuuu-checkin.yml
- 触发方式: 定时 + 手动（手动触发默认忽略时间窗口与当日缓存）

## 终端乱码快速修复（Windows）
- PowerShell（当前会话）: `[Console]::OutputEncoding = [System.Text.UTF8Encoding]::new()`，如需命令行程序统一用 UTF‑8，再执行 `chcp 65001`。
- PowerShell（持久化）: 打开 `notepad $PROFILE` 追加一行 `[Console]::OutputEncoding = [System.Text.UTF8Encoding]::new()`。
- CMD：执行 `chcp 65001` 后重新运行命令。

## 功能特性
- 多账号：支持 IKUUU_COOKIES（空行分隔账号），逐个调用统计成功或失败
- 时间控制：自定义时区（默认 Asia/Shanghai），每日运行时间与窗口（默认 08:15 ± 10 分钟）
- 稳健性：curl 可配置连接超时、最大时长、重试次数与间隔；非 JSON 响应自动回退
- 可观测性：写入 GitHub Job Summary，便于快速查看明细
- 安全：Cookie 存放在仓库 Secrets；脚本对输出消息做脱敏处理（邮箱、IP、HEX、uid、key、ip、expire_in 等）

## 快速开始
1. 启用 Actions：在仓库 Actions 页面点击启用
2. 配置 Cookie（二选一）
   - 单账号：新增 Secret `IKUUU_COOKIE`（完整 Cookie 字符串）
   - 多账号：新增 Secret `IKUUU_COOKIES`（见下文 Cookie 填写规范）
3. 可选：配置运行变量 `RUN_TZ`, `RUN_AT`, `RUN_WINDOW_MINUTES`, `RETRY`, `RETRY_DELAY`, `CONNECT_TIMEOUT`, `MAX_TIME`（建议使用 Actions -> Variables 保存）
4. 手动触发：Actions -> Ikuuu Checkin -> Run workflow

## 如何获取 Cookie
1. 登录 https://ikuuu.de，打开浏览器开发者工具（F12）-> Network
2. 访问或刷新 https://ikuuu.de/user，在该域名请求的 Request Headers 中复制 Cookie
3. 常见键：`lang`, `uid`, `email`, `key`, `ip`, `expire_in`

## Cookie 填写规范
- 单账号示例（`IKUUU_COOKIE`）
  ```
  lang=zh-cn; uid=YOUR_UID; email=your%40example.com; key=YOUR_KEY; ip=YOUR_IP_HASH; expire_in=TIMESTAMP
  ```
- 多账号示例（`IKUUU_COOKIES`），用空行分隔账号；同一账号可以拆成多行，脚本会自动合并连续的非空行；仅用分号 `;` 分隔键值对，不要使用逗号 `,`；Windows 的 CRLF 可直接粘贴，脚本会去除 `\r`
  ```
  lang=zh-cn; uid=1111111; email=user1%40example.com; key=xxxxxxxx; ip=aaaaaaaa; expire_in=TIMESTAMP

  lang=zh-cn; uid=2222222; email=user2%40example.com; key=yyyyyyyy; ip=bbbbbbbb; expire_in=TIMESTAMP
  ```
- 多行一个账号示例
  ```
  lang=zh-cn; uid=1111111; email=user1%40example.com;
  key=xxxxxxxx; ip=aaaaaaaa; expire_in=TIMESTAMP

  lang=zh-cn; uid=2222222; email=user2%40example.com;
  key=yyyyyyyy; ip=bbbbbbbb; expire_in=TIMESTAMP
  ```

## Secrets 与通知渠道
| 名称 | 必填 | 用途 | 说明 |
| - | - | - | - |
| `IKUUU_COOKIE` | 否（单账号） | 单账号 Cookie | 完整标准 Cookie 字符串 |
| `IKUUU_COOKIES` | 否（多账号） | 多账号 Cookie | 空行分隔账号，同一账号可多行 |
| `TG_BOT_TOKEN` | 否 | Telegram 通知 | Telegram Bot Token |
| `TG_CHAT_ID` | 否 | Telegram 通知 | 个人或群 Chat ID |
| `FEISHU_WEBHOOK` | 否 | 飞书通知 | 群机器人 Webhook URL（建议配置关键词） |
| `WECHAT_WEBHOOK` | 否 | 企业微信通知 | 群机器人 Webhook URL（建议配置关键词） |
| `DISCORD_WEBHOOK` | 否 | Discord 通知 | Webhook URL |

## Actions Variables（运行变量）
| 名称 | 默认值 | 说明 |
| - | - | - |
| `RUN_TZ` | Asia/Shanghai | 运行时区 |
| `RUN_AT` | 08:15 | 每日目标运行时间 HH:MM |
| `RUN_WINDOW_MINUTES` | 10 | 时间窗口，命中 RUN_AT ± 窗口 才执行 |
| `RETRY` | 2 | curl 重试次数 |
| `RETRY_DELAY` | 2 | curl 重试间隔秒 |
| `CONNECT_TIMEOUT` | 10 | 连接超时秒 |
| `MAX_TIME` | 30 | 请求最大时长秒 |

### 变量含义与取值建议
- `RUN_TZ`：IANA 时区名称，用于计算“今天”和目标时间，并生成每日缓存键。示例：`Asia/Shanghai`、`UTC`、`America/Los_Angeles`。留空时默认 `Asia/Shanghai`。
- `RUN_AT`：每日目标时间（24 小时制 `HH:MM`）。例如 `08:15`。仅当“定时触发”且当前时刻落在窗口内时才执行；手动触发不受此限制。
- `RUN_WINDOW_MINUTES`：窗口大小（分钟，整数）。表示允许在目标时间前后各 N 分钟内运行；跨午夜会取最小时间差（如 23:59 与 00:05 视为相差 6 分钟）。建议 5～20 分钟。
- `RETRY`：网络重试次数（curl 的 `--retry`，启用 `--retry-all-errors`）。设为 0 表示不重试；次数过大将拉长运行时间。建议 2～5 次。
- `RETRY_DELAY`：两次重试之间的等待秒数（curl 的 `--retry-delay`）。建议 2～5 秒。
- `CONNECT_TIMEOUT`：连接超时秒数（curl 的 `--connect-timeout`），主要覆盖 TCP/TLS 连接阶段。网络较慢时可增大到 15～30。
- `MAX_TIME`：单次请求的总超时秒数（curl 的 `--max-time`），包含连接与传输。到达上限会中止该次请求；通常与 `RETRY` 搭配使用。建议 20～60。

配置位置：仓库 Settings → Secrets and variables → Actions → Variables（也可在组织级变量中配置）。

## 运行逻辑
- 定时：*/15 * * * *（UTC）
- 时间窗口：命中 RUN_AT ± RUN_WINDOW_MINUTES 且当日未执行才真正签到
- 当日仅执行一次：使用 actions/cache 缓存 `.ikuuu-cache/stamp` 当日标记
- 成功判定：`ret == 1` 视为成功；或 `ret == 0` 且 `msg` 含“已签到/已经签到/似乎已签到”等提示亦视为成功
- 失败时：标记失败（便于告警）；Job Summary 展示每个账号的结果与信息

## 免责声明
本文档与示例仅用于学习与技术研究。请勿用于任何违反网站服务条款或当地法律法规的用途。由此产生的风险与后果由使用者自行承担。示例中的账号、邮箱、密钥均为占位符。
