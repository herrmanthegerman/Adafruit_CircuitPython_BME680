# Adafruit CircuitPython BME680

CircuitPython driver for the Bosch **BME680** environmental sensor
([Adafruit product 3660](https://www.adafruit.com/product/3660)).
It reads temperature, relative humidity, barometric pressure and gas
resistance, and derives altitude from the pressure. The sensor can be
connected over I2C or SPI.

The full reStructuredText documentation lives in [`README.rst`](README.rst)
and [`docs/`](docs/).

## Installation

On CircuitPython boards, copy `adafruit_bme680.py` and the
[Bus Device](https://github.com/adafruit/Adafruit_CircuitPython_BusDevice)
library to the board. The easiest way to get both is the
[Adafruit library bundle](https://github.com/adafruit/Adafruit_CircuitPython_Bundle).

On Linux single-board computers (e.g. Raspberry Pi), install from PyPI:

```shell
pip3 install adafruit-circuitpython-bme680
```

This also installs the dependencies `Adafruit-Blinka` and
`adafruit-circuitpython-busdevice`.

## Quick start

```python
import time
import board
import busio
import adafruit_bme680

i2c = busio.I2C(board.SCL, board.SDA)
sensor = adafruit_bme680.Adafruit_BME680_I2C(i2c)  # default address 0x77

# Set this to your location's sea-level pressure (hPa) for accurate altitude
sensor.sea_level_pressure = 1013.25

# The sensor heats itself a little; calibrate against a reference thermometer
temperature_offset = -5

while True:
    print("Temperature: %0.1f C" % (sensor.temperature + temperature_offset))
    print("Gas: %d ohm" % sensor.gas)
    print("Humidity: %0.1f %%" % sensor.humidity)
    print("Pressure: %0.3f hPa" % sensor.pressure)
    print("Altitude: %0.2f m" % sensor.altitude)
    time.sleep(1)
```

For SPI, use `Adafruit_BME680_SPI(spi, cs)` with a `busio.SPI` bus and a
`digitalio.DigitalInOut` chip-select pin (default baudrate 100 kHz).

## API overview

| Class | Constructor |
| --- | --- |
| `Adafruit_BME680_I2C` | `(i2c, address=0x77, debug=False, *, refresh_rate=10)` |
| `Adafruit_BME680_SPI` | `(spi, cs, baudrate=100000, debug=False, *, refresh_rate=10)` |

### Readings (read-only)

| Property | Unit |
| --- | --- |
| `temperature` | °C |
| `humidity` | % RH (clamped to 0–100) |
| `pressure` | hPa |
| `gas` | Ω (gas resistance) |
| `altitude` | m, computed from `pressure` and `sea_level_pressure` |

### Settings

| Property | Allowed values | Default |
| --- | --- | --- |
| `sea_level_pressure` | any float (hPa) | `1013.25` |
| `temperature_oversample` | 0, 1, 2, 4, 8, 16 | 8 |
| `pressure_oversample` | 0, 1, 2, 4, 8, 16 | 4 |
| `humidity_oversample` | 0, 1, 2, 4, 8, 16 | 2 |
| `filter_size` (IIR filter) | 0, 1, 3, 7, 15, 31, 63, 127 | 3 |

Invalid values raise `RuntimeError`.

## How it works

- On startup the driver soft-resets the chip, checks its chip ID (`0x61`),
  reads the factory calibration coefficients and configures the gas heater.
- Each property access triggers a single-shot ("forced mode") measurement of
  all channels, then applies Bosch's compensation formulas.
- `refresh_rate` limits how often a new measurement is taken (default: at most
  10 per second). Reads within that window return values from the previous
  measurement, so reading several properties in a row is cheap.
- Pass `debug=True` to print every raw register read and write.

## License

MIT, see [`LICENSE`](LICENSE).
