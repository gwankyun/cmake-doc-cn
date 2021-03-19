# CMake教程

## 简介

CMake教程提供了一个循序渐进的指南，涵盖了CMake帮助解决的常见构建系统问题。了解示例项目中各种主题是如何一起工作的可能会非常有帮助。教程文档和示例的源代码可以在CMake源代码树的`Help/guide/tutorial`目录中找到。每个步骤都有自己的子目录，其中包含可以用作起点的代码。教程示例是渐进的，因此每个步骤都为前一个步骤提供完整的解决方案。

## 一个基本的起点（第一步）

最基本的项目是由源代码文件构建的可执行文件。对于简单的项目，只需要一个三行`CMakeLists.txt`文件。这将是我们教程的起点。在`Step1`目录创建一个`CMakeLists.txt`文件，如下所示：

```cmake
cmake_minimum_required(VERSION 3.10)

# set the project name
project(Tutorial)

# add the executable
add_executable(Tutorial tutorial.cxx)
```

注意，这个例子在`CMakeLists.txt`文件中使用了小写命令。CMake支持大小写混合命令。`tutorial.cxx`的源代码在`Step1`目录中提供，可以用来计算一个数字的平方根。

### 添加版本号和配置的头文件

我们要添加的第一个特性是为我们的可执行文件和项目提供一个版本号。虽然在源码就能做到，但`CMakeLists.txt`更灵活。

首先，修改`CMakeLists.txt`文件，使用[project()](https://)命令设置项目名称和版本号。

```cmake
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)
```

然后，配置一个头文件来将版本号传递给源代码：

```cmake
configure_file(TutorialConfig.h.in TutorialConfig.h)
```

由于配置的文件将被写入到二进制目录中，所以我们必须将该目录添加到搜索包含文件的路径列表中。在`CMakeLists.txt`文件的末尾添加以下行：

```cmake
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
```

用你喜欢的编辑器，在源目录中创建`TutorialConfig.h.in`，内容如下:

```cpp
// the configured options and settings for Tutorial
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

当CMake配置这个头文件时，`@Tutorial_VERSION_MAJOR@`和`@Tutorial_VERSION_MINOR@`的值将被替换。

下一步，修改`tutorial.cxx`以包含已配置的头文件`TutorialConfig.h`。

最后，让我们通过更新`tutorial.cxx`来打印出可执行文件的名称和版本号，如下 :

```cpp
  if (argc < 2) {
    // report version
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
```

### 指定c++标准

接下来，让我们通过用`tutorial.cxx`中的`std:: stand`替换`atof`，为我们的项目添加一些c++ 11特性。同时，删除`#include <cstdlib>`。

```cpp
const double inputValue = std::stod(argv[1]);
```

我们需要在CMake代码中明确声明它应该使用正确的标志。在CMake中启用对特定C++标准的支持的最简单方法是使用[CMAKE_CXX_STANDARD](https://)变量。对于本教程，将`CMakellists.tx`文件中的[CMAKE_CXX_STANDARD](https://)变量设置为11，[CMAKE_CXX_STANDARD_REQUIRED](https://)设置为True。确保`CMAKE_CXX_STANDARD`在调用`add_executable`之前声明。

```cmake
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

### 构建和测试

运行[cmake](https://)可执行文件或[cmake-gui](https://)来配置项目，然后用你选择的构建工具构建它。

例如，我们可以从命令行导航到CMake源代码树的`Help/guide/tutorial`目录，并创建一个构建目录：

```shell
mkdir Step1_buildspan
```

接下来，导航到build目录，运行CMake来配置项目并生成一个本地构建系统：

```shell
cd Step1_build
cmake ../Step1
```

然后调用构建系统来实际编译/链接项目：

```shell
cmake --build .
```

最后，尝试用以下命令来使用新构建的`Tutorial` ：

```shell
Tutorial 4294967296
Tutorial 10
Tutorial
```

## 添加库（第二步）

现在我们将向我们的项目添加一个库。这个库将包含我们自己的计算数字平方根的实现。可执行文件可以使用这个库，而不是编译器提供的标准平方根函数。

在本教程中，我们将把这个库放入名为MathFunctions的子目录中。这个目录已经包含了一个头文件`MathFunctions.h`和一个源文件`mysqrt.cxx`。源文件有一个名为`mysqrt`的函数，功能类似于自带的`sqrt`函数。

将以下一行`CMakeLists.txt`文件添加到`MathFunctions`目录:

```cmake
add_library(MathFunctions mysqrt.cxx)
```

为了使用这个新库，我们将在顶层的`CMakeLists.txt`文件中添加一个[add_subdirectory()](https://)调用，以便构建这个库。我们将新库添加到可执行文件中，并将`MathFunctions`作为包含目录添加，以便能够找到`mysqrt.h`头文件。顶层`CMakeLists.txt`文件的最后几行现在应该是这样的：

```cmake
# add the MathFunctions library
add_subdirectory(MathFunctions)

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC MathFunctions)

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                          "${PROJECT_BINARY_DIR}"
                          "${PROJECT_SOURCE_DIR}/MathFunctions"
                          )
```

现在让我们使MathFunctions库成为可选的。虽然在本教程中没有必要这样做，但对于大型项目来说，这是很常见的情况。第一步是向顶层`CMakeLists.txt`文件添加一个选项。

```cmake
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(TutorialConfig.h.in TutorialConfig.h)
```

这个选项将在[cmake-gui](https://)和[ccmake](https://)中显示，默认值ON可以由用户更改。该设置将存储在缓存中，这样用户在每次在构建目录上运行CMake时就不需要设置该值。

下一个更改是使构建和链接MathFunctions库成为有条件的。为此，我们将顶层`CMakeLists.txt`文件的结尾修改为如下所示：

```cmake
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
  list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
endif()

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           ${EXTRA_INCLUDES}
                           )
```

注意，使用了变量`EXTRA_LIBS`来收集任何可选库，以便稍后链接到可执行文件中。变量`EXTRA_INCLUDES`类似地用于可选头文件。在处理许多可选组件时，这是一种传统方法，我们将在下一步讨论现代方法。

对源代码的相应更改相当简单。首先，在`tutorial.cxx`中，在需要的时候包含`MathFunctions.h`头文件：

```cpp
#ifdef USE_MYMATH
#  include "MathFunctions.h"
#endif
```

然后，在同一个文件中，让`USE_MYMATH`控制使用哪个平方根函数：

```cpp
#ifdef USE_MYMATH
  const double outputValue = mysqrt(inputValue);
#else
  const double outputValue = sqrt(inputValue);
#endif
```

由于源代码现在需要`USE_MYMATH`，我们可以通过以下一行把它添加到`TutorialConfig.h.in`中：

```cpp
#cmakedefine USE_MYMATH
```

**练习**：为什么在`USE_MYMATH`选项后面配置`TutorialConfig.h.in`很重要？如果我们把这两个颠倒过来会发生什么？

运行[cmake](https://)可执行文件或[cmake-gui](https://)来配置项目，用你选择的构建工具构建它。然后运行Tutorial可执行文件。

现在让我们更新`USE_MYMATH`的值。最简单的方法是使用[cmake-gui](https://)或[ccmake](https://)（如果你在终端上的话）。如果你想从命令行更改选项，试试：

```shell
cmake ../Step2 -DUSE_MYMATH=OFF
```

重新生成并再次运行。

哪个函数给出了更好的结果，sqrt还是mysqrt？

## 添加库的使用需求（第三步）

使用需求允许对库或可执行文件的链接和include行进行更好的控制，同时也允许对CMake内部目标的传递属性进行更多的控制。利用使用需求的主要命令是：

- [target_compile_definitions()](https://)
- [target_compile_options()](https://)
- [target_include_directories()](https://)
- [target_link_libraries()](https://)

让我们从[添加库（第二步）](https://)开始重构代码，以使用现代的CMake方法满足使用需求。我们首先声明，任何链接到MathFunctions的人都需要包括当前的源目录，而MathFunctions本身不需要。因此，这可以成为一个`INTERFACE `使用要求。

请记住，`INTERFACE`指的是消费者需要但生产者不需要的东西。在`MathFunctions/CMakeLists.txt`的末尾添加以下几行：

```cmake
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          )
```

现在我们已经指定了MathFunctions的使用要求，我们可以安全地从顶层的`CMakeLists.txt`中删除`EXTRA_INCLUDES`变量的使用，这里：

```cmake
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
endif()
```

和这里：

```cmake
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
```

一旦完成，运行[cmake](https://)命令或[cmake-gui](https://)来配置项目，然后用你选择的构建工具或使用`cmake --build .`在构建目录来构建它。

## 安装和测试（第四步）

现在我们可以开始向我们的项目添加安装规则和测试支持了。

### 安装规则

安装规则相当简单：对于MathFunctions，我们希望安装库和头文件，对于应用程序，我们希望安装可执行和配置的头文件。

所以在`MathFunctions/CMakeLists.txt`的末尾添加：

```cmake
install(TARGETS MathFunctions DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```

顶层`CMakeLists.txt`末尾添加：

```cmake
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )
```

这就是创建基本本地安装的全部内容。

运行[cmake](https://)或者[cmake-gui](https://)来配置并用构建工具来构建它。

接着使用[cmake](https://)命令的`install`选项（3.15版本开始，之前版本的CMake必须使用`make install`）在命令行安装。对于多配置的工具，记得用`--config`来指定配置。若使用IDE，只需构建`INSTALL`目标。这一步将安装相应的头文件、库和可执行文件，例子：

```cmake
cmake --install .
```

`CMAKE_INSTALL_PREFIX`变量用于指定安装目录。在运行`cmake --install`命令的时候，会被`--prefix`参数覆盖。例如：

```cmake
cmake --install . --prefix "/home/myuser/installdir"
```

导航到安装目录并验证程序能否运行。

### 测试支持

接下来测试一下我们的程序。在可以在顶级的`CMakeLists.txt`文件末尾启用测试，然后添加一些基本的测试用例来验证程序是否正常。

```cmake
enable_testing()

# does the application run
add_test(NAME Runs COMMAND Tutorial 25)

# does the usage message work?
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )

# define a function to simplify adding tests
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction(do_test)

# do a bunch of result based tests
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is [-nan|nan|0]")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
```

第一个测试只是验证程序能否运行，是否出现段错误或者崩溃，返回值是否为0。这就是基本的CMake测试。

下一个测试使用[PASS_REGULAR_EXPRESSION](https://)测试属性来验证测试输出是否包含某些字符串。这个例子中，验证当提供的参数数量不正确时，是否输出相关信息。最后，有一个`do_test`函数，它运行程序并验证计算出来的平方根对于给定的输入是否正确。对于每次调用`do_test`，都会将另一个测试添加到项目中，并通过的参数传递名称、输入及预期结果。

重新构建程序并进入程序目录，运行`ctest`命令：`ctest -N`和`ctest -VV`。对于多配置生成器（例如Visual Studio），必须指定配置类型。例如，要在调试模式下运行测试，可以在构建目录（而不是Debug目录）中进行`ctest -C Debug -VV`。或者，从IDE构建`RUN_TESTS`目标。

## 添加系统自省（第五步）

考虑向项目中添加一些依赖目标平台可能没有的特性代码。对于本例，我们将添加一些代码，这将取决于目标平台是否有`log`和`exp`函數。当然，几乎每个平台都有这些函数，但本教程假设它们并不常见。

如果平台有`log`和`exp`，那么我们将使用它们在`mysqrt`中计算平方根。首先在`MathFunctions/CMakeLists.txt`中使用`CheckSymbolExists`模块判断这些函数是否可用。在一些平台上，需要链接到m库。如果`log`和`exp`不可用，使用m库并重试。

```cmake
include(CheckSymbolExists)
check_symbol_exists(log "math.h" HAVE_LOG)
check_symbol_exists(exp "math.h" HAVE_EXP)
if(NOT (HAVE_LOG AND HAVE_EXP))
  unset(HAVE_LOG CACHE)
  unset(HAVE_EXP CACHE)
  set(CMAKE_REQUIRED_LIBRARIES "m")
  check_symbol_exists(log "math.h" HAVE_LOG)
  check_symbol_exists(exp "math.h" HAVE_EXP)
  if(HAVE_LOG AND HAVE_EXP)
    target_link_libraries(MathFunctions PRIVATE m)
  endif()
endif()
```

如果可以的话，使用[target_compile_definitions()](https://)指定`HAVE_LOG`和`HAVE_EXP`为`PRIVATE`编译器定义。

```cmake
if(HAVE_LOG AND HAVE_EXP)
  target_compile_definitions(MathFunctions
                             PRIVATE "HAVE_LOG" "HAVE_EXP")
endif()
```

如果`log`和`exp`在系统上可用，那么我们将在`mysqrt`函数中用来计算平方根。将以下代码添加到`MathFunctions/mysqrt.cxx`中的`mysqrt`函数中（返回結果前不要忘了`#endif`！）：

```cpp
#if defined(HAVE_LOG) && defined(HAVE_EXP)
  double result = exp(log(x) * 0.5);
  std::cout << "Computing sqrt of " << x << " to be " << result
            << " using log and exp" << std::endl;
#else
  double result = x;
```

同时还要修改`mysqrt.cxx`以包含`cmath`：

```cpp
#include <cmath>
```

运行[cmake](https://)命令或者[cmake-gui](https://)来配置并用构建工具构建它，然后运行Tutorial程序。

哪个函数给了更好的结果？sqrt还是mysqrt？

## 添加自定义命令和生成的文件（第六步）

假设，出于教学目的，我们决定不使用自带的`log`和`exp`函数，而希望生成一个包含预计算值的表，以便在`mysqrt`中使用。本节中，我们将创建表作为构建过程的一部分，并且将表编译到我们的程序中。

首先，删除`MathFunctions/CMakeLists.txt`中对`log`和`exp`的检查。然后删除`mysqrt.cxx`中对`HAVE_LOG`和`mysqrt.cxx`的检查，与此同时可以删除`#include `。

`MathFunctions`目录中有一個名为`MakeTable.cxx`的源文件来提供生成表。

检视这个文件后，可以看到这个表以C++代码展现，输出文件名通过参数传达。

下一步是将适当的命令添加文件中，以构建`MathFunctions/CMakeLists.txt`文件中以构建MakeTable程序并作为构建过程的一部分运行。需要一些命令来完成这一点。

首先在`MathFunctions/CMakeLists.txt`开头将`MakeTable`添加为其他可执行文件。

```cmake
add_executable(MakeTable MakeTable.cxx)
```

然后，我们添加一个自定义命令，指定如何通过运行MakeTable生成`Table.h`。

```cmake
add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  DEPENDS MakeTable
  )
```

接下来需要让CMake知道`mysqrt.cxx`信赖于那个生成的`Table.h`。这是通过将`Table.h`添加到MathFunctions的源码列表达到的。

```cmake
add_library(MathFunctions
            mysqrt.cxx
            ${CMAKE_CURRENT_BINARY_DIR}/Table.h
            )
```

我们必须将当前目录加入引入目录列表，`Table.h`能够被`mysqrt.cxx`找到并引用。

```cmake
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          PRIVATE ${CMAKE_CURRENT_BINARY_DIR}
          )
```

现在我们使用已生成的表。首先，修改`mysqrt.cxx`以引用`Table.h`。接着，我们重构mysqrt函数使用这个表：

```cpp
double mysqrt(double x)
{
  if (x <= 0) {
    return 0;
  }

  // use the table to help find an initial value
  double result = x;
  if (x >= 1 && x < 10) {
    std::cout << "Use the table to help find an initial value " << std::endl;
    result = sqrtTable[static_cast<int>(x)];
  }

  // do ten iterations
  for (int i = 0; i < 10; ++i) {
    if (result <= 0) {
      result = 0.1;
    }
    double delta = x - (result * result);
    result = result + 0.5 * delta / result;
    std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
  }

  return result;
}
```

进行[cmake](https://)或者[cmake-gui](https://)来配置并构建此项目。

当程序构建时会先构建`MakeTable`程序。它会进行`MakeTable`产生`Table.h`。最后，它会编译包括`Table.h`的`mysqrt.cxx`以产生MathFunctions库。

运行Tutorial程序以验证是否产生使用了这个表。

## 构建安装程序（第七步）

我们下个愿望是分发工程走让别人使用它。我们想同时分发源码和二进制在不同的平台。这里我们之前讨论的第四步有一点不同，必须要在源码中编译。在此例子中，我们会构建一个安装包以支持二进制安装及包管理。为了达到这个目标我们应该使用CPack创建不同平台的安装包。应该在顶层`CMakeLists.txt`的添加一条：

```cmake
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
include(CPack)
```

这就是我们对它作的所有改能变。我们在开始包含[InstallRequiredSystemLibraries](https://)。这个模块会包含当前项目在当前平台下所需的运行时库。接着我们用一些CPack变量以设置当前项目的许可证及版本号。版本号在教程之前的步骤中已经设置，`license.txt`已经添加在源码目录的最高层。

最终我们引用[CPack模块](https://)以使用这些变量或者其他属性以我于安装包。

下一步就是按照通常习惯构建程序并运行cpack命令。生成一个二进制包，你需要在二进制目录运行：

```shell
cpack
```

若想指定生成器，使用`-G`选项。对于多配置的构建，使用`-C`指定配置，如下所示：

```shell
cpack -G ZIP -C Debug
```

如果想创建一个源码分发包你应该输入：

```shell
cpack --config CPackSourceConfig.cmake
```

作为替代，运行`make package`命令或者IDE右击`Package`目标并`编译工程。`

运行在二进制目录找的安装包，验证是否如预期。

## 添加对仪表板的支持（第八步）

将测试结果添加到仪表板很简单。在前面我们已经添加了一系列测试到项目中。现在我们必须运行这些测试并将结果添加到仪表板中。为与做到这点，在顶层`CMakeLists.txt`中引用`CTest`模块。

替换：

```cmake
# enable testing
enable_testing()
```

为：

```cmake
# enable dashboard scripting
include(CTest)
```

`CTest`模块可以自动调用`enable_testing()`，所以我们可以将它将CMake文件中删掉。

我们同样需要创建一个`CTestConfig.cmake`文件在顶层目录以提交到仪表板。

```cmake
set(CTEST_PROJECT_NAME "CMakeTutorial")
set(CTEST_NIGHTLY_START_TIME "00:00:00 EST")

set(CTEST_DROP_METHOD "http")
set(CTEST_DROP_SITE "my.cdash.org")
set(CTEST_DROP_LOCATION "/submit.php?project=CMakeTutorial")
set(CTEST_DROP_SITE_CDASH TRUE)
```

[ctest](https://)命令运行时会读取此文件。你可以运行[cmake](https://)命令或者用[cmake-gui](https://)去配置这项目，但没去构建它。替代的，修改二进制树目录，并运行：

```shell
ctest [-VV] -C Debug -D Experimental
```

不要忘了，对于多配置的生成器（比如Visual Studio），配置必须指定：

```shell
ctest [-VV] -C Debug -D Experimental
```

或者直接在IDE中编译`Experimental`目标。

[ctest](https://)命令将构建并将结果提交到Kitware的公共仪表板：[https://my.cdash.org/index.php?project=CMakeTutorial](https://my.cdash.org/index.php?project=CMakeTutorial)。

## 混合使用靜態庫和共享庫（第九步）

在本节中，我们将展示如何使用[BUILD_SHARED_LIBS](https://)变量来控制[add_library()](https://)的默认行为，并允许控制没有显式类型的库（`STATIC`、`SHARED`、`MODULE`或者`OBJECT`）是如何构建的。

为此，我们需要将[BUILD_SHARED_LIBS](https://)添加到顶层`CMakeLists.txt`中。我们使用[option()](https://)命令，因为它能用户选择值为ON或者OFF。

接下来，我们将重构MathFunctions，使其成为一个使用`mysqrt`或`sqrt`封装的真正的库，而不是要求在代码处理这些逻辑。这也意味着`USE_MYMATH`将不再控制构建MathFunctions，而是控制这个库的行为。

第一步是像下面那样更新顶层`CMakeLists.txt`：

```cmake
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# control where the static and shared libraries are built so that on windows
# we don't need to tinker with the path to run the executable
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")

option(BUILD_SHARED_LIBS "Build using shared libraries" ON)

# configure a header file to pass the version number only
configure_file(TutorialConfig.h.in TutorialConfig.h)

# add the MathFunctions library
add_subdirectory(MathFunctions)

# add the executable
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC MathFunctions)
```

现在我们已经使MathFunctions始终被使用，需要更新这个库的逻辑。因此，在`MathFunctions/CMakeLists.txt`中需要创建一个当`USE_MYMATH`被启用时有条件构建和安装的SqrtLibrary。现在，由于这是一个教程，我们明确要求SqrtLibrary是静态构建的。

`MathFunctions/CMakeLists.txt`最终应该像下面那样：

```cmake
# add the library that runs
add_library(MathFunctions MathFunctions.cxx)

# state that anybody linking to us needs to include the current source dir
# to find MathFunctions.h, while we don't.
target_include_directories(MathFunctions
                           INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
                           )

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if(USE_MYMATH)

  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")

  # first we add the executable that generates the table
  add_executable(MakeTable MakeTable.cxx)

  # add the command to generate the source code
  add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
    COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
    DEPENDS MakeTable
    )

  # library that just does sqrt
  add_library(SqrtLibrary STATIC
              mysqrt.cxx
              ${CMAKE_CURRENT_BINARY_DIR}/Table.h
              )

  # state that we depend on our binary dir to find Table.h
  target_include_directories(SqrtLibrary PRIVATE
                             ${CMAKE_CURRENT_BINARY_DIR}
                             )

  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()

# define the symbol stating we are using the declspec(dllexport) when
# building on windows
target_compile_definitions(MathFunctions PRIVATE "EXPORTING_MYMATH")

# install rules
set(installable_libs MathFunctions)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
install(TARGETS ${installable_libs} DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```

接下来使用`mathfunctions`函数及`detail`命令空间修改`MathFunctions/mysqrt.cxx`：

```cpp
#include <iostream>

#include "MathFunctions.h"

// include the generated table
#include "Table.h"

namespace mathfunctions {
namespace detail {
// a hack square root calculation using simple operations
double mysqrt(double x)
{
  if (x <= 0) {
    return 0;
  }

  // use the table to help find an initial value
  double result = x;
  if (x >= 1 && x < 10) {
    std::cout << "Use the table to help find an initial value " << std::endl;
    result = sqrtTable[static_cast<int>(x)];
  }

  // do ten iterations
  for (int i = 0; i < 10; ++i) {
    if (result <= 0) {
      result = 0.1;
    }
    double delta = x - (result * result);
    result = result + 0.5 * delta / result;
    std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
  }

  return result;
}
}
}
```

我们还需要在`tutorial.cxx`中做一些修改，使它不再使用`USE_MYMATH`：

1. 总是包括`MathFunctions.h`
2. 总是使用`mathfunctions::sqrt`
3. 不要包含cmath

最后，更新`MathFunctions/MathFunctions.h`以使用dll导出的定义：

```cpp
#if defined(_WIN32)
#  if defined(EXPORTING_MYMATH)
#    define DECLSPEC __declspec(dllexport)
#  else
#    define DECLSPEC __declspec(dllimport)
#  endif
#else // non windows
#  define DECLSPEC
#endif

namespace mathfunctions {
double DECLSPEC sqrt(double x);
}
```

此时，如果您构建了所有内容，您可能会注意到，当我们将一个没有位置独立代码的静态库与一个有位置独立代码的库组合在一起时，链接会失败。解决这个问题的方法是显式地将SqrtLibrary的[POSITION_INDEPENDENT_CODE](https://)目标属性设置为True，不管构建类型是什么。

```cmake
  # state that SqrtLibrary need PIC when the default is shared libraries
  set_target_properties(SqrtLibrary PROPERTIES
                        POSITION_INDEPENDENT_CODE ${BUILD_SHARED_LIBS}
                        )

  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
```

**练习**：我们修改了`MathFunctions.h`以使用dll导出的定义。使用CMake文档你能找到一个帮助模块来简化这个吗?

## 添加生成器表达式（第十步）

[生成器表达式](Generator expressions)在生成生成系统期间计算，以生成特定于每个生成配置的信息。

[生成器表达式](https://)可以在许多目标属性的上下文中使用，比如[LINK_LIBRARIES](https://)、[INCLUDE_DIRECTORIES](https://)、[COMPILE_DEFINITIONS](https://)等。它们也可以在使用命令填充那些属性时使用，比如[target_link_libraries()](https://)、[target_include_directories()](https://)、[target_compile_definitions()](https://)等。

[生成器表达式](https://)可用于启用条件链接、编译时使用的条件定义、条件包含目录等等。这些条件可能基于构建配置、目标属性、平台信息或任何其他可查询的信息。

[生成器表达式](https://)有不同的类型，包括逻辑表达式、信息表达式和输出表达式

逻辑表达式用于创建条件输出。基本表达式是0和1表达式。`$<0:...>`结果为空字符串，并且`<1:...>`导致“…”的内容。它们也可以嵌套。

[生成器表达式](https://)的常见用法是有条件地添加编译器标志，例如用于语言级别或警告的标志。一个不错的模式是将此信息关联到允许传播此信息的`INTERFACE`接口目标。让我们首先构造一个`INTERFACE`目标，并指定所需的c++标准级别`11`，而不是使用[CMAKE_CXX_STANDARD](https://)。

所以下面的代码：

```cmake
# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

会被替换成：

```cmake
add_library(tutorial_compiler_flags INTERFACE)
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
```

接下来，我们为项目添加所需的编译器警告标志。由于警告标志会根据编译器的不同而变化，因此我们使用`COMPILE_LANG_AND_ID`生成器表达式来控制在给定的语言和一组编译器id中应用哪些标志，如下所示：

```cmake
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
```

我们看到警告标志被封装在`BUILD_INTERFACE`条件中。这样做是为了使已安装项目的使用者不会继承我们的警告标志。

**练习**：修改`MathFunctions/cmakellists.txt`，使所有目标都有一个[target_link_libraries()](https://)调用`tutorial_compiler_flags`。

## 添加导出配置（第十一步）

在[安装和测试（第四步）](https://)教程中，我们增加了CMake安装项目库和头文件的能力。在[构建安装程序（第七步）](https://)期间，我们添加了打包这些信息的功能，以便将其分发给其他人。

下一步是添加必要的信息，以便其他CMake项目可以使用我们的项目，无论是在构建目录、本地安装还是打包时。

第一步是更新我们的[install(TARGETS)](https://)命令，不仅指定`DESTINATION`，还指定`EXPORT`。`EXPORT`关键字生成并安装一个CMake文件，其中包含从安装树导入安装命令中列出的所有目标的代码。所以让我们继续，通过更新`MathFunctions/CMakeLists.txt`中的`install`命令来显式`EXPORT`MathFunctions库，如下所示：

```cmake
set(installable_libs MathFunctions tutorial_compiler_flags)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
install(TARGETS ${installable_libs}
        DESTINATION lib
        EXPORT MathFunctionsTargets)
install(FILES MathFunctions.h DESTINATION include)
```

现在我们已经导出了MathFunctions，我们还需要显式安装生成的`MathFunctionsTargets.cmake`文件。这是通过在顶层`CMakeLists.txt`的底部添加以下内容来实现的：

```cmake
install(EXPORT MathFunctionsTargets
  FILE MathFunctionsTargets.cmake
  DESTINATION lib/cmake/MathFunctions
)
```

此时，您应该尝试运行CMake。如果一切都设置正确，你会看到CMake将产生一个错误，看起来像：

```shell
Target "MathFunctions" INTERFACE_INCLUDE_DIRECTORIES property contains
path:

  "/Users/robert/Documents/CMakeClass/Tutorial/Step11/MathFunctions"

which is prefixed in the source directory.
```

CMake试图说明的是，在生成导出信息的过程中，它将导出一个本质上与当前机器相关联的路径，该路径在其他机器上无效。解决这个问题的方法是更新MathFunctions [target_include_directories()](https://)，以理解在构建目录和安装/包中使用它时需要不同的`INTERFACE`位置。这意味着将MathFunctions的[target_include_directories()](https://)调用转换成如下所示：

```cmake
target_include_directories(MathFunctions
                           INTERFACE
                            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                            $<INSTALL_INTERFACE:include>
                           )
```

一旦它被更新，我们可以重新运行CMake并验证它不再发出警告。

此时，我们已经让CMake正确地打包了所需的目标信息，但我们仍然需要生成`MathFunctionsConfig.cmake`。让cmake的[find_package()](https://)命令可以找到我们的项目。因此，让我们继续往项目的顶层添加一个名为`Config.cmake.in`的新文件。内附以下内容：

```cmake

@PACKAGE_INIT@

include ( "${CMAKE_CURRENT_LIST_DIR}/MathFunctionsTargets.cmake" )
```

然后，为了正确地配置和安装该文件，将以下文件添加到顶层`CMakeLists.txt`的底部：

```cmake
install(EXPORT MathFunctionsTargets
  FILE MathFunctionsTargets.cmake
  DESTINATION lib/cmake/MathFunctions
)

include(CMakePackageConfigHelpers)
# generate the config file that is includes the exports
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake"
  INSTALL_DESTINATION "lib/cmake/example"
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
  )
# generate the version file for the config file
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake"
  VERSION "${Tutorial_VERSION_MAJOR}.${Tutorial_VERSION_MINOR}"
  COMPATIBILITY AnyNewerVersion
)

# install the configuration file
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake
  DESTINATION lib/cmake/MathFunctions
  )
```

至此，我们已经为我们的项目生成了一个可重定位的CMake配置，可以在安装或打包项目之后使用。如果我们想要我们的项目也从一个构建目录中使用，我们只需要添加以下顶层`CMakeLists.txt`的底部：

```cmake
export(EXPORT MathFunctionsTargets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsTargets.cmake"
)
```

使用这个导出调用，我们现在生成一个`Targets.cmake`，允许配置`MathFunctionsConfig.cmake`文件，以供其他项目使用，无需安装。

## 打包调试和发布（第十二步）

**注意**：这个例子只适用于单配置生成器，而不适用于多配置生成器(例如Visual Studio)。

默认情况下，CMake的模型是一个构建目录只包含一个配置，可以是Debug、Release、MinSizeRel或RelWithDebInfo。但是，可以通过安装CPack来捆绑多个构建目录，并构建一个包含同一项目的多个配置的包。

首先，我们希望确保调试版本和发布版本对将要安装的可执行文件和库使用不同的名称。让我们使用*d*作为调试可执行文件和库的后缀。

在顶层`CMakeLists.txt`文件的开头设置[CMAKE_DEBUG_POSTFIX](https://)：

```cmake
set(CMAKE_DEBUG_POSTFIX d)

add_library(tutorial_compiler_flags INTERFACE)
```

和tutorial可执行文件上的[DEBUG_POSTFIX](https://)属性：

```cmake
add_executable(Tutorial tutorial.cxx)
set_target_properties(Tutorial PROPERTIES DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})

target_link_libraries(Tutorial PUBLIC MathFunctions)
```

让我们再向MathFunctions库添加版本号。在`MathFunctions/CMakeLists.txt`设置[VERSION](https://)和[SOVERSION](https://)属性：

```cmake
set_property(TARGET MathFunctions PROPERTY VERSION "1.0.0")
set_property(TARGET MathFunctions PROPERTY SOVERSION "1")
```

在`Step12`目录中，创建`debug`和`release`子目录。布局将看起来像：

```shell
- Step12
   - debug
   - release
```

现在我们需要设置调试和发布构建。我们可以使用[CMAKE_BUILD_TYPE](https://)来设置配置类型：

```shell
cd debug
cmake -DCMAKE_BUILD_TYPE=Debug ..
cmake --build .
cd ../release
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build .
```

现在调试版本和发布版本都已经完成了，我们可以使用一个定制的配置文件将这两个版本打包到一个版本中。在`Step12`目录中，创建一个名为`MultiCPackConfig.cmake`的文件。在这个文件中，首先包含[cmake](https://)可执行文件创建的默认配置文件。

接下来，使用`CPACK_INSTALL_CMAKE_PROJECTS`变量来指定要安装哪些项目。在这种情况下，我们希望同时安装调试和发布。

```cmake
include("release/CPackConfig.cmake")

set(CPACK_INSTALL_CMAKE_PROJECTS
    "debug;Tutorial;ALL;/"
    "release;Tutorial;ALL;/"
    )
```

在`Step12`目录下，运行[cpack](https://)，指定我们的配置文件`config`选项：

```shell
cpack --config MultiCPackConfig.cmake
```