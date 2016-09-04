# micropython-mcp4725
A [micropython](http://micropython.org) driver for the MCP4725 I²C DAC.

The MCP4725 is a digital to analog converter chip with a 12-Bit resolution on
the output. The MCP4725 works with a supply voltage from 3.3V to 5V. The output  
range is from 0V up to the supply voltage. 

The MCP4725 can be configured to two different addresses (0x62,0x63) on the I²C bus.
That way it is possible to use 2 MCP4725 on any I²C bus. The device supports
standard (100kbps) and fast (400kbps) and hi-speed (3.4Mbps) bus speeds. But
hi-speed seems not to be supported by any of the micropython hardware boards.
The fast baudrate of 400kbps works fine with the 
[WiPy](https://www.pycom.io/solutions/py-boards/wipy/) but compared with DACs
that are connected to a SPI-bus the MCP4725 update on the output voltage are pretty slow.

The MCP4725 supports 3 different power-down modes where the output voltage
driver shuts down and the device goes to sleep to save energy. The cips wakes up
from power-down mode whenever a output voltage update is send to the device.

The MCP4725 also has a small eeprom where the power-down mode and the initial
output voltage can be configured that are to be used when the MCP4725 is powered
up.

##Using the MCP4725 in your micropython project
You need only a few lines of code to add a MCP4725 to your project.

###Create and initialze the I²C bus of your micropython board
A micropython driver for an I²C device expects you to create and initialze a
``machine.I2C`` instance and pass that to the constructor of the driver code.

The arguments used in the code example below work for a WiPy board. If you try this with a
different board please check the pins to use for the I²C bus. 

```python

from machine import I2C
import mcp4725

#create a I2C bus
i2c=I2C(0,I2C.MASTER,baudrate=400000,pins=('GP15','GP14')) 

#create the MCP4725 driver
dac=mcp4725.MCP4725(i2c,mcp4725.BUS_ADDRESS[0])
```

###Update the output on the MCP4725
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

###Configure the MCP4725
TODO: write this chapter

###Read the settings and the output voltage of the MCP4725
The MCP4725 support a single read command that returns the current configuration as well as the configuration stored in the eeprom of the device. 
```python
>>>result=dac.read()
>>>print(result)
>>>(False,0,300,0,200)
```
The method returns a tuple with 5 items. The first one is a flag that tells you
wether a previous call to ``config()`` that included an update to the eeprom of
the DAC has finshed. If ``result[0]`` is ``False``` the eeprom was updated. if
``True`` the write to the eeprom is still in progress. 

The second item in the tuple is the current power-down-mode the device is in.
The third item is the current setting of the DAC-output.
The fourth item is the power-down-mode the device will be in on startup.
The fifth and last item is the output voltage of the device on startup. 

