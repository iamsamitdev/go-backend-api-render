# Environment Variables สำหรับ Render Deployment

## 🔒 Security First Approach

ไฟล์ `render.yaml` **ไม่มี** environment variables เพื่อความปลอดภัย  
ทุก sensitive data ต้องตั้งค่าใน **Render Dashboard** เท่านั้น

## 📋 Environment Variables ที่ต้องตั้งค่า

### 🔧 ใน Render Dashboard

**1. ไปที่**: Web Service → Environment Variables  
**2. เพิ่มตัวแปรต่อไปนี้**:

```bash
# Application Settings
APP_ENV=production
JWT_EXPIRES_IN=24h
AUTO_MIGRATE=true

# Database Settings  
DB_SSL=require

# Admin User Settings
ADMIN_EMAIL=admin@yourcompany.com
ADMIN_FIRST_NAME=Admin
ADMIN_LAST_NAME=User

# Sensitive Data (สร้างใหม่ทุกครั้ง)
JWT_SECRET=[ใส่ค่าที่สร้างจากคำสั่งด้านล่าง]
ADMIN_PASSWORD=[ใส่ค่าที่สร้างจากคำสั่งด้านล่าง]
```

## 🛠️ วิธีสร้าง Strong Secrets

### สำหรับ JWT_SECRET:
```bash
# สร้าง 32-character hex string
openssl rand -hex 32

# ตัวอย่างผลลัพธ์:
# 4f8a2e6b9c1d3f7a8e2b5c9f0d4a7e1b3c6f9a2e5b8d1c4f7a0e3b6c9f2d5a8e1b
```

### สำหรับ ADMIN_PASSWORD:
```bash
# สร้าง 16-character strong password
openssl rand -base64 16

# ตัวอย่างผลลัพธ์:
# 8K2mP9xN7qR5vT3wL6jE
```

### 🌐 Online Generators (ถ้าไม่มี OpenSSL):
- **UUID Generator**: https://www.uuidgenerator.net/
- **Strong Password**: https://passwordsgenerator.net/
- **Random Hex**: https://www.random.org/strings/

## 🔗 Database Connection

### Automatic Linking (แนะนำ):
1. สร้าง PostgreSQL database ใน Render ก่อน
2. ใน Web Service → Environment Variables
3. คลิก "Link to Database"  
4. เลือก PostgreSQL service ที่สร้างไว้
5. Render จะตั้งค่า `DATABASE_URL` อัตโนมัติ

### Manual Setup (ถ้าต้องการ):
```bash
DB_HOST=[Database Internal URL จาก Render]
DB_PORT=5432
DB_USER=[Database User]
DB_PASS=[Database Password]  
DB_NAME=[Database Name]
```

## ✅ Checklist ก่อน Deploy

- [ ] สร้าง PostgreSQL database ใน Render
- [ ] สร้าง JWT_SECRET ใหม่ (32 characters)
- [ ] สร้าง ADMIN_PASSWORD ใหม่ (strong password)
- [ ] ตั้ง ADMIN_EMAIL เป็นอีเมลจริง
- [ ] เชื่อมโยง database กับ web service
- [ ] ตรวจสอบว่า DB_SSL=require
- [ ] ตรวจสอบว่า AUTO_MIGRATE=true (สำหรับ deploy ครั้งแรก)

## 🚨 Security Warnings

### ❌ อย่าทำ:
- อย่าใส่ sensitive data ใน `render.yaml`
- อย่า commit `.env` files ที่มีข้อมูลจริง
- อย่าใช้ default passwords
- อย่าแชร์ secrets ผ่าน chat/email

### ✅ ควรทำ:
- ใช้ Render Dashboard สำหรับ secrets
- สร้าง strong passwords ทุกครั้ง
- เปลี่ยน ADMIN_PASSWORD หลัง deploy แรก
- ใช้ email จริงสำหรับ ADMIN_EMAIL
- Enable 2FA สำหรับ Render account

## 🔄 หลัง Deploy

### 1. เปลี่ยน Admin Password:
```bash
# เข้าไปที่ API และเปลี่ยนรหัสผ่าน admin
POST /api/v1/auth/change-password
```

### 2. ทดสอบ API:
```bash
# Health check
curl https://your-app.onrender.com/health

# Login test
curl -X POST https://your-app.onrender.com/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@yourcompany.com","password":"YOUR_ADMIN_PASSWORD"}'
```

### 3. Monitor Logs:
- ดู logs ใน Render Dashboard
- ตรวจสอบ database connections
- ดู API response times

---

## 📝 Environment Variables Reference

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `APP_ENV` | Yes | Application environment | `production` |
| `JWT_SECRET` | Yes | JWT signing secret | `4f8a2e6b9c1d...` |
| `JWT_EXPIRES_IN` | No | JWT expiration time | `24h` |
| `AUTO_MIGRATE` | No | Run DB migrations | `true` |
| `DB_SSL` | Yes | Database SSL mode | `require` |
| `ADMIN_EMAIL` | Yes | Admin user email | `admin@company.com` |
| `ADMIN_PASSWORD` | Yes | Admin user password | `8K2mP9xN7qR5...` |
| `ADMIN_FIRST_NAME` | No | Admin first name | `Admin` |
| `ADMIN_LAST_NAME` | No | Admin last name | `User` |

**หมายเหตุ**: Database connection variables จะถูกตั้งอัตโนมัติเมื่อ link database 