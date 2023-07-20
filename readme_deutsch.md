# Netz Burgenland Landis Gyr E450 Tasmota script ESP8266
Diese Anleitung ist eine Zusammenfassung, was ich mühevoll von verschiedenen Quellen gesammelt habe.

Das Script liest die Werte vom Netz Burgenland - Energie Burgenland Smart Meter aus. Nachher können die Daten leicht über MQTT in Home Assistant integriert werden.

[Man muss die Kundenschnittstelle freischalten lassen!!! (optisch)](https://www.netzburgenland.at/kundenservice/smart-metering/smart-metering/kundenschnittstelle.html).
Das [Formular](https://github.com/ungip/smartmeter-burgenland/blob/e5fbbfa4e9089c5f5da63bcec65ad0c09188d6ec/Freischaltung_Kundenschnittstelle.pdf) an die adresse: info.metering@netzburgenland.at senden
## Hardware
* ESP8266 oder ESP32
* [TTL - IR leser](https://www.amazon.de/dp/B0BPMVX4VW?psc=1&ref=ppx_yo2ov_dt_b_product_details)
* Kabel (habe alte USB-Kabel verwendet, die haben 4 Ader)
## Software
* Vorgefertigte Tasmota firmware herunterladen [hier](https://ottelo.jimdofree.com/stromz%C3%A4hler-auslesen-tasmota/)
* oder selbst kompilieren [hier](https://tasmota.github.io/docs/Smart-Meter-Interface/)
* flashen mit [Tasmotizer](https://github.com/tasmota/tasmotizer)
* modul konfigurieren: generic(0)
* wlan einrichten

!!!!!!!!!!! Zuerst ESP flashen erst dann mit dem TTL reader verbinden !!!!!!!!!!!!!
## Verkabelung
| IR Leser  | ESP8266      |
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
* Tasmota URL öffnen
* [MQTT einrichten](https://tasmota.github.io/docs/MQTT/) (MQTT broker wird benötigt zB. von Home Assistant)
* `console öffnen -> edit script -> enable script -> script unten kopieren`

Das untere Script ist für 2 TTL Leseköpfe (2 smart meter), funktioniert natürlich auch mit 1. Man muss dann die Reihen beginnend mit +2,1,r... bis zu # löschen und >M 2 auf >M 1 korrigieren.

In der Reihe +1,`3`,r... die `3` ist der GPIO pin, standard RX(GPIO3). Man kann aber beliebigen pin nehmen. (beim zweiten ist GPIO2, natürlich auch veränderbar...)

Eigene "Encyrption Key" einfügen, bekommt man vom [Netzbetreiber](https://www.netzburgenland.at/kundenservice/smart-metering/smart-metering/kundenschnittstelle.html). Der optische Port wird benötigt...


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

Nach dem Neustart sollten folgende Daten ersichtlich sein:

<p align="center">
    <img src=https://github.com/ungip/smartmeter-burgenland/blob/main/tasmota.png>    
</p>

## Home Assistant Integration
Home Assistant erkennt das Tasmota-Gerät automatisch

Wenn man die Daten im energy dashboard verwenden möchte, muss man die Sensordaten zuerst konfigurieren. Zum configuration.yaml folgende Templates hinzufügen:

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
Der energy dashboard verarbeitet nur sensoren mit kWh Einheit (zB. kwh_total_in, kwh_total_out)

### Quellen
* https://ottelo.jimdofree.com/stromz%C3%A4hler-auslesen-tasmota/
* https://tasmota.github.io/docs/Smart-Meter-Interface/
* https://www.photovoltaikforum.com/thread/137994-landis-gyr-e450-auslesen-wie-ich-es-mache/
* https://www.youtube.com/channel/UCiU--5PKQOMdfMTG7dTKc7g/videos
