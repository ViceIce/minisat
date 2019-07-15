cmake_minimum_required(VERSION 2.6 FATAL_ERROR)

project(minisat)

#--------------------------------------------------------------------------------------------------
# Configurable options:

option(STATIC_BINARIES "Link binaries statically." ON)
option(USE_SORELEASE   "Use SORELEASE in shared library filename." ON)

option(BUILD_EXECUTABLES "Build executables." ON)

#--------------------------------------------------------------------------------------------------
# Library version:

set(MINISAT_SOMAJOR   2)
set(MINISAT_SOMINOR   1)
set(MINISAT_SORELEASE 0)

# Compute VERSION and SOVERSION:
if (USE_SORELEASE)
  set(MINISAT_VERSION ${MINISAT_SOMAJOR}.${MINISAT_SOMINOR}.${MINISAT_SORELEASE})
else()
  set(MINISAT_VERSION ${MINISAT_SOMAJOR}.${MINISAT_SOMINOR})
endif()
set(MINISAT_SOVERSION ${MINISAT_SOMAJOR})

#--------------------------------------------------------------------------------------------------
# Dependencies:

if(BUILD_EXECUTABLES)
  find_package(ZLIB)
  include_directories(${ZLIB_INCLUDE_DIR})
endif()

include_directories(${minisat_SOURCE_DIR})

#--------------------------------------------------------------------------------------------------
# Compile flags:

add_definitions(-D__STDC_FORMAT_MACROS -D__STDC_LIMIT_MACROS)

#--------------------------------------------------------------------------------------------------
# Build Targets:

set(MINISAT_LIB_SOURCES
    minisat/utils/Options.cc
    minisat/utils/System.cc
    minisat/core/Solver.cc
    minisat/simp/SimpSolver.cc)

if(STATIC_BINARIES)
  add_library(minisat-lib STATIC ${MINISAT_LIB_SOURCES})
  target_link_libraries(minisat-lib ${ZLIB_LIBRARY})
  
  et_target_properties(minisat-lib PROPERTIES OUTPUT_NAME "minisat")
else()
  add_library(minisat-lib SHARED ${MINISAT_LIB_SOURCES})
  target_link_libraries(minisat-lib ${ZLIB_LIBRARY})
  set_target_properties(minisat-lib
    PROPERTIES
      OUTPUT_NAME "minisat" 
      VERSION ${MINISAT_VERSION}
      SOVERSION ${MINISAT_SOVERSION})
endif()

if(BUILD_EXECUTABLES)
  add_executable(minisat_core minisat/core/Main.cc)
  add_executable(minisat_simp minisat/simp/Main.cc)

  target_link_libraries(minisat_core minisat-lib)
  target_link_libraries(minisat_simp minisat-lib)
endif()


if(BUILD_EXECUTABLES)
  set_target_properties(minisat_simp       PROPERTIES OUTPUT_NAME "minisat")
endif()

#--------------------------------------------------------------------------------------------------
# Installation targets:

if(BUILD_EXECUTABLES)
  install(TARGETS minisat-lib minisat_core minisat_simp 
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)
else()
  install(TARGETS minisat-lib
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)
endif()

install(DIRECTORY minisat/mtl minisat/utils minisat/core minisat/simp
        DESTINATION include/minisat
        FILES_MATCHING PATTERN "*.h")
