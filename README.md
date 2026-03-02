# Quectel-RM520N-GL-5G-HAT-setup
This guide explains how to set up the RM520N-GL 5G Sub-6GHz modem (M.2 B-Key) with Raspberry Pi 5 for high-speed 5G connectivity.
This setup enables:
- 5G / 4G / 3G connectivity
- GNSS positioning
- High-speed USB 3.1 communication
- Dual SIM support
- Stable external power supply operation

## 🧰 Hardware Requirements
- Raspberry Pi 5
- RM520N-GL 5G M.2 module
- M.2 to 5G HAT carrier board
- 5G SIM card (ours was configured to use private 5G network)
- 4x 5G antennas (mandatory)
- Passive GNSS antenna (optional for GPS)
- USB 3.0 dual-plug cable
- External 5V 3A power supply (MANDATORY for stability)

**⚠️ Important:** The 5G module consumes high current during network registration.

Always power the HAT using the EXT PWR (5V 3A) port and switch to EXT PWR mode.

## 🔌 Hardware Connection
1. Insert the RM520N-GL module into the M.2 B-Key slot.
2. Insert SIM card into SIM1 slot.
3. Connect all 4 antennas.
4. Connect USB 3.0 cable from HAT → Raspberry Pi 5.
5. Connect external 5V power supply to HAT Type-C port.
6. Turn ON the power switch.
#### LED Indicators
🔴 PWR (Red): Module powered

🟢 NET (Green): Network registered

## 💻 Software Environment
Tested on:
- Raspberry Pi OS (64-bit, bookworm-legacy)
- Ubuntu 22.04
- Kernel 5.15+
No additional driver installation required on latest Linux kernels.

#### Private 5G setup:
For 5G cellular network we used our own private 5G setup using Open5GS core and srsRAN. We used Opencells programmable SIM card and USRP B210 as base station. 

To learn more about it, follow the [link]()

## 🔍 Verify Module Detection
After connecting hardware:
```bash
lsusb
ls /dev/ttyUSB*
```
Expected interfaces:
```bash
/dev/ttyUSB0  → Diagnostic
/dev/ttyUSB1  → GNSS NMEA
/dev/ttyUSB2  → AT Command Port
/dev/ttyUSB3  → Modem
```
If not detected:
- Check antennas
- Check SIM
- Reboot system
- Verify external power

## Configuration:
There are two ways to configure the modem. 
1. ECM(Ethernet Control Model) Mode
2. QMI(Qualcomm MSM Interface) Mode

### Which to choose?
#### ECM (Ethernet Control Model)
ECM makes your 5G modem appear like a normal Ethernet network card.

To Linux, it looks like:
```bash
usb0 → Ethernet device
```

You use:
```bash
sudo dhclient usb0
```
No complex modem handling needed.

Internally it works as:
```bash
Modem → Converts cellular data → Wraps it as Ethernet frames → Sends via USB
```

#### QMI (Qualcomm MSM Interface)
QMI is a low-level modem communication protocol developed by Qualcomm.

Instead of pretending to be Ethernet, it:
- Exposes a control channel
- Exposes a data channel
- Requires a special driver (qmi_wwan)
- Uses libqmi tools to manage connection

You typically use:
```bash
qmicli
```

Internally it works as:
```bash
OS ↔ QMI driver ↔ Modem firmware ↔ Cellular network
```
It gives more direct control over modem behavior.

**Note:** If you are using a private 5G network and want to be able to communicate with your modem directly via your host machine (that has Core setup) or you don't want the main IP to NAT it is better to use QMI Mode. 

But for simpler and easy setup choose ECM mode as it is more hassle free
