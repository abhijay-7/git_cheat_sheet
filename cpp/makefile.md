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