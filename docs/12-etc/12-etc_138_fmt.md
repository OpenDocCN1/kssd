
# 再谈 linux 反反调试---关于 Gdb Script

> 原文：[`www.pediy.com/kssd/pediy10/77809.html`](https://www.pediy.com/kssd/pediy10/77809.html)

实际上这个文章应该是上一篇文章的续集。

上一篇文章 用 `getppid` 获得父进程的 `pid` 来进行反调试。

我们一直是用手工修改返回来解决问题的。如果一次还可以，如果里面有很多此检查父进程的 `id` 怎么办。或者程序中用了很多不同的方法检测是不是被调试怎么办？会不会把调试变成体力劳动。我们一次性吧所有的反调试都解决掉那不是更好？。

和 `od` 有一个一样的特性，`gdb` 也是支持脚本的，不过他支持的是命令序列。

网上的这样的文章也不多。我在这里就来个 `gdb` 脚本反反调试的实验。

实际上要实现起来也不是很难。`ok！`我们在原来的基础上多加几次反调试的语句。但并没有增加技术上难度。

我在这里体现的不是反调试的方法而只是如何利用 `shell script` 和 `gdb script` 配合进行调试。学习进行中.....

首先还是给出被测试程序的源代码。

代码:

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
  
int get\_name\_by\_pid(pid\_t pid, char\* name)
 {
     int fd;
     char buf\[1024\] = {0};
     snprintf(buf, 1024, "/proc/%d/cmdline", pid);
     if ((fd = open(buf, O\_RDONLY)) == -1)
         return -1;
     read(fd, buf, 1024);
     strncpy(name, buf, 1023);
     return 0;
 }
int check\_user()
{
  char \*password="123",user\[5\]="";
  printf("password:");                   
  scanf("%s",user);
   if(!strcmp(password,user)) 
    {printf("success!\\n");return 0;}        
   else 
    {printf("Error!\\n");return -1;}
}
  
 int main(void)
 {
     
     char name\[1024\];  
     pid\_t ppid = getppid();   
         if (get\_name\_by\_pid(ppid, name))
         return -1;
     
     if (strcmp(name,"bash")==0||strcmp(name,"init")==0)
            { 
              pid\_t ppid = getppid();   
                 if (get\_name\_by\_pid(ppid, name))
                 return -1;
              if (strcmp(name,"bash")==0||strcmp(name,"init")==0)
              { 
                check\_user();
                 }
                 else if (strcmp(name,"gdb")==0 || strcmp(name,"strace")==0 || strcmp(name,"ltrace") == 0)
                     printf("Traced!\\n");
                 else
                     printf("Unknown! Maybe traced!\\n");
               }
     else if (strcmp(name,"gdb")==0 || strcmp(name,"strace")==0 || strcmp(name,"ltrace") == 0)
         printf("Traced!\\n");
     else
         printf("Unknown! Maybe traced!\\n");
  
     return 0;
 }
```

源码没有很大变化，只是在检测 `getppid` 次数上增加一次。用来说明 `GDB script` 的自动调试方法

`gdb` 的脚本的使用方法也很简单直接在 `gdb` 的命令行中输入 `source filename`

脚本的内容

代码:

```
echo  ----------welcome useing MyScript--------------\\n 
echo  --------------Gdb Script ----------------------\\n 
echo  -----------------------------------Beijihu-----\\n
r
b getppid
commands
rep
end
define rep
next
shell echo \`ps -A |grep "bash"\` >./gdb.Temp
shell echo 'print $eax=arr' >./anti.temp
shell sed -i s@arr@\`awk -F' ' '{ print $1}' ./gdb.Temp\`@g ./anti.temp
source ./anti.temp
shell rm ./\*.temp
c
end
```

详细解释一下具体命令的含义：

\---------------------------------beijihu's split-line------------------------------------------------------

`r`          #第一次运行。这个上次说为什么不知道这样做的原因大概是因为第一次运行需要加载器解决程序的共享函数库的重定向的问题

`b getppid`  #在 `getppid` 函数调用下断

`commands`   #定义 `getppid` 断点的调用命令序列

`rep`        #自定义的命令

`end`

`define rep` #定义一个命令序列

`next`       #步过，相当于 OD 的 F8

`shell echo \`ps -A |grep "bash"\` >./gdb.temp`

        #Shell 命令

`shell echo 'print $eax=arr' >./anti.temp`

        #shell 命令临时 建立一个 gbd script

`shell sed -i s@arr@\`awk -F' ' '{ print $1}' ./gdb.temp\`@g ./anti.temp`

        #shell 命令 置换 arr 为 bash 的 pid

`source ./anti.temp`

        #执行临时建的 gdb 脚本。实际上就是一个 `print $eax=pid`

`shell rm ./* .temp`

        #清理现场

`c`           #继续运行

`end`         #命令序列结束

\---------------------------------beijihu's split-line------------------------------------------------------

脚本调用...

代码:

```
(gdb) source anti.gdb
```

运行脚本后以下输出

代码:

```
\----------welcome useing MyScript--------------
\--------------Gdb Script ----------------------
\-----------------------------------Beijihu-----
(no debugging symbols found)
(no debugging symbols found)
(no debugging symbols found)
Traced!

Program exited normally.
Breakpoint 1 at 0xcc6880
```

代码:

```
(gdb) r
```

贴个图吧，这是结果：

![](img/ae828502c343283c6da7f28b9c755a7f.jpg)

大家看图片上面的输出还是有点错误的。这是由于我对 script 的编写方法还是有点不熟悉。但是显而易见它已经起作用了。

有什么更好的编写建议跟帖告知阿......
