# Start openocd for gdb_server over IP network

By default openocd start gdb_server listen port 3333 on interface of localhost only.

By adding `-c "bindto 0.0.0.0"` to the end of openocd, the gdb_server started will listen on all network interfaces.  
So as an example on a rasPi with GPIO_SWD deriver settings, it could start a gdb_server as 
`sudo  openocd -f interface/raspberrypi-swd.cfg -f target/rp2040.cfg -c "bindto 0.0.0.0" ` 



# GDB Client connect to remote GDB_Server by openocd on Raspberry Pi

```
$ gdb-multiarch

# target remote  gdb_server_ip:gbd_port_num
(gdb) target remote  rpZw:3333

# read symbol file and exe binary
(gdb)  file  path/my_program.elf

# load/flash program
(gdb)  load

# reset target /w halt
(gdb)  monitor reset 
  # or
(gdb)  monitor reset halt
# set breakpoint at a file line
(gdb)  b  10
 # or
(gdb)  b  function_name:10

# show information on breakpoints
(gdb)  info b

# delete a breakpoint num
(gdb)  d br 2

# show source files
(gdb)  info sources

# List specified function or line
	# list code about the function_entry
(gdb)  list _entry_point
	# list about the line in 'current file'
(gdb)  list 5
  # or  specific fule and line
(gdb)  list path/my_file.c:5

# next 
(gdb)  n

# step
(gdb)  s

# Examining Memory, x/nfu addr	,x addr	,x
(gdb)  x  0x100
(gdb)  x  main
(gdb)  x  _entry_point
(gdb)  x  

# Print the names and values of all registers except floating-point and vector registers
(gdb)  info registers

# print variable
(gdb)  p  variable

# run/continue the program
(gdb)  c

# pause running program
^c

# Print backtrace of all stack frames, or innermost COUNT frames
(gdb)  bt
  # or 2 levels of backtrace
(gdb)  bt 2

# Print value of expression EXP each time the program stops.
(gdb)  display main

# many helps
(gdb)  help
  # or
(gdb)  help info
(gdb)  help target
(gdb)  help break

# quit
(gdb)  q
  # or
(gdb)  Ctrl-d 

```



# Summary Flash Program through GDB client

```
$ gdb-multiarch  path/my_program.elf
(gdb) target remote  rpZw:3333

# read symbol file and exe binary
(gdb)  file  path/my_program.elf

# load/flash program
(gdb)  load

# reset target /w halt
(gdb)  monitor reset 
```



# Build RasPico project/firmware with -debug option

use the CMake directive ‘ -DCMAKE_BUILD_TYPE=Debug ’.

 

```shell
$ export PICO_SDK_PATH=../../pico-sdk
$ cmake -DCMAKE_BUILD_TYPE=Debug ..

```



# UART connection

In case want to use UART (GPIO_14/15; ) on Raspberry Pi to cross connect to UARTx on RasPico,  through /dev/ttyAMA0.

```
$ sudo chmod 777 /dev/ttyAMA0
$ screen /dev/ttyAMA0 115200
```

It may need a configure to connect GPIO_14/15 to UART function ttyAMA0 . 

[]: https://www.abelectronics.co.uk/kb/article/1035/raspberry-pi-3--4-and-zero-w-serial-port-usage	"Serial Port setup in Raspberry Pi OS"

 This is little tricky as the overlay functions on RasPi.  May not work with WiFi simultaneously. 

# Wire connections (Raspberry Pi Pico to Raspberry Pi)

 The following table shows all the necessary connections between Raspberry Pi and Raspberry Pi Pico that you need to make.

| Raspberry Pi Pico   [Debug Target] | Raspberry Pi  [GPIO_SWD_Master  GDB Server] |
| :--------------------------------: | :-----------------------------------------: |
|               SWDIO                |              GPIO 24 (PIN 18)               |
|              SWD GND               |                GND (PIN 20)                 |
|               SWCLK                |              GPIO 25 (PIN 22)               |



# Build openocd for RasPi GPIO SWD master interface 

On Raspberry Pi, in case the default openocd doesn't include raspberry_gpio_swd support, then build it manually. 

```
$ sudo apt install automake autoconf build-essential texinfo libtool libftdi-dev libusb-1.0-0-dev

$ git clone https://github.com/raspberrypi/openocd.git –recursive –branch rp2040 –depth=1
$ cd openocd/
$ ./conﬁgure –enable-ftdi –enable-sysfsgpio –enable-bcm2835gpio
$ make
$ sudo make install
```
