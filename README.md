# Oxygen X-Air V400E Integration for Home Assistant

![Home Assistant](https://img.shields.io/badge/Home%20Assistant-Integration-blue?style=for-the-badge&logo=home-assistant)
![Protocol](https://img.shields.io/badge/Protocol-Modbus%2FHTTP-orange?style=for-the-badge)
![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg?style=for-the-badge)

> 🐣 **First Project Alert:** This is my first release on GitHub.  
> It works on my machine! If it doesn't work on yours, please let me know nicely.

This repository documents the integration of **Oxygen X-Air V400E** (and compatible ventilation units) into **Home Assistant**.  
The solution works locally via HTTP requests using the internal protocol.

## 🚀 Features

* **Local Control:** No cloud dependency.
* **Full Monitoring:** Temperatures, airflow, fan speeds, and filter status.
* **Easy Dashboard:** Pre-configured Lovelace card.

<img width="543" alt="Oxygen dashboard" src="https://github.com/user-attachments/assets/6a86b0c1-85da-4653-8218-e6998c01068e" />

---

## 🛠 Prerequisites

1.  **Device:** Oxygen Ventilation unit (e.g., V400E).
2.  **Network:** The unit must be connected to your local network via Ethernet.
3.  **Home Assistant:** Installed and running.

---

## 📦 Installation Guide

### Step 1: File Setup
1.  Open **Visual Studio Code** (or your preferred file editor) in Home Assistant.
2.  Navigate to your configuration directory.
3.  Create a folder named `packages` (if it doesn't exist).
4.  Copy the `hvac.yaml` file from this repository into the `packages/` folder.

### Step 2: Configuration
You must update the IP address to match your unit.

1.  Open `packages/hvac.yaml`.
2.  Press `CTRL + F` to find and replace.
3.  **Find:** `192.168.0.XXX`
4.  **Replace with:** Your actual HVAC unit IP address (e.g., `192.168.1.22`).
    * *Example:* `resource: "http://192.168.0.XXX/cmd?GI57"` becomes `resource: "http://192.168.1.22/cmd?GI57"`.

### Step 3: Verify & Restart
1.  Go to **Developer Tools** > **YAML**.
2.  Click **Check Configuration**.
3.  If valid, restart Home Assistant.

### Step 4: Dashboard
1.  Open your Dashboard.
2.  Create a new card (Manual).
3.  Paste the code from `card.yaml` provided in this repository.
4.  Enjoy your control!

---

<br>

<details>
<summary><h2>⚠️ Advanced Documentation / Reverse Engineering</h2>
(Click to expand if you are a developer or want to use raw HTTP commands)
</summary>

<br>

> **Warning:** This section is intended for advanced users who want to control the unit directly via HTTP requests (curl, Python, Node-RED) or understand the underlying protocol.

The controller exposes data via `http://<IP>/cmd?XXX` endpoints.

### 📊 Reverse Engineered Data Points (Codes)

#### 📡 Read-Only Sensors (GI - Get Input)
| Code | Description | Unit | Notes |
| :--- | :--- | :--- | :--- |
| **GI0** | Supply Air Temp | °C | Value / 10 |
| **GI1** | Outdoor Air Temp | °C | Value / 10 |
| **GI2** | Extract Air Temp | °C | Value / 10 |
| **GI3** | Extract Air Humidity | % | Value / 10 |
| **GI4** | Exhaust Air Temp | °C | Value / 10 |
| **GI7** | Supply Filter Pressure | Pa | Handle integer overflow (>32768 = negative) |
| **GI8** | Extract Filter Pressure | Pa | Handle integer overflow (>32768 = negative) |
| **GI23** | Supply Fan Control | % | 29.0 = 30%, 40.0 = 50% |
| **GI24** | Extract Fan Control | % | 29.0 = 30%, 40.0 = 50% |
| **GI25** | Supply Fan Speed | RPM | |
| **GI26** | Extract Fan Speed | RPM | |
| **GI27** | Supply Filter Timer | h | |
| **GI28** | Extract Filter Timer | h | |
| **GI30** | Bypass Valve Position | % | 0 - open, 100 - closed |
| **GI31** | Preheater | % | 0 - off, 100 - on |
| **GI57** | Supply Voltage (Approx) | V | Values ~140-220 |

#### ⚙️ Setpoints & Status (SR - Set Register)
Accessible mostly via `cmd?GA` (Get All) or specific set commands.

| Code | Description | Notes |
| :--- | :--- | :--- |
| **SR0** | Status | 1=On, 0=Off |
| **SR1** | Current Fan Speed Target | % |
| **SR2** | Target Temperature | °C |
| **SR8** | Supply Flow Balance | +/- % |
| **SR9** | Extract Flow Balance | +/- % |
| **SR12** | Boost Speed | % |
| **SR13** | Boost Duration | min |
| **SR28** | Antifrost Temp Diff | |
| **SR30** | Defrost Time | |

### 🎮 Control Commands (HTTP GET)
Send these commands to `http://<IP>/cmd?<command>`

| Command | Example URI | Description |
| :--- | :--- | :--- |
| `on` | `/cmd?on=1` | Turn the unit **ON** |
| `off` | `/cmd?off=1` | Turn the unit **OFF** |
| `flowset` | `/cmd?flowset=50` | Set airflow intensity (0-100%) |
| `tempset` | `/cmd?tempset=20` | Set target temperature (°C) |
| `restart` | `/cmd?restart` | Reboot the controller |
| `status` | `/status` | Returns XML/JSON status summary |

### 🔍 Data Prefixes Dictionary
* **GI** (Get Input): Read-only sensor data.
* **GR** (Get Register): Internal parameters/counters (e.g., `GR1` = User target flow).
* **SR** (Set Register): User-configurable settings.
* **GA** (Get All): Returns a CSV dump of all parameters.
* **GS** (Get Schedule): Weekly schedule data.

</details>
