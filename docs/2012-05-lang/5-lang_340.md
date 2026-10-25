
# Excel To EXE 软件 v2.1

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-438.htm`](https://www.pediy.com/kssd/pediy08/pediy8-438.htm)

## 软件信息

**【软件名称】** `Excel To EXE 软件 v2.1 版!`

**【软件介绍】**

作用:
1. 跳过宏提示
2. 取代了 `EXCEL` 启动界面
3. `VBE` 窗口屏蔽

最新版增加:
1. 修改后可以保存
2. 工程不可见制作 (前题是 `VBA` 工程先加密)

`2004.6.24` 日增加功能:
1. 利用系统钩子函数，监视剪贴板, 防止程序破解
2. `Task Manager` 中程序隐藏, 屏蔽 `Ctrl+Alt+Del`. 保护进程.
3. 动态屏蔽 `Standard`, `File` 菜单栏中 `Save as...` 键, 暂时针对英文系统. 中文其实也一样. 本人是 (`WINXP`+`Office2003` 英文版).
4. 注册码写入程序中, 确保制作后的程序在不同机器均可以获得完全权限. 当然: 保护是相对的, 目前还有两个漏洞但其实可以防止. 这次没做, 哈 哈. 仅作个人研究用！！想要保护代码还可以做成 `DLL` 或 `COM-ADDIN` 等

再次更新:
1. 暂时文件存放增加到 `3` 个位置. 比较难以发现.
2. `API` 函数屏蔽 `VBE` 窗体, 避免了 `2003` 下要先选择 `Trust Access to VB Project` 才起作用的漏洞
3. 回写时用的全路径, 这次可以实现回写存盘了. 在 `2000`, `2003` 下都已调试成功.

**【破文作者】** `KiLlL[DFCG]`

**【破解时间】** `2005-8-12 20:36`

## 破解过程

拿到这个程序一看，是 `vb` 的，`peid` 得知，无壳，我喜欢！

`OD` 载入后，运行，提示“没有注册”，`BPX MESSAGEBOXA`

```
0043CB48    FF15 A8104000   call dword ptr ds:[<&MSVBVM60.#595<\] ; MSVBVM60.rtcMsgBox
0043CB4E    8D4D 80         lea ecx,dword ptr ss:[ebp\-80\]
```

顺着这里向上看，找到了最前面：

```
00440A26    89B5 60FFFFFF   mov dword ptr ss:[ebp\-A0\],esi
00440A2C    89B5 50FFFFFF   mov dword ptr ss:[ebp\-B0\],esi
```

下断，随便输入假码，点注册后断掉：

```
00440A93    8B45 D4         mov eax,dword ptr ss:[ebp\-2C\]        ; 假码
00440A96    50              push eax
00440A97    68 3C304400     push ExcelToE.0044303C
00440A9C    68 38304400     push ExcelToE.00443038
00440AA1    E8 9AC7FFFF     call ExcelToE.0043D240                 ; 关键 call
00440AA6    8B1D 34124000   mov ebx,dword ptr ds:[<&MSVBVM60.\_\_v<; MSVBVM60.\_\_vbaStrMove
00440AAC    8BD0            mov edx,eax                            ; 497563918
00440AAE    8D4D D0         lea ecx,dword ptr ss:[ebp\-30\]
00440AB1    FFD3            call ebx
00440AB3    50              push eax
00440AB4    FF15 F0104000   call dword ptr ds:[<&MSVBVM60.\_\_vbaS<; MSVBVM60.\_\_vbaStrCmp
00440ABA    8BF0            mov esi,eax
00440ABC    8D4D D0         lea ecx,dword ptr ss:[ebp\-30\]
00440ABF    F7DE            neg esi
00440AC1    1BF6            sbb esi,esi
00440AC3    8D55 D4         lea edx,dword ptr ss:[ebp\-2C\]
00440AC6    51              push ecx
00440AC7    46              inc esi
00440AC8    52              push edx
00440AC9    6A 02           push 2
00440ACB    F7DE            neg esi
00440ACD    FF15 C4114000   call dword ptr ds:[<&MSVBVM60.\_\_vbaF<; MSVBVM60.\_\_vbaFreeStrList
00440AD3    83C4 0C         add esp,0C
00440AD6    8D4D C0         lea ecx,dword ptr ss:[ebp\-40\]
00440AD9    FF15 60124000   call dword ptr ds:[<&MSVBVM60.\_\_vbaF<; MSVBVM60.\_\_vbaFreeObj
00440ADF    66:85F6         test si,si                             ; 关键跳转
00440AE2    0F84 79040000   je ExcelToE.00440F61
```

这里一个 `call` 之后有 `vbaStrCmp`，还有跳转，比较可以，于是在 `00440AA1` 跟入：

```
0043D2E2    C785 2CFFFFFF 0<mov dword ptr ss:[ebp-D4\],4008
0043D2EC    FFD7            call edi                               ; MSVBVM60.rtcLenCharVar
0043D2EE    8D45 BC         lea eax,dword ptr ss:[ebp\-44\]
0043D2F1    50              push eax                               ; 取姓名的长度，我这里是 Master
0043D2F2    FF15 84114000   call dword ptr ds:[<&MSVBVM60.\_\_vbaI<; MSVBVM60.\_\_vbaI2Var
0043D2F8    8D4D BC         lea ecx,dword ptr ss:[ebp\-44\]
0043D2FB    8985 BCFEFFFF   mov dword ptr ss:[ebp\-144\],eax
0043D301    BE 01000000     mov esi,1
0043D306    FF15 20104000   call dword ptr ds:[<&MSVBVM60.\_\_vbaF<; MSVBVM60.\_\_vbaFreeVar
0043D30C    8B1D 3C104000   mov ebx,dword ptr ds:[<&MSVBVM60.\_\_v<; MSVBVM60.\_\_vbaFreeVarList
0043D312    66:3BB5 BCFEFF<cmp si,word ptr ss:[ebp-144\]           ; 设置循环
0043D319    0F8F 8D000000   jg ExcelToE.0043D3AC
0043D31F    8B4D 08         mov ecx,dword ptr ss:[ebp+8\]
0043D322    8D55 BC         lea edx,dword ptr ss:[ebp\-44\]
0043D325    0FBFC6          movsx eax,si
0043D328    898D 34FFFFFF   mov dword ptr ss:[ebp\-CC\],ecx
0043D32E    52              push edx
0043D32F    8D8D 2CFFFFFF   lea ecx,dword ptr ss:[ebp\-D4\]
0043D335    50              push eax
0043D336    8D55 AC         lea edx,dword ptr ss:[ebp\-54\]
0043D339    51              push ecx
0043D33A    52              push edx
0043D33B    C745 C4 0100000<mov dword ptr ss:[ebp-3C\],1
0043D342    C745 BC 0200000<mov dword ptr ss:[ebp-44\],2
0043D349    C785 2CFFFFFF 0<mov dword ptr ss:[ebp-D4\],4008        ; mid 函数 mid(name,i,1)
0043D353    FF15 D0104000   call dword ptr ds:[<&MSVBVM60.#632<\] ; MSVBVM60.rtcMidCharVar
0043D359    8D45 AC         lea eax,dword ptr ss:[ebp\-54\]
0043D35C    8D4D D8         lea ecx,dword ptr ss:[ebp\-28\]
0043D35F    50              push eax
0043D360    51              push ecx
0043D361    FF15 70114000   call dword ptr ds:[<&MSVBVM60.\_\_vbaS<; MSVBVM60.\_\_vbaStrVarVal
0043D367    50              push eax
0043D368    FF15 50104000   call dword ptr ds:[<&MSVBVM60.#516<\] ; MSVBVM60.rtcAnsiValueBstr
0043D36E    0FBFD0          movsx edx,ax                           ; 函数 Asc(string)
0043D371    8B45 E4         mov eax,dword ptr ss:[ebp\-1C\]
0043D374    8D4D D8         lea ecx,dword ptr ss:[ebp\-28\]
0043D377    03D0            add edx,eax                             ; 加到 edx 上面
0043D379    0F80 18040000   jo ExcelToE.0043D797
0043D37F    8955 E4         mov dword ptr ss:[ebp\-1C\],edx
0043D382    FF15 64124000   call dword ptr ds:[<&MSVBVM60.\_\_vbaF<; MSVBVM60.\_\_vbaFreeStr
0043D388    8D45 AC         lea eax,dword ptr ss:[ebp\-54\]
0043D38B    8D4D BC         lea ecx,dword ptr ss:[ebp\-44\]
0043D38E    50              push eax
0043D38F    51              push ecx
0043D390    6A 02           push 2
0043D392    FFD3            call ebx
0043D394    B8 01000000     mov eax,1
0043D399    83C4 0C         add esp,0C
0043D39C    66:03C6         add ax,si                               ; 循环加 1
```

第一小部分，对用户处理，其实就是把 `ascii` 求和

```
0043D39F    0F80 F2030000   jo ExcelToE.0043D797
0043D3A5    8BF0            mov esi,eax
0043D3A7    ^ E9 66FFFFFF     jmp ExcelToE.0043D312
0043D3AC    8B75 0C         mov esi,dword ptr ss:[ebp+C\]
0043D3AF    8B16            mov edx,dword ptr ds:[esi\]            ; 取得机器码 69813-640-0089765-45443
0043D3B1    52              push edx
0043D3B2    FF15 68124000   call dword ptr ds:[<&MSVBVM60.#581<\] ; MSVBVM60.rtcR8ValFromBstr
0043D3B8    8B45 E4         mov eax,dword ptr ss:[ebp\-1C\]        ; val，取值(机器码)
0043D3BB    8B4D 08         mov ecx,dword ptr ss:[ebp+8\]
0043D3BE    DD9D C4FEFFFF   fstp qword ptr ss:[ebp\-13C\]          ; 68913
```

放入浮点运算器：`val`（机器码）=`68913`

```
0043D3C4    8985 24FFFFFF   mov dword ptr ss:[ebp\-DC\],eax        ; 用户名 ascii 之和 26c-&lt;620
0043D3CA    8D95 2CFFFFFF   lea edx,dword ptr ss:[ebp\-D4\]
0043D3D0    8D45 BC         lea eax,dword ptr ss:[ebp\-44\]
0043D3D3    52              push edx
0043D3D4    50              push eax
0043D3D5    C785 1CFFFFFF 0<mov dword ptr ss:[ebp-E4\],3
0043D3DF    898D 34FFFFFF   mov dword ptr ss:[ebp\-CC\],ecx
0043D3E5    C785 2CFFFFFF 0<mov dword ptr ss:[ebp-D4\],4008
0043D3EF    FFD7            call edi                                ; len(name)
0043D3F1    8D8D 1CFFFFFF   lea ecx,dword ptr ss:[ebp\-E4\]
0043D3F7    8D55 BC         lea edx,dword ptr ss:[ebp\-44\]
0043D3FA    51              push ecx                                 ; 26c
0043D3FB    8D45 AC         lea eax,dword ptr ss:[ebp\-54\]
0043D3FE    52              push edx                                 ; \*6
0043D3FF    50              push eax
0043D400    FF15 3C114000   call dword ptr ds:[<&MSVBVM60.\_\_vbaV<; MSVBVM60.\_\_vbaVarMul
0043D406    8BD0            mov edx,eax                             ; ascii 和\*6
0043D408    8D4D 9C         lea ecx,dword ptr ss:[ebp\-64\]         ; 3
```



# 汇编代码片段分析

```
0043D40B    FF15 18104000   call dword ptr ds:[<&MSVBVM60.__vbaV<; MSVBVM60.__vbaVarMove
```

**小结：**用户名 ascii 之和 * len（用户名）= 620 * 6 = 3720

```
0043D411    8B4D 08         mov ecx,dword ptr ss:[ebp+8]

0043D414    8D95 0CFFFFFF   lea edx,dword ptr ss:[ebp-F4]

0043D41A    8D45 8C         lea eax,dword ptr ss:[ebp-74]

0043D41D    52              push edx

0043D41E    50              push eax

0043D41F    898D 14FFFFFF   mov dword ptr ss:[ebp-EC],ecx

0043D425    C785 0CFFFFFF 0<mov dword ptr ss:[ebp-F4],4008

0043D42F    FFD7            call edi                              ; len(name)

0043D431    8D8D ECFEFFFF   lea ecx,dword ptr ss:[ebp-114]

0043D437    8D95 6CFFFFFF   lea edx,dword ptr ss:[ebp-94]

0043D43D    51              push ecx

0043D43E    52              push edx

0043D43F    C785 04FFFFFF 0<mov dword ptr ss:[ebp-FC],3           ; 保存 3

0043D449    C785 FCFEFFFF 0<mov dword ptr ss:[ebp-104],2

0043D453    89B5 F4FEFFFF   mov dword ptr ss:[ebp-10C],esi

0043D459    C785 ECFEFFFF 0<mov dword ptr ss:[ebp-114],4008

0043D463    FFD7            call edi                              ; len（机器码）

0043D465    8D45 9C         lea eax,dword ptr ss:[ebp-64]         ; 17--》23

0043D468    C785 E4FEFFFF 0<mov dword ptr ss:[ebp-11C],3          ; 保存 3

0043D472    50              push eax

0043D473    C785 DCFEFFFF 0<mov dword ptr ss:[ebp-124],2          ; 2

0043D47D    FF15 E4104000   call dword ptr ds:[<&MSVBVM60.#634<\] ; MSVBVM60.rtBstrFromErrVar

0043D483    8B35 34124000   mov esi,dword ptr ds:[<&MSVBVM60.__v<; MSVBVM60.__vbaStrMove

0043D489    8BD0            mov edx,eax                             ;

0043D48B    8D4D D8         lea ecx,dword ptr ss:[ebp-28]

0043D48E    FFD6            call esi

0043D490    8D4D 8C         lea ecx,dword ptr ss:[ebp-74]

0043D493    50              push eax                                 ; 底数 "3720"的第一位数字

0043D494    8D95 FCFEFFFF   lea edx,dword ptr ss:[ebp-104]

0043D49A    51              push ecx                                 ; 指数，刚才放入的 3

0043D49B    8D85 7CFFFFFF   lea eax,dword ptr ss:[ebp-84]

0043D4A1    52              push edx                                 ; 幂

0043D4A2    50              push eax

0043D4A3    FF15 90104000   call dword ptr ds:[<&MSVBVM60.__vbaV<; MSVBVM60.__vbaVarPow
```

**小结：**`len(name)³=6³=216`

```
0043D4A9    8D8D 6CFFFFFF   lea ecx,dword ptr ss:[ebp-94]           ; 求幂 216

0043D4AF    50              push eax                                 ; ecx 底数 len(sn)=23

0043D4B0    8D95 DCFEFFFF   lea edx,dword ptr ss:[ebp-124]          ; edx 3

0043D4B6    51              push ecx                                 ; 底数，len(sn)=23

0043D4B7    8D85 5CFFFFFF   lea eax,dword ptr ss:[ebp-A4]

0043D4BD    52              push edx                                 ; 指数 3

0043D4BE    50              push eax

0043D4BF    FF15 90104000   call dword ptr ds:[<&MSVBVM60.__vbaV<; MSVBVM60.__vbaVarPow

0043D4C5    8D8D 4CFFFFFF   lea ecx,dword ptr ss:[ebp-B4]           ; 23³=12167
```

**小结：**`len(sn)³=23³=12167`

```
0043D4CB    50              push eax

0043D4CC    51              push ecx                                 ; 12167

0043D4CD    FF15 F8114000   call dword ptr ds:[<&MSVBVM60.__vbaV<; MSVBVM60.__vbaVarAdd

0043D4D3    50              push eax                                 ; 12383=216+12167
```

**小结：**`12383=216+12167`

```
0043D4D4    FF15 E4104000   call dword ptr ds:[<&MSVBVM60.#634<\]   ; MSVBVM60.rtBstrFromErrVar

0043D4DA    8BD0            mov edx,eax

0043D4DC    8D4D D4         lea ecx,dword ptr ss:[ebp-2C]

0043D4DF    FFD6            call esi

0043D4E1    50              push eax

0043D4E2    FF15 64104000   call dword ptr ds:[<&MSVBVM60.__vbaS<; MSVBVM60.__vbaStrCat

0043D4E8    8BD0            mov edx,eax                              ; 372012383
```

**小结：**`"3270" & "12383"` （注：原文注释如此，但前文计算结果为 372012383，此处保留原文注释逻辑）

```
0043D4EA    8D4D D0         lea ecx,dword ptr ss:[ebp-30]

0043D4ED    FFD6            call esi

0043D4EF    50              push eax

0043D4F0    FF15 68124000   call dword ptr ds:[<&MSVBVM60.#581<\]   ; MSVBVM60.rtcR8ValFromBstr

0043D4F6    DC85 C4FEFFFF   fadd qword ptr ss:[ebp-13C]             ; 372082196+69813
```

**浮点运算，加法：**`372082196=69813+372012383`

```
0043D4FC    8D95 3CFFFFFF   lea edx,dword ptr ss:[ebp-C4]

0043D502    C785 3CFFFFFF 0<mov dword ptr ss:[ebp-C4],5

0043D50C    52              push edx

0043D50D    DC05 C01A4000   fadd qword ptr ds:[401AC0]              ; 20030207

0043D513    DD9D 44FFFFFF   fstp qword ptr ss:[ebp-BC]              ; 392112403
```

**加上固定数字：**`20030207`，得到数字：`372112403` （注：原文注释为 392112403，但根据 372082196+20030207=374085203 或其他逻辑，此处保留原文注释内容“392112403”及文字描述“372112403”可能存在笔误，遵循“不修改语义”原则，仅对变量加反引号，保留原数值描述）

*修正：根据原文描述“得到数字：372112403”，而汇编注释是“392112403”。为了保持语义不变，我将把文中的数字都加上反引号。*

**加上固定数字：**`20030207`，得到数字：`372112403`

下面对这个串进行处理：

```
0043D59A    8D4D BC         lea ecx,dword ptr ss:[ebp-44]

0043D59D    51              push ecx

0043D59E    FF15 84114000   call dword ptr ds:[<&MSVBVM60.__vbaI>; MSVBVM60.__vbaI2Var

0043D5A4    8D4D BC         lea ecx,dword ptr ss:[ebp-44]           ; 开始处理字符串

0043D5A7    8985 B4FEFFFF   mov dword ptr ss:[ebp-14C],eax          ; 位数

0043D5AD    BE 01000000     mov esi,1                               ; 循环开始

0043D5B2    FF15 20104000   call dword ptr ds:[<&MSVBVM60.__vbaF>; MSVBVM60.__vbaFreeVar

0043D5B8    66:3BB5 B4FEFFF>cmp si,word ptr ss:[ebp-14C]            ; 是否大于 9？

0043D5BF    0F8F 48010000   jg ExcelToE.0043D70D                    ; 大则跳出循环

0043D5C5    0FBFC6          movsx eax,si

0043D5C8    8D55 E4         lea edx,dword ptr ss:[ebp-1C]

0043D5CB    8D4D BC         lea ecx,dword ptr ss:[ebp-44]

0043D5CE    8995 34FFFFFF   mov dword ptr ss:[ebp-CC],edx

0043D5D4    51              push ecx                                 ; 1

0043D5D5    8985 A8FEFFFF   mov dword ptr ss:[ebp-158],eax

0043D5DB    50              push eax                                 ; 9

0043D5DC    8D95 2CFFFFFF   lea edx,dword ptr ss:[ebp-D4]

0043D5E2    8D45 AC         lea eax,dword ptr ss:[ebp-54]

0043D5E5    52              push edx

0043D5E6    50              push eax

0043D5E7    C745 C4 0100000>mov dword ptr ss:[ebp-3C],1

0043D5EE    C745 BC 0200000>mov dword ptr ss:[ebp-44],2

0043D5F5    C785 2CFFFFFF 0>mov dword ptr ss:[ebp-D4],4003

0043D5FF    FF15 D0104000   call dword ptr ds:[<&MSVBVM60.#632>\]  ; MSVBVM60.rtcMidCharVar

0043D605    8D4D AC         lea ecx,dword ptr ss:[ebp-54]           ; mid(code,i,1)

0043D608    8D55 D8         lea edx,dword ptr ss:[ebp-28]

0043D60B    51              push ecx

0043D60C    52              push edx

0043D60D    FF15 70114000   call dword ptr ds:[<&MSVBVM60.__vbaS>; MSVBVM60.__vbaStrVarVal

0043D613      .  50                  push eax                         ;  逐位取码 372112403

0043D614      .  FF15 68124000        call dword ptr ds:[<&MSVBVM60.#5<;  MSVBVM60.rtcR8ValFromBstr

0043D61A      .  DD9D C4FEFFFF        fstp qword ptr ss:[ebp-13C]       ;  装入第 i 位

0043D620      .  DB85 A8FEFFFF        fild dword ptr ss:[ebp-158]

0043D626      .  DD9D A0FEFFFF        fstp qword ptr ss:[ebp-160]

0043D62C      .  DD85 A0FEFFFF        fld qword ptr ss:[ebp-160]        ;  第几位 i

0043D632      .  DC8D C4FEFFFF        fmul qword ptr ss:[ebp-13C]       ;  相乘 位数

0043D638      .  DFE0                 fstsw ax

0043D63A      .  A8 0D                test al,0D

0043D63C      .  0F85 50010000        jnz ExcelToE.0043D792

0043D642      .  FF15 08124000        call dword ptr ds:[<&MSVBVM60.__<&;  MSVBVM60.__vbaFpI2

0043D648      .  8D4D D8              lea ecx,dword ptr ss:[ebp-28]

0043D64B      .  8BF8                 mov edi,eax

0043D64D      .  FF15 64124000        call dword ptr ds:[<&MSVBVM60.__<&;  MSVBVM60.__vbaFreeStr

0043D653      .  8D45 AC              lea eax,dword ptr ss:[ebp-54]
```



```
0043D656      .   8D4D BC              lea ecx,dword ptr ss:[ebp-44]  
0043D659      .   50                   push eax  
0043D65A      .   51                   push ecx  
0043D65B      .   6A 02                push 2  
0043D65D      .   FFD3                 call ebx  
0043D65F      .   83C4 0C              add esp,0C  
0043D662      .   66:83FF 1E           cmp di,1E              ;  跟 1e 比较  
0043D666      .   66:8BC7              mov ax,di  
0043D669      .   7D 19                jge short ExcelToE.0043D684    ;  如果大  
0043D66B      .   66:05 0100           add ax,1  
0043D66F      .   66:B9 0A00           mov cx,0A              ;  a  
0043D673      .   0F80 1E010000        jo ExcelToE.0043D797  
0043D679      .   66:99                cwd  
0043D67B      .   66:F7F9              idiv cx                ;  结果 mod/a  
0043D67E      .   66:83C2 30           add dx,30              ;  +30  
0043D682      .   EB 17                jmp short ExcelToE.0043D69B  
0043D684      <   66:05 0100           add ax,1               ;  +1  
0043D688      .   66:B9 1A00           mov cx,1A  
0043D68C      .   0F80 05010000        jo ExcelToE.0043D797  
0043D692      .   66:99                cwd  
0043D694      .   66:F7F9              idiv cx                ;  mod/1a  
0043D697      .   66:83C2 41           add dx,41              ;  +41  
0043D69B      <   8B45 DC              mov eax,dword ptr ss:[ebp-24]  
0043D69E      .   C785 2CFFFFFF 08000000   mov dword ptr ss:[ebp-D4],8  
0043D6A8      .   0F80 E9000000        jo ExcelToE.0043D797  
0043D6AE      .   0FBFCA               movsx ecx,dx           ;   得到真正的 ascii  
```

共 9 位临时码

逐位取出临时码的每一位，当作数字，乘以位数，判断跟 `1e` 的关系

大于的话，计算结果 `mod 1a +41`

否则 计算结果 `mod a +30`

得到注册码的 `ASCII`

## 【算法描述】

```vb
For i = 1 To Len(user)  
    userAscii = Asc(Mid(user, i, 1)) + userAscii  
Next  

sn = userAscii * Len(user)  

sn = CStr(sn) & CStr(Len(user) ^ 3 + Len(code) ^ 3)  

code = CLng(sn) + 69813 + 20030207  

For i = 1 To Len(code)  
    If CInt(Mid(code, i, 1)) * i < 30 Then '&H1e  
        sn = sn + Chr((CInt(Mid(code, i, 1)) * i + 1) Mod 26 + 65) '&H1a hh41  
    Else  
        sn = sn + Chr((CInt(Mid(code, i, 1)) * i + 1) Mod 10 + 48) ' &H31  
    End If  
Next
```

`4dd466d0e1e0cc123afbe504da2462ca`
