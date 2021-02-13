cmake_minimum_required(VERSION 2.8.12)

find_program(CCACHE_PROGRAM ccache)
if(CCACHE_PROGRAM)
    message(STATUS "LMDB: Found ccache package... Activating...")
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CCACHE_PROGRAM}")
endif()

set(LIB_MAJOR_VERSION "0")
set(LIB_MINOR_VERSION "9")
set(LIB_PATCH_VERSION "28")
set(LIB_VERSION_STRING "${LIB_MAJOR_VERSION}.${LIB_MINOR_VERSION}.${LIB_PATCH_VERSION}")

if(CMAKE_VERSION VERSION_LESS 3.0)
    project(lmdb C)
    enable_language(C)
else()
    cmake_policy(SET CMP0048 NEW)
    project(lmdb VERSION "${LIB_VERSION_STRING}" LANGUAGES C)
endif()

## This section describes our general CMake setup options
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_SKIP_INSTALL_RULES OFF FORCE)
set(CMAKE_SKIP_PACKAGE_ALL_DEPENDENCY ON FORCE)
set(CMAKE_SUPPRESS_REGENERATION ON)
set(CMAKE_POLICY_WARNING_CMP0048 OFF)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

## We only build static binaries -- this is left here for our dependencies
set(STATIC ON CACHE BOOL FORCE "Link libraries statically? Forced to ON")
add_definitions(-DSTATICLIB)

if(MSVC)
    add_definitions("/EHsc /DWIN32_LEAN_AND_MEAN /O2 /wd4267 /wd4996 /wd4244 /wd4333 /wd4146 /wd4172")
endif()

if(NOT MSVC)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c11 -O3 -Wno-return-local-addr")
endif()

set(LMDB_SRC
    mdb.c
    midl.c
)

add_library(lmdb STATIC ${LMDB_SRC})
target_include_directories(lmdb PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

if(WIN32)
    target_link_libraries(lmdb ws2_32)
endif()

set_property(TARGET lmdb PROPERTY C_STANDARD 90)
set_property(TARGET lmdb PROPERTY SOVERSION 1)

foreach(flag_var
        CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
        CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO
        CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
        CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
    string(REGEX REPLACE "/RTC[^ ]*" "" ${flag_var} "${${flag_var}}")
    string(REGEX REPLACE "/Od" "" ${flag_var} "${${flag_var}}")
endforeach(flag_var)
