# اجرای xWeek روی کانتینر ابری آروان

این نسخه با یک پچ کوچک، فایل‌های فرانت‌اند را هم از همان سرور Node سرو می‌کند و یک لایه احراز هویت Basic دارد. بدون این پچ، نسخه اصلی گیت‌هاب روی کانتینر بالا نمی‌آید.

## ۱) ساخت ایمیج

پیش‌نیاز: Docker Desktop (یا Docker با buildx). روی مک Apple Silicon حتماً `--platform linux/amd64` را نگه دار.

```bash
docker login registry.apps.ir-central1.arvancaas.ir
docker buildx build --platform linux/amd64 \
  --build-arg BASE_IMAGE=docker.arvancloud.ir/node:22-bookworm-slim \
  -t registry.apps.ir-central1.arvancaas.ir/xweek:v1 --push .
```

اگر آدرس رجیستری پنل شما متفاوت است، همان را جایگزین کنید.

## ۲) ساخت اپ در پنل

کانتینر ابری → ساخت اپلیکیشن → ایمیج کانتینر → ایمیج خصوصی:

- Image: `registry.apps.ir-central1.arvancaas.ir/xweek:v1`
- Port: `80`
- Disk: 1 GB، مسیر Mount: `/data`
- منابع: 1 CPU، 1 GB RAM، 1 GB ephemeral
- Replica: 1 (Multi-Zone خاموش؛ SQLite با چند نسخه سازگار نیست)
- Domain: زیردامنه رایگان آروان

متغیرهای محیطی:

| Key | Value |
|---|---|
| `OPENAI_API_KEY` | کلید Endpoint سرویس AIaaS |
| `OPENAI_BASE_URL` | آدرس Endpoint (بدون `/chat/completions`) |
| `OPENAI_MODEL` | نام دقیق مدل در پنل |
| `OPENAI_AUTH_SCHEME` | فقط اگر ۴۰۱ گرفتی: `apikey` |
| `APP_USER` | نام کاربری دلخواه |
| `APP_PASS` | رمز قوی |
| `HOST` | `0.0.0.0` |
| `PORT` | `80` |
| `DATABASE_PATH` | `/data/xweek.db` |
| `NODE_ENV` | `production` |

## ۳) بعد از بالا آمدن

```bash
# تنظیم منطقه زمانی (TZ در کد اثر ندارد)
curl -u USER:PASS -X PATCH https://YOUR-DOMAIN/api/settings \
  -H 'Content-Type: application/json' \
  -d '{"timezone":"Asia/Tehran"}'

# تست مدل و tool calling از کنسول اپ
node scripts/check-ai.js
```

## ۴) نگهداری

- از دیسک `/data` در پنل اسنپ‌شات بگیر.
- برای آپدیت: ایمیج با تگ جدید بساز، Push کن، در اپ همان تگ را Deploy کن.
- فقط یک Replica؛ SQLite و دیسک RWO.
