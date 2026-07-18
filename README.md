
![Logo](Team-GenZ.gif)

# Shokher-Hut — An IoT based Smart Cow Selling Hut During Eid al-Adha

**Team:** Team-GenZ  
**Author:** Kamrul Islam Arnob (connect.arnob@gmail.com)

The "Shokher Hut" project is an innovative IoT-based Cow Selling System aimed at revolutionizing the process of buying and selling cows in Bangladesh during Eid al-Adha. It integrates RFID access control, automated hasil (religious tax) collection, environmental monitoring, parking slot management, and a real-time web dashboard — combining IoT firmware (Arduino/ESP8266) with a React frontend.

![Logo](Prototype.jpg)

---

## Table of Contents

- [Use Cases](#use-cases)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Frontend Application](#frontend-application)
  - [Pages & Routes](#pages--routes)
  - [Components Overview](#components-overview)
  - [State Management](#state-management)
  - [Firebase Integration](#firebase-integration)
  - [Scripts](#scripts)
- [IoT / Firmware Components](#iot--firmware-components)
  - [ESP8266 - Hasil Booth](#1-esp8266---hasil-booth)
  - [ESP8266 - Monitor & Checkout](#2-esp8266---monitor--checkout)
  - [Arduino UNO R3 - Cow House](#3-arduino-uno-r3---main-cow-house)
  - [Arduino UNO R3 - Garage](#4-arduino-uno-r3---garage)
  - [Google Sheets Integration](#5-google-sheets-integration)
- [How to Use](#how-to-use)
  - [Prerequisites](#prerequisites)
  - [Frontend Setup](#frontend-setup)
  - [Arduino/ESP8266 Setup](#arduinoesp8266-setup)
  - [Firebase Setup](#firebase-setup)
- [Features](#features)
- [Benefits](#benefits)
- [Acknowledgements](#acknowledgements)
- [Appendix](#appendix)
- [Authors](#authors)

---

## Use Cases

| Use Case | Description |
|---|---|
| **Smart Hasil Collection** | Buyers scan an RFID card at the hasil booth; the system fetches cow/owner info, calculates 5% hasil tax, and opens the gate upon payment. |
| **Authorized Access Control** | Only authorized RFID cards can enter/exit the cow selling premises. Gate servos are controlled based on database verification. |
| **Real-Time Dashboard** | Market administrators monitor cow entries/exits, total hasil collected, sales figures, and recent activity — all updated in real time via Firebase. |
| **Parking Slot Management** | Three parking slots (P1, P2, P3) are tracked in real time. Slots show as Free, Booked (with TTL countdown), or Occupied. |
| **Environmental Monitoring** | Sensors track temperature, smoke/gas, rain, light, and water level inside the cow house. Automatic responses include fan control, shade deployment, water pumping, and fire alerts. |
| **Automated Garage Door** | A PIR motion sensor triggers a stepper motor to open/close the garage door for the cattle escalator. |
| **Cow Record Management** | Admin can add, edit, and delete authorized cards with owner info, cow ID, stall assignment, and hasil payment status. |
| **Audit Trail** | Every RFID scan, hasil transaction, and cow movement is logged with timestamps for full traceability. |

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | React 19, Vite 7, Material UI v7, Emotion, React Router v7 |
| **Backend / Database** | Firebase Realtime Database |
| **IoT Firmware** | Arduino (ESP8266, Arduino UNO R3) |
| **Sensors & Actuators** | RFID (MFRC522), DS18B20, smoke/gas sensor, rain sensor, LDR, ultrasonic, servo motors, stepper motor, DC motor, relay, buzzer |
| **Alternative Backend** | Google Sheets + Apps Script + PushingBox (legacy) |
| **Font** | Poppins (Google Fonts) |
| **Linting** | ESLint 9 (React Hooks & React Refresh) |

---

## Project Structure

```
Shokher-Hut/
├── README.md                          # You are here
├── Prototype.jpg                      # Hardware prototype image
├── Team-GenZ.gif                      # Team logo animation
├── sokher_hut_UI/                     # React frontend application
│   ├── index.html                     # Entry HTML
│   ├── package.json                   # Dependencies & scripts
│   ├── vite.config.js                 # Vite bundler config
│   ├── eslint.config.js               # ESLint config
│   └── src/
│       ├── main.jsx                   # React entry point
│       ├── App.jsx                    # Root component, router, navigation shell
│       ├── index.css                  # Global CSS variables & utilities
│       ├── theme.js                   # MUI theme (green/orange, Poppins font)
│       ├── firebase.js                # Firebase init & database export
│       └── pages/
│           ├── Dashboard.jsx          # Stats overview & activity log
│           ├── Cards.jsx              # Authorized cards CRUD
│           ├── Cows.jsx               # Cow records with merged card data
│           ├── Slots.jsx              # Parking slot visualization
│           ├── Transactions.jsx       # Hasil transactions table
│           └── RfidRecords.jsx        # Raw RFID records log
└── INO File/                          # Arduino/ESP8266 firmware
    ├── For Google sheet/
    │   ├── ESP8266-RFID-Scanner-with google Sheet.ino
    │   └── Google sheet AppScript code.txt
    ├── For Arduino UNO R3/
    │   ├── For Main Cow House.ino
    │   └── For Garage.ino
    └── For ESP8266/
        ├── shokher_hut_hasil_booth.ino
        └── shokher_hut_monitor_and_checkout.ino
```

---

## Frontend Application

The frontend is a single-page application (SPA) built with React 19, Vite 7, and Material UI v7. It serves as the administrative dashboard for the Shokher Hut system.

### Pages & Routes

| Route | Page Component | Description |
|---|---|---|
| `/` | `Dashboard` | Overview: 4 stat cards (Cows Entered, Exited, Hasil Collected, Total Sales) + recent activity log |
| `/cards` | `Cards` | CRUD for authorized RFID cards. Grid view with search, edit dialog (owner, phone, cow ID, hasil, stall, buyer info), delete |
| `/cows` | `Cows` | Table of cow records merged with authorized card data by `cow_id`. Columns: time, cow ID, card ID, owner, phone, stall, hasil amount, paid status, source |
| `/slots` | `Slots` | Visual parking slots (P1–P3). States: `EmptySlot` (Free), `BookedSlot` (Reserved + TTL countdown), `OccupiedSlot` (with owner/card details) |
| *(unused)* | `Transactions` | Hasil transactions table with pagination (defined but not routed) |
| *(unused)* | `RfidRecords` | Raw RFID scan log (defined but not routed) |

### Components Overview

| File | Component(s) | What it does |
|---|---|---|
| `App.jsx` | `App`, `AppContent` | Theme wrapper, responsive drawer (permanent on desktop, toggle on mobile), top AppBar with search, route definitions |
| `Dashboard.jsx` | `Dashboard` | Listens to 4 Firebase refs, computes stats (counts/sums), renders stat cards + recent activity table with type chips (Payment, Hasil, RFID, Other) |
| `Cards.jsx` | `Cards` | Listens to `/authorized_cards`, filters unpaid. Grid with edit/delete. Edit dialog has all card fields. On hasil payment, writes to both `/authorized_cards` and `/cows` |
| `Cows.jsx` | `Cows` | Listens to `/cows` (last 2000) and `/authorized_cards`, merges by `cow_id`. Displays enriched table |
| `Slots.jsx` | `Slots`, `EmptySlot`, `BookedSlot`, `OccupiedSlot` | Listens to `/parking_slots` and `/authorized_cards`. Maps slots to cards by stall. Renders 3 slot cards with distinct visuals per state |
| `Transactions.jsx` | `Transactions` | Listens to `/hasil_transactions` (last 100). Paginated table (6/12/24 per page) |
| `RfidRecords.jsx` | `RfidRecords` | Listens to `/rfid_records`. Simple table with card UID, gate, timestamp |
| `firebase.js` | Firebase DB instance | Initializes Firebase Realtime Database and exports the `database` object |
| `theme.js` | MUI theme | Custom theme: primary=green `#4CAF50`, secondary=orange `#FF9800`, font=Poppins, rounded shapes |

### State Management

No external state library is used. Each page manages its own data with React hooks:
- **`useState`** — local component state
- **`useEffect`** + **`onValue`** — Firebase real-time subscriptions (auto-cleanup on unmount)
- **`useMemo`** — client-side computed data (merged records, filtered lists)

### Firebase Integration

The app connects to Firebase Realtime Database at `sokher-hut-micro-default-rtdb.asia-southeast1.firebasedatabase.app` (configured in `src/firebase.js`).

**Database paths used:**

| Path | Purpose | Used By |
|---|---|---|
| `/authorized_cards/{uid}` | Registered RFID cards with owner info, hasil status, stall | Dashboard, Cards, Cows, Slots |
| `/cows/{id}` | Sold/processed cow records | Dashboard, Cows, Cards |
| `/hasil_transactions/{id}` | Payment transaction log | Dashboard, Transactions |
| `/rfid_records/{id}` | Raw RFID scan events | Dashboard, RfidRecords |
| `/parking_slots/{slot}` | Parking slot state & booking | Slots |

### Scripts

```bash
# In the sokher_hut_UI/ directory:
npm run dev      # Start dev server with HMR
npm run build    # Production build to dist/
npm run preview  # Preview production build
npm run lint     # Run ESLint
```

---

## IoT / Firmware Components

The `INO File/` directory contains Arduino sketches for the hardware layer.

### 1. ESP8266 — Hasil Booth

**File:** `INO File/For ESP8266/shokher_hut_hasil_booth.ino`

**Hardware:** ESP8266, MFRC522 RFID reader, servo motor (gate), buzzer, transport sensors

**Flow:**
1. Scan RFID card → reads/writes database (Deta Base v1 — legacy)
2. Serial interaction: enter owner name, cattle info, price
3. Calculates 5% hasil (religious tax)
4. Opens servo gate upon payment
5. Allocates transport parking slot (P1–P3)

### 2. ESP8266 — Monitor & Checkout

**File:** `INO File/For ESP8266/shokher_hut_monitor_and_checkout.ino`

**Hardware:** ESP8266, MFRC522 RFID reader, servo motor (gate)

**Flow:**
1. Scan RFID card → verify hasil_paid status from database
2. If paid → open gate (exit)
3. If unpaid → deny access

### 3. Arduino UNO R3 — Main Cow House

**File:** `INO File/For Arduino UNO R3/For Main Cow House.ino`

**Sensors:** DS18B20 (temperature), smoke/gas, rain, LDR (light), ultrasonic (water level)

**Actuators:** Servo (sunshade), DC motor (fan), relay (water pump), buzzer, LED

**Automatic features:**
- **Sunshade:** Deploys when rain detected or temperature exceeds threshold
- **Fan:** Activates when temperature > 40°C
- **Water pump:** Auto-fills when water level drops
- **Lighting:** LED turns on when LDR detects darkness
- **Fire detection:** Smoke + heat triggers emergency exit servo, buzzer, water pump, and fan

### 4. Arduino UNO R3 — Garage

**File:** `INO File/For Arduino UNO R3/For Garage.ino`

**Hardware:** Stepper motor, PIR motion sensor

**Flow:** Motion detected → stepper motor rotates to open/close garage for cattle escalator

### 5. Google Sheets Integration

**Files:**
- `INO File/For Google sheet/ESP8266-RFID-Scanner-with google Sheet.ino`
- `INO File/For Google sheet/Google sheet AppScript code.txt`

An alternative data path where the ESP8266 sends RFID scan + hasil data to Google Sheets via a web app (Apps Script), and receives PushingBox commands for gate control.

---

## How to Use

### Prerequisites

- Node.js 18+ and npm
- Arduino IDE (for firmware upload)
- A Firebase project with Realtime Database
- (Optional) Google account for Sheets integration

### Frontend Setup

```bash
# 1. Navigate to the frontend directory
cd sokher_hut_UI

# 2. Install dependencies
npm install

# 3. Configure Firebase
# Open src/firebase.js and update the firebaseConfig object
# with your own Firebase project credentials.

# 4. Start the development server
npm run dev

# 5. Build for production
npm run build
```

### Arduino/ESP8266 Setup

1. Open the desired `.ino` file in Arduino IDE.
2. Install required libraries:
   - `MFRC522` by GithubCommunity
   - `ESP8266WiFi`
   - `ESP8266HTTPClient`
   - `ArduinoJson`
   - `Servo`
   - `DallasTemperature` (for cow house)
   - `DHT` / `DHT_sensor_library` (for cow house)
3. Update WiFi credentials and database URLs in the sketch.
4. Select the correct board (ESP8266 or Arduino UNO R3) and COM port.
5. Upload the sketch.

### Firebase Setup

1. Create a Firebase project at [firebase.google.com](https://firebase.google.com)
2. Enable **Realtime Database** (choose a location near your region)
3. Configure security rules for your use case
4. Copy your `databaseURL` into `sokher_hut_UI/src/firebase.js`

**Expected database structure:**

```json
{
  "authorized_cards": {
    "uid_1": {
      "owner": "Name",
      "phone": "0123456789",
      "cow_id": "C001",
      "hasil": true,
      "hasil_amount": 500,
      "hasil_paid": false,
      "stall": "P1",
      "buyer_name": "Buyer Name",
      "buyer_phone": "0987654321",
      "info": "Additional info"
    }
  },
  "cows": {
    "id": {
      "cow_id": "C001",
      "card_uid": "uid_1",
      "owner": "Name",
      "hasil_amount": 500,
      "hasil_paid": true,
      "timestamp": 1234567890
    }
  },
  "hasil_transactions": {
    "tx_id": {
      "card_uid": "uid_1",
      "cow_id": "C001",
      "amount": 500,
      "status": "completed",
      "timestamp": 1234567890
    }
  },
  "rfid_records": {
    "id": {
      "card_uid": "uid_1",
      "gate": "entry",
      "timestamp": 1234567890
    }
  },
  "parking_slots": {
    "P1": {
      "Status": "free",
      "booked_by": "",
      "TTL": ""
    }
  }
}
```

---

## Features

### A. Computerized Hasil Counter

1. **Streamlined & Secure Transactions** — RFID-based access with real-time database verification
2. **Gate Access with Authorization** — Only authorized cards can enter the hasil booth
3. **Personalized Interaction** — Buyers are guided through cow selection and payment
4. **Seamless Exit** — Paid status unlocks exit gate

### B. Safety & Comfort Systems

1. **Early Fire Detection** — Smoke/temperature sensors trigger emergency response
2. **Optimal Temperature Control** — Auto fan activation above 40°C
3. **Solar-Powered Shade & Rain Protection** — Automatic canopy deployment

### C. Pick-up Management System

1. **Real-Time Slot Availability** — Live parking slot status (Free/Booked/Occupied)
2. **Cattle Escalator** — Automated garage door for stress-free cow movement

---

## Benefits

- **Holistic Welfare** — Cows' health, comfort, and safety are prioritized
- **Enhanced Safety** — Fire detection and environmental monitoring mitigate hazards
- **Secure Transactions** — Transparent hasil collection builds trust
- **Convenient Pick-up** — Streamlined parking and cattle movement
- **Transparent Process** — Full audit trail for every transaction

---

## Acknowledgements

- [Detaspace: The database documentation used in this project](https://deta.space/docs/en/)
- [ESP-Google-Sheet-Client by mobizt](https://github.com/mobizt/ESP-Google-Sheet-Client)

---

## Appendix

Here is the project report in IEEE format (prepared for an Electronics course):

[https://www.overleaf.com/read/wfqktdrkbzdk](https://www.overleaf.com/read/wfqktdrkbzdk)

---

## Authors

- [@KamrulIslamArnob](https://www.github.com/KamrulIslamArnob)

If you have any feedback, please reach out to me at connect.arnob@gmail.com
