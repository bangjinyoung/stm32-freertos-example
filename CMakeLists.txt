cmake_minimum_required(VERSION 3.22.1)

set(CMAKE_C_COMPILER_WORKS TRUE)

enable_language(C ASM)
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Edit project name
project(stm32-freertos-example)

# Edit CPU options
set(CPU_OPTIONS
        -mcpu=cortex-m4
        -mthumb
        -mfpu=fpv4-sp-d16
        -mfloat-abi=hard
)

# Edit Link script file
set(LINK_SCRIPT ${CMAKE_CURRENT_SOURCE_DIR}/STM32L475VGTx_FLASH.ld)

# Add C source files
file(GLOB C_SOURCES CONFIGURE_DEPENDS
        Core/Src/*.c
        Drivers/STM32L4xx_HAL_Driver/Src/*.c
)

# Add assembly files
set(ASM_SOURCES
        startup_stm32l475xx.s
)

set(EXECUTABLE ${PROJECT_NAME}.elf)

add_executable(${EXECUTABLE} ${C_SOURCES} ${ASM_SOURCES})

# Add defines
target_compile_definitions(${EXECUTABLE} PRIVATE
        USE_HAL_DRIVER
        STM32L475xx
)

# Add includes paths
target_include_directories(${EXECUTABLE} PRIVATE
        Core/Inc
        Drivers/STM32L4xx_HAL_Driver/Inc
        Drivers/STM32L4xx_HAL_Driver/Inc/Legacy
        Drivers/CMSIS/Device/ST/STM32L4xx/Include
        Drivers/CMSIS/Include
)

# Add compile options
target_compile_options(${EXECUTABLE} PRIVATE
        ${CPU_OPTIONS}
        
        -Og

        -Wall
        -fdata-sections
        -ffunction-sections

        -g 
        -gdwarf-2
)

# Add link options
target_link_options(${EXECUTABLE} PRIVATE
        ${CPU_OPTIONS}

        -specs=nano.specs

        -T${LINK_SCRIPT}
        -Wl,-Map=${PROJECT_NAME}.map,--cref
        -Wl,--gc-sections
        -Wl,--no-warn-rwx-segments
        -Wl,--print-memory-usage
)

# Print file size
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND ${CMAKE_SIZE_UTIL} ${EXECUTABLE})

# Output to hex and binary files
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} -O ihex ${EXECUTABLE} ${PROJECT_NAME}.hex
        COMMAND ${CMAKE_OBJCOPY} -O binary ${EXECUTABLE} ${PROJECT_NAME}.bin)