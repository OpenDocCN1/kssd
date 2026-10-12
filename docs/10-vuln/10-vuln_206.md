
# 简单分析 mplayer 播放器读取.m3u 格式文件漏洞

> 原文：[`www.pediy.com/kssd/pediy12/139600.html`](https://www.pediy.com/kssd/pediy12/139600.html)

## 前言

这段时间一直在努力学习漏洞分析,并参考了 failwest 大牛制作<0day 第二版>, 暴风 m3u 文件读取漏洞(看雪网->鲜果也曾对其进行分析),小弟不才,只能跟着大牛后面跑.

附:暴风 m3u 分析,是在同事 dge 的帮助下完成的调试,乘热打铁,自己分析了同一类型(mplayer 播放器)漏洞.也算是对自己知识的总结,第一次写文章,还请大家指责批评,不清楚的地方可以发邮件问我

## 测试环境

windows xp sp3_cn (实体机测试，没有使用虚拟机)
ollyice 看雪版
poc 和测试软件版本见:http://www.exploit-db.com/exploits/17565/

## 漏洞触发过程分析

`ollyice` 附加 `mplayer.exe` 进程,设置 OD 调试断点(工具栏-->选项-->调试选项--->异常标签)

------------------------------

忽略在 Kernel32 中的内存访问异常,并勾选以下选项
int3 中断
单步中断
整数除以 0
无效或特权指令
所有 FPU 异常

-----------------------------------------

将生成的 m3u 格式文件拖拽到 mplayer 播放器窗口中,F9 运行,OD 会自动下断在程序异常信息信息处.

观察 od 堆栈信息窗口,查看栈信息

```
0022E948   00000000
0022E94C   00561760  返回到 mplayer.00561760 来自 <jmp.&msvcrt.strcpy>(可疑)
0022E950   0022EBB8  ASCII "http://  AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"...
0022E954   003FD5C8  ASCII "http://  AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"...
0022E958   00000000
0022E95C   00000000
0022E960   00000000
0022E964   0022E974
0022E968   00000000
0022E96C   7C930098  返回到 ntdll.7C930098 来自 ntdll.7C922AB0
```

可以发现 jmp.&msvcrt.strcpy(可疑,再后面进行证实),鼠标右键点击该行,选择"反汇编窗口中跟随",来到汇编主窗体,对该函数下断点,重新载入 m3u 至 mplayer, F9 来到断点处,可以查看下堆栈信息和汇编信息

------------------------------------------

堆栈信息:

```
0022E950   0022EBB8  |dest = 0022EBB8(目标地址)
0022E954   003FD5C8  \\src = "http://  AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"...(源地址)
0022E958   00000000
0022E95C   00000000
0022E960   00000000
0022E964   0022E974
0022E968   00000000
0022E96C   7C930098  返回到 ntdll.7C930098 来自 ntdll.7C922AB0
```

汇编主窗体信息:

```
00561754  |.  895C24 04     mov     dword ptr [esp+4], ebx           ;  ||
00561758  |.  890424        mov     dword ptr [esp], eax             ;  ||
0056175B  |.  E8 A8032900   call    <jmp.&msvcrt.strcpy>             ; |\strcpy
00561760  |.  8D9424 680200>lea     edx, dword ptr [esp+268]         ;  |
00561767  |.  C74424 04 2F0>mov     dword ptr [esp+4], 2F            ;  |
0056176F  |.  891424        mov     dword ptr [esp], edx             ;  |
00561772  |.  E8 C1032900   call    <jmp.&msvcrt.strrchr>            ; \\strrchr
```

-----------------------------------------------

F7 进入 call:

```
77C16030 >  57              push    edi
77C16031    8B7C24 08       mov     edi, dword ptr [esp+8]
77C16035    EB 6A           jmp     short 77C160A1
77C16037    8DA424 00000000 lea     esp, dword ptr [esp]
77C1603E    8BFF            mov     edi, edi
77C16040 >  8B4C24 04       mov     ecx, dword ptr [esp+4]
77C16044    57              push    edi
77C16045    F7C1 03000000   test    ecx, 3
77C1604B    74 0F           je      short 77C1605C
77C1604D    8A01            mov     al, byte ptr [ecx]
77C1604F    41              inc     ecx
77C16050    84C0            test    al, al
77C16052    74 3B           je      short 77C1608F
77C16054    F7C1 03000000   test    ecx, 3
77C1605A  ^ 75 F1           jnz     short 77C1604D
77C1605C    8B01            mov     eax, dword ptr [ecx]
77C1605E    BA FFFEFE7E     mov     edx, 7EFEFEFF
77C16063    03D0            add     edx, eax
77C16065    83F0 FF         xor     eax, FFFFFFFF
77C16068    33C2            xor     eax, edx
77C1606A    83C1 04         add     ecx, 4
77C1606D    A9 00010181     test    eax, 81010100
77C16072  ^ 74 E8           je      short 77C1605C
77C16074    8B41 FC         mov     eax, dword ptr [ecx-4]
77C16077    84C0            test    al, al
77C16079    74 23           je      short 77C1609E
77C1607B    84E4            test    ah, ah
77C1607D    74 1A           je      short 77C16099
77C1607F    A9 0000FF00     test    eax, 0FF0000
77C16084    74 0E           je      short 77C16094
77C16086    A9 000000FF     test    eax, FF000000
77C1608B    74 02           je      short 77C1608F
77C1608D  ^ EB CD           jmp     short 77C1605C
77C1608F    8D79 FF         lea     edi, dword ptr [ecx-1]
77C16092    EB 0D           jmp     short 77C160A1
77C16094    8D79 FE         lea     edi, dword ptr [ecx-2]
77C16097    EB 08           jmp     short 77C160A1
77C16099    8D79 FD         lea     edi, dword ptr [ecx-3]
77C1609C    EB 03           jmp     short 77C160A1
77C1609E    8D79 FC         lea     edi, dword ptr [ecx-4]
77C160A1    8B4C24 0C       mov     ecx, dword ptr [esp+C]
77C160A5    F7C1 03000000   test    ecx, 3
77C160AB    74 19           je      short 77C160C6
77C160AD    8A11            mov     dl, byte ptr [ecx]
77C160AF    41              inc     ecx
77C160B0    84D2            test    dl, dl
77C160B2    74 64           je      short 77C16118
77C160B4    8817            mov     byte ptr [edi], dl
77C160B6    47              inc     edi
77C160B7    F7C1 03000000   test    ecx, 3
77C160BD  ^ 75 EE           jnz     short 77C160AD
77C160BF    EB 05           jmp     short 77C160C6
77C160C1    8917            mov     dword ptr [edi], edx(这里出现异常)
77C160C3    83C7 04         add     edi, 4
77C160C6    BA FFFEFE7E     mov     edx, 7EFEFEFF
77C160CB    8B01            mov     eax, dword ptr [ecx]
77C160CD    03D0            add     edx, eax
77C160CF    83F0 FF         xor     eax, FFFFFFFF
77C160D2    33C2            xor     eax, edx
77C160D4    8B11            mov     edx, dword ptr [ecx]
77C160D6    83C1 04         add     ecx, 4
77C160D9    A9 00010181     test    eax, 81010100
77C160DE  ^ 74 E1           je      short 77C160C1
77C160E0    84D2            test    dl, dl
77C160E2    74 34           je      short 77C16118
77C160E4    84F6            test    dh, dh
77C160E6    74 27           je      short 77C1610F
77C160E8    F7C2 0000FF00   test    edx, 0FF0000
77C160EE    74 12           je      short 77C16102
77C160F0    F7C2 000000FF   test    edx, FF000000
77C160F6    74 02           je      short 77C160FA
77C160F8  ^ EB C7           jmp     short 77C160C1
77C160FA    8917            mov     dword ptr [edi], edx
77C160FC    8B4424 08       mov     eax, dword ptr [esp+8]
77C16100    5F              pop     edi
77C16101    C3              retn
77C16102    66:8917         mov     word ptr [edi], dx
77C16105    8B4424 08       mov     eax, dword ptr [esp+8]
77C16109    C647 02 00      mov     byte ptr [edi+2], 0
77C1610D    5F              pop     edi
77C1610E    C3              retn
77C1610F    66:8917         mov     word ptr [edi], dx
77C16112    8B4424 08       mov     eax, dword ptr [esp+8]
77C16116    5F              pop     edi
77C16117    C3              retn
77C16118    8817            mov     byte ptr [edi], dl
77C1611A    8B4424 08       mov     eax, dword ptr [esp+8]
77C1611E    5F              pop     edi
77C1611F    C3              retn
```

可以发现,在调试过程中,红色地方,就是之前我们捕获的异常地址处,说明该函数在函数 cpy 时,发生了溢出

命令:`DD 0022EBB8`,查看内存信息,内容已经被覆盖,

```
0022EBB8  68 74 74 70 3A 2F 2F 20 41 41 41 41 41 41 41 41  http://  AAAAAAAA
```



```
0022EBC8  41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  AAAAAAAAAAAAAAAA  
0022EBD8  41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  AAAAAAAAAAAAAAAA  
0022EBE8  41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  AAAAAAAAAAAAAAAA  
0022EBF8  41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  AAAAAAAAAAAAAAAA  
0022EC08  41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  AAAAAAAAAAAAAAAA  
0022EC18  41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  AAAAAAAAAAAAAAAA  
0022EC28  41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  AAAAAAAAAAAAAAAA  
0022EC38  41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  AAAAAAAAAAAAAAAA  
0022EC48  41 41 41 41 41 41 41 41 41 41 41 41 41 41 41.....     AAAAAAAAAAAAAAA.....
```

## 原理分析
函数在拷贝的过程中,由于原地址数据超长,导致目标数据出现溢出.

## 总结
之前我看过 failwest 和仙果的文章,他们分别是按特征字符串".m3u"和"Readfile 函数"下断点并进行跟踪,查找异常函数,而我此篇文章却是直接分析的栈,可能会有些跳跃,大家可自行选择.![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")
