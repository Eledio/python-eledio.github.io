---
summary: Documentation site of Python Eledio library
authors:
    - Richard Kubicek
date: 2019-09-27
---
# Description of API for python-eledio library
This description allow to you easly write your own python script.

This script you can use for control of your Eledio devices. This sript you can upload to your [__Eledio Gateway__](https://www.eledio.com/en/hardware/) and control your gateways and your [__Eledio Extendres__](https://www.eledio.com/en/hardware/) which you want control from gateway.

This library is preinstalled in your [Eledio Gateway](https://www.eledio.com/en/hardware/).  You can start programming, after unpacking.


## Necessary parts
Each major drive script in python for your application must be made up with some necessary parts.

* [device description in _json_ file](#device-description-in-json-file)

* [import of library components](#import-of-library-components)

* [load of _json_ file](#load-of-json-file)

### Device description in _json_ file
* this file is for hardware abstraction layer to linux
* you are able to used your own names of identifiers, eg. you can rename _Temperature_ to _OutsideTemperature_
* every name of identifiers must consist only with English alphabet characters, without spaces and other punctuation
* if you need some reaclculation of readout values, you can use prepared values __a__ and __b__, with are parts of equation __y=a*x+b__, where __x__ is readout value from device, __y__ is value which you obtain in python script, eg. if you need temeperature in degree of fahrenheit, you can place __a=1.8__ and __b=32__ (which corresponds of equation ```T[°F] = T[°C]*1.8 + 32```)
#### Example of _json_ file, can be named as  config.json
```
{
  "devices": {
    "unit-40": {
      "bus": "i2c",
      "address": 40,
      "datacrc": 9718,
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
    "RE1Voltage": {
      "unit": "unit-40",
      "partition": "sdp",
      "offset": 4,
      "entrysize": 2,
      "depth": 0,
      "type": 1,
      "prescaler": 0,
      "a": 0.01,
      "b": 0,
      "datatype": "s16"
    },
    "Temperature": {
      "unit": "unit-40",
      "partition": "sdp",
      "offset": 6,
      "entrysize": 2,
      "depth": 0,
      "type": 1,
      "prescaler": 0,
      "a": 0.01,
      "b": 0,
      "datatype": "s16"
    },
    "gatewayHWAction": {
      "unit": "unit-40",
      "partition": "adp",
      "offset": 0,
      "entrysize": 2
    },
    "RE1": {
      "unit": "unit-40",
      "partition": "adp",
      "offset": 2,
      "entrysize": 1
    },
    "RE3": {
      "unit": "unit-40",
      "partition": "adp",
      "offset": 3,
      "entrysize": 1
    }
  }
}
```

### Import of library components
* consist with few of import statements, which are necessary for correct function

```
import json # necessary for description json file; required

from eledio import Eledio   # basic functionality; required
from eledio.component.mpu.i2c import SMBus  # communication interface with linux; required
from eledio.device.pcu import PcuFactory    # access to variables and components of base board, e.g: relay outputs, digital inputs, ...; required
from eledio.device.mpu import MpuFactory # access to variables and components of linux board, e.g.: wifi signal rssi, status LED color, ...
from eledio.device.srq import Srq   # interrupt system for quick access to variables which need intermediate handle, e.g.: RF433 receiver

```

### Load of _json_ file
* this part is required before fist try of attempt to variables with names form config _json_ part
#### Basic example of device registration
```
eledio = Eledio()
eledio.register_device_factory("i2c", PcuFactory(SMBus(0)))

# repeat for every configuration file
with open('config.json') as f:
    eledio.append_config(json.load(f))
```
## Readout values from the peripherals - sensors, to linux
Everytime, you want to read all of variables, e.g.: temperatures, digital input states, sensors values, ... you must to call ```eledio.load_inputs()```

After this calling, you can access to variables by their symbloc names in dict access.
For example:
```
eledio.load_inputs()

temperature = eledio["Temperature"]	# this statement put value of temperature after processing of linear equation (a and b coeficients) to the variable temperature

voltage = eledio["RE1Voltage"] # this statement put value of RMS voltage on relay 1 to variable voltage
```

## Put new values to the peripherals - actuators, connected to the hardware
For relay, digital outputs you are able to set states True/False. Used logic of every component is written in component description.
```
eledio["RE1"] = True # switch on relay 1
eledio["RE3"] = False # switch off relay 3

eledio.store_outputs()
```
After setting values for all of dict ```eledio``` parts, which you want to set, you need to call command ```eledio.store_outputs()```, which send that values to real hardware.
