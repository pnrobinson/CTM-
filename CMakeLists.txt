cmake_minimum_required (VERSION 3.0.0)
project (egtk)

#Options. Turn on with 'cmake -Dtest=ON
option(test "Build all tests." OFF)
set(PROJECT_NAME egtk)
set(CMAKE_CXX_FLAGS "-g -Wall -pedantic")
set(CMAKE_CXX_STANDARD 14) ## use C++14 [required by seqan2.4]

## Download and build gtest
configure_file(CMakeLists.gtest.in googletest-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/googletest-download )
if(result)
  message(FATAL_ERROR "CMake step for googletest failed: ${result}")
endif()
execute_process(COMMAND ${CMAKE_COMMAND} --build .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/googletest-download )
if(result)
  message(FATAL_ERROR "Build step for googletest failed: ${result}")
endif()

## Download seqan from GitHub
configure_file(CMakeLists.seqan.in seqan-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/seqan-download )
if(result)
  message(FATAL_ERROR "CMake step for seqan failed: ${result}")
endif()
set (CMAKE_MODULE_PATH ${CMAKE_BINARY_DIR}/seqan/util/cmake)
set (SEQAN_INCLUDE_PATH ${CMAKE_BINARY_DIR}/seqan/include)

execute_process(COMMAND ${CMAKE_COMMAND} --build .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/seqan-download )
if(result)
  message(FATAL_ERROR "Build step for seqan failed: ${result}")
endif()


## Download bpp-core from GitHub
configure_file(CMakeLists.bpp-core.in bpp-core-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/bpp-core-download )
if(result)
  message(FATAL_ERROR "CMake step for bpp-core failed: ${result}")
endif()
#set (CMAKE_MODULE_PATH ${CMAKE_BINARY_DIR}/seqan/util/cmake)
#set (SEQAN_INCLUDE_PATH ${CMAKE_BINARY_DIR}/seqan/include)

execute_process(COMMAND ${CMAKE_COMMAND} --build .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/bpp-core-download )
if(result)
  message(FATAL_ERROR "Build step for bpp-core failed: ${result}")
endif()

## Download bpp-seq from GitHub
configure_file(CMakeLists.bpp-seq.in bpp-seq-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/bpp-seq-download )
if(result)
  message(FATAL_ERROR "CMake step for bpp-seq failed: ${result}")
endif()

execute_process(COMMAND ${CMAKE_COMMAND} --build .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/bpp-seq-download )
if(result)
  message(FATAL_ERROR "Build step for bpp-seq failed: ${result}")
endif()

## Download bpp-seq-omics from GitHub
configure_file(CMakeLists.bpp-seq-omics.in bpp-seq-omics-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/bpp-seq-omics-download )
if(result)
  message(FATAL_ERROR "CMake step for bpp-seq-omics failed: ${result}")
endif()

execute_process(COMMAND ${CMAKE_COMMAND} --build .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/bpp-seq-omics-download )
if(result)
  message(FATAL_ERROR "Build step for bpp-seq-omics failed: ${result}")
endif()



# This section is copied from the bio++ documentation page on CMake compilation.
# http://biopp.univ-montp2.fr/wiki/index.php/CMake_compilation

# I will leave the original block commented, followed by my attempts to edit a copy.
# The original instructions are for building a project that uses bpp-phyl. We don't want
# that, but we do want bpp-seq-omics.

# We definitely want to use static compilation, so that users will not have to
# set environment varaiables at run-time.
# 
# --- the IMPROVED_FIND_LIBRARY macro (taken from the Bio++ CMakeLists.txt) ---
#macro(IMPROVED_FIND_LIBRARY OUTPUT_LIBS lib_name include_to_find)
#  FIND_PATH(${lib_name}_INCLUDE_DIR ${include_to_find})
#  SET(${lib_name}_NAMES ${lib_name} ${lib_name}lib ${lib_name}dll)
#  FIND_LIBRARY(${lib_name}_LIBRARY NAMES ${${lib_name}_NAMES})
#
#  IF(${lib_name}_LIBRARY)
#    MESSAGE("-- Library ${lib_name} found here:")
#    MESSAGE("    ${${lib_name}_INCLUDE_DIR}/")
#    MESSAGE("    ${${lib_name}_LIBRARY}")
#  ELSE(${lib_name}_LIBRARY)
#    MESSAGE(FATAL_ERROR "${lib_name} required but not found.")
#  ENDIF(${lib_name}_LIBRARY)
  
#  #add the dependency:
#  INCLUDE_DIRECTORIES(${${lib_name}_INCLUDE_DIR})
#  SET(${OUTPUT_LIBS} ${${OUTPUT_LIBS}} ${${lib_name}_LIBRARY})
#ENDMACRO(IMPROVED_FIND_LIBRARY)

#--- Libraries & includes ---
include_directories("include/")

#set(CMAKE_FIND_LIBRARY_SUFFIXES ".a ${CMAKE_FIND_LIBRARY_SUFFIXES}") # uncomment for static compilation
IMPROVED_FIND_LIBRARY(LIBS_BPP bpp-core "Bpp/Clonable.h")
IMPROVED_FIND_LIBRARY(LIBS_BPP bpp-seq "Bpp/Seq/Sequence.h")
IMPROVED_FIND_LIBRARY(LIBS_BPP bpp-seq-omics "Bpp/Seq/Io/MafBlock.h")

#--- Targets ---

set(exe ${PROJECT_NAME})

file(GLOB sources src/*.cpp)
file(GLOB includes include/*.h)
add_executable(${exe} ${sources} ${includes})

target_link_libraries(${exe} ${LIBS_BPP})

# END EDITS TO BPP BUILD



# Include the subdirectory "maf", which will build a static
# library with the maf parsing functionality
set (MAF_INCLUDE_DIR ${CMAKE_CURRENT_SRC_DIR}/maf)
add_subdirectory(maf)
include_directories(${MAF_INCLUDE_DIR})


if (test)
  if (APPLE)
    add_definitions(-DGTEST_USE_OWN_TR1_TUPLE)
    add_definitions(-D__GLIBCXX__)
  endif (APPLE)
  # Prevent overriding the parent project's compiler/linker
  # settings on Windows
  set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

  # Add googletest directly to our build. This defines
  # the gtest and gtest_main targets.
  add_subdirectory(${CMAKE_BINARY_DIR}/googletest-src
    ${CMAKE_BINARY_DIR}/googletest-build)

  #add_subdirectory(${CMAKE_BINARY_DIR}/googletest-build)
  enable_testing()
  include_directories(${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR})
  include_directories(bpp-core/src bpp-core-omics/src bpp-seq/src)
  add_executable(runUnitTests test.cpp)
  target_link_libraries(runUnitTests gtest gtest_main)
  target_link_libraries(runUnitTests maf)
  add_test(NAME runUnitTests COMMAND runUnitTests)

  # You can also omit NAME and COMMAND. The second argument could be some other
  # test executable.
  add_test(ResetsToZero2 runUnitTests)
endif()






add_executable(ctmpp main.cpp)


target_link_libraries(egtk maf)
ADD_DEPENDENCIES(egtk maf)
