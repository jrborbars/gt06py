# GT06 API Interface Documentation

## Introduction

> This document details the functional interfaces and usage methods of the terminal GT06 protocol. This module only implements the basic interface functions of the GT06 protocol. In actual development, secondary development is required based on the service platform being interfaced with, supplementing custom messages and message items specific to the service platform. Different service platforms will have their own definition rules; please refer to the development documentation provided by the platform for secondary development.

## Functional Interfaces

### GT06 Module Import

```python
from usr.gt06 import GT06

ip = "220.180.239.212"
port = 7611
domain = None
timeout = 5
retry_count = 3
life_time = 180

gt06_obj = GT06(ip=ip, port=port, domain=domain, timeout=timeout, retry_count=retry_count, life_time=life_time)
```

Parameters:

| Parameter | Type | Description |
| :--- | --- | --- |
| ip | str | Server IP address, default is None. Choose either ip or domain. |
| port | int | Server port number, default is None. |
| domain | str | Server domain name, default is None. Choose either domain or ip. |
| timeout | int | Timeout for reading message data, default is 5 seconds. |
| retry_count | int | Number of retries for server connection failure, default is 3 times. |
| life_time | int | Heartbeat transmission cycle, default is 180s. |

### set_callback

> - Sets the callback function used to receive message instructions issued by the server.
> - In the GT06 protocol, only when `protocol_no` is `0x80` does the server send instructions to the terminal. When a message with this protocol number is received, the `report_device_cmd` interface must be used to respond.
> - Depending on the service platform, there will be different message definitions. Similarly, any message issued by the server can be received, processed, and responded to via the callback function.

Parameters:

| Parameter | Type | Description |
| :--- | --- | --- |
| callback | function | Callback function. The callback function has one parameter `args`, which is a dictionary with 3 keys: `protocol_no`, `msg_no`, and `content`. See `Callback Function Parameter Description` for details. |

Callback Function Parameter Description:

| Parameter | Type | Description |
| :--- | --- | --- |
| protocol_no | int | Protocol number, default is `0x80`. |
| msg_no | int | Server message sequence number. |
| content | dict | Instruction information issued by the server. `server_flag` (int) - Server flag bit, `cmd_data` (str) - Instruction content. |

Return Value:

| Data Type | Description |
| :--- | --- |
| BOOL | `True` for success, `False` for failure. |

Example:

```python
def test_callback(args):
    protocol_no = args["protocol_no"]
    msg_no = args["msg_no"]
    content = args["content"]

gt06_obj.set_callback(test_callback)
# True
```

### set_device_status

> Sets the device status. This function is used to report device status information messages. The set status information will be stored. Since this message also serves as a heartbeat message, this method should be called promptly to update the device status information whenever device status parameters change.

Parameters:

| Parameter | Type | Description |
| :--- | --- | --- |
| defend | int | Whether armed: 1 - Yes, 0 - No. |
| acc | int | ACC status: 1 - High, 0 - Low. |
| charge | int | Whether charging: 1 - Connected to power and charging, 0 - Not connected to power. |
| alarm | int | Alarm information: 0 - Normal, 1 - Vibration alarm, 2 - Power failure alarm, 3 - Low battery alarm, 4 - SOS distress. |
| gps | int | Whether GPS is positioned: 1 - Yes, 0 - No. |
| power | int | Oil/Electricity status: 1 - Disconnected, 0 - Connected. |
| voltage_level | int | Voltage level: <br>0 - No power (shutdown), <br>1 - Extremely low power (insufficient for calls/SMS), <br>2 - Very low power (low battery alarm), <br>3 - Low power (normal use), <br>4 - Medium power, <br>5 - High power, <br>6 - Extremely high power. |
| gsm_signal | int | GSM signal strength level: <br>0 - No signal, <br>1 - Extremely weak signal, <br>2 - Weak signal, <br>3 - Good signal, <br>4 - Strong signal. |

Return Value:

| Data Type | Description |
| :--- | --- |
| BOOL | `True` for success, `False` for failure. |

Example:

```python
defend = 1
acc = 1
charge = 0
alarm = 1
gps = 1
power = 0
voltage_level = 5
gsm_signal = 4
gt06_obj.set_device_status(defend, acc, charge, alarm, gps, power, voltage_level, gsm_signal)
# True
```

### connect

> Connects to the server.

Parameters:

None

Return Value:

| Data Type | Description |
| :--- | --- |
| BOOL | `True` for success, `False` for failure. |

Example:

```python
gt06_obj.connect()
# True
```

### disconnect

> Disconnects from the server.

Parameters:

None

Return Value:

| Data Type | Description |
| :--- | --- |
| BOOL | `True` for success, `False` for failure. |

Example:

```python
gt06_obj.disconnect()
# True
```

### login

> Device login.

Parameters:

| Parameter | Type | Description |
| :--- | --- | --- |
| imei | str | Device IMEI number. |

Return Value:

| Data Type | Description |
| :--- | --- |
| BOOL | `True` for success, `False` for failure. |

Example:

```python
import modem

imei = modem.getDevImei()
gt06_obj.login(imei)
# True
```

### report_location

> Reports GPS & LBS positioning information and device status information. When device status information needs to be uploaded synchronously, ensure whether the device information has changed. If there are changes, call the `set_device_status` interface to update the device status information before calling this interface for reporting.

Parameters:

| Parameter | Type | Description |
| :--- | --- | --- |
| date_time | str | Time, data format: `YYMMDDHHmmss`. Example: `220707164353`. |
| satellite_num | int | Number of satellites, range: [0:15]. |
| latitude | float | Latitude, unit: degrees. |
| longitude | float | Longitude, unit: degrees. |
| speed | int | Speed, unit: km/h. |
| course | int | Course, unit: degrees. Range: [0:360), 0 degrees is true north. |
| lat_ns | int | Latitude orientation: 0 - South latitude, 1 - North latitude. |
| lon_ew | int | Longitude orientation: 0 - East longitude, 1 - West longitude. |
| gps_onoff | int | Whether GPS is positioned: 0 - Not positioned, 1 - Positioned. |
| is_real_time | int | Real-time/Differential GPS: 0 - Real-time GPS, 1 - Differential GPS. |
| mcc | int | Mobile Country Code. |
| mnc | int | Mobile Network Code. |
| lac | int | Location Area Code. |
| cell_id | int | Cell Tower ID. Range: [0x000000:0xFFFFFF]. |
| include_device_status | bool | Whether to upload device status synchronously: True - Yes, False - No, default is False. |

Return Value:

| Data Type | Description |
| :--- | --- |
| BOOL | `True` for success, `False` for failure. |

Example:

```python
import net
import utime

# Assuming now_time is defined elsewhere, e.g., utime.localtime()
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
    date_time, satellite_num, latitude, longitude, speed, course, lat_ns, lon_ew, gps_onoff, is_real_time,
    mcc, mnc, lac, cell_id, include_device_status
)
# True
```

### report_device_status

> - Reports device status information. This interface also serves as heartbeat information, sent every three minutes.
> - This interface is used in conjunction with the `set_device_status` interface. The `set_device_status` interface must be called to update device status information before calling this one.

Parameters:

None

Return Value:

| Data Type | Description |
| :--- | --- |
| BOOL | `True` for success, `False` for failure. |

Example:

```python
gt06_obj.report_device_status()
# True
```

### report_device_cmd

> Response interface for instructions issued by the server. This interface is for responding to instruction messages issued by the server. Specific instruction data rules are defined by the server.

Parameters:

| Parameter | Type | Description |
| :--- | --- | --- |
| server_flag | int | Server flag bit, obtained from callback function parameters. |
| cmd_data | str | Instruction execution result information from the device side. |

Return Value:

| Data Type | Description |
| :--- | --- |
| BOOL | `True` for success, `False` for failure. |

Example:

```python
server_flag = 12345
cmd_data = "DYD=Success!"
gt06_obj.report_device_cmd(server_flag, cmd_data)
# True
```
