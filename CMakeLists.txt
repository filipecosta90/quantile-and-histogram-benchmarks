## cmake flags
cmake_minimum_required (VERSION 3.0)
project(quantile-hist-bench)

# --- Build properties ---

# Set a default build type if none was specified
set(default_build_type "Release")

IF(NOT CMAKE_BUILD_TYPE)
    message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
    set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
            STRING "Choose the type of build." FORCE)
    # Set the possible values of build type for cmake-gui
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
            "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
ENDIF()

# Generate position-independent code (-fPIC on UNIX)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O3")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3")

if(ENABLE_PROFILE)
  message(STATUS "Enabling profile flags.")
  string (REPLACE "-DNDEBUG" "" CMAKE_C_FLAGS_RELEASE ${CMAKE_C_FLAGS_RELEASE})
  string (REPLACE "-DNDEBUG" "" CMAKE_CXX_FLAGS_RELEASE ${CMAKE_CXX_FLAGS_RELEASE})
  set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -g -ggdb -fno-omit-frame-pointer")
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -g -ggdb -fno-omit-frame-pointer")
  # enable vectorization report flags
  # using Clang
  if (CMAKE_C_COMPILER_ID MATCHES "Clang")
  set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -Rpass-analysis=loop-vectorize -Rpass=loop-vectorize -Rpass-missed=loop-vectorize")

  # using GCC
  elseif (CMAKE_C_COMPILER_ID STREQUAL "GNU")
  set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -ftree-vectorize -fopt-info-vec-all")

  # using Intel C++
  elseif (CMAKE_C_COMPILER_ID STREQUAL "Intel")
  set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -qopt-report=5 -qopt-report-phase=vec")

  # using Visual Studio C++
  elseif (CMAKE_C_COMPILER_ID STREQUAL "MSVC")
  # TBD
  endif()
endif(ENABLE_PROFILE)
set(BENCHMARK_ENABLE_TESTING OFF CACHE BOOL "Suppressing benchmark's tests" FORCE)

# --- Build directories ---
add_subdirectory(deps/benchmark)
include_directories(deps/benchmark/include)

add_subdirectory(deps/HdrHistogram_c)
include_directories(deps/HdrHistogram_c/src)

set(BUILD_BENCHMARK OFF CACHE BOOL "Suppressing benchmark's tests on t-digest" FORCE)
add_subdirectory(deps/t-digest-c)
include_directories(deps/t-digest-c/src)

add_executable(quantile-hist-bench-t-digest tdigest_benchmark.cpp)
target_link_libraries(quantile-hist-bench-t-digest tdigest  benchmark::benchmark)

add_executable(quantile-hist-bench-hdr_histogram hdr_histogram_benchmark.cpp)
target_link_libraries(quantile-hist-bench-hdr_histogram tdigest hdr_histogram  benchmark::benchmark)