cmake_minimum_required(VERSION 3.5)
project(limoncello)

if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING
      "Build type (Release or Debug)" FORCE)
endif()

# Require C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=native -fexceptions -O3")

# Other dependencies
find_package(Boost REQUIRED COMPONENTS system)
find_package(TBB REQUIRED)
find_package(Eigen3 REQUIRED)

if(TARGET TBB::tbb)
  set(LIMONCELLO_TBB_TARGET TBB::tbb)
elseif(DEFINED TBB_IMPORTED_TARGETS AND TBB_IMPORTED_TARGETS)
  set(LIMONCELLO_TBB_TARGET ${TBB_IMPORTED_TARGETS})
elseif(DEFINED TBB_LIBRARIES AND TBB_LIBRARIES)
  set(LIMONCELLO_TBB_TARGET ${TBB_LIBRARIES})
endif()

# ROS2 dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(pcl_conversions REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(tf2_eigen REQUIRED)

# Livox support
find_package(livox_interfaces QUIET)
find_package(livox_ros_driver QUIET)
find_package(livox_ros_driver2 QUIET)

# Header-only manif library (vendored in external/manif)
add_library(manif INTERFACE)
target_include_directories(manif INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/external/manif/include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/external/manif/external/tl>
  $<INSTALL_INTERFACE:include>
)

include_directories(
  include
)

add_executable(${PROJECT_NAME} src/main.cpp)

set(LIMONCELLO_ROS_DEPS
  rclcpp
  pcl_conversions
  sensor_msgs
  nav_msgs
  tf2
  tf2_ros
  tf2_eigen
)

if(livox_interfaces_FOUND)
  list(APPEND LIMONCELLO_ROS_DEPS livox_interfaces)
  target_compile_definitions(${PROJECT_NAME} PUBLIC LIMONCELLO_HAVE_LIVOX_INTERFACES)
endif()

if(livox_ros_driver_FOUND)
  list(APPEND LIMONCELLO_ROS_DEPS livox_ros_driver)
  target_compile_definitions(${PROJECT_NAME} PUBLIC LIMONCELLO_HAVE_LIVOX_ROS_DRIVER)
endif()

if(livox_ros_driver2_FOUND)
  list(APPEND LIMONCELLO_ROS_DEPS livox_ros_driver2)
  target_compile_definitions(${PROJECT_NAME} PUBLIC LIMONCELLO_HAVE_LIVOX_ROS_DRIVER2)
endif()

ament_target_dependencies(${PROJECT_NAME} PUBLIC ${LIMONCELLO_ROS_DEPS})

target_link_libraries(${PROJECT_NAME}
  PUBLIC
    ${Boost_LIBRARIES}
    ${LIMONCELLO_TBB_TARGET}
    manif
)

install(TARGETS
  ${PROJECT_NAME}
  DESTINATION lib/${PROJECT_NAME}
)

install(DIRECTORY
  launch
  config
  DESTINATION share/${PROJECT_NAME}
)

install(
  DIRECTORY external/manif/include/
  DESTINATION include
)

ament_package()
