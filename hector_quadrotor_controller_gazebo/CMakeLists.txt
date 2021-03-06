cmake_minimum_required(VERSION 2.8.3)
project(hector_quadrotor_controller_gazebo)

find_package(catkin REQUIRED COMPONENTS
  gazebo_ros_control
  hector_quadrotor_interface
)
include_directories(include ${catkin_INCLUDE_DIRS})

find_package(gazebo REQUIRED)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${GAZEBO_CXX_FLAGS}")
link_directories(${GAZEBO_LIBRARY_DIRS})
include_directories(${GAZEBO_INCLUDE_DIRS})

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES hector_quadrotor_controller_gazebo
  CATKIN_DEPENDS gazebo_ros_control hector_quadrotor_interface
)
option(USE_MAV_MSGS "Publish actuator commands as mav_msgs/Actuators" ON)
if(USE_MAV_MSGS)
  find_package(mav_msgs REQUIRED)
  # For ubuntu trusty, utopic and vivid find Eigen3 headers
  # see http://wiki.ros.org/jade/Migration#Eigen_CMake_Module_in_cmake_modules
  # and https://github.com/ros/cmake_modules/issues/35
  find_package(Eigen3 QUIET)
  if(NOT EIGEN3_FOUND)
    # Fallback to cmake_modules
    find_package(cmake_modules REQUIRED)
    find_package(Eigen REQUIRED)
    set(EIGEN3_INCLUDE_DIRS ${EIGEN_INCLUDE_DIRS})
    set(EIGEN3_LIBRARIES ${EIGEN_LIBRARIES})  # Not strictly necessary as Eigen is head only
    # Possibly map additional variables to the EIGEN3_ prefix.
  else()
    set(EIGEN3_INCLUDE_DIRS ${EIGEN3_INCLUDE_DIR})
  endif()
  add_definitions(-DUSE_MAV_MSGS)
  include_directories(${mav_msgs_INCLUDE_DIRS} ${EIGEN3_INCLUDE_DIRS})
  list(APPEND catkin_LIBRARIES ${mav_msgs_LIBRARIES})
endif()

add_library(hector_quadrotor_controller_gazebo
  src/quadrotor_hardware_gazebo.cpp
)
target_link_libraries(hector_quadrotor_controller_gazebo
  ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES}
)

install(TARGETS hector_quadrotor_controller_gazebo
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)

install(FILES
  quadrotor_controller_gazebo.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
