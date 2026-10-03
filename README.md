# ESP32 Marauder — Flash Kit + Web Serial Console

A ready-to-flash package of **ESP32 Marauder v1.10.2 (old hardware build)** firmware binaries, the full **wiring/pinout**, and a custom browser-based **Serial Console** (`serial_console.html`) for controlling the device over USB with no extra software.

> ⚠️ **Legal notice:** Marauder includes WiFi/Bluetooth tools that are unlawful to use against networks or devices you don't own or have explicit permission to test. This repo is for **education and authorized testing only**. You are responsible for how you use it.

---

## 📸 Project Pictures

<!-- Replace the paths below with your own images (e.g. put them in an /images folder) -->

| Hardware Build | Running Device |
| :---: | :---: |
| ![Hardware build](images/build.jpg) | ![Device running](images/device.jpg) |

| Wiring / Close-up | Serial Console |
| :---: | :---: |
| ![Wiring](images/wiring.jpg) | ![Serial console](images/console.png) |

---

## 📖 What Is ESP32 Marauder?

**ESP32 Marauder** is an open-source suite of **WiFi and Bluetooth offensive and defensive tools** that runs on an ESP32. It turns a cheap ESP32 + touchscreen into a portable device for **testing and analyzing WiFi and Bluetooth devices**. It was originally inspired by Spacehuhn's `esp8266_deauther`.

### Typical uses
- Learning how WiFi and BLE traffic works (beacons, probe requests, handshakes)
- Wireless security auditing of **your own** networks and devices
- Detecting suspicious devices nearby (BLE card skimmers, Pwnagotchis, deauth attacks)
- Capturing traffic to `.pcap` files on an SD card for analysis in Wireshark
- Embedded / IoT wireless experiments

### Core capabilities (from the original project)
| Category | Features |
| --- | --- |
| **WiFi sniffing** | Probe request sniff, beacon sniff, packet monitor (channel density graph), EAPOL/PMKID capture, deauth sniff |
| **WiFi beacon tools** | Beacon spam (custom list / random), Rick Roll beacon, generate/add/clear SSIDs |
| **Detection** | Detect Pwnagotchi, detect Espressif devices, detect deauthentication packets |
| **Bluetooth** | BLE sniffer, detect Bluetooth credit card skimmers |
| **Utility** | Join WiFi, shutdown WiFi/BLE to save RAM, on-screen draw app, save PCAP files to SD, OTA firmware update (web or SD) |

> The list above comes from the original project's README. Newer firmware (like v1.10.2) adds more features, so check `help` on the device and the [original releases](https://github.com/justcallmekoko/ESP32Marauder/releases) for the current list.

---

## 🔌 Pinout & Wiring

This is the reference wiring for a **2.8" ILI9341 TFT touchscreen** (with SD card slot) connected to an ESP32 dev board. Refer to the pinout sheet of your specific ESP32 board.

### TFT display, touch & SD card

| SD Card | 2.8" TFT | ESP32 |
| ------- | -------- | ----- |
|         | VCC      | VCC   |
|         | GND      | GND   |
|         | CS       | GPIO17 |
|         | RESET    | GPIO5 |
|         | D/C      | GPIO16 |
| SD_MOSI | MOSI     | GPIO23 |
| SD_SCK  | SCK      | GPIO18 |
|         | LED      | GPIO32 |
| SD_MISO | MISO     | GPIO19 |
|         | T_CLK    | GPIO18 |
|         | T_CS     | GPIO21 |
|         | T_DI     | GPIO23 |
|         | T_DO     | GPIO19 |
|         | T_IRQ    | *(not connected)* |
| SD_CS   |          | GPIO12 |

**Notes on the pinout**
- **Shared SPI bus:** the display, touch controller and SD card share the same SPI lines, which is why some pins repeat:
  - **GPIO23** → MOSI / T_DI / SD_MOSI
  - **GPIO18** → SCK / T_CLK / SD_SCK
  - **GPIO19** → MISO / T_DO / SD_MISO
- Each device has its own chip select: **TFT CS = GPIO17**, **Touch CS = GPIO21**, **SD CS = GPIO12**.
- **GPIO16** = display data/command (D/C), **GPIO5** = display reset, **GPIO32** = backlight (LED) control.
- **T_IRQ** (touch interrupt) is left unconnected.

### Battery & charge detection (optional)

For the analog battery circuit, use a **4:1 voltage divider** and (optionally) a MOSFET. For the charge detection circuit, use a **1:2 voltage divider**. Charge detection is optional and only changes the battery icon colour while charging.

| Battery | ESP32 |
| ------- | ----- |
| BAT +    | GPIO34 |
| MOSFET   | GPIO13 |
| CHARGE + | GPIO27 |

> If you use the analog battery circuit when building from source, set `#define BATTERY_ANALOG_ON` to `1` in `MenuFunctions.h`. Pre-built binaries (this repo) don't need this.

### Full pin summary

| ESP32 GPIO | Function |
| ---------- | -------- |
| GPIO5  | TFT RESET |
| GPIO12 | SD card CS |
| GPIO13 | Battery MOSFET |
| GPIO16 | TFT D/C |
| GPIO17 | TFT CS |
| GPIO18 | SPI SCK (TFT SCK, T_CLK, SD_SCK) |
| GPIO19 | SPI MISO (TFT MISO, T_DO, SD_MISO) |
| GPIO21 | Touch CS (T_CS) |
| GPIO23 | SPI MOSI (TFT MOSI, T_DI, SD_MOSI) |
| GPIO27 | Charge detection (CHARGE +) |
| GPIO32 | TFT backlight (LED) |
| GPIO34 | Battery voltage (BAT +, input only) |
| VCC / GND | Power for TFT |

> ⚠️ This pinout is the original reference design. If your board is a ready-made Marauder or a different variant, your wiring may differ, so check your board's documentation.

---

## 📦 What's Inside

| File | Purpose | Flash Address |
| --- | --- | --- |
| `esp32_marauder_ino_bootloader.bin` | Second-stage bootloader | `0x1000` |
| `esp32_marauder_ino_partitions.bin` | Partition table | `0x8000` |
| `boot_app0.bin` | OTA data / boot selector | `0xE000` |
| `esp32_marauder_v1_10_2_20260206_old_hardware.bin` | Marauder firmware (v1.10.2, old hardware build) | `0x10000` |
| `serial_console.html` | Web Serial API terminal for the device | — |

---

## ⚡ Flashing the Firmware

### Option 1 — Spacehuhn Web Flasher (easiest, no install)

Use the browser-based flasher: **https://esptool.spacehuhn.com/**

1. Open the link in **Google Chrome or Microsoft Edge** (desktop).
2. Plug in your ESP32 with a USB **data** cable.
3. Click **Connect** and select your ESP32's serial port.
4. Add each file with its address:

   | Address | File |
   | --- | --- |
   | `0x1000`  | `esp32_marauder_ino_bootloader.bin` |
   | `0x8000`  | `esp32_marauder_ino_partitions.bin` |
   | `0xE000`  | `boot_app0.bin` |
   | `0x10000` | `esp32_marauder_v1_10_2_20260206_old_hardware.bin` |

5. Click **Program** and wait until it reaches 100%.
6. Press the **EN/RST** button (or replug USB) to reboot into Marauder.

If it can't connect, hold the **BOOT** button while pressing Connect/Program.

### Option 2 — esptool (command line)

```bash
pip install esptool
```

```bash
esptool.py --chip esp32 --port COM3 --baud 921600 \
  --before default_reset --after hard_reset \
  write_flash -z --flash_mode dio --flash_freq 80m --flash_size detect \
  0x1000  esp32_marauder_ino_bootloader.bin \
  0x8000  esp32_marauder_ino_partitions.bin \
  0xE000  boot_app0.bin \
  0x10000 esp32_marauder_v1_10_2_20260206_old_hardware.bin
```

Replace `COM3` with your port (e.g. `/dev/ttyUSB0` on Linux). Lower the baud to `460800` or `115200` if flashing fails.

### Option 3 — Update from Marauder itself

On the device: `Device` → `Update Firmware` → **Web Update** or **SD Update** (copy the firmware `.bin` to the SD card root as `update.bin`).

---

## 🎮 How to Use Marauder

There are **two ways** to control the device after flashing.

### Method 1 — Touchscreen menu (standalone)

1. Power on the ESP32 with the screen connected.
2. Navigate the on-screen menu by touch. Main sections include **WiFi**, **Bluetooth**, **Device** and **General Apps**.
3. Pick a tool (e.g. a sniffer or scan), start it, and watch results on screen.
4. Insert a **FAT-formatted microSD card** to save `.pcap` captures and logs.
5. Use `Device` for device info, firmware updates and reboot.

### Method 2 — Serial Console (`serial_console.html`)

Control Marauder from your computer through the included web terminal. This works with or without a screen.

1. Open `serial_console.html` in **Chrome or Edge** (Firefox and Safari don't support Web Serial).
2. Set the baud rate to **115200**.
3. Click **Connect** and choose your ESP32's COM port.
4. Type a command and press **Enter** or **Send**.
5. Click **Disconnect** when finished.

**Console features:** live RX/TX indicators, connection status, adjustable baud rate, clear output, retro green-on-black terminal theme.

> Close any other program using the port (Arduino Serial Monitor, PuTTY, etc.) before connecting.

#### Example CLI commands

Type `help` in the console for the full list for your firmware version:

```text
help            # list available commands
scanap          # scan for access points
list -a         # list scanned APs
sniffbeacon     # sniff beacon frames
sniffprobe      # sniff probe requests
stopscan        # stop the current scan
```

> Command names can differ between firmware versions; always check `help`.

---

## 🛠️ Troubleshooting

| Problem | Fix |
| --- | --- |
| Port not showing up | Install CH340 / CP210x USB driver; try another cable or port |
| `Failed to connect to ESP32` | Hold BOOT while connecting; lower baud rate |
| Boot loop after flashing | Erase flash (`esptool.py erase_flash`) and re-flash all four files at the correct addresses |
| Blank / white screen | Recheck TFT wiring (CS, D/C, RESET, MOSI, SCK) against the pinout above |
| Touch not responding | Check T_CS (GPIO21) and shared SPI lines (GPIO18/19/23) |
| SD card not detected | Check SD_CS (GPIO12); use a FAT-formatted card (some Samsung cards are reported to fail) |
| Console shows garbage | Baud rate mismatch, use 115200 |
| "Web Serial API not supported" | Use Chrome or Edge on desktop |

---

## 📁 Repository Structure

```text
.
├── boot_app0.bin
├── esp32_marauder_ino_bootloader.bin
├── esp32_marauder_ino_partitions.bin
├── esp32_marauder_v1_10_2_20260206_old_hardware.bin
├── serial_console.html
├── images/            # your project photos
└── README.md
```

---

## 🙏 Credits & Shoutout

Huge thanks and full credit to **[justcallmekoko](https://github.com/justcallmekoko)**, the creator of **[ESP32 Marauder](https://github.com/justcallmekoko/ESP32Marauder)**. The firmware binaries and pinout used in this project come from his open-source work. Please go star the original repo, support the project, and check out his work:

- 🔗 GitHub: https://github.com/justcallmekoko/ESP32Marauder
- 🛒 Official hardware: [ESP32 Marauder on Tindie](https://www.tindie.com/products/justcallmekoko/esp32-marauder/)

Also thanks to **[Spacehuhn](https://github.com/spacehuhn)** for the original [esp8266_deauther](https://github.com/Spacehuhn/esp8266_deauther) that inspired Marauder, and for the [web flasher](https://esptool.spacehuhn.com/) used to upload the firmware.

All rights to the Marauder firmware belong to its original author and are subject to the license in the [original repository](https://github.com/justcallmekoko/ESP32Marauder/blob/master/LICENSE).

---

<p align="center">Built by <b>Harry</b> · Serial console powered by Hassan Ali</p>
