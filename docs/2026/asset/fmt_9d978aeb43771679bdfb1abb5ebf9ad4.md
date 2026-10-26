
# [原创]菜鸟 学注册机编写之 “查表”

测试环境

- 系统: xp sp3
- 调试器: od 1.10

高手不要见笑，仅供小菜玩乐，有不对或不足的地方还请多多指教，不胜感激!

1. 首先运行程序随便输入用户与注册码如下图所示:

![](https://bbs.kanxue.com/upload/attach/201411/536985_jl7cxoy9lw3k56a.jpg)

2. 载入OD通过下`MessageBoxA`函数, F9运行程序, 随便输入用户名与注册码, 点ok后断下，如下图所示:

![](https://bbs.kanxue.com/upload/attach/201411/536985_qjt4dg8h42s2kr3.jpg)

3. 然后堆栈回溯,来到如下关键地方

```
00415BB0                    6A FF           push -0x1
00415BB2                    68 58514200     push ahead_dv.00425158
00415BB7                    64:A1 00000000  mov eax,dword ptr fs:[0]
00415BBD                    50              push eax
00415BBE                    64:8925 0000000>mov dword ptr fs:[0],esp
00415BC5                    83EC 08         sub esp,0x8
00415BC8                    56              push esi
00415BC9                    8BF1            mov esi,ecx
00415BCB                    8D4C24 04       lea ecx,dword ptr ss:[esp+0x4]
00415BCF                    E8 74C90000     call <jmp.&MFC42.#??0CString@@QAE@XZ_540>
00415BD4                    6A 01           push 0x1
00415BD6                    8BCE            mov ecx,esi
00415BD8                    C74424 18 00000>mov dword ptr ss:[esp+0x18],0x0
00415BE0                    E8 7BC90000     call <jmp.&MFC42.#?UpdateData@CWnd@@QAEHH@Z_6334>        ; ----获得注册码
00415BE5                    E8 8EC90000     call <jmp.&MFC42.#?AfxGetModuleState@@YGPAVAFX_MODULE_ST>
00415BEA                    8B48 04         mov ecx,dword ptr ds:[eax+0x4]
00415BED                    E8 44CF0000     call <jmp.&MFC42.#?BeginWaitCursor@CCmdTarget@@QAEXXZ_16>
00415BF2                    8B46 64         mov eax,dword ptr ds:[esi+0x64]
00415BF5                    8B4E 60         mov ecx,dword ptr ds:[esi+0x60]
00415BF8                    50              push eax                                                 ; ----注册码
00415BF9                    51              push ecx                                                 ; ----用户名
00415BFA                    C64424 1C 01    mov byte ptr ss:[esp+0x1C],0x1
00415BFF                    E8 CCFCFFFF     call ahead_dv.004158D0                                   ; 关键CALL 注册码相同则返回1
00415C04                    83C4 08         add esp,0x8
00415C07                    85C0            test eax,eax
00415C09                    75 18           jnz short ahead_dv.00415C23                              ; 关键跳
00415C0B                    6A 40           push 0x40
00415C0D                    68 6C294300     push ahead_dv.0043296C                                   ; ASCII "Sorry"
00415C12                    68 40294300     push ahead_dv.00432940                                   ; ASCII "Invalid username or  

registration code      "
00415C17                    8BCE            mov ecx,esi
00415C19                    E8 82CB0000     call <jmp.&MFC42.#?MessageBoxA@CWnd@@QAEHPBD0I@Z_4224>   ; 提示无效注册码
00415C1E                    E9 BB000000     jmp ahead_dv.00415CDE
```

4. 分析关键CALL 004158D0 ; 关键CALL 注册码相同则返回1

```
004158D0                53              push ebx
004158D1                55              push ebp
004158D2                8B6C24 0C       mov ebp,dword ptr ss:[esp+0xC]
004158D6                56              push esi
004158D7                57              push edi
004158D8                BE 60354300     mov esi,ahead_dv.00433560
004158DD                8BC5            mov eax,ebp
004158DF                8A10            mov dl,byte ptr ds:[eax]                                 ; ---取用户名1字节
004158E1                8A1E            mov bl,byte ptr ds:[esi]
004158E3                8ACA            mov cl,dl                                                ; ---用户名第1字节
004158E5                3AD3            cmp dl,bl                                                ; ---判断是否为0
004158E7                75 1E           jnz short ahead_dv.00415907
004158E9                84C9            test cl,cl
004158EB                74 16           je short ahead_dv.00415903
004158ED                8A50 01         mov dl,byte ptr ds:[eax+0x1]
004158F0                8A5E 01         mov bl,byte ptr ds:[esi+0x1]
004158F3                8ACA            mov cl,dl
004158F5                3AD3            cmp dl,bl
004158F7                75 0E           jnz short ahead_dv.00415907
004158F9                83C0 02         add eax,0x2
004158FC                83C6 02         add esi,0x2
004158FF                84C9            test cl,cl
00415901              ^ 75 DC           jnz short ahead_dv.004158DF
00415903                33C0            xor eax,eax
00415905                EB 05           jmp short ahead_dv.0041590C
00415907                1BC0            sbb eax,eax
00415909                83D8 FF         sbb eax,-0x1
0041590C                85C0            test eax,eax
0041590E                74 51           je short ahead_dv.00415961
00415910                8B7C24 18       mov edi,dword ptr ss:[esp+0x18]                          ; --注册码
00415914                BE 60354300     mov esi,ahead_dv.00433560
00415919                8BC7            mov eax,edi
0041591B                8A10            mov dl,byte ptr ds:[eax]                                 ; ---取注册码1字节
0041591D                8A1E            mov bl,byte ptr ds:[esi]
0041591F                8ACA            mov cl,dl                                                ; 注册码第1字节
00415921                3AD3            cmp dl,bl                                                ; ---判断是否为0
00415923                75 1E           jnz short ahead_dv.00415943
00415925                84C9            test cl,cl
00415927                74 16           je short ahead_dv.0041593F
00415929                8A50 01         mov dl,byte ptr ds:[eax+0x1]
0041592C                8A5E 01         mov bl,byte ptr ds:[esi+0x1]
0041592F                8ACA            mov cl,dl
00415931                3AD3            cmp dl,bl
00415933                75 0E           jnz short ahead_dv.00415943
00415935                83C0 02         add eax,0x2
00415938                83C6 02         add esi,0x2
0041593B                84C9            test cl,cl
0041593D              ^ 75 DC           jnz short ahead_dv.0041591B
0041593F                33C0            xor eax,eax
00415941                EB 05           jmp short ahead_dv.00415948
00415943                1BC0            sbb eax,eax
00415945                83D8 FF         sbb eax,-0x1
00415948                85C0            test eax,eax
0041594A                74 15           je short ahead_dv.00415961
0041594C                57              push edi                                                 ; 注册码
0041594D                55              push ebp                                                 ; 用户名
0041594E                E8 3DFDFFFF     call ahead_dv.00415690                                   ; 算法
```

5. 算法分析
