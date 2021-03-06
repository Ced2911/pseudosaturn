CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

include(CMakeForceCompiler)
include(ExternalProject)

SET(CMAKE_SYSTEM_NAME SegaSaturn)

if (WIN32)
	set(SH_ELF_PATHS 
		"[HKEY_LOCAL_MACHINE\\SOFTWARE\\KPIT\\GNUSH v12.02-ELF;Path]/sh-elf/bin"
		"[HKEY_LOCAL_MACHINE\\SOFTWARE\\KPIT Cummins\\GNUSH v10.02-ELF;Path]/sh-elf/bin")
else (WIN32)	        
	set(SH_ELF_PATHS "")
endif (WIN32)

find_program(SH_ELF_GCC
	NAMES sh-elf-gcc
	PATHS
		${SH_ELF_PATHS})

find_program(SH_ELF_GPP
	NAMES sh-elf-g++
	PATHS
		${SH_ELF_PATHS})
		
# Specify the cross compiler.
CMAKE_FORCE_C_COMPILER(${SH_ELF_GCC} GNU)
CMAKE_FORCE_CXX_COMPILER(${SH_ELF_GPP} GNU)
SET(CMAKE_FIND_ROOT_PATH /usr/local/share/sh-elf/bin)

find_path(IAPETUS_INCLUDE_DIR 
          NAMES iapetus.h
          PATHS ${IAPETUS_ROOT_PATH}/include
                $ENV{ProgramFiles}/iapetus/include )

if (IAPETUS_INCLUDE_DIR STREQUAL "IAPETUS_INCLUDE_DIR-NOTFOUND")
   ExternalProject_Add(iapetus
      GIT_REPOSITORY "https://github.com/ced2911/iapetus.git"
      LOG_DOWNLOAD 1
      LOG_CONFIGURE 1
      LOG_BUILD 1
      INSTALL_COMMAND "" 
   )

   ExternalProject_Get_Property(iapetus source_dir)
   set(IAPETUS_INCLUDE_DIR ${source_dir}/src)
endif(IAPETUS_INCLUDE_DIR STREQUAL "IAPETUS_INCLUDE_DIR-NOTFOUND")

ExternalProject_Add(bin2c
	DOWNLOAD_COMMAND ""
	SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src/bin2c
	CMAKE_GENERATOR "${CMAKE_GENERATOR}"
	INSTALL_COMMAND ""
	BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}/bin2c)

set(CMAKE_BIN2C "${CMAKE_CURRENT_BINARY_DIR}/bin2c/bin2c")

PROJECT(PseudoSaturn)

include_directories(${IAPETUS_INCLUDE_DIR})

#CONFIGURE_FILE(config.h.cmake config.h)

set(PSEUDOSATURN_VERSION_MAJOR 0)
set(PSEUDOSATURN_VERSION_MINOR 8)
set(PSEUDOSATURN_VERSION_PATCH 32)
set(PSEUDOSATURN_VERSION "${PSEUDOSATURN_VERSION_MAJOR}.${PSEUDOSATURN_VERSION_MINOR}${PSEUDOSATURN_VERSION_PATCH}")

set(INSTALLER_VERSION_MAJOR 1)
set(INSTALLER_VERSION_MINOR 2)
set(INSTALLER_VERSION "${INSTALLER_VERSION_MAJOR}.${INSTALLER_VERSION_MINOR}")

set(CPACK_SOURCE_GENERATOR TGZ)
set(CPACK_PACKAGE_VERSION_MAJOR ${PSEUDOSATURN_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PSEUDOSATURN_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PSEUDOSATURN_VERSION_PATCH})
set(CPACK_PACKAGE_VENDOR "Cyber Warrior X")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/COPYING")
set(CPACK_SOURCE_PACKAGE_FILE_NAME "$PseudoSaturn-${PSEUDOSATURN_VERSION}")
set(CPACK_PACKAGE_FILE_NAME "PseudoSaturn-${PSEUDOSATURN_VERSION}")
include(CPack)

set(CMAKE_CONFIGURATION_TYPES Release)

find_library(IAPETUS_LIB
             NAMES iapetus
             PATHS ${IAPETUS_ROOT_PATH}/lib
                   $ENV{ProgramFiles}/iapetus/lib)

if (IAPETUS_LIB STREQUAL "IAPETUS_LIB-NOTFOUND")
   ExternalProject_Get_Property(iapetus binary_dir)
   #add_dependencies(${PROJECT_NAME} iapetus)
   set(IAPETUS_LIB "${binary_dir}/src/${CMAKE_FIND_LIBRARY_PREFIXES}iapetus.a")
endif(IAPETUS_LIB STREQUAL "IAPETUS_LIB-NOTFOUND")

set(PseudoSaturn_LIBRARIES ${IAPETUS_LIB})

option(PS_WANT_DEBUG "Enable Debug Code" OFF)
if (PS_WANT_DEBUG)
	add_definitions(-DENABLE_DEBUG=1)
endif()

option(PS_WANT_CHEATS "Enable experimental cheat engine" OFF)
if (PS_WANT_CHEATS)
	add_definitions(-DENABLE_CHEATS=1)
endif()

option(PS_WANT_BACKUPRAM "Enable Backup Ram" OFF)
if (PS_WANT_DEBUG)
	add_definitions(-DENABLE_BACKUPRAM=1)
endif()

add_subdirectory(src)