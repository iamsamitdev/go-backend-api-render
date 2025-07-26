# Deploy Go Backend API บน Render

คู่มือการ deploy Go Backend API บน Render cloud platform

## 📋 ข้อกำหนดเบื้องต้น

1. **Render Account**: สมัครฟรีที่ [render.com](https://render.com)
2. **GitHub Repository**: โค้ดต้องอยู่บน GitHub
3. **Database**: จะใช้ Render PostgreSQL (free tier)

## 🚀 วิธีการ Deploy

### วิธีที่ 1: ใช้ Render Dashboard (แนะนำ)

#### 1. **สร้าง PostgreSQL Database**:
1. ไปที่ Render Dashboard
2. คลิก "New +" → "PostgreSQL"
3. ตั้งค่า:
   - Name: `postgres-db`
   - Database: `ecommerce`
   - User: `ecommerce_user`
   - Region: `Singapore` (ใกล้ที่สุด)
   - Plan: `Starter (Free)`

#### 2. **สร้าง Web Service**:
1. คลิก "New +" → "Web Service"
2. เลือก "Build and deploy from a Git repository"
3. Connect GitHub repository
4. ตั้งค่า:
   - **Name**: `go-backend-api`
   - **Runtime**: `Docker`
   - **Dockerfile Path**: `./go-backend-api/Dockerfile.render`
   - **Docker Context**: `./go-backend-api`
   - **Region**: `Singapore`
   - **Plan**: `Starter (Free)`

#### 3. **ตั้งค่า Environment Variables** (ใน Render Dashboard):

⚠️ **สำคัญ**: อย่าใส่ environment variables ใน `render.yaml` เพื่อความปลอดภัย

ใน Render Dashboard → Environment Variables:

**Application Variables**:
```
APP_ENV=production
DB_SSL=require
JWT_SECRET=[สร้าง random string แกร่งๆ]
JWT_EXPIRES_IN=24h
AUTO_MIGRATE=true
ADMIN_EMAIL=admin@yourcompany.com
ADMIN_PASSWORD=[สร้าง strong password]
ADMIN_FIRST_NAME=Admin
ADMIN_LAST_NAME=User
```

**Database Variables** (จะถูกเชื่อมโยงอัตโนมัติ):
- เลือก "Link to Database" ใน Render Dashboard
- หรือตั้งแยก: `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASS`, `DB_NAME`

**วิธีสร้าง Strong Secrets**:
```bash
# JWT_SECRET (32 characters)
openssl rand -hex 32

# ADMIN_PASSWORD (16 characters)
openssl rand -base64 16
```

#### 4. **Deploy**:
1. คลิก "Create Web Service"
2. รอ build process (ประมาณ 5-10 นาที)
3. เมื่อสำเร็จจะได้ URL: `https://your-app-name.onrender.com`

### วิธีที่ 2: ใช้ render.yaml (Infrastructure as Code)

1. **Push render.yaml ไปยัง repo**
2. ใน Render Dashboard:
   - คลิก "New +" → "Blueprint"
   - เลือก repository
   - เลือกไฟล์ `go-backend-api/render.yaml`
   - คลิก "Apply"

⚠️ **หลังจากใช้ Blueprint**: ต้องตั้งค่า Environment Variables ใน Dashboard ตามขั้นตอนข้างต้น เพราะไฟล์ `render.yaml` ไม่มี sensitive data

## 🔧 การทดสอบ

### Health Check:
```bash
curl https://your-app-name.onrender.com/health
```

### API Documentation:
```bash
https://your-app-name.onrender.com/swagger/
```

### Test Endpoints:
```bash
# ดู categories
curl https://your-app-name.onrender.com/api/v1/categories

# ดู products
curl https://your-app-name.onrender.com/api/v1/products
```

## 📊 Render Free Tier Limits

### Web Service (Free):
- **RAM**: 512MB
- **CPU**: 0.1 CPU
- **Bandwidth**: 100GB/month
- **Build Time**: 500 minutes/month
- **Sleep**: หลัง 15 นาทีไม่มีการใช้งาน

### PostgreSQL (Free):
- **Storage**: 1GB
- **RAM**: 256MB
- **Connections**: 97 concurrent
- **Retention**: 90 days backup

## 🔍 Troubleshooting

### 1. **Build Failed**:
```bash
# ตรวจสอบ logs ใน Render Dashboard
# หรือ test build locally:
cd go-backend-api
docker build -f Dockerfile.render -t test-build .
```

### 2. **Database Connection Error**:
- ตรวจสอบ environment variables
- ตรวจสอบว่า PostgreSQL service รันอยู่
- ตรวจสอบ SSL requirement

### 3. **App Sleeping**:
```bash
# ใช้ uptime monitoring service เช่น:
# - UptimeRobot (ฟรี)
# - Pingdom
# - StatusCake
```

### 4. **Performance Issues**:
- Optimize Docker image size
- ใช้ multi-stage build (ทำแล้ว)
- Enable compression
- ใช้ CDN สำหรับ static files

## 🔒 Security Best Practices

### 1. **Environment Variables**:
- ✅ **ใช้ Render Dashboard** สำหรับ sensitive data
- ❌ **อย่าใส่ใน render.yaml** หรือ commit ลง git
- ✅ **ใช้ strong passwords และ random secrets**
- ✅ **เปิด environment encryption** ใน Render
- ✅ **จำกัดการเข้าถึง** team members

### 2. **Database**:
- เปิด SSL/TLS (`DB_SSL=require`)
- ใช้ strong database password
- Limit connections

### 3. **Application**:
- ใช้ HTTPS (Render จัดให้อัตโนมัติ)
- Validate input data
- Implement rate limiting
- Monitor logs

## 🚀 Production Optimization

### 1. **Upgrade Plans** (ถ้าต้องการ):
```
Web Service:
- Starter: $7/month (always on, custom domains)
- Standard: $25/month (more resources)

PostgreSQL:
- Starter: $7/month (25GB storage)
- Standard: $20/month (100GB storage)
```

### 2. **Custom Domain**:
1. ใน Render Dashboard → Settings → Custom Domains
2. เพิ่ม domain และ configure DNS
3. SSL certificate จะถูกสร้างอัตโนมัติ

### 3. **Monitoring**:
- ใช้ Render metrics
- Setup external monitoring
- Configure alerts

## 📱 CI/CD

Render จะ auto-deploy เมื่อ:
- Push ไปยัง main branch
- Merge pull request
- Manual deploy ใน dashboard

### Auto-deploy Configuration:
```yaml
# ใน render.yaml
services:
  - type: web
    autoDeploy: true  # default
    branch: main      # deploy branch
```

---

**🎉 หลังจาก deploy สำเร็จ**:
- API จะพร้อมใช้งานที่ URL ที่ Render จัดให้
- Database จะถูกสร้างและ migrate อัตโนมัติ
- Admin user จะถูกสร้างตาม configuration
- API Documentation พร้อมใช้ที่ `/swagger/`

สำหรับการใช้งานจริง แนะนำให้ upgrade เป็น paid plan เพื่อประสิทธิภาพที่ดีกว่าครับ! 