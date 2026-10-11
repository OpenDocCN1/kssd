
# COM Explorer 2.0 算法分析

> 原文：[`www.pediy.com/kssd/pediy10/72984.html`](https://www.pediy.com/kssd/pediy10/72984.html)

【文章标题】: `COM Explorer 2.0 算法分析`  
【文章作者】: `qifeon`  
【软件名称】: `COM Explorer`  
【下载地址】: 自己搜索下载  
【保护方式】: 注册码  
【使用工具】: `OD`, `PEID`  
【操作平台】: `WINXP SP2`  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

【详细过程】

一、`Peid` 查壳，显示无壳。编程语言为：`Microsoft Visual C++ 6.0`。

二、试运行软件，默认用户名为“`UNREGISTERED USER`”，不是一般为空。这点关系到后来对用户名的检验。直接点击“`unlock`”按钮，则程序直接退出。

初步探索有利于我们调试时理解程序流程。输入试炼码“`abcdefghijk,123456`”，出现错误对话框。很长，记了个“`re-enter`”。

三、分析过程

`OD` 载入程序，利用字符串插件，搜索“`re-enter`”来到

`Ultra String Reference`, 条目 214

`Address=00409082`

`Disassembly=push    004641AC`

`Text String=the license information you have entered could not be validated.\nplease re-enter the user id and serial number you have legaly`

`obtained from 4developers llc.\n\nif you have not ordered yet you can click one of the buttons below to see how to order com ...or`

双击后返回到程序代码处

代码:

```
00409074   > \85F6             test    esi, esi                               ;  由此处向上找跳转
 00409076   .  74 03            je      short 0040907B
 00409078   .  8B76 1C         mov     esi, dword ptr [esi+1C]
 0040907B   >  6A 40           push    40                                       ; /Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL
 0040907D   .  68 D4424600    push    004642D4                                  ; |invalid license
 00409082   .  68 AC414600    push    004641AC                                  ; |the license information you have entered could not be validated.\nplease re-enter ..
 00409087   .  56             push    esi                                       ; |返回处
 00409088   .  FF15 E8054500 call    dword ptr [<&USER32.MessageBoxA>>; \MessageBoxA
 0040908E   .  5E             pop     esi
 0040908F   .  5B             pop     ebx
 00409090   .  83C4 08        add     esp, 8
 00409093   .  C3             retn
```

******************************************************************************

`00409074   >` 处向上找跳转

******************************************************************************

代码:

```
00409028   > \6A FF           push    -1                                       ; /接着向上找跳转
 0040902A   .  FF15 FC054500 call    dword ptr [<&USER32.MessageBeep>>; \MessageBeep
 00409030   .  8BCE          mov     ecx, esi
 00409032   .  E8 69F9FFFF   call    004089A0
 00409037   .  85C0          test    eax, eax
 00409039   .  74 39         je      short 00409074                             ;  来到这个跳转
 0040903B   .  85F6          test    esi, esi
 0040903D   .  75 19         jnz     short 00409058
 0040903F   .  6A 40         push    40                                         ; /Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL
 00409041   .  68 18444600  push    00464418                                    ; |outdated license
 00409046   .  68 E4424600  push    004642E4                                    ; |the license information you have entered is for com explorer 1.x only.\n\nto unlock version 2.0,..
 0040904B   .  56           push    esi                                         ; |hOwner
 0040904C   .  FF15 E8054500 call    dword ptr [<&USER32.MessageBoxA>>; \MessageBoxA
 00409052   .  5E           pop     esi
 00409053   .  5B           pop     ebx
 00409054   .  83C4 08      add     esp, 8
 00409057   .  C3           retn
 00409058   >  8B76 1C      mov     esi, dword ptr [esi+1C]
 0040905B   .  6A 40        push    40                                           ; /Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL
 0040905D   .  68 18444600 push    00464418                                      ; |outdated license
 00409062   .  68 E4424600 push    004642E4                                      ; |the license information you have entered is for com explorer 1.x only.\n\nto unlock version 2.0,
 00409067   .  56          push    esi                                           ; |hOwner
 00409068   .  FF15 E8054500 call    dword ptr [<&USER32.MessageBoxA>>; \MessageBoxA
 0040906E   .  5E          pop     esi
 0040906F   .  5B          pop     ebx
 00409070   .  83C4 08     add     esp, 8
 00409073   .  C3          retn
```

******************************************************************************

段首处 `00409028   >` 再接着往上找跳转

******************************************************************************

代码:



```asm
00408F00   .  83EC 08       sub     esp, 8                              段首
  00408F03   .  53          push    ebx
  00408F04   .  56          push    esi
  00408F05   .  8BF1        mov     esi, ecx
  00408F07   .  68 FF000000 push    0FF
  00408F0C   .  68 68994600 push    00469968                              ;  abcdefghijk
  00408F11   .  8D8E A8040000 lea    ecx, dword ptr [esi+4A8]
  00408F17   .  E8 9F990200 call    004328BB
  00408F1C   .  68 FF000000 push    0FF
  00408F21   .  68 689A4600 push    00469A68                              ;  12345678
  00408F26   .  8D8E 6C040000 lea    ecx, dword ptr [esi+46C]
  00408F2C   .  E8 8A990200 call    004328BB
  00408F31   .  68 68994600 push    00469968                              ;  abcdefghijk
  00408F36   .  8D4C24 0C   lea     ecx, dword ptr [esp+C]
  00408F3A   .  E8 12600200 call    0042EF51
  00408F3F   .  8B00        mov     eax, dword ptr [eax]
  00408F41   .  68 98414600 push    00464198                              ;  unregistered user
  00408F46   .  50          push    eax
  00408F47   .  E8 33330100 call    0041C27F                              ;  比较用户名是否是“UNREGISTERED USER”？
  00408F4C   .  83C4 08     add     esp, 8
  00408F4F   .  8D4C24 08   lea     ecx, dword ptr [esp+8]
  00408F53   .  85C0        test    eax, eax                              ;  (initial cpu selection)
  00408F55   .  0F94C3      sete    bl                                    ;  是则置 bl=1.否则为 0
  00408F58   .  E8 865F0200 call    0042EEE3
  00408F5D   .  84DB        test    bl, bl
  00408F5F   .  74 0F       je      short 00408F70                        ;  bl=1 则程序直接跳向退出
  00408F61   .  6A 01       push    1
  00408F63   .  8BCE        mov     ecx, esi
  00408F65   .  E8 93A70200 call    004336FD
  00408F6A   .  5E          pop     esi
  00408F6B   .  5B          pop     ebx
  00408F6C   .  83C4 08     add     esp, 8
  00408F6F   .  C3          retn
  00408F70   >  E8 1BFCFFFF call    00408B90                              ;  算法 call，可以再向上找按钮事件代码开始处
  00408F75   .  85C0        test    eax, eax
  00408F77   .  0F84 AB000000 je    00409028                              ;  关键跳转
  00408F7D   .  8B15 509C4600 mov    edx, dword ptr [469C50]
  00408F83   .  8D4424 0C   lea     eax, dword ptr [esp+C]
  00408F87   .  8D4C24 08   lea     ecx, dword ptr [esp+8]
  00408F8B   .  33DB        xor     ebx, ebx
  00408F8D   .  50          push    eax                                   ; /pDisposition
  00408F8E   .  51          push    ecx                                   ; |pHandle
  00408F8F   .  53          push    ebx                                   ; |pSecurity => NULL
  00408F90   .  68 3F000F00  push    0F003F                              ; |Access = KEY_ALL_ACCESS
  00408F95   .  53          push    ebx                                   ; |Options => REG_OPTION_NON_VOLATILE
  00408F96   .  53          push    ebx                                   ; |Class => NULL
  00408F97   .  53          push    ebx                                   ; |Reserved => 0
  00408F98   .  52          push    edx                                   ; |Subkey => "Software\\4Developers\\COMExp"
  00408F99   .  68 02000080 push    80000002                              ; |hKey = HKEY_LOCAL_MACHINE
  00408F9E   .  895C24 30   mov     dword ptr [esp+30], ebx             ; |
  00408FA2   .  895C24 2C   mov     dword ptr [esp+2C], ebx             ; |
  00408FA6   .  FF15 14004500 call    dword ptr [<&ADVAPI32.RegCreateK>; \\RegCreateKeyExA
  00408FAC   .  3BC3        cmp     eax, ebx
  00408FAE   .  75 31       jnz     short 00408FE1
  00408FB0   .  A1 503E4600 mov     eax, dword ptr [463E50]
  00408FB5   .  8B4C24 08   mov     ecx, dword ptr [esp+8]
  00408FB9   .  57          push    edi
  00408FBA   .  68 00020000 push    200                                   ; /BufSize = 200 (512.)
  00408FBF   .  68 68994600 push    00469968                              ; |abcdefghijk
  00408FC4   .  6A 03       push    3                                     ; |ValueType = REG_BINARY
  00408FC6   .  53          push    ebx                                   ; |Reserved => 0
  00408FC7   .  50          push    eax                                   ; |ValueName => "4D"
  00408FC8   .  51          push    ecx                                   ; |hKey
  00408FC9   .  FF15 10004500 call    dword ptr [<&ADVAPI32.RegSetValu>; \\RegSetValueExA
  00408FCF   .  8B5424 0C   mov     edx, dword ptr [esp+C]
  00408FD3   .  8BF8        mov     edi, eax
  00408FD5   .  52          push    edx                                   ; /hKey
  00408FD6   .  FF15 40004500 call    dword ptr [<&ADVAPI32.RegCloseKe>; \\RegCloseKey
  00408FDC   .  3BFB        cmp     edi, ebx
  00408FDE   .  5F          pop     edi
  00408FDF   .  74 1B       je      short 00408FFC
  00408FE1   >  53          push    ebx
  00408FE2   .  53          push    ebx
  00408FE3   .  68 B4444600 push    004644B4                              ;  failed to record license information.\n\nplease contact support@4dev.com
  00408FE8   .  E8 E8200300 call    0043B0D5
  00408FED   .  6A 01       push    1
  00408FEF   .  8BCE        mov     ecx, esi
  00408FF1   .  E8 07A70200 call    004336FD
  00408FF6   .  5E          pop     esi
  00408FF7   .  5B          pop     ebx
  00408FF8   .  83C4 08     add     esp, 8
  00408FFB   .  C3          retn
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

段首下断，输入输入试炼码“abcdefghijk,123456”后中断在这里，就可以调试了。单步，`00408F70`处进入算法 `call` `00408B90`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

代码:



## 反汇编代码

```
00408B90   /$   6A FF         push    -1
  00408B92   |.   68 98B64400   push    0044B698                       ;  SE 处理程序安装
  00408B97   |.   64:A1 0000000>mov     eax, dword ptr fs:[0]
  00408B9D   |.   50            push    eax
  00408B9E   |.   64:8925 00000>mov     dword ptr fs:[0], esp
  00408BA5   |.   83EC 08       sub     esp, 8
  00408BA8   |.   57            push    edi
  00408BA9   |.   BF 68994600   mov     edi, 00469968                    ;  abcdefghijk
  00408BAE   |.   83C9 FF       or      ecx, FFFFFFFF
  00408BB1   |.   33C0          xor     eax, eax
  00408BB3   |.   F2:AE         repne   scas byte ptr es:[edi]
  00408BB5   |.   F7D1          not     ecx
  00408BB7   |.   49            dec     ecx                              ;  上面是典型的取用户名长度代码
  00408BB8   |.   83F9 08       cmp     ecx, 8                           ;  用户名长度与 8 相比较
  00408BBB   |.   894C24 08     mov     dword ptr [esp+8], ecx
  00408BBF   |.   7D 10         jge     short 00408BD1                   ;  小于 8 位则跳向失败
  00408BC1   |.   5F            pop     edi
  00408BC2   |.   8B4C24 08     mov     ecx, dword ptr [esp+8]
  00408BC6   |.   64:890D 00000>mov     dword ptr fs:[0], ecx
  00408BCD   |.   83C4 14       add     esp, 14
  00408BD0   |.   C3            retn
  00408BD1   |>   53            push    ebx
  00408BD2   |.   55            push    ebp
  00408BD3   |.   56            push    esi
  00408BD4   |.   68 68994600   push    00469968                         ;  abcdefghijk
  00408BD9   |.   E8 34300200   call    0042BC12                         ;  用户名地址送入 eax
  00408BDE   |.   83C4 04       add     esp, 4
  00408BE1   |.   8D4C24 10     lea     ecx, dword ptr [esp+10]
  00408BE5   |.   68 FC404600   push    004640FC                         ;  6769-
  00408BEA   |.   E8 62630200   call    0042EF51                         ;  ASCII "6769-"地址入 eax
  00408BEF   |.   33F6          xor     esi, esi                         ;  esi 清零
  00408BF1   |.   897424 20     mov     dword ptr [esp+20], esi
  00408BF5   |>   8B2D 483E4600 /mov     ebp, dword ptr [463E48]          ;  固定字符串"0123456789"地址入 ebp
  00408BFB   |.   83C9 FF       |or      ecx, FFFFFFFF
  00408BFE   |.   8BFD          |mov     edi, ebp                         ;  ebp 内地址传入 edi
  00408C00   |.   33C0          |xor     eax, eax
  00408C02   |.   F2:AE         |repne   scas byte ptr es:[edi]
  00408C04   |.   F7D1          |not     ecx
  00408C06   |.   49            |dec     ecx                              ;  取固定字符串长度，即 ecx=0Ah
  00408C07   |.   8BC6          |mov     eax, esi                         ;  eax=esi
  00408C09   |.   33D2          |xor     edx, edx
  00408C0B   |.   8BFD          |mov     edi, ebp
  00408C0D   |.   F7F1          |div     ecx                              ;  eax 值除以 ecx,商保存在 eax，余数保存在 edx
  00408C0F   |.   8B4C24 14     |mov     ecx, dword ptr [esp+14]          ;  [esp+14]内值为用户名长度，传入 ecx
  00408C13   |.   8BC6          |mov     eax, esi                         ;  eax=esi
  00408C15   |.   0FBE1C2A      |movsx   ebx, byte ptr [edx+ebp]          ;  固定字符串逐位扩展传入 ebx
  00408C19   |.   33D2          |xor     edx, edx
  00408C1B   |.   F7F1          |div     ecx                              ;  eax 值除以 ecx,商保存在 eax，余数保存在 edx
  00408C1D   |.   0FBE82 689946>|movsx   eax, byte ptr [edx+469968]       ;  用户名逐位扩展传入 eax
  00408C24   |.   8D1440        |lea     edx, dword ptr [eax+eax*2]       ;  edx=eax*3
  00408C27   |.   8D0490        |lea     eax, dword ptr [eax+edx*4]       ;  eax=eax+edx*4
  00408C2A   |.   8BD6          |mov     edx, esi                         ;  edx=esi
  00408C2C   |.   0FAFD6        |imul    edx, esi                         ;  edx=edx*esi
  00408C2F   |.   0FAFD6        |imul    edx, esi                         ;  edx=edx*esi
  00408C32   |.   0FAFD1        |imul    edx, ecx                         ;  edx=edx*ecx
  00408C35   |.   03D8          |add     ebx, eax                         ;  ebx=ebx+eax
  00408C37   |.   83C9 FF       |or      ecx, FFFFFFFF
  00408C3A   |.   33C0          |xor     eax, eax
  00408C3C   |.   03DA          |add     ebx, edx                         ;  ebx=ebx+edx
  00408C3E   |.   F2:AE         |repne   scas byte ptr es:[edi]
  00408C40   |.   F7D1          |not     ecx
  00408C42   |.   49            |dec     ecx                              ;  取固定字符串长度，即 ecx=0Ah
  00408C43   |.   8BC3          |mov     eax, ebx                         ;  ebx 内值传入 eax
  00408C45   |.   33D2          |xor     edx, edx
  00408C47   |.   F7F1          |div     ecx                              ;  eax 值除以 ecx,商保存在 eax，余数保存在 edx,设为 yu
  00408C49   |.   8D4C24 10     |lea     ecx, dword ptr [esp+10]
  00408C4D   |.   8A042A        |mov     al, byte ptr [edx+ebp]           ;  上面余数 yu 对应固定字符串字符值传入 al
  00408C50   |.   50            |push    eax                              ;  第 yu 位值入栈
  00408C51   |.   E8 44660200   |call    0042F29A                         ;  连接字符
  00408C56   |.   85F6          |test    esi, esi
  00408C58   |.   74 1D         |je      short 00408C77                   ;  esi 为 0 则跳
  00408C5A   |.   8BC6          |mov     eax, esi                         ;  eax=esi
  00408C5C   |.   33D2          |xor     edx, edx                         ;  edx=0
  00408C5E   |.   B9 03000000   |mov     ecx, 3                           ;  ecx=3
  00408C63   |.   F7F1          |div     ecx                              ;  eax 值除以 ecx,商保存在 eax，余数保存在 edx
  00408C65   |.   85D2          |test    edx, edx                         ;  这里根据 esi 值是否能整除以 3 确定字符串连接方式
  00408C67   |.   75 0E         |jnz     short 00408C77                   ;  不能整除则跳
  00408C69   |.   68 F8404600   |push    004640F8                         ;  -
  00408C6E   |.   8D4C24 14     |lea     ecx, dword ptr [esp+14]
  00408C72   |.   E8 FC650200   |call    0042F273                         ;  能整除则在前面连接字符串后连接"-"
  00408C77   |>   46            |inc     esi                              ;  esi 增 1
  00408C78   |.   83FE 09       |cmp     esi, 9                           ;  esi 内值与 9 相比较
  00408C7B   |.^ 0F82 74FFFFFF \jb      00408BF5                         ;  小于 9 则继续循环
  00408C81   |.   8B5424 10     mov     edx, dword ptr [esp+10]           ;  真正注册码
  00408C85   |.   68 689A4600   push    00469A68                         ;  12345678
  00408C8A   |.   52            push    edx                              ;  真正注册码入栈
  00408C8B   |.   E8 EF350100   call    0041C27F                         ;  真假码相比较
  00408C90   |.   83C4 08       add     esp, 8
  00408C93   |.   8D4C24 10     lea     ecx, dword ptr [esp+10]
  00408C97   |.   85C0          test    eax, eax
  00408C99   |.   0F94C0        sete    al                               ;  为真则置 al=1.否则为 0
  00408C9C   |.   25 FF000000   and     eax, 0FF
  00408CA1   |.   C74424 20 FFF>mov     dword ptr [esp+20], -1
  00408CA9   |.   8BF0          mov     esi, eax
  00408CAB   |.   E8 33620200   call    0042EEE3
  00408CB0   |.   8B4C24 18     mov     ecx, dword ptr [esp+18]
  00408CB4   |.   8BC6          mov     eax, esi
  00408CB6   |.   5E            pop     esi
  00408CB7   |.   5D            pop     ebp
  00408CB8   |.   5B            pop     ebx
  00408CB9   |.   5F            pop     edi
  00408CBA   |.   64:890D 00000>mov     dword ptr fs:[0], ecx
  00408CC1   |.   83C4 14       add     esp, 14
```

## 四、算法总结

经过上面分析，首先用户名要不少于 8 位。其次注册码由四部分构成，三个 `-` 连接而成，第一部分是固定字符 `6769-`，后面三部分取自固定字符串 `0123456789`。

程序利用了一个 9 次循环来取字符，取法由 `00408C47` 处计算所得余数找出对应字符，如整数 3 对应字符 `3`，依次连接到 `6769-` 后面。有所变化的就是注意后面两个连接符 `-` 位置分别在控制循环次数的 `ESI` 值能整除 3 时添加。即 `ESI` 为 3 和 6 时。从第二部分开始起（0 起始）的第 4 和第 7 位是连接符 `-`。

## 五、C 语言+内嵌汇编注册机代码

算法有些烦，抠出汇编代码，简单修改简洁些，内嵌在里面。注册机代码写得丑陋，但能用。兄弟们见谅。

代码：



```c
#include <stdio.h>
#include <string.h>

main()
{
  int len, nm, a, code;
  char reg[15];
  char name[20], firs[10] = "6769-";
  char rc1[20], rc2[20], rc3[20];
  char gd[11] = "0123456789";
  scanf("%s", name);
  len = strlen(name);
  if (len < 8)
    printf("%s\n", "用户名不能少于 8 位");
  else
  {
    for (a = 0; a < 9; a++)
    {
      __asm
      {
        mov esi, a
        mov ecx, 0Ah
        mov eax, esi
        xor edx, edx
        div ecx
        mov ecx, len
        mov eax, esi
        movsx ebx, byte ptr [edx + gd]
        xor edx, edx
        div ecx
        movsx eax, byte ptr [edx + name]
        lea edx, dword ptr [eax + eax * 2]
        lea eax, dword ptr [eax + edx * 4]
        mov edx, esi
        imul edx, esi
        imul edx, esi
        imul edx, ecx
        add ebx, eax
        or ecx, 0FFFFFFFFh
        xor eax, eax
        add ebx, edx
        mov ecx, 0Ah
        mov eax, ebx
        xor edx, edx
        div ecx
        mov nm, edx
      }
      code = nm + 0x30;
      reg[a] = code;
    }
    strncpy(rc1, reg + 0, 4);
    rc1[4] = 0x2D;
    rc1[5] = '\0';
    strncpy(rc2, reg + 4, 3);
    rc2[3] = 0x2D;
    rc2[4] = '\0';
    strncpy(rc3, reg + 7, 2);
    rc3[2] = '\0';
    strcat(rc1, rc2);
    strcat(rc1, rc3);
    strcat(firs, rc1);
    printf("%s", firs);
  }
}
```
