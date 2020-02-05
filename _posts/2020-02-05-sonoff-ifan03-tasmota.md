---
layout: post
title: Deckenventilator mit Sonoff iFan03 Steuerung
categories: [💡Smart Home]
excerpt: In Vorbereitung auf die heißen Sommertage habe ich den Deckenventilator - Westinghouse Bendan fürs Schlafzimmer gekauft. Zur Ansteuerung nutze ich das Sonoff iFan03 Module. Somit kann der Ventilator in Abhängigkeit von z.B. Temperatur, Luftfeuchtigkeit oder Zeit geschaltet werden.
---

In Vorbereitung auf die heißen Sommertage habe ich den Deckenventilator - [Westinghouse Bendan](https://www.amazon.de/dp/B002Y15CWO/ref=cm_sw_em_r_mt_dp_U_syVoEbBV1H1S0) fürs Schlafzimmer gekauft. Zur Ansteuerung nutze ich das [Sonoff iFan03](https://www.amazon.de/dp/B07TRTG8PS/ref=cm_sw_r_tw_dp_U_x_7NVoEb0HC0AWJ) Module. Somit kann der Ventilator in Abhängigkeit von z.B. Temperatur, Luftfeuchtigkeit oder Zeit geschaltet werden.

## Tools

Editor: [Visual Studio Code](https://code.visualstudio.com/download) mit [PlatformIO IDE](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide)  
Tasmota Version: [8.1.0.3](https://github.com/arendst/Tasmota/tree/master)  
Flashing Programm: [Tasmotizer](https://github.com/tasmota/tasmotizer)  
Flashing Tool: [USB zu TTL-Konverter-Modul](https://www.amazon.de/USB-TTL-Konverter-Modul-mit-eingebautem-CP2102/dp/B00AFRXKFU/ref=sr_1_3?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&keywords=USB+zu+TTL-Konverter-Modul+mit+eingebautem+in+CP2102&qid=1578948764&s=computers&sr=1-3)  
Kleinkram: [Jumperkabel](https://www.amazon.de/Female-Female-Male-Female-Male-Male-Steckbrücken-Drahtbrücken-bunt/dp/B01EV70C78/ref=sr_1_3?__mk_de_DE=ÅMÅŽÕÑ&crid=3D9JJ4C2W5VM4&keywords=jumper+kabel&qid=1579031684&sprefix=jumper%2Caps%2C150&sr=8-3)

## Tasmota flashen

>**Achtung:** _Niemals die 230V Versorgung nutzen um den Shelly zu flashen! Immer die 3.3V USB-TTL Konverter Versorgung nutzen._

### Pinout

Pinout der Plantine (Rückseite) für den Anschluss der seriellen Schnittstelle.

![sonoff-ifan03-pcb](/images/sonoff-ifan03.jpg)
[Quelle](https://templates.blakadder.com/sonoff_ifan03.html)

### Tasmotizer

* Port auswählen
* Select image:  *Release* `tasmota.bin`
* Enable: Backup original firmware
* Enable: Erase before flashing
* Send config - Module: Sonoff iFan03, WLAN/MQTT: deine Einstellungen hinterlegen

![Tasmotizer](/images/tasmotizer-menu-screen.png)

### Anlernen der Fernbedinung

1. Nach dem Flashen das Sonoff iFan03 Module wieder mit dem Gehäuse sicher verschließen.  
2. Nun einen Taster der Fernbedinung gedrückt halten, während das Module an 230V eingeschaltet wird. 
3. Anschließend ist die Fernbedinung verbunden und die Relais können damit geschaltet werden.

## Sonoff iFan03 an den Ventilator anschließen

Das Kabel welches am Klemmblock des Decken-Montagesockels angeschlossen ist, habe ich abgeklemmt und mit Wago Klemmen am Sonoff iFan03 verbunden. Somit kann der Sonoff iFan03 direkt per Steckverbindung mit dem Ventilatormotor verbunden werden und es müssen keine neuen Steckverbindungen verbaut werden.

![sonoff-ifan03-connection](/images/sonoff-ifan03-connect.jpg)

>**Achtung:** _Keine Gewähr auf die Anschlusstabelle. Anschluss nur durch eine qualifizierte Person durchführen lassen!_

| Klemmblock         | Sonoff Input | Sonoff Output | Westinghouse Bendan |
| ------------------ | ------------ |---------------|---------------------|
| L (braun)          | L (schwarz)  | FAN (schwarz) | Stecker (braun)     |
| N (blau)           | N (weiß)     | COM (weiß)    | Stecker (blau)      |
| -                  | -            | LIGHT (blau)  | Stecker (rot)       |
| Erdung (grün/gelb) | -            | -             | Erdung (grün/gelb)  |

Achtet darauf, alle Kabel ordentlich und platzsparend zu verbauen, denn die Abdeckung des Decken-Montagesockels hat nicht viel Platz.

## Home Assistant Config

Mein MQTT Topic: `ventilator_01` muss durch eures ersetzt werden!

### Allgemein

`configuration.yaml`

```yaml
homeassistant:
  customize: !include_dir_merge_named customize/
fan: !include_dir_merge_list fan/
light: !include_dir_merge_list light/
```

### Ventilator

`fan/ventilator01.yaml`

```yaml
- platform: mqtt  
  name: ventilator_01
  command_topic: "cmnd/ventilator_01/FanSpeed"
  speed_command_topic: "cmnd/ventilator_01/FanSpeed"    
  state_topic: "stat/ventilator_01/RESULT"
  speed_state_topic: "stat/ventilator_01/RESULT"
  state_value_template: >
    {% if value_json.FanSpeed is defined %}
      {% if value_json.FanSpeed == 0 -%}0{%- elif value_json.FanSpeed > 0 -%}ON{%- endif %}
    {% else %}
      {% if states.fan.ventilator_01.state == 'off' -%}0{%- elif states.fan.ventilator_01.state == 'on' -%}ON{%- endif %}
    {% endif %}
  speed_value_template: "{{ value_json.FanSpeed }}"
  availability_topic: tele/ventilator_01/LWT
  payload_off: "0"
  payload_on: "ON"
  payload_low_speed: "1"
  payload_medium_speed: "2"
  payload_high_speed: "3"
  payload_available: Online
  payload_not_available: Offline
  qos: 1
  retain: false
  speeds:
    - off
    - low
    - medium
    - high
```

### Lampe

`light/ventilator01.yaml`

```yaml
- platform: mqtt
  name: ventilator_01
  state_topic: "stat/ventilator_01/RESULT"
  value_template: "{{ value_json.POWER1 }}"
  command_topic: "cmnd/ventilator_01/POWER1"
  availability_topic: "tele/ventilator_01/LWT"
  payload_on: "ON"
  payload_off: "OFF"
  payload_available: "Online"
  payload_not_available: "Offline"
  retain: false
  qos: 1
```

### Customize

`customize/ventilator01.yaml`

```yaml
fan.ventilator_01:
  friendly_name: Deckenventilator
light.ventilator_01:
  friendly_name: Schlafzimmerlampe
  icon: mdi:ceiling-light
```
