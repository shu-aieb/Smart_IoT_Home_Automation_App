# Smart Home Switchboard Controller

The application is a native Android application that is developed to control, automate and monitor retrofittable wall switchboards of IoT.

This project had significantly more complicated automation schedules that were common to multiple user environments, extremely nested user environments, and automation schedules that were synced directly to physical microcontrollers.

## 📱 Interface Showcase
![UI Showcase](https://github.com/user-attachments/assets/f9fd971c-990b-4a26-a4e6-4c0dab387f8a)

## 🎯 The Problem & Purpose
Traditional smart home solutions are all cloud-based. When internet goes out, user loses control of their home. In addition, prior solutions provided no advanced scheduling to meet specific local requirements, such as fan modulation for temperature.

**The Solution:**
I have built a triple-redundancy network pipeline control application using Firebase Cloud, Local Wi-Fi Sockets and SMS fallback. The mobile app is a configuration engine, compiling the user's schedules and writing them directly to the hardware's local memory, so that automations occur reliably, regardless of network status. This ensures that the house will not require the mobile device to operate, even if it is destroyed or offline.

## 🏗️ Architecture & Tech Stack

*   **Platform:** Native Android (Java, XML)
*   **Backend / State Sync:** Firebase Realtime Database
*   **Hardware Target:** ESP8266 Microcontrollers
*   **Networking:** HTTPS, Local TCP/IP Sockets, GSM Telephony API
*   **Data Persistence:** SharedPreferences / SQLite (Local caching of room hierarchies)

### System Hierarchy
The data model was designed to allow theoretically unlimited growth for the end user:
`User Account` ➔ `Properties (Homes)` ➔ `Zones (Rooms)` ➔ `Switch Nodes (Lights/Fans)` ➔ `Automation Rules`

## ⚡ Key Engineering Decisions

### 1. The Automation & Scheduling Engine
Switch nodes support three distinct operational modes, requiring complex state validation before writing to the hardware:
*   **Manual Override:** Standard bidirectional ON/OFF toggling.
*   **Countdown Mode:** A localized timer (e.g., "Turn off in 45 minutes"). I integrated a hardware handshake where the device fires a physical buzzer sequence upon execution to notify the user in the room.
*   **Cron-Style Scheduling:** Users can program multiple ON/OFF configurations tied to specific days of the week. The app compiles this into a byte-efficient payload, validates it for time-overlap conflicts, and flashes it to the hardware's RTC (Real-Time Clock).

### 2. Thermostatic Fan Modulation
To simulate HVAC cooling without an air conditioner, I built a temperature-reactive fan controller. The app reads ambient temperature data from the hardware sensors. The user defines a target temperature curve, and the app calculates the necessary fan speed percentage, transmitting PWM (Pulse Width Modulation) commands to the switchboard's regulator to stabilize the room climate automatically.

### 3. Bi-directional State Resolution
One of the primary challenges was state desynchronization. If a user physically presses the wall switch, the app must reflect this change instantly. I implemented active socket listeners and Firebase observers that update the UI state optimistically, utilizing conflict resolution to revert the UI if the hardware fails to acknowledge the state change within an acceptable timeout window.

### 4. Triple-Redundancy Communication
I reused the robust networking pipeline, used in our agricultural controllers apps. The app attempts communication with local wi-fi, first if the user is on the same router, silently falls back to a cloud connection or GSM based SMS data sending protocal, based on the options selected by the user. For remote management during total ISP outages, the application can be connected with the device through local IP or SMS Protocal.


## 🐛 Edge Cases Handled
*   **Race Conditions:** Prevented UI tearing when a user rapidly toggles a switch in the app while the hardware is simultaneously reporting a physical button press.
*   **Schedule Conflicts:** Wrote validation logic to prevent users from accidentally scheduling an "ON" and "OFF" command for the exact same minute on the same day.
*   **Network Handoffs:** Built stable connection handlers that prevent socket crashes when the user walks out of the house (transitioning from Local Wi-Fi back to 4G/Firebase).
