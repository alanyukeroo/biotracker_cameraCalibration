cmake_minimum_required(VERSION 2.8.7 FATAL_ERROR)
project(cameracalibration)

#------------------------------------------------------------------------------
# Required CPM Setup - no need to modify - See: https://github.com/iauns/cpm
#------------------------------------------------------------------------------

set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm_packages" CACHE TYPE STRING)
find_package(Git)
if(NOT GIT_FOUND)
  message(FATAL_ERROR "CPM requires Git.")
endif()
if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
  message(STATUS "Cloning repo (https://github.com/iauns/cpm)")
  execute_process(
    COMMAND "${GIT_EXECUTABLE}" clone https://github.com/iauns/cpm ${CPM_DIR}
    RESULT_VARIABLE error_code
    OUTPUT_QUIET ERROR_QUIET)
  if(error_code)
    message(FATAL_ERROR "CPM failed to get the hash for HEAD")
  endif()
endif()
include(${CPM_DIR}/CPM.cmake)

#------------------------------------------------------------------------------
# CPM Modules
#------------------------------------------------------------------------------

if(NOT DEFINED CMAKECONFIG_PATH)
    CPM_AddModule("cmakeconfig"
        GIT_REPOSITORY "https://github.com/BioroboticsLab/cmakeconfig.git"
        GIT_TAG "master")
else()
    CPM_AddModule("cmakeconfig"
        SOURCE_DIR "${CMAKECONFIG_PATH}")
endif()

include_biotracker_core("master")

CPM_Finish()

biorobotics_config()

find_package(OpenCV REQUIRED)
find_package(Qt5Widgets REQUIRED)
find_package(Qt5OpenGL REQUIRED)

include_directories(
    ${PROJECT_SOURCE_DIR}
    SYSTEM ${Qt5Widgets_INCLUDE_DIRS}
    SYSTEM ${Qt5OpenGL_INCLUDE_DIRS}
)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)

add_definitions(${Qt5Widgets_DEFINITIONS})
add_definitions(-DQT_NO_KEYWORDS)

add_library(cameracalibration.tracker SHARED
    CameraCalib.cpp
)

target_link_libraries(cameracalibration.tracker ${OpenCV_LIBS})
