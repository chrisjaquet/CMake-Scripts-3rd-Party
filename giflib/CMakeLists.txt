IF(NOT USE_GIFLIB)
    return()
ENDIF()

PROJECT(giflib C)
SET(GIFLIB_VERSION "5.1.4")

SET(TARGET_LIB giflib)

# Try to download the source from the internet if no source folder is specified.
IF(NOT GIFLIB_SOURCE_DIR)
    DownloadAndExtract("https://sourceforge.net/projects/giflib/files/giflib-${GIFLIB_VERSION}.tar.gz/download"
        ${CMAKE_CURRENT_SOURCE_DIR}/giflib-${GIFLIB_VERSION}.tar.gz
        ${CMAKE_CURRENT_BINARY_DIR}
    )
    SET(_GIFLIB_SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/giflib-${GIFLIB_VERSION})
ELSE()
    SET(_GIFLIB_SOURCE_DIR ${GIFLIB_SOURCE_DIR})
ENDIF()
SET(GIFLIB_SOURCE_DIR ${_GIFLIB_SOURCE_DIR} CACHE PATH "Path where to find GIFLIB source")

SET(GIFLIB_VERSION_FILE "${GIFLIB_SOURCE_DIR}/lib/gif_lib.h")

IF(EXISTS "${GIFLIB_VERSION_FILE}") 
    FILE(STRINGS "${GIFLIB_VERSION_FILE}" GIFLIB_VERSION_FILE_CONTENTS REGEX "#define GIFLIB_[A-Z]+[ \t]+[0-9]+")

    STRING(REGEX REPLACE ".*#define GIFLIB_MAJOR[ \t]+([0-9]+).*" "\\1" GIFLIB_VERSION_MAJOR ${GIFLIB_VERSION_FILE_CONTENTS})
    STRING(REGEX REPLACE ".*#define GIFLIB_MINOR[ \t]+([0-9]+).*" "\\1" GIFLIB_VERSION_MINOR ${GIFLIB_VERSION_FILE_CONTENTS})
    STRING(REGEX REPLACE ".*#define GIFLIB_RELEASE[ \t]+([0-9]+).*" "\\1" GIFLIB_VERSION_RELEASE ${GIFLIB_VERSION_FILE_CONTENTS})
    
    SET(GIFLIB_VERSION "${GIFLIB_VERSION_MAJOR}.${GIFLIB_VERSION_MINOR}.${GIFLIB_VERSION_RELEASE}")
ENDIF()

SET(GIFLIB_PUBLIC_HEADERS
    ${GIFLIB_SOURCE_DIR}/lib/gif_lib.h
)

SET(GIFLIB_PRIVATE_HEADERS
    ${GIFLIB_SOURCE_DIR}/lib/gif_lib_private.h
    ${GIFLIB_SOURCE_DIR}/lib/gif_hash.h
)

SET(GIFLIB_SRCS
    ${GIFLIB_SOURCE_DIR}/lib/dgif_lib.c
    ${GIFLIB_SOURCE_DIR}/lib/egif_lib.c
    ${GIFLIB_SOURCE_DIR}/lib/gif_err.c
    ${GIFLIB_SOURCE_DIR}/lib/gif_font.c
    ${GIFLIB_SOURCE_DIR}/lib/gif_hash.c
    ${GIFLIB_SOURCE_DIR}/lib/gifalloc.c
    ${GIFLIB_SOURCE_DIR}/lib/quantize.c
)

# Add openbsd-reallocarray from 5.1.2 and up
IF(NOT GIFLIB_VERSION VERSION_LESS 5.1.2)
    SET(GIFLIB_SRCS
        ${GIFLIB_SRCS}
        ${GIFLIB_SOURCE_DIR}/lib/openbsd-reallocarray.c
    )
ENDIF()


IF(MSVC)
    # Create file to avoid include file name collision on other platforms
    CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/unistd.h.in unistd.h @ONLY)    

    # Add unistd.h, getopt.h, getopt.c and stdbool.h for windows build
    # getopt from: http://gist.github.com/ashelly/7776712
    # unistd.h from: http://stackoverflow.com/questions/341817/is-there-a-replacement-for-unistd-h-for-windows-visual-c
    # stdbool.h is just a simple redefinition of bool as int
    SET(GIFLIB_PUBLIC_HEADERS
        ${GIFLIB_PUBLIC_HEADERS}
        ${CMAKE_CURRENT_SOURCE_DIR}/stdbool.h
    )

    SET(GIFLIB_PRIVATE_HEADERS
        ${GIFLIB_PRIVATE_HEADERS}
        ${CMAKE_CURRENT_SOURCE_DIR}/getopt.h
        ${CMAKE_CURRENT_SOURCE_DIR}/unistd.h
    )

    SET(GIFLIB_SRCS
        ${GIFLIB_SRCS}
        ${CMAKE_CURRENT_SOURCE_DIR}/getopt.c
    )
ENDIF(MSVC)


# Dependent include files
INCLUDE_DIRECTORIES(${GIFLIB_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR})

IF(MSVC)
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /wd4018 /wd4028 /wd4244 /wd4996")
    ADD_DEFINITIONS(-D_CRT_SECURE_NO_WARNINGS -D_LIB -DWIN32)
    SET(CMAKE_DEBUG_POSTFIX "d")
ENDIF(MSVC)

IF(MINGW)
    ADD_DEFINITIONS(-D_CRT_SECURE_NO_WARNINGS -D_LIB)
ENDIF(MINGW)

ADD_LIBRARY(${TARGET_LIB} STATIC ${GIFLIB_SRCS} ${GIFLIB_PRIVATE_HEADERS} ${GIFLIB_PUBLIC_HEADERS})

IF(MINGW)
    SET_TARGET_PROPERTIES(${TARGET_LIB} PROPERTIES OUTPUT_NAME gif)
ENDIF()

# Runtime equals DLL-files
# Archive equals static libraries
# Library equals module libraries
IF(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL )
    INSTALL(TARGETS ${TARGET_LIB}
        RUNTIME DESTINATION ${EXECUTABLE_OUTPUT_PATH}
        LIBRARY DESTINATION ${LIBRARY_OUTPUT_PATH}
        ARCHIVE DESTINATION ${ARCHIVE_OUTPUT_PATH})
ENDIF()

IF(NOT SKIP_INSTALL_HEADERS AND NOT SKIP_INSTALL_ALL )
    INSTALL(FILES ${GIFLIB_PUBLIC_HEADERS} DESTINATION ${INCLUDE_OUTPUT_PATH})
ENDIF()
