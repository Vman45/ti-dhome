
RFXtrx433 is used to monitor and act on various proprietary systems such as:
* Weather Station
* Home Central Alarm

Listens on RfxTrx433 using nodejs and publishes to MQTT (mosquitto).


**CURRENTLY RUNNING RFXTRX433 FROM NODE-RED PROJECT FLOW**


# Hardware

![Parts](res/rfxtrx433-schema.jpg?raw=true "Hardware overview")


# Test

## Hardware

* Connect rfxtrx433 to USB and find which usb resource connected

```js
$ lsusb
$ dmesg
```

* Allow user access to resource and set speed

```js
$ sudo stty -F /dev/ttyUSB1 38400 cs8
$ sudo chmod 777 /dev/ttyUSB1
```

* Edit test script to point to USB devices or point Node-Red to correct USB device.

## Software

Below is sample which listens on USB plugged RfxTrx433 and then publishes temperature/humidity/rain over MQTT.



**[node-rfxcom](https://github.com/kalemena/node-rfxcom) has been patched to support rain sensor from Oregon**



```js
#!/usr/bin/env node

var rfxcom = require("rfxcom"),
    eyes = require('eyes'),
    mqtt = require('mqtt');

var rfxtrx = new rfxcom.RfxCom("/dev/ttyUSB0", {debug: false} );
var client = mqtt.createClient('1883', 'localhost');

client.on('connect', function() {
  console.log('Connected for publications...');
});

// Start RFXCom activity
rfxtrx.on("th2", function (evt) {
    client.publish('sensors/' + parseInt(evt.id,16) + '/entries/1/events/temperature', '' + evt.temperature);
    client.publish('sensors/' + parseInt(evt.id,16) + '/entries/1/events/humidity', '' + evt.humidity);
});

rfxtrx.on("temp2", function (evt) {
    client.publish('sensors/' + parseInt(evt.id,16) + '/entries/1/events/temperature', '' + evt.temperature);
});

rfxtrx.on("rain2", function (evt) {
    var value = { rainrate: evt.rainrate, raintotal: evt.raintotal };
    client.publish('sensors/' + parseInt(evt.id,16) + '/entries/1/events/rain', JSON.stringify(value));
});

rfxtrx.initialise(function () {
    console.log("Device initialized");
});
```







