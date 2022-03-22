# Using the SSD1306 display driver with an STM32 F3 board

This repo shows how to use (a fork of) the [ssd1306](https://github.com/dehre/ssd1306) library to drive a display over I2C.  
Here the STM32 F3 Discovery board is used, but the library works with other F3 boards too.

For the CMake setup I used, check out this [repo](https://github.com/MaJerle/stm32-cube-cmake-vscode).

## Wiring up

By default, the `ssd1306` library uses PB6 and PB7 as SCL and SDA pins, with internal pull-up resistors.

```c
// main.c
#include "ssd1306.h"

ssd1306_128x32_i2c_init(); // or whatever display you're using
```

```
   _________________________                        
  |           ______________|                       ______________
  |          | I2C1         |                      |              |
  |          |      SCL(PB6)|______________________|SCL           |
  |          |              |                      |              |
  |          |      SDA(PB7)|______________________|SDA           |
  |          |______________|                      |              |
  |                         |                      |              |
  |                      3V |______________________|VCC           |
  |                         |                      |              |
  |                      GND|______________________|GND           |
  |                         |                      |              |
  |_STM32F3_________________|                      |_OLED_Display_|

```

Alternatively, you can use PA9 and PA10 as SCL and SDA pins, if I2C2 is available on your board. To use these pins, replace `ssd1306_128x32_i2c_init()` with a call to these 2 functions shown below.

```c
#include "ssd1306.h
#include "intf/i2c/ssd1306_i2c.h"

ssd1306_i2cInitEx(1, 1, 0); // third param is the i2c address of the lcd display; use 0 to leave default
ssd1306_128x32_init();      // or whatever display you're using
```

```
   _________________________                        
  |           ______________|                       ______________
  |          | I2C2         |                      |              |
  |          |      SCL(PA9)|______________________|SCL           |
  |          |              |                      |              |
  |          |     SDA(PA10)|______________________|SDA           |
  |          |______________|                      |              |
  |                         |                      |              |
  |                      3V |______________________|VCC           |
  |                         |                      |              |
  |                      GND|______________________|GND           |
  |                         |                      |              |
  |_STM32F3_________________|                      |_OLED_Display_|
```

## Building & Flashing

```sh
# Configure CMake
cmake \
    --no-warn-unused-cli \
    -DCMAKE_EXPORT_COMPILE_COMMANDS:BOOL=TRUE \
    -DCMAKE_BUILD_TYPE:STRING=Debug \
    -DCMAKE_TOOLCHAIN_FILE:FILEPATH=gcc-arm-none-eabi.cmake \
    -Bbuild \
    -G Ninja

# Build
cmake --build build -j 8

# Flash
STM32_Programmer_CLI --connect port=swd --download build/stm32f3-ssd1306-display.elf -hardRst
```

## STM32 HAL Driver support

You don't need to worry about initializing and setting up the I2C peripheral, the `ssd1306` library takes care of it already.  

Make sure, however, that you include the STM32 HAL Driver library in your project; in particular, these files should be present:

```
STM32F3xx_HAL_Driver/Inc/stm32f3xx_hal_i2c.h
STM32F3xx_HAL_Driver/Inc/stm32f3xx_hal_i2c_ex.h
STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_i2c.c
STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_i2c_ex.c
```

PS: If you use CubeMX to generate the project's boilerplate, check that the i2c header file is indeed included in `Core/Inc/stm32f3xx_hal_conf.h`:

```c
#define HAL_I2C_MODULE_ENABLED

// ...

#ifdef HAL_I2C_MODULE_ENABLED
 #include "stm32f3xx_hal_i2c.h"
#endif /* HAL_I2C_MODULE_ENABLED */
```
