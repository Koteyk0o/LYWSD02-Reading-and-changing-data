# Get temperature and humidity data
Service: **ebe0ccb0-7a0a-4b0c-8a1a-6ff2997da3a6**

Characteristic: **ebe0ccc1-7a0a-4b0c-8a1a-6ff2997da3a6**

Write 0x01 to characteristic descriptor, and wait for notification with data.

First 2 bytes with temperature data, 3 byte with humidity data.
```python
class sensor_Delegate(btle.DefaultDelegate):
    def handleNotification(self, cHandle, data):
        humidity_byte = data[2]
        temp_bytes = data[:2]
        humidity = humidity_byte
        temperature = ((int.from_bytes(temp_bytes, byteorder='little')) / 100)
        
        print('Temperature: ' + str(temperature) + '°C', 'Humidity: ' + str(humidity) + '%')
```

This provides roughly the following data:
```python
Temperature: 28.84°C Humidity: 39%
Temperature: 28.85°C Humidity: 39%
Temperature: 28.82°C Humidity: 39%
```
# Get battery charge level
Service: **ebe0ccb0-7a0a-4b0c-8a1a-6ff2997da3a6**

Characteristic: **ebe0ccc4-7a0a-4b0c-8a1a-6ff2997da3a6**

Just read characteristic, characteristic provide 1 byte of data, with approximately charge level.

    Characteristic: ebe0ccc4-7a0a-4b0c-8a1a-6ff2997da3a6
        READ
        b'N'

```python
int.from_bytes(b'N', byteorder='little') = 78% battery charge level
```
# Changing Fahrenheit to Celsius
Service: **ebe0ccb0-7a0a-4b0c-8a1a-6ff2997da3a6**

Characteristic: **ebe0ccbe-7a0a-4b0c-8a1a-6ff2997da3a6**

Just write 1 byte to characteristic.

For Fahrenheit:
```python
(0x01).to_bytes(1, byteorder = 'little')
```
For Celsius:
```python
(0xff).to_bytes(1, byteorder = 'little')
```
# Reading and changing time
Service: **ebe0ccb0-7a0a-4b0c-8a1a-6ff2997da3a6**

Characteristic: **ebe0ccb7-7a0a-4b0c-8a1a-6ff2997da3a6**

For changing just write 5 bytes to characteristic.

This is a unix timestamp that is wrapped backwards and has a timezone byte at the end.
You can be convinced of it if you read bytes in this characteristic.

Read bytes (big-endian byte order) and convert it from bytes to hex.

You get `[95][2F][6A][5D][03]`, Rotate it.

And you get `[5D][6A][2F][95]` with `[03]` at the beginning, which means +3 hours relative to UTC.

In decimals `[5D][6A][2F][95]` mean `1567240085`.

We convert this unix epoch time into a normal human language, and we get `Saturday, 31 August 2019 year, 8:28:05`

If we add that last byte with the time zone, we will get the real time at which the data was received `Saturday, 31 August 2019 year, 11:28:05`
