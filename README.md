# 👁️ GooEye

**Advanced real-time Goofish.com (闲鱼) sniper bot with multi-user isolation, human-like behavior, and bilingual notifications.**

[![Python](https://img.shields.io/badge/Python-3.12+-blue.svg)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Production_Ready-green.svg)]()

GooEye is a highly asynchronous monitoring bot that tracks new listings on the Chinese P2P platform Goofish in real-time. It features **complete user isolation**, advanced anti-bot evasion (humanizer), and seamless session management directly via Telegram. 

Perfect for **resellers, collectors, and arbitrage traders** who need to act fast without getting banned.

---

## ✨ Features

### Core (Fully Implemented)
- **Multi-User Isolation** — Each user gets a dedicated headless Chromium profile and isolated cookie jar (`state_<user_id>.json`). Zero cross-contamination.
- **Telegram QR-Code Login** — Refresh sessions instantly. The bot generates a QR code, you scan it with your phone, and cookies are saved automatically. No SSH or manual file transfers needed.
- **Advanced Humanizer** — 4 configurable timing modes (Extreme to Ironclad), random task shuffling, simulated page scrolling, and adaptive backoff on errors to mimic real human behavior.
- **Dual Search Modes** — Monitor by **Keywords** or by **Photo** (upload an image, bot searches Goofish visually).
- **Smart Filters** — Price ranges, keyword inclusion/exclusion (strictly in titles to avoid spam), and configurable "freshness windows".
- **Rich Media Notifications** — High-quality photos, bilingual (ZH/RU) side-by-side translations, and inline action buttons.
- **Robust Deduplication** — SQLite database with TTL cleanup, ensuring you never get alerted for the same item twice.
- **Security** — Custom logging filters to automatically mask `BOT_TOKEN` and sensitive data.

### Roadmap (Future Enhancements)
- **Residential Proxy Rotation** — Fallback proxy switching on `429 Too Many Requests` or CAPTCHA triggers.
- **Price Drop Tracking** — Re-alerting when a previously seen item decreases in price.
- **Web Dashboard** — Optional lightweight UI for managing tasks and viewing analytics (beyond Telegram).

*(Note: Docker was explicitly excluded from the architecture to maintain minimal RAM/Disk footprint on small VPS environments, favoring native `systemd` + `venv` pragmatism).*

---

## 🏗️ Architecture

```text
[ Telegram Users ] <──(Polling/Commands)──> [ Telegram Bot Core (PTB v21+) ]
                                                    │
                                                    ▼
                                           [ Monitor Orchestrator ]
                                                    │
          ┌─────────────────────────────────────────┼─────────────────────────────────────────┐
          ▼                                         ▼                                         ▼
[Deduplication DB]                     [ Smart Filters ]                         [ Translator (ZH→RU) ]
 (SQLite, auto-cleanup)              (Price, Keywords, Freshness)                   (On-demand)
                                                    │
                                                    ▼
                                     [ Multi-User Stealth Scraper Engine ]
                                     (Dict of Playwright instances per user_id)
                                     (Humanizer: random delays, scroll, shuffle)
                                                    │
                                                    ▼
                                     [ Goofish (闲鱼) Platform API/DOM ]
```

---

## 📋 Requirements

- **Python 3.12+**
- **Playwright** — for headless browser automation
- **Telegram Bot Token** — from [@BotFather](https://t.me/BotFather)
- **Linux VPS** (Ubuntu 22.04/24.04 recommended, min. 2GB RAM) for 24/7 `systemd` deployment

---

## 🚀 Quick Start

```bash
# 1. Clone and setup environment
git clone https://github.com/sleepti3ht/gooeye.git
cd gooeye
python3 -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# 2. Install dependencies and browser binaries
pip install -r requirements.txt
python -m playwright install chromium --with-deps

# 3. Configure
cp .env.example .env
# Edit .env with your BOT_TOKEN, ALLOWED_USER_IDS, and preferred TIMING_MODE

# 4. Run locally
python main.py
```

> **Production Deployment:** The repository includes a ready-to-use `gooeye.service` file for `systemd`, ensuring auto-start on boot and automatic restarts on failure.

---

## 📁 Project Structure

```text
gooeye/
├── main.py                  # Entry point, asyncio lifecycle management
├── monitor.py               # Main orchestrator (manages dict of user scrapers)
├── tasks.py                 # Centralized task management (tasks.json with user_id)
├── config.py                # Settings validation and loading from .env
│
├── bot/                     
│   ├── handlers.py          # Telegram commands, conversations, and menus
│   ├── notifications.py     # Rich media card formatting and retry logic
│   └── qr_login.py          # Headless QR-code generation and cookie extraction
│
├── scraper/                 
│   ├── browser.py           # Multi-user BrowserManager (isolated profiles)
│   ├── goofish_scraper.py   # DOM parsing, mtop API interception, humanizer
│   └── anti_bot.py          # Stealth scripts and evasion helpers
│
├── filters/                 # Logic for price and strict keyword filtering
├── translator/              # Module for ZH → RU title/description translation
├── database/                # SQLite deduplication and old record cleanup
├── images/                  # Stored user-uploaded images for photo-search tasks
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
🕒 Posted: ~15 мин. назад
🔗 [Open Listing] [Translate Full]
```

---
## 📊 Performance & Anti-Bot Efficacy

Live metrics from production deployment (bot currently paused after 2+ days of continuous operation):

| Metric | Value |
|--------|-------|
| **Total items scanned** | 44,919 |
| **Total scan cycles** | ~1,497 |
| **Notifications sent** | 26 (100% delivery rate) |
| **CAPTCHAs triggered** | 0 |
| **HTTP 429 errors** | 0 |
| **Avg scan duration** | 29.1s (includes humanizer delays) |
| **Page load time** | 0.58s |
| **mtop API response** | 1.45s |
| **DOM parsing** | 0.03s |
| **Items per scan** | ~30 |
| **Scan interval** | 152s |
| **Active session cookies** | 6 (stable) |

### Latency Breakdown (per scan cycle)
| Component | Time | % of Total |
|-----------|------|------------|
| Page Load | 0.58s | 2% |
| mtop API Wait | 1.45s | 5% |
| Humanizer Delays | ~27s | 93% |
| DOM Parsing | 0.03s | <1% |
| **Total** | **29.1s** | 100% |

### Key Insights
- **Zero anti-bot triggers:** 44,919 items scanned across ~1,500 cycles with **0 CAPTCHAs** and **0 rate limits**. Humanizer strategy (random delays, scroll simulation, task shuffling) successfully mimics real user behavior.
- **High notification accuracy:** 26 matches detected, 26 notifications sent — **100% delivery rate** with no false positives or missed alerts.
- **Stable sessions:** Browser cookies persist across restarts (6 active cookies), no re-authentication required during 2+ day runtime.
- **Intentional latency:** The 29s avg scan time is a **design choice**, not a bottleneck. ~93% of cycle time is spent on humanizer delays to avoid triggering Alibaba's `baxia`/`fireyejs` anti-fraud systems. Raw parsing takes only 0.03s.

### Comparison: GooEye vs Typical Scrapers
| Approach | CAPTCHAs per 10k requests | Avg Latency | Session Stability |
|----------|---------------------------|-------------|-------------------|
| **GooEye (Humanizer)** | 0 | 29s/cycle | 2+ days |
| Aggressive scraper | 50-200 | 2-5s/cycle | 10-30 min |
| Headless browser (no stealth) | 100-500 | 1-3s/cycle | 5-15 min |
---

## 📈 Development Phases

- [x] **Phase 1: MVP** — Basic Playwright scraper, HTML parsing, SQLite deduplication.
- [x] **Phase 2: Core Features** — Translation layer, rich media cards, user allowlist, freshness filters.
- [x] **Phase 3: Production Ready** — `systemd` deployment, persistent cookie profiles, advanced Humanizer (4 modes).
- [x] **Phase 4: Multi-User & UX** — Isolated profiles per `user_id`, Telegram QR-Code login, Photo-based search tasks.
- [ ] **Phase 5: Scale & Resilience** — Residential proxy rotation fallback, price-drop tracking, advanced analytics.

---
## 🔧 Troubleshooting

### Browser crashes after 24h
- **Cause:** Memory leak in Playwright
- **Solution:** systemd service auto-restarts every 12h (`Restart=always`)

### CAPTCHA appears
- **Cause:** Timing mode too aggressive
- **Solution:** Switch to `HUMAN` or `IRONCLAD` mode in `.env`

### QR code expires
- **Cause:** Network latency
- **Solution:** Re-run `/login` command, ensure VPS has stable connection to China
---

## 🔐 Security & Architecture Philosophy

- **Pragmatic Solo-Dev Design:** Explicitly avoids Docker overhead. Native `venv` + `systemd` provides maximum performance and minimal RAM/Disk usage on small VPS instances (e.g., 4GB RAM can comfortably handle 10+ isolated user browsers).
- **Respectful Scraping:** Built-in adaptive delays, task shuffling, and scroll simulation to avoid triggering Alibaba's `baxia`/`fireyejs` anti-fraud systems.
- **Zero Secrets in Repo:** All sensitive data is strictly managed via `.env` and masked in console logs.

---

## 📧 Contact

- **Telegram:** @sleept1ght
- **Email:** sleepti3ht@gmail.com
