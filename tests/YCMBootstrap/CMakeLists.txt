#=============================================================================
# Copyright 2014 Istituto Italiano di Tecnologia (IIT)
#   Authors: Daniele E. Domenichelli <daniele.domenichelli@iit.it>
#
# Distributed under the OSI-approved BSD License (the "License");
# see accompanying file Copyright.txt for details.
#
# This software is distributed WITHOUT ANY WARRANTY; without even the
# implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
# See the License for more information.
#=============================================================================
# (To distribute this file outside of YCM, substitute the full
#  License text for the above reference.)

cmake_minimum_required(VERSION 3.0)
project(YCMbootstrap NONE)

include(FeatureSummary)

set(CMAKE_PREFIX_PATH )
set(ENV{CMAKE_PREFIX_PATH} )

file(MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/cmake")
set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_BINARY_DIR}/cmake")

get_filename_component(YCM_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../" REALPATH)
get_filename_component(YCM_BINARY_DIR "${CMAKE_CURRENT_BINARY_DIR}/../../" REALPATH)

file(COPY "${YCM_SOURCE_DIR}/tools/YCMBootstrap.cmake"
          "${YCM_SOURCE_DIR}/modules/IncludeUrl.cmake"
          DESTINATION "${CMAKE_CURRENT_BINARY_DIR}/cmake")

# Download files from local file system
set(YCM_BOOTSTRAP_BASE_ADDRESS file://${YCM_SOURCE_DIR} CACHE STRING "Base address of YCM repository")

if(EXISTS "${YCM_SOURCE_DIR}/.git")
    find_package(Git REQUIRED)
    execute_process(COMMAND "${GIT_EXECUTABLE}" rev-list --max-count=1 HEAD
                    WORKING_DIRECTORY ${YCM_SOURCE_DIR}
                    RESULT_VARIABLE error_code
                    OUTPUT_VARIABLE head_sha
                    OUTPUT_STRIP_TRAILING_WHITESPACE)
    if(NOT error_code)
        set(YCM_TAG "${head_sha}")
    endif()

    if(NOT EXISTS "${YCM_SOURCE_DIR}/.git/shallow")
        # This is a proper git repo, we can clone from it.
        set(YCM_STYLE LOCAL)
        set(YCM_REPOSITORY file://${YCM_SOURCE_DIR})

        if(EXISTS "${YCM_SOURCE_DIR}/downloads")
            # Perform initial clone and copy downloads folder to avoid multiple
            # downloads.
            execute_process(COMMAND ${GIT_EXECUTABLE} clone "${YCM_REPOSITORY}" "external/YCM"
                            WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}")

            file(COPY "${YCM_SOURCE_DIR}/downloads"
                 DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/external/YCM/")
        endif()
    endif()
else()
    message("Not a git repository")
endif()


if(NOT DEFINED YCM_TAG)
    set(YCM_TAG "v${YCM_VERSION_MAJOR}.${YCM_VERSION_MINOR}.${YCM_VERSION_PATCH}")
endif()

message("Using YCM_TAG = \"${YCM_TAG}\"")
message("Using YCM_STYLE = \"${YCM_STYLE}\"")
message("Using YCM_REPOSITORY = \"${YCM_REPOSITORY}\"")

set(YCM_BOOTSTRAP_VERBOSE TRUE)
set(NON_INTERACTIVE_BUILD TRUE)

if(YCM_BOOTSTRAP_TEST STREQUAL "BUILD_DIR")
    set(YCM_DIR ${YCM_BINARY_DIR} CACHE PATH "")
elseif(YCM_BOOTSTRAP_TEST STREQUAL "INSTALL_DIR")
    file(MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/build")
    execute_process(COMMAND ${CMAKE_COMMAND} ${YCM_SOURCE_DIR} -DCMAKE_INSTALL_PREFIX=${YCM_BINARY_DIR}/tests/CMakeFiles/TestPrefix
                    COMMAND ${CMAKE_COMMAND} --build ${CMAKE_CURRENT_BINARY_DIR}/build
                    COMMAND ${CMAKE_COMMAND} --build ${CMAKE_CURRENT_BINARY_DIR}/build --target install
                    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/build
                    OUTPUT_QUIET
                    ERROR_QUIET)
    list(APPEND CMAKE_PREFIX_PATH "${YCM_BINARY_DIR}/tests/CMakeFiles/TestPrefix")
elseif(YCM_BOOTSTRAP_TEST STREQUAL "NOT_USE_SYSTEM")
    set(YCM_DIR ${YCM_BINARY_DIR} CACHE PATH "")
    set(USE_SYSTEM_YCM FALSE)
elseif(YCM_BOOTSTRAP_TEST STREQUAL "DISABLE_FIND")
    set(YCM_DIR ${YCM_BINARY_DIR} CACHE PATH "")
    set(CMAKE_DISABLE_FIND_PACKAGE_YCM TRUE)
endif()

include(YCMBootstrap)

feature_summary(WHAT ALL INCLUDE_QUIET_PACKAGES)
