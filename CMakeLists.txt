cmake_minimum_required(VERSION 3.16)
project(ch32v003f4u6 C CXX ASM)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# IMPORTANT! change this path acording to your needs, you also need to change the .clangd file in order to add clangd support.
set(TOOLCHAIN_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../toolchain/RISC-V Embedded GCC12/bin")

set(TOOLCHAIN_PREFIX "${TOOLCHAIN_PATH}/riscv-wch-elf-")

set(CMAKE_C_COMPILER   "${TOOLCHAIN_PREFIX}gcc")
set(CMAKE_CXX_COMPILER "${TOOLCHAIN_PREFIX}g++")
set(CMAKE_ASM_COMPILER "${TOOLCHAIN_PREFIX}gcc")
set(CMAKE_OBJCOPY      "${TOOLCHAIN_PREFIX}objcopy")
set(CMAKE_SIZE         "${TOOLCHAIN_PREFIX}size")

set(ARCH_FLAGS -march=rv32ec -mabi=ilp32e -msmall-data-limit=0 -msave-restore)

#sources, add yours here
set(SOURCES
    CH32V_firmware_library/Startup/startup_ch32v00x.S
    CH32V_firmware_library/Core/core_riscv.c
    CH32V_firmware_library/Debug/debug.c
    CH32V_firmware_library/Peripheral/src/ch32v00x_adc.c
    CH32V_firmware_library/Peripheral/src/ch32v00x_dbgmcu.c
    CH32V_firmware_library/Peripheral/src/ch32v00x_dma.c
    CH32V_firmware_library/Peripheral/src/ch32v00x_exti.c
    CH32V_firmware_library/Peripheral/src/ch32v00x_flash.c
    CH32V_firmware_library/Peripheral/src/ch32v00x_gpio.c
    CH32V_firmware_library/Peripheral/src/ch32v00x_i2c.c
    CH32V_firmware_library/Peripheral/src/ch32v00x_iwdg.c
    CH32V_firmware_library/Peripheral/src/ch32v00x_misc.c
    CH32V_firmware_library/Peripheral/src/ch32v00x_opa.c
    CH32V_firmware_library/Peripheral/src/ch32v00x_pwr.c
    CH32V_firmware_library/Peripheral/src/ch32v00x_rcc.c
    CH32V_firmware_library/Peripheral/src/ch32v00x_spi.c
    CH32V_firmware_library/Peripheral/src/ch32v00x_tim.c
    CH32V_firmware_library/Peripheral/src/ch32v00x_usart.c
    CH32V_firmware_library/Peripheral/src/ch32v00x_wwdg.c
    User/main.c
    User/ch32v00x_it.c
    User/system_ch32v00x.c
)

add_executable(code_compilation ${SOURCES})

target_compile_options(code_compilation PRIVATE ${ARCH_FLAGS})
target_link_options(code_compilation PRIVATE ${ARCH_FLAGS})

target_include_directories(code_compilation PRIVATE
    CH32V_firmware_library/Peripheral/inc
    CH32V_firmware_library/Debug
    CH32V_firmware_library/Core
    User
)

# I tried to add whatevert mounriver got but might skipped something
target_compile_options(code_compilation PRIVATE
    -Os -g -fmessage-length=0 -fsigned-char
    -ffunction-sections -fdata-sections -fno-common
    -Wunused -Wuninitialized
    $<$<COMPILE_LANGUAGE:CXX>:-fno-exceptions -fno-rtti -fno-threadsafe-statics -std=gnu++17>
    $<$<COMPILE_LANGUAGE:C>:-std=gnu99>
    $<$<COMPILE_LANGUAGE:ASM>:-x assembler-with-cpp>
)

set(LINKER_SCRIPT "${CMAKE_SOURCE_DIR}/CH32V_firmware_library/Ld/Link.ld")
target_link_options(code_compilation PRIVATE
    -T${LINKER_SCRIPT} -nostartfiles -Xlinker --gc-sections
    --specs=nano.specs --specs=nosys.specs -lprintf
)
target_link_libraries(code_compilation PRIVATE c m)


add_custom_command(TARGET code_compilation POST_BUILD
    # gen alternative commands may not be battle tested
    COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:code_compilation> ${CMAKE_BINARY_DIR}/firmware.hex
    COMMAND ${CMAKE_OBJCOPY} -O binary -S $<TARGET_FILE:code_compilation> ${CMAKE_BINARY_DIR}/firmware.bin
    COMMAND ${CMAKE_SIZE} --format=berkeley $<TARGET_FILE:code_compilation>


# ALL MIGHTLY HACKY FIX, hijack compiled file sends it to project root as firmware.elf file and then copies monitorandflash bash script as it were the code. remove all the following commands to disable.
 #moves to root dir
 COMMAND ${CMAKE_COMMAND} -E rename $<TARGET_FILE:code_compilation> "${CMAKE_SOURCE_DIR}/firmware.elf"
# replaces target code for the flash_and_monitor.sh script located at root directory that script flashes and prints serial monitor
    COMMAND echo "#!/bin/bash" > $<TARGET_FILE:code_compilation>
    COMMAND echo "${CMAKE_SOURCE_DIR}/flash_and_monitor.sh ${CMAKE_SOURCE_DIR}/firmware.elf" >> $<TARGET_FILE:code_compilation>
    COMMAND chmod +x $<TARGET_FILE:code_compilation>
)
