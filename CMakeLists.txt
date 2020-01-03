cmake_minimum_required(VERSION 3.15)
project(CrossGuid VERSION 0.2.2 LANGUAGES CXX)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")

option(XG_TESTS "Build test runner" ON)

include_directories(.)
set(CMAKE_CXX_STANDARD 11)

add_library(crossguid
	Guid.cpp
	Guid.hpp
)

if (XG_TESTS)
	add_executable(xgtest test/TestMain.cpp test/Test.cpp)
	target_link_libraries(xgtest xg)
endif()


if(WIN32)
	target_include_directories(
	crossguid PRIVATE
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
	INTERFACE
	$<INSTALL_INTERFACE:include>
	)
	target_compile_definitions(
	crossguid PUBLIC
	GUID_WINDOWS
	)
	set_target_properties(crossguid
		PROPERTIES
			COMPILE_PDB_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}
			COMPILE_PDB_NAME crossguid
			COMPILE_PDB_NAME_DEBUG crossguidd
			OUTPUT_NAME crossguid
			OUTPUT_NAME_DEBUG crossguidd
	)
  install(FILES ${PROJECT_BINARY_DIR}/RelWithDebInfo/crossguid.pdb DESTINATION lib CONFIGURATIONS RelWithDebInfo)
  install(FILES ${PROJECT_BINARY_DIR}/Debug/crossguidd.pdb DESTINATION lib CONFIGURATIONS Debug)
elseif(APPLE)
	find_library(CFLIB CoreFoundation)
	target_link_libraries(xg ${CFLIB})
	add_definitions(-DGUID_CFUUID)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pedantic")
elseif(ANDROID)
	target_compile_definitions(xg PRIVATE GUID_ANDROID)
else()
	find_package(Libuuid REQUIRED)
	if (NOT LIBUUID_FOUND)
		message(FATAL_ERROR
			"You might need to run 'sudo apt-get install uuid-dev' or similar")
	endif()
	include_directories(${LIBUUID_INCLUDE_DIR})
	target_link_libraries(xg ${LIBUUID_LIBRARY})
	add_definitions(-DGUID_LIBUUID)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pedantic")
endif()



include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion
)

install(TARGETS crossguid EXPORT crossguid
  RUNTIME DESTINATION bin
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib)

install(FILES
  ${CMAKE_CURRENT_SOURCE_DIR}/guid.hpp
  DESTINATION include)

install(EXPORT crossguid
  FILE
    crossguid.cmake
  NAMESPACE
    crossguid::
  DESTINATION
    lib/cmake/crossguid
)
install(
  FILES
    cmake/crossguid-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/crossguid-config-version.cmake
  DESTINATION
    lib/cmake/crossguid
)