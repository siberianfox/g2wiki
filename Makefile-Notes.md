What gets compiled:

At the top of the Makefile is two variables, SOURCE_DIRS, and FIRST_LINK_SOURCES, that dictate where source files are located, and which ones are linked in first.

SOURCE_DIRS is a list of directories. In each named directory (relative to the Makefile), the following shell-globs are searched for:
<pre>
	*.cpp
	*.c
	*.S
</pre>

Any files that match those patterns are then compiled using the same options for each type. All of the *.cpp files use CPPFLAGS, all of the *.c files use CFLAGS, and all of the *.S files use ASFLAGS. (It should be noted that, AFAIK, we don’t actually have ay *.S files in the project, but the support for them is there if we need it.)

LDFLAGS is used when linking. You should rarely need to modify this directly.

Additionally, the same “includes” paths are added to the compile flags for all types of files. These are derived from a concatenation of two variables: DEVICE_INCLUDE_DIRS and USER_INCLUDE_DIRS, in that order. The order within these variables will be preserved as well, and will have an impact on the compilation search order for header files. Generally, only the USER_INCLUDE_DIRS should be edited on a project-wide basis, and is defined in the top section of the Makefile. The default definition adds SOURCE_DIRS to the list. Each item in *_INCLUDE_DIRS will have the prefix ‘-I’ (uppercase “eye”) added to it.

The contents of the variables DEVICE_LIBS and USER_LIBS contains library names to be included when linking. Each item in *_LIBS will have the prefix ‘-l’ (lowercase ‘ell’) added to it.

The contents of the variables DEVICE_LIB_DIRS and USER_LIB_DIRS contains lists of directories to search for the libraries when linking. Each item in *_LIB_DIRS will have the prefix ‘-L’ (uppercase ‘ell’) added to it.

One variable decides what platform the code will be compiled for: PLATFORM. It defaults to ‘due,’ which is the only platform I know works.



Each platform has it’s own section of the makefile, in an ifeq block, where it will define a few important variables:

* CROSS_COMPILE — this is the prefix of the programs used for compiling. For the arm, this should be 'arm-none-eabi-‘ (the trailing dash is important). This will yield program names like arm-none-eabi-gcc, arm-none-eabi-objcopy, etc.

* MEMORIES — this defines the different memory types that will be compiled for. These are actually arbitrary, and simply make wholly separate sub-platforms.

* DEVICE_RULES — This is a string that, when evaluated lated in the primary makefile, will generate any needed rules for that platform. You can think of the contents of this variable as an inline #include file, held in a variable. (See CREATE_DEVICE_LIBRARY later.)

* DEVICE_INCLUDE_DIRS — A device-specific list of include directories. Each item in this array will be prefixed with ‘-I’ and added to the compilation flags.

* DEVICE_LIBS contains library names to be included when linking. Each item in DEVICE_LIBS will have the prefix ‘-l’ (lowercase ‘ell’) added to it.

* DEVICE_LIB_DIRS — A device-specific lists of directories to search for the libraries when linking. Each item in DEVICE_LIB_DIRS will have the prefix ‘-L’ (uppercase ‘ell’) added to it.

* DEVICE_OBJECTS_(memory_type) — A list of items that must be linked in for this device type and memory type combination. Rules to make these objects must otherwise be provided, usually in DEVICE_RULES. 

Additionally, it may directly alter the following variables:

* SOURCE_DIRS — Adding directories to this list will also add the contents to OBJECTS (so adding them to  DEVICE_OBJECTS_(memory_type) is unnecessary) and creates rules for them with the standard options.

* CFLAGS, CPPFLAGS, ASFLAGS, and LDFLAGS — these can each be added to as necessary, when the other facilities to do so are inadequate.


What is being used right now  (generated with this command, and then reformatted some: make flash VERBOSE=2):

INCLUDES:
<pre>
	platform/atmel_sam/libsam
	platform/atmel_sam/libsam/include
	"CMSIS/CMSIS/Include"
	"CMSIS/Device/ATMEL"
	"CMSIS/Device/ATMEL/sam3xa/include"
	motate
	.
	arduino
	arduino/USB
	variants
</pre>

LIBRARIES:
<pre>
</pre>
	

SOURCE DIRECTORIES:
<pre>
	.
	arduino
	arduino/USB
	variants
	platform/atmel_sam/libsam/source
</pre>


Those yield these actual files being compiled using their respective compiler:

C SOURCES:
<pre>
	./config.c
	./config_app.c
	./json_parser.c
	./kinen.c
	arduino/WInterrupts.c
	arduino/cortex_handlers.c
	arduino/hooks.c
	arduino/iar_calls_sam3.c
	arduino/itoa.c
	arduino/wiring.c
	arduino/wiring_analog.c
	arduino/wiring_digital.c
	arduino/wiring_shift.c
	platform/atmel_sam/libsam/source/adc.c
	platform/atmel_sam/libsam/source/adc12_sam3u.c
	platform/atmel_sam/libsam/source/can.c
	platform/atmel_sam/libsam/source/dacc.c
	platform/atmel_sam/libsam/source/efc.c
	platform/atmel_sam/libsam/source/emac.c
	platform/atmel_sam/libsam/source/gpbr.c
	platform/atmel_sam/libsam/source/interrupt_sam_nvic.c
	platform/atmel_sam/libsam/source/pio.c
	platform/atmel_sam/libsam/source/pmc.c
	platform/atmel_sam/libsam/source/pwmc.c
	platform/atmel_sam/libsam/source/rstc.c
	platform/atmel_sam/libsam/source/rtc.c
	platform/atmel_sam/libsam/source/rtt.c
	platform/atmel_sam/libsam/source/spi.c
	platform/atmel_sam/libsam/source/ssc.c
	platform/atmel_sam/libsam/source/tc.c
	platform/atmel_sam/libsam/source/timetick.c
	platform/atmel_sam/libsam/source/trng.c
	platform/atmel_sam/libsam/source/twi.c
	platform/atmel_sam/libsam/source/udp.c
	platform/atmel_sam/libsam/source/udphs.c
	platform/atmel_sam/libsam/source/uotghs.c
	platform/atmel_sam/libsam/source/uotghs_device.c
	platform/atmel_sam/libsam/source/uotghs_host.c
	platform/atmel_sam/libsam/source/usart.c
	platform/atmel_sam/libsam/source/wdt.c
</pre>

C++ SOURCES:
<pre>
	./controller.cpp
	./main.cpp
	./stepper.cpp
	./system.cpp
	./xio.cpp
	arduino/IPAddress.cpp
	arduino/Print.cpp
	arduino/Reset.cpp
	arduino/RingBuffer.cpp
	arduino/Stream.cpp
	arduino/UARTClass.cpp
	arduino/USARTClass.cpp
	arduino/WMath.cpp
	arduino/WString.cpp
	arduino/cxxabi-compat.cpp
	arduino/syscalls_sam3.cpp
	arduino/wiring_pulse.cpp
	arduino/USB/CDC.cpp
	arduino/USB/HID.cpp
	arduino/USB/USBCore.cpp
	variants/variant.cpp
</pre>

And these files will be first in the link line:

LINK FIRST SOURCES:
<pre>
	arduino/syscalls_sam3.c
</pre>
