
# WYSIWYG Web Build V5.5.4 算法+网络验证分析

> 原文：[`www.pediy.com/kssd/pediy10/77296.html`](https://www.pediy.com/kssd/pediy10/77296.html)

- 【破文标题】WYSIWYG Web Build V5.5.4 算法+网络验证分析
- 【破文作者】Playboysen
- 【作者邮箱】playboysen@126.com
- 【破解工具】PEiD, OD
- 【破解平台】Windows XP
- 【软件大小】3.33 MB
- 【软件授权】国外软件/网页制作
- 【软件语言】英文
- 【原版下载】`http://www.wysiwygwebbuilder.com/webbuilder5.zip`
- 【保护方式】网络验证
- 【软件简介】WYSIWYG Web Builder 是一个很小巧的‘所见即所得’网页编辑器，非常适合初学者或想要做个简单网页的人使用，当然你也能手动去更改网页的程式码。
- 【破解声明】一点心得，愿与大家分享 o(∩\_∩)o 版权所有，转载注明作者！

## 破解内容

首先，我不得不提醒各位 This is an long long long long article，如果你没有耐心，就不要往下看了，我光分析代码就用了近 5 个小时，可见一斑吧 o(∩\_∩)o...

程序是网络验证，但是不同于一般验证方式，用户输入注册码后并不验证，而是先进行很长很麻烦的一段算法比较，看输入的像不像注册码等等，验证过程极其繁琐（作者很有耐心啊）。有好几个错误提示，MFC 程序，可以用 `MFCSpy` 或者这里的脚本 [`http://bbs.pediy.com/showthread.php?t=76990&viewgoodnees=1`](http://bbs.pediy.com/showthread.php?t=76990&viewgoodnees=1) 查找按钮事件（已找到 `00465B60`）或者下 `GetWindowTextA` 查找关键部分。我们设假码为“WB500-2L6W-560F-9X6G-5A38-2CF2”，且以“-”为界分别设每一部分注册码为 `T1`-`T2`-`T3`-`T4`-`T5`-`T6`（具体为什么这么设，你看了下面的算法比较就知道了，为了节约时间，少走弯路）

OK, follow me！

```
00465BE2   .  8D8F 94010000   lea ecx,dword ptr ds:\[edi+194\]
00465BE8   .  E8 C78A1300     call <jmp.&MFC42.#3874\_CWnd::GetWindowTextA>
00465BED   .  8D55 EC         lea edx,dword ptr ss:\[ebp-14\]                 ;  用户名
00465BF0   .  8D8F 54010000   lea ecx,dword ptr ds:\[edi+154\]
00465BF6   .  52              push edx
00465BF7   .  E8 B88A1300     call <jmp.&MFC42.#3874\_CWnd::GetWindowTextA>
00465BFC   .  8D4D E8         lea ecx,dword ptr ss:\[ebp-18\]                 ;  注册码
......
00465C30   .  8B45 E8         mov eax,dword ptr ss:\[ebp-18\]                 ;  用户名放入
00465C33   .  8B48 F8         mov ecx,dword ptr ds:\[eax-8\]                 ;  用户名长度放入
00465C36   .  85C9           test ecx,ecx
00465C38   .  0F84 34070000  je cf.00466372
00465C3E   .  6A 40          push 40
00465C40   .  8D4D E8       lea ecx,dword ptr ss:\[ebp-18\]                 ;  用户名地址放入
00465C43   .  E8 468C1300   call <jmp.&MFC42.#2763\_CString::Find>         ;  猜测是检测用户名中"@"前的字符数
00465C48   .  83CE FF       or esi,FFFFFFFF
00465C4B   .  3BC6          cmp eax,esi
00465C4D   .  0F84 22070000 je cf.00466375
00465C53   .  8B4D EC       mov ecx,dword ptr ss:\[ebp-14\]                 ;  假码放入
00465C56   .  8B41 F8       mov eax,dword ptr ds:\[ecx-8\]                 ;  假码长度放入
00465C59   .  85C0          test eax,eax
00465C5B   .  0F84 14070000 je cf.00466375
00465C61   .  6A 2D         push 2D                                  ;  "-"
00465C63   .  8D4D EC       lea ecx,dword ptr ss:\[ebp-14\]
00465C66   .  E8 238C1300   call <jmp.&MFC42.#2763\_CString::Find>         ;  检测注册码中第一个“-”出现的位置
00465C6B   .  3BC6          cmp eax,esi
00465C6D   .  0F84 02070000 je cf.00466375
00465C73   .  E8 548A1300   call <jmp.&MFC42.#1168\_AfxGetModuleState>
00465C78   .  8B55 EC       mov edx,dword ptr ss:\[ebp-14\]
00465C7B   .  8B58 04       mov ebx,dword ptr ds:\[eax+4\]
00465C7E   .  68 8C267600   push cf.0076268C                      ;/s2 = "KFW9K-7J2G8-BQT4Y-V3GXV-QM4RF"
00465C83   .  52            push edx                                  ; |s1
00465C84   .  895D D8       mov dword ptr ss:\[ebp-28\],ebx             ; |
00465C87   .  FF15 40966D00 call dword ptr ds:\[<&msvcrt.\_mbscmp>\] ; \\_mbscmp
00465C8D   .  83C4 08       add esp,8
00465C90   .  85C0         test eax,eax
00465C92   .  75 12        jnz short cf.00465CA6                     ;  这里很明显是在比较……
00465C94   .  50          push eax
00465C95   .  6A 30       push 30
00465C97   .  68 30267600 push cf.00762630                  ;  "You've entered a serial number which can only be used to activate WYSIWYG Web Builder 3.x"
00465C9C   .  E8 038E1300  call <jmp.&MFC42.#1200\_AfxMessageBox>
00465CA1   .  E9 DC060000  jmp cf.00466382
00465CA6   >  68 28267600  push cf.00762628                 ;  ASCII "WB400-"
00465CAB   .  8D4D EC     lea ecx,dword ptr ss:\[ebp-14\]
00465CAE   .  E8 BD8B1300 call <jmp.&MFC42.#2764\_CString::Find>         ;  检测注册码开头是不是“WB400-”
00465CB3   .  3BC6        cmp eax,esi
00465CB5   .  6A 00       push 0
00465CB7   .  74 11       je short cf.00465CCA
00465CB9   .  6A 30       push 30
00465CBB   .  68 CC257600 push cf.007625CC                ;  "You've entered a serial number which can only be used to activate WYSIWYG Web Builder 4.x"
00465CC0   .  E8 DF8D1300 call <jmp.&MFC42.#1200\_AfxMessageBox>
00465CC5   .  E9 B8060000 jmp cf.00466382
```

前期的几处比较，算是小试牛刀，嗦的还在后头呢

```
00465D52   >  \\8B87 D4010000  mov eax,dword ptr ds:\[edi+1D4\]
00465D58   .  85C0         test eax,eax
00465D5A   .  74 1E        je short cf.00465D7A
00465D5C   .  8BCF        mov ecx,edi
00465D5E   .  E8 FD0D0000  call cf.00466B60                 ;  一个较大的缓冲，在测试是否联网
00465D63   .  85C0        test eax,eax
00465D65   .  75 13       jnz short cf.00465D7A
00465D67   .  6A FF       push -1
00465D69   .  6A 30       push 30
00465D6B   .  68 EF730000 push 73EF
00465D70   .  E8 C38C1300  call <jmp.&MFC42.#1199\_AfxMessageBox>        ;  提示需要联网
00465D75   .  E9 93050000  jmp cf.0046630D
00465D7A   >  56          push esi                     ;  写入注册信息
00465D7B   .  68 C0257600  push cf.007625C0             ;  ASCII "NagCount"
00465D80   .  68 08A77500  push cf.0075A708             ;  ASCII "Settings"
......
00465DB4   .  E8 A78C0100  call cf.0047EA60
00465DB9   .  8B4F 60      mov ecx,dword ptr ds:\[edi+60\]
00465DBC   .  E8 DF890100  call cf.0047E7A0                 ;  多次跟踪发现这里比较关键,F7
00465DC1   .  8B4F 60      mov ecx,dword ptr ds:\[edi+60\]
00465DC4   .  E8 B7850100  call cf.0047E380                 ;  多次跟踪发现这里比较关键,F7
00465DC9   .  85C0        test eax,eax
00465DCB   .  75 13       jnz short cf.00465DE0             ;  关键跳
00465DCD   .  6A FF       push -1
00465DCF   .  6A 30       push 30
00465DD1   .  68 F1730000 push 73F1
00465DD6   .  E8 5D8C1300  call <jmp.&MFC42.#1199\_AfxMessageBox>        ;  错误提示
00465DDB   .  E9 2D050000  jmp cf.0046630D
00465DE0   >  8B87 D4010000  mov eax,dword ptr ds:\[edi+1D4\]
```

`F7` 跟入 `00465DC4`（因为 `00465DBC` 里面的主要代码和这里大致相同，故着重分析一处）

```
......
0047E452  |.  52         push edx
0047E453  |.  E8 6E051200  call <jmp.&MFC42.#1140\_AfxExtractSubString>
0047E458  |.  56         push esi                     ; /String
0047E459  |.  FF15 98826D00  call dword ptr ds:\[<&kernel32.lstrlen>\]     ; \\lstrlenA
0047E45F  |.  83F8 1E    cmp eax,1E                   ;  比较注册码长度是否是 30 位
0047E462  |.  0F85 48020000  jnz cf.0047E6B0
0047E468  |.  8B4424 20  mov eax,dword ptr ss:\[esp+20\]     ;  比较注册码开头是不是"WB500"
0047E46C  |.  68 E0367600  push cf.007636E0                   ; /s2 = "WB500"
0047E471  |.  50         push eax                     ; |s1
0047E472  |.  FF15 40966D00  call dword ptr ds:\[<&msvcrt.\_mbscmp>\]       ; \\_mbscmp
0047E478  |.  83C4 08    add esp,8
0047E47B  |.  85C0       test eax,eax
0047E47D  |.  0F85 2D020000  jnz cf.0047E6B0
```

看这里，出现了一个很令人鼓舞的字符串，我原以为是明码注册码，可是……



# 逆向工程分析记录

## 1. 本地注册码校验

首先对比输入的注册码字符串与预设字符串。

```
0047E4BE  |. /0F85 EC010000   jnz cf.0047E6B0
0047E4C4  |. |68 C0367600     push cf.007636C0                              ; /String2 = "WB500-7J2G-BQT4-V3GX-QM4R-FFFF"
0047E4C9  |. |56              push esi                                      ; |String1
0047E4CA  |. |FF15 90826D00   call dword ptr ds:[<&kernel32.lstrcmp>]       ; \lstrcmpA
0047E4D0  |. |85C0            test eax,eax
0047E4D2  |. |75 2C           jnz short cf.0047E500                         ;很不幸，这不是注册码
0047E4D4  |. |E8 F3011200     call <jmp.&MFC42.#1168_AfxGetModuleState>
0047E4D9  |. |8B40 04         mov eax,dword ptr ds:[eax+4]
0047E4DC  |. |56              push esi
0047E4DD  |. |68 B4257600     push cf.007625B4                        ;  ASCII "SerialNum"
0047E4E2  |. |68 08A77500     push cf.0075A708                        ;  ASCII "Settings"
0047E4E7  |. |8BC8            mov ecx,eax
0047E4E9  |. |E8 02021200     call <jmp.&MFC42.#6403_CWinApp::WriteProfileStrin>
```

核心算法处在这里。

## 2. 核心算法逻辑

代码逻辑主要涉及字符串解析与数值比较，利用 `strtol` 函数进行进制转换，并执行复杂的数学运算验证注册码片段（T2, T4, T5, T6）。

```
0047E500  |> \8B4424 18       mov eax,dword ptr ss:[esp+18]
0047E504  |.  55             push ebp
0047E505  |.  50             push eax                                 ; /Arg1
0047E506  |.  8BCF           mov ecx,edi                              ; |
0047E508  |.  E8 93FCFFFF    call cf.0047E1A0                          ; \cf.0047E1A0
0047E50D  |.  8B5424 20      mov edx,dword ptr ss:[esp+20]
0047E511  |.  8B35 04966D00  mov esi,dword ptr ds:[<&msvcrt.strtol>]   ;  msvcrt.strtol
0047E517  |.  8D4C24 2C      lea ecx,dword ptr ss:[esp+2C]
0047E51B  |.  6A 10          push 10                                 ; /radix = 10 (16.)
0047E51D  |.  51             push ecx                                 ; |endptr
0047E51E  |.  52             push edx                                 ; |s
0047E51F  |.  894424 3C      mov dword ptr ss:[esp+3C],eax           ; |
0047E523  |.  FFD6           call esi                                 ; \strtol 函数，具体功能一句话说不清，百度一下
......
0047E58F  |.  51             push ecx
0047E590  |.  50             push eax
0047E591  |.  8D4C24 44      lea ecx,dword ptr ss:[esp+44]
0047E595  |.  E8 B64AF9FF    call cf.00413050                        ;  由注册码中的 T2\T4\T5\T6 运算得出 T3
0047E59A  |.  8B5424 18      mov edx,dword ptr ss:[esp+18]
0047E59E  |.  8D4C24 2C      lea ecx,dword ptr ss:[esp+2C]
0047E5A2  |.  6A 10          push 10                                 ; /radix = 10 (16.)
0047E5A4  |.  51             push ecx                                 ; |endptr
0047E5A5  |.  52             push edx                                 ; |s
0047E5A6  |.  8BD8           mov ebx,eax                              ; |??
0047E5A8  |.  FFD6           call esi                                 ; \strtol 函数，具体功能一句话说不清，百度一下
0047E5AA  |.  83C4 0C        add esp,0C
0047E5AD  |.  894424 34      mov dword ptr ss:[esp+34],eax
0047E5B1  |.  8B4424 14      mov eax,dword ptr ss:[esp+14]
0047E5B5  |.  8BCF           mov ecx,edi
0047E5B7  |.  50             push eax                                 ; /Arg1
0047E5B8  |.  E8 E3FBFFFF    call cf.0047E1A0                        ; \也是一小部分算法，F7
0047E5BD  |.  8B4C24 10      mov ecx,dword ptr ss:[esp+10]
0047E5C1  |.  8BF0           mov esi,eax                              ;  >2
0047E5C3  |.  51             push ecx                                 ; /Arg1
0047E5C4  |.  8BCF           mov ecx,edi                              ; |
0047E5C6  |.  E8 D5FBFFFF    call cf.0047E1A0                        ; \大致是将所取部分第一个非数字的字符减去 50h 得出的值换算成十进制数再与原字符串连接
0047E5CB  |.  3BEB           cmp ebp,ebx                              ;  这里是比较 T3 是不是等于 0047E595 处运算出的值
0047E5CD  |.  5D             pop ebp
0047E5CE  |.  0F85 C0000000  jnz cf.0047E694
0047E5D4  |.  8B4C24 2C      mov ecx,dword ptr ss:[esp+2C]
0047E5D8  |.  3BF1           cmp esi,ecx                              ;  T5 算出的值要大于 T2 运算的值
0047E5DA  |.  0F8E B4000000  jle cf.0047E694
0047E5E0  |.  3BF0           cmp esi,eax                              ;  T5 算出的值要大于 T6 运算的值
0047E5E2  |.  0F8E AC000000  jle cf.0047E694
0047E5E8  |.  03C6           add eax,esi                              ;  这一段是（T5 运算值+T6 运算值+T2 运算值）是否等于 T4 运算值
0047E5EA  |.  03C1           add eax,ecx
0047E5EC  |.  8B4C24 30      mov ecx,dword ptr ss:[esp+30]
0047E5F0  |.  3BC1           cmp eax,ecx
0047E5F2  |.  0F85 9C000000  jnz cf.0047E694
0047E5F8  |.  E8 CF001200    call <jmp.&MFC42.#1168_AfxGetModuleState>
0047E5FD  |.  8B40 04        mov eax,dword ptr ds:[eax+4]
0047E600  |.  8D4C24 34      lea ecx,dword ptr ss:[esp+34]
```

我们的假码是经过算法分析后精心构造的，当然符合上面的严格校验，目的是能进入下面的网络验证，如果假码不符合上述验证，你就没资格进行网络验证（*^__^*）……

在调试器中，使用 `F8` 单步执行到以下位置：

```
00465E74   .  E8 41881300    call <jmp.&MFC42.#540_CString::CString>
00465E79   .  68 40257600    push cf.00762540                      ; "http://www.wysiwygwebbuilder.com/php/wb5reg.php?email="
......
00465E95   .  E8 0C8A1300    call <jmp.&MFC42.#939_CString::operator+=>
00465E9A   .  68 38257600    push cf.00762538                      ;  ASCII "&id="
......
00465EAD   .  50             push eax
00465EAE   .  E8 F3891300    call <jmp.&MFC42.#939_CString::operator+=>
00465EB3   .  68 30257600    push cf.00762530                      ;  ASCII "&sn="
00465EB8   .  8D4D DC        lea ecx,dword ptr ss:[ebp-24]
```

毫无疑问，网络验证的关键到了。

## 3. 网络验证请求

程序通过发送 HTTP 请求进行在线验证。

```
00465F71   .  E8 2E911300    call <jmp.&MFC42.#5808_CHttpFile::SendRequest>
00465F76   .  8B06           mov eax,dword ptr ds:[esi]
00465F78   .  8BCE           mov ecx,esi
00465F7A   .  FF50 38        call dword ptr ds:[eax+38]                    ;  得到返回值的长度
......
00465F9F   .  8D4D D4        lea ecx,dword ptr ss:[ebp-2C]
00465FA2   .  E8 A9871300    call <jmp.&MFC42.#5572_CString::ReleaseBuffer> ;  读取网站返回值
00465FA7   .  8B06           mov eax,dword ptr ds:[esi]                    ;  这里就出现了网站的返回值
00465FA9   .  8BCE           mov ecx,esi
00465FAB   .  FF50 54        call dword ptr ds:[eax+54]                    ;  关闭网络连接
00465FAE   .  85F6           test esi,esi
00465FB0   .  74 09          je short cf.00465FBB
......
00465FBB   >  8B75 C4        mov esi,dword ptr ss:[ebp-3C]
00465FBE   .  8BCE           mov ecx,esi
00465FC0   .  8B06           mov eax,dword ptr ds:[esi]
00465FC2   .  FF50 14        call dword ptr ds:[eax+14]                    ;  关闭 FTP 连接
00465FC5   .  85F6           test esi,esi
00465FC7   .  74 09          je short cf.00465FD2
......
00465FD2   >  8D4D 88        lea ecx,dword ptr ss:[ebp-78]
00465FD5   .  E8 BE901300    call <jmp.&MFC42.#1988_CInternetSession::Close> ;  关闭连接
```

我们通过上面的跟踪知道程序是发送这个地址来验证的：

`http://www.wysiwygwebbuilder.com/php/wb5reg.php?email=playboysen@126.com&id=F844ECED7CFD5DFD6D&sn=WB500-2L6W-560F-9X6G-5A38-2CF2`

我们在浏览器里输入此地址后返回一段固定值 `#4f3f7931bfbe3b61df362b54029b21bd#1#`。

多次跟踪知道其返回值是以 `#` 符号为界线，`4f3f7931bfbe3b61df362b54029b21bd` 是固定值（32 位，如果没猜错应该是 MD5 加密），后面的数才是关键。

代码:



```
......
00465FF6   .  E8 BF861300     call <jmp.&MFC42.#540\_CString::CString>
00465FFB   .  8B45 D4         mov eax,dword ptr ss:\[ebp-2C\]                   ;  取出刚才连接网站时的返回值
......
0046601C   .  E8 A5891300     call <jmp.&MFC42.#1140\_AfxExtractSubString>       ;  这个函数很是"顾名思义"哦
00466021   .  8B4D C8         mov ecx,dword ptr ss:\[ebp-38\]
00466024   .  51              push ecx                                          ; /s
00466025   .  FF15 E8956D00   call dword ptr ds:\[<&msvcrt.atoi>\]             ; \\atoi  把字符串转换成整型数
0046602B   .  83C4 04         add esp,4
0046602E   .  83F8 19         cmp eax,19                                          ; 这里在比较什么???
00466031   .  0F8E AC000000   jle cf.004660E3                                     ; 跳了就错
00466037   .  8B55 EC         mov edx,dword ptr ss:\[ebp-14\]
0046603A   .  8B4D D8         mov ecx,dword ptr ss:\[ebp-28\]
0046603D   .  52              push edx
0046603E   .  68 B4257600     push cf.007625B4                    ;  ASCII "SerialNum"
00466043   .  68 08A77500     push cf.0075A708                    ;  ASCII "Settings"
00466048   .  E8 A3861300     call <jmp.&MFC42.#6403\_CWinApp::WriteProfileStrin>

```

看到了吧，`0046602E`处在比较什么？

没错，在比较返回值最后面的数值，如果小于`25`就错，也就是说若网站返回类似`#4f3f7931bfbe3b61df362b54029b21bd#66#`就代表正确

还等什么？有`NetBox`或者`IIS`服务器的，用记事本伪造个`wb5reg.php`，里面就一句话`#4f3f7931bfbe3b61df362b54029b21bd#66#`，然后修改`hosts`文件，加上一句`127.0.0.1  www.wysiwygwebbuilder.com`，来搞定网络验证即可

我这里什么都没有，就不做测试了，不过一下几点哦

1. 不要忘了，程序在`00465D5E`处要测试网络的，你修改了`hosts`重定向网站后，程序会认为你没联网
2. 如果你多次输入假码联网验证，则网站返回值会变成`Many times to try...`什么的

从这个程序中，我们可以得到以下几点启示

1. 即使用网络验证，也不要太“单纯”，先用较复杂算法测试一下假码看再说
2. 网络验证前检测网络与主页联通是否正常
3. 算法不要太平铺直白，多用几个函数套用，把解密者烦死
4. 必须重启验证，而且验证函数要稍复杂些（以不影响启动速度为限哦）
