cmake_minimum_required(VERSION 3.16)
project(imu_sensor_broadcaster)

if(CMAKE_CXX_COMPILER_ID MATCHES "(GNU|Clang)")
  add_compile_options(-Wall -Wextra -Wpedantic -Werror=conversion -Werror=unused-but-set-variable
                      -Werror=return-type -Werror=shadow -Werror=format -Werror=range-loop-construct
                      -Werror=missing-braces)
endif()

# using this instead of visibility macros
# S1 from https://github.com/ros-controls/ros2_controllers/issues/1053
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

set(THIS_PACKAGE_INCLUDE_DEPENDS
  controller_interface
  generate_parameter_library
  hardware_interface
  pluginlib
  rclcpp
  rclcpp_lifecycle
  realtime_tools
  sensor_msgs
)

find_package(ament_cmake REQUIRED)
find_package(backward_ros REQUIRED)
foreach(Dependency IN ITEMS ${THIS_PACKAGE_INCLUDE_DEPENDS})
  find_package(${Dependency} REQUIRED)
endforeach()

generate_parameter_library(imu_sensor_broadcaster_parameters
  src/imu_sensor_broadcaster_parameters.yaml
)

add_library(imu_sensor_broadcaster SHARED
  src/imu_sensor_broadcaster.cpp
)
target_compile_features(imu_sensor_broadcaster PUBLIC cxx_std_17)
target_include_directories(imu_sensor_broadcaster PUBLIC
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include/imu_sensor_broadcaster>
)
target_link_libraries(imu_sensor_broadcaster PUBLIC
  imu_sensor_broadcaster_parameters
)
ament_target_dependencies(imu_sensor_broadcaster PUBLIC ${THIS_PACKAGE_INCLUDE_DEPENDS})

pluginlib_export_plugin_description_file(
  controller_interface imu_sensor_broadcaster.xml)

if(BUILD_TESTING)
  find_package(ament_cmake_gmock REQUIRED)
  find_package(controller_manager REQUIRED)
  find_package(hardware_interface REQUIRED)
  find_package(ros2_control_test_assets REQUIRED)

  add_definitions(-DTEST_FILES_DIRECTORY="${CMAKE_CURRENT_SOURCE_DIR}/test")
  ament_add_gmock(test_load_imu_sensor_broadcaster test/test_load_imu_sensor_broadcaster.cpp)
  target_include_directories(test_load_imu_sensor_broadcaster PRIVATE include)
  target_link_libraries(test_load_imu_sensor_broadcaster
    imu_sensor_broadcaster
  )
  ament_target_dependencies(test_load_imu_sensor_broadcaster
    controller_manager
    hardware_interface
    ros2_control_test_assets
  )

  add_rostest_with_parameters_gmock(test_imu_sensor_broadcaster
    test/test_imu_sensor_broadcaster.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/test/imu_sensor_broadcaster_params.yaml)
  target_include_directories(test_imu_sensor_broadcaster PRIVATE include)
  target_link_libraries(test_imu_sensor_broadcaster
    imu_sensor_broadcaster
  )
  ament_target_dependencies(test_imu_sensor_broadcaster
    hardware_interface
  )
endif()

install(
  DIRECTORY include/
  DESTINATION include/imu_sensor_broadcaster
)
install(
  TARGETS
    imu_sensor_broadcaster
    imu_sensor_broadcaster_parameters
  EXPORT export_imu_sensor_broadcaster
  RUNTIME DESTINATION bin
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  INCLUDES DESTINATION include
)

ament_export_targets(export_imu_sensor_broadcaster HAS_LIBRARY_TARGET)
ament_export_dependencies(${THIS_PACKAGE_INCLUDE_DEPENDS})
ament_package()
