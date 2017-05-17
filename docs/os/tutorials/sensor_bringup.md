# Bring up a Mynewt supported sensor

## Prerequisites

Ensure that you meet the following prerequisites before continuing with one of the tutorials.

* Have Internet connectivity to fetch remote Mynewt components. 
* Have a computer to build a Mynewt application and connect to the board over USB. 
* Have a Micro-USB cable to connect the board and the computer. 
* Install the Newt tool and toolchains (See Basic Setup).
* Install either the Jlink or OpenOCD debugger.
* Create a project space (directory structure) and populate it with the core code repository (apache-mynewt-core) or know how to as explained in Creating Your First Project.
* Read the Mynewt OS Concepts section.

## Hardware chosen for this tutorial example

(a) Adafruit BNO055 sensor
https://learn.adafruit.com/adafruit-bno055-absolute-orientation-sensor/overview

(b)nRF52 dev board

### Minicom is used as a serial terminal emulator in this tutorial.  Please make sure you have minicom and another serial emulator installed.

## Hardware set up

Sensor is connected to nRF52 dev board via I2C interface

### Pins used for I2C connection
* Power (V_in) - connect this pin on the sensor board to the 5V pin on the nRF52
* Clock Line (SCL) - connect this pin on the sensor board to pin 27 on the nRF52
* Data (SDA) - connect this pin on the sensor board to pin 26 on the nRF52
* Ground (GND) - connect this pin on the sensor board to GND pin on the nRF52

### Serial setup
To make the serial connection, you need a FTDI cable (http://www.ftdichip.com/Support/Documents/DataSheets/Cables/DS_USB_RS232_CABLES.pdf).  The USB end goes to the computer and the pins go to the nRF52 dev board

### Pins used for serial connection:
* RX - the RX wire (yellow) of the cable goes into Pin 6 (which is the TX on the nRF52)
* TX - the TX wire (orange) of the cable goes into Pin 8 (which is the RX on the nRF52)
* GND - the GND wire (black) of the cable goes into a ground pin on the nRF52.

### Connect the nrF52 board to the computer via USB-microUSB cable

## Steps to build

### 1.  Create 2 targets - Bootloader and Sensor application


```
$ newt target create nrf52_boot  
$ newt target set nrf52_boot bsp=@apache-mynewt-core/hw/bsp/nrf52dk  
$ newt target set nrf52_boot build_profile=optimized  
$ newt target set nrf52_boot app=@apache-mynewt-core/apps/boot  

$ newt target create nrf52_sensor  
$ newt target set nrf52_sensor bsp=@apache-mynewt-core/hw/bsp/nrf52dk  
$ newt target set nrf52_sensor build_profile=debug  
$ newt target set nrf52_sensor app=@apache-mynewt-core/apps/sensors_test  
```

```
$ newt target show
targets/nrf52_boot
    app=@apache-mynewt-core/apps/boot
    bsp=@apache-mynewt-core/hw/bsp/nrf52dk
    build_profile=optimized
targets/nrf52_sensor
    app=@apache-mynewt-core/apps/sensors_test
    bsp=@apache-mynewt-core/hw/bsp/nrf52dk
    build_profile=debug
``` 
  
### 2.  To enable I2C communications, the sensor target needs to be customized.  You can use the newt tool to override the defaults with custom values, or you could directly make the changes in the syscfg.yml file (both options shown below).

#### Option 1:

```
$ newt target set nrf52_sensor syscfg=BNO055_I2CBUS=0:I2C_0=1
```

BNO055_I2CBUS=0 assigns the number 0 to the I2C interface # (the # in I2C_#)
I2C_0=1 enables the above defined I2C_0

#### Option 2:
Edit the file “targets/nrf52_sensor/syscfg.yml” with the following settings.

syscfg.vals:
# Assign 0 to I2C interface #
   BNO055_I2CBUS: 0
   
# Enables the above defined I2C_0   
   I2C_0: 1

 To make sure your target looks good, do a "newt target show" as shown below

```
$ newt target show
targets/nrf52_boot
    app=@apache-mynewt-core/apps/boot
    bsp=@apache-mynewt-core/hw/bsp/nrf52dk
    build_profile=optimized
targets/nrf52_sensor
    app=@apache-mynewt-core/apps/sensors_test
    bsp=@apache-mynewt-core/hw/bsp/nrf52dk
    build_profile=debug
    syscfg=BNO055_I2CBUS=0:I2C_0=1
```

### 3. Build the two executibles

```
$ newt build nrf52_boot
Building target targets/nrf52_boot
Compiling repos/apache-mynewt-core/boot/bootutil/src/image_ec256.c
Compiling repos/apache-mynewt-core/boot/bootutil/src/image_ec.c
Compiling repos/apache-mynewt-core/boot/bootutil/src/image_rsa.c
<snip>
Linking ~/bin/targets/nrf52_boot/app/apps/boot/boot.elf
Target successfully built: targets/nrf52_boot
```

```
$ newt build nrf52_sensor
Building target targets/nrf52_sensor
Compiling repos/apache-mynewt-core/boot/bootutil/src/image_rsa.c
Compiling repos/apache-mynewt-core/boot/bootutil/src/image_ec.c
<snip>
Linking ~/bin/targets/nrf52_sensor/app/apps/sensors_test/sensors_test.elf
Target successfully built: targets/nrf52_sensor
```

### 4. Sign the application image

```
$ newt create-image nrf52_sensor 1.0
Compiling bin/targets/nrf52_sensor/generated/src/nrf52_sensor-sysinit-app.c
Archiving nrf52_sensor-sysinit-app.a
Linking ~/bin/targets/nrf52_sensor/app/apps/sensors_test/sensors_test.elf
App image succesfully generated: ~/bin/targets/nrf52_sensor/app/apps/sensors_test/sensors_test.img
```

### 5. Prepare flash for images by erasing what's in the flash first 

```
$ JLinkExe -device nRF52 -speed 4000 -if SWD
SEGGER J-Link Commander V6.14c (Compiled Mar 31 2017 17:42:24)
DLL version V6.14c, compiled Mar 31 2017 17:42:10
<snip>

J-Link>

J-Link>erase

<snip>

Erasing done.
J-Link>

```
### 6. Load the two images, nRF52_boot and nRF52_sensor

```
$ newt load nrf52_boot
Loading bootloader

$ newt load nrf52_sensor
Loading app image into slot 1
```

## The LED should be blinking now.

### 7. Talking to the sensor :  for this, we use Minicom (any other serial terminal emulator would work).

#### (a). Find the right tty - tty.usbserial-xxxxxxxx  

```
$ ls /dev/tty*
/dev/tty				
<snip>	
/dev/tty.usbmodem1411			
/dev/tty.usbserial-FTZ6XVPF		
/dev/ttyp0				
<snip>
```

#### (b) start minicom with the right tty

```
$ minicom -D /dev/tty.usbserial-FTZ6XVPF -b 115200

Welcome to minicom 2.7

OPTIONS: 
Compiled on Nov 24 2015, 16:14:21.
Port /dev/tty.usbserial-FTZ6XVPF, 16:41:00
Press Meta-Z for help on special keys
```

#### (c) communicate with the sensor.

For list of commands, type "?" in the terminal

```
——Minicom window———

?
3352:Commands:
3352:     stat    config       log      echo         ?    prompt 
3354:    ticks     tasks  mempools      date    sensor     flash 
3356: tcs34725    bno055 


```
```
sensor list
11561:sensor dev = accel1, type = 0x1 0x2 0x4 0x10 0x200 0x1000 0x2000 0x4000 
```
```
sensor type accel1
15963:sensor dev = accel1, 
type =
15963:    accelerometer: 0x1
15964:    magnetic field: 0x2
15965:    gyroscope: 0x4
15966:    temperature: 0x10
15966:    vector: 0x200                                                         
15967:    accel: 0x1000                                                         
15968:    gravity: 0x2000                                                       
15968:    euler: 0x4000     
```
```
sensor read accel1 0x1 -n 5                                                     
21458:ts: [ secs: 167 usecs: 646308 cputime: 168142055 ]                        
21460:x = -0.510000000 y = -7.559999936 z = 6.050000192                         
21461:ts: [ secs: 167 usecs: 666524 cputime: 168162271 ]                        
21462:x = -0.510000000 y = -7.519999968 z = 6.010000229                         
21463:ts: [ secs: 167 usecs: 682355 cputime: 168178102 ]                        
21464:x = -0.560000000 y = -7.550000192 z = 6.030000210                         
21465:ts: [ secs: 167 usecs: 697980 cputime: 168193727 ]                        
21466:x = -0.540000000 y = -7.530000224 z = 6.000000000                         
21467:ts: [ secs: 167 usecs: 713527 cputime: 168209274 ]                        
21468:x = -0.460000000 y = -7.539999936 z = 5.989999744                         
 ``` 
  
 ``` 
                                                                        
sensor read accel1 0x4 -n 5                                                     
24672:ts: [ secs: 192 usecs: 751160 cputime: 193246907 ]                        
24673:ts: [ secs: 192 usecs: 760754 cputime: 193256501 ]                        
24674:ts: [ secs: 192 usecs: 768566 cputime: 193264313 ]                        
24675:ts: [ secs: 192 usecs: 776379 cputime: 193272126 ]                        
24676:ts: [ secs: 192 usecs: 784192 cputime: 193279939 ]                        
```




