# RadioKey

## Hardware

### Flashers

 * https://github.com/marcelstoer/nodemcu-pyflasher
 * https://nodemcu.readthedocs.io/en/master/getting-started/
 * https://tasmota.github.io/docs/
 * https://www.espressif.com/en/support/download/other-tools (Flash Download Tools (ESP8266 & ESP32 & ESP32-S2))
 * https://github.com/espressif/esptool
 
#### Troubleshooting

* If you get `[upload] could not open port /dev/ttyACM0: [Errno 13] Permission denied: '/dev/ttyACM0'` error, try:
```
sudo usermod -a -G tty ${USER}
sudo chmod a+rw /dev/ttyACM0
```

### Controllers

#### Firmware components

* [RCSwitch](https://github.com/sui77/rc-switch)

#### ESP8266

* [Add ESP8266 to Arduino IDE Board Manager](https://github.com/esp8266/Arduino/#installing-with-boards-manager)
* Pin numbers in Arduino correspond directly to the ESP8266 GPIO pin numbers. pinMode, digitalRead, and digitalWrite functions work as usual, so to read GPIO2, call digitalRead(2).
* [ESP8266 Howto](https://tttapa.github.io/ESP8266/Chap07%20-%20Wi-Fi%20Connections.html)

##### Flashing firmware from espressif

1) Dowload actual firmware from https://www.espressif.com/en/products/modules/esp8266/resources or use included one.

2) Write bootloader:

```
esptool --port /dev/ttyUSB0 write_flash 0x00000 boot_v1.2.bin 0x01000 at/512+512/user1.1024.new.2.bin 0xfc000 esp_init_data_default.bin 0x7e000 blank.bin 0xfe000 blank.bin

esptool run
```

##### Firmware

* [ESP8266-01](https://github.com/RadioKey/Firmware_ESP8266-01)

### EV1527

* [EV1527 433.92MHz Manual](/datasheets/EV1527.pdf)

Code:

| Symbol | Signal level switch | 
|---|-------|
| 1 | 3H+1L |
| 0 | 1H+3L |


Message:

| Part | Signal level switch |
|---|-------|
| Preamble | 32 clk | 1H+31L |
| Secret code | 20x4 clk | e.g. 11011110111001100011 |
| Button code | 4x4 clk | e.g. 0001 | 

#### Sniffing

* https://www.gnuradio.org/
* https://www.rtl-sdr.com/
* https://github.com/NetHome/ProtocolAnalyzer - IR/RF protocol analyser
* [apt-get install rtl-433](https://github.com/merbanan/rtl_433) - generic data receiver, mainly for the 433.92 MHz, 868 MHz (SRD), 315 MHz, 345 MHz, and 915 MHz ISM bands.
* gqrx - GUI for monitore and record radio signal
* audacity - audio editor, allows to visualise signal


### Antenna

Frequency 433MHz - Wavelength 0.692 m

Length of dipole must me multiple of half the wavelength, e.g. 0.345.

### fs1000a / XY-MK-5V 

* [AliExpress](https://aliexpress.ru/item/32318951712.html)

![](/datasheets/fs1000a-XY-MK-5V-433mhz-module-pinout.png)

#### FS1000A (Transmitter Part) Features & Technical Specifications:

* Transmitter Model: FS1000A
* Transmitting Frequency: 433.92MHz, 315MHz, 330MHz
* Operating Voltage: 3V to 12V
* Transmitting Power: 10mW to 40mW 16dBm
* Transmitting Range: 20 to 100 meters through walls & 500 meters max in open area
* Data Transfer Rate: <10 Kilo Bytes Per Second (Range drops above 2400 bytes per second)
* Modulation Type: OOK
* Current Consumption: 20 to 28mA
* Current Consumption in Standby: 0mA
 

#### XY-MK-5V (Receiver Part) Features & Technical Specifications:

* Receiver Model: XY-MK-5V
* Transmitting Frequency: 433.92MHz, 315MHz, 330MHz
* Operating Voltage: 5V
* Receiving Sensitivity: 105DB
* Modulation Type: OOK
* Current Consumption in Standby: 4mA

## Services

### MQTT Server

Use following server for test purposes

#### Artemis 

```
# create instance
./bin/artemis create radiokey --require-login --user test --password test --autocreate --no-amqp-acceptor --no-hornetq-acceptor --no-stomp-acceptor
# start broker as standalone process
./radiokey/bin/artemis run
# start broker in background
./radiokey/bin/artemis-service startstart broker in background
```

HTTP Server started at http://localhost:8161
Artemis Jolokia REST API available at http://localhost:8161/console/jolokia
Artemis Console available at http://localhost:8161/console

#### Mosquito

Configure server in `/etc/mosquitto/mosquitto.conf

```
listener {port} {publicIp}
persistence false
```

Configure credentials:

```
sudo mosquitto_passwd /etc/mosquitto/pwfile your_user_name
```

Connect to response topic to check that hub is working and may listen to commands from moquitto server

```
mosquitto_sub -h {publicIp} -p {port} -t response
```

This will show heartbeats:

```
{"command":"heartbeat","mac":"AA:AA:AA:AA:AA:AA","deviceId":"HUBESP01","protocolVersion":"0.0.1"}
```

Now you may send commands to concrete hub using it's MAC address:

```
mosquitto_pub -h {publicIp} -p {port} -t "AA:AA:AA:AA:AA:AA" -m "{"command": "send", "protocol": "1", "repeats": "25", "pulseLength": "315", "code": "14607921", "bitLength": "24"}"
```

## MQTT Communication protocol

All commands sends to topic with name of hub's mac address.

### Scan

Switch hub to scan mode when it scans codes from air

```
{"command": "scan", "repeats": "10", "delayMs": "100"}
```

### Restart hub

```
{"command": "restart"}
```

### Reconfigure

Force hub reconfiguration: reset wi-fi settings and go to initialisation mode

```
{"command": "reconfigure"}
```

### Send key

Send kode of controllable device into air:

```
{"command": "send", "protocol": "1", "repeats": "25", "pulseLength": "315", "code": "14607921", "bitLength": "24"}
```
