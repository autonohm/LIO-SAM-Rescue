cmake_minimum_required(VERSION 3.16)
project(lio_sam)

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE Release)
endif()

find_package(ament_cmake REQUIRED)
find_package(rosidl_default_generators REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(visualization_msgs REQUIRED)
find_package(pcl_conversions REQUIRED)
find_package(pcl_msgs REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_geometry_msgs REQUIRED)
find_package(tf2_sensor_msgs REQUIRED)
find_package(tf2_eigen REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(OpenCV REQUIRED)
find_package(jsoncpp CONFIG REQUIRED)
find_package(PCL REQUIRED)
find_package(GTSAM REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(OpenMP REQUIRED)
find_package(pointcloud_coregistration_msgs REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/CloudInfo.msg"
  "srv/SaveMap.srv"
  DEPENDENCIES std_msgs sensor_msgs
)
rosidl_get_typesupport_target(cpp_typesupport_target ${PROJECT_NAME} "rosidl_typesupport_cpp")

set(ROS_DEPENDENCY_TARGETS
  rclcpp::rclcpp
  std_msgs::std_msgs
  sensor_msgs::sensor_msgs
  geometry_msgs::geometry_msgs
  nav_msgs::nav_msgs
  visualization_msgs::visualization_msgs
  pcl_conversions::pcl_conversions
  pcl_msgs::pcl_msgs
  tf2::tf2
  tf2_ros::tf2_ros
  tf2_eigen::tf2_eigen
  tf2_sensor_msgs::tf2_sensor_msgs
  tf2_geometry_msgs::tf2_geometry_msgs
  pointcloud_coregistration_msgs::pointcloud_coregistration_msgs
)

set(THIRD_PARTY_INCLUDE_DIRS ${PCL_INCLUDE_DIRS} ${OpenCV_INCLUDE_DIRS})
set(THIRD_PARTY_LIBRARIES ${PCL_LIBRARIES} ${OpenCV_LIBS} gtsam Eigen3::Eigen)

function(configure_lio_sam_target target)
  target_include_directories(${target} PRIVATE include/lio_sam ${THIRD_PARTY_INCLUDE_DIRS})
  target_link_libraries(${target} PRIVATE
    ${ROS_DEPENDENCY_TARGETS}
    ${THIRD_PARTY_LIBRARIES}
    "${cpp_typesupport_target}"
  )
endfunction()

add_executable(${PROJECT_NAME}_featureExtraction src/featureExtraction.cpp)
configure_lio_sam_target(${PROJECT_NAME}_featureExtraction)

add_executable(${PROJECT_NAME}_imageProjection src/imageProjection.cpp)
configure_lio_sam_target(${PROJECT_NAME}_imageProjection)

add_executable(${PROJECT_NAME}_imuPreintegration src/imuPreintegration.cpp)
configure_lio_sam_target(${PROJECT_NAME}_imuPreintegration)


add_executable(${PROJECT_NAME}_mapOptimization src/mapOptmization.cpp)
configure_lio_sam_target(${PROJECT_NAME}_mapOptimization)
target_link_libraries(${PROJECT_NAME}_mapOptimization PRIVATE OpenMP::OpenMP_CXX)

install(DIRECTORY launch DESTINATION share/${PROJECT_NAME})
install(DIRECTORY config DESTINATION share/${PROJECT_NAME})
install(TARGETS
  ${PROJECT_NAME}_featureExtraction
  ${PROJECT_NAME}_imageProjection
  ${PROJECT_NAME}_imuPreintegration
  ${PROJECT_NAME}_mapOptimization
  DESTINATION lib/${PROJECT_NAME}
)
install(DIRECTORY include/ DESTINATION include)

ament_export_include_directories(include)
ament_export_dependencies(rosidl_default_runtime)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
