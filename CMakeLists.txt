CMAKE_MINIMUM_REQUIRED(VERSION 3.4)

PROJECT(OpenSceneGraph-3rdPartyLibs)

INCLUDE(CMakeModules/FunctionDefinitions.cmake)

# Some global options
SET(CMAKE_DEBUG_POSTFIX "d")

# Only enable release and debug builds
IF(CMAKE_CONFIGURATION_TYPES)
  SET(CMAKE_CONFIGURATION_TYPES Debug Release)
  SET(CMAKE_CONFIGURATION_TYPES "${CMAKE_CONFIGURATION_TYPES}" CACHE STRING
    "Reset the configurations to what we need"
    FORCE)
ENDIF()

# Determine architecture bitness
IF(${CMAKE_SIZEOF_VOID_P} EQUAL 8)
    SET(BITS "x64")
ELSEIF (${CMAKE_SIZEOF_VOID_P} EQUAL 4)
    SET(BITS "x86")
ENDIF()

# Determine compiler name (and version for VS)
IF (MSVC)
    IF (${MSVC_VERSION} EQUAL 1600)
        SET(TOOLSET "v100") # Visual Studio 2010
    ELSEIF (${MSVC_VERSION} EQUAL 1700)
        SET(TOOLSET "v110") # Visual Studio 2012
    ELSEIF (${MSVC_VERSION} EQUAL 1800)
        SET(TOOLSET "v120") # Visual Studio 2013
    ELSEIF (${MSVC_VERSION} EQUAL 1900)
        SET(TOOLSET "v140") # Visual Studio 2015
    ELSEIF (${MSVC_VERSION} EQUAL 1910)
        SET(TOOLSET "v141") # Visual Studio 2017
    ENDIF()
ELSEIF(MINGW)
    SET(TOOLSET "mingw")
ELSE()
    MESSAGE(FATAL_ERROR "Unknown compiler/toolset being used.")
ENDIF()

# Output root directory name
SET(dirname "${TOOLSET}-${BITS}")

# Set output sub-directory names
SET(EXECUTABLE_OUTPUT_PATH "${dirname}/bin")
SET(ARCHIVE_OUTPUT_PATH "${dirname}/lib")
SET(LIBRARY_OUTPUT_PATH "${dirname}/lib")
SET(INCLUDE_OUTPUT_PATH "${dirname}/include")

# Configurations for specifying the 3rd-Party libraries to build
SET(USE_ZLIB ON CACHE BOOL "Use and compile ZLIB source")
SET(USE_LIBPNG ON CACHE BOOL "Use and compile LIBPNG source")
SET(USE_LIBJPEG ON CACHE BOOL "Use and compile LIBJPEG source")
SET(USE_LIBTIFF ON CACHE BOOL "Use and compile LIBTIFF source")
SET(USE_FREETYPE ON CACHE BOOL "Use and compile FREETYPE source")
SET(USE_GLUT ON CACHE BOOL "Use and compile GLUT source")
SET(USE_GIFLIB ON CACHE BOOL "Use and compile GIFLIB source")
SET(USE_MINIZIP ON CACHE BOOL "Use and compile MINIZIP source")
SET(USE_CURL ON CACHE BOOL "Use and compile cURL source")

# Add all the libraries
ADD_SUBDIRECTORY(zlib)
ADD_SUBDIRECTORY(libpng)
ADD_SUBDIRECTORY(libjpeg)
ADD_SUBDIRECTORY(libtiff)
ADD_SUBDIRECTORY(freetype)
ADD_SUBDIRECTORY(glut)
ADD_SUBDIRECTORY(giflib)
ADD_SUBDIRECTORY(minizip)
ADD_SUBDIRECTORY(curl)
