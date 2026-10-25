
# 对一个用 Win32Asm 编写的 TCP 端口扫描程序的分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-678.htm`](https://www.pediy.com/kssd/pediy08/pediy8-678.htm)

作者：`jhkdiy`  
邮件：`jhkdiy_gzb@21cn.net`  
论坛：`www.20cn.net`  
日期：06 年 3 月 24 日

有一段时间没有去过 `ASMCommunity Messageboard`（`http://www.asmcommunity.net/board/index.php`）了，以前直接可以访问。现在只能通过代理才能访问，至少我这边是这样。在里面随便逛了逛，无意看到一位论坛会员问怎样用 `Asm` 来做一个端口扫描器，结果其他会员给了一个源代码，在 `window` 的控制台窗口下运行，以阻塞模式扫描一个目标主机的端口。我觉得这个代码对刚学习 `Windows Socket` 编程的朋友来说很有启发性，所以现在详细讲解一下这份代码，希望对大家有所帮助。若各位读者想先试试该程序可以先用 `RadAsm` 建立一个 `console` 工程，然后将下面的代码粘贴到 `asm` 文件中，编译＆链接。在控制台下运行该程序即可。

源代码如下，只做了一点的格式编排：

```
; Conscan - CLI "test"
.386 
.model flat, stdcall 
option casemap :none

include windows.inc 
include user32.inc 
include kernel32.inc 
include masm32.inc
include wsock32.inc
includelib user32.lib 
includelib kernel32.lib 
includelib masm32.lib
includelib wsock32.lib

print macro lpszText:VARARG
 local txt
 .data
  txt db lpszText,13,10,0
 .code
  invoke StdOut,addr txt
ENDM

SOCKADDR_IN struct
 sin_family WORD ?
 sin_port WORD ?
 sin_addr DWORD ?
 sin_zero BYTE 8 dup (?)
SOCKADDR_IN ends

.data
 szusg db "usage: conscan <hostname>",13,10,0
 fmt db "%d OPEN",13,10,0
 sa SOCKADDR_IN <>
 wsa WSADATA <>
 sfd dd 0
 port dd 0
 pl dd 21
 dd 22
 dd 23
 dd 25
 dd 80
 dd 137
 dd 350
 dd 8080
 dd 6667
 dd 31337
 dd 0
 .data?
 hostname db 2024 dup (?)
 buffer db 100 dup (?)
 .code
conscan:
 call main
 call ExitProcess

main proc
 invoke GetCL,1,addr hostname
 cmp eax,1
 jne Arg_Error
   
 mov sa.sin_family, AF_INET
 lea edi,pl
Port_Scan_Loop:
 mov eax,[edi]
 cmp eax,0
 je Port_Scan_Complete
 inc edi
 mov port,eax
 invoke WSAStartup,101h,addr wsa
 invoke socket, AF_INET, SOCK_STREAM, 0
 mov sfd,eax
 invoke htons, port
 mov sa.sin_port, ax
 invoke gethostbyname, addr hostname    
 mov eax,[eax+12]
 mov eax,[eax]
 mov eax,[eax] 
 mov sa.sin_addr,eax
 invoke connect,sfd,addr sa,SIZEOF sa
 cmp eax, 0
 jne Port_Closed
 invoke wsprintf,addr buffer,addr fmt,port
 invoke StdOut,addr buffer
Port_Closed:
 invoke closesocket,sfd
 call WSACleanup
 jmp Port_Scan_Loop
Arg_Error:
 invoke StdOut,addr szusg
 ret
Port_Scan_Complete:
 print "-- Scan Complete --"
 ret
main endp
end conscan
```

整份代码才 100 行左右，那么，这短短的 100 行代码到底做了些什么呢？首先请将代码看一遍，如果是似明非明的话就需要继续看下面的解释了。我们就一段段来分析吧，首先是第一部分：

```
.386 
.model flat, stdcall 
option casemap :none
include windows.inc 
include user32.inc 
include kernel32.inc 
include masm32.inc
include wsock32.inc
includelib user32.lib 
includelib kernel32.lib 
includelib masm32.lib
includelib wsock32.lib 
```

开头的部分相信任何一位学 `win32asm` 的朋友都能看懂，`include` 和 `includelib` 的部分确实有需要留意的部分，那就是：

```
include wsock32.inc
includelib wsock32.lib
```

编写 `windows socket` 程序就要包含相应的 `windows socket` 头文件和库文件，当前可以使用的 `winsock` 接口动态链接库版本有 1.1 版本和 2.0 版本。`wsock32.inc` ＆ `wsock32.lib` 文件表示使用 32 位的 1.1 版本的 `winsock`，如果是使用 2.0 的 `winsock` 就需要使用 `WS2_32.inc` 和 `WS2_32.lib` 文件。如果对 `masm32.inc` 也不熟悉的话，在这里简单的说说，它是一系列常用的宏和函数的集合，主要是为了编程方便，减少重复编写同样的代码而做的，包含了许多实用的宏和函数，在这个代码中 `StdOut` 等函数就使用了这个文件。

再往下看，看到了一个宏：

```
print macro lpszText:VARARG
 local txt
 .data
  txt db lpszText,13,10,0
 .code
  invoke StdOut,addr txt
ENDM
```

该宏即使用了 `masm32.inc` 中的 `StdOut` 函数，该函数在 `windows` 标准控制台下输出一段文本，有点像 C 语言下的 `printf()` 函数。紧跟着看到一段结构体定义：

```
SOCKADDR_IN struct
 sin_family WORD ?
 sin_port WORD ?
 sin_addr DWORD ?
 sin_zero BYTE 8 dup (?)
SOCKADDR_IN ends
```

该结构用来将套接字绑定到 IP 地址和端口使用，当然在后续的接受和发送等等的工作中都会用到。各字段注释如下：

```
SOCKADDR_IN struct
 sin_family WORD ?            ;地址格式
 sin_port WORD ?            ;端口号（使用网络字节顺序）
 sin_addr DWORD ?          ;IP 地址（使用网络字节顺序）
 sin_zero BYTE 8 dup (?)      ;空字节
SOCKADDR_IN ends
```

`sin_family` 在不同的操作系统下有不同的值，但 `winsock` 环境一般都固定为 `AF_INET`，也可以是 `PF_INET`，在 `windows.inc` 中被定义为与 `AF_INET` 同样的值。`sin_port` 和 `sin_addr` 字段分别指定端口号和 IP 地址，放入这两个字段的数据字节顺序必须是 Internet 顺序。这到底是怎么一回事呢？学过汇编语言的朋友都知道，`Intel` 的 `CPU` 对内存操作数使用的是小尾顺序，这种方式是将位数高的字节排在内存地址高的位置，而位数低的字节排在内存的低地址处，例如 `12345678h`，如果以字类型来存储，在内存表示为 `78h 56h 34h 12h`。如果以双字来存储，表示为 `5678h 1234h`。而其它的 `CPU` 如 `Motorola` 的 `CPU` 使用的是大尾顺序，这种顺序刚好相反，低位数排在内存的高位，高位数排在内存的低位，例如 `12345678h` 表示为 `12h 34h 56h 78h`，这种方式比较适合我们一般的看书习惯，都是从左到右一路看过去。

本来这些东东不关我们事的，但是当两种不同 `CPU` 的主机通讯时就不行了，而 Internet 协议是一组开放的协议，需要在不同的计算机平台之间进行通信，所以在实现时不能包含与特定平台有关的东西，即做到平台无关性。这就需要采用一种共同的数据排列顺序才行，而 Internet 协议采用了大尾顺序，所以凡是 `80X86` 平台的 `winsock` 编程，当使用 IP 地址和端口号等参数时，都需要将它们转换为大尾顺序。

好了，到了数据定义的内容了，我们看看都定义了什么：

```
.data
 szusg db "usage: conscan <hostname>",13,10,0       ;使用帮助
 fmt db "%d OPEN",13,10,0               ;输出时显示打开的端口
 sa SOCKADDR_IN <>                   ;使用 connect 函数时用到
 wsa WSADATA <>                  ;使用 WSAStartup 函数时用到
 sfd dd 0                        ;保存 Socket 句柄
 port dd 0                       ;保存要扫描的端口
 pl dd 21                  ;这是程序将要扫描的端口列表
 dd 22
 dd 23
 dd 25
 dd 80
 dd 137
 dd 350
 dd 8080
 dd 6667
 dd 31337
 dd 0
.data?
 hostname db 2024 dup (?)      ;用于存贮域名
 buffer db 100 dup (?)    
```

在 `.data` 段中，值得注意的是定义了一个 `SOCKADDR_IN` 类型的 `sa` 变量，还有一个就是 `WSADATA` 型的 `wsa` 变量。`WSADATA` 是用来初始化 `winsock` 库的，在使用 `winsock` 函数之前必须先使用 `WSAStartup` 函数来装入并初始化动态连接库 `WS2_32.dll`。例如在代码段中的 `invoke WSAStartup,101h,addr wsa` 即表示使用 1.1 版本的 socket 并初始化。

在端口列表 `pl` 方面，这种直接定义在程序内部的方式缺少了灵活性，如果能将常扫描的端口存入一个文件里，需要扫描的时候才读入端口列表来扫描就好多了，注意，最后一个 0 是不能缺少的，实际上它表示端口列表的终止，而并不参加扫描，这种思想和 C 语言的字符串结构一样，不错。

好了，现在看看最重要的代码段了，看看程序到底在做什么了：

```
.code
conscan:
 call main           ;主函数，全部流程都在这里了。
 call ExitProcess     ;正常终止程序必须的。

main proc
 invoke GetCL,1,addr hostname       ;GetCL 是 masm32.inc 中函数
 cmp eax,1
 jne Arg_Error
   
 mov sa.sin_family, AF_INET
 lea edi,pl
Port_Scan_Loop:
 mov eax,[edi]
 cmp eax,0
 je Port_Scan_Complete
 inc edi
 mov port,eax
 invoke WSAStartup,101h,addr wsa
 invoke socket, AF_INET, SOCK_STREAM, 0
 mov sfd,eax
 invoke htons, port
 mov sa.sin_port, ax
 invoke gethostbyname, addr hostname    
 mov eax,[eax+12]
 mov eax,[eax]
 mov eax,[eax] 
 mov sa.sin_addr,eax
 invoke connect,sfd,addr sa,SIZEOF sa
 cmp eax, 0
 jne Port_Closed
 invoke wsprintf,addr buffer,addr fmt,port
 invoke StdOut,addr buffer
Port_Closed:
 invoke closesocket,sfd
 call WSACleanup
 jmp Port_Scan_Loop
Arg_Error:
 invoke StdOut,addr szusg
 ret
Port_Scan_Complete:
 print "-- Scan Complete --"
 ret
main endp
end conscan
```

整个代码段定义了一个 `main` 函数，从 `end conscan` 来看程序的入口点就是 `main` 函数的第一条指令了，先看看：

```
 invoke GetCL,1,addr hostname       
 cmp eax,1
 jne Arg_Error
 ......
Arg_Error:
 invoke StdOut,addr szusg
 ret
```

第一个 `GetCL` 函数的功能是获取命令行中的第一个参数，要注意的是第 0 个参数就是程序名本身，这一点和 C 语言中的程序参数一样。如果函数成功获取参数，则将参数字符串填充到 `hostname` 参数中，这个参数至少要有 128 字节的空间。并在 `eax` 中返回 1 表示执行成功，详细介绍请看 `masm32\help\masm32lib.hlp`。整段代码就是获取第一个参数，如果不成功则转到 `Arg_Error:` 中执行 `StdOut` 函数，输出程序的使用帮助。

现在最重要的来了：



#### 代码示例

```asm
    mov sa.sin_family, AF_INET
    lea edi,pl
Port_Scan_Loop:
    mov eax,[edi]
    cmp eax,0
    je Port_Scan_Complete
    inc edi
    mov port,eax
    invoke WSAStartup,101h,addr wsa
    invoke socket, AF_INET, SOCK_STREAM, 0
    mov sfd,eax
    invoke htons, port
    mov sa.sin_port, ax
    invoke gethostbyname, addr hostname    
    mov eax,[eax+12]
    mov eax,[eax]
    mov eax,[eax] 
    mov sa.sin_addr,eax
    invoke connect,sfd,addr sa,SIZEOF sa
    cmp eax, 0
    jne Port_Closed
    invoke wsprintf,addr buffer,addr fmt,port
    invoke StdOut,addr buffer
Port_Closed:
    invoke closesocket,sfd
    call WSACleanup
    jmp Port_Scan_Loop
Arg_Error:
    invoke StdOut,addr szusg
    ret
Port_Scan_Complete:
    print "-- Scan Complete --"
    ret
```

`mov sa.sin_family, AF_INET`  

`lea edi,pl`  

首先填充 `sa` 变量，`sin_family` 表示使用的协议族，这个没的说，一般都是 `AF_INET`。第二句是将 `pl` 数组的首地址给 `edi`，代表 `edi` 指向要扫描的端口列表。  

`Port_Scan_Loop:`  

`mov eax,[edi]`  

`cmp eax,0`  

`je Port_Scan_Complete`  

`inc edi`  

`mov port,eax`  

扫描循环开始了，首先将第一个要扫描的端口给 `eax`，立即比较 `eax` 是否为 `0`，如果为 `0` 则表示所有要扫描的端口都扫描过了，是结束扫描的时候了，如果真为 `0` 则跳到 `Port_Scan_Complete:` 下打印 `"-- Scan Complete --"` 信息后就退出整个程序。如果不为 `0` 则将 `edi` 加 `1`，以便指向下一个要扫描的端口。然后将端口号存入 `port` 变量中。  

1: `invoke WSAStartup,101h,addr wsa`  
2: `invoke socket, AF_INET, SOCK_STREAM, 0`  
3: `mov sfd,eax`  
4: `invoke htons, port`  
5: `mov sa.sin_port, ax`  
6: `invoke gethostbyname, addr hostname`  
7: `mov eax,[eax+12]`  
8: `mov eax,[eax]`  
9: `mov eax,[eax]`  
10: `mov sa.sin_addr,eax`  

第一行用来初始化动态链接库，`101h` 表明使用 1.1 的 `winsock`。第二行建立一个新的套接字，协议为 `AF_INET`，方式为 `SOCK_STREAM`，即为流套接字，这种套接字使用在 `TCP` 协议中，如果是使用 `UDP` 协议则应该用 `SOCK_DGRAM`。该函数的第三个参数为 `0`，这个参数是表示需要使用的协议类型，与第二个参数配合使用，因为 `SOCK_STREAM` 已经表示使用 `TCP` 协议了，所以这里设置为 `0`。同样 `SOCK_DGRAM` 表示使用 `UDP` 协议，也可将该参数设置为 `0`。`socket` 函数成功执行后返回一个 `socket` 句柄，以后要发送或接受数据都要靠它作为身份表示了。第三行即将 `socket` 句柄保存到 `sfd` 中。  

第四行 `htons` 函数是将 `port` 变量的内容由小尾顺序转换为 Internet 上使用的大尾顺序。第五行即将返回的端口填入 `sa` 的 `sin_port` 字段中，第六行使用 `gethostbyname` 函数将域名转换为 IP 地址，该函数返回指向位于 `winsock` 内部缓冲区中的一个 `hostent` 结构指针。结构体如下：  

```asm
hostent   STRUCT
    h_name  dword      ?    ;指针，指向和 IP 地址对应的主机名
    h_alias dword      ?    ;指针，指向一个包含别名指针的列表
    h_addr  word       ?    ;返回的 IP 地址类型
    h_len   word       ?    ;每个地址的长度
    h_list  dword      ?    ;指向一个指针列表，列表中保存指向各个 IP 地址的指针
hostent   ENDS
```

现在再看看代码，第六行执行完 `gethostbyname` 后返回一个指针，第七行的 `[eax+12]` 表示指向 `hostent` 结构体的 `h_list` 字段然后作为地址寻找 `h_list` 指向的内容，执行完第八行后，`eax` 即指向了一个指针列表，这个列表里的指针指向由 `gethostbyname` 那里得到的各个 IP 地址，因为一个域名对应多个 IP 地址是允许的。第九行又将 `eax` 作为地址，指向完第九行后 `eax` 里已经是实际返回的 IP 地址了。这时得到的 IP 地址已经是大尾顺序，可以直接使用而不需要转换。所以第十行代码直接将 IP 地址填入 `sa` 的 `sin_addr` 字段。  

> 可以表示为：
>
> ```text
>             (hostent 结构) (IP 地址指针列表)     (IP 地址)
>         gethostbyname 的返回值----> h_name               |                   |   
>                                   h_alias             |                   |   
>                                   h_addr              |                   |   
>                                   h_len                |                   |   
>                                   h_list -----> IP 地址 1 的指针 -----> IP 地址 1    
>                                               IP 地址 2 的指针 -----> IP 地址 2 
>                                               IP 地址 3 的指针 -----> IP 地址 3 
>                                                   .....   -----> IP 地址 n 
> ```

最后来看看这段代码：  

> ```asm
>  1: invoke connect,sfd,addr sa,SIZEOF sa
>  2: cmp eax, 0
>  3: jne Port_Closed
>  4: invoke wsprintf,addr buffer,addr fmt,port
>  5: invoke StdOut,addr buffer
>  6: Port_Closed:
>  7:     invoke closesocket,sfd
>  8:     call WSACleanup
>  9:     jmp Port_Scan_Loop
> ```

第一行用 `connect` 与 `sfd` 指定的套接字建立连接，而目标 IP 和端口在 `sa` 中指定。如果 `eax` 返回值为 `0` 表示目标 IP 的这个端口关闭了，跳到第七行，用 `closesocket` 函数关闭一个 `socket`，然后用 `WSACleanup` 释放 `winsock` 库，这代表不再使用 `winsock` 函数了。如果 `eax` 返回值不为 `0` 表示成功和目标 IP 的指定端口建立了连接，这时 `4` `5` 行的代码即在标准控制台窗口输出该端口已打开的信息。而第 `9` 行的代码又重新跳到开始处继续下一个端口的扫描，如此循环直到端口列表中端口号为 `0`。  

好了，整段代码解释完了，好像看似 `100` 多行的代码却隐含了如此多的知识，确实，直接用 `winsock` 函数编程属于较底层的编程了，最后来总结一下整个程序的流程吧：  

1. 先获取一个需要扫描的端口  
2. 如果端口号为 `0` 则跳到第 `11` 步  
3. 如果端口号不为 `0` 则做下一步    
4. 用 `WSAStartup` 初始化 `winsock` 库  
5. 用 `socket` 函数建立一个套接字  
6. 填充 `SOCKADDR_IN` 结构，用来指定套接字使用的 IP 地址＆端口号  
7. 用 `connect` 函数来连接一个套接字，真正连接到目标 IP 地址的指定端口  
8. 如果成功连接则输出端口已打开的信息  
9. 如果不成功则关闭当前这个 `socket` 和释放 `winsock` 库  
10. 跳到第 `1` 步骤继续执行  
11. 结束程序  

整个程序分析完了，但还要说几句话，实际上这个扫描程序的功能很有限，只能扫描一个 IP 的几个常用端口，而且是在控制台下，扫描之前也没有对这个 IP 地址进行 `Ping` 探测。有兴趣的朋友可以深入到这方面，要是能做到像 `SuperScan` 那样强大就好了。另一个方面是这份代码没有任何注释，大家都知道汇编代码很难阅读，而注释和格式编排是唯一可以增加可读性的方法了，而这份代码这两样都没有做好，也不知道这位外国朋友本身的技术如何。总之我是不提倡这种编程风格的，希望各位学 `winasm` 的朋友不要学这种方式。  

如果这篇文章有诉说错误或不恰当的地方请各位朋友批评指正。  

——`jhkdiy`
