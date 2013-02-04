# @author $username$ <$usermail$>
# @date $date$
# NULL CMake build script

cmake_minimum_required(VERSION 2.8.3)

###############################################################################
# options

option(BUILD_TESTS  "Build tests." OFF)
option(BUILD_GTEST  "Build gtest with the project. Usefull for windows systems, to prevent DLL-HELL" OFF)
option(GENERATE_DOC "Don't generate docs" ON)

###############################################################################
# general

project(NULL)

include("${PROJECT_SOURCE_DIR}/cmake/ext/nx_utils.cmake")
nx_configure()

set(NULL_VERSION_MAJOR 0)
set(NULL_VERSION_MINOR 0)
# configuration header
configure_file(
	"${PROJECT_SOURCE_DIR}/src/nullconfig.cfg"
	"${PROJECT_SOURCE_DIR}/src/nullconfig.hpp"
	ESCAPE_QUOTES
)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake/Modules")

include_directories("${PROJECT_SOURCE_DIR}/src")

###########################################################################
# docs

if(GENERATE_DOC)
	FIND_PACKAGE(Doxygen)
	if(DOXYGEN_FOUND)
		set(ENV{NULL_ROOT} ${PROJECT_SOURCE_DIR})
		ADD_CUSTOM_TARGET(doc COMMAND ${CMAKE_COMMAND} 
			-D PDIR="${PROJECT_SOURCE_DIR}"
			-P "${PROJECT_SOURCE_DIR}/doc/doxygen.cmake")
	else()
		message(WARNING "Doxygen not found. Documentation wouldn't be ganarated")
	endif()
endif()


###########################################################################
# tests

if(BUILD_TESTS)
	set(SOURCES_TEST
		./test/test.cpp)
	enable_testing()
	find_package(GTest REQUIRED)
	include_directories(${GTEST_INCLUDE_DIRS})
	set(TEST test_${PROJECT_NAME})
	add_executable(${TEST} ${SOURCES_TEST})
	if(NOT WIN32)
		SET_TARGET_PROPERTIES(${TEST} PROPERTIES LINK_FLAGS -pthread)
	endif(NOT WIN32)
	target_link_libraries("${TEST}" ${GTEST_LIBRARIES} ${LIBRARIES})
	nx_GTEST_ADD_TESTS("${TEST}" ${SOURCES_TEST})
endif()

###########################################################################
# installation

set(CPACK_SET_DESTDIR ON)

install(TARGETS ${TARGETS} DESTINATION bin)
#install(FILES <files> DESTINATION <dest>)

INCLUDE(InstallRequiredSystemLibraries)

SET(CPACK_PACKAGE_DESCRIPTION_SUMMARY "NULL DESC")
SET(CPACK_PACKAGE_VENDOR "ORG")
#SET(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/ReadMe.txt")
#SET(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/Copyright.txt")
SET(CPACK_PACKAGE_VERSION_MAJOR ${NULL_VERSION_MAJOR})
SET(CPACK_PACKAGE_VERSION_MINOR ${NULL_VERSION_MINOR})
SET(CPACK_PACKAGE_VERSION_PATCH "0")
SET(CPACK_PACKAGE_INSTALL_DIRECTORY "CMake ${CMake_VERSION_MAJOR}.${CMake_VERSION_MINOR}")
IF(WIN32 AND NOT UNIX)
	# NOT IMPLEMENTED
ELSE(WIN32 AND NOT UNIX)
	SET(CPACK_STRIP_FILES ${TARGETS})
	SET(CPACK_SOURCE_STRIP_FILES "")
ENDIF(WIN32 AND NOT UNIX)
#SET(CPACK_PACKAGE_EXECUTABLES "null" "null desc")

INCLUDE(CPack)

