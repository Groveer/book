# 基础用法

创建 CMake 项目大概需要以下几步：

1. 首先需要一个目录当中工作空间，也叫项目目录
2. 在这个目录中有一些 c/cpp 文件需要进行编译和链接
3. 此时，需要一个 CMakeLists.txt 文件来进行项目管理。

这里以标准 C 编写一个 demo，打印一下“hello, world！”，编写 main.c 文件，内容为：

```clike
#include <stdio.h>

int main(int argc, char *argv[])
{
    printf("hello, world!");
    return 0;
}
```

然后在当前目录编写 CMakeLists.txt，内容为：

```cmake
cmake_minimum_required(VERSION 3.13)

# If do't define version number, specify the version number
if (NOT DEFINED VERSION)
    set(VERSION "1.0.0")
endif()

project(cdemo
    LANGUAGES C
    HOMEPAGE_URL https://github.com/Groveer/cdemo
    DESCRIPTION "c program demo."
    VERSION ${VERSION})

include(GNUInstallDirs)
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED on)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC -Wall -Wextra")
set(CMAKE_SHARED_LINKER_FLAGS ${CMAKE_SHARED_LINKER_FLAGS} "-Wl,--as-needed")
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Install settings
if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set(CMAKE_INSTALL_PREFIX /usr)
endif ()

if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O1 -g -fsanitize=address -fno-omit-frame-pointer")
else ()
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Ofast")
endif ()

find_package(Doxygen)
#find_package(PkgConfig REQUIRED)
#pkg_search_module(XCB REQUIRED xcb)

file(GLOB_RECURSE SRCS
    "*.c"
)

set(BIN_NAME ${PROJECT_NAME})

add_executable(${BIN_NAME}
    ${SRCS})

target_compile_definitions(${BIN_NAME} PRIVATE VERSION="${CMAKE_PROJECT_VERSION}")

target_include_directories(${BIN_NAME} PUBLIC
)

target_link_libraries(${BIN_NAME} PRIVATE
)

install(TARGETS ${BIN_NAME} DESTINATION ${CMAKE_INSTALL_BINDIR})

```

