# Modified 2026 by Bruno Fernandes <bruno.fernandes@tum.de>: ndn-cxx 0.9 found through pkg-config,
# C++17 (required by ndn-cxx 0.9), an install target.
cmake_minimum_required(VERSION 3.10)
project(ndnperf CXX)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin")
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2")

set(SOURCE_FILES main.cpp client.cpp client.h module.h)

find_package(Boost COMPONENTS system thread program_options REQUIRED)
find_package(PkgConfig REQUIRED)
pkg_check_modules(NDN_CXX REQUIRED IMPORTED_TARGET libndn-cxx>=0.9.0)
find_package(Threads REQUIRED)

add_executable(ndnperf ${SOURCE_FILES})

target_link_libraries(ndnperf PkgConfig::NDN_CXX ${Boost_LIBRARIES} Threads::Threads)

install(TARGETS ndnperf RUNTIME DESTINATION bin)
