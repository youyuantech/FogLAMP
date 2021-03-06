cmake_minimum_required(VERSION 2.6)

set(CMAKE_CXX_FLAGS "-std=c++11 -O3")
set(UUIDLIB -luuid)
set(COMMONLIB -ldl)
 
# Locate GTest
find_package(GTest REQUIRED)
include_directories(${GTEST_INCLUDE_DIRS})

set(BOOST_COMPONENTS system thread)
# Late 2017 TODO: remove the following checks and always use std::regex
if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.9)
        set(BOOST_COMPONENTS ${BOOST_COMPONENTS} regex)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DUSE_BOOST_REGEX")
    endif()
endif()
find_package(Boost 1.53.0 COMPONENTS ${BOOST_COMPONENTS} REQUIRED)
include_directories(SYSTEM ${Boost_INCLUDE_DIR})

include_directories(../../../../C/common/include)
include_directories(../../../../C/services/common/include)
include_directories(../../../../C/services/core/include)
include_directories(../../../../C/thirdparty/rapidjson/include)
include_directories(../../../../C/thirdparty/Simple-Web-Server)

file(GLOB core_services "../../../../C/services/core/*.cpp")
file(GLOB common_services "../../../../C/services/common/*.cpp")
file(GLOB common_sources "../../../../C/common/*.cpp")
file(GLOB unittests "test_*.cpp")
 
# Link runTests with what we want to test and the GTest and pthread library
add_executable(RunTests ${core_services} ${common_services} ${common_sources} "main.cpp" ${unittests})
target_link_libraries(RunTests ${GTEST_LIBRARIES} pthread)
target_link_libraries(RunTests  ${Boost_LIBRARIES})
target_link_libraries(RunTests  ${UUIDLIB})
target_link_libraries(RunTests  ${COMMONLIB})

# Create C++ FogLAMP Core executable
add_executable(foglamp-core core_server.cpp ${core_services} ${common_services} ${common_sources})
target_link_libraries(foglamp-core ${Boost_LIBRARIES})
target_link_libraries(foglamp-core ${CMAKE_THREAD_LIBS_INIT})
target_link_libraries(foglamp-core ${UUIDLIB})
target_link_libraries(foglamp-core ${COMMONLIB})
target_link_libraries(foglamp-core -lssl -lcrypto)
