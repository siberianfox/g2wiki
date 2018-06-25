
### gQuintic Specs
- Runs [g2core](https://github.com/synthetos/g2) codebase

- Processor
  - Atmel SAMS70N19A ARM Cortex M7 processor
  - 300MHz internal clock
  - 32 and 64 bit single instruction floating point hardware
  - Multi-level cache, instruction pipeline, simultaneous instruction execution
  - All communications IO is DMA driven
  - USB is on-chip
  - We have seen 10x to 80x performance improvements relative to 84Mhz M3 core

- Host and Communications
  - USB-B connector for driving from "standard" external hosts
  - Host expansion connector for Linux host carrier board
    - Board design accommodates Linux daugterboards 
      - e.g. Beaglebone, rPi and variants, etc.
    - Host-to-board communications via 4 wire UART - RX/TX/RTS/CTS
    - 5volt @ 1500 ma available for host processor

- Motors
  - 5x Trinamic TMC2130 stepper motor drivers
  - 5x STEP / DIRECTION / ENABLE breakouts for external drivers (in lieu of Trinamic)  
    - selectable 3.3v/5v output levels
  - 4x "hobby servo" PWM outputs (uses the buffered digital outputs)
    - can be configured to interpret Gcode directly, or run as PWM devices

- Inputs
  - 10 protected 3.3v digital inputs  

- Outputs
  - 4 analog to digital inputs 
    - Straight ADC with 0.0v to 3.2v range
    - Use external adapter board for thermistor, PT100 or load cell
    - Analog inputs can be configured for thermistor / PT100 / PT1000 on-board for OEM volumes 
  - 4 buffered digital outputs - selectable 3.3v/5v level
  - 4 low-power FET digital outputs - up to 300 ma each (fans, etc.)
  - 3 high-power FET digital outputs - 2 extruders @ 6A, 1 heated bed @ 20A)
  - Neopixel LED array output with DMA firmware driver

- Other
  - SPI and I2C expansion port
  - Watchdog safety circuit to disable powered outputs
  - Can remove motor/output power w/o killing logic power
  - JTAG/SWD connection and real-time debugging using Atmel-ICE
  - ROHS
