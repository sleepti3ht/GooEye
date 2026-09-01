
# 👁️ GooEye

**Real-time Goofish.com (闲鱼) sniper bot with intelligent monitoring and bilingual notifications.**

[![Python](https://img.shields.io/badge/Python-3.12+-blue.svg)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Active_Beta-yellow.svg)]()

GooEye is an asynchronous monitoring bot that tracks new listings on the Chinese P2P platform Goofish in real-time. It bypasses basic anti-bot protection using persistent browser profiles, translates Chinese listings to Russian, and sends instant Telegram notifications with rich media cards.

Perfect for **resellers, collectors, and arbitrage traders** who need to act fast.

---

## ✨ Features

### Core (Implemented)
- **Real-time Monitoring** — Async polling with configurable intervals (~20s).
- **Anti-Bot Protection** — Playwright with **persistent profile** (cookie/session retention) to mimic real user trust and bypass basic anti-fraud, without needing expensive rotating proxies.
- **Bilingual Notifications** — Original Chinese + Russian translation side-by-side.
- **Smart Filters** — Keywords inclusion/exclusion, price range, and "freshness window" (e.g., only items posted in the last 10 minutes).
- **Rich Media Cards** — Photos, formatted text, and inline buttons in Telegram.
- **Two-Level Deduplication** — Timestamp (`publish_ts`) check + SQLite database with 7-day TTL to prevent duplicate alerts.
- **Security** — Custom `logging.Filter` to automatically mask `BOT_TOKEN` in all console and file logs.

### Advanced (Roadmap)
- Proxy rotation (residential IPs) for advanced Cloudflare bypass.
- CAPTCHA solving integration (e.g., 2captcha).
- Auto-renewal of sessions via QR-code directly in the Telegram interface.
- Docker containerization for one-click deployment.

---

## 🏗️ Architecture

```text
[ Telegram User ] <──(Polling)──> [ Telegram Bot Core (PTB v20+) ]
                                           │
                                           ▼
                                  [ Monitor Orchestrator ]
                                           │
         ┌─────────────────────────────────┼─────────────────────────────────┐
         ▼                               ▼                               ▼
[Deduplication DB]               [Smart Filters]                [Translator]
 (SQLite, TTL 7 days)          (Price, Keywords,              (ZH → RU)
                                Freshness Window)
                                           │
                                           ▼
                              [ Stealth Scraper Engine ]
                              (Playwright + Persistent Profile)
                              (No complex proxies needed, just 
                               natural browser behavior & saved sessions)
                                           │
                                           ▼
                                  [ Goofish (闲鱼) Platform ]
```

---

## 📋 Requirements

- **Python 3.12+**
- **Playwright** — for headless browser automation
- **Telegram Bot Token** — from [@BotFather](https://t.me/BotFather)
- **Linux VPS** (Ubuntu 24.04 recommended) for 24/7 `systemd` deployment

---

## 🚀 Quick Start (Local / Dev)

```bash
# 1. Clone and setup environment
git clone https://github.com/sleepti3ht/gooeye.git
cd gooeye
python3 -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# 2. Install dependencies and browser
pip install -r requirements.txt
python -m playwright install chromium --with-deps

# 3. Configure
cp .env.example .env
# Edit .env with your BOT_TOKEN and ALLOWED_USER_IDS

# 4. Run
python main.py
```

*(For production 24/7 deployment, the project includes a ready-to-use `systemd` service configuration).*

---

## 📁 Project Structure

```text
gooeye/
├── main.py                  # Entry point, asyncio lifecycle management
├── monitor.py               # Main monitoring loop orchestrator
├── config.py                # Settings validation and loading from .env
│
├── bot/                     # Telegram handlers and rich media notifications
├── scraper/                 # Playwright setup, stealth, and DOM parsing
├── filters/                 # Logic for price and keyword filtering
├── translator/              # Module for ZH → RU title translation
├── database/                # SQLite deduplication and old record cleanup
└── utils/                   # Custom logging with SecretFilter (token masking)
```

---

## 🎯 Example Notification

```text
📦 [Stone Island] Nylon Metal Jacket
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🇨🇳 Original:
STONE ISLAND 男士夹克和外套 ME-货号:S1... 
折扣:5.7折 全新正品

🇷🇺 Translation:
Мужские куртки и пальто Stone Island 
Скидка: 43%. Абсолютно новые, оригинальные.

💰 Price: ¥3156
🔗 [Open Listing] [Translate Full]
```

---

## 📈 Development Roadmap

- [x] **Phase 1: MVP** — Basic Playwright scraper, HTML parsing, SQLite deduplication.
- [x] **Phase 2: Core Features** — Translation layer, rich media cards, user allowlist, freshness window filters.
- [x] **Phase 3: Production Ready** — `systemd` deployment, auto-restart on failure, persistent cookie profile for anti-bot.
- [ ] **Phase 4: Advanced Anti-Bot** — Residential proxy rotation, automated QR-code session renewal.
- [ ] **Phase 5: Scale** — Docker support, multi-category monitoring, web dashboard for filter management.

---

## 🔐 Security & Ethics

- **Private Use Focus** — Designed for personal monitoring with strict `ALLOWED_USER_IDS` control.
- **Respectful Scraping** — Built-in delays and persistent profiles to avoid overloading the target server.
- **Zero Secrets in Repo** — All sensitive data is strictly managed via `.env` and masked in logs.

---

## 📧 Contact

- **Telegram:** @sleept1ght
