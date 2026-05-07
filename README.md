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


## Tunneling / HTTPS options

Telegram Mini Apps require a public **HTTPS** URL. Pick one:

### Option A — Cloudflare Quick Tunnel (fast test)

No account required, but URL is temporary and not production-stable.

```bash
cloudflared tunnel --url http://127.0.0.1:8787 --no-autoupdate
```

Cloudflare prints a URL like:

```text
https://random-words.trycloudflare.com
```

Use that URL in `setChatMenuButton`.

> Good for testing. Not recommended for permanent usage because the URL can change after restart.

### Option B — Cloudflare Named Tunnel (recommended)

Best for always-on private use with your own domain/subdomain.

High-level flow:

```bash
cloudflared tunnel login
cloudflared tunnel create vps-monitor
cloudflared tunnel route dns vps-monitor vps.example.com
```

Create `~/.cloudflared/config.yml`:

```yaml
tunnel: vps-monitor
credentials-file: /home/ubuntu/.cloudflared/<TUNNEL_ID>.json

ingress:
  - hostname: vps.example.com
    service: http://127.0.0.1:8787
  - service: http_status:404
```

Run:

```bash
cloudflared tunnel run vps-monitor
```

Then set your Telegram Mini App URL to:

```text
https://vps.example.com
```

### Option C — ngrok

Good if you already have ngrok and a static domain.

```bash
ngrok config add-authtoken <NGROK_TOKEN>
ngrok http 8787
```

For stable use, configure a static ngrok domain:

```bash
ngrok http --domain=your-static-domain.ngrok-free.app 8787
```

### Option D — Caddy/Nginx + domain

If your VPS has a public IP and domain DNS points to it, reverse proxy to local Flask:

```text
https://vps.example.com -> http://127.0.0.1:8787
```

Caddy example:

```caddyfile
vps.example.com {
  reverse_proxy 127.0.0.1:8787
}
```

## Ask an AI agent to install this app

You can paste this prompt into OpenClaw, Claude Code, Codex, Cursor, or another coding agent with VPS shell access.

```text
Install and configure Telegram VPS Monitor Mini App on this Linux VPS.

Repository:
https://github.com/adryndian/telegram-vps-monitor-miniapp

Requirements:
1. Clone repo to /opt/telegram-vps-monitor-miniapp or ~/telegram-vps-monitor-miniapp.
2. Create Python venv and install requirements.txt.
3. Create .env from .env.example.
4. Ask me for:
   - Telegram bot token
   - my Telegram numeric user ID
   - preferred public HTTPS method: Cloudflare Tunnel, ngrok, or domain reverse proxy
5. Set DASHBOARD_PASSWORD to a strong random value.
6. Set ALLOWED_TG_USER_ID to my Telegram user ID.
7. Set TELEGRAM_BOT_TOKEN to my bot token.
8. Keep TERMINAL_PASSWORD_FALLBACK=false.
9. Create a systemd user service or system service that runs:
   gunicorn -k gthread --threads 8 -b 127.0.0.1:8787 app:app
10. Start and enable the service.
11. Verify http://127.0.0.1:8787/api/metrics works.
12. Configure HTTPS tunnel/reverse proxy.
13. Use Telegram Bot API setChatMenuButton with text "VPS" and the HTTPS URL.
14. Test the Mini App from Telegram.
15. Do not commit or print secrets. Show only masked credentials.

Security:
- Never expose the app over plain HTTP publicly.
- Never commit .env.
- Terminal routes /terminal, /claude, /codex must only work for the allowlisted Telegram user.
```

## AI agent maintenance prompt

Use this when asking an AI agent to update an existing install:

```text
Update my Telegram VPS Monitor Mini App safely.

Tasks:
1. Go to the app directory.
2. Check git status and show me local changes before overwriting anything.
3. Pull latest changes from main.
4. Preserve .env.
5. Reinstall requirements if changed.
6. Restart the dashboard service.
7. Verify /api/metrics, /, /terminal, /claude, and /codex routes.
8. Confirm Telegram Mini App URL still works.
9. Do not reveal bot token, dashboard password, or tunnel credentials.
```
## License

MIT
