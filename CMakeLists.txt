cmake_minimum_required(VERSION 3.6)

# Global properties
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# Project name
project(cpptemplate)

# Doxygen
find_package(Doxygen)
if(DOXYGEN_FOUND)
  set(DOXYGEN "doxygen")
  if(NOT TARGET ${DOXYGEN})
    add_custom_command(OUTPUT "Doxyfile" COMMAND ${DOXYGEN_EXECUTABLE} "Doxyfile" WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/documents")
    add_custom_target(${DOXYGEN} DEPENDS "Doxyfile")
    set_target_properties(${DOXYGEN} PROPERTIES FOLDER doxygen)
  endif()
endif()

# CMake module path
set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# Compiler features
include(SetCompilerFeatures)
include(SetCompilerWarnings)
include(SetPlatformFeatures)
include(SystemInformation)

# External packages
find_package(Threads REQUIRED)

# Modules
add_subdirectory("modules")

# Link libraries
list(APPEND LINKLIBS Threads::Threads)

# System directories
include_directories(SYSTEM "${CMAKE_CURRENT_SOURCE_DIR}/modules")

# Library
file(GLOB_RECURSE LIB_HEADER_FILES "include/*.h")
file(GLOB_RECURSE LIB_INLINE_FILES "include/*.inl")
file(GLOB_RECURSE LIB_SOURCE_FILES "source/*.cpp")
set_source_files_properties(${LIB_SOURCE_FILES} PROPERTIES COMPILE_FLAGS "${PEDANTIC_COMPILE_FLAGS}")
add_library(cpptemplate ${LIB_HEADER_FILES} ${LIB_INLINE_FILES} ${LIB_SOURCE_FILES})
target_include_directories(cpptemplate PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/include")
target_link_libraries(cpptemplate ${LINKLIBS})
set_target_properties(cpptemplate PROPERTIES FOLDER libraries)
list(APPEND INSTALL_TARGETS cpptemplate)
list(APPEND LINKLIBS cpptemplate)

# Additional module components: benchmarks, examples, plugins, tests, tools and install
if(NOT CPPTEMPLATE_MODULE)

  # Examples
  file(GLOB EXAMPLE_HEADER_FILES "examples/*.h")
  file(GLOB EXAMPLE_INLINE_FILES "examples/*.inl")
  file(GLOB EXAMPLE_SOURCE_FILES RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}/examples" "examples/*.cpp")
  foreach(EXAMPLE_SOURCE_FILE ${EXAMPLE_SOURCE_FILES})
    string(REGEX REPLACE "(.*)\\.cpp" "\\1" EXAMPLE_NAME ${EXAMPLE_SOURCE_FILE})
    set(EXAMPLE_TARGET "cpptemplate-example-${EXAMPLE_NAME}")
    set_source_files_properties(examples/${EXAMPLE_SOURCE_FILE} PROPERTIES COMPILE_FLAGS "${PEDANTIC_COMPILE_FLAGS}")
    add_executable(${EXAMPLE_TARGET} ${EXAMPLE_HEADER_FILES} ${EXAMPLE_INLINE_FILES} examples/${EXAMPLE_SOURCE_FILE})
    target_link_libraries(${EXAMPLE_TARGET} ${LINKLIBS})
    set_target_properties(${EXAMPLE_TARGET} PROPERTIES FOLDER examples)
    list(APPEND INSTALL_TARGETS ${EXAMPLE_TARGET})
    list(APPEND INSTALL_TARGETS_PDB ${EXAMPLE_TARGET})
  endforeach()

  # Benchmarks
  file(GLOB BENCHMARK_HEADER_FILES "performance/*.h")
  file(GLOB BENCHMARK_INLINE_FILES "performance/*.inl")
  file(GLOB BENCHMARK_SOURCE_FILES RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}/performance" "performance/*.cpp")
  foreach(BENCHMARK_SOURCE_FILE ${BENCHMARK_SOURCE_FILES})
    string(REGEX REPLACE "(.*)\\.cpp" "\\1" BENCHMARK_NAME ${BENCHMARK_SOURCE_FILE})
    set(BENCHMARK_TARGET "cpptemplate-performance-${BENCHMARK_NAME}")
    set_source_files_properties(performance/${BENCHMARK_SOURCE_FILE} PROPERTIES COMPILE_FLAGS "${PEDANTIC_COMPILE_FLAGS}")
    add_executable(${BENCHMARK_TARGET} ${BENCHMARK_HEADER_FILES} ${BENCHMARK_INLINE_FILES} performance/${BENCHMARK_SOURCE_FILE})
    target_link_libraries(${BENCHMARK_TARGET} ${LINKLIBS} cppbenchmark)
    set_target_properties(${BENCHMARK_TARGET} PROPERTIES FOLDER performance)
    list(APPEND INSTALL_TARGETS ${BENCHMARK_TARGET})
    list(APPEND INSTALL_TARGETS_PDB ${BENCHMARK_TARGET})
  endforeach()

  # Tests
  file(GLOB TESTS_HEADER_FILES "tests/*.h")
  file(GLOB TESTS_INLINE_FILES "tests/*.inl")
  file(GLOB TESTS_SOURCE_FILES "tests/*.cpp")
  set_source_files_properties(${TESTS_SOURCE_FILES} PROPERTIES COMPILE_FLAGS "${PEDANTIC_COMPILE_FLAGS}")
  add_executable(cpptemplate-tests ${TESTS_HEADER_FILES} ${TESTS_INLINE_FILES} ${TESTS_SOURCE_FILES} ${Catch2})
  target_include_directories(cpptemplate-tests PRIVATE ${Catch2})
  target_link_libraries(cpptemplate-tests ${LINKLIBS})
  set_target_properties(cpptemplate-tests PROPERTIES FOLDER tests)
  list(APPEND INSTALL_TARGETS cpptemplate-tests)
  list(APPEND INSTALL_TARGETS_PDB cpptemplate-tests)

  # CTest
  enable_testing()
  add_test(cpptemplate-tests cpptemplate-tests --durations yes --order lex)

  # Install
  install(TARGETS ${INSTALL_TARGETS}
    RUNTIME DESTINATION "${PROJECT_SOURCE_DIR}/bin"
    LIBRARY DESTINATION "${PROJECT_SOURCE_DIR}/bin"
    ARCHIVE DESTINATION "${PROJECT_SOURCE_DIR}/bin")

  # Install *.pdb files
  if(MSVC)
    foreach(INSTALL_TARGET_PDB ${INSTALL_TARGETS_PDB})
      install(FILES $<TARGET_PDB_FILE:${INSTALL_TARGET_PDB}> DESTINATION "${PROJECT_SOURCE_DIR}/bin")
    endforeach()
  endif()

endif()
