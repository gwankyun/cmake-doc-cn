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

![GUI-Source-Binary](file:///C:/Program%20Files/CMake/doc/cmake/html/_images/GUI-Source-Binary.png)

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

### 在cmake-gui选择生成器

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
