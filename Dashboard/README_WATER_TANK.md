# คู่มือการใช้งาน Dashboard แสดงข้อมูลถังน้ำ 💧

## 📋 ภาพรวมระบบ

Dashboard นี้ใช้สำหรับแสดงข้อมูลถังน้ำแบบเรียลไทม์จาก PLC ผ่าน Node-RED โดยมีฟีเจอร์หลักดังนี้:

### ข้อมูลที่แสดงผล:
1. **ระดับน้ำ (Water Level)** - อ่านจาก PLC Address `D212` (max 8700 L)
2. **อัตราการไหล (Flow Rate)** - อ่านจาก PLC Address `D501` (L/min)
3. **อุณหภูมิ (Temperature)** - อ่านจาก PLC Address `D502` (°C)

---

## 🚀 วิธีการใช้งาน

### 1. เปิด Node-RED และ Import Flow

1. เปิด Node-RED ที่ `http://192.168.7.86:1880`
2. Import ไฟล์ **`node_red_flow.json`** เข้าไปใน Node-RED
3. กด **Deploy** เพื่อเปิดใช้งาน Flow

### 2. ตรวจสอบ API Endpoint

Node-RED จะสร้าง API endpoint อัตโนมัติ:
```
GET http://192.168.7.86:1880/machines
```

**Response Format:**
```json
{
  "machines": [
    { "id": 1, "presentCycle": "120.5", ... },
    { "id": 2, "presentCycle": "145.2", ... }
  ],
  "waterTank": {
    "level": 6500,       // ระดับน้ำ (L)
    "flow": 45.2,        // อัตราการไหล (L/min)
    "temp": 28.5,        // อุณหภูมิ (°C)
    "timestamp": "2026-01-12 12:54:32"
  }
}
```

### 3. เปิด Dashboard

1. เปิดไฟล์ **`index.html`** ด้วย Browser
2. เลือกโหมดการทำงาน:
   - **🛠 Simulation** - ใช้ข้อมูลจำลอง (ทดสอบ UI)
   - **🌐 Node-RED (HTTP)** - รับข้อมูลจริงจาก Node-RED

### 4. เข้าถึงแท็บแสดงข้อมูลถังน้ำ

คลิกที่แท็บ **"Water Tank"** เพื่อดูข้อมูลถังน้ำแบบละเอียด:

- แท็งค์แสดงผลแบบ 3D พร้อม animation บับเบิล
- ระดับน้ำแสดงทั้งแบบเปอร์เซ็นต์และลิตร
- ข้อมูล Flow Rate และ Temperature อัปเดตทุก 2 วินาที

---

## ⚙️ การตั้งค่า (Settings)

### เปลี่ยน Node-RED URL

1. คลิกปุ่ม **⚙ SETTINGS** บนหน้า Dashboard
2. ปรับ URL ถ้าไม่ใช่ `http://192.168.7.86:1880/machines`
3. คลิก **Close** เพื่อบันทึก

### ปรับค่าความจุถังน้ำ

ใน Water Tank Tab มี Slider สำหรับตั้งค่า:
- **Max Capacity** - ความจุสูงสุดของถัง (500-10000 L)
- **Set Water Level** - ปรับระดับน้ำแบบ Manual (0-100%)

---

## 🔄 โครงสร้างข้อมูล Node-RED Flow

### Flow ที่สำคัญ:

```
┌─────────────┐       ┌──────────────┐       ┌─────────────────┐
│ Inject Node │  ───→ │ PLC Read D212│  ───→ │ Function (D212) │
│ (Every 1s)  │       │ (Tank Level) │       │ → tankLevel     │
└─────────────┘       └──────────────┘       └─────────────────┘
                                                       │
                                                       ↓
                                              [UI Gauge + HTTP]
```

### Global Variable Structure:

Node-RED เก็บข้อมูลใน `global.waterTankData`:
```javascript
{
  level: 6500,              // จาก D212
  flow: 45.2,              // จาก D501
  temp: 28.5,              // จาก D502
  timestamp: "2026-01-12..." // เวลาล่าสุดที่อัปเดต
}
```

---

## 🐛 แก้ไขปัญหา

### ปัญหา: Dashboard ไม่แสดงข้อมูล

✅ **วิธีแก้:**
1. ตรวจสอบ Node-RED ทำงานอยู่หรือไม่ (เปิด `http://192.168.7.86:1880`)
2. กด **Deploy** ใน Node-RED อีกครั้ง
3. ตรวจสอบ URL ใน Settings ของ Dashboard ว่าถูกต้องหรือไม่
4. เปิด Browser Console (F12) ดู error message

### ปัญหา: ค่าน้ำไม่เปลี่ยนแปลง

✅ **วิธีแก้:**
1. ตรวจสอบ PLC เชื่อมต่ออยู่หรือไม่
2. ใน Node-RED ตรวจสอบว่า Connection Node (Fire tank) online หรือไม่
3. ลองกด Inject Node ใน Node-RED เพื่อบังคับอ่านค่าใหม่

### ปัญหา: CORS Error

✅ **วิธีแก้:**
- ใช้ Browser Extension เช่น "Allow CORS" หรือ
- ใช้ Local Server: `python -m http.server 8000` แล้วเปิด `http://localhost:8000/index.html`

---

## 📊 ข้อมูลเทคนิค

### Technology Stack:
- **Frontend**: React (CDN), HTML5, CSS3
- **Backend**: Node-RED (HTTP API)
- **PLC Communication**: Mitsubishi MC Protocol (TCP)
- **Data Format**: JSON

### Update Interval:
- Node-RED → PLC: ทุก 1 วินาที (Inject Node)
- Dashboard → Node-RED: ทุก 2 วินาที (HTTP Polling)

### PLC Address Mapping:
| Address | ข้อมูล | หน่วย | Range |
|---------|--------|-------|-------|
| D212 | ระดับน้ำ | Liters | 0-8700 |
| D501 | อัตราการไหล | L/min | 0-100 |
| D502 | อุณหภูมิ | °C | 0-100 |

---

## 📝 หมายเหตุ

- ข้อมูลถังน้ำจะถูกส่งไป Google Sheets อัตโนมัติ (ดูที่ Function node "function 198")
- Simulation Mode ใช้สำหรับทดสอบ UI โดยไม่ต้องเชื่อมต่อ PLC
- Tab "Water Tank" แสดงข้อมูลถังน้ำแบบเต็มหน้าจอ พร้อมควบคุม Manual

---

## 🎯 คำแนะนำการใช้งาน

1. **ใช้ Simulation ก่อน** เพื่อทดสอบว่า UI แสดงผลถูกต้อง
2. **เปลี่ยนเป็น HTTP Mode** เมื่อพร้อมเชื่อมต่อ Node-RED
3. **ตรวจสอบ Status Badge** ที่มุมบนซ้ายว่าแสดง "Connected (HTTP)" หรือไม่
4. **ใช้ Tab Water Tank** สำหรับดูข้อมูลถังน้ำแบบละเอียด

---

**สร้างโดย:** Antigravity AI Assistant  
**วันที่อัปเดต:** 2026-01-12
