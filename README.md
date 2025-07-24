# AMRIDM2MQTT: Send AMR/ERT Utility Meter Data Over MQTT

##### Copyright (c) 2018 Ben Johnson. Distributed under MIT License.

Using an [inexpensive rtl-sdr dongle](https://www.amazon.com/s/ref=nb_sb_noss?field-keywords=RTL2832U), it's possible to listen for signals from ERT compatible smart meters using rtlamr. This script runs as a daemon, launches rtl_tcp and rtlamr, and parses the output from rtlamr. When it detects data from your configured meters, it publishes the readings to MQTT for consumption by Home Assistant, OpenHAB, or custom scripts.

## Supported Meters

This service supports both water and gas meters that use the ERT (Encoder Receiver Transmitter) protocol. It can track:

### Water Meters
- Total consumption
- No usage periods
- Backflow detection
- Leak detection (both current and historical)
- All data is published to MQTT topics under `Home/WaterMeter/`

### Gas Meters
- Total consumption
- Published to MQTT topic `Home/GasMeterTotalValue`

## Features
- Real-time meter monitoring using rtl-sdr
- Automatic reconnection on connection loss
- Health check monitoring via HTTP
- Configurable meter ID filtering
- Systemd service integration
- Detailed logging with syslog integration


## Docker

If you use Docker and would rather launch this under a container see <README.Docker.md>.

## Requirements

Tested under Raspbian GNU/Linux 9.3 (stretch)

### rtl-sdr package

Install RTL-SDR package

`sudo apt-get install rtl-sdr`

Set permissions on rtl-sdr device

/etc/udev/rules.d/rtl-sdr.rules

`SUBSYSTEMS=="usb", ATTRS{idVendor}=="0bda", ATTRS{idProduct}=="2838", MODE:="0666"`

Prevent tv tuner drivers from using rtl-sdr device

/etc/modprobe.d/rtl-sdr.conf

`blacklist dvb_usb_rtl28xxu`

### git

`sudo apt-get install git`

### pip3 and paho-mqtt

Install pip for python 3

`sudo apt-get install python3-pip`

Install paho-mqtt package for python3

`sudo pip3 install paho-mqtt`

### golang & rtlamr

Install Go programming language & set gopath

`sudo apt-get install golang`

https://github.com/golang/go/wiki/SettingGOPATH

If only running go to get rtlamr, just set environment temporarily with the following command

`export GOPATH=$HOME/go`


Install rtlamr https://github.com/bemasher/rtlamr

`go get github.com/bemasher/rtlamr`

To make things convenient, I'm copying rtlamr to /usr/local/bin

`sudo cp ~/go/bin/rtlamr /usr/local/bin/rtlamr`

## Install

### Clone Repo
Clone repo into opt

`cd /opt`

`sudo git clone https://github.com/ragingcomputer/amridm2mqtt.git`

### Configure

Copy template to settings.py

`cd /opt/amridm2mqtt`

`sudo cp settings_template.py settings.py`

Edit file and replace with appropriate values for your configuration

`sudo nano /opt/amridm2mqtt/settings.py`

### Install Service and Start

Copy armidm2mqtt service configuration into systemd config

`sudo cp /opt/amridm2mqtt/amridm2mqtt.systemd.service /etc/systemd/system/amridm2mqtt.service`

Refresh systemd configuration

`sudo systemctl daemon-reload`

Start amridm2mqtt service

`sudo service amridm2mqtt start`

Set amridm2mqtt to run on startup

`sudo systemctl enable amridm2mqtt.service`

### Configure Home Assistant

To use these values in Home Assistant, add the following to your configuration:

#### Water Meter Configuration
```yaml
sensor:
  - platform: mqtt
    state_topic: "Home/WaterMeter/TotalValue"
    name: "Water Meter Total"
    unit_of_measurement: "gal"
    
  - platform: mqtt
    state_topic: "Home/WaterMeter/NoUse"
    name: "Water No Usage"
    device_class: "duration"
    
  - platform: mqtt
    state_topic: "Home/WaterMeter/BackFlow"
    name: "Water Backflow"
    
  - platform: mqtt
    state_topic: "Home/WaterMeter/LeakDetected"
    name: "Water Leak History"
    device_class: "problem"
    
  - platform: mqtt
    state_topic: "Home/WaterMeter/LeakNowDetected"
    name: "Water Leak Current"
    device_class: "problem"

#### Gas Meter Configuration
  - platform: mqtt
    state_topic: "Home/GasMeterTotalValue"
    name: "Gas Meter Total"
    unit_of_measurement: "CCF"
```

## Testing

Assuming you're using mosquitto as the server, and your meter's id is 12345678, you can watch for events using the command:

`mosquitto_sub -t "readings/12345678/meter_reading"`

Or if you've password protected mosquitto

`mosquitto_sub -t "readings/12345678/meter_reading" -u <user_name> -P <password>`
