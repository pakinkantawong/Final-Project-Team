# Agent Wallboard Backend 🚦

แบ็กเอนด์สำหรับระบบ Agent Wallboard ที่ให้บริการทั้ง REST API และ WebSocket เพื่ออัปเดตสถานะเอเจนต์แบบเรียลไทม์ เชื่อมต่อ SQLite เพื่อเก็บข้อมูลเอเจนต์พื้นฐาน และ MongoDB สำหรับสถานะและข้อความล่าสุด ใช้งานได้ทั้งสำหรับทีม Supervisor และ Agent พร้อมระบบจำกัดอัตราเรียกใช้งาน (rate limit) ในตัว

## คุณสมบัติเด่น ✨
- 🔐 รับรองความปลอดภัยด้วย JWT Authentication + middleware ตรวจสอบสิทธิ์
- 🔁 WebSocket (Socket.IO) แจ้งเตือนสถานะและข้อความใหม่แบบทันที พร้อม heartbeat ตรวจจับการหลุดเชื่อมต่อ
- 🧮 ข้อมูลเชิงสถิติจาก SQLite (รายชื่อเอเจนต์/ทีม) + MongoDB (สถานะ, ข้อความ) ในระบบเดียว
- 🛡️ Rate Limiting สำหรับทุก API และลิมิตพิเศษสำหรับเส้นทาง Auth ป้องกัน brute-force
- 🩺 Endpoint `/health` รายงานสถานะเซิร์ฟเวอร์และหน่วยความจำแบบเรียลไทม์
- 🧰 ตัวช่วยแปลงข้อมูล (`utils/transformers.js`) สำหรับคืนข้อมูลในรูปแบบที่ UI พร้อมใช้งาน

## สถาปัตยกรรมโดยสรุป 🧩
```
Client (Agent / Supervisor UI)
    │
    ├─ REST API (Express) → Authentication, Agents, Messages
    │
    ├─ Socket.IO Gateway ↔ Broadcast สถานะ/ข้อความเรียลไทม์
    │
    ├─ SQLite (agents, teams)   – เก็บข้อมูลอ้างอิงแบบอ่านบ่อย
    └─ MongoDB (statuses, messages) – เก็บเหตุการณ์/ข้อความล่าสุด
```

## โครงสร้างโฟลเดอร์หลัก 📁
```
backend-server/
├─ config/            # ตั้งค่าฐานข้อมูลและ helper path
├─ middleware/        # JWT auth และ global error handler
├─ models/            # Agent (SQLite), Status/Message (MongoDB)
├─ routes/            # auth, agents, messages REST endpoints
├─ socket/            # socketHandler.js ดูแล WebSocket events
├─ utils/             # ฟังก์ชันแปลงข้อมูลให้ UI
├─ server.js          # จุดเริ่มของแอป + บูต WebSocket
└─ package.json       # สคริปต์และ dependencies
```

## ข้อกำหนดก่อนใช้งาน 🧱
- Node.js ≥ 18 และ npm
- SQLite3 CLI หรือไฟล์ฐานข้อมูล `wallboard.db` ที่สร้างล่วงหน้า
- MongoDB (โลคัลหรือรีโมต) พร้อมสิทธิ์เชื่อมต่อ
- ไฟล์ `.env` กำหนดค่า CORS, JWT และพาธฐานข้อมูล

## ตั้งค่าและเริ่มต้น 🛠️
1. ติดตั้ง dependencies  
   ```bash
   npm install
   ```
2. เตรียมไฟล์ SQLite  
   - โครงสร้างค่าเริ่มต้นคาดหวังเส้นทาง `./database/sqlite/wallboard.db` (สามารถแก้ด้วยตัวแปร `SQLITE_DB_PATH`)  
   - หากยังไม่มีไฟล์ ให้สร้างไดเรกทอรีและสคีมาด้วยตนเอง เช่น
     ```bash
     mkdir -p database/sqlite
     sqlite3 database/sqlite/wallboard.db < schema.sql
     ```
3. ตั้งค่า MongoDB ให้พร้อมรับการเชื่อมต่อ และสร้างฐานข้อมูล `wallboard` หรือชื่ออื่นที่คุณต้องการ
4. สร้างไฟล์ `.env` (ตัวอย่างด้านล่าง) แล้วสตาร์ตเซิร์ฟเวอร์
   ```bash
   npm run dev   # ใช้ nodemon
   # หรือ
   npm start     # รันปกติ
   ```

### ตัวอย่างไฟล์ `.env` 🌱
```ini
PORT=3001
JWT_SECRET=super-secret-change-me
CORS_ORIGIN=http://localhost:5173,http://localhost:3000
SQLITE_DB_PATH=./database/sqlite/wallboard.db
MONGODB_URI=mongodb://127.0.0.1:27017/wallboard
```

## วิธีการใช้งาน 🚀
1. **สตาร์ตเซิร์ฟเวอร์**  
   ```bash
   npm run dev   # แนะนำในระหว่างพัฒนา
   # หรือ
   npm start     # สำหรับสภาพแวดล้อม production-like
   ```
   หลังบูตสำเร็จจะเห็น log สรุป REST routes และ WebSocket events ใน terminal

2. **เข้าสู่ระบบเพื่อขอ JWT** (ใช้ Agent หรือ Supervisor code ที่มีใน SQLite)  
   ```bash
   curl -X POST http://localhost:3001/api/auth/login \
     -H "Content-Type: application/json" \
     -d '{"agentCode":"AGENT001"}'
   ```
   - คำตอบที่ได้จะมี `data.token` นำไปใช้เรียก API อื่นได้
   - หากเป็น Supervisor ระบบจะส่ง `teamData` (หลัง transformer) กลับมาพร้อมกัน

3. **เรียก REST API ด้วย JWT**  
   ```bash
   TOKEN=<token จากขั้นตอนก่อน>
   curl -X GET http://localhost:3001/api/agents/team/1 \
     -H "Authorization: Bearer $TOKEN"
   ```
   - เส้นทาง `/api/agents/team/:teamId` จะคืนข้อมูลสถานะล่าสุดจาก MongoDB รวมกับข้อมูลพื้นฐาน SQLite
   - ใช้ `PUT /api/messages/:messageId/read` หรือ `PUT /api/agents/:agentCode/status` เพื่ออัปเดตข้อมูล

4. **ใช้งาน WebSocket (Socket.IO)**  
   ตัวอย่างสคริปต์ Node.js (ต้องติดตั้ง `socket.io-client`)
   ```bash
   npm install socket.io-client
   ```
   ```js
   // client.js
   const { io } = require('socket.io-client');
   const socket = io('http://localhost:3001', { transports: ['websocket'] });

   socket.on('connect', () => {
     socket.emit('agent_connect', { agentCode: 'AGENT001' });
   });

   socket.on('connection_success', payload => console.log('connected', payload));
   socket.on('agent_status_update', payload => console.log('status update', payload));
   socket.on('new_message', payload => console.log('message', payload));

   // ส่งสถานะใหม่
   setTimeout(() => socket.emit('update_status', { agentCode: 'AGENT001', status: 'Busy' }), 2000);
   ```
   - Supervisor สามารถเปลี่ยน event เป็น `supervisor_connect` และรับข้อมูล agent ออนไลน์ทันที
   - เมื่อปิดโปรแกรมหรือหลุดการเชื่อมต่อ เซิร์ฟเวอร์จะ broadcast `agent_disconnected` ให้ทุก Supervisor

5. **บันทึกผลการทดลอง**  
   - ใช้เทมเพลต “🧪 เทมเพลตสรุปผลการทดลอง” ที่ท้าย README เพื่อรวบรวมผลลัพธ์
   - สามารถแนบหมายเลข commit, เวอร์ชันฐานข้อมูล และผลการทดสอบในแต่ละครั้งเพื่ออ้างอิงภายหลัง

## สคริปต์ npm 🔧
- `npm start` รันเซิร์ฟเวอร์แบบปกติด้วย Node.js
- `npm run dev` รันผ่าน nodemon เพื่อรีโหลดอัตโนมัติเมื่อไฟล์เปลี่ยน

## REST API หลัก 🌐
| Method | Path                            | คำอธิบาย |
|--------|---------------------------------|-----------|
| POST   | `/api/auth/login`               | สร้าง JWT token สำหรับ Agent/Supervisor และดึงข้อมูลทีมที่ดูแล |
| POST   | `/api/auth/logout`              | ยืนยันการออกจากระบบ |
| GET    | `/api/agents/team/:teamId`      | ดึงรายชื่อเอเจนต์ในทีมพร้อมสถานะล่าสุด (ต้องมี JWT) |
| PUT    | `/api/agents/:agentCode/status` | อัปเดตสถานะเอเจนต์และบันทึกลง MongoDB |
| GET    | `/api/agents/:agentCode/history`| ประวัติการอัปเดตสถานะ (query `limit` ได้) |
| POST   | `/api/messages/send`            | ส่งข้อความแบบ direct หรือ broadcast (รองรับ priority) |
| GET    | `/api/messages/agent/:agentCode`| ดึงกล่องข้อความ (รองรับ `limit`, `unreadOnly`) |
| PUT    | `/api/messages/:messageId/read` | ทำเครื่องหมายว่าอ่านแล้ว |
| GET    | `/health`                       | ตรวจสอบสถานะและ uptime ของบริการ |

> ทุกเส้นทางภายใต้ `/api/*` ยกเว้น `/api/auth/login` และ `/api/auth/logout` ต้องส่ง Header `Authorization: Bearer <token>`

## WebSocket Events ⚡
**Client → Server**
- `agent_connect` `{ agentCode }`
- `supervisor_connect` `{ supervisorCode }`
- `update_status` `{ agentCode, status }`
- `send_message` `{ fromCode, toCode?, toTeamId?, content, type }`

**Server → Client**
- `connection_success` – ระบุสถานะเชื่อมต่อและข้อมูลเพิ่มเติม (เช่น online agents)
- `connection_error` – เมื่อข้อมูลเชื่อมต่อไม่ครบถ้วน
- `agent_connected` / `agent_disconnected` – broadcast แจ้งเตือนสถานะออนไลน์
- `agent_status_update` – ส่งเมื่อมีการอัปเดตสถานะผ่าน Socket หรือ REST
- `new_message` – ส่งต่อข้อความใหม่ (direct/broadcast)
- `status_updated`, `status_error`, `message_sent`, `message_error` – แจ้งผลคำสั่ง

> ระบบมี heartbeat ทุก 30 วินาทีเพื่อตรวจสอบ client ที่หลุดการเชื่อมต่อและประกาศ `agent_disconnected` อัตโนมัติ

## การจัดการฐานข้อมูล 🗄️
- **SQLite (`models/Agent.js`)**: อ่านข้อมูลเอเจนต์และทีมผ่านฟังก์ชัน `findByCode`, `findByTeam`, `findAll`
- **MongoDB (`models/Status.js`, `models/Message.js`)**: เก็บสถานะ (พร้อม index ตาม agent/team) และข้อความพร้อมสถานะ `isRead`
- ฟังก์ชัน `initSQLite` สร้างไดเรกทอรีฐานข้อมูลอัตโนมัติและตรวจสอบ permission; หากยังไม่มีสคีมาให้รันสคริปต์สร้างก่อนสตาร์ต
- `connectMongoDB` มี retry สูงสุด 5 ครั้งพร้อม exponential backoff

## จุดตรวจสอบและสังเกตการณ์ 📡
- `console.log` ใน `server.js` แสดงเส้นทาง REST & Event สำคัญหลังสตาร์ต
- Endpoint `GET /health` รายงาน `uptime`, `memory` และ timestamp
- สามารถเพิ่ม layer monitoring เพิ่มเติมได้ที่ middleware `middleware/errorHandler.js`

## 🧪 เทมเพลตสรุปผลการทดลอง (พร้อมอิโมจิ)
ใช้ส่วนนี้เพื่อบันทึกผลทดสอบ/POC โดยยึดรูปแบบเดียวกันในแต่ละสปรินต์



## แนวทางการทดสอบที่แนะนำ 🧪
- ทดสอบ REST API ด้วย Postman หรือ Thunder Client พร้อม token จริง
- ใช้ `socket.io-client` หรือหน้าเว็บจำลองจับ Event `agent_status_update`
- เพิ่ม unit tests สำหรับ utility (เช่น `transformAgents`) หากเริ่มเขียน test suite
- ตรวจสอบ rate limit ด้วยการยิงคำขอจำนวนมากใน 15 นาที (เครื่องมือเช่น k6, autocannon)

## เคล็ดลับการดีพลอย 🚀
- ใช้ `pm2` หรือ `forever` สำหรับ production process manager
- กำหนดค่า `CORS_ORIGIN` ให้ตรงกับ origin ทั้งหมดที่ UI ใช้งานจริง (คั่นด้วย comma)
- สำรองไฟล์ SQLite และตั้งค่า replica set ให้ MongoDB หากต้องการความพร้อมใช้งานสูง

พร้อมใช้งานแล้ว! หากต้องการปรับแต่งเพิ่มสามารถเริ่มต้นที่ไฟล์ `server.js` หรือเพิ่มโมดูลใน `routes/` และ `socket/` ⭐
