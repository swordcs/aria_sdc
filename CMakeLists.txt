cmake_minimum_required(VERSION 3.2)
project(aria)

#set(CMAKE_BUILD_TYPE Debug)
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread -Wall -Wno-long-long -Wno-unused-variable -Wno-variadic-macros -pedantic")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread -Wall -Wno-long-long -Wno-unused-variable -Wno-variadic-macros -pedantic -O2")
set(CMAKE_CXX_STANDARD 14)

find_library(jemalloc_lib jemalloc) # jemalloc 5.0

# additional target to perform clang-format run, requires clang-format

# get all project files
file(GLOB_RECURSE ALL_SOURCE_FILES benchmark/*.h common/*.h core/*.h protocol/*.h bench*.cpp)

add_compile_definitions(GLOG_USE_GLOG_EXPORT) # 在这里添加，不需要在每个引用glog头文件的地方添加GLOG_USE_GLOG_EXPORT和报错检查关闭，也可以过编译——Yu

add_custom_target(
        format
        COMMAND clang-format
        -style=LLVM
        -i
        -sort-includes
        ${ALL_SOURCE_FILES}
)

include_directories(${CMAKE_SOURCE_DIR})

include_directories(${CMAKE_SOURCE_DIR}/SDCs_injection)

file(GLOB_RECURSE COMMON_SOURCE_FILES common/*.cpp)
add_library(common STATIC ${COMMON_SOURCE_FILES})

if(APPLE)
    find_package(glog REQUIRED)
    find_package(gflags REQUIRED)
    target_link_libraries(common ${jemalloc_lib} glog::glog gflags)
else()
    target_link_libraries(common ${jemalloc_lib} glog gflags)
endif()

add_executable(bench_tpcc bench_tpcc.cpp)
target_link_libraries(bench_tpcc common)

add_executable(bench_ycsb bench_ycsb.cpp)
target_link_libraries(bench_ycsb common)