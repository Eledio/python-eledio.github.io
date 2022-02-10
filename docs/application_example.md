---
summary: Application example of Python Eledio library
authors:
    - Richard Kubicek
date: 2019-09-06
---
# Example of usages
There you can find some examples, how to use python-eledio library in your Eledio devices.

## State LED, buzzer and watchdog control
### Configuration _json_ file
File name: _mpu-config.json_
```
{
  "devices": {
    "mpu-dev": {
      "bus": "mpu"
    }
  },
  "identifiers": {
    "mpuWifiRssi": {
      "unit": "mpu-dev",
      "parameter": "wifi-rssi"
    },
    "mpuInternalTemperature": {
      "unit": "mpu-dev",
      "parameter": "temperature"
    },
    "mpuCpuPercent": {
      "unit": "mpu-dev",
      "parameter": "cpu-percent"
    },
    "mpuLed": {
      "unit": "mpu-dev",
      "parameter": "led"
    },
    "mpuBeeper": {
      "unit": "mpu-dev",
      "parameter": "beeper"
    },
    "mpuWatchdog": {
      "unit": "mpu-dev",
      "parameter": "watchdog"
    }
  }
}
```
### Usage
```
import json

from eledio import Eledio
from eledio.component.mpu.i2c import SMBus
from eledio.device.mpu import MpuFactory

if __name__ == "__main__":
    eledio = Eledio()
    eledio.register_device_factory("mpu", MpuFactory())

    with open('mpu-config.json') as f:
        eledio.append_config(json.load(f))

	eledio["mpuLed"] = (0x40, 0x00, 0x00) # set color of state LED in (R, G, B) format
	eledio["mpuBeeper"] = (1500, 10)	# after calling store_outputs() switch on beeper on frequency 1500 Hz for 1s (first parameter is frequency in Hz, second parameter is duration in hundred ms)

	'''
	Warning, next statement is dangerous, it couse restart of linux machine (watchdog).
	This function is good to use when you have your development complete, and you know, that you communicate with components and devices lower then every x second.
	'''
	eledio["mpuWatchdog"] = 120 # after calling store_outputs() this statement couse reset of linux every 120 seconds. If there is any communication with devicese, timeout is restarted.

    eledio.store_outputs()
```
## WiFi RSSI, linux CPU utilization
Use [same _json_](#configuration-json-file) file as in previous example.
### Usage
```
import json

from eledio import Eledio
from eledio.component.mpu.i2c import SMBus
from eledio.device.mpu import MpuFactory

if __name__ == "__main__":
    eledio = Eledio()
    eledio.register_device_factory("mpu", MpuFactory())

    with open('mpu-config.json') as f:
        eledio.append_config(json.load(f))

	eledio.load_inputs()

	rssi = eledio["mpuWifiRssi"] # place to variable rssi, WiFi signal RSSI
	utilization = eledio["mpuCpuPercent"] # place to variable utilitation, linux CPU utilization
```
## SRQ/IRQ handler
If you want to used quick access to variables like received code from RF433 control, you can use SRQ handler.

Inside SRQ handler, in this example function ```handle_srq```, you received dictionary of eledio identifiers and their new value after SRQ.
### Configuration _json_ file
File name: _map.json_
```
{
  "devices": {
    "unit-40": {
      "bus": "i2c",
      "address": 40,
      "datacrc": 43200,
      "compilermagic": 4006394777
    }
  },
  "identifiers": {
    "gatewayHWEvent": {
      "unit": "unit-40",
      "partition": "sdp",
      "offset": 0,
      "entrysize": 2,
      "depth": 0,
      "type": 1,
      "prescaler": 0,
      "a": 1,
      "b": 0,
      "datatype": "u16",
      "srq": 0
    },
    "gatewayHWState": {
      "unit": "unit-40",
      "partition": "sdp",
      "offset": 2,
      "entrysize": 2,
      "depth": 0,
      "type": 1,
      "prescaler": 0,
      "a": 1,
      "b": 0,
      "datatype": "u16"
    },
    "rf433": {
      "unit": "unit-40",
      "partition": "sdp",
      "offset": 4,
      "entrysize": 4,
      "depth": 0,
      "type": 1,
      "prescaler": 0,
      "a": 1,
      "b": 0,
      "datatype": "u32",
      "srq": 1
    },
    "gatewayHWAction": {
      "unit": "unit-40",
      "partition": "adp",
      "offset": 0,
      "entrysize": 2
    }
  }
}
```
### Usage
```
import json

from eledio import Eledio
from eledio.component.mpu.i2c import SMBus
from eledio.device.mpu import MpuFactory

def handle_srq(identifiers):
    """
    User handling of SRQ (called in context of eledio.wait_events)
    :param identifiers: dictionary of eledio identifiers and their new value after SRQ
    :return:
    """
    print("Service requests!", identifiers)

if __name__ == "__main__":
    eledio = Eledio()
    eledio.register_device_factory("i2c", PcuFactory(SMBus(0)))
    eledio.register_srq(Srq(), handle_srq())

    # repeat for every configuration file
    with open('map.json') as f:
        eledio.append_config(json.load(f))

 while True:
        # load state of inputs and outputs
        eledio.load_inputs()

        # do something else or wait some time
        eledio.wait_events(1.0)
```
## Error handler
In runtime and while you develop your solution, there could rise some errors. You can catch them by definition of _error handler_.
### Usage
In this example _error handler_ is represented by function ```handler_error```.
```
import json

from eledio import Eledio
from eledio.component.mpu.i2c import SMBus
from eledio.device.pcu import PcuFactory
from eledio.device.mpu import MpuFactory
from eledio.device.srq import Srq


def handle_srq(identifiers):
    """
    User handling of SRQ (called in context of eledio.wait_events)
    :param identifiers: dictionary of eledio identifiers and their new value after SRQ
    :return:
    """
    print("Service requests!", identifiers)


def handle_error(src, ex):
    """
    Handle error inside eledio library
    :param src:
    :param ex:
    :return:
    """
    print(src, ex)


if __name__ == "__main__":
    eledio = Eledio()
    eledio.register_device_factory("i2c", PcuFactory(SMBus(0)))
    eledio.register_device_factory("mpu", MpuFactory())
    eledio.register_srq(Srq(), handle_srq)
    eledio.error_handler = handle_error

    # repeat for every configuration file
    with open('config.json') as f:
        eledio.append_config(json.load(f))

    with open('mpu-config.json') as f:
        eledio.append_config(json.load(f))

    while True:
        # load state of inputs and outputs
        eledio.load_inputs()

        # manipulate with inputs and outputs by identifier
        # e.g.
        #   print(eledio["test1"])
        #   eledio["test2"] = True

        # apply final value to hardware
        eledio.store_outputs()

        # do something else or wait some time
        eledio.wait_events(1.0)
```
