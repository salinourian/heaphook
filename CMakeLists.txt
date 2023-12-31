cmake_minimum_required(VERSION 3.8)
project(heaphook)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
# uncomment the following section in order to fill in
# further dependencies manually.

find_package(tlsf REQUIRED)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # comment the line when a copyright and license is added to all source files
  set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # comment the line when this package is in a git repo and when
  # a copyright and license is added to all source files
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

add_executable(app src/app.cpp)

include(CheckSymbolExists)
check_symbol_exists(mallinfo2 malloc.h HAVE_MALLINFO2)

add_library(preloaded_heaptrack SHARED src/preloaded.cpp)
if(HAVE_MALLINFO2)
  target_compile_definitions(preloaded_heaptrack
    PRIVATE
      HAVE_MALLINFO2
  )
endif()

add_library(preloaded_tlsf SHARED src/preloaded_tlsf.cpp)
target_link_libraries(preloaded_tlsf PRIVATE tlsf::tlsf)
if(HAVE_MALLINFO2)
  target_compile_definitions(preloaded_tlsf
    PRIVATE
      HAVE_MALLINFO2
  )
endif()

install(TARGETS preloaded_heaptrack preloaded_tlsf DESTINATION lib)
install(TARGETS app DESTINATION bin)

ament_package()
