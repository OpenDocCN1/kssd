
```c
reg_t f_hookpre(file_t outf, dr_mcontext_t *mc, reg_t ret_addr)
{
for (reg_t i = 1; i <= 8; i++)
{
my_fprintf(outf, "[hook]:get arg_%d: "PIFMT" \n", i,my_getarg(mc, i));
}
//调用write函数也可以写入记录到trace日志文件，效果和my_fprintf一致
write(outf, "[hook]modify arg...\n", strlen("[hook]modify arg...\n"));
for (reg_t i = 1; i <= 8; i++)
{
my_fprintf(outf, "[hook]:set arg_%d\n", i);
my_setarg(mc, i, 1);
}
return 0;
}
```

同样可以不依赖`my_getarg`、`my_setarg`等插件模板中已经实现好的函数对寄存器等进行修改（利用`mc`参数）~

## 5. 总结

Windows上的调试分析工具很多且均比较完善了，Linux上`gdb`调试一些多进程程序都稍微麻烦点，另外稍微特殊的程序如利用错误section头部反调试的，虽然也可以`gdb`附加，但是使用`hutrace`来搞都没啥问题，特殊情况下`hutrace`无法运行的程序就要靠接下来介绍的`hzytrace`了，详见下一页基于`pin`实现的`hzytrace`具体介绍（https://bbs.pediy.com/thread-273188.htm）。

github地址：<mark class="encrypted">f95K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5N6h3S2#2x3o6M7H3y4W2)9J5c8X3S2#2N6s2u0S2j5$3f1`.</mark>

> 目录

> [1、hutrace工具系列第1篇之基本功能介绍](https://bbs.pediy.com/thread-273185.htm)

> [2、hutrace工具系列第2篇之Windows应用](https://bbs.pediy.com/thread-273186.htm)

> [3、hutrace工具系列第3篇之Linux应用](https://bbs.pediy.com/thread-273187.htm)

> [4、hutrace工具系列第4篇之衍生工具hzytrace](https://bbs.pediy.com/thread-273188.htm)

![](https://bbs.kanxue.com/upload/attach/202206/543033_WTUVZEUNY2KKH8C.png)

>     Linux上常用程序记录工具有`strace`、`ltrace`等，能够记录到程序的系统调用以及API函数调用信息，但是均无法得到进一步的调用位置等，`hutrace`则同时利用dynamorio实现了基本块以及指令状态等的记录，功能与Windows平台保持一致，本文仅对与win有差异的部分进行简单介绍，且Linux平台能公开的样例较少，内容偏少一点。

## 1. 基本用法

Linux下的`hutrace`使用与Windows基本一致：

```
# huhu @ huhu in ~/Desktop/my/build/bin64 [9:58:27] C:127
$ ./hutrace
ERROR: Usage error:
Usage:
-logdir              [     .]  Log directory to print library call data
-only_from_app       [ false]  Reports only library calls from the app
-trace_to_librarys   [    ""]  Trace specify librarys
-record_start_addr   [    ""]  record begin at start_addr
-record_end_addr     [    ""]  record finished at end_addr
-follow_children     [ false]  Trace child processes
-print_ret_addr      [ false] Print library call's return address
-print_ins_info      [ false] Print instructions disassemble info
-print_ins_reg       [ false] Print instructions disassemble reg
-print_ins_all       [ false] Print instructions trace info
-print_syscall       [ false] Print syscall info
-num_unknown_args    [2]  Number of unknown libcall args to print
-num_max_args        [6]  Maximum number of arguments to print
-config              [    ""]  The path to custom config file.
-ignore_underscore   [ false]  Ignores library routine names starting with "_".
-only_to_lib         [    ""]  Only reports calls to the library <lib_name>.
-help                [ false] Print this message.
-version             [ false] Print version number.
-verbose             [1]  Change verbosity.
-use_config          [  true]  Use config file
```

拿一个最简单的例子来演示，一个比较老的ctf的最简单题，flag解密后直接比较，不过其中夹杂了一些`main`函数等位置cc断点的检测：

```
$ ./hutrace -only_from_app -print_ret_addr -print_ins_info -- ./noccbytes
```

查看trace得到的API记录：

```
C7ffff3d8dbb7|7ffff3d8d8a0
7ffff3d8d8a0: jmp    <rel> qword ptr [0x00007ffff3f8ef98]
J7ffff3d8d8a0|7ffff6829a50
~~5820~~ libc.so.6!strcpy N:0x42
arg 1: WattoSays (type=char*, size=0x0)
and return to module id:4, retraddr:0x7ffff3d8dbbc,offset:0xbbc
7ffff3d8dbbc: lea    rdx, [rbp-0x30]
mov    rax, qword ptr [rbp-0x38]
mov    rsi, rdx
mov    rdi, rax
call 0x00007ffff3d8d8d0
and strcpy return value: 0x7fffffffda20
arg 0: WattoSays (type=char*, size=0x0)
C7ffff3d8dbca|7ffff3d8d8d0
7ffff3d8d8d0: jmp    <rel> qword ptr [0x00007ffff3f8efb0]
J7ffff3d8d8d0|7ffff68235f0
~~5820~~ libc.so.6!strcmp N:0x43                <----最终解密的password在此比较
arg 0: 1234567890 (type=char*, size=0x0)
arg 1: WattoSays (type=char*, size=0x0)
and return to module id:4, retraddr:0x7ffff3d8dbcf,offset:0xbcf
7ffff3d8dbcf: test   eax, eax
jnz 0x00007ffff3d8dbda
and strcmp return value: 0xffffffda
7ffff3d8dbda: lea    rdi, <rel> [0x00007ffff3d8de19]
call 0x00007ffff3d8d8f0
C7ffff3d8dbe1|7ffff3d8d8f0
J7ffff3d8d8f0|7ffff67f36a0
~~5820~~ libc.so.6!puts N:0x44
arg 0: Wrong Password (type=char*, size=0x0)
and return to module id:4, retraddr:0x7ffff3d8dbe6,offset:0xbe6
```

当然`ltrace`也可以做到哦~只是`hutrace`得到的信息更为丰富。

## 2. 多进程分析

Linux下进程创建方式比较多样，而Fork又有别于常规的进程创建方式，导致`hutrace`需要额外处理Linux上常见的Fork进程的记录问题，当然处理难度不大，来看下最简单的演示程序：

```
result = fork();
if(result == -1)
{
    printf("Fork error\n");
}
else if (result == 0)
{
    printf("The returned value is %d In child process!! My PID is %d\n",result,getpid());
}
else
{
    printf("The returned value is %d In father process!! My PID is %d\n",result,getpid());
}
```

开工：

```
$ ./hutrace -only_from_app -print_ret_addr -print_ins_info -- ./myproctest.out
The returned value is 7774 In father process!! My PID is 7773
The returned value is 0 In child process!! My PID is 7774
```

我们只看下子进程的记录日志：

![](https://bbs.kanxue.com/upload/attach/202206/543033_WK26U6JDYMGXYHF.png)

常规方式的进程创建没啥特别的，不再说了，可惜Linux上dynamorio不支持Windows上的`-syswide_on`功能。

## 3. Linux Syscall调用记录

上一篇中介绍的windows系统调用使用常见相对小众，Linux下就比较常见了，`hutrace`在linux系统上同样默认不记录syscall调用信息，而且不在`hutrace.config`中设置参数信息的话只打印系统调用名称，只有设置了参数信息才会打印系统调用的参数，实际使用时和打印API函数以及上一篇Window应用中介绍的使用方式类似，在`hutrace.config`配置文件中添加对应syscall的参数信息：

```
hutrace.config文件中设置：
int|syscall_read|LONG|__out hex^ARG2|int
int|syscall_write|LONG|char *
int|syscall_open|char *|LONG

运行hutrace记录ls程序运行的syscall：
# huhu @ huhu in ~/Desktop/my/build/bin64 [8:17:52]
$ ./hutrace -only_from_app -print_ret_addr -print_syscall -- /bin/ls -al
```

得到的部分日志记录：

