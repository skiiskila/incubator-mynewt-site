# Bring up a Mynewt supported sensor

## Pre-requisites

Ensure that you meet the following prerequisites before continuing with one of the tutorials.

Have Internet connectivity to fetch remote Mynewt components.
Have a computer to build a Mynewt application and connect to the board over USB.
Have a Micro-USB cable to connect the board and the computer.
Install the Newt tool and toolchains (See Basic Setup).
Install either the Jlink or OpenOCD debugger.
Create a project space (directory structure) and populate it with the core code repository (apache-mynewt-core) or know how to as explained in Creating Your First Project.
Read the Mynewt OS Concepts section.

## Hardware chosen for this tutorial example

(a) Adafruit BNO055 sensor
https://learn.adafruit.com/adafruit-bno055-absolute-orientation-sensor/overview

(b)nRF52 dev board

### Minicom is used as a serial terminal emulator in this tutorial.  Please make sure you have minicom and another serial emulator installed.

## Hardware set up

Sensor is connected to nRF52 dev board via I2C interface

### Pins used for I2C connection
Power (V_in) - connect this pin on the sensor board to the 5V pin on the nRF52
Clock Line (SCL) - connect this pin on the sensor board to pin 27 on the nRF52
Data (SDA) - connect this pin on the sensor board to pin 26 on the nRF52
Ground (GND) - connect this pin on the sensor board to GND pin on the nRF52

### Serial setup
To make the serial connection, you need a FTDI cable (http://www.ftdichip.com/Support/Documents/DataSheets/Cables/DS_USB_RS232_CABLES.pdf).  The USB end goes to the computer and the pins go to the nRF52 dev board

### Pins used for serial connection:
RX - the RX wire (yellow) of the cable goes into Pin 6 (which is the TX on the nRF52)
TX - the TX wire (orange) of the cable goes into Pin 8 (which is the RX on the nRF52)
GND - the GND wire (black) of the cable goes into a ground pin on the nRF52.

### Connect the nrF52 board to the computer via USB-microUSB cable

### Create 2 targets - Bootloader and Sensor application

$ newt target create nrf52_boot
$ newt target set nrf52_boot bsp=@apache-mynewt-core/hw/bsp/nrf52dk
$ newt target set nrf52_boot build_profile=optimized
$ newt target set nrf52_boot app=@apache-mynewt-core/apps/boot

$ newt target create nrf52_sensor
$ newt target set nrf52_sensor bsp=@apache-mynewt-core/hw/bsp/nrf52dk
$ newt target set nrf52_sensor build_profile=debug
$ newt target set nrf52_sensor app=@apache-mynewt-core/apps/sensors_test

```no-highlight
~/dev/testing$ newt target show
targets/nrf52_boot
    app=@apache-mynewt-core/apps/boot
    bsp=@apache-mynewt-core/hw/bsp/nrf52dk
    build_profile=optimized
targets/nrf52_sensor
    app=@apache-mynewt-core/apps/sensors_test
    bsp=@apache-mynewt-core/hw/bsp/nrf52dk
    build_profile=debug
``` 
  
To enable I2C communications, the sensor target needs to be customized.  You can use the newt tool to override the defaults with custom values, or you could directly make the changes in the syscfg.yml file (both options shown below).



