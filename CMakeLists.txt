cmake_minimum_required(VERSION 3.5)
project(moos)

set(moos_MAJOR_VERSION 10)
set(moos_MINOR_VERSION 5)
set(moos_PATCH_VERSION 0)
set(moos_VERSION
  ${moos_MAJOR_VERSION}.${moos_MINOR_VERSION}.${moos_PATCH_VERSION})

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++17
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 11)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

### Following lifted from the MOOS CMakefile
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH};${CMAKE_CURRENT_SOURCE_DIR}/core-moos/cmake)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
cmake_policy(SET CMP0077 NEW)
set(BUILD_TESTS OFF)
include(GitInfo)
include(PlatformDefines)

# god damn CMake
configure_file(${CMAKE_BINARY_DIR}/Core/libMOOS/include/MOOS/libMOOS/MOOSVersion.h
  ${CMAKE_BINARY_DIR}/core-moos/Core/libMOOS/include/MOOS/libMOOS/MOOSVersion.h)

# bypass the largely unecessary root MOOS CMakefile
add_subdirectory(core-moos/Core)

# because we're compiling all these in the same CMake project, find_package would have set this ordinarily.
set(MOOS_LIBRARIES MOOS)

add_subdirectory(essential-moos/Essentials)
add_subdirectory(geodesy-moos/libMOOSGeodesy)

### Preceeding lifted from the MOOS CMakefile

# uncomment the following section in order to fill in
# further dependencies manually.
# find_package(<dependency> REQUIRED)

target_include_directories(MOOS PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

target_include_directories(MOOSGeodesy PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

# Causes the visibility macros to use dllexport rather than dllimport,
# which is appropriate when building the dll but not consuming it.
# target_compile_definitions(moos PRIVATE "MOOS_BUILDING_LIBRARY")

install(
  DIRECTORY include/
  DESTINATION include
)
install(
  TARGETS MOOSDB MOOS
  EXPORT export_moos
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin)

install(
  TARGETS MOOSGeodesy
  EXPORT export_moosgeodesy
  LIBRARY DESTINATION lib)
  

# export targets
set(export_targets ${export_targets};moos)
export(EXPORT export_moos
  NAMESPACE MOOS
  FILE "${PROJECT_BINARY_DIR}/export_moos.cmake")
export(EXPORT export_moosgeodesy
  FILE "${PROJECT_BINARY_DIR}/export_moosgeodesy.cmake")

# Create the moosConfig.cmake
set(CONF_INCLUDE_DIRS "${CMAKE_INSTALL_PREFIX}/include")
configure_file(moosConfig.cmake.in
  "${PROJECT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/MOOSConfig.cmake" @ONLY)
configure_file(MOOSGeodesyConfig.cmake.in
  "${PROJECT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/MOOSGeodesyConfig.cmake" @ONLY)

# Create the moosConfigVersion.cmake
configure_file(moosConfigVersion.cmake.in
  "${PROJECT_BINARY_DIR}/MOOSConfigVersion.cmake" @ONLY)
# Create the moosConfigVersion.cmake
configure_file(moosConfigVersion.cmake.in
  "${PROJECT_BINARY_DIR}/MOOSGeodesyConfigVersion.cmake" @ONLY)

install(FILES
  "${PROJECT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/MOOSConfig.cmake"
  "${PROJECT_BINARY_DIR}/MOOSConfigVersion.cmake"
  DESTINATION "share/MOOS/cmake" COMPONENT dev)
install(FILES
  "${PROJECT_BINARY_DIR}/${CMAKE_FILES_DIRECTORY}/MOOSGeodesyConfig.cmake"
  "${PROJECT_BINARY_DIR}/MOOSGeodesyConfigVersion.cmake"
  DESTINATION "share/MOOSGeodesy/cmake" COMPONENT dev)

install(EXPORT export_moos
  NAMESPACE MOOS::
  DESTINATION "share/MOOS/cmake"
  FILE export_moos.cmake
  COMPONENT dev)

install(EXPORT export_moosgeodesy
  DESTINATION "share/MOOSGeodesy/cmake"
  FILE export_moosgeodesy.cmake
  COMPONENT dev)