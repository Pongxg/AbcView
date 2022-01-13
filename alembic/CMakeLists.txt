##-*****************************************************************************
##
## Copyright (c) 2009-2016,
##  Sony Pictures Imageworks Inc. and
##  Industrial Light & Magic, a division of Lucasfilm Entertainment Company Ltd.
##
## All rights reserved.
##
## Redistribution and use in source and binary forms, with or without
## modification, are permitted provided that the following conditions are
## met:
## *       Redistributions of source code must retain the above copyright
## notice, this list of conditions and the following disclaimer.
## *       Redistributions in binary form must reproduce the above
## copyright notice, this list of conditions and the following disclaimer
## in the documentation and/or other materials provided with the
## distribution.
## *       Neither the name of Industrial Light & Magic nor the names of
## its contributors may be used to endorse or promote products derived
## from this software without specific prior written permission.
##
## THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
## "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
## LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
## A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
## OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
## SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
## LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
## DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
## THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
## (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
## OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
##
##-*****************************************************************************

CMAKE_MINIMUM_REQUIRED(VERSION 3.13)

PROJECT(Alembic VERSION 1.8.2)

MESSAGE(STATUS "CMAKE SYSTEM NAME: ${CMAKE_SYSTEM_NAME}")

IF (RUN_FROM_MK)
    SET(CMAKE_FIRST_RUN CACHE STRING "CMake initialised from mk script")
    MARK_AS_ADVANCED (CMAKE_FIRST_RUN)
ENDIF()

IF (NOT DEFINED QUIET)
    SET(CMAKE_VERBOSE_MAKEFILE TRUE)
ENDIF()

#-******************************************************************************
# OPTIONS (set with -D<option>=<value>)
#-******************************************************************************
OPTION(USE_TESTS "Include Alembic tests" OFF)
OPTION(USE_BINARIES "Include binaries" ON)
OPTION(USE_STATIC_HDF5 "Build with static HDF5 libs" OFF)
OPTION(ALEMBIC_BUILD_LIBS "Build library, if off use external alembic libs" ON)
OPTION(ALEMBIC_ILMBASE_LINK_STATIC "IlmBase is a static library" OFF)
OPTION(ALEMBIC_SHARED_LIBS "Build shared libraries" ON)

# Set static/dynamic build options
SET(LIB_TYPE STATIC)
SET(RUNTIME_INSTALL_DIR lib)
SET(LIBRARY_INSTALL_DIR lib)
SET(ARCHIVE_INSTALL_DIR lib)
IF (ALEMBIC_SHARED_LIBS)
    SET(LIB_TYPE SHARED)
    SET(ARCHIVE_INSTALL_DIR lib)
    IF (WIN32)
        ADD_DEFINITIONS(-DALEMBIC_DLL)
    ENDIF()
ENDIF(ALEMBIC_SHARED_LIBS)

# Need to test this on multiple platforms, it is an easy way to enable
# memory checks like valgrind by doing:
# ctest -D ExperimentalMemCheck
# include (CTest)

# Cmake system specific flags
SET(WINDOWS FALSE)
IF ("${CMAKE_SYSTEM_NAME}" MATCHES "Windows")
    SET(WINDOWS TRUE)
ENDIF()

SET(DARWIN FALSE)
IF ("${CMAKE_SYSTEM_NAME}" MATCHES "Darwin")
    SET(DARWIN TRUE)
    # suppress rpath warning
    IF (POLICY CMP0042)
        CMAKE_POLICY(SET CMP0042 OLD)
    ENDIF()
ENDIF()

IF (APPLE)
  SET(CMAKE_MACOSX_RPATH ON)
ENDIF()

SET(LINUX FALSE)
IF ("${CMAKE_SYSTEM_NAME}" MATCHES "Linux")
    SET(LINUX TRUE)
ENDIF()

set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /Zi")
set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "${CMAKE_SHARED_LINKER_FLAGS_RELEASE} /DEBUG /OPT:REF /OPT:ICF")
set(CMAKE_CXX_STANDARD 14)

# Set visibility for GNU compilers
IF (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility=hidden")
ENDIF()

# Set some debug vs opt flags
if ("${CMAKE_BUILD_TYPE}" MATCHES "Debug" AND NOT MSVC)
    add_definitions(-Wall -Werror -Wextra -Wno-unused-parameter -Wno-deprecated)
    if((CMAKE_CXX_COMPILER_ID MATCHES "GNU" AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 7.0) OR
        CMAKE_CXX_COMPILER_ID MATCHES "CLANG")
        add_definitions( -Wno-error=implicit-fallthrough)
    endif()
endif()

IF (NOT ${WINDOWS})
    SET(EXTERNAL_MATH_LIBS "-lm")
ELSE()
    SET(EXTERNAL_MATH_LIBS "")
ENDIF()

IF (MSVC)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc")
    IF ((CMAKE_CXX_COMPILER_VERSION VERSION_LESS 16) AND
        (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 15))
        # MSVC15/MSVS2009 fix
        SET (CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} /bigobj" )
    ENDIF ()
    ADD_DEFINITIONS(-DPLATFORM_WINDOWS -DPLATFORM=WINDOWS)
    IF (NOT ALEMBIC_ILMBASE_LINK_STATIC)
        ADD_DEFINITIONS(-DOPENEXR_DLL)
    ENDIF()
ELSEIF (${DARWIN})
    #ADD_DEFINITIONS(-DPLATFORM_DARWIN -DPLATFORM=DARWIN "-arch x86_64")
    ADD_DEFINITIONS(-DPLATFORM_DARWIN -DPLATFORM=DARWIN)
ELSE()
    ADD_DEFINITIONS(-DPLATFORM_LINUX -DPLATFORM=LINUX)
ENDIF()

#-******************************************************************************
# INSTALLATION
#-******************************************************************************

IF (DEFINED ENV{ALEMBIC_INSTALL_PREFIX})
    SET( CMAKE_INSTALL_PREFIX
        $ENV{ALEMBIC_INSTALL_PREFIX}/alembic-${PROJECT_VERSION})
ENDIF()

# Tell me what my install location would be
MESSAGE(STATUS "The install dir is ${CMAKE_INSTALL_PREFIX}")

# check in the source directory
SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake/Modules)


#-******************************************************************************
# DEPENDENCIES
#-******************************************************************************

FIND_PACKAGE(Threads REQUIRED)


# HDF5
IF (USE_HDF5)
    FIND_PACKAGE(ZLIB REQUIRED)
    SET(ALEMBIC_WITH_HDF5 "1")
    INCLUDE("./cmake/AlembicHDF5.cmake")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DH5_USE_18_API")
ENDIF()

#-******************************************************************************
# BUILD LIBRARIES
#-******************************************************************************

# Alembic
IF (ALEMBIC_BUILD_LIBS)
    ADD_SUBDIRECTORY(lib)
    INCLUDE_DIRECTORIES("${CMAKE_SOURCE_DIR}/lib" "${PROJECT_BINARY_DIR}/lib")
    ADD_LIBRARY( Alembic::Alembic ALIAS Alembic )
ELSE()
    FIND_PACKAGE(Alembic REQUIRED CONFIG HINTS ${ALEMBIC_ROOT})
    MESSAGE(STATUS "Using external Alembic")
ENDIF()
