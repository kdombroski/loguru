cmake_minimum_required(VERSION 3.10)

project(loguru LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(LOGURU_USE_FMTLIB "Enable formatting with fmtlib" ON)
option(LOGURU_INSTALL "Enable install targets" ON)
option(LOGURU_TESTING "Enable tests" ON)
option(LOGURU_PIC "Build position independent code" OFF)

include(CheckLibraryExists)

find_package(Threads REQUIRED)

add_library(loguru STATIC loguru.cpp)
add_library(loguru::loguru ALIAS loguru)

target_link_libraries(loguru PUBLIC Threads::Threads)
target_include_directories(loguru PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/>"
    "$<INSTALL_INTERFACE:include/loguru>"
)

if(LOGURU_USE_FMTLIB)
    # if fmt is not already in the environment, find it
    if(NOT TARGET fmt::fmt)
        find_package(fmt REQUIRED)
    endif()

    target_link_libraries(loguru PUBLIC fmt::fmt)
    target_compile_definitions(loguru PUBLIC LOGURU_USE_FMTLIB)
endif()

# backtrace is unstable on alpine (musl), so forcibly disable it unless we know it works
# this isn't 100% correct (it only enables on glibc) but it will serve our purposes for now.
# NB that musl does not provide a corresponding identification #define .
include(CheckSymbolExists)
check_symbol_exists(__GLIBC__ "features.h" LIBC_IS_GLIBC)

target_compile_definitions(loguru PRIVATE LOGURU_STACKTRACES=0)

if(NOT LIBC_IS_GLIBC)
    # fix errno_as_text() on musl (use XSI version of strerror_r)
    target_compile_definitions(loguru PRIVATE _POSIX_C_SOURCE=200112L)
endif()

# files in public_header property get installed
set_target_properties(loguru PROPERTIES PUBLIC_HEADER "loguru.hpp")

# position independent code is important when building shared libraries
if(LOGURU_PIC)
    set_target_properties(loguru PROPERTIES POSITION_INDEPENDENT_CODE ${LOGURU_PIC})
endif()

if(ENABLE_TESTS)
    add_executable(tests test/loguru_test.cpp)
    target_link_libraries(tests PUBLIC loguru)

    include(CTest)
    add_test(NAME basic COMMAND tests)
    add_test(NAME ABORT_F COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> ABORT_F)
    add_test(NAME ABORT_S COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> ABORT_S)
    add_test(NAME assert COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> assert)
    add_test(NAME LOG_F_FATAL COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> LOG_F_FATAL)
    add_test(NAME LOG_S_FATAL COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> LOG_S_FATAL)
    add_test(NAME CHECK_NOTNULL_F COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> CHECK_NOTNULL_F)
    add_test(NAME CHECK_F COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> CHECK_F)
    add_test(NAME CHECK_EQ_F COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> CHECK_EQ_F)
    add_test(NAME CHECK_EQ_F_int COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> CHECK_EQ_F_int)
    add_test(NAME CHECK_EQ_F_unsigned COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> CHECK_EQ_F_unsigned)
    add_test(NAME CHECK_EQ_F_size_t COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> CHECK_EQ_F_size_t)
    add_test(NAME CHECK_EQ_F_message COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> CHECK_EQ_F_message)
    add_test(NAME CHECK_EQ_S COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> CHECK_EQ_S)
    add_test(NAME CHECK_LT_S COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> CHECK_LT_S)
    add_test(NAME CHECK_LT_S_message COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> CHECK_LT_S_message)
    add_test(NAME deep_abort COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> deep_abort)
    add_test(NAME SIGSEGV COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> SIGSEGV)
    add_test(NAME abort COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> abort)
    add_test(NAME error_context COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> error_context)
    add_test(NAME throw_on_fatal COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> throw_on_fatal)
    add_test(NAME throw_on_signal COMMAND ${CMAKE_COMMAND} -E env $<TARGET_FILE:tests> throw_on_signal)
    add_test(NAME callback COMMAND tests callback)
    # add_test(NAME hang COMMAND tests hang)

    set_property(TEST ABORT_F PROPERTY WILL_FAIL TRUE)
    set_property(TEST ABORT_S PROPERTY WILL_FAIL TRUE)
    set_property(TEST assert PROPERTY WILL_FAIL TRUE)
    set_property(TEST LOG_F_FATAL PROPERTY WILL_FAIL TRUE)
    set_property(TEST LOG_S_FATAL PROPERTY WILL_FAIL TRUE)
    set_property(TEST CHECK_NOTNULL_F PROPERTY WILL_FAIL TRUE)
    set_property(TEST CHECK_F PROPERTY WILL_FAIL TRUE)
    set_property(TEST CHECK_EQ_F PROPERTY WILL_FAIL TRUE)
    set_property(TEST CHECK_EQ_F_int PROPERTY WILL_FAIL TRUE)
    set_property(TEST CHECK_EQ_F_unsigned PROPERTY WILL_FAIL TRUE)
    set_property(TEST CHECK_EQ_F_size_t PROPERTY WILL_FAIL TRUE)
    set_property(TEST CHECK_EQ_F_message PROPERTY WILL_FAIL TRUE)
    set_property(TEST CHECK_EQ_S PROPERTY WILL_FAIL TRUE)
    set_property(TEST CHECK_LT_S PROPERTY WILL_FAIL TRUE)
    set_property(TEST CHECK_LT_S_message PROPERTY WILL_FAIL TRUE)
    set_property(TEST deep_abort PROPERTY WILL_FAIL TRUE)
    set_property(TEST SIGSEGV PROPERTY WILL_FAIL TRUE)
    set_property(TEST abort PROPERTY WILL_FAIL TRUE)
    set_property(TEST error_context PROPERTY WILL_FAIL TRUE)
    set_property(TEST throw_on_fatal PROPERTY WILL_FAIL TRUE)
    set_property(TEST throw_on_signal PROPERTY WILL_FAIL TRUE)
endif()

#
# Installation section
#

if (LOGURU_INSTALL)
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
endif()
