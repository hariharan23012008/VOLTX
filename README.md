# VoltX ⚡

### IoT-Based Smart Charging Cutoff & Battery Health Optimization System

> **Smart Charging. Smarter Battery Life.**

VoltX is an IoT-based smart charging system designed to automate smartphone charging control. Users can set a target battery percentage through a mobile application, and VoltX automatically controls an ESP32-connected relay to stop charging when the target level is reached.

For example, if the user sets a target of **80%**, VoltX monitors the battery level and automatically cuts off the charging power when the battery reaches **80%**.

VoltX also includes **DependencyGuard**, a dependency monitoring and failure-management system that tracks prerequisites, blockers, paused operations, skipped operations, abandonment reasons, downstream effects, and system recovery.

---

## 🚀 Features

### 🔋 Smart Target Charging

* Set a custom target battery percentage.
* Monitor the current battery percentage.
* Automatically continue charging while:

```text
Current Battery < Target Percentage
```

* Automatically stop charging when:

```text
Current Battery >= Target Percentage
```

* Send a `RELAY_OFF` command to the ESP32.
* Record the charging completion event.

### ⚡ Automatic Charging Cutoff

Example:

```text
Current Battery: 67%
Target Battery: 80%

67% < 80%
        ↓
Relay ON
        ↓
Charging Continues
```

When the battery reaches the target:

```text
Battery: 80%
Target: 80%
        ↓
Target Reached
        ↓
ESP32 Receives Command
        ↓
Relay OFF
        ↓
Charging Power Cut Off
        ↓
Status: AUTO CUTOFF
```

---

## 🏗️ System Architecture

```text
┌─────────────────────┐
│    Android App      │
│                     │
│ • Battery Monitor   │
│ • Target Setting    │
│ • Live Status       │
│ • DependencyGuard   │
└──────────┬──────────┘
           │
           │ Wi-Fi / API
           ▼
┌─────────────────────┐
│       Backend       │
│                     │
│ • Charging Logic    │
│ • Dependency Engine │
│ • Event Logging     │
│ • Real-time Updates │
└──────────┬──────────┘
           │
           │ Command / Status
           ▼
┌─────────────────────┐
│       ESP32         │
│                     │
│ • Wi-Fi Connection  │
│ • Relay Control     │
│ • Device Heartbeat  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│       Relay         │
│                     │
│ ON  → Charging      │
│ OFF → Cutoff        │
└──────────┬──────────┘
           │
           ▼
      Smartphone
```

---

# 🔄 Working Flow

```text
START
  │
  ▼
User Connects Phone to VoltX
  │
  ▼
Open VoltX Mobile Application
  │
  ▼
Read Current Battery Percentage
  │
  ▼
Set Target Percentage
Example: 80%
  │
  ▼
Start Smart Charging
  │
  ▼
Check Prerequisites
  │
  ├── Battery Data Available?
  ├── Wi-Fi Connected?
  ├── Backend Available?
  ├── ESP32 Connected?
  ├── Relay Available?
  └── Safety Condition Valid?
  │
  ▼
All Conditions Valid?
  │
 ┌┴───────────────┐
 │ YES            │ NO
 ▼                ▼
Start Charging    DependencyGuard
 │                │
 ▼                ▼
Monitor Battery   Detect Failure
 │                │
 ▼                ▼
Battery >= Target? Capture Reason
 │                │
 ├── NO           ▼
 │              Map Downstream
 ▼                Impact
Keep Relay ON      │
 │                 ▼
 ▼              Safe Action
Monitor Again      │
                  ▼
              Event History
```

---

# 🎯 Target Percentage Logic

```text
IF SmartCharging == ENABLED

    READ CurrentBatteryPercentage

    READ TargetBatteryPercentage

    IF CurrentBatteryPercentage < TargetBatteryPercentage

        Relay = ON
        ChargingStatus = ACTIVE

    ELSE IF CurrentBatteryPercentage >= TargetBatteryPercentage

        Relay = OFF
        ChargingStatus = AUTO_CUTOFF

        Reason = TARGET_PERCENTAGE_REACHED

        SaveChargingEvent()

        NotifyUser()
```

---

# 🛡️ DependencyGuard

DependencyGuard is VoltX's dependency monitoring system.

It continuously checks the dependencies required for smart charging.

```text
Battery Monitoring
        │
        ▼
Wi-Fi Communication
        │
        ▼
Backend / API
        │
        ▼
ESP32 Controller
        │
        ▼
Charging Decision
        │
        ▼
Relay Control
        │
        ▼
Charging Session
```

If any component fails, DependencyGuard:

1. Detects the failure.
2. Captures the reason.
3. Identifies affected downstream components.
4. Updates component states.
5. Applies a safe fallback action.
6. Saves the event.
7. Monitors system recovery.

---

## 🚨 Example: Wi-Fi Failure

```text
Wi-Fi Connection Lost
        │
        ▼
Wi-Fi Status = PAUSED
        │
        ▼
Reason Captured:
NETWORK_LOST
        │
        ▼
Dependency Mapping
        │
        ├── Backend/API → BLOCKED
        │
        ├── ESP32 → WAITING
        │
        ├── Charging Decision → BLOCKED
        │
        ├── Relay → SAFE HOLD
        │
        └── Charging → PAUSED
```

---

# 📊 Component Status

| Status       | Description                          |
| ------------ | ------------------------------------ |
| 🟢 ACTIVE    | Component is working normally        |
| 🟡 PAUSED    | Component is temporarily interrupted |
| 🟠 BLOCKED   | Waiting for a prerequisite           |
| 🔵 SKIPPED   | Operation intentionally not executed |
| 🔴 ABANDONED | Operation was terminated             |
| ⚪ SAFE_HOLD  | System maintains a safe state        |
| 🟣 WAITING   | Waiting for dependency recovery      |
| 🟢 RESTORED  | Component recovered successfully     |
| ⚫ UNKNOWN    | Component state cannot be confirmed  |

---

# 📱 Mobile Application

The VoltX Android application includes the following screens.

## 🏠 Dashboard

Displays:

* Current Battery Percentage
* Target Percentage
* Charging Status
* Relay Status
* ESP32 Connection
* Wi-Fi Status
* DependencyGuard Status

Example:

```text
┌─────────────────────────────────┐
│            VOLTX ⚡             │
├─────────────────────────────────┤
│                                 │
│       CURRENT BATTERY           │
│             78%                 │
│                                 │
│       TARGET: 80%               │
│                                 │
│  Charging: 🟢 ACTIVE            │
│  Relay: ON                      │
│  ESP32: CONNECTED               │
│  Wi-Fi: CONNECTED               │
│                                 │
│  DependencyGuard: NORMAL        │
│                                 │
│ [ START ]       [ STOP ]        │
└─────────────────────────────────┘
```

---

## 🎯 Set Target

Users can select a custom charging target.

```text
Target Percentage

        80%

    ───────●───────

[ 60% ] [ 70% ] [ 80% ]
[ 85% ] [ 90% ] [ 100% ]

[ SAVE TARGET ]
```

---

## 📡 Live Charging Monitor

```text
Current Battery: 76%

Target: 80%

███████████████░░

Charging: ACTIVE
Relay: ON
ESP32: CONNECTED
Wi-Fi: CONNECTED
```

When the target is reached:

```text
🎉 TARGET REACHED

Current Battery: 80%
Target: 80%

Charging: AUTO CUTOFF
Relay: OFF

Reason:
Target Percentage Reached
```

---

# 📜 Event History

VoltX records important charging and dependency events.

Example:

```text
TODAY – 10:45 PM

Charging Completed

Start Battery: 67%
Target Battery: 80%
Final Battery: 80%

Status:
AUTO CUTOFF

Relay:
OFF

Reason:
TARGET_PERCENTAGE_REACHED
```

---

# 🧪 Demo Mode

Demo Mode allows the complete system to be demonstrated even when ESP32 hardware is unavailable.

Available simulations:

* Simulate Wi-Fi Failure
* Simulate Backend Failure
* Simulate ESP32 Offline
* Simulate Missing Battery Data
* Simulate Invalid Battery Data
* Simulate Relay Failure
* Simulate Manual User Stop
* Simulate Target Reached
* Restore System

Example:

```text
[ SIMULATE WI-FI FAILURE ]

Wi-Fi → PAUSED
        ↓
Backend → BLOCKED
        ↓
ESP32 → WAITING
        ↓
Charging Decision → BLOCKED
        ↓
Relay → SAFE HOLD
        ↓
Charging → PAUSED
```

---

# 🔧 Technology Stack

## Android Frontend

* Kotlin
* Jetpack Compose
* Material Design 3
* MVVM Architecture
* StateFlow
* Coroutines
* Retrofit / Ktor Client
* Room Database
* DataStore
* Navigation Compose

## Backend

* Node.js
* TypeScript
* Express or Fastify
* PostgreSQL
* Prisma ORM
* REST API
* WebSocket / Socket.IO

## IoT Hardware

* ESP32 DevKit
* Relay Module
* Wi-Fi Communication
* Mobile Charging Output

---

# 📂 Project Structure

```text
VoltX/
│
├── frontend/
│   ├── app/
│   ├── data/
│   ├── domain/
│   ├── presentation/
│   ├── ui/
│   ├── navigation/
│   ├── viewmodel/
│   ├── network/
│   ├── database/
│   └── models/
│
├── backend/
│   ├── src/
│   │   ├── controllers/
│   │   ├── services/
│   │   ├── routes/
│   │   ├── middleware/
│   │   ├── models/
│   │   ├── dependency-engine/
│   │   ├── websocket/
│   │   ├── database/
│   │   └── config/
│   │
│   └── prisma/
│
├── esp32/
│   └── VoltX_ESP32.ino
│
├── docs/
│   ├── API_DOCUMENTATION.md
│   ├── DATABASE_SCHEMA.md
│   └── ESP32_INTEGRATION.md
│
├── README.md
└── SETUP.md
```

---

# 🔌 API Endpoints

| Method | Endpoint                        | Description                   |
| ------ | ------------------------------- | ----------------------------- |
| GET    | `/api/system/status`            | Get current system status     |
| POST   | `/api/charging/start`           | Start smart charging          |
| POST   | `/api/charging/stop`            | Stop charging manually        |
| POST   | `/api/charging/target`          | Set target battery percentage |
| GET    | `/api/charging/session/current` | Get current charging session  |
| GET    | `/api/charging/history`         | Get charging history          |
| GET    | `/api/events`                   | Get dependency events         |
| GET    | `/api/dependencies`             | Get dependency graph          |
| POST   | `/api/dependencies/evaluate`    | Evaluate dependencies         |
| POST   | `/api/device/heartbeat`         | ESP32 heartbeat               |
| POST   | `/api/device/status`            | Update ESP32 status           |
| POST   | `/api/device/relay`             | Send relay command            |
| POST   | `/api/device/battery`           | Update battery percentage     |
| POST   | `/api/system/recover`           | Recover system                |

---

# 🗄️ Database Models

The backend manages:

```text
USER
DEVICE
CHARGING_SESSION
SYSTEM_COMPONENT
DEPENDENCY
BLOCKER_EVENT
DOWNSTREAM_IMPACT
RELAY_COMMAND
BATTERY_READING
```

A dependency event stores:

```text
Component Name
Previous Status
Current Status
Reason
Timestamp
Affected Components
Fallback Action
Recovery Status
```

---

# 🔌 ESP32 Communication

The backend sends commands to the ESP32.

Example:

```json
{
  "command": "RELAY_OFF",
  "reason": "TARGET_PERCENTAGE_REACHED",
  "sessionId": "example-session-id"
}
```

Expected ESP32 acknowledgement:

```json
{
  "status": "SUCCESS",
  "relayState": "OFF",
  "timestamp": "2026-08-22T10:45:00Z"
}
```

The application updates the relay status after acknowledgement when available.

If acknowledgement fails:

```text
Relay Status = UNKNOWN

Reason:
RELAY_COMMAND_NOT_ACKNOWLEDGED
```

---

# 🛑 Manual Stop

The user can manually stop charging at any time.

```text
User Presses STOP
        ↓
Send RELAY_OFF
        ↓
ESP32 Acknowledges
        ↓
Relay OFF
        ↓
Charging Status = ABANDONED
        ↓
Reason = USER_STOPPED_CHARGING
        ↓
Save Event
```

---

# 🔄 System Recovery

When a failed dependency is restored:

```text
Wi-Fi Reconnected
        ↓
Backend Restored
        ↓
Battery Data Updated
        ↓
ESP32 Reconnected
        ↓
Charging Decision Recalculated
        ↓
Relay Updated
        ↓
Charging Resumed
OR
Auto Cutoff Applied
```

All recovery events are saved in the Event History.

---

# 🧪 Testing

The project should include tests for:

* Target percentage comparison
* Automatic cutoff
* Relay ON/OFF commands
* Dependency propagation
* Wi-Fi failure
* Backend failure
* ESP32 disconnection
* Relay acknowledgement failure
* Manual charging stop
* Event creation
* System recovery

---

# 🚀 Quick Start

## Backend

```bash
cd backend
npm install
```

Create the environment configuration:

```bash
cp .env.example .env
```

Configure the database and application settings.

Run database migrations:

```bash
npx prisma migrate dev
```

Start the backend:

```bash
npm run dev
```

---

## Android Application

Open the `frontend` folder in Android Studio.

Configure the backend API URL.

Build and run the application on:

* Android Emulator, or
* Physical Android Device.

---

## ESP32

1. Open the ESP32 source file.
2. Configure Wi-Fi credentials.
3. Configure the backend/server address.
4. Upload the firmware.
5. Connect the relay module.
6. Start the VoltX application.

---

# 🎬 Demo Scenario

### Step 1

Current Battery:

```text
67%
```

### Step 2

User sets:

```text
Target = 80%
```

### Step 3

User presses:

```text
START SMART CHARGING
```

### Step 4

System evaluates:

```text
67% < 80%
```

Result:

```text
Charging = ACTIVE
Relay = ON
```

### Step 5

Battery increases:

```text
70%
75%
78%
79%
```

Charging continues.

### Step 6

Battery reaches:

```text
80%
```

The system evaluates:

```text
80% >= 80%
```

### Step 7

VoltX automatically sends:

```text
RELAY_OFF
```

### Step 8

ESP32 switches:

```text
Relay = OFF
```

### Step 9

The application displays:

```text
TARGET REACHED

Charging Automatically Stopped

Target: 80%
Final Battery: 80%

Reason:
TARGET_PERCENTAGE_REACHED
```

---

# 🔮 Future Scope

* AI-based battery charging pattern analysis
* Battery health prediction
* Multiple device management
* Cloud synchronization
* Smart charging schedules
* Energy consumption analytics
* Push notifications
* Voice assistant integration
* Machine learning-based optimal target recommendation

---

# 👨‍💻 Project Goal

VoltX aims to combine **mobile application development, IoT communication, ESP32 control, relay automation, dependency mapping, and real-time monitoring** into a single smart charging platform.

The system provides a functional flow:

```text
Set Target Percentage
        ↓
Monitor Battery
        ↓
Evaluate Dependencies
        ↓
ESP32 Charging Control
        ↓
Automatic Relay Cutoff
        ↓
Event Logging
        ↓
Dependency Failure Detection
        ↓
Safe Recovery
```

## ⚡ VoltX

**Smart Charging • Automatic Cutoff • Dependency Monitoring • Safe Control**

Made for IoT, Embedded Systems, Smart Energy, and Battery Health Optimization.
