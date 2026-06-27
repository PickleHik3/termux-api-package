cmake_minimum_required(VERSION 3.0.0)
project(termux-api)
include(GNUInstallDirs)

set(TERMUX_PREFIX ${CMAKE_INSTALL_PREFIX})

set(TERMUX_API_APP_PACKAGE_NAME "com.termux.api" CACHE STRING
    "Termux:API app package name")
set(TERMUX_API_APP_JAVA_NAMESPACE "com.termux.api" CACHE STRING
    "Termux:API app Java namespace (package of TermuxApiReceiver and KeepAliveService)")
set(LISTEN_SOCKET_ADDRESS "${TERMUX_API_APP_PACKAGE_NAME}://listen" CACHE STRING
    "Termux:API listen socket abstract namespace address")
set(TERMUX_API_COMPONENT "${TERMUX_API_APP_PACKAGE_NAME}/${TERMUX_API_APP_JAVA_NAMESPACE}.TermuxApiReceiver" CACHE STRING
    "Termux:API broadcast receiver component for am broadcast")
set(TERMUX_API_SERVICE_COMPONENT "${TERMUX_API_APP_PACKAGE_NAME}/${TERMUX_API_APP_JAVA_NAMESPACE}.KeepAliveService" CACHE STRING
    "Termux:API keep-alive service component for am startservice/stopservice")

add_library(termux-api SHARED termux-api.c)
add_library(termux-api-static STATIC termux-api.c)
set_target_properties(termux-api-static PROPERTIES OUTPUT_NAME termux-api)

set(TERMUX_API_COMPILE_DEFINITIONS
    TERMUX_API_APP_PACKAGE_NAME="${TERMUX_API_APP_PACKAGE_NAME}"
    TERMUX_API_APP_JAVA_NAMESPACE="${TERMUX_API_APP_JAVA_NAMESPACE}"
    LISTEN_SOCKET_ADDRESS="${LISTEN_SOCKET_ADDRESS}"
    TERMUX_API_COMPONENT="${TERMUX_API_COMPONENT}")

target_compile_definitions(termux-api PRIVATE ${TERMUX_API_COMPILE_DEFINITIONS})
target_compile_definitions(termux-api-static PRIVATE ${TERMUX_API_COMPILE_DEFINITIONS})

add_executable(termux-api-broadcast termux-api-broadcast.c)
target_link_libraries(termux-api-broadcast termux-api-static)

# TODO: get list through regex or similar
set(script_files
  scripts/termux-api-start
  scripts/termux-api-stop
  scripts/termux-audio-info
  scripts/termux-battery-status
  scripts/termux-brightness
  scripts/termux-call-log
  scripts/termux-camera-info
  scripts/termux-camera-photo
  scripts/termux-clipboard-get
  scripts/termux-clipboard-set
  scripts/termux-contact-list
  scripts/termux-dialog
  scripts/termux-download
  scripts/termux-fingerprint
  scripts/termux-infrared-frequencies
  scripts/termux-infrared-transmit
  scripts/termux-job-scheduler
  scripts/termux-keystore
  scripts/termux-location
  scripts/termux-media-player
  scripts/termux-media-scan
  scripts/termux-microphone-record
  scripts/termux-nfc
  scripts/termux-notification
  scripts/termux-notification-channel
  scripts/termux-notification-list
  scripts/termux-notification-remove
  scripts/termux-saf-create
  scripts/termux-saf-dirs
  scripts/termux-saf-ls
  scripts/termux-saf-managedir
  scripts/termux-saf-mkdir
  scripts/termux-saf-read
  scripts/termux-saf-rm
  scripts/termux-saf-stat
  scripts/termux-saf-write
  scripts/termux-sensor
  scripts/termux-share
  scripts/termux-sms-inbox
  scripts/termux-sms-list
  scripts/termux-sms-send
  scripts/termux-speech-to-text
  scripts/termux-storage-get
  scripts/termux-telephony-call
  scripts/termux-telephony-cellinfo
  scripts/termux-telephony-deviceinfo
  scripts/termux-toast
  scripts/termux-torch
  scripts/termux-tts-engines
  scripts/termux-tts-speak
  scripts/termux-usb
  scripts/termux-vibrate
  scripts/termux-volume
  scripts/termux-wallpaper
  scripts/termux-wifi-connectioninfo
  scripts/termux-wifi-enable
  scripts/termux-wifi-scaninfo
)

make_directory(scripts)
foreach(file ${script_files})
  configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/${file}.in
    ${file} @ONLY
  )
endforeach()

configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/termux-callback.in
  termux-callback @ONLY
)

install(
  FILES ${CMAKE_BINARY_DIR}/termux-api-broadcast
  DESTINATION ${CMAKE_INSTALL_PREFIX}/libexec
  PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE
    GROUP_READ GROUP_EXECUTE
    WORLD_READ WORLD_EXECUTE
)

# Create a symlink for termux-api-broadcast->termux-api for backwards
# compatibility
INSTALL(CODE "execute_process( \
  COMMAND ${CMAKE_COMMAND} -E create_symlink \
  termux-api-broadcast \
  ${CMAKE_INSTALL_PREFIX}/libexec/termux-api \
  )"
)

install(
  FILES
    ${CMAKE_BINARY_DIR}/libtermux-api.so
    ${CMAKE_BINARY_DIR}/libtermux-api.a
  TYPE LIB
)

install(
  FILES ${CMAKE_CURRENT_SOURCE_DIR}/termux-api.h
  TYPE INCLUDE
)

foreach(file ${script_files})
  install(
    FILES ${CMAKE_BINARY_DIR}/${file}
    TYPE BIN
    PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE
  )
endforeach()

install(
  FILES ${CMAKE_BINARY_DIR}/termux-callback
  DESTINATION ${CMAKE_INSTALL_PREFIX}/libexec
  PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE
    GROUP_READ GROUP_EXECUTE
    WORLD_READ WORLD_EXECUTE
)
