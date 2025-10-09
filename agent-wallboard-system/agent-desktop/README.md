# 🧑‍💼 Agent Desktop Wallboard

โฟลเดอร์ `agent-desktop` เป็นส่วนติดต่อผู้ใช้เดสก์ท็อปของระบบ Agent Wallboard โดยสร้างจาก React 18 และครอบด้วย Electron เพื่อให้แพ็กและรันได้ทั้ง Windows, macOS และ Linux พร้อมการเชื่อมต่อเรียลไทม์ผ่าน Socket.IO และการแจ้งเตือนแบบเดสก์ท็อป

## ✨ ไฮไลต์ของโปรเจ็กต์
- UI ใช้ React พร้อมโครงสร้าง component แยกส่วน (Login, Status, Message, ErrorBoundary)
- กระบวนการ Electron ครบทั้ง `main.js`, การสร้าง tray, single-instance lock และแจ้งเตือนผ่าน IPC
- บริการเชื่อมต่อ API, Socket และ Notification แยกใน `src/services`
- รองรับการ build เป็นไฟล์ติดตั้งผ่าน `electron-builder` สำหรับ 3 ระบบปฏิบัติการ

## ⚙️ สิ่งที่ต้องเตรียมก่อนเริ่ม
- Node.js 18 ขึ้นไป (ทดสอบล่าสุดด้วย v20.19.3)
- npm 8 ขึ้นไป
- Backend ของ Wallboard ที่เปิด REST (`/api`) และ Socket ที่พอร์ต 3001
- MongoDB instance (ใช้โดย backend)

## 📁 โครงสร้างหลักของโฟลเดอร์
```text
agent-desktop/
├── main.js                 # main process ของ Electron และระบบ tray
├── public/
│   ├── index.html          # HTML template เมื่อ build React
│   └── assets/
│       ├── icon.png        # ไอคอนแอป (ใช้ใน tray/build)
│       └── tray-icon.png   # ไอคอนใน system tray
├── src/
│   ├── App.js              # ควบคุม state หลัก การเชื่อม Socket และ Notification
│   ├── components/         # ส่วนแสดงผลย่อย เช่น LoginForm, AgentInfo, StatusPanel
│   ├── services/           # ตัวเรียกใช้งาน API, Socket.IO และ Notification
│   ├── styles/             # ไฟล์ CSS ของหน้าจอ
│   └── utils/              # ฟังก์ชัน Utility (logger, validation, date formatter)
├── test-complete.sh        # สคริปต์ตรวจสุขภาพโปรเจ็กต์แบบรวดเร็ว
├── package.json            # กำหนด dependencies และ npm scripts
└── .env.example            # ตัวอย่างค่าตัวแปรสภาพแวดล้อม
```

## 🚀 ขั้นตอนการเริ่มใช้งาน
1. ติดตั้ง dependency  
   ```bash
   npm install
   ```
2. ตั้งค่าตัวแปรสภาพแวดล้อม  
   ```bash
   cp .env.example .env
   ```
   จากนั้นแก้ไข `REACT_APP_API_URL` และ `REACT_APP_SOCKET_URL` ให้ตรงกับ backend จริง
3. เปิด backend (REST และ Socket) รวมถึง MongoDB ให้พร้อมก่อนเปิดแอป

### 🔄 โหมดพัฒนา React (Web)
```bash
npm start
```
- เปิดเบราว์เซอร์ที่ `http://localhost:3000` เพื่อดู UI
- โหมดนี้ใช้ Hot Reload และเหมาะสำหรับทดสอบ component อย่างรวดเร็ว

### 🖥️ โหมดพัฒนา Electron + React
```bash
npm run electron-dev
```
- คำสั่งนี้เรียก `npm start` และเปิด Electron โดยอัตโนมัติผ่าน `wait-on`
- ใช้สำหรับดูพฤติกรรม tray, IPC, และการแจ้งเตือนจริงบนเดสก์ท็อป

### 📦 สร้างแอปพลิเคชันแพ็กเกจ
```bash
npm run build         # สร้าง React production bundle
npm run electron      # รัน build กับ Electron โดยไม่เปิด devtools
npm run dist          # สร้างไฟล์ติดตั้งด้วย electron-builder
```
> มีสคริปต์ย่อย `dist:win`, `dist:mac`, `dist:linux` เพื่อสร้างแพ็กเกจเฉพาะแพลตฟอร์ม

## 🌐 การตั้งค่า .env ที่สำคัญ
| ตัวแปร | ค่าเริ่มต้น | ความหมาย |
| --- | --- | --- |
| `REACT_APP_API_URL` | `http://localhost:3001/api` | จุดเชื่อมต่อ REST ของ backend |
| `REACT_APP_SOCKET_URL` | `http://localhost:3001` | ฐาน URL สำหรับ Socket.IO |
| `NODE_ENV` | `development` | โหมดที่ใช้รัน React |
| `ELECTRON_IS_DEV` | `true` | บังคับเปิด devtools และชี้ไปยัง `localhost:3000` |

> เมื่อ build production ให้แก้ `ELECTRON_IS_DEV=false` และรัน `npm run build` ก่อน `npm run electron`

## 📡 การเชื่อมต่อและฟีเจอร์หลัก
- `src/services/api.js` จัดการ login/logout, ดึงข้อความและอัปเดตสถานะผ่าน REST พร้อมแนบ token อัตโนมัติ
- `src/services/socket.js` ดูแลการเชื่อมต่อ Socket.IO, รับเหตุการณ์สถานะ/ข้อความแบบเรียลไทม์ และส่ง status update
- `src/services/notifications.js` รองรับการแจ้งเตือนผ่าน `window.electronAPI` (ต้องมี `preload.js`) และ fallback ไปยัง Web Notification
- Component หลัก:
  - `LoginForm` ตรวจสุขภาพ backend ก่อนอนุญาตให้ login พร้อม validate agent code
  - `AgentInfo`, `StatusPanel`, `MessagePanel` แสดงข้อมูลสถานะและข้อความแบบเรียลไทม์
  - `ErrorBoundary` ป้องกัน UI crash และแจ้งเตือนผู้ใช้

## 🧰 สคริปต์ที่ใช้บ่อย
- `npm test` รันชุดทดสอบของ react-scripts
- `npm run electron` เปิด Electron โดยโหลดไฟล์จากโฟลเดอร์ `build`
- `npm run build` สร้าง production bundle สำหรับ React
- `npm run dist:*` สร้างไฟล์ติดตั้งแบบ platform-specific ผ่าน electron-builder
- `./test-complete.sh` ตรวจสุขภาพเบื้องต้น (Node, .env, ไอคอน, Backend, MongoDB)

