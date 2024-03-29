# Copyright 2020-2021. Alexandro Sanchez Bach.

cmake_minimum_required(VERSION 3.12)

# Version
set(ORBITAL_VERSION_MAJOR 0)
set(ORBITAL_VERSION_MINOR 1)
set(ORBITAL_VERSION_PATCH 0)

# Directories
set(ORBITAL_DIR_ROOT       ${CMAKE_CURRENT_LIST_DIR})
set(ORBITAL_DIR_CMAKE      "${ORBITAL_DIR_ROOT}/cmake")
set(ORBITAL_DIR_EXTERNALS  "${ORBITAL_DIR_ROOT}/externals")
set(ORBITAL_DIR_SOURCES    "${ORBITAL_DIR_ROOT}/src")

# Project
project(orbital CXX)

# Dependencies
find_package(SDL2 REQUIRED)
find_package(imgui REQUIRED)
find_package(fmt CONFIG REQUIRED)
find_package(RapidJSON CONFIG REQUIRED)
find_package(Vulkan REQUIRED)
find_package(ZLIB REQUIRED)
find_library(BOTAN_LIBRARIES NAMES BOTAN2 botan2 BOTAN botan)
find_library(CAPSTONE_LIBRARIES NAMES capstone_dll capstone)

# Sources
macro(ORBITAL_FILES_APPEND)
    file(GLOB FILES_APPEND CONFIGURE_DEPENDS ${ARGV})
    list(APPEND ORBITAL_SOURCES ${FILES_APPEND})
endmacro()
macro(ORBITAL_SOURCES_APPEND)
    ORBITAL_FILES_APPEND(${ARGV0}/*.h)
    ORBITAL_FILES_APPEND(${ARGV0}/*.cpp)
endmacro()

ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/analysis)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/analysis/orbis)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/host)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/host/graphics)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/hardware)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/hardware/aeolia)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/hardware/aeolia/hpet)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/hardware/aeolia/msic)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/hardware/aeolia/uart)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/hardware/liverpool)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/hardware/liverpool/gca)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/hardware/liverpool/gmc)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/hardware/liverpool/oss)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/hardware/liverpool/smu)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/hardware/liverpool/sam)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/software)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/software/sbl)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/ui)
ORBITAL_SOURCES_APPEND(${ORBITAL_DIR_SOURCES}/orbital/ui/imgui)

# Target
add_executable(orbital ${ORBITAL_SOURCES})
target_include_directories(orbital PUBLIC ${ORBITAL_DIR_EXTERNALS})
target_include_directories(orbital PUBLIC ${ORBITAL_DIR_SOURCES})
target_include_directories(orbital PUBLIC ${RAPIDJSON_INCLUDE_DIRS})
target_include_directories(orbital PUBLIC ${Vulkan_INCLUDE_DIRS})
target_link_libraries(orbital PRIVATE fmt::fmt SDL2::SDL2 imgui::imgui ZLIB::ZLIB
    ${Vulkan_LIBRARIES} ${BOTAN_LIBRARIES} ${CAPSTONE_LIBRARIES})
if (EXISTS ${ORBITAL_DIR_EXTERNALS}/core.cmake)
    include(${ORBITAL_DIR_EXTERNALS}/core.cmake)
else()
    message(FATAL_ERROR "
        Orbital depends on an unreleased third-party library and cannot be built without the required `core.cmake` script.
        Functionality related to PS4 emulation/introspection is open-sourced *only* as documentation for fellow developers and hackers.
        To build Orbital, wait for the upcoming release of `core.cmake` or reimplement one yourself that includes/links QEMU instead.
        Please do NOT ask for help/support related to `core.cmake` issues.")
endif()

# Properties
set_target_properties(orbital PROPERTIES CXX_STANDARD 20)
set_target_properties(orbital PROPERTIES CXX_STANDARD_REQUIRED ON)
set_target_properties(orbital PROPERTIES POSITION_INDEPENDENT_CODE 1)
