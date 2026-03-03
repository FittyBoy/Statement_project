# 📊 StatementExcel — PDF ธนาคาร → Excel

แปลง PDF Statement จากธนาคารไทยเป็น Excel อัตโนมัติ
รองรับ KBank · SCB · BBL · KTB

---

## 🚀 โครงสร้างโปรเจกต์

```
pdf2excel/
├── frontend/
│   └── index.html          ← Landing page + Web App UI (ไฟล์เดียวใช้ได้เลย)
├── backend/
│   ├── main.py             ← FastAPI server + PDF parsers + Excel generator
│   └── requirements.txt    ← Python dependencies
└── README.md
```

---

## ⚙️ ติดตั้งและรัน

### Backend (FastAPI)

```bash
cd backend

# สร้าง virtual env
python -m venv venv
source venv/bin/activate      # Mac/Linux
# venv\Scripts\activate       # Windows

# ติดตั้ง dependencies
pip install -r requirements.txt

# รัน server
python main.py
# หรือ
uvicorn main:app --reload --port 8000
```

Server จะรันที่ http://localhost:8000
Swagger docs: http://localhost:8000/docs

### Frontend

```bash
# เปิด frontend/index.html ด้วย browser ได้เลย
# หรือรัน local server:
cd frontend
python -m http.server 3000
# เปิด http://localhost:3000
```

---

## 🌐 Deploy Production

### Option A: Vercel (Frontend) + Railway (Backend) — แนะนำ

**Frontend → Vercel**
1. Push โค้ดขึ้น GitHub
2. ไปที่ vercel.com → Import project
3. เลือก folder `frontend/`
4. Deploy ฟรีได้เลย

**Backend → Railway**
1. ไปที่ railway.app → New Project
2. Deploy from GitHub → เลือก `backend/`
3. Railway จะ detect Python อัตโนมัติ
4. Set environment variables ถ้าต้องการ
5. ได้ URL เช่น `https://your-app.railway.app`

6. แก้ไข frontend/index.html บรรทัด API_URL:
```js
const API_URL = 'https://your-app.railway.app';
```

### Option B: Docker

```dockerfile
# backend/Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```bash
docker build -t statementexcel-api ./backend
docker run -p 8000:8000 statementexcel-api
```

### Option C: VPS (DigitalOcean/AWS/Vultr)

```bash
# ติดตั้ง nginx + uvicorn
sudo apt install nginx
pip install gunicorn

# รัน backend
gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker -b 0.0.0.0:8000

# nginx config
server {
    listen 80;
    server_name yourdomain.com;

    location /api/ {
        proxy_pass http://localhost:8000/;
    }

    location / {
        root /var/www/frontend;
        try_files $uri $uri/ /index.html;
    }
}
```

---

## 📡 API Endpoints

### POST /convert
แปลง PDF เป็น Excel

```bash
curl -X POST http://localhost:8000/convert \
  -F "file=@statement.pdf" \
  -F "bank=kbank" \
  -F "is_pro=false" \
  --output result.xlsx
```

**Parameters:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| file | PDF | ✅ | ไฟล์ PDF statement |
| bank | string | ✅ | `kbank`, `scb`, `bbl`, `ktb` |
| is_pro | boolean | ❌ | เปิด auto-categorize (Pro feature) |

**Response Headers:**
```
X-Transaction-Count: 47
X-Processing-Time: 3.2
X-Total-Debit: 15420.50
X-Total-Credit: 85000.00
```

### GET /banks
ดูรายชื่อธนาคารที่รองรับ

### GET /health
Health check

---

## 💳 Freemium Model — วิธี Implement

### แผนปัจจุบัน
| แผน | จำกัด | ราคา |
|-----|------|------|
| Free | 3 ไฟล์/เดือน | ฟรี |
| Pro | ไม่จำกัด + AI categorize | 299/เดือน |
| Business | Pro + 10 users + API | 990/เดือน |

### Stack ที่แนะนำเพิ่มเติม

**Auth + Quota Management:**
- [Supabase](https://supabase.com) — Auth + PostgreSQL ฟรี
  - เก็บ user, quota_used, plan
  - Row-level security

**Payment:**
- [Omise](https://omise.co) — รองรับบัตรเครดิต + PromptPay ไทย
- [2C2P](https://2c2p.com) — สำหรับ enterprise

**Queue (สำหรับ traffic เยอะ):**
- [Redis](https://upstash.com) + [Celery](https://docs.celeryq.dev) — queue งาน PDF
- ป้องกัน server ล่มเวลาคนอัปพร้อมกันเยอะ

---

## 🔒 ความปลอดภัย

- ไฟล์ถูกลบอัตโนมัติหลังแปลงเสร็จ
- ไม่เก็บข้อมูลทางการเงินใน database
- ใช้ HTTPS ทุก request
- File size limit 20MB
- Validate file type (PDF only)

---

## 📈 Scale รองรับ Traffic เยอะ

```
User → Cloudflare (CDN/DDoS) 
     → Vercel (Frontend, Edge) 
     → Load Balancer 
     → FastAPI instances (x3) 
     → Redis Queue 
     → Workers (PDF processing)
     → Supabase (DB + Storage)
```

---

## 🤝 ต้องการพัฒนาต่อ?

- [ ] เพิ่มธนาคาร: UOB, CIMB, Krungsri, TTB
- [ ] OCR สำหรับ PDF ที่เป็นภาพ (ใช้ Tesseract / Google Vision)
- [ ] Dashboard วิเคราะห์การใช้จ่าย
- [ ] Export เป็น Google Sheets โดยตรง
- [ ] Mobile app (React Native)
- [ ] Zapier/Make integration

---

Made with ❤️ for Thai SMEs
