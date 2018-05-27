
## gQuintic Specs
- Runs [g2core](https://github.com/synthetos/g2) codebase

- Processor
  - Atmel SAMS70N19A ARM Cortex M7 processor
  - 300MHz internal clock
  - 32 and 64 bit single instruction floating point hardware
  - Multi-level cache, instruction pipeline, simultaneous instruction execution
  - All communications IO is DMA driven
  - USB is on-chip (w/custom USB drivers - field tested for about 2 years now)

- Host and Communications
  - USB-B connector for driving from "standard" external hosts
  - Host expansion connector for Linux host carrier board
    - Board design accommodates wide variety of linux hosts such as nanoPi family, Beaglebone, rPi, etc.
    - Host-to-board communications via 4 wire UART - RX/TX/RTS/CTS
    - 1.2 Amps 5v @ 1200 ma available for host processor

- Motors
  - 5x Trinamic TMC2130 stepper motor drivers
    - Will handle up to 1.7 amps;
    - Good for almost all NEMA17 motors, will do some NEMA23s
  - 5x STEP / DIRECTION / ENABLE breakouts for external drivers (in lieu of Trinamics)  
    - selectable 3.3v/5v output levels
  - 4x "hobby servo" PWM outputs (uses the buffered digital outputs)
    - can be configured to interpret Gcode directly, or run as PWM devices

- Inputs
  - 10 input protected 3.3v digital inputs  
  - 4x analog to digital inputs 
    - configured as straight-through 0v-3.2v ADC channels
    - external analog front-end boards can be used to run 100K thermistors, PT100, PT1000 RTDs
    - OEM boards can be configured on-board for these front-ends

- Outputs
  - 4x buffered digital outputs - selectable 3.3v/5v level
  - 4x low-power FET digital outputs - up to 300 ma each (fans, etc.)
  - 2x high-power 6amp FET digital outputs - for extruders
  - 1x high-power 20A FET digital outputs - for heated bed
  - 0v-10v DAC output for spindle controller 
  - Neopixel LED array output with DMA firmware driver

- Other
  - Can be configured for separate 24v motor and logic input to support eStop that cuts motor power while leaving controller board and host live to handle whatever hit the fan.
  - SPI and I2C expansion port
  - Watchdog safety circuit to disable powered outputs
  - Can remove motor/output power w/o killing logic power
  - JTAG/SWD connection and real-time debugging using Atmel-ICE
  - ROHS

### Watchdog Safety Circuit
Most product safety certifications (e.g. CE for Europe) for things like machine tools require that a redundant hardware failsafe be provided to shut down the machine in various failure cases. This should be in hardware, unless you have a software solution that is separate form the main SW, and the failsafe SW itself then needs to be certified (which we want to avoid).

The gQuintic has a discrete multivibrator circuit made of two cockroach FETs (BSS84, BSS138) that asserts as long as a pulse train is present on the input. If the pulse train is not present or fails the safety circuit de-asserts. All the output FETs and the step signal are gated by this SafeOut signal. The pulse train generator is in the main loop of the MCU. 

Here are conditions where the pulse train would fail:
* The MCU is booting - and has not started the main loop yet
* The MCU is being flashed (and therefore not executing)
* Motor power is applied but for some reason logic power has failed or not applied (the MCU is therefore not functioning)
* The firmware is off in the weeds and the main loop has been terminated accordingly (Note: there are a series of assertions that are also run as part of the main loop that will shut down the machine if a fatal error is detected)

### External Shutdown Options

THe 