cmake_minimum_required(VERSION 3.20)

# Project name and version
project(s21_matrix_oop VERSION 1.0.0 LANGUAGES CXX)

# Set C++20 standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Prefer GCC compiler
if(NOT CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    message(WARNING "Project is designed for GCC compiler. Current compiler: ${CMAKE_CXX_COMPILER_ID}")
endif()

# Set build type to Release if not specified
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

# Compiler flags
set(CMAKE_CXX_FLAGS "-Wall -Wextra -Werror")
set(CMAKE_CXX_FLAGS_DEBUG "-g -O0")
set(CMAKE_CXX_FLAGS_RELEASE "-O2 -DNDEBUG")

# Find clang-format for code formatting (Google Style)
find_program(CLANG_FORMAT_EXECUTABLE
    NAMES clang-format
    DOC "Path to clang-format executable"
)

# Source files location
set(SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(TEST_DIR ${CMAKE_CURRENT_SOURCE_DIR}/tests)

# Collect source files from src directory
file(GLOB_RECURSE SOURCES
    "${SRC_DIR}/*.cpp"
    "${SRC_DIR}/*.cc"
)

file(GLOB_RECURSE HEADERS
    "${SRC_DIR}/*.h"
    "${SRC_DIR}/*.hpp"
)

# Create static library target
if(SOURCES)
    add_library(s21_matrix_oop STATIC ${SOURCES} ${HEADERS})
    
    # Set include directories
    target_include_directories(s21_matrix_oop PUBLIC ${SRC_DIR})
    
    # Set output name to match requirement (s21_matrix_oop.a)
    set_target_properties(s21_matrix_oop PROPERTIES
        OUTPUT_NAME s21_matrix_oop
        ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
    )
else()
    message(STATUS "No source files found in ${SRC_DIR}. Library target not created.")
    # Create a dummy target for the case when src is empty
    add_custom_target(s21_matrix_oop
        COMMAND ${CMAKE_COMMAND} -E echo "No source files to compile. Please add source files to ${SRC_DIR}"
    )
endif()

# Find and setup GTest
find_package(GTest QUIET)

if(NOT GTest_FOUND)
    # Download and build GTest if not found
    include(FetchContent)
    FetchContent_Declare(
        googletest
        GIT_REPOSITORY https://github.com/google/googletest.git
        GIT_TAG        v1.14.0
    )
    
    # For Windows: Prevent overriding the parent project's compiler/linker settings
    set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
    
    FetchContent_MakeAvailable(googletest)
endif()

# Test files
file(GLOB_RECURSE TEST_SOURCES
    "${TEST_DIR}/*.cpp"
    "${TEST_DIR}/*.cc"
)

# Create test executable if test files exist
if(TEST_SOURCES AND SOURCES)
    add_executable(s21_matrix_tests ${TEST_SOURCES})
    
    target_link_libraries(s21_matrix_tests
        s21_matrix_oop
        GTest::gtest
        GTest::gtest_main
    )
    
    target_include_directories(s21_matrix_tests PRIVATE ${SRC_DIR})
    
    # Enable testing
    enable_testing()
    add_test(NAME MatrixTests COMMAND s21_matrix_tests)
    
elseif(TEST_SOURCES AND NOT SOURCES)
    message(STATUS "Test files found but no source files. Please add source files to ${SRC_DIR}")
else()
    message(STATUS "No test files found in ${TEST_DIR}. Test target not created.")
    # Create a dummy test target
    add_custom_target(test
        COMMAND ${CMAKE_COMMAND} -E echo "No test files found. Please add test files to ${TEST_DIR}"
    )
endif()

# Custom targets to match Makefile requirements

# Note: 'all' target is provided by CMake automatically and depends on all normal targets
# 'clean' target is provided by CMake automatically

# 'test' target
if(TARGET s21_matrix_tests)
    add_custom_target(run_tests
        COMMAND ${CMAKE_CTEST_COMMAND} --verbose
        DEPENDS s21_matrix_tests
        COMMENT "Running all tests"
    )
    # Alias for compatibility with Makefile
    add_custom_target(check DEPENDS run_tests)
else()
    add_custom_target(run_tests
        COMMAND ${CMAKE_COMMAND} -E echo "No tests to run. Please add test files to ${TEST_DIR}"
    )
    add_custom_target(check DEPENDS run_tests)
endif()

# Code formatting target using clang-format (Google Style)
if(CLANG_FORMAT_EXECUTABLE)
    # Copy clang-format config to src directory if it doesn't exist
    if(NOT EXISTS ${SRC_DIR}/.clang-format)
        configure_file(
            ${CMAKE_CURRENT_SOURCE_DIR}/materials/linters/.clang-format
            ${SRC_DIR}/.clang-format
            COPYONLY
        )
    endif()
    
    # Get all source and header files
    file(GLOB_RECURSE ALL_SOURCE_FILES
        "${SRC_DIR}/*.cpp"
        "${SRC_DIR}/*.cc"
        "${SRC_DIR}/*.h"
        "${SRC_DIR}/*.hpp"
        "${TEST_DIR}/*.cpp"
        "${TEST_DIR}/*.cc"
        "${TEST_DIR}/*.h"
        "${TEST_DIR}/*.hpp"
    )
    
    # Only create format targets if there are files to format
    if(ALL_SOURCE_FILES)
        # Format target
        add_custom_target(format
            COMMAND ${CLANG_FORMAT_EXECUTABLE} -i ${ALL_SOURCE_FILES}
            COMMENT "Formatting code with clang-format (Google Style)"
        )
        
        # Check format target
        add_custom_target(check-format
            COMMAND ${CLANG_FORMAT_EXECUTABLE} -n ${ALL_SOURCE_FILES}
            COMMENT "Checking code format with clang-format (Google Style)"
        )
    else()
        add_custom_target(format
            COMMAND ${CMAKE_COMMAND} -E echo "No source files found for formatting"
            COMMENT "No source files to format"
        )
        add_custom_target(check-format
            COMMAND ${CMAKE_COMMAND} -E echo "No source files found for format checking"
            COMMENT "No source files to check format"
        )
    endif()
else()
    message(STATUS "clang-format not found. Format targets will not be available.")
    add_custom_target(format
        COMMAND ${CMAKE_COMMAND} -E echo "clang-format not found. Please install clang-format."
    )
    add_custom_target(check-format
        COMMAND ${CMAKE_COMMAND} -E echo "clang-format not found. Please install clang-format."
    )
endif()

# Coverage target (optional, requires gcov)
add_custom_target(coverage
    COMMAND ${CMAKE_COMMAND} -E echo "Coverage analysis requires gcov and lcov tools"
    COMMENT "Generate code coverage report"
)

# Installation rules
if(SOURCES)
    install(TARGETS s21_matrix_oop
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin
    )
    
    install(FILES ${HEADERS}
        DESTINATION include/s21_matrix_oop
    )
endif()

# Print configuration information
message(STATUS "=== s21_matrix_oop Configuration ===")
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "C++ standard: ${CMAKE_CXX_STANDARD}")
message(STATUS "Compiler: ${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER_VERSION}")
message(STATUS "Source directory: ${SRC_DIR}")
message(STATUS "Test directory: ${TEST_DIR}")
if(SOURCES)
    message(STATUS "Library target: s21_matrix_oop")
else()
    message(STATUS "Library target: Not created (no source files)")
endif()
if(TARGET s21_matrix_tests)
    message(STATUS "Test target: s21_matrix_tests")
else()
    message(STATUS "Test target: Not created (no test files)")
endif()
message(STATUS "clang-format: ${CLANG_FORMAT_EXECUTABLE}")
message(STATUS "================================")