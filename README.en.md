# Ikuuu Check-in (GitHub Actions)

This README uses ASCII punctuation and UTF‑8 encoding.

- Workflow file: `.github/workflows/ikuuu-checkin.yml`
- Triggers: schedule + manual (manual run ignores time window and daily cache)

## Encoding tip (Windows)
- PowerShell (current session): `[Console]::OutputEncoding = [System.Text.UTF8Encoding]::new()`; optionally `chcp 65001`.
- Persist: edit `notepad $PROFILE` and add the line above.

## Features
- Multi-account: `IKUUU_COOKIES` (one account per paragraph separated by blank line)
- Time control: timezone (Asia/Shanghai), daily target time (08:15), window (+/- 10 min)
- Robust curl: timeouts, retries
- Observability: GitHub Job Summary with details per account
- Security: cookies in Secrets, runtime message masking

## Quick start
1. Enable Actions in repository
2. Secrets:
   - Single account: `IKUUU_COOKIE`
   - Multiple accounts: `IKUUU_COOKIES`
3. Variables (optional): `RUN_TZ`, `RUN_AT`, `RUN_WINDOW_MINUTES`, `RETRY`, `RETRY_DELAY`, `CONNECT_TIMEOUT`, `MAX_TIME`
4. Manual run: Actions -> Ikuuu Checkin -> Run workflow

## Cookie format
```
lang=zh-cn; uid=YOUR_UID; email=your%40example.com; key=YOUR_KEY; ip=YOUR_IP_HASH; expire_in=TIMESTAMP
```

Multiple accounts (blank line separated):
```
lang=zh-cn; uid=1111111; email=user1%40example.com; key=xxxxxxxx; ip=aaaaaaaa; expire_in=TIMESTAMP

lang=zh-cn; uid=2222222; email=user2%40example.com; key=yyyyyyyy; ip=bbbbbbbb; expire_in=TIMESTAMP
```

## Secrets and notifications
| name | required | usage |
| - | - | - |
| `IKUUU_COOKIE` | no | single account cookie |
| `IKUUU_COOKIES` | no | multi-account cookies |
| `TG_BOT_TOKEN` / `TG_CHAT_ID` | no | Telegram |
| `FEISHU_WEBHOOK` | no | Lark/Feishu |
| `WECHAT_WEBHOOK` | no | WeCom |
| `DISCORD_WEBHOOK` | no | Discord |

## Variables
| name | default | note |
| - | - | - |
| `RUN_TZ` | Asia/Shanghai | timezone |
| `RUN_AT` | 08:15 | HH:MM |
| `RUN_WINDOW_MINUTES` | 10 | window size |
| `RETRY` | 2 | curl retries |
| `RETRY_DELAY` | 2 | seconds |
| `CONNECT_TIMEOUT` | 10 | seconds |
| `MAX_TIME` | 30 | seconds |

## Logic
- Schedule: `*/15 * * * *` (UTC)
- Run only in window and once per day (cache)
- Success: `ret == 1`, or `ret == 0` with message like "already checked in"
- On failure: job marked failed; details listed in summary

## Disclaimer
This project and examples are for study only. Do not use in violation of ToS or laws. All examples use placeholders.

