
### 反汇编代码

```
00566CEB   .  52                  push    edx                             ; |hInst => 00F60000  
00566CEC   .  FF15 7C976A00 call    dword ptr [<&user32.LoadStringA>>; \\LoadStringA  
00566CF2   .  6A 40               push    40                              ; /Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL  
00566CF4   .  8D4424 34          lea     eax, dword ptr [esp+34]           ; |  
00566CF8   .  68 6CEA7200        push    0072EA6C                          ; |Title = ""  
00566CFD   .  50                 push    eax                               ; |Text  
00566CFE   .  56                 push    esi                               ; |hOwner  
00566CFF   .  FF15 0C976A00 call    dword ptr [<&user32.MessageBoxA>>; \\MessageBoxA  
00566D05   .  6A 00              push    0                                 ; /Result = 0  
00566D07   .  56                 push    esi                               ; |hWnd  
00566D08   .  FF15 F0966A00 call    dword ptr [<&user32.EndDialog>]        ; \\EndDialog  
00566D0E   .  5F                 pop     edi  
00566D0F   .  B8 01000000        mov     eax, 1  
00566D14   .  5E                 pop     esi  
00566D15   .  81C4 68010000      add     esp, 168  
00566D1B   .  C3                 retn  
00566D1C   >  8B15 086E7300      mov     edx, dword ptr [736E08]           ;  xnviewzh.00F60000  
00566D22   .  8D4C24 30          lea     ecx, dword ptr [esp+30]  
00566D26   .  6A 40              push    40                              ; /Count = 40 (64.)  
00566D28   .  51                 push    ecx                               ; |Buffer  
00566D29   .  68 93130000        push    1393                              ; |RsrcID = STRING "Invalid registration"  
00566D2E   .  52                 push    edx                               ; |hInst => 00F60000  
00566D2F   .  FF15 7C976A00 call    dword ptr [<&user32.LoadStringA>>; \\LoadStringA  
00566D35   .  6A 10              push    10                              ; /Style = MB_OK|MB_ICONHAND|MB_APPLMODAL  
00566D37   .  8D4424 34          lea     eax, dword ptr [esp+34]           ; |  
00566D3B   .  68 6CEA7200        push    0072EA6C                          ; |Title = ""  
00566D40   .  50                 push    eax                               ; |Text  
00566D41   .  56                 push    esi                               ; |hOwner  
00566D42   .  FF15 0C976A00 call    dword ptr [<&user32.MessageBoxA>>; \\MessageBoxA  
00566D48   .  5F                 pop     edi  
00566D49   .  B8 01000000        mov     eax, 1  
00566D4E   .  5E                 pop     esi  
00566D4F   .  81C4 68010000      add     esp, 168  
00566D55   .  C3                 retn  
```

### 二、由 00566BF8 call 005021B0

关键 `CALL`，跟进来到：（这里是用户名计算部份）

```
005021B0  /$  8B5424 04         mov     edx, dword ptr [esp+4]            ;  //用户名送给 EDX  
005021B4  |.  53                 push    ebx  
005021B5  |.  55                 push    ebp  
005021B6  |.  56                 push    esi  
005021B7  |.  57                 push    edi  
005021B8  |.  8BFA               mov     edi, edx                         ;  //用户名送给 EDI  
005021BA  |.  83C9 FF            or      ecx, FFFFFFFFF                   ;  //ECX=FFFFFFFF  
005021BD  |.  33C0               xor     eax, eax                         ;  //EAX 清零（用于搜索用户名中是否有 0）  
005021BF  |.  F2:AE              repne   scas byte ptr es:[edi]           ;  //循环搜索字符  
005021C1  |.  F7D1               not     ecx                              ;  //ECX 取反，得出（用户名长度+1）的值  
005021C3  |.  49                 dec     ecx                              ;  //ECX-1，得到用户名长度  
005021C4  |.  BE 188A7000        mov     esi, 00708A18                    ;  //ESI=00708A18  
005021C9  |.  8BE9               mov     ebp, ecx                         ;  //用户名长度送给 EBP  
005021CB  |.  B9 05000000        mov     ecx, 5                           ;  //ECX=5  
005021D0  |.  BF 90087300        mov     edi, 00730890                    ;  //EDI=00730890  
005021D5  |.  F3:A5              rep     movs dword ptr es:[edi], dword p>;  //00708A18 中的内容（密码表）送给 00730890 地址中  
005021D7  |.  8BF0               mov     esi, eax                         ;  //ESI=EAX=0  
005021D9  |.  74 21              je      short 005021FC  
005021DB  |>  8A0C16             /mov    cl, byte ptr [esi+edx]           ;  //依次送用户名的 ASCII 值给 CL  
005021DE  |.  8AD9               |mov    bl, cl                           ;  //ASCII 值送给 BL  
005021E0  |.  3298 90087300      |xor    bl, byte ptr [eax+730890]        ;  //ASCII 值与相应位数上的固定值异或运算  
005021E6  |.  40                 |inc     eax                              ;  //EAX+1，用于比较是否到第五位  
005021E7  |.  83F8 05            |cmp     eax, 5                           ;  //EAX 与 5 比较，（看来第 5 位是特殊）  
005021EA  |.  881C16             |mov    byte ptr [esi+edx], bl           ;  //异或值替换 ASCII 值  
005021ED  |.  8888 8F087300      |mov    byte ptr [eax+73088F], cl        ;  //ASCII 值替换固定值，这两行就是字符交换  
005021F3  |.  75 02              |jnz    short 005021F7                   ;  //不等则跳，  
005021F5  |.  33C0               |xor    eax, eax                         ;  //若 EAX=5 即第五位时，EAX 清零，进行第二轮交换  
005021F7  |>  46                 |inc     esi                              ;  //标志位+1，用于取下一位  
005021F8  |.  3BF5               |cmp     esi, ebp                         ;  //标志位与长度比较，看是否循环完  
005021FA  |.^ 72 DF             \jb     short 005021DB                   ;  //小于则跳，未完继续（循环 1）  
005021FC  |>  33FF               xor     edi, edi                         ;  //EDI 清零  
005021FE  |.  33C9               xor     ecx, ecx                         ;  //ECX 清零  
00502200  |.  85ED               test    ebp, ebp                         ;  //较验用户名长度是否为空  
00502202  |.  76 26              jbe     short 0050222A                   ;  //为空则跳（标志小于等于 0 则跳）  
00502204  |>  8A9F 95087300      /mov    bl, byte ptr [edi+730895]        ;  //依次取固定值给 BL（从第六位固定值开始取）  
0050220A  |.  8BF5               |mov    esi, ebp                         ;  //用户名长度送给 ESI  
0050220C  |.  2BF1               |sub    esi, ecx                         ;  //长度-ECX（ECX 初值为 0）  
0050220E  |.  4E                 |dec     esi                              ;  //再-1，这三行代码用来从倒数第一位依次往前取值  
0050220F  |.  8A0416             |mov    al, byte ptr [esi+edx]           ;  //第一次循环交换后的用户名倒数第一位开始依次送给 AL，  
00502212  |.  32D8               |xor    bl, al                           ;  //与固定值异或运算  
00502214  |.  47                 |inc     edi                              ;  //EDI+1，比较是否到了第五位  
00502215  |.  881C16             |mov    byte ptr [esi+edx], bl  
00502218  |.  8887 94087300      |mov    byte ptr [edi+730894], al        ;  //这两行也是字符交换  
0050221E  |.  83FF 05            |cmp    edi, 5                           ;  //EDI 与 5 比较，看是否为第 5 位  
00502221  |.  75 02              |jnz    short 00502225                   ;  //不是则跳  
00502223  |.  33FF               |xor    edi, edi                         ;  //是第 5 位则 EDI 清零，进行第二轮交换  
00502225  |>  41                 |inc    ecx                              ;  //标志位加 1，  
00502226  |.  3BCD               |cmp     ecx, ebp                         ;  //标志位与长度比较，看是否取完  
00502228  |.^ 72 DA             \jb     short 00502204                   ;  //未完继续（循环 2）  
0050222A  |>  33F6               xor     esi, esi                         ;  //ESI 清零  
0050222C  |.  33FF               xor     edi, edi                         ;  //EDI 清零  
0050222E  |.  85ED               test    ebp, ebp                         ;  //用户名是否为空  
00502230  |.  76 21              jbe     short 00502253                   ;  //为空则跳  
00502232  |>  8A0417             /mov    al, byte ptr [edi+edx]           ;  //循环二交换后的用户名 ASCII 值依次送给 AL  
00502235  |.  8A8E 9A087300      |mov    cl, byte ptr [esi+73089A]        ;  //依次取固定值送给 CL，第 11 位固定值开始取值  
0050223B  |.  32C8               |xor    cl, al                           ;  //固定值与 ASCII 值异或运算  
0050223D  |.  46                 |inc     esi                              ;  //ESI+1，用来比较是否到了第 5 位，（第二轮较换）  
0050223E  |.  880C17             |mov    byte ptr [edi+edx], cl  
00502241  |.  8886 99087300      |mov    byte ptr [esi+730899], al        ;  //这两行代码交换值  
00502247  |.  83FE 05            |cmp    esi, 5                           ;  //ESI 与 5 比较  
0050224A  |.  75 02              |jnz    short 0050224E                   ;  //不等则跳，  
```
