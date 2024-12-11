cmake_minimum_required(VERSION 3.20)
project(libsofa)


set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED YES)

# Enable optimization and Link Time Optimization (LTO) for Release builds
set(CMAKE_BUILD_TYPE Release)  # Set default to Release, so optimizations are enabled
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3 -flto")

# Check the system we're using and set the output directory based on architecture
if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
    # For 32-bit and 64-bit Windows
    if("${CMAKE_SIZEOF_VOID_P}" EQUAL "8")
        message(STATUS "Target is 64 bits on Windows")
        set(SOFA_EXT_LIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/build/lib/win/x64" CACHE FILEPATH "description")
        set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/build/lib/win/x64")
    else()
        message(STATUS "Target is 32 bits on Windows")
        set(SOFA_EXT_LIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/build/lib/win/i386" CACHE FILEPATH "description")
        set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/build/lib/win/i386")
    endif()

    # Find libraries for Windows
    find_library(SZ_LIB szip HINTS ${SOFA_EXT_LIB_PATH})
    find_library(M_LIB m ${SOFA_EXT_LIB_PATH})
    find_library(DL_LIB dl ${SOFA_EXT_LIB_PATH})

elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    # Linux specific settings
    set(SOFA_EXT_LIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/build/lib/linux" CACHE FILEPATH "description")
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/build/lib/linux")
    find_library(M_LIB m)
    find_library(DL_LIB dl)

elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Darwin")
    # macOS specific settings
    set(SOFA_EXT_LIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/build/lib/macos" CACHE FILEPATH "description")
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/build/lib/macos")

    # Add ARM64 check for macOS
    if(APPLE AND CMAKE_OSX_ARCHITECTURES STREQUAL "arm64")
        message(STATUS "Target is ARM64 on macOS")
        set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/build/lib/macos/arm64")
    endif()

    find_library(M_LIB m)
    find_library(DL_LIB dl)

else()
    message(SEND_ERROR "Unknown system: ${CMAKE_SYSTEM_NAME}. Only Windows, Linux, and Darwin (macOS) are supported.")
endif()

# Shipped libraries
find_library(HDF5_LIB hdf5 HINTS ${SOFA_EXT_LIB_PATH})
find_library(HDF5_HL_LIB hdf5_hl HINTS ${SOFA_EXT_LIB_PATH})
find_library(NETCDF_LIB netcdf HINTS ${SOFA_EXT_LIB_PATH})
find_library(NETCDF_CXX_LIB netcdf_c++4 HINTS ${SOFA_EXT_LIB_PATH})
find_library(CURL_LIB curl HINTS ${SOFA_EXT_LIB_PATH})
find_library(Z_LIB z HINTS ${SOFA_EXT_LIB_PATH})

set(SOFA_EXT_INCLUDE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/inc" CACHE FILEPATH "description")
include_directories(${SOFA_EXT_INCLUDE_PATH})

# Collect all source and header files
file(GLOB SOFA_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
file(GLOB SOFA_HEADERS "${CMAKE_CURRENT_SOURCE_DIR}/inc/*.h")

# These aren't strictly part of the library
list(REMOVE_ITEM SOFA_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/src/json2sofa.cpp")
list(REMOVE_ITEM SOFA_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/src/sofa2json.cpp")

# Add the library using the gathered files
add_library(sofa STATIC ${SOFA_SOURCES} ${SOFA_HEADERS})

# Generate example programs
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/build/test")

add_executable(sofainfo "${CMAKE_CURRENT_SOURCE_DIR}/test/sofainfo.cpp")
target_link_libraries(sofainfo sofa ${NETCDF_CXX_LIB} ${NETCDF_LIB} ${HDF5_HL_LIB} ${HDF5_LIB} ${SZ_LIB} ${Z_LIB} ${CURL_LIB} ${M_LIB} ${DL_LIB})

add_executable(sofamisc "${CMAKE_CURRENT_SOURCE_DIR}/test/sofamisc.cpp")
target_link_libraries(sofamisc sofa ${NETCDF_CXX_LIB} ${NETCDF_LIB} ${HDF5_HL_LIB} ${HDF5_LIB} ${SZ_LIB} ${Z_LIB} ${CURL_LIB} ${M_LIB} ${DL_LIB})

add_executable(sofaexamples "${CMAKE_CURRENT_SOURCE_DIR}/test/sofaexamples.cpp")
target_link_libraries(sofaexamples sofa ${NETCDF_CXX_LIB} ${NETCDF_LIB} ${HDF5_HL_LIB} ${HDF5_LIB} ${SZ_LIB} ${Z_LIB} ${CURL_LIB} ${M_LIB} ${DL_LIB})