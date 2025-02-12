cmake_minimum_required(VERSION 3.0)
project(bxzstr)

## For FindZstd
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

if(DEFINED ZLIB_FOUND)
  if(ZLIB_FOUND)
    set(BXZSTR_Z_SUPPORT 1)
  else()
    set(BXZSTR_Z_SUPPORT 0)
  endif()
else()
  find_package(ZLIB)
  if(ZLIB_FOUND)
    set(BXZSTR_Z_SUPPORT 1)
  else()
    set(BXZSTR_Z_SUPPORT 0)
  endif()
endif()

if(DEFINED BZIP2_FOUND)
  if(BZIP2_FOUND)
    set(BXZSTR_BZ2_SUPPORT 1)
  else()
    set(BXZSTR_BZ2_SUPPORT 0)
  endif()
else()
  find_package(BZip2)
  if(BZIP2_FOUND)
    set(BXZSTR_BZ2_SUPPORT 1)
  else()
    set(BXZSTR_BZ2_SUPPORT 0)
  endif()
endif()

if(DEFINED LIBLZMA_FOUND)
  if(LIBLZMA_FOUND)
    set(BXZSTR_LZMA_SUPPORT 1)
  else()
    set(BXZSTR_LZMA_SUPPORT 0)
  endif()
else()
  find_package(LibLZMA)
  if(LIBLZMA_FOUND)
    set(BXZSTR_LZMA_SUPPORT 1)
  else()
    set(BXZSTR_LZMA_SUPPORT 0)
  endif()
endif()

if(NOT TARGET libzstd_static)
  if(DEFINED ZSTD_FOUND)
    if(ZSTD_FOUND)
      set(BXZSTR_ZSTD_SUPPORT 1)
    else()
      set(BXZSTR_ZSTD_SUPPORT 0)
    endif()
  else()
    find_package(Zstd)
    if(ZSTD_FOUND)
      set(BXZSTR_ZSTD_SUPPORT 1)
    else()
      set(BXZSTR_ZSTD_SUPPORT 0)
    endif()
  endif()
else()
  add_library(Zstd::Zstd ALIAS libzstd_static)
  set(ZSTD_FOUND ON)
  set(BXZSTR_ZSTD_SUPPORT 1)
endif()

configure_file(include/config.hpp.in ${CMAKE_CURRENT_SOURCE_DIR}/include/config.hpp @ONLY)

add_library(bxzstr INTERFACE)
target_include_directories(bxzstr INTERFACE include)

if(DEFINED ZLIB_FOUND AND ZLIB_FOUND)
  target_link_libraries(bxzstr INTERFACE z)
endif()

if(DEFINED BZIP2_FOUND AND BZIP2_FOUND)
  target_link_libraries(bxzstr INTERFACE bz2)
endif()

if(DEFINED LIBLZMA_FOUND AND LIBLZMA_FOUND)
  target_link_libraries(bxzstr INTERFACE lzma)
endif()

if(DEFINED ZSTD_FOUND AND ZSTD_FOUND)
  target_link_libraries(bxzstr INTERFACE Zstd::Zstd)
endif()

## Download googletest if building tests
if(CMAKE_BUILD_TESTS)
  if (DEFINED CMAKE_GOOGLETEST_HEADERS)
    message(STATUS "googletest headers provided in: ${CMAKE_GOOGLETEST_HEADERS}")
  else()
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/test/CMakeLists-googletest.txt.in ${CMAKE_CURRENT_BINARY_DIR}/external/googletest-download/CMakeLists.txt)
    execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
      RESULT_VARIABLE result
      WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/external/googletest-download )
    if(result)
      message(FATAL_ERROR "CMake step for googletest failed: ${result}")
    endif()
    execute_process(COMMAND ${CMAKE_COMMAND} --build .
      RESULT_VARIABLE result
      WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/external/googletest-download )
    if(result)
      message(FATAL_ERROR "Build step for googletest failed: ${result}")
    endif()
    set(CMAKE_GOOGLETEST_HEADERS ${CMAKE_CURRENT_BINARY_DIR}/external/googletest/googletest/include)
    add_subdirectory(${CMAKE_CURRENT_BINARY_DIR}/external/googletest
                     ${CMAKE_CURRENT_BINARY_DIR}/external/googletest/build)
  endif()
  set_target_properties(gmock PROPERTIES EXCLUDE_FROM_ALL 1)
  set_target_properties(gmock_main PROPERTIES EXCLUDE_FROM_ALL 1)
  include_directories("${CMAKE_GOOGLETEST_HEADERS}")
  include_directories(${CMAKE_CURRENT_SOURCE_DIR}/test/include ${CMAKE_CURRENT_SOURCE_DIR}/include)
  enable_testing()
  add_executable(runTests
    ${CMAKE_CURRENT_SOURCE_DIR}/test/src/compression_types_unittest.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/test/src/z_stream_wrapper_unittest.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/test/src/bz_stream_wrapper_unittest.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/test/src/lzma_stream_wrapper_unittest.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/test/src/zstd_stream_wrapper_unittest.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/test/src/bxzstr_ofstream_integrationtest.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/test/src/bxzstr_ifstream_integrationtest.cpp)
  add_test(runTests runTests)

  if(DEFINED ZLIB_FOUND)
    target_link_libraries(runTests gtest gtest_main z)
  endif()

  if(DEFINED BZIP2_FOUND)
    target_link_libraries(runTests gtest gtest_main bz2)
  endif()

  if(DEFINED LIBLZMA_FOUND)
    target_link_libraries(runTests gtest gtest_main lzma)
  endif()

  if(DEFINED ZSTD_FOUND)
    target_link_libraries(runTests gtest gtest_main zstd)
  endif()
endif()
