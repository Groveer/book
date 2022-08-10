# 基础用法

创建 CMake 项目大概需要以下几步：

1. 首先需要一个目录当中工作空间，也叫项目目录
2. 在这个目录中有一些 c/cpp 文件需要进行编译和链接
3. 此时，需要一个 CMakeLists.txt 文件来进行项目管理。

这里以标准 C 编写一个 demo，打印一下“hello, world！”，编写 main.c 文件，内容为：

```c
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

然后执行下面的语句就可以成功生成二进制：

```shell
cmake -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build
```

## 对 CMakeLists.txt 文件进行解释

此时，在build目录中生成了 cdemo 二进制文件和一些编译信息，然后回过头来对 CMakeLists.txt 进行分析。

```cmake
cmake_minimum_required(VERSION 3.13)
```
cmake_minimum_required 指定需要最低版本的 cmake。当系统中的版本低于指定版本时，它将停止处理项目并且报告错误。


```cmake
if (NOT DEFINED VERSION)
    set(VERSION "1.0.0")
endif()
```
这段代码的意思是：当未指定 VERSION 这个变量时，将自动指定 VERSION 变量的值为“1.0.0”。在使用 cmake 命名时，-D后接变量名表示定义某个变量，具体语法为：-D<var>=<value>，这里指定版本号是为了添加宏定义以方便代码中指定二进制版本号。

* 有时候将一些变量定义为 YES/NO 或者 ON/OFF 会有意想不到的结果。


```cmake
project(cdemo
    LANGUAGES C
    HOMEPAGE_URL https://github.com/Groveer/cdemo
    DESCRIPTION "c program demo."
    VERSION ${VERSION})
```
一个工程项目必须使用 project 方法来描述工程信息，其中，工程名称必须指定，其他为可选项。具体语法为：
```cmake
project(<PROJECT-NAME> [<language-name>...])
project(<PROJECT-NAME>
        [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [DESCRIPTION <project-description-string>]
        [HOMEPAGE_URL <url-string>]
        [LANGUAGES <language-name>...])
```


```cmake
include(GNUInstallDirs)
```
包含 GNUInstallDirs 后，cmake 将会提供一些 GNU 标准安装目录，并且会适配各大发行版，强烈推荐使用此方法进行安装。参考[官方文档](https://cmake.org/cmake/help/latest/module/GNUInstallDirs.html?highlight=gnuinstalldirs)。


```cmake
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
```
这两行代码设置 C/C++ 标准，建议 C 标准用 11 以上，C++ 标准使用 17 以上，若指定的标准，当前编译器不支持，并不会报错，而是会降低标准，只有设置了 CMAKE_C_STANDARD_REQUIRED 变量才会进行报错，而不是自动降低标准。


```cmake
set(CMAKE_INCLUDE_CURRENT_DIR ON)
```
将当前路径包含在 include 路径中，这样在搜索头文件时，可以在当前路径进行搜索，当前路径指的是 CMakeLists.txt 文件所在的路径。


```cmake
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC -Wall -Wextra")
```
指定编译器的编译参数：
1. -fPIC 会生成位置无关的代码，当在开发库的过程中，务必加上此参数。
2. -Wall 会打开绝大多数的编译警告。
3. -Wextra 则会打开一些 -Wall 无法打开的编译警告，所以与 -Wall一起配合使用更好。


```cmake
set(CMAKE_SHARED_LINKER_FLAGS ${CMAKE_SHARED_LINKER_FLAGS} "-Wl,--as-needed")
```
设置链接器的链接参数，该参数表示仅链接自己所需的库。默认情况下，如果用户链接了程序本身并不需要的库，链接器会一起链接起来，会明显拖慢程序的启动过程。


```cmake
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```
设置该变量后，在执行 cmake 命令后，会在编译目录输出一个 json 形式的编译命令信息，该信息方便 clangd 或 ccls 这种 lsp 语言服务器使用。


```cmake
if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set(CMAKE_INSTALL_PREFIX /usr)
endif ()
```
在进行 install 过程中，设置了安装路径前缀，就可以很方便的指定安装路径，而不必每次都指定全路径。这段代码的意思是，如果将安装路径设置为默认值，则指定安装路径为 “/usr”。


```cmake
if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O1 -g -fsanitize=address -fno-omit-frame-pointer")
else ()
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Ofast")
endif ()
```
这里根据不同的编译类型，添加不同的编译参数，通常人们希望 Debug 类型并不需要进行很大的优化，而且需要排查哪些地方有内存泄漏；而在 Release 类型下，通常是希望尽可能的优化。


```cmake
find_package(PkgConfig REQUIRED)
pkg_search_module(XCB REQUIRED xcb)
```
在 cmake 中，有一套规则来查找程序所需要的库：使用 find_package 将在 /usr/lib/cmake 目录下查找对应名称的模块，以 .cmake 为后缀，然后使用 <name>_INCLUDE_DIRS 来指定头文件路径，使用 <name>_LIBRARY_DIRS 来指定链接库路径。

但是 PkgConfig 却是各例外，它并没有 .cmake 文件，而是 cmake 的内置模块，目的是为了兼容 pkg-config 管理的包，当使用 “find_package(PkgConfig REQUIRED)” 后，就可以使用 “pkg_search_module” 方法来查找 pkg-config 管理的库，并且使用方式与 find_package 保持一致，也是使用 <name>_INCLUDE_DIRS 和 <name>_LIBRARY_DIRS 变量。


```cmake
file(GLOB_RECURSE SRCS
    "*.c"
)
```
该方法使用通配符的方式定义了一个变量 SRCS ，它会查找当前目录下的所有 .c 文件，并且将他们组成文件列表，将在后面提供给 add_executable 进行使用。使用该方法的好处是，不必每次增删文件都进行修改 CMakeList.txt 文件，只需要重新执行 cmake 命令即可。


```cmake
set(BIN_NAME ${PROJECT_NAME})

add_executable(${BIN_NAME}
    ${SRCS})

target_compile_definitions(${BIN_NAME} PRIVATE VERSION="${CMAKE_PROJECT_VERSION}")

target_include_directories(${BIN_NAME} PUBLIC
)

target_link_libraries(${BIN_NAME} PRIVATE
)
```
这段代码是生成二进制最关键的代码，它指定了要生成二进制的名称，通过哪些源文件来生成，添加了哪些宏，从哪些路径查找头文件，链接了哪些库等等。

对于宏定义，建议使用 target_compile_definitions 方法来指定具体的二进制名称，而不是使用add_compile_definitions 对所有二进制添加宏定义。

在一个 CMakeLists.txt 中，可以添加多个 add_executable 或 add_library 来生成多个可执行文件或库。一般来说，每个 CMakeLists.txt 中生成一个主程序和一个单元测试程序即可，多个子项目可使用子目录的方式进行构建。

```cmake
install(TARGETS ${BIN_NAME} DESTINATION ${CMAKE_INSTALL_BINDIR})
```
install 方式安装目标文件或目录到指定的目录，其语法如下：
```
install(TARGETS <target>... [...])
install(IMPORTED_RUNTIME_ARTIFACTS <target>... [...])
install({FILES | PROGRAMS} <file>... [...])
install(DIRECTORY <dir>... [...])
install(SCRIPT <file> [...])
install(CODE <code> [...])
install(EXPORT <export-name> [...])
install(RUNTIME_DEPENDENCY_SET <set-name> [...])
```
我们最常用的一般是3个： TARGETS（编译出来的目标二进制）、FILES（指定路径的文件，通常是配置文件或服务文件）、DIRECTORY（一般是配置文件较多时使用）。

---

至此，一个简单的 cmake 项目就大功告成了，下篇文章将介绍一些 cmake 的高级用法以及一些比较优雅的用法。

