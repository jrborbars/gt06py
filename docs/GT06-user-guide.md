# GT06 User Guide

## **Precautions**

- This project only provides the GT06 protocol client functionality. The server side needs to interface with a third-party provided server or an open-source server built by yourself.
- This project only provides basic functional interfaces for the GT06 protocol. Specific business logic needs to be developed separately.
- This project only implements basic messages of the GT06 protocol. Message functions defined by specific service platforms require secondary development. The data format for downlink data from the server is provided in the definition documentation of the service platform.
- The GT06 protocol requires storage and retransmission of data that fails to send. This project does not perform failed data storage; storage and retransmission of failed data must be implemented at the business layer.
- The heartbeat starts automatically after a successful login; there is no need to start the heartbeat manually.
- Downlink data from the server is received and processed through a callback function registered via the `set_callback` function.

## Functional Interaction Diagram

![](./media/gt06.png)

## Usage Instructions

### 1. Functional Module Import and Initialization

> **Replace `ip`, `port`, and `domain` according to your actual service environment.**

```python
from usr.gt06 import GT06

ip = "127.0.0.1"
port = 8888
domain = None
timeout = 5
retry_count = 3
life_time = 180

gt06_obj = GT06(
    ip=ip, port=port, domain=domain, timeout=timeout,
    retry_count=retry_count, life_time=life_time
)
```

### 2. Setting the Callback Function

> - Handles business data issued by the server. The data format must be processed according to the rules defined by the interfaced service platform.
> - Note that some server instructions may contain `unicode` encoded data. Currently, QuecPython does not support decoding this; it supports instructions in `utf-8` encoding format.

```python
def test_callback(args):
    protocol_no = args["protocol_no"]
    msg_no = args["msg_no"]
    content = args["content"]
    if protocol_no == 0x80:
        server_flag = content.get("server_flag")
        cmd_data = content.get("cmd_data")
        # TODO: Business logic processing and message response, example below:
        if cmd_data == "DYD,000000#":
            device_cmd = "DYD=Success!"
            global gt06_obj
            gt06_obj.report_device_cmd(server_flag, device_cmd)

set_callback_res = gt06_obj.set_callback(test_callback)
```

### 3. Setting Device Status Information

> This step is used for sending heartbeat information. It can also be sent by manually calling the device information reporting interface (Step 8).

```python
defend = 1
acc = 1
charge = 0
alarm = 1
gps = 1
power = 0
voltage_level = 5
gsm_signal = 4
set_device_status_res = gt06_obj.set_device_status(
    defend, acc, charge, alarm, gps, power, voltage_level, gsm_signal
)
```

### 4. Connecting to the Server

> The connection function will retry three times. If all three retries fail, it returns `False` and starts a timer. The device will automatically restart after 20 minutes. During this period, the user can repeatedly call this interface. If the connection is successful, the restart timer task will be canceled.

```python
connect_res = gt06_obj.connect()
```

### 6. Device Login

> Login using the device's IMEI number as the unique ID. The device's IMEI must be registered on the server side beforehand for a successful login.

```python
import modem

imei = modem.getDevImei()
login_res = gt06_obj.login(imei)
```

### 7. Reporting Device Location Information

> This interface can synchronously report device status information. To do so, first perform Step 3, then simply set the `include_device_status` parameter to `True`.

```python
import net
import utime

# Assuming now_time is defined elsewhere
date_time = ("{:0>2d}" * 6).format(*now_time[:6])[2:]
satellite_num = 12
latitude = 31.824845156501
longitude = 117.24091089413
speed = 120
course = 126
lat_ns = 1
lon_ew = 0
gps_onoff = 1
is_real_time = 1
mcc = net.getServingMcc()
mnc = net.getServingMnc()
lac = net.getServingLac()
cell_id = net.getServingCi()
include_device_status = False

gt06_obj.report_location(
    date_time, satellite_num, latitude, longitude, speed,
    course, lat_ns, lon_ew, gps_onoff, is_real_time,
    mcc, mnc, lac, cell_id, include_device_status
)
```

### 8. Reporting Device Status Information

> - This message also serves as a heartbeat message and is automatically sent every three minutes. It is recommended that users periodically call Step 3 to update device status or create a trigger function to update when the status changes.
> - This interface can also be called manually to report device status. However, before reporting, ensure whether the device status has changed; it is recommended to call Step 3 to update the status before reporting.

```python
gt06_obj.report_device_status()
```
