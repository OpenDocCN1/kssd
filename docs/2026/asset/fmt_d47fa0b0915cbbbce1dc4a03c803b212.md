
# [原创]CTF2017-003

系统环境：Microsoft Windows XP Professional 版本 2002 Service Pack 3（新萝卜城家园GHOST XP SP3 电脑城装机版 V2014.04 32位）

使用工具：Ollydbg（吾爱破解专版）

第一次在看雪发帖，写得不好，献丑了。。。

## 0x0 anti-anti-debug

`OD`载入，输入假码，发现程序退出。

```
004313F2 . 33C0 xor eax,eax
004313F4 F7F0 div eax ; 终止
```

看上面的代码发现程序使用了`kernel32!SetUnhandledExceptionFilter`来进行反调试。

直接给`kernel32!UnhandledExceptionFilter`下端点，单步走的时候发现程序有使用`ntdll!NtQueryInformationProcess`来查询`DebugPort`，和`0`比较，如果相等则处理异常，否则不处理。

处理方法：

将下面的这段代码中的`je`修改成`jmp`即可。

```
7C864D87 39BD DCFEFFFF cmp dword ptr ss:[ebp-0x124],edi
7C864D8D 0F84 96000000 je kernel32.7C864E2
```

改成

```
7C864D87 39BD DCFEFFFF cmp dword ptr ss:[ebp-0x124],edi
7C864D8D E9 97000000 jmp kernel32.7C864E29
7C864D92 90 nop
```

然后`CTRL+G`输入`KiUserExceptionDispatcher`，给下面的`ZwContinue`下端点。

```
7C92E490 6A 00 push 0x0
7C92E492 51 push ecx
7C92E493 E8 C6EBFFFF call ntdll.ZwContinue ; 在这里下断点
```

`F9`运行，然后查看`ntdll!ZwContinue`的第一个参数（`CONTEXT`结构体），如果了解`CONTEXT`结构体的大致参数的话很容易能找到转移的地址。

最后发现返回地址就是`div eax`指令的下一行，所以改成`nop`然后保存即可。

```
004313F4 90 nop ; 终止
004313F5 90 nop
```

然后`F9`，程序自己退出了，我推测是使用了`kernel32!ExitProcess`函数，没想到真的断下来了。

最后找到调用这个的函数的地址就行了，我的方法比较笨，就是寻找返回地址处的代码的函数头，然后给这个函数头地址下端点，重新载入程序，再输入注册码。这么来几次就能找到了。

最后发现是在`004367C1`处调用的退出函数，上面有一个`jnz`，将这个`jnz`改成`jmp`然后保存即可。

```
004367BF /75 0A jnz short crackMe.004367CB
004367C1 |6A 00 push 0x0
```

改成

```
004367BB |. 837D F8 01 cmp [local.2],0x1
004367BF EB 0A jmp short crackMe.004367CB
```

至此，程序已经可以在`OD`中调试了。

## 0x1 寻找注册码

直接给`GetWindowTextA`下断点，然后随便输入一串假码，点击[验证序列号]。

来到程序领空以后`F8`单步。

`F8`单步走到`00435133`处就无法返回了，不过没关系，给下一条指令下断点，然后直接`F9`执行到断点处即可单步调试了。

然后程序计算了一个字符串：

```
0043513B . C785 78E7FFFF>mov dword ptr ss:[ebp-0x1888],0x0 ; 计次变量置0
00435145 . EB 0F jmp short crackMe2.00435156
00435147 > 8B85 78E7FFFF mov eax,dword ptr ss:[ebp-0x1888]
0043514D . 83C0 01 add eax,0x1
00435150 . 8985 78E7FFFF mov dword ptr ss:[ebp-0x1888],eax ; 计次变量+1
00435156 > 83BD 78E7FFFF>cmp dword ptr ss:[ebp-0x1888],0x20 ; 循环0x20次
0043515D . 7D 2C jge short crackMe2.0043518B
0043515F . 8B85 78E7FFFF mov eax,dword ptr ss:[ebp-0x1888]
00435165 . 0FB68C05 8CEB>movzx ecx,byte ptr ss:[ebp+eax-0x1474] ; 取出被格式化的字节
0043516D . 51 push ecx
0043516E . 68 A4B14800 push crackMe2.0048B1A4 ; %02x
00435173 . 8B95 78E7FFFF mov edx,dword ptr ss:[ebp-0x1888]
00435179 . 8D8455 84E7FF>lea eax,dword ptr ss:[ebp+edx\*2-0x187C]
00435180 . 50 push eax
00435181 . E8 7F8DFFFF call crackMe2.0042DF05 ; 格式化 每次格式化一个字节的字符串
00435186 . 83C4 0C add esp,0xC
00435189 .^ EB BC jmp short crackMe2.00435147
```

其中，被格式化的数据如下：

```
0012E5DC 18 39 20 F0 0E 15 A0 43 3E E3 A8 FC 90 DD 9A C1 9 ?燙>悒鼝輾
0012E5EC 64 C4 14 2C CF 63 CA 18 9A 8F 64 5E C9 6F F8 DE d?,蟘?殢d^蒾
```

最后的结果：

`183920f00e15a0433ee3a8fc90dd9ac164c4142ccf63ca189a8f645ec96ff8de`

然后程序获取了格式化出来的字符串的长度：

```
0043518B > \8D85 84E7FFFF lea eax,dword ptr ss:[ebp-0x187C] ; 格式化后的字符串
00435191 . 50 push eax
00435192 . E8 FD85FFFF call crackMe2.0042D794 ; 获取字符串长度
00435197 . 83C4 04 add esp,0x4
```

又获取了注册码的长度：

```
0043519B . 8D8D D8F7FFFF lea ecx,dword ptr ss:[ebp-0x828] ; 注册码
004351A1 . 51 push ecx
004351A2 . E8 ED85FFFF call crackMe2.0042D794 ; 获取字符串长度
004351A7 . 83C4 04 add esp,0x4
```

然后用一个缓冲区的地址减去格式化后的字符串的长度：

```
004351AA . 8DB405 D8F7FF>lea esi,dword ptr ss:[ebp+eax-0x828] ; 一个缓冲区
004351B1 . 8D95 84E7FFFF lea edx,dword ptr ss:[ebp-0x187C] ; 格式化后的字符串
004351B7 . 52 push edx
004351B8 . E8 D785FFFF call crackMe2.0042D794 ; 获取字符串长度
004351BD . 83C4 04 add esp,0x4
004351C0 . 2BF0 sub esi,eax ; 缓冲区的地址减去格式化后的字符串的长度
```

最后的地址处的数据（记为`data`）如下：

```
0012F1F1 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................
0012F201 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................
0012F211 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 CC ...............
0012F221 CC CC CC CC CC CC CC 31 32 33 34 35 36 37 38 39 烫烫烫?23456789
```

然后和格式化后的字符串比较：

```
004351C2 . 56 push esi ; 减后的地址
004351C3 . 8D85 84E7FFFF lea eax,dword ptr ss:[ebp-0x187C]
004351C9 . 50 push eax ; 格式化后的字符串
004351CA . E8 5889FFFF call crackMe2.0042DB27 ; 字符串比较
004351CF . 83C4 0C add esp,0xC
004351D2 . 85C0 test eax,eax ; 如果返回0则失败(不相等)
004351D4 . 75 3E jnz short crackMe2.00435214
```

查看一下`data`的数据，不难发现`data`中有注册码的数据，据此猜测，注册码的长度应为`0x40`个字节，再结合后面和这段数据比较的字符串，不难猜想出注册码可能是格式化后的字符串，因此用原始程序输入这个格式化后的文本然后点击[验证序列号]尝试一下，最后的结果如下：

![](upload/attach/201710/759802_50zuj23sycg2qs7.png)

因此注册码（即序列号）为：

`183920f00e15a0433ee3a8fc90dd9ac164c4142ccf63ca189a8f645ec96ff8de`
