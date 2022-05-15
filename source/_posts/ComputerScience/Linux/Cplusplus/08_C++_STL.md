---
title: 08_C++_STL
date: 2022/4/18/ 16:00
categories:
  - 计算机科学
tags:
  - 嵌入式
  - C++语言
  - 笔记
sticky: true
valine:
  placeholder: "1. 提问前请先仔细阅读本文档⚡\n2. 页面显示问题💥，请提供控制台截图📸或者您的测试网址\n3. 其他任何报错💣，请提供详细描述和截图📸，祝食用愉快💪"
---



# C++STL

## STL概述

长久以来，软件界一直希望建立一种可重复利用的东西，以及一种得以制造出”可重复运用的东西”的方法，让程序员的心血不止于随时间的迁移，人事异动而烟消云散，从函数(functions),类别(classes),函数库(function libraries),类别库(class libraries)、各种组件，从模块化设计，到面向对象(object oriented),为的就是复用性的提升。

复用性必须建立在某种标准之上。但是在许多环境下，就连软件开发最基本的数据结构(data structures)和算法(algorithm)都未能有一套标准。大量程序员被迫从事大量重复的工作，竞然是为了完成前人己经完成而自己手上并未拥有的程序代码，这不仅是人力资源的浪费，也是挫折与痛苦的来源。

为了建立数据结构和算法的一套标准，并且降低他们之间的耦合关系，以提升各自的独立性、弹性、交互操作性（相
互合作性，interoperability),诞生了sTL。

## STL基本概念

STL(Standard Template Library,标准模板库)，是惠普实验室开发的一系列软件的统称。现在主要出现在c++中，但是在引入c+之前该技术已经存在很长时间了。

STL从广义上分为：容器(container)、算法(algorithm)、迭代器(iterator),容器和算法之间通过迭代器进行无缝连接。

STL几乎所有的代码都采用了模板类或者模板函数，这相比传统的由函数和类组成的库来说提供了更好的代码重用机会。STL(Standard Template Library)标准模板库，在我们c+标准程序库中隶属于STL的占到了80%以上。

## STL的六大组件简介

STL提供了六大组件，彼此之间可以组合套用，这六大组件分别是：容器、算法、迭代器、仿函数、适配器（配接器)、空间配置器。

- 容器：各种数据结构，如vector、.lst、deque、set、map等，用来存放数据，从实现角度来看，STL容器是一种class template.

- 算法：各种常用的算法，如sort、find、copy、for_each。从实现的角度来看，sTL算法是一种function template.

- 迭代器：扮演了容器与算法之间的胶合剂，共有五种类型，从实现角度来看，迭代器是一种将operator*,operator->,operator++,operator--等指针相关操作予以重载的class template.,所有STL容器都附带有自己专属的迭代器，只有容器的设计者才知道如何遍历自己的元素。原生指针(native pointer)也是一种迭代器。

- 仿函数：本质是函数对象，行为类似函数，可作为算法的某种策略。从实现角度来看，仿函数是一种重载了operator()的class或者class template

- 适配器：一种用来修饰容器或者仿函数或迭代器接口的东西（一般用来扩展参数接口）。

- 空间配置器：负责空间的配置与管理。从实现角度看，配置器是一个实现了动态空间配置、空间管理、空间释放的class template

STL六大组件的交互关系，容器通过空间配置器取得数据存储空间，算法通过迭代器存储容器中的内容，仿函数可以协助算法完成不同的策略的变化，适配器可以修饰仿函数。

## STL的优点

STL是C++的一部分，因此不用额外安装什么，它被内建在你的编译器之内。

STL的一个重要特性是将数据和操作分离。数据由容器类别加以管理，操作则由可定制的算法定义。迭代器在两者之间充当“粘合剂”以使算法可以和容器交互运作。

程序员可以不用思考STL具体的实现过程，只要能够熟练使用STL就OK了。这样他们就可以把精力放在程序开发的别的方面。

STL具有**高可重用性**，**高性能**，**高移植性**，**跨平台**的优点。

- 高可重用性：STL中几乎所有的代码都采用了模板类和模版函数的方式实现，这相比于传统的由函数和类组成的库来说提供了更好的代码重用机会。关于模板的知识，已经给大家介绍了。

- 高性能：如map可以高效地从十万条记录里面查找出指定的记录，因为map是采用红黑树的变体实现的。

- 高移植性：如在项目A上用STL编写的模块，可以直接移植到项目B上。

## STL的三大组件

### 容器

容器，置物之所也。

研究数据的特定排列方式，以利于搜索或排序或其他特殊目的，这一门学科我们称为数据结构。大学信息类相关专业里面，与编程最有直接关系的学科，首推数据结构与算法。几乎可以说任何特定的数据结构都是为了实现某种特定的算法。STL容器就是将运用最广泛的一些数据结构实现出来。

常用的数据结构：数组(array),链表(list),tree(树)，栈(stack),队列(queue),集合(set),映射表(map),根据数据在容器中的排列特性，这些数据分为序列式容器和关联式容器两种。

序列式容器强调值的排序，序列式容器中的每个元素均有固定的位置，除非用删除或插入的操作改变这个位置。

Vector容器、Deque容器、List容器等。

关联式容器是非线性的树结构，更准确的说是二叉树结构。各元素之间没有严格的物理上的顺序关系，也就是说元素在容器中并没有保存元素置入容器时的逻辑顺序。关联式容器另一个显著特点是：在值中选择一个值作为关键字key,这个关键字对值起到索引的作用，方便查找。Set/multiset容器Map/multimap容器。

容器可以嵌套容器！

### 算法

算法，问题之解法也。

以有限的步骤，解决逻辑或数学上的问题，这一门学科我们叫做算法(Algorithms),

广义而言，我们所编写的每个程序都是一个算法，其中的每个函数也都是一个算法，毕竞它们都是用来解决或大或小的逻辑问题或数学问题。

STL收录的算法经过了数学上的效能分析与证明，是极具复用价值的，包括常用的排序，查找等等。特定的算法往往搭配特定的数据结构，算法与数据结构相辅相成。

算法分为：质变算法和非质变算法。

质变算法：是指运算过程中会更改区间内的元素的内容。例如拷贝，替换，删除等等

非质变算法：是指运算过程中不会更改区间内的元素内容，例如查找、计数、遍历、寻找极值等等

### 迭代器

迭代器(iterator)是一种抽象的设计概念，现实程序语言中并没有直接对应于这个概念的实物。在<>一书中提供了23中设计模式的完整描述，其中iterator模式定义如下：**提供一种方法，使之能够依序寻访某个容器所含的各个元素，而又无需暴露该容器的内部表示方式。**

迭代器的设计思维是STL的关键所在，STL的中心思想在于将容器(container)和算法(algorithms)分开，彼此独立设计，最后再一贴胶着剂将他们撮合在一起。从技术角度来看，容器和算法的泛型化并不困难，c++的class template和function template可分别达到目标，如何设计出两个之间的良好的胶着剂，才是大难题。
迭代器的种类：

- **输入迭代器**    提供对数据的只读访问    只读，支持`++`、`=+`、`!=`
- **输出迭代器**    提供对数据的只写访问    只写，支持`++`
- **前向迭代器**    提供读写操作，并能向前推进迭代器    读写，支持`++`、`==`、`!=`
- **双向迭代器**    提供读写操作，并能向前和向后操作    读写，支持`++`、`--`
- **随机访问迭代器**    提供读写操作，并能以跳跃的方式访问容器的任意数据，是功能最强的迭代器    读写，支持`++`、`--`、`[n]`、`-n`、`<`、`<=`、`>`、`>=`

练习1：`v.begin()和v.end()`

```c++
#include <iostream>
#include <vector>
using namespace std;

void test01()
{
    vector<int> v;//本质是一个动态数组
    v.push_back(2);//尾插元素
    v.push_back(3);
    v.push_back(5);
    v.push_back(7);
    //若要访问容器内的容器 需要拿到该容器的迭代器(指针)
    vector<int>::iterator it_start = v.begin();//获取容器的起始迭代器，它指向容器的第一元素。
    vector<int<::iterator it_end = v.end();////获取容器的末尾迭代器，它指向容器的最后一个元素的下一个位置。
    for(;it_start!=it_end; ++it_start)
        cout<<*it_start<<" ";
    cout<<endl;
}
```

练习2：迭代器通过`for_each`遍历

```c++
#include <iostream>
#include <vector>
#include <alogthms>
using namespace std;
void print(int a)
{
    cout<<a<<endl;
}
void test01()
{
    vector<int> v;//本质是一个动态数组
    v.push_back(2);//尾插元素
    v.push_back(3);
    v.push_back(5);
    v.push_back(7);
    //若要访问容器内的容器 需要拿到该容器的迭代器(指针)
    vector<int>::iterator it_start = v.begin();//获取容器的起始迭代器，它指向容器的第一元素。
    vector<int<::iterator it_end = v.end();////获取容器的末尾迭代器，它指向容器的最后一个元素的下一个位置。
    for_each(it_start, it_end, print);
}
```

练习3：容器中存放的是自定类型，如何使用迭代器访问元素

```c++
class person
{
public:
    person(int age)
    {
        this->age = age;
    }
    int age;
}
void test01()
{
    vector<person> v;
    person p1(1);
    person p2(2);
    person p3(3);
    person p4(4);
    v.push_back(p1);
    v.push_back(p2);
    v.push_back(p3);
    v.push_back(p4);
    vector<person>::iterator start = v.start();
    vector<person>::iterator end = v.end();
    for(;start!=end; ++start)
        cout<< (*start).age<<" ";
    cout<<endl;
    
}
```



练习4: 容器嵌套容器

```c++
void test01()
{
    vector<int> child1, child2,child3,child4;
    vector<vector<int>> parent;
    parent.push_back(child1);
    parent.push_back(child2);
    parent.push_back(child3);
    parent.push_back(child4);
    child1.push_back(1);
    child2.push_back(1);
    child3.push_back(1);
    child4.push_back(1);
    
    vector<vector<int>>::iterator p_start = parent.start();//获取的是父容器的首元素的迭代器
    vector<vector<int>>::iterator p_end = parent.end();//获取的是父容器的末尾元素下一个位置的迭代器
    for(;p_start!=p_end; ++p_start)
    {
        vector<int>::iterator c_start = (*p_start).begin();//获取的是子容器的首元素的迭代器
        vector<int>::iterator c_end = (*p_start).end();//获取的是子容器的首元素的迭代器
        cout<<"在第"<<p_start<<"个子容器中的元素:"<<endl;
        for(;c_start!=c_end; ++c_start)
        {
            cout<<*c_start<<" ";
        }
        cout<<endl;
        
    }
    
}
```

## 常用容器

### string容器

#### string容器基本概念

c风格字符串（以空字符结尾的字符数组）太过复杂难于掌握，不适合大程序的开发，所以c+标准库定义了一种string类，定义在头文件`#include<string>`。

**String和c风格字符串对比**：

- Char*是一个指针，String是一个类
  string封装了char,管理这个字符串，是一个char型的容器。*
- String封装了很多实用的成员方法
  查找find,拷贝copy,删除delete替换replace,插入insert
- 不用考虑内存释放和越界
  string管理char*所分配的内存。每一次string的复制，取值都由string类负责维护，不用担心复制越界和取值越界等。算法

#### string容器常用操作

#####  string构造函数

```c++
string();//升创建一个空的字符串例如：string str.
string(const stringe&str);//使用一个string对象初始化另一个string对象
string(const char*s);//使用字符串s初始化
string(int n,char c);//使用n个字符c初始化
```

#####  string基本赋值操作

```c++
string& perator=(const chari* s);//char*类型字符串赋值给当前的字符串
string& operator=(const string& s);//把字符串s赋给当前的字符串
string& operator=(char c);//字符赋值给当前的字符串
string& assign(const char* s);//把字符串s赋给当前的字符串
string& assign(const char* s,int n);//把字符串s的前n个字符赋给当前的字符串
string& assign(const string& s);//把字符串s赋给当前字符串
string& assign(int n,char c);//用n个字符c赋给当前字符串
string& assign(const string& s,int start,,int n);//将s从start开始n个字符值给字符串
```

##### string存器字符串操作

```c++
char& operator[](int n);//通过[]方式获取字符
char& at(int n);//通过at方法获取字符串
```

##### string拼接操作

```c++
string& operator+=(const string& str);//重找+=操作符
string& operator+=(const char* str);//重找+=操作符
string& operator+=(const char c);//重找+=操作符
string& append(const char* s);//把字符串s连接到当前字符串结尾
string& append(const char* s,int n);//把字符串s的前n个字符连接到当前字符串结尾
string& append(const string &s);//同operator+=()
string& append(const string &s,int pos,int n);//把字符串s中从pos开始的n个字符连接到当前字符串结尾
string& append(int n,char c);//在当前字符串结尾添加n个字符c
```

##### string查找和替换

```c++
int find(const string &str, int pos=0)const;//查找str第一次出现位置，从pos开始查找
int find(const char* s,int pos=0)const;//查找s第一次出现位置，从pos开始查找
int find(const char* s,int pos,int n)const;//从pos位置查找s的前n个字符第一次位置
int find(const char c,int pos=0)const,;//查找字符c第一次出现位置
int rfind(const string& str,int pos=npos)const;//查找str最后一次位置，从pos开始查找
int rfind(const char* s,int pos=npos)const;//查找s最后一次出现位置，从pos开始查找
int rfind(const char* s,int pos,int n)const;//从pos查找s的前n个字符最后一次位置
int rfind(const char c,int pos=e)const;//查找字符c最后一次出现位置
string& replace(int pos,int n,const string& str);//替换从pos开始n个字符为字符串str
stringe& repLace(int pos,int n,const char*s);//替换从pos开始的n个字符为字符串s


```

#####  string比较操作

```c++
/*
compare函数在时返回1，<时返回-1，==时返回0。
比较区分大小写，比较时参考字典顺序，排越前面的越小。
大写的A比小写的a小。
*/
int compare(const string& s)const;//与字符串s比较
int compare(const char* s)const;//与字符串sl比较
```

##### string子串

```c++
string substr(int pos=0,intn=npos)const;/返回由pos开始的n个字符组成的字符串
```

#####  string插入和删除操作

```c++
string&insert(int pos,const char*s);//插入字符串
string&insert(int pos,const string &str);//插入字符串
string&insert(int pos,int n,char c);//在指定位置插入n个字符c
stringe&erase(int pos,int n=npos);//别除从Pos开始的n个字符
```

##### string和c-style字符串转换

```c++
//string转char*
string str = "itcast";
const char*cstr str.c_str();
//char*转string
char*s =  "itcast";
string str(s);
```

在c++中存在一个从const char到string的隐式类型转换，却不存在从一个string对象到C_string的自动类型转换。对于string类型的字符串，可以通过c_str()函数返回string对象对应的C_string。

通常，程序员在整个程序中应坚持使用string类对象，直到必须将内容转化为char时才将其转换为C_string。

提示：

为了修改string字符串的内容，下标操作符0和at都会返回字符的引用。但当字符串的内存被重新分配之后，可能发生错误。

### vector容器

#### vector容器基本概念

vector的数据安排以及操作方式，与array非常相似，两者的唯一差别在于空间的运用的灵活性。Array是静态空间一旦配置了就不能改变，要换大一点或者小一点的空间，可以，一切琐碎得由自己来，首先配置一块新的空间，然后将旧空间的数据搬往新空间，再释放原来的空间。Vector是动态空间，随着元素的加入，它的内部机制会自动扩充空间以容纳新元素。因此vector的运用对于内存的合理利用与运用的灵活性有很大的帮助，我们再也不必害怕空间不足而一开始就要求一个大块头的array了。

Vector的实现技术，关键在于其对大小的控制以及重新配置时的数据移动效率，一旦vector旧空间满了，如果客户每新增一个元素，vector内部只是扩充一个元素的空间，实为不智，因为所谓的扩充空间（不论多大），一如刚才所说，是”配置新空间:arrow_right:数据移动:arrow_right:释放旧空间”的大工程，时间成本很高，应该加入某种未雨绸缪的考虑，稍后我们便可以看到vector的空间配置策略。

![image-20220421083301676](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220421083301676.png)

#### vector迭代器

Vector维护一个线性空间，所以不论元素的型别如何，普通指针都可以作为vector的迭代器，因为vector迭代器所需要的操作行为，如`operator.` ,`operator->`, `operator++`, `operator--`, `operator+`, `operator-`, `operator+=`与`operator-=`, 普通指针天生具备。Vector支持随机存取，而普通指针正有着这样的能力。所以vector提供的是随机访问送代器RandomAccess Iterators).

#### vector的数据结构

Vector所采用的数据结构非常简单，线性连续空间，它以两个迭代器Myfirst和Mylast分别指向配置得来的连续空间中目前已被使用的范围，并以迭代器_Myend指向整块连续内存空间的尾端。

为了降低空间配置时的速度成本，vector实际配置的大小可能比客户端需求大一些，以奋将来可能的扩充，这边是容量的概念。换句话说，一个vector的容量永远大于或等于其大小，一旦容量等于大小，便是满载，下次再有新增元素，整个vector容器就得另觅居所。

注意：

所谓动态增加大小，并不是在原空间之后续接新空间（因为无法保证原空间之后尚有可配置的空间），而是一块更大的内存空间，然后将原数据拷贝新空间，并释放原空间。因此，**对vector的任何操作，一旦引起空间的重新配置，指向原vector的所有迭代器就都失效了。**这是程序员容易犯的一个错误，务必小心。

#### vector常用API操作

#####  vector构造函数

```c++
vector<T> v;//采用模板实现类实现，默认构造函数
vector(v.begin(),v.end());//将v[begin(),end()]区间中的元素拷贝给本身。
vector(n,eLem);//构造函数将n个eLem拷贝给本身。
vector(const vector &vec);//拷贝构造函数
```

##### vector常用赋值操作

```c++
assign(begin,end);//将[beg,end)区间中的数据拷贝赋值给本身。
assign(n,elem);//将n个elem拷贝赋值给本身。
vectore& operator=(const vector&vec);//重载等号操作符
swap(vec);//将vec与本身的元素互换,只是交换了指针的指向
```

#####  vector大小操作

```c++
size();//返回容器中元素的个数
empty();//判断容器是否为空
resize(int num);//重新指定容器的长度为num,若容器变长，则以默认值填充新位置。如果容器变短，则末尾超出容器长度的元素被删除。
resize(int num,eLem);//重新指定容器的长度为num,若容器变长，则以eLem值填充新位置。如果容器变短，则末尾超出容器长度的元素被删除。
capacity();//容器的容量
reserve(int len);//容器预留len个元素长度，预留位置不初始化，元素不可访问。
```

##### vector数据存取操作

```c++
at(int idx);//返回索引idx所指的数据，如果idx越界，抛出out_of_range异常
operator[];//返回索引idx所指的数据，越界时，运行直接报错
front();//返回容器中第一个数据元素
back();//返回容器中最后一个数据元素
```

##### vector插入和删除操作

```c++
insert(const_iterator pos,int count, ele);//送代器指向位置pos插入count个元素ele.
push_back(eLe);//尾部插入元素eLe
pop_back();//别除最后一个元素
erase(const_iterator start,.const_iterator end);//擦除迭代器从start到end之间的元素
erase(const_.iterator pos);//擦除迭代器指向的元素
clear();//清除容器中所有元素
```

##### 案例1：巧用swap函数压缩内存空间

```c++
void test()
{
    vector<int> v1;
    for(int i = 0; i < 10000; i++)
        v1.push_back(i);
    cout<<v1.size()<<" "<<v1.capacity()<<endl;//10000 10136
    v1.resize(10);
    cout<<v1.size()<<" "<<v1.capacity()<<endl;//10 10136
    vector<int>(v1).swap(v1);//与匿名对象交换空间，匿名对象在创建的行就会被销毁，交换到匿名对象的空间会被释放 
    cout<<v1.size()<<" "<<v1.capacity()<<endl;//10 10
}
```

##### 案例2：计算重新开辟内存空间的次数

```c++
void test()
{
    vector<int> v;
    int *p = NULL;
    int count = 0;
    for(int i = 0; i < 10000; ++i)
    {
        v.push_back(i);
        if(p!=&v[0])
        {
            cout++;
            p = &v[0];
        }
    }
}
```

##### 案例3：vector容器排序

```c++
#include <alorgthms>
#include <vector>

int compare(int a, int b)
{
    return a<b;
}

void print(int a)
{
    cout<<a<<endl;
}

void test()
{
    vector<int> v;
    v.push_back(8);
    v.push_back(49);
    v.push_back(1);
    v.push_back(2);
    v.push_back(5);
    v.push_back(4);
    v.push_back(4);
    sort(v.begin(), v.end());//默认升序排列
    sort(v.begin(), v.end(), compare);//根据函数compare的功能来排序
    for_each(v.begin(), v.end(), print);
}
```



## 算法

### 函数对象(仿函数)

重载函数调用操作符的类，其对象常称为函数对象(function object),即它们是行为类似函数的对象，也叫仿函数(functor)),其实就是重载"()”操作符，使得类对象可以像函数那样调用。

注意：

1. 函数对象（仿函数）是一个类，不是一个函数。
2. 函数对象（仿函数）重载了"()”操作符使得它可以像函数一样调用。

分类：

假定某个类有一个重载的operator(),而且重载的operator(）要求获取一个参数，我们就将这个类称为”一元仿函数”(unary functor)；相反，如果重载的operator()要求获取两个参数，就将这个类称为“二元仿函数”(binary
functor)

函数对象的作用主要是什么？STL提供的算法往往都有两个版本，其中一个版本表现出最常用的某种运算，另一版本则允许用户通过template参数的形式来指定所要采取的策略。

通过对象函数来指定算法所采取的策略

```c++
#include <vector>
#include <alogthms>
#include <string>
class Print
{
public:
    void operator()(int a){cout<<a<<endl;}
};
class Compare
{
public:
    int operator()(int a, int b){return a>b;}
}
void test()
{
    vector<int> v;
    v.push_back(5);
    v.push_back(6);
    v.push_back(5);
    v.push_back(4);
    v.push_back(1);
    for_each(v.begin(), v.end(), Print());//Print()为匿名对象
    sort(v.begin(), v.end(), Compare());//Compare()为匿名对象--->Compare().operator()(int,int);
    for_each(v.begin(), v.end(), Print());//Print()为匿名对象 --->Print().operator()(int);
}
```



### 谓词

谓词是指普通函数或重载的operator()**返回值是bool类型**的**函数对象**（仿函数）。如果operator接受一个参数，那么叫做一元谓词，如果接受两个参数，那么叫做二元谓词，谓词可作为一个判断式。

### 内置的函数对象（仿函数）





STL内建了一些函数对象。分为：算数类函数对象，关系运算类函数对象，逻辑运算类仿函数。这些仿函数所产生的对象，用法和一般函数完全相同，当然我们还可以产生无名的临时对象来履行函数功能。使用内建函数对象，需要引入头文件`#include<fictional>`

6个算数类函数对象，除了negate是一元运算，其他都是二元运算。

```c++
tempLate<cLass T>T plus<T>//加法仿函数
template<cLass T>T minus<T>//减法仿函数
tempLate<cLass T>T multiplies<T>//乘法仿函数
tempLate.<cLass T>T divides<T)//除法仿函数
tempLate<cLass T>T moduLus<T>//取模仿函数
tempLate<cLass T>T negate<T>//取反仿函数
```

6个关系运算类函数对象，每一种都是二元运算。

```c++
tempLate<cLass T>bool equal_to<T>//等于
template.<cLass T>bool not_equal_.to<T>//不等于
tempLate<cLass T>bool greater<T>//
template<cLass T)bool greater_equal<T>//大于等于
tempLate<cLass T>bool less<T>//小于
template<cLass T>bool less_equal<T>//小于等于
```

逻辑运算类运算函数，not为一元运算，其余为二元运算。

```c++
tempLate<cLass T>bool logical_and<T>//逻辑与
tempLate<cLass T>bool logical or<T>//逻辑或
tempLate<cLass T)bool logical_not<T>//逻辑非
```



### 函数对象的适配器

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
using namespace std;

class Adapter1st: public binary_function<int, int,void>
{
public:
	void operator()(int i, int val) const
	{
		cout << i << "+" << val << "=" << i + val << endl;
	}

};

class Compare
{
public:
	bool operator()(int a, int b)
	{
		return a > b;
	}
};
struct print
{
	void operator()(int a) { cout << a << endl; }
};

void test()
{
	vector<int> v;
	v.push_back(1);
	v.push_back(3);
	v.push_back(6);
	v.push_back(11);
	v.push_back(5);
	for_each(v.begin(), v.end(), bind2nd(Adapter1st(), 2));
	sort(v.begin(), v.end(), Compare());
	for_each(v.begin(), v.end(), print());
	for_each(v.begin(), v.end(), bind2nd(Adapter1st(), 2));

}

int main()
{
	test();
}

```





## 智能指针

### 什么是智能指针？

智能指针是存储指向动态分配（位于堆）对象指针的类，用于生存期控制，能确保在离开指针所在作用域时，自动正确地销毁动态分配的对象，以防止内存泄漏。

智能指针通常通过引用计数技术实现：每使用一次，内部引用计数+1；每析构一次，内部引用计数-1，当减为0时，删除所指堆内存。

C++11提供3种智能指针：std::shared_ptr, std::unique_ptr, std::weak_ptr

头文件：

下面围绕这这种智能指针进行探讨。

### shared_ptr

shared_ptr是共享的智能指针，使用引用计数，允许多个shared_ptr指针指向同一个对象。只有在最后一个shared_ptr析构时，引用计数为0，内存才会被释放。

#### shared_ptr基本用法

**1. 初始化**
可以通过 构造函数、make_shared辅助函数、reset方法 来初始化shared_ptr，如：

```cpp
// 智能指针的初始化方式
shared_ptr<int> p(new int(1)); // 传参构造
shared_ptr<int> p2 = p; // copy构造
shared_ptr<int> p3; // 创建空的shared_ptr，不指向任何内存
p3.reset(new int(2)); // reset方法 替换p3管理对象指针为传入指针参数
auto p4 = make_shared<int>(3); // 利用make_shared辅助函数，创建shared_ptr

if (p3) 
	cout << "p3 is not null" << endl;

else 
	cout << "p3 is null" << endl;
```

应该优先使用make_shared创建智能指针，因为更高效。
**TIPS：**
1）如果智能指针中引用计数 > 0，reset将导致引用计数-1；
2）除了通过引用计数，还可以通过智能指针的operator bool类型操作符，来判断指针所指内容是否为空（未初始化）；

**错误做法：将一个原始指针直接赋值给一个智能指针。**

```cpp
// 错误的智能指针创建方法
shared_ptr<int> p = new int(1); // 编译错误，不允许直将原始指针赋值给智能指针
```

**2. 获取原始指针**

通过shared_ptr的get方法来获取原始指针。如：

```cpp
shared_ptr<int> p(new int(1));
int* rawp = p.get();
cout << *rawp << endl; // 打印1
```

注意：get获取原始指针并不会引起引用计数变化。

**3. 指定删除器**

智能指针的默认删除器是operator delete，初始化的时候可以指定自定义删除器。如：

```cpp
// 自定义删除器DeleteIntPtr
void DeleteIntPtr(int* p) {
	delete p;
}
shared_ptr<int> p(new int, DeleteIntPtr); // 为shared_ptr指定自定义删除器
```

删除器何时调用？

当p的引用计数为0时，自动调用删除器DeleteIntPtr释放对象的内存。删除器可以是函数，也可以是lambda表达式，甚至任意可调用对象。

如：

```cpp
// 为shared_ptr指定删除器示例
shared_ptr<int> p(new int, DeleteIntPtr); // 为指向int的shared_ptr指定删除器，删除器是自定义函数
shared_ptr<int> p1(new int, [](int* p) { delete p; });// 删除器是lambda表达式
	
function<void(int*)> f = DeleteIntPtr;
shared_ptr<int> p2(new int, f); // 删除器是函数对象

shared_ptr<int> p3(new int[10], [](int* p) { delete[] p; });   // 为指向数组的shared_ptr指定删除器
	
shared_ptr<int> p4(new int[10], std::default_delete<int[]>()); // 删除器是default_delete	
```

shared_ptr默认删除器是删除delete T对象的，并不是针对数组。如果要删除数组，就需要为数组指定delete[]删除器。或者，可以通过封装一个make_shared_array方法来让shared_ptr支持数组：

```cpp
template<typename T>
shared_ptr<T> make_shared_array(size_t size) {
	return shared_ptr<T>(new T[size], default_delete<T[]>());
}
// 使用make_shared_array，创建指向数组的shared_ptr
shared_ptr<int> p = make_shared_array<int>(10);
shared_ptr<char> p1 = make_shared_array<char>(10);
```

#### 使用shared_ptr的陷阱

1. **不要将原始指针赋值给shared_ptr**

```cpp
shared_ptr<int> p = new int; // 编译错误
```

2. **不要将一个原始指针初始化多个shared_ptr**

```cpp
int* rawp = new int;
shared_ptr<int> p1(rawp);
shared_ptr<int> p2(rawp); // 逻辑错误，可能导致程序崩溃
```

3. **不要在函数实参中创建shared_ptr**

```cpp
void func(shared_ptr<int> p, int a);
int g();

func(shared_ptr<int>(new int), g()); // 有缺陷
```

由于C++的函数参数计算顺序在不同的编译器（不同的默认调用惯例）下，可能不一样，一般从右到左，也可能从左到右，因而可能的过程是先new int，然后调用g()。如果恰好g()发送异常，而shared_ptr 尚未创建，那么int内存就泄漏了。

**正确写法是先创建智能指针，然后调用函数**：

```cpp
// 函数参数是shared_ptr时，正确写法
shared_ptr<int> p(new int());
f(p, g());
```

4. **通过shared_from_this()返回this指针。**不要将this指针作为shared_ptr返回出来，因为this本质是一个裸指针。因此，直接传this指针可能导致重复析构。

​		例如，

```cpp
// 将this作为shared_ptr返回，从而导致重复析构的错误示例
struct A {
	shared_ptr<A> GetSelf() {
		return shared_ptr<A>(this); // 不要这样做，可能导致重复析构
	}
	~A() {
		cout << "~A()" << endl;
	}
};

shared_ptr<A> p1(new A);
shared_ptr<A> p2 = p1->GetSelf(); // A的对象将被析构2次，从而导致程序崩溃
```

本例中，用同一个指针（this）构造了2个智能指针p1, p2（两者无任何关联），离开作用域后，this会被构造的2个智能指针各自析构1次，从而导致重复析构的错误。

正确返回this的shared_ptr做法：让目标类通过派生std::enable_shared_from_this类，然后使用base class的成员函数shared_from_this来返回this的shared_ptr。

```cpp
struct A : public enable_shared_from_this<A> {
	shared_ptr<A> GetSelf() {
		return shared_from_this();
	}
	~A() {
		cout << "~A()" << endl;
	}
};

shared_ptr<A> p1(new A);
shared_ptr<A> p2 = p1->GetSelf(); // OK
cout << p2.use_count() << endl;   // 打印2，注意这里会引起指向A的raw pointer对应的shared_ptr的引用计数+1
```

5. **避免循环引用。循环引用会导致内存泄漏。**

   一个典型的循环引用case：

```cpp
struct A;
struct B;

struct A {
	std::shared_ptr<B> bptr;
	~A() { cout << "A is delete!" << endl; }
};
struct B {
	std::shared_ptr<A> aptr;
	~B() { cout << "B is delete!" << endl; }
};

void test() {
	shared_ptr<A> ap(new A);
	shared_ptr<B> bp(new B);
	ap->bptr = bp;
	bp->aptr = ap;
	// A和B对象应该都被删除，然而实际情况是都不会被删除：没有调用析构函数
}
```

解决循环引用的有效方法是使用weak_ptr。例子中，可以将A和B中任意一个成员变量，由shared_ptr修改为weak_ptr。

### unique_ptr

#### unique_ptr基本用法

独占型智能指针，不允许与其他智能指针共享内部指针，不允许将一个unique_ptr赋值给另外一个unique_ptr。

错误用法：

```cpp
// 将一个unique_ptr赋值给另外一个unique_ptr是错误的
unique_ptr<int> p(new int);
unique_ptr<int> p1 = p;            // 错误，unique_ptr不允许复制
```

正确用法：可以移动（std::move）。移动后，原来的unique_ptr不再拥有原来指针的所有权了，所有权移动给了新unique_ptr。

```cpp
unique_ptr<int> p(new int);
unique_ptr<int> p1 = p;            // 错误，unique_ptr不允许复制
unique_ptr<int> p2 = std::move(p); // OK
```

**自定义make_unique创建unique_ptr**
shared_ptr有辅助方法make_shared可以创建智能指针，但C++11中没有类似的make_unique（C++14才提供）。
自定义make_unique方法（需要在C++11环境下运行）：

```cpp
// 支持普通指针
template<class T, class... Args> inline
typename enable_if<!is_array<T>::value, unique_ptr<T>>::type
make_unique(Args&&... args) {
	return unique_ptr<T>(new T(std::forward<Args>(args)...));
}

// 支持动态数组
template<class T> inline
typename enable_if<is_array<T>::value && extent<T>::value==0, unique_ptr<T>>::type
make_unique(size_t size) {
	typedef typename remove_extent<T>::type U;
	return unique_ptr<T>(new U[size]());
}

// 过滤掉定长数组的情况
template<class T, class... Args>
typename enable_if<extent<T>::value != 0, void>::type make_unique(Args&&...) = delete;

// 使用自定义make_unique创建unique_ptr
unique_ptr<int> p = make_unique<int>(10);
cout << *p << endl;
```

#### unique_ptr与shared_ptr的区别

如果希望同一时刻，只有一个智能指针管理资源，就用unique_ptr；如果希望多个智能指针管理同一个资源，就用shared_ptr。
除了独占性外，unique_ptr与shared_ptr的区别：
1）unique_ptr可以指向一个数组，shared_ptr不能

```cpp
unique_ptr<int []> ptr(new int[10]); // OK：智能指针ptr指向元素个数为10的int数组
ptr[9] = 9; // 设置最后一个元素为9

shared_ptr<int []> ptr(new int[10]); // 错误：C++11中，shared_ptr不能通过让模板参数为数组，从而让智能指针直接指向数组，因为默认删除器是delete，而数组需要delete[]（C++17中已经可用支持）
```

2）指定删除器时，不能像shared_ptr那样直接传入lambda表达式

```cpp
shared_ptr<int> p(new int(1), [](int* p) { delete p; });  // OK
unique_ptr<int> p2(new int(1), [](int* p) { delete p; }); // 错误：为unique_ptr指定删除器时，需要确定删除器的类型
```

因为为unique_ptr指定删除器时，不能像shared_ptr那样，需要确定删除器的类型。像这样：

```cpp
unique_ptr<int, void(*)(int *)> p2(new int(1), [](int* p) { delete p; }); // OK
```

如果lambda表达式捕获了变量，这种写法就是错误的：

```cpp
unique_ptr<int, void(*)(int *)> p2(new int(1), [&](int* p) { delete p; }); // 错误：lambda无法转换为函数指针
```

因为lambda没有捕获变量时，可以直接转换为函数指针，而不会变量后，无法转换。
如果希望unique_ptr删除器支持已不会变量的lambda，可以将模板实参由函数类型修改为std::function类型（可调用对象）：

```cpp
unique_ptr<int, function<void(int *)>> p2(new int(1), [&](int* p) { delete p; });
```

#### 自定义unique_ptr删除器

```cpp
#include <memory>
#include <iostream>
#include <functional>
using namespace std;
struct MyDeleter{
	void operator()(int* p) {
		cout << "delete" << endl;
		delete p;
	}
};

unique_ptr<int, MyDeleter> p(new int(1));
cout << *p << endl;
```

### weak_ptr

弱引用智能指针weak_ptr用来监视shared_ptr，不会引起引用计数变化，也不管理shared_ptr内部指针，主要为了监视shared_ptr生命周期。weak_ptr没有重载操作符*和->，因为不共享指针，不能操作资源。

weak_ptr主要作用：
1）监视shared_ptr管理的资源是否存在；
2）用来返回this指针；
3）解决循环引用问题；

#### weak_ptr基本用法

1）通过use_count() 获得当前观测资源的引用计数。

```cpp
shared_ptr<int> sp(new int(10));
weak_ptr<int> wp(sp);
cout << wp.use_count() << endl; // 打印1
```

2）通过expired() 判断所观测的资源释放已经被释放。

```cpp
{
	shared_ptr<int> sp(new int(10)); // sp所指向内容非空
	weak_ptr<int> wp(sp);
	if (wp.expired()) {
		cout << "weak_ptr 无效，所监视的智能指针已经被释放" << endl;
	}
	else
		cout << "weak_ptr 有效" << endl; // 输出 "weak_ptr 有效"
}
{
	shared_ptr<int> sp; // sp所指向的内容为空
	weak_ptr<int> wp(sp);
	if (wp.expired()) {
		cout << "weak_ptr 无效，所监视的智能指针已经被释放" << endl; // 输出 "weak_ptr 无效..."
	}
	else
		cout << "weak_ptr 有效" << endl;
}
```

3）通过lock() 获取所监视的shared_ptr。

```cpp
weak_ptr<int> wp;
void f() {
	shared_ptr<int> sp(new int(10));
	wp = sp;

	if (wp.expired()) {
		cout << "weak_ptr 无效，所监视的智能指针已经被释放" << endl;
	}
	else {
		cout << "weak_ptr 有效" << endl; // 打印 "weak_ptr 有效"
		auto spt = wp.lock();
		cout << *spt << endl; // 打印10
	}
}
```

#### weak_ptr返回this指针

上文提到不能直接将this指针返回为shared_ptr，需要通过继承enable_shared_from_this类，然后通过继承的shared_from_this()访问来返回智能指针。这是因为enable_shared_from_this类中有个weak_ptr，用于监测this智能指针，调用shared_from_this()方法时，会调用内部weak_ptr的lock()方法，将监测的shared_ptr返回。

之前提到的那个例子：

```cpp
struct A : public std::enable_shared_from_this<A> {
	std::shared_ptr<A> GetSelf() {
		return shared_from_this();
	}
	~A() {
		cout << "A is deleted" << endl;
	}
};

shared_ptr<A> spy(new A);
shared_ptr<A> p = spy->GetSelf(); // OK. 如果A不用继承自enable_shared_from_this类的方法，直接传this指针给shared_ptr，会导致重复析构的问题

// 只会输出一次 "A is deleted"
```

#### weak_ptr解决循环引用问题

前面提到shared_ptr存在的循环引用的经典问题：A类持有指向B对象的shared_ptr，B类持有指向A对象的shared_ptr，导致2个shared_ptr引用计数无法归0，从而导致shared_ptr所指向对象无法正常释放。

```cpp
struct A;
struct B;

struct A {
	shared_ptr<B> bptr;
	~A() { cout << "A is deleted" << endl; }
};
struct B {
	shared_ptr<A> bptr;
	~B() { cout << "B is deleted" << endl; }
};

void test() {
	shared_ptr<A> pa(new A);
	shared_ptr<B> pb(new B);
	pa->bptr = pb;
	pb->aptr = pa;
} // 离开函数作用域后，A、B对象应该销毁，但事实没有被销毁，从而导致内存泄漏
```

用weak_ptr解决循环引用问题，具体方法是将A或B类中，shared_ptr类型修改为weak_ptr。

```cpp
struct A;
struct B;

struct A {
	shared_ptr<B> bptr;
	~A() { cout << "A is deleted" << endl; }
};

struct B {
	weak_ptr<A> aptr; // 将B中指向A对象的智能指针，由shared_ptr修改为weak_ptr
	~B() { cout << "B is deleted" << endl; }
};

void test() {
	shared_ptr<A> pa(new A);
	shared_ptr<B> pb(new B);
	pa->bptr = pb;
	pb->aptr = pa;
} // OK
```

### 通过智能指针管理第三方库分配的内存

第三方库提供的接口，通常是用的原始指针，而非智能指针。那么，我们在用完第三方库之后，如何通过智能指针管理第三方库分配的内存呢？

我们先看第三方库一般的用法：

```cpp
// GetHandler()获取第三方库句柄
// Create, Release是第三方库提供的资源创建、释放接口
void* p = GetHandler()->Create();
// do something...
GetHandler()->Release();
```

实际上，上面这段代码是不安全的，原因在于：1）使用第三方库分配时，可能忘记调用Release接口；2）发生了异常，或者提前返回，实际并没有调用Release接口。从而导致资源无法正常释放。

使用智能指针管理第三方库，不要显式调用释放接口，即使发生异常或者忘记调用，也能正常释放资源。
如上面一般用法，可以改写成用智能指针的方式：

```cpp
// OK
void* p = GetHandler()->Create();
shared_ptr<void> sp(p, [this](void* p) { GetHandle()->Release(p); };
```

上面代码可以保证任何时候，都能正确释放第三方库分配的内存。虽然能解决问题，但还很繁琐，因为每个第三方库分配内存的地方，就要调用这段代码。可将这段代码提炼出来作为一个公共函数，以简化调用：

```cpp
// 存在安全隐患
// 将创建智能指针，用于管理第三方库的代码段封装到一个函数
shared_ptr<void> Guard(void* p) {
	return shared_ptr<void> sp(p, [this](void* p) { GetHandler()->Release(p); });
}

void* p = GetHandler()->Create();
auto sp = Guard(p);
// do something with sp...
```

上面这段代码存在安全隐患：客户可能并不会利用Guard返回的临时shared_ptr，构造一个新的shared_ptr，这样p所创建的资源会立即释放。

```cpp
// 安全隐患演示
void* p = GetHandler()->Create();
Guard(p); // 该句结束后，p就会被释放
// do something with p...
```

这种调用方式中，Guard(p)是一个右值，语句结束后创建的资源会立即释放，从而导致p提前释放，p成为野指针，而后面继续访问p可能导致程序异常。虽然用`auto sp = Guard(p);`赋值不存在问题，但是客户可能会忘记，也就是说这种写法不够安全。

**如何解决由于忘记赋值导致指针提前释放的问题？**
答：可以用一个宏来解决这个问题，通过宏来强制创建一个临时智能指针。代码如：

```cpp
// OK
#define GUARD(p) std::shared_ptr<void> p##p(p, [](void* p) { GetHandler()->Release(p); })

void* p = GetHandler()->Create();
GUARD(p); // 安全：会在当前作用域下，创建名为pp的shared_ptr<void>
```

当然，如果只希望用独占性的管理第三方库的资源，可以用unique_ptr。

```cpp
// OK
#define GUARD(p) std::unique_ptr<void, void(*)(int*)> p##p(p, [](void* p) { GetHandler()->Release(p); })
```

**小结：**
1）使用宏定义方式的优势：即使忘记对智能指针赋值，也能正常运行，安全又方便。
2）使用GUARD这种智能指针管理第三方库的方式，其本质是智能指针，能在各种场景下正确释放内存。



## Lambda

所谓Lambda是一份功能定义式，可以被定义于语句或表达式内部，因此可以将其当作内敛函数使用

`[]{}`最简单的Lambda表达式







