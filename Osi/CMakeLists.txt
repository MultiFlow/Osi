set(NAME "OSI")

# PTHREAD
if(PTHREADS_FOUND)
  set(${NAME}_PTHREADS
      "1"
      CACHE INTERNAL "Use pthread")
endif()

set(COIN_${NAME}_CHECKLEVEL
    "0"
    CACHE INTERNAL "${NAME} check level")
set(COIN_${NAME}_VERBOSITY
    "0"
    CACHE INTERNAL "${NAME} verbosity level")
configure_file(config.h.cmake.in config.h)
configure_file(config_osi.h.cmake.in config_osi.h)

# Osi Libary
file(GLOB _SRCS "src/Osi/*.cpp")
file(GLOB _HDRS "src/Osi/*.hpp")

# CPLEX if available
find_package(CPLEX)
if(CPLEX_FOUND)
  # Append OsiCplex src and heagers
  message(STATUS "Adding OsiCplex")
  file(GLOB _SRCS_CPLEX "src/OsiCpx/*.cpp")
  file(GLOB _HDRS_CPLEX "src/OsiCpx/*.hpp")
  set(_SRCS ${_SRCS} ${_SRCS_CPLEX})
  set(_HDRS ${_HDRS} ${_HDRS_CPLEX})
endif()

# GUROBI if available
find_package(GUROBI)
if(GUROBI_FOUND)
  message(STATUS "Adding OsiGrb")
  # Append OsiGrb src and heagers
  file(GLOB _SRCS_GUROBI "src/OsiGrb/*.cpp")
  file(GLOB _HDRS_GUROBI "src/OsiGrb/*.hpp")
  set(_SRCS ${_SRCS} ${_SRCS_GUROBI})
  set(_HDRS ${_HDRS} ${_HDRS_GUROBI})
endif()

# GLPK if available (note: version 4.48!)
find_package(GLPK)
if(GLPK_FOUND AND GLPK_VERSION VERSION_EQUAL "4.48")
  message(STATUS "Adding OsiGlpk")
  file(GLOB _SRCS_GLPK "src/OsiGlpk/*.cpp")
  file(GLOB _HDRS_GLPK "src/OsiGlpk/*.hpp")
  set(_SRCS ${_SRCS} ${_SRCS_GLPK})
  set(_HDRS ${_HDRS} ${_HDRS_GLPK})
endif()

add_library(Osi ${_SRCS} ${_HDRS})
target_include_directories(
  Osi
  PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/Osi>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
         $<INSTALL_INTERFACE:include/coin>)
target_compile_definitions(
  Osi
  PUBLIC HAVE_CONFIG_H
  PRIVATE OSI_BUILD)
if(CMAKE_VERSION VERSION_LESS "3.8.2")
  set_property(TARGET Osi PROPERTY CXX_STANDARD 11)
  set_property(TARGET Osi PROPERTY CXX_STANDARD_REQUIRED ON)
else()
  target_compile_features(Osi PUBLIC cxx_std_11)
endif()

target_link_libraries(Osi PUBLIC Coin::CoinUtils)

# Link optional solvers
if(CPLEX_FOUND)
  target_include_directories(
    Osi PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/OsiCplex>)
  target_link_libraries(Osi PRIVATE CPLEX::CPLEX)
endif()

if(GUROBI_FOUND)
  target_include_directories(
    Osi PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/OsiGrb>)
  target_link_libraries(Osi PRIVATE GUROBI::GUROBI)
endif()

if(GLPK_FOUND)
  target_include_directories(
    Osi PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/OsiGlpk>)
  target_link_libraries(Osi PRIVATE GLPK::GLPK)
endif()

set_target_properties(
  Osi
  PROPERTIES PUBLIC_HEADER "${_HDRS};${CMAKE_CURRENT_BINARY_DIR}/config_osi.h"
             VERSION ${PROJECT_VERSION}
             SOVERSION ${PROJECT_VERSION_MAJOR})
add_library(Coin::Osi ALIAS Osi)

# Install
include(GNUInstallDirs)
install(
  TARGETS Osi
  EXPORT ${PROJECT_NAME}Targets
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/coin
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
