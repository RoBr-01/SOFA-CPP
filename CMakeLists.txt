cmake_minimum_required(VERSION 3.20)
cmake_policy(SET CMP0042 NEW)  # Enable RPATH for macOS
set(CMAKE_MACOSX_RPATH ON)

# add_makedist()

project(libsofa LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED YES)

set(HDF5_ROOT "/opt/homebrew/Cellar/hdf5/1.14.6")

# Ensure compatibility on macOS
if(APPLE)
    set(CMAKE_OSX_ARCHITECTURES "x86_64")  # Force x86_64 on macOS
endif()

include(FetchContent)

# Fetch ZLIB
FetchContent_Declare(
    zlib
    URL https://zlib.net/fossils/zlib-1.2.8.tar.gz
)
FetchContent_MakeAvailable(zlib)

# Fetch CURL
FetchContent_Declare(
    curl
    URL https://curl.se/download/curl-7.37.1.zip
)
FetchContent_MakeAvailable(curl)

# # Fetch HDF5
# FetchContent_Declare(
#     hdf5
#     URL https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.8/hdf5-1.8.13/src/hdf5-1.8.13.tar.gz
# )
# FetchContent_MakeAvailable(hdf5)

# Ensure HDF5 is correctly detected
set(HDF5_ROOT "${hdf5_SOURCE_DIR}")
set(HDF5_INCLUDE_DIR "${HDF5_ROOT}/include")
set(HDF5_DIR "${hdf5_BINARY_DIR}")
set(HDF5_NO_FIND_PACKAGE_CONFIG TRUE)

# Try finding HDF5 again
find_package(HDF5 REQUIRED COMPONENTS C HL)

if(NOT HDF5_FOUND)
    message(FATAL_ERROR "HDF5 was not found! Please check if FetchContent correctly downloaded it.")
endif()

# Fetch NetCDF
FetchContent_Declare(
    netcdf
    URL https://github.com/Unidata/netcdf-c/archive/refs/tags/v4.3.2.zip
)

# FetchContent_Declare(
#     netcdf
#     URL https://github.com/Unidata/netcdf-c/releases/download/v4.7.4/netcdf-c-4.7.4.tar.gz
# )
FetchContent_MakeAvailable(netcdf)

# Fetch NetCDF-CXX4
FetchContent_Declare(
    netcdf_cxx4
    URL https://github.com/Unidata/netcdf-cxx4/archive/refs/tags/v4.2.1.zip
)
FetchContent_MakeAvailable(netcdf_cxx4)


# Build configuration options
set(CMAKE_BUILD_TYPE Release)
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3 -flto")

# Output directories based on OS
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    set(SOFA_EXT_LIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/build/lib/win/x64")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    set(SOFA_EXT_LIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/build/lib/linux")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    set(SOFA_EXT_LIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/build/lib/macos")
    if(APPLE AND CMAKE_OSX_ARCHITECTURES STREQUAL "arm64")
        set(SOFA_EXT_LIB_PATH "${CMAKE_EXT_LIB_PATH}/arm64")
    endif()
else()
    message(FATAL_ERROR "Unknown system: ${CMAKE_SYSTEM_NAME}. Only Windows, Linux, and macOS (Darwin) are supported.")
endif()

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${SOFA_EXT_LIB_PATH}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/build/test")

# Include directories for Sofa
set(SOFA_EXT_INCLUDE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/inc")
include_directories(${SOFA_EXT_INCLUDE_PATH})

# Gather source and header files
file(GLOB SOFA_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
file(GLOB SOFA_HEADERS "${CMAKE_CURRENT_SOURCE_DIR}/inc/*.h")

# Exclude certain source files
list(REMOVE_ITEM SOFA_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/src/json2sofa.cpp" "${CMAKE_CURRENT_SOURCE_DIR}/src/sofa2json.cpp")

# Create static library for Sofa
add_library(sofa STATIC ${SOFA_SOURCES} ${SOFA_HEADERS})
target_link_libraries(sofa PRIVATE zlib ${HDF5_LIBRARIES} netcdf_cxx4 netcdf curl)

# Optional executables
# add_executable(sofainfo "${CMAKE_CURRENT_SOURCE_DIR}/test/sofainfo.cpp")
# target_link_libraries(sofainfo PRIVATE sofa ${HDF5_LIBRARIES})

# add_executable(sofamisc "${CMAKE_CURRENT_SOURCE_DIR}/test/sofamisc.cpp")
# target_link_libraries(sofamisc PRIVATE sofa ${HDF5_LIBRARIES})

# add_executable(sofaexamples "${CMAKE_CURRENT_SOURCE_DIR}/test/sofaexamples.cpp")
# target_link_libraries(sofaexamples PRIVATE sofa ${HDF5_LIBRARIES})