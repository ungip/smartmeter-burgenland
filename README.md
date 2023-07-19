# Netz Burgenland Landis Gyr E450 Tasmota script ESP8266
This sketch was made to read and decrypt the data from my Smart Meter (Landis+Gyr E450 Netz Burgenland) and publish it via MQTT. This way the data can be easily integrated into Home Assistant

## Hardware
* ESP8266 or ESP32
* [TTL - IR reader](https://www.amazon.de/dp/B0BPMVX4VW?psc=1&ref=ppx_yo2ov_dt_b_product_details)
* cable (I used old usb cable, it has 4 wires)
## Software
* download custom Tasmota firmware from [here](https://ottelo.jimdofree.com/stromz%C3%A4hler-auslesen-tasmota/) (german)
* or build your own [here](https://tasmota.github.io/docs/Smart-Meter-Interface/)
* flash it with [Tasmotizer](https://github.com/tasmota/tasmotizer)

!!!!!!!!!!! Flash your ESP before connecting to TTL reader !!!!!!!!!!!!!
## Wiring
## Script
## Home Assistant integration
