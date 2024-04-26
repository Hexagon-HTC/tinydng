cmake_minimum_required(VERSION 3.1)

file(READ "${CMAKE_CURRENT_SOURCE_DIR}/include/tiny_dng_loader.h" STR_HPP)
file(READ "${CMAKE_CURRENT_SOURCE_DIR}/include/tiny_dng_writer.h" STR_HPP)
file(READ "${CMAKE_CURRENT_SOURCE_DIR}/include/miniz.c" STR_HPP)
file(READ "${CMAKE_CURRENT_SOURCE_DIR}/include/miniz.h" STR_HPP)
file(READ "${CMAKE_CURRENT_SOURCE_DIR}/include/stb_image.h" STR_HPP)

project(tinydng VERSION 0.1.0)
file(WRITE "${tinydng_BINARY_DIR}/version" "0.1.0")

# Enable a high level of warnings.
if (CMAKE_COMPILER_IS_GNUCXX)
	# The -Wno-long-long is required in 64bit systems when including sytem headers.
	# The -Wno-variadic-macros was needed for Eigen3, StdVector.h
	add_compile_options(-Wall -Wshadow -Wno-long-long -Wno-variadic-macros)
	if (NOT "${CMAKE_BUILD_TYPE}" STREQUAL "Debug")
		add_compile_options(-O2 -mtune=native)
	endif()
	# Workaround: Eigen <3.4 produces *tons* of warnings in GCC >=6. See http://eigen.tuxfamily.org/bz/show_bug.cgi?id=1221
	if (NOT ${CMAKE_CXX_COMPILER_VERSION} LESS "6.0")
		add_compile_options(-Wno-ignored-attributes -Wno-int-in-bool-context)
	endif()
endif()

if(MSVC)
	 add_definitions( "/W3 /D_CRT_SECURE_NO_WARNINGS /nologo" )
endif()

# Solution Folder options:
if (${CMAKE_SOURCE_DIR} STREQUAL ${PROJECT_SOURCE_DIR})
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set_property(GLOBAL PROPERTY PREDEFINED_TARGETS_FOLDER "CMakeTargets")
endif()

add_definitions ( -DTINYDNG_PATH="${CMAKE_SOURCE_DIR}" )

include(GNUInstallDirs)
if ($ENV{VERBOSE})
	message(STATUS "CMAKE_INSTALL_INCLUDEDIR: ${CMAKE_INSTALL_INCLUDEDIR}")
	message(STATUS "CMAKE_INSTALL_DATADIR   : ${CMAKE_INSTALL_DATADIR}")
	message(STATUS "CMAKE_INSTALL_LIBDIR    : ${CMAKE_INSTALL_LIBDIR}")
	message(STATUS "CMAKE_INSTALL_DOCDIR    : ${CMAKE_INSTALL_DOCDIR}")
	message(STATUS "CMAKE_INSTALL_PREFIX    : ${CMAKE_INSTALL_PREFIX}")
endif()

# Set relative install directories
set(INSTALL_INCLUDE_DIR "${CMAKE_INSTALL_INCLUDEDIR}")
set(INSTALL_CMAKE_DIR "${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/cmake/")
set(INSTALL_COPYRIGHT_DIR "${CMAKE_INSTALL_DOCDIR}")
if(NOT DEFINED PKGCONFIG_INSTALL_DIR)
  set(PKGCONFIG_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/pkgconfig")
endif()

# Save all executables (unit tests & examples) in the same place:
set(EXECUTABLE_OUTPUT_PATH ${${PROJECT_NAME}_BINARY_DIR}/bin CACHE PATH "Output directory for executables")

# Define tinydng lib (header-only)
add_library(tinydng INTERFACE)

# Tell CMake which C++ features we need
target_compile_features(tinydng
	INTERFACE
		cxx_auto_type
		cxx_decltype
		cxx_deleted_functions
)


target_include_directories(tinydng
	INTERFACE
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
		$<INSTALL_INTERFACE:${INSTALL_INCLUDE_DIR}>)

  install(TARGETS tinydng EXPORT tinydngTargets)
  add_library(tinydng::tinydng ALIAS tinydng)


# --------------------------------------------------------------------
# Install/uninstall targets
# --------------------------------------------------------------------

  # Variable for pkgconfig file:
  set(tinydng_pkgconfig_INSTALL_PREFIX "${CMAKE_INSTALL_PREFIX}")
  # Generate the pkg-config file:
configure_file(
	"${tinydng_SOURCE_DIR}/script/tinydng.pc.in"
	"${tinydng_BINARY_DIR}/tinydng.pc" @ONLY IMMEDIATE )

# Generate the cmake config and cmake config-version file:
include(CMakePackageConfigHelpers)

configure_package_config_file(
    "${tinydng_SOURCE_DIR}/script/tinydngConfig.cmake.in"
    "${tinydng_BINARY_DIR}/tinydngConfig.cmake"
    INSTALL_DESTINATION ${INSTALL_CMAKE_DIR}
    PATH_VARS INSTALL_INCLUDE_DIR)


# Setting CMAKE_SIZEOF_VOID_P to the empty string has the same
# effect as the ARCH_INDEPENDENT option of
# write_basic_package_version_file(), but works with older CMake
# versions before 3.14
set(backup_of_CMAKE_SIZEOF_VOID_P "${CMAKE_SIZEOF_VOID_P}")
set(CMAKE_SIZEOF_VOID_P "")

write_basic_package_version_file(
    "${tinydng_BINARY_DIR}/tinydngConfigVersion.cmake"
    VERSION ${tinydng_VERSION}
    COMPATIBILITY AnyNewerVersion)

set(CMAKE_SIZEOF_VOID_P "${backup_of_CMAKE_SIZEOF_VOID_P}")

# Uninstall target, for "make uninstall"
configure_file(
  "${tinydng_SOURCE_DIR}/script/cmake_uninstall.cmake.in"
  "${tinydng_BINARY_DIR}/cmake_uninstall.cmake"
  @ONLY IMMEDIATE)

if (NOT TARGET uninstall)
  add_custom_target(uninstall
    "${CMAKE_COMMAND}" -P "${tinydng_BINARY_DIR}/cmake_uninstall.cmake")
else()
  add_custom_target(tinydng_uninstall
    "${CMAKE_COMMAND}" -P "${tinydng_BINARY_DIR}/cmake_uninstall.cmake")
  add_dependencies(uninstall tinydng_uninstall)
endif()


  export(EXPORT tinydngTargets
       NAMESPACE tinydng::
       FILE "${tinydng_BINARY_DIR}/tinydngTargets.cmake")

export(PACKAGE tinydng)

install(EXPORT tinydngTargets
        NAMESPACE tinydng::
        DESTINATION "${INSTALL_CMAKE_DIR}")

install(
  FILES "${tinydng_BINARY_DIR}/tinydng.pc"
  DESTINATION "${PKGCONFIG_INSTALL_DIR}" )

install(
  FILES "${tinydng_BINARY_DIR}/tinydngConfig.cmake"
        "${tinydng_BINARY_DIR}/tinydngConfigVersion.cmake"
  DESTINATION "${INSTALL_CMAKE_DIR}" )

  install(
	FILES "${tinydng_SOURCE_DIR}/include/tiny_dng_loader.h"
        "${tinydng_SOURCE_DIR}/include/tiny_dng_writer.h"
        "${tinydng_SOURCE_DIR}/include/stb_image.h"
        "${tinydng_SOURCE_DIR}/include/miniz.h"
        "${tinydng_SOURCE_DIR}/include/miniz.c"

	DESTINATION "${INSTALL_INCLUDE_DIR}" )

