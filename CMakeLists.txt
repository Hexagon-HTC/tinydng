cmake_minimum_required(VERSION 3.1)

file(READ "${CMAKE_CURRENT_SOURCE_DIR}/tiny_dng_loader.h" STR_HPP)
file(READ "${CMAKE_CURRENT_SOURCE_DIR}/tiny_dng_writer.h" STR_HPP)

project(tinydng)

# Solution Folder options:
if (${CMAKE_SOURCE_DIR} STREQUAL ${PROJECT_SOURCE_DIR})
	set_property(GLOBAL PROPERTY USE_FOLDERS ON)
	set_property(GLOBAL PROPERTY PREDEFINED_TARGETS_FOLDER "CMakeTargets")
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
target_compile_features(tinydng	INTERFACE)

target_include_directories(tinydng
	INTERFACE
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
		$<INSTALL_INTERFACE:${INSTALL_INCLUDE_DIR}>)

  install(TARGETS tinydng EXPORT tinydngTargets)
  add_library(tinydng::tinydng ALIAS tinydng)


  # Generate the pkg-config file:
configure_file(
	"${tinydng_SOURCE_DIR}/tinydng.pc.in"
	"${tinydng_BINARY_DIR}/tinydng.pc" @ONLY IMMEDIATE )

# Generate the cmake config and cmake config-version file:
include(CMakePackageConfigHelpers)

configure_package_config_file(
    "${tinydng_SOURCE_DIR}/tinydngConfig.cmake.in"
    "${tinydng_BINARY_DIR}/tinydngConfig.cmake"
    INSTALL_DESTINATION ${INSTALL_CMAKE_DIR}
    PATH_VARS INSTALL_INCLUDE_DIR)

  export(EXPORT tinydngTargets
       NAMESPACE tinydng::
       FILE "${tinydng_BINARY_DIR}/tinydngTargets.cmake")

export(PACKAGE tinydng)

install(EXPORT tinydngTargets
        NAMESPACE tinydng::
        DESTINATION "${INSTALL_CMAKE_DIR}")

install(
  FILES "${tinydng_BINARY_DIR}/tinydngConfig.cmake"
        "${tinydng_BINARY_DIR}/tinydngConfigVersion.cmake"
  DESTINATION "${INSTALL_CMAKE_DIR}" )
