---
layout: page
title: New
permalink: /new/
---

# Shelly 2.5 Rollladen- /Jalousiesteuerung mit Tasmota
## Tools
Editor: [Visual Studio Code](https://code.visualstudio.com/download) mit [PlatformIO IDE](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide)  
Tasmota Version: [8.1.0.3](https://github.com/arendst/Tasmota/tree/master)  
Flashing Programm: [Tasmotizer](https://github.com/tasmota/tasmotizer)  
Flashing Tool: [USB zu TTL-Konverter-Modul](https://www.amazon.de/USB-TTL-Konverter-Modul-mit-eingebautem-CP2102/dp/B00AFRXKFU/ref=sr_1_3?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&keywords=USB+zu+TTL-Konverter-Modul+mit+eingebautem+in+CP2102&qid=1578948764&s=computers&sr=1-3) 

## Firmware kompilieren
Da die Default Tasmota Firmware keine Rollladen-/Jalousiesteuerung unterstützt, muss der Quelltext angepasst werden und eine eigene Firmware kompiliert werden. 

In der Datei `Tasmota/tasmota/my_user_config.h` folgendes auskommentieren:
```cpp
// — Optional modules ——————————————
define USE_SHUTTER
```

PlatformIO Build starten und die Firmware wird kompiliert.
Speicherort: `Tasmota/build_output/firmware/tasmota.bin`

## Shelly 2.5 flashen
**Notiz:**	_Niemals die 230V Versorgung nutzen um den Shelly zu flashen! 	Immer die 3.3V USB-TTL Konverter Versorgung nutzen._

Den Shelly 2.5 mit dem USB-TTL Konverter verbinden.
Ich empfehle die Stiftleisten aus der [Tasmota Documentation](https://tasmota.github.io/docs/#/devices/Shelly-2.5)  
![GPIO Pinout](https://shelly.cloud/wp-content/uploads/2019/01/pin_out-650x397.png)
![GPIO Pinout](https://user-images.githubusercontent.com/11555742/69892658-23d43f80-1308-11ea-8caf-fcd719626f74.png)
* Port auswählen
* Open Image:  `Tasmota/build_output/firmware/tasmota.bin`
* Enable: Backup original Firmware
* Enable: Erase before flashing
* Send config - WLAN Einstellungen hinterlegen  

## Tasmota Config
Den Shelly Neustarten und per Webinterface verbinden.
Im Menü unter `Configuration/Configure Other/Template` folgendes Template hinterlegen:
```json
{"NAME":"Shelly 2.5","GPIO":[56,0,17,0,21,83,0,0,6,82,5,22,156],"FLAG":2,"BASE":18}
```

```
PowerRetain 1
SwitchMode1 1
SwitchMode2 1
ShutterRelay1 1
ShutterMode 0
Backlog PulseTime1 0; PulseTime2 0
Backlog Interlock 1,2; Interlock ON
Restart 1
ShutterOpenDuration1 29
ShutterCloseDuration1 29
ShutterClose1
ShutterSetHalfway1 54
Restart 1
rule1 on energy#power[1]>160 do backlog power1 0; power2 0 endon
rule1 1
rule1 5
rule2 on energy#power[2]>160 do backlog power1 0; power2 0 endon
rule2 1
rule2 5
SetOption42 73
```
