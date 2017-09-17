# HOWTO configure, compile, wire, print and assemble the induino meteostation with printable hardware
------------------------

## Summary
The induino MeteoStation is a complete weather station with arduino firmware, indilib driver and a webinterface with current status and graphs that range back in time from 3 hours to one month.
The indilib driver is capable of issuing alerts, like daytime, clouded, frezzing and more. Indi clients will react to alerts and can be configured to close your observatory and stop the imaging session in unsafe conditions.
MeteoStation does not have a wind or rain sensor, but can be used in combination with wunderground weather trough Meta weather for an extended weather service. There is also seldom rain without the detection of clouds.

This howto will show you how to setup and build the induino meteostation.
I will use the Adafruit Trinket Pro 5v, but other compatible arduinos could be used.
Keep in mind that the SDA and SDL ports are used for the Ir and pressure sensor. Digital pin 3 is used for the humidity sensor and analog 0 for the ir radiance sensor.
For the Trinket pro we will also use ground, 5v, RX and TX of the FTDI header, as the Trinket does not support serial console over the USB port.

Meteostation can be used as a 'standalone' weather station for any none observatory use. This could be a raspberry pi or almost any other linux box running apache with indilib installed locally.

## Table of content
1. [Short on configuring the Arduino IDE](#arduino)
2. [Soddering and connecting the sensors](#soddering)
3. [Configuring, compiling, uploading and debuuging meteoTEST](#meteotest)
4. [Configuring, compiling and uploading induinoMETEO](#induinometeo)
5. [Connecting induinoMeteo to KStars with indi_duino driver](#indiduino)
6. [Connecting meteostationWEB to induinoMETEO, and serving trough Apache](#meteostationweb)
7. [Printing and assembling the MeteoStation printable hardware](#meteostationprint)
8. [Links to sensors, printable hardware and resources](#usefullinks)

<a name="arduino"></a>
## Short on configuring the Arduino IDE
First you must set up the Arduino IDE with the Adafruit Pro Trinket board configuration.
Follow the steps on Adafruit's web page to add the boards.

See Adafruit's [IDE Setup](https://learn.adafruit.com/adafruit-arduino-ide-setup/arduino-1-dot-6-x-ide) for 'Additional Boards Manager URL'

![arduino-board](media/arduinoTRINKET/1-select_board.png) 

If you are using a FTDI cable with no software reset, you can skip adding the port.
When you want to upload the firmware, start off by compiling the sceatch [Ctrl + R] then connect the USB to FTDI dongle to your pc and hit [Ctrl + U] to upload.

The IDE will autoselect the /dev/ttyUSBX port.

![no-port](media/arduinoTRINKET/2-no_port.png)

If the autodetect fails, then connect the dongle, select port and short out the ground and reset pin on the FTDI header. This will reset / reboot the arduino and put it in upload mode.
 

<a name="soddering"></a>
## Soddering and connecting the sensors
The pins being used is marked out with red centers.
- A0 (Analog 0) is IR Radiance input
- SDA and SCL (A4 and A5) is used for MLX90614 (IR) sensor and BMP 180 (pressure) sensor
- 3 (Digital 3) is used for the DHT22 (humidity) sensor
- The FTDI header is used for serial to USB connection

The pins with green centers are the FTDI reset pins. This will reboot board and put it in upload mode.

![pinout](media/arduinoTRINKET/3-pinout.png)

If you are building this device with the hardware used in this tutorial, you will not need to change any pinnnumbers in the firmware. For links to the hardware being used, see the last section of this tutorial.

When setting up for the first time I woud ideally just sodder the pins on the underside of the arduino, place it on a breakout board and have easy access to test the sensors. I ended up by soddering on some of the pins and using wires with connectors. Before completing the build I did have to remove the pins I added for gnd and +5v and sodder the wires, as it was using to mutch space.

![soddering](media/arduinoTRINKET/4-soddering.png)

![connecting](media/arduinoTRINKET/5-connecting.png)


<a name="meteotest"></a>
## Configuring, compiling, uploading and debuuging meteoTEST
Start out by not changing anything in the firmware, compile and upload, then open the Serial Monitor [Ctrl + Shift + M]

If you see a output like this, the you are good to go. Double check that the sensor values change when you subjet them to temperature, light and humidity changes.

![define-sensors](media/induinoMETEOTEST/3-serial_monitor.png)

If you see no output at all, then the firmware might have crashed before you connected the serial monitor.
To verify that you have compiled and uploaded the firmware correctly, then comment out all the `#define USE_*_Sensor` and upload again. If you see a output in the monitor with (X sensor skipped, not defined, ending with a RESULT with all 0 values), then you can start adding the sensors back, until you see witch fails.
When the failing sensor is located, then it's just a matter of checking wiring and pinouts.

![define-sensors](media/induinoMETEOTEST/1-define_sensors.png)

If you are not using the default pins, then they can be changed in the firmware.

![define-sensors](media/induinoMETEOTEST/2-define_PINS.png)

You do not need to edit anything else in the firmware!

<a name="induinometeo"></a>
## Configuring, compiling and uploading induinoMETEO
The compiling and upload part of this is the same as for the meteoTEST firmware, but there are more configuration that can be done.

Let's start off with the sensors used. I do recomend that if you are for some reason not using a sensor, then comment out the definition of it.

![define-use](media/induinoMETEO/1-define_USE.png)

Below 0 celcius is of cource considered frezzing, but you can change it.
I'm imaging in Norway and most of the time during winter, the temperatures will be below 0 degrees. When the firmware detects temperatures below the set value, then the frezzing pin will be set to true. As this is an alert in the indi_duino meteo driver, then clients will react to unsafe eather conditions. I will set mine to -10, but you know best what you need.

![define-frezzing](media/induinoMETEO/2-define_FREZZING.png)

Next is to select witch sensor we should use for ambient temperature. In my installation i found that the BMP 180 gave the best result. You should in most cases be good by leaving this as it is.

![define-t](media/induinoMETEO/3-define_T.png)

Using a different pin for DHT22? Then update here.

![define-dhtpin](media/induinoMETEO/4-define_DHTPIN.png)

As there are many options of small and low power colar cells, then it is not possible to know what reading yours will have when the sun is setting. To get the correct value for this, then simply check the readout of the irradiance sensor, just after the sun sets under the horizon. This will give you a Daytime alert in the meteo driver, so you know not to open your observatory roof (and or dustcap).

![define-daylight](media/induinoMETEO/5-define_DAYLIGHT.png)

Using a different pin for Ir radiance? Update accordingly.

![define-irradiancepin](media/induinoMETEO/6-define_IRRADIANCEPIN.png)

If you are either reading a cloud cover when there are no clouds visible to the sensor, or you have a full overcast with the sensor reporting less than 100% clods, then you can edit this.

![define-cloud](media/induinoMETEO/7-define_CLOUD.png)

Depending on your use, you might want a clouded warning when it's more than 15% clods, or you might be happy with less than 50%? Edit this value to customize the alert (cloud flag).

![define-cloud-pnt](media/induinoMETEO/8-define_CLOUD_PCNT.png)


<a name="indiduino"></a>
## Connecting induinoMeteo to KStars with indi_duino driver
You can start the driver locally as any other driver trough Ekos by selecting 'Arduino MeteoStation'

If you want to start the driver on localhost you will need to create a FIFO file and select the correct skeleton file.
This oneliner should get you started.
```
killall indiserver; rm /tmp/INDIFIFO; mkfifo /tmp/INDIFIFO; indiserver -f /tmp/INDIFIFO & echo start indi_duino -n \"MeteoStation\" -s \"/usr/local/share/indi/meteostation_sk.xml\" >/tmp/INDIFIFO
```
It first stops indiserver. Removes any old FIFO file. Creates a new one. Starts indiserver with the FIFO file and the indi_duino driver as MeteoStation with meteostation_ks.xml skeleton file.

After starting select the correct port in 'Connection' tab.

![indiduino-connect](media/indiDUINO/1-indiduino_connect.png)

Double check that all sensors report a value in the 'Raw Sensors' tab, and the METEO tab should show you the current weather state.

![indiduino-raw](media/indiDUINO/2-indiduino_raw.png)

![indiduino-meteo](media/indiDUINO/3-indiduino_meteo.png)



<a name="meteostationweb"></a>
## Connecting meteostationWEB to induinoMETEO, and serving trough Apache
This section assumes you have the basic Apache installation up.
Here are some Apache2 basics on help.ubuntu [HTTPD - Apache2](https://help.ubuntu.com/lts/serverguide/httpd.html)
You do not need any ssl, as there is no login on the meteostationWEB page.

I will also assume that you are on a distro with python installed.

To get meteostationWEB up from here, then.

__Install dependencies__

`sudo apt-get install python-rrdtool python-simplejson python-utidylib`

__Copy the meteostationWEB folder to your home directory.__

`cp -r indi/3rdparty/indi-duino/add-on/meteostationWEB ~/`

__Create CHART folder and set permission__

```
mkdir ~/meteostationWEB/html/CHART
chmod 775 ~/meteostationWEB/html/CHART
```

__Create symlink to html directory. This assumes default apache root directory = /var/www, it could also be /var/www/html__

```bash
# Do not use ~/meteostationWEB or $HOME/meteostationWEB the symlink needs fullpath
sudo ln -s /home/you/meteostationWEB/html /var/www/meteo
sudo chown -R [you]:www-data ~/meteostationWEB/html
```

__Reload apache__

```
sudo systemctl reload apache2.service
```

You should now edit the file ~/meteostationWEB/meteoconfig.py and set your defaults and indi connection options.

**Connection**

For connecting to, and starting indiserver on localhost, there are no need to change connection settings.

```bash
#1). Local
#by defining INDISERVER as localhost,
#and leaving INDITUNNEL="false",
#then indiserver will be started locally on port INDIPORT
INDISERVER="localhost"
INDITUNNEL="false"
INDISTARTREMOTE="false"
INDIPORT="7624"
```

To connect to indiserver running on a different host on the same network, then edit #2). Remote, and comment out #1). Local
(You should allways comment out the configs not used).

```bash
#2). Remote
#by defining INDISERVER with hostname,
#and leaving INDITUNNEL="false",
#meteostationWEB will connect to indiserver at INDISERVER:INDIPORT
INDISERVER="192.168.10.123"
INDITUNNEL="false"
INDISTARTREMOTE="false"
INDIPORT="7624" # Edit this if indiserver has been started on non default port
```

To start and connect to indiserver on any remote host, with ssh passwordless keys and up to date indi.

```bash
#3). Tunnel with indistartup
#by defining INDISERVER as localhost,
#and setting INDITUNNEL="true" and INDISTARTREMOTE="true",
#then meteostationWEB will open a ssh connection to SSHSERVER:SSHPORT,
#and start indiserver on remote machine on INDIREMOTEPORT,
#and tunnel indiserver to INDISERVER:INDIPORT
INDISERVER="localhost"     # You should not change this
INDITUNNEL="true"
INDIPORT="7624"            # The port that will be used for indiserver on localhost
INDIREMOTEPORT="7624"      # The port indiserver is running on the remote host
INDISTARTREMOTE="true"
SSHSERVER="indiserver.mydomain.com"
SSHPORT="22"
```

To connect to indiserver running on any other host with, ssh passwordless keys enabled.
Mostly the same settings as #3).

```bash
#4). Tunnel with allready started indiserver
#by defining INDISERVER as localhost,
#and setting INDITUNNEL="true" and INDISTARTREMOTE="false",
#then meteostationWEB will open a ssh connection to SSHSERVER:SSHPORT,
#and tunnel indiserver on remote machine running on INDIREMOTEPORT to INDISERVER:INDIPORT
INDISERVER="localhost"
INDITUNNEL="true"
INDIPORT="7624"
INDIREMOTEPORT="7624"
INDISTARTREMOTE="false"
SSHSERVER="indiserver.mydomain.com"
SSHPORT="22"
```

IN INDI SETTING AND DEBUG you should set what the device port is

```bash
##### INDI SETTINGS AND DEBUG #####
#1). Basic indi
INDIDEVICE="MeteoStation"
INDIDEVICEPORT="/dev/ttyUSB0"
```

And finnaly for connection, you should set the path of your ssh key, and your ssh host username, (if you are using ssh).

```bash
##### SSH TUNNEL AN INDI EXEC #####
#Should only need to edit #1), and only if using ssh
#1). Key and user
SSHKEYDIR="~/.ssh/id_rsa"
SSHUSERNAME="magnus_e"
```

**Defaults**

You should atleast set the site related defaults. The RRD Related options is for the Advanced tab, to get expanded weather for your area.

```bash
##### SITE RELATED ####
OWNERNAME="Nacho Mas"
SITENAME="MADRID"
ALTITUDE=630
#Visit http://weather.uwyo.edu/upperair/sounding.html
#See the sounding location close your site
SOUNDINGSTATION="08221"

##### RRD RELATED #####
#PATH TO GRAPHs
CHARTPATH="./html/CHART/"
#EUMETSAT lastimagen. Choose one from:
#http://oiswww.eumetsat.org/IPPS/html/latestImages.html
#This is nice but only work at daylight time:
#EUMETSAT_LAST="http://oiswww.eumetsat.org/IPPS/html/latestImages/EUMETSAT_MSG_RGB-naturalcolor-westernEurope.jpg"
#This show rain
#EUMETSAT_LAST="http://oiswww.eumetsat.org/IPPS/html/latestImages/EUMETSAT_MSG_MPE-westernEurope.jpg"
#and this cloud cover at IR 39. Work at night
EUMETSAT_LAST="http://oiswww.eumetsat.org/IPPS/html/latestImages/EUMETSAT_MSG_IR039E-westernEurope.jpg"
```

**The meteostationWEB user interface**

The index shows the current METEO status.

![meteoweb-gauges](media/meteostationWEB/1-meteoweb_gauges.png)

Graps of METEO data, ranging from 3 hours to 1 month back.

![meteoweb-graphs](media/meteostationWEB/2-meteoweb_graphs.png)


<a name="meteostationprint"></a>
## Printing and assembling the MeteoStation printable hardware

![meteoprint-full](media/meteostationPRINT/1-MeteoStation-full.jpg)

Download the [3d printable MeteoStation on thingiverse](https://www.thingiverse.com/thing:2371144)

If you do not have a 3d printer, you can get the parts printet trough a service like [3D Hubs](https://www.3dhubs.com/).

Sumirizing the build steps.

- You need approximately 125m of PETG, 2.65m of TPU and 1.25m of PLA to print this housing.
- Where there are several versions for one part, allways print the one with highest version number.
- Use pictures as assembly guide. (More pictures on thingiverse)
- Sensors for temperature and humidity goes in the 'wind tunnel' betwean the bottom and middle part.
- IR Radiance sensor mounts under the acryllic glass, and the IR sensor in the opening of the top part.
- You need 2mm thick 9x5 cm acryllic perspex glass, that can be worked on with wood tools.
- Use the part MeteoStation-windowTemplate printed in PLA as a guide to cut the glass to size, and cut holes.

Screws used are all countersunk, and here is 4x 4x50mm + 6x 4x15mm

Finished product is close to 10 cm^3

![meteoprint-finishing-1](media/meteostationPRINT/2-finishing_1.jpg)

![meteoprint-finishing-2](media/meteostationPRINT/3-finishing_2.jpg)

![meteoprint-finishing-3](media/meteostationPRINT/4-finishing_3.jpg)

![meteoprint-done](media/meteostationPRINT/5-done.jpg)


<a name="usefullinks"></a>
## Links to sensors, printable hardware and resources
1. Adafruit
    * [Pro trinket - 5v 16MHz](https://www.adafruit.com/product/2000)
2. 3d prinatble box
    * [MeteoStation on thingiverse](https://www.thingiverse.com/thing:2371144)
3. 3d printing filament
    * [PETG - approx 125m](https://3dnet.no/collections/1-75/products/petg-1-75-1-0-kg?variant=9139275075)
    * [TPU  - approx 2.65m](https://3dnet.no/collections/1-75/products/tpe-1-75-0-5-kg?variant=1060433999)
    * [PLA - approx 1.25m](https://3dnet.no/collections/1-75/products/pla-1-75)
4. 3d printing service
    * [3D Hubs](https://www.3dhubs.com/)
5. Banggood sensors + misc
    * [Acrylic Perspex 2mm thickness](https://www.banggood.com/100x100mm-Acrylic-Perspex-Sheet-Cutting-Panel-Plastic-Satin-Gloss-for-Construction-p-1154629.html?rmmds=search)
    * [USB to TTL](https://www.banggood.com/USB-To-TTL-Debug-Serial-Port-Cable-For-Raspberry-Pi-3B-2B-COM-Port-p-1055396.html?rmmds=search)
    * [Solar panel max 25 x 55mm](https://www.banggood.com/Solar-panel-LED-Spike-Spot-Light-Landscape-Garden-Yard-Path-Lawn-Outdoor-Solar-Lamps-p-1148114.html?rmmds=detail-left-hotproducts)
    * [BMP 180](https://www.banggood.com/BMP180-Digital-Barometric-Pressure-Sensor-Module-Board-p-930690.html?rmmds=search)
    * [DHT 22 AM2302](https://www.banggood.com/Digital-DHT22-AM2302-Temperature-Humidity-Sensor-Module-For-Arduino-p-965081.html?rmmds=search)
    * [MLX90614ESF](https://www.banggood.com/MLX90614ESF-AAA-Non-contact-Human-Body-Infrared-IR-Temperature-Sensor-Module-For-Arduino-p-1100990.html?rmmds=search)
6. Recources
    * [Arduino IDE](https://www.arduino.cc/en/Main/Software)
    * [Pinouts](https://learn.adafruit.com/introducing-pro-trinket/pinouts)
    * [FTDI](https://learn.adafruit.com/introducing-pro-trinket/using-ftdi)
    * [IDE Setup](https://learn.adafruit.com/adafruit-arduino-ide-setup/arduino-1-dot-6-x-ide)
