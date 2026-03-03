# CMake 学习收获

## 2026.3.3

### 感悟
总的来说，CMake 本身是一个元构建系统，先根据 CMakeLists.txt 中的规则生成编译代码的配置文件，再借助其它工具(如Make)真正编译代码。

CMake 一个重要的设计理念为：将**可执行文件**(executable)与**库**(libraries)都视为**target**对象。

再为**target**对象设置其属性：编译选项Compile Options, 宏定义Compile Definitions, 头文件路径Include Directories, 链接依赖Link Libraries。

以及用可见性关键字指定各属性的可见性：PRIVATE, PUBLIC, INTERFACE。

这样可以避免过多的全局配置，实现隔离与封装。

### CMake 基本使用方法
```
cmake_minimum_required(VERSION <min>[...<policy_max>] [FATAL_ERROR])
```
`cmake_minimum_required()`用于指定项目所需的 CMake 最低版本。必须在 CMakeLists.txt 的开头指定，先于 `project()`

```
project(<PROJECT-NAME>
        [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [DESCRIPTION <project-description-string>]
        [HOMEPAGE_URL <url-string>]
        [LANGUAGES <language-name>...])
```
`project()`用于指定项目的名称，并存入*PROJECT_NAME*变量中，若在顶层的 CMakeLists.txt 调用，同时还会存入*CMAKE_PROJECT_NAME*中。

```
set(<variable> <value>... [PARENT_SCOPE])
```
`set()`用于设定一个变量至给定的值，若未提供值，则等同于`unset()`。

```
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```
用于指定后续target对象的C++版本

```
add_executable(<name> [WIN32] [MACOSX_BUNDLE]
               [EXCLUDE_FROM_ALL]
               [source1] [source2 ...])
```
`add_executable()`用于创建目标可执行文件并指定名字、所需资源文件。3.11版本后，资源文件也可在后续通过`target_sources()`添加。

```
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [<source>...])
```
`add_library()`用于创建库，并指定这个库所需的资源文件。

`add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL] [SYSTEM])`
`add_subdirectory()`用于为当前项目添加子目录，并处理子目录的 CMakeLists.txt，可以实现大型项目的模块化。

```
target_include_directories(<target> [SYSTEM] [AFTER|BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```
`target_include_directories()`用于为 target 指定头文件包含路径，并为各路径指定访问属性。

```
target_link_libraries(<target>
                      <PRIVATE|PUBLIC|INTERFACE> <item>...
                     [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)
```
`target_link_libraries()`用于指定 target 对象的依赖关系，同时使用可见性关键字进行隔离封装

```
target_compile_definitions(<target>
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```
`target_compile_definitions()`用于为 target 设置预处理器定义，支持多种定义形式，如：`target_compile_definitions(project_1 PRIVATE VERSION="1.0.0")`

```
configure_file(<input> <output>
               [NO_SOURCE_PERMISSIONS | USE_SOURCE_PERMISSIONS |
                FILE_PERMISSIONS <permissions>...]
               [COPYONLY] [ESCAPE_QUOTES] [@ONLY]
               [NEWLINE_STYLE [UNIX|DOS|WIN32|LF|CRLF] ])
```
`configure_file()`将 input 文件(通常为 XXX.Y.in)复制转换为 output 文件(XXX.YY)，在复制时将形如`${VAR}`, `@VAR@`的 cmake 变量替换为对应的值。
同时处理`#cmakedefine`：对于`#cmakedefine VAR value`，若VAR的值为ON/TRUE/1，替换为`#define VAR value`，否则替换为`/* #undef VAR */`。

`option(<variable> "<help_text>" [value])`
`option()`用于设置一些预定义的 boolean 值，未指定 value 时默认为 *OFF*。

```
if(<condition>)
  <commands>
elseif(<condition>) # optional block, can be repeated
  <commands>
else()              # optional block
  <commands>
endif()
```
`if()`类似于C++中的条件控制语句，其中的 condition 有许多形式及相关关键字：值(常量、变量、字符串)；逻辑操作符(**无短路**)；存在性检验操作符；文件操作；比较操作；版本比较；路径比较；变量展开；