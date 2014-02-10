cmake_minimum_required(VERSION 2.8.3)

project(sba)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

find_package(catkin REQUIRED COMPONENTS
    bpcg
    roscpp
    message_generation
    visualization_msgs
)
find_package(Boost REQUIRED COMPONENTS thread)
find_package(Eigen REQUIRED)
find_package(SuiteSparse REQUIRED)
find_package(LAPACK REQUIRED)
include_directories(
    include
    ${Eigen_INCLUDE_DIRS}
    ${Boost_INCLUDE_DIRS}
    ${SuiteSparse_INCLUDE_DIRS}
    ${LAPACK_INCLUDE_DIRS}
    ${catkin_INCLUDE_DIRS}
)

#add_definitions(-Wall -Wno-missing-field-initializers)
#add_definitions(-DSBA_CHOLMOD -DSBA_DSIF)
add_definitions(-DSBA_CHOLMOD)

add_message_files(FILES
    CameraNode.msg
    Frame.msg
    Projection.msg
    WorldPoint.msg
)
generate_messages(DEPENDENCIES visualization_msgs)

catkin_package(
    LIBRARIES sba
    INCLUDE_DIRS include
    DEPENDS Eigen SuiteSparse
    CATKIN_DEPENDS bpcg
    CFG_EXTRAS sba-extra.cmake
)

add_library(sba
    src/sba.cpp src/spa.cpp src/spa2d.cpp src/csparse.cpp src/proj.cpp src/node.cpp src/sba_file_io.cpp
)
target_link_libraries(sba blas lapack cholmod cxsparse)

# SBA library with ROS & utilities, including reading from file and visualization.
add_library(sba_vis src/visualization.cpp)
target_link_libraries(sba_vis sba ${catkin_LIBRARIES})

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h")

install(TARGETS sba DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})
