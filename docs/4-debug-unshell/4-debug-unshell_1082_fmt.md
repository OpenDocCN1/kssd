
# Themida V1.1.1.0 无驱动版试炼普通保护方式脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-702.htm`](https://www.pediy.com/kssd/pediy07/pediy7-702.htm)

Themida V1.1.1.0 无驱动版试炼普通保护方式脱壳

下载页面：`http://www.oreans.com`

软件大小：6.10 M

加入时间：15-Nov-2005

软件简介：Advanced Windows software protection system, developed for software developers who wish to protect their applications against advanced reverse engineering and software cracking.

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教

【调试环境】：`WinXP`、`OllyDBG`、`PEiD`、`LordPE`、`ImportREC`

【目标下载】：[`www.pediy.com/bbs/pediy7/pediy7-702.rar`](http://www.pediy.com/bbs/pediy7/pediy7-702.rar)

—————————————————————————————————

## 【脱壳过程】：

首先需要说明的是 `Themida V1.1.1.0` 没有使用驱动，可能只是暂时放弃吧。虽然 2005.12.02 升级成 `Themida V1.1.1.5`，但是主页还只提供 `V1.1.1.0 Demo` 下载。不要用本文的方法去 `OllyDBG` 调试有驱动的其他版本 `Themida`，那样只会让你的电脑重启。

没有使用驱动的 `Themida` 几乎对 `OllyDBG` 没有反调试，直接用原版的 `OllyDBG` 即可运行起来。但是 `Themida` 的 `Virtual Machine` 非常强悍，要还原代码是困难的。感谢 `heXer` 和 `shoooo` 的帮忙。

下面以 `Themida V1.1.1.0 Demo` 来加壳 `Win98` 记事本演示，无 `SDK`、`CodeReplace`，强度相对来说降低很多。

—————————————————————————————————

## 一、EP

调试前需要修改一个地方，用 `WinHex` 打开 `Themida V1.1.1.0.Test.exe`，修改 `PE+60` 处的 `SizeOfStackReserve` 值为 `00380000`，或者直接用 `PETools` 修改，这是为了方便后面补区段。

设置 `OllyDBG` 忽略所有异常选项。用 `IsDebug` 插件去掉 `OllyDBG` 的调试器标志。

```
0040D014     B8 00000000        mov eax,0

//进入 OllyDBG 后暂停在这
0040D019     60                 pushad
0040D01A     0BC0               or eax,eax
0040D01C     74 58              je short 0040D076
0040D01E     E8 00000000        call 0040D023
0040D023     58                 pop eax
0040D024     05 43000000        add eax,43
0040D029     8038 E9            cmp byte ptr ds:[eax],0E9
0040D02C     75 03              jnz short 0040D031
0040D02E     61                 popad
0040D02F     EB 35              jmp short 0040D066
```

—————————————————————————————————

## 二、输入表处理

`Alt+M` 打开内存察看窗口，在代码段设置内存写入断点。`Shift+F9`

```
0059E22D     F3:A4              rep movs byte ptr es:[edi],byte ptr ds:[esi]

//中断，壳解压各段
0059E22F     C685 21201F06 56   mov byte ptr ss:[ebp+61F2021],56
0059E236     68 396D1FD4        push D41F6D39
0059E23B     FFB5 D9201F06      push dword ptr ss:[ebp+61F20D9]
0059E241     8D85 38942606      lea eax,dword ptr ss:[ebp+6269438]
0059E247     FFD0               call eax
```

在 `0059E22D` 处我们要 `F7` 一次再 `F8`，否则会长时间无反应

至 `0059E22F` 时再 `Shift+F9`，会中断在壳处理输入表的 `005A1DA5` 处

下面这段代码很长，其实 `Themida` 的输入表处理还是比较简单的。

```
005A1535     8B9D 5111F06       mov ebx,dword ptr ss:[ebp+61F1151]
005A153B     8B0B               mov ecx,dword ptr ds:[ebx]
005A153D     83F9 00            cmp ecx,0
005A1540     0F84 8D090000      je 005A1ED3

//输入表处理完成后此处跳转
005A1546     50                 push eax
005A1547     51                 push ecx
005A1548     60                 pushad
005A1549     33C0               xor eax,eax
005A154B     8985 C5181F06      mov dword ptr ss:[ebp+61F18C5],eax
005A1551     BE 3C000000        mov esi,3C
005A1556     037424 20          add esi,dword ptr ss:[esp+20]
005A155A     66:AD              lods word ptr ds:[esi]
005A155C     034424 20          add eax,dword ptr ss:[esp+20]
005A1560     8B70 78            mov esi,dword ptr ds:[eax+78]
005A1563     037424 20          add esi,dword ptr ss:[esp+20]
005A1567     8B7E 18            mov edi,dword ptr ds:[esi+18]
005A156A     89BD F9141F06      mov dword ptr ss:[ebp+61F14F9],edi
005A1570     85FF               test edi,edi
005A1572     0F85 0A000000      jnz 005A1582
005A1578     E8 630F0000        call 005A24E0
005A157D     E9 91000000        jmp 005A1613
005A1582     51                 push ecx
005A1583     8BD7               mov edx,edi
005A1585     6BD2 04            imul edx,edx,4
005A1588     8995 750D1F06      mov dword ptr ss:[ebp+61F0D75],edx
005A158E     6A 04              push 4
005A1590     68 00100000        push 1000
005A1595     52                 push edx
005A1596     6A 00              push 0
005A1598     FF95 AD251F06      call dword ptr ss:[ebp+61F25AD]
005A159E     8985 AD2F1F06      mov dword ptr ss:[ebp+61F2FAD],eax
005A15A4     8BD0               mov edx,eax
005A15A6     59                 pop ecx
005A15A7     E8 340F0000        call 005A24E0
005A15AC     56                 push esi
005A15AD     AD                 lods dword ptr ds:[esi]
005A15AE     034424 24          add eax,dword ptr ss:[esp+24]
005A15B2     97                 xchg eax,edi
005A15B3     8BDF               mov ebx,edi
005A15B5     57                 push edi
005A15B6     32C0               xor al,al
005A15B8     AE                 scas byte ptr es:[edi]
005A15B9     0F85 F9FFFFFF      jnz 005A15B8
005A15BF     5E                 pop esi
005A15C0     2BFB               sub edi,ebx
005A15C2     52                 push edx
005A15C3     8BD7               mov edx,edi
005A15C5     8BBD D9011F06      mov edi,dword ptr ss:[ebp+61F01D9]
005A15CB     83C9 FF            or ecx,FFFFFFFF
005A15CE     33C0               xor eax,eax
005A15D0     8A06               mov al,byte ptr ds:[esi]
005A15D2     32C1               xor al,cl
005A15D4     46                 inc esi
005A15D5     8B0487             mov eax,dword ptr ds:[edi+eax*4]
005A15D8     C1E9 08            shr ecx,8
005A15DB     33C8               xor ecx,eax
005A15DD     4A                 dec edx
005A15DE     0F85 EAFFFFFF      jnz 005A15CE
005A15E4     8BC1               mov eax,ecx
005A15E6     F7D0               not eax
005A15E8     5A                 pop edx
005A15E9     8902               mov dword ptr ds:[edx],eax
005A15EB     83C2 04            add edx,4
005A15EE     52                 push edx
005A15EF     FF85 C5181F06      inc dword ptr ss:[ebp+61F18C5]
005A15F5     8B95 C5181F06      mov edx,dword ptr ss:[ebp+61F18C5]
005A15FB     3995 F9141F06      cmp dword ptr ss:[ebp+61F14F9],edx
005A1601     0F84 0A000000      je 005A1611
005A1607     5A                 pop edx
005A1608     5E                 pop esi
005A1609     83C6 04            add esi,4
005A160C     E9 9BFFFFFF        jmp 005A15AC
005A1611     5A                 pop edx
005A1612     5E                 pop esi
005A1613     61                 popad
005A1614     59                 pop ecx
005A1615     58                 pop eax
005A1616     C785 D9181F06 0000>mov dword ptr ss:[ebp+61F18D9],0
005A1620     C785 B9061F06 0000>mov dword ptr ss:[ebp+61F06B9],0
005A162A     83BD 70A72C06 00   cmp dword ptr ss:[ebp+62CA770],0
005A1631     0F84 08000000      je 005A163F
005A1637     8D9D 913D2B06      lea ebx,dword ptr ss:[ebp+62B3D91]
005A163D     FFD3               call ebx
005A163F     FF85 89201F06      inc dword ptr ss:[ebp+61F2089]
005A1645     83BD 89201F06 64   cmp dword ptr ss:[ebp+61F2089],64
005A164C     0F82 62000000      jb 005A16B4
005A1652     C785 89201F06 0100>mov dword ptr ss:[ebp+61F2089],1
005A165C     60                 pushad
005A165D     8DB5 04A82C06      lea esi,dword ptr ss:[ebp+62CA804]
005A1663     8DBD C6C02C06      lea edi,dword ptr ss:[ebp+62CC0C6]
005A1669     2BFE               sub edi,esi
005A166B     8BD7               mov edx,edi
005A166D     8BBD D9011F06      mov edi,dword ptr ss:[ebp+61F01D9]
005A1673     83C9 FF            or ecx,FFFFFFFF
005A1676     33C0               xor eax,eax
005A1678     8A06               mov al,byte ptr ds:[esi]
005A167A     32C1               xor al,cl
005A167C     46                 inc esi
005A167D     8B0487             mov eax,dword ptr ds:[edi+eax*4]
005A1680     C1E9 08            shr ecx,8
005A1683     33C8               xor ecx,eax
005A1685     4A                 dec edx
005A1686     0F85 EAFFFFFF      jnz 005A1676
005A168C     8BC1               mov eax,ecx
```



```
005A168E     F7D0                  not eax  

005A1690     3985 1D1D1F06        cmp dword ptr ss:\[ebp+61F1D1D\],eax  

005A1696     0F84 17000000        je 005A16B3  

005A169C     83BD 9D1D1F06 00     cmp dword ptr ss:\[ebp+61F1D9D\],0  

005A16A3     0F85 0A000000        jnz 005A16B3  

//自校验  

//Patch①、jmp 005A16B3    ★  

005A16A9     C785 990C1F06 0100>mov dword ptr ss:\[ebp+61F0C99\],1  

005A16B3     61                   popad  

005A16B4     B9 BDEB8C32          mov ecx,328CEBBD  

005A16B9     BA A0F0804D          mov edx,4D80F0A0  

005A16BE     AD                   lods dword ptr ds:\[esi\]  

005A16BF     89B5 65031F06        mov dword ptr ss:\[ebp+61F0365\],esi  

005A16C5     C746 FC 00000000     mov dword ptr ds:\[esi-4\],0  

005A16CC     3D EEEEEEEE          cmp eax,EEEEEEEE  

005A16D1     0F85 20000000        jnz 005A16F7  

005A16D7     813E DDDDDDDD        cmp dword ptr ds:\[esi\],DDDDDDDD  

005A16DD     0F85 14000000        jnz 005A16F7  

005A16E3     C706 00000000        mov dword ptr ds:\[esi\],0  

005A16E9     83C6 04              add esi,4  

005A16EC     89B5 65031F06        mov dword ptr ss:\[ebp+61F0365\],esi  

005A16F2     E9 A7070000          jmp 005A1E9E  

005A16F7     8BD8                 mov ebx,eax  

005A16F9     3385 990C1F06        xor eax,dword ptr ss:\[ebp+61F0C99\]  

005A16FF     C1C8 03              ror eax,3  

005A1702     2BC2                 sub eax,edx  

005A1704     C1C0 10              rol eax,10  

005A1707     33C1                 xor eax,ecx  

005A1709     899D 990C1F06        mov dword ptr ss:\[ebp+61F0C99\],ebx  

005A170F     3D 00000100          cmp eax,10000  

005A1714     0F83 45000000        jnb 005A175F  

005A171A     813E BBBBBBBB        cmp dword ptr ds:\[esi\],BBBBBBBB  

005A1720     0F85 39000000        jnz 005A175F  

005A1726     C706 00000000        mov dword ptr ds:\[esi\],0  

005A172C     83C6 04              add esi,4  

005A172F     89B5 65031F06        mov dword ptr ss:\[ebp+61F0365\],esi  

005A1735     8B9D 51111F06        mov ebx,dword ptr ss:\[ebp+61F1151\]  

005A173B     8B0B                 mov ecx,dword ptr ds:\[ebx\]  

005A173D     8BD0                 mov edx,eax  

005A173F     60                   pushad  

005A1740     8BC2                 mov eax,edx  

005A1742     2B85 89281F06        sub eax,dword ptr ss:\[ebp+61F2889\]  

005A1748     C1E0 02              shl eax,2  

005A174B     0385 91051F06        add eax,dword ptr ss:\[ebp+61F0591\]  

005A1751     96                   xchg eax,esi  

005A1752     AD                   lods dword ptr ds:\[esi\]  

005A1753     03C1                 add eax,ecx  

005A1755     894424 1C            mov dword ptr ss:\[esp+1C\],eax  

005A1759     61                   popad  

005A175A     E9 7C000000          jmp 005A17DB  

005A175F     51                   push ecx  

005A1760     52                   push edx  

005A1761     33C9                 xor ecx,ecx  

005A1763     8B95 AD2F1F06        mov edx,dword ptr ss:\[ebp+61F2FAD\]  

005A1769     3B02                 cmp eax,dword ptr ds:\[edx\]  

005A176B     0F84 38000000        je 005A17A9  

005A1771     83C2 04              add edx,4  

005A1774     41                   inc ecx  

005A1775     3B8D F9141F06        cmp ecx,dword ptr ss:\[ebp+61F14F9\]  

005A177B     0F85 E8FFFFFF        jnz 005A1769  

005A1781     8DB5 2DA72C06        lea esi,dword ptr ss:\[ebp+62CA72D\]  

005A1787     8DBD A51A1F06        lea edi,dword ptr ss:\[ebp+61F1AA5\]  

005A178D     AC                   lods byte ptr ds:\[esi\]  

005A178E     84C0                 test al,al  

005A1790     0F84 06000000        je 005A179C  

005A1796     AA                   stos byte ptr es:\[edi\]  

005A1797     E9 F1FFFFFF          jmp 005A178D  

005A179C     B8 07000000          mov eax,7  

005A17A1     8D8D 174C1F06        lea ecx,dword ptr ss:\[ebp+61F4C17\]  

005A17A7     FFE1                 jmp ecx  

005A17A9     898D C5181F06        mov dword ptr ss:\[ebp+61F18C5\],ecx  

005A17AF     5A                   pop edx  

005A17B0     59                   pop ecx  

005A17B1     56                   push esi  

005A17B2     8B9D 51111F06        mov ebx,dword ptr ss:\[ebp+61F1151\]  

005A17B8     8B0B                 mov ecx,dword ptr ds:\[ebx\]  

005A17BA     8B85 C5181F06        mov eax,dword ptr ss:\[ebp+61F18C5\]  

005A17C0     D1E0                 shl eax,1  

005A17C2     0385 011E1F06        add eax,dword ptr ss:\[ebp+61F1E01\]  

005A17C8     33F6                 xor esi,esi  

005A17CA     96                   xchg eax,esi  

005A17CB     66:AD                lods word ptr ds:\[esi\]  

005A17CD     C1E0 02              shl eax,2  

005A17D0     0385 91051F06        add eax,dword ptr ss:\[ebp+61F0591\]  

005A17D6     96                   xchg eax,esi  

005A17D7     AD                   lods dword ptr ds:\[esi\]  

005A17D8     03C1                 add eax,ecx  

005A17DA     5E                   pop esi  

005A17DB     83BD DD151F06 01     cmp dword ptr ss:\[ebp+61F15DD\],1  

005A17E2     0F84 39000000        je 005A1821  

//下面判断是否是特殊 DLL 的特殊函数，是则加密。当然不希望其加密啦  

//Patch②、jmp 005A180C    ★  

005A17E8     3B8D F1151F06        cmp ecx,dword ptr ss:\[ebp+61F15F1\]  

//Kernel32.DLL ?  

005A17EE     0F84 2D000000        je 005A1821  

005A17F4     3B8D B5011F06        cmp ecx,dword ptr ss:\[ebp+61F01B5\]  

//USER32.DLL ?  

005A17FA     0F84 21000000        je 005A1821  

005A1800     3B8D 410F1F06        cmp ecx,dword ptr ss:\[ebp+61F0F41\]  

//ADVAPI32.DLL ?  

005A1806     0F84 15000000        je 005A1821  

005A180C     8D9D E4B82C06        lea ebx,dword ptr ss:\[ebp+62CB8E4\]  

005A1812     FFD3                 call ebx  

005A1814     8BF8                 mov edi,eax  

005A1816     8985 ED091F06        mov dword ptr ss:\[ebp+61F09ED\],eax  

005A181C     E9 62050000          jmp 005A1D83  

//跳开下面的比较和加密  

005A1821     8D9D E4B82C06        lea ebx,dword ptr ss:\[ebp+62CB8E4\]  

005A1827     FFD3                 call ebx  

005A1829     83BD DD151F06 00     cmp dword ptr ss:\[ebp+61F15DD\],0  

005A1830     0F84 1D000000        je 005A1853  

005A1836     3B85 C9201F06        cmp eax,dword ptr ss:\[ebp+61F20C9\]  

005A183C     0F84 0C000000        je 005A184E  

005A1842     3B85 ED0D1F06        cmp eax,dword ptr ss:\[ebp+61F0DED\]  

005A1848     0F85 05000000        jnz 005A1853  

005A184E     E9 B9FFFFFF          jmp 005A180C  

005A1853     3B85 55121F06        cmp eax,dword ptr ss:\[ebp+61F1255\]  

005A1859     0F85 18000000        jnz 005A1877  

005A185F     83BD 45141F06 00     cmp dword ptr ss:\[ebp+61F1445\],0  

005A1866     0F85 0B000000        jnz 005A1877  

005A186C     8D85 A89A2C06        lea eax,dword ptr ss:\[ebp+62C9AA8\]  

005A1872     E9 95FFFFFF          jmp 005A180C  

005A1877     3B85 55121F06        cmp eax,dword ptr ss:\[ebp+61F1255\]  

005A187D     0F84 89FFFFFF        je 005A180C  

005A1883     83BD 29A72C06 01     cmp dword ptr ss:\[ebp+62CA729\],1  

005A188A     0F85 17000000        jnz 005A18A7  

005A1890     3B85 88A72C06        cmp eax,dword ptr ss:\[ebp+62CA788\]  

005A1896     0F85 0B000000        jnz 005A18A7  

005A189C     8D85 64E35700        lea eax,dword ptr ss:\[ebp+57E364\]  

005A18A2     E9 6DFFFFFF          jmp 005A1814  

005A18A7     33FF                 xor edi,edi  

005A18A9     83BD 39301F06 00     cmp dword ptr ss:\[ebp+61F3039\],0  

005A18B0     0F84 0D020000        je 005A1AC3  

005A18B6     3B85 74A72C06        cmp eax,dword ptr ss:\[ebp+62CA774\]  

005A18BC     75 07               jnz short 005A18C5  

005A18BE     8B85 09221F06        mov eax,dword ptr ss:\[ebp+61F2209\]  

005A18C4     47                   inc edi  

005A18C5     3B85 7CA72C06        cmp eax,dword ptr ss:\[ebp+62CA77C\]  

005A18CB     75 07               jnz short 005A18D4  

005A18CD     8B85 151A1F06        mov eax,dword ptr ss:\[ebp+61F1A15\]  

005A18D3     47                   inc edi  

005A18D4     3B85 78A72C06        cmp eax,dword ptr ss:\[ebp+62CA778\]  

005A18DA     75 07               jnz short 005A18E3  

005A18DC     8B85 95131F06        mov eax,dword ptr ss:\[ebp+61F1395\]  

005A18E2     47                   inc edi  
```



```
005A18E3 3B85 80A72C06 cmp eax,dword ptr ss:[ebp+62CA780]
005A18E9 75 07 jnz short 005A18F2
005A18EB 8B85 C12B1F06 mov eax,dword ptr ss:[ebp+61F2BC1]
005A18F1 47 inc edi
005A18F2 3B85 84A72C06 cmp eax,dword ptr ss:[ebp+62CA784]
005A18F8 75 07 jnz short 005A1901
005A18FA 8B85 DD271F06 mov eax,dword ptr ss:[ebp+61F27DD]
005A1900 47 inc edi
005A1901 3B85 88A72C06 cmp eax,dword ptr ss:[ebp+62CA788]
005A1907 75 07 jnz short 005A1910
005A1909 8B85 512A1F06 mov eax,dword ptr ss:[ebp+61F2A51]
005A190F 47 inc edi
005A1910 3B85 8CA72C06 cmp eax,dword ptr ss:[ebp+62CA78C]
005A1916 75 07 jnz short 005A191F
005A1918 8B85 110A1F06 mov eax,dword ptr ss:[ebp+61F0A11]
005A191E 47 inc edi
005A191F 3B85 90A72C06 cmp eax,dword ptr ss:[ebp+62CA790]
005A1925 75 07 jnz short 005A192E
005A1927 8B85 8D141F06 mov eax,dword ptr ss:[ebp+61F148D]
005A192D 47 inc edi
005A192E 3B85 94A72C06 cmp eax,dword ptr ss:[ebp+62CA794]
005A1934 75 07 jnz short 005A193D
005A1936 8B85 61221F06 mov eax,dword ptr ss:[ebp+61F2261]
005A193C 47 inc edi
005A193D 3B85 98A72C06 cmp eax,dword ptr ss:[ebp+62CA798]
005A1943 75 07 jnz short 005A194C
005A1945 8B85 25131F06 mov eax,dword ptr ss:[ebp+61F1325]
005A194B 47 inc edi
005A194C 3B85 A0A72C06 cmp eax,dword ptr ss:[ebp+62CA7A0]
005A1952 75 07 jnz short 005A195B
005A1954 8B85 550F1F06 mov eax,dword ptr ss:[ebp+61F0F55]
005A195A 47 inc edi
005A195B 3B85 9CA72C06 cmp eax,dword ptr ss:[ebp+62CA79C]
005A1961 75 07 jnz short 005A196A
005A1963 8B85 5D2F1F06 mov eax,dword ptr ss:[ebp+61F2F5D]
005A1969 47 inc edi
005A196A 3B85 A4A72C06 cmp eax,dword ptr ss:[ebp+62CA7A4]
005A1970 75 07 jnz short 005A1979
005A1972 8B85 7D2A1F06 mov eax,dword ptr ss:[ebp+61F2A7D]
005A1978 47 inc edi
005A1979 3B85 A8A72C06 cmp eax,dword ptr ss:[ebp+62CA7A8]
005A197F 75 07 jnz short 005A1988
005A1981 8B85 0D281F06 mov eax,dword ptr ss:[ebp+61F280D]
005A1987 47 inc edi
005A1988 3B85 ACA72C06 cmp eax,dword ptr ss:[ebp+62CA7AC]
005A198E 75 07 jnz short 005A1997
005A1990 8B85 A1111F06 mov eax,dword ptr ss:[ebp+61F11A1]
005A1996 47 inc edi
005A1997 3B85 B0A72C06 cmp eax,dword ptr ss:[ebp+62CA7B0]
005A199D 75 07 jnz short 005A19A6
005A199F 8B85 ED251F06 mov eax,dword ptr ss:[ebp+61F25ED]
005A19A5 47 inc edi
005A19A6 3B85 B4A72C06 cmp eax,dword ptr ss:[ebp+62CA7B4]
005A19AC 75 07 jnz short 005A19B5
005A19AE 8B85 95001F06 mov eax,dword ptr ss:[ebp+61F0095]
005A19B4 47 inc edi
005A19B5 3B85 B8A72C06 cmp eax,dword ptr ss:[ebp+62CA7B8]
005A19BB 75 07 jnz short 005A19C4
005A19BD 8B85 9D251F06 mov eax,dword ptr ss:[ebp+61F259D]
005A19C3 47 inc edi
005A19C4 3B85 BCA72C06 cmp eax,dword ptr ss:[ebp+62CA7BC]
005A19CA 75 07 jnz short 005A19D3
005A19CC 8B85 511D1F06 mov eax,dword ptr ss:[ebp+61F1D51]
005A19D2 47 inc edi
005A19D3 3B85 C0A72C06 cmp eax,dword ptr ss:[ebp+62CA7C0]
005A19D9 75 07 jnz short 005A19E2
005A19DB 8B85 55271F06 mov eax,dword ptr ss:[ebp+61F2755]
005A19E1 47 inc edi
005A19E2 3B85 C4A72C06 cmp eax,dword ptr ss:[ebp+62CA7C4]
005A19E8 75 07 jnz short 005A19F1
005A19EA 8B85 C1131F06 mov eax,dword ptr ss:[ebp+61F13C1]
005A19F0 47 inc edi
005A19F1 3B85 CCA72C06 cmp eax,dword ptr ss:[ebp+62CA7CC]
005A19F7 75 07 jnz short 005A1A00
005A19F9 8B85 15011F06 mov eax,dword ptr ss:[ebp+61F0115]
005A19FF 47 inc edi
005A1A00 3B85 C8A72C06 cmp eax,dword ptr ss:[ebp+62CA7C8]
005A1A06 75 07 jnz short 005A1A0F
005A1A08 8B85 9D271F06 mov eax,dword ptr ss:[ebp+61F279D]
005A1A0E 47 inc edi
005A1A0F 3B85 D0A72C06 cmp eax,dword ptr ss:[ebp+62CA7D0]
005A1A15 75 07 jnz short 005A1A1E
005A1A17 8B85 31111F06 mov eax,dword ptr ss:[ebp+61F1131]
005A1A1D 47 inc edi
005A1A1E 3B85 D4A72C06 cmp eax,dword ptr ss:[ebp+62CA7D4]
005A1A24 75 07 jnz short 005A1A2D
005A1A26 8B85 390D1F06 mov eax,dword ptr ss:[ebp+61F0D39]
005A1A2C 47 inc edi
005A1A2D 3B85 D8A72C06 cmp eax,dword ptr ss:[ebp+62CA7D8]
005A1A33 75 07 jnz short 005A1A3C
005A1A35 8B85 65281F06 mov eax,dword ptr ss:[ebp+61F2865]
005A1A3B 47 inc edi
005A1A3C 3B85 DCA72C06 cmp eax,dword ptr ss:[ebp+62CA7DC]
005A1A42 75 07 jnz short 005A1A4B
005A1A44 8B85 A9101F06 mov eax,dword ptr ss:[ebp+61F10A9]
005A1A4A 47 inc edi
005A1A4B 3B85 E0A72C06 cmp eax,dword ptr ss:[ebp+62CA7E0]
005A1A51 75 07 jnz short 005A1A5A
005A1A53 8B85 5D121F06 mov eax,dword ptr ss:[ebp+61F125D]
005A1A59 47 inc edi
005A1A5A 3B85 E4A72C06 cmp eax,dword ptr ss:[ebp+62CA7E4]
005A1A60 75 07 jnz short 005A1A69
005A1A62 8B85 35101F06 mov eax,dword ptr ss:[ebp+61F1035]
005A1A68 47 inc edi
005A1A69 3B85 E8A72C06 cmp eax,dword ptr ss:[ebp+62CA7E8]
005A1A6F 75 07 jnz short 005A1A78
005A1A71 8B85 D1211F06 mov eax,dword ptr ss:[ebp+61F21D1]
005A1A77 47 inc edi
005A1A78 3B85 ECA72C06 cmp eax,dword ptr ss:[ebp+62CA7EC]
005A1A7E 75 07 jnz short 005A1A87
005A1A80 8B85 8D2A1F06 mov eax,dword ptr ss:[ebp+61F2A8D]
005A1A86 47 inc edi
005A1A87 3B85 DD161F06 cmp eax,dword ptr ss:[ebp+61F16DD]
005A1A8D 75 07 jnz short 005A1A96
005A1A8F 8B85 D11E1F06 mov eax,dword ptr ss:[ebp+61F1ED1]
005A1A95 47 inc edi
005A1A96 3B85 F0A72C06 cmp eax,dword ptr ss:[ebp+62CA7F0]
005A1A9C 75 07 jnz short 005A1AA5
005A1A9E 8B85 8D061F06 mov eax,dword ptr ss:[ebp+61F068D]
005A1AA4 47 inc edi
005A1AA5 3B85 F4A72C06 cmp eax,dword ptr ss:[ebp+62CA7F4]
005A1AAB 75 07 jnz short 005A1AB4
005A1AAD 8B85 C91D1F06 mov eax,dword ptr ss:[ebp+61F1DC9]
005A1AB3 47 inc edi
005A1AB4 3B85 F8A72C06 cmp eax,dword ptr ss:[ebp+62CA7F8]
005A1ABA 75 07 jnz short 005A1AC3
005A1ABC 8B85 AD2A1F06 mov eax,dword ptr ss:[ebp+61F2AAD]
005A1AC2 47 inc edi
005A1AC3 0BFF or edi,edi
005A1AC5 0F84 05000000 je 005A1AD0
005A1ACB E9 44FDFFFF jmp 005A1814
005A1AD0 3B85 29231F06 cmp eax,dword ptr ss:[ebp+61F2329]
005A1AD6 0F85 0B000000 jnz 005A1AE7
```



```
005A1ADC     8D85 A46E2B06       lea eax,dword ptr ss:[ebp+62B6EA4]  
005A1AE2     E9 2DFDFFFF        jmp 005A1814  
005A1AE7     3B85 912F1F06      cmp eax,dword ptr ss:[ebp+61F2F91]  
005A1AED     0F85 18000000      jnz 005A1B0B  
005A1AF3     83BD 29A72C06 01   cmp dword ptr ss:[ebp+62CA729],1  
005A1AFA     0F85 0B000000      jnz 005A1B0B  
005A1B00     8D85 E7E25700      lea eax,dword ptr ss:[ebp+57E2E7]  
005A1B06     E9 09FDFFFF        jmp 005A1814  
005A1B0B     3B85 64A72C06      cmp eax,dword ptr ss:[ebp+62CA764]  
005A1B11     0F84 0C000000      je 005A1B23  
005A1B17     3B85 68A72C06      cmp eax,dword ptr ss:[ebp+62CA768]  
005A1B1D     0F85 05000000      jnz 005A1B28  
005A1B23     E9 ECFCFFFF        jmp 005A1814  
005A1B28     BE 00000000        mov esi,0  
005A1B2D     83FE 01            cmp esi,1  
005A1B30     0F85 45000000      jnz 005A1B7B  
005A1B36     3B85 58A72C06      cmp eax,dword ptr ss:[ebp+62CA758]  
005A1B3C     0F85 0B000000      jnz 005A1B4D  
005A1B42     8D85 7F755700      lea eax,dword ptr ss:[ebp+57757F]  
005A1B48     E9 C7FCFFFF        jmp 005A1814  
005A1B4D     3B85 5CA72C06      cmp eax,dword ptr ss:[ebp+62CA75C]  
005A1B53     0F85 0B000000      jnz 005A1B64  
005A1B59     8D85 F5755700      lea eax,dword ptr ss:[ebp+5775F5]  
005A1B5F     E9 B0FCFFFF        jmp 005A1814  
005A1B64     3B85 60A72C06      cmp eax,dword ptr ss:[ebp+62CA760]  
005A1B6A     0F85 0B000000      jnz 005A1B7B  
005A1B70     8D85 3A765700      lea eax,dword ptr ss:[ebp+57763A]  
005A1B76     E9 99FCFFFF        jmp 005A1814  
005A1B7B     8BC0               mov eax,eax  
005A1B7D     BE 01000000        mov esi,1  
005A1B82     0BF6               or esi,esi  
005A1B84     0F85 05000000      jnz 005A1B8F  
005A1B8A     E9 7DFCFFFF        jmp 005A180C  
005A1B8F     8BF0               mov esi,eax  
005A1B91     89B5 71231F06      mov dword ptr ss:[ebp+61F2371],esi  
005A1B97     89B5 210C1F06      mov dword ptr ss:[ebp+61F0C21],esi  
005A1B9D     803E E9            cmp byte ptr ds:[esi],0E9  
005A1BA0     0F85 26000000      jnz 005A1BCC  
005A1BA6     8B7E 01            mov edi,dword ptr ds:[esi+1]  
005A1BA9     03FE               add edi,esi  
005A1BAB     8BDE               mov ebx,esi  
005A1BAD     81C3 00400000      add ebx,4000  
005A1BB3     3BBD 71231F06      cmp edi,dword ptr ss:[ebp+61F2371]  
005A1BB9     0F82 08000000      jb 005A1BC7  
005A1BBF     3BFB               cmp edi,ebx  
005A1BC1     0F86 05000000      jbe 005A1BCC  
005A1BC7     E9 40FCFFFF        jmp 005A180C  
005A1BCC     8BBD 190E1F06      mov edi,dword ptr ss:[ebp+61F0E19]  
005A1BD2     C785 49101F06 0000>mov dword ptr ss:[ebp+61F1049],0  
005A1BDC     60                 pushad  
005A1BDD     89B5 210C1F06      mov dword ptr ss:[ebp+61F0C21],esi  
005A1BE3     8D9D 6CBE2C06      lea ebx,dword ptr ss:[ebp+62CBE6C]  
005A1BE9     FFD3               call ebx  
005A1BEB     0F82 22000000      jb 005A1C13  
005A1BF1     8D9D 604D2A06      lea ebx,dword ptr ss:[ebp+62A4D60]  
005A1BF7     FFD3               call ebx  
005A1BF9     0F83 DEFFFFFF      jnb 005A1BDD  
005A1BFF     8BB5 210C1F06      mov esi,dword ptr ss:[ebp+61F0C21]  
005A1C05     89B5 49101F06      mov dword ptr ss:[ebp+61F1049],esi  
005A1C0B     8D9D E23D2B06      lea ebx,dword ptr ss:[ebp+62B3DE2]  
005A1C11     FFD3               call ebx  
005A1C13     8B85 71231F06      mov eax,dword ptr ss:[ebp+61F2371]  
005A1C19     8985 210C1F06      mov dword ptr ss:[ebp+61F0C21],eax  
005A1C1F     61                 popad  
005A1C20     8D9D 99BA2C06      lea ebx,dword ptr ss:[ebp+62CBA99]  
005A1C26     FFD3               call ebx  
005A1C28     8D9D 1CBB2C06      lea ebx,dword ptr ss:[ebp+62CBB1C]  
005A1C2E     FFD3               call ebx  
005A1C30     8D9D BDBD2C06      lea ebx,dword ptr ss:[ebp+62CBDBD]  
005A1C36     FFD3               call ebx  
005A1C38     0F83 0C000000      jnb 005A1C4A  
005A1C3E     8385 210C1F06 05   add dword ptr ss:[ebp+61F0C21],5  
005A1C45     E9 D6FFFFFF        jmp 005A1C20  
005A1C4A     8D9D E6BD2C06      lea ebx,dword ptr ss:[ebp+62CBDE6]  
005A1C50     FFD3               call ebx  
005A1C52     0F83 08000000      jnb 005A1C60  
005A1C58     83C2 04            add edx,4  
005A1C5B     E9 32000000        jmp 005A1C92  
005A1C60     8D9D 604D2A06      lea ebx,dword ptr ss:[ebp+62A4D60]  
005A1C66     FFD3               call ebx  
005A1C68     0F83 0B000000      jnb 005A1C79  
005A1C6E     8BB5 210C1F06      mov esi,dword ptr ss:[ebp+61F0C21]  
005A1C74     E9 27070000        jmp 005A23A0  
005A1C79     8B8D 210C1F06      mov ecx,dword ptr ss:[ebp+61F0C21]  
005A1C7F     89B5 210C1F06      mov dword ptr ss:[ebp+61F0C21],esi  
005A1C85     2BCE               sub ecx,esi  
005A1C87     F7D9               neg ecx  
005A1C89     2BF1               sub esi,ecx  
005A1C8B     F3:A4              rep movs byte ptr es:[edi],byte ptr ds:[esi]  
005A1C8D     E9 8EFFFFFF        jmp 005A1C20  
005A1C92     8D9D 913D2B06      lea ebx,dword ptr ss:[ebp+62B3D91]  
005A1C98     FFD3               call ebx  
005A1C9A     8BC7               mov eax,edi  
005A1C9C     2B85 190E1F06      sub eax,dword ptr ss:[ebp+61F0E19]  
005A1CA2     8985 59241F06      mov dword ptr ss:[ebp+61F2459],eax  
005A1CA8     8B85 190E1F06      mov eax,dword ptr ss:[ebp+61F0E19]  
005A1CAE     57                 push edi  
005A1CAF     50                 push eax  
005A1CB0     8D8D 9C3E2B06      lea ecx,dword ptr ss:[ebp+62B3E9C]  
005A1CB6     FFD1               call ecx  
005A1CB8     8B85 E1201F06      mov eax,dword ptr ss:[ebp+61F20E1]  
005A1CBE     50                 push eax  
005A1CBF     57                 push edi  
005A1CC0     8B85 190E1F06      mov eax,dword ptr ss:[ebp+61F0E19]  
005A1CC6     50                 push eax  
005A1CC7     8D8D C3402B06      lea ecx,dword ptr ss:[ebp+62B40C3]  
005A1CCD     FFD1               call ecx  
005A1CCF     8BD0               mov edx,eax  
005A1CD1     8BC8               mov ecx,eax  
005A1CD3     2B8D E1201F06      sub ecx,dword ptr ss:[ebp+61F20E1]  
005A1CD9     83BD 5D0A1F06 00   cmp dword ptr ss:[ebp+61F0A5D],0  
005A1CE0     0F84 2B000000      je 005A1D11  
005A1CE6     8B85 01041F06      mov eax,dword ptr ss:[ebp+61F0401]  
005A1CEC     2B85 5D0A1F06      sub eax,dword ptr ss:[ebp+61F0A5D]  
005A1CF2     3BC1               cmp eax,ecx  
005A1CF4     0F86 17000000      jbe 005A1D11  
005A1CFA     8B85 B50E1F06      mov eax,dword ptr ss:[ebp+61F0EB5]  
005A1D00     0385 5D0A1F06      add eax,dword ptr ss:[ebp+61F0A5D]  
005A1D06     8985 ED091F06      mov dword ptr ss:[ebp+61F09ED],eax  
005A1D0C     E9 43000000        jmp 005A1D54  
005A1D11     51                 push ecx  
005A1D12     8BC1               mov eax,ecx  
005A1D14     48                 dec eax  
005A1D15     0D FF0F0000        or eax,0FFF  
005A1D1A     40                 inc eax  
005A1D1B     8985 01041F06      mov dword ptr ss:[ebp+61F0401],eax  
005A1D21     0185 DD091F06      add dword ptr ss:[ebp+61F09DD],eax  
005A1D27     C785 5D0A1F06 0000>mov dword ptr ss:[ebp+61F0A5D],0  
005A1D31     6A 40              push 40  
005A1D33     68 00100000        push 1000  
005A1D38     51                 push ecx  
005A1D39     6A 00              push 0  
005A1D3B     FF95 AD251F06      call dword ptr ss:[ebp+61F25AD]  
005A1D41     FF95 69201F06      call dword ptr ss:[ebp+61F2069]  
005A1D47     8985 B50E1F06      mov dword ptr ss:[ebp+61F0EB5],eax  
005A1D4D     8985 ED091F06      mov dword ptr ss:[ebp+61F09ED],eax  
005A1D53     59                 pop ecx  
005A1D54     FFB5 ED091F06      push dword ptr ss:[ebp+61F09ED]  
005A1D5A     FFB5 E1201F06      push dword ptr ss:[ebp+61F20E1]  
005A1D60     57                 push edi  
```



# 逆向工程分析：DLL 函数处理与补丁注入

## 原始汇编代码片段

```asm
005A1D61     FFB5 190E1F06      push dword ptr ss:[ebp+61F0E19]  
005A1D67     8D85 57432B06      lea eax,dword ptr ss:[ebp+62B4357]  
005A1D6D     FFD0               call eax  
005A1D6F     018D 5D0A1F06      add dword ptr ss:[ebp+61F0A5D],ecx  
005A1D75     8BBD ED091F06      mov edi,dword ptr ss:[ebp+61F09ED]  
005A1D7B     8BB5 E1201F06      mov esi,dword ptr ss:[ebp+61F20E1]  
005A1D81     F3:A4              rep movs byte ptr es:[edi],byte ptr ds:[esi]  
005A1D83     8BB5 65031F06      mov esi,dword ptr ss:[ebp+61F0365]  
005A1D89     AD                 lods dword ptr ds:[esi]  
005A1D8A     C746 FC 00000000   mov dword ptr ds:[esi-4],0  
005A1D91     C1C0 05            rol eax,5  
005A1D94     05 BDEB8C32        add eax,328CEBBD  
005A1D99     0385 D9211F06      add eax,dword ptr ss:[ebp+61F21D9]  
005A1D9F     8B8D ED091F06      mov ecx,dword ptr ss:[ebp+61F09ED]  
005A1DA5     8908               mov dword ptr ds:[eax],ecx ; SHELL32.ShellExecuteA  

//上面 Shift+F9 后中断在这里  
//Patch③、   jmp 005AF000     ★  

005A1DA7     AD                 lods dword ptr ds:[esi]  
005A1DA8     C746 FC 00000000   mov dword ptr ds:[esi-4],0  
005A1DAF     89B5 65031F06      mov dword ptr ss:[ebp+61F0365],esi  
005A1DB5     83F8 FF            cmp eax,-1  
005A1DB8     0F85 20000000      jnz 005A1DDE  
005A1DBE     813E DDDDDDDD      cmp dword ptr ds:[esi],DDDDDDDD  
005A1DC4     0F85 14000000      jnz 005A1DDE  
005A1DCA     C706 00000000      mov dword ptr ds:[esi],0  
005A1DD0     83C6 04            add esi,4  
005A1DD3     89B5 65031F06      mov dword ptr ss:[ebp+61F0365],esi  
005A1DD9     E9 38F8FFFF        jmp 005A1616  
005A1DDE     C1C0 03            rol eax,3  
005A1DE1     0385 D9211F06      add eax,dword ptr ss:[ebp+61F21D9]  
005A1DE7     83BD 212B1F06 01   cmp dword ptr ss:[ebp+61F2B21],1  
005A1DEE     0F84 9D000000      je 005A1E91  
005A1DF4     813E AAAAAAAAA     cmp dword ptr ds:[esi],AAAAAAAA  
005A1DFA     0F85 12000000      jnz 005A1E12  
005A1E00     83C6 04            add esi,4  
005A1E03     C746 FC 00000000   mov dword ptr ds:[esi-4],0  
005A1E0A     97                 xchg eax,edi  
005A1E0B     B0 E9              mov al,0E9  
005A1E0D     E9 03000000        jmp 005A1E15  
005A1E12     97                 xchg eax,edi  
005A1E13     B0 E8              mov al,0E8  
005A1E15     50                 push eax  
005A1E16     83BD DD151F06 01   cmp dword ptr ss:[ebp+61F15DD],1  
005A1E1D     0F84 3E000000      je 005A1E61  
005A1E23     B8 00010000        mov eax,100  
005A1E28     83BD 70A72C06 00   cmp dword ptr ss:[ebp+62CA770],0  
005A1E2F     0F84 08000000      je 005A1E3D  
005A1E35     8D9D B0462B06      lea ebx,dword ptr ss:[ebp+62B46B0]  
005A1E3B     FFD3               call ebx  
005A1E3D     803F 90            cmp byte ptr ds:[edi],90  
005A1E40     0F84 08000000      je 005A1E4E  
005A1E46     83C7 05            add edi,5  
005A1E49     E9 43000000        jmp 005A1E91  
005A1E4E     83F8 50            cmp eax,50  
005A1E51     0F82 0A000000      jb 005A1E61  
005A1E57     B0 90              mov al,90  
005A1E59     AA                 stos byte ptr es:[edi]  
005A1E5A     58                 pop eax  
005A1E5B     AA                 stos byte ptr es:[edi]  
005A1E5C     E9 24000000        jmp 005A1E85  

//Patch④、   jmp 005AF014   ★  

005A1E61     58                 pop eax  
005A1E62     AA                 stos byte ptr es:[edi]  
005A1E63     807F FF E9         cmp byte ptr ds:[edi-1],0E9  
005A1E67     0F85 18000000      jnz 005A1E85  

//Patch⑤、   jmp 005AF036   ★  

005A1E6D     83BD 70A72C06 00   cmp dword ptr ss:[ebp+62CA770],0  
005A1E74     0F84 08000000      je 005A1E82  
005A1E7A     8D9D 80462B06      lea ebx,dword ptr ss:[ebp+62B4680]  
005A1E80     FFD3               call ebx  
005A1E82     8847 04            mov byte ptr ds:[edi+4],al  

//Patch⑥、   NOP    ★  去掉加密填充  

005A1E85     8B85 ED091F06      mov eax,dword ptr ss:[ebp+61F09ED]  
005A1E8B     2BC7               sub eax,edi  
005A1E8D     83E8 04            sub eax,4  
005A1E90     AB                 stos dword ptr es:[edi]  

//Patch⑦、   NOP    ★  去掉加密填充  

005A1E91     AD                 lods dword ptr ds:[esi]  
005A1E92     C746 FC 00000000   mov dword ptr ds:[esi-4],0  
005A1E99     E9 11FFFFFF        jmp 005A1DAF  

//循环处理每个 DLL 的函数  

//Patch⑧、   jmp 005AF05F     ★  

005A1E9E     89B5 65031F06      mov dword ptr ss:[ebp+61F0365],esi  
005A1EA4     52                 push edx  
005A1EA5     68 00800000        push 8000  
005A1EAA     6A 00              push 0  
005A1EAC     FFB5 AD2F1F06      push dword ptr ss:[ebp+61F2FAD]  
005A1EB2     FF95 B1241F06      call dword ptr ss:[ebp+61F24B1]  
005A1EB8     5A                 pop edx  
005A1EB9     8B8D 51111F06      mov ecx,dword ptr ss:[ebp+61F1151]  
005A1EBF     C701 00000000      mov dword ptr ds:[ecx],0  
005A1EC5     83C1 04            add ecx,4  
005A1EC8     898D 51111F06      mov dword ptr ss:[ebp+61F1151],ecx  
005A1ECE     E9 62F6FFFF        jmp 005A1535  

//循环处理所有 DLL 的函数  

005A1ED3     E9 4B060000        jmp 005A2523  

//此处下断，输入表处理完成后中断在这里
```

---

## 补丁策略说明

在下面找一段空地写 Patch 代码，放 `005AF000` 处吧。

## Patch ③ 代码

原始跳转点：
```asm
005A1DA5     8908               mov dword ptr ds:[eax],ecx ; SHELL32.ShellExecuteA  
//Patch③、   jmp 005AF000     ★  
```

补丁实施代码：
```asm
005AF000     A3 00F45A00        mov dword ptr ds:[5AF400],eax  

//保存 EAX 值于[5AF400]  

005AF005     8908               mov dword ptr ds:[eax],ecx  

//005A1DA5 及其下 3 行代码挪这里执行  

005AF007     AD                 lods dword ptr ds:[esi]  
005AF008     C746 FC 00000000   mov dword ptr ds:[esi-4],0  
005AF00F     E9 9B2DFFFF        jmp 005A1DAF  

//返回去继续流程
```

---

## Patch ④ 代码

原始跳转点：
```asm
005A1E5C     E9 24000000        jmp 005A1E85  

//Patch④、   jmp 005AF014   ★  
```

补丁实施代码：
```asm
005B9014     50                 push eax  
005B9015     A1 00945B00        mov eax,dword ptr ds:[5B9400]  
005B901A     8947 FC            mov dword ptr ds:[edi-4],eax  

//放入正确的 API 保存地址  

005B901D     807F FB E8         cmp byte ptr ds:[edi-5],0E8  

//E8 ？  

005B9021     75 08              jnz short 005B902B  
005B9023     66:C747 FA FF15    mov word ptr ds:[edi-6],15FF  

//则是 call dword ptr ds:[XXXXXXXX]  

005B9029     EB 06              jmp short 005B9031  
005B902B     66:C747 FA FF25    mov word ptr ds:[edi-6],25FF  

//否则是 jmp dword ptr ds:[XXXXXXXX]  

005B9031     58                 pop eax  
005B9032     E9 0026FFFF        jmp 005AB637  

//继续流程
```

---

## Patch ⑤ 代码

原始跳转点：
```asm
005A1E67     0F85 18000000      jnz 005A1E85  

//Patch⑤、   jmp 005AF036   ★  
```

补丁实施代码：
```asm
005AF036     50                 push eax  
005AF037     A1 00F45A00        mov eax,dword ptr ds:[5AF400]  
005AF03C     8947 01            mov dword ptr ds:[edi+1],eax  
005AF03F     807F FF E8         cmp byte ptr ds:[edi-1],0E8  
005AF043     75 08              jnz short 005AF04D  
005AF045     66:C747 FF FF15    mov word ptr ds:[edi-1],15FF  
005AF04B     EB 06              jmp short 005AF053  
005AF04D     66:C747 FF FF25    mov word ptr ds:[edi-1],25FF  
005AF053     58                 pop eax  
005AF054     0F85 2B2EFFFF      jnz 005A1E85  
005AF05A     E9 0E2EFFFF        jmp 005A1E6D
```

---

## Patch ⑥ 与 ⑦ 代码

原始跳转点（Patch ⑥）：
```asm
//Patch⑥、   NOP    ★  去掉加密填充  
```

原始跳转点（Patch ⑦）：
```asm
005A1E90     AB                 stos dword ptr es:[edi]  

//Patch⑦、   NOP    ★  去掉加密填充  
```

---

## Patch ⑧ 代码

原始跳转点：
```asm
005A1E99     E9 11FFFFFF        jmp 005A1DAF  

//循环处理每个 DLL 的函数  

//Patch⑧、   jmp 005AF05F     ★  
```

补丁实施代码：
```asm
005AF05F     83C7 04            add edi,4  
005AF062     E9 482DFFFF        jmp 005A1DAF  

//继续流程
```

---

## 补丁代码汇总

```asm
005AF000     A3 00F45A00        mov dword ptr ds:[5AF400],eax  
```



### 二进制代码

```
005AF005     8908              mov dword ptr ds:[eax],ecx  

005AF007     AD                lods dword ptr ds:[esi]  

005AF008     C746 FC 00000000  mov dword ptr ds:[esi-4],0  

005AF00F     E9 9B2DFFFF       jmp 005A1DAF  

005AF014     50                push eax  

005AF015     A1 00F45A00       mov eax,dword ptr ds:[5AF400]  

005AF01A     8907              mov dword ptr ds:[edi],eax  

005AF01C     807F FF E8        cmp byte ptr ds:[edi-1],0E8  

005AF020     75 08             jnz short 005AF02A  

005AF022     66:C747 FE FF15  mov word ptr ds:[edi-2],15FF  

005AF028     EB 06             jmp short 005AF030  

005AF02A     66:C747 FE FF25  mov word ptr ds:[edi-2],25FF  

005AF030     58                pop eax  

005AF031     E9 4F2EFFFF       jmp 005A1E85  

005AF036     50                push eax  

005AF037     A1 00F45A00       mov eax,dword ptr ds:[5AF400]  

005AF03C     8947 01           mov dword ptr ds:[edi+1],eax  

005AF03F     807F FF E8        cmp byte ptr ds:[edi-1],0E8  

005AF043     75 08             jnz short 005AF04D  

005AF045     66:C747 FF FF15  mov word ptr ds:[edi-1],15FF  

005AF04B     EB 06             jmp short 005AF053  

005AF04D     66:C747 FF FF25  mov word ptr ds:[edi-1],25FF  

005AF053     58                pop eax  

005AF054     0F85 2B2EFFFF     jnz 005A1E85  

005AF05A     E9 0E2EFFFF       jmp 005A1E6D  

005AF05F     83C7 04           add edi,4  

005AF062     E9 482DFFFF       jmp 005A1DAF  
```

从 `OllyDBG` 中二进制代码复制如下：

```
A3 00 F4 5A 00 89 08 AD C7 46 FC 00 00 00 00 E9 9B 2D FF FF 50 A1 00 F4 5A 00 89 07 80 7F FF E8  

75 08 66 C7 47 FE FF 15 EB 06 66 C7 47 FE FF 25 58 E9 4F 2E FF FF 50 A1 00 F4 5A 00 89 47 01 80  

7F FF E8 75 08 66 C7 47 FF FF 15 EB 06 66 C7 47 FF FF 25 58 0F 85 2B 2E FF FF E9 0E 2E FF FF 83  

C7 04 E9 48 2D FF FF  
```

### 三、OEP

```
005A1ED3     E9 4B060000       jmp 005A2523  
```

//此处下断，输入表处理完成后中断在这里

对于 `Themida` 处理后的 `OEP` 查找的确有点麻烦，目前还没有发现更简便的方法。

保持此 `OllyDBG`，新开一个 `OllyDBG`，载入 `Themida V1.1.1.0.Test.eXe`

直接 `Shift+F9` 让其运行起来，然后 `Ctrl+G`：`005A1ED3`，来到输入表处理结束的地方

`Ctrl+B` 在整个段块搜索 Hex 值：`9D C3 E9`

```
005A8C29     9D                popfd  
```

//找到这里

```
005A8C2A     C3                retn  

005A8C2B     E9 08000000       jmp 005A8C38  
```

现在可以关闭后开的 `OllyDBG` 了，在第一个 `OllyDBG` 里面下断：HE `005A8C29`

单击 `Themida` 的启动 Nag，`OllyDBG` 中断在 `005A8C29` 处

```
005A8C29     9D                popfd  
```

//中断在这里

```
005A8C2A     C3                retn  
```

//返回 `005A08D3` 飞向“光明之巅”

```
005A08D3     68 ACE9750C      push 0C75E9AC  
```

//作为 `OEP` 吧 下面 `Themida` 就开始 `OEP` 处理了

```
005A08D8     E9 2BFBFFFF      jmp 005A0408  

005A08DD     68 48F5D325      push 25D3F548  

005A08E2     E9 21FBFFFF      jmp 005A0408  
```

运行 `ImportREC`，填入 `OEP RVA=001A08D3`、`IAT RVA=000062E0`、`IAT Size=0000023C`，获取输入表。

运行 `LordPE` 先 dump full 此进程，存为 `dump.eXe`，修复输入表。

`Themida` 的 `Virtual Machine` 是不容易还原的，补上这部分壳代码吧

`OllyDBG` 中 `Alt+M` 察看内存：

| 地址 | 大小（十进制） | 物主 | 区段 | 包含 |
| :--- | :--- | :--- | :--- | :--- |
| `00400000` | `00001000` (4096.) | `Themida_` `00400000` | | `PE header` |
| `00401000` | `00006000` (24576.) | `Themida_` `00400000` | | `code` |
| `00407000` | `00005000` (20480.) | `Themida_` `00400000` | `.rsrc` | `data,resources` |
| `0040C000` | `00001000` (4096.) | `Themida_` `00400000` | `.idata` | `imports` |
| `0040D000` | `001A5000` (1724416.) | `Themida_` `00400000` | `Themida` | `SFX` |
| `005C0000` | `00009000` (36864.) | | `005C0000` | |
| `006C0000` | `0003D000` (249856.) | | `006C0000` | |
| `00700000` | `00041000` (266240.) | | `00700000` | |
| `00750000` | `00006000` (24576.) | | `00750000` | |
| `00760000` | `00041000` (266240.) | | `00760000` | |
| `007B0000` | `00009000` (36864.) | | `007B0000` | |
| `00870000` | `00002000` (8192.) | | `007B0000` | |
| `00880000` | `00103000` (1060864.) | | `00880000` | |
| `00990000` | `00006000` (24576.) | | `00990000` | |
| `009A0000` | `0016A000` (1482752.) | | `009A0000` | |
| `00CA0000` | `00003000` (12288.) | | `00CA0000` | |
| `00CB0000` | `00008000` (32768.) | | `00CB0000` | |
| `00CC0000` | `00001000` (4096.) | | `00CC0000` | |
| `00CD0000` | `00001000` (4096.) | | `00CD0000` | |
| `00CE0000` | `00004000` (16384.) | | `00CE0000` | |
| `00CF0000` | `00002000` (8192.) | | `00CF0000` | |
| `00D00000` | `00001000` (4096.) | | `00D00000` | |
| `00D10000` | `00001000` (4096.) | | `00D10000` | |
| `00D20000` | `00001000` (4096.) | | `00D20000` | |
| `00D30000` | `00010000` (65536.) | | `00D30000` | |
| `00D40000` | `00010000` (65536.) | | `00D40000` | |
| `00D50000` | `00010000` (65536.) | | `00D50000` | |
| `00D60000` | `00010000` (65536.) | | `00D60000` | |
| `00D70000` | `00010000` (65536.) | | `00D70000` | |
| `00D80000` | `00010000` (65536.) | | `00D80000` | |
| `00D90000` | `00010000` (65536.) | | `00D90000` | |
| `00DA0000` | `00010000` (65536.) | | `00DA0000` | |
| `00DB0000` | `00001000` (4096.) | | `00DB0000` | |

用 `LordPE` Dump Region 以下壳区段：

*   `00CF0000-00CF2000.dmp`
*   `00D00000-00D01000.dmp`
*   `00D10000-00D11000.dmp`
*   `00D20000-00D21000.dmp`
*   `00D30000-00DB1000.dmp`

用 `LordPE` 把这些区段 load 入 `dump.eXe`，注意修正各区段的 `Voffset`

只保留“Validate PE”选项来 Rebuilder `dump.eXe`

`F7` 继续走，看看为何应该补这些壳代码段

```
005A0408     6A 00             push 0  

005A040A     9C                pushfd  

005A040B     60                pushad  

005A040C     E8 00000000       call 005A0411  

005A0411     5D                pop ebp  

005A0412     81ED 1D9E2C06     sub ebp,62C9E1D  

005A0418     B8 F3A22C06       mov eax,62CA2F3  

005A041D     03C5              add eax,ebp  

005A041F     50                push eax  

005A0420     8BB5 7D131F06     mov esi,dword ptr ss:[ebp+61F137D]  

005A0426     BB 01000000       mov ebx,1  

005A042B     8D46 28           lea eax,dword ptr ds:[esi+28]  

005A042E     F0:8618           lock xchg byte ptr ds:[eax],bl  

005A0431     0ADB              or bl,bl  

005A0433     75 02             jnz short 005A0437  

005A0435     EB 0C             jmp short 005A0443  

005A0437     60                pushad  

005A0438     6A 00             push 0  

005A043A     FF95 B9291F06     call dword ptr ss:[ebp+61F29B9]  

005A0440     61                popad  

005A0441     EB EB             jmp short 005A042E  

005A0443     58                pop eax  

005A0444     8946 68           mov dword ptr ds:[esi+68],eax  

005A0447     B8 02000000       mov eax,2  

005A044C     8946 6C           mov dword ptr ds:[esi+6C],eax  

005A044F     C74424 24 D9401F06 mov dword ptr ss:[esp+24],61F40D9  

005A0457     016C24 24         add dword ptr ss:[esp+24],ebp  

005A045B     61                popad  

005A045C     9D                popfd  

005A045D     C3                retn  
```

//返回到 `004CA6CD`

```
004CA6CD     9C                pushfd  

004CA6CE     60                pushad  

004CA6CF     E8 00000000       call 004CA6D4  

004CA6D4     5D                pop ebp  

004CA6D5     81ED E0401F06     sub ebp,61F40E0  

004CA6DB     8BB5 7D131F06     mov esi,dword ptr ss:[ebp+61F137D]  

004CA6E1     8B0424            mov eax,dword ptr ss:[esp]  

004CA6E4     8986 9C000000     mov dword ptr ds:[esi+9C],eax  

004CA6EA     8B4424 04         mov eax,dword ptr ss:[esp+4]  

004CA6EE     8986 94000000     mov dword ptr ds:[esi+94],eax  

004CA6F4     8B4424 08         mov eax,dword ptr ss:[esp+8]  

004CA6F8     8986 A4000000     mov dword ptr ds:[esi+A4],eax  

004CA6FE     8B4424 0C         mov eax,dword ptr ss:[esp+C]  

004CA702     83C0 08           add eax,8  

004CA705     8986 AC000000     mov dword ptr ds:[esi+AC],eax  

004CA70B     8B4424 10         mov eax,dword ptr ss:[esp+10]  

004CA70F     8946 7C           mov dword ptr ds:[esi+7C],eax  

004CA712     8B4424 14         mov eax,dword ptr ss:[esp+14]  

004CA716     8986 8C000000     mov dword ptr ds:[esi+8C],eax  
```



```
004CA71C     8B4424 18            mov eax,dword ptr ss:[esp+18]  
004CA720     8986 84000000       mov dword ptr ds:[esi+84],eax  
004CA726     8B4424 1C           mov eax,dword ptr ss:[esp+1C]  
004CA72A     8946 74             mov dword ptr ds:[esi+74],eax  
004CA72D     8B4424 20           mov eax,dword ptr ss:[esp+20]  
004CA731     8946 70             mov dword ptr ds:[esi+70],eax  
004CA734     66:8CC8             mov ax,cs  
004CA737     66:8986 E4000000    mov word ptr ds:[esi+E4],ax  
004CA73E     66:8CD8             mov ax,ds  
004CA741     66:8986 E6000000    mov word ptr ds:[esi+E6],ax  
004CA748     66:8CC0             mov ax,es  
004CA74B     66:8986 E8000000    mov word ptr ds:[esi+E8],ax  
004CA752     66:8CD0             mov ax,ss  
004CA755     66:8986 EA000000    mov word ptr ds:[esi+EA],ax  
004CA75C     C746 38 000000F0    mov dword ptr ds:[esi+38],F0000000  
004CA763     8BB5 7D131F06       mov esi,dword ptr ss:[ebp+61F137D]  
004CA769     8B7E 68             mov edi,dword ptr ds:[esi+68]  
004CA76C     8B07                mov eax,dword ptr ds:[edi]  
004CA76E     03C5                add eax,ebp  
004CA770     8946 4C             mov dword ptr ds:[esi+4C],eax  
004CA773     0347 04             add eax,dword ptr ds:[edi+4]  
004CA776     8946 50             mov dword ptr ds:[esi+50],eax  
004CA779     8B4424 24           mov eax,dword ptr ss:[esp+24]  
004CA77D     8B76 68             mov esi,dword ptr ds:[esi+68]  
004CA780     83C6 08             add esi,8  
004CA783     E9 03000000         jmp 004CA78B  
004CA788     83C6 0C             add esi,0C  
004CA78B     3906                cmp dword ptr ds:[esi],eax  
004CA78D     0F85 F5FFFFFF        jnz 004CA788  
004CA793     8B46 04             mov eax,dword ptr ds:[esi+4]  
004CA796     03C5                add eax,ebp  
004CA798     8BBD 7D131F06       mov edi,dword ptr ss:[ebp+61F137D]  
004CA79E     8907                mov dword ptr ds:[edi],eax  
004CA7A0     8BA5 D5401F06       mov esp,dword ptr ss:[ebp+61F40D5]  
004CA7A6     81C4 FC1F0000       add esp,1FFC  
004CA7AC     FF67 58             jmp dword ptr ds:[edi+58]  

//走到这里   ★  
//[edi+58]=[00DB0058]=00DB07D0   修补代码结束地址所在段   ★  
//ESP=00CF1FFC                   修补代码开始地址所在段   ★
```

注意：如果刚开始时没有修改 `SizeOfStackReserve` 值为 `00380000`，则此程序这里的值低于基址，导致不好处理。

修补区段后导致 `dumped_.eXe` 巨大，呵呵，此脱壳没啥意义，聊作游戏。

最后再申明：不要用本文的方法去 `OllyDBG` 调试有驱动的其他版本 `Themida`，那样只会让你的电脑重启。

```
—————————————————————————————————

        ,     \_/  
      /| \_.-~/            \\_     ,        青春都一晌
     ( /~   /               \\~.\_ |\\
     `\\    \_/               \    ~\ )          忍把浮名  
 _-~~~-.)  )\_//;;,.           \\_  //'
/'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了破解轻狂  
\`~ \_( ,\_..--\\ (      ,;'' /    ~--   /.\_\`\\  
 /~~//'   /`~\\           ) /--..\_, )\_  \`~
  "  \`~"  "       `"      /~'\`\\    \`\\\\~~\\  
                          "      "   "~'  ""
                    UnPacKed By :  fly
                       2005-12-02 22:00
```

`4dd466d0e1e0cc123afbe504da2462ca`
