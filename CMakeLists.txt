# Copyright (C) 2024+ GPL 3 and higher by Ingo Höft, <Ingo@Hoeft-online.de>
# Redistribution only with this Copyright remark. Last modified: 2024-11-04

# REF: [Install third party POSIX Threads for Windows](https://stackoverflow.com/q/69275376/5014688)

# Configure and build with:
# cmake -S . -B build [-D CMAKE_BUILD_TYPE=Release|Debug]
# cmake --build build --config Release|Debug

cmake_minimum_required(VERSION 3.29)
include(cmake/project-header.cmake)

# set the project name and version
project(INSPECT_PTHREADS4W VERSION 0001
        DESCRIPTION "Inspect Posix Threads for Windows"
        HOMEPAGE_URL "https://github.com/ingo-h")

enable_testing()

option(PT4W_BUILD_TESTING "Build the time and space consuming test suite for pthreads4w" OFF)

# Set the default build type
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build, options are: None, Debug, Release, RelWithDebInfo, or MinSizeRel." FORCE)
    message(STATUS "No build type specified, defaulting to Release.")
endif()


##########################################
# POSIX Threads for Windows (pthreads4w) #
##########################################
if(WIN32)
    include(ExternalProject)
    message(CHECK_START "Add external project \"POSIX Threads for Windows\"")

    ExternalProject_Add(pthreads4w
        # Download
        GIT_REPOSITORY  https://github.com/jwinarske/pthreads4w.git
        GIT_TAG         origin/cmake
        GIT_SHALLOW     ON

        # Configure
        CMAKE_GENERATOR "NMake Makefiles"
        CMAKE_CACHE_ARGS -DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE} -DBUILD_TESTING:BOOL=${PT4W_BUILD_TESTING}
        CONFIGURE_HANDLED_BY_BUILD ON
    )

    ExternalProject_Get_Property(pthreads4w SOURCE_DIR)
    file(GLOB_RECURSE pthreads4w_INCLUDE_FILE CONFIGURE_DEPENDS ${SOURCE_DIR}/PTHREADS-BUILT/pthread.h)
    cmake_path(GET pthreads4w_INCLUDE_FILE PARENT_PATH PTHREADS4W_INCLUDE_DIR)

    file(GLOB_RECURSE pthreads4w_SHARED_LIBRARY CONFIGURE_DEPENDS ${SOURCE_DIR}/PTHREADS-BUILT/pthreadVC3*.lib)
    set(UPNP_PTHREADS_SHARED_LIBRARY "${pthreads4w_SHARED_LIBRARY}" CACHE INTERNAL
        "Name of the linkable shared library for POSIX threads" FORCE)

    file(GLOB_RECURSE pthreads4w_STATIC_LIBRARY CONFIGURE_DEPENDS ${SOURCE_DIR}/PTHREADS-BUILT/libpthreadVC3*.lib)
    set(UPNP_PTHREADS_STATIC_LIBRARY "${pthreads4w_STATIC_LIBRARY}" CACHE INTERNAL
        "Name of the linkable static library for POSIX threads" FORCE)

    # Output with cmake option --log-level=DEBUG
    message(DEBUG "  DEBUG: pthreads4w SOURCE_DIR = ${SOURCE_DIR}")
    message(DEBUG "  DEBUG: PTHREADS4W_INCLUDE_DIR = ${PTHREADS4W_INCLUDE_DIR}")
    message(DEBUG "  DEBUG: UPNP_PTHREADS_SHARED_LIBRARY = ${UPNP_PTHREADS_SHARED_LIBRARY}")
    message(DEBUG "  DEBUG: UPNP_PTHREADS_STATIC_LIBRARY = ${UPNP_PTHREADS_STATIC_LIBRARY}")

    message(CHECK_PASS "done")

endif() # WIN32
