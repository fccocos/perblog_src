## 原子操作和竞争条件

1. 所有的系统调用都是以==原子操作==[^1]的方式执行的，保证系统调用运行期间不会因为其他的线程或进程所中断。
2. 原子操作可以很好的规避==竞争状态==(race conditions)[^2]

### 以独占方式创建一个文件

当创建文件时，要将O_EXCL作为open的标志之一，如果打开的文件已存在，则open函数返回一个错误。

O_EXCL标志保证了当前进程是打开文件的创建者，此时对于文件是否存在的检查和创建文件就同属于一个原子操作。

试图以独占的方式打开文件的错误代码

```c
int fd=open(argv[0],O_WRONLY);
if(fd!=-1) //打开成功
{
    printf("[PID %ld] File \"%s\" already exists\n",(long)getpid(), argv[0]);
    if(close(fd)==-1)
        errExit("close");
}else
{
    if(errno == ENOENT) //由于意外的理由而失败
    {
        errExit("open");
    }else
    {
        fd = open(argv[1], O_WRONLY|OCREATE,0600);
        if(fd == -1)
            errExit("open");
    	printf("[PID %ld] File \"%s\" already exists\n"(long)getpid(), argv[0]);
    }
}
```

该代码意图打开开一个文件，如果文件不存在就新建一个文件。但该代码中存在一个问题，当第一次调用open时，希望打开的文件还不存在，而当第二次调用的时候其他进程已经创建好了该文件，此时该进程将得出错误的结论：目标文件夹是由自己创建的。出现这样错误的结论的原因是，无论目标文件是否存在与否，该进程对open的第二次调用一定会成功，因为有其他进程创建了该文件。出现这种情况的原因有，当A进程打开文件失败后，cpu的使用时间用完，此时内核调度器将B进程加入到CPU中，此时B进程和A进程一样，先打开文件，如果失败，就创建文件，创建完文件后，CPU的使用时间用完；此时内核又将在等待的A进程唤醒并分配CPU时间片，A进程继续执行，创建文件，文件创建成功，程序结束，如下所示。

![未能以独占的方式创建文件](pic\05_linux系统编程_深入探究文件IO\未能以独占的方式创建文件.svg)



还有一种原因是进程A和进程B并行执行，也会出现上述情况。

### 向文件尾部追加数据

多进程同时向同一文件尾部追加数据，如果使用如下代码：

```c
lseek(fd,0,SEEK_END);
write(fd,buf, len);
```

将会出现和前面所述一样的缺陷，如果第一个进程执行到lseek和write之间，被执行相同代码的第二个进程所中断，那么这两个进程会在写数据前，将文件偏移量设置到相同位置，而当第一个进程再次获得调度时，会覆盖第二个进程已经写入得数据。

要规避这一问题，就需要将文件偏移和数据写入操作纳入同一原子操作。

在打开文件的时候加入O_APPEND标志就可以保证这一点。

注意：有些文件系统（如NFS）不支持O_APPEND标志，这种情况下，内核会按照上述代码的方式，用非原子操作的调用序列，从而导致文件脏写入问题。

## 文件控制操作:fnctl()

```c
#include <fcntl.h>
int fcntl(int fd, int cmd, ...);
```

cmd为参数所支持的操作范围

省略号可以将将其设置为不同的类型，内核会根据cmd参数的值来确定该参数的数据类型

## 文件打开的状态标志

fcntl调用用于对已经打开的文件，获取或修改文件的状态标志。

我们通过cmd参数的F_GETFL来获取打开的文件的状态标志。`flags = fcntl(fd, F_GETFL)`

可以通过获取到的标志与某一状态标志按位与(&),来判断当前打开的文件是否有该标志。`if(flags&O_SYNC)`

可以同cmd参数的F_SETFL来修改已打开文件的某些状态。

允许修改的标志有O_NONBLOCK(非阻塞)、O_APPEND(追加)、O_NOATIME、O_ASYNC(异步)、O_DIRECT(直接读写)

文件不是由调用程序打开的，所以程序无法使用open调用来控制文件的状态标志。这种情况下可以使用fcntl()来修改该文件的状态标志

当文件的描述符是通过open之外的系统调用获取的(dup,dup2,pipe,socket等)。这种情况也可以使用fcntl来修改文件的状态标志



打开的文件的状态标志修改步骤

1. 通过fcntl()的F_GETGL来获取状态标志的副本 `flags = fcntl(fd, F_GETFL)`
2. 将获取的状态标志与要设置的标志按位或 `flags |= O_APPEND`
3. 再次调用fcntl(), 通过F_SETFL来更新状态标志 `fcntl(fd, F_SETFL, flags)`

## 文件描述符和打开文件之间的关系

**进程级的文件描述符表**：用于存放多个文件描述符的相关信息

- 控制文件描述符操作的一组标志
- 文件指针--对打开文件句柄的引用

**系统级的打开文件表**：用存放打开文件的打开文件句柄，打开文件句柄中存放有文件的全部信息

- 文件偏移量
- 打开文件的状态标志
- 文件访问模式
- 信号驱动IO的相关设置
- i-node指针--对文件i-node对象的引用

**文件系统的i-node表:**用于存放文件的i-node信息

- 文件类型和访问权限
- 一个指针，该指针指向该文件所持有的锁的列表
- 文件的各种属性，包括文件的大小以及不同类型操作相关的时间戳

**文件描述符和打开文件之间的关系**

![文件描述符和打开文件之间的关系](F:\PerBlog\source\_posts\ComputerScience\Linux\linux系统编程\pic\05_linux系统编程_深入探究文件IO\文件描述符和打开文件之间的关系.svg)



同一进程中不同描述指向同一个打开文件句柄，可能是通过调用dup()、dup2()或fcntl()造成的

不同进程的同一描述符指向同一个打开文件句柄，造成这种现象的原因可能是：

1. 调用了fork函数，即两个进程为父子关系；
2. 某进程通过套接字将一个打开的文件描述符传递给了另一个进程造成的。

同一进程的不同描述符指向不同的打开文件句柄，但文件句柄指向了同一个i-node。造成这种情况的原因有两个：

1. 各个进程同时打开同一文件
2. 同一个进程多次打开同一文件

总结上述情况，可以得出结论：

1. 两个不同的文件描述符，若指向同一个文件句柄，将共享同一文件偏移量。所以，通过一个文件描述符来修改文件的偏移量，那么从另一个文件描述符可以观察到这一变化。无论两个文件描述符是否属于不同的进程。
2. 要想获得和修改已打开文件的状态，可以通过fcntl的F_GETFL和F_SETFL来实现。无论由哪一个进程实现，都是一样的。
3. 文件描述符标志是进程和文件描述符所私有的，对于该标志的修改不会影响其他的进程或不同进程的文件描述符。

![文件描述符、打开文件句柄和i-node之间的关系](F:\PerBlog\source\_posts\ComputerScience\Linux\linux系统编程\pic\05_linux系统编程_深入探究文件IO\文件描述符、打开文件句柄和i-node之间的关系.svg)

## 复制文件描述符

### dup()

dup调用用一个已经打开的文件描述符oldfd来复制一个新的描述符newfd, 两者指向同一个打开文件句柄，且系统自动使用未使用的编号最小的文件描述符。

```c
#include <unistd.h>
int dup(int oldfd);
```

如果想要使用已打开的文件描述符来存放新复制的文件描述符，那么需要先关闭已打开的文件描述符。

### dup2

dup2调用功能与dup调用类似，只是我们可以指定新的描述符。

```c
#include <unistd.h>
int dup2(int oldfd, int newfd);
```

当newfd为已打开的文件描述符，则dup2会自动关闭该文件描述符.

如果调用成功，这返回副本的文件描述符编号，dup2调用失败并返回EBADF,且不关闭newfd。如果oldfd有效，且与newfd的值相等，那么dup2将什么也不做，不关闭newdf并将其作为调用结果返回。



### fcntl()的F_DUPFD

`newfd = fcntl(oldfd, F_DUPFD, startfd)`

startfd指定了新描述符所在范围的下线，依然是最小未使用规则。

### 复制文件描述符的不足

虽然指向同一个打开文件句柄的不同描述符可以共享文件的偏移量和状态标志，然而新的文件描述符拥有自己的一套文件描述符标志(私有的)，且close-on-exec标志(FD_CLOEXEC)总是处于关闭状态。



### dup3

```c
#define _GNU_SOURCE
#include <unistd.h>
int dup3(int oldfd, int newfd, int flags);
```

dup3只支持一个标志`O_CLOEXEC`, 这个标志可以促使内核文件为新文件描述符设置close-on-exec标志(`FD_CLOSEXEC`)

## 在文件特定偏移量处的IO：`pread`和`pwrite`

`pread`和`pwrite`属于原子操作，将文件的偏移操作和读写操作绑定为同属原子操作, 因而`pread`和`pwrite`在多进程或多线程中使用来避免进程间的竞态。

```c
#include <unistd.h>
ssize_t pread(int fd, void *buf, size_t count, off_t offset);
//Returns number of bytes read, 0 on EOF, or -1 on error
ssize_t pwrite(int fd, void *buf, size_t count, off_t offset);
//Returns number of bytes write, 0 on EOF, or -1 on error
```



## 分散输入和集中输出：`readv`和`writev`

分散输入-集中输出: Scatter-Gather IO

readv和writev的操作对象是多个缓冲区，即可以一次性的对多个缓冲区进行读写。

分散输入就是将文件中的内容读到多个缓冲区中，按顺序依次填充缓冲区，即`readv`。而`readv`将文件中的内容读到多个缓冲区中的操作是原子的。

集中输出就是将多个缓冲区中的内容写到文件中，按顺序依次获取缓冲区中的内容，即`writev`。而`writev`将多个缓冲区中的内容写到文件中的操作是原子的。

```c
#include <sys/uio.h>
ssize_t readv(int fd, conts struct iovec *iov, int iovcnt);
//Returns number of bytes readv, 0 on EOF, or -1 on error
ssize_t writev(int fd, conts struct iovec *iov, int iovcnt);
//Returns number of bytes writev, 0 on EOF, or -1 on error

struct iovec{
    void *iov_base; //缓冲起始地址
    size_t iov_ben; //缓冲区的容量
}
//iovcnt为缓冲区的个数
```

### 在指定偏移量的位置分散输入和集中输出：`preadv`和`pwritev`

两者与`preadv`和`pwritev`相似。只是多加了一个偏移功能，且偏移操作和读写操作同属一个原子操作。

```c
#define _BSD_SOURCE
#include <sys/uio.h>
ssize_t preadv(int fd, const struct iovec* iov, int iovcnt, off_t offset);
//Returns number of bytes preadv, 0 on EOF, or -1 on error
ssize_t pwritev(int fd, const struct iovec* iov, int iovcnt, off_t offset);
//Returns number of bytes pwritev, 0 on EOF, or -1 on error
```



## 截断文件: `truncate`和`ftruncate`

`truncate`和`ftruncate`都可以对文件进行截短，既可以将文件截短也可以将文件截长，截长的部分用'\0'进行填充，形成空洞文件。

`truncate`以文件路径名为参数，对文件进行截断，但是要求文件必须要有写权限并且文件可以访问。如果文件时符号链接，那么`truncate`调用会对其进行解引用

`ftruncate`以文件描述符为参数，对文件进行截断，但要求文件必须以写权限的方式打开文件。该调用不会修改文件的偏移量

```c
#include <unistd.h>
int truncate(const char* name, off_t length);
int ftruncate(int fd, off_t length);
// Both return 0 on success, or -1 on error
```

## 非阻塞IO: O_NONBLOCK

## 大文件IO

1. 使用功能测试宏`define _LARGEFILE64_SOURCE`来启动过渡型LFS_API接口，用来处理在32位机器上的64位的大文件
2. 64位操作文件的API与32位版本命名相同，只是在函数名后面加了64加以区。`open64`、`lseek64`、`truncate64`、`stat64`...
3. 在打开大文件是一定要指定O_LARGEFILE标志
4. `struct stat64`类似于`struct stat`
5. `off64_t`: 64位类型，表示文件偏移量
6. 使用大文件操作的API，即过渡型的`FLS_API`的另一种方法是
    - 在编译的使用，将宏`_FILE_OFFSET_BITS`的值定义位64，如下所示:
    ```shell
    cc -D_FILE_OFFSET_BITS=64 prog.c
    ```
    - 在包含的所有头文件之前添加宏定义
    ```c
    #define _FILE_OFFSET_BITS 64
    #include <stdio.h>
    #include <stdlib.h>
    ...
    ```



## /dev/fd目录

打开/dev/fd目录中的文件等同于复制相应的文件描述符

/dev/fd目录中的文件主要用于shell中

/dev/stdin --> /dev/fd/0

/dev/stdout-->/dev/fd/1

/dev/stderr-->/dev/fd/2



## 创建临时文件

临时文件仅供其在运行期间使用，程序终止后删除。

```c
#include <stdlib.h>
int mkstemp(char* template);
//return fd on success, or -1 on error
```

template参数为路径名的形式，文件名的最后6个字符必须为`xxxxxx`, 这六个字符将会被替换，保证文件的唯一性，且修改后的字符串会通过template参数传回，因此template参数必须指定为字符数组，而非字符串常量。

`char templage[]="/tmp/somestringxxxxxx"`

文件拥有者对临时文件具有读写权限，其他的没有权限

临时文件打开时使用了`O_EXCL`标志，确保文件以独占的形式访问。

通常，打开临时文件不久后，就需要用unlink系统调用来将文件名和临时文件解绑，此时文件名被删除，而临时文件没有被删除。临时文件只有在调用close后才会删除。

```c
#include <stdio.h>
FILE *tmpfile(void);
//Returns file ponter on success, or NULL on error
```

tmpfile调用执行成功后，将返回一个文件流供stdio库函数使用。文件流官兵后自动删除临时文件。

tmpfile打开文件后，会从内部调用unlink函数来删除函数名。



















[^1]: 将某系统调用所要完成的给个动作作为不可中断的操作，一次性加以执行
[^2]: 操作共享资源的两个进程（或线程），其结果取决于一个无法预期的顺序，即这些进程（或线程）获得CPU使用权的先后相对顺序。

