
---
> **Firmware:** `SNMP_Modbus_Gateway` v1.0.2  
> **Platform:** Tibbo TPP2W(G2) in a TPB2L(G2) enclosure

<br>

---

## Bookmarks

- [1. Overview](#1-overview)
- [2. Hardware](#2-hardware)
- [3. Network configuration and services](#3-network-configuration-and-services)
- [4. Device Console](#4-device-console)
  - [4.1 Database](#41-database)
  - [Column reference](#column-reference)
  - [4.2 Settings](#42-settings)
    - [4.2.1 Info](#421-info)
    - [4.2.2 Config](#422-config)
    - [4.2.3 Network](#423-network)
    - [4.2.4 SNMP](#424-snmp)
    - [4.2.4.1 Scalar Instance Object-ID](#4241-scalar-instance-object-id)
  - [4.3 System](#43-system)
    - [4.3.1 Reboot Device](#431-reboot-device)
    - [4.3.2 Initialize Settings](#432-initialize-settings)
- [5. Deployment Example](#5-deployment-example)
  - [5.1 Connect the Modbus BP#02 to the Gateway](#51-connect-the-modbus-bp02-to-the-gateway)
  - [5.2 Connect the Network Cable and Power Adapter](#52-connect-the-network-cable-and-power-adapter)
  - [5.3 Connect to the Device Console](#53-connect-to-the-device-console)
  - [5.4 Configure Settings](#54-configure-settings)
    - [5.4.1 Polling Interval](#541-polling-interval)
    - [5.4.2 Network](#542-network)
    - [5.4.3 SNMP](#543-snmp)
  - [5.5 Configure Modbus Mapping](#55-configure-modbus-mapping-in-table_devices_input)
  - [5.6 Restart Device](#56-restart-device)
  - [5.7 Verify Modbus Polled Data in `TABLE_DEVICES_OUTPUT`](#57-verify-modbus-polled-data-in-table_devices_output)
  - [5.8 Verify Modbus Polled Data via SNMP](#58-verify-modbus-polled-data-via-snmp)
  - [5.9 Write Data to Modbus](#59-write-data-to-modbus)
    - [5.9.1 Read Current Value](#591-read-current-value)
    - [5.9.2 Write New Value](#592-write-new-value)
    - [5.9.3 Verify SNMP Write Response](#593-verify-snmp-write-response)
    - [5.9.4 Verify Written Data via SNMPGET](#594-verify-written-data-via-snmpget)
    - [5.9.5 Verify Written Data in the Device Console](#595-verify-written-data-in-the-device-console)
  - [5.10 Change Settings via SNMP](#510-change-settings-via-snmp)
    - [5.10.1 Review the Setting in `TABLE_SETTINGS`](#5101-review-the-setting-in-table_settings)
    - [5.10.2 Write Network Settings Using SNMP](#5102-write-network-settings-using-snmp)
      - [5.10.2.1 Disable DHCP](#51021-disable-dhcp)
      - [5.10.2.2 Set the IP Address](#51022-set-the-ip-address)
      - [5.10.2.3 Set the Subnet Mask](#51023-set-the-subnet-mask)
      - [5.10.2.4 Set the Gateway Address](#51024-set-the-gateway-address)
    - [5.10.3 Verify Written Settings via SNMPGET](#5103-verify-written-settings-via-snmpget)
    - [5.10.4 Verify Written Settings in the Device Console](#5104-verify-written-settings-in-the-device-console)
    - [5.10.5 Reboot Device via SNMP](#5105-reboot-device-via-snmp)
      - [5.10.5.1 Send 1st Reboot Signal](#51051-send-1st-reboot-signal)
      - [5.10.5.2 Verify 1st Reboot Signal](#51052-verify-1st-reboot-signal)
      - [5.10.5.3 Send 2nd Reboot Signal within 10 Seconds](#51053-send-2nd-reboot-signal-within-10-seconds)
- [6. Commissioning checklist](#6-commissioning-checklist)
- [7. Troubleshooting](#7-troubleshooting)

<br><br>

---

<a id="1-overview"></a>
## 1. Overview [↑](#bookmarks "Back to Bookmarks")

The SNMP–Modbus Gateway lets **SNMP monitoring software read** values from
**Modbus equipment** over **Ethernet**. It is used when equipment such as meters, sensors, 
controllers, or other industrial devices communicates with Modbus, while the
customer's monitoring system uses SNMP.

The gateway regularly reads the Modbus values selected by the user and makes
them available as SNMP values.
A monitoring system can then display the values, record them, or use them to raise alarms.

Where specifically configured and authorized, the gateway can also accept an
**SNMP write** command and send the corresponding Modbus write to the field device.
This **remote-control function** should be enabled only for approved control points.

![Modbus field device to SNMP monitoring-system gateway flow](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_flow.png)

The user configures which Modbus values to read, and how each value appears to SNMP software.
This means the same gateway can be used with different types of
Modbus equipment instead of being limited to a single sensor or meter.

<br><br>

---

<a id="2-hardware"></a>
## 2. Hardware [↑](#bookmarks "Back to Bookmarks")

| Item | Detail |
| --- | --- |
| Controller | Tibbo TPP2W(G2) Tibbo Project PCB |
| Enclosure | TPB2L(G2), DIN-rail mounting kit, vibration-proof kit |
| Network | On-board 10/100 Base-T Ethernet with RJ45 |
| Field interface | Tibbit #05 RS-485 port for Modbus RTU |
| Field connector | Tibbit #21 (4-terminal block) or Tibbit #20 (9-terminal block) |
| Power | Tibbit #10 power supply module in socket 11 |
| Power adapter | APR-P0008 adapter |
| Storage | 1 MB AT45DB081E dataflash, LittleFS file system |
| Status LEDs | On-board red/green LEDs used for the setup-error indication |
| MD button | On-board setup (MD) button. Hold for 5 s and release to reset all settings to their defaults |
| RST button | On-board restart (RST) button |

<br><br>

---

<a id="3-network-configuration-and-services"></a>
## 3. Network configuration and services [↑](#bookmarks "Back to Bookmarks")

| Item | Setting |
| --- | --- |
| Ethernet | static IP/netmask/gateway, or DHCP  |
| SNMP | SNMPv2c and SNMPv3 agent, port 161, configurable read/write communities and SNMPv3 security credentials |
| Time sync | SNTP against 216.239.35.0 (time.google.com), applied with the configured time zone |
| Device Console | HTTP, port 80, 3 concurrent connections |

<br>

> **NOTE:**
> - Out of the box, the device uses the **default** static IP address
> `192.168.1.40`. This placeholder makes the device reachable on first
> power-up; change it to match your network.
> - A **network connection** is required for communication between the device
> and its configured SNTP servers, as well as authorized SNMP managers.

<br><br>

---

<a id="4-device-console"></a>
## 4. Device Console [↑](#bookmarks "Back to Bookmarks")

> The browser-based configuration interface is called the **Device Console**. It
> may also be referred to as the Web Console in AppBlocks documentation.

<br>

<a id="41-database"></a>
### 4.1 Database [↑](#bookmarks "Back to Bookmarks")

For every value you want to monitor, configure the gateway with the relevant Modbus device and register.
The gateway reads that register at the **configured polling interval** and makes the latest value available to
your SNMP monitoring system.

| Table | Storage | Entries | User-defined | After reboot | Purpose |
| --- | --- | --- | --- | --- | --- |
| `TABLE_DEVICES_INPUT` | Flash | Max. 30 | Yes | Retained | Modbus-to-SNMP mappings |
| `TABLE_DEVICES_OUTPUT` | RAM | Max. 30 | No | Rebuilt | Latest polling results |
| `TABLE_SETTINGS` | RAM | 22 predefined | No | Recreated | Device Console settings |

<br>

> **NOTE:**
> - Use `TABLE_DEVICES_INPUT` to define which Modbus values the gateway monitors.
> - `TABLE_DEVICES_OUTPUT` shows the latest polling results. It is not saved and is rebuilt after each restart.
> - `TABLE_SETTINGS` contains the settings used by the Device Console.
> - **Restart the device after changing `TABLE_DEVICES_INPUT` or a Setting**. Changes are saved immediately but take
>   effect only after restart.

<br>

<a id="column-reference"></a>
### Column reference [↑](#bookmarks "Back to Bookmarks")

The following table explains the columns shown in the Device Console database
tables.

| Column | Used by table | Meaning |
| --- | --- | --- |
| `NAME` | Input, Output, Settings | A descriptive name for the mapping or setting. |
| `DEVICE_ID` | Input, Output | The Modbus address of the field device. |
| `REG_ADDR` | Input, Output | The Modbus register address to read or write. |
| `FUNC_CODE` | Input | The Modbus function code used to access the register. |
| `OID_COLUMN` | Input, Output, Settings | The identifier used to associate the entry with its SNMP object. |
| `HASH_ID` | Output | An internal unique identifier for the polling result. |
| `VALUE` | Output | The latest value read from the Modbus device. |
| `INFO` | Settings | Additional information describing the Device Console setting. |

<br>

> **NOTE:**
> - **Register addressing and value interpretation:**
> Enter the register address (REG_ADDR) exactly as shown in the connected Modbus device's
> register map.
> - **Warning:** If the register address (REG_ADDR) is incorrect, the gateway may not receive a result
> when polling the Modbus device.

<br>

<a id="42-settings"></a>
### 4.2 Settings [↑](#bookmarks "Back to Bookmarks")

Use the Settings area to review device information and configure polling,
network, and SNMP access. 

> **Reminder:**
> - Restart the device after changing a setting.
> - The new value takes effect after restart.

<a id="421-info"></a>
#### 4.2.1 Info [↑](#bookmarks "Back to Bookmarks")

The `Info` group displays the device's restart counters and the current and maximum number of `TABLE_DEVICES_INPUT` records.

![Device Console Settings, Info group](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_info.png)

> **NOTE:**
>
> - The Reboot Counter can only be reset in the Device Console, not via SNMP.
> - The Reboot Rollover Counter records how many times the Reboot Counter has reached `65535` and rolled over to `1`. The total number of registered reboots is `Reboot Rollover Counter * 65535 + Reboot Counter`.

<a id="422-config"></a>
#### 4.2.2 Config [↑](#bookmarks "Back to Bookmarks")

Use the `Config` group to set the **Polling Interval** and **Debugprint**.

> **NOTE:** The supported `Polling Interval` range is 0.1 to 60 seconds.

![Device Console Settings, Config group](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_config.png)

<a id="423-network"></a>
#### 4.2.3 Network [↑](#bookmarks "Back to Bookmarks")

The Network group lets you enable DHCP or configure a static Ethernet IP
address, netmask, and gateway.

> **NOTE:**
> - When DHCP is enabled, the static Ethernet address fields are **Unavailable**.
> - DHCP may assign a different IP address and interrupt your Device Console connection.
> - To keep the device reachable at a known address, a **static** IP address is recommended.

![Device Console Settings, Network group](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_network.png)

<a id="424-snmp"></a>
#### 4.2.4 SNMP [↑](#bookmarks "Back to Bookmarks")

Use the `SNMP` group to configure how your monitoring system connects to the gateway.
You can set the SNMP port, read and write community strings, device description, system object ID, and SNMPv3 user and security settings.

![Device Console Settings, SNMP basic settings](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_snmp_basic_settings.png)

> **NOTE:**
> - **Read:** Use SNMP monitoring software to read gateway values with `snmpget` or `snmpwalk`.
> - **Write:** Authorized SNMP clients can configure **ALL** gateway SNMP objects with `snmpset`.
> - For SNMPv2c, an empty write community disables `snmpset` requests.

<br>

![Device Console Settings, SNMPv3 security settings](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_snmp_security_settings.png)

<a id="4241-scalar-instance-object-id"></a>
#### 4.2.4.1 Scalar Instance Object-ID [↑](#bookmarks "Back to Bookmarks")

Each mapped gateway value is available through a **full SNMP OID**.
Its composition is:

```text
<SysObjectID>.1.<OID_COLUMN>.0
```

> **NOTE:**
> - By default, `sysObjectID` is set to `1.3.6.1.4.1.20738`.
> - The `sysObjectID` may be extended with a child OID, such as `45.46`, to create `1.3.6.1.4.1.20738.45.46`.
> - The final `.0` is required when querying an individual value.
> - Each `OID_COLUMN` value must be **UNIQUE**.
> - To minimize mistakes, use the full OID returned by `snmpwalk`.

<br>

<a id="43-system"></a>
### 4.3 System [↑](#bookmarks "Back to Bookmarks")

The `System` area lets you restart the device or restore its default settings.

<a id="431-reboot-device"></a>
#### 4.3.1 Reboot Device [↑](#bookmarks "Back to Bookmarks")

Click **Reboot Device** to restart the device.

> Restart the device after changing `TABLE_DEVICES_INPUT` or Device Console
> settings to apply the new configuration.

<a id="432-initialize-settings"></a>
#### 4.3.2 Initialize Settings [↑](#bookmarks "Back to Bookmarks")

Click **Initialize Settings** to restore the device's default configuration.

> **MD-button:**
>
> You can also restore the default settings using the on-board **MD button**:
>
> - Press and hold the button for at least **5 seconds**, then release it.
> - Releasing the button *before* 5 seconds have elapsed has **no effect**.
>
> **NOTE:** Restart the device after initializing settings or using the MD
> button so that the restored settings take effect.

<br>

![Device Console, System area](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_system.png)


<br><br>

---

<a id="5-deployment-example"></a>
## 5. Deployment Example [↑](#bookmarks "Back to Bookmarks")

This example shows how to connect a Modbus device to the gateway, configure a
Modbus value for polling, and read it through an SNMP monitoring system using
`snmpwalk`.

It also demonstrates how an authorized SNMP client can use `snmpset` to write
a Modbus value and change a device setting.

<br>

<a id="51-connect-the-modbus-bp02-to-the-gateway"></a>
### 5.1 Connect the Modbus BP#02 to the Gateway [↑](#bookmarks "Back to Bookmarks")

The deployment steps use a Tibbo Modbus BP#02, a sensor for ambient temperature
and relative humidity. The BP#02 is only used as an example; you can connect
another compatible Modbus device instead.

> **IMPORTANT:** Before replacing a Modbus sensor or changing its wiring,
> disconnect power from the device. Reconnect power only after the new sensor
> and all wiring have been connected correctly.

Connect the BP#02 to the TPP2W(G2) as shown below. Connect each wire to the
terminal with the matching label: `VCC` to `VCC`, `GND` to `GND`, `Tx` to
`Tx`, and `Rx` to `Rx`. When using a different Modbus device, follow its
documentation for the required power and communication connections.

![Connecting the BP#02 sensor to the TPP2W(G2)](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/tpp2wg2_modbus.png)

> **TIP:** More Tibbo RS-485 Modbus sensor models are available [here](https://tibbo.com/store/rs485-modbus-sensors.html).

<br>

<a id="52-connect-the-network-cable-and-power-adapter"></a>
### 5.2 Connect the Network Cable and Power Adapter [↑](#bookmarks "Back to Bookmarks")

Connect an Ethernet cable from the device's RJ45 port to your local network.
Then connect the supplied power adapter to the device. The device starts when
power is connected. Allow up to 20 seconds for the device to fully boot and
become ready for use.

![Connecting the Ethernet cable and power adapter to the TPP2W(G2)](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/tpp2wg2_ethernet_power.png)

<a id="53-connect-to-the-device-console"></a>
### 5.3 Connect to the Device Console [↑](#bookmarks "Back to Bookmarks")

Using your preferred web browser, navigate to [http://192.168.1.40](http://192.168.1.40). The **Home** page should be displayed.

![Device Console, Home area](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_home.png)

<br>

<a id="54-configure-settings"></a>
### 5.4 Configure Settings [↑](#bookmarks "Back to Bookmarks")

Open **Settings** in the Device Console. Review the network, SNMP, and polling
settings required for your installation. Restart the device after making any
changes so that the new settings are applied.

> For an overview of the available settings, see [Section 4.2, Settings](#42-settings).

<a id="541-polling-interval"></a>
#### 5.4.1 Polling Interval [↑](#bookmarks "Back to Bookmarks")

Set the **Polling Interval** in the **Config** group to a value that suits your
application. This setting determines how often the gateway reads values from
the connected Modbus device.

<a id="542-network"></a>
#### 5.4.2 Network [↑](#bookmarks "Back to Bookmarks")

Choose the network method required by your installation in the **Network**
group:

- Enable **DHCP** if your network automatically assigns IP addresses.
- Leave **DHCP** disabled to use a static IP address, then enter the IP address,
  netmask, and gateway supplied by your network administrator.

> This guide uses `192.168.1.40` throughout the example.


After changing the network settings and restarting the device, reconnect to
the Device Console using its new IP address.

<a id="543-snmp"></a>
#### 5.4.3 SNMP [↑](#bookmarks "Back to Bookmarks")

Configure the SNMP port and access credentials required by your monitoring
system in the **SNMP** group. Configure either SNMPv2c community strings or
SNMPv3 user and security settings, as required by your organization.

Enable write access only when it is required for approved control points. Use
credentials and security settings approved by your organization.

> This guide uses the default SNMP settings throughout the example.

<br>

<a id="55-configure-modbus-mapping-in-table_devices_input"></a>
### 5.5 Configure Modbus Mapping in `TABLE_DEVICES_INPUT` [↑](#bookmarks "Back to Bookmarks")

Open **Database** in the Device Console. In `TABLE_DEVICES_INPUT`, add a
mapping for each Modbus value that you want to make available through SNMP.

> **NOTE:**
> - Enter the `DEVICE_ID`, `REG_ADDR`, and `FUNC_CODE` exactly as specified in
>   the connected field device's Modbus register map or datasheet.
> - Each `OID_COLUMN` value must be **UNIQUE**. For details, see
>   [Section 4.2.4.1, Scalar Instance Object-ID](#4241-scalar-instance-object-id).
> - `TABLE_DEVICES_INPUT` supports a maximum of **30** mappings.
> - `OID_COLUMN = 207` is used later in this guide to demonstrate **Write Data
>   to Modbus**.

<br>

![Device Console Database, TABLE_DEVICES_INPUT](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_database_table_devices_input.png)



<br>

<a id="56-restart-device"></a>
### 5.6 Restart Device [↑](#bookmarks "Back to Bookmarks")

Restart the device so that the new configuration is applied. Allow up to 20
seconds for the device to fully boot and become ready for use.


<br>

<a id="57-verify-modbus-polled-data-in-table_devices_output"></a>
### 5.7 Verify Modbus Polled Data in `TABLE_DEVICES_OUTPUT` [↑](#bookmarks "Back to Bookmarks")

In **Database**, select `TABLE_DEVICES_OUTPUT`. Confirm that the mapping
configured in `TABLE_DEVICES_INPUT` appears in this table and that its `VALUE`
field shows the expected value read from the Modbus device.

The `TABLE_DEVICES_OUTPUT` is a live view of the gateway's polling
results, updated at the configured polling interval.

> **NOTE:**
> - If the expected mapping or value is **not** shown, check the Modbus wiring.
> - Also check the `DEVICE_ID`, `REG_ADDR`, `FUNC_CODE`, and `OID_COLUMN` configured in
>   `TABLE_DEVICES_INPUT`.
> - You can ignore the `HASH_ID`, since it is an internal identifier used by
>   the gateway.
> - Click **Refresh** to update the displayed table data.

![Device Console Database, TABLE_DEVICES_OUTPUT](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_database_table_devices_output.png)



<br>

<a id="58-verify-modbus-polled-data-via-snmp"></a>
### 5.8 Verify Modbus Polled Data via SNMP [↑](#bookmarks "Back to Bookmarks")

This example uses a Linux computer and SNMPv2c to read the Modbus value
published by the gateway. The SNMP settings used below are configured in
[Section 4.2.4, SNMP](#424-snmp).

Use `snmpwalk` to view **ALL** SNMP values available under the gateway's
`SysObjectID`.

```text
snmpwalk -v2c -c <read-community> <device-ip> <SysObjectID>
```

For this example, use the configured read community, device IP address, and
`SysObjectID`:

```bash
snmpwalk -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46
```

![Linux terminal, SNMP walk example](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpwalk_example.png)

The command displays the available SNMP values and their full OIDs. Locate the
value with `OID_COLUMN` `401` in the command output.

Use `snmpget` to read one specific value. The full scalar OID has the following
format:

```text
<SysObjectID>.1.<OID_COLUMN>.0
```

Use the following command to read the value with `OID_COLUMN` `401`:

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.401.0
```

![Linux terminal, SNMP get example](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpget_example.png)

> **NOTE:**
> - Replace `public`, `192.168.1.40`, and `1.3.6.1.4.1.20738.45.46` when
>   the corresponding settings of your device are different.
> - SNMPv3 can also be used. Use the username, authentication settings, and
>   privacy settings configured in the Device Console. The general command
>   format is:
>
>   ```text
>   snmpwalk -v3 -l authPriv -u <username> -a <auth-protocol> \
>     -A <authentication-password> -x <privacy-protocol> \
>     -X <privacy-password> <device-ip> <SysObjectID>
>   ```
>
>   ```text
>   snmpget -v3 -l authPriv -u <username> -a <auth-protocol> \
>     -A <authentication-password> -x <privacy-protocol> \
>     -X <privacy-password> <device-ip> <SysObjectID>.1.<OID_COLUMN>.0
>   ```
>
> - Other SNMP tools can be used instead of the Linux command line. For
>   example, Windows users can use Paessler SNMP Tester.



<br>

<a id="59-write-data-to-modbus"></a>
### 5.9 Write Data to Modbus [↑](#bookmarks "Back to Bookmarks")

This section shows how an authorized SNMP client can write a value to a Modbus
register through the gateway. In this example, `OID_COLUMN 207` identifies the
configured Modbus write mapping. The SNMP settings used below are configured in
[Section 4.2.4, SNMP](#424-snmp).

> **NOTE:**
> - The SNMP write command sends the requested value to the connected Modbus
>   device.
> - Use this function only for approved control points and only when it is safe
>   to change the value on the field device.

<a id="591-read-current-value"></a>
#### 5.9.1 Read Current Value [↑](#bookmarks "Back to Bookmarks")

Before writing a new value, use `snmpget` to read the current value of the
mapping with `OID_COLUMN` set to `207`.

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.207.0
```

![Linux terminal, SNMP get OID 207 before write](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpget_oid_207_before_write_example.png)

> The current value will be used later to compare with the newly written value.

<a id="592-write-new-value"></a>
#### 5.9.2 Write New Value [↑](#bookmarks "Back to Bookmarks")

Use the following command to write the value `4546` to `OID_COLUMN` `207`:

```bash
snmpset -v2c -c private 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.207.0 i 4546
```

![Linux terminal, SNMP set OID 207](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpset_oid_207_write_example.png)

> The command sends the value to the connected Modbus device.

<a id="593-verify-snmp-write-response"></a>
#### 5.9.3 Verify SNMP Write Response [↑](#bookmarks "Back to Bookmarks")

The gateway also provides a confirmation of the most recent SNMP write through
`OID_COLUMN` `65534`. Run the following command immediately after the write:

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65534.0
```

![Linux terminal, SNMP get OID 65534 after write](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpget_oid_65534_after_write_example.png)

Confirm that the returned response shows `SUCCESS`.

> **NOTE:**
> - The SNMP write response shows whether the most recent `snmpset` command
>   succeeded (`SUCCESS`) or failed.
> - The response remains available for 30 seconds, then resets to `IDLE`.
> - If the most recent `snmpset` command **failed**, follow the hint shown in the
>   response and check the Modbus wiring and configuration before trying again.

<a id="594-verify-written-data-via-snmpget"></a>
#### 5.9.4 Verify Written Data via SNMPGET [↑](#bookmarks "Back to Bookmarks")

Use `snmpget` to confirm the written value through SNMP:

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.207.0
```

![Linux terminal, SNMP get OID 207 after write](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpget_oid_207_after_write_example.png)

Confirm that the returned value is `4546`.

<a id="595-verify-written-data-in-the-device-console"></a>
#### 5.9.5 Verify Written Data in the Device Console [↑](#bookmarks "Back to Bookmarks")

Wait for at least one polling interval after sending the `snmpset` command.
Then open **Database** in the Device Console and select
`TABLE_DEVICES_OUTPUT`. Confirm that the entry with `OID_COLUMN` `207` shows
the new `VALUE` of `4546`.

![Device Console Database, TABLE_DEVICES_OUTPUT OID 207](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_database_table_devices_output_oid_207.png)

> **NOTE:**
> - `TABLE_DEVICES_OUTPUT` is READ-ONLY and does not update automatically.
>   Click **Refresh** to update the displayed data.
> - Modbus writes take effect immediately. Restarting the device is therefore
>   NOT necessary.

<br>

<a id="510-change-settings-via-snmp"></a>
### 5.10 Change Settings via SNMP [↑](#bookmarks "Back to Bookmarks")

Authorized SNMP clients can change device settings. Each setting is identified
in `TABLE_SETTINGS` by its `OID_COLUMN`.

> **NOTE:**
> - Changing network or SNMP settings **can interrupt your connection** to the device.
> - A changed device setting is **NOT** applied until the device restarts.

<a id="5101-review-the-setting-in-table_settings"></a>
#### 5.10.1 Review the Setting in `TABLE_SETTINGS` [↑](#bookmarks "Back to Bookmarks")

In **Database**, select `TABLE_SETTINGS`. Locate the setting you want to change
and note its `OID_COLUMN`.

> **NOTE:**
> - `TABLE_SETTINGS` is read-only.
> - Device settings can be changed through the Device Console or SNMP.


<a id="5102-write-network-settings-using-snmp"></a>
#### 5.10.2 Write Network Settings Using SNMP [↑](#bookmarks "Back to Bookmarks")

Most installations use a static IP address that matches the local network. For
example, configure the device with the following settings:

- `NET_DHCP` to `disabled`
- `NET_IP` to `192.168.1.121`
- `NET_MASK` to `255.255.255.0`
- `NET_GATEWAY` to `192.168.1.1`

The marked rows below identify these settings and their `OID_COLUMN` values.

![Device Console Database, TABLE_SETTINGS network settings](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_database_table_settings_network.png)

Send the following commands one at a time. The commands use the device's
current IP address, `192.168.1.40`.

> After each `snmpset` command, run the response command within 30 seconds to confirm that the write succeeded.

<a id="51021-disable-dhcp"></a>
##### 5.10.2.1 Disable DHCP [↑](#bookmarks "Back to Bookmarks")

Disable DHCP:

```bash
snmpset -v2c -c private 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65503.0 s disabled
```

![Linux terminal, SNMP set NET_DHCP](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpset_netdhcp_example.png)

Immediately verify the response:

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65534.0
```

![Linux terminal, SNMP response after setting NET_DHCP](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpget_netdhcp_response_example.png)

Confirm that the returned response shows `SUCCESS`.

<a id="51022-set-the-ip-address"></a>
##### 5.10.2.2 Set the IP Address [↑](#bookmarks "Back to Bookmarks")

Set the device IP address:

```bash
snmpset -v2c -c private 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65505.0 s 192.168.1.121
```

![Linux terminal, SNMP set NET_IP](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpset_netip_example.png)

Immediately verify the response:

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65534.0
```

![Linux terminal, SNMP response after setting NET_IP](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpget_netip_response_example.png)

Confirm that the returned response shows `SUCCESS`.

<a id="51023-set-the-subnet-mask"></a>
##### 5.10.2.3 Set the Subnet Mask [↑](#bookmarks "Back to Bookmarks")

Set the subnet mask:

```bash
snmpset -v2c -c private 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65506.0 s 255.255.255.0
```

![Linux terminal, SNMP set NET_MASK](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpset_netmask_example.png)

Immediately verify the response:

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65534.0
```

![Linux terminal, SNMP response after setting NET_MASK](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpget_netmask_response_example.png)

Confirm that the returned response shows `SUCCESS`.

<a id="51024-set-the-gateway-address"></a>
##### 5.10.2.4 Set the Gateway Address [↑](#bookmarks "Back to Bookmarks")

Set the gateway address:

```bash
snmpset -v2c -c private 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65504.0 s 192.168.1.1
```

![Linux terminal, SNMP set NET_GATEWAY](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpset_netgateway_example.png)

Immediately verify the response:

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65534.0
```

![Linux terminal, SNMP response after setting NET_GATEWAY](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpget_netgateway_response_example.png)

Confirm that the returned response shows `SUCCESS`.

<a id="5103-verify-written-settings-via-snmpget"></a>
#### 5.10.3 Verify Written Settings via SNMPGET [↑](#bookmarks "Back to Bookmarks")

To verify that the settings have changed correctly, use the following commands:

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65503.0
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65505.0
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65506.0
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65504.0
```

> The current IP address `192.168.1.40` is used because the new address does
> **not** take effect until the device restarts.

The results of the above commands are shown here:

![Linux terminal, SNMP get network settings](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpget_network_example.png)

> The settings are stored but are not active until the device restarts.

<a id="5104-verify-written-settings-in-the-device-console"></a>
#### 5.10.4 Verify Written Settings in the Device Console [↑](#bookmarks "Back to Bookmarks")

In the Device Console:

1. Select **Settings**.
2. Open the **Network** tab.
3. Click **Refresh** to load the stored settings.
4. Confirm that Ethernet DHCP is **Disabled** and that the Ethernet IP,
   Netmask, and Gateway fields show the values written in
   [5.10.2](#5102-write-network-settings-using-snmp).

![Device Console Settings, Network settings written via SNMP](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_settings_network_example.png)

> The settings are stored but are not active until the device restarts.

<a id="5105-reboot-device-via-snmp"></a>
#### 5.10.5 Reboot Device via SNMP [↑](#bookmarks "Back to Bookmarks")

To apply the changed settings, send two reboot signals within 10 seconds. The
reboot setting is `SYS_REBOOT_DEVICE`, with OID column `65520`.

![Device Console Database, SYS_REBOOT_DEVICE setting](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_webconsole_database_table_settings_system_reboot.png)

<a id="51051-send-1st-reboot-signal"></a>
##### 5.10.5.1 Send 1st Reboot Signal [↑](#bookmarks "Back to Bookmarks")

Send the first reboot signal:

```bash
snmpset -v2c -c private 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65520.0 s true
```

![Linux terminal, SNMP set first reboot signal](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpset_reboot_device_example.png)

> The device does not reboot after this first signal. It starts a 10-second
> confirmation window for the second signal.

<a id="51052-verify-1st-reboot-signal"></a>
##### 5.10.5.2 Verify 1st Reboot Signal [↑](#bookmarks "Back to Bookmarks")

Immediately check the response and reboot setting:

```bash
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65534.0
snmpget -v2c -c public 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65520.0
```

Confirm that the response OID column `65534` reports `SUCCESS`, and that
`SYS_REBOOT_DEVICE` (OID column `65520`) reports `true (1/2)`.

![Linux terminal, SNMP get first reboot signal response](https://raw.githubusercontent.com/tibbotech/snmp_modbus_gateway_assets/refs/heads/main/assets/snmp_modbus_gateway_snmpget_reboot_device_example.png)

<a id="51053-send-2nd-reboot-signal-within-10-seconds"></a>
##### 5.10.5.3 Send 2nd Reboot Signal within 10 Seconds [↑](#bookmarks "Back to Bookmarks")

Within 10 seconds of the first signal, send the same command again:

```bash
snmpset -v2c -c private 192.168.1.40 1.3.6.1.4.1.20738.45.46.1.65520.0 s true
```

> **NOTE:**
> - The device now restarts and can stop responding before an SNMP response is received. 
> - Allow it to come back online before connecting to its new IP address.

<!-- TODO: Add a terminal screenshot of the second reboot `snmpset` command
     and the resulting connection loss or reboot. -->



<br><br>

--- 

<a id="6-commissioning-checklist"></a>
## 6. Commissioning checklist [↑](#bookmarks "Back to Bookmarks")

1. Install and check the RS-485 wiring.
2. Connect the Ethernet cable and power adapter. Wait for the device to boot.
3. Open the Device Console at the device's current IP address.
4. In **Device Console > Settings > Config**, check the polling interval and
   adjust it if needed.
5. In **Device Console > Settings > Network**, configure the Ethernet settings
   required by the local network.
6. In **Device Console > Settings > SNMP**, configure the port and access
   credentials required by the SNMP manager.
7. Restrict management access to authorized hosts.
8. Gather the Modbus register map, communication parameters, data format, and
   expected values.
9. In **Device Console > Database > TABLE_DEVICES_INPUT**, create the
   Modbus-to-SNMP mappings using this information.
10. Restart the device to apply the mappings and settings.
11. Confirm that the gateway polls the expected Modbus value.
12. Run `snmpwalk` on the enterprise branch, then use the returned full scalar
   OID with `snmpget`.
13. Where safe, change the physical input or simulated register. Wait for a
   Modbus poll and confirm that the SNMP value changes.
14. If enabled, test each write mapping safely. Check the SNMP response and
   verify the field-device register.
15. After changing settings in the Device Console or via SNMP, verify the
   stored values and restart the device.

> For an SNMP reboot, send two signals within 10 seconds. Confirm that the device comes back online.

<br><br>

---

<a id="7-troubleshooting"></a>
## 7. Troubleshooting [↑](#bookmarks "Back to Bookmarks")

| Symptom | Checks |
| --- | --- |
| SNMP responds but value is stale or zero | Verify the Modbus poll result, unit ID, function code, register addressing, serial settings, scaling, and byte/word order. An SNMP response alone does not prove the field link works. |
| SNMP GET returns `No Such Object` or `No Such Instance` | Walk the configured enterprise branch and query the full scalar OID, including its `.0` instance suffix. |
| No Modbus response | Check RS-485 polarity, termination/biasing, serial parameters, selected Tibbit/socket, unit ID, and whether the target actually supports the requested function/register. |
| `TABLE_DEVICES_OUTPUT` is empty | Add at least one mapping to `TABLE_DEVICES_INPUT`, restart the device, wait for a polling interval, then click **Refresh**. |
| Cannot add another Modbus mapping | `TABLE_DEVICES_INPUT` supports a maximum of 30 entries. Remove an unused mapping before adding another one. |
| Polling is too slow or does not start as expected | Check **Settings > Config > Polling Interval**. The supported range is 0.1 to 60 seconds. |
| SNMP SET fails | Confirm that the write community is correct and not empty, the object is read/write, the caller is authorized, the write mapping exists, and the field device accepts the function/value. |
| SNMP response OID `65534` returns `IDLE` | Query it within 30 seconds of the `snmpset` command; the response then resets to `IDLE`. |
| Device does not reboot through SNMP | Send `true` to `SYS_REBOOT_DEVICE` twice within 10 seconds. The first signal only starts the confirmation timer. |
| Default initialization did not take effect | Restart the device after **Initialize Settings** or an MD-button reset. |
| Value has the wrong magnitude | Check signedness, scale factor, register count, and byte/word order. |

