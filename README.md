# Smart Home Switchboard Controller

The application is a native Android application that is developed to control, automate and monitor retrofittable wall switchboards of IoT.

This project had significantly more complicated automation schedules that were common to multiple user environments, extremely nested user environments, and automation schedules that were synced directly to physical microcontrollers.

## 📱 Interface Showcase
The Home/Room Dashboard, the Fan Regulation screen, and the Automation/Timer setup screen are shown side-by-side as an illustration.These screens are displayed side by side as an example: Home/Room Dashboard, Fan Regulation screen, and the Automation/Timer setup screen.

## 🎯 The Problem & Purpose
Traditional smart home solutions are all cloud-based. When internet goes out, user loses control of their home. In addition, prior solutions provided no advanced scheduling to meet specific local requirements, such as fan modulation for temperature.

**The Solution:**
I built myself a triple-redundancy network pipeline control application using Firebase Cloud, Local Wi-Fi Sockets and SMS fallback. The mobile app is a configuration engine, compiling the users' schedules and writing them directly to the hardware's local memory, so that automations occur reliably, regardless of network status. This ensures that the house will not require the mobile device to operate, even if it is destroyed or offline.

## 🏗️ Architecture & Tech Stack

What is the native platform for this application? (Java, XML, native Android).
*   **Frontend / UI / UX:** Google Apps Script (GAS)
*   **Electronic Components:** ESP8266 Microcontroller
The networking options available are: HTTPS, Local TCP/IP Sockets, GSM Telephony API.
This is for local caching of room hierarchies, which is found in SharedPreferences / SQLite.This is for local caching of room hierarchies, which is found in SharedPreferences / SQLite.

### System Hierarchy
The data model was designed to allow theoretically unlimited growth for the end user:
To access the automation rules, proceed to User Account, Properties (Homes), Zones (Rooms), Switch Nodes (Lights/Fans) and Automation Rules.

## ⚡ Key Engineering Decisions

### 1. The Automation & Scheduling Engine
Switch nodes can be operated in three different modes and it is complex to validate the state before writing to the hardware.
Manual Override: Normal bidirectional ON/OFF switching.
Countdown Mode: A timer that allows for the setting of a timer (e.g., "Turn off in 45 minutes") for a specific area. Added a hardware handshake, that whenever the device runs the code, it triggers a buzzer sound sequence, which informs the user in the room.
Multiple ON/OFF schedules can be programmed by the user based on the days of the week, Cron-Style Scheduling. This is then combined into a payload which is compact in terms of bytes and checked to prevent time-overlaps, and then flashed to the RTC (Real-Time Clock) of the hardware.

### 2. Thermostatic Fan Modulation
I designed a fan controller to control the fan and simulate HVAC cooling for a house without an air conditioner. The app reads the ambient temperature data from the ambient temperature sensor hardware. The user sets a target temperature curve, the application will calculate the percentage to be used for the fan speed, and PWM (Pulse Width Modulation) signals will be sent to the regulator in the switchboard to control the room temperature automatically.

### 3. Bidirectional State Resolution
The state desynchronization was one of the main problems. For any user who physically presses the wall switch, the app should update the wall switch immediately. I used active socket listeners and Firebase observers to update the UI state optimistically, and used the conflict resolution to roll back the UI in case the hardware does not accept the change in state within a reasonable time window.

### 4. Triple-Redundancy Communication
I took over the strong networking pipe that was our agricultural controllers. The app is configured to try the cloud connection first and then automatically fail over to an IP-based connection if the user is on same router – during periods of total ISP failure, it will open a raw SMS command interface for remote management.

## 🐛 Edge Cases Handled
Race conditions: UI doesn't tear when the user rapidly flips a switch in the app and the hardware reports a button press.
Add validation to stop the user from scheduling "ON" and "OFF" instructions at the same minute of the same day.
Improved socket stability with stable connection handlers during Network Handoffs, which means that the app does not crash while the user is moving from Local Wi-Fi to 4G / Firebase.
