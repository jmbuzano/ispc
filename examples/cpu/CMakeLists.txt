#
#  Copyright (c) 2018-2023, Intel Corporation
#
#  SPDX-License-Identifier: BSD-3-Clause

#
# ispc examples CMakeLists.txt
#
cmake_minimum_required(VERSION 3.12)

set (PROJECT_NAME ispc_examples)
project(${PROJECT_NAME} C CXX)

# Cross-compilation options
option(ISPC_CROSS "Build ISPC with cross compilation support" OFF)
option(ISPC_WINDOWS_TARGET "Build ISPC with Windows target support" ON)
option(ISPC_LINUX_TARGET "Build ISPC with Linux target support" ON)
option(ISPC_MACOS_TARGET "Build ISPC with macOS target support" OFF)
option(ISPC_IOS_TARGET "Build ISPC with iOS target support" ON)
option(ISPC_ANDROID_TARGET "Build ISPC with Android target support" ON)
option(ISPC_PS_TARGET "Build ISPC with PS4 and PS5 targets support" ON)

if(ISPC_CROSS)
    if (WIN32)
        set(ISPC_IOS_TARGET OFF)
        if ((ISPC_LINUX_TARGET OR ISPC_ANDROID_TARGET OR ISPC_PS_TARGET) AND NOT ISPC_GNUWIN32_PATH)
            message(FATAL_ERROR "Set ISPC_GNUWIN32_PATH variable for cross compilation to Linux/Android/PS4 e.g. C:/gnuwin32")
        endif()
        if (ISPC_MACOS_TARGET AND NOT ISPC_MACOS_SDK_PATH)
            message(FATAL_ERROR "Set ISPC_MACOS_SDK_PATH variable for cross compilation to MacOS e.g. C:/iusers/MacOSX10.14.sdk")
        endif()
    elseif (APPLE)
        set(ISPC_MACOS_TARGET ON)
        set(ISPC_WINDOWS_TARGET OFF)
        set(ISPC_PS_TARGET OFF)
        set(ISPC_FREEBSD_TARGET OFF)
        if ((ISPC_LINUX_TARGET OR ISPC_ANDROID_TARGET) AND NOT ISPC_ANDROID_NDK_PATH)
            message(FATAL_ERROR "Set ISPC_ANDROID_NDK_PATH variable for cross compilation to Linux/Android e.g. /Users/Shared/android-ndk-r20")
        endif()
        if (ISPC_IOS_TARGET AND NOT ISPC_IOS_SDK_PATH)
            set(command "xcrun" "--show-sdk-path" "--sdk" "iphoneos")
            execute_process(COMMAND ${command}
                OUTPUT_VARIABLE ISPC_IOS_SDK_PATH
                OUTPUT_STRIP_TRAILING_WHITESPACE
            )
            message(STATUS "Using iOS SDK path ${ISPC_IOS_SDK_PATH}")
        endif()
    else()
        set(ISPC_WINDOWS_TARGET OFF)
        set(ISPC_PS_TARGET OFF)
        set(ISPC_IOS_TARGET OFF)
        if (ISPC_MACOS_TARGET AND NOT ISPC_MACOS_SDK_PATH)
            message(FATAL_ERROR "Set ISPC_MACOS_SDK_PATH variable for cross compilation to MacOS e.g. /iusers/MacOSX10.14.sdk")
        endif()
    endif()
else()
    if (WIN32)
        set(ISPC_WINDOWS_TARGET ON)
    elseif (APPLE)
        set(ISPC_MACOS_TARGET ON)
    elseif (CMAKE_SYSTEM_NAME STREQUAL "FreeBSD")
        set(ISPC_FREEBSD_TARGET ON)
    else()
        set(ISPC_LINUX_TARGET ON)
    endif()
endif()

if(CMAKE_BUILD_TYPE)
    # Validate build type
    set(CONFIGURATION_TYPES "Debug;Release;RelWithDebInfo")

    string(FIND "${CONFIGURATION_TYPES}" "${CMAKE_BUILD_TYPE}" MATCHED_CONFIG)
    if (${MATCHED_CONFIG} EQUAL -1)
        message(FATAL_ERROR "CMAKE_BUILD_TYPE (${CMAKE_BUILD_TYPE}) allows only the following values: ${CONFIGURATION_TYPES}")
    endif()
else(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release")
    message(STATUS "Build type not specified: Use Release by default.")
endif(CMAKE_BUILD_TYPE)

include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/AddISPCExample.cmake)

if(WIN32)
    add_compile_definitions(WIN32)
    if(MSVC)
        add_compile_options($<$<COMPILE_LANGUAGE:C,CXX>:/EHsc>)
    endif()
endif()

if (NOT DEFINED ISPC_EXECUTABLE)
    find_program (ISPC_EXECUTABLE ispc)
    if (NOT ISPC_EXECUTABLE)
        message(FATAL_ERROR "Failed to find ispc" )
    endif()
endif()

set (EXAMPLES_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/..)

add_subdirectory(aobench)
add_subdirectory(aobench_instrumented)
add_subdirectory(deferred)
add_subdirectory(gmres)
add_subdirectory(mandelbrot)
add_subdirectory(mandelbrot_tasks)
add_subdirectory(noise)
add_subdirectory(options)
add_subdirectory(perfbench)
add_subdirectory(rt)
add_subdirectory(sgemm)
add_subdirectory(simple)
add_subdirectory(sort)
add_subdirectory(stencil)
add_subdirectory(volume_rendering)