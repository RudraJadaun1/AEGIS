# Project AEGIS
**No Radio. No Cloud. No Leaks.**  
A fully offline hardware  for storing and generating Time-based One-Time Passwords (TOTP) with maximum security.

><img width="658" height="761" alt="image" src="https://github.com/user-attachments/assets/4993c92b-23a1-4115-bc2f-4ef59b50db29" />

> <img width="1028" height="775" alt="image" src="https://github.com/user-attachments/assets/3df3f15b-1e69-46a1-9b03-70e810fe2e81" />


---

## About The Project  
In an age of cloud breaches, remote malware, and software-based 2FA apps on connected smartphones, these measures are no longer sufficient for critical assets.  

**Project AEGIS** is a dedicated hardware token based on a **"Zero Trust"** philosophy.  
It does not have a USB data connection to the internet, Bluetooth, or Wi-Fi.  
It keeps your cryptographic texts separate from the network, ensuring that only someone with the physical device and a password can create your 2FA codes.



---

## Key features device comes with -
**Air-Gapped Security:** Zero wireless connectivity. Bad actors cannot hack what they cannot reach.
**Dual-Core Isolation:** The security functions run on a different core as the interface for users.
Hardware Root of Trust: TOTP secrets reside inside two tamper-resistant cryptographic chips - ATECC608B, not in the flash memory of the microcontroller.
**High accuracy MEMS RTC:** DS3231MZ module for precision timekeeping, maintains time for years without syncing.
A physical slide switch: This completely cuts off the battery and is known as a "hardware kill switch."
It comes with an oled 128x64 display and a rotery encoder dial for selecting accounts.

---
Hardware Design



The main part of the device i\I made is the **Seeed Studio XIAO ESP32-S3**,it have dual-core power with a small Size so that device is small enough



The architecture was designed to minimize attack surfaces.1. XIAO ESP32-S3



- **Why:** This has Dual-core Xtensa LX7 processor running at 240MHz. BLE and WiFi radios are strictly disabled by us in this firmware.- **Purpose:**  It drives the display, coordinates user input, and queries secure elements.



2.Dual ATECC608B Crypto Chips for more protection against attacks -

Why: Private keys are stored in hardware-protected slots with extra protection by us with epoxy resin. TOTP seeds cannot be extracted from firmware dumps.

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


## Key Circuit Details
- **Bus Isolation:** Crypto #1 is sharing "Noisy Bus" (D4/D5) with OLED and RTC; Crypto #2 on "Quiet Bus" (D0/D1) alone.
Input Ladder: 4 buttons wired into a resistor ladder network to read all 4 via a single Analog Pin (D7).
- **Noise Filtering:** There is a 100nF decoupling capacitor next to the power pin on each IC.
---

## PCB Layout
Front Layer (UI): OLED, rotary encoder, tactile buttons, Xiao.
Back Layer (Security Core): crypto chips, RTC, battery connections, Coin cell.   
  
> <img width="477" height="401" alt="image" src="https://github.com/user-attachments/assets/74dfb036-3e95-450f-a924-d0c8df2675ea" />
 

---

Firmware
Custom firmware in **C++ Arduino**, optimized for security.

Structure of Code
- **Secure Boot:** Initializes I2C buses, verifies crypto chips, and halts if missing (tampering detected).
- Dual-core tasking
Core 0: SHA1 hashing and secure element communication.
Core 1: UI, animations, input polling for smooth dial operation. - **TOTP Engine:** RFC 6238 implementation: reads RTC epoch, hashes with the secret using HMAC-SHA1, and truncates to the 6 digit code.
