cmake_minimum_required(VERSION 3.4)

include(version.cmake)
include("standard/Standard.cmake")

# All of our binaries go to one place:  The binaries output directory.  We only want to tinker
# with this if we're building by ourselves, otherwise we just do whatever the enclosing project
# wants us to do.
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Libraries not only all wind up in the libraries directory, but we also keep them all together
# here by putting them in the same place, regardless of whether they are debug or release.  This
# makes globbing them together much easier.
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# Standard environment - Position independent code, Symbols hidden by default, and standard C++11
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_CXX_VISIBILITY_PRESET hidden)
set(CMAKE_VISIBILITY_INLINES_HIDDEN ON)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

project(LeapC++ VERSION ${LeapC++_VERSION})

set_property(GLOBAL PROPERTY USE_FOLDERS ON)
parse_version(LeapC++ ${LeapC++_VERSION})

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake-modules")

if(WIN32)
  if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(CMAKE_LIBRARY_ARCHITECTURE x64)
  else()
    set(CMAKE_LIBRARY_ARCHITECTURE x86)
  endif()
endif()

# Options
option(BUILD_TESTING "Build Tests" OFF)

if(BUILD_SWIG)
  find_package(PythonInterp 3)
  find_package(PythonLibs 3)
  if(NOT (DEFINED PYTHON_EXECUTABLE AND DEFINED PYTHON_INCLUDE_DIR AND DEFINED PYTHON_LIBRARY))
    message(FATAL_ERROR "Python 3 not found.")
  endif()
  find_package(JNI 8.0 REQUIRED)
  find_package(SWIG 3.0 REQUIRED)
endif()

find_package(LeapSDK 4.0.0 REQUIRED)

# Recurse through source directories
add_subdirectory(contrib)
add_subdirectory(src)

option(BUILD_SAMPLES "Build the samples" ON)
if(BUILD_SAMPLES)
  add_subdirectory(samples)
endif()

# Now we can generate the version and install stuff
generate_version()
combined_installer(
  VENDOR "Leap Motion"
  CONTACT "wgray@leapmotion.com"
)
