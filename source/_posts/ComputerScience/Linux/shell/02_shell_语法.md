## shell脚本的定义与执行

1、定义以开头：`#!/bin/bash`

`#!`用来声明脚本由什么shell解释，否则使用默认shell

2、单个`#`号代表注释当前行

3、执行：

`chmod +xtest.sh`  `./test.sh`增加可执行权限后执行
`bash test.sh`直接指定使用`bash`解释`test.sh`

**二种执行脚本的方式不同点：**

- `./`和`bash`执行过程基本一致，后者明确指定`bash解样器`去执行脚本，脚本中`#!`指定
  的解释器不起作用

- 前者首先检测`#!`,使用#指定的shell,如果没有使用默认的shell

- 用`./`和`bash`去执行会在后台启动一个新的shell去执行脚本

- 用`.`去执行脚本不会启动新的shell,直接由当前的shell去解释执行脚本。

## 变量

### 自定义变量

**定义变量:** 变量名=变量值，如：`num=10`

**引用变量**：$变量名，如：`i=$snum` 把变量num的值付给变量

**显示变量:** 使用echo命令可以显示单个变量取值, `echo $num`

**清除变量：** 使用unset命令清除变量的值，`unset varname`

**变量的其它用法：**

- `read string` 从键盘输入一个字符串付给变量string

- `readonly var=100` 定义一个只读变量，只能仕定义时初始化，以后不能改变，不能被清除。

- `export var=300` 使用export说明的变量，会被导出为环境变量，其它shell均可使用

注意：

1. 此时必须使用source2var.sh才可以生效
2. shell脚本中，赋值操作时不能在等号`=`两边加空格
3. shell脚本中的变量是没有数据类型的

### 环境变量

shell在开始执行时就已经定义了一些和系统的工作环境有关的变量，我们在shell中可以直接使用$name引用

定义：

一般在~/.bashrc或/etc/profile文件中（系统自动调用的脚本）使用export设置，允计用户后来更改   `VARNAME=value;`    `export VARNAME`

<font color=red>传统上，所有环境变量均为大写</font>

**显示环境变量**

使用`env`命令可以查看所有的环境变量。

**清除环境变量**

使用`unset`命令清除环境变量

**常见环境变量：**

`HOME`:用于保存注册目录的完全路径名。

`PATH`:用丁保存用冒号分隔的目录路径名，shell将按PATH变量中给山的顺序搜索这些目录，找到的第一个与命令名称一致的可执行文件将被执行。

`PATH=$HOME/bin:/bin:/usr/bin;`  `export PATH`

`HOSTNAME`:主机名

`SHELL`:默认的shell命令解析器

`LOGNAME`:此变量保存登录名

终端中设置的环境变量是临时的，可以在`~/.bashrc`或`/etc/profile`中设置环境变量，当设置完环境变量后需要重启终端或者重开一个终端。

### 预设变量.

`$#`：传给shell脚本参数的数量

`$*`：传给shell脚本参数的内容, `$@`与其相当

`$1.$2.$3....$9`：运行脚本时传递给其的参数，用空格隔开, 如果参数超过9个就用`${}`

`$?`：命令执行后返回的状态。`$?`用于检查上一个命令执行是否正确（在Linux中，命令退出状态为0表示该命令正确执行，任何非0值表示命令出错）。

`$0`：当前执行的进程名。

`$$`：当前进程的进程号。`$$`变量最常见的用途是用作临时文件的名字以保证临时文件不会重复

```shell
#!/bin/bash

# 自定定义变量
echo '*****************'
echo 1. 给只定义变量复制时等号两边不能有括号
echo 2. 用$变量名来引用变量
echo 3. 用\"unset 变量名\"清楚变量值
echo 4. 可以\"用read 变量名\"来读取
echo 5. 用readonly定义的变量在定义的时候必须初始化，且不能被修改和清楚，相当于该变量只读
echo 6. 使用export定义的变量为环境变量，其他的shell都可以使用，相当于该变量变成了一个全局变量，但该变量时一个临时的，当系统重启或shell重启后会被清除。
echo '*****************'

num=10 #自定的普通变量
export envval=中国人 #环境变量
read string #可输入的变量

# 输出自定普通变量
echo 自定普通变量 num=$num
# 清除普通变量的值
unset num
# 打印num现在的值
echo num=$num
# 打印输入变量的值
echo 输入变量的值 string=$string
# 打印环境变量的值
echo 环境变量的值 envval=$envval

echo 当前shell脚本02_自定义变量.sh运行到此结束
```



### 有类型的变量

```shell
declare -i num #将num声明为int
declare -r val #将val声明为readonly,该变量只读，无法unset
declare -x env_val#将env_val标记为环境变量，通过export导出
declare -a #指定为索引数组（普通数组）; 查看普通数组
declare -A #指定为关联数组；查看关联数组
```

### shell数组

- 数组定义: `数组名=(elem1 elem2 elem3 ...)`
- 数组赋值
  - 一次赋一个值  `array[0] = val`
  - 一次赋多个值
    - `array=(var1 var2 var3 ...)`
    - `array=(命令)`，注意命令要用倒引号括起来
    - `array=(any const var)`, 可以赋值任何常量
- 取值方式：`${array[index]}`
  - `${array[i]}`, 取索引为i的数组的元素
  - 用`@`或`*`来获取数组中的所有元素
    - `${array[0]}` 获取数组的第一个元素
    - `${array[*]}` 获取数组中所有元素
    - `${#array[*]}` 获取数组中元素的个数
    - `${!array[@]}` 获取数组元素的索引
    - `${array[@]:1:2}` 访问指定的元素；1表示索引为1的元素开始获取；2表示获取后面的2个元素

### shell的关联数组

- 声明方式：`declare -A 数组名称`
- 关联数组赋值
  - 一次赋一个值 `array[key]=value`
  - 一次赋多个值 `array=([k1]=v1 [k2]=v2 ...)`
- 查看关联数组 `declare -A`
- 取值方式与普通数组类似，只是`index`替换为了`key`





### 其他变量(扩展)

1. 取出一个目录下的目录和文件: `dirname`和`basename`
2. 变量"内容"的替换和删除
   - 一个`%`代表从右向左去掉一个`key`
   - 两个`%%`代表从右向左最大去掉一个`key`
   - 一个`#`代表从左向右去掉一个`key`
   - 两个`#`代表从左向右最大去掉一个`key`

**用例测试**

```shell
#!/bin/bash
url=www.taobao.com
echo ${#url} #获取字符串的长度
echo ${url#*.}#以"."为分割符，删除从左到右的第一个匹配到的字符串，包括"."分割符，显示后面的字符串
echo ${url##*.}#以"."为分隔符，删除从左到右的匹配到的最大字符串，包括"."分割符，显示后面的字符串
echo ${url%.*}#和${url#*.}的匹配方向相反，功能相同
echo ${url%%.*}#和${url##*.}的匹配方向相反，功能相同
```

## shell脚本格式化输出

### echo命令

- 功能：将内容输出到默认显示设备
- 语法: `echo [-ne] [字符串]`
- OPTIONS
  - `-n`  不要自动换行
  - `-e` 若字符串中出现一下字符，则特别处理，而不会将其当成普通字符处理
  - 转义字符
    - `\a`  发出警告声
    - `\b`  删除前一个字符
    - `\t`  插入tab
    - `\n`  换行且光标移至行首
    - `\c`  最后不加上换行符
    - `\f`  换行但光标位置不变
    - `\r`  光标移至行首，但不换行
    - `\v`  与`\f`相同
    - `\\`    插入`\\`字符
    - `\0nnn` 打印`nnn`(八进制数)所代表的ASCII字符
    - `\xNN`   打印`NN`（十六进制）所代表的ASCII字符

### 输出颜色字体

- 格式：`echo -e "\033[字的背景颜色;字体颜色m字符串\033[0m"`
- 注意事项
  - 字的背景颜色和字体颜色之间用英文分号隔开
  - 字体颜色用字母`m`隔开
  - 字符串前后可以没有空格，有空格同样输出空格
  - 以`\033[`开始以`\033[0m`结束
  - 一定要有`-e`选项，否则不会对具有特殊意义的字符进行转义
- 字体颜色范围：`30--37`
  - `30`  黑
  - `31`  红
  - `32`  绿
  - `33`  黄
  - `34`  蓝
  - `35`  紫
  - `36`  天蓝
  - `37`  白
- 字的背景色范围：`40--47`
  - `40`  黑
  - `41`  红
  - `42`  绿
  - `43`  黄
  - `44`  蓝
  - `45`  紫
  - `46`  天蓝
  - `47`  白
- 控制选项说明
  - `\033[0m` 关闭所有属性
  - `\033[1m` 设置高亮度
  - `\033[4m` 下划线
  - `\033[5m` 闪烁
  - `\033[7m` 反显
  - `\033[8m` 消隐
  - `\033[nA` 光标向上移n
  - `\033[nB` 光标向下移n
  - `\033[nC` 光标向右移n
  - `\033[nD` 光标向左移n
  - `\033[y;xH` 设置光标的位置
  - `\033[2j` 清屏
  - `\033[K` 清除从光标到行尾的内容
  - `\033[s` 保存光标位置
  - `\033[u` 回复光标位置
  - `\033[?25l` 隐藏光标
  - `\033[25h` 显示光标

**测试用例：**

> 输出一个水果购物界面
>
> 需求：
>
> （1）echo 输出缩进
>
> （2）颜色控制

## shell脚本用户交互

### read命令

- 功能：默认接受键盘的输入，回车符代表输入结束
- 应用场景：人机交互
- 命令选项
  - `-p` 打印信息
  - `-t` 限定时间
  - `-s` 不回显
  - `-n` 输入字符个数



## shell运算解析

### 赋值运算

- 赋值运算符`=`
- 注意
  - 字符串必须要用引号括起来
  - 复制号两边不能有空格

### 算数运算

#### 运算符与命令

- 四则运算符

  - `+` 加
  - `-` 减
  - `*` 乘
  - `/` 除
  - `%` 取模
  - `**` 求幂

- 运算命令

  - 整型命令

    - `expr`

      `expr 15 % 3`

    - `let`

      `let num=12+3`

    - `$(())`

      `echo $(( 15/3 ))`

    - `bc`

  - 浮点运算

    - `bc`

#### 整型运算

- `expr`命令：只能用来做整数运算，格式比较古板，注意空格
- `let`命令：只能用来做整数运算，且运算元素必须是变量，无法直接对整数做运算
- `$(())`命令：可以用来做数学运算

#### 浮点运算

浮点运算采用过的命令组合方式来实现的`echo "scale=N;表达式"|bc`

```shell
# echo "scale=3;12.1111+0.003"|bc
12.114
```



## 脚本变量的特殊用法

`""`(双引号）:包含的变量会被解释
`''`(单引号）:包含的变量会当做字符申解释

``(数字键1左面的反引号）:反引号中的内容作为系统命令，并执行其内容，可以替
换输出为一个变量

$ echo " today is `date` " "

today is 2012年07月29日星期日12:55:21CST

`\`转义字符：同C语言\n \t \r \a等echo命令需要转义

`()`(命令序列)：由子shell进程来完成，不影响当前shell的变量

`{}`{命令序列}：在当前shell中执行，会影响当前变量

<font color=red>注意：`{}` `[]` `()`的两边用空格隔开</font>

## 调试测试语句

在与shell脚本，经常遇到的回款就是判断字符串是合相等，可能认要检查文件状态或进行数字测试，只有这些测试完成才能做下一步动作

test命令：用于测试字符串、文件状态和数字

test命令有两种格式：

test condition或[ condition ]

使用方括号时，要注意在条件两边加上空格

shell 脚本中的条件测试如下：

文件测试、字符串测试、数字测试、复合测试

测试语句一般与后面讲的条件语句联合使用

### 文件

文件测试：测试文件状态的条件表达式

**1)按照文件类型**

```shell
-e 文件名 # 文件是否存在
-s 文件名 # 是否为非空
-b 文件名 # 块设备文件
-c 文件名 # 字符设备文件
-d 文件名 # 目录文件
-f 文件名 # 普通文件
-L 文件名 # 软链接文件
-S 文件名 # 套接字文件
-p 文件名 # 管道文件
```



**2)按照文件权限**

```shell
-r 文件名 # 可读
-W 文件名 # 可写
-X 文件名 # 可执行
```



**3)两个文件之间的比较**

```shell
文件1 -nt 文件2 # 文件1的修改时间是合比文件2新
文件1 -ot 文件2 # 文件1的修改时间是否比文件2旧
文件1 -ef 文件2 # 两个文件的inode节点号是否一样，用于判断是否是硬
```



```shell
#!/bin/bash

echo 'please input a filename >>>'
read FILE

test -e $FILE
echo "存在？$?"

test -s $FILE
echo "非空？$?"

[ -r $FILE ]
echo 可读？$?

[ -w $FILE ]
echo 可写? $?

[ -x $FILE ]
echo 可执行？$?

test -b $FILE
echo 块文件? $?

test -c $FILE
echo 字符文件? $?

test -S $FILE
echo 套接字文件? $?

test -L $FILE
echo 硬链接文件? $?

test -d $FILE
echo 目录文件? $?

test -p $FILE
echo 管道文件? $?

```

### 字符串

**字符串测试**

`s1 = s2`    测试两个字符串的内容是否完全一样

`s1 != s2`  测试两个宁符串的内容是否有差异

`-z s1`       测试s1字符串的长度是否为0

`-n s1`       测试s1字符串的长度是否不为0.

```shell
 #!/bin/bash
echo "请输入一个字符串str1>>>"
read str1
echo "请输入一个字符串str2>>>"
read str2
[ -z $str1 ]
echo " str1 is null? $?"
[ -n $str1 ]
echo " str1 is not null? $?"
[ -z $str2 ]
echo " str2 is null? $?"
[ -n $str2 ]
echo " str2 is not null? $?"
test $str1 = $str2
echo " str1==str2? $?"
test $str1 != $str2
echo " str1!=str2? $?"
```

**数字**

```shell
a -eq b # 测试a与b是否相等
a -ne b # 测试a与b是否不相等
a -gt b # 测试a是否大于b
a -ge b # 测试a是否大于等于b
a -lt b # 测试a是否小丁b
a -le b # 测试a是合小于等于b
```

| 中文单词 |   英文单词    | shell比较符 |
| :------: | :-----------: | :---------: |
|   相等   |     equal     |     -eq     |
|  不相等  |   not equal   |     -ne     |
| 大于等于 | greater equal |     -ge     |
| 小于等于 |  less equal   |     -le     |
|   大于   |   less than   |     -gt     |
|   小于   | greater than  |     -lt     |

```shell
#!/bin/bash
echo "请输入一个数字num1>>>"
read num1
echo "请输入一个数字num2>>>"
read num2
[ $num2 -eq $num1 ]
echo " num1==num2? $?"
[ $num1 -ne $num2 ]
echo " num1!=num2? $?"
[ $num1 -ge $num2 ]
echo " num1>=num2? $?"
[ $num1 -le $num2 ]
echo "num1<=num2 $?"
test $num1 -gt $num2
echo " num1>num2? $?"
test $num1 -lt $num2
echo " num1<num2? $?"
```

### 复合测试

**命令执行控制：**

```shell
&&:
command1 && command2.
&&左边命令（command1)执行成功（即返回0)shell才执行&&右边的命令(command2)

||:
command1 || command2
||左边的命令（command1)未执行成功（即返回非0)shell才执行||右边的命令(command2)
```

**多重条件判定**

```shell
-a
#(and)两状况同时成立！
test -r file -a -x file
#file同时具有r与x权限时，才为true.

-O
#(or)两状况任何一个成立！。
test -r file -o -x file
#file具有r或x权限时，就传回true.

!
#相反状态
test!-x file
#当file不且有x时同传true
```



```shell
  1 #!/bin/bash
  2
  3 echo "输入一个数字num》》》"
  4 read num
  5
  6 # 判断0<=num<=100
  7
  8 echo "命令执行控制"
  9 test $num -le 100 && test $num -ge 0
 10 echo "0<=num<=100?$?"
 11
 12 echo "多重条件判断"
 13 test $num -le 100 -a $num -ge 0
 14 echo "0<=num<=100?$?"
 15
 16 echo "输入一个文件名filename》》》"
 17 read filename
 18
 19 echo "命令执行控制"
 20 [ -r $filename ] && [ -x $filename ] || [ -f $filename ]
 21 echo "$?"
 22
 23 echo "多重条件判断"
 24 test -r $filename -a -x $filename -o -f $filename
 25 echo "$?"
```

## 控制语句

`if` `case` `for` `while` `until` `break`

### if控制语句

格式一：

```shell
if [ 条件1 ]; then
	#执行第一段程序
else
	#执行第二段程序
fi
```

格式二：

```shell
if [ 条件1 ]; then
	#执行第一段程序
elif [ 条件2 ]; then
    #执行第二段程序
else
	#指向第三段程序
fi
```

### if高级用法

1. 条件符号使用双括号，可以在条件中植入数学表达式`if (())`

   ```shell
   if (( (5+5-5)*5/5 > 10 ));then
   	echo "yes"
   else 
   	echo "no"
   fi
   ```

2. 使用双方括号，可以在条件中使用通配符

   ```shell
   #为字符串提供高级功能，模式匹配r* 匹配r开头的字符串
   for var in ac ac rx bx rvv vt;do
   		if [[ "$var" == r* ]];then
   			echo "$var"
   		fi
   done
   ```

3. 可以用逻辑符`&&` `||` `!` 来替代`if`语句

### case控制语句

```shell
case "$变量" in
	"第一个变量内容")
	#程序段一
	;;
	"第二个变量的内容")
	#程序段二
	;;
	*)
	#其他程序段
	exit 1#如果想继续执行可以不写
	;;
esac
```

### for控制语句

**形式一：**

```shell
for (( 初始值; 限制值; 执行步阶 ))
do  
	# 程序段
done

```

初始值：变量在循环中的初始值

限制值：当变量值在这个限制范围内时，就继续进行循环

执行步阶：每做一次循环，变量的变化量

declare 是bash的一个内建命令，可以用来声明shell变量、设置变量的属性。declare也可以写作typeset。

`declare -i s`代表强制把`s`变量当作`int`型参数运算

```shell
#!/bin/bash

declare -i num
for (( i=0; i<=100; ++i ))
do
	num+=i;
done
echo $num
```

**形式二：**

```shell
for var in con1 con2 con3 ...
do 
	#程序代码
done
```

第一次循环时，$var的内容为con1

第二次循环时，$var的内容为con2

第三次循环时，$var的内容为con3

......

```shell
```

```shell
#!/bin/bash

# example 1
#注意：for循环中的算子不能加$
for i in 1 2 3 4 5 6 7 8 9
do
	echo $i
done
# example 2
for name in `ls`
do
	if [ -f $name ]; then
		echo "$name 是一个普通文件"
	elif [ -r $name ]; then
		echo "$name 是一个可读文件"
	elif [ -p $name ]; then
		echo "$name是一个管道文件"
	else
		echo "(- ^ -)"
	fi
done
```

### while控制语句

```shell
while [ condition ]
do 
	# code fregment
done
```

**代码用例**

```shell
#!/bin/bash
declare -i s
declare -i i
for [ "$i" != "101" ]
do
	s+=i;
	i=i+1;
done
echo "$s"
```

### unitl控制语句

```shell
until [ condition ]
do
	# code fregment
done
```

`until`与`while`相反，只有当`conditon`不成立的时候才执行循环体

**代码用例**

```shell
#!/bin/bash
declare -i i
declare -i s
until [ "$i" = "101" ]
do
	s+=i
	i=i+1
done
echo $s
```

### break continue

break 跳出循环，常用于case语句

continue 跳出本次循环，继续执行下一次循环

## 函数

有些脚本段间互相重复，如果能只写一次代码块而在任何地方都能引用那就提高了代码的可重用性。

shell允许将一组命令集或语句形成一个可用块，这些快称为shell函数。

**定义函数的两种格式：**

**格式一：**

```shell
函数名 () {
	#code fragment
}
```

**格式二**：

```shell
function 函数名 () {
	#code fragment
}
```

函数可以放在同一个文件中作为一段代码，也可以放在只包含函数的单独文件中。

所有函数在使用前必须定义，必须将函数放在脚本开始部分，直至shell解释器首次发现它时，才可以使用.

**调用函数的格式为：**`函数名param1param2......`

**使用参数同在一般脚本中使用特殊变量**：`$1,$2...$9一样`

**函数可以使用return提前结束并带回返回值**：`return从函数中返回，用最后状态命令决定返回值。`  `return0无错误返回`

```shell
myadd()
{
	A=$1
	B=$2
	SUM=`expr $A+$B`
	return $SUM
}
myadd 10 20
#函数的返回值一般可以通过$?可以获取，但是$?可以获取到，但是$?获取到的最大值是255，如果超过这个值，会出错
echo "$?"
myadd 100 200
#echo "$?" # 会报错
#在shell中，除了()中定义的变量，只要不做任何修饰，都可以认为是全局变量，可以在任意位置进行调用
echo "SUM = $SUM"
```

