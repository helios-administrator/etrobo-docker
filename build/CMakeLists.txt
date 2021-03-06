cmake_minimum_required(VERSION 2.8.7)
set(PROJECT_NAME_STR YAMIJI)
project(${PROJECT_NAME_STR} C CXX)

find_package(GTest REQUIRED)
find_package(Threads REQUIRED)

# make sure that we have support for C++11
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fprofile-arcs -ftest-coverage -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fprofile-arcs -ftest-coverage -std=c++0x")
else()
        message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

if(CMAKE_COMPILER_IS_GNUCXX)
    add_definitions(-Wall -ansi -Wno-deprecated -pthread -std=c++11)
endif()

if(MSVC)
    #vc 2012 fix for vararg templates
    set(MSVC_COMPILER_DEFS "-D_VARIADIC_MAX=10")
endif() 

#-------------------
# set common include folder for module
#-------------------
set(COMMON_INCLUDES ${PROJECT_SOURCE_DIR}/product/include)

#-------------------
# Module source
#-------------------
set(PROJECT_LIB_NAME ${PROJECT_NAME_STR}_impl)
set(SOURCE_DIR ${PROJECT_SOURCE_DIR}/product/src)

include_directories(${COMMON_INCLUDES})
file(GLOB SRC_FILES ${SOURCE_DIR}/*.cpp)
add_library(${PROJECT_LIB_NAME} ${SRC_FILES})


#-------------------
# Test
#-------------------
add_definitions(${MSVC_COMPILER_DEFS})
enable_testing()
set(PROJECT_TEST_NAME ${PROJECT_NAME_STR}_test)
include_directories(${GTEST_INCLUDE_DIRS} ${COMMON_INCLUDES})

file(GLOB TEST_SRC_FILES ${PROJECT_SOURCE_DIR}/product/test/*.cpp)
add_executable(${PROJECT_TEST_NAME} ${TEST_SRC_FILES} ${SRC_FILES})


if(NOT WIN32 OR MINGW)
    target_link_libraries(${PROJECT_TEST_NAME}
        GTest::GTest
        GTest::Main
        )
else()
    target_link_libraries(${PROJECT_TEST_NAME}
        debug ${GTEST_LIBS_DIR}/DebugLibs/${CMAKE_FIND_LIBRARY_PREFIXES}gtest${CMAKE_FIND_LIBRARY_SUFFIXES}
        optimized ${GTEST_LIBS_DIR}/ReleaseLibs/${CMAKE_FIND_LIBRARY_PREFIXES}gtest${CMAKE_FIND_LIBRARY_SUFFIXES}
        )
    target_link_libraries(${PROJECT_TEST_NAME}
        debug ${GTEST_LIBS_DIR}/DebugLibs/${CMAKE_FIND_LIBRARY_PREFIXES}gtest_main${CMAKE_FIND_LIBRARY_SUFFIXES}
        optimized ${GTEST_LIBS_DIR}/ReleaseLibs/${CMAKE_FIND_LIBRARY_PREFIXES}gtest_main${CMAKE_FIND_LIBRARY_SUFFIXES}
        )
endif()

target_link_libraries(${PROJECT_TEST_NAME} ${CMAKE_THREAD_LIBS_INIT})

add_test(test1 ${PROJECT_TEST_NAME})

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/)

if (CMAKE_BUILD_TYPE STREQUAL "Coverage")
    include(CodeCoverage)
    setup_target_for_coverage(test_coverage_info ${PROJECT_TEST_NAME} coverage)
    find_package(PythonInterp 3)
    find_package(PythonLibs 3)
    setup_target_for_coverage_html(test_coverage ${PROJECT_TEST_NAME} coverage)
    setup_target_for_coverage_cobertura(test_coverage_xml ${PROJECT_TEST_NAME} coverage)

    SET(CMAKE_CXX_FLAGS "-g -O0 -fprofile-arcs -ftest-coverage")
    SET(CMAKE_C_FLAGS "-g -O0 -fprofile-arcs -ftest-coverage")
endif() #CMAKE_BUILD_TYPE STREQUAL "Coverage"