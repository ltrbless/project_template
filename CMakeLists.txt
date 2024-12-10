cmake_minimum_required(VERSION 3.15) # 更新CMake最低版本要求
project(template_project LANGUAGES CXX)

# 设置C++标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)


# 选项
option(UNIT_TESTS "Unit test project" ON)
option(ENABLE_TBB    "Enable TBB"    OFF)
option(ENABLE_CGAL   "Enable CGAL"    OFF)

if(ENABLE_CGAL)
	message("Using CGAL")
endif()

if(ENABLE_TBB)
	message("Using TBB")
endif()

set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)

if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "No build type selected, default to Release")
    set(CMAKE_BUILD_TYPE Release)
endif()

find_package(OpenMP)

if(NOT EXTERNAL_DIR)
    set(EXTERNAL_DIR "${CMAKE_SOURCE_DIR}/extern")
endif()

if(OPENMP_FOUND)
    message(STATUS "OpenMP found")
endif()

if(MSVC)
    add_compile_options(/bigobj)
endif()

# CGAL
if(ENABLE_CGAL AND NOT TARGET CGAL::CGAL)
	
	SET(CGAL_DIR "${CMAKE_SOURCE_DIR}/extern/cgal")
	find_package(CGAL REQUIRED COMPONENTS Core)
	if(NOT CGAL_FOUND)
		message("Not find CGAL, please you download cgal from github to extern/cgal.")
	endif()

	#set(TBB_ROOT "${CMAKE_SOURCE_DIR}/extern/tbb")
if(ENABLE_TBB)
	find_package(TBB REQUIRED)
	include(CGAL_TBB_support)
endif()

endif()

# TBB
if(ENABLE_TBB AND NOT TARGET tbb::tbb)
	set(TBB_BUILD_STATIC ON CACHE BOOL " " FORCE)
	set(TBB_BUILD_SHARED OFF CACHE BOOL " " FORCE)
	set(TBB_BUILD_TBBMALLOC OFF CACHE BOOL " " FORCE)
	set(TBB_BUILD_TBBMALLOC_PROXY OFF CACHE BOOL " " FORCE)
	set(TBB_BUILD_TESTS OFF CACHE BOOL " " FORCE)
	set(TBB_NO_DATE ON CACHE BOOL " " FORCE)

	add_subdirectory(${EXTERNAL_DIR}/tbb tbb)
	set_target_properties(tbb_static PROPERTIES
		INTERFACE_INCLUDE_DIRECTORIES "${EXTERNAL_DIR}/tbb/include"
	)
	if(NOT MSVC)
		set_target_properties(tbb_static PROPERTIES
			COMPILE_FLAGS "-Wno-implicit-fallthrough -Wno-missing-field-initializers -Wno-unused-parameter -Wno-keyword-macro"
		)
		set_target_properties(tbb_static PROPERTIES POSITION_INDEPENDENT_CODE ON)
	endif()
	add_library(tbb::tbb ALIAS tbb_static)
endif()

# C++11 threads
find_package(Threads REQUIRED)

file(GLOB SOURCES 
    ${PROJECT_SOURCE_DIR}/src/*.cc 
    ${PROJECT_SOURCE_DIR}/src/*.cpp
    ${PROJECT_SOURCE_DIR}/src/*.h
    ${PROJECT_SOURCE_DIR}/src/*.hpp
)

#### add library
add_library(template_lib STATIC ${SOURCES})
target_include_directories(template_lib PUBLIC ${PROJECT_SOURCE_DIR}/src)
target_include_directories(template_lib PUBLIC ${EXTERNAL_DIR}/cli11)
target_include_directories(template_lib PUBLIC ${EXTERNAL_DIR}/eigen)
target_include_directories(template_lib PUBLIC ${EXTERNAL_DIR}/libigl/include)
target_include_directories(template_lib PUBLIC ${EXTERNAL_DIR}/spdlog/include)

## CGAL
if(ENABLE_CGAL)
target_compile_definitions(template_lib PRIVATE USE_CGAL)
target_link_libraries(template_lib PUBLIC CGAL::CGAL CGAL::CGAL_Core)
if(TARGET CGAL::TBB_support)
	target_link_libraries(template_lib PUBLIC CGAL::TBB_support)
endif()
endif()

if(ENABLE_TBB)
	target_include_directories(template_lib PUBLIC ${EXTERNAL_DIR}/tbb/include)
	target_link_libraries(template_lib PUBLIC tbb::tbb)
endif()


if(OPENMP_FOUND)
    target_link_libraries(template_lib PUBLIC OpenMP::OpenMP_CXX)
endif()

if(UNIT_TESTS)
    add_subdirectory(test)
endif()
