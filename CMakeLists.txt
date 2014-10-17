cmake_minimum_required(VERSION 2.8.3)
project(elas_ros)
find_package(catkin REQUIRED COMPONENTS roscpp std_msgs sensor_msgs nav_msgs message_filters image_transport cv_bridge image_geometry tf pcl_ros geometry_msgs message_generation)
find_package(OpenCV REQUIRED)
find_package(PCL REQUIRED)
find_package(Boost REQUIRED COMPONENTS signals)
find_package(OpenMP REQUIRED)


add_message_files(DIRECTORY msg FILES ElasFrameData.msg)

generate_messages(DEPENDENCIES std_msgs sensor_msgs)

catkin_package()

#include(${CMAKE_CURRENT_SOURCE_DIR}/libelas/CMakeLists.txt)

# use sse3 instruction set
SET(CMAKE_CXX_FLAGS "-msse3")

include_directories(include libelas/src ${catkin_INCLUDE_DIRS} ${OpenCV_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS})

if (OPENMP_FOUND)
  include_directories(${OpenMP_INCLUDE_DIRS})
  set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
  set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
  add_library(elas
            libelas/src_omp/descriptor.cpp
            libelas/src_omp/elas.cpp
            libelas/src_omp/filter.cpp
            libelas/src_omp/matrix.cpp
            libelas/src_omp/triangle.cpp)
  target_link_libraries(elas ${OpenMP_LIBRARIES})
elseif (OPENMP_NOTFOUND)
  add_library(elas
            libelas/src/descriptor.cpp
            libelas/src/elas.cpp
            libelas/src/filter.cpp
            libelas/src/matrix.cpp
            libelas/src/triangle.cpp)
endif()

add_executable(elas_ros src/elas.cpp)
add_dependencies(elas_ros elas_ros_msgs_gencpp)
target_link_libraries(elas_ros elas ${catkin_LIBRARIES} ${OpenCV_LIBRARIES})

add_executable(pc_construction src/pc_construction.cpp)
add_dependencies(pc_construction elas_ros_msgs_gencpp)
target_link_libraries(pc_construction ${catkin_LIBRARIES} ${pcl_LIBRARIES} ${Boost_LIBRARIES})
