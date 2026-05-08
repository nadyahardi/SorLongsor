# 📋 Panduan Setup — EWS Tanah Longsor
### TechnoCorner 2026 | Multi-Depth Sensing + LoRa Ra-02

---

## 🗂️ Struktur File

| File | Fungsi |
|------|--------|
| `node_esp32.ino` | Upload ke ESP32 Node (di bukit, pakai baterai) |
| `gateway_esp32.ino` | Upload ke ESP32 Gateway (di rumah, listrik) |
| `google_apps_script.js` | Deploy di script.google.com sebagai Web App |

---

## 📦 Library yang Dibutuhkan (Arduino IDE)

Install via **Tools → Manage Libraries**:
1. `LoRa` by Sandeep Mistry
2. `ArduinoJson` by Benoit Blanchon (versi 6.x)

---

## 🔧 Langkah Setup (Urutan Penting!)

### STEP 1 — Google Apps Script

1. Buka [script.google.com](https://script.google.com)
2. Buat project baru → paste isi `google_apps_script.js`
3. Ganti `EMAIL_ALERT` dengan email kamu
4. Klik **Deploy → New Deployment**
   - Type: **Web App**
   - Execute as: **Me**
   - Who has access: **Anyone**
5. Klik **Deploy** → copy URL yang muncul (panjang, diawali `https://script.google.com/macros/s/...`)
6. **Buat Google Sheet baru** di akun yang sama (nama bebas)
   - GAS akan otomatis buat tab "SensorLog" saat data pertama masuk

### STEP 2 — Fonnte (Notif WhatsApp Gratis)

1. Daftar di [fonnte.com](https://fonnte.com)
2. Masuk ke dashboard → **Add Device**
3. Scan QR dengan WhatsApp yang ingin dijadikan pengirim
4. Setelah connected, klik device → copy **Token**
5. Token ini dimasukkan ke `gateway_esp32.ino`

> ⚠️ **Catatan Fonnte gratis:** Batas ~100 pesan/bulan untuk akun free.
> Dengan cooldown 15 menit, ini cukup untuk lomba/demo.

### STEP 3 — Gateway ESP32

Buka `gateway_esp32.ino`, ganti bagian ini:
```cpp
const char* WIFI_SSID    = "NAMA_WIFI_KAMU";    // ← WiFi kamu
const char* WIFI_PASS    = "PASSWORD_WIFI";      // ← Password WiFi
const char* GAS_URL      = "https://script.google.com/..."; // ← URL GAS Step 1
const char* FONNTE_TOKEN = "TOKEN_FONNTE_KAMU";  // ← Token Fonnte Step 2
const char* WA_TARGET    = "6281234567890";       // ← No. WA tujuan (tanpa +)
```
Upload ke ESP32 Gateway.

### STEP 4 — Node ESP32

Buka `node_esp32.ino`, sesuaikan kalibrasi sensor:
```cpp
#define CAL_DRY    3200   // Celup sensor ke udara, catat nilai ADC
#define CAL_WET    1500   // Celup sensor ke air, catat nilai ADC
```
Upload ke ESP32 Node.

### STEP 5 — Dashboard Looker Studio (Opsional tapi keren buat lomba!)

1. Buka [lookerstudio.google.com](https://lookerstudio.google.com)
2. **Create → Report → Add Data → Google Sheets**
3. Pilih spreadsheet dari Step 1
4. Buat chart:
   - **Line Chart**: Timestamp (X) vs SM20/SM40/SM60 (Y)
   - **Gauge Chart**: Nilai SM tertinggi
   - **Table**: Log data terbaru
5. Share link → bisa diakses siapa saja tanpa login!

---

## 🔋 Estimasi Daya Node (18650 ~1800mAh)

| Kondisi | Konsumsi |
|---------|----------|
| Deep Sleep | ~0.01 mA |
| Aktif + LoRa TX | ~180 mA |
| Durasi aktif/siklus | ~3 detik |
| Interval tidur | 5 menit |

**Estimasi umur baterai:**
- Arus rata-rata ≈ (180mA × 3s + 0.01mA × 297s) / 300s ≈ **1.8 mA + 0.01 mA ≈ 1.81 mA**
- Umur ≈ 1800 mAh / 1.81 mA ≈ **~1000 jam ≈ 41 hari** 🎉

---

## 📡 Arsitektur Sistem

```
[Lereng/Bukit]                    [Rumah/Basecamp]
┌─────────────────┐               ┌──────────────────────────┐
│  ESP32 Node     │  LoRa 433MHz  │  ESP32 Gateway           │
│  4x Soil Sensor │ ────────────► │  WiFi ke Internet        │
│  18650 Battery  │   ~600m NLOS  │                          │
│  Deep Sleep 5m  │               │  ┌─ Google Sheets (log)  │
└─────────────────┘               │  ├─ Fonnte WA (notif)    │
                                  │  └─ Email (alert)        │
                                  └──────────────────────────┘
                                               │
                                               ▼
                                    [Looker Studio Dashboard]
                                    Bisa diakses siapa saja
                                    via link/URL
```

---

## ⚡ Troubleshooting Umum

| Masalah | Solusi |
|---------|--------|
| LoRa tidak terdeteksi | Cek wiring SPI, pastikan VCC = 3.3V bukan 5V |
| RSSI sangat rendah (<-120dBm) | Antena belum terpasang atau longgar |
| Data tidak masuk Sheets | Cek GAS URL, pastikan Deploy ulang setelah edit script |
| WA tidak terkirim | Cek token Fonnte, pastikan device masih connected di dashboard |
| Sensor selalu 0% atau 100% | Lakukan kalibrasi ulang CAL_DRY dan CAL_WET |
| WiFi tidak connect | Pastikan SSID/password benar, ESP32 hanya support 2.4GHz |

---

## 🎯 Tips untuk Presentasi Lomba

1. **Demo live:** Siapkan WiFi hotspot HP sebagai backup jika WiFi venue tidak stabil
2. **Data dummy:** Jalankan `sendTestData()` di GAS untuk isi sheet sebelum presentasi
3. **Alert demo:** Celupkan sensor ke air → trigger AWAS → tunjukkan WA masuk
4. **Dashboard:** Buka Looker Studio di tablet/layar besar untuk efek visual
