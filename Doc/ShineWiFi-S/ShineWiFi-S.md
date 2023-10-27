# Description of Hardware

Collection of Information around this Hardware

## ShineWifi-S

BeoQ, December 2021, based on inspection of pictures by otti


### Physical build

The stick has a plastic case with a standard Sub-D 9-Pin connector, as used in legacy serial ports. 
It can be locked to the inverter. There is a small 'key' hole for the button.


### Electrical build

The PCB is equipped with the following major functional parts

* Switchmode voltage regulator
* ESP8266 in a ESP07 package
* Simple RS232 level shifter
* RTC on I2C-bus with CR1220 backup battery
* 8MB Flash on SPI-bus
* Three status LEDs around the button
* One pushbutton 'key'
* WiFi Antenna wire on IPEX connector


There is an unpopulated connector CN1  
1: 3V3  
2: GND  
3: RX0  
4: TX0  
5: ?  
6: GPIO0  
7: GND  


### Modbus Communication

Then, the ShineWiFi-S is the Modbus master, while the inverter responds to Modbus address #1 at 9600 Baud, 8N1

Modbus Protocol description v3.05 dated 2013 seems to be applicable.

It uses Modbus command 0x03 (Read Input Registers) for all the volatile information (i.e voltages, power, ...)  
It uses Modbus command 0x04 (Read Holding Registers) for static information (i.e. capabilities, firmware version, ...)



### Loading Firmware

Initial loading of Firmware to the stick is rather easy:

Remove the stick's PCB from the housing. 
You need some power supply for the module while programming, e.g. if you use an USB-serial module that can supply the 3.3V while programming.

One way to do this:
Temporarily  connect the header's Pins GPIO0 and GND while powering-on the stick.
Connect your USB-to-Serial module to the pinheader and power. Note that Rx and Tx labels are from the point of view of the microcontroller; i.e. you need to cross Rx and Tx.
Upload of the compiled binary of the firmware. (using arduino-ide, esptool or avrdude, whatever you prefer)

You could also use the 9-Pin serial port, if you power 8V from external and take care of GPIO0 (untested).

Another way to do this (Otti):
Add a 1k resistor between the output of the SW1 and GPIO0. So it is possible to put the device into boot mode on start up


Updating an installed firmware is very easy using OTA the buildin config webserver (192.168.4.1):
Use:  http://&lt;config ip&gt;/ and click on update.


### Arduino IDE settings
* Board: Generic ESP8266 Module
* Flash Mode: DIO
* Crystal Freq:: 26 MHz
* Flash Freq: 40 MHz
* Upload Using: Serial
* CPU Freq: 80 MHz
* Flash Size: 4 MB (FS:1MB OTA~1019KB)
* UploadSpeed: 115200
