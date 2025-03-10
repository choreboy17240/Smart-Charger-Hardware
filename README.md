# Battery On-Board Charger

## Overview

This repository contains the design files for a Battery On-Board Charger, a DC-DC converter
designed to safely charge sealed lead-acid (SLA) batteries from external power sources. The 
system operates as a buck-boost converter with intelligent microcontroller control for charge
management.

Please see the accompanying software repository on GitHub for an example Arduino STM32 
application that can be built under PlatformIO and used to program the on-board STM32 micro
using an ST-Link programmer.  Programming is done using a custom"SWD-Lite" interface that I 
use for compact, low-cost projects.  SWD-Lite uses a standard 1x4 2.54mm male header to 
provide the absolute minimum number of signals needed or the ST-Link programmer to operate.

I'm sharing this design information and the associated software in hopes that others might 
find it interesting or helpful. It's a hobby project for me, so your mileage may vary!

This is my first attempt to share a KiCad (v9.0) project, so I'd be interested to hear
about whether there are issues or not.  I've exported the global custom symbols and footprints
that I used in the design, so hopefully this should be a standalone PWB design.

## Hardware Specifications

#### Battery Charging
- **Input Voltage**: 12-24V DC
- **Output Voltage**: Adjustable (typically 12V-15.5V for lead-acid batteries)
- **Maximum Charge Current**: 500-600mA
- **Controller**: STM32G030K8T6 (64KB Flash, 8KB RAM)
- **Power Stage**: XL6008 DC-DC converter (Buck-Boost topology)
- **Current Sensing**: INA219 I2C sensor
- **Voltage Control**: MCP4726A0 I2C DAC
- **Status Indicator**: RGB LED
- **Interfaces**: UART, I2C, SWD programming port
- **Operating Temperature**: Room temp

#### Battery Undervoltage Protection 
- **Low-Voltage Cutout**: Adjustable limit (trim pot)

## Key Features

- Software-adjustable charge voltage via I2C DAC
- Real-time current and voltage monitoring
- Temperature-compensated charging profiles
- Multi-stage charging (bulk, absorption, float)
- Status indication via RGB LED
- UART interface for diagnostics and monitoring
- Surge and reverse polarity protection
- Thermal management and over-current protection

## Power Conversion Design

The design utilizes an XL6008 DC-DC converter configured in a buck-boost topology:
- Input filtering through hybrid aluminum capacitors
- High-efficiency power inductors (47µH Sunlord MWSA1004S)
- Schottky diode rectification (B360AE)
- High-quality output filtering with polymer capacitors
- Feedback network with DAC-controlled reference voltage

## PCB Design Considerations

- 2-layer PCB with optimized (kinda) thermal design
- Ground plane for noise reduction and thermal dissipation
- Separate analog and digital ground regions
- Thermal management for XL6008 regulator
- EMI/EMC design practices for switching converter
- Power and signal path optimization

## Undervoltage Protection Circuit

The design incorporates a hardware-based undervoltage protection circuit
to prevent battery damage while under load:

- **Components**:
  - LM339 Quad Comparator for voltage threshold detection
  - MMBT3906 PNP transistor (Q1) for relay driver
  - NS_SRS_SPDT relay (K1) for power disconnection
  - Precision resistor divider network for voltage sensing

- **Operation**:
  - Continuously monitors the battery voltage through a resistive divider
  - When battery voltage falls below a preset threshold (adjustable via the
    RV1 trim pot)
  - Comparator triggers, driving the PNP transistor into conduction
  - Relay energizes, disconnecting the load from the battery
  - Hysteresis provided by R11 (1MΩ) prevents oscillation at threshold
  - D4 provides flyback protection for relay coil

- **Benefits**:
  - Prevents deep discharge of batteries
  - Hardware-based protection operates independently of microcontroller
  - Adjustable threshold for different battery types
  - Fast response time for critical undervoltage conditions
  - Failsafe design ensures load disconnection on power loss

The circuit provides essential protection for battery-powered systems, preventing
damage from excessive discharge while allowing normal operation above the safe 
voltage threshold.

## Thermal Performance

The design has been thermally optimized for enclosed operation:
- Maximum regulator temperature: ~52°C (125°F) at 600mA continuous charging
- Thermal steady state reached after ~30 minutes of operation
- Heat-generating components spaced for optimal thermal dissipation
- Current limited to 600mA for reliability in enclosed environments
- Charging current is constrained by the small heat sink are available for the 
  XL6008 converter, driven by the small packaging space availabe in my project.
- Control software limits the maximum current to that range automatically, but
  this is configurable.
- Redesign the PWB to increase the heat sink area or add a glue-on heat sink
  to the XL6008 if you want higher charging current.

## Usage Notes

1. Battery is connected through an SPDT switch with a center-off position.
1. Connect 12-24V power supply to the input terminals
2. Connect battery to the output terminals
3. The system automatically determines charge state and applies appropriate charging profile
4. RGB LED indicates charging status:
   - Red: Fast bulk charging
   - Yellow: Absorption charging
   - Green: Float/maintenance charging or standbu
   - Blinking: Error condition

## Software Configuration

The STM32G030 microcontroller requires appropriate firmware to control the charging process:
- Voltage setpoint control via MCP4726A0 DAC
- Current and voltage monitoring via INA219
- Temperature compensation
- Charging state machine
- Safety monitoring

See the companion firmware repository for microcontroller code.

## Limitations

- Maximum continuous charging current limited to 600mA for thermal management
- Not suitable for high-current applications without additional cooling
- Designed for lead-acid batteries (AGM, Gel, Flooded)
- Requires appropriate firmware for operation

## License

MIT open-source license - see the LICENSE file for details.

## Revision History

- 1.0: Initial design released to JLCPCB for prototype boards
- 1.1: Minor updates for silkscreen fixes

## Contact

Email: choreboy17240@gmail.com
