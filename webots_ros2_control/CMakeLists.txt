cmake_minimum_required(VERSION 3.5)
project(webots_ros2_control)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Dependencies
find_package(ament_cmake REQUIRED)
find_package(hardware_interface REQUIRED)
find_package(controller_manager REQUIRED)
find_package(pluginlib REQUIRED)
find_package(rclcpp REQUIRED)
find_package(webots_ros2_driver REQUIRED)

list(GET webots_ros2_driver_INCLUDE_DIRS 0 webots_ros2_driver_INCLUDE)
include_directories(
  ${webots_ros2_driver_INCLUDE}/webots/cpp
)

# webots_ros2 Plugin
add_library(
  ${PROJECT_NAME}
  SHARED
  src/Ros2Control.cpp
)
target_include_directories(
  ${PROJECT_NAME}
  PRIVATE
  include
)
ament_target_dependencies(
  ${PROJECT_NAME}
  hardware_interface
  controller_manager
  pluginlib
  rclcpp
  webots_ros2_driver
)

# Prevent pluginlib from using boost
target_compile_definitions(${PROJECT_NAME} PUBLIC "PLUGINLIB__DISABLE_BOOST_FUNCTIONS")
pluginlib_export_plugin_description_file(webots_ros2_driver webots_ros2_control.xml)

# ros2_control System
add_library(
  ${PROJECT_NAME}_system
  SHARED
  src/Ros2ControlSystem.cpp
)
target_include_directories(
  ${PROJECT_NAME}_system
  PRIVATE
  include
)
ament_target_dependencies(
  ${PROJECT_NAME}_system
  hardware_interface
  pluginlib
  rclcpp
  webots_ros2_driver
)

# Prevent pluginlib from using boost
target_compile_definitions(${PROJECT_NAME}_system PUBLIC "PLUGINLIB__DISABLE_BOOST_FUNCTIONS")
pluginlib_export_plugin_description_file(${PROJECT_NAME} webots_ros2_control_system.xml)

# Common
install(TARGETS
  ${PROJECT_NAME}
  ${PROJECT_NAME}_system
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)
ament_export_include_directories(
  include
)

ament_export_dependencies(
  hardware_interface
  pluginlib
  rclcpp
)
ament_export_libraries(
  ${PROJECT_NAME}
  ${PROJECT_NAME}_system
)

ament_package()
