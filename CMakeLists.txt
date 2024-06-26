cmake_minimum_required(VERSION 3.9)
project(fastcsim)

option(enable_testcases "Build all of tests." ON)
option(enable_coverage "Test coverage." OFF)

set(COMMON_WARNINGS					"-Wall -Wno-unused-function -Wno-unused-variable")
set(CMAKE_CXX_FLAGS           		"${COMMON_WARNINGS}")

if (enable_coverage)
	message(WARNING "Compiler optimizations are disabled, as they are not expected for coverage tests.")
	set(CMAKE_CXX_FLAGS_DEBUG           "-O0 -g")
	set(CMAKE_CXX_FLAGS_RELEASE         "-O0")
	set(CMAKE_CXX_FLAGS_PROFILE         "-O0 -pg")
else ()
	set(CMAKE_CXX_FLAGS_DEBUG           "-g")
	set(CMAKE_CXX_FLAGS_RELEASE         "-O2")
	set(CMAKE_CXX_FLAGS_PROFILE         "-O2 -pg")
endif ()

set(CMAKE_MODULE_LINKER_FLAGS_PROFILE "-pg")
set(CMAKE_SHARED_LINKER_FLAGS_PROFILE "-pg")

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

#
# Check platform
#
if (WIN32)
	add_definitions(-DCSIM_IN_WIN32)
elseif (UNIX)
	add_definitions(-DCSIM_IN_UNIX)
else ()
	message(FATAL_ERROR "Unsupported platform ${CMAKE_SYSTEM}")
endif ()
if (CYGWIN)
    add_definitions(-DCSIM_IN_CYGWIN)
endif ()

#
# Find bison
#
find_package(BISON 2.4 REQUIRED)
if(BISON_FOUND)
else()
message(FATAL_ERROR "BISON is required but not found")
endif()

#
# Find Flex
#
find_package(FLEX 2.5.9  REQUIRED)
if(FLEX_FOUND)
else()
  message(FATAL_ERROR "BISON is required but not found" )
endif()

macro (add_coverage target_name)
	if (enable_coverage)
		target_compile_options(${target_name} PRIVATE
			-fprofile-arcs
			-ftest-coverage)
	endif ()
endmacro ()

include_directories(
	"${PROJECT_SOURCE_DIR}/include"
	"${PROJECT_SOURCE_DIR}/core"
)

add_subdirectory("${PROJECT_SOURCE_DIR}/core/")

if (enable_testcases)
	enable_testing()
	add_subdirectory("${PROJECT_SOURCE_DIR}/testcases")
endif ()

add_subdirectory("${PROJECT_SOURCE_DIR}/doc")

