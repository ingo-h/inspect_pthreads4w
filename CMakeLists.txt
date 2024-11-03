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
        EXCLUDE_FROM_ALL
        SYSTEM
    )

    # Check if population has already been performed
    FetchContent_GetProperties(pthreads4w)
    if(NOT pthreads4w_POPULATED)
        # Fetch the content using previously declared details.
        # This caches pthreads4w_SOURCE_DIR, pthreads4w_BINARY_DIR and pthreads4w_POPULATED.
        FetchContent_Populate(pthreads4w)
        #FetchContent_MakeAvailable(pthreads4w)

        execute_process(COMMAND cmake -S ${pthreads4w_SOURCE_DIR} -B ${pthreads4w_BINARY_DIR} -G "NMake Makefiles" -D CMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -D BUILD_TESTING=${PT4W_BUILD_TESTING}
                        WORKING_DIRECTORY build/_deps/pthreads4w-src
                        RESULT_VARIABLE PT4W_RETURN_CODE
                        ERROR_VARIABLE PT4W_ERROR_MESSAGE
                        COMMAND_ECHO STDOUT)
        if(NOT ${PT4W_RETURN_CODE} EQUAL 0)
            # This will stop the installation
            message(FATAL_ERROR "${PT4W_RETURN_CODE}: ${PT4W_ERROR_MESSAGE}")
        endif()
    endif() # NOT pthreads4w_POPULATED

    message(CHECK_PASS "done")

    add_subdirectory(${pthreads4w_SOURCE_DIR} ${pthreads4w_BINARY_DIR} EXCLUDE_FROM_ALL SYSTEM)

endif() # WIN32
