# 导入导出指南

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

## 创建可重定位包

## 使用包配置文件

## 添加组件
