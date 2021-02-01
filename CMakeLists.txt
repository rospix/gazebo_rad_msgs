cmake_minimum_required(VERSION 2.8.3)
project(gazebo_rad_msgs)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -fno-diagnostics-color")

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
  message_generation
  roscpp
  std_msgs
  std_srvs
  sensor_msgs
  geometry_msgs
  cmake_modules
  )

# For Gazebo msgs
find_package(Protobuf REQUIRED)

set(PROTOBUF_IMPORT_DIRS)
foreach(ITR ${GAZEBO_INCLUDE_DIRS})
  if(ITR MATCHES ".*gazebo-[0-9.]+$")
    set(PROTOBUF_IMPORT_DIRS "${ITR}/gazebo/msgs/proto")
  endif()
endforeach()

set(msgs
  gazebo_msgs/RadiationObstacle.proto
  gazebo_msgs/RadiationSource.proto
)

# set(msgs
#   gazebo_msgs/RadiationObstacle.proto
# )
################################################
## Declare ROS messages, services and actions ##
################################################

add_message_files(DIRECTORY ros_msgs/radiation_source FILES

  RadiationSource.msg
  DebugSetActivity.msg
  DebugSetMaterial.msg
  )

add_message_files(DIRECTORY ros_msgs/radiation_obstacle FILES

  RadiationObstacle.msg
  )

add_message_files(DIRECTORY ros_msgs/plotting FILES

  Ellipse.msg
  )

add_message_files(DIRECTORY ros_msgs/timepix FILES

  Timepix.msg
  )

add_message_files(DIRECTORY ros_msgs/diagnostics FILES

  TimepixDiagnostics.msg
  )

add_message_files(DIRECTORY ros_msgs/diagnostics FILES

  SourceDiagnostics.msg
  )

add_message_files(DIRECTORY ros_msgs/diagnostics FILES

  ObstacleDiagnostics.msg
  )

## Generate added messages and services with any dependencies listed here
generate_messages(DEPENDENCIES

  std_msgs
  geometry_msgs
  sensor_msgs
  )

###################################
## catkin specific configuration ##
###################################

catkin_package(
  CATKIN_DEPENDS roscpp std_msgs geometry_msgs message_runtime sensor_msgs gazebo_ros
  LIBRARIES gazebo_rad_msgs
  )

###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations
# include_directories(include)
include_directories(
  ${catkin_INCLUDE_DIRS}
  ${GAZEBO_INCLUDE_DIRS}
  )

link_directories(${GAZEBO_LIBRARY_DIRS})

PROTOBUF_GENERATE_CPP(PROTO_SRCS PROTO_HDRS ${msgs})

add_library(gazebo_rad_msgs SHARED ${PROTO_SRCS} ${PROTO_HDRS})

target_link_libraries(gazebo_rad_msgs
  ${catkin_LIBRARIES}
  ${GAZEBO_LIBRARIES}
  ${PROTOBUF_LIBRARY}
  )

add_dependencies(gazebo_rad_msgs
  ${${PROJECT_NAME}_EXPORTED_TARGETS}
  ${catkin_EXPORTED_TARGETS}
  )

# message(${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_INCLUDE_DESTINATION})

# # DOES NOT WORK
# install(FILES ${PROTO_HDRS}
#         DESTINATION ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_INCLUDE_DESTINATION})

# compy the proto headers manually, since the install() does not work?!
foreach(HDR_MSG ${PROTO_HDRS})
  add_custom_command(TARGET gazebo_rad_msgs
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
    COMMAND bash -c "cp ${HDR_MSG} ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_INCLUDE_DESTINATION}"
    )
endforeach()

add_custom_command(TARGET gazebo_rad_msgs
  PRE_BUILD
  WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
  COMMAND bash -c "${CMAKE_CURRENT_SOURCE_DIR}/link_msg_files.sh"
  )
