cmake_minimum_required(VERSION 3.6)

function(halide_project name folder)
  add_executable("${name}" ${ARGN})
  target_link_libraries("${name}" PRIVATE ${CMAKE_SOURCE_DIR}/packages/halide/WIN64/$<CONFIGURATION>/Halide.lib)
  target_link_libraries("${name}" PRIVATE ${CMAKE_SOURCE_DIR}/packages/libpng.1.6.23.1/build/native/lib/x64/v140/dynamic/$<CONFIGURATION>/libpng16.lib)
  target_link_libraries("${name}" PRIVATE ${CMAKE_SOURCE_DIR}/packages/OpenCL.lib)
  target_include_directories("${name}" PRIVATE "${CMAKE_SOURCE_DIR}/packages/libpng.1.6.23.1/build/native/include")
  target_include_directories("${name}" PRIVATE "${CMAKE_SOURCE_DIR}/packages/libpng/include")
  target_include_directories("${name}" PRIVATE "${CMAKE_SOURCE_DIR}/packages/halide/WIN64/include")
  target_include_directories("${name}" PRIVATE "${CMAKE_SOURCE_DIR}/packages/halide/WIN64/tools")
  set_target_properties("${name}" PROPERTIES FOLDER "${folder}")
  if (MSVC)
    set_target_properties("${name}" PROPERTIES LINK_FLAGS "/STACK:8388608,1048576")
    target_compile_definitions("${name}" PRIVATE _CRT_SECURE_NO_WARNINGS)
    target_link_libraries("${name}" PRIVATE Kernel32)
  endif()
  add_custom_command(TARGET ${name} POST_BUILD        # Adds a post-build event to MyTest
    COMMAND ${CMAKE_COMMAND} -E copy_if_different  # which executes "cmake - E copy_if_different..."
        "${CMAKE_SOURCE_DIR}/packages/halide/WIN64/$<CONFIGURATION>/Halide.dll"      # <--this is in-file
        $<TARGET_FILE_DIR:${name}>)                 # <--this is out-file path
  add_custom_command(TARGET ${name} POST_BUILD        # Adds a post-build event to MyTest
    COMMAND ${CMAKE_COMMAND} -E copy_if_different  # which executes "cmake - E copy_if_different..."
        "${CMAKE_SOURCE_DIR}/packages/libpng.redist.1.6.23.1/build/native/bin/x64/v140/dynamic/$<CONFIGURATION>/libpng16.dll"      # <--this is in-file
        $<TARGET_FILE_DIR:${name}>)                 # <--this is out-file path
  add_custom_command(TARGET ${name} POST_BUILD        # Adds a post-build event to MyTest
    COMMAND ${CMAKE_COMMAND} -E copy_if_different  # which executes "cmake - E copy_if_different..."
        "${CMAKE_SOURCE_DIR}/packages/zlib.v140.windesktop.msvcstl.dyn.rt-dyn.1.2.8.8/lib/native/v140/windesktop/msvcstl/dyn/rt-dyn/x64/$<CONFIGURATION>/zlib.dll"      # <--this is in-file
        $<TARGET_FILE_DIR:${name}>)                 # <--this is out-file path

  add_custom_command(TARGET ${name} POST_BUILD        # Adds a post-build event to MyTest
    COMMAND ${CMAKE_COMMAND} -E copy_directory  # which executes "cmake - E copy_if_different..."
        "${CMAKE_SOURCE_DIR}/data"      # <--this is in-file
        $<TARGET_FILE_DIR:${name}>)                 # <--this is out-file path
endfunction(halide_project)

# Generator
halide_project(halide_test "apps" halide_test.cpp)
set(halide_test_h "${CMAKE_CURRENT_BINARY_DIR}/halide_test.h")
set(halide_test_lib "${CMAKE_CURRENT_BINARY_DIR}/halide_test${CMAKE_STATIC_LIBRARY_SUFFIX}")
# Final executable
#add_executable(blur_test test.cpp ${halide_blur_h})
#target_link_libraries(blur_test PRIVATE "${halide_blur_lib}")
#target_include_directories(blur_test PRIVATE "${CMAKE_CURRENT_BINARY_DIR}")
if (NOT WIN32)
  target_link_libraries(blur_test PRIVATE dl pthread)
endif()

if (NOT MSVC)
  target_compile_options(blur_test PRIVATE "-std=c++11")
  check_cxx_compiler_flag("-Wno-cast-qual" SUPPORTS_NO_CAST_QUAL)
  if (SUPPORTS_NO_CAST_QUAL)
    target_compile_options(blur_test PRIVATE "-Wno-cast-qual")
  endif()
  target_compile_options(blur_test PRIVATE "-std=c++11" "-msse2")
  if (OPENMP_FOUND)
    target_compile_options(blur_test PRIVATE ${OpenMP_CXX_FLAGS})
    target_link_libraries(blur_test PRIVATE ${OpenMP_CXX_FLAGS})
  else()
    target_compile_options(blur_test PRIVATE "-Wno-unknown-pragmas")
  endif()
endif()


# FIXME: Cannot use halide_add_generator_dependency() because
# halide_blur doesn't handle the commandline args passed.
#add_custom_command(OUTPUT "${halide_test_h}" "${halide_test_lib}"
#                   COMMAND halide_test
#                   WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
#                   COMMENT "Generating halide_test"
#                  )
