## 0.  CUnit的编译和安装

```sh
1. 下载CUnit-2.1.0, https://sourceforge.net/projects/cunit
2. $tar -zxvf CUnit的压缩包
3. $cd CUnit-2.1.0
4. $sudo apt install libtool
5. $sudo apt-get install libsysfs-dev
6. $sudo apt install automake
7. $sudo apt install autoconf
需要注意的是，如果没有configure.ac文件，则将configure.in改名为configure.ac
8. $mv configure.in  configure.ac
9. $aclocal
10. $autoheader
11. $autoconf
12. $automake # 如果在此处有"./ltmain.sh not found"报错,请执行13的命令 
12.1 $automake --add-missing
13. $libtoolize --automake --copy --debug --force # 执行完后，继续执行12
14. $automake # 如果还是出现报错，请按照提示下载相关程序, 重复次步骤,直到不报错为止
15. $./configure # 当上述步骤都执行成功，则会在目录生成一个configure文件；如果你想指定生成库目录的路径，则可以在使用'./configure --prefix=/path/to'
16. $make 
17. $sudo make install # 最好加上sudo,因为CUnit配置的路径需要管理员权限才能进入

到此安装完毕，可以进入到`/usr/local/lib`中查看是否生成CUnit的动态库
```



## 1. 使用 CUnit 进行单元测试简介

### 1.1. 描述

`CUnit `是一个用于在 C 中编写、管理和运行单元测试的系统。它被构建为一个库（静态或动态），与用户的测试代码链接。

`CUnit `使用简单的框架来构建测试结构，并提供一组丰富的断言来测试常见数据类型。此外，还提供了几个不同的接口来运行测试和报告结果。其中包括用于代码控制的测试和报告的自动化界面，以及允许用户动态运行测试和查看结果的交互式界面。

对典型用户有用的数据类型和函数在以下头文件中声明：

| 头文件                                                  | 描述                                                         |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| **#include <[CUnit/CUnit.h](headers/CUnit.h)>**         | 用于测试用例的 ASSERT 宏，并包括其他框架标头。               |
| **#include <[CUnit/CUError.h](headers/CUError.h)>**     | 处理函数和数据类型时出错。*由 CUnit.h 自动包含。*            |
| **#include <[CUnit/TestDB.h](headers/TestDB.h)>**       | 测试注册表、套件和测试的数据类型定义和操作函数。*由 CUnit.h 自动包含。* |
| **#include <[CUnit/TestRun.h](headers/TestRun.h)>**     | 用于运行测试和检索结果的数据类型定义和函数。*由 CUnit.h 自动包含。* |
| **#include <[CUnit/Automated.h](headers/Automated.h)>** | 具有 xml 输出的自动化界面。                                  |
| **#include <[CUnit/Basic.h](headers/Basic.h)>**         | 具有非交互式输出到标准输出的基本界面。                       |
| **#include <[CUnit/Console.h](headers/Console.h)>**     | 交互式控制台界面。                                           |
| **#include <[CUnit/CUCurses.h](headers/CUCurses.h)>**   | 交互式控制台界面 （*nix）。                                  |
| **#include <[CUnit/Win.h](headers/Win.h)>**             | Windows 界面（尚未实现）。                                   |



### 1.2. 结构

CUnit 是独立于平台的框架与各种用户界面的组合。核心框架为管理测试注册表、套件和测试用例提供了基本支持。用户界面便于与框架交互以运行测试和查看结果。

CUnit的组织方式类似于传统的单元测试框架：

```sh
                      Test Registry 
                            |
             ------------------------------
             |                            |
          Suite '1'      . . . .       Suite 'N'
             |                            |
       ---------------             ---------------
       |             |             |             |
    Test '11' ... Test '1M'     Test 'N1' ... Test 'NM'
```

一个测试注册表中可以多个套件，一个套件可以有多个测试用例。（树结构）

单个测试用例被打包到套件中，这些套件在活动测试注册表中注册。套件可以具有设置和拆卸功能，这些功能在运行套件测试之前和之后自动调用。注册表中的所有套件/测试都可以使用单个函数调用运行，也可以运行选定的套件或测试。

### 1.3. 一般用法

使用 CUnit 框架的典型步骤序列是：

1. 为测试[编写函数](writing_tests.html)（如有必要，还可以设置初始化/清理）。
2. 初始化测试注册表 - [CU_initialize_registry（）](test_registry.html#init)
3. 将套件添加到测试注册表 - [CU_add_suite（）](managing_tests.html#addsuite)
4. 将测试添加到套件 - [CU_add_test（）](managing_tests.html#addtest)
5. 使用适当的接口运行测试，例如[CU_console_run_tests](running_tests.html#console)
6. 清理测试注册表 - [CU_cleanup_registry](test_registry.html#cleanup)

## 2. 编写 CUnit 测试用例

### 2.1. 测试函数

`CUnit`“test”是一个具有签名的C函数：`void test_func（void）`

对测试函数的内容没有限制，只是它不应该修改`CUnit`框架（例如，添加套件或测试，修改测试注册表或启动测试运行）。测试函数可以调用其他函数（这些函数也不能修改框架）。注册测试将导致在运行测试时运行其函数。

返回最多 2 个整数的例程的示例测试函数可能如下所示：

```
    int maxi(int i1, int i2)
    {
      return (i1 > i2) ? i1 : i2;
    }

    void test_maxi(void)
    {
      CU_ASSERT(maxi(0,2) == 2);
      CU_ASSERT(maxi(0,-2) == 0);
      CU_ASSERT(maxi(2,2) == 2);
    }
```

### 2.2. `CUnit` 断言

`CUnit` 提供了一组用于测试逻辑条件的断言。这些断言的成功或失败由框架跟踪，并且可以在测试运行完成时查看。

每个断言测试单个逻辑条件，如果条件的计算结果是`CU_FALSE` ,则断言失败，测试函数将继续，除非用户选择断言的“xxx_FATAL”版本。在这种情况下，测试函数将中止并立即返回。**应谨慎使用断言的致命版本！**一旦 FATAL 断言失败，测试函数就没有机会自行清理。但是，正常的[套件清理功能](managing_tests.html#addsuite)不受影响，并且在任一情况下都会运行。

还有特殊的“断言”，用于在不执行逻辑测试的情况下向框架注册[通过](#pass)或[失败](#fail)。这些对于测试控制流或其他不需要逻辑测试的条件非常有用：

```
    void test_longjmp(void)
    {
      jmp_buf buf;
      int i;

      i = setjmp(buf);
      if (i == 0) {
        run_other_func();
        CU_PASS("run_other_func() succeeded.");
      }
      else
        CU_FAIL("run_other_func() issued longjmp.");
    }
```

由已注册的测试函数调用的其他函数可以自由地使用 CUnit 断言。这些断言将计入调用函数。它们还可以使用FATAL版本的断言 - 失败将中止原始测试函数及其整个调用链。

`CUnit `定义的断言包括：**#include <[CUnit/CUnit.h](headers/CUnit.h)>**

| 断言函数                                                     | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| CU_ASSERT(int expression)<br>CU_ASSERT_FATAL(int expression)<br>CU_TEST(int expression)<br>CU_TEST_FATAL(int expression) | 断言*表达式*为（非零）`CU_TRUE`                              |
| CU_ASSERT_TRUE(value)<br>CU_ASSERT_TRUE_FATAL(value)         | 断言*值为*`CU_TRUE (non-zero)`                               |
| CU_ASSERT_FALSE(value)<br>CU_ASSERT_FALSE_FATAL(value)       | 断言*值为* （零）`CU_FALSE`                                  |
| CU_ASSERT_EQUAL(actual, expected)<br>CU_ASSERT_EQUAL_FATAL(actual, expected) | 断言*实际* = = *预期*                                        |
| CU_ASSERT_NOT_EQUAL(actual, expected)<br>CU_ASSERT_NOT_EQUAL_FATAL(actual, expected) | 断言*实际* ！= *预期*                                        |
| CU_ASSERT_PTR_EQUAL(actual, expected)<br>CU_ASSERT_PTR_EQUAL_FATAL(actual, expected) | 断言指针*实际* = = *预期*                                    |
| CU_ASSERT_PTR_NOT_EQUAL(actual, expected)<br>CU_ASSERT_PTR_NOT_EQUAL_FATAL(actual, expected) | 断言指针*实际* ！= *预期*                                    |
| CU_ASSERT_PTR_NULL(value)<br>CU_ASSERT_PTR_NULL_FATAL(value) | 断言指针*值* == 空                                           |
| CU_ASSERT_PTR_NOT_NULL(value)<br>CU_ASSERT_PTR_NOT_NULL_FATAL(value) | 断言指针*值* ！= 空                                          |
| CU_ASSERT_STRING_EQUAL(actual, expected)<br>CU_ASSERT_STRING_EQUAL_FATAL(actual, expected) | 断言*实际*字符串和*预期*字符串是等效的                       |
| CU_ASSERT_STRING_NOT_EQUAL(actual, expected)<br>CU_ASSERT_STRING_NOT_EQUAL_FATAL(actual, expected) | 断言*字符串实际*字符串和*预期*字符串不同                     |
| CU_ASSERT_NSTRING_EQUAL(actual, expected, count)<br>CU_ASSERT_NSTRING_EQUAL_FATAL(actual, expected, count) | 断言*实际*和*预期的*第一次计数字符相同                       |
| CU_ASSERT_NSTRING_NOT_EQUAL(actual, expected, count)<br>CU_ASSERT_NSTRING_NOT_EQUAL_FATAL(actual, expected, count) | 断言*实际*和*预期的*第一次计数字符不同                       |
| CU_ASSERT_DOUBLE_EQUAL(actual, expected, granularity)<br>CU_ASSERT_DOUBLE_EQUAL_FATAL(actual, expected, granularity) | 断言\|*实际* - *预期*\|<= \|*粒度*\| *必须将此断言链接到数学库。* |
| CU_ASSERT_DOUBLE_NOT_EQUAL(actual, expected, granularity)<br>CU_ASSERT_DOUBLE_NOT_EQUAL_FATAL(actual, expected, granularity) | 断言\|*实际* - *预期*\|> \|*粒度*\| *必须将此断言链接到数学库。* |
| CU_PASS(message)                                             | 使用指定的消息注册传递断言。不执行逻辑测试。                 |
| CU_FAIL(message)<br>CU_FAIL_FATAL(message)                   | 使用指定的消息注册失败的断言。不执行逻辑测试。               |





### 2.3. 已指定的 v1 断言

以下断言从版本 2 开始已弃用。若要使用这些断言，必须使用定义USE_DEPRECATED_CUNIT_NAMES来编译用户代码。请注意，它们的行为与版本 1 中的行为相同（失败时发出“return”语句）。

**#include <[CUnit/CUnit.h](headers/CUnit.h)>**



| **已弃用的名称**           | **等效的新名称**                    |
| :------------------------- | :---------------------------------- |
| `ASSERT`                   | `CU_ASSERT_FATAL`                   |
| `ASSERT_TRUE`              | `CU_ASSERT_TRUE_FATAL`              |
| `ASSERT_FALSE`             | `CU_ASSERT_FALSE_FATAL`             |
| `ASSERT_EQUAL`             | `CU_ASSERT_EQUAL_FATAL`             |
| `ASSERT_NOT_EQUAL`         | `CU_ASSERT_NOT_EQUAL_FATAL`         |
| `ASSERT_PTR_EQUAL`         | `CU_ASSERT_PTR_EQUAL_FATAL`         |
| `ASSERT_PTR_NOT_EQUAL`     | `CU_ASSERT_PTR_NOT_EQUAL_FATAL`     |
| `ASSERT_PTR_NULL`          | `CU_ASSERT_PTR_NULL_FATAL`          |
| `ASSERT_PTR_NOT_NULL`      | `CU_ASSERT_PTR_NOT_NULL_FATAL`      |
| `ASSERT_STRING_EQUAL`      | `CU_ASSERT_STRING_EQUAL_FATAL`      |
| `ASSERT_STRING_NOT_EQUAL`  | `CU_ASSERT_STRING_NOT_EQUAL_FATAL`  |
| `ASSERT_NSTRING_EQUAL`     | `CU_ASSERT_NSTRING_EQUAL_FATAL`     |
| `ASSERT_NSTRING_NOT_EQUAL` | `CU_ASSERT_NSTRING_NOT_EQUAL_FATAL` |
| `ASSERT_DOUBLE_EQUAL`      | `CU_ASSERT_DOUBLE_EQUAL_FATAL`      |
| `ASSERT_DOUBLE_NOT_EQUAL`  | `CU_ASSERT_DOUBLE_NOT_EQUAL_FATAL`  |

## 3. 测试注册表

### 3.1. 简介

\#include <[CUnit/TestDB.h](headers/TestDB.h)>（由<[CUnit/CUnit.h](headers/CUnit.h)自动包含）>)

```c
  typedef struct CU_TestRegistry                                    //测试组测结构体
  typedef CU_TestRegistry*  CU_pTestRegistry                        //测试注册结构体指针
  CU_ErrorCode     CU_initialize_registry(void)                     //初始化注册表
  void             CU_cleanup_registry(void)                        //清除组测表
  CU_BOOL          CU_registry_initialized(void)                    //判断注册表是否被初始化
  CU_pTestRegistry CU_get_registry(void)                            //获取注册表
  CU_pTestRegistry CU_set_registry(CU_pTestRegistry pTestRegistry)  //设置注册表
  CU_pTestRegistry CU_create_new_registry(void)                     // 创建一个新的注册表
  void             CU_destroy_existing_registry(CU_pTestRegistry* ppRegistry)
```

### 3.2. 内部结构

测试注册表是套件和相关测试的存储库。CUnit 维护一个活动的测试注册表，当用户添加套件或测试时，该注册表会更新。此活动注册表中的套件是在用户选择运行所有测试时运行的套件。

CUnit 测试注册表是在 `headers/TestDB.h` 中声明`CU_TestRegistry`数据结构。它包括存储在注册表中的套件和测试总数的字段，以及指向已注册套件链接列表标题的指针。

```
  typedef struct CU_TestRegistry
  {
    unsigned int uiNumberOfSuites;
    unsigned int uiNumberOfTests;
    CU_pSuite    pSuite;
  } CU_TestRegistry;

  typedef CU_TestRegistry* CU_pTestRegistry;
```

用户通常只需要在使用前初始化注册表，然后在使用后进行清理。但是，还提供了其他功能来在必要时操作注册表。

### 3.3. 初始化

- `CU_ErrorCode CU_initialize_registry（void）`

在使用之前，必须初始化活动的` CUnit `测试注册表。用户在调用任何其他 `CUnit `函数之前应调用 `CU_initialize_registry（）`。如果不这样做，可能会导致崩溃。如果多次调用此函数，则在创建新注册表之前，将清理（即销毁！）任何现有注册表。此函数不能在测试运行期间调用（即从 测试函数或套件 初始化/清理函数）。

返回错误状态代码：

| CUE_SUCCESS  | 初始化成功。   |
| ------------ | -------------- |
| CUE_NOMEMORY | 内存分配失败。 |



- `CU_BOOL CU_registry_initialized（void）`

此函数可用于检查注册表是否已初始化。如果注册表安装程序分布在多个文件上，而这些文件需要确保注册表已准备好进行测试注册，则这可能很有用。

### 3.4. 清理

- `void CU_cleanup_registry(void)`

测试完成后，用户应调用此函数来清理和释放框架使用的内存。这应该是调用的最后一个` CUnit `函数（使用 [CU_initialize_registry（） 或 CU_set_registry（）](#init) 还原测试注册表[除外](#setreg)）。

不调用 `CU_cleanup_registry（）` 将导致内存泄漏。可以多次调用它，而不会创建错误条件。*请注意，此函数将销毁注册表中的所有套件（以及关联的测试）。*清理注册表后，不应取消 引用指向已注册套件和测试的指针。此函数不能在测试运行期间调用（即从测试函数或套件初始化/清理函数）。

调用CU_cleanup_registry（） 只会影响 CUnit 框架维护的内部[CU_TestRegistry](#CU_TestRegistry)。销毁用户拥有的任何其他测试注册表是用户的责任。这可以通过调用[CU_destroy_existing_registry（）](#destroy)来显式完成，也可以通过使用CU_set_registry[（）](#setreg)使注册表处于活动状态并再次调用CU_cleanup_registry（）来隐式完成。

### 3.5. 其他注册表函数

其他注册表功能主要用于内部和测试目的。但是，一般用户可能会发现它们的用途，并且应该注意它们。

这些包括：

- `CU_pTestRegistry CU_get_registry(void)`

返回指向活动测试注册表的指针。注册表是[数据类型CU_TestRegistry](#CU_TestRegistry)的变量。不建议直接操作内部测试注册表 - 应改用 API 函数。框架保留注册表的所有权，因此返回的指针将通过调用[CU_cleanup_registry（）或CU_initialize_registry（](#cleanup)[）而](#init)失效。

`CU_pTestRegistry CU_set_registry（CU_pTestRegistry pTestRegistry）`

将活动注册表替换为指定的注册表。返回指向上一个注册表的指针。***调用方有责任销毁旧注册表\***。这可以通过为返回的指针调用 [CU_destroy_existing_registry（）](#destroy) 来明确地完成。或者，可以使用 [CU_set_registry（）](#setreg) 使注册表处于活动状态，并在调用 [CU_cleanup_registry（）](#cleanup) 时隐式销毁注册表。应注意不要显式销毁设置为活动注册表的注册表。这可能会导致多个释放相同的内存，并可能崩溃。

- `CU_pTestRegistry CU_create_new_registry(void)`

创建一个新的注册表并返回指向该注册表的指针。新的注册表将不包含任何套件或测试。调用方有责任通过前面描述的机制之一销毁新注册表。

- `void CU_destroy_existing_registry (CU_pTestRegistry* ppregistry)`

销毁并释放指定测试注册表的所有内存，包括任何已注册的套件和测试。不应为设置为活动测试注册表的注册表调用此函数（例如，使用CU_get_registry[（）](#getreg)检索CU_pTestRegistry指针）。这将导致在调用[CU_cleanup_registry（）](#cleanup) 时多个释放相同的内存。ppRegistry 可能不是 ，但它指向的指针可能是 。在这种情况下，该函数不起作用。请注意，*ppRegistry 将在返回时设置为。`NULL``NULL`

### 3.6. 不推荐使用的 v1 数据类型和函数

从版本 2 开始，以下数据类型和函数已弃用。若要使用这些不推荐使用的名称，必须使用定义USE_DEPRECATED_CUNIT_NAMES来编译用户代码。

**#include <[CUnit/TestDB.h](headers/TestDB.h)>**（由[CUnit/CUnit.h](headers/CUnit.h)自动包含>）。



| **已弃用的名称**                                             | **等效的新名称**                                             |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `_TestRegistry`                                              | [CU_TestRegistry](#CU_TestRegistry)                          |
| `_TestRegistry.uiNumberOfGroupsPTestRegistry->uiNumberOfGroups` | `CU_TestRegistry.uiNumberOfSuitesCU_pTestRegistry->uiNumberOfSuites` |
| `_TestRegistry.pGroupPTestRegistry->pGroup`                  | `CU_TestRegistry.pSuiteCU_pTestRegistry->pSuite`             |
| `PTestRegistry`                                              | [CU_pTestRegistry](#CU_TestRegistry)                         |
| `initialize_registry()`                                      | [CU_initialize_registry（）](#init)                          |
| `cleanup_registry()`                                         | [CU_cleanup_registry（）](#cleanup)                          |
| `get_registry()`                                             | [CU_get_registry（）](#getreg)                               |
| `set_registry()`                                             | [CU_set_registry（）](#setreg)                               |

## 4. 管理测试和套件

为了使测试由 CUnit 运行，必须将其添加到已向测试注册表注册的测试集合（套件）[中](test_registry.html)。

### 4.1. 简介

\#include <[CUnit/TestDB.h](headers/TestDB.h)>（由<[CUnit/CUnit.h](headers/CUnit.h)自动包含）>)

```c
  typedef struct CU_Suite  //CUnit的套件结构体
  typedef CU_Suite*  CU_pSuite //套件指针

  typedef struct CU_Test  // Cunit的测试结构体
  typedef CU_Test*  CU_pTest //测试指针

  typedef void (*CU_TestFunc)(void) //测试函数回调
  typedef int  (*CU_InitializeFunc)(void) //初始化函数回调
  typedef int  (*CU_CleanupFunc)(void) // 清理函数回调

  /* 添加套件*/
  CU_pSuite CU_add_suite(const char* strName,
                         CU_InitializeFunc pInit,
                         CU_CleanupFunc pClean); 

  /*添加测试用例*/
  CU_pTest  CU_add_test(CU_pSuite pSuite,
                        const char* strName,
                        CU_TestFunc pTestFunc);

  typedef struct CU_TestInfo;// 测试用例信息结构体
  typedef struct CU_SuiteInfo; // 套件信息结构体

  /*注册套件*/
  CU_ErrorCode CU_register_suites(CU_SuiteInfo suite_info[]);
  CU_ErrorCode CU_register_nsuites(int suite_count, ...);
  
 /*设置套件激活*/
  CU_ErrorCode CU_set_suite_active(CU_pSuite pSuite, CU_BOOL fNewActive);
  /*设置测试用例激活*/
  CU_ErrorCode CU_set_test_active(CU_pTest, CU_BOOL fNewActive);

  /*设置套件名字 */
  CU_ErrorCode CU_set_suite_name(CU_pSuite pSuite, const char *strNewName);
  /*设置套件初始化函数*/
  CU_ErrorCode CU_set_suite_initfunc(CU_pSuite pSuite, CU_InitializeFunc pNewInit);
  /*设置套件清理函数*/
  CU_ErrorCode CU_set_suite_cleanupfunc(CU_pSuite pSuite, CU_CleanupFunc pNewClean);

  /*设置测试用例名字*/
  CU_ErrorCode CU_set_test_name(CU_pTest pTest, const char *strNewName);
  /*设置测试用例函数*/
  CU_ErrorCode CU_set_test_func(CU_pTest pTest, CU_TestFunc pNewFunc);
      
  /*获取套件*/
  CU_pSuite CU_get_suite(const char* strName);
  /*通过pos来获取套件的位置*/
  CU_pSuite CU_get_suite_at_pos(unsigned int pos);
  /*通过套件句柄获取套件的位置*/
  unsigned int CU_get_suite_pos(CU_pSuite pSuite);
  /*通过套件名字获取套件位置*/
  unsigned int CU_get_suite_pos_by_name(const char* strName);
  
  /*获取测试用例*/
  CU_pTest CU_get_test(CU_pSuite pSuite, const char *strName);
  /*通过pos获取测试用例*/
  CU_pTest CU_get_test_at_pos(CU_pSuite pSuite, unsigned int pos);
  /*通过测试用例句柄获取测试用例*/
  unsigned int CU_get_test_pos(CU_pSuite pSuite, CU_pTest pTest);
  /*通过名字获取测试用例*/
  unsigned int CU_get_test_pos_by_name(CU_pSuite pSuite, const char *strName);
```

### 4.2. 将套件添加到注册表

`CU_pSuite CU_add_suite（const char* strName， CU_InitializeFunc pInit， CU_CleanupFunc pClean）`

创建具有指定名称、初始化函数和清理函数的新测试集合（套件）。新套件已向[测试注册表](test_registry.html)注册（并由其拥有），因此在添加任何套件之前必须[初始化](test_registry.html#init)注册表。当前实现不支持创建独立于测试注册表的套件。此函数不能在测试运行期间调用（即从测试函数或套件初始化/清理函数）。

建议套件的名称在注册表中的所有套件中是唯一的。这有助于按名称查找套件，从而仅查找具有给定名称的第一个套件。初始化和清理函数是可选的，并且作为指针传递给要在运行套件中包含的测试之前和之后调用的函数。这允许套件设置和拆卸临时夹具以支持运行测试。这些函数不带任何参数，如果它们成功完成，则应返回零（否则为非零）。如果套件不需要这些函数中的一个或两个，请传递给`CU_add_suite（）`。

将返回指向新套件的指针，这是向套件添加测试所必需的。如果发生致命错误，则指针将为NULL。此外，框架[错误代码](error_handling.html)设置为以下之一：

| CUE_SUCCESS      | 套件创建成功。               |
| ---------------- | ---------------------------- |
| CUE_NOREGISTRY   | 错误：注册表尚未初始化。     |
| CUE_NO_SUITENAME | 错误： strName 是 。`NULL`   |
| CUE_DUP_SUITE    | 警告：套件的名称不是唯一的。 |
| CUE_NOMEMORY     | 错误：内存分配失败。         |

### 4.3. 向套件添加测试

`CU_pTest CU_add_test（CU_pSuite pSuite， const char* strName， CU_TestFunc pTestFunc）`

创建具有指定名称和测试函数的新测试，并将其注册到指定的套件。套件必须已使用 [CU_add_suite（） 创建](#addsuite)。当前实现不支持创建独立于已注册套件的测试。此函数不能在测试运行期间调用（即从测试函数或套件初始化/清理函数）。

建议测试的名称在添加到单个套件的所有测试中是唯一的。这有助于按名称查找测试，这将发现只有第一个测试具有给定名称。测试函数不能为 ，并且指向运行测试时要调用的函数。测试函数既没有参数也没有返回值。

返回指向新测试的指针。如果在创建测试期间发生致命错误，则返回`NULL` 。另外。框架[错误代码](error_handling.html)设置为以下之一：

| CUE_SUCCESS     | 套件创建成功。                   |
| --------------- | -------------------------------- |
| CUE_NOREGISTRY  | 错误：注册表尚未初始化。         |
| CUE_NOSUITE     | 错误：指定的套件已或无效。`NULL` |
| CUE_NO_TESTNAME | 错误： strName 是 。`NULL`       |
| CUE_NO_TEST     | 错误：pTestFunc 是或无效。`NULL` |
| CUE_DUP_TEST    | 警告：测试的名称不是唯一的。     |
| CUE_NOMEMORY    | 错误：内存分配失败。             |



### 4.4. 管理测试的快捷方式

`#define CU_ADD_TEST（suite， test） （CU_add_test（suite， #test， （CU_TestFunc）test））`

此宏根据测试函数名称==自动生成唯一的测试名称==，并将其添加到指定的套件中。用户应检查返回值以验证是否成功。

`CU_ErrorCode CU_register_suites（CU_SuiteInfo suite_info[]）`

`CU_ErrorCode CU_register_nsuites（int suite_count， ...）`

对于具有许多测试和套件的大型测试结构，管理测试/套件关联和注册是繁琐且容易出错的。`CUnit `提供了一个特殊的注册系统来帮助管理套件和测试。它的主要好处是集中注册套件和相关测试，并最大限度地减少用户需要编写的错误检查代码的数量。

测试用例首先分组到`CU_TestInfo`实例数组中（在<[CUnit/TestDB.h](headers/TestDB.h)>中定义）：

```
CU_TestInfo test_array1[] = {
  { "testname1", test_func1 },
  { "testname2", test_func2 },
  { "testname3", test_func3 },
  CU_TEST_INFO_NULL,
};
```

每个数组元素都包含单个测试用例的（唯一）名称和测试函数。数组必须以保存值的元素结尾，宏`CU_TEST_INFO_NULL`方便地定义了该值。单个`CU_TestInfo`数组中包含的测试用例==构成了将注册到单个测试套件的测试集。==

然后，在一个或多个`CU_SuiteInfo`实例数组中定义套件信息（在 <[CUnit/TestDB.h](headers/TestDB.h)> 中定义）：

```
CU_SuiteInfo suites[] = {
  { "suitename1", suite1_init-func, suite1_cleanup_func, test_array1 },
  { "suitename2", suite2_init-func, suite2_cleanup_func, test_array2 },
  CU_SUITE_INFO_NULL,
};
```

这些数组元素中的每一个都包含（唯一的）名称、套件初始化函数、套件清理函数以及单个套件的`CU_TestInfo`数组。像往常一样，如果给定的套件不需要它，则可以将其用于初始化或清理功能。数组必须以 all- 元素结尾，可以使用宏`CU_SUITE_INFO_NULL`。

然后，可以在单个语句中注册CU_SuiteInfo数组中定义的所有套件：

```
CU_ErrorCode error = CU_register_suites(suites);
```

如果在注册任何套件或测试期间发生错误，则会返回错误代码。错误代码与正常[套件注册](#addsuite)和[测试添加](#addtest)操作返回的错误代码相同。函数 `CU_register_nsuites（） `用于用户希望在单个语句中注册多个`CU_SuiteInfo`数组的情况：

```
CU_ErrorCode error = CU_register_nsuites(2, suites1, suites2);
```

此函数接受可变数量的`CU_SuiteInfo`数组。第一个参数指示正在传递的实际数组数。

### 4.5. 套件和测试的激活

`CU_ErrorCode CU_set_suite_active（CU_pSuite pSuite， CU_BOOL fNewActive）`

`CU_ErrorCode CU_set_test_active（CU_pTest pTest， CU_BOOL fNewActive）`

这些函数可用于停用或重新激活各个测试和套件。套件或测试不会在测试运行期间执行，除非它处于活动状态。==默认情况下，所有套件和测试在创建时都处于活动状态。==当前活动状态分别以 `pSuite->fActive` 和 `pTest->fActive` 的形式提供。如果处于活动状态，则该标志将为“如果处于活动状态”这使客户端能够选择要动态运行的测试子集。请注意，停用测试或套件，然后专门请求[运行](running_test/html#overview)它是一个框架错误。这些函数在成功时返回，并且 （或 ）如果相应的套件（或 test） 为        `CU_TRUE` `CU_FALSE` `CUE_SUCCESS` `CUE_NOSUITE` `CUE_NOTEST` `NULL`

### 4.6. 修改套件和测试的其他属性

通常，套件和测试的属性是在创建时设置的。在某些情况下，客户端可能希望操作它们以动态修改测试结构。为此提供了以下函数，应使用这些函数而不是直接设置数据结构成员的值。所有产品在成功时返回，失败时返回指示的错误代码。提供[查找功能](#lookup)以帮助客户查找特定的测试和套件。`CUE_SUCCESS`

CU_ErrorCode **CU_set_suite_name**（CU_pSuite pSuite， const char *strNewName）
CU_ErrorCode **CU_set_test_name**（CU_pTest pTest， const char *strNewName）

这些函数更改已注册套件和测试的名称。当前名称可用作 `pSuite->pName `和 `pTest->pName`数据结构成员。如果套件或测试是 ，则分别返回 或。如果 `strNewName` 为 ，则分别返回 。`NULLCUE_NOSUITE` `CUE_NOTEST`  `CUE_NO_SUITENAME ` `CUE_NO_TESTNAME`

`CU_ErrorCode CU_set_suite_initfunc（CU_pSuite pSuite， CU_InitializeFunc pNewInit）`

`CU_ErrorCode CU_set_suite_cleanupfunc（CU_pSuite pSuite， CU_CleanupFunc pNewClean）`

这些函数更改已注册套件的初始化和清理函数。当前函数可用作 `*pSuite->pInitializeFunc`和 `*pSuite->pCleanupFunc`数据结构成员。如果套件是然后返回`NULLCUE_NOSUITE`

`CU_ErrorCode CU_set_test_func（CU_pTest pTest， CU_TestFunc pNewFunc）`

此函数更改已注册测试的测试函数。当前测试函数可用作 `pTest->pTestFunc` 数据结构成员。如果 `pTest` 或 `pNewFunc` 是 ，则返回`NULLCUE_NOTEST`

### 4.7. 查找各个套件和测试

在大多数情况下，客户端将引用已注册的套件和测试作为从 [CU_add_suite（）](#addsuite) 和 [CU_add_test（）](#addtest) 返回的指针。有时，客户端可能需要能够检索对套件或测试的引用。当客户端具有有关实体的一些信息（名称或注册顺序）时，提供以下函数来协助客户端完成此操作。如果对套件或测试一无所知，客户端将需要迭代内部数据结构以枚举套件和测试。客户端 API 中不直接支持此功能。

`CU_pSuite CU_get_suite（const char* strName）`

`CU_pSuite CU_get_suite_at_pos（unsigned int pos）`

`unsigned int CU_get_suite_pos（CU_pSuite pSuite）`

`unsigned int CU_get_suite_pos_by_name（const char* strName）`

这些函数有助于查找在活动[测试注册表](test_registry.html)中注册的套件。前 2 个函数允许按名称或位置查找套件，如果找不到套件，则返回`NULL`。该位置是 [1 .. 范围内的 1 基索引。`CU_get_registry（）->uiNumberOfSuites`.当注册具有重复名称的套件时，这可能很有用，在这种情况下，按名称查找只能检索具有该名称的第一个套件。第二个 函数可帮助客户端识别已注册套件的位置。如果找不到套件，则返回 0。此外，所有这些函数都将` CUnit `错误状态设置为“如果未[初始化](test_registry.html#init)注册表”。根据需要， 如果 是 ，则设置。`NULLCUE_NOREGISTRY` `CUE_NO_SUITENAME` `strName` `NULLCUE_NOSUITE` `pSuiteNULL`

`CU_pTest CU_get_test（CU_pSuite pSuite， const char *strName）`

`CU_pTest CU_get_test_at_pos（CU_pSuite pSuite， unsigned int pos）`

`unsigned int CU_get_test_pos（CU_pSuite pSuite， CU_pTest pTest）`

`unsigned int CU_get_test_pos_by_name（CU_pSuite pSuite， const char *strName）`

这些函数便于查找在套件中注册的测试。前 2 个函数允许按名称或位置查找测试，如果找不到测试，则返回。该位置是从 1 开始的索引，范围为 [1 .. pSuite->uiNumberOfSuites]。当注册具有重复名称的测试时，这可能很有用，在这种情况下，按名称查找只能检索具有该名称的第一个测试。第二个 2 个函数可帮助客户端识别测试在套件中的位置。如果找不到测试，则返回 0。此外，所有这些函数都将 CUnit 错误状态设置为 if 注册表未[初始化](test_registry.html#init)，如果为 。根据需要， 如果 是 ，则设置，如果 是 ，则设置。`NULLCUE_NOREGISTRY` `CUE_NOSUITE` `pSuite` `NULLCUE_NO_TESTNAME` `strName` `NULLCUE_NOTEST` `pTest` `NULL`

### 4.8. 不推荐使用的 v1 数据类型和函数

从版本 2 开始，以下数据类型和函数已弃用。若要使用这些不推荐使用的名称，必须使用定义USE_DEPRECATED_CUNIT_NAMES来编译用户代码。

**#include <[CUnit/TestDB.h](headers/TestDB.h)>**（由[CUnit/CUnit.h](headers/CUnit.h)自动包含>）。



| **已弃用的名称**      | **等效的新名称**                                     |
| --------------------- | ---------------------------------------------------- |
| `TestFunc`            | [CU_TestFunc](#synopsis)                             |
| `InitializeFunc`      | [CU_InitializeFunc](#synopsis)                       |
| `CleanupFunc`         | [CU_CleanupFunc](#synopsis)                          |
| `_TestCase`           | [CU_Test](#synopsis)                                 |
| `PTestCase`           | [CU_pTest](#synopsis)                                |
| `_TestGroup`          | [CU_Suite](#synopsis)                                |
| `PTestGroup`          | [CU_pSuite](#synopsis)                               |
| `add_test_group()`    | [CU_add_suite（）](#addsuite)                        |
| `add_test_case()`     | [CU_add_test（）](#addtest)                          |
| `ADD_TEST_TO_GROUP()` | [CU_ADD_TEST（）](#shortcuts)                        |
| `test_case_t`         | [CU_TestInfo](#regsuites)                            |
| `test_group_t`        | [CU_SuiteInfo](#regsuites)                           |
| `test_suite_t`        | 无等效项 - 使用[CU_SuiteInfo](#regsuites)            |
| `TEST_CASE_NULL`      | [CU_TEST_INFO_NULL](#regsuites)                      |
| `TEST_GROUP_NULL`     | [CU_SUITE_INFO_NULL](#regsuites)                     |
| `test_group_register` | [CU_register_suites（）](#regsuites)                 |
| `test_suite_register` | 无等效项 - 使用 [CU_register_suites（）](#regsuites) |

## 5. 运行测试

### 5.1. 简介

`#include <CUnit/Automated.h>`

```
  void         CU_automated_run_tests(void)
  CU_ErrorCode CU_list_tests_to_file(void)
  void         CU_set_output_filename(const char* szFilenameRoot)
```

\#include <CUnit/Basic.h>

```
  typedef enum    CU_BasicRunMode
  CU_ErrorCode    CU_basic_run_tests(void)
  CU_ErrorCode    CU_basic_run_suite(CU_pSuite pSuite)
  CU_ErrorCode    CU_basic_run_test(CU_pSuite pSuite, CU_pTest pTest)
  void            CU_basic_set_mode(CU_BasicRunMode mode)
  CU_BasicRunMode CU_basic_get_mode(void)
  void            CU_basic_show_failures(CU_pFailureRecord pFailure)
```

\#include <CUnit/Console.h>

```
  void CU_console_run_tests(void)
```

\#include <CUnit/CUCurses.h>

```
  void CU_curses_run_tests(void)
```

\#include <CUnit/TestRun.h>（由<CUnit/CUnit.h自动包含）>)

```
  unsigned int CU_get_number_of_suites_run(void)
  unsigned int CU_get_number_of_suites_failed(void)
  unsigned int CU_get_number_of_tests_run(void)
  unsigned int CU_get_number_of_tests_failed(void)
  unsigned int CU_get_number_of_asserts(void)
  unsigned int CU_get_number_of_successes(void)
  unsigned int CU_get_number_of_failures(void)

  typedef struct CU_RunSummary
  typedef CU_Runsummary* CU_pRunSummary
  const CU_pRunSummary CU_get_run_summary(void)

  typedef struct CU_FailureRecord
  typedef CU_FailureRecord*  CU_pFailureRecord
  const CU_pFailureRecord CU_get_failure_list(void)
  unsigned int CU_get_number_of_failure_records(void)

  void CU_set_fail_on_inactive(CU_BOOL new_inactive)
  CU_BOOL CU_get_fail_on_inactive(void)
```



### 5.2. 在 CUnit 中运行测试

CUnit 支持在所有已注册的套件中运行所有测试，但也可以运行单个测试或套件。在每次运行期间，框架都会跟踪运行、通过和失败的套件、测试和断言的数量。请注意，每次启动测试运行时都会清除以前的结果（即使它失败）。如果客户端希望从特定测试运行中排除单个套件或测试，则可以[将其停用](managing_tests.html#activation)。但是，停用套件或测试，然后显式请求运行它是[一个框架错误](error_handling.html#errorhandling)。

虽然 CUnit 提供了用于运行套件和测试的基元函数，但大多数用户都希望使用简化的用户界面之一。这些接口处理与框架交互的详细信息，并为用户提供测试详细信息和结果的输出。

CUnit 库中包含以下接口：



| **接口**          | **平台**   | **描述**                     |
| ----------------- | ---------- | ---------------------------- |
| 自动化(automated) | all        | 非交互式输出到 xml 文件      |
| basic             | all        | 非交互式，可选输出到标准输出 |
| console           | all        | 用户控制下的交互式控制台模式 |
| curses            | Linux/Unix | 用户控制下的交互式curses模式 |

如果这些接口不够，客户端还可以使用<[CUnit/TestRun.h](headers/TestRun.h)>中定义的原始框架 API。有关如何直接与原始 API 交互的示例，请参阅各种接口的源代码。

### 5.3. automate模式

自动化界面是非交互式的。客户端启动测试运行，并将结果输出到 XML 文件。还可以将已注册的测试和套件的列表报告到 XML 文件。

以下功能包含自动化接口 API：

`void CU_automated_run_tests(void)`

运行所有已注册（和活动）套件中的所有测试。测试结果将输出到名为 *ROOT-Results.xml。*文件名 *ROOT* 可以使用 [CU_set_output_filename（）](#auto-setroot) 进行设置，也可以使用默认的` CUnitAutomated-Results.xml`。请注意，如果在每次运行之前未设置 `distict `文件名` ROOT`，则结果文件将被覆盖。

文档类型定义文件 （*CUnit-Run.dtd*） 和 XSL 样式表 （*CUnit-Run.xsl*） 都支持结果文件。这些在源树和安装树的*“共享”*子目录中提供。

`CU_ErrorCode CU_list_tests_to_file(void)`

列出要归档的已注册套件和关联的测试。列表文件名为 *ROOT-Listing.xml*。文件名 *ROOT* 可以使用 [CU_set_output_filename（）](#auto-setroot) 进行设置，也可以使用默认的 `CUnitAutomated`。请注意，如果在每次运行之前未设置 `distict` 文件名 `ROOT`，则列表文件将被覆盖。

文档类型定义文件 （*CUnit-List.dtd*） 和 XSL 样式表 （*CUnit-List.xsl*） 都支持列表文件。这些在源树和安装树的*“共享”*子目录中提供。

另请注意，列表文件不是由[CU_automated_run_tests（）](#auto-run)自动生成的。客户端代码必须在需要时显式请求列表。

`void CU_set_output_filename（const char* szFilenameRoot）`

设置结果和列表文件的输出文件名。`szFilenameRoot` 用于通过分别附加 *-Results.xml* 和 *-Listing.xml* 来构造文件名。

### 5.4. basic模式

基本界面也是非交互式的，结果输出到标准输出。此接口支持运行单个套件或测试，并允许客户端代码控制每次运行期间显示的输出类型。此接口为希望简化对 CUnit API 的访问的客户端提供了最大的灵活性。

提供以下公共功能：

`CU_ErrorCode CU_basic_run_tests(void)`

在所有已注册的套件中运行所有测试。仅执行激活的套件，如果遇到并跳过非激活套件，则不会将其视为错误。返回测试运行期间发生的第一个错误代码。输出类型由当前运行模式控制，该模式可以使用[CU_basic_set_mode（）进行](#basic-setmode)设置。

`CU_ErrorCode CU_basic_run_suite（CU_pSuite suite）`

在单个指定的套件中运行所有测试。返回测试运行期间发生的第一个错误代码。CU_basic_run_suite本身设置`CUE_NOSUITE pSuite` 是否为执行，如果 `pSuite` 未处于激活状态，则`CUE_SUITE_INACTIVE`。输出类型由当前运行模式控制，该模式可以使用[CU_basic_set_mode（）进行](#basic-setmode)设置。

`CU_ErrorCode CU_basic_run_test（CU_pSuite suite，CU_pTest pTest）`

在指定的套件中运行单个测试。返回测试运行期间发生的第一个错误代码。`CU_basic_run_test`本身,如果是 `pSuite `则设置为`CUE_NOSUITE`,如果是`pTest`则设置为`CUE_NOTEST`, 如果`pSuite`没有激活，设置为`CUE_SUITE_INACTIVE ` ,如果 `pTest` 不是 `pSuite` 中的注册测试，设置为`CUE_TEST_NOT_IN_SUITE`，如果 `pTest` 不活动，设置为CUE_TEST_INACTIVE。输出类型由当前运行模式控制，该模式可以使用[CU_basic_set_mode（）进行](#basic-setmode)设置。

`void  CU_basic_set_mode （CU_BasicRunMode mode）`

设置基本运行模式，该模式在测试运行期间控制输出。选项包括：

| CU_BRM_NORMAL  | 打印失败和运行摘要。           |
| -------------- | ------------------------------ |
| CU_BRM_SILENT  | 除错误消息外，不打印任何输出。 |
| CU_BRM_VERBOSE | 运行详细信息的最大输出。       |



`CU_BasicRunMode CU_basic_get_mode(void)`

检索当前的基本运行模式代码。

`void CU_basic_show_failures（CU_pFailureRecord pFailure）`

将所有故障的摘要打印到 `stdout`。不依赖于运行模式。

### 5.5. 交互式控制台模式

控制台界面是交互式的。客户端需要做的就是启动控制台会话，用户以交互方式控制测试运行。这包括选择和运行已注册的套件和测试，以及查看测试结果。要启动控制台会话，请使用

void CU_console_run_tests(void)

### 5.6. 交互式`curses`模式

curses界面是交互式的。客户端需要做的就是启动 curses 会话，用户以交互方式控制测试运行。这包括选择和运行已注册的套件和测试，以及查看测试结果。使用此接口需要将 `ncurses `库链接到应用程序中。要启动curses会话，请使用

`void CU_curses_run_tests(void)`

### 5.7. 修改一般运行时行为

以下函数允许客户端在测试运行期间修改框架的行为：

`void CU_set_fail_on_inactive（CU_BOOL new_inactive）`

`CU_BOOL CU_get_fail_on_inactive（void）`

默认运行时行为用于将非活动套件和测试报告为失败。这样，客户端就会知道他们的测试结构已被部分停用。如果客户端希望忽略非活动套件和测试，则可以使用这些函数修改行为。值 of 表示框架将忽略非活动实体; 将把它们视为失败。`CU_FALSE` `CU_TRUE`

### 5.8. 获取测试结果

这些接口显示测试运行的结果，但客户端代码有时可能需要直接访问结果。这些结果包括各种运行计数，以及保存故障详细信息的故障记录链表。请注意，每次启动新的测试运行时，或者[初始化](test_registry.html#init)或[清理](test_registry.html#cleanup)注册表时，都会覆盖测试结果。

用于访问测试结果的函数包括：

`unsigned  int CU_get_number_of_suites_run（void）`

`unsigned int CU_get_number_of_suites_failed（void）`

`unsigned int CU_get_number_of_tests_run（void）`

`unsigned int CU_get_number_of_tests_failed（void）`

`unsigned int CU_get_number_of_asserts（void）`

`unsigned int CU_get_number_of_successes（void）`

`unsigned int CU_get_number_of_failures（void）`

这些函数报告上次运行或失败的套件、测试和断言的数量。如果套件的初始化或清理函数返回非，或者如果套件处于非活动状态，并且框架设置为将[非活动套件/测试视为失败](#modifying-inactive)，则认为套件失败。如果测试的任何断言失败，或者如果测试在相同条件下处于非活动状态，则测试将失败。最后 3 个函数都引用单个断言。非活动套件（或测试）不计入运行的套件（测试）数。这样做的结果是，即使套件（或测试）未报告为已运行，它也可能失败。

若要检索已注册套件和测试的总数，请分别使用 [CU_get_registry（）−>uiNumberOfSuites](test_registry.html#stuct) 和 [CU_get_registry（）−>uiNumberOfTests](test_registry.html#stuct)。

`CU_pRunSummary CU_get_run_summary（void）`

一次检索所有测试结果计数。返回值是指向包含计数的已保存结构的指针。此数据类型在 <[CUnit/TestRun.h](headers/TestRun.h)> 中定义（由 <[CUnit/CUnit.h](headers/CUnit.h)> 自动包含）：

```
typedef struct CU_RunSummary
{
  unsigned int nSuitesRun;
  unsigned int nSuitesFailed;
  unsigned int nTestsRun;
  unsigned int nTestsFailed;
  unsigned int nAsserts;
  unsigned int nAssertsFailed;
  unsigned int nFailureRecords;
} CU_RunSummary;


typedef CU_Runsummary* CU_pRunSummary;
```

与返回的指针关联的结构变量归框架所有，因此用户不应释放或以其他方式更改它。*请注意，一旦启动另一个测试运行，指针可能会失效。*

`CU_pFailureRecord CU_get_failure_list（void）`

检索记录上次测试运行期间发生的任何故障的链接列表（无故障）。返回值的数据类型在 <[CUnit/TestRun.h](headers/TestRun.h)> 中声明（由 <[CUnit/CUnit.h](headers/CUnit.h)> 自动包含）。每个故障记录都包含有关故障位置和性质的信息：`NULL`

```
typedef struct CU_FailureRecord
{
  unsigned int  uiLineNumber;
  char*         strFileName;
  char*         strCondition;
  CU_pTest      pTest;
  CU_pSuite     pSuite;

  struct CU_FailureRecord* pNext;
  struct CU_FailureRecord* pPrev;

} CU_FailureRecord;

typedef CU_FailureRecord*  CU_pFailureRecord;
```

与返回的指针关联的结构变量归框架所有，因此用户不应释放或以其他方式更改它。*请注意，一旦启动另一个测试运行，指针可能会失效。*

`unsigned int  CU_get_number_of_failure_records（void）`

检索 `CU_get_failure_list（）` 返回的失败链接列表中的`CU_FailureRecords`数。请注意，这可能大于失败断言的数量，因为包括套件初始化和清理失败。

### 5.9. 不推荐使用的 v1 数据类型和函数

从版本 2 开始，以下数据类型和函数已弃用。若要使用这些不推荐使用的名称，必须使用定义`USE_DEPRECATED_CUNIT_NAMES`来编译用户代码。

| **已弃用的名称**        | **等效的新名称**                                             |
| ----------------------- | ------------------------------------------------------------ |
| `automated_run_tests()` | [CU_automated_run_tests（）](#auto-run) 加 [CU_list_tests_to_file（）](#auto-list) |
| `set_output_filename()` | [CU_set_output_filename（）](#auto-setroot)                  |
| `console_run_tests()`   | [CU_console_run_tests（）](#console-run)                     |
| `curses_run_tests()`    | [CU_curses_run_tests（）](#curses-run)                       |

## 6. 错误处理

### 6.1. 简介

\#include <[CUnit/CUError.h](headers/CUError.h)>（由<[CUnit/CUnit.h](headers/CUnit.h)自动包含）>)

```
  typedef enum CU_ErrorCode
  CU_ErrorCode   CU_get_error(void);
  const char*    CU_get_error_msg(void);

  typedef enum CU_ErrorAction
  void           CU_set_error_action(CU_ErrorAction action);
  CU_ErrorAction CU_get_error_action(void);
```



### 6.2. `CUnit`错误处理

大多数` CUnit` 函数设置指示框架错误状态的错误代码。某些函数返回代码，而其他函数仅设置代码并返回其他值。提供了两个函数来检查框架错误状态：

`CU_ErrorCode CU_get_error（void）`

`const char* CU_get_error_msg（void）`

第一个返回错误代码本身，而第二个返回描述错误状态的消息。错误代码是在 <[CUnit/CUError.h](headers/CUError.h)> 中定义的`CU_ErrorCode`类型。定义了以下错误代码值：`enum`



| **错误值**            | **描述**                                                     |
| --------------------- | ------------------------------------------------------------ |
| CUE_SUCCESS           | 无错误情况。                                                 |
| CUE_NOMEMORY          | 内存分配失败。                                               |
| CUE_NOREGISTRY        | 测试注册表未初始化。                                         |
| CUE_REGISTRY_EXISTS   | 尝试在不CU_cleanup_registry（） 的情况下CU_set_registry（）。 |
| CUE_NOSUITE           | 必需的CU_pSuite指针为 NULL。                                 |
| CUE_NO_SUITENAME      | 未提供必需CU_Suite名称。                                     |
| CUE_SINIT_FAILED      | 套件初始化失败。                                             |
| CUE_SCLEAN_FAILED     | 套件清理失败。                                               |
| CUE_DUP_SUITE         | 不允许使用重复的套件名称。                                   |
| CUE_SUITE_INACTIVE    | 已请求对非活动套件进行测试运行。                             |
| CUE_NOTEST            | CU_TestFunc指针的必需CU_pTest为 NULL。                       |
| CUE_NO_TESTNAME       | 未提供必需CU_Test名称。                                      |
| CUE_DUP_TEST          | 不允许重复的测试用例名称。                                   |
| CUE_TEST_NOT_IN_SUITE | 测试未在指定的套件中注册。                                   |
| CUE_TEST_INACTIVE     | 已请求为非活动测试运行测试。                                 |
| CUE_FOPEN_FAILED      | 打开文件时出错。                                             |
| CUE_FCLOSE_FAILED     | 关闭文件时出错。                                             |
| CUE_BAD_FILENAME      | 请求的文件名错误（NULL、空、不存在等）。                     |
| CUE_WRITE_ERROR       | 写入文件时出错。                                             |



### 6.3. 框架错误时的行为

遇到错误情况时的默认行为是设置错误代码并继续执行。在这种情况下，失败的断言不被视为“框架错误”。包括所有其他错误条件，包括套件初始化或清理失败，非活动套件或显式运行的测试等。有时，客户端可能更愿意在框架错误时停止测试运行，甚至希望测试应用程序退出。此行为可由用户设置，并为其提供以下功能：

`void CU_set_error_action（CU_ErrorAction action）`

`CU_ErrorAction CU_get_error_action（void）`

错误操作代码是在 <[CUnit/CUError.h](headers/CUError.h)> 中定义的CU_ErrorAction类型。定义了以下错误操作代码：`enum`



| **错误值**  | **描述**                             |
| ----------- | ------------------------------------ |
| CUEA_IGNORE | 发生错误情况时，应继续运行（默认）   |
| CUEA_FAIL   | 发生错误情况时应停止运行             |
| CUEA_ABORT  | 当出现错误情况时，应用程序应退出（） |



### 6.4. 已弃用的 v1 变量和函数

从版本 2 开始，以下变量和函数已弃用。若要使用这些不推荐使用的名称，必须使用定义USE_DEPRECATED_CUNIT_NAMES来编译用户代码。



| **已弃用的名称** | **等效的新名称**                       |
| ---------------- | -------------------------------------- |
| `get_error()`    | [CU_get_error_msg（）](#getmsg)        |
| `error_code`     | 没有。使用[CU_get_error（）](#getcode) |





## 7. `CUnit`源代码中的例子解析

### Examples目录结构

```shell
.
├── AutomatedTest # 自动模式的测试样例，在第5章中的5.3小节中介绍
│   ├── AutomatedTest.c
│   ├── AutomatedTest.dsp
│   ├── AutomatedTest_v1.c
│   ├── Jamfile
│   ├── Makefile
│   ├── Makefile.am
│   ├── Makefile.in
│   └── README
├── BasicTest  # 基础模式的测试样例，在第5章的第5.4小节中介绍
│   ├── BasicTest.c
│   ├── BasicTest.dsp
│   ├── Jamfile
│   ├── Makefile
│   ├── Makefile.am
│   ├── Makefile.in
│   └── README
├── ConsoleTest  # console模式的测试样例，在第5章中的5.5小节中介绍
│   ├── ConsoleTest.c
│   ├── ConsoleTest.dsp
│   ├── ConsoleTest_v1.c
│   ├── Jamfile
│   ├── Makefile
│   ├── Makefile.am
│   ├── Makefile.in
│   └── README
├── CursesTest # curses模式的测试样例，在第5章中的5.5小节中介绍
│   ├── CursesTest.c
│   ├── CursesTest_v1.c
│   ├── Jamfile
│   ├── Makefile
│   ├── Makefile.am
│   ├── Makefile.in
│   └── README
├── Demo_fprintf # 输出模板，如何格式化的输出CUnit的输出
│   └── CUnitExample.c
├── ExampleTests.c # 在上面的样例中所需要的公共函数和变量
├── ExampleTests.h
├── Jamfile
├── Makefile
├── Makefile.am
├── Makefile.in
├── WinTest  # 具有窗口的CUnit
│   ├── Jamfile
│   ├── ReadMe.txt
│   ├── StdAfx.cpp
│   ├── StdAfx.h
│   ├── WinTest.cpp
│   ├── WinTest.dsp
│   └── WinTest_v1.cpp
└── wxWidgetsTest
    ├── Makefile.am
    ├── README
    ├── wxWidgetsTest.c
    └── wxWidgetsTest.rc
```

### 分析`ExampleTest`

首先，先分析`ExampleTest.h`和`ExampleTest.cpp`

`ExampleTest.h`文件

```c
#ifndef CUNIT_EXAMPLETESTS_H_SEEN
#define CUNIT_EXAMPLETESTS_H_SEEN
//兼容c++的编译器
#ifdef __cplusplus
extern "C" {
#endif
/*此函数用于添加测试用例*/
void AddTests(void); 
/*此函数用于打印样例中输出结果*/
void print_example_results(void);

#ifdef __cplusplus
}
#endif

#endif  /* CUNIT_EXAMPLETESTS_H_SEEN */
```

`ExampleTest.cpp`文件

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>

#include "CUnit.h"
#include "ExampleTests.h"

/* WARNING - MAINTENANCE NIGHTMARE AHEAD
 *
 * If you change any of the tests & suites below, you also need
 * to keep track of changes in the result statistics and reflect
 * any changes in the result report counts in print_example_results().
 *
 * Yes, this could have been designed better using a more
 * automated mechanism.  No, it was not done that way.
 */

/* Suite initialization/cleanup functions */
static int suite_success_init(void) { return 0; }
static int suite_success_clean(void) { return 0; }

static int suite_failure_init(void) { return 1;}
static int suite_failure_clean(void) { return 1; }

/*CU_ASSERT的作用是对一个表达断言为GU_TURE(非零值)*/
/*下列三个函数用于模拟测试成功的函数*/
static void testSuccess1(void) { CU_ASSERT(1); }
static void testSuccess2(void) { CU_ASSERT(2); }
static void testSuccess3(void) { CU_ASSERT(3); }
/*模拟测试套件失败*/
static void testSuiteFailure1(void) { CU_ASSERT(0); }
static void testSuiteFailure2(void) { CU_ASSERT(2); }
/*模拟测试失败的函数*/
static void testFailure1(void) { CU_ASSERT(0); }
static void testFailure2(void) { CU_ASSERT(0); }
static void testFailure3(void) { CU_ASSERT(0); }

/* Test functions for CUnit assertions */
/*以下为CUnit断言的测试函数*/
/*简单的断言测试*/
static void testSimpleAssert(void)
{
  CU_ASSERT(1);//non-zero, assert successfully
  CU_ASSERT(!0);//non-zero
  CU_TEST(1);//non-zero

  CU_ASSERT(0);//zero, assert failed
  CU_ASSERT(!1);//zero
  CU_TEST(0);//zero
}

/*失败的断言测试*/
static void testFail(void)
{
  CU_FAIL("This is a failure.");//用指定的消息注册一个通过断言
  CU_FAIL("This is another failure.");
}
/*为True的断言测试 CU_ASSERT_TURE*/
static void testAssertTrue(void)
{
  CU_ASSERT_TRUE(CU_TRUE);
  CU_ASSERT_TRUE(!CU_FALSE);

  CU_ASSERT_TRUE(!CU_TRUE);
  CU_ASSERT_TRUE(CU_FALSE);
}
/*为false的断言测试 CU_ASSERT_FALSE*/
static void testAssertFalse(void)
{
  CU_ASSERT_FALSE(CU_FALSE);
  CU_ASSERT_FALSE(!CU_TRUE);

  CU_ASSERT_FALSE(!CU_FALSE);
  CU_ASSERT_FALSE(CU_TRUE);
}
/*相等断言测试，实际值与期望值相等*/
static void testAssertEqual(void)
{
  CU_ASSERT_EQUAL(10, 10);
  CU_ASSERT_EQUAL(0, 0);
  CU_ASSERT_EQUAL(0, -0);
  CU_ASSERT_EQUAL(-12, -12);

  CU_ASSERT_EQUAL(10, 11);
  CU_ASSERT_EQUAL(0, 1);
  CU_ASSERT_EQUAL(0, -1);
  CU_ASSERT_EQUAL(-12, 12);
}
/*不相等断言测试，实际值与期望值不相等*/
static void testAssertNotEqual(void)
{
  CU_ASSERT_NOT_EQUAL(10, 11);
  CU_ASSERT_NOT_EQUAL(0, -1);
  CU_ASSERT_NOT_EQUAL(-12, -11);

  CU_ASSERT_NOT_EQUAL(10, 10);
  CU_ASSERT_NOT_EQUAL(0, -0);
  CU_ASSERT_NOT_EQUAL(0, 0);
  CU_ASSERT_NOT_EQUAL(-12, -12);
}
/*相同指针的断言*/
static void testAssertPtrEqual(void)
{
  CU_ASSERT_PTR_EQUAL((void*)0x100, (void*)0x100);

  CU_ASSERT_PTR_EQUAL((void*)0x100, (void*)0x101);
}
/*不同指针的断言*/
static void testAssertPtrNotEqual(void)
{
  CU_ASSERT_PTR_NOT_EQUAL((void*)0x100, (void*)0x101);

  CU_ASSERT_PTR_NOT_EQUAL((void*)0x100, (void*)0x100);
}
/*空指针断言*/
static void testAssertPtrNull(void)
{
  CU_ASSERT_PTR_NULL((void*)(NULL));
  CU_ASSERT_PTR_NULL((void*)(0x0));

  CU_ASSERT_PTR_NULL((void*)0x23);
}
/*非空指针断言*/
static void testAssertPtrNotNull(void)
{
  CU_ASSERT_PTR_NOT_NULL((void*)0x100);

  CU_ASSERT_PTR_NOT_NULL(NULL);
  CU_ASSERT_PTR_NOT_NULL((void*)0x0);
}
/*字符串相等断言*/
static void testAssertStringEqual(void)
{
  char str1[] = "test";
  char str2[] = "test";
  char str3[] = "suite";

  CU_ASSERT_STRING_EQUAL(str1, str2);

  CU_ASSERT_STRING_EQUAL(str1, str3);
  CU_ASSERT_STRING_EQUAL(str3, str2);
}
/*字符串不相等断言*/
static void testAssertStringNotEqual(void)
{
  char str1[] = "test";
  char str2[] = "test";
  char str3[] = "suite";

  CU_ASSERT_STRING_NOT_EQUAL(str1, str3);
  CU_ASSERT_STRING_NOT_EQUAL(str3, str2);

  CU_ASSERT_STRING_NOT_EQUAL(str1, str2);
}
/*指定长度字符串相等断言*/
static void testAssertNStringEqual(void)
{
  char str1[] = "test";
  char str2[] = "testgfsg";
  char str3[] = "tesgfsg";

  CU_ASSERT_NSTRING_EQUAL(str1, str2, strlen(str1));
  CU_ASSERT_NSTRING_EQUAL(str1, str1, strlen(str1));
  CU_ASSERT_NSTRING_EQUAL(str1, str1, strlen(str1) + 1);

  CU_ASSERT_NSTRING_EQUAL(str2, str3, 4);
  CU_ASSERT_NSTRING_EQUAL(str1, str3, strlen(str1));
}

static void testAssertNStringNotEqual(void)
{
  char str1[] = "test";
  char str2[] = "tevt";
  char str3[] = "testgfsg";

  CU_ASSERT_NSTRING_NOT_EQUAL(str1, str2, 3);
  CU_ASSERT_NSTRING_NOT_EQUAL(str1, str3, strlen(str1) + 1);

  CU_ASSERT_NSTRING_NOT_EQUAL(str1, str2, 2);
  CU_ASSERT_NSTRING_NOT_EQUAL(str2, str3, 2);
}
/*浮点数相等断言*/
static void testAssertDoubleEqual(void)
{
  CU_ASSERT_DOUBLE_EQUAL(10, 10.0001, 0.0001);
  CU_ASSERT_DOUBLE_EQUAL(10, 10.0001, -0.0001);
  CU_ASSERT_DOUBLE_EQUAL(-10, -10.0001, 0.0001);
  CU_ASSERT_DOUBLE_EQUAL(-10, -10.0001, -0.0001);

  CU_ASSERT_DOUBLE_EQUAL(10, 10.0001, 0.00001);
  CU_ASSERT_DOUBLE_EQUAL(10, 10.0001, -0.00001);
  CU_ASSERT_DOUBLE_EQUAL(-10, -10.0001, 0.00001);
  CU_ASSERT_DOUBLE_EQUAL(-10, -10.0001, -0.00001);
}

static void testAssertDoubleNotEqual(void)
{
  CU_ASSERT_DOUBLE_NOT_EQUAL(10, 10.001, 0.0001);
  CU_ASSERT_DOUBLE_NOT_EQUAL(10, 10.001, -0.0001);
  CU_ASSERT_DOUBLE_NOT_EQUAL(-10, -10.001, 0.0001);
  CU_ASSERT_DOUBLE_NOT_EQUAL(-10, -10.001, -0.0001);

  CU_ASSERT_DOUBLE_NOT_EQUAL(10, 10.001, 0.01);
  CU_ASSERT_DOUBLE_NOT_EQUAL(10, 10.001, -0.01);
  CU_ASSERT_DOUBLE_NOT_EQUAL(-10, -10.001, 0.01);
  CU_ASSERT_DOUBLE_NOT_EQUAL(-10, -10.001, -0.01);
}
/*测试用例的中断*/
static void testAbort(void)
{
  CU_TEST_FATAL(CU_TRUE);
  CU_TEST_FATAL(CU_FALSE);
  fprintf(stderr, "\nFatal assertion failed to abort test in testAbortIndirect1\n");
  exit(1);
}
/*测试用例的间接中断*/
static void testAbortIndirect(void)
{
  testAbort();
  fprintf(stderr, "\nFatal assertion failed to abort test in testAbortIndirect2\n");
  exit(1);
}

static void testFatal(void)
{
  CU_TEST_FATAL(CU_TRUE);
  testAbortIndirect();
  fprintf(stderr, "\nFatal assertion failed to abort test in testFatal\n");
  exit(1);
}

static CU_TestInfo tests_success[] = {
  { "testSuccess1", testSuccess1 },
  { "testSuccess2", testSuccess2 },
  { "testSuccess3", testSuccess3 },
	CU_TEST_INFO_NULL,
};

static CU_TestInfo tests_failure[] = {
  { "testFailure1", testFailure1 },
  { "testFailure2", testFailure2 },
  { "testFailure3", testFailure3 },
	CU_TEST_INFO_NULL,
};

static CU_TestInfo tests_suitefailure[] = {
  { "testSuiteFailure1", testSuiteFailure1 },
  { "testSuiteFailure2", testSuiteFailure2 },
	CU_TEST_INFO_NULL,
};

static CU_TestInfo tests_simple[] = {
  { "testSimpleAssert", testSimpleAssert },
  { "testFail", testFail },
	CU_TEST_INFO_NULL,
};

static CU_TestInfo tests_bool[] = {
  { "testAssertTrue", testAssertTrue },
  { "testAssertFalse", testAssertFalse },
	CU_TEST_INFO_NULL,
};

static CU_TestInfo tests_equal[] = {
  { "testAssertEqual", testAssertEqual },
  { "testAssertNotEqual", testAssertNotEqual },
	CU_TEST_INFO_NULL,
};

static CU_TestInfo tests_ptr[] = {
  { "testAssertPtrEqual", testAssertPtrEqual },
  { "testAssertPtrNotEqual", testAssertPtrNotEqual },
	CU_TEST_INFO_NULL,
};

static CU_TestInfo tests_null[] = {
  { "testAssertPtrNull", testAssertPtrNull },
  { "testAssertPtrNotNull", testAssertPtrNotNull },
	CU_TEST_INFO_NULL,
};

static CU_TestInfo tests_string[] = {
  { "testAssertStringEqual", testAssertStringEqual },
  { "testAssertStringNotEqual", testAssertStringNotEqual },
	CU_TEST_INFO_NULL,
};

static CU_TestInfo tests_nstring[] = {
  { "testAssertNStringEqual", testAssertNStringEqual },
  { "testAssertNStringNotEqual", testAssertNStringNotEqual },
	CU_TEST_INFO_NULL,
};

static CU_TestInfo tests_double[] = {
  { "testAssertDoubleEqual", testAssertDoubleEqual },
  { "testAssertDoubleNotEqual", testAssertDoubleNotEqual },
	CU_TEST_INFO_NULL,
};

static CU_TestInfo tests_fatal[] = {
  { "testFatal", testFatal },
	CU_TEST_INFO_NULL,
};
//套件组
static CU_SuiteInfo suites[] = {
  { "suite_success_both",  suite_success_init, suite_success_clean, NULL, NULL, tests_success},
  { "suite_success_init",  suite_success_init, NULL,                NULL, NULL, tests_success},
  { "suite_success_clean", NULL,               suite_success_clean, NULL, NULL, tests_success},
  { "test_failure",        NULL,               NULL,                NULL, NULL, tests_failure},
  { "suite_failure_both",  suite_failure_init, suite_failure_clean, NULL, NULL, tests_suitefailure}, /* tests should not run */
  { "suite_failure_init",  suite_failure_init, NULL,                NULL, NULL, tests_suitefailure}, /* tests should not run */
  { "suite_success_but_failure_clean", NULL,   suite_failure_clean, NULL, NULL, tests_suitefailure}, /* tests will run, suite counted as running, but suite tagged as a failure */
  { "TestSimpleAssert",    NULL,               NULL,                NULL, NULL, tests_simple},
  { "TestBooleanAssert",   NULL,               NULL,                NULL, NULL, tests_bool},
  { "TestEqualityAssert",  NULL,               NULL,                NULL, NULL, tests_equal},
  { "TestPointerAssert",   NULL,               NULL,                NULL, NULL, tests_ptr},
  { "TestNullnessAssert",  NULL,               NULL,                NULL, NULL, tests_null},
  { "TestStringAssert",    NULL,               NULL,                NULL, NULL, tests_string},
  { "TestNStringAssert",   NULL,               NULL,                NULL, NULL, tests_nstring},
  { "TestDoubleAssert",    NULL,               NULL,                NULL, NULL, tests_double},
  { "TestFatal",           NULL,               NULL,                NULL, NULL, tests_fatal},
	CU_SUITE_INFO_NULL,
};

//添加测试用例
void AddTests(void)
{
  assert(NULL != CU_get_registry());//判断有没有注册表
  assert(!CU_is_test_running()); //测试用例是否在运行

	/* Register suites. 注册套件组*/
	if (CU_register_suites(suites) != CUE_SUCCESS) {
		fprintf(stderr, "suite registration failed - %s\n",
			CU_get_error_msg());//打印错误信息
		exit(EXIT_FAILURE);
	}
}

void print_example_results(void)
{
  fprintf(stdout, "\n\nExpected Test Results:"
                  "\n\n  Error Handling  Type      # Run   # Pass   # Fail"
                  "\n\n  ignore errors   suites%9u%9u%9u"
                    "\n                  tests %9u%9u%9u"
                    "\n                  asserts%8u%9u%9u"
                  "\n\n  stop on error   suites%9u%9u%9u"
                    "\n                  tests %9u%9u%9u"
                    "\n                  asserts%8u%9u%9u\n\n",
                  14, 14, 3,
                  31, 10, 21,
                  89, 47, 42,
                  4, 4, 1,
                  12, 9, 3,
                  12, 9, 3);
}

```

ExampleTest的主要意图，用于测试CUnit中的断言，整个程序的框架为

1. 声明测试用例函数
2. 管理测试测试用例
3. 管理套件
4. 将测试用例加入到套件
5. 套件加入到注册表
6. 初始化注册表
7. 运行测试
8. 清理注册表

### 分析automated模式样例

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include "Automated.h"
#include "ExampleTests.h"

int main(int argc, char* argv[])
{
  CU_BOOL Run = CU_FALSE ;

  setvbuf(stdout, NULL, _IONBF, 0);

  if (argc > 1) {
    if (!strcmp("-i", argv[1])) {
      Run = CU_TRUE ;
      CU_set_error_action(CUEA_IGNORE);
    }
    else if (!strcmp("-f", argv[1])) {
      Run = CU_TRUE ;
      CU_set_error_action(CUEA_FAIL);
    }
    else if (!strcmp("-A", argv[1])) {
      Run = CU_TRUE ;
      CU_set_error_action(CUEA_ABORT);
    }
    else if (!strcmp("-e", argv[1])) {
      print_example_results();
    }
    else {
      printf("\nUsage:  AutomatedTest [option]\n\n"
               "        Options: -i  Run, ignoring framework errors [default].\n"
               "                 -f  Run, failing on framework error.\n"
               "                 -A  Run, aborting on framework error.\n"
               "                 -e  Print expected test results and exit.\n"
               "                 -h  Print this message.\n\n");
    }
  }
  else {
    Run = CU_TRUE;
    CU_set_error_action(CUEA_IGNORE);
  }

  if (CU_TRUE == Run) {
    if (CU_initialize_registry()) {
      printf("\nInitialization of Test Registry failed.");
    }
    else {
        /*核心在此*/
      AddTests();
      CU_set_output_filename("TestAutomated");
      CU_list_tests_to_file();
      CU_automated_run_tests();
      CU_cleanup_registry();
    }
  }

  return 0;
}

```

### 分析console模式样例

```c
if (CU_TRUE == Run) {
    if (CU_initialize_registry()) {
      printf("\nInitialization of Test Registry failed.");
    }
    else {
      AddTests();
      CU_console_run_tests();
      CU_cleanup_registry();
    }
  }
```

### 分析basic模式样例

```c
if (CU_initialize_registry()) {
    printf("\nInitialization of Test Registry failed.");
  }
  else {
    AddTests();
    CU_basic_set_mode(mode);
    CU_set_error_action(error_action);
    printf("\nTests completed with return value %d.\n", CU_basic_run_tests());
    CU_cleanup_registry();
  }

```

### 分析Curses模式样例

```c
 if (CU_TRUE == Run) {
    if (CU_initialize_registry()) {
      printf("\nInitialization of Test Registry failed.");
    }
    else {
      AddTests();
      CU_curses_run_tests();
      CU_cleanup_registry();
    }
  }
```



