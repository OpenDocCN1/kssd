
`void test()`  `{`  `004010B0`  `push        ebp`  `004010B1`  `mov         ebp,esp`  `004010B3`  `push` `0FFFFFFFFh`  `004010B5`  `push` `48AE08h`  `004010BA`  `push        offset _except_handler3 (``04029A8h``)`  `004010BF`  `mov         eax,dword ptr fs:[``00000000h``]`  `004010C5`  `push        eax`  `004010C6`  `mov         dword ptr fs:[``0``],esp`  `004010CD`  `add         esp,``0FFFFFFB8h`  `004010D0`  `push        ebx`  `004010D1`  `push        esi`  `004010D2`  `push        edi`  `004010D3`  `mov         ecx,offset _D3472036_test@cpp (``048F012h``)`  `004010D8`  `call        __CheckForDebuggerJustMyCode (``0401210h``)`  `__try`  `004010DD`  `mov         dword ptr [ebp``-``4``],``0`          `/``/` `进入``try``块，trylevel赋值为``0`  `{`  `printf(``"try块执行\n"``);`  `004010E4`  `push        offset string` `"try\xbf\xe9\xd6\xb4\xd0\xd0\n"` `(``04701B0h``)`  `004010E9`  `call        printf (``04011C0h``)`  `004010EE`  `add         esp,``4`  `/``/` `z` `=` `x` `/` `y;`  `}`  `004010F1`  `mov         dword ptr [ebp``-``4``],``0FFFFFFFFh`      `/``/` `离开``try``块，trylevel赋值为``-``1`  `004010F8`  `call        test``+``4Fh` `(``04010FFh``)` `/``/` `调用``finally``块中的代码`  `004010FD`  `jmp         $LN8 (``040110Dh``)`  `__finally`  `{`  `printf(``"finally代码执行\n"``);`  `004010FF`  `push        offset string` `"finally\xb4\xfa\xc2\xeb\xd6\xb4\xd0\xd0\n"` `(``04701BCh``)`  `00401104`  `call        printf (``04011C0h``)`  `00401109`  `add         esp,``4`  `$LN9:`  `0040110C`  `ret`  `}`  `}`  `0040110D`  `mov         ecx,dword ptr [ebp``-``10h``]`  `}`  `}`  `00401110`  `mov         dword ptr fs:[``0``],ecx`  `00401117`  `pop         edi`  `00401118`  `pop         esi`  `00401119`  `pop         ebx`  `0040111A`  `mov         esp,ebp`  `0040111C`  `pop         ebp`  `0040111D`  `ret` |

但是出现异常的时候，是无法执行离开`try`块的代码的。而此时之所以`finally`块的代码会被执行，就依赖于栈展开机制。

此时查看`scopetable`数组的值，可以看到此时的注册表达式的起始地址为0，异常处理块的起始地址变为`finally`块的起始地址。此时异常处理函数在执行代码时，根据注册表达式的起始地址为0得知，此时需要将第三个参数作为`finally`块的地址进行调用。

```
0x0048AE08  ff ff ff ff
0x0048AE0C  00 00 00 00
0x0048AE10  ff 10 40 00        // finally块的地址
```

而对于以下的代码，程序在输出异常处理块的内容之前会先输出`finally`块中的内容

```
DWORD x = 1900, y = 0, z = 0;
void test()
{
__try
{
__try
{
z = x / y;
}
__finally
{
printf("finally代码执行\n");
}
}
__except (EXCEPTION_EXECUTE_HANDLER)
{
printf("except代码执行\n");
}
}
```

代码的反汇编结果就是异常处理块和终结处理块的组合

```
void test()
{
00401000  push        ebp
00401001  mov         ebp,esp
00401003  push 0FFFFFFFFh
00401005  push 48ADF8h
0040100A  push        offset _except_handler3 (0402938h)
0040100F  mov         eax,dword ptr fs:[00000000h]
00401015  push        eax
00401016  mov         dword ptr fs:[0],esp
0040101D  add         esp,0FFFFFFB8h
00401020  push        ebx
00401021  push        esi
00401022  push        edi
00401023  mov         dword ptr [ebp-18h],esp
00401026  mov         ecx,offset _D3472036_test@cpp (048F012h)
0040102B  call        __CheckForDebuggerJustMyCode (04011A0h)
__try
00401030  mov         dword ptr [ebp-4],0          // 进入try块，将trylevel赋值为0
{
__try
00401037  mov         dword ptr [ebp-4],1          // 进入try块，将trylevel赋值为1
{
z = x / y;
0040103E  mov         eax,dword ptr [x (048D000h)] // try块中的代码
00401043  xor         edx,edx
00401045  div         eax,dword ptr [y (048D910h)]
0040104B  mov         dword ptr [z (048D914h)],eax
}
00401050  mov         dword ptr [ebp-4],0      // 离开try块，将trylevel赋值为0
00401057  call        test+5Eh (040105Eh) // 执行finally块中的代码
0040105C  jmp         test+6Ch (040106Ch)
__finally
{
printf("finally代码执行\n");
0040105E  push        offset string "finally\xb4\xfa\xc2\xeb\xd6\xb4\xd0\xd0\n" (04701B0h) // finally块中的代码
00401063  call        printf (0401150h)
00401068  add         esp,4
0040106B  ret
}
}
0040106C  mov         dword ptr [ebp-4],0FFFFFFFFh
00401073  jmp         $LN7+17h (0401090h)
__except (EXCEPTION_EXECUTE_HANDLER)
00401075  mov         eax, 1              // 条件过滤表达式
00401078  ret
00401079  mov         esp,dword ptr [ebp-18h]
{
printf("except代码执行\n"); // except块中的内容
0040107C  push        offset string "except\xb4\xfa\xc2\xeb\xd6\xb4\xd0\xd0\n" (04701C4h)
00401081  call        printf (0401150h)
00401086  add         esp,4
}
}
00401089  mov         dword ptr [ebp-4],0FFFFFFFFh
}
}
00401090  mov         ecx,dword ptr [ebp-10h]
00401093  mov         dword ptr fs:[0],ecx
0040109A  pop         edi
0040109B  pop         esi
0040109C  pop         ebx
0040109D  mov         esp,ebp
0040109F  pop         ebp
004010A0  ret
```

此时查看`socpetable`数组的值，可以看到外层`try`块的元素内容没有变化，而内层`try`块的条件过滤表达式变为0，异常处理块地址也变成了`finally`块的地址。因此，异常处理函数在执行的时候，就可以根据条件过滤表达式是否为0来判断要不要执行第三个成员保存的函数地址。

```
0x0048ADF8  ff ff ff ff  // 第一个块的previoustrylevel
0x0048ADFC  75 10 40 00  // 第一个块条件过滤表达式地址
0x0048AE00  79 10 40 00  // 第一个块异常处理地址
0x0048AE04  00 00 00 00  // 第二个块的previoustrylevel
0x0048AE08  00 00 00 00  // 第二个块条件过滤表达式
0x0048AE0C  5e 10 40 00  // 第二个块finally块的地址
```

异常处理函数就通过这种方式实现了在执行`except`块的代码之前先执行`finally`块的代码，结果如下：

![avatar](https://bbs.kanxue.com/upload/attach/202112/835440_MCEFXBU9CNSMKQU.png)

# 六.未处理异常

一个新建进程的起始地址其实并不是PE文件中的`AddressOfEntryPoint`字段的偏移地址，而是`kernel32.dll`中`BaseProcessStart`函数，该函数的伪代码如下：

```
void BaseProcessStart(PPROCESS_START_ROUTINE lpfnEntryPoint)
{
__try
{
NtSetInformationThread(GetCurrentThread(),
ThreadQuerySetWin32StartAddress,
&lpfnEntryPoint, sizeof(lpfnEntryPoint));
ExitThread((lpfnEntryPoint)());
}
__except(UnhandledExceptionFilter(GetExceptionInformation()))
{
ExitThread(GetExceptionCode());
}
}
```

根据以上伪代码可以知道，在应用程序的入口函数被调用前，`BaseProcessStart`会为其先设置一个结构化异常处理器，它是初始线程中最早注册的异常处理器。因为当分发异常时，系统是从最晚注册的异常处理器来查找的，所以这个最早注册的结构化异常处理器是最后得到处理机会的。这样便保证了只有当应用程序自己设计的代码没有处理异常时，这个默认的结构化异常处理器才会得到处理机会。

以上代码中的`lpfnEntryPoint`指向的就是入口函数。在`try`块中，根据函数的返回值作为参数来调用`ExitThread`函数退出线程。而如果执行到了异常处理块中，则会将`GetExceptionCode`函数的返回值作为参数调用`ExitThread`函数。

当程序出现异常的时候，是否要执行异常块中的代码则是由函数`UnhandledExceptionFilter`决定的，该函数的执行流程如下：

1. 通过`NtQueryInformationProcess`查询当前进程是否被调试。如果正在被调试，函数将会返回`EXCEPTION_CONTINUE_SEARCH`，随后调用`ZwRaiseException`来进行第二次异常分发

2. 如果当前进程没有被调试：

   - 查询是否通过`SetUnhandlerExceptionFilter`注册了处理函数，如果已经注册了，那么就调用已注册的处理函数

   - 如果没有注册处理函数，就会弹出窗口让用户选择是终止程序还是启动即时调试器。如果用户启动了即时调试器，接下来就会由调试器来接管进程，函数就会返回`EXCEPTION_EXECUTION_HANDLER`

# 七.参考资料

> - 《软件调试（第二版）》卷一
> - 《软件调试（第二版）》卷二

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2021-12-31 21:26

被1900编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
