The SAM-ICE would not debug. It would go into debug mode, but it would keep running without stopping on the breakpoint. When stopped it was on an assembly instruction. Here's a sequence of events:

Environment / Initial conditions:
<pre>
Running AtmelStudio6.1-2440-beta
XP VMware image on OSX 10.8.2
XP is up-to date with latest .NET and XP patches
VM is running 2.5 Gb RAM
Segger J-Link V4.59e(beta) Compiled 11:12:12 on Jan 14 2013
Target is Arduino Due connected to SAM-ICE with Olimex JTAG adapter cable
</pre>

Shell manifest
<pre>
Atmel Studio 6 (Version: 6.1.2440 - beta)
© 2013 Atmel Corp.
All rights reserved.


OS Version: Microsoft Windows NT 5.1.2600 Service Pack 3
Platform: Win32NT


Installed Packages: Shell VSIX manifest - 6.1
Shell VSIX manifest
Version: 6.1
Package GUID: 5aa6ea3e-da7b-48c1-9b2a-cab2329d32ac
Company: Atmel Coproration


Installed Packages: Atmel ARM GNU Toolchain - 4.7.2.87
ARM Toolchain
Version: ARM_Toolchain_Version:4.7.2.87 GCC_VERSION:4.7.2
Package GUID: D83C9208-1D2D-4665-9760-EB9EE264CF8F
Company: Atmel
HelpUrl: 
Release Description: ARM Toolchain

CMSIS
Version: 3.0
Package GUID: D83C9208-1D2D-4665-9760-EB9EE264CF8F
Company: Atmel
HelpUrl: 
Release Description: ARM Support File Version



Installed Packages: AVR macro Assembler - 6.1.5
AVR Assembler
Version: 6.1.5
Package GUID: 03CB4AE1-80EA-40C7-B561-98CC87EA539C
Company: Atmel
HelpUrl: 
Release Description: AVR Assembler For 8-Bit Devices



Installed Packages: Atmel AVR (32 bit) GNU Toolchain - 3.4.2.365
AVR Toolchain 32
Version: AVR32_Toolchain_Version:3.4.2.365 GCC_VERSION:4.4.7
Package GUID: DB6D383F-C5D9-4E7E-BBF9-F37C6EEB59FD
Company: Atmel
HelpUrl: 
Release Description: AVR Toolchain For 32-Bit Devices



Installed Packages: Atmel AVR (8 bit) GNU Toolchain - 3.4.2.876
AVR Toolchain 8 Bit
Version: AVR8_Toolchain_Version:3.4.2.876 GCC_VERSION:4.7.2
Package GUID: 2C7AA7CF-94C6-463C-81DA-4AA03B613C3B
Company: Atmel
HelpUrl: 
Release Description: AVR Toolchain For 8-Bit Devices



Installed Packages: Atmel Gallery - 1.3
Atmel Gallery
Version: 1.3
Package GUID: AtmelStudioExtensionManager
Company: Atmel


Installed Packages: Atmel Kits - 1.1.104
Atmel Kits
Version: 1.1.104
Package GUID: bea809ab-462e-4535-99f1-3f9ced2f09ff
Company: Atmel


Installed Packages: Atmel Software Framework - 3.7.2.705
ASF
Version: 3.7.2
Package GUID: 519cc26f-02f6-4ace-8bf7-30c1cdea1f02
Company: Atmel
HelpUrl: http://asf.atmel.com/3.7.2
Release Description: ASF - 3.7.2 Release

ASF
Version: 3.6.0
Package GUID: 519cc26f-02f6-4ace-8bf7-30c1cdea1f02
Company: Atmel
HelpUrl: http://asf.atmel.com/3.6.0
Release Description: ASF - 3.6.0 Release

ASF
Version: 3.5.1
Package GUID: 519cc26f-02f6-4ace-8bf7-30c1cdea1f02
Company: Atmel
HelpUrl: http://asf.atmel.com/3.5.1
Release Description: ASF - 3.5.1 Release

ASF
Version: 3.5.0
Package GUID: 519cc26f-02f6-4ace-8bf7-30c1cdea1f02
Company: Atmel
HelpUrl: http://asf.atmel.com/3.5.0
Release Description: ASF - 3.5.0 Release

ASF
Version: 3.4.1
Package GUID: 519cc26f-02f6-4ace-8bf7-30c1cdea1f02
Company: Atmel
HelpUrl: http://asf.atmel.com/3.4.1
Release Description: ASF - 3.4.1 Release

ASF
Version: 3.3.0
Package GUID: 519cc26f-02f6-4ace-8bf7-30c1cdea1f02
Company: Atmel
HelpUrl: http://asf.atmel.com/3.3.0
Release Description: ASF - 3.3.0 Release



Installed Packages: AtmelToolchainProvider - 6.1.0.0
AtmelToolchainProvider
Version: 6.1.0.0
Package GUID: AtmelToolchainProvider.Atmel.83804b14-6626-4e13-bfdc-3a0135fa98f1
Company: Atmel


Installed Packages: Visual Assist X for Atmel Studio - 10.7.1920.0
Visual Assist X for Atmel Studio
Version: 10.7.1920.0
Package GUID: 7997A33C-B154-4b75-B2AC658CD58C9510
Company: Whole Tomato Software
</pre>

In Output window:
<pre>
06:30:40: [ERROR] Invalid context 1576_bp_00000000,, ModuleName: TCF (TCF command: Breakpoints:getProperties failed.)
06:30:40: [ERROR] Invalid context 1576_bp_00000001,, ModuleName: TCF (TCF command: Breakpoints:getProperties failed.)
</pre>

In Call Stack window (after the next attempt)
<pre>
TinyG3.elf! no-debug-info
</pre>

Output / build window. Populated by the build iinvoked from Debug button:
<pre>
------ Build started: Project: TinyG3, Configuration: Debug ARM ------
Build started.
Project "TinyG3.cppproj" (default targets):
Target "PreBuildEvent" skipped, due to false condition; ('$(PreBuildEvent)'!='') was evaluated as (''!='').
Target "CoreBuild" in file "C:\Program Files\Atmel\Atmel Studio 6.1\Vs\Compiler.targets" from project "Z:\Alden\Projects\proj64_ArduinoDue\g2\TinyG3\TinyG3.cppproj" (target "Build" depends on it):
	Task "RunCompilerTask"
		C:\Program Files\Atmel\Atmel Studio 6.1\shellUtils\make.exe all 
		make: Nothing to be done for `all'.
	Done executing task "RunCompilerTask".
	Task "RunOutputFileVerifyTask"
				Program Memory Usage 	:	38368 bytes   7.3 % Full
				Data Memory Usage 		:	10144 bytes   10.3 % Full
	Done executing task "RunOutputFileVerifyTask".
Done building target "CoreBuild" in project "TinyG3.cppproj".
Target "PostBuildEvent" skipped, due to false condition; ('$(PostBuildEvent)' != '') was evaluated as ('' != '').
Target "Build" in file "C:\Program Files\Atmel\Atmel Studio 6.1\Vs\Avr.common.targets" from project "Z:\Alden\Projects\proj64_ArduinoDue\g2\TinyG3\TinyG3.cppproj" (entry point):
Done building target "Build" in project "TinyG3.cppproj".
Done building project "TinyG3.cppproj".

Build succeeded.
========== Build: 1 succeeded or up-to-date, 0 failed, 0 skipped ==========
</pre>

Drop out of debug and build externally:
<pre>
------ Rebuild All started: Project: TinyG3, Configuration: Debug ARM ------
Build started.
Project "TinyG3.cppproj" (ReBuild target(s)):
Target "PreBuildEvent" skipped, due to false condition; ('$(PreBuildEvent)'!='') was evaluated as (''!='').
Target "CoreRebuild" in file "C:\Program Files\Atmel\Atmel Studio 6.1\Vs\Compiler.targets" from project "Z:\Alden\Projects\proj64_ArduinoDue\g2\TinyG3\TinyG3.cppproj" (target "ReBuild" depends on it):
	Task "RunCompilerTask"
		C:\Program Files\Atmel\Atmel Studio 6.1\shellUtils\make.exe clean all 
		rm -rf  arduino/cortex_handlers.o arduino/cxxabi-compat.o arduino/hooks.o arduino/iar_calls_sam3.o arduino/IPAddress.o arduino/itoa.o arduino/main.o arduino/Print.o arduino/Reset.o arduino/RingBuffer.o arduino/Stream.o arduino/syscalls_sam3.o arduino/UARTClass.o arduino/USARTClass.o arduino/WInterrupts.o arduino/wiring.o arduino/wiring_analog.o arduino/wiring_digital.o arduino/wiring_pulse.o arduino/wiring_shift.o arduino/WMath.o arduino/WString.o cmsis/src/startup_sam3xa.o cmsis/src/system_sam3xa.o stepper.o system.o system/source/adc.o system/source/adc12_sam3u.o system/source/can.o system/source/dacc.o system/source/efc.o system/source/emac.o system/source/gpbr.o system/source/interrupt_sam_nvic.o system/source/pio.o system/source/pmc.o system/source/pwmc.o system/source/rstc.o system/source/rtc.o system/source/rtt.o system/source/spi.o system/source/ssc.o system/source/tc.o system/source/timetick.o system/source/trng.o system/source/twi.o system/source/udp.o system/source/udphs.o system/source/uotghs.o system/source/uotghs_device.o system/source/uotghs_host.o system/source/usart.o system/source/wdt.o tinyg2.o variants/variant.o arduino/cortex_handlers.d arduino/cxxabi-compat.d arduino/hooks.d arduino/iar_calls_sam3.d arduino/IPAddress.d arduino/itoa.d arduino/main.d arduino/Print.d arduino/Reset.d arduino/RingBuffer.d arduino/Stream.d arduino/syscalls_sam3.d arduino/UARTClass.d arduino/USARTClass.d arduino/WInterrupts.d arduino/wiring.d arduino/wiring_analog.d arduino/wiring_digital.d arduino/wiring_pulse.d arduino/wiring_shift.d arduino/WMath.d arduino/WString.d cmsis/src/startup_sam3xa.d cmsis/src/system_sam3xa.d stepper.d system.d system/source/adc.d system/source/adc12_sam3u.d system/source/can.d system/source/dacc.d system/source/efc.d system/source/emac.d system/source/gpbr.d system/source/interrupt_sam_nvic.d system/source/pio.d system/source/pmc.d system/source/pwmc.d system/source/rstc.d system/source/rtc.d system/source/rtt.d system/source/spi.d system/source/ssc.d system/source/tc.d system/source/timetick.d system/source/trng.d system/source/twi.d system/source/udp.d system/source/udphs.d system/source/uotghs.d system/source/uotghs_device.d system/source/uotghs_host.d system/source/usart.d system/source/wdt.d tinyg2.d variants/variant.d  
		rm -rf "TinyG3.elf" "TinyG3.a" "TinyG3.hex" "TinyG3.bin" "TinyG3.lss" "TinyG3.eep" "TinyG3.map" "TinyG3.srec"
		Building file: ../arduino/cortex_handlers.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "arduino/cortex_handlers.d" -MT"arduino/cortex_handlers.d" -MT"arduino/cortex_handlers.o"   -o"arduino/cortex_handlers.o" "../arduino/cortex_handlers.c" 
		Finished building: ../arduino/cortex_handlers.c
		Building file: ../arduino/cxxabi-compat.cpp
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O0 -ffunction-sections -fno-rtti -fno-exceptions -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -fpermissive -MD -MP -MF "arduino/cxxabi-compat.d" -MT"arduino/cxxabi-compat.d" -MT"arduino/cxxabi-compat.o"   -o"arduino/cxxabi-compat.o" "../arduino/cxxabi-compat.cpp" 
		Finished building: ../arduino/cxxabi-compat.cpp
		Building file: ../arduino/hooks.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "arduino/hooks.d" -MT"arduino/hooks.d" -MT"arduino/hooks.o"   -o"arduino/hooks.o" "../arduino/hooks.c" 
		Finished building: ../arduino/hooks.c
		Building file: ../arduino/iar_calls_sam3.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "arduino/iar_calls_sam3.d" -MT"arduino/iar_calls_sam3.d" -MT"arduino/iar_calls_sam3.o"   -o"arduino/iar_calls_sam3.o" "../arduino/iar_calls_sam3.c" 
		Finished building: ../arduino/iar_calls_sam3.c
		Building file: ../arduino/IPAddress.cpp
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O0 -ffunction-sections -fno-rtti -fno-exceptions -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -fpermissive -MD -MP -MF "arduino/IPAddress.d" -MT"arduino/IPAddress.d" -MT"arduino/IPAddress.o"   -o"arduino/IPAddress.o" "../arduino/IPAddress.cpp" 
		Finished building: ../arduino/IPAddress.cpp
		Building file: ../arduino/itoa.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "arduino/itoa.d" -MT"arduino/itoa.d" -MT"arduino/itoa.o"   -o"arduino/itoa.o" "../arduino/itoa.c" 
		Finished building: ../arduino/itoa.c
		Building file: ../arduino/main.cpp
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O0 -ffunction-sections -fno-rtti -fno-exceptions -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -fpermissive -MD -MP -MF "arduino/main.d" -MT"arduino/main.d" -MT"arduino/main.o"   -o"arduino/main.o" "../arduino/main.cpp" 
		Finished building: ../arduino/main.cpp
		Building file: ../arduino/Print.cpp
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O0 -ffunction-sections -fno-rtti -fno-exceptions -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -fpermissive -MD -MP -MF "arduino/Print.d" -MT"arduino/Print.d" -MT"arduino/Print.o"   -o"arduino/Print.o" "../arduino/Print.cpp" 
		Finished building: ../arduino/Print.cpp
		Building file: ../arduino/Reset.cpp
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O0 -ffunction-sections -fno-rtti -fno-exceptions -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -fpermissive -MD -MP -MF "arduino/Reset.d" -MT"arduino/Reset.d" -MT"arduino/Reset.o"   -o"arduino/Reset.o" "../arduino/Reset.cpp" 
		Finished building: ../arduino/Reset.cpp
		Building file: ../arduino/RingBuffer.cpp
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O0 -ffunction-sections -fno-rtti -fno-exceptions -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -fpermissive -MD -MP -MF "arduino/RingBuffer.d" -MT"arduino/RingBuffer.d" -MT"arduino/RingBuffer.o"   -o"arduino/RingBuffer.o" "../arduino/RingBuffer.cpp" 
		Finished building: ../arduino/RingBuffer.cpp
		Building file: ../arduino/Stream.cpp
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O0 -ffunction-sections -fno-rtti -fno-exceptions -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -fpermissive -MD -MP -MF "arduino/Stream.d" -MT"arduino/Stream.d" -MT"arduino/Stream.o"   -o"arduino/Stream.o" "../arduino/Stream.cpp" 
		Finished building: ../arduino/Stream.cpp
		Building file: ../arduino/syscalls_sam3.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "arduino/syscalls_sam3.d" -MT"arduino/syscalls_sam3.d" -MT"arduino/syscalls_sam3.o"   -o"arduino/syscalls_sam3.o" "../arduino/syscalls_sam3.c" 
		Finished building: ../arduino/syscalls_sam3.c
		Building file: ../arduino/UARTClass.cpp
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O0 -ffunction-sections -fno-rtti -fno-exceptions -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -fpermissive -MD -MP -MF "arduino/UARTClass.d" -MT"arduino/UARTClass.d" -MT"arduino/UARTClass.o"   -o"arduino/UARTClass.o" "../arduino/UARTClass.cpp" 
		Finished building: ../arduino/UARTClass.cpp
		Building file: ../arduino/USARTClass.cpp
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O0 -ffunction-sections -fno-rtti -fno-exceptions -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -fpermissive -MD -MP -MF "arduino/USARTClass.d" -MT"arduino/USARTClass.d" -MT"arduino/USARTClass.o"   -o"arduino/USARTClass.o" "../arduino/USARTClass.cpp" 
		Finished building: ../arduino/USARTClass.cpp
		Building file: ../arduino/WInterrupts.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "arduino/WInterrupts.d" -MT"arduino/WInterrupts.d" -MT"arduino/WInterrupts.o"   -o"arduino/WInterrupts.o" "../arduino/WInterrupts.c" 
		Finished building: ../arduino/WInterrupts.c
		Building file: ../arduino/wiring.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "arduino/wiring.d" -MT"arduino/wiring.d" -MT"arduino/wiring.o"   -o"arduino/wiring.o" "../arduino/wiring.c" 
		Finished building: ../arduino/wiring.c
		Building file: ../arduino/wiring_analog.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "arduino/wiring_analog.d" -MT"arduino/wiring_analog.d" -MT"arduino/wiring_analog.o"   -o"arduino/wiring_analog.o" "../arduino/wiring_analog.c" 
		Finished building: ../arduino/wiring_analog.c
		Building file: ../arduino/wiring_digital.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "arduino/wiring_digital.d" -MT"arduino/wiring_digital.d" -MT"arduino/wiring_digital.o"   -o"arduino/wiring_digital.o" "../arduino/wiring_digital.c" 
		Finished building: ../arduino/wiring_digital.c
		Building file: ../arduino/wiring_pulse.cpp
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O0 -ffunction-sections -fno-rtti -fno-exceptions -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -fpermissive -MD -MP -MF "arduino/wiring_pulse.d" -MT"arduino/wiring_pulse.d" -MT"arduino/wiring_pulse.o"   -o"arduino/wiring_pulse.o" "../arduino/wiring_pulse.cpp" 
		Finished building: ../arduino/wiring_pulse.cpp
		Building file: ../arduino/wiring_shift.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "arduino/wiring_shift.d" -MT"arduino/wiring_shift.d" -MT"arduino/wiring_shift.o"   -o"arduino/wiring_shift.o" "../arduino/wiring_shift.c" 
		Finished building: ../arduino/wiring_shift.c
		Building file: ../arduino/WMath.cpp
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O0 -ffunction-sections -fno-rtti -fno-exceptions -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -fpermissive -MD -MP -MF "arduino/WMath.d" -MT"arduino/WMath.d" -MT"arduino/WMath.o"   -o"arduino/WMath.o" "../arduino/WMath.cpp" 
		Finished building: ../arduino/WMath.cpp
		Building file: ../arduino/WString.cpp
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O0 -ffunction-sections -fno-rtti -fno-exceptions -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -fpermissive -MD -MP -MF "arduino/WString.d" -MT"arduino/WString.d" -MT"arduino/WString.o"   -o"arduino/WString.o" "../arduino/WString.cpp" 
		Finished building: ../arduino/WString.cpp
		Building file: ../cmsis/src/startup_sam3xa.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "cmsis/src/startup_sam3xa.d" -MT"cmsis/src/startup_sam3xa.d" -MT"cmsis/src/startup_sam3xa.o"   -o"cmsis/src/startup_sam3xa.o" "../cmsis/src/startup_sam3xa.c" 
		Finished building: ../cmsis/src/startup_sam3xa.c
		Building file: ../cmsis/src/system_sam3xa.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "cmsis/src/system_sam3xa.d" -MT"cmsis/src/system_sam3xa.d" -MT"cmsis/src/system_sam3xa.o"   -o"cmsis/src/system_sam3xa.o" "../cmsis/src/system_sam3xa.c" 
		Finished building: ../cmsis/src/system_sam3xa.c
		Building file: .././stepper.cpp
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O0 -ffunction-sections -fno-rtti -fno-exceptions -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -fpermissive -MD -MP -MF "stepper.d" -MT"stepper.d" -MT"stepper.o"   -o"stepper.o" ".././stepper.cpp" 
		In file included from ..\motate/motatePins.h:52:0,
		                 from .././stepper.cpp:145:
Z:\Alden\Projects\proj64_ArduinoDue\g2\TinyG3\motate\utility\SamPins.h(379,17): 'Motate::nullPin' defined but not used [-Wunused-variable]
		Finished building: .././stepper.cpp
		Building file: .././system.cpp
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O0 -ffunction-sections -fno-rtti -fno-exceptions -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -fpermissive -MD -MP -MF "system.d" -MT"system.d" -MT"system.o"   -o"system.o" ".././system.cpp" 
		Finished building: .././system.cpp
		Building file: ../system/source/adc.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/adc.d" -MT"system/source/adc.d" -MT"system/source/adc.o"   -o"system/source/adc.o" "../system/source/adc.c" 
		Finished building: ../system/source/adc.c
		Building file: ../system/source/adc12_sam3u.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/adc12_sam3u.d" -MT"system/source/adc12_sam3u.d" -MT"system/source/adc12_sam3u.o"   -o"system/source/adc12_sam3u.o" "../system/source/adc12_sam3u.c" 
		Finished building: ../system/source/adc12_sam3u.c
		Building file: ../system/source/can.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/can.d" -MT"system/source/can.d" -MT"system/source/can.o"   -o"system/source/can.o" "../system/source/can.c" 
		Finished building: ../system/source/can.c
		Building file: ../system/source/dacc.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/dacc.d" -MT"system/source/dacc.d" -MT"system/source/dacc.o"   -o"system/source/dacc.o" "../system/source/dacc.c" 
		Finished building: ../system/source/dacc.c
		Building file: ../system/source/efc.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/efc.d" -MT"system/source/efc.d" -MT"system/source/efc.o"   -o"system/source/efc.o" "../system/source/efc.c" 
		Finished building: ../system/source/efc.c
		Building file: ../system/source/emac.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/emac.d" -MT"system/source/emac.d" -MT"system/source/emac.o"   -o"system/source/emac.o" "../system/source/emac.c" 
		Finished building: ../system/source/emac.c
		Building file: ../system/source/gpbr.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/gpbr.d" -MT"system/source/gpbr.d" -MT"system/source/gpbr.o"   -o"system/source/gpbr.o" "../system/source/gpbr.c" 
		Finished building: ../system/source/gpbr.c
		Building file: ../system/source/interrupt_sam_nvic.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/interrupt_sam_nvic.d" -MT"system/source/interrupt_sam_nvic.d" -MT"system/source/interrupt_sam_nvic.o"   -o"system/source/interrupt_sam_nvic.o" "../system/source/interrupt_sam_nvic.c" 
		Finished building: ../system/source/interrupt_sam_nvic.c
		Building file: ../system/source/pio.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/pio.d" -MT"system/source/pio.d" -MT"system/source/pio.o"   -o"system/source/pio.o" "../system/source/pio.c" 
		Finished building: ../system/source/pio.c
		Building file: ../system/source/pmc.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/pmc.d" -MT"system/source/pmc.d" -MT"system/source/pmc.o"   -o"system/source/pmc.o" "../system/source/pmc.c" 
		Finished building: ../system/source/pmc.c
		Building file: ../system/source/pwmc.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/pwmc.d" -MT"system/source/pwmc.d" -MT"system/source/pwmc.o"   -o"system/source/pwmc.o" "../system/source/pwmc.c" 
		Finished building: ../system/source/pwmc.c
		Building file: ../system/source/rstc.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/rstc.d" -MT"system/source/rstc.d" -MT"system/source/rstc.o"   -o"system/source/rstc.o" "../system/source/rstc.c" 
		Finished building: ../system/source/rstc.c
		Building file: ../system/source/rtc.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/rtc.d" -MT"system/source/rtc.d" -MT"system/source/rtc.o"   -o"system/source/rtc.o" "../system/source/rtc.c" 
		Finished building: ../system/source/rtc.c
		Building file: ../system/source/rtt.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/rtt.d" -MT"system/source/rtt.d" -MT"system/source/rtt.o"   -o"system/source/rtt.o" "../system/source/rtt.c" 
		Finished building: ../system/source/rtt.c
		Building file: ../system/source/spi.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/spi.d" -MT"system/source/spi.d" -MT"system/source/spi.o"   -o"system/source/spi.o" "../system/source/spi.c" 
		Finished building: ../system/source/spi.c
		Building file: ../system/source/ssc.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/ssc.d" -MT"system/source/ssc.d" -MT"system/source/ssc.o"   -o"system/source/ssc.o" "../system/source/ssc.c" 
		Finished building: ../system/source/ssc.c
		Building file: ../system/source/tc.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/tc.d" -MT"system/source/tc.d" -MT"system/source/tc.o"   -o"system/source/tc.o" "../system/source/tc.c" 
		Finished building: ../system/source/tc.c
		Building file: ../system/source/timetick.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/timetick.d" -MT"system/source/timetick.d" -MT"system/source/timetick.o"   -o"system/source/timetick.o" "../system/source/timetick.c" 
		Finished building: ../system/source/timetick.c
		Building file: ../system/source/trng.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/trng.d" -MT"system/source/trng.d" -MT"system/source/trng.o"   -o"system/source/trng.o" "../system/source/trng.c" 
		Finished building: ../system/source/trng.c
		Building file: ../system/source/twi.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/twi.d" -MT"system/source/twi.d" -MT"system/source/twi.o"   -o"system/source/twi.o" "../system/source/twi.c" 
		Finished building: ../system/source/twi.c
		Building file: ../system/source/udp.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/udp.d" -MT"system/source/udp.d" -MT"system/source/udp.o"   -o"system/source/udp.o" "../system/source/udp.c" 
		Finished building: ../system/source/udp.c
		Building file: ../system/source/udphs.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/udphs.d" -MT"system/source/udphs.d" -MT"system/source/udphs.o"   -o"system/source/udphs.o" "../system/source/udphs.c" 
		Finished building: ../system/source/udphs.c
		Building file: ../system/source/uotghs.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/uotghs.d" -MT"system/source/uotghs.d" -MT"system/source/uotghs.o"   -o"system/source/uotghs.o" "../system/source/uotghs.c" 
		Finished building: ../system/source/uotghs.c
		Building file: ../system/source/uotghs_device.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/uotghs_device.d" -MT"system/source/uotghs_device.d" -MT"system/source/uotghs_device.o"   -o"system/source/uotghs_device.o" "../system/source/uotghs_device.c" 
		Finished building: ../system/source/uotghs_device.c
		Building file: ../system/source/uotghs_host.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/uotghs_host.d" -MT"system/source/uotghs_host.d" -MT"system/source/uotghs_host.o"   -o"system/source/uotghs_host.o" "../system/source/uotghs_host.c" 
		Finished building: ../system/source/uotghs_host.c
		Building file: ../system/source/usart.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/usart.d" -MT"system/source/usart.d" -MT"system/source/usart.o"   -o"system/source/usart.o" "../system/source/usart.c" 
		Finished building: ../system/source/usart.c
		Building file: ../system/source/wdt.c
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-gcc.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O1 -ffunction-sections -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -std=gnu99 -MD -MP -MF "system/source/wdt.d" -MT"system/source/wdt.d" -MT"system/source/wdt.o"   -o"system/source/wdt.o" "../system/source/wdt.c" 
		Finished building: ../system/source/wdt.c
		Building file: .././tinyg2.cpp
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O0 -ffunction-sections -fno-rtti -fno-exceptions -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -fpermissive -MD -MP -MF "tinyg2.d" -MT"tinyg2.d" -MT"tinyg2.o"   -o"tinyg2.o" ".././tinyg2.cpp" 
Z:\Alden\Projects\proj64_ArduinoDue\g2\TinyG3\tinyg2.cpp(112,13): 'void _unit_tests()' defined but not used [-Wunused-function]
		Finished building: .././tinyg2.cpp
		Building file: ../variants/variant.cpp
		Invoking: ARM/GNU C Compiler : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -mthumb -D__SAM3X8E__ -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\CMSIS\Include" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL" -I"C:\PROGRA~1\Atmel\ATMELT~1\ARMGCC~1\Native\472~1.87\CMSIS_~1\Device\ATMEL\sam3xa\include" -I"..\arduino" -I"..\system" -I"..\system\include" -I"..\system\source" -I"..\variants" -I"..\motate" -I"..\motate\utility"  -O0 -ffunction-sections -fno-rtti -fno-exceptions -mlong-calls -g3 -Wall -mcpu=cortex-m3 -c -fpermissive -MD -MP -MF "variants/variant.d" -MT"variants/variant.d" -MT"variants/variant.o"   -o"variants/variant.o" "../variants/variant.cpp" 
		Finished building: ../variants/variant.cpp
		Building target: TinyG3.elf
		Invoking: ARM/GNU Linker : 
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-g++.exe" -o TinyG3.elf  arduino/cortex_handlers.o arduino/cxxabi-compat.o arduino/hooks.o arduino/iar_calls_sam3.o arduino/IPAddress.o arduino/itoa.o arduino/main.o arduino/Print.o arduino/Reset.o arduino/RingBuffer.o arduino/Stream.o arduino/syscalls_sam3.o arduino/UARTClass.o arduino/USARTClass.o arduino/WInterrupts.o arduino/wiring.o arduino/wiring_analog.o arduino/wiring_digital.o arduino/wiring_pulse.o arduino/wiring_shift.o arduino/WMath.o arduino/WString.o cmsis/src/startup_sam3xa.o cmsis/src/system_sam3xa.o stepper.o system.o system/source/adc.o system/source/adc12_sam3u.o system/source/can.o system/source/dacc.o system/source/efc.o system/source/emac.o system/source/gpbr.o system/source/interrupt_sam_nvic.o system/source/pio.o system/source/pmc.o system/source/pwmc.o system/source/rstc.o system/source/rtc.o system/source/rtt.o system/source/spi.o system/source/ssc.o system/source/tc.o system/source/timetick.o system/source/trng.o system/source/twi.o system/source/udp.o system/source/udphs.o system/source/uotghs.o system/source/uotghs_device.o system/source/uotghs_host.o system/source/usart.o system/source/wdt.o tinyg2.o variants/variant.o   -Wl,-Map="TinyG3.map" -Wl,--start-group -lm  -Wl,--end-group -L"../cmsis/linkerScripts"  -Wl,--gc-sections -mcpu=cortex-m3 -Tsam3x8e_flash.ld 
		Finished building target: TinyG3.elf
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-objcopy.exe" -O binary "TinyG3.elf" "TinyG3.bin"
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-objcopy.exe" -O ihex -R .eeprom -R .fuse -R .lock -R .signature  "TinyG3.elf" "TinyG3.hex"
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-objcopy.exe" -j .eeprom --set-section-flags=.eeprom=alloc,load --change-section-lma .eeprom=0 --no-change-warnings -O binary "TinyG3.elf" "TinyG3.eep" || exit 0
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-objdump.exe" -h -S "TinyG3.elf" > "TinyG3.lss"
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-objcopy.exe" -O srec -R .eeprom -R .fuse -R .lock -R .signature  "TinyG3.elf" "TinyG3.srec"
		"C:\Program Files\Atmel\Atmel Toolchain\ARM GCC\Native\4.7.2.87\arm-gnu-toolchain\bin\arm-none-eabi-size.exe" "TinyG3.elf"
		   text	   data	    bss	    dec	    hex	filename
		  38368	      0	  10144	  48512	   bd80	TinyG3.elf
	Done executing task "RunCompilerTask".
	Task "RunOutputFileVerifyTask"
				Program Memory Usage 	:	38368 bytes   7.3 % Full
				Data Memory Usage 		:	10144 bytes   10.3 % Full
	Done executing task "RunOutputFileVerifyTask".
Done building target "CoreRebuild" in project "TinyG3.cppproj".
Target "PostBuildEvent" skipped, due to false condition; ('$(PostBuildEvent)' != '') was evaluated as ('' != '').
Target "ReBuild" in file "C:\Program Files\Atmel\Atmel Studio 6.1\Vs\Avr.common.targets" from project "Z:\Alden\Projects\proj64_ArduinoDue\g2\TinyG3\TinyG3.cppproj" (entry point):
Done building target "ReBuild" in project "TinyG3.cppproj".
Done building project "TinyG3.cppproj".

Build succeeded.
========== Rebuild All: 1 succeeded, 0 failed, 0 skipped ==========
</pre>

Ran debug and stop again. Same problem. No break. Output/build is:
<pre>
------ Build started: Project: TinyG3, Configuration: Debug ARM ------
Build started.
Project "TinyG3.cppproj" (default targets):
Target "PreBuildEvent" skipped, due to false condition; ('$(PreBuildEvent)'!='') was evaluated as (''!='').
Target "CoreBuild" in file "C:\Program Files\Atmel\Atmel Studio 6.1\Vs\Compiler.targets" from project "Z:\Alden\Projects\proj64_ArduinoDue\g2\TinyG3\TinyG3.cppproj" (target "Build" depends on it):
	Task "RunCompilerTask"
		C:\Program Files\Atmel\Atmel Studio 6.1\shellUtils\make.exe all 
		make: Nothing to be done for `all'.
	Done executing task "RunCompilerTask".
	Task "RunOutputFileVerifyTask"
				Program Memory Usage 	:	38368 bytes   7.3 % Full
				Data Memory Usage 		:	10144 bytes   10.3 % Full
	Done executing task "RunOutputFileVerifyTask".
Done building target "CoreBuild" in project "TinyG3.cppproj".
Target "PostBuildEvent" skipped, due to false condition; ('$(PostBuildEvent)' != '') was evaluated as ('' != '').
Target "Build" in file "C:\Program Files\Atmel\Atmel Studio 6.1\Vs\Avr.common.targets" from project "Z:\Alden\Projects\proj64_ArduinoDue\g2\TinyG3\TinyG3.cppproj" (entry point):
Done building target "Build" in project "TinyG3.cppproj".
Done building project "TinyG3.cppproj".

Build succeeded.
========== Build: 1 succeeded or up-to-date, 0 failed, 0 skipped ==========
</pre>

Output / backend agent:
<pre>
10 30 17 521: sgr Could not read hardware status!
10 30 17 521: bp Error clearing breakpoint at address 0x80382
10 30 17 536: bp Error clearing breakpoint at address 0x82a5a
10 30 40 925:  Invalid context 1576_bp_00000000,
10 30 40 940:  Invalid context 1576_bp_00000001,
10 35 05 036:  Invalid context 1576_bp_00000002,
10 37 17 189:  Invalid context 1576_bp_00000003,
</pre>
