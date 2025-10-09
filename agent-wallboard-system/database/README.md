# 🧭 Agent Wallboard Database Toolkit

## 📋 ภาพรวม
โปรเจ็กต์ย่อยนี้รวบรวมสคริปต์และสคีมาฐานข้อมูลสำหรับระบบ Agent Wallboard โดยเตรียมไว้ทั้ง SQLite และ MongoDB เพื่อช่วยทีมพัฒนาและทดสอบฟีเจอร์สรุปสถานะเอเจนต์ได้รวดเร็ว ผู้ใช้สามารถเลือกใช้งานฐานข้อมูลตามสภาพแวดล้อมของตนเอง พร้อมเครื่องมือสำรองและรีเซ็ตข้อมูลในโฟลเดอร์เดียวกัน

## 📁 โครงสร้างไดเรกทอรี
- `sqlite/` – ไฟล์สคีมา (`init.sql`), ข้อมูลตัวอย่าง (`sample_data.sql`), และสคริปต์ `setup.sh` สำหรับสร้าง `wallboard.db`
- `mongodb/` – สคริปต์ `sample_data.js` สำหรับป้อนข้อมูลตัวอย่างผ่าน Mongoose และ `setup.sh` ที่ตรวจสอบการรัน MongoDB ให้พร้อมก่อนเรียกใช้
- `backup.sh` – สำรอง `sqlite/wallboard.db` และทำ `mongodump` ไปยังโฟลเดอร์ `backups/<timestamp>`
- `reset_all.sh` – รีเซ็ตฐานข้อมูลทั้งสองแบบ (มีการยืนยันก่อนลบข้อมูล)
- `.env` – เก็บค่าคอนฟิกเพิ่มเติมเมื่อต้องการ override ค่าเริ่มต้น (ไม่บังคับ)

## 🗂️ การใช้งานฐานข้อมูล SQLite
1. ตรวจสอบว่ามี `sqlite3` พร้อมใช้งาน แล้วรันสคริปต์ตั้งต้น:
   ```bash
   cd database/sqlite
   ./setup.sh
   ```
2. สคริปต์จะล้างไฟล์ `wallboard.db` (ถ้ามี), สร้างสคีมา, ใส่ข้อมูลตัวอย่าง และสรุปจำนวนระเบียน
3. ทดสอบสถานะปัจจุบันของฐานข้อมูลด้วยคำสั่งด้านล่าง (ผลลัพธ์ที่วัดได้จริงอยู่ในเซกชันสรุปผล):
   ```bash
   cd database/sqlite
   sqlite3 wallboard.db <<'SQL'
   SELECT 'teams' AS table_name, COUNT(*) AS total FROM teams
   UNION ALL
   SELECT 'agents', COUNT(*) FROM agents
   UNION ALL
   SELECT 'system_config', COUNT(*) FROM system_config;
   SQL
   ```

## 🍃 การใช้งานฐานข้อมูล MongoDB
1. เปิดบริการ MongoDB แล้วตรวจสอบด้วย `mongosh --eval "db.adminCommand('ping')"`
2. ตั้งค่า `MONGODB_URI` หากต้องการเชื่อมต่อโฮสต์อื่น (ค่าเริ่มต้นคือ `mongodb://localhost:27017/wallboard`)
3. รันสคริปต์ตั้งต้นเพื่อป้อนข้อมูลตัวอย่าง:
   ```bash
   cd database/mongodb
   ./setup.sh
   # หรือหากติดตั้ง dependency ไว้แล้ว สามารถเรียกเพียง
   node sample_data.js
   ```
4. สคริปต์จะติดตั้ง `mongoose` (หากยังไม่มี), ล้างข้อมูลคอลเลกชันเดิม, ใส่ข้อมูลตัวอย่าง และสรุปจำนวนเอกสารในแต่ละคอลเลกชัน

## 🧪 สรุปผลการทดลอง
| # | การทดลอง | รายละเอียด | ผลลัพธ์ |
|---|-----------|-------------|---------|
| 1 | ตรวจสอบข้อมูล SQLite | รัน `sqlite3 wallboard.db` เพื่อนับข้อมูลใน `teams`, `agents`, `system_config` | ได้ค่า `teams=3`, `agents=13`, `system_config=8` |
| 2 | วิเคราะห์สคริปต์ MongoDB | ทบทวน `mongodb/sample_data.js` เพื่อสรุปจำนวนเอกสารที่จะถูก insert | สคริปต์เตรียม `messages=6`, `agent_status=9`, `connection_logs=8` พร้อมข้อความยืนยันใน console |

หมายเหตุ: หากยังไม่ได้เปิด MongoDB จึงยังไม่มีการรันทดลองจริงในหัวข้อที่ 2 โปรดรัน `mongodb/setup.sh` ภายหลังเพื่อรับค่าจริงและตรวจสอบด้วย `mongosh wallboard --eval 'db.messages.countDocuments()'`

## 💾 สคริปต์เสริมที่ควรรู้
- `./backup.sh` – สร้างโฟลเดอร์ `backups/` ตามเวลาปัจจุบัน รวมทั้งสำเนา SQLite และผล `mongodump`
- `./reset_all.sh` – ตั้งค่าใหม่ทั้งหมด (ต้องตอบ `yes` ก่อนทำงาน) และให้คำสั่งตรวจสอบผลลัพธ์ภายหลัง

## ✅ เช็กลิสต์หลังการใช้งาน
- ✨ ใช้ SQLite เมื่ออยากได้ฐานข้อมูลเร็ว ใช้งานสคริปต์เดียวจบ
- 🍃 ใช้ MongoDB เมื่อระบบหลักของทีมทำงานบน NoSQL หรือมีบริการ MongoDB อยู่แล้ว
- 💡 หลังรันสคริปต์ ให้บันทึกผลลัพธ์จาก console ไว้อ้างอิงร่วมกับสรุปด้านบน

ขอให้สนุกกับการทดลองและปรับแต่งระบบ Agent Wallboard! 😄
