cmake_minimum_required(VERSION 3.0)
cmake_policy(SET CMP0054 NEW)
project(neon_regex)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "")

find_path(PCRE_INCLUDE_DIRS NAMES pcre2.h)
find_library(PCRE_LIBRARY NAMES pcre2-8)
include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(PCRE
    DEFAULT_MSG
    PCRE_LIBRARY
    PCRE_INCLUDE_DIRS)

if (PCRE_FOUND)
    add_library(PCRE::PCRE UNKNOWN IMPORTED)
    set_target_properties(PCRE::PCRE PROPERTIES INTERFACE_INCLUDE_DIRECTORIES "${PCRE_INCLUDE_DIRS}")
    set_target_properties(PCRE::PCRE PROPERTIES IMPORTED_LOCATION "${PCRE_LIBRARY}")
    set(PCRE_TARGET "PCRE::PCRE")
else ()
    if (NOT EXISTS "pcre2-10.10/CMakeLists.txt")
        execute_process(
            COMMAND cmake -E tar zxvf pcre2-10.10.tar.gz
            WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        )
    endif()

    if (WIN32)
        execute_process(
            COMMAND cmake -E copy "pcre2-10.10/src/config.h.generic" "pcre2-10.10/src/config.h"
            COMMAND cmake -E copy "pcre2-10.10/src/pcre2.h.generic" "pcre2-10.10/src/pcre2.h"
            COMMAND cmake -E copy "pcre2-10.10/src/pcre2_chartables.c.dist" "pcre2-10.10/src/pcre2_chartables.c"
        )
        add_library(pcre2 STATIC
            "pcre2-10.10/src/pcre2_auto_possess.c"
            "pcre2-10.10/src/pcre2_chartables.c"
            "pcre2-10.10/src/pcre2_compile.c"
            "pcre2-10.10/src/pcre2_config.c"
            "pcre2-10.10/src/pcre2_context.c"
            "pcre2-10.10/src/pcre2_dfa_match.c"
            "pcre2-10.10/src/pcre2_error.c"
            "pcre2-10.10/src/pcre2_jit_compile.c"
            "pcre2-10.10/src/pcre2_maketables.c"
            "pcre2-10.10/src/pcre2_match.c"
            "pcre2-10.10/src/pcre2_match_data.c"
            "pcre2-10.10/src/pcre2_newline.c"
            "pcre2-10.10/src/pcre2_ord2utf.c"
            "pcre2-10.10/src/pcre2_pattern_info.c"
            "pcre2-10.10/src/pcre2_serialize.c"
            "pcre2-10.10/src/pcre2_string_utils.c"
            "pcre2-10.10/src/pcre2_study.c"
            "pcre2-10.10/src/pcre2_substitute.c"
            "pcre2-10.10/src/pcre2_substring.c"
            "pcre2-10.10/src/pcre2_tables.c"
            "pcre2-10.10/src/pcre2_ucd.c"
            "pcre2-10.10/src/pcre2_valid_utf.c"
            "pcre2-10.10/src/pcre2_xclass.c"
        )
        target_compile_definitions(pcre2 PRIVATE
            HAVE_CONFIG_H
            PCRE2_CODE_UNIT_WIDTH=8
            PCRE2_STATIC
        )
        target_include_directories(pcre2 PUBLIC "pcre2-10.10/src")
        set(PCRE_TARGET "pcre2")
    else()
        execute_process(
            COMMAND cmake -E make_directory "${CMAKE_CURRENT_SOURCE_DIR}/include"
        )

        add_custom_command(
            OUTPUT lib/libpcre2-8.a
            COMMAND cd ${CMAKE_CURRENT_SOURCE_DIR}/pcre2-10.10 && CFLAGS=-fPIC ./configure --prefix=${CMAKE_CURRENT_SOURCE_DIR} && make && make install
            DEPENDS pcre2-10.10/configure
        )
        add_custom_target(libpcre DEPENDS lib/libpcre2-8.a)
        add_library(PCRE STATIC IMPORTED GLOBAL)
        add_dependencies(PCRE libpcre)
        set_target_properties(PCRE PROPERTIES INTERFACE_INCLUDE_DIRECTORIES "${CMAKE_CURRENT_LIST_DIR}/include")
        set_target_properties(PCRE PROPERTIES IMPORTED_LOCATION "${CMAKE_CURRENT_LIST_DIR}/lib/libpcre2-8.a")
        set(PCRE_TARGET "PCRE")
    endif()
endif()

add_library(neon_regex SHARED regex.cpp)
target_link_libraries(neon_regex ${PCRE_TARGET})
