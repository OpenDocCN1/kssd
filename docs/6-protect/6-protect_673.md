
# VMCrackME----A Preliminary Virtual Machine From Top To Bottom

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-480.htm`](https://www.pediy.com/kssd/pediy09/pediy09-480.htm)

> VMCrackME----A Preliminary Virtual Machine From Top To Bottom

> 【目标】       本文的主角是来自 KuNgBiM 大侠发的一个带虚拟机的 CrackME
>               链接：`http://bbs.pediy.com/showthread.php?threadid=40679`
>
> 【作者】       cyclotron
>
> 【目的】       研究虚拟机的运作机理
>
> 【联系方式】   `cyclotron [at] citiz.net`
>
> 【个人主页】   `cyclotron.yculblog.com`

> 这个 CrackME 比较有意思的地方是附带了一个完整的虚拟机解释引擎，代码量比起 Themida，VMProtect 和 Code Virtualizer 来自然是捉襟见肘。不过麻雀虽小，五脏俱全，我们可以通过这个引擎体验一下虚拟机世界的乐趣，同时再次瞻仰一下以 softworm 为代表的牛群。

> 首先简单介绍一下这个虚拟机的体系结构，下面这些本地变量的命名是根据 cyclotron 对虚拟机执行引擎的理解类比 x86 体系定义的，把它们定义为其他的名称完全不会影响到你对虚拟机的分析：

```
00401060 var_lpVMStack   = dword ptr -130h    ; 虚拟机堆栈区基址指针
               ; 注意这个堆栈是向下增长的
00401060 var_lpVArg      = dword ptr -30h     ; 指向输入参数的指针（处理为字节串）
00401060 var_VargIndex   = dword ptr -2Ch     ; 输入参数按字节顺序的访问序号
00401060 var_lpVRetVal   = dword ptr -28h     ; 指向返回值的指针（处理为字节串）
00401060 var_VRetValIndex= dword ptr -24h     ; 返回值按字节顺序的访问序号
00401060 var_RegEsi      = dword ptr -20h
00401060 var_RegEdi      = dword ptr -1Ch
00401060 var_RegEcx      = dword ptr -18h
00401060 var_RegEip      = dword ptr -14h
00401060 var_RegDl       = byte ptr -10h
00401060 var_RegEsp      = dword ptr -0Fh
00401060 var_RegEax      = dword ptr -0Bh
00401060 var_SFlag       = byte ptr -3
00401060 var_ZFlag       = byte ptr -2
00401060 arg_lpVArg      = dword ptr  8       ; 指向输入参数的指针（处理为字节串）
00401060 arg_lpVRetVal   = dword ptr  0Ch     ; 指向返回值的指针（处理为字节串）
00401060 arg_lpVMCode    = dword ptr  10h     ; 虚拟机代码区指针
00401060 arg_lpVMData    = dword ptr  14h     ; 虚拟机数据区指针
```

> 这个 CrackME 中有 6 个地方调用了这个虚拟机执行引擎，这也就意味着我们必须分析 6 组虚拟机伪代码，其中与注册相关的调用代码列举如下：

```
004017AF           push    offset lpVMData  ; 虚拟机数据区指针
004017B4           push    offset lpVI_4020A9 ; 虚拟机代码区指针
004017B9           push    0
004017BB           lea     eax, [ebp+var_11]  ; lpVArg --> USERNAME
004017BE           push    eax
004017BF           call    VI_Interpreter ; 调用解释引擎
004017C4           mov     eax, offset lpVMData
004017C9           add     eax, 4
004017CC           mov     dword ptr [eax], 0
004017D2           mov     dword ptr [eax+4], 1
004017D9           push    offset lpVMData
004017DE           push    offset lpVMCode_4020CF ; 虚拟机代码区指针
004017E3           push    0
004017E5           lea     eax, [ebp+var_22]  ; lpVArg --> Serial
004017E8           push    eax
004017E9           call    VI_Interpreter ; 调用解释引擎
004017EE           push    offset lpVMData
004017F3           push    offset lpVI_4020F7 ; 虚拟机代码区指针
004017F8           push    offset aWrong  ; "Wrong!"
004017FD           push    0
004017FF           call    VI_Interpreter ; 调用解释引擎
00401804           mov     eax, uType
00401809           push    eax          ; uType
0040180A           push    offset aWrong  ; lpCaption
0040180F           push    offset aWrong  ; lpText
00401814           push    hWnd         ; hWnd
0040181A           call    MessageBoxA
0040181F           jmp     short loc_401831
```

> 接着我们来分析一下这个这个虚拟机解释引擎的全貌：

> 虚拟机解释器要访问的的跳转表：

```
00402000 lpJumpAddrTable dd offset VNop
00402004             dd offset VPushImm32
00402008             dd offset VLoadMem32
0040200C             dd offset VSaveMem32
00402010             dd offset VAdd
00402014             dd offset VSub
00402018             dd offset VRet10
0040201C             dd offset VPushRegEax   ; All the Arithmetic Results are Returned in RegEax
                                               ;  and should be thereafter Pushed into the Stack Manually
00402020             dd offset VMul
00402024             dd offset VDiv
00402028             dd offset VAnd
0040202C             dd offset Vor
00402030             dd offset VXor
00402034             dd offset VPop2RegEax
00402038             dd offset VPushByteVArg
0040203C             dd offset VSaveRegEax2ByteVRetVal
00402040             dd offset VJmp
00402044             dd offset VJz
00402048             dd offset VJnz
0040204C             dd offset VJs
```

> 虚拟机配备了一个基于堆栈，单操作数的伪指令系统，每条指令都会将返回值保存在 `var_RegEax`，以便后面的指令访问，其指令集已经清晰地罗列在上面的跳转表中。

> 注意：伪指令的解释引擎在代码中并不是按照上述地址表的顺序排列的，cyclotron 下面所有关于伪指令解释的序号均为该伪指令在跳转表中的序号，以方便大家整理反编译器。

```
00401060 ; ************** S U B R O U T I N E ****************************
00401060
00401060 ; A Single-Operand Based Virtual Machine (Stack Based)
00401060 ; Attributes: bp-based frame
00401060
00401060 VI_Interpreter  proc near                  ; CODE XREF: DialogFunc+109p
00401060                           ; DialogFunc+20Bp
00401060                           ; DialogFunc+22Dp
00401060                           ; DialogFunc+24Dp
00401060                           ; DialogFunc+277p
00401060                           ; DialogFunc+28Dp
00401060
00401060 var_lpVMStack   = dword ptr -130h
00401060 var_lpVArg      = dword ptr -30h
00401060 var_VargIndex   = dword ptr -2Ch
00401060 var_lpVRetVal   = dword ptr -28h
00401060 var_VRetValIndex= dword ptr -24h
00401060 var_RegEsi      = dword ptr -20h
00401060 var_RegEdi      = dword ptr -1Ch
00401060 var_RegEcx      = dword ptr -18h
00401060 var_RegEip      = dword ptr -14h
00401060 var_RegDl       = byte ptr -10h
00401060 var_RegEsp      = dword ptr -0Fh
00401060 var_RegEax      = dword ptr -0Bh
00401060 var_SFlag       = byte ptr -3
00401060 var_ZFlag       = byte ptr -2
00401060 arg_lpVArg      = dword ptr  8
00401060 arg_lpVRetVal   = dword ptr  0Ch
00401060 arg_lpVMCode    = dword ptr  10h
00401060 arg_lpVMData    = dword ptr  14h
00401060
00401060           push    ebp
00401061           mov     ebp, esp
00401063           add     esp, 0FFFFFED0h
00401069           mov     [ebp+var_RegEdi], 0
00401070           mov     [ebp+var_RegEcx], 0
00401077           mov     [ebp+var_RegEsp], 0
0040107E           mov     [ebp+var_SFlag], 0
00401082           mov     [ebp+var_ZFlag], 0
00401086           mov     [ebp+var_RegEax], 0
0040108D           lea     eax, [ebp+var_lpVMStack]
00401093           mov     [ebp+var_RegEsp], eax
00401096           mov     eax, [ebp+arg_lpVMData]
00401099           mov     [ebp+var_RegEsi], eax
0040109C           mov     eax, [ebp+arg_lpVArg]
0040109F           mov     [ebp+var_lpVArg], eax
004010A2           mov     eax, [ebp+arg_lpVRetVal]
004010A5           mov     [ebp+var_lpVRetVal], eax
004010A8           mov     [ebp+var_VRetValIndex], 0
004010AF           mov     [ebp+var_VargIndex], 0  ; 上面这段代码对虚拟机的寄存器系统进行初始化
004010B6           mov     eax, [ebp+arg_lpVMCode]; 取得伪代码区指针
004010B9           mov     [ebp+var_RegEip], eax
004010BC
004010BC InterpretingEntry:       ; 每一条伪指令解释完毕都返回到这里
```



```
004010BC                     inc     [ebp+var_RegEip]
004010BF                     mov     eax, [ebp+var_RegEip]
004010C2                     mov     al, [eax]         ; 从伪代码区取出一个字节
004010C4                     mov     [ebp+var_RegDl], al
004010C7                     mov     eax, offset lpJumpAddrTable
004010CC                     movzx   ebx, [ebp+var_RegDl]
004010D0                     shl     ebx, 2      ; x4
004010D3                     add     eax, ebx            ; 检索跳转表
004010D5                     jmp     dword ptr [eax]  ; 进入解释引擎，类似于 VB P-code
004010D5 VI_Interpreter  endp
```

0. `VNop`，空操作伪指令

```
004010D7 ; ************************** S U B R O U T I N E ***************************
004010D7
004010D7 ; Attributes: thunk
004010D7
004010D7 VNop            proc near
004010D7                 jmp     short InterpretingEntry
004010D7 VNop            endp
```

1. `VPushImm32`，将该指令的操作数（长度为一个字节）无符号扩展为一个 `dword`，压入虚拟机堆栈区，并保存至 `var_RegEax`

```
004010D9 ; ************************** S U B R O U T I N E ***************************
004010D9
004010D9 ; Attributes: bp-based frame
004010D9
004010D9 VPushImm32      proc near
004010D9
004010D9 var_RegEdi      = dword ptr -1Ch
004010D9 var_RegEcx      = dword ptr -18h
004010D9 var_RegEip      = dword ptr -14h
004010D9 var_RegDl       = byte ptr -10h
004010D9 var_RegEsp      = dword ptr -0Fh
004010D9 var_RegEax      = dword ptr -0Bh
004010D9
004010D9                 add     [ebp+var_RegEsp], 4
004010DD                 mov     eax, [ebp+var_RegEsp]
004010E0                 mov     [ebp+var_RegEdi], eax
004010E3                 inc     [ebp+var_RegEip]
004010E6                 mov     eax, [ebp+var_RegEip]
004010E9                 mov     al, [eax]    ; Fetch the Immediate Byte Operand
004010EB                 mov     [ebp+var_RegDl], al
004010EE                 movzx   eax, [ebp+var_RegDl]
004010F2                 mov     [ebp+var_RegEax], eax
004010F5                 mov     [ebp+var_RegEcx], eax
004010F8                 mov     ebx, [ebp+var_RegEdi]
004010FB                 mov     [ebx], eax
004010FD                 jmp     short InterpretingEntry
004010FD VPushImm32      endp
```

2. `VLoadMem32`，操作数为一个字节的内存地址，从虚拟机数据区取出一个 `dword`，压入虚拟机堆栈区，并保存至 `var_RegEax`

```
004010FF ; ************************** S U B R O U T I N E ***************************
004010FF
004010FF ; Attributes: bp-based frame
004010FF
004010FF VLoadMem32      proc near
004010FF
004010FF var_RegEsi      = dword ptr -20h
004010FF var_RegEdi      = dword ptr -1Ch
004010FF var_RegEcx      = dword ptr -18h
004010FF var_RegEip      = dword ptr -14h
004010FF var_RegDl       = byte ptr -10h
004010FF var_RegEsp      = dword ptr -0Fh
004010FF var_RegEax      = dword ptr -0Bh
004010FF
004010FF                 inc     [ebp+var_RegEip]
00401102                 mov     eax, [ebp+var_RegEip]
00401105                 movzx   eax, byte ptr [eax]  ; Fetch the Immediate Byte Operand
00401108                 mov     [ebp+var_RegDl], al
0040110B                 mov     eax, [ebp+var_RegEsi]  ; Pointer to VMData
0040110E                 movzx   ebx, [ebp+var_RegDl]
00401112                 shl     ebx, 2
00401115                 add     eax, ebx
00401117                 mov     [ebp+var_RegEdi], eax
0040111A                 mov     ebx, [eax]    ; Fetch the Dword
0040111C                 mov     [ebp+var_RegEcx], ebx
0040111F                 add     [ebp+var_RegEsp], 4
00401123                 mov     eax, [ebp+var_RegEsp]
00401126                 mov     [ebp+var_RegEdi], eax
00401129                 mov     ebx, [ebp+var_RegEcx]
0040112C                 mov     [eax], ebx    ; Save the Dword to Stack
0040112E                 mov     [ebp+var_RegEax], ebx
00401131                 jmp     short InterpretingEntry
00401131 VLoadMem32      endp
```

3. `VSaveMem32`，从虚拟机堆栈区弹出一个 `dword`，保存 `var_RegEax` 至虚拟机数据区

```
00401133 ; ************************** S U B R O U T I N E ***************************
00401133
00401133 ; Attributes: bp-based frame
00401133
00401133 VSaveMem32      proc near
00401133
00401133 var_RegEsi      = dword ptr -20h
00401133 var_RegEdi      = dword ptr -1Ch
00401133 var_RegEcx      = dword ptr -18h
00401133 var_RegEip      = dword ptr -14h
00401133 var_RegDl       = byte ptr -10h
00401133 var_RegEsp      = dword ptr -0Fh
00401133 var_RegEax      = dword ptr -0Bh
00401133
00401133                 inc     [ebp+var_RegEip]
00401136                 mov     eax, [ebp+var_RegEip]
00401139                 mov     al, [eax]    ; Fetch the Immediate Byte Operand
0040113B                 mov     [ebp+var_RegDl], al
0040113E                 mov     eax, [ebp+var_RegEsi]  ; Pointer to VMData
00401141                 movzx   ebx, [ebp+var_RegDl]
00401145                 shl     ebx, 2
00401148                 add     eax, ebx
0040114A                 mov     ebx, [ebp+var_RegEax]  ; the Return Value of the Last Virtual Instruction
0040114D                 mov     [ebp+var_RegEdi], eax
00401150                 mov     [eax], ebx
00401152                 sub     [ebp+var_RegEsp], 4
00401156                 mov     eax, [ebp+var_RegEsp]
00401159                 mov     [ebp+var_RegEdi], eax
0040115C                 mov     eax, [eax]
0040115E                 mov     [ebp+var_RegEax], eax
00401161                 mov     [ebp+var_RegEcx], eax
00401164                 jmp     InterpretingEntry
00401164 VSaveMem32      endp
```

4. `VAdd`，从虚拟机堆栈区弹出一个 `dword`，与上一条指令的返回值相加，结果保存在 `var_RegEax` 中，并设置标志位

```
00401169 ; ************************** S U B R O U T I N E ***************************
00401169
00401169 ; Attributes: bp-based frame
00401169
00401169 VAdd            proc near
00401169
00401169 var_RegEdi      = dword ptr -1Ch
00401169 var_RegEcx      = dword ptr -18h
00401169 var_RegEsp      = dword ptr -0Fh
00401169 var_RegEax      = dword ptr -0Bh
00401169 var_SFlag       = byte ptr -3
00401169 var_ZFlag       = byte ptr -2
00401169
00401169                 mov     eax, [ebp+var_RegEax]
0040116C                 sub     [ebp+var_RegEsp], 4
00401170                 mov     ebx, [ebp+var_RegEsp]
00401173                 mov     [ebp+var_RegEdi], ebx
00401176                 mov     edx, [ebx]
00401178                 mov     [ebp+var_RegEcx], edx
0040117B                 add     [ebp+var_RegEcx], eax
0040117E                 setz    [ebp+var_ZFlag]  ; 根据计算结果置 Z 标志
00401182                 sets    [ebp+var_SFlag]  ; 根据计算结果置 S 标志
00401186                 push    [ebp+var_RegEcx]
00401189                 pop     [ebp+var_RegEax]
0040118C                 mov     eax, [ebp+var_RegEcx]
0040118F                 mov     [ebx], eax
00401191                 jmp     InterpretingEntry
00401191 VAdd            endp
```

5. `VSub`，从虚拟机堆栈区弹出一个 `dword`，从上一条指令的返回值中减去之，结果保存在 `var_RegEax` 中，并设置标志位



> 00401196 ; `****` `S` `U` `B` `R` `O` `U` `T` `I` `N` `E` `****`  
> 00401196  
> 00401196 ; Attributes: bp-based frame  
> 00401196  
> 00401196 `VSub` `proc` `near`  
> 00401196  
> 00401196 `var_RegEdi` `=` `dword` `ptr` `-1Ch`  
> 00401196 `var_RegEcx` `=` `dword` `ptr` `-18h`  
> 00401196 `var_RegEsp` `=` `dword` `ptr` `-0Fh`  
> 00401196 `var_RegEax` `=` `dword` `ptr` `-0Bh`  
> 00401196 `var_SFlag` `=` `byte` `ptr` `-3`  
> 00401196 `var_ZFlag` `=` `byte` `ptr` `-2`  
> 00401196  
> 00401196 `mov` `eax,` `[ebp+var_RegEax]`  
> 00401199 `sub` `[ebp+var_RegEsp],` `4`  
> 0040119D `mov` `ebx,` `[ebp+var_RegEsp]`  
> 004011A0 `mov` `[ebp+var_RegEdi],` `ebx`  
> 004011A3 `mov` `edx,` `[ebx]`  
> 004011A5 `mov` `[ebp+var_RegEcx],` `edx`  
> 004011A8 `xchg` `eax,` `[ebp+var_RegEcx]`  
> 004011AB `sub` `[ebp+var_RegEcx],` `eax`  
> 004011AE `setz` `[ebp+var_ZFlag]` ; 根据计算结果置 Z 标志  
> 004011B2 `sets` `[ebp+var_SFlag]` ; 根据计算结果置 S 标志  
> 004011B6 `push` `[ebp+var_RegEcx]`  
> 004011B9 `pop` `[ebp+var_RegEax]`  
> 004011BC `mov` `eax,` `[ebp+var_RegEcx]`  
> 004011BF `mov` `[ebx],` `eax`  
> 004011C1 `jmp` `InterpretingEntry`  
> 004011C1 `VSub` `endp`  

> 8. `VMul`，从虚拟机堆栈区弹出一个 `dword`，与上一条指令的返回值相乘，结果保存在 `var_RegEax` 中，并设置标志位

> 004011C6 ; `****` `S` `U` `B` `R` `O` `U` `T` `I` `N` `E` `****`  
> 004011C6  
> 004011C6 ; Attributes: bp-based frame  
> 004011C6  
> 004011C6 `VMul` `proc` `near`  
> 004011C6  
> 004011C6 `var_RegEdi` `=` `dword` `ptr` `-1Ch`  
> 004011C6 `var_RegEcx` `=` `dword` `ptr` `-18h`  
> 004011C6 `var_RegEsp` `=` `dword` `ptr` `-0Fh`  
> 004011C6 `var_RegEax` `=` `dword` `ptr` `-0Bh`  
> 004011C6 `var_SFlag` `=` `byte` `ptr` `-3`  
> 004011C6 `var_ZFlag` `=` `byte` `ptr` `-2`  
> 004011C6  
> 004011C6 `mov` `eax,` `[ebp+var_RegEax]`  
> 004011C9 `sub` `[ebp+var_RegEsp],` `4`  
> 004011CD `mov` `ebx,` `[ebp+var_RegEsp]`  
> 004011D0 `mov` `[ebp+var_RegEdi],` `ebx`  
> 004011D3 `mov` `ebx,` `[ebx]`  
> 004011D5 `mov` `[ebp+var_RegEcx],` `ebx`  
> 004011D8 `xor` `edx,` `edx`  
> 004011DA `mov` `word` `ptr` `ds:loc_4011E3,` `0E3F7h` ; SMC  
> 004011E3  
> 004011E3 `loc_4011E3:`  
> 004011E3 `neg` `eax` ; Change to "mul ebx"  
> 004011E5 `mov` `[ebp+var_RegEcx],` `eax`  
> 004011E8 `setz` `[ebp+var_ZFlag]` ; 根据计算结果置 Z 标志  
> 004011EC `sets` `[ebp+var_SFlag]` ; 根据计算结果置 S 标志  
> 004011F0 `push` `[ebp+var_RegEcx]`  
> 004011F3 `pop` `[ebp+var_RegEax]`  
> 004011F6 `mov` `eax,` `[ebp+var_RegEcx]`  
> 004011F9 `mov` `ebx,` `[ebp+var_RegEsp]`  
> 004011FC `mov` `[ebx],` `eax`  
> 004011FE `jmp` `InterpretingEntry`  
> 004011FE `VMul` `endp`  

> 9. `VDiv`，从虚拟机堆栈区弹出一个 `dword`，用上一条指令的返回值中除以之，结果保存在 `var_RegEax` 中，并设置标志位

> 00401203 ; `****` `S` `U` `B` `R` `O` `U` `T` `I` `N` `E` `****`  
> 00401203  
> 00401203 ; Attributes: bp-based frame  
> 00401203  
> 00401203 `VDiv` `proc` `near`  
> 00401203  
> 00401203 `var_RegEdi` `=` `dword` `ptr` `-1Ch`  
> 00401203 `var_RegEcx` `=` `dword` `ptr` `-18h`  
> 00401203 `var_RegEsp` `=` `dword` `ptr` `-0Fh`  
> 00401203 `var_RegEax` `=` `dword` `ptr` `-0Bh`  
> 00401203 `var_SFlag` `=` `byte` `ptr` `-3`  
> 00401203 `var_ZFlag` `=` `byte` `ptr` `-2`  
> 00401203  
> 00401203 `mov` `eax,` `[ebp+var_RegEax]`  
> 00401206 `sub` `[ebp+var_RegEsp],` `4`  
> 0040120A `mov` `ebx,` `[ebp+var_RegEsp]`  
> 0040120D `mov` `[ebp+var_RegEdi],` `ebx`  
> 00401210 `mov` `ebx,` `[ebx]`  
> 00401212 `mov` `[ebp+var_RegEcx],` `ebx`  
> 00401215 `xor` `edx,` `edx`  
> 00401217 `mov` `word` `ptr` `ds:loc_401220,` `0F3F7h` ; SMC  
> 00401220  
> 00401220 `loc_401220:`  
> 00401220 `xor` `eax,` `eax` ; change to "div ebx"  
> 00401222 `mov` `[ebp+var_RegEcx],` `eax`  
> 00401225 `setz` `[ebp+var_ZFlag]` ; 根据计算结果置 Z 标志  
> 00401229 `sets` `[ebp+var_SFlag]` ; 根据计算结果置 S 标志  
> 0040122D `push` `[ebp+var_RegEcx]`  
> 00401230 `pop` `[ebp+var_RegEax]`  
> 00401233 `mov` `eax,` `[ebp+var_RegEcx]`  
> 00401236 `mov` `ebx,` `[ebp+var_RegEsp]`  
> 00401239 `mov` `[ebx],` `eax`  
> 0040123B `jmp` `InterpretingEntry`  
> 0040123B `VDiv` `endp`  

> A. `VAnd`，从虚拟机堆栈区弹出一个 `dword`，与上一条指令的返回值中相“与”，结果保存在 `var_RegEax` 中

> 00401240 ; `****` `S` `U` `B` `R` `O` `U` `T` `I` `N` `E` `****`  
> 00401240  
> 00401240 ; Attributes: bp-based frame  
> 00401240  
> 00401240 `VAnd` `proc` `near`  
> 00401240  
> 00401240 `var_RegEdi` `=` `dword` `ptr` `-1Ch`  
> 00401240 `var_RegEcx` `=` `dword` `ptr` `-18h`  
> 00401240 `var_RegEsp` `=` `dword` `ptr` `-0Fh`  
> 00401240 `var_RegEax` `=` `dword` `ptr` `-0Bh`  
> 00401240  
> 00401240 `mov` `eax,` `[ebp+var_RegEax]`  
> 00401243 `sub` `[ebp+var_RegEsp],` `4`  
> 00401247 `mov` `ebx,` `[ebp+var_RegEsp]`  
> 0040124A `mov` `[ebp+var_RegEdi],` `ebx`  
> 0040124D `mov` `edx,` `[ebx]`  
> 0040124F `mov` `[ebp+var_RegEcx],` `edx`  
> 00401252 `xchg` `eax,` `[ebp+var_RegEcx]`  
> 00401255 `and` `[ebp+var_RegEcx],` `eax`  
> 00401258 `push` `[ebp+var_RegEcx]`  
> 0040125B `pop` `[ebp+var_RegEax]`  
> 0040125E `mov` `eax,` `[ebp+var_RegEcx]`  
> 00401261 `mov` `[ebx],` `eax`  
> 00401263 `jmp` `InterpretingEntry`  
> 00401263 `VAnd` `endp`  

> B. `VOr`，从虚拟机堆栈区弹出一个 `dword`，与上一条指令的返回值中相“或”，结果保存在 `var_RegEax` 中

> 00401268 ; `****` `S` `U` `B` `R` `O` `U` `T` `I` `N` `E` `****`  
> 00401268  
> 00401268 ; Attributes: bp-based frame  
> 00401268  
> 00401268 `VOr` `proc` `near`  
> 00401268  
> 00401268 `var_RegEdi` `=` `dword` `ptr` `-1Ch`  
> 00401268 `var_RegEcx` `=` `dword` `ptr` `-18h`  
> 00401268 `var_RegEsp` `=` `dword` `ptr` `-0Fh`  
> 00401268 `var_RegEax` `=` `dword` `ptr` `-0Bh`  
> 00401268  
> 00401268 `mov` `eax,` `[ebp+var_RegEax]`  
> 0040126B `sub` `[ebp+var_RegEsp],` `4`  
> 0040126F `mov` `ebx,` `[ebp+var_RegEsp]`  
> 00401272 `mov` `[ebp+var_RegEdi],` `ebx`  
> 00401275 `mov` `edx,` `[ebx]`  
> 00401277 `mov` `[ebp+var_RegEcx],` `edx`  
> 0040127A `xchg` `eax,` `[ebp+var_RegEcx]`  
> 0040127D `or` `[ebp+var_RegEcx],` `eax`  
> 00401280 `push` `[ebp+var_RegEcx]`  
> 00401283 `pop` `[ebp+var_RegEax]`  
> 00401286 `mov` `eax,` `[ebp+var_RegEcx]`  
> 00401289 `mov` `[ebx],` `eax`  
> 0040128B `jmp` `InterpretingEntry`  
> 0040128B `VOr` `endp`  



## C. `VXor`

从虚拟机堆栈区弹出一个 `dword`，与上一条指令的返回值“异或”，结果保存在 `var_RegEax` 中

```
00401290 ; ********** S U B R O U T I N E **********
00401290 
00401290 ; Attributes: bp-based frame
00401290 
00401290 VXor            proc near
00401290 
00401290 var_RegEdi      = dword ptr -1Ch
00401290 var_RegEcx      = dword ptr -18h
00401290 var_RegEsp      = dword ptr -0Fh
00401290 var_RegEax      = dword ptr -0Bh
00401290 
00401290                mov     eax, [ebp+var_RegEax]
00401293                sub     [ebp+var_RegEsp], 4
00401297                mov     ebx, [ebp+var_RegEsp]
0040129A                mov     [ebp+var_RegEdi], ebx
0040129D                mov     edx, [ebx]
0040129F                mov     [ebp+var_RegEcx], edx
004012A2                xchg    eax, [ebp+var_RegEcx]
004012A5                xor     [ebp+var_RegEcx], eax
004012A8                push    [ebp+var_RegEcx]
004012AB                pop     [ebp+var_RegEax]
004012AE                mov     eax, [ebp+var_RegEcx]
004012B1                mov     [ebx], eax
004012B3                jmp     InterpretingEntry
004012B3 VXor            endp
```

## 7. `VPushRegEax`

将 `var_RegEax` 的值压入虚拟机堆栈区

```
004012B8 ; ********** S U B R O U T I N E **********
004012B8 
004012B8 ; All the Arithmetic Results are Returned in RegEax
004012B8 ;  and should be thereafter Pushed into the Stack Manually
004012B8 ; Attributes: bp-based frame
004012B8 
004012B8 VPushRegEax     proc near
004012B8 
004012B8 var_RegEdi      = dword ptr -1Ch
004012B8 var_RegEcx      = dword ptr -18h
004012B8 var_RegEsp      = dword ptr -0Fh
004012B8 var_RegEax      = dword ptr -0Bh
004012B8 
004012B8                add     [ebp+var_RegEsp], 4
004012BC                mov     ebx, [ebp+var_RegEax]
004012BF                mov     [ebp+var_RegEcx], ebx
004012C2                mov     eax, [ebp+var_RegEsp]
004012C5                mov     [ebp+var_RegEdi], eax
004012C8                mov     [eax], ebx
004012CA                jmp     InterpretingEntry
004012CA VPushRegEax     endp
```

## D. `VPop2RegEax`

从虚拟机堆栈区弹出一个 `dword`，保存至 `var_RegEax`

```
004012CF ; ********** S U B R O U T I N E **********
004012CF 
004012CF ; Attributes: bp-based frame
004012CF 
004012CF VPop2RegEax     proc near
004012CF 
004012CF var_RegEdi      = dword ptr -1Ch
004012CF var_RegEcx      = dword ptr -18h
004012CF var_RegEsp      = dword ptr -0Fh
004012CF var_RegEax      = dword ptr -0Bh
004012CF 
004012CF                sub     [ebp+var_RegEsp], 4
004012D3                mov     eax, [ebp+var_RegEsp]
004012D6                mov     [ebp+var_RegEdi], eax
004012D9                mov     ebx, [eax]
004012DB                mov     [ebp+var_RegEcx], ebx
004012DE                mov     [ebp+var_RegEax], ebx
004012E1                jmp     InterpretingEntry
004012E1 VPop2RegEax     endp
```

## E. `VPushByteVArg`

将输入参数的一个字节无符号扩展以后压入虚拟机堆栈区，并保存至 `var_RegEax`

```
004012E6 ; ********** S U B R O U T I N E **********
004012E6 
004012E6 ; Attributes: bp-based frame
004012E6 
004012E6 VPushByteVArg    proc near
004012E6 
004012E6 var_lpVArg      = dword ptr -30h
004012E6 var_VArgIndex   = dword ptr -2Ch
004012E6 var_RegEdi      = dword ptr -1Ch
004012E6 var_RegEcx      = dword ptr -18h
004012E6 var_RegEsp      = dword ptr -0Fh
004012E6 var_RegEax      = dword ptr -0Bh
004012E6 
004012E6                mov     eax, [ebp+var_lpVArg]
004012E9                add     eax, [ebp+var_VArgIndex]
004012EC                inc     [ebp+var_VArgIndex]
004012EF                movzx   eax, byte ptr [eax]
004012F2                mov     [ebp+var_RegEax], eax
004012F5                add     [ebp+var_RegEsp], 4
004012F9                mov     ebx, [ebp+var_RegEsp]
004012FC                mov     [ebp+var_RegEdi], ebx
004012FF                mov     [ebp+var_RegEcx], eax
00401302                mov     [ebx], eax
00401304                jmp     InterpretingEntry
00401304 PushByteVArg    endp
```

## F. `VSaveRegEax2ByteVRetVal`

从堆栈中弹出一个 `dword`，保存 `var_RegEax` 的最低字节至返回字节串

```
00401309 ; ********** S U B R O U T I N E **********
00401309 
00401309 ; Attributes: bp-based frame
00401309 
00401309 VSaveRegEax2ByteVRetVal proc near
00401309 
00401309 var_lpVRetVal=  dword ptr -28h
00401309 var_VRetValIndex=  dword ptr -24h
00401309 var_RegEdi      = dword ptr -1Ch
00401309 var_RegEcx      = dword ptr -18h
00401309 var_RegEsp      = dword ptr -0Fh
00401309 var_RegEax      = dword ptr -0Bh
00401309 
00401309                mov     eax, [ebp+var_RegEax]
0040130C                sub     [ebp+var_RegEsp], 4
00401310                mov     ebx, [ebp+var_RegEsp]
00401313                mov     [ebp+var_RegEdi], ebx
00401316                mov     [ebp+var_RegEcx], eax
00401319                mov     ebx, [ebx]
0040131B                mov     [ebp+var_RegEax], ebx
0040131E                mov     ebx, [ebp+var_lpVRetVal]
00401321                add     ebx, [ebp+var_VRetValIndex]
00401324                inc     [ebp+var_VRetValIndex]
00401327                mov     [ebx], al
00401329                jmp     InterpretingEntry
00401329 VSaveRegEax2ByteVRetVal endp
```

## 10. `VJmp`

无条件跳转指令，偏移量为操作码后面紧跟的一个 `word`（相对于伪代码区基址）

```
0040132E ; ********** S U B R O U T I N E **********
0040132E 
0040132E ; Attributes: bp-based frame
0040132E 
0040132E VJmp            proc near
0040132E 
0040132E var_RegEip      = dword ptr -14h
0040132E var_RegDl       = byte ptr -10h
0040132E var_7           = dword ptr -7
0040132E arg_lpVMCode     = dword ptr  10h
0040132E 
0040132E                inc     [ebp+var_RegEip]
00401331                mov     eax, [ebp+var_RegEip]
00401334                movzx   eax, byte ptr [eax]
00401337                mov     [ebp+var_RegDl], al
0040133A                shl     eax, 8
0040133D                mov     [ebp+var_7], eax
00401340                inc     [ebp+var_RegEip]
00401343                mov     eax, [ebp+var_RegEip]
00401346                movzx   eax, byte ptr [eax]
00401349                mov     [ebp+var_RegDl], al
0040134C                add     [ebp+var_7], eax
0040134F                mov     eax, [ebp+var_7]
00401352                add     eax, [ebp+arg_lpVMCode]
00401355                mov     [ebp+var_RegEip], eax
00401358                jmp     InterpretingEntry
00401358 VJmp            endp
```

## 11. `VJz`

条件跳转指令，当 `var_ZFlag=1` 时触发，偏移量为操作码后面紧跟的一个 `word`（相对于伪代码区基址）



0040135D ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

0040135D

0040135D ; Attributes: bp-based frame

0040135D

0040135D VJz                 proc near

0040135D

0040135D var\_RegEip      = dword ptr -14h

0040135D var\_RegDl       = byte ptr -10h

0040135D var\_7           = dword ptr -7

0040135D var\_ZFlag       = byte ptr -2

0040135D arg\_lpVMCode     = dword ptr  10h

0040135D

0040135D                 inc     [ebp+var\_RegEip]

00401360                 mov     eax, [ebp+var\_RegEip]

00401363                 movzx   eax, byte ptr [eax]

00401366                 mov     [ebp+var\_RegDl], al

00401369                 shl     eax, 8

0040136C                 mov     [ebp+var\_7], eax

0040136F                 inc     [ebp+var\_RegEip]

00401372                 mov     eax, [ebp+var\_RegEip]

00401375                 movzx   eax, byte ptr [eax]

00401378                 mov     [ebp+var\_RegDl], al

0040137B                 add     [ebp+var\_7], eax

0040137E                 mov     eax, [ebp+var\_7]

00401381                 cmp     [ebp+var\_ZFlag], 1  ; 标志位测试

00401385                 jnz     short loc\_40138D

00401387                 add     eax, [ebp+arg\_lpVMCode]

0040138A                 mov     [ebp+var\_RegEip], eax

0040138D

0040138D loc\_40138D:

0040138D                 jmp     InterpretingEntry

0040138D VJz             endp

12. VJnz，条件跳转指令，当 `var_ZFlag=0` 时触发，偏移量为操作码后面紧跟的一个 word（相对于伪代码区基址）

00401392 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

00401392

00401392 ; Attributes: bp-based frame

00401392

00401392 VJnz            proc near

00401392

00401392 var\_RegEip      = dword ptr -14h

00401392 var\_RegDl       = byte ptr -10h

00401392 var\_7           = dword ptr -7

00401392 var\_ZFlag       = byte ptr -2

00401392 arg\_lpVMCode     = dword ptr  10h

00401392

00401392                 inc     [ebp+var\_RegEip]

00401395                 mov     eax, [ebp+var\_RegEip]

00401398                 movzx   eax, byte ptr [eax]

0040139B                 mov     [ebp+var\_RegDl], al

0040139E                 shl     eax, 8

004013A1                 mov     [ebp+var\_7], eax

004013A4                 inc     [ebp+var\_RegEip]

004013A7                 mov     eax, [ebp+var\_RegEip]

004013AA                 movzx   eax, byte ptr [eax]

004013AD                 mov     [ebp+var\_RegDl], al

004013B0                 add     [ebp+var\_7], eax

004013B3                 mov     eax, [ebp+var\_7]

004013B6                 cmp     [ebp+var\_ZFlag], 0  ; 标志位测试

004013BA                 jnz     short loc\_4013C2

004013BC                 add     eax, [ebp+arg\_lpVMCode]

004013BF                 mov     [ebp+var\_RegEip], eax

004013C2

004013C2 loc\_4013C2:

004013C2                 jmp     InterpretingEntry

004013C2 VJnz            endp

13. VJs，条件跳转指令，当 `var_SFlag=1` 时触发，偏移量为操作码后面紧跟的一个 word（相对于伪代码区基址）

004013C7 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

004013C7

004013C7 ; Attributes: bp-based frame

004013C7

004013C7 VJs             proc near

004013C7

004013C7 var\_RegEip      = dword ptr -14h

004013C7 var\_RegDl       = byte ptr -10h

004013C7 var\_7           = dword ptr -7

004013C7 var\_SFlag       = byte ptr -3

004013C7 arg\_lpVMCode     = dword ptr  10h

004013C7

004013C7                 inc     [ebp+var\_RegEip]

004013CA                 mov     eax, [ebp+var\_RegEip]

004013CD                 movzx   eax, byte ptr [eax]

004013D0                 mov     [ebp+var\_RegDl], al

004013D3                 shl     eax, 8

004013D6                 mov     [ebp+var\_7], eax

004013D9                 inc     [ebp+var\_RegEip]

004013DC                 mov     eax, [ebp+var\_RegEip]

004013DF                 movzx   eax, byte ptr [eax]

004013E2                 mov     [ebp+var\_RegDl], al

004013E5                 add     [ebp+var\_7], eax

004013E8                 mov     eax, [ebp+var\_7]

004013EB                 cmp     [ebp+var\_SFlag], 1  ; 标志位测试

004013EF                 jnz     short loc\_4013F7

004013F1                 add     eax, [ebp+arg\_lpVMCode]

004013F4                 mov     [ebp+var\_RegEip], eax

004013F7

004013F7 loc\_4013F7:

004013F7                 jmp     InterpretingEntry

004013F7 VJs             endp

6. VRet10，返回指令，从虚拟机解释引擎中返回

004013FC ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

004013FC

004013FC

004013FC VRet10          proc near

004013FC                 leave

004013FD                 retn    10h

004013FD VRet10          endp ; sp =  4

004013FD

在这套解释系统中，比较值得注意的是它的堆栈操作与 x86 下有所不同：

入栈 —— `add esp, 4`
`mov [esp], xxx`

出栈 —— `sub esp, 4`
`mov xxxx, [esp]`

看起来有些不可思议，不过由于该引擎同时用到了 `var_RegEax` 来传递数据，所以没有出现问题。

以上面的分析为基础，我们来反编译一下 004020CF 处的伪指令组——对序列号的处理：

004020BF lpVMData        dd 0
004020C3                dd 0
004020C7                dd 1
004020CB ; UINT uType
004020CB uType           dd 10h                 ;
004020CF lpVMCode\_4020CF db    0 ;               ; VNop
004020D0                db 1, 30h               ; VPushImm32 30h
004020D2                db  0Eh ;               ; VPushByteVArg
004020D3                db    5 ;               ; VSub
004020D4                db 2 dup(2)             ; VLoadMem32 004020C7
004020D6                db    8 ;               ; VMul
004020D7                db 2, 1                ; VLoadMem32 004020C3
004020D9                db    4 ;               ; VAdd
004020DA                db 3, 1                ; VSaveMem32 004020C3
004020DC                db 2 dup(2)            ; VLoadMem32 004020C7
004020DE                db 1, 0Ah              ; VPushImm32 0Ah
004020E0                db    8 ;               ; VMul
004020E1                db 3, 2                ; VSaveMem32 004020C7
004020E3                db 1, 30h              ; VPushImm32 30h
004020E5                db  0Eh ;               ; VPushByteVArg
004020E6                db    7 ;               ; VPushRegEax
004020E7                db 1, 0                ; VPushImm32 0
004020E9                db    5 ;               ; VSub
004020EA                db  0Dh ;               ; VPop2RegEax
004020EB                db 13h, 0, 3           ; VJs 004020D2
004020EE                db  0Dh ;               ; VPop2RegEax
004020EF                db 1, 0EBh             ; VPushImm32 0EBh
004020F1                db 2, 1                ; VLoadMem32 004020C3
004020F3                db    5 ;               ; VSub
004020F4                db 3, 1                ; VSaveMem32 004020C3
004020F6                db    6 ;               ; VRet10

这段伪代码的作用是把输入的序列号字符串转换为十进制数，然后减去 0EBh，保存至 004020C3

其他部分的虚拟机代码也可以做类似的分析，这里留给读者自己思考:D

cyclotron
2007.3.10

`4dd466d0e1e0cc123afbe504da2462ca`
