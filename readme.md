# Pightstand
## A raspberry-pi based nightstand clock

### Summary

A nightstand clock as a programming project.

![image of running device](img/IMG_20181117_190132.jpg)
[Link to picture of running device](https://photos.app.goo.gl/F47QVzqR2oV6EbBj6)


### Details

#### Hardware

| Item | Note | Link |
| --- | --- | --- |
| raspberry pi | model B+ (40-pin) | [current](https://www.adafruit.com/product/3775) |
| PaPiRus Screen HAT | 2.7" screen, 4 buttons | [US](https://www.adafruit.com/product/3420) [UK](https://uk.pi-supply.com/products/papirus-epaper-eink-screen-hat-for-raspberry-pi) |
| microSD card | 16GB SanDisk | |
| USB WiFi adapter | Raspberry Pi foundation version | |
| USB power supply | 5V 2.5A recommended | |


#### Software

| Libarary | Link |
| --- | --- |
| Papirus | [GitHub](https://github.com/PiSupply/PaPiRus/tree/master/hardware/PaPiRus%20HAT) |
| Python Imaging Library | [PIL](https://pillow.readthedocs.io/en/stable/) |
| Python API for WeMo | [pywemo](https://github.com/pavoni/pywemo) |
| Python client library for [3M50 Radio Thermostat](http://www.radiothermostat.com/filtrete/products/3M-50/) | [github](https://github.com/mhrivnak/radiotherm) |



### Notes

#### General Principles

+  use native python distribution by default
+  use in-memory filesystem (`/dev/shm`) for temporary files to prevent wear of the SD card
+  use JSON to format data at rest
+  create separate programs for each task (_a la_ microservices)


#### papirus-temp

This is a shell script that reads the on-board temperature sensor.  This provides the board temperature, which is regularly higher than the room temperature, and not much use to this application.  This is also [available in the vendor repo](https://github.com/PiSupply/PaPiRus/blob/master/bin/papirus-temp.sh), although it's been replaced by a python script.

#### papiswitch

This is a python program that monitors the buttons on the PaPiRus Screen HAT:

+  checks a PID file to make sure it's not already running
+  event loop waits for keypress
+  keypress is (de-bounced)[https://www.arduino.cc/en/tutorial/debounce]
+  first detected (WeMo plug)[https://www.belkin.com/us/p/P-F7C063/] is toggled
+  quits on error

A cron job relaunches this every minute.  Sometimes it fails to toggle the light, and I haven't started to debug it.

#### papirus-image

This is a python program that loads the image on the e-ink display once it is generated by `papigen`, below.

#### papigen

This is the core, pulling together various data and formatting the display.  This is also executed via cron every minute.

+  screen gets full refresh once an hour
  +  this causes the screen to flash, which I've decided is desirable
+  polls my thermostat for current house temperature 
+  polls [openweathermap](https://openweathermap.org/api) every twenty minutes
+  executes the GNU [cal](https://www.gnu.org/software/gcal/) utility to generate current month calendar
+  creates bitmap image for loading to display

#### cron

```
# m h  dom mon dow   command
* * * * * [ -x bin/papigen ] && bin/papigen && bin/papirus-image >/dev/null 2>&1 
* * * * * [ -x bin/papiswitch ] && bin/papiswitch > /dev/null 2>&1 &
```



### History

| Date | Note |
| :---: | --- |
| 9 Feb 2016 | ordered papirus from pisupply |
| 20 Mar 2016 | submitted merge request for `papirus-temp` to PiSupply |
| 17 Nov 2018 | started nightstand clock idea instead of shoveling snow; initial commit |
| 15 Dec 2018 | pressing button toggles wemo lightswitch |
| 4 May 2019 | sanitized, commited to public repository |




