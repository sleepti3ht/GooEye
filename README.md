# 👁️ GooEye

**Real-time Goofish.com sniper bot with intelligent monitoring and bilingual notifications.**

[![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Development-yellow.svg)]()


GooEye is an asynchronous monitoring bot that tracks new listings on Goofish.com in real-time. It bypasses anti-bot protection, translates Chinese listings to Russian/English, and sends instant Telegram notifications with rich media cards.

Perfect for **resellers, collectors, and arbitrage traders** who need to act fast.

---

##  Features

### Core
- **Real-time Monitoring** — 15-30 second polling with intelligent rate limiting
- **Anti-Bot Protection** — Playwright + rotating proxies + user-agent rotation + behavior emulation
- **Bilingual Notifications** — Original Chinese + Russian/English translation side-by-side
- **Smart Filters** — Keywords, price range, categories, exclude rules
- **Rich Media Cards** — Photos, formatted text, inline buttons in Telegram
- **Deduplication** — SQLite database prevents duplicate notifications

### Advanced
- **Cloudflare Bypass** — Headless browser with stealth mode
- **CAPTCHA Handling** — Optional 2captcha integration
- **Price Parsing** — Multi-currency support (¥, $, ₽, €) with auto-conversion
- **Intelligent Polling** — Random delays (±20%) + exponential backoff on 429/403
- **User Allowlist** — Only authorized Telegram users can access the bot
- **Health Checks** — Auto-restart on failures, logging with metrics

---

## 🏗️ Architecture

```
┌─────────────────┐
│  Goofish.com    │
│  (Anti-Bot)     │
└────────────────┘
         │
         ▼
┌─────────────────┐
│  Playwright     │ ← Rotating Proxies + User-Agents
│  Scraper        │ ← Behavior Emulation
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Data Extract   │ → Title, Price, Photos, Description
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Translation    │ → CN → RU/EN (Google Translate API)
│  Layer          │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Filter Engine  │ → Keywords, Price, Categories
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Deduplication  │ → SQLite (item_id + TTL)
────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Telegram Bot   │ → Media Group + Inline Buttons
└─────────────────┘
```

---

## 📋 Requirements

- **Python 3.11+**
- **Playwright** — for browser automation
- **Telegram Bot Token** — from [@BotFather](https://t.me/BotFather)
- **Proxy Service** (optional but recommended) — Bright Data, Oxylabs, or free proxies
- **Google Translate API** (optional) — or use free `deep-translator`

---

## 🚀 Quick Start

### 1. Install Dependencies

```bash
git clone https://github.com/sleepti3ht/gooeye.git
cd gooeye
pip install -r requirements.txt
playwright install
```

### 2. Configure

Create `.env` file:

```bash
# Telegram
TELEGRAM_BOT_TOKEN=your_bot_token_here
ALLOWED_USERS=123456789,987654321  # Telegram user IDs

# Goofish
GOOFISH_CATEGORY_URL=https://goofish.com/category/your-category
POLL_INTERVAL=20  # seconds

# Translation
TRANSLATE_TO=ru  # ru, en, or both

# Filters
MIN_PRICE=0
MAX_PRICE=100000
KEYWORDS_INCLUDE=Stone Island,CP Company
KEYWORDS_EXCLUDE=replica,fake

# Proxy (optional)
PROXY_LIST=proxy1:port,proxy2:port
USE_ROTATING_PROXIES=true

# Database
DATABASE_URL=sqlite:///gooeye.db
```

### 3. Run

```bash
python main.py
```

---

##  Project Structure

```
gooeye/
├── main.py                  # Entry point
├── config.py                # Configuration loader
├── requirements.txt
── README.md
├── .env.example
│
├── scraper/
│   ├── __init__.py
│   ├── browser.py           # Playwright setup
│   ├── goofish_scraper.py   # Main scraper
│   └── anti_bot.py          # Proxy + UA rotation
│
├── parser/
│   ├── __init__.py
│   ├── html_parser.py       # BeautifulSoup parser
│   ── data_extractor.py    # Field extraction
│
├── translator/
│   ├── __init__.py
│   └── translator.py        # CN → RU/EN
│
├── filters/
│   ├── __init__.py
│   ├── keyword_filter.py
│   ── price_filter.py
│
├── database/
│   ├── __init__.py
│   └── models.py            # SQLite models
│
├── telegram/
│   ├── __init__.py
│   ├── bot.py               # Telegram bot
│   └── notifications.py     # Rich media sender
│
└── utils/
    ├── __init__.py
    ├── logger.py
    └── rate_limiter.py
```

---

## 🎯 Example Notification

```
📦 [Stone Island] Nylon Metal Jacket
━━━━━━━━━━━━
🇨🇳 Original:
STONE ISLAND 男士夹克和外套 ME-货号:S1
54100091S0010V0093 尺码:M/L/XL/XXL
折扣:5.7折 全新正品,欧洲代购...

🇷🇺 Translation:
Stone Island Men's Jacket & Coats
Item Code: S1-54100091S0010V0093
Sizes: M/L/XL/XXL | Discount: 43% off
Brand new, authentic, European purchase...

💰 Price: ¥3156 (~$435 / ~40,000₽)
🔗 [Open Listing] [Translate Full] [Save to Favorites]
```

---

## ️ Development Roadmap

### Phase 1: MVP 
- [x] Basic Playwright scraper
- [x] HTML parsing (title, price, photos)
- [x] SQLite deduplication
- [ ] Telegram notifications (text only)
- [ ] Basic filters (keywords, price)

### Phase 2: Core Features 
- [ ] Translation layer (CN → RU/EN)
- [ ] Rich media cards (photos + formatted text)
- [ ] Inline buttons (translate, favorite, share)
- [ ] User allowlist
- [ ] Intelligent polling (random delays)

### Phase 3: Anti-Bot Hardening 
- [ ] Proxy rotation (Bright Data / Oxylabs)
- [ ] User-agent rotation
- [ ] Behavior emulation (scrolling, random clicks)
- [ ] CAPTCHA solving (2captcha API)
- [ ] Cloudflare bypass optimization

### Phase 4: Advanced Features 
- [ ] Multi-category monitoring
- [ ] Price history tracking
- [ ] Statistics dashboard (web UI)
- [ ] Auto-buy integration (if API available)
- [ ] Multi-user support (subscription tiers)

### Phase 5: Scale & Monetization (Future)
- [ ] Support for Taobao, 1688, Weidian
- [ ] Web dashboard for filter management
- [ ] Analytics: items found, conversion rate
- [ ] Premium features (priority notifications, unlimited filters)
- [ ] Mobile app (React Native / Flutter)

---

## 🔐 Security & Ethics

- **Private Use Only** — This tool is for personal monitoring, not commercial scraping
- **Respect Rate Limits** — Built-in delays prevent server overload
- **No Credentials in Repo** — All secrets in `.env` (never commit!)
- **User Allowlist** — Only authorized users can access the bot
- **Compliance** — Follow Goofish.com ToS and local laws

---

## 📊 Performance Metrics

Target benchmarks:
- **Latency:** < 15 seconds from listing to notification
- **Reliability:** 0 IP bans (thanks to proxy rotation)
- **Accuracy:** > 90% relevant items (smart filters)
- **Translation Speed:** < 2 seconds per item

---

## 🤝 Contributing

This is a private project for now. If you're interested in contributing:
1. Open an issue with your proposal
2. Wait for approval
3. Fork and create a PR

---

## 📧 Contact

- **Telegram:** @sleept1ght

---

## ⚖️ License

MIT License — use at your own risk.

---

<div align="center">

**Made with 👁️ by sleepti3ht**

*Those who know, know.*

</div>

