# cmake的好处

```
跨平台，并可生成 native 编译配置文件，在 Linux/Unix 平台，生成 makefile，在苹果平台，可以生成 xcode，在 Windows 平台，可以生成 MSVC 的工程文件。

能够管理大型项目。

简化编译构建过程和编译过程。Cmake 的工具链非常简单：cmake+make。

高效率，按照 KDE 官方说法，CMake 构建 KDE4 的 kdelibs 要比使用 autotools 来构建 KDE3.5.6 的 kdelibs 快 40%，主要是因为 Cmake 在工具链中没有 libtool

可扩展，可以为 cmake 编写特定功能的模块，扩充 cmake 功能。
```





# 构建级别

CMake具有许多内置的构建配置，可用于编译工程。 这些配置指定了代码优化的级别，以及调试信息是否包含在二进制文件中。

这些优化级别，主要有：

Release —— 不可以打断点调试，程序开发完成后发行使用的版本，占的体积小。 它对代码做了优化，因此速度会非常快，

在编译器中使用命令： -O3 -DNDEBUG 可选择此版本。

Debug ——调试的版本，体积大。

在编译器中使用命令： -g 可选择此版本。

MinSizeRel—— 最小体积版本

在编译器中使用命令：-Os -DNDEBUG可选择此版本。

RelWithDebInfo—— 既优化又能调试。

一般在 make 后加 -DCMAKE_BUILD_TYPE=XXX来指定构建级别，也可以写入CMakeLists.txt

/usr/bin/cmake --build /mnt/c/Users/18181/CLionProjects/cpp_redis/build --config Debug --target all -j 8

# 命令

## add_executable()

生成可执行文件

## add_library()

普通库

```cmake
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            source1 [source2 ...])
    SHARED,动态库
	STATIC,静态库
	MODULE,在使用 dyld 的系统有效,如果不支持 dyld,则被当作 SHARED 对待。
	如果未指定库类型，则查看BUILD_SHARED_LIBS当前状态是否为ON
```

对象库//文档https://cmake.org/cmake/help/latest/command/add_library.html

```cmake
add_library(<name> OBJECT [<source>...])
```

接口库

```cmake
add_library(<name> INTERFACE)
```

别名库

```cmake
add_library(<name> ALIAS <target>)
```

## SET()

该命令可以为普通变量、缓存变量、环境变量赋值。

常见用法：

```cmake
set(CMAKE_BUILD_TYPE "Debug")//设定构建类型
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_PKGCONFIG_OUTPUT_DIRECTORY ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/pkgconfig)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)//设定是否生成json配置文件
```



**普通变量：**

```cmake
set(<variable> <value>... [PARENT_SCOPE])
```

<value>处可以设置零个或多个参数。多个参数将以分号分隔的列表形式加入，以形成要设置的实际变量值。零参数将导致未设置普通变量。见unset() 命令显式取消设置变量。

设置的变量值 作用域属于整个 CMakeLists.txt 文件。(一个工程可能有多个CMakeLists.txt)

当这个语句中加入PARENT_SCOPE后，表示要设置的变量是父目录中的CMakeLists.txt设置的变量。

比如有如下目录树：

```
├── CMakeLists.txt
└── src
    └── CMakeLists.txt

```

并且在 顶层的CMakeLists.txt中包含了src目录：`add_subdirectory(src)`

那么，顶层的CMakeLists.txt就是父目录，

如果父目录中有变量Bang,在子目录中可以直接使用（比如用message输出Bang，值是父目录中设置的值）并且利用set()修改该变量Bang的值，但是如果希望在出去该子CMakeLists.txt对该变量做出的修改能够得到保留，那么就需要在set()命令中加入Parent scope这个变量。当然，如果父目录中本身没有这个变量，子目录中仍然使用了parent scope，那么出了这个作用域后，该变量仍然不会存在。

**缓存变量：**

```cmake
set(<variable> <value>... CACHE <type> <docstring> [FORCE])
```

首先什么是CACHE变量，就是在运行cmake的时候，变量的值可能会被缓存到一份文件里即build命令下的CMakeCache.txt，当你重新运行cmake的时候，那些变量会默认使用这个缓存里的值。这个变量是全局变量，整个CMake工程都可以使用该变量。

在这个文件里，只要运行cmake …命令，自动会出现一些值，比如 CMAKE_INSTALL_PREFIX ，如果设置 set(CMAKE_INSTALL_PREFIX “/usr”) ，虽然CACHE缓存文件里还有这个CMAKE_INSTALL_PREFIX 变量，但是因为我们显示得设置了一个名为CMAKE_INSTALL_PREFIX 的正常变量，所以之后使用CMAKE_INSTALL_PREFIX ，值是我们设置的正常变量的值。

如果加上CACHE关键字，则设置的这个变量会被写入缓存文件中（但如果本身缓存文件中有这个变量，则不会覆盖缓存中的变量）。只有加上FORCE关键字，这个被写入文件的值会覆盖之前文件中存在的同名变量。

加上CACHE关键字，<type>和<docstring>是必需的。

**环境变量**

```
set(ENV{<variable>} [<value>])
```

设置一个 Environment Variable 到给定值。随后的调用$ENV{<varible>}将返回此新值。

此命令仅影响当前的CMake进程，不影响调用CMake的进程，也不影响整个系统环境，也不影响后续构建或测试过程的环境。

如果在空字符串之后ENV{}或如果没有参数，则此命令将清除环境变量的任何现有值。

之后的参数将被忽略。如果发现其他参数，则会发出作者警告。

## find_package()

```cmake
find_package(Boost 1.46.1 REQUIRED COMPONENTS unit_test_framework)  
```

把一整个依赖包的头文件包含路径、库路径、库名字、版本号等情况都获取到

![21](.\img\21.png)

`version`和`EXACT`: 都是可选的，`version`指定的是版本，如果指定就必须检查找到的包的版本是否和`version`兼容。如果指定`EXACT`则表示必须完全匹配的版本而不是兼容版本就可以。

`QUIET` 可选字段，表示如果查找失败，不会在屏幕进行输出（但是如果指定了`REQUIRED`字段，则`QUIET`无效，仍然会输出查找失败提示语）。

`MODULE` 可选字段。前面提到说“如果Module模式查找失败则回退到Config模式进行查找”，但是假如设定了`MODULE`选项，那么就只在Module模式查找，如果Module模式下查找失败并不回落到Config模式查找。

`REQUIRED`可选字段。表示一定要找到包，找不到的话就立即停掉整个cmake。而如果不指定`REQUIRED`则cmake会继续执行。

`COMPONENTS`，`components`:可选字段，表示查找的包中必须要找到的组件(components），如果有任何一个找不到就算失败，类似于`REQUIRED`，导致cmake停止执行。

`OPTIONAL_COMPONENTS`和`components`：可选的模块，找不到也不会让cmake停止执行。



## target_include_directories()

指定目标包含的头文件路径

## target_link_libraries()

指定目标链接的库

## target_compile_options()

指定目标的编译选项

# 关键字

## PRIVATE|PUBLIC|INTERFACE

规定了生成目标时所使用的**依赖的使用范围**

```cmake
target_link_libraries(math
  PUBLIC
    ${LAPACK_LIBRARIES}
  )
```

```
cmake-test/                 工程主目录，main.c 调用 libhello-world.so
├── CMakeLists.txt
├── hello-world             生成 libhello-world.so，调用 libhello.so 和 libworld.so
│   ├── CMakeLists.txt
│   ├── hello               生成 libhello.so 
│   │   ├── CMakeLists.txt
│   │   ├── hello.c
│   │   └── hello.h         libhello.so 对外的头文件
│   ├── hello_world.c
│   ├── hello_world.h       libhello-world.so 对外的头文件
│   └── world               生成 libworld.so
│       ├── CMakeLists.txt
│       ├── world.c
│       └── world.h         libworld.so 对外的头文件
└── main.c
```

**调用关系：**

```cmake
                                 ├────libhello.so
可执行文件────libhello-world.so
                                 ├────libworld.so
```

**关键字用法说明：**

**PRIVATE**：私有的。生成 libhello-world.so时，只在 hello_world.c 中包含了 hello.h，libhello-world.so **对外**的头文件——hello_world.h 中不包含 hello.h。而且 main.c 不会调用 hello.c 中的函数，或者说 main.c 不知道 hello.c 的存在，那么在 hello-world/CMakeLists.txt 中应该写入：

```cmake
target_link_libraries(hello-world PRIVATE hello)
target_include_directories(hello-world PRIVATE hello)
```

**INTERFACE**：接口。生成 libhello-world.so 时，只在libhello-world.so **对外**的头文件——hello_world.h 中包含 了 hello.h， hello_world.c 中不包含 hello.h，即 libhello-world.so 不使用 libhello.so 提供的功能，只使用 hello.h 中的某些信息，比如结构体。但是 main.c 需要使用 libhello.so 中的功能。那么在 hello-world/CMakeLists.txt 中应该写入：

```cmake
target_link_libraries(hello-world INTERFACE hello)
target_include_directories(hello-world INTERFACE hello)
```

**PUBLIC**：公开的。**PUBLIC = PRIVATE + INTERFACE**。生成 libhello-world.so 时，在 hello_world.c 和 hello_world.h 中都包含了 hello.h。并且 main.c 中也需要使用 libhello.so 提供的功能。那么在 hello-world/CMakeLists.txt 中应该写入：

```cmake
target_link_libraries(hello-world PUBLIC hello)
target_include_directories(hello-world PUBLIC hello)
```

实际上，这三个关键字指定的是目标文件依赖项的使用**范围（scope）**或者一种**传递（propagate）**。[官方说明](https://link.zhihu.com/?target=https%3A//cmake.org/cmake/help/v3.15/manual/cmake-buildsystem.7.html%23transitive-usage-requirements)

可执行文件依赖 libhello-world.so， libhello-world.so 依赖 libhello.so 和 libworld.so。

1. main.c 不使用 libhello.so 的任何功能，因此 libhello-world.so 不需要将其依赖—— libhello.so 传递给 main.c，hello-world/CMakeLists.txt 中使用 PRIVATE 关键字；
2. main.c 使用 libhello.so 的功能，但是libhello-world.so 不使用，hello-world/CMakeLists.txt 中使用 INTERFACE 关键字；
3. main.c 和 libhello-world.so 都使用 libhello.so 的功能，hello-world/CMakeLists.txt 中使用 PUBLIC 关键字；

# 单元测试

单元测试是一个软件开发过程，其中最小的可测试部分应用程序，称为单元，被单独和独立地审查以便正常运行。 这可能涉及采用类、函数或算法并编写可以运行的测试用例来验证单元是否正常工作。

CMake 包含一个名为链接的工具：https://cmake.org/Wiki/CMake/Testing_With_CTest[CTest]这允许您启用 `make test` 目标来运行自动化测试，例如单元测试。

有许多可用的单元测试框架可用于帮助自动化并简化单元测试的开发。 在cmake-tuitor-05中，展示了如何使用其中一些框架并使用 CMake 测试实用程序 CTest 调用它们。

常用做法是把单元测试编写为库文件，需要时链接

## enable_testing()

启用Ctest

## add_test()

将写好的测试加入测试集

```cmake
option(BUILD_TESTS "option for test" OFF)
option(BUILD_EXAMPLES "option for examples" ON)
if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(tests)
  ExternalProject_Add("googletest"
                      GIT_REPOSITORY "https://github.com/google/googletest.git"
                      CMAKE_ARGS "-DCMAKE_INSTALL_PREFIX=${PROJECT_SOURCE_DIR}/deps")
  # Reset variable to false to ensure tacopie does no build tests
  set(BUILD_TESTS false)
endif(BUILD_TESTS)
```



# CPack

将cmake打包为deb文件发布

```
install(CODE "FILE(MAKE_DIRECTORY ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})")
install(CODE "FILE(MAKE_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})")
# install cpp_redis
install(DIRECTORY ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/ DESTINATION lib USE_SOURCE_PERMISSIONS)
install(DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/ DESTINATION bin USE_SOURCE_PERMISSIONS)
install(DIRECTORY ${CPP_REDIS_INCLUDES}/ DESTINATION include USE_SOURCE_PERMISSIONS)
```



# 常用变量

${PROJECT_BINARY_DIR}：build目录

${CMAKE_SOURCE_DIR}：makelists所在的目录

```cmake
file( GLOB src "*.cc" )

file( GLOB inc "*.h")

add_executable(EventLoop ${src} ${inc})

```

