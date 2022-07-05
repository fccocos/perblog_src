# 系统调用I/O函数

<font color=skyblue>系统调用中操作I/O的函数，都是针对文件描述符的</font>.

通过文件描述符可以直接对相应的文件进行操作。如：open、close、write、read、ioctl

## 文件描述符

在对文件进行读写操作前，需要先打开该文件。内核为每个进程维护一个打开文件的列表，该表被称为文件表（file table)。该表由一些叫做文件描述符(file descriptors)(常缩写作fds)的非负整数进行索引。列表中的每项均包含一个打开文件的信息，其中包括一个指向文件备份inode内存拷贝的指针和元数据(例如文件位置和访问模式等）。用户空间和内核空间都把文件描述符作为每个进程的唯一cookies。打开一个文件返回一个文件描述符，而接下来的操作（读写等等）则把文件描述符作为基本参数。

子进程默认会获得一份父进程的文件表拷贝。其中打开文件列表、访问模式，当前文件位置等信息都是一致的。进程中文件表的变化（例如子进程关闭文件）也不会影响其他进程的文件表。然而，可以让子进程和父进程共享后者的文件表（就像线程那样）。

[![LjvX1H.png](https://s1.ax1x.com/2022/04/29/LjvX1H.png)](https://imgtu.com/i/LjvX1H)

文件描述符由C语言的int类型表示。不使用fd_t这个特殊类型来表示，虽然看上去有些古怪，但实际上，这是沿袭了Unix的传统。每个Linux进程有一个打开文件数的上限。文件描述符从0开始，直到比上限小1。默认的上限是1,024,但最多可以将该值设定为1,048,576。负数不是合法的文件描述符，所以-1常常被用来表示一个函数不能返回合法文件描述符的错误。

每个进程按照惯例会至少有三个打开的文件描述符：0,1和2,除非进程显式的关闭它们。文件描述符0是标准输入（stdin),文件描述符1是标准输出（stdout),而文件描述符2是标准错误（stder)。C标准库提供了预处理器宏：STDINFILENO,STDOUTFILENO和STDERRFILENO宏，以取代对以上整数的直接引用。

文件描述符是非负整数。打开现存文件或新建文件时，系统（内核）会返回一个文件描述符。文件描述符用来指定已打开的文件。

```c
#define STDIN_FILENO 0//标准输入的文件描述符
#define STDOUT_FILENO 1//标准输出的文件描述符
#define STDERR_FILENO 2//标准错误的文件描述符
```

程序运行起来后这三个文件描述符是默认打开的。

文件IO的文件描述符和标准IO的文件指针的对应关系

| 文件IO                                                       | 标准IO   |
| ------------------------------------------------------------ | -------- |
| 0                                                            | `stdin`  |
| 1                                                            | `stdout` |
| 2                                                            | `stderr` |
| 如果自己打开文件，会返回文件描述符，而文件描述符一般按照从小到大依次创建的顺序 |          |

## open函数

### 函数描述

通过open()系统调用来打开一个文件并获取到该文件的文件描述符

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int open(const char* name, int flags);
int open(const char* name, int flags, mode_t mode);
```

open()系统调用通过参数name给出的文件与一个成功返回的文件描述符相关联，文件位置指针被设定为零，而文件则根据flags给出的标志为打开

### open函数的flags参数

<font color=red>flags参数必须指定为O_RDONLY，O_WRONLY，O_RDWR这三个中的其中一个</font>

进程必须要有足够的权限才能够调用open系统调用对文件进行相关的操作，例如，flags设置为`O_RDONLY`，open系统调用对该文件拥有读取的权限，可以对文件进行读操作，但不可以对文件进行修改操作。

flag参数可以设置多个值，多个值之间进行按位或运算，用来设置打开文件请求的行为

|  flags参数  | 说明                                                         |
| :---------: | :----------------------------------------------------------- |
|  O_APPEND   | 文件以追加模式打开(文件指针指向文件末尾)                     |
|   O_ASYNC   | 当文件可读或可写时产生一个信号（默认是SIGIO）。该标志仅用于终端和套接字，不能用于普通文件 |
|   O_CREAT   | 如果name指定的文件不存在，就有内核来创建；如果文件存在，这个标志无效，除非给出了O_EXCL标志 |
|  O_DIRECT   | 打开文件用于直接I/O（不通过缓冲区）                          |
| O_DIRECTORY | 如果name不是一个目录，open调用将会失败。这个标志用于在opendir()内部使用 |
|   O_EXCL    | 和O_CREAT一起给出的时候，如果有name给定的文件已经存在，则open()调用失败。用来防止文件创建时出现竞争 |
| O_LARGEFILE | 给定文件打开时使用64位偏移量，这样可以打开大于2G的文件。在64位系统中隐含使用该参数打开文件的 |
|  O_NOCTTY   | 如果给出的name指向一个终端设备（也就说，/dev/tty),它将不会成为这个进程的控制终端，即使该进程目前没有控制终端。这个标志不大常用。 |
| O_NOFOLLOW  | 如果name是一个符号链接，open()调用会失败。通常，会解析该链接并且打开目标文件。如果给出路径的其它部分是链接，该调用仍可用。 |
| O_NONBLOCK  | 如果可以，文件将在非阻塞模式下打开。open()调用不会，任何其它操作都不会使该进程在I/O中阻塞（sleep)。这种情况可能只用于FIFO。 |
|   O_SYNC    | 打开文件用于同步I/O。在数据写到磁盘之前写操作不会完成；一般的读操作已是同步的，所以这个标志对读操作没有影响。POSIX额外定义了O_DSYNC和O_RSYNC;在Linux上，这些标志与O_SYNC同义。（参见本章后面的“O_SYNC标 |
|   O_TRUNC   | 如果文件存在，且为普通文件，并允许写，将文件的长度截断为0。对于FIFO或者终端设备，该参数被忽略。在其它文件类型上则没有定义。因为截断文件需要写权限，所以O_TRUNC和O_RDONLY同时使用也是没有定义的。* |

### open函数的mode参数

当使用了`O_CREAT`参数后，必须指明mode参数，否则会出错。

当文件创建时，mode参数提供新文件的权限。系统并不在本次打开文件时检查权限，所以可以进行任意可行的操作，即使将文件设为只读，但任然可以在本次打开的文件中进行写操作。

mode参数是常见的Unix权限位集合，像八进制数0644(所有者可以读写，其他人只能读）。从技术层面来讲，POSIX允许对特定实现指定具体的值，允许不同的Unix系统设置任何自己需要的位。为补偿权限位上的不可移植性，POSIX引入了一组可以进行按位或操作的常数，以满足对于mode参数的需求。

| mode参数 |              含义              |
| :------: | :----------------------------: |
| S_IRWXU  |    所有者拥有读写和执行权限    |
| S_IRUSR  |       所有者拥有读权限。       |
| S_IWUSR  |       所有者拥有写权限。       |
| S_IXUSR  |      所有者拥有执行权限。      |
| S_IRWXG  |     组拥有读写和执行权限。     |
| S_IRGRP  |         组拥有读权限。         |
| S_IWGRP  |         组拥有写权限。         |
| S_IXGRP  |        组拥有执行权限。        |
| S_IRWXO  | 任何其他人都有读写和执行权限。 |
| S_IROTH  |     任何其他人都有读权限。     |
| S_IWOTH  |     任何其他人都有写权限。     |
| S_IXOTH  |    任何其他人都有执行权限。    |

记忆法：`S_I`+"读|写|执行"+"所有者|组拥有|其他"

读R 写W 执行X  所有者USR|U 组拥有GRP|G 其他|OTH|O

当RWX在一起的时候，所有者|组拥有|其他就要只写一个字母U|G|O，

当RWX分开单独写的时候,所有者|组拥有|其他就要只写三个字母USR|GRP|OTH.

正真在写mode参数的时候，可以使用八进制的形式进行书写，例如`0744`即当前用户拥有DWX, 组用户拥有D, 其他用户拥有D

1     2     4 ->八进制 1+2+4 = 7

X    W    D

7 =  4 +2+1   <--> 7 = D+W+X

3 = 2+1   <-->  3=W+X

5 = 4+1  <--> 5 =  D + X

1 =X          2 =W           3= WX          4= D         5= DX         6= WX     7= DWX



## Creat函数

### 函数描述

`open(name,O_WRONLY|O_CREAT|O_TRUNC,0664)`等价于`Creat(name, 0664)`，该操作的意义在于，以只读的权限创建一个文件，如果文件存在，则将该文件的长度截为0.

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int creat(const char* name, mode_t mode);
```



### 返回值和错误码

open()和creat()调用成功返回一个文件描述符，失败返回-1，并将errno设置为一个合适的错误值。



## read函数

### 函数描述

read()系统调用的作用就是读取文件，是一个种最基本、最常用的文件读取机制。

```c
#include <unistd.h>
ssize_t read(int fd, void* buf, size_t len);
```

该系统调用由fd指向的文件的当前偏移量开始最多读取len个字节到buf中。成功返回写入到buf中的字节数，失败返回-1，并设置errno。fd所指向的文件的位置指针随着读取的字节而向前移动，位置指针移动的距离取决于之前读取的字节数决定。默认情况下，第一次读取时，位置指针指向文件的开头，随着读取的次数增加，位置指针就会向后移动，直到文件末尾，结束读取，这是一次成功的读取，因此会返回最后一次读取内容的字节数。

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>

int main(int agrc, char *argv[])
{
    //用于接收打开文件的文件描述符
    int fd;
    //用于接收返回状态
    int ret;
    char buf[1024];
    //清空buf
    memset(buf, 0, sizeof(buf));

    fd = open("test.txt", O_RDONLY);
    if (fd < 0)
    {
        perror("OPEN_ERROR:");
        exit(1);
    }
    printf("以只读的方式成功的打开了文件\n");

    printf("********************************\n");
    while (ret = read(fd, buf, sizeof(buf)))
    {
        printf("%d\t\"%s\"\n", ret, buf);
    }
    printf("*******************************\n");

    ret = close(fd);
    if (ret == -1)
    {
        perror("CLOSE_ERROR:");
    }
    printf("关闭文件成功\n");
}

运行结果：
    
以只读的方式成功的打开了文件
********************************
456     "你知道你和痘痘的区别嘛？痘痘会消失而你不会您爹妈成亲生死簿亲自签的字，阎罗殿拜的堂，奈何桥交的杯，鬼门关行的房，小男孩和小胖子就是你爹妈交换戒指时的礼炮，令堂黄泉路怀胎十月孟婆亲自给您接生，拉生出来您这不人不鬼的阴阳人白哲特曾经说过，坚强的信念能赢得强者的心，并使他们变得更坚强。 这似乎解答了我的疑惑。"
*******************************
关闭文件成功
```

### 返回值

1. 返回一个比len小的非零正整数是合法，出现这种情况的原因有：

   - 可供读取的字节数本来就比len要小
   - 系统调用可能被信号打断
   - 管道被破坏(fd做成的管道)
   - ......

2. 返回0，此时位置指针移到了文件末尾(EOF), 没有字节可读。

   EOF并不是一种错误（也因此不用返回-1);它仅仅表示文件位置指针已经位于文件最后一个有效偏移量之后，之后没有任何数据可读了。然而，如果一个调用需要读len个字节，但却没一个字节可读，调用将阻塞（睡眠）,直到那些字节可以读取为止（假设文件描述符没有在非阻塞模式下打开；参见“非阻塞读取”）。注意这与返回EOF时不同。就是，“没有数据可读”和“数据末尾”是不同的。在EOF的情况下，到达了文件末尾。在阻塞的情况下，读操作在等待更多的数据一一例如在从套接字或者设备文件读取的时。

3. 有些错误是可以恢复的。

   比如，当readO调用在未读取任何字节前被一个信号打断，它会返回-1(如果为0,则可能和EOF混淆）,并设置enno为EINTR。在这种情况下，你可以重新提交读取请求。

对read()的调用确实会有很多可能的结果：

- 调用返回一个等于len的值。所有1en个被读取字节存储在buf中。结果和预期一致。

  - 调用返回了一个大于零但小于len的值。

  - 出现在一个信号打断了读取过程，

  - 在读取中发生了一个错误，

  - 在读入len个字节前已抵达EOF。

  - 再次进行读取（更新了buf和len的值）将读入剩余字节到buf的剩余空间中

- 调用返回0。这标志着EOF。没有可以读入的数据。

- 调用阻塞了，因为没有可用的用来读取的数据。这在非阻塞模式下不会发生。

- 调用返回-1,并且ermo被设置为EINTR。这表示在读入字节之前收到了一个信号，可以重新进行调用。

- 调用返回-1,并且emo被设置为EAGAIN。这表示读取会因没有可用的数据而阻塞，而读请求应该在之后重开，这只在非阻塞模式下发生。

- 调用返回-1,并且ermo被设置不同于EINTR或EAGAIN的值，这表示某种更严重的错误。

read函数案例：输出文件中所有的内容，并处理所有可能出错的情况

函数声明：

```c
#ifndef READ_H
#define READ_H

#define FAILED 0
#define EMPTY 1

typedef unsigned int  r_size;

/**
 * @brief 该函数用于一次读取文件中的内容，提供的文件必须已经创建，且该函数只有该文件的读权限
 * 
 * @param name 文件路径
 * @param content 为一个输出变量，用于存放文本中的数据
 * @param block_size 缓冲区的大小
 * @return r_size 成功返回文件长度，即文件的大小；失败返回FAILED并设置errno; 如果为空文件，返回EMPTY
 */
r_size OneTimeRead(const char* name, char** content, r_size block_size);
#endif //READ_H
```

函数定义：

```c
#include "read.h"
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <errno.h>

r_size OneTimeRead(const char *name, char **content, r_size block_size)
{
    int size = block_size, tmp = 0;
    int fd, ret;

    char *bufptr = (char *)malloc(block_size);
    *content = (char *)malloc(block_size);

    //打开文件
    if ((fd = open(name, O_RDONLY)) == -1)
    {
        perror("OPEN_ERROR");
        return 0;
    }

    //读文件
    while ((ret = read(fd, bufptr, block_size)) != 0)
    {
        if (ret == -1)
        {
            if (errno == EINTR)
                continue;
            perror("READ_ERROR");
            return 0;
        }

        strcpy(*content + tmp, bufptr);
        size += ret;
        *content = (char *)realloc(*content, size);
        tmp = size - ret;
    }

    if (bufptr != NULL)
    {
        free(bufptr);
        bufptr = NULL;
    }

    if (size == block_size)
        return EMPTY;

    return size;
}

```

### 非阻塞读

有时候，程序员不希望当没有可读数据时让read()调用阻塞。相反，他们倾向于在没有可读数据时，让调用立即返回。这种情况被称为非阻塞I/O;它允许应用在从不阻塞的状况下进行I/O操作，如果是在操作多个文件时，不至于丢失其他文件中的可用数据。

所以，还有一个errno的值需要检查EAGAIN。像先前讨论的那样，如果给出的文件描述符在非阻塞模下打开（open0中给定O_NONBLOCK;参见“open(的flags参数”）并且没有可读数据，read()调用会返回-1,且设置errno为EAGAIN而不是阻塞掉。在进行非阻塞I/O时，你必须检查EAGAIN,否则将可能因导数据缺失而导致严重的错误。



### read调用大小限制

size_t和ssize_t类型由POSIX确定。size_t类型用来存储用字节衡量大小的值。ssize_t类型是有符号的size_类型（负值用来表示错误）。在32位系统上，对应的C类型一般分别是unsignedint和int。由于两种类型常常一起使用，ssize_t的较小范围就潜在地给size_t的范围作出了限制。

size_t的最大值为SIZE_MAX;ssize的最大值为SSIZE_MAX。如果len比SSIZE_MAX大，read()调用的结果是未定义的。在大部分Linux系统上，SSIZE_MAX是LONG_MAX,在32位系统上即0x7fffffff。这个数字对一次读入来说已经足够大了，但还是需要记住它。如果你要用前面的读循环代码作为一种通用的读取方式，你可能需要增加如下代码：

`if(len>SSIZE_MAX) len=SSIZE_MAX`



## write函数

### 函数描述

wirte系统调用是最常见的写文件操作，它与read系统调用相对应。

```c
#include <unistd.h>

ssize_t write(int fd, const void* buf, size_t count);
```

一个write系统调用从由文件描述符fd指向的文件的当前位置开始，将buf中之多count个字节的内容写入到文件中。<font color=red>不支持定位的文件（如==字符设备==[^1]）总是从文件的“开头”开始写。</font>

### 返回值和错误码

成功时，返回写入到文件中的字节数，并更新文件位置指针的指向；错误时，返回-1并将errno设置为相应的值。如果写入0个字节或没有内容可写的时候，会返回0。

| 错误码 | 意义                                                         |
| ------ | ------------------------------------------------------------ |
| EBADF  | 非法的文件描述符或者不是以读方式打开的                       |
| EFAULT | buf指针的指向不在进程地址空间内                              |
| EFBIG  | 写操作将使文件大小超过进程的最大文件限制，或者内部实现的限制 |
| EINVAL | 文件描述符所指向的对象不能进行写操作                         |
| EIO    | 发生了一个底层IO错误                                         |
| ENOSPC | 文件描述符所在的文件系统没有足够的空间                       |
| EPIPE  | 定文件描述符关联的管道或套接字的读端被关闭。进程也将收到一个SIGPIPE信号。SIGPIPE信号的默认行为是终止收到信号的进程。因而，进程只能在选择忽略，阻塞或者处理该信号时得到这个值。 |

在用write系统调用进行编程的时候，与read系统调用类似，在进行部分写的时候，需要对write系统调用的返回值进行判断并作出相关处理。如下所示：

```c
unsigned long word = 1720;
size_t count;
ssize_t nr;
count = sizeof(word);
nr = wirte(fd, &word, count);
if(nr == -1)
    //error,check errno
else if(nr != count)
    //可能出现的错误， 但错误在errno中没有出现
    
```

### 部分写

向对于read()的返回部分的情况，write()不太可能返回一个部分写的结果。对于普通文件，除非发生一个错误，否则write将保证写入所有的内容。

<font color=skyblue>对于普通文件，没有必要进行循环写</font>

对于其他类型的文件，如套接字等，需要有个循环来保证正的将要写入的内容写入进去了。于此同时，循环写的另一个好处就是，在进行下一次write调用之前可以通过返回值来判定上一次部分写的情况。如下所示：

```c
ssize_t ret,nr;
size_t len;
while(len!=0&&(ret = write(fd, buf, len))!=0)
{
    if(ret == -1)
    {
        if(errno == EINTR)
            continue;
        perror("WRITE_ERROR");
        break;
    }
    len-=ret;
    buf += ret;
}
```

上述代码对于普通文件基本上没有太大的意义，即循环部分写对于不同文件来说没有太大的作用。因此<font color=green>对于普通文件的写，能一次性写完就一次写完。</font>

### 追加模式

当文件以追加模式(`O_APPEND`)打开的时候，写操作不会从当前位置开始，而是从文件的末尾开始写。

在多进程中，如果文件不以追加模式打开的话，将会出现内容覆盖的情况，即第一个进程写的内容可能会被第二个进程写的内容覆盖，也就是说第一个进程中文件位置指针的指向的修改并不会传递给第二个进程。也就是说多个进程之间如果不进行显式的同步则不能追加写操作，因为它们存在竞争条件。

追加模式避免了这样的问题。它保证文件位置总是指向文件末尾，这样所有的写操作总是追加的，即便有多个写者。你可以认为每个写请求之前的文件位置更新操作是原子操作。文件位置更新至刚刚写入的数据结尾。这和下一个write()调用无关，因为更新文件位置是自动完成的，但可能因为某些奇怪的原因会影响到下一个read()调用。

追加模式在某些任务中很管用，如更新日志文件。

### 非阻塞写

在非阻塞模式下`O_NONBLOCK`，发起写操作产生阻塞时，write系统调用返回-1, 并设置errno值为EAGAIN，此时写操作请求应该在等待一段事件后重新发起。处理普通文件不会出现这种情况

### write大小限制

一次能够写入的最大值为`SSIZE_MAX`，如果超过该值，write调用的结果是未定义的，最小值为0，write调用将立即返回0

### :star:write的行为（在内核中写延迟）

当一个write()调用返回时，内核已将==所提供的缓冲区数据==**复制**到了==内核缓冲区==中，但**<u>却没有保证数据已写到目的文件</u>**。write调用返回对于这样的情况来讲确实太快了。处理器与硬盘的速度差异使这种情况非常明显。

❤当用户空间应用发起write()系统调用时，Linux内核进行几项检查，然后直接将数据拷贝至一个缓冲区中。稍后，在后台，内核收集所有这样的“脏”缓冲区，将它们排好序，并写入到磁盘上（此过程称为==回写==[^2]）。这使得write调用马上被调用并立刻返回。<font color=red>内核可以将写入操作推迟到空闲阶段，并将很多写操作一起处理。</font>

**这种延迟写不会改变POSIX所定义的语义。**举例来说，如果一个read调用希望读取刚刚写到缓冲区中但尚未写入磁盘的数据，请求将从缓冲区中响应，而不是读取磁盘上“陈旧”的数据。这种行为实际上提高了效率，因为read只需从内存缓存中读而不用到硬盘中找。读写请求如预计般交错，而结果也如预期那样——当然，前提是在数据写入磁盘前，系统没有崩溃！在这种情况下，即使对于一个应用程序来讲写操作已经成功了，但数据并没有写入到磁盘。

**另外一个关于延迟写的问题是对强制写顺序的不可能性。**尽管一个应用可能会注意安排它的写请求以使它们将按特定顺序写入磁盘，出于性能方面的考虑，内核将按它觉得合适的顺序重新安排。在系统崩溃时才是问题，而最终所有缓冲区都将正确地写回。尽管如此，绝大多数的应用实际上并不关心写顺序。

**最后一个延迟写的问题是关于特定I/O错误的报告。**任何在回写中出现的I/O错误——比方说，一个物理磁盘驱动器错误——不会被报告给发起写请求的进程。实际上，<u>缓冲区是和进程完全无关的</u>。多进程可能会更新同一片缓冲区中的数据，而进程可能在数据仍在缓冲区未回写到磁盘前就退出了。另外，你怎么才能在事后与一个写操作失败的进程通讯呢？

内核会尝试将延迟写的风险最小化。**为保证数据适时写入，内核创立了一种缓存最大时效机制，并将所有脏的缓存在它们超过给定时效前写入磁盘。**用户可以通过/proc/sys/vm/dirty_expire_centiseconds来配置这个值。这个值以十毫秒计（百分之一秒）。

强制文件缓存写回也是可以的，甚至可以将所有的写操作同步。

## 同步IO

### 引子

有些应用想要控制数据写入磁盘的时间，Linux内核提供了一些选择来允许用性能换取同步操作

### fsync()和fdatasync()

两个系统调用都可以确认数据写入磁盘。

```c
#include <unistd.h>

int fsync(int fd);
int fdatasync(int fd);
```

调用fsync0可以保证fd对应文件的脏数据回写到磁盘上。文件描述符fd必须是以写方式打开的。该调用**回写数据以及建立的时间截和inode中的其他属性等元数据**。在驱动器确认数据已经全部写入之前不会返回。

在将数据写入硬盘缓存时，fsync()是不可能知道数据是否已经在磁盘上了。磁盘可能报告说数据已写入，但数据可能还在磁盘驱动器的缓存上。幸运的是，在磁盘驱动器缓存中的数据将会很快写入到磁盘。

fdatasync()系统调用完成的事情和fsync()一样，区别在于它**仅写入数据**。该调用不保证元数据同步到磁盘上，故此可能快一些。一般来说这就够了。

这两个调用都**不保证**任何已经更新的包含该文件的**目录项同步到磁盘上**。这意味着如果==文件链接==[^3]刚刚被更改过，文件数据可能会成功写入磁盘，但却没有关联到相应的目录项上，致使文件不可访问。**为保证任何对目录项的更新也同步到磁盘上，必须对目录本身也调用fsync()进行同步。**

### 返回值和错误码

成功时，两个调用都返回0。失败时，都返回-1，并将errno设置为以下三个值之一：

| 错误码 | 意义                                                         |
| ------ | ------------------------------------------------------------ |
| EBADF  | 不是一个可以写入的合法描述符                                 |
| EINVAL | 文件描述符指向的对象不支持同步                               |
| EIO    | 在同步时发生了一个底层的IO错误。这表示一个真正的IO错误，此类错误经常在错误发生出被捕获 |

一般来讲，即使在相应文件系统上实现了fdatasync()而未实现fsync),在调用fsync()时也会失败。“偏执”的应用可能会在fsync()返回EINVAL时尝试用fdatasync0,如下所示：

```c
if(fsync(fd) == -1)
{
    if(errno == EINVAL)
        if(fdatasync(fd) == -1)
            perror("fdatasync");
    else perror("fsync");
}
```



在POSIX标准中fsync()是必要的，而fdatasync()是可选的，因此fsync()在所有常见的、面向普通文件的Linux文件系统都已经实现了。然而，特殊文件类型（可能是那些没有元数据需要同步的文件类型）或者不常见的文件系统或许只实现了fdatasync()。

### sync()

sync系统调用可以用来对磁盘上的所有缓冲区进行同步，尽管其效率不高，但任然被广泛使用：

```c
#include <unistd.h>
void sync(void)
```

该函数没有参数，也没有返回值。它总是返回成功，并确保所有的缓冲区（包括数据和元数据）都能够写入磁盘

标准中并不要求sync()一直等待到所有缓冲区都写到磁盘才返回；只需要调用它来**启动将整个缓冲区写入磁盘的过程**即可。由此，一般建议同步多次以确保所有的数据都安全的写入磁盘。然而**对于Linux来讲，sync()一定要等到所有的缓冲区都写入才返回。因而，调用一次sync()就够了。**

sync()真正派上用场的地方是在**工具sync的实现中**。**<u>应用程序</u>则使用fsync()和fdatasync)将文件描述符指定的数据同步到磁盘。**需要注意的是，可能在一个
繁忙的系统上sync()操作可能需要几分钟的时间来完成。

### O_SYNC标志

O_SYNC在open系统调用中使用，使所有在文件上的IO操作同步

```c
int fd;
fd = open(fd,O_WRONLY|O_SYNC);
if(fd==-1)
{
    perror("open");
    return -1;
}
```

读请求是同步的，即直到数据需要返回给用户的时候才会返回。

写请求一般都是非同步的，因为写调用的返回与数据写入到磁盘没有必然的联系。（是内存和外设之间的速度不匹配所造成的，即延迟写）

O_SYNC标志可以将可将wirte的调用返回和磁盘写回进行强制关联，从而保证write调用进行IO同步。

对write调用进行IO同步后，相当于在wirte操作之后将隐式的调用fysnc()，然而Linux内核实现的O_SYNC会更加的有效

O_SYNC将在写操作中稍稍影响用户和内核的时间，但主要的时间消耗是在IO等待上，即等待ＩＯ完成的时间。Ｏ＿ＳＹＮＣ的总耗时会增加一到两个数量级，也就是说文件越大，时间消耗成数量级增长，所以同步ＩＯ一般情况下是最后的选择。

如果只是确保数据会写到磁盘，那么应该使用ｆｓｙｎｃ（）或者ｆｄａｔａｓｙｎｃ（），因为需要更少的调用，相对于Ｏ＿ＳＹＮＣ来说，开销更少。

### Ｏ＿ＤＳＹＮＣ和Ｏ＿ＲＳＹＮＣ

在Linux中O_DSYNC、O_RSYNC与O_SYNC一样，他们具有相同的行为。

O_DSYNC标志指定每次写操作后只是普通数据同步，元素数据不同步，就像在写请求后隐式地调用了fdatasync()一样。因为O_SYNC提供了更好的保证，所以没有明确支持O_DSYNC时并不会导致功能缺失；只是在O_SYNC有更强要求的情况下有一点性能损失。

O_RSYNC标志要求读请求像写请求那样进行同步。因此，该标志只能和O_SYNC或O_DSYNC一起使用。O_RSYNC标志保证任何读操作的副作用也是同步的，即读操作更新的元数据也必须在调用返回之前写入磁盘，实际上，就是read系统调用返回前，访问文件时间必须更新到磁盘上的inode中。

## 直接IO

在open0中使用ODIRECT标志会使内核最小化IO管理的影响。**使用该标志时，I/O操作将忽略页缓存机制，直接对用户空间缓冲区和设备进行初始化**。**所有的I/O将是同步的；操作在完成之前不会返回。**

当使用直接I/O时，**请求长度，缓冲区对齐，和文件偏移必须是设备扇区大小（通常是512字节）的整数倍**。在2.6内核之前，要求更严格：在2.4中，所有的东西都必须对齐到文件系统逻辑块大小（一般是4KB)。为保持兼容性，应用需要对齐到更大的（而且更不便的）逻辑块大小。

## colse关闭文件

### 函数描述

程序完成对某文件的操作之后，可以使用close（）系统调用将文件描述符和对应的文件解除关联。

```c
#include <unistd.h>
int close(int fd)
```

close()调用解除了已经打开的文件描述符的关联，并分离进程和文件的关联。给定的文件描述不再有效，除非对该描述符重新绑定。

close()调用成功返回0，失败返回-1并设置errno。

>需要注意的是，关闭文件和文件写入磁盘没有关系，如果想要确保文件在关闭之前写到磁盘，需要进行IO同步操作。

### close()系统调用的副作用

当最后一个引用某文件的**文件描述符关闭后**，在内核中表示文件的数据结构会被释放。当该数据结构被释放的时候，与文件关联的inode的内存拷贝也会被清除。

如果没有什么连接到该inode了，这个inode可能会从内存中清楚，但也有可能保留在内存中(内核为了效率保留了一些inode)。

如果文件已经从磁盘上解除链接，但是在解除前仍保持打开状态，它在被关闭且inode从内存中移除前不会真的被删除，所以，close()的调用可能会使某个已经解除链接的文件最终从磁盘上被删除。

### 错误码

1. 一定要检查close的返回值，因为某些操作可能有延迟，其错误可能在后面才出现，而close可以报告这些错误
2. 在出错时可能出现的errno值有`EBADF(非法描述符)`、`EIO（底层IO错误）`
3. 在出错时不能出现的errno值为`EINTR(产生信号中断)`
4. 如果忽略出现的错误，文件描述合法的情况下，文件总是会被关闭，并且与其关联的数据结构会被释放

## 用lseek()查找

lseek()系统调用能够对给定的文件描述符引用的文件位置设定指定值。除了更新文件位置，没有其它的作用，并且不初始化任何IO。

```c
#include <sys/types.h>
#include <unistd.h>
off_t lseek(int fd, off_t pos, int origin);
```

参数origin可以设置为SEEK_CUR、SEEK_END、SEEK_SET

- SEEK_CUR  当前值，  将当前文件位置设置为当前值+pos,pos可以为负值、零或者正值
- SEEK_END 文件长度值， 将当前文件位置设置为文件长度+pos
- SEEK_SET 文件开头即0 ,  将当前文件位置设置为0+pos

参数pos为偏移步长。

lseek()系统调用成功返回0，失败返回-1并设置errno值。

`(off_t)-1`为最大偏移量，用于判断lseek设置偏移位置是否出错

lseek()常见的用法就是用来定位当前文件的开始和末尾，或者确定某个文件描述符引用的当前位置。

### 文件末尾之后进行查找

lseek()是可以在文件指针超过文件末尾之后进行查找的。如果在文件末尾之后的位置对文件进行写操作，则会在新旧长度之间创建新的空间，并由零来填充。这种由零填充方式称为“空洞(hole)”, 空洞不占用任何物理上的磁盘空间。暗示着文件系统上所有文件的大小加起来可以超过磁盘的物理大小。带空洞的文件叫做"稀疏文件(sparse file)"。稀疏文件可以节省可观的空间并提高效率，因为操作那些空洞并不会引发任何物理IO

### 错误码

| 错误码    | 意义                                                         |
| --------- | ------------------------------------------------------------ |
| EBADF     | 非法描述符                                                   |
| EINVAL    | origin的值不是预设的三个值中的一个或者最终计算文件位置为负数 |
| EOVERFLOW | 文件偏移不能被off_t表示，即产生溢出                          |
| ESPIPE    | 文件描述符关联到了一个不能执行查找擦做的对象上，例如，管道、FIFO或套接字 |

### 限制

参数pos的最大值为off_t类型的大小，off_t定义为long类型。内核将偏移量存储成long long类型，这种处理方式在64位机器中没有问题，但是在32位机器上可能会出现EOVERFLOW错误

## 定位读写

### 函数描述

Linux提供了两种read()和write()的变体来替代1seek0,每个调用都以需要读写的文件位置为参数。完成时，不修改文件位置。

```c
#include <unistd.h>
ssize_t pread(int fd, void* buf, size_t count, off_t pos);
ssize_t pwrite(int fd, void* buf, size_t count, off_t pos);
```

pread()调用从文件描述符fd的pos文件位置读取count个字节到buf中

pwrite()调用从文件描述fd的pos文件位置将buf中的count个字节写入到文件中

pwrite和pread除了能够指定文件位置外，与普通的write和read几乎没有区别

pwrite和pread的功能就是在文件中指定位置进行读写，从功能实现的角度来看，无非就是在进行write和read操作之前使用lseek进行定位，但仍有三点区别：

1. pwrite和pread使用更加地方便
2. pwrite和pread完成操作后，不修改文件位置
3. 避免了任何使用lseek时可能出现的潜在竞争。例如，由于线程共享文件描述符，可能在一个线程调用lseek后，但尚未进行读写操作前，另一个线程修改文件位置，此时我们可以使用pwrite和pread来避免产生这种竞争。

### 错误码

成功返回读写的字节数。pread返回0表示EOF，而pwrite返回0表示没有写入任何内容。

出错时均返回-1并设置errno的值。对于pread，会返回read和lseek会出现的errno值；pwrite同样会返回write和sleek会出现的errno值

## 文件截断

```c
#include <sys/types.h>
#include <unistd.h>
int ftruncate(int fd, off_t len);
int truncate(const char* path, off_t len);
```

上述两个调用都用于截断文件，都是将文件截短到len指定的长度。

ftruncate通过文件描述符(可写)来截断文件，而truncate通过path指定的文件路径（文件具有写权限）来截断文件

ftruncate和truncate成功返回0，失败返回-1并设置errno值

ftruncate和truncate最常见的用法就是将原来的文件截短，截短之后的数据会被忽略且不可读。

ftruncate和truncate当然也可以将文件截断得比原来长，其类似于”在文件末尾之后进行写操作“，扩展出来的字节将全部由0填充



## :star:IO多路复用

IO多路复用允许应用在多个文件描述符上同时阻塞，并在其中某个可以读写时收到通知。这时IO多路复用就成了应用的关键所在，一般来讲I/O多路复用的设计遵循以下原则：

1. I/O多路复用：当任何文件描述符准备好I/O时告诉我

2. 在一个或更多文件描述符就绪前始终处于睡眠状态。

3. 唤醒：哪个准备好了？

4. 在不阻塞的情况下处理所有I/O就绪的文件描述符。

5. 返回第一步，重新开始。

Linux提供了三种I/O多路复用方案：select,poll和epol。

### select

select()系统调用提供了一种实现同步IO多路复用的机制

```c
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
int select(int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
FD_CLR(int fd, fd_set* set);
FD_ISSET(int fd, fd_set* set);
FD_SET(int fd, fd_set* set);
FD_ZERO(int fd, fd_set* set);
```

在指定的文件描述符准备好IO之前或者超过一定的时间限制，select()调用会阻塞。

检测的文件描述符可以分为三类，分别等待不同的事件。

- 监测readfds集合中的文件描述符，确认其中是否有可读的数据，也就是说，多操作可以无阻塞的完成
- 监测writefds集合中的文件描述符，确认其中是否有一个写操作可以不阻塞地完成
- 检测exceptfds中的文件描述符，确认其中是否有出现异常发生或者出现外带数据（套接字）

返回成功时，每个集合只包含对应类型的IO就绪的文件描述符。

第一个参数n,等于所有集合中文件描述符的最大值加一。我们可以这样做，将select需要找到最大的文件描述符值并将其加一后传递给第一个参数。

timeout参数是一个指向timeval结构体的指针

```c
#include <sys/time.h>
struct timeval{
    long tv_sec;
    long tv_usec;
};
```

如果这个参数不是NULL,即使此时没有文件描述符处于I/O就绪状态，select()调用也将在tv_sec秒tv_usec微秒后返回。返回时，这个结构体的状态在大多俗话Unix系统中都是未定义的。这样的话，每次调用前都必须重新初始化（还有集合中的文件描述符）。较新版本的Linux会自动将该值改为剩余的时间。这样，如果时限是5秒，在某个文件描述符准备好时过去了3秒，tv.tv_sec在返回时就还是2。

如果时限中的两个值都是零，调用会立即返回，并报告调用时所有事件对应的文件描述符均不可用，且不等待任何后续事件。

集合中的文件操作符并不直接操作，而是通过辅助宏来进行管理。这就允许Unix系统按其所希望的方式来实现，大多数系统，将其实现为位数组。

FD_ZERO从指定集合中移除所有文件描述符。在每次使用select()之前，需要调用该宏。

FD_SET向指定集合中添加一个文件描述符，而FD_CLR从指定集合中移除一个文件描述符。设计良好的代码应该从不使用FD_CLR。一般来讲，很少使用该宏。

FDISSET测试一个文件描述符在不在给定集合中。如果在，则返回一个非
零值，否则用0表示不在。一般在selectO调用返回后使用FDISSET来检查一个
文件描述符是否就绪。

由于文件描述符集合是静态建立的，所以对于文件描述符数量的上限和文件描述符的最大值均有限制，二者都由FD_SETSIZE设定。在Linux上，这个值是1024。

#### 返回值和错误码

成功返回在所有三个集合中IO就绪的文件描述符的数目。如果有时限，返回值可能为0；错误返回-1并设置errno的值

| EBADF  | 某个集合中的一个描述符非法               |
| ------ | ---------------------------------------- |
| EINTR  | 等待时捕捉到了一个信号，可以重新发起调用 |
| EINVAL | 参数n为负数，或者给出的时限不合法        |
| ENOMEM | 没有足够的内存完成请求                   |
|        |                                          |

#### select实例程序

```c
#include "select.h"
#include <stdio.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>

void select_model()
{
    struct timeval tv;
    fd_set readfds;
    int ret;

    //等待标准输入
    FD_ZERO(&readfds);
    FD_SET(STDIN_FILENO, &readfds);
    //等上5秒
    tv.tv_sec = TIMEOUT;
    tv.tv_usec = 0;
    //立刻阻塞
    ret = select(STDIN_FILENO + 1,
                 &readfds,
                 NULL,
                 NULL,
                 &tv);

    if (ret == -1)
    {
        perror("SELECT");
        return 1;
    }
    else if (!ret)
    {
        printf("%d senconds elapsed.\n", TIMEOUT);
        return 0;
    }

    //文件描述符就绪
    if (FD_ISSET(STDIN_FILENO, &readfds))
    {
        char buf[BUF_LEN + 1];
        int len;
        //确保是非阻塞的
        len = read(STDIN_FILENO, buf, BUF_LEN);
        if (len == -1)
        {
            perror("READ");
            return 1;
        }
        if (len)
        {
            buf[len] = 0;
            printf("read: %s\n", buf);
        }
        return 0;
    }
    fprintf(stderr, "This should not hapen!\n");
    return 1;
}
```

#### 用select()实现可移植的sleep()

```c
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>

int sleep(int s, int ms)
{
    struct timeval tv;
    tv.sec = s;
    tv.usec = ms;
    int ret = select(0, NULL, NULL, NULL, &tv);
    if(ret==-1)
    {
        perror("SLECET");
        return -1;
    }
    return 0;
}
```



### pselect

```c
#define _XOPEN_SOURCE 600
#include <sys/select.h>
int pselect(int n,fd_set* readfds, fd_set* writefds, fd_set* exceptfds, const struct timespec *timeout, const sigset_t *sigmask);
FD_CLR(int fd, fd_set* set);
FD_ISSET(int fd, fd_set* set);
FD_SET(int fd, fd_set* set);
FD_ZERO(int fd, fd_set* set);
```

pselect0和select0有三点不同：

1. pselectO的timeout参数使用了timespec结构，而不是timeval结构。time-spec使用秒和纳秒，不是秒和毫秒，从理论上来讲更精确一些。实际上，两者在毫秒精度上已经都不可靠了。

2. pselect()调用并不修改timeout参数。这个参数在后续调用时也不需要重新初始化。

3. select()调用没有sigmask参数。当这个参数被设置为零时，pselect()的行为等同于select()。

timespec结构体的定义：

```c
struct timespec{
    long tv_sec;
    long tv_nsec;
};
```

添加pselect()到Unix工具箱的主要原因是为了增加sigmask参数，以此来解决信号和等待文件描述符之间的竞争条件（信号在第九章深入讨论）。假设一个信号处理程序设置了一个全局标记（大部分信号处理程序都这么干）,进程在每次调用select0前都要检查这个标记。现在，假如在检查标记和调用之间接收到信号，应用可能会阻塞，并不再响应该信号。pselect()提供了一组可阻塞的信号，可以解决这个问题。阻塞的信号直到解除阻塞才会被处理。一旦pselectO返回，内核就恢复旧的信号掩码。详见第九章。

2.6.16内核之前，Linux实现的pselectO还不是一个系统调用，而是由glibc提供的一个简单的对select()的封装。该方法使竞争条件出现的风险最小化，但是没有根本消除。当真正引入一个系统调用，才彻底解决竞争问题。

如果不考虑pselectO中（相对不大的）改进，大多数应用会继续使用se-lectO,部分是出于习惯，其他则是考虑可移植性。

### poll

poll0系统调用是SystemV的IO多路复用解决方案。它解决了一些selecto的不足，不过selectO仍经常被使用（还是出于习惯和可移植性的考虑）:

```c
#include <sys/poll.h>
int poll (struct pollfd *fds, unsigned int nfds,int timeout);
```

与select0使用的三个基于位掩码的文件描述符集合不同，poll0使用一个简单的nfds个pollfd结构体构成的数组，fds指向该数组。结构体定义如下：

```c
#include <sys/poll.h>
struct pollfd {
    int fd;/* file descriptor*/
    short events;/*requested events to watch*/
    short revents;/*returned events witnessed*/
};
```

每个pollfd结构体指定监视单一的文件描述符。可以传递多个结构体，使得poll()监视多个文件描述符。每个结构体的events字段是要监视的文件描述符事件的一组位掩码。用户设置这个字段。revents字段则是发生在该文件描述符上的事件的位掩码。内核在返回时设置这个字段。所有在events字段请求的事件都可能在revents字段中返回。下面是合法的事件：

- POLLIN                         没有数据可读。
- POLLRDNORM          有正常数据可读。
- POLLRDBAND            有优先数据可读。
- POLLPRI                       有高优先级数据可读。
- POLLOUT                      写操作不会阻塞。
- POLLWRNORM           写正常数据不会阻塞。
- POLLBAND                   写优先数据不会阻塞。
- POLLMSG                      有一个SIGPOLL消息可用。

另外，如下事件可能在revents中返回：

- POLLER                            给出文件描述符上有错误。
- POLLHUP                         文件描述符上有挂起事件。
- POLLNVAL                       给出的文件描述符非法。

这些在events中没有意义，而总是在合适时返回。使用pollo,不像select()那样，你无需另外请求报告异常。

POLLIN|POLLPRI等价于selectO的读事件，而POLLOUT|POLLWRBAND等价于selectO的写事件。POLLIN等价于POLLRDNORM|POLLRDBAND,而
POLLOUT等价于POLLWRNORM

举例来说，监视一个文件描述符是否可读写，我们需设置events为POLLIN|POLLOUT。返回时，我们将在revents中是否有相应的标志。如果设置了POLLIN,文件描述符就需要能非阻塞读。如果设置了POLLOUT,文件描述符需要能非阻塞写。这些标志并不相互排斥：二者都可以设置，表示可以在文件描述符上读写，并都不会阻塞。

timeout参数指定在任何I/O就绪前需要等待时间的长度，以毫秒计。负值表示永远等待。一个零值表示调用立即返回，列出所有未准备好的I/O,但不等带任何其他事件。这种情况下，poll0就如同其名，轮询一次后立即返回。

#### 返回和错误码

成功时，pollo返回具有非零revents字段的文件描述符个数。超时前没有任何事件发生则返回零。失败时返回-1,erno被设置为下列值之一：

- EBADF           一个或更多结构体中有非法的文件描述符。
- EFAULT        指向fds的指针超出了进程地址空间。
- EINTR            在请求事件发生前收到了一个信号，可以重新调用。
- EINVAL          nfds参数超过了RLIMIT_NOFILE值
- ENOMEM      没有足够的内存完成请求。

### ppoll



### poll和select的对比

尽管它们完成一样的工作，但poll0系统调用仍然优于selectO:

- poll()无需使用者计算最大的文件描述符值加一和传递该参数。

- poll()在应对较大值的文件描述符时更具效率。想像一下用selectO监视值为900的文件描述符——内核需要检查每个集合中的每个比特位，直到第九百个。

- select()的文件描述符集合是静态大小的，所以要作出权衡：要么集合很小，限制了select()可以监视的文件描述符的最大值，要么较大，但是效率不高。尤其是当不能确定集合的组成是否稀疏时，对较大位掩码的操作效率不高。使用poll0则可以创建合适大小的数组。只需要监视一项或仅仅传递一个结构体。

- 若用select(),文件描述符集合会在返回时重新创建，这样的话之后每个调用都必须重新初始化它们。poll()系统调用分离了输入（events字段）和输出（revents字段）,数组无需改变即而重用。

- select()的timeout参数在返回时是未定义的。可移植的代码需要重新初始化它。然而pselect()没有这个问题。

但是select()系统调用的确有几个不错的地方：

- 由于某些Unix系统不支持poll(),所以select()的可移植性更好。

- select()提供了更好的超时方案：直到微秒级。ppoll()和pselect()在理论上都提供纳秒级的精度，但在实际中，没有任何调用可以可靠的提供哪怕是微秒级的精度。



















[^1]: 字符设备的数据组织方式是先进先出的线性队列，由于不支持随机访问，故而无法支持随机读写，即文件的位置指针不能随意改变
[^2]: 将数据写会到磁盘上
[^3]: 文件链接的相关信息包括该文件的目录信息

## 计算机专业单词

synchronization  同步

asynchronization 异步
