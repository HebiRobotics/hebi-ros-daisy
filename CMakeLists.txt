cmake_minimum_required (VERSION 3.1)
project (hebi-ros-daisy)

SET (CMAKE_CXX_STANDARD 11)
SET (CMAKE_CXX_STANDARD_REQUIRED ON)
SET (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  roslib
  std_msgs
  sensor_msgs
  geometry_msgs
  hebi_cpp_api
)

## System dependencies are found with CMake's conventions
find_package(Boost REQUIRED COMPONENTS system)

catkin_package(
  CATKIN_DEPENDS roscpp
  CATKIN_DEPENDS message_runtime
  CATKIN_DEPENDS hebi_cpp_api
)

###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(
  src
  ${catkin_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
)

## Mark other files for installation (e.g. launch and bag files, etc.)
install(FILES
  launch/daisy.launch
  resources/gains18.xml
  resources/gains3.xml
  resources/hex_config.xml
  resources/left.hrdf
  resources/right.hrdf
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

## I don't think this is the _right_ way to do this, but it works...
file (COPY ${CMAKE_CURRENT_SOURCE_DIR}/resources/gains18.xml
      DESTINATION ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_BIN_DESTINATION})
file (COPY ${CMAKE_CURRENT_SOURCE_DIR}/resources/gains3.xml
      DESTINATION ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_BIN_DESTINATION})
file (COPY ${CMAKE_CURRENT_SOURCE_DIR}/resources/hex_config.xml
      DESTINATION ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_BIN_DESTINATION})
file (COPY ${CMAKE_CURRENT_SOURCE_DIR}/resources/left.hrdf
      DESTINATION ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_BIN_DESTINATION})
file (COPY ${CMAKE_CURRENT_SOURCE_DIR}/resources/right.hrdf
      DESTINATION ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_BIN_DESTINATION})

add_executable(hexapod_control src/hexapod_control.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/robot/leg.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/robot/hexapod.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/robot/step.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/robot/hexapod_parameters.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/xml_util/pugixml.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/xml_util/xml_helpers.cpp
)
add_dependencies(hexapod_control ${catkin_EXPORTED_TARGETS})

target_link_libraries(hexapod_control ${catkin_LIBRARIES})

add_executable(controller src/controller_node.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/input/mobile_io.cpp
)
add_dependencies(controller ${catkin_EXPORTED_TARGETS})

target_link_libraries(controller ${catkin_LIBRARIES})
