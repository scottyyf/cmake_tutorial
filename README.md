

### 安装

```
yum install gcc gcc-c++
```

1. 源码
   * download tar文件并解压
   * cd到文件，./bootstrap --prefix=/usr/local
   * make
   * make install

### 新手起步

#### step1 基础

1. 制作CMakeLists.txt文件

   ```cmake
   cmake_minimum_required(VERSION 3.10)
   
   # 设置项目名称, 版本 3.1
   project(Turorial VERSION 3.1)
   
   # 配置header文件，该文件将定义一些参数给tutorial.cxx使用
   configure_file(TutorialConfig.h.in TutorialConfig.h)
   
   # 增加可运行脚本Tutorial, 根据tutorial.cxx进行制作
   add_executable(Tutorial tutorial.cxx)
   
   # 配置文件也将写入binary文件后，需要添加该路径到寻址地址
   target_include_directories(Tutorial PUBLIC "{PROJECT_BINARY_DIR}")
   ```

2. 生成头文件TutorialConfig.h.in

   ```c++
   #define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
   #define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
   ```

3. 生成主文件tutorial.cxx

   ```c++
   #include <iostream>
   #include "TutorialConfig.h"
   
   int main(int argc, char **argv){
   	if (argc < 5) {
   	    // report version
   	   std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
                        << Tutorial_VERSION_MINOR << std::endl;
              std::cout << "Usage: " << argv[0] << " number" << std::endl;
              return 1;
   	}
   }
   ```

4. 执行

   * cmake .
   * cmake --build .

#### step2 添加library

* 可动态控制是否要添加一个lib
* TODO



#### step3添加usage requirements for library

* 更好的控制library或可执行文件的链接
* 控制cmake中的过渡属性更多控制
* 使用要求 的命令有
  * target_compile_definitions()
  * target_compile_options()
  * target_include_directories()
  * target_link_libraries()

#### step4 installig and testing

开始添加安装的策略和测试支持

如： 

* 对MathFunctions来说，我们需要安装library和header file
* 对app来说，我们需要安装可执行文件和配置好的header



#### step5 添加系统introspection

系统是否有这个函数，

* 我们先检测对应module是否可用

* 如果不存在，那么需要链接到其他库

* 然后再次检测是否可用，如果是后来链接的，那么需要添加这个lib(m)。 target_link_libraries

  ```cmak
  # MathFunctions/CMakeLists.txt
  target_include_directories(MathFunctions INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}) # 先需要这句
  # do this system provide log an exp function
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
      target_link_libraries(MathFunctions PROVATE m)
    endif()
  endif()
  ```

#### step6 添加自定义命令和生成的文件

假设，我们不需要使用log,exp，而是生成一个table来预先生成值给mysqrt function使用

1. 删除MathFunctions/CMakeLists.txt中的log和exp的检测，然后删除mysqrt.cxx中的HAVE_LOG和HAVE_EXP部分
2. 在MathFunctions新建MakeTable.cxx的source file
3. 在MathFunctions/CMakeLists.txt中加入MakeTable的可执行程序executable,在运行程序时来build它



#### step7 打包一个installer

发布给别人，使别人可以使用

提供binary和source code

**cpack** yyds

```cmake
# 收集当前系统平台runtime的library
include(InstallRequiredSystemLibraries)
# 设置cpack的一些变量，license和版本信息，版本信息在
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
include(CPack)
```

#### step8 上传test结果到dashboard

* 添加tests到project中， 测试样例
* 在top-level的CMakeLists.txt中增加CTest module. 会自动调用enable_testing()

```cmake
include(CTest)
```

TODO

#### step9 选取static或shared libraries

* BUILD_SHARED_LIBS来控制add_library()的行为
* 控制没有(STATIC SHARED MODILE OBJECT)明确标识的libraries如何被build

> > > 这个地方和c强相关的概念，pass



#### step10 添加generator expressions

在系统build期间，生成每个build的特殊信息

如： cmake --build 期间打印一条消息

todo



#### step11 添加导出配置

添加必要的信息，这样别的cmake projects可以使用我们的project

1. install命令不仅需要DESTINATION参数，而且还需要EXPORT参数。
   * EXPORT参数生成一个cmake的文件，该文件包含所有被import的code

#### step12 打包调试也发布



### 带着项目执行

#### 命令行

**在cmake文档中的cmake-commands中可以查询到相关字段的具体细节**

##### project

设置项目的名称，并将该名称存储到PROJECT_NAME变量，如果CMakeLists.txt是顶层文件的话，CMAKE_PROJECT_NAME将自动设置为该名字

设置该变量还将内部自动获取变量

| 变量名                    | 解释                                                         |
| ------------------------- | ------------------------------------------------------------ |
| PROJECT_SOURCE_DIR        | 当前工程源码路径                                             |
| <project_name>_SOURCE_DIR | 指定工程的源码路径，若是当前的项目，则和上一个变量的值一致   |
| PROJECT_BINARY_DIR        | 当前工程的二进制路径                                         |
| <PROJECT_NAME>_BINARY_DIR | 同上                                                         |
| CMAKE_PROJECT_NAME        | 如果CMakeLists.txt是顶层文件的话，CMAKE_PROJECT_NAME将自动设置为该名字 |
|                           |                                                              |

如果后面project(project_name [VERSION version]), 那么这里就会有一些自动变量，如PROJECT_VERSION_MAJOR等等

DESCRIPTION <DESC STRING> : 如果设置了这个，那么就会有PROJECT_DESCRIPTION, <PROJECT_NAME>_DESCRIPTION,对于顶层的cmakelist文件，还会有CMAKE_PROJECT_DESCRIPTION

HOMEPAGE_url <URL> 项目主页url，同理会有自动的一些变量生成

LANGUAGES <LANGUAGES> 特殊的，这里LANGUAGES可以忽略这个关键字，不用写，NONE跳过该设置，默认是c c++,build project时需要的语言

##### list

和set差不多，list命令将创建新的变量

子命令, list是一个使用;分隔的string，如 set(var a b c d e)将创建一个list a;b;c;d;e

| APPEND      |      |
| ----------- | ---- |
| INSERT      |      |
| FILTER      |      |
| PREPEND     |      |
| POP_BACK    |      |
| POP_FRONT   |      |
| REMOVE_AT   |      |
| REMOVE_ITEM |      |
| REVERSE     |      |

list(sub_command ... <output var>)

list(APPEND <LIST> [ELEMENT] ) 将元素添加到list中，list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cpack)将更新CMAKE_MODULE_PATH的值

##### set

* 普通变量和cache变量；查找变量时，先查找是否有普通变量，没有才会查找是否存在于cache中。**应避免同时使用普通和cache变量**
  - 普通变量可在内部脚本使用
  - cache变量，在整个项目cmake运行中都有效
  - 顶层中设置了对应的cache，那么就这个生效，下层中的设置不生效

```cmake
set(<var> <value> [[CACHE <type> <docstring> [FORCE]] |PARENT_SCOPE ])
```

* CACHE：将var放在cache中，除非已在cache中。提供该参数，那么type docstring就必须提供
* type：可选值 ，是cmake gui中用来选择对应类型的组件

|          |                                                         |
| -------- | ------------------------------------------------------- |
| FILEPATH | 选择文件的对话框                                        |
| PATH     | 目录选择                                                |
| STRING   | string                                                  |
| BOOL     | ON/OFF                                                  |
| INTERNAL | 用于持久变量，没有gui条目。用于不给用户显示，不让其更改 |

* FORCE，那么cache var就会被改变，应避免使用
* PARENT_SCOPE

> > set(extra_libs ${extra_libs} MyFunctions) 这里的意思是，set(var a b c)如果var有值，那么就是一个list

##### add_subdirectory

> > 添加外部项目文件,进行build

在构建(build)过程中，添加一个子目录

> > add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])

* source_dir指示到那个目录去找CMakeLists.txt和code files,如果是相对路径，那么从当前目录开始，也可以是绝对路径
* binary_dir指示输出文件放置的路径，默认和source_dir一致



##### include

> > 加载并执行cmake代码

load and run cmake code from a file or module

> > include(<file|module> [OPTIONAL] [RESULT_VARIABLE <var>] [NO_POLIC_SCOPE])

* 变量使用调用者的作用域

* OPTIONAL如果提供了，文件不存在也不会报错
* RESULT_VARIABLE 提供后，那么var将会赋值成include的那个file的全称，如果文件不存在则为NOTFOUND
* 如果提供的是一个module， 那么就会在CMAKE_MODULE_PATH路径下查找名为<module_name>.cmake的文件



##### if

```cmake
if(condition)
   command
elseif(condition)
   command
else()
   command
endif()
```

* True : 1, ON, YES, TRUE, Y， 非0数字
* False: 0, OFF， NO， FALSE, N, IGNORE, NOTFOUND,空字符，以-NOTFOUND结尾的
* 对应变量，可以不要${}来获取值，**如set(var1 "var2"), if(var1)而不用if(${var1})**

|                                              |                                                              |
| -------------------------------------------- | ------------------------------------------------------------ |
| if(NOT <condition>)                          | if(EXISTS file_or_dir) 文件路径是否存在                      |
| if(cond1 AND cond2)                          | if(<var\|str> MATCHES regex) 是否匹配，是否相等等等          |
| if(cond1 OR cond2)                           | if(<var\|str> VERSION_LESS <var\|str>) 版本号比较            |
| if((cond1) AND (cond2 OR (cond3)))           |                                                              |
| if(COMMAND command-name)                     | 如果comamnd_name是存在的命令                                 |
| if(POLICY policy-id)                         |                                                              |
| if(TARGET target-name)                       | target-name被add_executable()，add_library,add_custom_target创建出来则为True |
| if(DEFINED <name>\|CACHE <name>\|ENV <name>) | 如果对应的被定义了则为True                                   |



##### add_definitions

给source 文件汇编时，加上-D参数

> > add_definitions(-Dfoo -Dbar ...)

对别人代码做实验时，不对源码做破坏，又可以添加自己的功能

之前是在程序中使用#define进行定义

现在： 

```cmake
cmake -Dfoo=1 ..#打开
```

源码中:

```c++
#ifdef foo...#else    ...#endif
```



##### file

> > 文件操作

file(ACTION ...)

具体参考文档，太多

| 读   | 写   | 文件系统 | 路径转化 | 传送 | 锁   | 压缩 |
| ---- | ---- | -------- | -------- | ---- | ---- | ---- |
|      |      | COPY     |          |      |      |      |

> > file(COPY . DESTINATION ${CMAKE_CURRENT_BINARY_DIR} PATTERN *.xml ) 
> >
> > 将xml文件拷贝到目的地去， 对于CMAKE_CURRENT_BINARY_DIR，add_subdirectory会自动创建的，值为对应的目录



##### install

> > make install的规则，CMAKE_INSTALL_PREFIX将定义在哪安装

> > install(FILES file_name DESTINATION dst_dir COMPONENT ha)

运行时添加-DCOMPONENT ha将只会安装ha对应的组件，不指定则安装所有

* install(TARGETS)

安装MyLib库的位置，当使用lib时，那么生成的文件可能是.o或.a结尾的未见

> > 
> >
> > ```cmake
> > install(TARGETS MyLib     EXPORT MyLibTargets      LIBRARY DESTINATION lib  # 动态库安装路径     ARCHIVE DESTINATION lib  # 静态库安装路径     RUNTIME DESTINATION bin  # 可执行文件安装路径     PUBLIC_HEADER DESTINATION include  # 头文件安装路径     )
> > ```

* install(FILES)

将文件(头文件)拷贝到对应位置

##### aux_source_directory

查找所有在dir中的**source**文件，并赋值给后面的var

> > aux_source_directory(<dir> <var>) 

**这个好像就只认cc结尾的文件，.h结尾他是不认得**



##### target_link_libraries

> > target_link_libraries(Demo sub_dir_lib_name) 适用多层目录项目
> >
> > 子目录则需要配置 
> >
> > add_library(sub_dir_lib_name srcs)

add_subdirectory后，会build子目录，此时，如果上层(当前层)使用子目录的编译的库，那么可以静态导入

这里就可以使用该命令



##### include_directories

程序中使用了#include "xx.h" ,如果xx.h不在当前目录，而是在子目录下，那么需要使用该命令

相当于python的sys.path添加寻址路径

当#include使用了相对路径则不需要该命令



##### configure_file

拷贝一个文件到另一个位置并更改内容

> > configure_file(<input> <output>)

input中的#cmakedefine VAR将替换成#define VAR或/* #undef VAR */

> 

foo.h.in

```cmake
#cmakedefine FOO_ENABLE#cmakedefine FOO_STRING "@FOO_STRING@"
```

CMakeLists.txt

```cmake
option(FOO_ENABLE "foo enable " ON)if(FOO_ENABLE)	set(FOO_STRING "foo")endif()configure_file(foo.h.in foo.h @ONLY)
```

如果FOO_ENABLE,那么将得到文件foo.h

```c++
#define FOO_ENABLE#define FOO_STRINT "foo"
```

否则

```c++
/* #undef FOO_ENABLE *//* #undef FOO_STRING */
```



##### add_test

使make test可以进行

1. enable_testing()开启测试开关
2. add_test (<测试的名称> Demo 5 2)测试Demo可执行文件的正确
3. set_tests_properties (<测试的名称> PROPERTIES PASS_REGULAR_EXPRESSION "Usage: .* base exponent") 测试结果是否有对应的描述

> > 
> >
> > ```cmake
> > # 定义一个宏，用来简化测试工作macro (do_test arg1 arg2 result)	add_test (test_${arg1}_${arg2} Demo ${arg1} ${arg2})  set_tests_properties (test_${arg1}_${arg2}    PROPERTIES PASS_REGULAR_EXPRESSION ${result})endmacro (do_test)# 使用该宏进行一系列的数据测试do_test (5 2 "is 25")do_test (10 5 "is 100000")do_test (2 10 "is 1024")
> > ```



##### string

字符的一系列操作

|                                 |                                |
| ------------------------------- | ------------------------------ |
| string(TOUPPER <str> <out_var>) | 将str转化为大写并赋值给out_var |



##### foreach

循环

```cmake
foreach(i in $<list_var>)   doendeach()
```





#### 內建变量

內建变量可通过查询cmake-variales进行查询

全大写的，CMAKE打头的是内部

|                             |                                                              |
| --------------------------- | ------------------------------------------------------------ |
| CMAKE_MODULE_PATH           | ：分隔的列表或directories用来查找cmake模块，当使用include()和find_packge命令时 |
| CMAKE_FIND_LIBRARY_SUFFIXES | 使用find_library()时，查找的suffixes,如在windows中查找foo时，这里定义为lib，那么将查找*foo.lib |
| CMAKE_CURRENT_BINARY_DIR    | add_subdirectory将创建对应的值，自动给对应CMakeLists.txt中的该变量 |
| CMAKE_INCLUDE_CURRENT_DIR   | 如果开启，那么各子级CMAKE_CURRENT_SOURCE_DIR和CMAKE_CURRENT_BINARY_DIR将自动被include_directories包含。**c能include <自己写的头文件>**，我看还是这个功能 |
| CMAKE_INSTALL_PREFIX        | make install的目的地，当DESTINATION是相对路径时              |
|                             |                                                              |
|                             |                                                              |



#### CPACK

> > 设置了一些CPACK的变量后，然后就可以make package了,这个过程将自动产生spec文件

cpack 的一些通用內建变量呢，在cmake的module 对应cpack

|                                           |                                                              |
| ----------------------------------------- | ------------------------------------------------------------ |
| CPACK_PACKAGE_NAME                        |                                                              |
| CPACK_PACKAGE_VERSION_MAJOR\|MINOR\|PATCH | 版本，默认和project()中的版本一致                            |
| CPACK_PACKAGE_VERSION                     | 包的版本全称                                                 |
| CPACK_PACKAGE_DESCRIPTION_FILE            | text文件，用来描述project，在CPACK_PACKAGE_DESCRIPTION没设置时 |
| CPACK_PACKAGE_DESCRIPTION_SUMMARY         | short desc for project，如果CMAKE_PROJECT_DESCRIPTIONS设置了，否则CMAKE_PROJECT_NAME将作为default值 |
| CPACK_PACKAGING_INSTALL_PREFIX            | 制作包，包文件的前缀                                         |
| CMAKE_PREFIX_PATH                         | find_packages, find_xx这些程序查找的指定目录                 |

CPACKRPM,用来创建rpm pkg的一个cpack包

|                           |                                   |
| ------------------------- | --------------------------------- |
| CPACK_RPM_PACKAGE_RELEASE | RPM包本身的编号，而不是内容的版本 |

##### COMPONENT相关

需要说明：

> > 可以同时设置CPACK_RPM_XX对应的描述，同时设置CPACK_DEB_XX的值，

1. 将每个component注册给总体component 变量CPACK_COMPONENTS_ALL

```cmake
set(CPACK_COMPONENTS_ALL ha)list(APPEND CPACK_COMPONENTS_ALL hadr)# 允许component, rpm deb不相同的set(CPACK_RPM_COMPONENT_INSTALL ON)set(CPACK_DEB_COMPONENT_INSTALL ON)
```

2. 给每个component做设置，包的名称，描述等等

```cmake
foreach(component_name in ${CPACK_COMPONENTS_ALL})	string(TUPPER ${component} COMPONENT)	set(CPACK_COMPONENT_${COMPONENT}_DESCRIPTION "对应的值")	# 设置rpm查询的包名	if(NOT CPACK_RPM_${COMPONENT)_PACKAGE_NAME)		set(CPACK_RPM_${COMPONENT}_PACKAGE_NAME "${CPACK_PACKAGE_NAME}-${component}")	endif()	# 设置architecture	if(NOT CPACK_RPM_${COMPONENT}_PACKAGE_ARCHITECTURE)	   set.。。	endif()	# 设置rpm包名	if(NOT CPACK_RPM_${COMPONENT}_FILE_NAME)		set(CPACK_RPM_${COMPONENT}_FILE_NAME "。。。。")	endif()endeach()
```



3. 在每个component实际定义的cmakelist中的install函数中加入COMPONENT参数

```cmake
install(FILES        hadr.md        DESTINATION        dist-packages        COMPONENT hadr)
```



#### make

> > make install

将resources(定义在install（）函数中)拷贝到DESTINATION的位置

> > make package

生成rpm文件



#### CMAKE MODULE



|                                       |                           |                               |
| ------------------------------------- | ------------------------- | ----------------------------- |
| CheckFunctionExists(<function> <var>) | 检查c的函数是否可以linked | 是否存在并将结果放在var变量中 |
| CPackDeb,CPackRPM                     | 配合Cpack生成deb rpm包    |                               |
|                                       |                           |                               |
|                                       |                           |                               |

> > CPACK_RPM_<COMPONENT>\_XXX 变量是对应component的变量；CPACK_RPM\_XXX是CPACKRPM的特殊变量

#### 生成各平台的包

##### rpm包

* 必要条件
  * rpm-build包
  * 执行make package生成包

步骤：

1. 写好cmakelist.txt这些
2. 创建build DIR
3. 在build中cmake .., 这里注意用CMAKE_BINARY_DIR参数

| include (InstallRequiredSystemLibraries) | include (CPack)                 |                                              |
| ---------------------------------------- | ------------------------------- | -------------------------------------------- |
| CPACK_GENERATOR                          | 指定将生成何种包                | RPM DEB等可选,可不提供，cpack -G RPM即可指定 |
| CPACK_PACKAGE_DESCRIPTION_SUBMMARY       | 包中的desc字段                  |                                              |
| CPACK_PACKAGE_NAME                       | 包名                            |                                              |
| CPACK_PACKAGE_FILE_NAME                  | 生成的包的文件名                | httpd-2.1.x86_64                             |
| CPACK_PACKAGE_VERSION                    | 包查询到的版本                  |                                              |
| CPACK_RPM_PACKAGE_ARCHITECTURE           | 平台                            | x86_64,amd64                                 |
| CPACK_RPM_PACKAGE_RELEASE                | 1%{?dist}                       |                                              |
| CPACK_RPM_PACKAGE_LICENSE MIT            | 查询到的lic类型                 |                                              |
| CPACK_PACKAGE_VENDOR                     | 开发者                          |                                              |
| CPACK_PACKAGE_CONTACT                    | 联系                            |                                              |
| CPACK_RPM_PACKAGE_URL                    | url                             |                                              |
| CPACK_RPM_PACKAGE_DESCRIPTION            |                                 |                                              |
| CPACK_RPM_PACKAGE_REQUIRES               | python >= 2.7.0, httpd >= 2.4.6 | 这里将自动安装需要的包                       |
| CPACK_RPM_CHANGELOG_FILE                 | 包可查询的changelog             |                                              |
| CPACK_RPM_PRE_INSTALL_SCRIPT_FILE        | 安装前跑的脚本                  | ${CMAKE_BINARY_DIR}/uninstall.sh             |
| CPACK_RPM_DEFAULT_USER                   | 安装的文件所属用户              | 这些文件的用户属性                           |
| CPACK_PACKAGING_INSTALL_PREFIX           | 安装时前缀，即安装到到哪里      | /opt/ha                                      |
| CPACK_RPM_COMPONENT_INSTALL              | 是否允许按组件打包              |                                              |
|                                          |                                 |                                              |







