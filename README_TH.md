# RubberSync Web v2 — ระบบบริหารจัดการสวนยาง

เวอร์ชันนี้ปรับให้ตรงกับ Requirements หลัก 3 ระบบมากขึ้น: ระบบติดต่อสื่อสาร, ระบบจัดการรอบขายยางและแจ้งเตือน, และระบบเบิกค่าใช้จ่าย

## ฟังก์ชันหลัก
- ผู้ใช้ 4 บทบาท: เจ้าของสวน, คนดูแลสวน, คนดูแลคนงาน, คนงานกรีดยาง
- ห้องสื่อสารกลุ่มภายใน RubberSync
- ส่งข้อความจาก RubberSync ไป Discord ผ่าน Webhook
- รับข้อความจาก Discord กลับมาแสดงใน RubberSync ผ่าน Discord Bot Token + Channel ID
- เก็บแหล่งที่มาของข้อความว่า RubberSync หรือ Discord และกันข้อความ Discord ซ้ำด้วย Discord Message ID
- รอบขายยาง: วันเก็บยาง, วันขายยาง, ราคายาง/ราคาประมูล, ประวัติรอบขาย
- แจ้งเตือนกำหนดการและแจ้งเมื่อมีการเปลี่ยนวันเก็บ/วันขาย
- คนงานสร้างคำขอเบิก พร้อมแนบใบเสร็จ/หลักฐาน PNG/JPG/WebP สูงสุด 5 MB
- เจ้าของสวนอนุมัติ/ไม่อนุมัติ
- คนดูแลสวนหรือเจ้าของสวนบันทึกการจ่ายเงินและแนบสลิป
- ตรวจรายการที่อาจเบิกซ้ำจาก ผู้ขอ + ประเภท + จำนวนเงิน + วันที่
- เก็บประวัติคำขอและสถานะย้อนหลัง
- PWA ใช้บน Windows/macOS/Android/iPhone ผ่าน Browser

## วิธีเปิดบน Windows
1. ติดตั้ง Node.js 18 ขึ้นไป
2. แตก ZIP
3. เปิดโฟลเดอร์ที่มี `server.js` และ `START_WEB.bat`
4. ดับเบิลคลิก `START_WEB.bat`
5. เปิด `http://localhost:8080`

มือถือที่อยู่ Wi-Fi เดียวกับคอม ใช้ URL IP ที่ Terminal แสดง เช่น `http://192.168.1.110:8080`

## บัญชีทดสอบ
รหัสผ่านทุกบัญชี: `12345678`

- เจ้าของสวน: `0800000000`
- คนดูแลสวน: `0800000002`
- คนดูแลคนงาน: `0800000003`
- คนงานกรีดยาง: `0800000001`

ผู้สมัครสมาชิกใหม่จะได้บทบาท `คนงานกรีดยาง` โดยอัตโนมัติ

## การตั้ง Discord
### 1) ส่ง RubberSync -> Discord
กำหนด `DISCORD_WEBHOOK_URL`

PowerShell:
```powershell
$env:DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/..."
node server.js
```

### 2) รับ Discord -> RubberSync
Webhook ปกติรับข้อความจาก Discord กลับไม่ได้ จึงต้องใช้ Discord Bot ที่มีสิทธิ์ `View Channel` และ `Read Message History` แล้วกำหนด:

```powershell
$env:DISCORD_BOT_TOKEN="..."
$env:DISCORD_CHANNEL_ID="..."
node server.js
```

เมื่อเปิดหน้าสื่อสาร RubberSync จะดึงข้อความล่าสุดจาก Channel และบันทึกโดยใช้ Discord Message ID ป้องกันการซ้ำ

## การเชื่อม Supabase
1. เปิดโปรเจกต์ Supabase แล้วไปที่ **SQL Editor**
2. เปิดไฟล์ `supabase/schema.sql` ในโปรเจกต์นี้ คัดลอกไปรันใน SQL Editor เพื่อสร้างตาราง `rubbersync_app_state`
3. ตั้งค่า `SUPABASE_URL` และ `SUPABASE_SERVICE_ROLE_KEY` ในไฟล์ `.env` ที่อยู่โฟลเดอร์เดียวกับ `server.js`
   - URL ดูได้จาก **Project Settings > API**
   - ใช้ secret/service-role key สำหรับฝั่งเซิร์ฟเวอร์เท่านั้น ห้ามใส่ใน `public/` หรือส่งให้ frontend
4. ปิดแล้วเปิดเซิร์ฟเวอร์ใหม่ แอปจะย้ายข้อมูลจาก `data/db.json` ไป Supabase ให้อัตโนมัติเมื่อแถว `main` ยังไม่มี
5. เปิด `http://localhost:8080/api/health` แล้วตรวจฟิลด์ `"database": "supabase"`

แอปเก็บข้อมูลปัจจุบันเป็น JSONB หนึ่งแถวเพื่อรักษารูปแบบเดิมของระบบ และรองรับการรันเซิร์ฟเวอร์เพียงหนึ่ง instance; ข้อมูลรูปภาพใน `public/uploads/` ยังคงอยู่บนเครื่องเซิร์ฟเวอร์ ไม่ได้ย้ายไป Supabase Storage

## ข้อมูลที่บันทึก
- เมื่อไม่ได้ตั้งค่า Supabase: `data/db.json` เก็บผู้ใช้ คำขอ รอบขาย ข้อความ การแจ้งเตือน ประกาศ และรายการจ่ายเงิน
- เมื่อตั้งค่า Supabase: ตาราง `rubbersync_app_state` เก็บข้อมูลแอป และ `data/db.json` จะไม่ถูกใช้อ่าน/เขียน
- `public/uploads/` เก็บใบเสร็จและสลิป

## หมายเหตุสำหรับ Production
- ต้องเปลี่ยน `AUTH_SECRET`
- ควรใช้ HTTPS
- ควรย้ายฐานข้อมูลจาก JSON ไป PostgreSQL/MySQL เมื่อใช้งานหลายคนจริง
- Discord Bot Token ต้องเก็บเป็น environment variable เท่านั้น ห้ามใส่ใน frontend หรือ commit ขึ้น Git
