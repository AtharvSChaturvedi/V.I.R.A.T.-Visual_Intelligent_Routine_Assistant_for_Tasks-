# V.I.R.A.T. — Visual Intelligent Routine Assistant for Tasks

> An ESP32-based smart study desk assistant that uses motion detection, task scheduling, and real-time Blynk IoT integration to help you stay focused and on track.

---

## 🧠 What is V.I.R.A.T.?

V.I.R.A.T. is an embedded IoT system built on the **ESP32** microcontroller. It sits on your study table and automatically detects when you arrive, displays your scheduled tasks on an OLED screen, tracks time per task, and alerts you as deadlines approach — all controllable remotely via the **Blynk** mobile app.

---

## ✨ Features

- **PIR Motion Detection** — Automatically wakes up and shows your schedule when you sit down
- **Task Scheduling (up to 5 tasks)** — Define task names and durations (hours + minutes) via the Blynk app
- **OLED Display (128×32)** — Scrolling task display, active task timer countdown, and system status
- **LED Indicators** — Visual feedback for active tasks, timer warnings, and task completion
- **Push Button Control** — Start and complete tasks with a single press
- **Blynk Integration** — Remote schedule management, real-time status monitoring, reset and sync controls
- **Timer Warnings** — Blink alerts when ≤5 minutes remain on a task
- **Serial Debug Console** — Commands for schedule display, reset, status, sync, and display testing
- **Offline Mode** — Continues operating if WiFi/Blynk is unavailable

---

## 🔧 Hardware Requirements

| Component | Details |
|---|---|
| ESP32 Dev Board | Any standard 38-pin ESP32 |
| PIR Motion Sensor | HC-SR501 or equivalent |
| OLED Display | 128×32, SSD1306 driver, I2C |
| Push Button | Momentary tactile switch |
| LEDs × 2 | With appropriate resistors (220Ω) |
| Jumper Wires | — |
| USB Cable | For programming and serial monitor |

---

## 📌 Pin Configuration

| Pin | Component | Notes |
|---|---|---|
| GPIO 12 | PIR Sensor | Digital input |
| GPIO 14 | LED 1 | Digital output |
| GPIO 13 | LED 2 | Digital output |
| GPIO 27 | Push Button | Input with internal pull-up |
| GPIO 21 | OLED SDA | I2C data |
| GPIO 22 | OLED SCL | I2C clock |

---

## 📱 Blynk App Setup

### Template Details
| Field | Value |
|---|---|
| Template ID | `TMPL3NAQ4IR0Z` |
| Template Name | `VIRAT Study Automation` |

### Virtual Pin Map

| Virtual Pin | Widget Type | Purpose |
|---|---|---|
| V1 | Text Input | Task 1 Name |
| V2 | Number Input | Task 1 Hours |
| V3 | Number Input | Task 1 Minutes |
| V4 | Text Input | Task 2 Name |
| V5 | Number Input | Task 2 Hours |
| V6 | Number Input | Task 2 Minutes |
| V7 | Text Input | Task 3 Name |
| V8 | Number Input | Task 3 Hours |
| V9 | Number Input | Task 3 Minutes |
| V10 | Text Input | Task 4 Name |
| V11 | Number Input | Task 4 Hours |
| V12 | Number Input | Task 4 Minutes |
| V13 | Text Input | Task 5 Name |
| V14 | Number Input | Task 5 Hours |
| V15 | Number Input | Task 5 Minutes |
| V16 | Button | Sync Schedule |
| V17 | Button | Reset All Tasks |
| V18 | Label | Current Task (Read-only) |
| V19 | Label | System State (Read-only) |
| V20 | Label | WiFi Status (Read-only) |

---

## 🚀 Getting Started

### 1. Install Dependencies

Install the following libraries in your Arduino IDE (via Library Manager or manual install):

- `BlynkSimpleEsp32`
- `Adafruit SSD1306`
- `Adafruit GFX Library`
- `WiFi` (built-in with ESP32 board package)

### 2. Configure Credentials

Open the `.ino` file and update the following fields:

```cpp
char ssid[] = "YOUR_WIFI_SSID";
char pass[] = "YOUR_WIFI_PASSWORD";
#define BLYNK_AUTH_TOKEN "YOUR_BLYNK_AUTH_TOKEN"
```

### 3. Flash the ESP32

1. Select **ESP32 Dev Module** as your board in Arduino IDE
2. Choose the correct COM port
3. Upload the sketch

### 4. Set Up Blynk

1. Create a new template in [Blynk Cloud](https://blynk.cloud) matching the Template ID
2. Add widgets for each virtual pin listed above
3. Configure V16 and V17 as momentary push buttons
4. Configure V18, V19, V20 as display labels

---

## 🔄 System State Machine

```
         Motion Detected
IDLE ────────────────────► SHOWING_SCHEDULE
  ▲                               │
  │    No motion (15s timeout)    │  Button Press (select task)
  │◄──────────────────────────────┘
  │                               ▼
  │                          TASK_ACTIVE
  │                               │
  │    Button Press (complete)    │  ≤5 min remaining
  │◄──────────────────────────────┤────────────► TASK_TIMER_WARNING
  │                               │                     │
  │◄──────────────────────────────┘◄────────────────────┘
       Timer expires / Task done
```

---

## 🖥️ Serial Console Commands

Connect at **115200 baud** and use the following commands:

| Command | Description |
|---|---|
| `SCHEDULE` | Print all pending tasks to Serial |
| `RESET` | Reset all tasks to incomplete |
| `STATUS` | Print current state, active task, WiFi/Blynk status |
| `DISPLAY` | Force schedule display mode (for testing) |
| `SYNC` | Manually push current schedule to Blynk app |

---

## ⚙️ Configuration Constants

You can tune these values at the top of the sketch:

```cpp
const unsigned long cooldown = 10000;          // Motion re-trigger cooldown (ms)
const unsigned long buttonDebounce = 500;      // Button debounce time (ms)
const unsigned long scheduleScrollDelay = 3000; // Time per task on display (ms)
const unsigned long blynkUpdateInterval = 5000; // How often to push status to Blynk (ms)
const int calibrationTime = 10;                // PIR warm-up time (seconds)
```

---

## 📋 How to Use

1. **Power on** — The device calibrates the PIR sensor (10 seconds), then connects to WiFi and Blynk
2. **Set your schedule** — Open the Blynk app and enter task names and durations on V1–V15
3. **Sit at your desk** — Motion triggers the OLED to scroll through your pending tasks
4. **Start a task** — Press the button when the task you want to do is shown on screen
5. **Work** — LEDs stay on; the display shows elapsed/remaining time
6. **Complete a task** — Press the button again to mark it done (LEDs flash as confirmation)
7. **Repeat** for subsequent tasks

---

## 🐛 Troubleshooting

**OLED not displaying:**
- Check SDA/SCL wiring to GPIO 21/22
- The code tries both I2C addresses `0x3C` and `0x3D` automatically
- Run with Serial monitor to see if "OLED init failed" is printed

**WiFi not connecting:**
- Verify SSID and password in the sketch
- Ensure your network is 2.4GHz (ESP32 does not support 5GHz)
- Device will fall back to offline mode after 30 seconds

**PIR not triggering:**
- Allow 10 seconds of calibration time after power-on
- Adjust sensitivity and delay potentiometers on the HC-SR501 module

**Blynk not syncing tasks:**
- Press the Sync button (V16) in the Blynk app to push the current schedule
- Use `SYNC` in serial monitor as an alternative
- Check that auth token matches exactly

---

## 📄 License

This project is open source and free to use for personal and educational purposes.

---

## 👤 Author

**V.I.R.A.T.** — Visual Intelligent Routine Assistant for Tasks  
Built with ❤️ for focused, productive study sessions.
