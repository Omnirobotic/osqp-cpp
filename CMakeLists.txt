# Copyright 2020 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

cmake_minimum_required(VERSION 3.14)

project(osqp-cpp)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Options.
option(OSQP-CPP_BUILD_TESTS
  "Whether to build tests." ON
)

include(FetchContent)

find_package(absl REQUIRED)
find_package(Eigen3 3.3.7 REQUIRED NO_MODULE)
find_package(osqp REQUIRED)

# Googletest
if (OSQP-CPP_BUILD_TESTS)
  enable_testing()
  if (NOT TARGET gtest OR NOT TARGET gmock OR NOT TARGET gtest_main)
    message(STATUS "osqp-cpp: `googletest` targets not found. Attempting to fetch contents...")
    set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
    FetchContent_Declare(
      googletest
      GIT_REPOSITORY https://github.com/google/googletest.git
      GIT_TAG        origin/main
    )
    FetchContent_MakeAvailable(googletest)
    include(GoogleTest)
  else()
    message(STATUS "osqp-cpp: `googletest` targets found.")
  endif()
endif()

message(STATUS "osqp-cpp: Adding osqp-cpp library...")
add_library(osqp-cpp src/osqp++.cc)
target_link_libraries(osqp-cpp PUBLIC absl::strings absl::status absl::statusor Eigen3::Eigen PRIVATE osqp::osqpstatic ${CMAKE_DL_LIBS})
target_include_directories(osqp-cpp PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>")
message(STATUS "osqp-cpp: Added osqp-cpp library.")

# Build and register tests.
if (OSQP-CPP_BUILD_TESTS)
  message(STATUS "osqp-cpp: Adding osqp-cpp tests...")
  add_executable(osqp_test test/osqp++_test.cc)
  target_link_libraries(osqp_test gtest gmock gtest_main absl::status absl::span osqp-cpp)
  gtest_discover_tests(osqp_test)
  message(STATUS "osqp-cpp: Added osqp-cpp tests.")
endif()

install(TARGETS ${PROJECT_NAME}
        DESTINATION lib)

install(FILES include/osqp++.h
        DESTINATION include/${PROJECT_NAME})
