# 高级用法

使用 CMake 创建一个简单的基于 Qt 的开发库，并且逐步添加单元测试和文档示例（doxygen）。

## 一个简单的基于 Qt 的开发库

首先基于上一章的基础用法创建一个简单的 CMakeLists.txt，并且创建两个目录：include、src，其中include 目录放置 test.h，src 目录放置 test.cpp 文件

```cmake
cmake_minimum_required(VERSION 3.13)

# If do't define version number, specify the version number
if (NOT DEFINED VERSION)
    set(VERSION "1.0.0")
endif()

project(qtdemo
    LANGUAGES CXX
    HOMEPAGE_URL https://github.com/Groveer/qtdemo
    DESCRIPTION "qt library template"
    VERSION ${VERSION})

# specify install dir
include(GNUInstallDirs)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED on)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -Wall -Wextra")
set(CMAKE_SHARED_LINKER_FLAGS ${CMAKE_SHARED_LINKER_FLAGS} "-Wl,--as-needed")
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Install settings
if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set(CMAKE_INSTALL_PREFIX /usr)
endif ()

if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O1 -g -fsanitize=address -fno-omit-frame-pointer")+
else ()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Ofast")
endif ()

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Core)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Core)
# find_package(PkgConfig REQUIRED)
# pkg_search_module(XCB REQUIRED xcb)

file(GLOB_RECURSE INCLUDE_FILES "include/*.h")
file(GLOB_RECURSE SRCS
    "src/*.h"
    "src/*.cpp"
)

set(BIN_NAME ${PROJECT_NAME})

add_library(${BIN_NAME} SHARED
    ${INCLUDE_FILES}
    ${SRCS})

set_target_properties(${BIN_NAME} PROPERTIES
    VERSION ${CMAKE_PROJECT_VERSION}
    SOVERSION ${CMAKE_PROJECT_VERSION_MAJOR})

target_compile_definitions(${BIN_NAME} PRIVATE VERSION="${CMAKE_PROJECT_VERSION}")

target_include_directories(${BIN_NAME} PUBLIC
    Qt${QT_VERSION_MAJOR}::Core
)

target_link_libraries(${BIN_NAME} PRIVATE
    Qt${QT_VERSION_MAJOR}::Core
)

install(FILES ${INCLUDE_FILES} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${BIN_NAME})
install(TARGETS ${BIN_NAME} DESTINATION ${CMAKE_INSTALL_LIBDIR})
```

与基础用法相同的部分，不做过多介绍，主要说明一下不同的部分：

1. 如果是 C 项目，编译参数为：CMAKE\_C\_FLAGS，而 C++ 是 CMAKE\_CXX\_FLAGS
2. 设置了两个 Qt 专用的变量：CMAKE\_AUTOMOC、CMAKE\_AUTORCC，一个是元编译器，一个是资源编译。
3. 使用 QT\_VERSION\_MAJOR 变量来指定 Qt 的版本，该方法可以兼容 Qt6、Qt5。
4. 使用 add\_library 来生成 .so 库，而非 add\_executable。
5. 使用了 set\_target\_properties 方法来设置库的属性，主要是指定了它的版本号，用来在编译时生成带版本号的 .so 文件。

从这个简单的例子可以看出，相对于基础用法，我们仅仅是加了一些对于 Qt 的支持，并没有新增很多东西。

编译此项目的命令：

```
cmake -B build -DCMAKE_BUILD_TYPE=Debug
```

```
cmake --build build
```

## 添加单元测试程序

基于上面的例子，在项目根目录再创建一个文件夹：tests，用于放置单元测试有关的源文件

那么 CMakeLists.txt 中则需要添加以下内容：

```cmake
if (UNITTEST)
    enable_testing()
    file(GLOB_RECURSE TEST_FILES "tests/*.cpp")
    add_executable(ut-${BIN_NAME}
        ${INCLUDE_FILES}
        ${SRCS}
        ${TEST_FILES})

    if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        target_compile_options(ut-${BIN_NAME} PRIVATE -fprofile-instr-generate -ftest-coverage)
    endif()
    if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        target_compile_options(ut-${BIN_NAME} PRIVATE -fprofile-arcs -ftest-coverage)
    endif()

    target_include_directories(ut-${BIN_NAME} PUBLIC
        Qt${QT_VERSION_MAJOR}::Core
    )

    target_link_libraries(ut-${BIN_NAME} PRIVATE
        Qt${QT_VERSION_MAJOR}::Core
        -lpthread
        -lgcov
        -lgtest
    )

    add_test(NAME ut-${BIN_NAME} COMMAND ut-${BIN_NAME})
endif ()
```

而且，我们希望项目仅在 Debug 模式中编译单元测试，而其他模式不必编译，此时就需要一个变量进行控制：UNITTEST，这个变量需要在上面的编译类型判断中进行设置：

```cmake
if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O1 -g -fsanitize=address -fno-omit-frame-pointer")+
else ()
```

改为：

```cmake
if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O1 -g -fsanitize=address -fno-omit-frame-pointer")+
    set(UNITTEST ON)
else ()
```

此时，我们就实现了在 Debug 模式添加了单元测试，在 Release 模式中不要编译单元测试的需求。



在添加单元测试的代码中，着重讲一下之前未见过的函数：

```cmake
enable_testing()
```

这个方法是为了告诉 cmake，该项目启用了单元测试，其是为了配合后面的方法进行使用：

```cmake
add_test(NAME ut-${BIN_NAME} COMMAND ut-${BIN_NAME})
```

这里添加了一条测试用例，告诉 cmake 这个用例的名称，以及要执行的命令。



```cmake
if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    target_compile_options(ut-${BIN_NAME} PRIVATE -fprofile-instr-generate -ftest-coverage)
endif()
if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    target_compile_options(ut-${BIN_NAME} PRIVATE -fprofile-arcs -ftest-coverage)
endif()
```

这段代码是为了适配不同的编译器，但其目的是相同的，都是为了添加编译选项，用以支持生产单元测试覆盖率的内容。

[这里](https://cmake.org/cmake/help/latest/variable/CMAKE\_LANG\_COMPILER\_ID.html#variable:CMAKE\_%3CLANG%3E\_COMPILER\_ID)罗列了一些编译器的ID。

此时，执行下面的命令就可以编译单元测试并运行单元测试了：

```shell
cmake -B build -DCMAKE_BUILD_TYPE=Debug
```

```
cmake --build build
```

```
ctest --test-dir build -VV
```

## 利用 doxygen 以生成开发文档

为了将文档和代码进行隔离，在项目根目录创建 docs 目录，并在该目录创建 test.dox 文件，关于 dox 文件如何编写，不在本篇文章的范围内，故此不做过多说明。

```cmake
find_package(Doxygen)

if (NOT DEFINED BUILD_DOCS)
    set (BUILD_DOCS ON)
endif ()

if (BUILD_DOCS AND DOXYGEN_FOUND)
    set (QCH_INSTALL_DESTINATION ${CMAKE_INSTALL_PREFIX}/share/qt/doc CACHE STRING "QCH install location")
    set (DOXYGEN_GENERATE_HTML "YES" CACHE STRING "Doxygen HTML output")
    set (DOXYGEN_GENERATE_XML "NO" CACHE STRING "Doxygen XML output")
    set (DOXYGEN_GENERATE_QHP "YES" CACHE STRING "Doxygen QHP output")
    set (DOXYGEN_FILE_PATTERNS *.cpp *.h *.md *.dox CACHE STRING "Doxygen File Patterns")
    set (DOXYGEN_PROJECT_NUMBER ${CMAKE_PROJECT_VERSION} CACHE STRING "") # Should be the same as this project is using.
    set (DOXYGEN_EXTRACT_STATIC YES)
    set (DOXYGEN_OUTPUT_LANGUAGE "Chinese")
    set (DOXYGEN_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/docs/)
    set (DOXYGEN_QHG_LOCATION "qhelpgenerator")
    set (DOXYGEN_QHP_NAMESPACE "org.deepin.dde.${CMAKE_PROJECT_NAME}")
    set (DOXYGEN_QCH_FILE "${CMAKE_PROJECT_NAME}.qch")
    set (DOXYGEN_QHP_VIRTUAL_FOLDER ${CMAKE_PROJECT_NAME})
    set (DOXYGEN_HTML_EXTRA_STYLESHEET "" CACHE STRING "Doxygen custom stylesheet for HTML output")
    set (DOXYGEN_TAGFILES "qtcore.tags=qthelp://doc.qt.io/qt-5/" CACHE STRING "Doxygen tag files")
    set (DOXYGEN_MACRO_EXPANSION "YES")
    set (DOXYGEN_EXPAND_ONLY_PREDEF "YES")
    set (DOXYGEN_PREDEFINED
        "\"DDEMO_BEGIN_NAMESPACE=namespace Dtk { namespace Demo {\""
        "\"DDEMO_END_NAMESPACE=}}\""
        "\"DDEMO_USE_NAMESPACE=using namespace Dtk::Demo;\""
    )

    doxygen_add_docs (doxygen
        ${CMAKE_CURRENT_SOURCE_DIR}/include
        ${CMAKE_CURRENT_SOURCE_DIR}/docs
        ALL
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        COMMENT "Generate documentation via Doxygen"
    )

    install (FILES ${CMAKE_CURRENT_BINARY_DIR}/docs/html/${CMAKE_PROJECT_NAME}.qch DESTINATION ${QCH_INSTALL_DESTINATION})
endif ()
```

看起来这里的变量设置的有点多，其实这些变量在官方文档都有说明，只是在变量前面加上 CMAKE 前缀，就可以被 cmake 识别，官方文档参考这里。

