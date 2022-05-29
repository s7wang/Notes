# 00_1 重修C++之CMake大法好



## 1 简述

本人以前对CMake有一点接触，不是很熟练但是会一些基本的使用，最近由于在“重修C++”学习一些更加偏向工程实践的内容，所以就需要对CMake有一个系统的了解，本次学习的目标在于会用、会写、会改。

cmake 是一个支持跨平台的项目构建器，我们可以使用cmake构建、安装项目或直接运行其他响应的构建编译工具。其目的是为了描述如何自动实现从源代码到项目构建再到可执行文件和库的工作流程。

要生产带有CMake的构建系统，必须有下面几个部分：

> 1. 源    树：简单说就是包含项目提供的源文件的顶级目录。通过该目录能够获取到所有项目构建所需的源文件。
> 2. 构建树：储存构建系统文件和构建输出文件的顶级目录（一般都是build）
> 3. 生成器：这里还不太懂，貌似系统默认就好。



### 生成一个项目构建系统

cmake常用的命令模式是  `cmake [<options>] <path-to-source>` 和 `cmake [<options>] <path-to-existing-build>` 或 `cmake [<options>] -S <path-to-source> -B <path-to-build>` 通常省略选项，直接接源树的目录，可以是绝对目录也可以是相对目录。但是源目录下必须包含一个cmake脚本CMakeLists.txt。如下例子：

```txt
[wangs7@localhost build]$ cmake ../src
# 其中目录的结构应该是下面的样子
HelloWorld/
├── build
└── src
	├── CMakeLists.txt
    └── HelloWorld.cpp
# 也就是说<path-to-source>目录下必须包含一个正确的CMakeLists.txt

[wangs7@localhost build]$ cmake .
# 其中目录的结构应该是下面的样子
HelloWorld/
├── build
|   └── CMakeLists.txt
└── src
    └── HelloWorld.cpp
# 也就是说这种模式下<path-to-existing-build>目录下
# 也必须包含一个正确的CMakeLists.txt

# z
```

其实更常见的做法是将CMakeLists.txt放在工程目录下，然后在构建树的根目录下执行上层目录的CMakeLists.txt，如下：

```txt
[wangs7@localhost build]$ cmake ..
# 其中目录的结构应该是下面的样子
HelloWorld/
├── build
├── src
│   └── HelloWorld.cpp
└── CMakeLists.txt
# 也就是说源树和构建树中都不包含CMakeLists.txt,
```



### 选项

`-S <path-to-source>` 

> 要生成的CMake项目的根目录的路径。



`-B <path-to-build>` 

> CMake构建要使用的目录。如果目录不存在，CMake将创建该目录。



`-C <initial-cache>` 

> 预加载脚本，输入缓存项数据。
>
> 当CMake第一次在空生成树中运行时，它将创建一个CMakeCache.txt文件，并使用项目的可自定义设置填充该文件。此选项可用于指定在首次通过项目的CMake listfiles之前从中加载缓存项的文件。加载的条目优先于项目的默认值。给定的文件应该是包含使用缓存选项的set（）命令的CMake脚本，而不是缓存格式文件。

注：脚本中对 `CMAKE_SOURCE_DIR` 和 `CMAKE_BINARY_DIR` 的引用计算为顶级源代码和构建树。

```txt
# 对于下面这样的目录结构
HelloWorld/
├── build
├── src
│   └── HelloWorld.cpp
└── CMakeLists.txt

# 在脚本中添加两行打印信息
message("-- CMAKE_SOURCE_DIR is "${CMAKE_SOURCE_DIR})
message("-- CMAKE_BINARY_DIR is "${CMAKE_BINARY_DIR})
# 执行cmake可以看到打印信息
[wangs7@localhost build]$ cmake ..
-- CMAKE_SOURCE_DIR is /home/wangs7/VSCode/test_cmake
-- CMAKE_BINARY_DIR is /home/wangs7/VSCode/test_cmake/build
```



`-D <var>:<type>=<value>` 或 ` -D <var>=<value>` 

> 创建或更新CMake缓存项。
>
> 当CMake第一次在空生成树中运行时，它将创建一个CMakeCache.txt文件，并使用项目的可自定义设置填充该文件。此选项可用于指定==优先于项目默认值==的设置。可以根据需要为任意多个缓存条目重复该选项。如果`<type>`是已经给定的，这个类型必须是 set() 命令文档为其缓存签名指定的类型之一；如果这个条目不存在且类型已存在，则将创建该条目而不包含任何类型。如果项目中的命令将类型设置为 PATH 或 FILEPATH，则`<value>`将转换为绝对路径。
>
> 此选项也可以作为单个参数提供：`-D<var>:<type>=<value>` 或 `-D<var>=<value>` 。

```txt
# 例如使用 -D 设置调试选项，下面两种都可以
[wangs7@localhost build]$ cmake -D CMAKE_BUILD_TYPE=Debug ..
[wangs7@localhost build]$ cmake -DCMAKE_BUILD_TYPE=Debug ..
```



`-U <globbing_expr>`

> 删除匹配项。
>
> 此选项可用于从文件中删除一个或多个变量，支持使用和对表达式进行全局化。可以根据需要对任意多个条目重复该选项。



`-G <generator-name>`

> 指定构建系统生成器。
>
> CMake可能在某些平台上支持多个本机构建系统。生成器负责生成特定的生成系统。cmake生成器手册中规定了可能的生成器名称。
>
> 如果未指定，CMake将检查 CMAKE_GENERATOR 环境变量，否则返回到内置默认选择。



选项还有很多，其实常用的也就这些，甚至只有前几个可能会经常用。





## 2 一个简单的 CMakeLisits.txt



对于我这种简单使用cmake的人来说，最重要的就是 CMakeLists.txt 文件了，我们需要能看懂文件，然后会写文件，最后能修改别人的文件。



### 2.1 环境变量



|            变量            |          值           | 备注                                                         |
| :------------------------: | :-------------------: | ------------------------------------------------------------ |
|     CMAKE_PREFIX_PATH      |       环境前缀        | 其初始值取自调用进程环境，可以设置为目录列表，要通过find_xxx() 搜索来指定前缀。 |
| CMAKE_BUILD_PARALLEL_LEVEL |      构建线程数       | 指定构建过程中并发进程的最大数目，其初始值取自调用进程环境。 |
|     CMAKE_CONFIG_TYPE      |       配置类型        | 未给出显式配置时，生成项目和ctest生成处理程序的默认生成配置。 |
|      CMAKE_GENERATOR       |        生成器         | 指定在没有生成器时使用的 CMake 默认生成器。如果所提供的值没有说出 CMake 已知的生成器的名称，则使用内部默认值。（我的环境下 CMAKE_GENERATOR is Unix Makefiles） |
|          DESTDIR           |       目标目录        | 在UNIX上，可以使用DESTDIR机制来重新定位整个安装。DESTDIR表示目标目录。makefile用户通常使用它在非默认位置安装软件。 |
|          LDFLAGS           |      链接器标志       | 将仅由CMake在第一次配置中用于确定默认链接器标志，之后LDFLAGS的值将作为CMake_EXE_linker_flags_INIT、CMake_SHARED_linker_flags_INIT和CMake_MODULE_linker_flags_INIT存储在缓存中。 |
|    \<PackageName\>_ROOT    | \<PackageName\>的路径 | find_package(\<PackageName\>) will search in prefixes specified by the \<PackageName\>_ROOT environment variable. |



上面是一些系统环境的变量，下面是一些关于编译的环境变量。

|    变量     |          值           | 备注                                                         |
| :---------: | :-------------------: | ------------------------------------------------------------ |
|     CC      |        C编译器        | 编译C语言文件的首选可执行文件。将仅由CMake在第一次配置中使用以确定C编译器。之后CC的值作为 ` CMake_C_COMPILER` 存储在缓存中。对于任何配置运行（包括第一次），如果定义了 ` CMake_C_COMPILER` 变量，则环境变量将被忽略。 |
|   CFLAGS    |  编译C文件的编译标志  | 编译C文件时要使用的默认编译标志。将仅由CMake在第一次配置中使用，以确定CC默认编译标志，之后CFLAGS的值将作为`CMAKE_C_FLAGS_INIT`标志存储在缓存中。如果定义了`CMAKE_C_FLAGS_INIT`变量，则环境变量将被忽略。 |
|   CUDACXX   |      cuda编译器       | cuda编译器。对应变量 `CMAKE_CUDA_COMPILER`，说明同上         |
|  CUDAFLAGS  |    cuda的编译标志     | 编译CUDA文件时使用的默认编译标志。将仅由CMake在第一次配置中用于确定CUDA默认编译标志，之后CUDAFLAGS的值将作为 `CMAKE_CUDA_FLAGS` 标志存储在缓存中。 |
| CUDAHOSTCXX |    cuda主机编译器     | 编译CUDA语言文件时用于编译主机代码的首选可执行文件。 `CMAKE_CUDA_HOST_COMPILER` |
|     CXX     |       C++编译器       | 编译C++语言文件的首选可执行文件。将仅由CMake在第一次配置中使用以确定C编译器。之后CXX的值作为 ` CMake_CXX_COMPILER` 存储在缓存中。对于任何配置运行（包括第一次），如果定义了 ` CMake_CXX_COMPILER` 变量，则环境变量将被忽略。 |
|  CXXFLAGS   | 编译C++文件的编译标志 | 编译C++文件时要使用的默认编译标志。将仅由CMake在第一次配置中使用，以确定CC默认编译标志，之后CXXFLAGS的值将作为`CMAKE_CXX_FLAGS_INIT`标志存储在缓存中。如果定义了`CMAKE_CXX_FLAGS_INIT`变量，则环境变量将被忽略。 |



再下边就是一些路径相关的变量了

|           变量           |      值      | 备注                                                         |
| :----------------------: | :----------: | ------------------------------------------------------------ |
|     CMAKE_BINARY_DIR     | 构建树根目录 | PROJECT_BINARY_DIR、\<projectname\>_BINARY_DIR这三个变量指代的内容是一致的,如果是 in source 编译,指得就是工程顶层目录,如果是 out-of-source 编译,指的是工程编译发生的目录。PROJECT_BINARY_DIR 跟其他指令稍有区别,现在,你可以理解为他们是一致的。 |
|     CMAKE_SOURCE_DIR     | 工程顶层目录 | PROJECT_SOURCE_DIR、\<projectname\>_SOURCE_DIR这三个变量指代的内容是一致的,不论采用何种编译方式,都是工程顶层目录。也就是在 in source 编译时,他跟 CMAKE_BINARY_DIR 等变量一致。PROJECT_SOURCE_DIR 跟其他指令稍有区别,现在,你可以理解为他们是一致的。 |
| CMAKE_CURRENT_SOURCE_DIR |      -       | 指的是当前处理的 CMakeLists.txt 所在的路径。                 |
| CMAKE_CURRENT_LIST_FILE  |      -       | 调用这个变量的 CMakeLists.txt 的完整路径。                   |
| CMAKE_CURRENT_LIST_LINE  |      -       | 当前行号                                                     |
|    CMAKE_MODULE_PATH     |      -       | 这个变量用来定义自己的 cmake 模块所在的路径。如果工程比较复杂,有可能会编写一些 cmake 模块,这些 cmake 模块是随工程发布的,为了让 cmake 在处理CMakeLists.txt 时找到这些模块,需要通过 SET 指令,将 cmake 模块路径设置一下。比如`SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)`这时候你就可以通过 INCLUDE 指令来调用自己的模块了。 |
|  EXECUTABLE_OUTPUT_PATH  |      -       | 可执行程序的输出路径                                         |
|   LIBRARY_OUTPUT_PATH    |      -       | 库文件输出路径                                               |
|       PROJECT_NAME       |      -       | 工程名                                                       |



系统信息

> 1,`CMAKE_MAJOR_VERSION`,CMAKE 主版本号,比如 2.4.6 中的 2
> 2,`CMAKE_MINOR_VERSION`,CMAKE 次版本号,比如 2.4.6 中的 4
> 3,`CMAKE_PATCH_VERSION`,CMAKE 补丁等级,比如 2.4.6 中的 6
> 4,`CMAKE_SYSTEM`,系统名称,比如 Linux-2.6.22
> 5,`CMAKE_SYSTEM_NAME`,不包含版本的系统名,比如 Linux
> 6,`CMAKE_SYSTEM_VERSION`,系统版本,比如 2.6.22
> 7,`CMAKE_SYSTEM_PROCESSOR`,处理器名称,比如 i686.
> 8,`UNIX`,在所有的类 UNIX 平台为 TRUE,包括 OS X 和 cygwin
> 9,`WIN32`,在所有的 win32 平台为 TRUE,包括 cygwin

一些常用的变量暂时就这些了，以后遇到还会补充。



### 2.2 构建系统

这一部分尝试实现项目的构建，单纯实现可执行程序或者库的构建部分（可以生成目标文件即可），暂不考虑其他构建细节。

为了测试这部分的使用方法我写了一个小程序，计算整数的非负数次幂运算。目录结构如下：

```txt
./
├── build
├── src
│   ├── include
│   │   ├── mypow.cpp
│   │   └── mypow.h
│   └── main.cpp
└── CMakeLists.txt
```



接下来来编辑 CMakeLists.txt 文件。

```cmake
cmake_minimum_required(VERSION 3.12) #需要cmake的最小版本

project(app_mypow) #项目名称

set(SRC ${CMAKE_SOURCE_DIR}/src) #设置源文件路径
set(ICD ${SRC}/include) #设置头文件路径

add_library(mypow ${ICD}/mypow.cpp) #定义库
add_executable(exec ${SRC}/main.cpp) #定义可执行文件

target_link_libraries(exec mypow) #将可执行文件和库链接起来
```

可执行文件和库是使用`add_executable()`和`add_library()`命令定义的。生成的二进制文件具有针对目标平台的适当前缀、后缀和扩展名。二进制目标之间的依赖关系使用`target_link_libraries()` 。

这里主要关注`add_executable()` 、 `add_library()` 和 `target_link_libraries()` 三条命令。

`add_executable()`

```cmake
###################### 正常的可执行文件 ###############################################
add_executable(<name> [WIN32] [MACOSX_BUNDLE]
               [EXCLUDE_FROM_ALL]
               [source1] [source2 ...])
# <name> 目标可执行文件的名字，需要全局统一并且遵循平台规范。
# 默认情况下结果输出再构建树的根目录下。
# [WIN32] [MACOSX_BUNDLE] 为系统支持选项，UNIX/Linux 下不考虑。
# EXCLUDE_FROM_ALL 当指定了该参数，则子目录下的目标不会被父目录下的目标文件包含进去，
# 父目录的CMakeLists.txt不会构建子目录的目标文件，必须在子目录下显式去构建。
# 当父目录的目标依赖于子目录的目标，则子目录的目标仍然会被构建出来以满足依赖关系（例如使用了target_link_libraries）。
# 添加可执行文件的源参数可以使用语法为$<…>的“生成器表达式”。也可以直接列举文件名。

###################### 导入的可执行文件 ################################################
add_executable(<name> IMPORTED [GLOBAL])
###################### 可执行文件的别名 ################################################
add_executable(<name> ALIAS <target>)
# 这两种暂时不会用，以后遇到再说
```

`add_library()`

```cmake
###################### 正常的库 ###############################################
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [source1] [source2 ...])
# <name> 目标库的名字，会自动按照平台规范命名，如名为mypow的静态库在UNIX下会生成名为libmypow.a的文件。
# [STATIC | SHARED | MODULE] 指定生成 [静态 | 共享 | 模块] 类型的库文件。
# 静态库是链接其他目标时使用的对象文件的存档。
# 共享库动态链接并在运行时加载。
# 模块库是不链接到其他目标的插件，但可以在运行时使用类似dlopen的功能动态加载。
# 如果没有明确给出类型，则根据变量BUILD_SHARED_LIBS的当前值是否为on，该类型是静态类型还是共享类型。
# 对于共享库和模块库，POSITION_INDEPENDENT_CODE target属性将自动设置为ON。
# 简单说大部分情况，未指明类型默认生成静态库。 
# EXCLUDE_FROM_ALL 和源文件添加方法同上。

###################### 导入的库文件 ##############################################
add_library(<name> <SHARED|STATIC|MODULE|OBJECT|UNKNOWN> IMPORTED
            [GLOBAL])
######################### 对象库 ################################################
add_library(<name> OBJECT <src>...)
######################## 库的别名 ###############################################
add_library(<name> ALIAS <target>)
######################### 接口库 ################################################
add_library(<name> INTERFACE [IMPORTED [GLOBAL]])
# 这几种种暂时不会用，以后遇到再说
```

`target_link_libraries()`

```cmake
##################### 目标及其依赖项的库 ###########################
target_link_libraries(<target> <item>...)
# 默认情况下，具有此签名的库依赖项是可传递的。当此目标链接到另一个目标时，链接到此目标的库也将显示在另一个目标的链接行上。
# 其他形式
target_link_libraries(<target>
                      <LINK_PRIVATE|LINK_PUBLIC> <lib>...
                     [<LINK_PRIVATE|LINK_PUBLIC> <lib>...]...)
target_link_libraries(<target> LINK_INTERFACE_LIBRARIES <item>...)   
# 这几种种暂时不会用，以后遇到再说
```

所以结合 CMakeLists.txt 文件的内容，对于这个简单的构建过程主要分三步，一、添加库；二、添加可执行程序；三、链接。其实如果不需要生成库文件也可以省略第一步，然后把库文件相关代码添加到可执行文件的源文件列表中。



### 2.3 编译 & 运行

最后来构建编译项目，进到 build 目录下，执行 cmake .. 成功生成 makefile 后，执行 make，就能看到在build目录下生成了库文件 libmypow.a 和 可执行文件 exec。

```txt
[wangs7@localhost build]$ cmake ..
-- The C compiler identification is GNU 8.4.1
-- The CXX compiler identification is GNU 8.4.1
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/wangs7/VSCode/test_cmake/build
[wangs7@localhost build]$ make
Scanning dependencies of target mypow
[ 25%] Building CXX object CMakeFiles/mypow.dir/src/include/mypow.cpp.o
[ 50%] Linking CXX static library libmypow.a
[ 50%] Built target mypow
Scanning dependencies of target exec
[ 75%] Building CXX object CMakeFiles/exec.dir/src/main.cpp.o
[100%] Linking CXX executable exec
[100%] Built target exec
```



运行 exec。

```txt
[wangs7@localhost build]$ ./exec 
2^6 = 64
2^7 = 128
```



### 2.4 安装

这里来说一下简单的安装，还是上一个项目，现在设计将可执行文件、库文件、以及头文件都安装到指定的目录下，需要使用到 `install` 在 CMakeLists.txt 中的后面添加上这几行。

```cmake
# 将可执行文件安装在 /home/wangs7/mypow 目录下
install(TARGETS exec
        DESTINATION  $ENV{HOME}/mypow)
# 将库文件安装在 /home/wangs7/mypow 目录下
install(TARGETS mypow
        DESTINATION $ENV{HOME}/mypow)
# 复制 include目录以及其中所有的头文件到 /home/wangs7/mypow 目录下
install(DIRECTORY ${ICD} 
        DESTINATION $ENV{HOME}/mypow
        FILES_MATCHING PATTERN "*.h")
```

删除build下的所有文件，重新执行cmake和make命令，最后执行make install命令，可以看到shell中输出如下内容。

```txt
[wangs7@localhost build]$ make install
[ 50%] Built target mypow
[100%] Built target exec
Install the project...
-- Install configuration: ""
-- Installing: /home/wangs7/mypow/exec
-- Installing: /home/wangs7/mypow/libmypow.a
-- Installing: /home/wangs7/mypow/include
-- Installing: /home/wangs7/mypow/include/mypow.h
[wangs7@localhost build]$ 
```

到该目录下检查内容是否正常，均正常。

下面附赠一些 CMake 中附带宏及其默认值（主要用于install）

安装目标文件：

| Target Type      | GNUInstallDirs Variable       | Built-In Default |
| :--------------- | :---------------------------- | :--------------- |
| `RUNTIME`        | `${CMAKE_INSTALL_BINDIR}`     | `bin`            |
| `LIBRARY`        | `${CMAKE_INSTALL_LIBDIR}`     | `lib`            |
| `ARCHIVE`        | `${CMAKE_INSTALL_LIBDIR}`     | `lib`            |
| `PRIVATE_HEADER` | `${CMAKE_INSTALL_INCLUDEDIR}` | `include`        |
| `PUBLIC_HEADER`  | `${CMAKE_INSTALL_INCLUDEDIR}` | `include`        |



安装文件：

| `TYPE` Argument | GNUInstallDirs Variable          | Built-In Default        |
| :-------------- | :------------------------------- | :---------------------- |
| `BIN`           | `${CMAKE_INSTALL_BINDIR}`        | `bin`                   |
| `SBIN`          | `${CMAKE_INSTALL_SBINDIR}`       | `sbin`                  |
| `LIB`           | `${CMAKE_INSTALL_LIBDIR}`        | `lib`                   |
| `INCLUDE`       | `${CMAKE_INSTALL_INCLUDEDIR}`    | `include`               |
| `SYSCONF`       | `${CMAKE_INSTALL_SYSCONFDIR}`    | `etc`                   |
| `SHAREDSTATE`   | `${CMAKE_INSTALL_SHARESTATEDIR}` | `com`                   |
| `LOCALSTATE`    | `${CMAKE_INSTALL_LOCALSTATEDIR}` | `var`                   |
| `RUNSTATE`      | `${CMAKE_INSTALL_RUNSTATEDIR}`   | `<LOCALSTATE dir>/run`  |
| `DATA`          | `${CMAKE_INSTALL_DATADIR}`       | `<DATAROOT dir>`        |
| `INFO`          | `${CMAKE_INSTALL_INFODIR}`       | `<DATAROOT dir>/info`   |
| `LOCALE`        | `${CMAKE_INSTALL_LOCALEDIR}`     | `<DATAROOT dir>/locale` |
| `MAN`           | `${CMAKE_INSTALL_MANDIR}`        | `<DATAROOT dir>/man`    |
| `DOC`           | `${CMAKE_INSTALL_DOCDIR}`        | `<DATAROOT dir>/doc`    |





## 3 丰富 CMakeLisits.txt 的功能



## 4 多级 CMakeLisits.txt 





























