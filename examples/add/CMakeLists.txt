cmake_minimum_required(VERSION 3.25...3.30)
project(mylib-add LANGUAGES CXX)

include("../../cmake/utils.cmake")

if(PROJECT_IS_TOP_LEVEL)
    find_package(mylib 1.0.0 REQUIRED)
endif()

set(sources main.cpp)
source_group(TREE "${CMAKE_CURRENT_SOURCE_DIR}" FILES ${sources})

add_executable(mylib-add)
target_sources(mylib-add PRIVATE ${sources})
target_link_libraries(mylib-add PRIVATE mylib::mylib)

if(NOT PROJECT_IS_TOP_LEVEL)
    win_copy_deps_to_target_dir(mylib-add mylib::mylib)
endif()
