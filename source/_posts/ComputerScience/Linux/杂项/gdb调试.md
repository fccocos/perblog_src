# 调试core文件

使用`ulimit -a`查看当前系统受限文件
使用`ulimit -c unlimited`对core文件解除限制
`gdb 二进制文件 core`

指定一个进程来调试一个程序，`gdb program pid` `gdb -p pid`
gdb 常用命令
r run 
l list
b break
bt  查看栈
c continue
n next
e edit [file:]function
s step 跳进或跳出函数 
h help
q quit

# gdb多线程调试

## 调试常用命令

1. `info thread` 查看可切换调试的线程
2. `thread tid` 切换线程
3. `set scheduler -locking on` 只运行当前线程
4. `set scheduler -locking off`运行全部线程
5. `thread apply tid gdb_cmd` 指定线程执行gdb执行
6. `thread apply all gdb_cmd` 全部的线程执行gdb指令

测试用例 thread_test.c

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
int x=0,y=0;
pthread_t th1,th2;

//第一个线程
void* th1_do(void* arg)
{
  for(x=0;x<100;x++)
  {
    printf("x=%d\n",x);
    sleep(1);
  }
  pthread_exit(NULL);
}

//第二个线程
void* th2_do(void* arg)
{
  for(y=0;y<100;y++)
  {
    printf("y=%d\n",y);
    sleep(1);
  }
  pthread_exit(NULL);
}
int main()
{
  //创建线程
  if(pthread_create(&th1,NULL,th1_do,(void*)0)!=0)
  {
     printf("pthread_create th1 failed.\n"); return -1;
  }
  
  if(pthread_create(&th2,NULL,th2_do,(void*)0)!=0)
  {
     printf("pthread_create th2 failed.\n"); return -1;
  }
}
printf("111\n");
pthread_join(th1,NULL);

printf("222\n");
pthread_join(th2,NULL);
```
## 多线成调试流程
1. 生成可执行程序 `gcc -o thread_test -g thread_test.c`
2. 进入调试界面 `gdb thread_test`
3. 查看并切换线程（通过线程号来操作）
```shell
(gdb) b program_line # 打断点
(gdb) r #运行程序到第一个断点处
(gdb) info threads #查看线程号(此时只有主线程号)
(gdb) n #执行创建的子线程函数
(gdb) info threads #查看线程号（此时有两个线程号）
(gdb) n # 继续执行
(gdb) info threads
(gdb) n
```
4. 只运行当前进程
```shell
(gdb) set scheduler-locking on
(gdb) n # 单步调试
```
4. 运行全部线程
```shell
(gdb) set sheduler-locking off
(gdb) n
```
5. 指定线程执行gdb命令
```shell
(gdb) thread apply 2 n # 让子线程2执行gdb的命令next
```

# GDB多进程调试

可以在fork函数调用之前，通过设置GDB调试工具跟踪父进程或跟踪子进程，默认跟踪父进程

## 常用命令
1. `l` 查看代码
2. `b` 加断点
3. `ib` 查看所有断点信息
4. `n` 单步执行
5. `c` 继续执行，直到遇到下一个断点停止运行
6. `show follow-fork-mode` 查看跟踪的是父进程还是子进程
7. `set follow-fork-mode [parent(default)|child]` 设置调试父进程还是子进程
8. `set detach-on-fork [on|off]` 设置调试模式
9. `show datach-on-fork` 查看调试模式
10. `info inferiors` 查看调试信息
11. `inferior id` 切换当前进程
12. `detach inferiors id` 使用进程脱离GDB调试

测试用例`gdbmultipro.c`

```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

static void InsertSort(int *arr)
{
  int* p=NULL;
  int* q=NULL;
  int soldier=0;
  for(p=arr+1; p; ++p)
  {
     int tmp=*p;
     for(q=p-1;q>=arr; --q)
     {
         if(*p>*q)
         {
           *(p+1) = tmp;
           break;
         }
         *p = *(p+1);
     }
  }
}


static void swap(int* a, int* b)
{
  int temp=0;
  temp = *a;
  *a = *b;
  *b = temp;
}

static void BubbleSort(int *arr)
{
  int* p=NULL;
  int* q=NULL;
  int len = 0;
  q=arr;
  while(q++) len++;
  for(q=(arr+len); q>=arr; q--)
  {
    for(p=arr; (p+1)<q; p++)
    {
      if(*p>*(p+1))
      {
        swap(p, p+1);
      }
    }
  }
}

int main()
{
  int mpid,cpid;
  //主进程做插入排序
  //子进程做选择排序
  int sort[]={8,3,43,12,34,534,56,0,123,44,32,34};
  int ret = fork();
  if(ret>0)
  {
    mpid = getpid();
    int ppid = getppid();//获取父进程号
    InsertSort(sort);
    int* i=sort;
    printf("mpid=%d, ppid=%d\n",mpid, ppid);
    for(; i; i++)
      printf("%d\n", *i),sleep(1);
    
  }
  else
  {
    cpid = getpid();
    int ppid = getppid();
    BubbleSort(sort);
    int* i=sort;
    for(;i;i++)
      printf("%d\n", *i),sleep(1);
  }
 return 0;
}

```
## 多进程调试流程

1. 编译程序 `gcc -o gdbmultipro gdbmultipro.c -g `

2. 进入调试终端 `gdb gdbmulitpro`
   gdb默认调试的是主进程，子进程直接结束，父进程停在断点处

3. 让GDB调试子程序
   1. 用`show follow-fork-mode`查看跟踪的是主进程还子进程
   2. 用`set follow-fork-mode child` 切换为跟踪子进程
   3. 用`l`查看子进程代码
   4. 用`b`为子进程程序打断点
   5. 用`ib`查看子进程程序的所有断点
   6. 用`r`命令运行程序，父进程默认结束，子进程运行
   7. 用'n'命令进行单步调试，用's'命令跳入和跳出函数
   
4. 设置调试模式
  1. 先用`show detach-on-fork`查看调试模式
  2. 然后用'set detach-on-fork [on|off]' 来切换调试模式， 默认为on,表示调试当前进程，其他的进程继续运行；off表示，调试当前进程时，其他进程挂起 
  3. 用`info inferiors` 查看调试信息
  4. 用`set inferior id`切换进程进行调试
  5. 用`detach inferiors id` 使进程脱离GDB调试 



