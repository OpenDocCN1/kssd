
# `iRider.exe`

**2.20BETA主程序脱壳 Armadillo 1.xx - 2.xx**

Armadillo 1.xx - 2.xx这个版本的壳大概已经不多贱了吧，下载了各国外的软件居然是用它加壳的，正好让我这个菜鸟练练。我这次脱壳是参看了leo_cyl1的“Armadillo 2.52加壳原理分析和改进的脱壳方法”一文，fly兄的“壹次脱壳法――Armadillo 双进程标准壳 快速脱壳”一文，weiyi75兄的“爱的中体验之Armadillo3.x双进程之Mr.Captor”一文（初学破解，weiyi75是谁，我不知道，呵呵，真想认识一下）。

要不是几位托神的脱壳教程，恐怕我这辈子也不知道还有个“双进程”。

- 软件名称：`iRider.exe` 2.20BETA
- 软件大小：3.61MB
- 加壳方式：Armadillo 1.xx - 2.xx
- 破解工具：OllyDBG1.10 fly修改版，ImportREC1.6

软件简介：iRider是一种让你能够快速，方便的熟练操作打开成倍网页的新一代浏览器，并且在你工作时候能够更快的浏览和下载页面.

iRider使用方便是因为它是使用了十分简单并且功能强大的方式的非常人性化网页浏览器，而且能使用你十分常用的操作方式来打开你频繁使用的书签内的所以站点.

## 一、寻找OEP

```
004A50B9 i>/$  55                push ebp                //OD载入，停在这里，下断点bp OpenMuteA，F9运行
004A50BA   |.  8BEC              mov ebp,esp
004A50BC   |.  6A FF             push -1
004A50BE   |.  68 68D24B00       push iRider.004BD268
004A50C3   |.  68 004B4A00       push iRider.004A4B00                 ;  SE handler installation
004A50C8   |.  64:A1 00000000    mov eax,dword ptr fs:[0]
004A50CE   |.  50                push eax
004A50CF   |.  64:8925 00000000  mov dword ptr fs:[0],esp
.................
```

```
77E62391 k>- E9 E8EBFDFF         jmp kernel32.77E40F7E        //到这，看堆栈
77E62396     837D 10 00          cmp dword ptr ss:[ebp+10],0
77E6239A     56                  push esi
77E6239B     0F84 C2E30100       je kernel32.77E80763
77E623A1     64:A1 18000000      mov eax,dword ptr fs:[18]
77E623A7     FF75 10             push dword ptr ss:[ebp+10]
77E623AA     8DB0 F80B0000       lea esi,dword ptr ds:[eax+BF8]
77E623B0     8D45 F8             lea eax,dword ptr ss:[ebp-8]
77E623B3     50                  push eax
```

堆栈：

```
0012F5B8    0049E90C  /CALL 到 OpenMutexA 来自 iRider.0049E906
0012F5BC    001F0001  |Access = 1F0001
0012F5C0    00000000  |Inheritable = FALSE
0012F5C4    0012FBF8  \MutexName = "AD4::DA9B953645"                                    //注意
0012F5C8    0012FF2C
```

Ctrl+G 401000

```
00401000     0000                add byte ptr ds:[eax],al
00401002     0000                add byte ptr ds:[eax],al
00401004     0000                add byte ptr ds:[eax],al
00401006     0000                add byte ptr ds:[eax],al
00401008     0000                add byte ptr ds:[eax],al
0040100A     0000                add byte ptr ds:[eax],al
0040100C     0000                add byte ptr ds:[eax],al
0040100E     0000                add byte ptr ds:[eax],al
```

OD直接双击修改,填入以下代码。

```
00401000    60              PUSHAD
00401001    9C              PUSHFD
00401002    68 DCFB1200     PUSH 12FBF8                     //堆栈内容
00401007    33C0            XOR EAX,EAX
00401009    50              PUSH EAX
0040100A    50              PUSH EAX
0040100B    E8 687BA677     CALL KERNEL32.CreateMutexA
00401010    9D              POPFD
00401011    61              POPAD
00401012  - E9 75C7A677     JMP KERNEL32.OpenMutexA
```

F9，运行，再次中断在这里，此时Ctrl+G 401000 撤销刚才的修改

```
77E62391 k>- E9 E8EBFDFF         jmp kernel32.77E40F7E
77E62396     837D 10 00          cmp dword ptr ss:[ebp+10],0
77E6239A     56                  push esi
77E6239B     0F84 C2E30100       je kernel32.77E80763
77E623A1     64:A1 18000000      mov eax,dword ptr fs:[18]
77E623A7     FF75 10             push dword ptr ss:[ebp+10]
77E623AA     8DB0 F80B0000       lea esi,dword ptr ds:[eax+BF8]
77E623B0     8D45 F8             lea eax,dword ptr ss:[ebp-8]
77E623B3     50                  push eax
```

这里，对`VirtualProtect`下硬件执行断点，运行

```
77E4169E k>  55                  push ebp               //中断，F9运行
77E4169F     8BEC                mov ebp,esp
77E416A1     FF75 14             push dword ptr ss:[ebp+14]
77E416A4     FF75 10             push dword ptr ss:[ebp+10]
77E416A7     FF75 0C             push dword ptr ss:[ebp+C]
77E416AA     FF75 08             push dword ptr ss:[ebp+8]
77E416AD     6A FF               push -1
77E416AF     E8 A4BB0100         call kernel32.VirtualProtectEx
77E416B4     5D                  pop ebp
77E416B5     C2 1000             retn 10
```

直到堆栈中第二次出现

```
0012DEE4    003E30D2  /CALL 到 VirtualProtect 来自 003E30D0
0012DEE8    00400000  |Address = iRider.00400000
0012DEEC    00000040  |Size = 40 (64.)
0012DEF0    00000004  |NewProtect = PAGE_READWRITE                                        //注意观看
0012DEF4    0012DF10  \pOldProtect = 0012DF10
```

Ctrl+F9

```
003E30D2     8B45 F4             mov eax,dword ptr ss:[ebp-C]         ; iRider.00400000
003E30D5     8D4D FC             lea ecx,dword ptr ss:[ebp-4]
003E30D8     2BDF                sub ebx,edi
003E30DA     51                  push ecx
003E30DB     0158 3C             add dword ptr ds:[eax+3C],ebx                             //这里是修改文件头，修改ebx为0
003E30DE     FF75 FC             push dword ptr ss:[ebp-4]
003E30E1     6A 40               push 40
003E30E3     50                  push eax
```

F8一步步运行，注意堆栈变化

```
003E30FF     8D4D F8             lea ecx,dword ptr ss:[ebp-8]
003E3102     E8 F9DEFEFF         call 003D1000
003E3107     83E0 03             and eax,3
003E310A     8D4D F8             lea ecx,dword ptr ss:[ebp-8]
003E310D     40                  inc eax
003E310E     66:0147 06          add word ptr ds:[edi+6],ax                            //这里修改块数，改ax为0
003E3112     E8 E9DEFEFF         call 003D1000
```

以后就是要小心了跟踪了，我没什么经验，只得一路F8，直到看到两个连续的call便进入

```
003E3C42     3348 44             xor ecx,dword ptr ds:[eax+44]
003E3C45     6A 00               push 0
003E3C47     3348 08             xor ecx,dword ptr ds:[eax+8]
003E3C4A     03F9                add edi,ecx
003E3C4C     E8 D791FFFF         call 003DCE28
003E3C51     50                  push eax
003E3C52     FFD7                call edi                             ; iRider.0043FE5B                 //oep
003E3C54     8BD8                mov ebx,eax
```

到达OEP处用插件直接脱壳，就可以了。

## 二、修复输入表

脱壳后，用ImproREC试着修复，当然不成功了，随便着一个未修复的指针，然后用OD重新载入程序，直接`d ******`，其他步骤同上，观察数据窗口，看IAT何时被恢复，记下按F9的次数`n`，从头再来，按`n-1`次F9，然后下断点`bp GetModuleHandleA`，F9运行

中断后，Ctrl+F9回到程序领空，几下F8，来到这里

```
003D4B74     8B4D 08             mov ecx,dword ptr ss:[ebp+8]
003D4B77     3BC8                cmp ecx,eax
003D4B79     75 07               jnz short 003D4B82
003D4B7B     B8 18D33E00         mov eax,3ED318
003D4B80     EB 30               jmp short 003D4BB2
003D4B82     393D D8D73E00       cmp dword ptr ds:[3ED7D8],edi
003D4B88     B8 D8D73E00         mov eax,3ED7D8
003D4B8D     74 0C               je short 003D4B9B
003D4B8F     3B48 08             cmp ecx,dword ptr ds:[eax+8]
003D4B92     EB 1B               je short 003D4BAF                           //Magic Jump，nop it
003D4B94     83C0 0C             add eax,0C
003D4B97     3938                cmp dword ptr ds:[eax],edi
003D4B99   ^ 75 F4               jnz short 003D4B8F
003D4B9B     FF75 0C             push dword ptr ss:[ebp+C]
003D4B9E     FF75 08             push dword ptr ss:[ebp+8]
003D4BA1     E8 41000000         call 003D4BE7
```

从ImprotREC中找到最后一个指针的地址，记住它，在OD中对它下内存访问断点，F9，中断后Ctrl+G 3D4B92，撤销刚才的修改，F9，正常运行程序。ImprotREC恢复，全部指针有效，fix it。

## 三、修复程序

fix后的程序不能正常运行，程序提示“Registration error 86. Please contact iRider support.”，注意是程序提示，不是系统的提示，好办，W32dasm分析，找到改字符串，发现有很多，统统改掉。下面是其中的几处

```
0042EA76     FF15 94424700       call dword ptr ds:[<&kernel32.GetEnv>; kernel32.GetEnvironmentVariableA
0042EA7C     85C0                test eax,eax
0042EA7E     EB 1C               jnz short dump_.0042EA9C                                        //改为jmp
0042EA80     33DB                xor ebx,ebx
0042EA82     68 00200000         push 2000
0042EA87     53                  push ebx
0042EA88     68 A4C84800         push dump_.0048C8A4                  ; ASCII "Registration error 86. Please contact iRider support."
0042EA8D     53                  push ebx
0042EA8E     FF15 04474700       call dword ptr ds:[<&user32.MessageB>; USER32.MessageBoxA
0042EA94     6A 56               push 56
0042EA96     FF15 D4434700       call dword ptr ds:[<&kernel32.ExitPr>; kernel32.ExitProcess
0042EA9C     8D4C24 10           lea ecx,dword ptr ss:[esp+10]
0042EAA0     E8 ED2C0200         call dump_.00451792
```
