cmake_minimum_required(VERSION 3.2)
project(cpp-statsd-client VERSION 0.0.0)

# Introduce variables:
# * CMAKE_INSTALL_LIBDIR
# * CMAKE_INSTALL_BINDIR
# * CMAKE_INSTALL_INCLUDEDIR
include(GNUInstallDirs)

option(BUILD_TESTS "Build tests" OFF)
option(HUNTER_ENABLED "Use the Hunter package manager" OFF)

set(CPP-STATSD-CLIENT_HDR_LIST include/cpp-statsd-client/StatsdClient.hpp include/cpp-statsd-client/UDPSender.hpp)
add_library(cpp-statsd-client INTERFACE)

target_include_directories(cpp-statsd-client INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
)

find_package(Threads REQUIRED)
target_link_libraries(cpp-statsd-client INTERFACE Threads::Threads)
if(WIN32)
  target_link_libraries(cpp-statsd-client INTERFACE ws2_32)
endif()

if(NOT HUNTER_ENABLED)
  set(CMAKE_CXX_STANDARD 17)
endif()

if(BUILD_TESTS)
  add_executable(testStatsdClient ${CMAKE_CURRENT_SOURCE_DIR}/tests/testStatsdClient.cpp)
  target_link_libraries(testStatsdClient cpp-statsd-client)
endif()

set(GENERATED_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(VERSION_CONFIG "${GENERATED_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
set(PROJECT_CONFIG "${GENERATED_DIR}/${PROJECT_NAME}Config.cmake")

set(CONFIG_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")

set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(NAMESPACE "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${VERSION_CONFIG}" COMPATIBILITY ExactVersion
)

configure_package_config_file(
    "cmake/Config.cmake.in"
    "${PROJECT_CONFIG}"
    INSTALL_DESTINATION "${CONFIG_INSTALL_DIR}"
)

install(
    TARGETS cpp-statsd-client
    EXPORT "${TARGETS_EXPORT_NAME}"
    INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
)

install(
    FILES ${CPP-STATSD-CLIENT_HDR_LIST}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
)

install(
    FILES "${PROJECT_CONFIG}" "${VERSION_CONFIG}"
    DESTINATION "${CONFIG_INSTALL_DIR}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${NAMESPACE}"
    DESTINATION "${CONFIG_INSTALL_DIR}"
)
