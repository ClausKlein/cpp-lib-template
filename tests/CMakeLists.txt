cmake_minimum_required(VERSION 3.25...3.30)
project(mylib-tests LANGUAGES CXX)

#----------------------------------------------------------------------------------------------------------------------
# general settings and options
#----------------------------------------------------------------------------------------------------------------------

include("../cmake/utils.cmake")

if(PROJECT_IS_TOP_LEVEL)
    enable_testing()

    # Neither of these two are technically needed, but they make the expectation clear
    set_if_undefined(CMAKE_CXX_STANDARD 17)
    set_if_undefined(CMAKE_CXX_EXTENSIONS FALSE)
endif()

#----------------------------------------------------------------------------------------------------------------------
# testing framework
#----------------------------------------------------------------------------------------------------------------------

include(FetchContent)
FetchContent_Declare(
    googletest URL https://github.com/google/googletest/archive/refs/tags/v1.15.2.tar.gz DOWNLOAD_EXTRACT_TIMESTAMP ON
    DOWNLOAD_NO_PROGRESS ON FIND_PACKAGE_ARGS NAMES GTest
)

set(gtest_force_shared_crt ON CACHE BOOL "" FORCE) # do not override parent project's runtime settings on Windows
set(INSTALL_GTEST OFF)
set(BUILD_GMOCK OFF)

FetchContent_MakeAvailable(googletest)

#----------------------------------------------------------------------------------------------------------------------
# tests dependencies
#----------------------------------------------------------------------------------------------------------------------

if(PROJECT_IS_TOP_LEVEL)
    find_package(mylib 1.0.0 QUIET)
    if(NOT mylib_FOUND)
        message(STATUS "find_package(mylib) was NOT found, use as subproject ...")
        # test if the targets are usable if used as subproject
        add_subdirectory(.. mylib EXCLUDE_FROM_ALL)
    endif()
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

target_link_libraries(mylib-tests PRIVATE mylib::mylib GTest::gtest_main)

if(NOT PROJECT_IS_TOP_LEVEL)
    # test if the targets are findable from the build directory
    # cmake-format: off
    add_test(find-package-test
        ${CMAKE_CTEST_COMMAND}
        -C ${CMAKE_BUILD_TYPE}
        --build-and-test
            "${CMAKE_CURRENT_SOURCE_DIR}"
            "${CMAKE_CURRENT_BINARY_DIR}/find-package-test"
        --build-generator ${CMAKE_GENERATOR}
        --build-makeprogram ${CMAKE_MAKE_PROGRAM}
        --build-options
            "-DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}"
            "-DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}"
            "-DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}"
    )
    # cmake-format: on

    win_copy_deps_to_target_dir(mylib-tests mylib::mylib)
endif()

include(GoogleTest)
gtest_discover_tests(mylib-tests)
