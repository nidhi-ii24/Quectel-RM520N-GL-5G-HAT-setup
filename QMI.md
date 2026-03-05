# RM520N-GL 5G HAT – QMI Mode Setup Guide (Raspberry Pi)

This guide explains how to configure the RM520N-GL 5G modem in QMI mode on a Raspberry Pi and establish a working 5G data connection.

QMI (Qualcomm MSM Interface) provides a direct communication interface with Qualcomm modem firmware, allowing efficient high-speed networking through the qmi_wwan Linux driver.

### 1. Verify Device Detection
After connecting the modem to the Raspberry Pi, check if the USB device is detected.
```bash
lsusb
```bash
You should see a Quectel modem device.

Next check serial ports:
```bash
ls /dev/ttyUSB*
```

Expected output:
```bash
/dev/ttyUSB0
/dev/ttyUSB1
/dev/ttyUSB2
/dev/ttyUSB3
```
Typical Usage:
|Port|Purpose|
|----|-------|
|/dev/ttyUSB0|Diagnostic Port|
|/dev/ttyUSB1|GNSS NMEA output|
|/dev/ttyUSB2|AT command port|
|/dev/ttyUSB3|Modem interface|

### 2. Install Minicom for AT Commands
```bash
sudo apt-get install minicom
```

### 3. Remove Conflicting Services
Some services interfere with modem control.
```bash
sudo apt purge modemmanager -y
sudo apt purge network-manager -y
```

### 4. Enter Minicom for Testing
```bash
sudo minicom -D /dev/ttyUSB2
```
Test communication:
```bash
AT
OK
```

### 5. Verify SIM and Network Status
Run the following AT commands.

#### Check SIM status:
```bash
AT+CPIN?
```
Expected response:
```bash
+CPIN: READY
```
#### Check SIM slot
```bash
AT+QUIMSLOT?
```
Response:
```bash
1
```

#### Check LTE Registration
```bash
AT+CEREG?
```
Expected output: (since we are connecting to 5G Network and not LTE network, must be 0,1 in case of 4G)
```bash
0,0
```

#### Check 5G Registration
```bash
AT+C5GREG?
```
Expected output:
```bash
0,1
```

#### Get Network Information
```bash
AT+QNWINFO
```
Expected output:
```bash
+QNWINFO: "NR5G-SA","<PLMN>","NR5G BAND n78",<bandwidth>
```

### 6. Configure APN
#### Check current APN configuration:
```bash
AT+CGDCONT?
```
#### Set APN:
```bash
AT+CGDCONT=1,"IP","internet"
```
#### Expected Output:
```bash
+CGDCONT: 1,"IP","internet"
```
#### Activate PDP context:
```bash
AT+CGACT=1,1
```
#### Verify IP assignment:
```bash
AT+CGCONTRDP
```
#### Example response:
```bash
+CGCONTRDP: 1,5,"internet","10.45.0.2","255.255.255.255","10.45.0.1"
```

### 7. Enable QMI Mode
Switch modem USB networking mode.
```bash
AT+QCFG="usbnet",0
```
Where:
|Mode|Meaning|
|----|-------|
|0| QMI mode|
|1| ECM mode|

Restart modem:
```bash
AT+CFUN=1,1
```
Wait about 30 seconds for reboot.

### 8. Verify Driver
Check USB driver tree:
```bash
lsusb -t
```
You must see:
```bash
qmi_wwan
```

### 9. Verify Network Registration
Check modem serving system.
```bash
sudo qmicli -d /dev/cdc-wdm0 --nas-get-serving-system
```
Expected output should include:
- Registered state
- PLMN
- Network type

Example:
```bash
Registration state: 'registered'
Radio interface: '5GNR'
```

### 10. Fix Transaction Timeout (If Occurs)
If the command times out, reset USB authorization.
```bash
sudo sh -c 'echo 0 > /sys/bus/usb/devices/usb1/authorized'
sleep 3
sudo sh -c 'echo 1 > /sys/bus/usb/devices/usb1/authorized'
```
Retry:
```bash
sudo qmicli -d /dev/cdc-wdm0 --nas-get-serving-system
```

### 11. Start Network Session
#### Create a QMI client.
```bash
sudo qmicli -d -p /dev/cdc-wdm0 --wds-noop --client-no-release-cid
```
Example output:
```bash
CID: 15
```
Remember the **CID number**.
#### Start the network session:
```bash
sudo qmicli -d -p /dev/cdc-wdm0 --wds-start-network="apn=internet,ip-type=4" --client-cid=15 --client-no-release-cid
```
Expected output:
```bash
Network started
```

### 12. Retrieve Network Settings
```bash
sudo qmicli -d -p /dev/cdc-wdm0 --wds-get-current-settings --client-cid=15 --client-no-release-cid
```
Information returned includes:
- IP address
- Gateway
- DNS
- Subnet mask

### 13. Configure Network Inteface
Check available interfaces.
```bash
ip link
```
You should see:
```bash
wwan0
```

#### Enable raw IP mode
```bash
sudo ip link set wwan0 down
echo Y | sudo tee /sys/class/net/wwan0/qmi/raw_ip
sudo ip link set wwan0 up
```
Verify:
```bash
cat /sys/class/net/wwan0/qmi/raw_ip
```
Expected:
```bash
Y
```

### 14.Obtain IP Address
```bash
sudo udhcpc -i wwan0
```
Expected:
```bash
udhcpc: broadcasting discover
udhcpc: lease obtained
```
Check routes:
```bash
ip route
```
You should see routing via **wwan0**.

### 15. Test Connectivity
Ping test:
```bash
ping -I wwan0 8.8.8.8
```
If successful, the **5G connection is active**.

### 16. Routing Configuration (Core Machine)
Check tunnel interface:
```bash
ip link show ogstun
```
Expected:
```bash
<UP, LOWER_UP>
```
Add route:
```bash
sudo ip route add 10.45.0.0/16 via 10.45.0.1
```
Enable IP forwarding:
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```
Allow SSH forwarding:
```bash
sudo iptables -A FORWARD -d 10.45.0.0/16 -p tcp --dport 22 -j ACCEPT
```

### 17. Access Raspberry Pi from Core Machine
From the core machine:
```bash
ssh <raspi-ip>
```
This confirms end-to-end connectivity through the 5G modem.

