
# 万能五笔

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-258.htm`](https://www.pediy.com/kssd/pediy08/pediy8-258.htm)

在一个软件已经成品的发布了，BUG 的出现是不可预见的（可以软件发布期缺少测试或本身不可能测试到的），所以撑握成品软件后期 BUG 调试技术是每个软件开发人员必须撑握的（今天看了一些软件开发的书，废话多了点。）

工具：OD （OD 已成为现在 WINDOWS 主流用户级调试工具，不但用来破解方便，调试软件 BUG 也很 OK）

被调试软件：万能五笔 EXE 外挂版

前天看到我深圳健华医药公司的美美用万能五笔打字挺爽的（介面好看而字库大），以前有不少打不出的字现在都能打了，不错（以后改用万能五笔）[要说下今天辞职（深圳健华医药公司）行政部搞了我一天才给我批下来，操他妈的害我坐在那等了一天。]

今天来网吧，前几天想的万能五笔现在下来用，呵下了个绿色版，去掉了 COPYSO 网搜，一开始用还可以，但制作绿色版的作者不负积应，去了链接，但指标还在，看着不爽，害我在他搞的版本上搞了个更好看的，就在这时我发现了个万能五笔的 BUG（我当时打开了一个空白记事本，无意中在 COPYSO 点了下，哈哈，软件被 WINDOWS 中止了），这下好了，我试了下最新原安装版也是如此。好了，故事开始了.....

安装了原版，我试了原版，觉了那个 COPYSO 还真不错，不过失望的是：用的是 COPYSO 搜索引擎，能力大不行，虽然软件提供了引擎选择，不过很不爽，COPYSO 都是默认的，重起软件又到回 COPYSO，小的就大胆的 DIY 了一个万能五笔 baidu 版（最爱 baidu 百度——全球最大中文搜索引擎 中国人的吗！）百度引擎接口：`http://www.baidu.com/s?wd=` 这里就不说怎 DIY 了，呵，用上了百度，可那个 BUG....我哭啊！

为了让万能五笔与 baidu 完美结缘，结定动手去掉它（谁叫这个不起眼 BUG，竟被我发现了，倒霉，知到了有 BUG，就等于知到了这软件的不完美性，中意了没办法），下面开工。。

先说明下：创建进程一舟殳都用到两个函数 `ShellExecute` 和 `WinExec` 它们都是只创建，后期工作就不管了（被创建进程死活就不理了），不过有一个函数：`CreateProcess` 就不同了，在此不多说（我都是在罗云杉的 32 位 AMS 那学的，ASM 真的很有用哦，学破解的同包们，能了解到很多函数）。OK

OD 打开万能五笔运行然后在命令行下：`BP CreateProcessA`

运行那个 BUG 过程，哦程序没断下来，不过出现了异常（如果不下 `BP CreateProcessA` 异常将不会出现，为什么自己看书 `CreateProcess` 回调异常）

调试此程序不可以用键盘，不要问为什么？用了 OD 就会死掉。(用 OD 里的单步按钮)

```
003A004E    C2 0400           retn    4-------------->异常  
003A0051    3A2D 29000000     cmp     ch, [29]  
003A0057    0000              add     [eax], al  
003A0059    0000              add     [eax], al  
003A005B    0000              add     [eax], al  
003A005D    0000              add     [eax], al  
003A005F    0000              add     [eax], al  
```

返回在 KERNEL32.DLL

```
77E53887    5E               pop     esi  
77E53888    C9               leave  
77E53889    C2 1000          retn    10  
77E5388C >  6A 18            push    18  
77E5388E    68 2830E677      push    77E63028  
77E53893    E8 406A0000      call    77E5A2D8  
```

异常交给 WINDOWS 了

```
7801B27B    50               push    eax  
7801B27C    E8 B5FDFFFF      call    RpcRaiseException  
7801B281    CC               int3------------------------->int3 异常中断  
7801B282    56               push    esi  
```

应用程序错误（好像在不可执行代码段执行了代码，你们看下 ASM 之 DLL 那章吧，我说的可能不对）在 NTDLL 提示

```
77FB4DB3    8B1C24           mov     ebx, [esp]  
77FB4DB6    51               push    ecx  
77FB4DB7    53               push    ebx  
77FB4DB8    E8 ACBDFAFF      call    77F60B69---------------->在些出现应用程序错误程序（哈哈，我笑了，不知你地笑了没）  
77FB4DBD    0AC0             or      al, al  
77FB4DBF    74 0C            je      short 77FB4DCD  
```

![](img/e5cce00b52fcf946e1d4fb928e7c4f02.jpg)

哈哈，这就是 WINDOWS 的每力所在，异常调试：点取消调试

```
77E41BBC >  55               push    ebp  ------------->异常回调  
77E41BBD    8BEC             mov     ebp, esp  
77E41BBF    6A 00            push    0  
77E41BC1    FF75 2C          push    dword ptr [ebp+2C]  
77E41BC4    FF75 28          push    dword ptr [ebp+28]  
77E41BC7    FF75 24          push    dword ptr [ebp+24]  
77E41BCA    FF75 20          push    dword ptr [ebp+20]  
77E41BCD    FF75 1C          push    dword ptr [ebp+1C]  
77E41BD0    FF75 18          push    dword ptr [ebp+18]  
77E41BD3    FF75 14          push    dword ptr [ebp+14]  
77E41BD6    FF75 10          push    dword ptr [ebp+10]  
77E41BD9    FF75 0C          push    dword ptr [ebp+C]  
77E41BDC    FF75 08          push    dword ptr [ebp+8]  
77E41BDF    6A 00            push    0  
77E41BE1    E8 20170100      call    CreateProcessInternalA  
77E41BE6    5D               pop     ebp  
77E41BE7    C2 2800          retn    28  
```

下面是堆栈........（看到了吧，不用我说了吧！）

```
0135F334   77E7347C  /CALL 到 CreateProcessA 来自 kernel32.77E73477  
0135F338   00000000  |ModuleFileName = NULL  
0135F33C   0135F384  |CommandLine = "drwtsn32 -p 872 -e 196 -g"  
0135F340   00000000  |pProcessSecurity = NULL  
0135F344   00000000  |pThreadSecurity = NULL  
0135F348   00000001  |InheritHandles = TRUE  
0135F34C   00000000  |CreationFlags = 0  
0135F350   00000000  |pEnvironment = NULL  
0135F354   00000000  |CurrentDir = NULL  
0135F358   0135F484  |pStartupInfo = 0135F484  
0135F35C   0135F374  \\pProcessInfo = 0135F374  
0135F360   0047D5F0  wnwb.0047D5F0  
0135F364   00A50530  
```

继续走到

```
004631CD  |>  \\83C8 FF        or      eax, FFFFFFFF               ;  Case 1 of switch 004630C6  
004631D0  |.  EB 09           jmp     short 004631DB  
004631D2  |>  FF75 0C         push    dword ptr [ebp+C]            ; /pExceptionInfo; Case 0 of switch 004630C6  
004631D5      FF15 70D14700  call    [<&KERNEL32.UnhandledExceptionFi>;  -------->此函数引发了异常  
004631DB  |>  5E             pop     esi  
004631DC  |.  5B             pop     ebx                            ;  0125FFA4  
004631DD  |.  C9             leave  
004631DE  \\.  C3             retn  
```

哈哈，原来 `UnhandledExceptionFi` 函数引发了异常

异常处找到了接下来是修复了（不知到你们想到修复方案没），我可想到了，利用正常搜索过程返回程序入口，就和正常搜索过程返回、程序就正常了，你说是不是（呵呵，光说没用还是要找到能用才行，），OK 来一次正常的，选几个文字点搜索，以前下的 `BP CreateProcessA` 起作用了

一直走到：

```
0045DB21  |.  8906           mov     [esi], eax  
0045DB23  |.  A1 4C2A4A00    mov     eax, [4A2A4C]  
0045DB28  |.  85C0           test    eax, eax  
0045DB2A  |.  74 02          je      short 0045DB2E  
0045DB2C  |.  FFD0           call    eax  
0045DB2E  |>  8365 FC 00     and     dword ptr [ebp-4], 0  
0045DB32  |.  FF76 4C        push    dword ptr [esi+4C]  
0045DB35  |.  FF56 48        call    [esi+48]  
0045DB38  |.  50             push    eax  
0045DB39  |.  E8 37000000    call    0045DB75----------------------在此 call 0045DB75 正常搜索过程返回  
0045DB3E  |.  59             pop     ecx  
0045DB3F  |.  834D FC FF     or      dword ptr [ebp-4], FFFFFFFF  
0045DB43  |.  33C0           xor     eax, eax  
0045DB45  |.  8B4D F0        mov     ecx, [ebp-10]  
0045DB48  |.  64:890D 00000>mov     fs:[0], ecx  
0045DB4F  |.  5F             pop     edi  
0045DB50  |.  5E             pop     esi  
0045DB51  |.  5B             pop     ebx  
0045DB52  |.  C9             leave  
0045DB53  \\.  C2 0400        retn    4  
```

呵。。来修复了

===============================================================================

在 `004631D5` `UnhandledExceptionFi` 函数引发了异常

所以 `004631D5` `call [<&KERNEL32.UnhandledExceptionFi>`改为：`call 0045DB75`

OK 保存，哈哈，看到效果没，此 BUG 修复得很完美，所以我把我的修改版命名为：万能五笔严重[BUG]baidu 版

以后修改得更完美时发布此版本，记得用 baidu 搜哦/最爱 baidu

![](img/096180ac9779aa9862c53171fea3f6c7.jpg)

By 小 Q 2006 .7 .19 零晨 在深圳健华医药公司辞职的第一天

转贴请注明出处，版权所有[小 Q]qyc LOVE pediy.com

`4dd466d0e1e0cc123afbe504da2462ca`
