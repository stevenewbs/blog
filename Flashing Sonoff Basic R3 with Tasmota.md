# Flashing a Sonoff Basic R3 with Tasmota

### Quick and Dirty howto

This is my "I just want to get this done" guide as of August 2020 (lots of the other guides are a bit outdataed and the Tasmota firmware main binary is now too big to flash initially). 
eWelink firmware 3.5+ changed the process of getting into Diy Mode and connecting to your wifi (see [here](https://github.com/itead/Sonoff_Devices_DIY_Tools/issues/75#issuecomment-609036245)).


## Useful things to have open

* Sonof API Docs - https://github.com/itead/Sonoff_Devices_DIY_Tools/blob/master/SONOFF%20DIY%20MODE%20Protocol%20Doc%20v2.0%20Doc.pdf
* Tasmota Manual flash guide - https://tasmota.github.io/docs/Sonoff-DIY/#manual-flash
* Tasmota binary releases - http://thehackbox.org/tasmota/release/
* Sonoff Basic R3 Tasmota template - https://templates.blakadder.com/sonoff_basic_R3.html

Get all these open ahead of time just for ease. 

## Process

* Update the Basic R3 using eWelink app - must be > 3.1 (install app, pulg device into power, quick pair, update, delete device (resets wifi settings))
* Open up the BR3 and put the jumper on the "OTA" pins. Jumper is included in the little bag of screws. 
* Power up the BR3 and long press (5s) the power button - blue led should short flash continuously
* You should now see an ITEAD-XXX access point available. Connect to it (password is 12345678) and open http://10.10.7.1.
* Enter the wifi network details. 
* Find the IP the device was given (easiest on the router DHCP table).
* Grab the tasmota-lite.bin from the tasmota downloads page and get its sha256 sum. Put the file on the web server, available on the same network. 
* Follow the zeroconf commands as per the [Tasmota docs](https://tasmota.github.io/docs/Sonoff-DIY/#flash-the-firmware-and-confirm):
  - Info to make sure you have the right device.
  - Unlock OTA 
  - Flash - point to the file on th web server. 
  - Wait ....
* Usual tasmota-xxx AP should now be available - normal process to hook it up to your wifi:
  - Connect
  - Enter wifi details
  - Configure the template
* Finally update Firmware to normal tasmota binary - there seems to be a size limitation in the eWelink firmware for updates. Tasmota will happily update to a file bigger than this limit. 
* Remember to `SetOption19 on` AND configure MQTT server (homeassistant is happy).
