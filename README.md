# Bluetooth-Based Secure Locker with Access Logging

## Project Overview

The **Bluetooth-Based Secure Locker with Access Logging** is an embedded security system developed using the **LPC2148 ARM7 microcontroller** and **Embedded C**.

The locker uses two-level authentication:

1. A **4-digit Level-1 password** is received from an Android phone through an **HC-05 Bluetooth module** using UART1.
2. After successful Level-1 authentication, the user enters a **4-digit Level-2 password** using a **4x4 keypad**.
3. Only when both passwords are correct does the system operate the DC motor through an **L293D H-bridge** to unlock the locker.

The system also provides RTC-based access logging, tamper detection, buzzer alerts, and an administrator menu for changing passwords and RTC settings.

## Main Features

- Two-level password authentication
- Level-1 password through Bluetooth
- Level-2 password through keypad
- Password storage in external **AT24C256 EEPROM**
- 16x2 LCD user interface
- RTC date and time
- Timestamped access/event logging through UART0
- Tamper detection using an active-LOW switch
- Buzzer alert for unauthorized access/tamper events
- DC motor control through L293D
- Automatic locker opening and closing
- Administrator menu using **EINT2 external interrupt**
- RTC setting from the admin menu
- Level-1 and Level-2 password modification
- Bluetooth stale-command watchdog for incomplete commands

## Hardware Requirements

- LPC2148 ARM7 Microcontroller
- 16x2 LCD
- 4x4 Matrix Keypad
- AT24C256 EEPROM
- HC-05 Bluetooth Module
- DC Motor
- L293D Motor Driver
- Buzzer
- Tamper Switch
- Admin Push Button
- 12 MHz Crystal

## Software Requirements

- Embedded C
- Keil C / Keil µVision
- Flash Magic

## Pin Connections

### LCD - 4-bit Mode

| LCD Signal | LPC2148 Pin |
|---|---|
| RS | P0.16 |
| EN | P0.17 |
| D4 | P0.18 |
| D5 | P0.19 |
| D6 | P0.20 |
| D7 | P0.21 |

### UART0 - PC / Access Log

| Signal | LPC2148 Pin |
|---|---|
| TXD0 | P0.0 |
| RXD0 | P0.1 |

UART0 is used for monitoring and timestamped audit logs.

### I2C0 - 24C256 EEPROM

| Signal | LPC2148 Pin |
|---|---|
| SCL0 | P0.2 |
| SDA0 | P0.3 |
| EEPROM VCC | 3V3 |

### Tamper Switch

| Signal | LPC2148 Pin |
|---|---|
| Tamper input | P0.4 |

The tamper switch is **active LOW**.

### Admin Push Button

| Signal | LPC2148 Pin |
|---|---|
| EINT2 / Admin button | P0.7 |

The button is configured for a falling-edge EINT2 interrupt.

### UART1 - HC-05 Bluetooth

| Signal | LPC2148 Pin |
|---|---|
| MCU TXD1 → HC-05 RXD | P0.8 |
| MCU RXD1 ← HC-05 TXD | P0.9 |

### 4x4 Keypad

| Keypad | LPC2148 Pin |
|---|---|
| Rows | P1.16 - P1.19 |
| Columns | P1.20 - P1.23 |

### DC Motor - L293D

| Signal | LPC2148 Pin |
|---|---|
| IN1 | P1.24 |
| IN2 | P1.25 |

### Buzzer

| Signal | LPC2148 Pin |
|---|---|
| Buzzer signal | P1.26 |

## Project Software Structure

The uploaded Keil project contains these source and header files:

```text
Secure_Locker_Major_Project/
│
├── main.c
│
├── lcd.c
├── lcd.h
├── lcd_defines.h
│
├── keypad.c
├── keypad.h
│
├── uart.c
├── uart.h
│
├── bluetooth.c
├── bluetooth.h
│
├── eeprom.c
├── eeprom.h
│
├── rtc.c
├── rtc.h
│
├── motor.c
├── motor.h
│
├── buzzer.c
├── buzzer.h
│
├── security.c
├── security.h
│
├── menu.c
├── menu.h
│
├── delay.c
├── delay.h
│
├── defines.h
├── types.h
│
├── Startup.s
└── locker_project.uvproj
```

## Module Description

| Module | Purpose |
|---|---|
| `main.c` | Main application flow and integration of all modules |
| `lcd.c / lcd.h` | 16x2 LCD driver in 4-bit mode |
| `lcd_defines.h` | LCD pin definitions |
| `keypad.c / keypad.h` | 4x4 keypad scanning and key input |
| `uart.c / uart.h` | UART0 and UART1 communication |
| `bluetooth.c / bluetooth.h` | HC-05 Bluetooth command handling |
| `eeprom.c / eeprom.h` | I2C and 24C256 EEPROM read/write operations |
| `rtc.c / rtc.h` | RTC initialization, time/date handling and RTC persistence |
| `motor.c / motor.h` | DC motor forward, reverse and stop control |
| `buzzer.c / buzzer.h` | Buzzer control and alert patterns |
| `security.c / security.h` | Tamper detection, event logging and default password initialization |
| `menu.c / menu.h` | Administrator menu, RTC editing and password editing |
| `delay.c / delay.h` | Delay functions |
| `defines.h` | Project-wide constants and configuration |
| `types.h` | User-defined data types |
| `Startup.s` | ARM startup code |
| `locker_project.uvproj` | Keil project file |

## System Working

### Normal Authentication Flow

```text
Power ON
   ↓
System Initialization
   ↓
Restore RTC / Initialize Passwords
   ↓
Waiting for Bluetooth Password
   ↓
Receive Level-1 Password through HC-05
   ↓
Compare with Password in EEPROM
   ↓
 ┌───────────────┐
 │ Correct?      │
 └───────┬───────┘
     Yes │ No
         │
         ↓
   Enter Level-2
   Password
   using Keypad
         ↓
   Compare with
   EEPROM Password
         ↓
 ┌───────────────┐
 │ Correct?      │
 └───────┬───────┘
     Yes │ No
         │
         ↓
   Access Granted
         ↓
 Motor Forward
         ↓
 Locker Opens
         ↓
 Access Period
         ↓
 Motor Reverse
         ↓
 Locker Closes
         ↓
 Return to Idle / RTC Display
```

If either password is incorrect, access is denied and the buzzer is activated.

## Bluetooth Password Format

The Level-1 password is a **4-digit password**.

The project expects the Bluetooth command to be terminated with `#`.

Example:

```text
1234#
```

The default Level-1 password in the project source is:

```text
1234
```

The default Level-2 keypad password is:

```text
5678
```

These factory defaults are populated into EEPROM on the first boot when the EEPROM is considered uninitialized.

These default credentials are intended for first-boot initialization only. Both passwords should be changed through the Administrator Menu before the locker is used in any real-world deployment.

## EEPROM

The external **AT24C256 EEPROM** is used for non-volatile storage.

The project uses EEPROM for:

- Level-1 Bluetooth password
- Level-2 keypad password
- EEPROM initialization marker
- Saved RTC date/time/day values

The password memory locations defined in `defines.h` are:

```text
EEPROM_L1_ADDR = 0x0010
EEPROM_L2_ADDR = 0x0020
```

## RTC and Access Logging

The LPC2148 RTC provides date and time information for event logging.

Important system events are timestamped and transmitted through **UART0**, allowing a PC/terminal to monitor the locker activity.

Examples of logged events include:

- System boot
- Bluetooth authentication
- Successful authentication
- Failed password attempts
- Locker opening
- Locker closing
- Tamper detection
- Administrator actions
- RTC changes

## Tamper Detection

The tamper switch is connected to **P0.4** and is active LOW.

When tampering is detected:

- The system displays a tamper alert.
- The buzzer is activated.
- The event is recorded in the audit log.
- Normal password authentication is blocked while the tamper condition remains active.
- The administrator interrupt can still be serviced.

## Administrator Menu

The admin push button is connected to **P0.7** and uses **EINT2**.

Pressing the administrator button interrupts normal operation and opens the configuration menu.

The current menu provides:

```text
1 = RTC
2 = Password
3 = Set
```

### RTC Settings

The administrator can modify:

- Hour
- Minute
- Second
- Date
- Month
- Year
- Day of week

The updated RTC values are also saved to EEPROM.

### Password Settings

The administrator can update either:

- Level-1 Bluetooth password
- Level-2 keypad password

The password change process requires:

```text
Old Password
      ↓
New Password
      ↓
Confirm New Password
```

The new password is stored in EEPROM so it remains available after reset.

## Motor Control

The DC motor is controlled through an **L293D H-bridge**.

The project uses:

```text
P1.24 → IN1
P1.25 → IN2
```

The configured motor timing is:

```text
MOTOR_ROTATE_MS = 500 ms
MOTOR_SETTLE_MS = 200 ms
```

The motor:

1. Rotates forward to open the locker.
2. Stops while the locker remains open.
3. Waits for the configured access period.
4. Rotates in reverse to close the locker.
5. Stops after closing.

The motor timing is hardware-dependent and may need tuning for the actual mechanical locker.

## Clock Configuration

The project configures the LPC2148 for:

```text
Crystal frequency : 12 MHz
Core clock (CCLK) : 60 MHz
Peripheral clock   : 15 MHz
```

UART communication is initialized at:

```text
9600 baud
```

## Bluetooth Command Watchdog

The Bluetooth input buffer uses a `#` terminator.

If digits are received but the `#` terminator is not received within the configured timeout, the incomplete command is automatically cleared.

Configured timeout:

```text
BT_PENDING_TIMEOUT_MS = 3000 ms
```

This prevents an incomplete previous command from being combined with a later Bluetooth password.

## Build and Run

1. Open `locker_project.uvproj` in Keil µVision.
2. Make sure the LPC2148 device/project configuration is selected.
3. Build the project.
4. Generate the required output file such as the HEX file.
5. Program the LPC2148 using the appropriate programming method/Flash Magic.
6. Connect the required hardware modules according to the pin configuration.
7. Open a UART0 terminal on the monitoring PC to observe access logs.
8. Use a Bluetooth serial Android application (e.g., **Arduino BlueControl** or any generic Bluetooth Terminal app) to send the Level-1 password to the HC-05 module.

## Known Limitations

The current implementation has the following limitations:

- Factory-default passwords are hardcoded in source and should be changed before deployment.
- EEPROM writes for passwords and RTC values are not wear-leveled.
- The system does not detect or recover from power loss occurring mid-motor-cycle.
- Bluetooth authentication relies on standard HC-05 pairing and is not encrypted beyond Bluetooth Classic security.
- The system supports a single administrator level; multi-user or role-based access is not implemented.

## Future Enhancements

Planned or possible improvements for future versions of this project include:

- EEPROM wear-leveling or rotating storage for frequently written values.
- Power-loss and motor-state recovery using EEPROM state flags.
- Migration of Bluetooth authentication to an encrypted channel such as BLE with pairing keys.
- Support for multiple user profiles with individual access logs.
- A dedicated mobile app interface in place of raw serial command input for Level-1 authentication.

## Repository Contents

The repository should contain the actual source code, header files, startup file, and Keil project files.

Generated build artifacts such as the following do not need to be included in the source-code repository:

```text
*.o
*.d
*.crf
*.lst
*.axf
*.map
*.hex
*.bak
*.plg
*.tra
*.lnp
*.sct
*.htm
```

## Project Information

**Project:** Bluetooth-Based Secure Locker with Access Logging

**Microcontroller:** LPC2148 ARM7TDMI-S

**Programming:** Embedded C

**IDE:** Keil µVision / Keil C

**Bluetooth:** HC-05

**EEPROM:** AT24C256

**Display:** 16x2 LCD

**Keypad:** 4x4 Matrix Keypad

**Motor Driver:** L293D

**Logging:** UART0 + RTC

## Author

Mohini Gomare

Embedded Systems Project - LPC2148 ARM7 Microcontroller.
