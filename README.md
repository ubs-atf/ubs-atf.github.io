# Kafedra davomat tizimi — bosqichma-bosqich qoʻllanma

Tizim nima qiladi: kafedra mudiri xodim va vaqtni roʻyxatdan tanlaydi → maʼlumot bazaga yoziladi →
Kadrlar boʻlimiga va xodimning Telegram botiga avtomatik xabar ketadi → ishlagan soat/daqiqa hisoblanadi →
oy oxirida Excel hisobot tayyor.

## Loyiha tuzilmasi
```
davomat/
├─ requirements.txt     kutubxonalar roʻyxati
├─ .env.example         sozlamalar namunasi (.env ga nusxalanadi)
├─ app/
│  ├─ db.py             SQLite baza va jadvallar
│  ├─ report.py         ishlagan vaqtni hisoblash, oylik hisobot, Excel
│  ├─ bot.py            Telegram: xabar yuborish + xodimni kod bilan ulash
│  └─ main.py           server (API) — hamma narsani bogʻlaydi
└─ static/index.html    mudir va kadrlar uchun sayt
```

## 1-bosqich. Dasturlarni oʻrnatish
1. Python 3.10+ ni python.org dan oʻrnating (Windowsda "Add Python to PATH" ni belgilang).
2. Loyiha papkasini oching va terminalda:
   ```
   python -m venv venv
   venv\Scripts\activate          (Linux/Mac: source venv/bin/activate)
   pip install -r requirements.txt
   ```

## 2-bosqich. Telegram bot yaratish
1. Telegramda **@BotFather** ga yozing → `/newbot`.
2. Bot nomi va `..._bot` bilan tugaydigan username bering.
3. BotFather bergan **tokenni** nusxalang (keyingi bosqichda kerak).
4. (Ixtiyoriy) Kadrlar boʻlimi ham Telegramda xabar olishi uchun: botni ular bilan guruhga qoʻshing yoki
   kadrlar xodimi botga yozsin, keyin brauzerda
   `https://api.telegram.org/bot<TOKEN>/getUpdates` ni ochib `"chat":{"id": ...}` raqamini oling → `HR_CHAT_ID`.

## 3-bosqich. Sozlash
`.env.example` faylini `.env` deb nusxalang va toʻldiring: `BOT_TOKEN`, `HEAD_PASSWORD`, `HR_PASSWORD`
(murakkab parol qoʻying!), kerak boʻlsa `WORK_START` (ish boshlanishi) va `LUNCH_MIN` (tushlik).

## 4-bosqich. Ishga tushirish
```
uvicorn app.main:app --host 0.0.0.0 --port 8000
```
Brauzerda `http://localhost:8000` ni oching. Birinchi marta `davomat.db` fayli avtomatik yaratiladi.
> Faqat **bitta** jarayon ishga tushiring (`--workers` ishlatmang) — bot bitta nusxada ishlashi kerak.

## 5-bosqich. Xodimlarni kiritish va botga ulash
1. Kadrlar paroli bilan kiring → "Xodimlar va Telegram ulanishi" → F.I.Sh. va lavozimni qoʻshing.
2. Har bir xodimga **6 xonali kod** beriladi. Xodim botni Telegramda ochib shu kodni yuboradi.
3. Bot "✅ ulandingiz" deydi, jadvalda holat "ulangan" boʻladi. Shundan keyin xabarlar unga keladi.

## 6-bosqich. Sinov
1. Mudir paroli bilan kiring → xodim, "Ishga keldi", soat va minutni tanlab "Yuborish".
2. Xodim Telegramida xabar kelishini, kadrlar oynasida jurnal va hisobotni tekshiring.
3. Kechqurun "Ishdan ketdi" ni yuboring — xodimga ishlagan vaqti (masalan, 8 soat 05 daq) yoziladi.

## 7-bosqich. Doimiy ishlashi uchun serverga joylash
Variant A — universitet/kafedra kompyuteri: shu kompyuter yoqiq turishi va foydalanuvchilar tarmogʻida
boʻlishi kerak (`http://KOMPYUTER-IP:8000`). Windowsda "Task Scheduler" bilan avto-ishga tushirish mumkin.

Variant B — VPS (Ubuntu), tavsiya etiladi. `/etc/systemd/system/davomat.service`:
```
[Unit]
Description=Davomat
After=network.target
[Service]
WorkingDirectory=/opt/davomat
ExecStart=/opt/davomat/venv/bin/uvicorn app.main:app --host 127.0.0.1 --port 8000
Restart=always
[Install]
WantedBy=multi-user.target
```
`systemctl enable --now davomat`, soʻng Nginx + Let's Encrypt (HTTPS) ulang. HTTPSsiz parollar ochiq ketadi!

## 8-bosqich. Xavfsizlik va zaxira
- Parollarni murakkab qiling, `.env` ni hech kimga bermang.
- `davomat.db` faylidan har kuni nusxa oling (cron yoki qoʻlda).
- Xodimlarning shaxsiy maʼlumotlari saqlanadi — foydalanuvchilar sonini cheklang.

## 9-bosqich. Keyingi rivojlantirish
Bayram/dam olish kunlari, kasallik/komandirovka belgisi, har bir xodim uchun alohida ish jadvali,
mudirga kunlik xulosa xabari, bir nechta kafedra, hisobotni PDF qilish.

## Hisoblash qoidasi
Ishlagan vaqt = ketish − kelish − tushlik (`LUNCH_MIN`). Kelish `WORK_START` dan keyin boʻlsa — kechikish.
