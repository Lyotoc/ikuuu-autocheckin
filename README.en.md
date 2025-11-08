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
3. Variables (optional): `IKUUU_BASE_URL`, `RUN_TZ`, `RUN_AT`, `RUN_WINDOW_MINUTES`, `RETRY`, `RETRY_DELAY`, `CONNECT_TIMEOUT`, `MAX_TIME`
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
> Note: All variables below have built‑in defaults in the workflow. You can run without setting them; configure Actions Variables only if you need to override.
| name | default | note |
| - | - | - |
| `RUN_TZ` | Asia/Shanghai | timezone |
| `RUN_AT` | 08:15 | HH:MM |
| `RUN_WINDOW_MINUTES` | 10 | window size |
| `RETRY` | 2 | curl retries |
| `RETRY_DELAY` | 2 | seconds |
| `CONNECT_TIMEOUT` | 10 | seconds |
| `MAX_TIME` | 30 | seconds |
| `IKUUU_BASE_URL` | https://ikuuu.de | base URL for requests (no trailing slash) |

### Details and guidance
- `RUN_TZ`: IANA timezone name used to compute “today” and the target time, also part of the daily cache key. Examples: `Asia/Shanghai`, `UTC`, `America/Los_Angeles`. Defaults to `Asia/Shanghai`.
- `RUN_AT`: Daily target time in 24‑hour `HH:MM` (e.g. `08:15`). The job will actually run only when the current time falls into the window. Manual runs ignore this restriction.
- `RUN_WINDOW_MINUTES`: Window size in minutes (integer). It allows running within ±N minutes around `RUN_AT`. The calculation wraps across midnight (e.g. 23:59 vs 00:05 is 6 minutes). Suggested 5–20.
- `RETRY`: Number of network retries (`curl --retry` with `--retry-all-errors`). Set 0 to disable. Too many retries increases runtime. Suggested 2–5.
- `RETRY_DELAY`: Seconds between retries (`curl --retry-delay`). Suggested 2–5 seconds.
- `CONNECT_TIMEOUT`: Seconds for connect phase (`curl --connect-timeout`), mainly TCP/TLS handshake. Increase to 15–30 for slow networks.
- `MAX_TIME`: Per‑request total timeout (`curl --max-time`) including connect and transfer. The attempt is aborted when reaching this limit; usually used together with `RETRY`. Suggested 20–60.
- `IKUUU_BASE_URL`: Base site for check‑in requests. Defaults to `https://ikuuu.de`. If the service migrates to another domain or you use a mirror, change this variable accordingly; cookies must be obtained from the same domain.

Where to configure: Repository Settings → Secrets and variables → Actions → Variables (organization‑level variables also work).
Note: obtain cookies from the same domain as `IKUUU_BASE_URL` (open `${IKUUU_BASE_URL}/user` and copy Cookie from Request Headers).

## Logic
- Schedule: `15 0 * * *` (UTC)
- Run only in window and once per day (cache)
- Success: `ret == 1`, or `ret == 0` with message like "already checked in"
- On failure: job marked failed; details listed in summary

## Disclaimer
This project and examples are for study only. Do not use in violation of ToS or laws. All examples use placeholders.
