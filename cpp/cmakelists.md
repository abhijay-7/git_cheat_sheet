# CMakeLists.txt Cheatsheet

## Basic Structure

```cmake
cmake_minimum_required(VERSION 3.16)
project(MyProject VERSION 1.0 LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Add executable
add_executable(myapp main.cpp utils.cpp)
```

## Project Declaration

```cmake
project(MyProject 
    VERSION 1.2.3
    DESCRIPTION "My awesome project"
    LANGUAGES CXX C
)
```

## Variables

### Setting Variables
```cmake
set(SOURCES main.cpp utils.cpp helper.cpp)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")
set(MY_LIBRARY_VERSION 1.0)
```

### Built-in Variables
```cmake
# Directories
${CMAKE_SOURCE_DIR}         # Top-level source directory
${CMAKE_BINARY_DIR}         # Top-level build directory
${CMAKE_CURRENT_SOURCE_DIR} # Current source directory
${CMAKE_CURRENT_BINARY_DIR} # Current build directory
${PROJECT_SOURCE_DIR}       # Project root source directory
${PROJECT_BINARY_DIR}       # Project root build directory

# System info
${CMAKE_SYSTEM_NAME}        # OS name (Linux, Windows, Darwin)
${CMAKE_SYSTEM_VERSION}     # OS version
${CMAKE_HOST_SYSTEM_NAME}   # Build machine OS

# Compiler info
${CMAKE_CXX_COMPILER}       # C++ compiler path
${CMAKE_C_COMPILER}         # C compiler path
${CMAKE_BUILD_TYPE}         # Debug, Release, etc.
```

## Targets

### Executables
```cmake
add_executable(myapp main.cpp)
add_executable(myapp ${SOURCES})

# Executable with multiple sources
add_executable(calculator
    main.cpp
    calculator.cpp
    operations.cpp
)
```

### Libraries
```cmake
# Static library
add_library(mylib STATIC lib.cpp lib.h)

# Shared library
add_library(mylib SHARED lib.cpp lib.h)

# Header-only library (INTERFACE)
add_library(mylib INTERFACE)
target_include_directories(mylib INTERFACE include/)
```

### Custom Targets
```cmake
add_custom_target(docs
    COMMAND doxygen Doxyfile
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    COMMENT "Generating documentation"
)
```

## Linking and Dependencies

### Linking Libraries
```cmake
# Link libraries to target
target_link_libraries(myapp mylib)
target_link_libraries(myapp pthread m)

# Modern CMake approach with visibility
target_link_libraries(myapp 
    PRIVATE mylib
    PUBLIC otherlib
)
```

### Include Directories
```cmake
# Old way (avoid)
include_directories(include/)

# Modern way
target_include_directories(myapp 
    PRIVATE src/
    PUBLIC include/
    INTERFACE interface/
)
```

### Compile Features and Options
```cmake
# Compile features
target_compile_features(myapp PRIVATE cxx_std_17)

# Compile options
target_compile_options(myapp PRIVATE -Wall -Wextra)

# Compile definitions
target_compile_definitions(myapp PRIVATE DEBUG=1)
```

## Finding Packages

### find_package
```cmake
# Find system packages
find_package(Threads REQUIRED)
find_package(OpenCV REQUIRED)
find_package(Boost REQUIRED COMPONENTS system filesystem)

# Use found packages
target_link_libraries(myapp 
    Threads::Threads
    ${OpenCV_LIBS}
    Boost::system
    Boost::filesystem
)
```

### pkg_config
```cmake
find_package(PkgConfig REQUIRED)
pkg_check_modules(GTK3 REQUIRED gtk+-3.0)

target_link_libraries(myapp ${GTK3_LIBRARIES})
target_include_directories(myapp PRIVATE ${GTK3_INCLUDE_DIRS})
```

## File Operations

### Globbing Files
```cmake
# Glob source files (not recommended for libraries)
file(GLOB SOURCES "src/*.cpp")
file(GLOB_RECURSE ALL_SOURCES "src/*.cpp" "src/*.h")

# Better approach - explicit file lists
set(SOURCES
    src/main.cpp
    src/utils.cpp
    src/helper.cpp
)
```

### File Operations
```cmake
# Copy files
configure_file(config.h.in config.h)
file(COPY assets/ DESTINATION ${CMAKE_BINARY_DIR}/assets/)

# Generate files
file(WRITE ${CMAKE_BINARY_DIR}/version.txt "${PROJECT_VERSION}")
```

## Conditionals and Loops

### Conditionals
```cmake
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_definitions(myapp PRIVATE DEBUG_MODE=1)
elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
    target_compile_definitions(myapp PRIVATE NDEBUG=1)
endif()

# System-specific code
if(WIN32)
    target_sources(myapp PRIVATE windows_specific.cpp)
elseif(UNIX)
    target_sources(myapp PRIVATE unix_specific.cpp)
endif()
```

### Loops
```cmake
set(COMPONENTS core gui network)
foreach(component ${COMPONENTS})
    find_package(Qt5${component} REQUIRED)
    target_link_libraries(myapp Qt5::${component})
endforeach()
```

## Build Types and Configurations

```cmake
# Set default build type
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

# Custom build types
set(CMAKE_CXX_FLAGS_PROFILING "-O2 -g -pg")

# Configuration-specific settings
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_definitions(myapp PRIVATE DEBUG_BUILD)
endif()
```

## Subdirectories and Modules

### Subdirectories
```cmake
# Add subdirectory
add_subdirectory(src)
add_subdirectory(tests)
add_subdirectory(external/fmt)

# Optional subdirectories
if(BUILD_TESTS)
    add_subdirectory(tests)
endif()
```

### Modules
```cmake
# Include custom modules
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")
include(MyCustomModule)
```

## Testing

### CTest
```cmake
enable_testing()

add_executable(test_math test_math.cpp)
target_link_libraries(test_math mylib)

add_test(NAME MathTests COMMAND test_math)
```

### GoogleTest
```cmake
find_package(GTest REQUIRED)

add_executable(unit_tests test_main.cpp test_utils.cpp)
target_link_libraries(unit_tests 
    mylib
    GTest::GTest
    GTest::Main
)

add_test(NAME UnitTests COMMAND unit_tests)
```

## Installation

```cmake
# Install targets
install(TARGETS myapp mylib
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

# Install headers
install(DIRECTORY include/ 
    DESTINATION include
    FILES_MATCHING PATTERN "*.h"
)

# Install files
install(FILES README.md LICENSE 
    DESTINATION share/doc/myapp
)
```

## Complete Example

```cmake
cmake_minimum_required(VERSION 3.16)
project(Calculator 
    VERSION 1.0.0
    DESCRIPTION "A simple calculator application"
    LANGUAGES CXX
)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Options
option(BUILD_TESTS "Build unit tests" ON)
option(BUILD_DOCS "Build documentation" OFF)

# Find dependencies
find_package(Threads REQUIRED)

# Sources
set(LIB_SOURCES
    src/calculator.cpp
    src/operations.cpp
    src/parser.cpp
)

set(APP_SOURCES
    src/main.cpp
)

# Create library
add_library(calclib STATIC ${LIB_SOURCES})

target_include_directories(calclib
    PUBLIC 
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    PRIVATE
        src
)

target_compile_features(calclib PUBLIC cxx_std_17)

# Create executable
add_executable(calculator ${APP_SOURCES})

target_link_libraries(calculator 
    PRIVATE 
        calclib
        Threads::Threads
)

# Compiler-specific options
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(calclib PRIVATE -Wall -Wextra -Wpedantic)
    target_compile_options(calculator PRIVATE -Wall -Wextra -Wpedantic)
endif()

# Build type specific settings
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_definitions(calclib PRIVATE DEBUG_MODE=1)
endif()

# Testing
if(BUILD_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif()

# Documentation
if(BUILD_DOCS)
    find_package(Doxygen)
    if(DOXYGEN_FOUND)
        add_subdirectory(docs)
    endif()
endif()

# Installation
install(TARGETS calculator calclib
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

install(DIRECTORY include/ 
    DESTINATION include
    FILES_MATCHING PATTERN "*.h"
)
```

## Useful Commands

### Running CMake
```bash
# Configure
cmake -B build -S .
cmake -B build -DCMAKE_BUILD_TYPE=Release

# Build
cmake --build build
cmake --build build --parallel 4
cmake --build build --target install

# Test
ctest --test-dir build
ctest --test-dir build --verbose
```

### Generator-specific
```bash
# Unix Makefiles
cmake -G "Unix Makefiles" -B build

# Ninja
cmake -G Ninja -B build

# Visual Studio
cmake -G "Visual Studio 16 2019" -A x64 -B build
```

## Best Practices

1. Use `target_*` commands instead of global commands
2. Specify `PUBLIC`/`PRIVATE`/`INTERFACE` visibility
3. Use modern CMake (3.5+ features)
4. Avoid `file(GLOB)` for source files in libraries
5. Use `find_package` instead of manual library detection
6. Set minimum CMake version appropriately
7. Use generator expressions for conditional logic
8. Organize code into subdirectories
9. Support out-of-source builds
10. Provide installation rules for libraries

## Common Pitfalls

- **Global commands**: Use `target_*` instead of global `include_directories()`
- **In-source builds**: Always build in separate directory
- **Hard-coded paths**: Use CMake variables and generator expressions
- **Missing PRIVATE/PUBLIC**: Always specify link/include visibility
- **Old CMake patterns**: Avoid deprecated commands and variables