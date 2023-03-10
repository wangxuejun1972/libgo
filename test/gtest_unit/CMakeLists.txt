cmake_minimum_required(VERSION 2.8)

###################################################################################
project(unit_test)

if (CMAKE_BUILD_TYPE)
else()
    #set(CMAKE_BUILD_TYPE RELEASE)
    set(CMAKE_BUILD_TYPE DEBUG)
endif()

message("------------ Options -------------")
message("  CMAKE_BUILD_TYPE: ${CMAKE_BUILD_TYPE}")
message("  CMAKE_COMMAND: ${CMAKE_COMMAND}")
message("-------------- Env ---------------")
message("  CMAKE_SOURCE_DIR: ${CMAKE_SOURCE_DIR}")
message("  CMAKE_BINARY_DIR: ${CMAKE_BINARY_DIR}")
message("----------------------------------")

if (UNIX)
    set(CMAKE_CXX_FLAGS "-std=c++11 -fPIC -Wall -m64 ${CMAKE_CXX_FLAGS}")
    set(CMAKE_CXX_FLAGS_DEBUG "-g ${CMAKE_CXX_FLAGS}")
    set(CMAKE_CXX_FLAGS_RELEASE "-g -O3 ${CMAKE_CXX_FLAGS}")
elseif (WIN32)
    # windows platform
    #add_definitions(-D_CRT_SECURE_NO_WARNINGS)
    #set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MDd")
    #set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MD")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd /EHsc")
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT /EHsc")
endif()

option(SMALL_TEST "run the test with small mem and cpu" OFF)
if (${SMALL_TEST} STREQUAL "ON")
    add_definitions(-DSMALL_TEST)
endif()

include(${PROJECT_SOURCE_DIR}/boost.cmake)

include_directories(${PROJECT_SOURCE_DIR}/../../third_party/gtest/include)
include_directories(${PROJECT_SOURCE_DIR}/../..)
include_directories(${PROJECT_SOURCE_DIR}/../../libgo)
link_directories(${PROJECT_SOURCE_DIR}/../../build)
link_directories(${PROJECT_BINARY_DIR}/gtest)
link_directories(${PROJECT_BINARY_DIR}/../..)
add_subdirectory(${PROJECT_SOURCE_DIR}/../../third_party/gtest ${PROJECT_BINARY_DIR}/gtest)

aux_source_directory(${PROJECT_SOURCE_DIR} SRC_LIST)
aux_source_directory(${PROJECT_SOURCE_DIR}/hook SRC_LIST)

if (UNIX)
    include_directories(${PROJECT_SOURCE_DIR}/../../libgo/linux)
    #set(STATIC_LINK_ARG -Wl,--whole-archive -lpthread -Wl,--no-whole-archive -static)
    set(STATIC_LINK_ARG -static)
    set(LINK_ARGS libgo gtest_main gtest ${Boost_LIBRARIES} pthread dl)
elseif (WIN32)
    include_directories(${PROJECT_SOURCE_DIR}/../../libgo/windows)
    set(LINK_ARGS libgo gtest gtest_main ${Boost_LIBRARIES} ws2_32)

    list(REMOVE_ITEM SRC_LIST ${PROJECT_SOURCE_DIR}/hook/file.cpp)
    list(REMOVE_ITEM SRC_LIST ${PROJECT_SOURCE_DIR}/io_timed.cpp)
    list(REMOVE_ITEM SRC_LIST ${PROJECT_SOURCE_DIR}/hook/poll.cpp)
    list(REMOVE_ITEM SRC_LIST ${PROJECT_SOURCE_DIR}/sleep.cpp)
    list(REMOVE_ITEM SRC_LIST ${PROJECT_SOURCE_DIR}/select.cpp)
    list(REMOVE_ITEM SRC_LIST ${PROJECT_SOURCE_DIR}/protect.cpp)
    list(REMOVE_ITEM SRC_LIST ${PROJECT_SOURCE_DIR}/hook/dns.cpp)
    list(REMOVE_ITEM SRC_LIST ${PROJECT_SOURCE_DIR}/hook/dup.cpp)
    list(REMOVE_ITEM SRC_LIST ${PROJECT_SOURCE_DIR}/hook/signal.cpp)
endif()

if (APPLE)
    list(REMOVE_ITEM SRC_LIST ${PROJECT_SOURCE_DIR}/hook/dns.cpp)
endif()

message("SRC_LIST: ${SRC_LIST}")

foreach(var ${SRC_LIST})
    string(REGEX REPLACE ".*/" "" source ${var})
    string(REGEX REPLACE ".cpp" "" tgt ${source})
    #message("${TEST_LINK_FLAGS}")

    add_executable(${tgt}.t ${var})
    if (WIN32)
        target_link_libraries(${tgt}.t ${LINK_ARGS})
        set_target_properties(${tgt}.t PROPERTIES COMPILE_FLAGS "/wd4819 /wd4267 /wd4996")
        set_target_properties(${tgt}.t PROPERTIES INSTALL_RPATH ${PROJECT_SOURCE_DIR}/../../build)
    elseif (UNIX)
        target_link_libraries(${tgt}.t ${LINK_ARGS})
    endif()

    #add_executable(s_${tgt}.t ${var})
    #target_link_libraries(s_${tgt}.t ${TEST_LINK_FLAGS} ${LINK_ARGS} ${STATIC_LINK_ARG})
    #if (WIN32)
    #    set_target_properties(s_${tgt}.t PROPERTIES COMPILE_FLAGS "/wd4819 /wd4267 /wd4996")
    #elseif (UNIX)
    #    set_target_properties(s_${tgt}.t PROPERTIES LINK_FLAGS " -Wl,--whole-archive -lstatic_hook -lc -lpthread -Wl,--no-whole-archive")
    #endif()

endforeach(var)

