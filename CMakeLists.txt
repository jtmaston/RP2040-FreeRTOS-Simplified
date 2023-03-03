cmake_minimum_required(VERSION 3.14)

# Set project data
set(PROJECT_NAME "RP2040_FREERTOS_PROJECT_TEMPLATE")
set(BUILD_NUMBER "0")


# Set app name(s) and version(s)
set(APP_1_NAME "Firmware")
set(APP_1_VERSION_NUMBER "0")

# Specify the app(s) source code
set(APP_1_SRC_DIRECTORY "${CMAKE_SOURCE_DIR}/App-Template")

# FROM 1.3.0 -- Move common source code to a separate directory
set(COMMON_CODE_DIRECTORY "${CMAKE_SOURCE_DIR}/Common")

# Set app-side debugging "ON" or "OFF"
set(DO_DEBUG "ON")

# Set env variable 'PICO_SDK_PATH' to the local Pico SDK
# Comment out the set() if you have a global copy of the
# SDK set and $PICO_SDK_PATH defined in your $PATH
set(ENV{PICO_SDK_PATH} "${CMAKE_SOURCE_DIR}/pico-sdk")

# Set flags and directory variables
set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -DTSDEBUG")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DTSDEBUG")
set(FREERTOS_CFG_DIRECTORY "${CMAKE_SOURCE_DIR}/Config")
set(FREERTOS_SRC_DIRECTORY "${CMAKE_SOURCE_DIR}/FreeRTOS-Kernel")

# Include the Pico SDK
include(Config/pico_sdk_import.cmake)

# Name the project
project(${PROJECT_NAME}
        LANGUAGES C CXX ASM
        DESCRIPTION "Am patru meserii - pun faianta, bat la cuie ..."
)

# Initialise the Pico SDK
pico_sdk_init()

# Add FreeRTOS as a library
add_library(FreeRTOS STATIC
    ${FREERTOS_SRC_DIRECTORY}/event_groups.c
    ${FREERTOS_SRC_DIRECTORY}/list.c
    ${FREERTOS_SRC_DIRECTORY}/queue.c
    ${FREERTOS_SRC_DIRECTORY}/stream_buffer.c
    ${FREERTOS_SRC_DIRECTORY}/tasks.c
    ${FREERTOS_SRC_DIRECTORY}/timers.c
    ${FREERTOS_SRC_DIRECTORY}/portable/MemMang/heap_3.c
    ${FREERTOS_SRC_DIRECTORY}/portable/GCC/ARM_CM0/port.c
)

# Build FreeRTOS
target_include_directories(FreeRTOS PUBLIC
    ${FREERTOS_CFG_DIRECTORY}/
    ${FREERTOS_SRC_DIRECTORY}/include
    ${FREERTOS_SRC_DIRECTORY}/portable/GCC/ARM_CM0
)

# Include the apps' source code
# Should we set app-level debugging?
# NOTE Equivalent of `#define DEBUG 1`
if(${DO_DEBUG})
    add_compile_definitions(DEBUG=1)
    message(STATUS "App-side debugging enabled for ${APP_1_NAME}")
else()
    message(STATUS "App-side debugging disabled for ${APP_1_NAME}")
endif()

# Make project data accessible to compiler
add_compile_definitions(APP_NAME="${APP_1_NAME}")
add_compile_definitions(APP_VERSION="${APP_1_VERSION_NUMBER}")
add_compile_definitions(BUILD_NUM=${BUILD_NUMBER})

# Include app source code file(s)
add_executable(${APP_1_NAME}
        App/main.c
        )

# Link to built libraries
target_link_libraries(${APP_1_NAME} LINK_PUBLIC
        pico_stdlib
        FreeRTOS)

# Enable/disable STDIO via USB and UART
pico_enable_stdio_usb(${APP_1_NAME} 1)
pico_enable_stdio_uart(${APP_1_NAME} 1)

# Enable extra build products
pico_add_extra_outputs(${APP_1_NAME})
set(CMAKE_INSTALL_PREFIX E:/)
install(FILES ${CMAKE_BINARY_DIR}/Firmware.uf2 DESTINATION E:/)

