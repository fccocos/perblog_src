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



















