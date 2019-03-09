cmake_minimum_required (VERSION 3.0.0)
project (ctmpp)

#Options. Turn on with 'cmake -Dtest=ON
option(test "Build all tests." OFF)
set(PROJECT_NAME ctmpp)
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









#--- Targets ---

#set(exe ${PROJECT_NAME})

#file(GLOB sources src/*.cpp)
#file(GLOB includes include/*.h)
#add_executable(${exe} ${sources} ${includes})






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
  add_executable(runUnitTests test.cpp)
  target_link_libraries(runUnitTests gtest gtest_main)
  add_test(NAME runUnitTests COMMAND runUnitTests)

  # You can also omit NAME and COMMAND. The second argument could be some other
  # test executable.
  add_test(ResetsToZero2 runUnitTests)
endif()






add_executable(ctmpp main.cpp)



