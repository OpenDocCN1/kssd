
```
00455D4A   .   8BCE                mov     ecx, esi  

00455D4C   .   E8 A5880000         call    <jmp.&MFC42.#3092_CWnd::GetDlgItem>          ;  改变第一个文本框属性  

00455D51   .   8BC8                mov     ecx, eax  

00455D53   .   E8 10890000         call    <jmp.&MFC42.#2642_CWnd::EnableWindow>       ;  变为“Disabled”  

00455D58   .   6A 00               push    0  

00455D5A   .   68 2D040000        push    42D  

00455D5F   .   8BCE                mov     ecx, esi  

00455D61   .   E8 90880000         call    <jmp.&MFC42.#3092_CWnd::GetDlgItem>          ;  改变第二个文本框属性  

00455D66   .   8BC8                mov     ecx, eax  

00455D68   .   E8 FB880000         call    <jmp.&MFC42.#2642_CWnd::EnableWindow>       ;  变为“Disabled”  

00455D6D   .   6A 00               push    0  

00455D6F   .   68 2E040000        push    42E  

00455D74   .   8BCE                mov     ecx, esi  

00455D76   .   E8 7B880000         call    <jmp.&MFC42.#3092_CWnd::GetDlgItem>          ;  改变第三个文本框属性  

00455D7B   .   8BC8                mov     ecx, eax  

00455D7D   .   E8 E6880000         call    <jmp.&MFC42.#2642_CWnd::EnableWindow>       ;  变为“Disabled”  

00455D82   .   6A 00               push    0  

00455D84   .   68 2F040000        push    42F  

00455D89   .   8BCE                mov     ecx, esi  

00455D8B   .   E8 66880000         call    <jmp.&MFC42.#3092_CWnd::GetDlgItem>          ;  改变第四个文本框属性  

00455D90   .   8BC8                mov     ecx, eax  

00455D92   .   E8 D1880000         call    <jmp.&MFC42.#2642_CWnd::EnableWindow>       ;  变为“Disabled”  

00455D97   .   6A 00               push    0  

00455D99   .   68 30040000        push    430  

00455D9E   .   8BCE                mov     ecx, esi  

00455DA0   .   E8 51880000         call    <jmp.&MFC42.#3092_CWnd::GetDlgItem>          ;  改变第五个文本框属性  

00455DA5   .   8BC8                mov     ecx, eax  

00455DA7   .   E8 BC880000         call    <jmp.&MFC42.#2642_CWnd::EnableWindow>       ;  变为“Disabled”  

00455DAC   .   6A 00               push    0  

00455DAE   .   6A 01               push    1  

00455DB0   .   8BCE                mov     ecx, esi  

00455DB2   .   E8 3F880000         call    <jmp.&MFC42.#3092_CWnd::GetDlgItem>          ;  改变第一个按钮属性  

00455DB7   .   8BC8                mov     ecx, eax  

00455DB9   .   E8 AA880000         call    <jmp.&MFC42.#2642_CWnd::EnableWindow>       ;  变为“Disabled”  

00455DBE   >   8D4C24 24           lea     ecx, dword ptr [esp+24]  

00455DC2   .   C68424 24010000 0   mov     byte ptr [esp+124], 5  

00455DCA   .   E8 79870000         call    <jmp.&MFC42.#800_CString::~CString>  

00455DCF   .   8D4C24 2C           lea     ecx, dword ptr [esp+2C]  

00455DD3   .   C68424 24010000 0   mov     byte ptr [esp+124], 4  

00455DDB   .   E8 68870000         call    <jmp.&MFC42.#800_CString::~CString>  

00455DE0   .   8D4C24 18           lea     ecx, dword ptr [esp+18]  

00455DE4   .   C68424 24010000 0   mov     byte ptr [esp+124], 3  

00455DEC   .   E8 57870000         call    <jmp.&MFC42.#800_CString::~CString>  

00455DF1   .   8D4C24 10           lea     ecx, dword ptr [esp+10]  

00455DF5   .   C68424 24010000 0   mov     byte ptr [esp+124], 2  

00455DFD   .   E8 46870000         call    <jmp.&MFC42.#800_CString::~CString>  

00455E02   .   8D4C24 1C           lea     ecx, dword ptr [esp+1C]  

00455E06   .   C68424 24010000 0   mov     byte ptr [esp+124], 1  

00455E0E   .   E8 35870000         call    <jmp.&MFC42.#800_CString::~CString>  

00455E13   .   8D4C24 14           lea     ecx, dword ptr [esp+14]  

00455E17   .   C68424 24010000 0   mov     byte ptr [esp+124], 0  

00455E1F   .   E8 24870000         call    <jmp.&MFC42.#800_CString::~CString>  

00455E24   .   8D4C24 20           lea     ecx, dword ptr [esp+20]  

00455E28   .   C78424 24010000 F   mov     dword ptr [esp+124], -1  

00455E33   .   E8 10870000         call    <jmp.&MFC42.#800_CString::~CString>  

00455E38   .   8B8C24 1C010000     mov     ecx, dword ptr [esp+11C]  

00455E3F   .   5F                  pop     edi  

00455E40   .   5E                  pop     esi  

00455E41   .   5D                  pop     ebp  

00455E42   .   64:890D 00000000    mov     dword ptr fs:[0], ecx  

00455E49   .   5B                  pop     ebx  

00455E4A   .   81C4 18010000       add     esp, 118  

00455E50   .   C3                  retn  

```

跟进 `00455AAB` 来到：

```
004557F0  /$   8B4424 08            mov     eax, dword ptr [esp+8]                          ;  MD5 加密运算开始  
004557F4  |.   53                  push    ebx  
004557F5  |.   8B5C24 08           mov     ebx, dword ptr [esp+8]  
004557F9  |.   55                  push    ebp  
004557FA  |.   8B6C24 14           mov     ebp, dword ptr [esp+14]  
004557FE  |.   85C0                test    eax, eax  
00455800  |.   C645 00 00          mov     byte ptr [ebp], 0  
00455804  |.   7E 4D              jle     short 00455853  
00455806  |.   56                  push    esi  
00455807  |.   57                  push    edi  
00455808  |.   894424 1C           mov     dword ptr [esp+1C], eax  
0045580C  |>   8A0B                /mov    cl, byte ptr [ebx]  
0045580E  |.   8D4424 14           |lea     eax, dword ptr [esp+14]  
00455812  |.   50                  |push    eax  
00455813  |.   51                  |push    ecx  
00455814  |.   E8 A7FFFFFF         |call    004557C0  
00455819  |.   8D7C24 1C           |lea     edi, dword ptr [esp+1C]  
0045581D  |.   83C9 FF             |or      ecx, FFFFFFFF  
00455820  |.   33C0                |xor     eax, eax  
00455822  |.   83C4 08             |add     esp, 8  
00455825  |.   F2:AE               |repne   scas byte ptr es:[edi]  
00455827  |.   F7D1                |not     ecx  
00455829  |.   2BF9                |sub     edi, ecx  
0045582B  |.   8BF7                |mov     esi, edi  
0045582D  |.   8BD1                |mov     edx, ecx  
0045582F  |.   8BFD                |mov     edi, ebp  
00455831  |.   83C9 FF             |or      ecx, FFFFFFFF  
00455834  |.   F2:AE               |repne   scas byte ptr es:[edi]  
00455836  |.   8BCA                |mov     ecx, edx  
00455838  |.   4F                  |dec     edi  
00455839  |.   C1E9 02             |shr     ecx, 2  
0045583C  |.   F3:A5               |rep     movs dword ptr es:[edi], dword ptr [esi]  
0045583E  |.   8B4424 1C           |mov     eax, dword ptr [esp+1C]  
00455842  |.   8BCA                |mov     ecx, edx  
00455844  |.   83E1 03             |and     ecx, 3  
00455847  |.   43                  |inc     ebx  
00455848  |.   48                  |dec     eax  
00455849  |.   F3:A4               |rep     movs byte ptr es:[edi], byte ptr [esi]  
0045584B  |.   894424 1C           |mov     dword ptr [esp+1C], eax  
0045584F  |.^  75 BB              \jnz    short 0045580C                            ;  循环运算  
00455851  |.   5F                  pop     edi  
00455852  |.   5E                  pop     esi  
00455853  |>   5D                  pop     ebp                                    ;  ASCII "4B6A57EE120000AD290685C74932EB9D"  
00455854  |.   5B                  pop     ebx  
00455855  \\.  C3                  retn                                        ;  返回程序  

```

### 注册算法

算法与“用户名”、“地区”无关，标准的 `MD5` 算法加密

`SN = MD5（注册日期+"goodsoft"+邮箱）`

★目标程序、文章及注册机源程序见附件★

--------------------------------------------------------------------------------

### 经验总结

新年快乐！没什么好总结的，凑合看吧！

--------------------------------------------------------------------------------

### 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 02 月 18 日 AM 10:43:46
