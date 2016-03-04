# 反GDB调试
***

## 1. 文件描述符

* 因为GDB等调试器会打开出了0、1、2之外的文件描述符

```
#include <stdio.h>
#include <unistd.h>
int main(int argc,char *argv[])
{
    if(close(3)==-1)
        printf("No GDB\n");//关闭不成功，说明没有被调试
    else
        printf("Debugged by GDB\n");//关闭成功，说明被调试
    return 0;
}
```

## 2. 进程树

* 如果是正常启动的时候，父进程的进程标识符总是等于会话标识符
* 如果是在调试器下启动，那么两者就不同

```
#include <stdio.h>
#include <unistd.h>
int main(int argc,char *argv[])
{
    if(getppid() != getsid(0) )
        printf("Go away Debugger\n");
    else
        printf("OK\n");
    return 0;
}
```

## 3. 插入INT3

* 大多数的调试器严格控制SIGTRAP信号（trace、breakpoint、trap）
* 如果执行了指令INT 03

  正常状态下，程序会收到SIGTRAP信号，然后调用handler()
  调试状态下，调试器获取控制权，中断程序进行，

* 如果让程序的继续运行，就不会调用handler()函数，只是接着向下执行。
* 在断点后接加密部分，handler解密

```
#include <stdio.h>
#include <unistd.h>
#include <signal.h>
int key='a';
//使用加密部分进行加密，如果正常运行，就可以正常解密
//如果调试，那么无法解密
void handler(int signum)
{
    key-=7;//解密部分
}


int main(int argc,char *argv[])
{
    signal(SIGTRAP,handler);
    printf("Starting......\n");
    key+=7;//加密部分
    __asm__("int3");
    printf("key:%c\n",key);
    return 0;
}
```
## 4. 检测断点
#### 4.1. 第一种方法
* 一般而言，函数在内存中的次序和源代码的次序是一样的
* 就是说，如果源代码在文本的前面，那么函数位于低地址
* 下面的代码通过累加从foo到main之间所有代码的值，构成一个校验码
* _MY_CRC是没有断点时的累加值
* 如果调试时加入了断点，那么得到的就会不同

```
#include <stdio.h>
#define _MY_CRC 0x15aa
int check(void);
void foo(void)
{
    printf("foo\n");
}
void bar(void)
{
    printf("bar\n");
}
int main(int argc,char *argv[])
{
    if(check())
        foo();
    if(check())
    bar();
    return 0;
}
int check(void)
{
    int a=0;
    unsigned char *p;
    for(p=(unsigned char *)foo;p<(unsigned char *)main;p++)
        a += *p;
    printf("a:%x\n",a);
    if(a != _MY_CRC)
    {
        printf("Go away,debugger\n");
        return 1;
    }
    else
    {
        printf("All OK\n");
        return 0;
    }
}
```

#### 4.2. 第二种方法

* 软件断点机器指令INT 03h
* 操作码 CCh
* 绕过方法：将断点代码在执行是全部改成icebp，也可以当作断点

```
#include <stdio.h>
#include <stdlib.h>
void foo()
{
    printf("Hello\n");
}
int main(int argc,char *argv[])
{
    if ((*(volatile unsigned *)((unsigned)foo + 3) & 0xff) == 0xcc) {
        printf("Go away,debugger\n");
        exit(1);
     }
    return 0;
}
```

## 5.ptrace

####5.1.1. 如果已经被追踪，那么调用ptrace必然失败

```
#include <stdio.h>
#include <sys/ptrace.h>
int main(int argc, const char *argv[])
{
    if(ptrace(PTRACE_TRACEME,0,1,0)<0){
        printf("Go away debugger\n");
    }
    return 0;
}
```
####5.1.2. 如何绕过
* 绕过方法一
  
  在ptrace设置断点，跳过ptrace函数
  
  注意要令返回值为0，然后继续运行

```
(gdb)b ptrace
(gdb)pret
(gdb)set $eax=0
(gdb)c
```

* 绕过方法二
   
   构建自己的动态库用来代替实际用的动态库
   

第一步

```
// -- ptrace.c --
int ptrace(int i, int j, int k, int l)
{
printf(" PTRACE CALLED!\n");
}
// -- EOF --
```

第二步

```
$ gcc -shared ptrace.c -o ptrace.so
$  gdb ./antiptrace
gdb> set environment LD_PRELOAD ./ptrace.so
gdb> run
```

#### 5.2. ptrace不能使用两次

* ptrace不能使用两次，因为试图跟踪一个已经被跟踪的进程会出现错误
* 让父进程使用PTRACE_ATTACH
* 子进程使用PTRACE_TRACEME
* 子进程被父进程跟踪
* 并且在父进程中执行一些必要的操作，比如父进程执行一部分解密，子进程再解密一部分，父进程再机密一部分，这样就不能杀死父进程

```
#include <stdio.h>
#include <unistd.h>
#include <sys/ptrace.h>
#include <signal.h>
#include <stdlib.h>
int main(int argc,char *argv[])
{
    pid_t child;
    int status;
    switch((child=fork()))
    {
        case 0:
            ptrace(PTRACE_TRACEME);//子进程让其他进程来追踪自己
            printf("I'm child\n");
            // secret part
            exit(1);
            break;
        case -1:
            printf("ERROR\n");
            break;
        default:
            if(ptrace(PTRACE_ATTACH,child))//父进程追踪子进程
            {
                kill(child,SIGKILL);
                exit(2);
            }
            while(waitpid(child,&status,0) != -1)
                ptrace(PTRACE_CONT,child,0,0);//唤醒子进程
            exit(0);
            break;
    }
    return 0;
}
```

## 6. 命令行参数

####6.1. argv[0]

* 如果正常启动，那么命令行参数是的第一个就是程序的文件名
* 这里检测是否为gdb

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
void gdbbye(void);
int main(void)
{
    gdbbye();
    return 0;
}
void gdbbye(void)
{
    char name[BUFSIZ]="";
    FILE *fp;
    snprintf(name,BUFSIZ,"/proc/%d/cmdline",getppid());
    if((fp = fopen(name,"r")) == NULL){
        name[0]='\0';
        return;
    }
    if(fgets(name,BUFSIZ,fp) == NULL){
        name[0]='\0';
        return;
    }
    if(!strcmp(&name[strlen(name)-3],"gdb")){
        printf("Bye,GDB\n");
        exit(0);
    }
    fclose(fp);
}
```

#### 6.2. $_

* 环境变量叫$_,它保存的是上一个执行的命令的最后一个参数。

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
int main(int argc, const char *argv[])
{
    printf("_:%s\n",getenv("_"));
    printf("argv[0]:%s\n",argv[0]);
    if(strcmp(argv[0],(char *)getenv("_"))){
        printf("Go away debugger\n");
        exit(1);
    }
    return 0;
}
```


## 7. 让objdump失效

* 这种方法就是可以欺骗objdump
* 绕过方法就是动态调试，或者使用高级的返汇编工具

#### 7.1. 第一种方法

* 第一种情况在label中使用的是0x90，代表的是nop
  
  这是一个单字节的操作符，不需要其他的操作数，所以，objdump可以反汇编成功

```
#include <stdio.h>
#include <sys/ptrace.h>
int main(int argc, const char *argv[])
{
__asm__(
         "start:\n"
             "jmp label+1\n"
         "label: .byte  0x90\n"
             "movl $0xf001,%eax\n"
); 
    return 0;
}
```
#### 7.2. 第一种方法
* 第二种情况使用的是0xe9，代表的是jmp

 这个不是一个单字节的操作数，需要其他操作数，objdump会把后面的字节当作jmp的操作数，所以objdump会反汇编失败

```
#include <stdio.h>
#include <sys/ptrace.h>
int main(int argc, const char *argv[])
{
__asm__(
         "start:\n"
             "jmp label+1\n"
         "label: .byte  0xe9\n"
             "movl $0xf001,%eax\n"
); 
    return 0;
}
```
