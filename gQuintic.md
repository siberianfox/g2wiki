
### gQuintic Specs
- Runs [g2core](https://github.com/synthetos/g2) codebase

- Processor
  - Atmel SAMS70N19A ARM Cortex M7 processor
  - 300MHz internal clock
  - 32 and 64 bit single instruction floating point hardware
  - Multi-level cache, instruction pipeline, simultaneous instruction execution
  - All communications IO is DMA driven
  - USB is on-chip

- Host and Communications
  - USB-B connector for driving from "standard" external hosts
  - Host expansion connector for Linux host carrier board
    - Board design accommodates Next Thing Co C.H.I.P.
    - Can also be used with Intel Edison, Beaglebone, rPi, etc.
    - Host-to-board communications via 4 wire UART - RX/TX/RTS/CTS
    - 1.2 Amps 5v @ 1200 ma available for host processor

- Motors
  - 5x Trinamic TMC2130 stepper motor drivers
  - 5x STEP / DIRECTION / ENABLE breakouts for external drivers (in lieu of Trinamic)  
    - selectable 3.3v/5v output levels
  - 4x "hobby servo" PWM outputs (uses the buffered digital outputs)
    - can be configured to interpret Gcode directly, or run as PWM devices

- Inputs
  - 12x protected 3.3v digital inputs  

- Outputs
  - 4x analog to digital inputs 
    - configured as thermistor inputs but can be setup as resistor-divided ADC inputs
  - 4x buffered digital outputs - selectable 3.3v/5v level
  - 4x low-power FET digital outputs - up to 500 ma each (fans, etc.)
  - 3x high-power FET digital outputs - 2x extruders @ 6A, 1x heated bed @ 20A)
  - Neopixel LED array output with DMA firmware driver

- Other
  - SPI and I2C expansion port
  - Watchdog safety circuit to disable powered outputs
  - Can remove motor/output power w/o killing logic power
  - JTAG/SWD connection and real-time debugging using Atmel-ICE
  - ROHS
