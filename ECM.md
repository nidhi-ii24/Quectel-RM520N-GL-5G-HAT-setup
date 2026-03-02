# 📡 Configure 5G Network (ECM Mode – Recommended for Linux)
### Step 1 – Install Minicom
```bash
sudo apt install minicom
```
### Step 2 – Remove Conflicting Services
```bash
sudo apt purge modemmanager -y
sudo apt purge network-manager -y
```
**Note:** The official documentation first performs step 2, then 1 but that removes your network interfaces, hence not allowing you to install packages.

#### Test communication:
Minicom uses AT commands for communication. 

To use minicom run:
```bash
sudo minicom -D /dev/ttyUSB2
```
Now run the following command:

```bash
AT
OK
```
### Step 3 – Set USB Network Mode (ECM)
```bash
AT+QCFG="usbnet",1
AT+CGDCONT=1,"IPV4V6","YOUR_APN"
AT+CFUN=1,1
```
Replace YOUR_APN with your SIM provider APN.

Wait 30 seconds for reboot.

### Step 4 – Acquire IP Address
Check interface:
```bash
ip addr
```
You should see usb0.

Obtain IP:
```bash
sudo dhclient usb0
```
### Step 5 – Test Internet
```bash
ping google.com -I usb0
```
If successful, 5G is connected
