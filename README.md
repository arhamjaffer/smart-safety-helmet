# smart-safety-helmet
IoT helmet with real-time impact detection, GPS tracking, and live web dashboard for remote monitoring &amp; offline logging
<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0d1117,50:1a1a2e,100:E7352C&height=180&section=header&text=Smart%20Safety%20Helmet&fontSize=42&fontColor=ffffff&animation=fadeIn&fontAlignY=40&desc=ESP32%20%7C%20GPS%20Tracking%20%7C%20Impact%20Detection%20%7C%20Live%20Dashboard&descAlignY=62&descColor=a0aec0"/>

[![ESP32](https://img.shields.io/badge/ESP32-E7352C?style=for-the-badge&logo=espressif&logoColor=white)]()
[![Arduino](https://img.shields.io/badge/Arduino_IDE-00979D?style=for-the-badge&logo=arduino&logoColor=white)]()
[![IoT](https://img.shields.io/badge/IoT-Live_Dashboard-58A6FF?style=for-the-badge)]()
[![Status](https://img.shields.io/badge/Status-✅_Completed-brightgreen?style=for-the-badge)]()
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)]()

</div>

---

## 📌 Overview

A **smart motorcycle safety helmet** powered by **ESP32** that detects accidents in real-time using impact sensors, tracks the rider's live GPS location, and streams all data to a **web dashboard** accessible from any browser — no app install needed.

If an impact is detected above a threshold, the system **automatically logs the event** with timestamp and GPS coordinates, enabling emergency response or remote monitoring by family members.

> Built and tested as a working prototype at NUST-PNEC, Karachi.

---

## 🖼️ Project Gallery

| Helmet Setup | 
|-------------|
| ![Helmet](https://github.com/arhamjaffer/smart-safety-helmet/blob/main/smart_helmet_prototype.png) | 

## ✨ Features

| Feature | Description |
|---------|-------------|
| 🚨 **Impact Detection** | FSR sensors detect collision force above a set threshold |
| 📍 **GPS Tracking** | Real-time location via NEO-6M GPS module |
| 🌐 **Live Web Dashboard** | HTML/CSS/JS dashboard served directly from ESP32 |
| 📶 **WiFi Connectivity** | ESP32 hosts its own access point or connects to existing WiFi |
| 💾 **Offline Logging** | Impact events logged locally when no internet is available |
| 🔴 **Alert LED** | Visual indicator on helmet when impact is detected |
| ⏱️ **Timestamped Events** | Every impact logged with time and GPS coordinates |
| 🔋 **Battery Powered** | Compact Li-ion powered, suitable for helmet mounting |

---

## 🧰 Hardware Used

| Component | Model | Purpose |
|-----------|-------|---------|
| Microcontroller | ESP32 (38-pin) | Main brain — WiFi + processing |
| GPS Module | NEO-6M | Real-time location tracking |
| Force Sensor | FSR 402 (×2) | Impact / collision detection |
| LED | Red 5mm | Visual impact alert |
| Resistors | 10kΩ pull-down | FSR signal conditioning |
| Power Supply | Li-ion 3.7V + TP4056 | Battery + charging circuit |
| Boost Converter | MT3608 | 3.7V → 5V for stable ESP32 power |
| Wiring | 22 AWG flexible | Helmet-safe flexible wiring |

---

## 🗺️ Circuit Overview

```
FSR Sensor 1 ──┐
FSR Sensor 2 ──┤──→ ESP32 ADC Pins (GPIO 34, 35)
               │
NEO-6M GPS ────┤──→ ESP32 UART (GPIO 16 RX, 17 TX)
               │
Alert LED ─────┤──→ ESP32 GPIO (GPIO 2) + 330Ω resistor
               │
Li-ion Battery─┤──→ TP4056 → MT3608 Boost → ESP32 VIN (5V)
```

> Full schematic available in [`/Schematic/`](Schematic/)

---

## 🗂️ Project Structure

```
smart-safety-helmet/
│
├── 📁 Code/
│   ├── smart_helmet.ino           # Main Arduino sketch
│   ├── dashboard.h                # Web dashboard HTML (embedded)
│   └── config.h                   # WiFi credentials & thresholds
│
├── 📁 Schematic/
│   └── smart_helmet_schematic.pdf # Circuit diagram
│
├── 📁 Dashboard/
│   └── index.html                 # Standalone dashboard preview
│
├── 📁 images/
│   ├── helmet.jpg                 # Physical prototype photo
│   ├── dashboard.png              # Dashboard screenshot
│   └── alert.png                  # Impact alert screenshot
│
└── README.md
```

---

## 💻 How It Works — System Flow

```
Helmet Powers ON
      ↓
ESP32 Boots → Connects to WiFi / Starts Access Point
      ↓
GPS Module locks onto satellites → starts streaming coordinates
      ↓
FSR Sensors continuously sampled (every 100ms)
      ↓
    ┌─────────────────────────────────────────┐
    │  FSR reading > threshold (impact)?      │
    └─────────────────────────────────────────┘
         YES                        NO
          ↓                          ↓
  Alert LED ON              Continue monitoring
  Log event to memory
  (timestamp + GPS coords)
  Send alert to dashboard
          ↓
  Dashboard updates live
  (accessible via browser on same WiFi)
```

---

## 🌐 Web Dashboard

The ESP32 serves a **built-in web dashboard** — no cloud, no app, just open a browser:

```
http://192.168.4.1        ← if connected to helmet's own WiFi hotspot
http://<local-ip>         ← if helmet joined your home WiFi
```

**Dashboard shows:**
- 📍 Live GPS coordinates (with Google Maps link)
- 🚨 Impact event log (time + location)
- 📊 Real-time FSR force readings
- 🟢 / 🔴 Connection status indicator

---

## 🔧 Setup & Flash Instructions

### 1. Install Dependencies
```
Arduino IDE → Board Manager → Install: ESP32 by Espressif
Arduino IDE → Library Manager → Install:
  - TinyGPS++ by Mikal Hart
  - ESPAsyncWebServer
  - AsyncTCP
```

### 2. Configure WiFi & Thresholds
Open `config.h` and update:
```cpp
#define WIFI_SSID     "YourWiFiName"
#define WIFI_PASSWORD "YourPassword"
#define IMPACT_THRESHOLD  800    // FSR raw ADC value (0–4095)
#define GPS_BAUD          9600
```

### 3. Flash to ESP32
```bash
# Clone the repo
git clone https://github.com/arhamjaffer/smart-safety-helmet.git

# Open Code/smart_helmet.ino in Arduino IDE
# Select Board: ESP32 Dev Module
# Select correct COM Port
# Upload!
```

### 4. Access Dashboard
- Connect phone/laptop to helmet's WiFi hotspot **SmartHelmet_AP**
- Open browser → `http://192.168.4.1`

---

## 📊 Impact Detection Logic

```cpp
// Threshold-based impact detection (simplified)
int fsrReading = analogRead(FSR_PIN);

if (fsrReading > IMPACT_THRESHOLD) {
    digitalWrite(LED_PIN, HIGH);      // Alert LED ON
    logImpactEvent(fsrReading, gps);  // Save to memory
    sendAlertToDashboard();           // Update web UI
    delay(2000);
    digitalWrite(LED_PIN, LOW);
}
```

---

## ✅ Testing Results

| Test | Result |
|------|--------|
| GPS lock time (open sky) | ~35 seconds |
| Impact detection response time | < 100ms |
| Dashboard load time (local WiFi) | < 1 second |
| Battery life (fully charged) | ~4–6 hours |
| WiFi range (helmet AP) | ~15 meters |
| Offline log capacity | 100+ events |

---

## 🚀 Future Improvements

- [ ] SOS SMS alert via GSM module (SIM800L) on impact
- [ ] Mobile app (Flutter) for real-time notifications
- [ ] Accelerometer (MPU6050) for tilt/fall detection
- [ ] Cloud logging via Firebase or MQTT broker
- [ ] Speed monitoring via GPS data

---

## 🙋 Author

**Arham Jaffer**
B.E. Electrical Engineering — NUST-PNEC, Karachi 🇵🇰
Co-Founder @ [GenXi Tech Solutions](https://www.genxitechsolutions.com/)

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/arham-jaffer-7911a3308)
[![Email](https://img.shields.io/badge/Email-arhamkaimkhani1949%40gmail.com-D14836?style=flat-square&logo=gmail)](mailto:arhamkaimkhani1949@gmail.com)
[![WhatsApp](https://img.shields.io/badge/WhatsApp-Chat-25D366?style=flat-square&logo=whatsapp)](https://wa.me/923078331121)

---

## 📄 License

This project is open-source under the [MIT License](LICENSE). Free to use and build upon — just give credit!

---

<div align="center">

⭐ **If this project helped you, please star the repo!** ⭐

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:E7352C,50:1a1a2e,100:0d1117&height=100&section=footer"/>
</div>
