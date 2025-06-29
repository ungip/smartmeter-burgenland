# Netz Burgenland Landis Gyr E450 Tasmota script ESP8266
This sketch was made to read and decrypt the data from my Smart Meter (Landis+Gyr E450 Netz Burgenland) and publish it via MQTT. This way the data can be easily integrated into Home Assistant

[here in german...](readme_deutsch.md)
## Hardware
* ESP8266 or ESP32
* [TTL - IR reader](https://www.amazon.de/dp/B0BPMVX4VW?psc=1&ref=ppx_yo2ov_dt_b_product_details)
* cable (I used old usb cable, it has 4 wires)
## Software
* download custom Tasmota firmware from [here](https://ottelo.jimdofree.com/stromz%C3%A4hler-auslesen-tasmota/) (german) or [here](https://github.com/ottelo9/tasmota-sml-images/releases)
* or build your own [here](https://tasmota.github.io/docs/Smart-Meter-Interface/)
* flash it with [Tasmotizer](https://github.com/tasmota/tasmotizer)
* configure module: generic(0)
* configure wifi

!!!!!!!!!!! Flash your ESP before connecting to TTL reader !!!!!!!!!!!!!
## Wiring
| IR reader  | ESP8266      |
| ---------- | ------------ |
| VCC        | 3.3V         |
| GND        | GND          |
| RX1        | GPIO3 (RX)   |
| TX1        | GPIO1 (TX)   |
| RX2        | GPIO2        |
| TX2        | GPIO1 (TX)   |

<p align="center">
    <img src=https://github.com/ungip/smartmeter-burgenland/blob/main/wire.jpg>    
</p>

## Script
* open Tasmota url
* [setup MQTT](https://tasmota.github.io/docs/MQTT/) (you need a running MQTT broker in Home Assistant)
* `open console -> edit script -> enable script -> copy code below`

The script is for 2 TTL readers connected to the ESP, but you can use 1 too. Than you have to delete the lines begining with +2,1,r... and correct >M 2 to >M1

In the line +1,`3`,r... the `3` is the GPIO pin, standard RX(GPIO3). You can change it if you like (by the 2nd meter is GPIO2, you can change it too...) You can use any GPIO pin of the ESP.

You have to change the text "Encyrption Key" to the key which you recive from Netz Burgenland.


```
>D
>B
smlj=0
=>sensor53 r
>R
smlj=0
>S
if upsecs>22
then
smlj|=1
endif
>M 2
+1,3,r,0,9600,smartmeter1
1,=so3,512
1,=so4,Encyrption Key
1,020Ex1UUuu@1,Spannung L1,V,V_L1,0
1,020Ex4UUuu@1,Spannung L2,V,V_L3,0
1,020Ex7UUuu@1,Spannung L3,V,V_L3,0
1,020Ex10UUuu@100,Strom L1,A,A_L1,2
1,020Ex13UUuu@100,Strom L2,A,A_L2,2
1,020Ex16UUuu@100,Strom L3,A,A_L3,2
1,020Ex19UUuuUUuu@1,Leistung +P,W,W_IN,0
1,020Ex24UUuuUUuu@1,Leistung -P,W,W_OUT,0
1,020Ex29UUuuUUuu@1000,Zählerstand +P,kWh,kWh_total_IN,3
1,020Ex34UUuuUUuu@1000,Zählerstand -P,kWh,kWh_total_OUT,3
+2,2,r,0,9600,smartmeter2
2,=so3,512
2,=so4,Encyrption Key
2,020Ex1UUuu@1,Spannung L1,V,V_L1,0
2,020Ex4UUuu@1,Spannung L2,V,V_L3,0
2,020Ex7UUuu@1,Spannung L3,V,V_L3,0
2,020Ex10UUuu@100,Strom L1,A,A_L1,2
2,020Ex13UUuu@100,Strom L2,A,A_L2,2
2,020Ex16UUuu@100,Strom L3,A,A_L3,2
2,020Ex19UUuuUUuu@1,Leistung +P,W,W_IN,0
2,020Ex24UUuuUUuu@1,Leistung -P,W,W_OUT,0
2,020Ex29UUuuUUuu@1000,Zählerstand +P,kWh,kWh_total_IN,3
2,020Ex34UUuuUUuu@1000,Zählerstand -P,kWh,kWh_total_OUT,3
#
```

After restart, you have to see this data on the main page:

<p align="center">
    <img src=https://github.com/ungip/smartmeter-burgenland/blob/main/tasmota.png>    
</p>

## Home Assistant integration
Home Assistant should recognize your Tasmota device automatically

If you want to add the meter reading to the energy dashboard, you have to customize the type of the data. Add this template to your configuration.yaml:

```yaml
template:
  - sensor:
    - name: "Stomzähler Haus"
      unit_of_measurement: 'kWh'
      device_class: "energy"
      state_class: "total_increasing"
      state: >-
            {{ float(states('sensor.tasmota_smartmeter1_kwh_total_in')) | round(3) }}
```
You can make this to all sensor data, but the energy dashboard works only with kWh measurements (eg. kwh_total_in, kwh_total_out)
