#
# C-API for OpenCascade wraps OpenCascase library to the C\C++ compiler independent C library
# Copyright (C) 2018 Vitaly Kovalev
#
# This file is part of C-API for OpenCascade.
#
# This library is free software: you can redistribute it and/or modify
# it under the terms of the GNU Lesser General Public
# License as published by the Free Software Foundation, either
# version 2.1 of the License, or (at your option) any later version.
#
# This library is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public
# License along with this library.  If not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA
#

project(opencascade_capi)

cmake_minimum_required(VERSION 2.8)

find_package(PythonInterp 3.4 REQUIRED)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")

set(OPENCASCADE_INCLUDE_DIR "" CACHE PATH "Include directory for OpenCASCADE library")
set(OPENCASCADE_LIBRARIES_PATH_DEBUG "" CACHE PATH "Path to OpenCASCADE debug library")
set(OPENCASCADE_LIBRARIES_PATH_RELEASE "" CACHE PATH "Path to OpenCASCADE release library")

FILE(GLOB_RECURSE OPENCASCADE_LIBRARIES_DEBUG "${OPENCASCADE_LIBRARIES_PATH_DEBUG}/*.lib")
FILE(GLOB_RECURSE OPENCASCADE_LIBRARIES_RELEASE "${OPENCASCADE_LIBRARIES_PATH_RELEASE}/*.lib")

if(NOT OPENCASCADE_INCLUDE_DIR OR NOT (OPENCASCADE_LIBRARIES_DEBUG OR OPENCASCADE_LIBRARIES_RELEASE))
  message(FATAL_ERROR "OpenCASCADE library is not found. Please specify exact pathes to OpenCASCADE library.")
endif()

include_directories(
  ${opencascade_capi_SOURCE_DIR}/src/
  ${OPENCASCADE_INCLUDE_DIR}
)

add_library(opencascade_capi SHARED
  ${opencascade_capi_SOURCE_DIR}/src/AutoGenWrap.cpp
)

#target_link_libraries(opencascade_capi debug ${OPENCASCADE_LIBRARIES_DEBUG} optimized ${OPENCASCADE_LIBRARY_RELEASE})
target_link_libraries(opencascade_capi ${OPENCASCADE_LIBRARY_RELEASE})

find_file(BCAPI_SCRIPT Capi.py PATHS ${opencascade_capi_SOURCE_DIR}/src/beautiful-capi/source/ NO_DEFAULT_PATH)

if(NOT BCAPI_SCRIPT)
  message(FATAL_ERROR "Beautiful-capi component is not found. Please update git submodules.")
endif()

add_custom_command(
  OUTPUT
    ${opencascade_capi_SOURCE_DIR}/src/AutoGenWrap.cpp
  COMMAND
    ${PYTHON_EXECUTABLE}
    ${opencascade_capi_SOURCE_DIR}/src/beautiful-capi/source/Capi.py
    -i ${opencascade_capi_SOURCE_DIR}/src/opencascade.xml
    -p ${opencascade_capi_SOURCE_DIR}/src/opencascade_params.xml
    -o ${opencascade_capi_SOURCE_DIR}/include
    -s ${opencascade_capi_SOURCE_DIR}/src/snippets
    -w ${opencascade_capi_SOURCE_DIR}/src/AutoGenWrap.cpp
  MAIN_DEPENDENCY
    ${opencascade_capi_SOURCE_DIR}/src/opencascade.xml
  DEPENDS
    ${opencascade_capi_SOURCE_DIR}/src/opencascade_params.xml
  WORKING_DIRECTORY
    ${opencascade_capi_SOURCE_DIR}/src/beautiful-capi/source/
)
