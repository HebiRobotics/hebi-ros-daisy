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
)

## System dependencies are found with CMake's conventions
find_package(Boost REQUIRED COMPONENTS system)

catkin_package(
  CATKIN_DEPENDS roscpp
  CATKIN_DEPENDS message_runtime
)

###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(
  src
  hebi-cpp
  hebi-cpp/src
  hebi-cpp/hebi/include
  hebi-cpp/Eigen
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
  # C API
  hebi-cpp/src/command.cpp
  hebi-cpp/src/feedback.cpp
  hebi-cpp/src/group_command.cpp
  hebi-cpp/src/group_feedback.cpp
  hebi-cpp/src/group_info.cpp
  hebi-cpp/src/group.cpp
  hebi-cpp/src/info.cpp
  hebi-cpp/src/robot_model.cpp
  hebi-cpp/src/log_file.cpp
  hebi-cpp/src/lookup.cpp
  hebi-cpp/src/mac_address.cpp
  hebi-cpp/src/trajectory.cpp

  # Hexapod core code
  ${CMAKE_CURRENT_SOURCE_DIR}/src/robot/leg.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/robot/hexapod.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/robot/step.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/robot/hexapod_parameters.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/xml_util/pugixml.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/xml_util/xml_helpers.cpp
)
add_dependencies(hexapod_control ${catkin_EXPORTED_TARGETS})

target_link_libraries(hexapod_control ${catkin_LIBRARIES} ${PROJECT_SOURCE_DIR}/hebi-cpp/hebi/lib/linux_x86_64/libhebi.so)
