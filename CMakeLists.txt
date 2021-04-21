cmake_minimum_required(VERSION 3.10)

project(loguru LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(fmt REQUIRED)

add_library(loguru STATIC loguru.cpp)

target_link_libraries(loguru PUBLIC dl fmt::fmt)
target_compile_definitions(loguru PUBLIC LOGURU_USE_FMTLIB)
target_include_directories(loguru PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})