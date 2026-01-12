# 📊 ระบบ Dashboard ถังน้ำ - สรุปการใช้งาน

## ✨ ภาพรวม

ระบบนี้แสดงข้อมูลถังน้ำแบบเรียลไทม์ โดยสามารถเลือกแหล่งข้อมูลได้ 3 แบบ:

### 🎯 Data Sources:
1. **🛠 Simulation** - ข้อมูลจำลอง (ทดสอบ UI)
2. **🌐 Node-RED** - ดึงจาก PLC ผ่าน Node-RED
3. **📊 Google Sheets** - ดึงจาก Google Sheets ที่เก็บ historical data

---

## 🚀 วิธีการเริ่มต้นใช้งาน

### ขั้นตอนที่ 1: เลือก Data Source

เปิด `index.html` ในบราว์เซอร์ แล้วเลือกโหมดจาก dropdown ด้านบนขวา:

```
┌─────────────────────────────────┐
│  📊 Google Sheets               │  ← เลือกตรงนี้
│  🌐 Node-RED (HTTP)             │
│  🛠 Simulation                  │
└─────────────────────────────────┘
```

### ขั้นตอนที่ 2: ตั้งค่า URL

1. คลิก **⚙ SETTINGS**
2. เลือก Mode ที่ต้องการ
3. กรอก URL ให้ตรง:

| Mode | URL Example |
|------|-------------|
| **Google Sheets** | `https://script.google.com/macros/s/AKfycby.../exec` |
| **Node-RED** | `http://192.168.7.86:1880/machines` |
| **Simulation** | ไม่ต้องตั้งค่า |

4. คลิก **Close**

### ขั้นตอนที่ 3: ดูข้อมูล

คลิกที่แท็บ **"Water Tank"** เพื่อดูข้อมูลถังน้ำแบบละเอียด

---

## 📋 การเตรียมข้อมูล Google Sheets

### 1. สร้าง Header ใน Sheet ชื่อ "Tank"

| A | B | C | D | E |
|---|---|---|---|---|
| **Timestamp** | **Level** | **Status** | **Flow** | **Temp** |
| 12/1/2026 16:52:26 | 7378 | NORMAL | 45.2 | 28.5 |

### 2. Deploy Google Apps Script

ดูคู่มือครบถ้วนที่: **`GOOGLE_SHEETS_SETUP.md`**

สรุปสั้นๆ:
1. เปิด Google Sheets → Extensions → Apps Script
2. Copy โค้ดจาก `google_apps_script_api.gs`
3. Deploy → New deployment → Web app
4. Who has access: **Anyone**
5. คัดลอก URL ที่ได้

### 3. ทดสอบ API

เปิด URL ใน Browser ควรเห็น:
```json
{
  "waterTank": {
    "timestamp": "2026-01-12T09:52:26.000Z",
    "level": 7378,
    "status": "NORMAL",
    "flow": 45.2,
    "temp": 28.5
  },
  "machines": []
}
```

---

## 🎨 ฟีเจอร์ที่มี

### Water Tank Display
- 🌊 **3D Water Tank** - แสดงระดับน้ำแบบ Animated
- 💧 **Bubbles Effect** - ฟองน้ำลอยขึ้นแบบเรียลไทม์
- 📊 **Real-time Stats**:
  - Total Volume (Liters)
  - Flow Rate (L/min)
  - Temperature (°C)
  
### Manual Controls
- 🎚 **Set Water Level** - ปรับระดับน้ำด้วยตัวเอง (Simulation)
- ⚙️ **Max Capacity** - ตั้งค่าความจุสูงสุด (500-10000 L)

### Status Monitoring
- 🟢 **Connected** - เชื่อมต่อสำเร็จ
- 🔴 **Error** - มีปัญหา
- 🟡 **Connecting...** - กำลังเชื่อมต่อ

---

## 🔄 การทำงานของระบบ

### Simulation Mode
```
Dashboard → Generate Random Data → Display
(อัปเดตทุก 2 วินาที)
```

### Node-RED Mode
```
PLC (D212, D501, D502) → Node-RED → HTTP API → Dashboard
(อัปเดตทุก 2 วินาที)
```

### Google Sheets Mode
```
PLC → Node-RED → Google Sheets → Apps Script API → Dashboard
(อัปเดตทุก 5 วินาที เพื่อประหยัด Quota)
```

---

## 📊 โครงสร้างข้อมูล

### ข้อมูลที่แสดงผล:

| ฟิลด์ | รายละเอียด | หน่วย | แหล่งที่มา |
|------|-----------|-------|-----------|
| **Level** | ระดับน้ำปัจจุบัน | Liters | PLC D212 |
| **Flow** | อัตราการไหล | L/min | PLC D501 |
| **Temp** | อุณหภูมิน้ำ | °C | PLC D502 |
| **Status** | สถานะถังน้ำ | NORMAL/LOW | คำนวณจาก Level |
| **Timestamp** | เวลาอัปเดตล่าสุด | ISO 8601 | System |

---

## 🐛 แก้ไขปัญหา

### ❌ "Error (Google Sheets)"

**สาเหตุที่เป็นไปได้:**
1. URL ไม่ถูกต้อง
2. Apps Script ยังไม่ Deploy
3. Permission ตั้งค่าผิด

**วิธีแก้:**
1. ตรวจสอบ URL ใน Settings
2. Deploy Apps Script ใหม่ โดยเลือก "Anyone" access
3. ลองเปิด URL ใน Browser tab ใหม่

### ❌ ข้อมูลแสดง 0 ทั้งหมด

**สาเหตุ:** Sheet ไม่มีข้อมูล

**วิธีแก้:**
1. เปิด Google Sheets
2. ตรวจสอบว่ามีข้อมูลในบรรทัดที่ 2 ลงไป
3. ถ้าไม่มี ให้รอ Node-RED ส่งข้อมูลมา หรือใส่ข้อมูลทดสอบเอง

### ❌ CORS Error

**สาเหตุ:** Browser block cross-origin request

**วิธีแก้:**
- ใช้ **Google Sheets mode** แทน (Google Scripts รองรับ CORS)
- หรือใช้ Local Server: `python -m http.server 8000`

---

## 📁 ไฟล์ที่สำคัญ

| ไฟล์ | รายละเอียด |
|------|-----------|
| `index.html` | Dashboard หลัก ⭐ |
| `styles.css` | CSS สำหรับ UI |
| `google_apps_script_api.gs` | Apps Script สำหรับ Google Sheets |
| `node_red_flow.json` | Node-RED flow สำหรับอ่าน PLC |
| `GOOGLE_SHEETS_SETUP.md` | คู่มือติดตั้ง Google Apps Script |
| `README_WATER_TANK.md` | คู่มือการใช้งานทั่วไป |

---

## 💡 เคล็ดลับ

### เปลี่ยนความเร็วการอัปเดต

ใน `index.html` line ~396:
```javascript
// Google Sheets (ช้ากว่าเพราะ quota limit)
const interval = setInterval(fetchData, 5000); // 5 วินาที

// เปลี่ยนเป็น
const interval = setInterval(fetchData, 10000); // 10 วินาที
```

### เพิ่มข้อมูล Historical Data

ใน Google Apps Script เพิ่ม endpoint:
```javascript
?action=getAll  // ดึงข้อมูลทั้งหมด
```

### Debug Mode

เปิด Browser Console (F12) เพื่อดู:
- Network requests
- Response data
- JavaScript errors

---

## 🎯 แนะนำการใช้งาน

| Scenario | Data Source ที่แนะนำ |
|----------|---------------------|
| **ทดสอบ UI** | 🛠 Simulation |
| **ดูข้อมูลเรียลไทม์** | 🌐 Node-RED |
| **ดูข้อมูลย้อนหลัง** | 📊 Google Sheets |
| **ไม่มี PLC** | 📊 Google Sheets (ใส่ข้อมูลเอง) |

---

## 📞 สรุปการติดตั้ง

### สำหรับ Google Sheets Mode:

```bash
1. ✅ เตรียม Google Sheets (Sheet ชื่อ "Tank")
2. ✅ Deploy Apps Script (Google Apps Script API)
3. ✅ คัดลอก URL
4. ✅ เปิด index.html → Settings → เลือก Google Sheets
5. ✅ วาง URL → Close
6. ✅ คลิก Water Tank tab → เห็นข้อมูล!
```

### สำหรับ Node-RED Mode:

```bash
1. ✅ Import node_red_flow.json
2. ✅ Deploy
3. ✅ Test API: http://192.168.7.86:1880/machines
4. ✅ เปิด index.html → เลือก Node-RED mode
5. ✅ ดูข้อมูลใน Water Tank tab
```

---

## 🔗 ทรัพยากรเพิ่มเติม

- [Google Apps Script Documentation](https://developers.google.com/apps-script)
- [Node-RED Documentation](https://nodered.org/docs/)
- [React Hooks Reference](https://react.dev/reference/react)

---

**สร้างโดย:** Antigravity AI Assistant  
**วันที่อัปเดตล่าสุด:** 2026-01-12  
**เวอร์ชัน:** 2.0 (เพิ่ม Google Sheets Support)

---

## 🙏 ขอบคุณที่ใช้งาน!

หากมีคำถามหรือต้องการความช่วยเหลือ สามารถเปิด issue หรือติดต่อได้เลยครับ 😊
