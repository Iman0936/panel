# 🏴‍☠️ Luffy Panel

A lightweight VLESS-over-WebSocket proxy panel built with FastAPI, deployable on [Render](https://render.com) or [Railway](https://railway.app).

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/luffy-sh-op/LUFFY_PANEL)
&nbsp;&nbsp;
[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template?template=https://github.com/luffy-sh-op/LUFFY_PANEL)

---

## ✨ Features

- **VLESS over WebSocket (TLS)** tunneling
- Multi-inbound management with per-user traffic quotas
- Connection limits per inbound (max IPs), enforced live on the proxy WebSocket handler
- Expiry date support per inbound
- Subscription link (`/sub/<uid>`) compatible with v2rayNG, Hiddify, etc.
- Clean IP / alternative address management (single or bulk delete)
- Real-time dashboard: CPU, memory, hourly traffic chart
- **Live log streaming** over WebSocket (`/ws/live-logs`)
- **Telegram bot** with full command set, inline menus, and auto alerts (quota/expiry)
- Telegram bot configurable directly from the panel UI (no env vars required)
- Bilingual UI (English / Persian) — panel and bot both
- Dark & Light mode
- Session-based authentication with password change
- **Persistent storage** to a local JSON file (`panel_db.json`)
- Keep-alive mechanism for free-tier hosting

---

## 🗂️ Project Structure

```
.
├── main.py             # FastAPI application (gateway + panel UI + Telegram bot)
├── panel_db.json        # Auto-generated persistent storage (created at runtime)
├── requirements.txt    # Python dependencies
├── render.yaml         # Render deployment config
└── Procfile             # Process entry point
```

---

## 🚀 Deploy on Render

### One-click via `render.yaml`

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/luffy-sh-op/LUFFY_PANEL)

1. Fork or push this repo to GitHub.
2. Go to [render.com](https://render.com) → **New Web Service** → connect your repo.
3. Render will auto-detect `render.yaml` and configure everything.
4. Set your `ADMIN_PASSWORD` environment variable (default: `admin`).

> 💡 **Tip:** For better speed, set the **Region** to **Frankfurt (EU)** in Render settings.

### Manual Setup

| Field | Value |
|---|---|
| **Environment** | Python |
| **Build Command** | `pip install -r requirements.txt` |
| **Start Command** | `python main.py` |

### 🌐 Render & Cloudflare Clean IPs

> **This panel on Render routes through Cloudflare's clean IPs exclusively.**
>
> Render's infrastructure sits behind Cloudflare's network, so all VLESS+WS configs will automatically use **Cloudflare clean IP ranges** — which are generally unblocked and stable in restricted regions.
>
> ✅ Use the panel URL directly — Cloudflare CDN handles routing automatically.
>
> If configs don't connect, try manually entering a known Cloudflare clean IP (e.g. `104.21.x.x` or `172.67.x.x`) in your client instead of the hostname.

> ⚠️ **Storage note:** Render's free tier filesystem is ephemeral across redeploys. `panel_db.json` will survive simple restarts of the same running instance, but a fresh deploy may reset it unless you attach a persistent disk.

---

## 🚂 Deploy on Railway

### One-click deploy

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template?template=https://github.com/luffy-sh-op/LUFFY_PANEL)

1. Fork or push this repo to GitHub.
2. Go to [railway.app](https://railway.app) → **New Project** → **Deploy from GitHub repo** → select your repo.
3. Wait for the deployment to finish. You'll be given a URL — that's your service domain. To access the panel, just add `/login` to the end of your domain.

### ⚠️ Railway IP Addresses

> **Railway does NOT use Cloudflare. It uses its own dedicated IP ranges.**
>
> Railway's outbound IPs typically fall in the range **`69.46.46.x`**, so your configs will use Railway's own IPs — not Cloudflare's. These may or may not be accessible depending on your network restrictions.
>
> **If configs don't work on Railway:**
> 1. Check whether the `69.46.46.x` range is reachable from your network.
> 2. Enable **Fragment Mode** in your v2ray / v2rayNG client (see section below).
> 3. Switch to Render for Cloudflare clean IP routing.

---

## 🔧 Fragment Mode (v2rayNG / v2ray)

If your configurations are not connecting — especially on Railway — enable **Fragment Mode** in your client:

**v2rayNG (Android):**
1. Go to **Settings → Fragment**
2. Enable Fragment and set: Packets `tlshello`, Length `10-30`, Interval `10-20`
3. Reconnect

**v2ray (Desktop):** Add to your `outbound` → `streamSettings`:

```json
"sockopt": {
  "dialerProxy": "fragment",
  "tcpKeepAliveIdle": 100
}
```

Fragment mode splits the TLS ClientHello packet to bypass deep packet inspection (DPI) firewalls.

---

## ▶️ Run Locally

```bash
pip install -r requirements.txt
python main.py
```

Panel will be available at: `http://localhost:8000/login`

> After deploying on Render or Railway, access your panel at: `https://yourdomain/login`

---

## ⚙️ Environment Variables

| Variable | Description | Default |
|---|---|---|
| `ADMIN_PASSWORD` | Panel login password | `admin` |
| `SECRET_KEY` | Session & hash secret (auto-generated) | random |
| `PORT` | Server port | `8000` |

> ⚠️ **Change `ADMIN_PASSWORD` before deploying to production.**
>
> 💡 Telegram bot token and admin ID are **no longer required as env vars** — configure them directly from the panel's Settings section (see [Telegram Bot](#-telegram-bot) below). They're saved into `panel_db.json` and the bot restarts automatically when updated.

---

## 📦 Dependencies

```
fastapi==0.104.1
uvicorn==0.24.0
websockets==12.0
httpx==0.25.1
psutil==5.9.6
pyTelegramBotAPI
```

> `pyTelegramBotAPI` is optional — if it's not installed, the app runs fine without the Telegram bot feature and logs a warning on startup.

---

## 📌 Static IPs

| Platform | Static IP? | Notes |
|---|---|---|
| **Render** (Free) | ❌ No | Shared Cloudflare IPs; clean and stable |
| **Render** (Paid) | ✅ Yes | Available on Starter plan and above |
| **Railway** | ✅ Optional | Enable via Settings → Networking → Static IP (paid feature) |

---

## 🔌 API Endpoints

### Auth
| Method | Path | Description |
|---|---|---|
| `POST` | `/api/login` | Login with password |
| `POST` | `/api/logout` | Logout |
| `GET` | `/api/me` | Check session status |
| `POST` | `/api/change-password` | Change admin password |

### Panel Settings (Telegram bot config)
| Method | Path | Description |
|---|---|---|
| `GET` | `/api/settings` | Get current Telegram bot token & admin ID |
| `POST` | `/api/settings` | Update Telegram bot token/admin ID — bot restarts automatically |

### Inbounds
| Method | Path | Description |
|---|---|---|
| `GET` | `/api/links` | List all inbounds |
| `POST` | `/api/links` | Create new inbound |
| `PATCH` | `/api/links/{uid}` | Edit inbound (active state, quota, label, max connections, expiry, usage reset) |
| `DELETE` | `/api/links/{uid}` | Delete inbound (also force-closes its active connections) |

### Subscription
| Method | Path | Description |
|---|---|---|
| `GET` | `/sub/{uid}` | Base64 subscription (v2ray/Hiddify compatible) |

### Clean IPs
| Method | Path | Description |
|---|---|---|
| `GET` | `/api/addresses` | List alternative addresses |
| `POST` | `/api/addresses` | Add address |
| `DELETE` | `/api/addresses/{index}` | Remove a single address by index |
| `DELETE` | `/api/addresses` | Remove **all** addresses at once |

### System
| Method | Path | Description |
|---|---|---|
| `GET` | `/stats` | Server stats (auth required) |
| `GET` | `/health` | Health check |
| `WS` | `/ws/live-logs` | Real-time log stream (last 150 lines, then live updates) |

---

## 🌐 VLESS Config Format

```
vless://<uuid>@<domain>:443?encryption=none&security=tls&type=ws&host=<domain>&path=/ws/<uuid>&sni=<domain>&fp=chrome&alpn=http/1.1#Luffy-<name>
```

---

## 🖥️ Panel Pages

| Page | Description |
|---|---|
| **Dashboard** | Traffic, uptime, CPU/memory, hourly chart, live log viewer |
| **Inbounds** | Create/edit/delete users, copy config, QR code, per-user connection limit |
| **Traffic** | Total stats |
| **Clean IP** | Manage alternative subscription addresses (single or bulk delete) |
| **Settings** | Configure Telegram bot token & admin ID |
| **Security** | Change password |

---

## 🤖 Telegram Bot

Configure the bot from the panel's **Settings** page (token + your numeric Telegram admin ID) — no redeploy needed, it connects live. The bot UI is bilingual (Persian/English, switchable via an inline button) and supports the following commands:

| Command | Description |
|---|---|
| `/start` | Welcome message with main inline menu |
| `/stats` | Server dashboard: domain, CPU, memory, uptime, active connections, total traffic, inbound count |
| `/users` | List all users with usage, expiry date, and on/off status |
| `/top` | Top 5 users ranked by traffic usage |
| `/create [name] [limit_GB] [days]` | Create a new inbound. Use `0` for unlimited quota / no expiry. Example: `/create Ali 15 30` |
| `/addaddr [ip_or_domain]` | Add a clean IP/domain to the subscription pool |
| `/disable [username]` | Disable a user's access |
| `/enable [username]` | Re-enable a user's access |
| `/reset [username]` | Reset a user's traffic usage back to 0 |

The bot also proactively notifies the admin:
- ⚠️ **Quota alert** — sent once a user reaches their traffic limit
- ⏰ **Expiry alert** — sent once a user's subscription expires

---

## 📱 Client Setup (v2rayNG / Hiddify)

1. Open the panel and go to **Inbounds**.
2. Click **Sub** to copy the subscription URL.
3. In your client app, add a new subscription with that URL.
4. Update subscription — configs will appear automatically.

---

## ⚠️ Notes

- Inbounds, custom addresses, and panel/bot settings are persisted to a local file (`panel_db.json`) and reloaded automatically on startup.
- On ephemeral hosting (e.g. Render free tier across redeploys), this file may not survive a full redeploy — for guaranteed persistence across redeploys, attach a persistent disk or switch to an external database (e.g. SQLite on a mounted volume).
- The keep-alive task pings `/health` every 10 minutes to prevent Render free-tier spin-down.
- Per-inbound `max_connections` is enforced live: once the limit is reached, new connection attempts for that UID are rejected until an existing session disconnects.

---

## 🤝 Contributing

1. Fork the repository
2. Create a new branch: `git checkout -b feature/amazing-feature`
3. Commit your changes: `git commit -m 'Add amazing feature'`
4. Push to your branch: `git push origin feature/amazing-feature`
5. Open a **Pull Request**

---

## 📄 License

MIT — use freely, modify as needed.

---

[My Telegram channel](https://t.me/Luffy_sh_op)

---
---
---

# 🏴‍☠️ لوفی پنل

یک پنل پراکسی سبک VLESS-over-WebSocket ساخته‌شده با FastAPI، قابل استقرار روی [Render](https://render.com) یا [Railway](https://railway.app).

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/luffy-sh-op/LUFFY_PANEL)
&nbsp;&nbsp;
[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template?template=https://github.com/luffy-sh-op/LUFFY_PANEL)

---

## ✨ امکانات

- تانلینگ **VLESS روی WebSocket (TLS)**
- مدیریت چند اینباند با محدودیت ترافیک برای هر کاربر
- محدودیت تعداد اتصال (IP) برای هر اینباند، به‌صورت زنده روی هندلر پراکسی اعمال می‌شود
- پشتیبانی از تاریخ انقضا برای هر اینباند
- لینک اشتراک (`/sub/<uid>`) سازگار با v2rayNG، Hiddify و غیره
- مدیریت آی‌پی تمیز / آدرس‌های جایگزین (حذف تکی یا دسته‌جمعی)
- داشبورد لحظه‌ای: CPU، حافظه، نمودار ترافیک ساعتی
- **استریم لاگ زنده** روی WebSocket (`/ws/live-logs`)
- **ربات تلگرام** با مجموعه کامل دستورات، منوهای inline و هشدار خودکار (اتمام حجم/انقضا)
- قابلیت تنظیم ربات تلگرام مستقیم از داخل پنل (بدون نیاز به env var)
- رابط کاربری دو زبانه (فارسی / انگلیسی) — هم پنل، هم ربات
- حالت تاریک و روشن
- احراز هویت مبتنی بر session با امکان تغییر رمز
- **ذخیره‌سازی دائمی** در یک فایل JSON محلی (`panel_db.json`)
- مکانیزم keep-alive برای هاستینگ رایگان

---

## 🗂️ ساختار پروژه

```
.
├── main.py              # اپلیکیشن FastAPI (گیت‌وی + رابط پنل + ربات تلگرام)
├── panel_db.json         # ذخیره‌سازی دائمی (خودکار ساخته می‌شود)
├── requirements.txt     # وابستگی‌های پایتون
├── render.yaml          # تنظیمات استقرار Render
└── Procfile              # نقطه ورود پروسه
```

---

## 🚀 استقرار روی Render

### یک‌کلیکی با `render.yaml`

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/luffy-sh-op/LUFFY_PANEL)

1. ریپو را fork کنید یا روی GitHub آپلود کنید.
2. به [render.com](https://render.com) بروید ← **New Web Service** ← ریپو را متصل کنید.
3. Render به‌صورت خودکار `render.yaml` را شناسایی و همه چیز را تنظیم می‌کند.
4. متغیر `ADMIN_PASSWORD` را تنظیم کنید (پیش‌فرض: `admin`).

> 💡 **نکته:** برای سرعت بهتر، **Region** را روی **Frankfurt (EU)** تنظیم کنید.

### تنظیم دستی

| فیلد | مقدار |
|---|---|
| **محیط** | Python |
| **دستور Build** | `pip install -r requirements.txt` |
| **دستور Start** | `python main.py` |

### 🌐 Render و آی‌پی‌های تمیز Cloudflare

> **⭐ این پنل روی Render فقط از آی‌پی‌های تمیز Cloudflare استفاده می‌کند.**
>
> زیرساخت Render پشت شبکه Cloudflare قرار دارد، بنابراین تمام کانفیگ‌های VLESS+WS به‌صورت خودکار از **آی‌پی‌های تمیز Cloudflare** عبور می‌کنند — که معمولاً آنبلاک و پایدار هستند.
>
> ✅ URL پنل را مستقیم استفاده کنید — Cloudflare CDN مسیریابی را خودکار انجام می‌دهد.
>
> اگر کانفیگ‌ها وصل نشدند، یک آی‌پی تمیز شناخته‌شده Cloudflare (مثل `104.21.x.x` یا `172.67.x.x`) را در کلاینت خود به جای hostname وارد کنید.

> ⚠️ **نکته ذخیره‌سازی:** فایل‌سیستم پلن رایگان Render بین دیپلوی‌های جدید پایدار نیست. `panel_db.json` بین ریستارت‌های ساده همون اینستنس می‌مونه، ولی با یک دیپلوی جدید ممکنه پاک بشه؛ برای پایداری کامل بین دیپلوی‌ها، یک دیسک دائمی وصل کنید.

---

## 🚂 استقرار روی Railway

### استقرار یک‌کلیکی

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template?template=https://github.com/luffy-sh-op/LUFFY_PANEL)

1. ریپو را fork کنید یا روی GitHub آپلود کنید.
2. به [railway.app](https://railway.app) بروید ← **New Project** ← **Deploy from GitHub repo** ← ریپو را انتخاب کنید.
3. صبر کنید تا deploy شود. بعد از deploy یک URL به شما داده می‌شود که آن دامنه سرویس شماست. برای ورود به پنل کافیست به آخر دامنه‌تان `/login` اضافه کنید.

### ⚠️ آی‌پی‌های Railway

> **⭐ Railway از Cloudflare استفاده نمی‌کند و از آی‌پی‌های اختصاصی خودش استفاده می‌کند.**
>
> آی‌پی‌های خروجی Railway معمولاً در رنج **`69.46.46.x`** هستند، بنابراین کانفیگ‌های شما از آی‌پی‌های خود Railway عبور می‌کنند — نه از Cloudflare. این آی‌پی‌ها ممکن است بسته به محدودیت‌های شبکه شما در دسترس باشند یا نباشند.
>
> **اگر کانفیگ‌ها روی Railway کار نکرد:**
> 1. بررسی کنید که رنج `69.46.46.x` از شبکه شما در دسترس است.
> 2. **حالت Fragment را در کلاینت v2ray / v2rayNG فعال کنید** (بخش زیر را ببینید).
> 3. برای استفاده از آی‌پی‌های تمیز Cloudflare، به Render بروید.

---

## 🔧 فعال‌کردن Fragment Mode (در v2rayNG / v2ray)

اگر کانفیگ‌ها وصل نمی‌شوند — به‌خصوص روی Railway — **حالت Fragment را فعال کنید:**

**v2rayNG (اندروید):**
1. به **Settings → Fragment** بروید
2. Fragment را فعال کنید و تنظیم کنید: Packets روی `tlshello`، Length روی `10-30`، Interval روی `10-20`
3. مجدداً وصل شوید

**v2ray (دسکتاپ):** به `outbound` → `streamSettings` اضافه کنید:

```json
"sockopt": {
  "dialerProxy": "fragment",
  "tcpKeepAliveIdle": 100
}
```

حالت Fragment بسته TLS ClientHello را تقسیم می‌کند تا از فایروال‌های DPI عبور کند.

---

## ▶️ اجرای محلی

```bash
pip install -r requirements.txt
python main.py
```

پنل در این آدرس در دسترس است: `http://localhost:8000/login`

> بعد از استقرار روی Render یا Railway، از این آدرس وارد پنل شوید: `https://yourdomain/login`

---

## ⚙️ متغیرهای محیطی

| متغیر | توضیح | پیش‌فرض |
|---|---|---|
| `ADMIN_PASSWORD` | رمز ورود به پنل | `admin` |
| `SECRET_KEY` | مخفی session و هش (خودکار تولید می‌شود) | تصادفی |
| `PORT` | پورت سرور | `8000` |

> ⚠️ **بعد از استقرار در محیط عمومی، `ADMIN_PASSWORD` را تغییر دهید.**
>
> 💡 توکن و آیدی ادمین ربات تلگرام دیگه **نیازی به env var نیست** — مستقیم از بخش تنظیمات پنل (به بخش [ربات تلگرام](#-ربات-تلگرام) مراجعه کنید) قابل تنظیمه. این مقادیر در `panel_db.json` ذخیره می‌شن و با هر تغییر، ربات خودکار ری‌استارت می‌شه.

---

## 📦 وابستگی‌ها

```
fastapi==0.104.1
uvicorn==0.24.0
websockets==12.0
httpx==0.25.1
psutil==5.9.6
pyTelegramBotAPI
```

> نصب `pyTelegramBotAPI` اختیاریه — اگر نصب نباشه، برنامه بدون قابلیت ربات تلگرام به‌درستی اجرا می‌شه و فقط یک هشدار موقع استارت نمایش می‌ده.

---

## 📌 آی‌پی استاتیک

| پلتفرم | آی‌پی استاتیک؟ | توضیحات |
|---|---|---|
| **Render** (رایگان) | ❌ خیر | آی‌پی‌های مشترک Cloudflare؛ تمیز و پایدار |
| **Render** (پولی) | ✅ بله | از پلان Starter به بالا در دسترس |
| **Railway** | ✅ اختیاری | از طریق Settings → Networking → Static IP فعال شود (ویژگی پولی) |

---

## 🔌 مسیرهای API

### احراز هویت
| متد | مسیر | توضیح |
|---|---|---|
| `POST` | `/api/login` | ورود با رمز |
| `POST` | `/api/logout` | خروج |
| `GET` | `/api/me` | بررسی وضعیت session |
| `POST` | `/api/change-password` | تغییر رمز ادمین |

### تنظیمات پنل (کانفیگ ربات تلگرام)
| متد | مسیر | توضیح |
|---|---|---|
| `GET` | `/api/settings` | گرفتن توکن و آیدی ادمین تلگرام فعلی |
| `POST` | `/api/settings` | به‌روزرسانی توکن/آیدی ادمین — ربات خودکار ری‌استارت می‌شود |

### اینباندها
| متد | مسیر | توضیح |
|---|---|---|
| `GET` | `/api/links` | لیست همه اینباندها |
| `POST` | `/api/links` | ایجاد اینباند جدید |
| `PATCH` | `/api/links/{uid}` | ویرایش اینباند (وضعیت فعال، حجم، نام، محدودیت اتصال، انقضا، صفر کردن مصرف) |
| `DELETE` | `/api/links/{uid}` | حذف اینباند (اتصالات فعال آن نیز فوراً قطع می‌شود) |

### اشتراک
| متد | مسیر | توضیح |
|---|---|---|
| `GET` | `/sub/{uid}` | اشتراک Base64 (سازگار با v2ray/Hiddify) |

### آی‌پی تمیز
| متد | مسیر | توضیح |
|---|---|---|
| `GET` | `/api/addresses` | لیست آدرس‌های جایگزین |
| `POST` | `/api/addresses` | افزودن آدرس |
| `DELETE` | `/api/addresses/{index}` | حذف یک آدرس بر اساس اندیس |
| `DELETE` | `/api/addresses` | حذف **همه** آدرس‌ها با یک درخواست |

### سیستم
| متد | مسیر | توضیح |
|---|---|---|
| `GET` | `/stats` | آمار سرور (نیاز به احراز هویت) |
| `GET` | `/health` | بررسی سلامت سرور |
| `WS` | `/ws/live-logs` | استریم لاگ زنده (۱۵۰ خط آخر، سپس به‌روزرسانی لحظه‌ای) |

---

## 🌐 فرمت کانفیگ VLESS

```
vless://<uuid>@<domain>:443?encryption=none&security=tls&type=ws&host=<domain>&path=/ws/<uuid>&sni=<domain>&fp=chrome&alpn=http/1.1#Luffy-<name>
```

---

## 🖥️ صفحات پنل

| صفحه | توضیح |
|---|---|
| **داشبورد** | ترافیک، آپتایم، CPU/حافظه، نمودار ساعتی، نمایش لاگ زنده |
| **اینباندها** | ایجاد/ویرایش/حذف کاربر، کپی کانفیگ، کد QR، محدودیت اتصال هر کاربر |
| **ترافیک** | آمار کلی |
| **آی‌پی تمیز** | مدیریت آدرس‌های جایگزین اشتراک (حذف تکی یا دسته‌جمعی) |
| **تنظیمات** | کانفیگ توکن و آیدی ادمین ربات تلگرام |
| **امنیت** | تغییر رمز |

---

## 🤖 ربات تلگرام

ربات را از صفحه **تنظیمات** پنل کانفیگ کنید (توکن + آیدی عددی تلگرام شما به‌عنوان ادمین) — نیازی به دیپلوی مجدد نیست، به‌صورت زنده وصل می‌شود. رابط ربات دو زبانه است (فارسی/انگلیسی، با یک دکمه inline قابل تغییر) و دستورات زیر را پشتیبانی می‌کند:

| دستور | توضیح |
|---|---|
| `/start` | پیام خوش‌آمد به همراه منوی اصلی inline |
| `/stats` | داشبورد سرور: دامنه، CPU، رم، آپ‌تایم، اتصالات فعال، ترافیک کل، تعداد اینباند |
| `/users` | لیست همه کاربران با میزان مصرف، تاریخ انقضا و وضعیت روشن/خاموش |
| `/top` | ۵ کاربر برتر بر اساس میزان مصرف ترافیک |
| `/create [نام] [حجم_GB] [روز]` | ساخت اینباند جدید. از `0` برای نامحدود/بدون انقضا استفاده کنید. مثال: `/create Ali 15 30` |
| `/addaddr [آی‌پی_یا_دامنه]` | افزودن آی‌پی/دامنه تمیز به استخر اشتراک |
| `/disable [نام‌کاربری]` | غیرفعال کردن دسترسی کاربر |
| `/enable [نام‌کاربری]` | فعال‌سازی مجدد دسترسی کاربر |
| `/reset [نام‌کاربری]` | صفر کردن مصرف ترافیک کاربر |

ربات همچنین به‌صورت خودکار به ادمین اطلاع می‌دهد:
- ⚠️ **هشدار اتمام حجم** — وقتی کاربر به سقف مصرفش برسد
- ⏰ **هشدار انقضا** — وقتی اشتراک کاربر تمام شود

---

## 📱 راه‌اندازی کلاینت (v2rayNG / Hiddify)

1. پنل را باز کنید و به **اینباندها** بروید.
2. روی **Sub** کلیک کنید تا لینک اشتراک کپی شود.
3. در اپ کلاینت، یک اشتراک جدید با آن لینک اضافه کنید.
4. اشتراک را آپدیت کنید — کانفیگ‌ها به‌صورت خودکار نمایش داده می‌شوند.

---

## ⚠️ نکات مهم

- اینباندها، آدرس‌های تمیز و تنظیمات پنل/ربات در یک فایل محلی (`panel_db.json`) ذخیره می‌شوند و موقع استارت به‌صورت خودکار بارگذاری می‌شوند.
- روی هاستینگ‌های موقتی (مثلاً پلن رایگان Render بین دیپلوی‌های جدید)، ممکن است این فایل بین دیپلوی‌ها از بین برود — برای پایداری تضمینی بین دیپلوی‌ها، یک دیسک دائمی وصل کنید یا از یک دیتابیس خارجی (مثلاً SQLite روی یک volume) استفاده کنید.
- تسک keep-alive هر ۱۰ دقیقه به `/health` پینگ می‌زند تا از خواب رفتن سرویس رایگان Render جلوگیری کند.
- محدودیت `max_connections` هر اینباند به‌صورت زنده اعمال می‌شود: وقتی به سقف برسد، تلاش‌های اتصال جدید برای آن UID رد می‌شوند تا یکی از سشن‌های فعال قطع شود.

---

## 🤝 Contributing

1. Fork the repository
2. Create a new branch: `git checkout -b feature/amazing-feature`
3. Commit your changes: `git commit -m 'Add amazing feature'`
4. Push to your branch: `git push origin feature/amazing-feature`
5. Open a **Pull Request**

---

## 📄 لایسنس

MIT — آزادانه استفاده و ویرایش کنید.

---

[چنل تلگراممون](https://t.me/Luffy_sh_op)
