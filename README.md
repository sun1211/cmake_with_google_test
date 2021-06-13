# How to Set up a CMakeLists.txt to compile the executable and its corresponding test using GTest

**1. we require CMake 3.11 to have access to the FetchContent module**
```
# set minimum cmake version
cmake_minimum_required(VERSION 3.11 FATAL_ERROR)

# project name and language
project(recipe-03 LANGUAGES CXX)

# require C++11
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

# example library
add_library(sum_integers sum_integers.cpp)

# main code
add_executable(sum_up main.cpp)
target_link_libraries(sum_up sum_integers)
```
**2. Checking for ENABLE_UNIT_TESTS.**
By default, it is ON, but we want to have the possibility to turn it OFF, in case we do not have any network to download the Google Test sources
```
option(ENABLE_UNIT_TESTS "Enable unit tests" ON)
message(STATUS "Enable testing: ${ENABLE_UNIT_TESTS}")

if(ENABLE_UNIT_TESTS)
  # all the remaining CMake code will be placed here
endif()
```
  - we first include the FetchContent module, declare a new content to fetch, and query its properties
  ```
  include(FetchContent)

  FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG        release-1.8.0
  )

  FetchContent_GetProperties(googletest)
  ```
  - The example also contains some workarounds, for compilation using Visual Studio
  ```
  if(NOT googletest_POPULATED)
  FetchContent_Populate(googletest)

  # Prevent GoogleTest from overriding our compiler/linker options
  # when building with Visual Studio
  set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
  # Prevent GoogleTest from using PThreads
  set(gtest_disable_pthreads ON CACHE BOOL "" FORCE)

  # adds the targers: gtest, gtest_main, gmock, gmock_main
  add_subdirectory(
    ${googletest_SOURCE_DIR}
    ${googletest_BINARY_DIR}
    )

  # Silence std::tr1 warning on MSVC
  if(MSVC)
    foreach(_tgt gtest gtest_main gmock gmock_main)
      target_compile_definitions(${_tgt}
        PRIVATE
          "_SILENCE_TR1_NAMESPACE_DEPRECATION_WARNING"
        )
    endforeach()
  endif()
endif()
```
**3. Define the cpp_test executable target and specify its sources**
```
add_executable(cpp_test "")

target_sources(cpp_test
  PRIVATE
    test.cpp
  )

target_link_libraries(cpp_test
  PRIVATE
    sum_integers
    gtest_main
  )
  ```
  **4. Commands to define the unit test**
  ```
  enable_testing()

add_test(
  NAME google_test
  COMMAND $<TARGET_FILE:cpp_test>
  )
  ```
  
  # How to run
  ```
$ mkdir -p build
$ cd build
$ cmake ..
$ cmake --build .
$ ./cpp_test
```
