cmake_minimum_required(VERSION 2.8 FATAL_ERROR)


project(ardupilot_sitl_gazebo)


#######################
## Find Dependencies ##
#######################

find_package(gazebo REQUIRED)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${GAZEBO_CXX_FLAGS}")

find_package(PkgConfig)
pkg_check_modules(GSTREAMER REQUIRED gstreamer-1.0)
pkg_check_modules(GST_APP REQUIRED gstreamer-app-1.0)

if("${GAZEBO_VERSION}" VERSION_LESS "7.0")
    message(FATAL_ERROR "You need at least Gazebo 7.0. Your version: ${GAZEBO_VERSION}")
else()
    message("Gazebo version: ${GAZEBO_VERSION}")
endif()

###########
## Build ##
###########

include_directories(
        ${PROJECT_SOURCE_DIR}
        include
        ${GAZEBO_INCLUDE_DIRS}
        )

link_libraries(
        ${CMAKE_CURRENT_BINARY_DIR}
        ${GAZEBO_LIBRARIES}
        )

link_directories(
        ${GAZEBO_LIBRARY_DIRS}
        )

set (plugins_single_header
        ArduPilotPlugin
        ArduCopterIRLockPlugin
        ArduCopterPlugin
        GimbalSmall2dPlugin
        )

add_library(ArduCopterIRLockPlugin SHARED src/ArduCopterIRLockPlugin.cc)
target_link_libraries(ArduCopterIRLockPlugin ${GAZEBO_LIBRARIES})

add_library(ArduPilotPlugin SHARED src/ArduPilotPlugin.cc)
target_link_libraries(ArduPilotPlugin ${GAZEBO_LIBRARIES})

add_library(ArduCopterPlugin SHARED src/ArduCopterPlugin.cc)
target_link_libraries(ArduCopterPlugin ${GAZEBO_LIBRARIES})

add_library(GimbalSmall2dPlugin SHARED src/GimbalSmall2dPlugin.cc)
target_link_libraries(GimbalSmall2dPlugin ${GAZEBO_LIBRARIES})

if (GSTREAMER_FOUND)
    message("GStreamer FOUND")
    include_directories(${GSTREAMER_INCLUDE_DIRS} ${GST_APP_INCLUDE_DIRS} ${OpenCV_INCLUDE_DIRS})
    add_library(gazebo_gst_camera_plugin SHARED src/gazebo_gst_camera_plugin.cpp)
    target_link_libraries(gazebo_gst_camera_plugin ${GAZEBO_LIBRARIES} ${GSTREAMER_LIBRARIES} ${GST_APP_LIBRARIES} ${OpenCV_LIBRARIES})
    install(TARGETS gazebo_gst_camera_plugin DESTINATION ${GAZEBO_PLUGIN_PATH})
endif()

install(TARGETS ArduCopterIRLockPlugin DESTINATION ${GAZEBO_PLUGIN_PATH})
install(TARGETS ArduPilotPlugin DESTINATION ${GAZEBO_PLUGIN_PATH})
install(TARGETS ArduCopterPlugin DESTINATION ${GAZEBO_PLUGIN_PATH})
install(TARGETS GimbalSmall2dPlugin DESTINATION ${GAZEBO_PLUGIN_PATH})
