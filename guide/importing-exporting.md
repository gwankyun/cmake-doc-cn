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

## 导出目标

## 创建浮动包

## 使用包配置文件

## 添加组件
