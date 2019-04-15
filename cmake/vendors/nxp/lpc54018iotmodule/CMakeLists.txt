cmake_minimum_required(VERSION 3.13)

set(lpc54018iotmodule_dir "${AFR_3RDPARTY_DIR}/mcu_vendor/nxp/LPC54018")
set(board_demos_dir "${AFR_ROOT_DIR}/demos/nxp/lpc54018iotmodule/common")
set(board_tests_dir "${AFR_ROOT_DIR}/tests/nxp/lpc54018iotmodule/common")

if(AFR_IS_TESTING)
    set(board_dir "${board_tests_dir}")
else()
    set(board_dir "${board_demos_dir}")
endif()

# -------------------------------------------------------------------------------------------------
# Amazon FreeRTOS Console metadata
# -------------------------------------------------------------------------------------------------

afr_set_board_metadata(ID "NXP-LPC54018-IoT-Module")
afr_set_board_metadata(DISPLAY_NAME "LPC54018 IoT Module")
afr_set_board_metadata(DESCRIPTION "Development kit for ARM® Cortex®-M4 based LPC54018 MCU")
afr_set_board_metadata(VENDOR_NAME "NXP")
afr_set_board_metadata(FAMILY_NAME "LPC5401x_54S0xx")
afr_set_board_metadata(CODE_SIGNER "null")
afr_set_board_metadata(DATA_RAM_MEMORY "360KB")
afr_set_board_metadata(PROGRAM_MEMORY "128MB")
afr_set_board_metadata(SUPPORTED_IDE "IAREmbeddedWorkbench;MCUXpresso")
afr_set_board_metadata(IDE_IAREmbeddedWorkbench_NAME "IAR Embedded Workbench")
afr_set_board_metadata(IDE_IAREmbeddedWorkbench_COMPILER "IAR")
afr_set_board_metadata(IDE_IAREmbeddedWorkbench_RECOMMENDED "true")
afr_set_board_metadata(IDE_MCUXpresso_NAME "MCUXpresso")
afr_set_board_metadata(IDE_MCUXpresso_COMPILER "GCC")
afr_set_board_metadata(IDE_MCUXpresso_RECOMMENDED "false")

# -------------------------------------------------------------------------------------------------
# Compiler settings
# -------------------------------------------------------------------------------------------------
afr_mcu_port(compiler)
set(
    defined_symbols
    __REDLIB__
    MXL12835F
    __USE_CMSIS
    BOARD_DEBUG_UART_TYPE=DEBUG_CONSOLE_DEVICE_TYPE_USBCDC
    IMG_BAUDRATE=96000000
    XIP_IMAGE
    BOARD_USE_VIRTUALCOM
    USB_STACK_USE_DEDICATED_RAM=1
    SDK_DEBUGCONSOLE=1
    CPU_LPC54018JET180=1
    PRINTF_FLOAT_ENABLE=0
    USB_STACK_FREERTOS
    USE_RTOS=1
    FSL_RTOS_FREE_RTOS
    A_LITTLE_ENDIAN
    CR_INTEGER_PRINTF
    USB_DEVICE_CONFIG_LPCIP3511HS=1
    __MCUXPRESSO
    DEBUG
    SDK_OS_FREE_RTOS
    CPU_LPC54018JET180_cm4
    CPU_LPC54018JET180
)
target_compile_definitions(
    AFR::compiler::mcu_port
    INTERFACE $<$<NOT:$<COMPILE_LANGUAGE:ASM>>:${defined_symbols}>
)

set(common_flags "-mcpu=cortex-m4" "-mfpu=fpv4-sp-d16" "-mfloat-abi=hard" "-mthumb")
set(c_flags "-std=gnu99" "-O0" "-fno-common" "-g" "-Wall" "-c" "-ffunction-sections" "-fdata-sections" "-ffreestanding" "-fno-builtin" "-fomit-frame-pointer")
set(asm_flags "-c" "-x" "assembler-with-cpp")

# Compiler flags
target_compile_options(
    AFR::compiler::mcu_port  
    INTERFACE
        ${common_flags}
        $<$<NOT:$<COMPILE_LANGUAGE:ASM>>:${c_flags}>
        -D__REDLIB__ -specs=redlib.specs -MMD -MP
)

target_compile_options(
    AFR::compiler::mcu_port        
    INTERFACE
        ${common_flags}
        $<$<NOT:$<COMPILE_LANGUAGE:C>>:${asm_flags}>
)

#file(COPY "${lpc54018iotmodule_dir}/mcuxpresso/aws_demos_Debug.ld" DESTINATION . )
#file(COPY "${lpc54018iotmodule_dir}/mcuxpresso/aws_demos_Debug_memory.ld" DESTINATION . )
#file(COPY "${lpc54018iotmodule_dir}/mcuxpresso/aws_demos_Debug_library.ld" DESTINATION . )


# Linker flags
target_link_options(
    AFR::compiler::mcu_port
    INTERFACE 
        -nostdlib
        "SHELL:-Xlinker -print-memory-usage" "SHELL:-Xlinker --gc-sections" "SHELL:-Xlinker -Map=aws_demos.map"
        ${common_flags}
        -T "aws_demos_Debug.ld"
)

target_link_directories(
    AFR::compiler::mcu_port
    INTERFACE
         "${lpc54018iotmodule_dir}/mcuxpresso" 
)

target_link_libraries(
    AFR::compiler::mcu_port
    INTERFACE
        "${lpc54018iotmodule_dir}/mcuxpresso/libpower_hardabi.a"
)

# -------------------------------------------------------------------------------------------------
# Amazon FreeRTOS portable layers
# -------------------------------------------------------------------------------------------------

set(vendor "nxp")
set(board "lpc54018iotmodule")
set(portable_dir "portable/${vendor}/${board}")

afr_glob_src(lpc54018_src DIRECTORY "${lpc54018iotmodule_dir}")
afr_glob_src(drivers_src DIRECTORY "${lpc54018iotmodule_dir}/drivers")
afr_glob_src(mcuxpresso_src DIRECTORY "${lpc54018iotmodule_dir}/mcuxpresso")
afr_glob_src(usb_device_src DIRECTORY "${lpc54018iotmodule_dir}/middleware/usb/device")
afr_glob_src(wifi_qca_src DIRECTORY "${lpc54018iotmodule_dir}/middleware/wifi_qca" RECURSE)
afr_glob_src(utilities_src DIRECTORY "${lpc54018iotmodule_dir}/utilities" RECURSE)

#message(WARN "${wifi_qca_src}")

# Kernel
afr_mcu_port(kernel)

target_sources(
    AFR::kernel::mcu_port
    INTERFACE
        ${lpc54018_src}
        ${drivers_src}
        ${mcuxpresso_src}
        ${usb_device_src}
        ${wifi_qca_src}
        ${utilities_src}
        ${lpc54018iotmodule_dir}/middleware/usb/osa/usb_osa_freertos.c
        "${AFR_MODULES_DIR}/FreeRTOS/portable/GCC/ARM_CM4F/port.c"
        "${AFR_MODULES_DIR}/FreeRTOS/portable/GCC/ARM_CM4F/portmacro.h"
        "${AFR_MODULES_DIR}/FreeRTOS/portable/MemMang/$<IF:${AFR_IS_TESTING},heap_4.c,heap_5.c>"
)
set(
    kernel_inc_dirs
    "${AFR_MODULES_DIR}/FreeRTOS/portable/GCC/ARM_CM4F"
    "${lpc54018iotmodule_dir}"
    "${lpc54018iotmodule_dir}/drivers"
    "${lpc54018iotmodule_dir}/cmsis_drivers"
    "${lpc54018iotmodule_dir}/middleware/wifi_qca"
    "${lpc54018iotmodule_dir}/middleware/wifi_qca/common_src/hcd"
    "${lpc54018iotmodule_dir}/middleware/wifi_qca/common_src/include"
    "${lpc54018iotmodule_dir}/middleware/wifi_qca/common_src/stack_common"
    "${lpc54018iotmodule_dir}/middleware/wifi_qca/common_src/wmi"
    "${lpc54018iotmodule_dir}/middleware/wifi_qca/custom_src/include"
    "${lpc54018iotmodule_dir}/middleware/wifi_qca/custom_src/stack_custom"
    "${lpc54018iotmodule_dir}/middleware/wifi_qca/include"
    "${lpc54018iotmodule_dir}/middleware/wifi_qca/include/AR6002"
    "${lpc54018iotmodule_dir}/middleware/wifi_qca/include/AR6002/hw2.0/hw"
    "${lpc54018iotmodule_dir}/middleware/wifi_qca/port"
    "${lpc54018iotmodule_dir}/middleware/wifi_qca/port/boards/lpc54018iotmodule/freertos"
    "${lpc54018iotmodule_dir}/middleware/wifi_qca/port/boards/lpc54018iotmodule/freertos/gt202"
    "${lpc54018iotmodule_dir}/middleware/wifi_qca/port/drivers/flexcomm_freertos"
    "${lpc54018iotmodule_dir}/middleware/wifi_qca/port/env/freertos"
    "${lpc54018iotmodule_dir}/middleware/usb/device"
    "${lpc54018iotmodule_dir}/middleware/usb/include"
    "${lpc54018iotmodule_dir}/middleware/usb/osa"
    "${lpc54018iotmodule_dir}/utilities"
    "${lpc54018iotmodule_dir}/utilities/io"
    "${lpc54018iotmodule_dir}/utilities/log"
    "${lpc54018iotmodule_dir}/utilities/str"
    "${lpc54018iotmodule_dir}/CMSIS/Include"
    "${AFR_MODULES_DIR}/FreeRTOS/portable/CCS/ARM_CM3"
    "${board_dir}/config_files"
    "${board_dir}/application_code/nxp_code"
    # Need aws_clientcredential.h
    "$<IF:${AFR_IS_TESTING},${AFR_TESTS_DIR},${AFR_DEMOS_DIR}>/include"

)
target_include_directories(
    AFR::kernel::mcu_port
    INTERFACE $<$<NOT:$<COMPILE_LANGUAGE:ASM>>:${kernel_inc_dirs}>
)

# WiFi
afr_mcu_port(wifi)
target_sources(
    AFR::wifi::mcu_port
    INTERFACE "${AFR_MODULES_DIR}/wifi/${portable_dir}/aws_wifi.c"
)

# PKCS11
afr_mcu_port(pkcs11)
target_sources(
    AFR::pkcs11::mcu_port
    INTERFACE 
        "${AFR_MODULES_DIR}/pkcs11/${portable_dir}/aws_pkcs11_pal.c"
        "${AFR_MODULES_DIR}/pkcs11/${portable_dir}/hw_poll.c"
)
# Link to AFR::pkcs11_mbedtls if you want to use default implementation based on mbedtls.
target_link_libraries(
    AFR::pkcs11::mcu_port
    INTERFACE 
        3rdparty::mbedtls
        AFR::pkcs11_mbedtls
)

# Secure sockets
afr_mcu_port(secure_sockets)
target_link_libraries(
    AFR::secure_sockets::mcu_port
    INTERFACE AFR::tls
)
target_sources(
    AFR::secure_sockets::mcu_port
    INTERFACE "${AFR_MODULES_DIR}/secure_sockets/${portable_dir}/aws_secure_sockets.c"
)

# -------------------------------------------------------------------------------------------------
# Amazon FreeRTOS demos and tests
# -------------------------------------------------------------------------------------------------
set(CMAKE_EXECUTABLE_SUFFIX ".axf")

if(AFR_IS_TESTING)
    set(exe_target aws_tests)
else()
    set(exe_target aws_demos)
endif()

afr_glob_src(board_code_src DIRECTORY "${board_dir}/application_code/nxp_code")

add_executable(
    ${exe_target}
    ${board_code_src}
    "${board_dir}/application_code/main.c"
    "${AFR_MODULES_DIR}/pkcs11/${portable_dir}/hw_poll.c"
    "${lpc54018iotmodule_dir}/mcuxpresso/startup_lpc54018.c"
)

target_link_libraries(
    ${exe_target}
    PRIVATE
        AFR::kernel
        AFR::wifi
        AFR::utils
)

find_program(gcc_objcopy arm-none-eabi-objcopy)

if(NOT gcc_objcopy)
    message(FATAL_ERROR "Cannot find arm-none-eabi-objcopy.")
endif()

set(output_file "$<TARGET_FILE_DIR:${exe_target}>/${exe_target}.hex")
add_custom_command(
    TARGET ${exe_target} POST_BUILD
    COMMAND "${gcc_objcopy}" -v -O ihex "$<TARGET_FILE:${exe_target}>" "${output_file}"
)

