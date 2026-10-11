
### 内存断点范围调整与消息提示

```
00419279  .|  51              push    ecx                         ; |Arg4 => 00400533
0041927A  .|  8D8424 08010000 lea     eax, dword ptr [esp+108]    ; |
00419281  .|  50              push    eax                         ; |Arg3
00419282  .|  68 86214B00    push    004B2186                    ; |ollydbg 在全部指定的地址范围（%s）中无法激活内存断点。断点已经减少范围为 %08lx..%08lx.
00419287  .|  8D5424 10      lea     edx, dword ptr [esp+10]      ; |
0041928B  .|  52              push    edx                         ; |Arg1
0041928C  .|  E8 9BD90800    call    004A6C2C                    ; \\_sprintf
00419291  .|  83C4 14        add     esp, 14
00419294  .|  6A 10          push    10                          ; /Style = MB_OK
00419296  .|  68 0E224B00    push    004B220E                    ; |内存断点的范围被减少
0041929B  .|  8D4C24 08      lea     ecx, dword ptr [esp+8]       ; |
0041929F  .|  51              push    ecx                         ; |Text
004192A0  .|  A1 7C3B4D00    mov     eax, dword ptr [4D3B7C]     ; |
004192A5  .|  50              push    eax                         ; |hOwner
004192A6  .|  E8 6B620900    call    <jmp.&USER32.MessageBoxA>  ; \\MessageBoxA
```

将内存断点的范围缩小为设置失败前已经成功设置的内存页范围：

```
004192AB  .|  8BD3            mov     edx, ebx
004192AD  .|  2B15 3C814D00   sub     edx, dword ptr [4D813C]     ; Ollydbg.00400533
004192B3  .|  8915 40814D00   mov     dword ptr [4D8140], edx
004192B9  .|  891D 48814D00   mov     dword ptr [4D8148], ebx
```

设置检查标识为 1，标识已经检查完毕，然后返回 0 并退出：

```
004192BF  |>  C705 34814D00 0>mov     dword ptr [4D8134], 1
004192C9  .|  33C0            xor     eax, eax
004192CB  |>  81C4 1C020000   add     esp, 21C
004192D1  .|  5D              pop     ebp
004192D2  .|  5F              pop     edi
004192D3  .|  5E              pop     esi
004192D4  .|  5B              pop     ebx
004192D5  \.  C3              retn
```

### 内存断点处理流程说明

最后是内存断点的处理，该部分用 `WaitForDebugEvent` 函数捕获了内存断点异常，然后对内存断点进行检查以及 OD 界面进行相关设置，由于之后没有用 `ContinueDebugEvent` 函数，被调试程序被断在了该异常处：

1.  得到线程信息结构体，将结构体指针赋值给局部变量；
2.  根据异常信息结构体中的异常事件标识，跳转到相应的异常的处理分支处；
3.  给异常地址变量赋值，若得到线程信息，则根据线程信息的 `eip` 赋值，否则根据异常地址赋值，并且将异常地址赋值给几个全局变量；
4.  得到异常所在的模块信息，提取异常所在处的汇编代码，并检查模块信息是否正确，汇编代码有没有获得，根据情况做相应的错误处理；
5.  若是模块为自解压（SFX）模式，则进行相应的检查以及错误处理；
6.  检查内存断点是否在 `kernel32.dll` 中，弹出提示窗口，并将断点去除；
7.  最后调整优先级并退出

这里是所有异常的处理函数，除了内存断点相关的代码，其他都会被省略：

#### 首先保存环境，拉伸栈空间

```
0042EBD0  /$  55             push    ebp
0042EBD1  .|  8BEC          mov     ebp, esp
0042EBD3  .|  81C4 04F0F>add     esp, -0FFC
0042EBD9  .|  50             push    eax
0042EBDA  .|  81C4 00F5F>add     esp, -0B00
```

得到调试线程的线程信息结构体，该函数在《OD 获得当前 cpu 工作状态的方法》中已详细分析：

```
0042EBE0  .|  53             push    ebx
0042EBE1  .|  56             push    esi
0042EBE2  .|  57             push    edi
0042EBE3  .|  8B35 1C574>mov     esi, dword ptr [4D571C]  ; DebugEvent.dwThreadId
0042EBE9  .|  56             push    esi
0042EBEA  .|  E8 5DF8FFF>call    0042E44C       ; GetThreadInfo
```

#### 将得到的寄存器结构体指针赋值给参数

```
0042EBEF  .|  8BF8          mov     edi, eax
0042EBF1  .|  8B45 08      mov     eax, dword ptr [ebp+8]
0042EBF4  .|  59             pop     ecx
0042EBF5  .|  8938          mov     dword ptr [eax], edi  ; [ebp+8] = pReg
```

#### 根据异常事件标识跳转到相应的处理函数

```
0042EBF7  .|  8B15 14574>mov     edx, dword ptr [4D5714] ; DebugEvent.dwDebugEventCode
0042EBFD  .|  83FA 09      cmp     edx, 9                ;  Switch (cases 1..9)
0042EC00  .|  0F87 EE270>ja       004313F4
0042EC06  .|  FF2495 0DE>jmp      dword ptr [edx*4+42EC0D]
```

#### 调试异常的跳表

```
0042EC0D  .|  F4134300     dd      Ollydbg.004313F4    ; Default
0042EC11  .|  35EC4200     dd      Ollydbg.0042EC35    ; EXCEPTION_DEBUG_EVENT
0042EC15  .|  FF0C4300     dd      Ollydbg.00430CFF    ; CREATE_THREAD_DEBUG_EVENT
0042EC19  .|  D70D4300     dd      Ollydbg.00430DD7  ; CREATE_PROCESS_DEBUG_EVENT
0042EC1D  .|  3F0F4300     dd      Ollydbg.00430F3F    ; EXIT_THREAD_DEBUG_EVENT
0042EC21  .|  37104300     dd      Ollydbg.00431037    ; EXIT_PROCESS_DEBUG_EVENT
0042EC25  .|  2D114300     dd      Ollydbg.0043112D   ; LOAD_DLL_DEBUG_EVENT
0042EC29  .|  B7114300     dd      Ollydbg.004311B7    ; UNLOAD_DLL_DEBUG_EVENT
0042EC2D  .|  76124300     dd      Ollydbg.00431276    ; OUTPUT_DEBUG_STRING_EVENT
0042EC31  .|  C7134300     dd      Ollydbg.004313C7    ; RIP_EVENT
```

#### 处理调试异常

这里为调试异常处理处，给异常标识赋值，并将异常结构体指针赋值给局部变量：

```
0042EC35  |>  8B0D 0C364>mov     ecx, dword ptr [4E360C] ;
0042EC3B  .|  33C0          xor     eax, eax
0042EC3D  .|  894D EC      mov     dword ptr [ebp-14], ecx
0042EC40  .|  A3 0C364E0>mov     dword ptr [4E360C], eax
0042EC45  .|  C745 A4 20>mov     dword ptr [ebp-5C], 004D5720  ; DebugEvent.u
```

检查异常信息结构体中寄存器结构体是否得到，若得到则跳过下面取寄存器信息：

```
0042EC4C  .|  85FF          test    edi, edi
0042EC4E  .|  75 0D        jnz     short 0042EC5D
```

若没有得到了寄存器信息，则将异常信息中异常地址赋值给参数：

```
0042EC50  .|  8B55 A4      mov     edx, dword ptr [ebp-5C]
0042EC53  .|  33DB          xor     ebx, ebx
0042EC55  .|  8B4A 0C      mov     ecx, dword ptr [edx+C]
0042EC58  .|  894D D8      mov     dword ptr [ebp-28], ecx  ; 参数赋值为 ExceptionAddress
0042EC5B  .|  EB 22        jmp     short 0042EC7F
```

若得到了寄存器信息，则将寄存器中的 `EIP` 地址赋值给参数，检查异常标识，若不为 0 则修改寄存器里面的标识，并设置寄存器已被修改：

```
0042EC5D  |>  8B47 2C      mov     eax, dword ptr [edi+2C]
0042EC60  .|  8945 D8      mov     dword ptr [ebp-28], eax  ; 参数赋值为 t_reg.r_eip
0042EC63  .|  837D EC 00  cmp     dword ptr [ebp-14], 0    ; 检查异常标识
0042EC67  .|  8B5F 10      mov     ebx, dword ptr [edi+10]    ; ebx : t_reg.r_ecx
0042EC6A  .|  74 13        je      short 0042EC7F      ; 若标识为 0 则跳过下面的操作
0042EC6C  .|  F647 31 01  test    byte ptr [edi+31], 1
0042EC70  .|  74 0D        je      short 0042EC7F
0042EC72  .|  8167 30 FF>and      dword ptr [edi+30], FFFFFE>  ; 修改标识
0042EC79  .|  C707 01000>mov      dword ptr [edi], 1    ; 设置寄存器修改标识
```

检查是否是单步异常，若是则跳过下面的代码，若不是则跳转到下面继续判断是什么异常：

```
0042EC7F  |>  8B45 A4      mov     eax, dword ptr [ebp-5C]
0042EC82  .|  8138 03000>cmp      dword ptr [eax], 80000003
0042EC88  .|  74 07        je      short 0042EC91
0042EC8A  .|  33D2          xor     edx, edx
0042EC8C  .|  8955 DC      mov     dword ptr [ebp-24], edx
0042EC8F  .|  EB 79        jmp     short 0042ED0A
```

跳过对单步的分析，不是重点。。。：

```
0042EC91  |>  6A 02        push    2                       ; /Arg4 = 00000002
0042EC93  .|  6A 01        push    1                       ; |Arg3 = 00000001
0042EC95  .|  8B4D D8      mov     ecx, dword ptr [ebp-28]    ; |
0042EC98  .|  49             dec     ecx                      ; |
0042EC99  .|  51             push    ecx                      ; |Arg2
0042EC9A  .|  8D45 BB      lea     eax, dword ptr [ebp-45]    ; |
0042EC9D  .|  50             push    eax                      ; |Arg1
0042EC9E  .|  E8 6926030>call    _Readmemory               ; \\_Readmemory
0042ECA3  .|  83C4 10      add     esp, 10
0042ECA6  .|  83F8 01      cmp     eax, 1
0042ECA9  .|  74 07        je      short 0042ECB2
0042ECAB  .|  33D2          xor     edx, edx
0042ECAD  .|  8955 DC      mov     dword ptr [ebp-24], edx
0042ECB0  .|  EB 58        jmp     short 0042ED0A
0042ECB2  |>  33C0          xor     eax, eax
0042ECB4  .|  8A45 BB      mov     al, byte ptr [ebp-45]
0042ECB7  .|  3D CC00000>cmp      eax, 0CC
0042ECBC  .|  75 09        jnz     short 0042ECC7
0042ECBE  .|  C745 DC 01>mov      dword ptr [ebp-24], 1
0042ECC5  .|  EB 43        jmp     short 0042ED0A
0042ECC7  |>  83F8 03      cmp     eax, 3
0042ECCA  .|  74 07        je      short 0042ECD3
0042ECCC  .|  33D2          xor     edx, edx
0042ECCE  .|  8955 DC      mov     dword ptr [ebp-24], edx
0042ECD1  .|  EB 37        jmp     short 0042ED0A
0042ECD3  |>  6A 02        push    2                       ; /Arg4 = 00000002
0042ECD5  .|  6A 01        push    1                       ; |Arg3 = 00000001
0042ECD7  .|  8B4D D8      mov     ecx, dword ptr [ebp-28]    ; |
```
