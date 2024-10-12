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
    DOWNLOAD_NO_PROGRESS ON FIND_PACKAGE_ARGS NAMES GTest
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
    find_package(mylib QUIET)
    if(NOT mylib_FOUND)
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

if(NOT is_top_level)
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
