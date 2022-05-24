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

`"dependent_option_variable"` 为被依赖选项变量 。

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

1. 通过设置默认构建类型，来达到控制项目，使用使用优化编译，还是关闭优化启用调试。
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





















