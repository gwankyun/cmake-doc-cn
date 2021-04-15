# 导入导出指南

[TOC]

## 简介

在本指南中，我们将介绍[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标的概念，并演示如何将现有的可执行文件或库文件从磁盘导入到CMake项目中。然后，我们将展示CMake如何支持从一个基于CMake的项目导出目标，并将它们导入到另一个项目中。最后，我们将演示如何用配置文件打包一个项目，以方便集成到其他CMake项目中。本指南和完整的示例源代码可以在CMake源码树的`Help/guide/importing-exporting`目录中找到。

## 导入目标

[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标用于将CMake项目外部的文件转换为项目内部的逻辑目标。[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标是使用[add_executable()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/add_executable.html#command:add_executable)和[add_library()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/add_library.html#command:add_library)命令的`IMPORTED`选项创建的。[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标不会生成构建文件。一旦导入，[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标可以像项目中的任何其他目标一样被引用，并提供对外部可执行文件和库的方便、灵活的引用。

默认情况下，[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标变量在创建它的目录及其下方具有作用域。我们可以使用`GLOBAL`选项来扩展可见性，这样就可以在构建系统中全局访问目标。

关于[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标的详细信息可以通过设置名称以`IMPORTED_`和`INTERFACE_`开头的属性来指定。例如，[IMPORTED_LOCATION](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED_LOCATION.html#prop_tgt:IMPORTED_LOCATION)包含到磁盘上目标的完整路径。

### 导入可执行文件

首先，我们将介绍一个简单的示例，该示例创建一个[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)的可执行目标，然后从[add_custom_command()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/add_custom_command.html#command:add_custom_command)命令引用它。

我们需要做一些准备工作来开始。我们希望创建一个可执行文件，在运行时在当前目录中创建一个基本的`main.cc`文件。这个项目的细节并不重要。导航到`Help/guide/importing-exporting/MyExe`目录，创建一个build目录，运行[cmake](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake.1.html#manual:cmake(1))命令构建并安装项目。

```shell
$ cd Help/guide/importing-exporting/MyExe
$ mkdir build
$ cd build
$ cmake ..
$ cmake --build .
$ cmake --install . --prefix <install location>
$ <install location>/myexe
$ ls
[...] main.cc [...]
```

现在我们可以将这个可执行文件导入到另一个CMake项目中。本节的源代码可以在`Help/guide/importing-exporting/Importing`中找到。在CMakeLists文件中，使用[add_executable()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/add_executable.html#command:add_executable)命令创建一个名为`myexe`的新目标。使用`IMPORTED`选项告诉CMake这个目标引用了位于项目外部的一个可执行文件。不会生成任何规则来构建它，并且[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标属性将被设置为`true`。

```CMake
add_executable(myexe IMPORTED)
```

接下来，使用[set_property()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/set_property.html#command:set_property)命令设置目标的[IMPORTED_LOCATION](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED_LOCATION.html#prop_tgt:IMPORTED_LOCATION)属性。这将告诉CMake目标在磁盘上的位置。该位置可能需要调整到上一步中指定的`<install location>`。

```CMake
set_property(TARGET myexe PROPERTY
            IMPORTED_LOCATION "../InstallMyExe/bin/myexe")
```

现在我们可以引用这个[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标，就像项目中构建的任何目标一样。在这个例子中，让我们假设我们想要在我们的项目中使用生成的源文件。在[add_custom_command()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/add_custom_command.html#command:add_custom_command)命令中使用[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标：

```CMake
add_custom_command(OUTPUT main.cc COMMAND myexe)
```

由于`COMMAND`指定了一个可执行的目标名称，它将自动被上面的[IMPORTED_LOCATION](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED_LOCATION.html#prop_tgt:IMPORTED_LOCATION)属性给出的可执行文件的位置所替换。

最后，使用[add_custom_command()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/add_custom_command.html#command:add_custom_command)的输出：

```CMake
add_executable(mynewexe main.cc)
```

### 导入库

以类似的方式，可以通过[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标访问其他项目的库。

**注意**:本节没有提供示例的完整源代码，仅作为读者的练习。

在CMakeLists文件中，添加一个[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)库，并指定它在磁盘上的位置：

```cmake
add_library(foo STATIC IMPORTED)
set_property(TARGET foo PROPERTY
             IMPORTED_LOCATION "/path/to/libfoo.a")
```

然后在我们的项目中使用[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)库：

```cmake
add_executable(myexe src1.c src2.c)
target_link_libraries(myexe PRIVATE foo)
```

在Windows上，.dll文件可以和它的.lib导入库一起导入：

```cmake
add_library(bar SHARED IMPORTED)
set_property(TARGET bar PROPERTY
             IMPORTED_LOCATION "c:/path/to/bar.dll")
set_property(TARGET bar PROPERTY
             IMPORTED_IMPLIB "c:/path/to/bar.lib")
add_executable(myexe src1.c src2.c)
target_link_libraries(myexe PRIVATE bar)
```

带有多个配置的库可以通过单个目标导入：

```cmake
find_library(math_REL NAMES m)
find_library(math_DBG NAMES md)
add_library(math STATIC IMPORTED GLOBAL)
set_target_properties(math PROPERTIES
  IMPORTED_LOCATION "${math_REL}"
  IMPORTED_LOCATION_DEBUG "${math_DBG}"
  IMPORTED_CONFIGURATIONS "RELEASE;DEBUG"
)
add_executable(myexe src1.c src2.c)
target_link_libraries(myexe PRIVATE math)
```

生成的构建系统将在发布配置中构建`myexe`到`m.lib`，在调试配置中构建`md.lib`。

## 导出目标

虽然[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标本身是有用的，但它们仍然要求导入它们的项目知道目标文件在磁盘上的位置。[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标真正的强大之处在于，提供目标文件的项目还提供了一个CMake文件来帮助导入目标文件。可以通过设置一个项目来生成必要的信息，以便其他CMake项目可以很容易地从构建目录、本地安装或打包使用它。

在其余部分中，我们将逐步介绍一组示例项目。第一个项目将创建并安装一个库以及相应的CMake配置和包文件。第二个项目将使用生成的包。

让我们从`Help/guide/importing-exporting/MathFunctions`目录的`MathFunctions`项目开始。这里有一个`MathFunctions.h`头文件，里面声明了一个`sqrt`函数：

```cpp
#pragma once

namespace MathFunctions {
double sqrt(double x);
}
```

以及相关的源文件`MathFunctions.cxx`：

```cpp
#include "MathFunctions.h"

#include <cmath>

namespace MathFunctions {
double sqrt(double x)
{
  return std::sqrt(x);
}
}
```

不要担心C++文件的细节，这只是一个简单的示例，可在许多构建系统上编译和运行。

现在我们可以为`MathFunctions`项目创建一个`CMakeLists.txt`文件。首先分别用[cmake_minimum_required()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/cmake_minimum_required.html#command:cmake_minimum_required)和[project()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/project.html#command:project)指定版本和名称：

```cmake
cmake_minimum_required(VERSION 3.15)
project(MathFunctions)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

使用[add_library()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/add_library.html#command:add_library)命令创建一个名为`MathFunctions`的库：

```cmake
add_library(MathFunctions STATIC MathFunctions.cxx)
```

然后使用[target_include_directories()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/target_include_directories.html#command:target_include_directories)命令为目标指定包含的目录：

```cmake
target_include_directories(MathFunctions
                           PUBLIC
                           "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
                           "$<INSTALL_INTERFACE:include>"
)
```

我们需要告诉CMake，我们想要使用不同的包含目录，这取决于我们是在构建库还是在安装位置使用它。如果我们不这样做，当CMake创建导出信息时，它将导出一个特定于当前构建目录的路径，对其他项目无效。我们可以使用[生成器表达式来](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7))指定在构建库时是否包含当前源目录。否则，在安装时，请包含包含目录。有关更多细节，请参阅[创建可重定位包](#创建可重定位包)一节。

[install(TARGETS)](file:///C:/Program%20Files/CMake/doc/cmake/html/command/install.html#command:install)和[install(EXPORT)](file:///C:/Program%20Files/CMake/doc/cmake/html/command/install.html#command:install)命令一起工作，安装两个目标（在我们的例子中是一个库）和一个CMake文件，该文件旨在方便地将目标导入到另一个CMake项目中。

首先，在[install(TARGETS)](file:///C:/Program%20Files/CMake/doc/cmake/html/command/install.html#command:install)命令中，我们将指定目标、导出名称和告诉CMake在何处安装目标的目标。

```cmake
install(TARGETS MathFunctions
        EXPORT MathFunctionsTargets
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include
)
```

这里，`EXPORT`选项告诉CMake创建一个名为`MathFunctionsTargets`的导出。生成的[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标设置了适当的属性来定义它们的[使用需求](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake-buildsystem.7.html#target-usage-requirements)，例如[INTERFACE_INCLUDE_DIRECTORIES](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/INTERFACE_INCLUDE_DIRECTORIES.html#prop_tgt:INTERFACE_INCLUDE_DIRECTORIES)、[INTERFACE_COMPILE_DEFINITIONS](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/INTERFACE_COMPILE_DEFINITIONS.html#prop_tgt:INTERFACE_COMPILE_DEFINITIONS)和其他相关的内置`INTERFACE_`属性。在[COMPATIBLE_INTERFACE_STRING](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/COMPATIBLE_INTERFACE_STRING.html#prop_tgt:COMPATIBLE_INTERFACE_STRING)中列出的用户定义属性的`INTERFACE`变体和其他[兼容的接口属性](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake-buildsystem.7.html#compatible-interface-properties)也会传播到生成的[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标。例如，在本例中，[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标将使用`INCLUDES DESTINATION`属性指定的目录填充其[INTERFACE_INCLUDE_DIRECTORIES](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/INTERFACE_INCLUDE_DIRECTORIES.html#prop_tgt:INTERFACE_INCLUDE_DIRECTORIES)属性。由于给出了一个相对路径，它被视为相对于[CMAKE_INSTALL_PREFIX](file:///C:/Program%20Files/CMake/doc/cmake/html/variable/CMAKE_INSTALL_PREFIX.html#variable:CMAKE_INSTALL_PREFIX)。

注意，我们还没有要求CMake安装导出。

我们不希望忘记使用[install(FILES)](file:///C:/Program%20Files/CMake/doc/cmake/html/command/install.html#command:install)命令安装`MathFunctions.h`头文件。头文件应该安装到`include`目录中，如上面的[target_include_directories()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/target_include_directories.html#command:target_include_directories)命令所指定的那样。

```cmake
install(FILES MathFunctions.h DESTINATION include)
```

现在`MathFunctions`库和头文件已经安装好了，我们还需要显式安装`MathFunctionsTargets`导出详细信息。按照[install(TARGETS)](file:///C:/Program%20Files/CMake/doc/cmake/html/command/install.html#command:install)命令的定义，使用[install(EXPORT)](file:///C:/Program%20Files/CMake/doc/cmake/html/command/install.html#command:install)命令导出`MathFunctionsTargets`中的目标。

```cmake
install(EXPORT MathFunctionsTargets
        FILE MathFunctionsTargets.cmake
        NAMESPACE MathFunctions::
        DESTINATION lib/cmake/MathFunctions
)
```

这段代码与我们在[导入库](#导入库)一节中手工创建的示例非常相似。注意`${_IMPORT_PREFIX}`是相对于文件位置计算的。

外部项目可以使用[include()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/include.html#command:include)命令加载该文件，并从安装树中引用`MathFunctions`库，就像在自己的树中构建一样。
例如：

```cmake
 include(${INSTALL_PREFIX}/lib/cmake/MathFunctionTargets.cmake)
 add_executable(myexe src1.c src2.c )
 target_link_libraries(myexe PRIVATE MathFunctions::MathFunctions)
```

第1行加载目标CMake文件。尽管我们只导出了一个目标，但该文件可以导入任意数量的目标。它们的位置是相对于文件位置计算的，以便可以容易地移动安装树。第3行引用了导入的`MathFunctions`库。生成的构建系统将从库的安装位置链接到库。

可执行文件也可以使用相同的过程导出和导入。

任意数量的目标安装都可以与相同的导出名称相关联。导出名称被认为是全局的，因此任何目录都可以提供目标安装。[install(EXPORT)](file:///C:/Program%20Files/CMake/doc/cmake/html/command/install.html#command:install)命令只需要调用一次，就可以安装一个引用所有目标的文件。下面是一个示例，演示了如何将多个导出组合成一个导出文件，即使它们位于项目的不同子目录中。

```cmake
# A/CMakeLists.txt
add_executable(myexe src1.c)
install(TARGETS myexe DESTINATION lib/myproj
        EXPORT myproj-targets)

# B/CMakeLists.txt
add_library(foo STATIC foo1.c)
install(TARGETS foo DESTINATION lib EXPORTS myproj-targets)

# Top CMakeLists.txt
add_subdirectory (A)
add_subdirectory (B)
install(EXPORT myproj-targets DESTINATION lib/myproj)
```

### 创建包

此时，`MathFunctions`项目正在导出其他项目需要使用的目标信息。我们可以通过生成一个配置文件使这个项目更容易被其他项目使用，这样CMake[find_package()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/find_package.html#command:find_package)命令就可以找到我们的项目。

首先，我们需要向`CMakeLists.txt`文件添加一些内容。首先，包含[CMakePackageConfigHelpers](file:///C:/Program%20Files/CMake/doc/cmake/html/module/CMakePackageConfigHelpers.html#module:CMakePackageConfigHelpers)模块，以访问一些用于创建配置文件的helper函数。

```cmake
include(CMakePackageConfigHelpers)
```

然后，我们将创建一个包配置文件和一个包版本文件。

#### 创建包配置文件

使用[CMakePackageConfigHelpers](file:///C:/Program%20Files/CMake/doc/cmake/html/module/CMakePackageConfigHelpers.html#module:CMakePackageConfigHelpers)提供的[configure_package_config_file()](file:///C:/Program%20Files/CMake/doc/cmake/html/module/CMakePackageConfigHelpers.html#command:configure_package_config_file)命令来生成包配置文件。注意，应该使用这个命令而不是普通的[configure_file()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/configure_file.html#command:configure_file)命令。通过避免安装的配置文件中的硬编码路径，它有助于确保生成的包是可重定位的。提供给`INSTALL_DESTINATION`的路径必须是`MathFunctionsConfig.cmake`文件将安装的目标。我们将在下一节中研究包配置文件的内容。

```cmake
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake"
  INSTALL_DESTINATION lib/cmake/MathFunctions
)
```

使用[INSTALL(files)](file:///C:/Program%20Files/CMake/doc/cmake/html/command/install.html#command:install)命令安装生成的配置文件。`MathFunctionsConfigVersion.cmake`和`MathFunctionsConfig.cmake`都安装在相同的位置，从而完成了包。

```cmake
install(FILES
          "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake"
          "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake"
        DESTINATION lib/cmake/MathFunctions
)
```

现在我们需要创建包配置文件本身。在本例中，`Config.cmake.in`文件非常简单，但足以允许下游使用[IMPORTED](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED)目标。

```cmake
@PACKAGE_INIT@

include("${CMAKE_CURRENT_LIST_DIR}/MathFunctionsTargets.cmake")

check_required_components(MathFunctions)
```

文件的第一行只包含字符串`@PACKAGE_INIT@`。这将在配置文件时展开，并允许使用以`PACKAGE_`为前缀的可重定位路径。它还提供了`set_and_check()`和`check_required_components()`宏。

`check_required_components`帮助宏通过检查所有必需的组件的`<Package>_<Component>_FOUND`变量来确保所有被请求的、非可选的组件都已经找到。这个宏应该在包配置文件的末尾调用，即使包没有任何组件。通过这种方式，CMake可以确保下游项目没有指定任何不存在的组件。如果`check_required_components`失败，`<Package>_FOUND`变量被设置为FALSE，那么这个包被认为没有找到。

宏`set_and_check()`应该在配置文件中使用，而不是用于设置目录和文件位置的常规`set()`命令。如果引用的文件或目录不存在，宏将失败。

如果`MathFunctions`包应该提供任何宏，那么这些宏应该放在单独的文件中，该文件安装在与`MathFunctionsConfig.cmake`文件相同的位置，并包含在该文件中。

**包的所有必需依赖项也必须在包配置文件中找到。** 让我们假设在我们的项目中需要`Stats`库。在CMakeLists文件中，我们将添加：

```cmake
find_package(Stats 2.6.4 REQUIRED)
target_link_libraries(MathFunctions PUBLIC Stats::Types)
```

由于`Stats::Types`目标是`MathFunctions`的`PUBLIC`依赖项，下流也必须找到`Stats`包并链接到`Stats::Types`库。应该在配置文件中找到`Stats`包以确保这一点。

```cmake
include(CMakeFindDependencyMacro)
find_dependency(Stats 2.6.4)
```

[CMakeFindDependencyMacro](file:///C:/Program%20Files/CMake/doc/cmake/html/module/CMakeFindDependencyMacro.html#module:CMakeFindDependencyMacro)模块中的`find_dependency`宏可以传播这个包是`REQUIRED`还是`QUIET`，等等。如果没有找到依赖项，`find_dependency`宏还将`MathFunctions_FOUND`设置为False，并诊断`MathFunctions`包不能在没有`Stats`包的情况下使用。

**练习：** 向`MathFunctions`项目添加所需的库。

#### 创建包版本文件

[CMakePackageConfigHelpers](file:///C:/Program%20Files/CMake/doc/cmake/html/module/CMakePackageConfigHelpers.html#module:CMakePackageConfigHelpers)模块提供了[write_basic_package_version_file()](file:///C:/Program%20Files/CMake/doc/cmake/html/module/CMakePackageConfigHelpers.html#command:write_basic_package_version_file)命令来创建一个简单的包版本文件。当调用[find_package()](file:///C:/Program%20Files/CMake/doc/cmake/html/command/find_package.html#command:find_package)来确定与请求版本的兼容性，并设置一些特定于版本的变量，如`<PackageName>_VERSION`， `<PackageName>_VERSION_MAJOR`， `<PackageName>_VERSION_MINOR`，等等时，CMake将读取该文件。更多细节请参阅[cmake-packages](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake-packages.7.html#manual:cmake-packages(7))文档。

```cmake
set(version 3.4.1)

set_property(TARGET MathFunctions PROPERTY VERSION ${version})
set_property(TARGET MathFunctions PROPERTY SOVERSION 3)
set_property(TARGET MathFunctions PROPERTY
  INTERFACE_MathFunctions_MAJOR_VERSION 3)
set_property(TARGET MathFunctions APPEND PROPERTY
  COMPATIBLE_INTERFACE_STRING MathFunctions_MAJOR_VERSION
)

# generate the version file for the config file
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake"
  VERSION "${version}"
  COMPATIBILITY AnyNewerVersion
)
```

在我们的示例中，`MathFunctions_MAJOR_VERSION`被定义为一个[COMPATIBLE_INTERFACE_STRING](file:///C:/Program%20Files/CMake/doc/cmake/html/prop_tgt/COMPATIBLE_INTERFACE_STRING.html#prop_tgt:COMPATIBLE_INTERFACE_STRING)，这意味着它必须在任何依赖项的依赖项之间兼容。通过在这个版本和下一个版本的`MathFunctions`中设置这个自定义的user属性，如果试图使用版本3和版本4，[cmake](file:///C:/Program%20Files/CMake/doc/cmake/html/manual/cmake.1.html#manual:cmake(1))将发出诊断。如果包的不同主要版本被设计成不兼容的，那么包可以选择使用这种模式。

### 从构建目录导出目标

### 构建并安装一个包

## 创建一个浮动包

## 使用包配置文件

## 添加组件
