
A python raspberry pi driver for the MCP4725 I²C DAC.

The MCP4725 is a digital to analog converter chip with a 12-Bit resolution on
the output. The MCP4725 works with a supply voltage from 3.3V to 5V. The output range runs from 0V up to the supply voltage. 

The MCP4725 can be configured to use one of two different addresses (0x60,0x61) on the I²C bus.
That way it is possible to use 2 MCP4725 on any I²C bus. The device supports
standard (100kbps) and fast (400kbps) and hi-speed (3.4Mbps) bus speeds.

The MCP4725 supports 3 different power-down modes where the output voltage
driver shuts down and the device goes to sleep to save energy. The chips wakes up
from power-down mode whenever a output voltage update is send to the device.

The MCP4725 also has a small eeprom where the power-down mode and the initial
output voltage can be configured that are to be used when the MCP4725 is powered
up.


###   Create and initialze the device on the I²C bus of your micropython board
import busio

```python

import busio
import board
import mcp4725

#create a I2C bus
i2c = busio.I2C(board.SCL, board.SDA)

#create the MCP4725 driver
dac = mcp4725.MCP4725(i2c, 0x60)
```

###   Update the output on the MCP4725
The simple way to update the output on the DAC is to write a new value to the
device
```python
dac.write(1200)
```
The actual voltage on the output depends on the supply voltage the powers the
MCP4725. If it runs on 3.3V the above command would drive the output to
``(3300/4096)*1200 = 996mV`` if the DAC is powered with 5V the output will be
``1464mV``. If the argument to the ``write(value)`` is negative the output will
be set to ``0V``. If the value argument is bigger than 4095 the output 
will be set to the maximum output voltage of the DAC. 

On power-up the MCP4725 will be initialized with the value read from the
internal eeprom of the device.

###   Configure the MCP4725
The power-down mode (see
[Datasheet](http://www.microchip.com/wwwproducts/en/en532229)) and the output
value of the DAC can be configured for the active session and for future
sessions by saving the settings in an internal eeprom of the DAC.

Power-down modes are selected by the keys in the POWER_DOWN_MODE dict.
```python
POWER_DOWN_MODE = {'Off':0, '1k':1, '100k':2, '500k':3}
```

```python
#configure the DAC to go into power-down mode and set the output value to maximum output.
dac.config('100k',4096)
```
After running the command the output value will is set to its maximum, but since the
DAC is in power-down mode this will not be measurable on the output pin.

```python
#configure the DAC to output ``Vdd/2`` on power-up or after a reset
dac.config('Off',2048, eeprom=True)
```
This configuration will be saved in the eeprom of the DAC and will be used
everytime the DAC is powered up or reset.

###   Read settings and output voltage of the MCP4725
The MCP4725 support a single read command that returns the current configuration as well as the configuration stored in the eeprom of the device. 
```python
result=dac.read()
print(result)

>>>(False,'Off',300,'1k',200)
```
The method returns a tuple with 5 items. 

1. The busy-flag of the eeprom on the dac. If ``True`` the DAC is busy writing
   the values for power-down mode and the startup output value to its internal
eeprom. If ``False`` the DAC is ready for a new config setting.
2. The current power-down configuration of the DAC. Returns a string with the
   active setting. (see
[Datasheet](http://www.microchip.com/wwwproducts/en/en532229))
3. The current outout value
4. The power-down configuration stored in the eeprom. This setting takes effect
   when the DAC is reset or powered up.
5. The output value  configuration stored in the eeprom. This setting takes effect
   when the DAC is reset or powered up.

The result shown in the code example will read as

* Write to eeprom of the chip has finshed
* The power-down mode is ``'Off'``, the DACs output on.
* The output is set to ``300/4096`` of the supply voltage of the DAC.
* On powerup or after a reset the DAC will go into power-down mode ``'1k'`` (see
[Datasheet](http://www.microchip.com/wwwproducts/en/en532229)).
* The output voltage on startup or after a reset will be ``200/4096`` of the supply
  voltage.
