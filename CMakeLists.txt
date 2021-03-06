cmake_minimum_required (VERSION 2.6)
project (gbolt)
# The version number.
set(GBOLT_VERSION_MAJOR 1)
set(GBOLT_VERSION_MINOR 0)

option(GBOLT_PERFORMANCE "Performance Mode" OFF)
option(GBOLT_SERIAL "Serial Mode" OFF)
if(GBOLT_PERFORMANCE)
  add_definitions(-DGBOLT_PERFORMANCE)
endif(GBOLT_PERFORMANCE)

# Mode
set(CMAKE_BUILD_TYPE RelWithDebInfo)
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -std=c++11")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O3 -g -std=c++11")

# Packages
if(GBOLT_SERIAL)
  add_definitions(-DGBOLT_SERIAL)
else()
  find_package(OpenMP)
  if (OPENMP_FOUND)
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
  endif()
endif(GBOLT_SERIAL)
 
# configure a header file to pass some of the CMake settings
# to the source code
configure_file(
  "${PROJECT_SOURCE_DIR}/config.h.in"
  "${PROJECT_SOURCE_DIR}/include/config.h"
  )
 
# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
set(GBOLT_INCLUDE_DIR "${PROJECT_SOURCE_DIR}/include/gbolt")
set(CXXOPTS_INCLUDE_DIR "${PROJECT_SOURCE_DIR}/extern/cxxopts/include"
	CACHE INTERNAL "${PROJECT_NAME} exec: Include Directories" FORCE)

#INCLUDE_DIRECTORIES(${GBOLT_INCLUDE_DIR} ${CXXOPTS_INCLUDE_DIR})

set(GBOLT_SOURCE_DIR "${PROJECT_SOURCE_DIR}/src")
set(SOURCES
  "${GBOLT_SOURCE_DIR}/history.cc"
  "${GBOLT_SOURCE_DIR}/database.cc"
  "${GBOLT_SOURCE_DIR}/output.cc"
  "${GBOLT_SOURCE_DIR}/gbolt_count.cc"
  "${GBOLT_SOURCE_DIR}/gbolt_execute.cc"
  "${GBOLT_SOURCE_DIR}/gbolt_extend.cc"
  "${GBOLT_SOURCE_DIR}/gbolt_mine.cc")

set(HEADERS
  "${GBOLT_INCLUDE_DIR}/common.h"
  "${GBOLT_INCLUDE_DIR}/config.h"
  "${GBOLT_INCLUDE_DIR}/database.h"
  "${GBOLT_INCLUDE_DIR}/gbolt.h"
  "${GBOLT_INCLUDE_DIR}/graph.h"
  "${GBOLT_INCLUDE_DIR}/history.h"
  "${GBOLT_INCLUDE_DIR}/output.h"
  "${GBOLT_INCLUDE_DIR}/path.h"
  )

add_library(libgbolt SHARED ${SOURCES} ${HEADERS})

#target_include_directories(libgbolt PUBLIC ${GBOLT_INCLUDE_DIR})

target_include_directories(libgbolt
		PUBLIC
		$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>)



# add the executable
add_executable(gbolt  "${GBOLT_SOURCE_DIR}/gbolt.cc" )
target_include_directories(gbolt
	PRIVATE
	${CXXOPTS_INCLUDE_DIR})


target_link_libraries(gbolt libgbolt)

find_library(JEMALLOC NAMES jemalloc PATHS ${JEMALLOC_DIR} PATH_SUFFIXES lib NO_DEFAULT_PATH)
if (JEMALLOC)
  TARGET_LINK_LIBRARIES(gbolt ${JEMALLOC})
endif()

set_target_properties(libgbolt PROPERTIES OUTPUT_NAME "gbolt")

install(TARGETS libgbolt
	EXPORT PROJECT_exports
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT gbolt_lib
		ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT gbolt_lib
		RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}     COMPONENT gbolt_lib
	)

install(DIRECTORY include/gbolt
		DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
		COMPONENT gbolt_dev)
