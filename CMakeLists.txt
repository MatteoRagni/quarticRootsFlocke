# Copyright (c) 2020, Matteo Ragni
# All rights reserved.
# 
#   Based on the work of Enrico Bertolazzi
#   http://ebertolazzi.github.io/Clothoids/
# 
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are
# met:
# 
#  * Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
#  * Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in
#    the documentation and/or other materials provided with the distribution
# 
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE
# LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
# SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
# CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.

cmake_minimum_required(VERSION 3.14)

## OPTIONS
option(POLYNOMIAL_ROOTS_BUILD_SHARED 
  "Enables shared library build, defaults to BUILD_SHARED_LIBS" 
  ${BUILD_SHARED_LIBS})

set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_VERBOSE_MAKEFILE HIGH)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
if (NOT APPLE)
  set(CMAKE_INSTALL_RPATH $ORIGIN)
endif()

if(NOT APPLE)
  set(CMAKE_INSTALL_RPATH $ORIGIN)
endif()

file(READ "./VERSION" POLYNOMIAL_ROOTS_VERSION)
project(
  PolynomialRoots
  VERSION ${POLYNOMIAL_ROOTS_VERSION}
  HOMEPAGE_URL "https://github.com/MatteoRagni/quarticRootsFlocke")

## SOURCES
set(POLYNOMIAL_ROOTS_SRCS
  PolynomialRoots-1-Quadratic.cc
  PolynomialRoots-2-Cubic.cc
  PolynomialRoots-3-Quartic.cc
  PolynomialRoots-Jenkins-Traub.cc
  PolynomialRoots-Utils.cc)

set(POLYNOMIAL_ROOTS_HDRS
  PolynomialRoots-Utils.hh
  PolynomialRoots.hh)

set(POLYNOMIAL_ROOTS_SRCS_BUILD)
foreach(s ${POLYNOMIAL_ROOTS_SRCS})
  set(POLYNOMIAL_ROOTS_SRCS_BUILD ${POLYNOMIAL_ROOTS_SRCS_BUILD} src/${s})
endforeach()

set(POLYNOMIAL_ROOTS_HDRS_BUILD)
set(POLYNOMIAL_ROOTS_HDRS_INSTALL)
set(POLYNOMIAL_ROOTS_HDRS_INSTALL_TARGET)
foreach(h ${POLYNOMIAL_ROOTS_HDRS})
  set(POLYNOMIAL_ROOTS_HDRS_BUILD ${POLYNOMIAL_ROOTS_HDRS_BUILD} 
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/${h}>")
  set(POLYNOMIAL_ROOTS_HDRS_INSTALL ${POLYNOMIAL_ROOTS_HDRS_INSTALL} 
    "$<INSTALL_INTERFACE:include/${PROJECT_NAME}/${h}>")
  set(POLYNOMIAL_ROOTS_HDRS_INSTALL_TARGET ${POLYNOMIAL_ROOTS_HDRS_INSTALL_TARGET} 
    "${CMAKE_CURRENT_SOURCE_DIR}/include/${h}")
endforeach()
set(POLYNOMIAL_ROOTS_HDRS_TARGET ${POLYNOMIAL_ROOTS_HDRS_BUILD} ${POLYNOMIAL_ROOTS_HDRS_INSTALL})

## TARGETS
add_library(PolynomialRootsStatic STATIC)
set(POLYNOMIAL_ROOTS_EXPORT_TARGETS "PolynomialRootsStatic")
target_sources(PolynomialRootsStatic
  PRIVATE "${POLYNOMIAL_ROOTS_SRCS_BUILD}"
  PUBLIC "${POLYNOMIAL_ROOTS_HDRS_TARGET}")
target_compile_definitions(PolynomialRootsStatic PUBLIC "_USE_MATH_DEFINES")
target_include_directories(PolynomialRootsStatic PUBLIC 
 "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
 "$<INSTALL_INTERFACE:include/PolynomialRoots>")
set_target_properties(PolynomialRootsStatic PROPERTIES
  POSITION_INDIPENDENT_CODE ON
  EXPORT_NAME PolynomialRoots_Targets)
add_library(PolynomialRoots::Static ALIAS PolynomialRootsStatic)

if(POLYNOMIAL_ROOTS_BUILD_SHARED)
  set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
  set(POLYNOMIAL_ROOTS_EXPORT_TARGETS ${POLYNOMIAL_ROOTS_EXPORT_TARGETS} "PolynomialRootsDynamic")
  add_library(PolynomialRootsDynamic SHARED)
  target_sources(PolynomialRootsDynamic
    PRIVATE "${POLYNOMIAL_ROOTS_SRCS_BUILD}"
    PUBLIC "${POLYNOMIAL_ROOTS_HDRS_TARGET}")
  target_compile_definitions(PolynomialRootsDynamic PUBLIC "_USE_MATH_DEFINES")
  target_include_directories(PolynomialRootsDynamic PUBLIC 
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
  "$<INSTALL_INTERFACE:include/PolynomialRoots>")
  set_target_properties(PolynomialRootsDynamic PROPERTIES
    POSITION_INDIPENDENT_CODE ON
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}
    EXPORT_NAME PolynomialRoots_Targets)
  add_library(PolynomialRoots::Dynamic ALIAS PolynomialRootsDynamic)
endif()

## Packaging
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  "${PROJECT_NAME}ConfigVersion.cmake"
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion)

set(PROJECT_INSTALL_CMAKEDIR
  "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}"
  CACHE STRING "Path To ${PROJECT_NAME} Cmake Files")

install(TARGETS ${POLYNOMIAL_ROOTS_EXPORT_TARGETS}
  EXPORT "${PROJECT_NAME}_Targets"
  RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
          COMPONENT "${PROJECT_NAME}_Install"
  LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
          COMPONENT "${PROJECT_NAME}_Install"
          NAMELINK_COMPONENT "${PROJECT_NAME}_Install"
  ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
          COMPONENT "${PROJECT_NAME}_Install"
  INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}")

install(FILES
  ${POLYNOMIAL_ROOTS_HDRS_INSTALL_TARGET}
  DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}"
  COMPONENT "${PROJECT_NAME}_Install")

# Packaging specific installs

export(EXPORT "${PROJECT_NAME}_Targets"
  FILE "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}/${PROJECT_NAME}Targets.cmake"
  NAMESPACE "${PROJECT_NAMESPACE}::")

configure_file("cmake/${PROJECT_NAME}Config.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
  @ONLY)

install(EXPORT "${PROJECT_NAME}_Targets"
  DESTINATION "${PROJECT_INSTALL_CMAKEDIR}"
  NAMESPACE "${PROJECT_NAMESPACE}::"
  FILE "${PROJECT_NAME}Targets.cmake"
  COMPONENT "${PROJECT_NAME}_Install")

install(FILES
  "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
  "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
  DESTINATION "${PROJECT_INSTALL_CMAKEDIR}"
  COMPONENT "${PROJECT_NAME}_Install")
