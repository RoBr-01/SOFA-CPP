cmake_minimum_required(VERSION 3.20)
project(libsofa VERSION 1.1.0)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED YES)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
include(FetchContent)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/build/test")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/build/lib")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/build/lib")

# Find NetCDF library and include directory manually
find_library(NETCDF_LIB NAMES netcdf PATHS "/usr/local/lib" REQUIRED)
find_path(NETCDF_INCLUDE_DIR NAMES netcdf.h PATHS "/usr/local/include" REQUIRED)

# Find the NetCDF C++ library (libnetcdf-cxx4)
find_library(NETCDF_CXX_LIB NAMES netcdf-cxx4 PATHS "/opt/homebrew/Cellar/netcdf-cxx/4.3.1_3/lib" REQUIRED)
find_path(NETCDF_CXX_INCLUDE_DIR NAMES ncAtt.h PATHS "/opt/homebrew/Cellar/netcdf-cxx/4.3.1_3/include" REQUIRED)

if (NETCDF_LIB AND NETCDF_INCLUDE_DIR AND NETCDF_CXX_LIB AND NETCDF_CXX_INCLUDE_DIR)
    message(STATUS "NetCDF found: ${NETCDF_LIB}")
    message(STATUS "NetCDF C++ found: ${NETCDF_CXX_LIB}")
    include_directories(${NETCDF_INCLUDE_DIR} ${NETCDF_CXX_INCLUDE_DIR})
else()
    message(FATAL_ERROR "NetCDF or NetCDF C++ not found!")
endif()

# Find json-c library and include directory manually
find_library(JSON_C_LIB NAMES json-c PATHS "/opt/homebrew/Cellar/json-c/0.18/lib" REQUIRED)
find_path(JSON_C_INCLUDE_DIR NAMES json.h PATHS "/opt/homebrew/Cellar/json-c/0.18/include/json-c" REQUIRED)

if (JSON_C_LIB AND JSON_C_INCLUDE_DIR)
    message(STATUS "json-c found: ${JSON_C_LIB}")
    include_directories(${JSON_C_INCLUDE_DIR})
else()
    message(FATAL_ERROR "json-c not found!")
endif()

# SOFA Headers
include_directories(${CMAKE_SOURCE_DIR}/inc)

# SOFA Source Files
file(GLOB_RECURSE SOFA_SOURCES "${CMAKE_SOURCE_DIR}/src/*.cpp")

# Define static library
add_library(sofa STATIC ${SOFA_SOURCES} ${SOFA_HEADERS})

# Link NetCDF and NetCDF C++ manually
target_link_libraries(sofa PUBLIC ${NETCDF_LIB} ${NETCDF_CXX_LIB} ${JSON_C_LIB})

# Generate Test Executables
add_executable(sofainfo "${CMAKE_CURRENT_SOURCE_DIR}/test/sofainfo.cpp")
target_link_libraries(sofainfo PRIVATE sofa ${NETCDF_LIB} ${NETCDF_CXX_LIB} ${JSON_C_LIB})

add_executable(sofamisc "${CMAKE_CURRENT_SOURCE_DIR}/test/sofamisc.cpp")
target_link_libraries(sofamisc PRIVATE sofa ${NETCDF_LIB} ${NETCDF_CXX_LIB} ${JSON_C_LIB})

add_executable(sofaexamples "${CMAKE_CURRENT_SOURCE_DIR}/test/sofaexamples.cpp")
target_link_libraries(sofaexamples PRIVATE sofa ${NETCDF_LIB} ${NETCDF_CXX_LIB} ${JSON_C_LIB})