cmake_minimum_required(VERSION 3.10)

project(loguru LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(CheckLibraryExists)

# if fmt is not already in the environment, find it
if(NOT TARGET fmt::fmt)
    find_package(fmt REQUIRED)
endif()

check_library_exists(execinfo backtrace "" LINK_TO_EXECINFO)

add_library(loguru STATIC loguru.cpp)
add_library(loguru::loguru ALIAS loguru)

target_link_libraries(loguru PUBLIC dl fmt::fmt)
target_compile_definitions(loguru PUBLIC LOGURU_USE_FMTLIB)
target_include_directories(loguru PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/>"
    "$<INSTALL_INTERFACE:include/loguru>"
)

if(LINK_TO_EXECINFO)
    target_link_libraries(loguru PUBLIC execinfo)
endif()

# files in public_header property get installed
set_target_properties(loguru PROPERTIES PUBLIC_HEADER "loguru.hpp")

#
# Installation section
#

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/config.cmake.in
    "${CMAKE_CURRENT_BINARY_DIR}/loguru-config.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/loguru)

install(TARGETS loguru
    EXPORT loguru-targets
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/loguru )

export(EXPORT loguru-targets NAMESPACE loguru:: FILE loguru-targets.cmake)

install(EXPORT loguru-targets
    FILE loguru-targets.cmake
    NAMESPACE loguru::
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/loguru")

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/loguru-config.cmake
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/loguru")
