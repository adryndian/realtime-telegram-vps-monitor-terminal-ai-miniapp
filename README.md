# Telegram VPS Monitor Mini App

Lightweight internal VPS monitor that runs as a Telegram Mini App.

Built for private server operators who want a small dashboard inside Telegram:

- CPU, load, RAM, disk, uptime
- service status checks
- top processes
- optional web terminal
- Claude Code / Codex CLI launch buttons
- Telegram Mini App auth via `initData`
- Flask + vanilla JS, no frontend build step

> ⚠️ Security note: the terminal feature gives shell access to your VPS. Use only behind Telegram `initData` validation, a private tunnel/access layer, and your own allowlisted Telegram user ID.

## Stack

- Python Flask
- Flask-Sock WebSocket terminal
- Gunicorn
- Vanilla HTML/CSS/JS
- xterm.js vendored under `static/vendor/`

## Quick start

```bash
git clone https://github.com/YOUR_USERNAME/telegram-vps-monitor-miniapp.git
cd telegram-vps-monitor-miniapp
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
nano .env
python app.py
```

Open locally:

```text
http://127.0.0.1:8787
```

## Environment

See `.env.example`.

Important fields:

- `DASHBOARD_PASSWORD` — fallback dashboard password
- `ALLOWED_TG_USER_ID` — only this Telegram user can use the Mini App auth
- `TELEGRAM_BOT_TOKEN` — optional; used for Mini App auth if not integrated with another local config
- `TERMINAL_PIN` — optional extra terminal PIN
- `TERMINAL_PASSWORD_FALLBACK=false` — recommended

## Telegram Mini App setup

Your dashboard must be served via HTTPS.

Examples:

- Cloudflare Tunnel
- Tailscale Funnel
- ngrok static domain
- domain + Caddy/Nginx HTTPS

Set the bot menu button:

```bash
curl -X POST "https://api.telegram.org/bot$BOT_TOKEN/setChatMenuButton" \
  -H "Content-Type: application/json" \
  -d '{
    "menu_button": {
      "type": "web_app",
      "text": "VPS",
      "web_app": {"url": "https://your-domain.example"}
    }
  }'
```

For a specific chat:

```bash
curl -X POST "https://api.telegram.org/bot$BOT_TOKEN/setChatMenuButton" \
  -H "Content-Type: application/json" \
  -d '{
    "chat_id": 123456789,
    "menu_button": {
      "type": "web_app",
      "text": "VPS",
      "web_app": {"url": "https://your-domain.example"}
    }
  }'
```

## systemd user service

Example:

```ini
[Unit]
Description=Telegram VPS Monitor Mini App
After=network.target

[Service]
Type=simple
WorkingDirectory=/path/to/telegram-vps-monitor-miniapp
EnvironmentFile=/path/to/telegram-vps-monitor-miniapp/.env
ExecStart=/path/to/telegram-vps-monitor-miniapp/.venv/bin/gunicorn -k gthread --threads 8 -b 127.0.0.1:8787 app:app
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

## Routes

- `/` — monitor dashboard
- `/terminal` — shell terminal
- `/claude` — terminal launching Claude Code CLI
- `/codex` — terminal launching Codex CLI
- `/api/metrics` — JSON metrics

## Security checklist

- Do not commit `.env`
- Do not expose without HTTPS
- Set `ALLOWED_TG_USER_ID`
- Prefer `TERMINAL_PASSWORD_FALLBACK=false`
- Use private access layer if possible
- Rotate tunnel URLs/passwords if leaked
- Do not share screenshots containing public tunnel URLs

## License

MIT
