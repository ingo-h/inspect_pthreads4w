# Copyright (C) 2024+ GPL 3 and higher by Ingo Höft, <Ingo@Hoeft-online.de>
# Redistribution only with this Copyright remark. Last modified: 2024-11-03

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


##########################################
# POSIX Threads for Windows (pthreads4w) #
##########################################
if(WIN32)
    # This dependent project must use generator "NMake Makefiles" that differs
    # from default generator "Visual Studio *". NMake Makefiles does not
    # support platform specification with CMake option '-A'. So we cannot use
    # add_directory() because this will also compile the dependent project with
    # "Visual Studio *". We have to configure and build it in separate
    # processes.

    include(FetchContent)
    message(CHECK_START "Download and configuring POSIX Threads for Windows")

    FetchContent_Declare(
        pthreads4w
        GIT_REPOSITORY  https://github.com/jwinarske/pthreads4w.git
        GIT_TAG         origin/cmake
        GIT_SHALLOW     ON
    )
    # Check if population has already been performed
    FetchContent_GetProperties(pthreads4w)
    if(NOT pthreads4w_POPULATED)
        message(STATUS "NOT POPULATED")
        # Fetch the content using previously declared details.
        # This caches pthreads4w_SOURCE_DIR, pthreads4w_BINARY_DIR and pthreads4w_POPULATED.
        FetchContent_Populate(pthreads4w)

        configure_file(cmake/CMakePresets-pthreads4w.json
                       _deps/pthreads4w-src/CMakePresets.json
                       COPYONLY)
        #FetchContent_Populate(pthreads4w)
        message(STATUS "CMakePresets should have been copied")
    endif()

    #FetchContent_MakeAvailable(pthreads4w)
    message(CHECK_PASS "done")

endif()
