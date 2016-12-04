# MyMultisensors

MyMultisensors is a compact multi sensors capable node for your/my home automation.
Optmized for very low power, ambiant sensors node, battery powered and wireless, i can easily add in each room. 

<img src="https://raw.githubusercontent.com/scalz/MyMultisensors/master/Img/MyMultisensors_top.jpg" alt="MyMultisensors top"> 

<img src="https://raw.githubusercontent.com/scalz/MyMultisensors/master/Img/MyMultisensors_bottom.jpg" alt="MyMultisensors bottom"> 

I have to say, this is one of my favorite board! well, you think i'm biased :) a bit true ;)
But the truth is i've never seen the same board before (with my requirements) or i wouldn't have made it!

I put multiple "constraints" on myself to get the best of what i can do myself imho.
- Most common sensors used indoor : Temperature/Humidity/Motion/Ambiant Light/Door Contact. Using the best low power sensors i know.
- Entry level MCU so noobs can easily play with it (well known 328p)
- Tiny so i can reuse the board if i need a simple motion or door contact device. One pcb allows multiple devices. Standardisation of main ambiant indoor devices with box.
- Thin, like a simple temperature/humidity tag. So coincell 2032. Depending on the scenario, I need to be able to change the battery holder for CR2032/CR2450/AAA.
- Very very low power. 2years min with a simple good brand CR2032 (depending of usecase). Choosing the right parts for low power.
- Mysensors Wireless programming and Signing/Authentication features are mandatory for a device which can do some security tasks like Motion detection.
- RFM69 and its nice range even with the low power RFM69CW version onboard. I'm finalizing the nrf24 version because it's a nice board, that would be too bad for nrf24 user ;)
- I also tried to take care of the routing; and to have strong, unbroken GND plane as it is very important to get best range, especially with monopole wire I'm using (it's also possible to use a planar pcb ant though) 
- and not the least constraint! I've tried to keep cost reasonable regarding if i would buy it, or people etc..

As everything is customizable, the choice of the battery or the custom pcb size (compact), the easy and quite common way is printing a custom box..

I think i've "almost" achieved my goals :)

<img src="https://raw.githubusercontent.com/scalz/MyMultisensors/master/Img/schematic.png" alt="schematic"> 

We can see that it's possible to use different kind of battery, and different antennas (wire or planar pcb).

###General specs
------

<img src="https://raw.githubusercontent.com/scalz/MyMultisensors/master/Img/MyMultisensors_batteries.jpg" alt="MyMultisensors batteries"> 

|Specs|  |
|---|---|---|
|Size |49*25.4 (mm) |
| MCU | Atmel328p TQFP 150nA deep sleep|
| Radio | RFM69CW 100nA in deep sleep, NRF24 is coming |
| Authentication security | ATSHA204A chip footprint 150nA in deep sleep|
| OverTheAir Wireless programming | SPI flash eeprom footprint 200nA in deep sleep |
| Temperature/Humidity | SI7021 60nA in deep sleep |
| Ambiant Light Lux | OPT3001 300nA in deep sleep, with footprint for optional pullup resistor |
| Motion detection | LHI968 good quality/price, with its circuit, active 1.6uA |
| Door contact | footprint for soldering an optional pullup resistor >1M | 
| Indicator Led | Yes, on D9 |
| Extension connector | FTDI + D3 for connecting Door contact | 
| Pogo pins | AVRSPI programming |
| Battery connectors | CR2032 240mAh, CR2450 500mAh, 2xAAA 1200mAh |
| Power Supply | Reverse-polarity protection. 3V3 max |

####Onboard safety

Onboard, there is a reverse polarity protection. But for power savings, better efficiency etc .. there is no voltage regulator. So it's possible to use others batteries, or ftdi of course, but the max input voltage is 3.3V max.

####Extension Connectors

For simplicity, and trying to boost my assembling process (i use reflow), i'm using pogopins. I've done a simple adapter like this..
- AVRSPI : for burning the bootloader. Here i'm using the Sensebender Micro Atmel 328p Internal 8Mhz BOD 1.8V, in Boards Manager/Mysensors boards
- FTDI : for uploading the sketch. Without an optimized sketch, no big power savings. it's hw/sw.. still work in progress, i'll never be happy :)

####Power consumption, what can we expect ?

Power consumption has been checked with uCurrent Gold + old voltcraft vc506..

Measurement done with PIR enabled, sensors sleeping but without spiflash ic nor Door contact

| Mode | Varta coincell CR2032 | Duracell 2xAAA |
|---|---|---|---|
|MCU in deep sleep, interrupt on change |4uA | 3uA|
|MCU in deep sleep, interrupt on change Wdt Enabled | 8uA | 11uA|
|MCU running 1Mhz | 732uA | 737uA|

Few simple maths because battery lifetime can change for multiple reasons (battery brand, internal resistance of coincell etc.). I won't explain everything here.

Still important to remember, is we can't expect to have the complete 240mAh of a coincell. Better bet on 200mAh, for a good quality coincell. For bad quality, it can be worse! Bad clone coincell can have only 50% of usable energy! Try clone and good brands, you will see what i mean ;) it's up to you if you want to spend your time to replace batteries :)

As RFM69 radio power consumption is low during sleep mode, no need to disable it, so it allows to use listenmode and wakeup from mcu from deep sleep for instance.

Say a power consumption of 5uA during sleep mode, the PIR is enabled of course, but not triggered (note it is possible to disable the PIR motion sensor to save power).
And our RFM69 needs 30mA during 40ms. Rough estimation!  it does not spend this whole 40ms transmitting of course! And with automatic RSSI and power level control, the radio can consumes less power in TX.
On other hand, it does not take in account the signing time if feature is used.

So 30mA for 40ms, say 10 motions detection/hour, give us a theoretical estimation of 2.33years.
Not so bad..but the node can last a lot more if the software is well optimized and flexible to let user to set dynamically during days, the sensibility and motion detection settings etc..

####Arduino Pin description

|Arduino sketch define| Arduino Pin | description |
|---|---|---|
| MY_OTA_FLASH_SS | 8 | EEprom CS pin |
| ATSHA204_PIN | 17 | ATSHA204A SOT23 pin (for authentication) |
| PIR_INT_PINH  | 6 | PIR motion sensor pin H |
| PIR_INT_PINL | 7 | PIR motion sensor pin L |
| MYNODE_LED_PIN | 9 | Status led pin |
| REED_SWITCH_PIN | 3 | Reed switch pin interrupt on int1 D3 |
| AMBIANT_LIGHT_INT | A1  | Ambiant light sensor pin interrupt |

### Some ideas for the Mysensors sketch
------

|General Node settings | Default Settings | Notes|
|---|---|---|---|
| MY_NODE_NAME | MYS_MOTION| Node/sketch name|
|MY_DEBUG|1.0| General Level Mysensors Debug messages. Comment it to save memory!|
|MY_DEBUG_SKETCH|1.0|Sketch Level Debug messages. Comment it to save memory!|
|MY_NODE_ID|AUTO| AUTO or set a fixed ID|
|PARENT_NODE_ID|| Uncomment it and set your parent node id if needed|
|RELEASE |1.0| Sketch version|
|MY_BAUD_RATE|38400||
|isMetric |true| true/false|
|//atsha204Class sha204(sha204Pin);||Uncomment to enable signing|
|//MY_OTA_FIRMWARE_FEATURE||Uncomment to enable OTA wireless programming|
|MY_RFM69_NETWORKID|100| Uncomment and set your preferred network id|
|MY_RFM69_ATC_TARGET_RSSI_DBM|-80| Uncomment and set your preferred RSSI target|
|LIGHT_H_LIMIT|100|High Threshold for Ambiant light sensor|
|LIGHT_L_LIMIT|20|Low Threshold for Ambiant light sensor|
|LIGHT_TRANSMIT_THRESHOLD|50| Delta needed for updating Ambiant sensor to controller|
|HUMI_TRANSMIT_THRESHOLD|5|Delta needed for updating Humidity sensor to controller|
|TEMP_TRANSMIT_THRESHOLD|1|Delta needed for updating Temperature sensor to controller|
|FORCE_TRANSMIT_INTERVAL|10| Intervalle time between transmit update|
|BATTERY_USB| |Uncomment if using ftdi, for batt voltage reading|
|BATTERY_VARTA_2032| |Uncomment if using CR2032, for batt voltage reading|
|BATTERY_DURAC_2450| |Uncomment if using CR2450, for batt voltage reading|
|BATTERY_DURAC_IND_2AAA| |Uncomment if using 2xAAA, for batt voltage reading|

|MyPirHelper.h | Default Settings | Notes |
|---|---|---|---|
| PIR_SETTLE_TIME | 20000| in milliseconds/ better 30000. Waiting for PIR settling at node init|
| PIR_DEFAULT_PULSES | 8| Programmable pulse counter : 1... 255 pulses but for 255 pulses you'll need to dance for a nice motion :)|
| PIR_DEFAULT_WINDOWTIME | 1| Window time : 0 to 3.  Range: 8s... 16s.  Time in sec = 8 * (value+1)|
| PIR_DEFAULT_CANCELTIME | 2| Cancel Time : 0 to 10. Range: 8s... 80s.  Time = 8 * (value+1)|
| PIR_DEFAULT_BLINDTIME | 2| Blind Time  : 0 to 15. Range: 0s... 120s. Time = 8 * value|

All these infos can be found in the source files.


####External Libraries needed
- Opt3001      : https://github.com/node-it/EnvironOne/blob/master/src/Opt3001.h
- SI7021       : from Marcus Sorensen <marcus@electron14.com> 

####From the controller view

These are the CHILD_IDs used for sensors, or for dynamic settings.

|CHILD NODE ID/CMD | Default ID | Notes|
|---|---|---|---|
| CHILD_ID_PIR | 1|PIR sensor |
| CHILD_ID_PIR_EN | 2|Enable PIR |
| CHILD_ID_TEMP  | 3|Temperature sensor|
| CHILD_ID_HUM | 4|Humidity Child node ID|
| CHILD_ID_LIGHT  | 5|Ambiant light Child node ID|
| CHILD_ID_DOOR | 6|Door reed switch|
| CMID_FULL   | 10|Command from controller with all params below in one tx. below is for debug..|
| CMID_PULSES   | 11|Command from controller to set PIR number of pulse for windows time  |
| CMID_WINDOWTIME   | 12|Command from controller to set PIR window time  |
| CMID_BLINDTIME | 13|Command from controller to set PIR blind time|
| CMID_CANCELTIME | 14|Command from controller to set PIR cancel time|
| CMID_EN_LIGHT | 15|Command from controller to enable Light sensor threshold|
| CMID_LIGHT_MODE | 16|Command from controller to set Light sensor special mode (enable pir at certain light level only...)|
| CMID_LIGHT_H  | 17|Command from controller to set Light sensor high threshold |
| CMID_LIGHT_L | 18|Command from controller to set Light sensor low threshold |
| CMID_LIGHT_DELTA | 19|Command from controller to set difference of lux for update controller |
| CMID_TRANSMIT_INTERVAL | 20|Command from controller to set time between sensor updates|
| CMID_TH_DELTA | 21|Command from controller to set difference of temp/hum needed for update controller |
| CMID_EN_LED  | 22|Command from controller to set led|

###Known issues
------ 

###TODO
------
- improve sketch

###Donations
------

I'm trying to make opensource projects. I do this for free and sharing spirit. I don't do ads etc..
But if you think information here is worth of some money, or want to reward me, feel free to send any amount through paypal.

[![](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=PWVDL2P64FDVU)  

Or you can also order pcb here :
I will earn a little percentage that will allow me to order proto pcb and share more fun design.

Or pay me a protein smoothie if you see me! oh well, a beer is ok too :)

### Contributors
------
Always special thanks to:
- Mysensors Team for its great work
- Adafruit, Sparkfun, TI, Atmel etc.. for all educational infos they share

###Links, reference and license** 
------
- https://www.openhardware.io/view/75/MyMultisensors
- http://www2.st.com/content/ccc/resource/technical/document/application_note/b8/84/29/41/21/00/44/41/DM00096551.pdf/files/DM00096551.pdf/jcr:content/translations/en.DM00096551.pdf
- http://www.ti.com/lit/ug/tiduau1a/tiduau1a.pdf

Copyright Scalz (2016). released under the CERN Open Hardware Licence v1.2
