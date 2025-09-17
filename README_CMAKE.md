# CMake Build Instructions for s21_matrix_oop

This project includes a `CMakeLists.txt` file for building the s21_matrix_oop library using CMake.

## Prerequisites

- CMake 3.20 or higher
- GCC compiler (recommended)
- clang-format (optional, for code formatting)

## Quick Start

```bash
# Configure the project
mkdir build
cd build
cmake ..

# Build the project
cmake --build .

# Run tests (if available)
make run_tests
# or
ctest --verbose
```

## Available Targets

- **all** (default): Builds all targets including library and tests
- **s21_matrix_oop**: Builds the static library (output: `lib/libs21_matrix_oop.a`)
- **s21_matrix_tests**: Builds and links the test executable (if test files exist)
- **run_tests**: Runs all tests using CTest
- **check**: Alias for run_tests (Makefile compatibility)
- **test**: Dummy target when no tests are available
- **format**: Formats all source code using clang-format (Google Style)
- **check-format**: Checks code formatting without modifying files
- **coverage**: Placeholder for coverage analysis
- **clean**: Removes build artifacts

## Project Structure

```
CPP1_Matrix/
├── CMakeLists.txt          # Main CMake configuration
├── src/                    # Source files (.cpp, .h)
│   ├── s21_matrix_oop.h   # Main header file
│   ├── s21_matrix_oop.cpp # Implementation files
│   └── .clang-format      # Copied automatically from materials/linters/
├── tests/                  # Test files (.cpp, .h)
│   └── test_*.cpp         # GTest test files
├── materials/linters/      # Code style configuration
│   └── .clang-format      # Google style configuration
└── build/                  # Build directory (created by user)
    ├── lib/               # Output libraries
    └── s21_matrix_tests   # Test executable
```

## Build Configuration

The CMakeLists.txt automatically configures:

- **C++20 standard** as required
- **GCC compiler preference** with warning for other compilers
- **Google Code Style** integration with clang-format
- **GTest framework** (downloaded automatically if not found)
- **Static library creation** matching Makefile requirements
- **Comprehensive compiler warnings** (-Wall -Wextra -Werror)

## Usage Examples

### Building in Debug mode
```bash
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Debug
cmake --build .
```

### Building in Release mode
```bash
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build .
```

### Running specific tests
```bash
cd build
./s21_matrix_tests
```

### Code formatting
```bash
cd build
make format          # Format all source files
make check-format    # Check formatting without changes
```

## Integration with IDEs

This CMakeLists.txt is compatible with:
- CLion
- Visual Studio Code with CMake Tools extension
- Qt Creator
- Any IDE with CMake support

## Makefile Compatibility

The CMakeLists.txt provides equivalent functionality to the required Makefile targets:
- `all` → Default CMake target
- `clean` → `make clean` or `cmake --build . --target clean`
- `test` → `make run_tests` or `ctest`
- `s21_matrix_oop.a` → `make s21_matrix_oop` (output: `lib/libs21_matrix_oop.a`)

## Notes

- GTest is automatically downloaded and built if not found on the system
- The clang-format configuration is automatically copied from `materials/linters/.clang-format`
- All build outputs are organized in the `build/` directory
- The library follows the exact naming convention required: `s21_matrix_oop.a`