# Growatt_ShineWiFi
Firmware replacement for Growatt ShineWiFi-S (serial).

# How to install

* Checkout this repo
* Setup the IDE of your choice
    * For the original Arduino IDE follow the instruction in the main ino [file](https://github.com/otti/Growatt_ShineWiFi-S/blob/master/SRC/ShineWiFi-ModBus/ShineWiFi-ModBus.ino) (esp8266 only)
* Rename and adapt [Config.h.example](https://github.com/otti/Growatt_ShineWiFi-S/blob/master/SRC/ShineWiFi-ModBus/Config.h.example) to Config.h with your compile time settings

After you obtained an image you want to flash:

* Flash to an esp32/esp8266 ([details](https://github.com/otti/Growatt_ShineWiFi-S/blob/master/Doc/)).
* Connect to the setup wifi called GrowattConfig (PW: growsolar) and configure the firmware via the webinterface at http://192.168.4.1
* If you need to reconfigure the stick later on you have to either press the ap button (configured in Config.h) or reset the stick twice within 10sec

# Flashing the Growatt ShineWiFi-S

Loading the firmware to the stick is rather easy:

You will need:
* Null modem cable (this is a serial cable that crosses the TX/RX lines)
* A USB to serial flasher - I used one with a CH340 chipset from eBay
* Something that can output 3.3v - I used an Arduino
* Bulldog clip and a crocodile clip

I soldered a connector onto the ShineStick to make some of the connections easier

1. Take the ShineStick out of the plastic case, there are some screws near the DSub connector
2. Connect the ShineStick to the null modem cable
3. Connect the null modem cable to your serial flasher
4. Connect a crocodile clip across GPIO0 and GND. Connect a crocodile clip to the outside DSub casing of ShineStick
5. Connect GND and 3.3v from your source to the ShineStick pins
6. Start your flashing tool
7. Try try try again when it doesn't work

## Features
Implemented Features:
* Built-in simple Webserver
* The inverter is queried using Modbus Protocol
* The data received will be transmitted by MQTT to a server of your choice.
* The data received is also provied as JSON
* Show a simple live graph visualization  (`http://<ip>`) with help from highcharts.com
* It supports convenient OTA firmware update (`http://<ip>/firmware`)
* It supports basic access to arbitrary modbus data
* It tries to autodected which stick type to use
* Wifi manager with own access point for initial configuration of Wifi and MQTT server (IP: 192.168.4.1, SSID: GrowattConfig, Pass: growsolar)
* Currently Growatt v1.20, v1.24 and v3.05 protocols are implemented and can be easily extended/changed to fit anyone's needs
* TLS support for esp32

Not supported:
* It does not make use the RTC or SPI Flash of these boards.
* It does not communicate to Growatt Cloud at all.

## Supported sticks/microcontrollers
* ShineWifi-S with a Growatt Inverter connected via serial (Modbus over RS232 with level shifter)
* ShineWifi-X with a Growatt Inverter connected via USB (USB-Serial Chip from Exar)
* Wemos-D1 with a Growatt Inverter connected via USB (USB-Serial Chip: CH340)
* NODEMCU V1 (ESP8266) with a Growatt Inverter connected via USB (USB-Serial Chip: CH340)
* ShineWifi-T (untested, please give feedback)
* Lolin32 (ESP32) with a Growatt Inverter connected via USB

I tested several ESP8266-boards with builtin USB-Serial converters so far only boards with CH340 do work (CP21XX and FTDI chips do not work). Almost all ESP8266 modules with added 9-Pin Serial port and level shifter should work with little soldering via Serial.

See the short descriptions to the devices (including some pictures) in the "Doc" directory.

## Supported Inverters
* Growatt 1000-3000S 
* Growatt MIC 600-3300TL-X (Protocol 124 via USB/Protocol 120 via Serial)
* Growatt MID 3-25ktl3-x (Protocol 124 via USB)
* Growatt MOD 3-15ktl3-x-xh (Protocol 120 via USB)
* Growatt MID 25-40ktl3-x (Protocol 120 via USB)
* Growatt SPH 4000-10000ktl3-x BH (Protocol 124 via Serial)
* And others ....

## Modbus Protocol Versions
The documentation from Growatt on the Modbus interface is avaliable, search for "Growatt PV Inverter Modbus RS485 RTU Protocol" on Google.

The older inverters apparently use Protocol v3.05 from year 2013.
The newer inverters apparently use protocol v1.05 from year 2018.
There is also a new protocol version v1.24 from 2020. (used with SPH4-10KTL3 BH-UP inverter)


## JSON Format Data
For IoT applications the raw data can now read in JSON format (application/json) by calling `http://<ip>/status`

## Homeassistant configuration

Add the following to your home assistant configuration
```   
mqtt:
  sensor:
    - state_topic: "energy/solar"
      unique_id: "growatt_osf_wr_total_production"
      name: "Growatt.TotalGenerateEnergy"
      unit_of_measurement: "kWh"
      value_template: "{{ float(value_json.TotalGenerateEnergy) | round(1) }}"
      device_class: energy
      state_class: total_increasing
      json_attributes_topic: "energy/solar"
    - state_topic: "energy/solar"
      unique_id: "growatt_osf_panels_1"
      name: "Growatt.PanelsWatts1"
      unit_of_measurement: "W"
      value_template: "{{ float(value_json.PV1InputPower) | round(1) }}"
      device_class: energy
      state_class: measurement
      json_attributes_topic: "energy/solar"
    - state_topic: "energy/solar"
      unique_id: "growatt_osf_panels_2"
      name: "Growatt.PanelsWatts2"
      unit_of_measurement: "W"
      value_template: "{{ float(value_json.PV2InputPower) | round(1) }}"
      device_class: energy
      state_class: measurement
      json_attributes_topic: "energy/solar"
    - state_topic: "energy/solar"
      unique_id: "growatt_osf_batterycharged"
      name: "Growatt.BatteryCharged"
      unit_of_measurement: "%"
      value_template: "{{ int(value_json.SOC) }}"
      state_class: measurement
      json_attributes_topic: "energy/solar"
    - state_topic: "energy/solar"
      unique_id: "growatt_osf_battery_charge"
      name: "Growatt.BatteryChargePower"
      unit_of_measurement: "W"
      value_template: "{{ float(value_json.ChargePower) | round(1) }}"
      device_class: energy
      state_class: measurement
      json_attributes_topic: "energy/solar"
    - state_topic: "energy/solar"
      unique_id: "growatt_osf_battery_discharge"
      name: "Growatt.BatteryDischargePower"
      unit_of_measurement: "W"
      value_template: "{{ float(value_json.DischargePower) | round(1) }}"
      device_class: energy
      state_class: measurement
      json_attributes_topic: "energy/solar"
    - state_topic: "energy/solar"
      unique_id: "growatt_osf_energydemand"
      name: "Growatt.EnergyDemand"
      unit_of_measurement: "W"
      value_template: "{{ int(value_json.INVPowerToLocalLoad) }}"
      device_class: energy
      state_class: measurement
      json_attributes_topic: "energy/solar"
    - state_topic: "energy/solar"
      unique_id: "growatt_osf_energytogridtoday"
      name: "Growatt.PowerToGrid"
      unit_of_measurement: "W"
      value_template: "{{ float(value_json.ACPowerToGrid) | round(1) }}"
      device_class: energy
      state_class: measurement
      json_attributes_topic: "energy/solar"
    - state_topic: "energy/solar"
      unique_id: "growatt_osf_energyfromgrid"
      name: "Growatt.EnergyFromGrid"
      unit_of_measurement: "W"
      value_template: "{{ float(value_json.ACPowerToUser) | round(1) }}"
      device_class: energy
      state_class: measurement
      json_attributes_topic: "energy/solar"
    - state_topic: "energy/solar"
      name: 'Inverter status'
      value_template: >-
        {% set v = value_json.InverterStatus %}
        {% if v == -1 %} Communication error
        {% elif v == 0 %} Standby
        {% elif v == 1 %} Normal
        {% elif v == 2 %} Discharge
        {% elif v == 3 %} Fault
        {% elif v == 4 %} Flash
        {% elif v == 5 %} PV Charging
        {% elif v == 6 %} AC Charging
        {% elif v == 7 %} Combined Charging
        {% elif v == 8 %} Combined Charging & Bypass
        {% elif v == 9 %} PV Charging & Bypass
        {% elif v == 10 %} AC Charging & Bypass
        {% elif v == 11 %} Bypass
        {% elif v == 12 %} PV Charge and Discharge
        {% else %} Unknown
        {% endif %}
```
To extract the current AC Power you have to add a sensor template.

```
template:
  - sensor:
      - name: "Growatt inverter AC Power"
        unique_id: "growatt_osf_ac_power"
        unit_of_measurement: "W"
        state: "{{ float(state_attr('sensor.growatt_inverter', 'OutputPower')) }}"

- platform: template
  sensors:
    totalpanelpowergeneration:
      value_template: >-
        {{ (((states("sensor.growatt_panelswatts1") | float) + (states("sensor.growatt_panelswatts2") | float)) / 1000) | round(1) }}
      unit_of_measurement: 'kW'
      device_class: 'energy'
```

## Home assistant energy dashboard
Remember the HASS energy dashboard should only be connected to meters - the selection tool will often allow you to pick devices that look like they work but create garbage graphs.

### Add Integral helpers for anything you want to measure that produces Watts 
* PanelsWatts1, PanelsWatts2 - The energy being generated by your panels
* BatteryCharged, BatteryDischargePower - The energy going in/out of the battery
* ACPowerToGrid - Energy you are sending to the grid
* ACPowerToUser - Energy being consumed

### Add energy meters for each of the above

### Add those meters to your energy dashboard


## Change log

See [here](CHANGELOG.md)

## Acknowledgements

This arduino sketch will replace the original firmware of the Growatt ShineWiFi stick.

This project is based on Jethro Kairys work on the Modbus interface
https://github.com/jkairys/growatt-esp8266

Some keywords:

ESP8266, ESP-07S, Growatt 1000S, Growatt 600TL, ShineWifi, Arduino, MQTT, JSON, Modbus, Rest
