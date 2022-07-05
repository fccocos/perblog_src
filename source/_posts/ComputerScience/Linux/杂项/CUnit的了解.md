## 一、CUnit简介

CUnit是一个对C语言编写的程序进行单元测试的框架。它作为一个静态链接库被链接到用户的测试代码中。它提供了一种简洁的框架来建立测试架构，并提供丰富的断言(Assertion)来测试通用数据类型。除此之外，它还提供了许多不同的结构来运行测试用例和报告测试结果。


## 二、CUint测试流程
1. 编写单元测试函数(有必要的话要写suite的init/cleanup函数)。
2. 调用函数CU_initialize_registry()初始化测试注册单元(Test Registry)。
3. 调用函数CU_add_suite() 将测试包(suite)添加到测试注册单元(Test Registry)中。
4. 调用函数CU_add_test()将测试用例添加到测试包(suite)中。
5. 使用合适的接口来运行测试用例。
6. 调用函数CU_cleanup_registry清除测试注册单元(Test Registry)。

> ### tips: 编写单元测试函数==》初始化测试注册单元==》将测试包添加到测试注册单元==》添加测试用例到测试注册单元==》选用合适的接口运行测试用例==》清除测试注册单元


## 三、CUint四种测试模式

1. Automated Output to xml file Non-interactive `将结果输入到XML文档中，便于生成报告。`
```c
CU_set_output_filename("testxxx");
CU_list_tests_to_file();
CU_automated_run_tests();
```

2. Basic  Flexible programming interface Non-interactive `每一次运行结束后在standard output中显示测试结果，不能保留测数据结果`

3. Console Console interface (ansi C) Interactive `可以在终端中，进行人机交互`
```c
CU_basic_set_mode(CU_BRM_VERBOSE);
CU_basic_run_tests();
```

4. Curses Graphical interface (Unix)Interactive `unix的图形界面`


## 四、CUnit的安装

```shell
1. 下载CUnit-2.1.0, https://sourceforge.net/projects/cunit
2. $tar -zxvf CUnit的压缩包
3. $cd CUnit-2.1.0
4. $sudo apt install libtool
5. $sudo apt install libsysm-dev
6. $sudo apt install autocmake
7. $sudo apt install autoconf
需要注意的是，如果没有configure.ac文件，则将configure.in改名为configure.ac
8. $mv configure.in  configure.ac
9. $aclocal
10. $autohader
11. $autoconf
12. $automake # 如果在此处有"./ltmain.sh not found"报错,请执行13的命令
13. $libtoolize --automake --copy --debug --force # 执行完后，继续执行12
14. $automake # 如果还是出现报错，请按照提示下载相关程序, 重复次步骤,直到不报错为止
15. $./configure # 当上述步骤都执行成功，则会在目录生成一个configure文件；如果你想指定生成库目录的路径，则可以在使用'./configure --prefix=/path/to'
16. $make 
17. $sudo make install # 最好加上sudo,因为CUnit配置的路径需要管理员权限才能进入

到此安装完毕，可以进入到`/usr/local/lib`中查看是否生成CUnit的动态库
```


## CUnit测试被测函数

根据被测函数编写相应的测试函数，建议以testXXX格式命名

使用断言来判断是否成功，例如CU_ASSERT、CU_ASSERT_TRUE、CU_ASSERT_FALSE、CU_ASSERT_EQUAL等

1. Cunit初始化CU_initialize_registry()

2. 添加注册套件CU_add_suite()

3. 添加注册函数CU_add_test()

4. 调用测试套件并设置测试报告输出模式

5. 清理注册信息CU_cleanup_registry()


## CUnit的断言

|断言          |作用                      |
|:------------ |:------------------------ |
|CU_PASS(msg)  | 做一条"通过"的断言       |
|CU_FAIL(msg)  | 故意做一条"错误"的断言   |
|CU_TEST(value)| 测试条件|
|CU_ASSERT_TRUE(value)|断言正确|
|CU_ASSERT_FALSE(value)|断言错误|
|CU_ASSERT_EQUAL(value)|断言相等|
|CU_ASSERT_PTR_EQUAL(actual,expected)|断言指针指向同一区域|
|CU_ASSERT_PTR_NOT_EQUAL(actual,expected)|断言指针指向不同区域|
|CU_ASSERT_STRING_EQUAL(actual,expected)|断言字符串内容相等|
|CU_ASSERT_STRING_NOT_EQUAL(actual,expected)|断言字符串内容不相等|
|CU_ASSERT_DOUBLE_EQUAL(actual,expected,granlarity)|断言`double actal == expected within the specified tolerance`|
|CU_ASSERT_DOUBLE_NOT_EQUAL(actual,expected,granlarity)|断言`double actal i= expected within the specified tolerance`|


## CUnit测试用例

代码组织形式

```shell
|
|__ func.c # 被测函数
|__ test_case.c # 测试用例 
|__ test_run.c # 运行测试用例

```







