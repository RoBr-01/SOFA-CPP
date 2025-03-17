cmake_minimum_required(VERSION  3.20)
cmake_policy(SET CMP0042 NEW) 
set(CMAKE_MACOSX_RPATH ON)
project(libsofa)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED YES)

set(CMAKE_OSX_ARCHITECTURES "x86_64")

include(FetchContent)

# Fetch and integrate zlib
FetchContent_Declare(
    zlib
    URL https://zlib.net/fossils/zlib-1.2.8.tar.gz
)

FetchContent_Declare(
    curl
    URL https://curl.se/download/curl-7.37.1.zip
)

FetchContent_Declare(
    hdf5
    URL https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.8/hdf5-1.8.13/src/hdf5-1.8.13.tar.gz
)

# Fetch and integrate NetCDF-C
FetchContent_Declare(
    netcdf
    URL https://github.com/Unidata/netcdf-c/archive/refs/tags/v4.3.2.zip
)

# Fetch and integrate NetCDF-CXX4
FetchContent_Declare(
    netcdf_cxx4
    URL https://github.com/Unidata/netcdf-cxx4/archive/refs/tags/v4.2.1.zip
)

FetchContent_MakeAvailable(zlib curl hdf5 netcdf netcdf_cxx4)


# Change this to use the cmakepresets.json instead, much more flexible
# Provide a profile for building the tests separately
set(CMAKE_BUILD_TYPE Release)
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3 -flto")

# Set architecture-specific output directories
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    if(CMAKE_SIZEOF_VOID_P EQUAL 8)
        message(STATUS "Target is 64-bit on Windows")
        set(SOFA_EXT_LIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/build/lib/win/x64")
    else()
        message(STATUS "Target is 32-bit on Windows")
        set(SOFA_EXT_LIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/build/lib/win/i386")
    endif()
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    message(STATUS "Target is Linux")
    set(SOFA_EXT_LIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/build/lib/linux")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    message(STATUS "Target is macOS")
    set(SOFA_EXT_LIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/build/lib/macos")
    if(APPLE AND CMAKE_OSX_ARCHITECTURES STREQUAL "arm64")
        message(STATUS "Target is ARM64 on macOS")
        set(SOFA_EXT_LIB_PATH "${CMAKE_EXT_LIB_PATH}/arm64")
    endif()
else()
    message(FATAL_ERROR "Unknown system: ${CMAKE_SYSTEM_NAME}. Only Windows, Linux, and macOS (Darwin) are supported.")
endif()

set(HDF5_INCLUDE_DIR "/opt/homebrew/include")
set(HDF5_LIB "/opt/homebrew/lib/libhdf5.dylib")
set(HDF5_HL_LIB "/opt/homebrew/lib/libhdf5_hl.dylib")

set(HDF5_LIBRARIES ${HDF5_LIB} ${HDF5_HL_LIB})
set(HDF5_INCLUDE_DIRS ${HDF5_INCLUDE_DIR})
set(HDF5_C_LIBRARY ${HDF5_LIB})
set(HDF5_HL_LIBRARY ${HDF5_HL_LIB})

set(HDF5_ROOT "/opt/homebrew")
find_package(HDF5 REQUIRED COMPONENTS C HL)

# Now manually call find_package again with hints
# find_package(HDF5 COMPONENTS C HL REQUIRED)

# Set output directories
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${SOFA_EXT_LIB_PATH}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/build/test")

# Include directories
set(SOFA_EXT_INCLUDE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/inc")
include_directories(${SOFA_EXT_INCLUDE_PATH})

# Gather source and header files
file(GLOB SOFA_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
file(GLOB SOFA_HEADERS "${CMAKE_CURRENT_SOURCE_DIR}/inc/*.h")

# Exclude specific source files from the static library
list(REMOVE_ITEM SOFA_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/src/json2sofa.cpp" "${CMAKE_CURRENT_SOURCE_DIR}/src/sofa2json.cpp")

# Create the static library
add_library(sofa STATIC ${SOFA_SOURCES} ${SOFA_HEADERS})
target_link_libraries(sofa PRIVATE zlib hdf5::hdf5 hdf5::hdf5_hl)

# # Example programs
# add_executable(sofainfo "${CMAKE_CURRENT_SOURCE_DIR}/test/sofainfo.cpp")
# target_link_libraries(sofainfo PRIVATE sofa ${NETCDF_CXX_LIB} ${NETCDF_LIB} ${HDF5_HL_LIB} ${HDF5_LIB} ${SZ_LIB} ${CURL_LIB} ${M_LIB} ${DL_LIB})

# add_executable(sofamisc "${CMAKE_CURRENT_SOURCE_DIR}/test/sofamisc.cpp")
# target_link_libraries(sofamisc PRIVATE sofa ${NETCDF_CXX_LIB} ${NETCDF_LIB} ${HDF5_HL_LIB} ${HDF5_LIB} ${SZ_LIB} ${CURL_LIB} ${M_LIB} ${DL_LIB})

# add_executable(sofaexamples "${CMAKE_CURRENT_SOURCE_DIR}/test/sofaexamples.cpp")
# target_link_libraries(sofaexamples PRIVATE sofa ${NETCDF_CXX_LIB} ${NETCDF_LIB} ${HDF5_HL_LIB} ${HDF5_LIB} ${SZ_LIB} ${CURL_LIB} ${M_LIB} ${DL_LIB})