cmake_minimum_required(VERSION 3.0)
cmake_policy(SET CMP0054 NEW)
project(neon_regex)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "")

if (NOT EXISTS pcre2-10.10/CMakeLists.txt)
    execute_process(
        COMMAND tar zxvCf ${CMAKE_CURRENT_SOURCE_DIR} pcre2-10.10.tar.gz
    )
endif()
add_custom_command(
    OUTPUT lib/libpcre2-8.a
    COMMAND cd ${CMAKE_CURRENT_SOURCE_DIR}/pcre2-10.10 && CFLAGS=-fPIC ./configure --prefix=${CMAKE_CURRENT_SOURCE_DIR} && make && make install
    DEPENDS pcre2-10.10/configure
)
add_custom_target(
    pcre
    DEPENDS lib/libpcre2-8.a
)

add_library(neon_regex SHARED regex.cpp)
target_include_directories(neon_regex PRIVATE include)
target_link_libraries(neon_regex ${CMAKE_CURRENT_SOURCE_DIR}/lib/libpcre2-8.a)
add_dependencies(neon_regex pcre)
