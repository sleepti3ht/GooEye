
<div align="center">

# 👁️ GooEye

[![Python](https://img.shields.io/badge/Python-3.12+-18181b?style=flat&logo=python)](https://python.org)
[![Playwright](https://img.shields.io/badge/Playwright-Automation-18181b?style=flat&logo=playwright)](https://playwright.dev)
[![License](https://img.shields.io/badge/License-MIT-18181b?style=flat)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Production_Ready-facc15?style=flat)]()

</div>

> ⚡ **Advanced real-time Goofish.com (闲鱼) sniper bot.** Multi-user isolation, human-like behavior evasion, and bilingual notifications engineered for high-frequency arbitrage.

GooEye is a highly asynchronous monitoring system that tracks new listings on the Chinese P2P platform Goofish in real-time. It features **complete user isolation**, advanced anti-bot evasion (humanizer), and seamless session management directly via Telegram. 

Designed for **resellers, collectors, and arbitrage traders** who require sub-minute reaction times without triggering platform anti-fraud systems.

---

## 📊 Engineering Metrics & Business Value

Live metrics from a 2+ day continuous production deployment demonstrate the efficacy of the architectural choices:

| Metric | Value | Business / Engineering Impact |
|--------|-------|-------------------------------|
| **Items Scanned** | 44,919 | High-throughput monitoring capability. |
| **CAPTCHAs Triggered** | **0** | Zero account ban risk. Humanizer successfully bypasses `baxia`/`fireyejs`. |
| **HTTP 429 Errors** | **0** | Perfect rate-limit adherence. No wasted requests or temporary IP bans. |
| **Notification Delivery** | **100%** (26/26) | Maximum capture rate for arbitrage opportunities. Zero false positives. |
| **Avg Scan Duration** | 29.1s | **Intentional latency.** 93% of this time is humanizer delay, trading raw speed for session longevity. |
| **Raw DOM Parsing** | 0.03s | Highly optimized extraction logic once the page is loaded. |

---

## 🏗️ Architecture & Design Decisions

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

### ⚠️ Architectural Constraints & Mitigations
As a system handling persistent browser contexts and concurrent I/O, the following constraints are actively managed:
1. **Memory Management:** Headless Chromium instances are prone to gradual memory leaks. *Mitigation:* The architecture explicitly avoids Docker overhead in favor of native `systemd` with a configured `Restart=always` and a 12-hour lifecycle to gracefully flush V8 heap memory.
2. **Rate-Limiting (429):** Aggressive polling triggers immediate IP/session bans on Goofish. *Mitigation:* The `Humanizer` enforces a mandatory ~27s delay per cycle (mimicking human reading/scrolling time), reducing request frequency to a safe ~152s interval per task.
3. **Race Conditions:** Concurrent task updates from multiple users can corrupt shared state. *Mitigation:* All writes to `tasks.json` and the SQLite deduplication database are guarded by `asyncio.Lock` primitives to ensure serializable isolation.

---

## ✨ Core Features

- **Multi-User Isolation:** Dedicated headless Chromium profile and isolated cookie jar (`state_<user_id>.json`) per user. Zero cross-contamination.
- **Telegram QR-Code Login:** Refresh sessions instantly. The bot generates a QR code, you scan it with your phone, and cookies are saved automatically. No SSH or manual file transfers needed.
- **Advanced Humanizer:** 4 configurable timing modes (Extreme to Ironclad), random task shuffling, simulated page scrolling, and adaptive backoff on errors.
- **Dual Search Modes:** Monitor by **Keywords** or by **Photo** (upload an image, bot searches Goofish visually).
- **Smart Filters:** Price ranges, keyword inclusion/exclusion (strictly in titles to avoid spam), and configurable "freshness windows".
- **Rich Media Notifications:** High-quality photos, bilingual (ZH/RU) side-by-side translations, and inline action buttons.
- **Security:** Custom logging filters automatically mask `BOT_TOKEN` and sensitive PII in console outputs.

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

## 📱 User Interface & Observability

Built-in Telegram bot interface provides **real-time transparency** — no SSH or log parsing required:

- **Live Statistics:** Scan intervals, success rates, anti-bot counters (429/CAPTCHA).
- **Session Health Monitoring:** Cookie status, last scan timestamp, browser state.
- **Performance Breakdown:** Page load, mtop API wait, parsing time per scan.
- **One-Click Controls:** Pause/resume, QR login, task management via inline buttons.

*Example Stats View:*
![GooEye Statistics](screenshots/statistics.png)

---

## 📈 Development Phases

- [x] **Phase 1: MVP** — Basic Playwright scraper, HTML parsing, SQLite deduplication.
- [x] **Phase 2: Core Features** — Translation layer, rich media cards, user allowlist, freshness filters.
- [x] **Phase 3: Production Ready** — `systemd` deployment, persistent cookie profiles, advanced Humanizer (4 modes).
- [x] **Phase 4: Multi-User & UX** — Isolated profiles per `user_id`, Telegram QR-Code login, Photo-based search tasks.
- [ ] **Phase 5: Scale & Resilience** — Residential proxy rotation fallback, price-drop tracking, advanced analytics dashboard.

---

## 🔧 Troubleshooting

| Symptom | Root Cause | Mitigation |
|---------|------------|------------|
| **Browser crashes after 24h** | V8/Chromium memory leak over time. | Rely on `systemd` auto-restart (`Restart=always`, max 12h uptime). |
| **CAPTCHA appears** | Timing mode too aggressive for current IP reputation. | Switch to `HUMAN` or `IRONCLAD` mode in `.env`. |
| **QR code expires** | Network latency between VPS and Alibaba auth servers. | Re-run `/login` command. Ensure VPS has stable routing to China. |

---

## 🔐 Security Philosophy

- **Pragmatic Solo-Dev Design:** Explicitly avoids Docker overhead. Native `venv` + `systemd` provides maximum performance and minimal RAM/Disk usage on small VPS instances (e.g., 4GB RAM can comfortably handle 10+ isolated user browsers).
- **Respectful Scraping:** Built-in adaptive delays, task shuffling, and scroll simulation to avoid triggering anti-fraud systems.
- **Zero Secrets in Repo:** All sensitive data is strictly managed via `.env` and masked in console logs via custom `SecretFilter`.
