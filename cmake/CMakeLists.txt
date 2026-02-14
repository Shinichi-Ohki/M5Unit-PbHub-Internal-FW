cmake_minimum_required(VERSION 3.22)

set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR ARM)

# Use ARM GNU Toolchain (installed via brew install --cask gcc-arm-embedded)
set(TOOLCHAIN_DIR /Applications/ArmGNUToolchain/15.2.rel1/arm-none-eabi)
set(CMAKE_C_COMPILER ${TOOLCHAIN_DIR}/bin/arm-none-eabi-gcc)
set(CMAKE_CXX_COMPILER ${TOOLCHAIN_DIR}/bin/arm-none-eabi-g++)
set(CMAKE_ASM_COMPILER ${TOOLCHAIN_DIR}/bin/arm-none-eabi-gcc)
set(CMAKE_OBJCOPY ${TOOLCHAIN_DIR}/bin/arm-none-eabi-objcopy)
set(CMAKE_SIZE ${TOOLCHAIN_DIR}/bin/arm-none-eabi-size)

set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

set(CMAKE_C_FLAGS_INIT "-mcpu=cortex-m0 -mthumb")
set(CMAKE_CXX_FLAGS_INIT "-mcpu=cortex-m0 -mthumb")
set(CMAKE_ASM_FLAGS_INIT "-mcpu=cortex-m0 -mthumb")
set(CMAKE_EXE_LINKER_FLAGS_INIT "-mcpu=cortex-m0 -mthumb")

project(PbHub C ASM)

set(OPT_LEVEL "-O2")
set(DEBUG_FLAGS "-g3")
set(WARN_FLAGS "-Wall -Wextra -Wpedantic")

set(CPU_FLAGS "-mcpu=cortex-m0 -mthumb")
set(FPU_FLAGS "-msoft-float")
set(DEFINES "-DUSE_FULL_LL_DRIVER -DUSE_HAL_DRIVER -DSTM32F030x6")

set(CMAKE_C_FLAGS "${CPU_FLAGS} ${FPU_FLAGS} ${OPT_LEVEL} ${DEBUG_FLAGS} ${WARN_FLAGS} ${DEFINES} -ffunction-sections -fdata-sections -std=c99")
set(CMAKE_ASM_FLAGS "${CPU_FLAGS} ${DEFINES} -x assembler-with-cpp")
set(CMAKE_EXE_LINKER_FLAGS "${CPU_FLAGS} -T${CMAKE_SOURCE_DIR}/stm32f030f4.ld -specs=nano.specs -specs=nosys.specs -Wl,--gc-sections -Wl,-Map=${PROJECT_NAME}.map -Wl,--print-memory-usage")

set(CMAKE_C_FLAGS_DEBUG "-Og -g3")
set(CMAKE_C_FLAGS_RELEASE "-O2 -g0")

set(CODE_DIR ${CMAKE_SOURCE_DIR}/../code)

include_directories(
    ${CODE_DIR}/Core/Inc
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Inc
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Inc/Legacy
    ${CODE_DIR}/Drivers/CMSIS/Device/ST/STM32F0xx/Include
    ${CODE_DIR}/Drivers/CMSIS/Include
)

set(SOURCES
    # Application
    ${CODE_DIR}/Core/Src/main.c
    ${CODE_DIR}/Core/Src/gpio.c
    ${CODE_DIR}/Core/Src/adc.c
    ${CODE_DIR}/Core/Src/i2c.c
    ${CODE_DIR}/Core/Src/i2c_ex.c
    ${CODE_DIR}/Core/Src/tim.c
    ${CODE_DIR}/Core/Src/flash.c
    ${CODE_DIR}/Core/Src/ws2812.c
    ${CODE_DIR}/Core/Src/stm32f0xx_it.c
    ${CODE_DIR}/Core/Src/stm32f0xx_hal_msp.c
    ${CODE_DIR}/Core/Src/system_stm32f0xx.c

    # HAL Driver
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_rcc.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_rcc_ex.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_gpio.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_dma.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_cortex.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_pwr.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_pwr_ex.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_flash.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_flash_ex.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_exti.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_i2c.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_i2c_ex.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_tim.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_tim_ex.c

    # LL Driver
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_ll_rcc.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_ll_utils.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_ll_exti.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_ll_i2c.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_ll_gpio.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_ll_dma.c
    ${CODE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/stm32f0xx_ll_adc.c

    # Startup
    ${CMAKE_SOURCE_DIR}/startup_stm32f030x6.s
)

add_executable(${PROJECT_NAME} ${SOURCES})

add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:${PROJECT_NAME}> ${PROJECT_NAME}.hex
    COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${PROJECT_NAME}> ${PROJECT_NAME}.bin
    COMMAND ${CMAKE_SIZE} $<TARGET_FILE:${PROJECT_NAME}>
    COMMENT "Building ${PROJECT_NAME}.hex and ${PROJECT_NAME}.bin"
)
