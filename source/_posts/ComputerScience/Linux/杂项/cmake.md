## 从可执行文件到库目录

### 切换生成器

```shell
$cmake --help # cmake支持的生成器

# 1. 使用-G选项来切换生成器
$cmake -G "Gennerator" ..
# 2. 构建项目
$cmaek --build . ##将generator的命令封装到一个跨平台的接口中
```

### 构建和链接静态库

```cmake
## 构建静态库
add_library(static_lib_name lib_type lib_src)
# static_lib_name 为生成静态库的名字
# lib_type 为库的类型，如果构建的是静态库，则为STATIC
# lib_src 为构建静态库的源码

## 链接静态库
# 一般在add_executable后面使用
link_target_libraries(exe_name lib1 lib2 ...)
# 第一参数必须为可执行文件的名字
# 后面的参数为库的名字， 可以为动态库、静态库。。。
```

### 构建和链接动态库

```cmake
## 构建动态库
add_library(shared_name SHARED share_src)
## 链接动态库
link_target_libraries(exe_name lib1 lib2)
```

### add_library 第二参数的说明

- ==STATIC==: 用于创建静态库，即编译文件的打包存档
- ==SHARED==:用于创建动态库，即可以动态链接，并在运行时加载的库
- ==OBJECT==: 可将给定`add_library`的列表中的源码编译到目标文件中，不将他们归档到静态库中也不能将它们链接到共享对象中。如果需要一次性创建静态库和动态库，那么使用对象库尤其有用。
- ==MODULE==: 又为`DSO`组(动态共享对象组)。它们不链接到项目中的任何目标，不过可以进行动态加载。该参数可以用于构建运行时插件。

- ==IMPORTED==：此类库目标表示位于项目外部的库。此类库的主要用途是，对现有依赖项进行构建。

- ==INTERFACE==：与 IMPORTED 库类似。不过，该类型库可变，没有位置信息。它主要用于项目之外的目标构建使用

- ==ALIAS==：顾名思义，这种库为项目中已存在的库目标定义别名。不过，不能为 IMPORTED 库选择别名

#### 使用OBJECT参数来构建动态库和静态库

```cmake
cmake_minimum_required(VERSION 3.3 FATAL_ERROR)
project(demo)
# 构建对象库
add_library(message-objs 
    OBJECT 
        Message.hpp Message.cpp)

# 设置链接属性，在低版本的cmake中需要
set_target_properties(message-objs
    PROPERTIES 
        POSITION_INDEPENDENT_CODE 1
                    )

# 通过对象库来构建静态库
add_library(message-static
    STATIC 
        $<TARGET_OBJECTS:message-objs>
            )
# 通过对象库来构建动态库
add_library(message-shared
    SHARED
        $<TARGET_OBJECTS:message-objs>
            )

add_executable(main main.cpp)

target_link_libraries(main message-static)

```

`$<TARGET_OBJECTS:message-objs>` 生成器表达式

生成器表达式是`CMake`在生成时(即配置之后)构造，用于生成特定于配置的构建输出。

#### 使静态库和动态库都输出相同的名字

```cmake
cmake_minimum_required(VERSION 3.3 FATAL_ERROR)

project(demo LANGUAGES CXX)

# 构建对象库
add_library(message-objs 
    OBJECT 
        message.hpp message.cpp)

set_target_properties(message-objs 
    PROPERTIES 
        POSITION_INDEPENDENT_CODE 1)

# 通过对象库构建共享库
add_library(message-shared 
    SHARED
        $<TARGET_OBJECTS:message-objs>)

# 将共享库message-shared名字改为message
set_target_properties(message-shared
    PROPERTIES
        OUTPUT_NAME "message")

# 通过对象库构建静态库
add_library(message-static
    STATIC
        $<TARGET_OBJECTS:message-objs>)

# 将静态库message-static输出为message

set_target_properties(message-static
    PROPERTIES 
        OUTPUT_NAME "message")

add_executable(main main.cpp)

target_link_libraries(main  message-static)
```

### 用条件语句控制编译

#### 语法

```cmake
# 用法一
IF(value)
ELSE()
ENDIF()
# 用法二
IF(value)
ELSE(value)
ENDIF(value)

```

#### 用`if-else`语句实现在不同的两种行为之间进行切换

1. 将`Message.hpp`和`Message.cpp`构建成一个库，然后将生成库链接到`main`可执行文件种
2. 将`Message.hpp`，`Message.cpp`和`main.cpp`构建成一个可执行文件，但不生成库文件

```cmake
# 1. 定义最低cmake版本、项目名称和支持的语言
cmake_minimum_required(VERSION 3.3 FATAL_ERROR)
project(demo LANGUAGES CXX)

# 2. 定义一个变量USE_LIBRARY, 设置为OFF, 并打印输出
set(USE_LIBRARY OFF) # USE_LIBRARY为一个逻辑变量
message(STATUS 
    "Compile suorces into a library?${USE_LIBRARY}")

# 3. 设置全局变量BUILD_SHARED_LIBS为OFF后，
#    add_library的第二个省略后，将构建一个静态库
set(BUILD_SHARED_LIBS OFF) # BUILD_SHARED_LIBS为一个全局的逻辑变量

# 4. 设置一个列表变量_source, 用来存放message的源代码
list(APPEND _source Message.hpp Message.cpp)

# 5. 用if-else语句来控制编译流程
if(USE_LIBRARY)
    add_library(message ${_SOURCE})
    add_executable(main main.cpp)
    target_link_libraries(main message)
else()
    add_executable(main main.cpp ${_source})
endif()
```

### 逻辑变量

- 如果将逻辑变量设置为以下任意一种： 1 、 ON 、 YES 、 true 、 Y 或非零数，则逻辑变量为 true 。
- 如果将逻辑变量设置为以下任意一种： 0 、 OFF 、 NO 、 false 、 N 、 IGNORE、NOTFOUND 、空字符串，或者以 -NOTFOUND 为后缀，则逻辑变量为 false 。

### 向用户显示选项

用 `option() `命令，以选项的形式显示逻辑开关，用于外部设置，从而切换构建系统的生成行为。然后用`-D`选项改变逻辑开关的状态。

`-D `开关用于为`CMake`设置任何类型的变量：逻辑变量、路径等等。

#### option()的工作原理

option()函数原型

```cmake
option(<option_variable> "help message" [initial_value])
```

通过option函数可以设置一个选项变量，并给出提示信息和初始化值

`<option_variable>` 表示选项变量

`"help message"` 为选项变量提供说明 

`[initial_value]` 为可选项，为选项变量初始化

#### option的应用

```cmake
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
project(demo)
# 设置选项变量USE_LIBRARY并初始化为OFF
option(USE_LIBRARY "Compile sources into a library" OFF)
# 设置环境变量BUILD_SHARED_LIBS为OFF，此时add_library
# 的第二参数省略时，默认编译为静态库
set(BUILD_SHARED_LIBS OFF)
# 定义一个列表变量，用于存放源文件
list(APPEND _source Message.hpp Message.cpp)
# 构建对象库
add_library(message-objs OBJECT ${_source})
# 设置目标属性
set_target_properties(message-objs
    PROPERTIES
        POSITION_INDEPENDENT_CODE 1)       
# 用if-else来控制编译流程
if(USE_LIBRARY)
	# 通过对象库来构建静态库
	add_library(massage-static 
    	$<TARGET_OBJECTS:message-objs>)
	# 设置massage-static的输出名字
	set_target_properties(massage-static
    	PROPERTIES 
        	OUTPUT_NAME "message")
	# 通过对象库构建动态库
	add_library(message-shared 
  	  	SHARED 
  	      	$<TARGET_OBJECTS:message-objs>)
	# 设置message-shared的输出名字
	set_target_properties(message-shared 
    	PROPERTIES 
        	OUTPUT_NAME "message")
	# 添加可知文件
	add_executable(main main.cpp)
	# 目标链接库文件
	target_link_libraries(main massage-static)
else()
	add_executable(main main.cpp ${_source})
endif()
```

执行shell命令

```shell
$mkdir -p build
$cd build
$cmake -D USE_LIBARRY=ON ..
$cmake --build .
```

### 定义选项变量之间的依赖

#### `cmake_dependent_option`的原理

`cmake_dependent_option`通过==被依赖选项变量== 的值的设定来控制==依赖选项变量==的值。同时`cmake_dependent_option`可以创建新的选项变量

#### `cmake_dependent_option`的原型

```cmake
cmake_dependent_option(< dependent option variable> "help message" value1 "option_variable"  value2)
```

`<dependent option variable>` 为依赖选项变量

`"help message"` 为提示信息

`value1` 为依赖选项变量的值

`"option_variable"` 为被依赖选项变量 。

`value2`为被依赖选项变量的值

#### `cmake_dependent_option`的注意事项

> # `cmake_dependent_option`为`cmake`的一个名为`CMakeDependentOption`模块中的函数。
>
> # 在使用的使用要用`include`来包含当前模块

#### `cmake_dependent_option`的应用

```cmake
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
project(demo LANGUAGES CXX)

set(BUILD_SHARED_LIBS OFF)
list(APPEND _source Message.hpp Message.cpp)

option(USE_LIBRARY "Compile sources into libraries" OFF)

# 为BUILD_SHARED_LIBRARY和BUILD_STATIC_LIBRARY构建依赖
# 将扩展包CmakeDependentOption包含进来
include(CMakeDependentOption)
# 为BUILD_SHARED_LIBRARY设置依赖
cmake_dependent_option(
    BUILD_SHARED_LIBRARY "Compile sources into a shared library" ON
    "USE_LIBRARY" ON
)
# BUILD_STATIC_LIBRARY构建依赖
cmake_dependent_option(
    BUILD_STATIC_LIBRARY "Comile sources into a static library" OFF
    "USE_LIBRARY" ON
)

add_library(message-objs OBJECT ${_source})
set_target_properties(message-objs PROPERTIES POSITION_INDEPENDENT_CODE 1)

if(USE_LIBRARY)
    #构建静态库
    if(BUILD_STATIC_LIBRARY)
        add_library(message $<TARGET_OBJECTS:massage-objs>)
        set_target_properties(message PROPERTIES OUTPUT_NAME "message")
        add_executable(main main.cpp)
        target_link_libraries(main message)
    endif()
    #构建动态库
    if(BUILD_SHARED_LIBRARY)
        add_library(message SHARED $<TARGET_OBJECTS:message-objs)
        set_target_properties(message PROPERTIES OUTPUT_NAME "message")
        add_executable(main main.cpp)
        target_link_libraries(main message)
    endif()

else()
    add_executable(main main.cpp ${_source})
endif()
```

在shell中执行

```shell
$mkdir -p build
$cd build
$cmake .. # 直接生成可执行文件
$cmake -D USE_LIBRARY=ON # 生成动态库，并通过链接动态库生成可执行文件
$cmake -D USE_LIBRARY=OFF # 直接生成可执行文件
$cmake -D USE_LIBRARY=ON -D BUILD_SHARED_LIBRARY=OFF .. # 什么都不生成
$cmake -D USE_LIBRARY=ON -D BUILD_SHARED_LIBRARY=ON  .. # 生成动态库，并通过链接动态库生成可执行文件
$cmake -D USE_LIBRARY=ON -D BUILD_STATIC_LIBRARY=OFF .. # 生成动态库，并通过链接动态库生成可执行文件
$cmake -D USE_LIBRARY=ON -D BUILD_STATIC_LIBRARY=ON .. # 生成动态库，并通过链接动态库生成可执行文件1;生成静态库，并通过链接静态库生成可执行文件2
$cmake -D USE_LIBRARY=OFF -D BUILD_STATIC_LIBRARY=ON ..# 直接生成可执行文件
$cmake -D USE_LIBRARY=ON -D BUILD_STATIC_LIBRARY=ON -D BUILD_SHARED_LIBRARY=OFF .. # 生成静态库，并通过链接静态库生成可执行文件2
$cmake --build .

```

> # `>`+`#` to create a tip block
>
> ## `>`+`##` to warning block
>
> ### `>`+`###` to error block
>
> #### `>`+`####` to remind block

###   指定编译器

#### 指定编译器的方式

- 根据平台和生成器选择编译器

- 将编译器标志设置为默认值

`CMake`将语言的编译器存储在 `CMAKE_<LANG>_COMPILER` 变量中，其中`<LANG> `是受支持的任何一种语言，对于我们的目的是 CXX 、 C 或 Fortran 。

#### 设置`CMAKE_<LANG>_COMPILER`的方式

1. 使用`cmake`的`-D`选项

   ```shell
   $cmake -D CMAKE_CXX_COMPILER=clang++ ..
   ```

2. 通过环境变量

   ```shell
   $env CXX=clang++ cmake ..
   ```

> # 建议使用第一种方式

配置时，`CMake`会进行一系列平台测试，以确定哪些编译器可用，以及它们是否适合当前的项目。一个合适的编译器不仅取决于我们所使用的平台，还取决于我们想要使用的生成器。`CMake`执行的第一个测试基于项目语言的编译器的名称

#### 查看编译器和编译器标志信息

```shell
$cmake --system-information 
```

#### `cmake`提供的格外编译变量

- `CMAKE_<LANG>_COMPILER_LOADED` :如果为项目启用了语言 `<LANG> `，则将设置为 `TRUE `。
- `CMAKE_<LANG>_COMPILER_ID `:编译器标识字符串，编译器供应商所特有。例如，` GCC `用于`GNU`编译器集合，` AppleClang` 用于`macOS`上的`Clang`, `MSVC` 用于`Microsoft Visual Studio`编译器。注意，不能保证为所有编译器或语言定义此变量。
- `CMAKE_COMPILER_IS_GNU<LANG> `:如果语言 <LANG> 是GNU编译器集合的一部分，则将此逻辑变量设置为 TRUE 。注意变量名的 `<LANG>` 部分遵循`GNU`约定：`C`语言为 `CC` ,` C++`语言
  为 `CXX` , `Fortran`语言为 `G77` 。
- `CMAKE_<LANG>_COMPILER_VERSION `:此变量包含一个字符串，该字符串给定语言的编译器版本。版本信息在 `major[.minor[.patch[.tweak]]]` 中给出。但是，对于 `CMAKE_<LANG>_COMPILER_ID `，不能保证所有编译器或语言都定义了此变量。

`CMAKE_<LANG>_COMPILER_LOADED` 逻辑变量 ，表式编译器是否加载

`CMAKE_<LANG>_COMPILER_ID` 字符串变量，表示编译的开发商

`CMAKE_COMPILER_IS_GNU<LANG>` 逻辑变量，表示是否属于GNU的编译器

`CMAKE_<LANG>_COMPILER_VERSION` 字符串变量， 表示编译器版本

```cmake
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
project(cmake_test_version_5 LANGUAGES C CXX)

# 获取CMAKE_CXX_COMPILER_LOADED的值,并输出
message(STATUS "Is the C++ compiler loadeds? ${CMAKE_CXX_COMPILER_LOADED}")
if(CMAKE_CXX_COMPILER_LOADED)
    message(STATUS "The c++ compiler ID is: ${CMAKE_CXX_COMPILER_ID}")
    message(STATUS "Is the c++ from GNU? ${CMAKE_COMPILER_IS_GNUCXX}")
    message(STATUS "The c++ compiler version is: ${CMAKE_CXX_COMPILER_VERSION}")
endif()

# 获取CMAKE_CXX_COMPILER_LOADED的值,并输出
message(STATUS "Is the C compiler loadeds? ${CMAKE_C_COMPILER_LOADED}")
if(CMAKE_CXX_COMPILER_LOADED)
    message(STATUS "The C compiler ID is: ${CMAKE_C_COMPILER_ID}")
    message(STATUS "Is the C from GNU? ${CMAKE_COMPILER_IS_GNUC  }")
    message(STATUS "The C compiler version is: ${CMAKE_C_COMPILER_VERSION}")
endif()
```



### 切换构建类型

`CMake`可以配置构建类型，例如：Debug、Release等。配置时，可以为Debug或Release构建设置相关的选项或属性，例如：编译器和链接器标志。==控制生成构建系统==使用的配置变量是 `CMAKE_BUILD_TYPE `。该变量默认为空，`CMake`识别的值为:

1.  `Debug`：用于在没有优化的情况下，使用带有调试符号构建库或可执行文件。
2.  `Release`：用于构建的优化的库或可执行文件，不包含调试符号。
3. `RelWithDebInfo`：用于构建较少的优化库或可执行文件，包含调试符号。
4. `MinSizeRel`：用于不增加目标代码大小的优化方式，来构建库或可执行文件。

`CMAKE_BUILD_TYPE` 设置项目的构建类型

`CMAKE_<LANG>_FLAGS_DEBUG` debug的配置标志

`CMAKE_<LANG>_FLAGS_RELEASE` release的配置标志标志

`CMAKE_<LANG>_FLAGS_RELWITHDEBINFO` release with debug information的配置标志

`CMAKE_<LANG>_FLAGS_MINSIZEREL` minimal size release 的配置标志

```cmake
# 通过CMAKE_BUILD_TYPE来设置构建类型
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif()
# 打印构建类型
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
# 打印出CMAKE设置的对应编译标志
message(STATUS "C flags, Debug configuration: ${CMAKE_C_FLAGS_DEBUG}")
message(STATUS "C flags, Release configuration: ${CMAKE_C_FLAGS_RELEASE}")
message(STATUS "C flags, Release configuration with Debug info: ${CMAKE_C_FLAGAS_RELWITHDEBINFO}")
message(STATUS "C flags, minimal Release configuration: ${CMAKE_C_FLAGS_MINSIZEREL}")
message(STATUS "C++ flags, Debug configuration: ${CMAKE_CXX_FLAGS_DEBUG}")
message(STATUS "C++ flags, Release configuration: ${CMAKE_CXX_FLAGS_RELEASE}")
message(STATUS "C++ flags, Release configuration with Debug info: ${CMAKE_CXX_FLAGAS_RELWITHDEBINFO}")
message(STATUS "C++ flags, minimal Release configuration: ${CMAKE_CXX_FLAGS_MINSIZEREL}")
```

使用默认构建类型

```shell
$ mdkir -p build
$ cd build
$ cmake ..

-- Build type: Release
-- C flags, Debug configuration: /MDd /Zi /Ob0 /Od /RTC1
-- C flags, Release configuration: /MD /O2 /Ob2 /DNDEBUG
-- C flags, Release configuration with Debug info:
-- C flags, minimal Release configuration: /MD /O1 /Ob1 /DNDEBUG
-- C++ flags, Debug configuration: /MDd /Zi /Ob0 /Od /RTC1
-- C++ flags, Release configuration: /MD /O2 /Ob2 /DNDEBUG
-- C++ flags, Release configuration with Debug info:
-- C++ flags, minimal Release configuration: /MD /O1 /Ob1 /DNDEBUG
```

> ## default generator is `Visual Studio 17 2022`

使用Debug类型进行构建

```shell
$cmake -D CMAKE_BUILD_TYPE=Debug ..

-- Build type: Debug
-- C flags, Debug configuration: /MDd /Zi /Ob0 /Od /RTC1
-- C flags, Release configuration: /MD /O2 /Ob2 /DNDEBUG
-- C flags, Release configuration with Debug info:
-- C flags, minimal Release configuration: /MD /O1 /Ob1 /DNDEBUG
-- C++ flags, Debug configuration: /MDd /Zi /Ob0 /Od /RTC1
-- C++ flags, Release configuration: /MD /O2 /Ob2 /DNDEBUG
-- C++ flags, Release configuration with Debug info:
-- C++ flags, minimal Release configuration: /MD /O1 /Ob1 /DNDEBUG
```

总结：

1. 通过设置默认构建类型，来达到控制项目，使用优化编译，还是关闭优化启用调试。
2. 通过`CMakeLists.txt`文件中设置的默认构建类型，可用通过命令行`-D CMAKE_BUILD_TYPE=<build_type>`来进行覆盖

**为不同的编译器和不同的构建类型，扩展或调整编译器标志。**



### 复合配置生成器

单配置生成器：`Unix Makefiles` `Ninja` `MSYS Makefiles`等

多配置生成器：`Visual Studio` `Xcode`

使用cmake的复合配置生成器变量`CMAKE_CONFIGURATION_TYPES`来生成多个版本项目

`CMAKE_CONFIGURATION_TYPES`为一个列表变量，需要传入一个值列表

例子：对Visual Studio 的`cmake`调用

```shell
$mkdir -p build
$cd build
$ cmake -G "Visual Studio 17 2022" -D CMAKE_CONFIGURATION_TYPES="Rlease;Debug"
```

将为Release和Debug配置生成一个构建树。然后，您可以使 --config 标志来决定构建这两个中的哪一个:

```shell
cmake --build . --config Release
```

> #### 当使用单配置生成器开发代码时，为Release版和Debug创建单独的构建目录，两者使用相同的源代码。这样，就可以在两者之间切换，而不用重新配置和编译。

> # 单配置生成器不可以使用多配置选项

### 设置编译器选项

`CMAKE_<LANG>_FLAGS_<CONFIG>` 是一个全局变量，用于设置编译器选项

#### 使用`target_compile_options`来添加编译选项

```cmake
target_compile_options(
	<name>
	<visiblity>
	<compile options>
	)
<name> --[lib|project|exe]
<visiblity>--[private|public|interface]
<compile options>--[-fPIC|-Wall|-Wextra|-O2|...]
```

#### `cmake`中编译器可见性

- PRIVATE: 编译选项只会应用指定目标，不会传递与目标相关的目标
- PUBLIC：编译选项会应用于指定目标和使用它的目标
- INTERFACE：编译选项将只应用于指定目标，并传递给目标相关的目标

```cmake
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
project(geometry LANGUAGES C CXX)
# 打印当前标志
message(STATUS "C++ compiler flags: ${CMAKE_CXX_FLAGS}")
list(APPEND _source 
        geometry_circle.hpp
        geometry_circle.cpp
        geometry_polygon.hpp
        geometry_polygon.cpp
        geometry_rhombus.hpp
        geometry_rhombus.cpp
        geometry_square.hpp
        geometry_square.cpp
    )
# 对于无法在windows上使用的编译标志我们需要进行处理
list(APPEND _flags
        "-fPIC"
        "-Wall"
    )
if(NOT WIN32)
    list(APPEND _flags "-Wextra" "-Wpedantic")
endif()
add_library(geometry STATIC ${_source})
# 为geometry设置编译选项
target_compile_options(geometry
    PRIVATE 
        ${_flags}
    )
add_executable(compute_area compute_area.cpp)
# 为compute_area设置编译选项
target_compile_options(compute_area
    PRIVATE
        "-fPIC"
    )
target_link_libraries(compute_area geometry)
```

#### 查看构建时的编译标志

1. 用命令构建时，设置环境变量`VERBOSE=1`

   ```shell
   $cmake --build . -- VERBOSE=1
   ```

2. 使用`CMAKE_<LANG>_FLAGS` 

   ```shell
   $cmake -D CMAKE_CXX_FLAGS="-fno-exceptions -fno-rtti"
   ```

3. 在`CMakeLists.txt`文件中使用`target_compile_options`

   ```cmake
   target_compile_options(
   	geometry
   	PRIVATE
   	"-fno-rtti -fpic -Wall -Wextra -wpedantic"
   )
   target_compile_options(
   	geometry
   	PRIVATE
   	"-fno-exception -fno-rtti -fpic"
   )
   ```

   

#### 跨平台时为不同的平台设置特定的编译标志

1. 使用`CMAKE_<LANG>_FLAGS_<CONFIGS>`。需要确定的是，给定的编译标志在特定的平台是有效的。

   使用`CMAKE_<LANG>_COMPILER_ID`来确定使用的是那一个生产厂商的编译器。如，`GNU`, `Clang`

   ```cmake
   1. if(CMAKE_CXX_COMPILER_ID MATCHES GNU)
   2. list(APPEND CMAKE_CXX_FLAGS "-fno-rtti" "-fno-exceptions")
   3. list(APPEND CMAKE_CXX_FLAGS_DEBUG "-Wsuggest-final-types" "-	Wsuggest-finalmethods" "-Wsuggest-override")
   4. list(APPEND CMAKE_CXX_FLAGS_RELEASE "-O3" "-Wno-unused")
   5. endif()
   6. if(CMAKE_CXX_COMPILER_ID MATCHES Clang)
   7. list(APPEND CMAKE_CXX_FLAGS "-fno-rtti" "-fno-exceptions" "-	Qunusedarguments""-fcolor-diagnostics")
   8. list(APPEND CMAKE_CXX_FLAGS_DEBUG "-Wdocumentation")
   9. list(APPEND CMAKE_CXX_FLAGS_RELEASE "-O3" "-Wno-unused")
   10. endif()
   ```

   

2. 定义特定的标志列表

```cmake
set(COMPILER_FLAGS)
set(COMPILER_FLAGS_DEBUG)
set(COMPILER_FLAGS_RELEASE)
if(CMAKE_CXX_COMPILER_ID MATCHES GNU)
list(APPEND CXX_FLAGS "-fno-rtti" "-fno-exceptions")
list(APPEND CXX_FLAGS_DEBUG "-Wsuggest-final-types" "-Wsuggest-		final-methods" "-Wsuggest-override")
list(APPEND CXX_FLAGS_RELEASE "-O3" "-Wno-unused")
endif()
if(CMAKE_CXX_COMPILER_ID MATCHES Clang)
list(APPEND CXX_FLAGS "-fno-rtti" "-fno-exceptions" "-Qunused-	arguments" "-fcolor-diagnostics")
list(APPEND CXX_FLAGS_DEBUG "-Wdocumentation")
list(APPEND CXX_FLAGS_RELEASE "-O3" "-Wno-unused")
endif()
target_compile_option(compute-areas
	PRIVATE
 	${CXX_FLAGS}
	"$<$<CONFIG:Debug>:${CXX_FLAGS_DEBUG}>"
 	"$<$<CONFIG:Release>:${CXX_FLAGS_RELEASE}>"
 )

```



- `CMAKE_<LANG>_COMPILER_ID `不能保证为所有编译器都定义。
- 一些标志可能会被弃用，或者在编译器的较晚版本中引入。
- CMAKE_<LANG>_COMPILER_VERSION 变量不能保证为所有语言和供应商都提供定义。
- 检查所需的标志集是否与给定的编译器一起工作，这样项目中实际上只使用有效的标志。(更健壮的替代方法)

### 为语言定义标准

`<lang>_STANDARD` 为一个属性变量，所以需要使用`set_target_properties`来设置它

#### 使用`<lang>_STANDARD`的步骤

1. 要求在Windows上导出所有库符号
2. 使用`set_target_properties`为你的库文件和可执行文件添加`CXX_STANDARD 14`, `CXX_EXTENSIONS OFF`, `CXX_STANDARD_REQUIRED ON`属性，另外还需要给库文件添加`POSITION_INDEPENDENT_CODE 1`属性

#### 以上设置的目标属性的含义

- `CXX_STANDARD`: 设置需要的标准，如`C++98`,`C++11`, `C++14` 等

- `CXX_EXTENSIONS `：如果设置为`OFF`, 就会只启用`ISO C++`标准的编译器标志，而不使用特定编译器的扩展

- `CXX_STANDARD_REQUIRED ON`：指定所选标准的版本。如果所选版本不可用，`cmake`会停止配置并出现错误。当被设置为`OFF`时，`cmake`会自动去寻找最新的适合当前要求的标准。

> # 如果语言标准是所有目标共享的全局属性，那么可以将`CMAKE_<LANG>_STANDARD `、 `CMAKE_<LANG>_EXTENSIONS `和 `CMAKE_<LANG>_STANDARD_REQUIRED` 变量设置为相应的值。所有目标上的对应属性都将使用这些设置。

### 控制流

- `while-endwhile`
- `foreach-endforeach`

#### 列表

- 列表是用分号分隔的字符串组，如`a;b;c;d`

- 列表可以由 list 或 set 命令创建。如`list(APPEND var a b c)`, `set(var a b c)`

#### `foreach-endforeach`的四种用法

- `foreach(loop_var arg1 arg2 ...)`或`foreach(lop_var ${list_var})`, 显式列表循环。
- `foreach(loop_var range total)`或`froeach(loop_var start stop [step])`, 指定范围循环，一般都是对整数进行循环
- `foreach(loop_var IN LISTS [list1|...])`, 对列表变量循环，参数解释为列表，其内容自动展开
- `foreach(loop_var IN ITEMS [item1|...])`, 对变量循环，参数的内容不会展开

## 环境监测

### 检查操作系统

使用`CMAKE_SYSTEM_NAME`来检测操作系统

```cmake
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
	message(STATUS "Configuring on/for Linux")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
	message(STATUS "Configuring on/for macOS")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Windows")
	message(STATUS "Configuring on/for Windows")
elseif(CMAKE_SYSTEM_NAME STREQUAL "AIX")
	message(STATUS "Configuring on/for IBM AIX")
else()
	message(STATUS "Configuring on/for ${CMAKE_SYSTEM_NAME}")
endif()
```

### 处理与平台相关的源码

源码

```cmake
#include <cstdlib>
#include <iostream>
#include <string>

std::string say_hello() {
#ifdef IS_WINDOWS
	return std::string("Hello from Windows!");
 #elif IS_LINUX
	return std::string("Hello from Linux!");
#elif IS_MACOS
	return std::string("Hello from macOS!");
#else
	return std::string("Hello from an unknown system!");
#endif
}

int main() {
	std::cout << say_hello() << std::endl;
	return EXIT_SUCCESS;
}
```

`CMakeLists.txt`的与处理与平台相关的源码相关的部分代码

```cmake
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
	target_compile_definitions(hello-world PUBLIC "IS_LINUX")
endif()
if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
	target_compile_definitions(hello-world PUBLIC "IS_MACOS")
endif()
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
	target_compile_definitions(hello-world PUBLIC "IS_WINDOWS")
endif()
```

使用`CMAKE_SYSTEM_NAME`来判断操作系统，然后使用`target_compile_definitions`来定义源码中的编译指令。我们可以定义可见性来限定目标，如`PRIVATE`, `PUBLIC`, `INTERFACE`

我们也可以使用`add_definitions`来定义源码中的编译指令，但是它的缺点是会修改编译整个项目的定义。

### 处理与编译器相关的源码

源码

```c++
1. #include <cstdlib>
2. #include <iostream>
3. #include <string>
4.
5. std::string say_hello() {
6. #ifdef IS_INTEL_CXX_COMPILER
7. // only compiled when Intel compiler is selected
8. // such compiler will not compile the other branches
9. return std::string("Hello Intel compiler!");
10. #elif IS_GNU_CXX_COMPILER
11. // only compiled when GNU compiler is selected
12. // such compiler will not compile the other branches
13. return std::string("Hello GNU compiler!");
14. #elif IS_PGI_CXX_COMPILER
15. // etc.
16. return std::string("Hello PGI compiler!");
17. #elif IS_XL_CXX_COMPILER
18. return std::string("Hello XL compiler!");
19. #else
20. return std::string("Hello unknown compiler - have we met before?");
21. #endif
22. }
23.
24. int main() {
25. std::cout << say_hello() << std::endl;
26. std::cout << "compiler name is " COMPILER_NAME << std::endl;
27. return EXIT_SUCCESS;
28. }
```

```cmake
target_compile_definitions(hello-world PUBLIC
"COMPILER_NAME=\"${CMAKE_CXX_COMPILER_ID}\"")
2.
3. if(CMAKE_CXX_COMPILER_ID MATCHES Intel)
4. target_compile_definitions(hello-world PUBLIC "IS_INTEL_CXX_COMPILER")
5. endif()
6. if(CMAKE_CXX_COMPILER_ID MATCHES GNU)
7. target_compile_definitions(hello-world PUBLIC "IS_GNU_CXX_COMPILER")
8. endif()
9. if(CMAKE_CXX_COMPILER_ID MATCHES PGI)
10. target_compile_definitions(hello-world PUBLIC "IS_PGI_CXX_COMPILER")
11. endif()
12. if(CMAKE_CXX_COMPILER_ID MATCHES XL)
13. target_compile_definitions(hello-world PUBLIC "IS_XL_CXX_COMPILER")
14. endif()
```



### 检测处理器体系结构

```c++
1. #include <cstdlib>
2. #include <iostream>
3. #include <string>
4.
5. #define STRINGIFY(x) #x
6. #define TOSTRING(x) STRINGIFY(x)
7.
8. std::string say_hello()
9. {
10. std::string arch_info(TOSTRING(ARCHITECTURE));
11. arch_info += std::string(" architecture. ");
12. #ifdef IS_32_BIT_ARCH
13. return arch_info + std::string("Compiled on a 32 bit host processor.");
14. #elif IS_64_BIT_ARCH
15. return arch_info + std::string("Compiled on a 64 bit host processor.");
16. #else
17. return arch_info + std::string("Neither 32 nor 64 bit, puzzling ...");
18. #endif
19. }
20.
21. int main()
22. {
23. std::cout << say_hello() << std::endl;
24. return EXIT_SUCCESS;
25. }
```

`CMAKE_SIZEOF_VOID_P EQUAL`的使用

```cmake
1. if(CMAKE_SIZEOF_VOID_P EQUAL 8)
2. target_compile_definitions(arch-dependent PUBLIC "IS_64_BIT_ARCH")
3. message(STATUS "Target is 64 bits")
4. else()
5. target_compile_definitions(arch-dependent PUBLIC "IS_32_BIT_ARCH")
6. message(STATUS "Target is 32 bits")
7. endif()
```

让预处理器了解主机处理器架构, 用`CMAKE_HOST_SYSTEM_PROCESSOR`来检测系统的体系结构

```cmake
1. if(CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "i386")
2. message(STATUS "i386 architecture detected")
3. elseif(CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "i686")
4. message(STATUS "i686 architecture detected")
5. elseif(CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "x86_64")
6. message(STATUS "x86_64 architecture detected")
7. else()
8. message(STATUS "host processor architecture is unknown")
9. endif()
10. target_compile_definitions(arch-dependent
11. PUBLIC "ARCHITECTURE=${CMAKE_HOST_SYSTEM_PROCESSOR}"
12. )
```



### 检测处理器指令集

通过配置`config.h.in`来定义由`cmake`查询到的机器中的处理器指令集的信息，然后由`processor-info.cpp`来显示。

`config.h.in`中的内容如下所示

```c++
#pragma once

#define NUMBER_OF_LOGICAL_CORES @_NUMBER_OF_LOGICAL_CORES@
#define NUMBER_OF_PHYSICAL_CORES @_NUMBER_OF_PHYSICAL_CORES@
#define TOTAL_VIRTUAL_MEMORY @_TOTAL_VIRTUAL_MEMORY@
#define AVAILABLE_VIRTUAL_MEMORY @_AVAILABLE_VIRTUAL_MEMORY@
#define TOTAL_PHYSICAL_MEMORY @_TOTAL_PHYSICAL_MEMORY@
#define AVAILABLE_PHYSICAL_MEMORY @_AVAILABLE_PHYSICAL_MEMORY@
#define IS_64BIT @_IS_64BIT@
#define HAS_FPU @_HAS_FPU@
#define HAS_MMX @_HAS_MMX@
#define HAS_MMX_PLUS @_HAS_MMX_PLUS@
#define HAS_SSE @_HAS_SSE@
#define HAS_SSE2 @_HAS_SSE2@
#define HAS_SSE_FP @_HAS_SSE_FP@
#define HAS_SSE_MMX @_HAS_SSE_MMX@
#define HAS_AMD_3DNOW @_HAS_AMD_3DNOW@
#define HAS_AMD_3DNOW_PLUS @_HAS_AMD_3DNOW_PLUS@
#define HAS_IA64 @_HAS_IA64@
#define OS_NAME "@_OS_NAME@"
#define OS_RELEASE "@_OS_RELEASE@"
#define OS_VERSION "@_OS_VERSION@"
#define OS_PLATFORM "@_OS_PLATFORM@"
```

`processor-info.cpp`的代码如下

```c++
#include "config.h"

#include <cstdlib>
#include <iostream>

int main()
{
    std::cout << "Number of logical cores: "
              << NUMBER_OF_LOGICAL_CORES << std::endl;
    std::cout << "Number of physical cores: "
              << NUMBER_OF_PHYSICAL_CORES << std::endl;
    std::cout << "Total virtual memory in megabytes: "
              << TOTAL_VIRTUAL_MEMORY << std::endl;
    std::cout << "Available virtual memory in megabytes: "
              << AVAILABLE_VIRTUAL_MEMORY << std::endl;
    std::cout << "Total physical memory in megabytes: "
              << TOTAL_PHYSICAL_MEMORY << std::endl;
    std::cout << "Available physical memory in megabytes: "
              << AVAILABLE_PHYSICAL_MEMORY << std::endl;
    std::cout << "Processor is 64Bit: "
              << IS_64BIT << std::endl;
    std::cout << "Processor has floating point unit: "
              << HAS_FPU << std::endl;
    std::cout << "Processor supports MMX instructions: "
              << HAS_MMX << std::endl;
    std::cout << "Processor supports Ext. MMX instructions: "
           << HAS_MMX_PLUS << std::endl;
    std::cout << "Processor supports SSE instructions: "
              << HAS_SSE << std::endl;
    std::cout << "Processor supports SSE2 instructions: "
              << HAS_SSE2 << std::endl;
    std::cout << "Processor supports SSE FP instructions: "
              << HAS_SSE_FP << std::endl;
    std::cout << "Processor supports SSE MMX instructions: "
              << HAS_SSE_MMX << std::endl;
    std::cout << "Processor supports 3DNow instructions: "
              << HAS_AMD_3DNOW << std::endl;
    std::cout << "Processor supports 3DNow+ instructions: "
              << HAS_AMD_3DNOW_PLUS << std::endl;
    std::cout << "IA64 processor emulating x86 : "
              << HAS_IA64 << std::endl;
    std::cout << "OS name: "
              << OS_NAME << std::endl;
    std::cout << "OS sub-type: "
              << OS_RELEASE << std::endl;
    std::cout << "OS build ID: "
              << OS_VERSION << std::endl;
    std::cout << "OS platform: "
              << OS_PLATFORM << std::endl;
    return EXIT_SUCCESS;
}
```

`CMakeLists.txt`

```cmake
cmake_mininum_required(VERSION 3.10 FATAL_ERROR)
project(processor-info
	VERSION 1.0.0
	DESCRIPTION "查询当前机器的处理器指令集"
	HOMEPAGE_URL "www.gunfirefc.top"
	LANGUAGES CXX)
add_executable(processor-info "")
target_sources(processor-info
	PRIVATE
		processor-info.cpp)
target_include_directories(processor-info
	PRIVATE
		${PROJECT_BINARY_DIR})
foreach(key IN ITEMS
  	    NUMBER_OF_LOGICAL_CORES
        NUMBER_OF_PHYSICAL_CORES
        TOTAL_VIRTUAL_MEMORY
        AVAILABLE_VIRTUAL_MEMORY
        TOTAL_PHYSICAL_MEMORY
        AVAILABLE_PHYSICAL_MEMORY
        IS_64BIT
        HAS_FPU
        HAS_MMX
        HAS_MMX_PLUS
        HAS_SSE
        HAS_SSE2
        HAS_SSE_FP
        HAS_SSE_MMX
        HAS_AMD_3DNOW
        HAS_AMD_3DNOW_PLUS
        HAS_IA64
        OS_NAME
        OS_RELEASE
        OS_VERSION
        OS_PLATFORM
        )
        cmake_host_system_information(RESULT _${key} QUERY ${key})
endforeach()
configure_file(config.h.in config.h @ONLY)
```

​        `cmake_host_system_information(RESULT _${key} QUERY ${key})`

`configure_file(config.h.in config.h @ONLY)`

构建源文件

```sh
$mkdir build && cd build
$cmake ..
$cmake --build ..
$./processor-info
# 如果想要查看不同platform和不同genarator的处理器指令集
$cmake -G "MinGW Makefiles" .. # 使用的是windows中的MinGW生成器来编译源文件
$cmake -G "Vuisual Studio 7" .. # 使用Macrosoft的生成器
# 如果不知道cmake支持那些生成器，可以使用如下命令
$cmake --help
```

编译结果:

在Linux中编译

```sh
Number of logical cores: 12
Number of physical cores: 6
Total virtual memory in megabytes: 2048
Available virtual memory in megabytes: 2048
Total physical memory in megabytes: 6218
Available physical memory in megabytes: 3860
Processor is 64Bit: 1
Processor has floating point unit: 1
Processor supports MMX instructions: 1
Processor supports Ext. MMX instructions: 0
Processor supports SSE instructions: 1
Processor supports SSE2 instructions: 1
Processor supports SSE FP instructions: 0
Processor supports SSE MMX instructions: 0
Processor supports 3DNow instructions: 0
Processor supports 3DNow+ instructions: 0
IA64 processor emulating x86 : 0
OS name: Linux
OS sub-type: 5.10.102.1-microsoft-standard-WSL2
OS build ID: #1 SMP Wed Mar 2 00:30:59 UTC 2022
OS platform: x86_64
```

在windows中的编译

```sh
Number of logical cores: 12
Number of physical cores: 6
Total virtual memory in megabytes: 13423
Available virtual memory in megabytes: 2797
Total physical memory in megabytes: 8047
Available physical memory in megabytes: 1046
Processor is 64Bit: 1
Processor has floating point unit: 1
Processor supports MMX instructions: 1
Processor supports Ext. MMX instructions: 0
Processor supports SSE instructions: 1
Processor supports SSE2 instructions: 1
Processor supports SSE FP instructions: 0
Processor supports SSE MMX instructions: 0
Processor supports 3DNow instructions: 0
Processor supports 3DNow+ instructions: 0
IA64 processor emulating x86 : 0
OS name: Windows
OS sub-type:  Professional
OS build ID:  (Build 19043)
OS platform: AMD64
```



### 为Eigen库使能向量化

$处理器的向量功能可以提高代码的性能，对于某些类型的运算更为高效，例如：线性代数$

使用开源库`Eigen`来展示，如何使能向量化，以便于使用线性代数的Eigen C++库加速可执行文件。

#### 安装`Eigen`库

1. [3.4.0 · libeigen / eigen · GitLab](https://gitlab.com/libeigen/eigen/-/releases/3.4.0)
2. 解压源码
3. Eigen库的两种安装方法
   1. 将Eigen源码中的Eigen目录直接复制到指定的工程目录中即可
   2. 用cmake编译

#### 使用Eigen库的一个样例

`linear-algebra.cpp`

```CPP
#include <chrono>
#include <iostream>

#include "Eigen/Dense"

EIGEN_DONT_INLINE 
double simple_function(Eigen::VectorXd &va, Eigen::VectorXd &vb)
{
    double d= va.dot(vb);
    return d;
}

int main() 
{
    int len=1000000;
    int num_repetitions = 100;

    Eigen::VectorXd va = Eigen::VectorXd::Random(len);
    Eigen::VectorXd vb = Eigen::VectorXd::Random(len);

    double result;
    auto start = std::chrono::system_clock::now();
    for(auto i = 0; i < num_repetitions;i++)
    {
        result = simple_function(va, vb);
    }
    auto end = std::chrono::system_clock::now();
    auto elapsed_senconds = end-start;

    std::cout <<"result: "<<result<<std::endl;
    std::cout << "elapsed seconds: "<<elapsed_senconds.count()<<std::endl;
    
}
```

`CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

project(linear-algebra LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMKAE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED on)

include(CheckCXXCompilerFlag)
check_cxx_compiler_flag("-march=native" _march_native_works)
check_cxx_compiler_flag("-xHost" _xhost_works)

set(_CXX_FLAGS )
if(_march_native_works)
    message(STATUS "Using processor's vector instructions (-march=native compiler flag set)")
    set(_CXX_FLAGS "-march=native")
elseif(_xhost_works)
    message(STATUS "Using processor's vector instructions (-xHost compiler flag set)")
    set(_CXX_FLAGS "-xHost")
else()
    message(STATUS "No suitable compiler flag found for vectorization")
endif()

add_executable(linear-algebra-optimaze linear-algebra.cpp)

target_compile_options(linear-algebra-optimaze 
    PRIVATE 
        ${_CXX_FLAGS})
```

编译

```sh
$mkdir build && cd build
$cmake ..
-- The CXX compiler identification is GNU 11.2.0
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Performing Test _march_native_works
-- Performing Test _march_native_works - Success
-- Performing Test _xhost_works
-- Performing Test _xhost_works - Failed
-- Using processor's vector instructions (-march=native compiler flag set)
-- Configuring done
-- Generating done
-- Build files have been written to: /mnt/f/Station/Linux_Proj/vscodeProj/cmake/demo04/build
```

运行

```sh
# 不带优化编译选项的执行程序
$./linear-algebra-unoptimaze

result: -261.505
elapsed seconds: 2012780600

# 带优化编译选项的执行程序
$./linear-algebra-optimaze

result: -261.505
elapsed seconds: 801839700
```

#### 工作原理

大多数处理器提供向量指令集，代码可以利用这些特性，获得更高的性能。由于线性代数运算可以从Eigen库中获得很好的加速，所以在使用Eigen库时，就要考虑向量化。我们所要做的就是，指示编译器为我们检查处理器，并为当前体系结构生成本机指令。不同的编译器供应商会使用不同的标志来实现这一点：GNU编译器使用` -march=native `标志来实现这一点，而Intel编译器使用 `-xHost `标志。使用 `CheckCXXCompilerFlag.cmake `模块提供的 `check_cxx_compiler_flag `函数进行编译器标志的检查:

`check_cxx_compiler_flag("-march=native" _march_native_works)`

这个函数接受两个参数:

- 第一个是要检查的编译器标志。

- 第二个是用来存储检查结果(true或false)的变量。如果检查为真，我们将工作标志添加到 _CXX_FLAGS 变量中，该变量将用于为可执行目标设置编译器标志。

## 检测外部库和程序

### 预备知识

1. 获取`cmake`现有模块列表命令    ` cmake --help-module-list`
2. 显示`cmake`内置命令的打印文档   `cmake --help-command-list`
3. 显示`cmake`某个内置命令的详细说明 `cmake --help-command <command>`
4. `find`家族命令
   - `find_file` 在对应路径下查找命名文件
   - `find_library` 查找一个库目录
   - `find_package` 从外部项目查找和加载设置
   - `find_path` 查找包含指定文件的目录
   - `find_program` 找到一个可执行程序

### 检测python解释器

```cmake
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
project(findPython LANGUAGES NONE)
# 查找python解释器
find_package(PythonInterp REQUIRED)
# 执行Python命令并捕获他的输出和返回值
execute_process(
    COMMAND
        ${PYTHON_EXECUTABLE} "-c" "print('Hello, world')"
    RESULT_VARIABLE _status
    OUTPUT_VARIABLE _hello_world
    ERROR_QUIET
    OUTPUT_STRIP_TRAILING_WHITESPACE
)
# 打印Python命令的输出和返回值
message(STATUS "RESULT_VARIABLE is <${_status}>")
message(STATUS "OUTPUT_VARIABLE is <${_hello_world}>")
```

运行如下命令：

```sh
$mkdir -p build && cd build
$cmake ..
```

运行结果:

```sh
-- Found PythonInterp: /usr/bin/python3.8 (found version "3.8.10")
-- RESULT_VARIABLE is <0>
-- OUTPUT_VARIABLE is <Hello, world>
-- Configuring done
-- Generating done
-- Build files have been written to: /mnt/f/Station/Linux_Proj/vscodeProj/cmake/chapter_3/01_FindPython/build
```

`FindPythonInterp.cmake`包中附带`cmake`变量

- `PYTHONINTERP_FOUND`  是否找到python解释器
- `PYTHON_EXECUTABLE` Python解释器到可执行文件的路径
- `PYTHON_VERSION_STRING` Python解释器的完整版本信息
- `PYTHON_VERSION_MAJOR` Python解释器的主版本号
- `PYTHON_VERSION_MINOR` Python解释器的次版本号
- `PYTHON_VERSION_PATCH` Python解释器的补丁版本号

使用`-D`选项来指定Python解释器

- `cmake -D PYTHON_EXECUTABLE=custom/location/python ..`

- `cmake --help-module FindPythonInterp` 查看`PtyonInterp`包的说明

使用`CMakePrintHelpers`模块来格式化打印变量

```cmake
include(CMakePrintHelper)
cmake_print_variables(var1 var2 ...)
```

### 检测python库

可以使用Python工具来分析和操作程序的输出。然而，还有更强大的方法可以将解释语言(如Python)与编译语言(如C或C++)组合在一起使用。一种是扩展Python，通过编译成共享库的C或C++模块在这些类型上提供新类型和新功能，这是第9章的主题。另一种是将Python解释器嵌入到C或C++程序中。两种方法都需要下列条件:

- Python解释器的工作版本
- Python头文件Python.h的可用性
- Python运行时库libpython

在C语言中嵌入python代码,如下所示

```c
#include <Python.h>

int main(int argc, char* argv[])
{
    Py_SetProgramName((wchar_t *)argv[0]);
    Py_Initialize();
    PyRun_SimpleString("from time import time, ctime\n"
                       "print('Today is', ctime(time()))\n");
    Py_Finalize();
    return 0;
}
```

在`CMakeLists.txt`中查找`Python`的解释器、开发环境（包括头文件路径、库目录路径等...）、编译器

```cmake
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

project(hello-embeded-py LANGUAGES C)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_EXTENSIONS OFF)
set(CMAKE_C_STANDARD_REQUIRED ON)

# 查找python解释器，"REQUIRED"表示当前查找的包是必要的
find_package(PythonInterp REQUIRED)

# 查找Python的头文件和库目录, "EXACT"表示限制CMAKE检测指定的版本
find_package(PythonLibs ${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR} EXACT REQUIRED)

# 在3.12以上的cmake ，可以使用find_packege的新的python检测模块替换以上两条命令
# find_package(Python COMPONENTS Interpreter Development REQUIRED)

# message(STATUS "Sytem has the Python requested components: ${Python_FOUND}")
# message(STATUS "Sytem has the Python interpreter: ${Python_Interpreter_FOUND}")
# message(STATUS "Path to the Python interpreter: ${Python_EXECUTABLE}")
# message(STATUS "A short stirng unique to the interpreter: ${Python_INTERPRETER_ID}")
# message(STATUS "Standard platform dependent installation directory: ${Python_STDLIB}")
# message(STUATS "Standard platform dependent installation directory: ${Python_STDARCH}")
# message(STATUS "Third-parity platform independent installation directory: ${Python_SITELIB}")
# message(STATUS "Third-party platform dependent installation directory: ${Python_SITEARCH}")
# message(STATUS "System has the Python developments artifacts: ${Python_Development_FOUND}")
# message(STATUS "The Python include directories: ${Python_INCLUDE_DIRS}")
# message(STATUS "The Python library directories: ${Python_LIBRARY_DIRS}")
# message(STATUS "The Python libraries: ${Python_LIBRARIES}")
# message(STATUS "The Python runtiome library directories: ${Python_RUNTIME_LIBRARY_DIRS}")


# 添加一个可执行目标
add_executable(hello-embeded-py hello-embeded-py.c)

# 可执行文件包含python.h
target_include_directories(hello-embeded-py PRIVATE ${PYTHON_INCLUDE_DIRS})

# 将可执行文件连接到Python库
target_link_libraries(hello-embeded-py PRIVATE ${PYTHON_LIBRARIES})

```

 如果你的python不是安装在标准路径，则需要使用`CMAKE`脚手架的`-D`来指定python的解释器，头文件以及库，如下所示

```sh
$cmake -D PYTHON_EXECUTABLE=custom/path/pythonInterp -D PYTHON_LIBRARY=custom/path/pythonlibs -D PYTHON_INCLUDE_DIR=custom/path/pythoninclude
```

### 检测Python模块和包





































































## 总结

### 命令行总结

#### 第一章

1. `cmake -H. -Bbuild`  等价于 `cd build && cmake ..`
2. `cmake --build .` 构建当前目录中的项目
3. `cmake --build . --target <target-name>` 
   - `all`是默认目标，将在项目中构建所有的目标
   - `clean`删除所有的生成文件
   - `rebuild_cache` 将调用CMake为源文件生成依赖
   - `edit_cache` 这个目标允许直接编辑缓存
   - `test` 在`CTest`的帮助下运行测试套件
   - `install` 执行目录安装规则
   - `package` 此目标将调用`CPack`为目录生成可发布的包
   - `help`

4. `cmake --help` 查看`cmake`支持的生成器
5. `cmake -G <Generator>` 切换Generator
6. `cmake --help-module <name-of-moudle>` 打印某个模块的手册页
7. `cmake -D [<custom-option>|<cmake-environ-varible>]=<value>`
8. `cmake --build . --config Rlease`  切换构建版本，如果项目中同时配置了`Release`和`Debug`两个版本

#### 第三章









 	













