# Project AEGIS
**No Radio. No Cloud. No Leaks.**  
A fully offline hardware vault for storing and generating Time-based One-Time Passwords (TOTP) with maximum security.

><img width="658" height="761" alt="image" src="https://github.com/user-attachments/assets/4993c92b-23a1-4115-bc2f-4ef59b50db29" />

> <img width="1028" height="775" alt="image" src="https://github.com/user-attachments/assets/3df3f15b-1e69-46a1-9b03-70e810fe2e81" />


---

## About The Project
In an era of cloud breaches ,remote malware, software-based 2FA apps on connected smartphones are no longer enough for critical assets.

**Project AEGIS**  is a dedicated hardware token built on a **"Zero Trust"** philosophy.  
It lacks a USB data connection to the internet, Bluetooth, and Wi-Fi.  
It keeps your cryptographic texts separate from the network, making sure that your 2FA codes can only be created by someone holding the physical device with a password.



---

## Key Features -
**Air-Gapped Security:** Zero wireless connectivity.  Attackers cannot hack what they cannot reach.  
**Dual-Core Isolation:** Security functions are carried out on a separate core from the user interface.  
**Hardware Root of Trust:** TOTP secrets are stored inside two tamper-resistant cryptographic chips - ATECC608B, not in the microcontroller's flash memory.  
**A high-accuracy MEMS RTC:** DS3231MZ module is used in precision timekeeping to keep time for years without syncing.  
**A physical slide switch:** This totally disconnects the battery is known as a "hardware kill switch."  
**A clear 128x64 display with a tactile dial** for choosing accounts is part of the OLED and rotary interface.  

---

## Hardware Architecture

The device is built around the **Seeed Studio XIAO ESP32-S3**, chosen for its dual-core power and compact footprint.  
The architecture is designed to minimize attack surfaces.

### 1. XIAO ESP32-S3
 - **Why:** Dual-core Xtensa LX7 processor running at 240MHz.  BLE and Wi-Fi radios are disabled in firmware.  
 - **Function:**  drives the display, coordinates user input, and queries secure elements.
   
   <img width="1400" height="1050" alt="image" src="https://github.com/user-attachments/assets/87b28849-92e9-4488-ba8c-0940f1110e8f" />

### 2.Dual ATECC608B Crypto Chips - 
**Why:** Private keys stored in hardware-protected slots.  TOTP seeds cannot be extracted from firmware dumps.  
 - **Dual-Head Design:**  To prevent address conflicts and double storage capacity, two chips are placed on different I2C buses (Wire and Wire1).

   <img width="800" height="600" alt="image" src="https://github.com/user-attachments/assets/0fa446ba-0468-4c35-9670-317bfc95d32b" />

   
### 3.  DS3231MZ RTC
 The reason for this is that TOTP demands exact timing.  Standard MCUs drift seconds per day.  
 - **Power:** Dedicated CR2032 coin cell keeps RTC running when the main battery is dead.

 - <img width="550" height="462" alt="image" src="https://github.com/user-attachments/assets/063f1271-2008-4145-bdaa-f7ce302265c6" />


### 4. Power & Protection - 
 - **Power Source:** 3.7V LiPo Battery (2500mAh) for months of usage.  
 - **Monitoring:** Voltage divider allows OS to display battery voltage.  
 - **Isolation:** SPDT slide switch cuts main power line for true "off" state.

---

## Circuit Design
- Designed in **KiCad 8.0**. 

> <img width="3507" height="2480" alt="image" src="https://github.com/user-attachments/assets/1c110e31-b853-4df5-901f-0ecd1a4764d7" />
 
> <img width="706" height="522" alt="image" src="https://github.com/user-attachments/assets/8dad6b93-66d0-47ba-be48-d1655a477708" />


### Key Circuit Details
- **Bus Isolation:** Crypto #1 shares "Noisy Bus" (D4/D5) with OLED and RTC; Crypto #2 on "Quiet Bus" (D0/D1) alone.  
- **Input Ladder:** 4 buttons wired into a resistor ladder network to read all 4 via a single Analog Pin (D7).  
- **Noise Filtering:** Every IC has a 100nF decoupling capacitor next to its power pin.  

---

## PCB Layout
- **Front Layer (UI):** OLED, rotary encoder, tactile buttons, Xiao.  
- **Back Layer (Security Core):** Crypto chips, RTC, battery connections, Coin cell.    
  
> <img width="477" height="401" alt="image" src="https://github.com/user-attachments/assets/74dfb036-3e95-450f-a924-d0c8df2675ea" />
 

---

## Firmware 
Custom firmware written in **C++ (Arduino)**, optimized for security and responsiveness.

### Code Structure
- **Secure Boot:** Initializes I2C buses and verifies crypto chips; halts if missing (tampering detected).  
- **Dual-Core Tasking:**  
  - Core 0: SHA1 hashing and secure element communication.  
  - Core 1: UI, animations, input polling for smooth dial operation.  
- **TOTP Engine:** Implements RFC 6238; reads RTC epoch, hashes with secret using HMAC-SHA1, truncates to 6-digit code.  
