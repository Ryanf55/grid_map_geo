cmake_minimum_required(VERSION 3.14.4)
project(grid_map_geo)

if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

# Set policy for 3.16 for CMP0076 defaulting to ON
cmake_policy(VERSION 3.16)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU" OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(grid_map_msgs REQUIRED)
find_package(grid_map_ros REQUIRED)
find_package(grid_map_core REQUIRED)
find_package(rclcpp REQUIRED)

find_package(Eigen3 REQUIRED)
find_package(GDAL 3 REQUIRED)


# Reverse compatability for GDAL<3.5
# https://gdal.org/development/cmake.html
if(NOT TARGET GDAL::GDAL)
  add_library(GDAL::GDAL ALIAS ${GDAL_LIBRARY})
endif()

# Build
add_library(${PROJECT_NAME}
  src/grid_map_geo.cpp
)

target_include_directories(${PROJECT_NAME}
  PUBLIC
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
  "$<INSTALL_INTERFACE:include/${PROJECT_NAME}>"
  ${GDAL_INCLUDE_DIR}
)

target_link_libraries(${PROJECT_NAME} Eigen3::Eigen GDAL::GDAL)

ament_target_dependencies(${PROJECT_NAME} SYSTEM
  grid_map_core
  grid_map_ros
)

add_executable(test_tif_loader
  src/test_tif_loader.cpp
)

target_include_directories(test_tif_loader
  PRIVATE
  include
  ${GDAL_INCLUDE_DIR}
)

target_link_libraries(test_tif_loader
  ${PROJECT_NAME}
)

# Export
ament_export_targets(${PROJECT_NAME}Targets HAS_LIBRARY_TARGET)
ament_export_dependencies(GDAL)

# Install
install(
  TARGETS
  ${PROJECT_NAME}
  test_tif_loader
  EXPORT ${PROJECT_NAME}Targets
  ARCHIVE DESTINATION lib/${PROJECT_NAME}
  LIBRARY DESTINATION lib/${PROJECT_NAME}
  RUNTIME DESTINATION lib/${PROJECT_NAME}
)

install(
  DIRECTORY include/${PROJECT_NAME}/
  DESTINATION include/${PROJECT_NAME}
  FILES_MATCHING PATTERN "*.hpp"
)

install(DIRECTORY
  launch
  DESTINATION share/${PROJECT_NAME}/
)

# Test
include(CTest)
if(BUILD_TESTING)
  add_subdirectory(test)
endif()

ament_package()
