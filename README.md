# SNMP Modbus Gateway Documentation

Documentation and supporting screenshots for the Tibbo SNMP Modbus Gateway.

The gateway polls Modbus RTU devices over RS-485 and exposes their values through
SNMP. It can also write supported SNMP values back to Modbus registers.

## Documentation

See the [Product Guide](product_guide.md) for:

- Hardware connections and network requirements
- Device Console configuration
- Modbus-to-SNMP mapping
- SNMPGET, SNMPSET, and SNMPWALK examples
- Commissioning and troubleshooting

## Import the AppBlocks Application

The repository includes `SNMP_Modbus_Gateway.json`, which contains the complete
AppBlocks application. Import this file into AppBlocks to open and work with the
full gateway application project.

## Supported Platform

- Firmware: `SNMP_Modbus_Gateway` v1.0.2
- Hardware: Tibbo TPP2W(G2) in a TPB2L(G2) enclosure
- Field interface: RS-485 / Modbus RTU
- Management interface: Ethernet / SNMPv2c

## Repository Contents

- `product_guide.md` — Complete configuration and commissioning guide.
- `SNMP_Modbus_Gateway.json` — Complete AppBlocks application for import.
- `assets/` — Screenshots used by the guide.
- `task_docs/` — Documentation tasks and notes.

## Important

Changes to Modbus mappings and device settings are stored immediately, but require
a device restart before becoming active.
