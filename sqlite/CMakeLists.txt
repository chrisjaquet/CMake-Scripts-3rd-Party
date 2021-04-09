# This CMakeFile is for building SQLite from the sqlite-amalgamation distribution available at https://www.sqlite.org
#
# It can be used to build the library either as a shared library or statically.
#
# Update as necessary for the specific version of SQLite you are trying to build.
#
# This file was written based on build instructions found on the official SQLite website. It is released under the terms
# of the Unlicense:
#
# Originally written by Chris Jaquet.
#
# ------------------------------------------------------------------------------
#     This is free and unencumbered software released into the public domain.
#
#     Anyone is free to copy, modify, publish, use, compile, sell, or
#     distribute this software, either in source code form or as a compiled
#     binary, for any purpose, commercial or non-commercial, and by any
#     means.
#
#     In jurisdictions that recognize copyright laws, the author or authors
#     of this software dedicate any and all copyright interest in the
#     software to the public domain. We make this dedication for the benefit
#     of the public at large and to the detriment of our heirs and
#     successors. We intend this dedication to be an overt act of
#     relinquishment in perpetuity of all present and future rights to this
#     software under copyright law.
#
#     THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
#     EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
#     MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
#     IN NO EVENT SHALL THE AUTHORS BE LIABLE FOR ANY CLAIM, DAMAGES OR
#     OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE,
#     ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
#     OTHER DEALINGS IN THE SOFTWARE.
#
#     For more information, please refer to <http://unlicense.org/>
#
# ------------------------------------------------------------------------------
#
cmake_minimum_required(VERSION 3.9)

project(SQLite
    VERSION 3.30.1
)

set(CMAKE_DEBUG_POSTFIX "d")
option(SQLITE_BUILD_SHARED "Enable to build the library as shared library (DLL)" OFF)

option(SQLITE_ENABLE_FTS4 "Include the full-text search engine code in SQLite." ON)
option(SQLITE_ENABLE_RTREE "Include the R-Tree extension." ON)
option(SQLITE_ENABLE_COLUMN_METADATA "This enables some extra APIs that are required by some common systems, including Ruby-on-Rails." ON)

mark_as_advanced(SQLITE_ENABLE_FTS4 SQLITE_ENABLE_RTREE SQLITE_ENABLE_COLUMN_METADATA)

if(UNIX)
    set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
    set(THREADS_PREFER_PTHREAD_FLAG TRUE)
    find_package(Threads REQUIRED)
endif()

# Check which definitions need to be applied ---------------------------------------------------------------------------
set(SQLITE_COMPILE_DEFINITIONS)
if(SQLITE_ENABLE_FTS4)
    list(APPEND SQLITE_COMPILE_DEFINITIONS -DSQLITE_ENABLE_FTS4)
endif()
if(SQLITE_ENABLE_RTREE)
    list(APPEND SQLITE_COMPILE_DEFINITIONS -DSQLITE_ENABLE_RTREE)
endif()
if(SQLITE_ENABLE_COLUMN_METADATA)
    list(APPEND SQLITE_COMPILE_DEFINITIONS -DSQLITE_ENABLE_COLUMN_METADATA)
endif()

# The library ----------------------------------------------------------------------------------------------------------
set(SQLITE_LIB_SOURCES
    src/sqlite3.c
)
set(SQLITE_LIB_HEADERS
    include/sqlite3.h
    include/sqlite3ext.h
)

if(SQLITE_BUILD_SHARED)
    add_library(sqlite_lib SHARED ${SQLITE_LIB_SOURCES} ${SQLITE_LIB_HEADERS})
else()
    add_library(sqlite_lib STATIC ${SQLITE_LIB_SOURCES} ${SQLITE_LIB_HEADERS})
endif()
target_include_directories(sqlite_lib
    PUBLIC
        $<INSTALL_INTERFACE:include>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src
)
target_compile_definitions(sqlite_lib PUBLIC ${SQLITE_COMPILE_DEFINITIONS})
set_target_properties(sqlite_lib PROPERTIES
    OUTPUT_NAME sqlite3
    PDB_NAME_DEBUG sqlite3${CMAKE_DEBUG_POSTFIX}
)

# On Linux we need to include pthreads.
if(UNIX)
    target_link_libraries(sqlite_lib
        Threads::Threads
        ${CMAKE_DL_LIBS}
    )
endif()

# The shell application ------------------------------------------------------------------------------------------------
set(SQLITE_EXE_SOURCES
    src/sqlite3.c
    src/shell.c
)
set(SQLITE_EXE_HEADERS
    include/sqlite3.h
    include/sqlite3ext.h
)
add_executable(sqlite
    ${SQLITE_EXE_SOURCES}
    ${SQLITE_EXE_HEADERS}
)
set_target_properties(sqlite PROPERTIES
    DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX}
)
target_include_directories(sqlite
    PRIVATE
        $<INSTALL_INTERFACE:include>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        ${CMAKE_CURRENT_SOURCE_DIR}/src
)
target_compile_definitions(sqlite PUBLIC ${SQLITE_COMPILE_DEFINITIONS})
if(UNIX)
    target_link_libraries(sqlite
        Threads::Threads
    )
endif()

# Install Targets ------------------------------------------------------------------------------------------------------
install(TARGETS sqlite sqlite_lib
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    INCLUDES DESTINATION include
)

install(DIRECTORY include/
    DESTINATION include
    PATTERN "*.h"
)

if(WIN32)
    if(SQLITE_BUILD_SHARED)
        install(FILES $<TARGET_PDB_FILE:sqlite_lib>
            CONFIGURATIONS Debug
            DESTINATION bin
        )
    endif()
endif()
