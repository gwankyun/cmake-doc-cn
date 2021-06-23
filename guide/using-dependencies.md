# 使用依赖项指南

[TOC]

## 介绍

对于希望使用CMake来使用第三方二进制包的开发人员来说，关于如何最佳地做到这一点有多种可能性，这取决于CMake对第三方库的感知程度。

与软件包一起提供的CMake文件包含查找每个构建依赖项的说明。有些构建依赖项是可选的，因为如果缺少依赖项，构建可能通过不同的特性集获得成功，而有些依赖项是必需的。CMake会搜索每个依赖项的已知位置，所提供的软件可能会为CMake提供额外的提示或位置来找到每个依赖项。

如[cmake(1)](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake.1.html#manual:cmake(1))没有找到所需的依赖项，缓存就会填充一个包含`NOTFOUND`值的条目。这个值可以通过在命令行或在[ccmake(1)](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/ccmake.1.html#manual:ccmake(1))或[cmake-gui(1)](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake-gui.1.html#manual:cmake-gui(1))工具中指定来替换。有关设置缓存项的更多信息，请参见[用户交互指南](user-interaction.md)。

## 提供配置文件包的库

第三方提供用于CMake的二进制库最方便的方法是提供[配置文件包](第三方提供用于CMake的二进制库最方便的方法是提供配置文件包。这些包是随库附带的文本文件，指导CMake如何使用库二进制文件和相关的头文件、助手工具和库提供的CMake宏。)。这些包是随库附带的文本文件，指导CMake如何使用库二进制文件和相关的头文件、助手工具和库提供的CMake宏。

配置文件通常可以在名称匹配`lib/cmake/<PackageName>`的目录中找到，尽管它们可能在其他位置。`</packagename>`对应于在CMake代码中使用[find_package()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/find_package.html#command:find_package)命令，例如`find_package(PackageName REQUIRED)`。

`lib/cmake/<PackageName>`目录将包含一个名为`<PackageName>Config.cmake`或者`<PackageName>-config.cmake`的文件。这是CMake包的入口点。一个名为`<PackageName>-config.cmake`的可选文件也可能存在于该目录中。CMake使用这个文件来确定第三方包的版本是否满足指定版本约束的[find_package()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/find_package.html#command:find_package)命令的使用。使用[find_package()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/find_package.html#command:find_package)时指定版本是可选的，即使存在`ConfigVersion`文件。

如果找到了`Config.cmake`并且满足了可选指定的版本，那么CMake的[find_package()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/find_package.html#command:find_package)命令就会考虑要找到的包，并且假定整个库包是设计的完整的。

可能有其他文件提供CMake宏或[导入目标](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake-buildsystem.7.html#imported-targets)供您使用。CMake不强制这些文件的任何命名约定。通过使用CMake的[include()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/include.html#command:include)命令，它们与主`Config`文件相关。

[调用CMake](user-interaction.md)的目的是使用第三方二进制文件包，需要CMake的[find_package()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/find_package.html#command:find_package)命令成功找到包。若包位于CMake的已知目录，那[find_package()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/find_package.html#command:find_package)调用应该成功。这些已知的cmake目录跟平台相关。例如，使用标准系统包管理器安装在Linux上的包将自动在`/usr`前缀中找到。类似的，Windows上，将会自动找到安装在`Program Files`中的软件包。

那些无法自动找到的包通常位于CMake无法预测的位置，例如`/opt/mylib`或`$HOME/dev/prefix`。这是一种常见的情况，CMake为用户提供了几种方法来指定在哪里找到这些库。

在[调用CMake](user-interaction.md)时，可以设置[CMAKE_PREFIX_PATH](file:///C:/Program Files/CMake/doc/cmake/html/variable/CMAKE_PREFIX_PATH.html#variable:CMAKE_PREFIX_PATH)变量。它被视为搜索[配置文件包](file:///C:/Program Files/CMake/doc/cmake/html/manual/cmake-packages.7.html#config-file-packages)的路径列表。安装在`/opt/somepackage`中的包通常会安装配置文件，例如`/opt/somepackage/lib/cmake/ somepackage/SomePackageConfig.cmake`。在这种情况下，应该将`/opt/some`包添加到[CMAKE_PREFIX_PATH](file:///C:/Program Files/CMake/doc/cmake/html/variable/CMAKE_PREFIX_PATH.html#variable:CMAKE_PREFIX_PATH)中。

也可以用前缀填充环境变量`CMAKE_PREFIX_PATH`来搜索包。与`PATH`环境变量一样，这是一个列表，需要使用特定于平台的环境变量列表项分隔符（`:`在Unix和`;`在Windows上）。

## 从包中导入目标

提供配置文件包的第三方包也可以提供[导入的目标](importing-exporting.md)。这些将在包含与包相关的特定于配置的文件路径的文件中指定，例如库的调试和发布版本。

第三方包文档通常会指出在成功地为库导入`find_package`之后可用的导入目标的名称。这些导入的目标名称可以与[target_link_libraries()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/target_link_libraries.html#command:target_link_libraries)命令一起使用。

一个简单使用第三方库的完整示例如下：

```cmake
cmake_minimum_required(VERSION 3.10)
project(MyExeProject VERSION 1.0.0)

find_package(SomePackage REQUIRED)
add_executable(MyExe main.cpp)
target_link_libraries(MyExe PRIVATE SomePrefix::LibName)
```

关于开发CMake构建系统的更多信息，请参见[cmake-buildsystem(7)](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7))。

### 库不提供配置文件包

