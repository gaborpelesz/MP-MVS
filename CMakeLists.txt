cmake_minimum_required (VERSION 3.22)
project(MPMVS)

set(PROJECT_PATH ${CMAKE_CURRENT_SOURCE_DIR}) #for genenrate config.h file
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/include/ProjectPath.h.in ${CMAKE_CURRENT_SOURCE_DIR}/include/ProjectPath.h)

include(CheckLanguage)
check_language(CUDA)
if (CMAKE_CUDA_COMPILER)
    enable_language(CUDA)
    message(" -- CUDA FOUND")
    set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -O3 --use_fast_math --maxrregcount=128 --ptxas-options=-v -std=c++11 --compiler-options -Wall -gencode arch=compute_80,code=sm_80")
    # set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -lineinfo") # for CUDA profiling!
else()
    message(FATAL_ERROR "-- CUDA NOT FOUND: Fatal error")
endif()

find_package(Threads REQUIRED)
find_package(OpenCV REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(OpenMP REQUIRED)
if (OPENMP_FOUND)
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

if(CMAKE_COMPILER_IS_GNUCXX)
    add_definitions(-std=c++11)
    add_definitions(-pthread)
    add_definitions(-Wall)
    add_definitions(-Wextra)
    add_definitions(-pedantic)
    add_definitions(-Wno-unused-function)
    add_definitions(-Wno-switch)
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O3 -ffast-math -march=native") # extend release-profile with fast-math
endif()

include_directories(
        ${EIGEN3_INCLUDE_DIRS}
        ${OpenCV_INCLUDE_DIRS}
        ./include
)

#Set ncnn install path， please set ncnn path
set(ncnn_DIR "/home/xuan/ncnn/build/install/lib/cmake/ncnn")
option(USE_NCNN "Build NCNN to Sky region detect" OFF)
include_directories(
        ./SkySegment
        ./SkySegment/include
)
if(USE_NCNN)
    add_definitions(-DBUILD_NCNN)
    find_package(ncnn REQUIRED)
    add_subdirectory(./SkySegment)
endif ()

cuda_add_executable(
    MPMVS
    src/main.cpp
    src/PatchMatch.cpp
    src/PatchMatch.cu
    src/utility.cpp
    )

target_link_libraries(
    MPMVS
    ${OpenCV_LIBS}
    OpenMP::OpenMP_CXX
    )

if(USE_NCNN)
    target_link_libraries(MPMVS SkySegment)
endif ()

#add_executable(DepthmapViewer
#               src/DepthmapViewer.cpp
#              )
#target_link_libraries(DepthmapViewer
#                      ${OpenCV_LIBS}
#                      )
