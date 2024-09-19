cmake_minimum_required(VERSION 3.12)
project(inekf)

find_package(ament_cmake REQUIRED)
find_package (Threads)
find_package(Eigen3 REQUIRED)

option(USE_CPP11 "Use flag -std=c++11" ON)
option(USE_MUTEX "Use mutex in code" OFF)

message("USE_CPP11: " ${USE_CPP11})
message("USE_MUTEX: " ${USE_MUTEX})

if (USE_MUTEX AND NOT USE_CPP11)
    message(FATAL_ERROR "You cannot have USE_CPP11 = OFF AND USE_MUTEX = ON")
endif (USE_MUTEX AND NOT USE_CPP11)

# Set compiler flags
if (USE_CPP11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
endif(USE_CPP11)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -DEIGEN_NO_DEBUG -march=native -Wl,--no-as-needed")
if (USE_MUTEX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DINEKF_USE_MUTEX=true")
else (USE_MUTEX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DINEKF_USE_MUTEX=false")
endif(USE_MUTEX)
SET(CMAKE_CXX_COMPILER /usr/bin/g++)

message("CMAKE_CXX_FLAGS: " ${CMAKE_CXX_FLAGS})
unset(USE_CPP11)
unset(USE_MUTEX)

include_directories(include ${EIGEN3_INCLUDE_DIRS})

add_library(inekf SHARED
    src/InEKF.cpp
    src/RobotState.cpp
    src/LieGroup.cpp
    src/NoiseParams.cpp
)

# install
install(
  TARGETS ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include/${PROJECT_NAME}
)

install(DIRECTORY include/ DESTINATION include/${PROJECT_NAME})

# ament_export_dependencies(${dependencies})
ament_export_include_directories("include/${PROJECT_NAME}")
ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_package()