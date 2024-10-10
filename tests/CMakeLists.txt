cmake_minimum_required(VERSION 3.25...3.30)
project(mylib-tests)

#----------------------------------------------------------------------------------------------------------------------
# general settings and options
#----------------------------------------------------------------------------------------------------------------------

include("../cmake/utils.cmake")
string(COMPARE EQUAL "${CMAKE_SOURCE_DIR}" "${CMAKE_CURRENT_SOURCE_DIR}" is_top_level)

if(is_top_level)
    enable_testing()

    # Neither of these two are technically needed, but they make the expectation clear
    set_if_undefined(CMAKE_CXX_STANDARD 20)
    set_if_undefined(CMAKE_CXX_EXTENSIONS FALSE)
endif()

#----------------------------------------------------------------------------------------------------------------------
# testing framework
#----------------------------------------------------------------------------------------------------------------------

include(FetchContent)
FetchContent_Declare(
    googletest URL https://github.com/google/googletest/archive/refs/tags/v1.15.2.tar.gz DOWNLOAD_EXTRACT_TIMESTAMP ON
)

set(gtest_force_shared_crt ON CACHE BOOL "" FORCE) # do not override parent project's runtime settings on Windows
set(INSTALL_GTEST OFF)

# For simplicity, always build googletest as static library. This prevents mylib-tests executable from
# complaining about missing googletest DLLs on Windows.
set(BUILD_SHARED_LIBS OFF)

FetchContent_MakeAvailable(googletest)

#----------------------------------------------------------------------------------------------------------------------
# tests dependencies
#----------------------------------------------------------------------------------------------------------------------

if(is_top_level)
    find_package(mylib REQUIRED)
endif()

#----------------------------------------------------------------------------------------------------------------------
# tests sources
#----------------------------------------------------------------------------------------------------------------------

set(sources add_test.cpp)
source_group(TREE "${CMAKE_CURRENT_SOURCE_DIR}" FILES ${sources})

#----------------------------------------------------------------------------------------------------------------------
# tests target
#----------------------------------------------------------------------------------------------------------------------

add_executable(mylib-tests)
target_sources(mylib-tests PRIVATE ${sources})

target_link_libraries(mylib-tests PRIVATE mylib::mylib gtest_main)

if(NOT is_top_level)
    win_copy_deps_to_target_dir(mylib-tests mylib::mylib)
endif()

include(GoogleTest)
gtest_discover_tests(mylib-tests)
