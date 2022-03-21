TODO: nicer readme.

## Building

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

You don't need to worry about initializing and setting up the I2C peripheral, the `ssd1306` takes care of it already.  

Make sure, however, that these files, part of the STM32 HAL Driver, are compiled with your project:

```
STM32F3xx_HAL_Driver/Inc/stm32f3xx_hal_i2c.h
STM32F3xx_HAL_Driver/Inc/stm32f3xx_hal_i2c_ex.h
STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_i2c.c
STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_i2c_ex.c
```

PS: If you use CubeMX to generate the project's boilerplate, check in `Core/Inc/stm32f3xx_hal_conf.h` that the i2c header file is indeed included:

```c
#define HAL_I2C_MODULE_ENABLED

// ...

#ifdef HAL_I2C_MODULE_ENABLED
 #include "stm32f3xx_hal_i2c.h"
#endif /* HAL_I2C_MODULE_ENABLED */
```
