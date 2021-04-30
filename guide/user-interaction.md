# 用户交互指南

[TOC]

## 简介

当软件包为基于CMake的构建系统提供了软件的源代码时，软件的消费者需要运行一个CMake用户交互工具来构建它。

行为良好的基于cmake的构建系统不会在源目录中创建任何输出，所以通常情况下，用户执行一个源外构建并在那里执行构建。首先，必须指示CMake生成一个合适的构建系统，然后用户调用构建工具来处理生成的构建系统。生成的构建系统是特定于用来生成它的机器的，并且是不可再分布的。提供的源软件包的每个消费者都需要使用CMake来生成特定于他们系统的构建系统。

生成的构建系统通常应该被视为只读的。作为主要构件的CMake文件应该完全指定构建系统，并且应该没有理由在IDE中手动填充属性，例如在生成构建系统之后。CMake会定期重写生成的构建系统，因此用户的修改会被覆盖。

通过提供CMake文件，本手册中描述的功能和用户界面可用于所有基于CMake的构建系统。

当处理提供的CMake文件时，CMake工具可能会向用户报告错误，比如报告编译器不受支持，或者编译器不支持必需的编译选项，或者无法找到依赖项。这些错误必须由用户通过选择不同的编译器、[安装依赖项](using-dependencies.md)或指示CMake在哪里找到它们来解决。

### cmake命令行工具

[cmake(1)](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake.1.html#manual:cmake(1))的一个简单但典型的用法是创建一个build目录并在那里调用cmake：

```console
$ cd some_software-1.4.2
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_INSTALL_PREFIX=/opt/the/prefix
$ cmake --build .
$ cmake --build . --target install
```

建议在到源的单独目录中构建，因为这样可以保持源目录的原始状态，允许使用多个工具链构建单个源，并允许通过简单地删除构建目录轻松地清除构建工件。

CMake工具可能会报告针对软件提供商的警告，而不是针对软件消费者的警告。这样的警告以“此警告是给项目开发人员的”结尾。用户可以通过向[cmake(1)](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake.1.html#manual:cmake(1))传递`-Wno-dev`标志来禁用此类警告。

### cmake-gui工具

更习惯GUI界面的用户可以使用[cmake-gui(1)](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake-gui.1.html#manual:cmake-gui(1))工具来调用CMake并生成构建系统。

必须首先填充源目录和二进制目录。总是建议为源文件和构建文件使用不同的目录。

![选择源码及二进制目录](file:///C:/Program%20Files/CMake/doc/cmake/html/_images/GUI-Source-Binary.png)

## 生成一个构建系统

有一些用户界面工具可以用来从CMake文件生成构建系统。[ccmake(1)](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/ccmake.1.html#manual:ccmake(1))和[cmake-gui(1)](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake-gui.1.html#manual:cmake-gui(1))工具通过设置各种必要的选项指导用户。可以调用[cmake(1)](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake.1.html#manual:cmake(1))工具来在命令行上指定选项。本手册描述了可以使用任何用户界面工具设置的选项，尽管每种工具设置选项的方式不同。

### 命令行环境

当使用命令行构建系统(如`Makefiles`或`Ninja`)调用[cmake(1)](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake.1.html#manual:cmake(1))时，有必要使用正确的构建环境以确保构建工具可用。CMake必须能够根据需要找到合适的[构建工具](file:///C:/Program%20Files/CMake/doc/cmake/html/variable/CMAKE_MAKE_PROGRAM.html#variable:CMAKE_MAKE_PROGRAM)、编译器、链接器和其他工具。

在Linux系统上，适当的工具通常在系统范围内的位置提供，并且可以通过系统包管理器随时安装。用户提供的或安装在非默认位置的其他工具链也可以使用。

在交叉编译时，一些平台可能需要设置环境变量，或者可能提供设置环境的脚本。

Visual Studio提供了多个命令提示符和`vcvarsal.bat`脚本，用于为命令行构建系统设置正确的环境。虽然在使用Visual Studio生成器时并不一定需要使用相应的命令行环境，但这样做无坏处。

当使用Xcode时，可以安装多个Xcode版本。使用哪种方法可以有很多不同的选择，但最常见的方法是：

- 在Xcode IDE的首选项中设置默认版本。
- 通过`xcode-select`命令行工具设置默认版本。
- 在运行CMake和构建工具时，通过设置`DEVELOPER_DIR`环境变量来覆盖默认版本。

为了方便起见，[cmake-gui(1)](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake-gui.1.html#manual:cmake-gui(1))提供了一个环境变量编辑器。

### 命令行-G选项

CMake根据平台默认选择一个生成器。通常，默认生成器足以允许用户继续构建软件。

用户可以使用`-G`选项覆盖默认生成器：

```shell
$ cmake .. -G Ninja
```

`cmake ——help`的输出包括一个可供用户选择的[生成器](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake-generators.7.html#manual:cmake-generators(7))列表。注意，生成器名称是区分大小写的。

在类Unix系统（包括Mac OS X）上，默认情况下使用[Unix Makefiles](file:///C:/Program%20Files/CMake/doc/cmake/html/generator/Unix%20Makefiles.html#generator:Unix%20Makefiles)生成器。该生成器的一个变体也可以在各种环境的Windows上使用，比如[NMake Makefiles](file:///C:/Program%20Files/CMake/doc/cmake/html/generator/NMake%20Makefiles.html#generator:NMake%20Makefiles)和[MinGW Makefiles](file:///C:/Program%20Files/CMake/doc/cmake/html/generator/MinGW%20Makefiles.html#generator:MinGW%20Makefiles)生成器。这些生成器生成一个`Makefile`变体，可以用`make`、`gmake`、`nmake`或类似工具执行。有关目标环境和工具的更多信息，请参见单个生成器文档。

[Ninja](file:///C:/Program%20Files/CMake/doc/cmake/html/generator/Ninja.html#generator:Ninja)生成器适用于所有主要平台。`Ninja`是一个用法类似于`make`的构建工具，但侧重于性能和效率。

在Windows上，可以使用[cmake(1)](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake.1.html#manual:cmake(1))为Visual Studio IDE生成解决方案。Visual Studio版本可以通过IDE的产品名来指定，其中包含一个四位数字的年份。别名也可以用来表示Visual Studio版本，比如两个数字对应于visualc++编译器的产品版本，或者两者的组合：

```shell
$ cmake .. -G "Visual Studio 2019"
$ cmake .. -G "Visual Studio 16"
$ cmake .. -G "Visual Studio 16 2019"
```

Visual Studio生成器可以针对不同的架构。可以使用 *-A* 选项指定目标架构：

```shell
cmake .. -G "Visual Studio 2019" -A x64
cmake .. -G "Visual Studio 16" -A ARM
cmake .. -G "Visual Studio 16 2019" -A ARM64
```

在苹果，[Xcode](file:///C:/Program%20Files/CMake/doc/cmake/html/generator/Xcode.html#generator:Xcode)生成器可能被用来为Xcode IDE生成项目文件。

一些ide，如KDevelop4, QtCreator和CLion，对基于cmake的构建系统有本地支持。这些ide提供了选择要使用的底层生成器的用户界面，通常是在`Makefile`或基于`Ninja`的生成器之间进行选择。

注意，在第一次调用CMake之后，不可能用`-G`来更改生成器。要更改生成器，必须删除构建目录，并且必须从头开始构建。

当生成Visual Studio项目和解决方案文件时，在最初运行[cmake(1)](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake.1.html#manual:cmake(1))时，可以使用其他几个选项。

Visual Studio工具集可以通过`-T`选项来指定：

```shell
$ # Build with the clang-cl toolset
$ cmake.exe .. -G "Visual Studio 16 2019" -A x64 -T ClangCL
$ # Build targeting Windows XP
$ cmake.exe .. -G "Visual Studio 16 2019" -A x64 -T v120_xp
```

`-A`选项指定_target_体系结构，而`-T`选项可用于指定所使用的工具链的详细信息。例如，可以使用 *-Thost=x64* 来选择64位版本的主机工具。下面演示了如何使用64位工具，以及如何构建64位目标体系结构：

```shell
$ cmake .. -G "Visual Studio 16 2019" -A x64 -Thost=x64
```

### 在cmake-gui选择生成器

“Configure”按钮会触发一个新的对话框来选择要使用的CMake生成器。

![配置一个生成器](file:///C:/Program%20Files/CMake/doc/cmake/html/_images/GUI-Configure-Dialog.png)

命令行中可用的所有生成器在[cmake-gui(1)](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake-gui.1.html#manual:cmake-gui(1))中也可用。

![选择一个生成器](file:///C:/Program%20Files/CMake/doc/cmake/html/_images/GUI-Choose-Generator.png)

当选择生成器时，可以使用更多选项来设置要生成的体系结构。

![选择Visual Studio生成器的体系结构](file:///C:/Program%20Files/CMake/doc/cmake/html/_images/VS-Choose-Arch.png)

## 设置构建变量

### 在命令行设置变量

### 在cmake-gui设置变量

### CMake缓存

## 预设

### 在命令行使用预设

### 在cmake-gui使用预设

## 调用构建系统

### 选择一个目标

### 指定一个构建程序

## 软件安装

## 运行测试