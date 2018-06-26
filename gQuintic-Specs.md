# gQuintic Specs
- Runs [g2core](https://github.com/synthetos/g2) codebase

- Processor
  - Atmel SAMS70N19A ARM Cortex M7 processor
  - 300MHz internal clock
  - 32 and 64 bit single instruction floating point hardware
  - Multi-level cache, instruction pipeline, simultaneous instruction execution
  - All communications IO is DMA driven
  - We have seen 10x to 80x performance improvements relative to 84Mhz M3 core

- Host and Communications
  - USB-B connector for driving from "standard" external hosts
  - USB is on-chip (w/custom USB drivers - field tested for about 2 years now)
  - Host expansion connector for Linux host carrier board
    - Board design accommodates Linux daughterboards 
      - e.g. Beaglebone, rPi and variants, etc.
    - Host-to-board communications via 4 wire UART - RX/TX/RTS/CTS
    - 5volt @ 1500 ma available for host processor

- Motors
  - 5 motor outouts - may be a mix of:
    - 5 Trinamic TMC2130 stepper motor drivers
      - Will handle up to 1.7 amps;
      - Good for almost all NEMA17 motors, will do some NEMA23s
    - 5 STEP / DIRECTION / ENABLE breakouts for external drivers  
      - selectable 3.3v/5v output levels
  - 4 "hobby servo" PWM outputs (uses the buffered digital outputs)
    - can be configured to interpret Gcode directly, or run as PWM devices

- Digital Inputs
  - 10 digital inputs
    - ESD protected & RC deglitched
    - light pullup to 3.3v
    - 3.3v and ground available on each connector  

- Analog Inputs
  - 4 ADC inputs (analog to digital)
    - configured as straight-through 0.0v - 3.3v ADC channels
    - use external RTD front-end boards to run 100K thermistors, PT100, PT1000
    - use external load cell front-end boards to terminate standard 1mv/V load cells  
    - OEM boards can be configured on-board for RTD termination

- Digital Outputs
  - 4 buffered digital outputs - jumper selectable for 3.3v/5v level
  - 4 low-power FET digital outputs - up to 300 ma each (for fans, etc.)
  - 3 high-power FET digital outputs 
    - 2 extruders @ 6A
    - 1 heated bed @ 20A
  - Neopixel LED array output with DMA firmware driver
  - All digital output channels can be PWM driven

- Analog Outputs
  - 0-10 volt digital to analog converter for running variable frequency drive spindles
    - uses one of the 4 low-power FETs channels for the VFD DAC

- Other
  - Configured for daughterboard to carry Linux host computer
    - Supply up to 5v @ 1500ma to off-board host
    - Host connector provides
       - 4-wire UART RX/TX/RTS/CTS
       - Safety signal from board (board-is-sane assertion signal, as well as sanity pulse-train signal)
       - RESET and ERASE lines drivable from host
       - Host interrupt line driven from controller MCU
       - Logic 3.3v and ground 
  - SPI and I2C expansion port
  - Watchdog safety circuit to disable powered outputs
     - Safety signal de-assert causes the following
       - Stop STEP signals from reaching on-board and off-board drivers
       - (Does not change state of driver ENABLES, as this could be a safety issue)
       - Shut down all FETs, including extruder, heated bed, and low-power FETs
     - Safety signal de-asserts in the following conditions
       - Board encounters alarm, shutdown or panic conditions (settable as compiler options)
       - Board is erased or reset
       - Board firmware fault (main loop stops operating)
  - Support for external ESTOP
    - Can remove 24v motor/heater/FET power w/o killing logic power
    - Can remove 24v motor power w/o damaging on-board drivers
    - Power removed is detectable by CPU - can be used to initiate shutdown sequence
  - JTAG/SWD connection and real-time debugging using Atmel-ICE
  - ROHS

## Features

### Watchdog Safety Circuit
Most product safety certifications (e.g. CE for Europe) for things like machine tools require that a redundant hardware failsafe be provided to shut down the machine in various failure cases. This should be in hardware, unless you have a software solution that is separate from the main SW, and the failsafe SW itself then needs to be certified (which we want to avoid).

gQuintic has a discrete multivibrator circuit made of two cockroach FETs (BSS84, BSS138) that asserts as long as a pulse train is present on the input. If the pulse train is not present or fails the safety circuit de-asserts. All the output FETs, the Step signals and the DAC voltage are gated by this SafeOut signal. The pulse train generator is in the main loop of the MCU. 

Here are conditions where SafeOut would de-assert form hardware:
* The MCU is booting - and has not started the main loop yet
* The MCU is being flashed (and therefore not executing)
* Motor power is applied but for some reason logic power has failed or not applied (the MCU is therefore not functioning)

In addition, there are some software conditions that remove the pulse train and de-assert SafeOut:
* The firmware is off in the weeds and the main loop has been terminated accordingly 
* A failsafe assertion in the code has triggered, placing the firmware into a PANIC or ALARM state
* An external shutdown has been detected placing the system into a SHUTDOWN state (see Emergency Stop, below)

### Emergency Stop and External Shutdown Options
Background and philosophy: Since the controller may be the cause of a problem requiring an emergency stop, the controller should never be in the critical path of an emergency stop.

With that in mind, the board is design so that motor and FET power can be killed by an external switch independently of the processor continuing to operate. The incoming Vmot (typically 24v) has two connectors - One high-current to power the motors and FETs, another low-current to feed the regulator stages. These can be bridged for single-connection configurations.

If Vmot drops a number of things happen:
* The motors, FETs and DAC de-power - this includes the high-current FETS (heaters), and the low-current FETs and the DAC output will drop to zero (The DAC is 0-10v, typically used for spindle speed)
* The Trinamic rail voltages drop in order to prevent them from blowing up (which they will if you don't remove power)
* The CPU receives a LO on a pin, which can be used to signal the firmware that an external Estop has occurred (really that Vmot has been lost). The firmware (as it stands now) enters a Shutdown sequence, where it stops motion, shuts off the Safety signal, and informs the host that this has occurred.
