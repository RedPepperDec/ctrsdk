cmake_minimum_required(VERSION 3.24)

set(CMAKE_TOOLCHAIN_FILE ${CMAKE_SOURCE_DIR}/project/ARMCC.cmake)
include(${CMAKE_SOURCE_DIR}/project/Toolchain.cmake)
set_toolchain()

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

project(ctrsdk C CXX)

if (WIN32)
    set(LOCALE_FLAG --locale=japanese)
else()
    set(LOCALE_FLAG --locale=ja_JP.SJIS)
endif()

__compiler_armcc(C)
__compiler_armcc(CXX)

file(GLOB_RECURSE SOURCES sources/*.c sources/*.cpp)

add_library(ctrsdk ${SOURCES})
add_library(crt0 OBJECT sources/crt0/MPCore/crt0.cpp)

target_include_directories(crt0 PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_include_directories(ctrsdk PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)

function(set_armcc_options target)
    target_compile_options(${target} PUBLIC
        --apcs=//interwork --arm
        --cpu=MPCore --fpmode=fast -Otime
        --data-reorder --split_sections
        --multibyte-chars --signed_chars
        ${LOCALE_FLAG}
    )
    target_compile_options(${target} PUBLIC $<$<COMPILE_LANGUAGE:C>:--c99>)
    target_compile_options(${target} PUBLIC $<$<COMPILE_LANGUAGE:CXX>:
        --cpp --force_new_nothrow --no_vfe
        --no_exceptions --no_rtti --no_rtti_data
        -O3 --forceinline
    >)
endfunction()
set_armcc_options(crt0)
set_armcc_options(ctrsdk)

