cmake_minimum_required(VERSION 3.0.2)

project(WavGain)

option(ENABLE_RECURSIVE "Enable recursive processing and pattern matching" ON)


#------------------------------------------------------------
# Detect 64 bits
#------------------------------------------------------------

if (CMAKE_SIZEOF_VOID_P EQUAL 4)
  set(HAVE_64_BIT 0)
else ()
  set(HAVE_64_BIT 1)
endif ()

# Name of package
set(PACKAGE           "WavGain")
# Define to the address where bug reports for this package should be sent.
set(PACKAGE_BUGREPORT "bugs@wavgain.org")
# Define to the full name of this package.
set(PACKAGE_NAME      "WavGain")
# Define to the full name and version of this package.
set(PACKAGE_STRING    "WavGain")
# Define to the one symbol short name of this package.
set(PACKAGE_TARNAME   "wavgain")
# Define to the version of this package.
set(PACKAGE_VERSION   "1.3.1")

if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/wavgain_binary" 
      CACHE PATH "Cbc install prefix" FORCE)
endif (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)

include(CheckIncludeFile)

check_include_file("dirent.h"     HAVE_DIRENT_H)
check_include_file("errno.h"     HAVE_ERRNO_H)
check_include_file("inttypes.h"  HAVE_INTTYPES_H)
check_include_file("memory.h"    HAVE_MEMORY_H)
check_include_file("ndir.h"      HAVE_NDIR_H)
check_include_file("stdint.h"    HAVE_STDINT_H)
check_include_file("stdlib.h"    HAVE_STDLIB_H)
check_include_file("strings.h"   HAVE_STRINGS_H)
check_include_file("string.h"    HAVE_STRING_H)
check_include_file("sys/dir.h"   HAVE_SYS_DIR_H)
check_include_file("sys/ndir.h"  HAVE_SYS_NDIR_H)
check_include_file("sys/stat.h"  HAVE_SYS_STAT_H)
check_include_file("sys/types.h" HAVE_SYS_TYPES_H)
check_include_file("unistd.h"    HAVE_UNISTD_H)
check_include_file("utime.h"     HAVE_UTIME_H)

include(CheckFunctionExists)

check_function_exists(vprintf  HAVE_DOPRNT)
check_function_exists(vprintf  HAVE_VPRINTF)
check_function_exists(floor    HAVE_FLOOR)
check_function_exists(getcwd   HAVE_GETCWD)
check_function_exists(malloc   HAVE_MALLOC)
check_function_exists(memmove  HAVE_MEMMOVE)
check_function_exists(memset   HAVE_MEMSET)
check_function_exists(pow      HAVE_POW)
check_function_exists(strchr   HAVE_STRCHR)
check_function_exists(strdup   HAVE_STRDUP)
check_function_exists(strerror HAVE_STRERROR)

# Define if `stat' has the bug that it succeeds when given the zero-length file name argument.
##undef HAVE_STAT_EMPTY_STRING_BUG
# Define if `lstat' dereferences a symlink specified with a trailing slash.
##undef LSTAT_FOLLOWS_SLASHED_SYMLINK
# Define if you have the ANSI C header files.
##define STDC_HEADERS

configure_file(${CMAKE_SOURCE_DIR}/config.h.in ${CMAKE_BINARY_DIR}/config.h)

include_directories(${CMAKE_SOURCE_DIR}
                    ${CMAKE_BINARY_DIR})

add_definitions(-DHAVE_CONFIG_H)
if (WIN32)
  add_definitions(-DHAVE_SSE2 -D_CRT_SECURE_NO_WARNINGS)
endif ()

set(WAVGAIN_SRCS audio.c
                 dither.c
		 gain_analysis.c
		 getopt1.c
		 getopt.c
		 main.c
		 misc.c
		 recurse.c
		 wavegain.c)

add_executable(wavgain ${WAVGAIN_SRCS})
if (NOT WIN32)
  target_link_libraries(wavgain m)
endif ()

#
# Packaging
#

set(CPACK_PACKAGE_NAME      "${PACKAGE_NAME}")
set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${PACKAGE_VERSION}-${CMAKE_SYSTEM_NAME}")

message(STATUS "Package filename: ${CPACK_PACKAGE_FILE_NAME}")

set(CPACK_PACKAGE_INSTALL_DIRECTORY "${PACKAGE_NAME}")
set(CPACK_PACKAGE_VERSION "${PACKAGE_VERSION}")

set(CPACK_PACKAGE_RELOCATABLE TRUE)

if (WIN32)
  set(CPACK_PACKAGE_INSTALL_REGISTRY_KEY "${CPACK_PACKAGE_NAME}")
  set(CPACK_MONOLITHIC_INSTALL ON)
  set(CPACK_NSIS_CONTACT "${PACKAGE_BUGREPORT}")
  set(CPACK_NSIS_MODIFY_PATH ON)
  set(CPACK_NSIS_PACKAGE_NAME "WavGain ${PACKAGE_VERSION}")
elseif (APPLE)
  set(CPACK_COMPONENTS_ALL_IN_ONE_PACKAGE ON)
  set(CPACK_PACKAGE_DEFAULT_LOCATION "/opt/${CPACK_PACKAGE_NAME}")
  set(CPACK_PACKAGING_INSTALL_PREFIX "/")
else (WIN32)
  set(CPACK_COMPONENTS_ALL_IN_ONE_PACKAGE ON)
  set(CPACK_PACKAGE_DEFAULT_LOCATION "/opt/${CPACK_PACKAGE_NAME}")
  set(CPACK_PACKAGING_INSTALL_PREFIX "/")
  
  set(CPACK_DEBIAN_PACKAGE_SECTION "Audio")
  set(CPACK_DEBIAN_PACKAGE_MAINTAINER "${PACKAGE_BUGREPORT}")
  
  set(CPACK_RPM_COMPONENT_INSTALL ON)
  set(CPACK_RPM_PACKAGE_RELOCATABLE ON)
  set(CPACK_RPM_PACKAGE_LICENSE "Copyrighted")
  set(CPACK_RPM_PACKAGE_GROUP "Audio")
  set(CPACK_RPM_PACKAGE_URL "${PACKAGE_URL}")
  set(CPACK_RPM_PACKAGE_SUMMARY "WavGain is a tool to correct the gain of several wav files.")
  set(CPACK_RPM_PACKAGE_DESCRIPTION "The WavGain is a tool to correct the gain of several wav files.")
endif ()

include(CPack)

