
# 金秘书家庭理财 1.461 算法分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-62.htm`](https://www.pediy.com/kssd/pediy09/pediy09-62.htm)

> 【文章标题】: 金秘书家庭理财 1.461 算法分析  
> 【文章作者】: `KuNgBiM`  
> 【作者邮箱】: `kungbim@163.com`  
> 【作者主页】: `http://www.crkcn.com`  

> 【详细过程】

> 这个程序是帮朋友搞的，顺便来下算法。

> 试炼信息：

> - 用户名：`KuNgBiM`
> - 地区：`Chengdu`
> - 邮箱：`kungbim@163.com`
> - 注册日期：`20070218`
> - 密码：`99999999999999999999999999999999`

> 程序无壳，`OD` 装载程序，利用字符插件搜索来到：

```
00455860   .  6A FF             push    -1                                          ;  查找后来到这里设断，F9 运行
00455862   .  68 3CA14600       push    0046A13C                                    ;  SE 处理程序安装
00455867   .  64:A1 00000000    mov     eax, dword ptr fs:[0]
0045586D   .  50               push    eax
0045586E   .  64:8925 00000000  mov     dword ptr fs:[0], esp
00455875   .  81EC 0C010000    sub     esp, 10C
0045587B   .  53               push    ebx
0045587C   .  55               push    ebp
0045587D   .  56               push    esi
0045587E   .  8BF1             mov     esi, ecx
00455880   .  57               push    edi
00455881   .  8D4C24 20        lea     ecx, dword ptr [esp+20]
00455885   .  E8 EE8C0000      call    <jmp.&MFC42.#540_CString::CString>
0045588A   .  8D4C24 14        lea     ecx, dword ptr [esp+14]
0045588E   .  C78424 24010000 0>mov     dword ptr [esp+124], 0
00455899   .  E8 DA8C0000      call    <jmp.&MFC42.#540_CString::CString>
0045589E   .  8D4C24 1C        lea     ecx, dword ptr [esp+1C]
004558A2   .  C68424 24010000 0>mov     byte ptr [esp+124], 1
004558AA   .  E8 C98C0000      call    <jmp.&MFC42.#540_CString::CString>
004558AF   .  8D4C24 10        lea     ecx, dword ptr [esp+10]
004558B3   .  C68424 24010000 0>mov     byte ptr [esp+124], 2
004558BB   .  E8 B88C0000      call    <jmp.&MFC42.#540_CString::CString>
004558C0   .  8D4C24 18        lea     ecx, dword ptr [esp+18]
004558C4   .  C68424 24010000 0>mov     byte ptr [esp+124], 3
004558CC   .  E8 A78C0000      call    <jmp.&MFC42.#540_CString::CString>
004558D1   .  8D4424 20        lea     eax, dword ptr [esp+20]
004558D5   .  8BCE             mov     ecx, esi
004558D7   .  50               push    eax
004558D8   .  68 2B040000      push    42B
004558DD   .  C68424 2C010000 0>mov     byte ptr [esp+12C], 4
004558E5   .  E8 388E0000      call    <jmp.&MFC42.#3097_CWnd::GetDlgItemTextA>    ;  获取第一个文本框中字符
004558EA   .  8D4C24 14        lea     ecx, dword ptr [esp+14]                     ;  ASCII "KuNgBiM"
004558EE   .  51               push    ecx
004558EF   .  68 2D040000      push    42D
004558F4   .  8BCE             mov     ecx, esi
004558F6   .  E8 278E0000      call    <jmp.&MFC42.#3097_CWnd::GetDlgItemTextA>    ;  获取第二个文本框中字符
004558FB   .  8D5424 1C        lea     edx, dword ptr [esp+1C]                     ;  ASCII "kungbim@163.com"
004558FF   .  8BCE             mov     ecx, esi
00455901   .  52               push    edx
00455902   .  68 2E040000      push    42E
00455907   .  E8 168E0000      call    <jmp.&MFC42.#3097_CWnd::GetDlgItemTextA>    ;  获取第三个文本框中字符
0045590C   .  8D4424 10        lea     eax, dword ptr [esp+10]                     ;  ASCII "Chengdu"
00455910   .  8BCE             mov     ecx, esi
00455912   .  50               push    eax
00455913   .  68 2F040000      push    42F
00455918   .  E8 058E0000      call    <jmp.&MFC42.#3097_CWnd::GetDlgItemTextA>    ;  获取第四个文本框中字符
0045591D   .  8D4C24 18        lea     ecx, dword ptr [esp+18]                     ;  ASCII "20070218"
00455921   .  51               push    ecx
00455922   .  68 30040000      push    430
00455927   .  8BCE             mov     ecx, esi
00455929   .  E8 F48D0000      call    <jmp.&MFC42.#3097_CWnd::GetDlgItemTextA>    ;  获取第五个文本框中字符
0045592E   .  8B5424 20        mov     edx, dword ptr [esp+20]                     ;  ASCII
"9999999999999999999999999999999"
00455932   .  8B3D 8CB94600     mov     edi, dword ptr [<&MSVCRT._mbscmp>]         ;  msvcrt._mbscmp
00455938   .  68 20154800      push    00481520                                    ;  /s2 = ""
0045593D   .  52               push    edx                                         ;  |s1
0045593E   .  FFD7            call    edi                                           ;  \\检查用户名是否规范
00455940   .  83C4 08         add     esp, 8
00455943   .  85C0            test    eax, eax
00455945   .  74 4C          je      short 00455993                                  ;  跳则 GAME OVER
00455947   .  8B4424 14       mov     eax, dword ptr [esp+14]
0045594B   .  68 20154800      push    00481520
00455950   .  50               push    eax
00455951   .  FFD7            call    edi                                           ;  检查邮箱长度是否有效
00455953   .  83C4 08         add     esp, 8
00455956   .  85C0            test    eax, eax
00455958   .  74 39          je      short 00455993                                  ;  跳则 GAME OVER
0045595A   .  8B4C24 1C       mov     ecx, dword ptr [esp+1C]
0045595E   .  68 20154800      push    00481520
00455963   .  51               push    ecx
00455964   .  FFD7            call    edi                                           ;  检查地区是否规范
00455966   .  83C4 08         add     esp, 8
00455969   .  85C0            test    eax, eax
0045596B   .  74 26          je      short 00455993                                  ;  跳则 GAME OVER
0045596D   .  8B5424 10       mov     edx, dword ptr [esp+10]
00455971   .  68 20154800      push    00481520
00455976   .  52               push    edx
00455977   .  FFD7            call    edi                                           ;  检查注册日期是否规范
00455979   .  83C4 08         add     esp, 8
0045597C   .  85C0            test    eax, eax
0045597E   .  74 13          je      short 00455993                                  ;  跳则 GAME OVER
00455980   .  8B4424 18       mov     eax, dword ptr [esp+18]
00455984   .  68 20154800      push    00481520
00455989   .  50               push    eax
0045598A   .  FFD7            call    edi                                           ;  检查密码是否规范
0045598C   .  83C4 08         add     esp, 8
0045598F   .  85C0            test    eax, eax
00455991   .  75 10          jnz     short 004559A3                                  ;  必须跳，不跳则 GAME OVER
00455993   >  6A 00           push    0
00455995   .  6A 00           push    0
00455997   .  68 CC114800      push    004811CC                                    ;  尊敬的客户!你需要在右边的网站上注册后,
正确的填写用户、邮箱、地区、注册日期和取得的密码就可以注册啦!
0045599C   .  8BCE             mov     ecx, esi
0045599E   .  E8 A78C0000      call    <jmp.&MFC42.#4224_CWnd::MessageBoxA>
004559A3   >  8D4C24 10        lea     ecx, dword ptr [esp+10]
004559A7   .  E8 268C0000      call    <jmp.&MFC42.#6282_CString::TrimLeft>
004559AC   .  8D4C24 10        lea     ecx, dword ptr [esp+10]
004559B0   .  E8 178C0000      call    <jmp.&MFC42.#6283_CString::TrimRight>
004559B5   .  8D4C24 14        lea     ecx, dword ptr [esp+14]
004559B9   .  E8 148C0000      call    <jmp.&MFC42.#6282_CString::TrimLeft>
004559BE   .  8D4C24 14        lea     ecx, dword ptr [esp+14]
```



```
  004559C2   .  E8 058C0000        call    <jmp.&MFC42.#6283_CString::TrimRight>  
  004559C7   .  B9 10000000        mov     ecx, 10  
  004559CC   .  33C0              xor     eax, eax  
  004559CE   .  8D7C24 55         lea     edi, dword ptr [esp+55]  
  004559D2   .  C64424 54 00      mov     byte ptr [esp+54], 0  
  004559D7   .  F3:AB             rep     stos dword ptr es:[edi]  
  004559D9   .  8D4C24 2C         lea     ecx, dword ptr [esp+2C]  
  004559DD   .  8D6E 60           lea     ebp, dword ptr [esi+60]  
  004559E0   .  E8 938B0000       call    <jmp.&MFC42.#540_CString::CString>  
  004559E5   .  8D4C24 24         lea     ecx, dword ptr [esp+24]  
  004559E9   .  C68424 24010000 0>mov     byte ptr [esp+124], 5  
  004559F1   .  E8 828B0000       call    <jmp.&MFC42.#540_CString::CString>            ;  取固定字符串  
  004559F6   .  8D4C24 10         lea     ecx, dword ptr [esp+10]  
  004559FA   .  68 C0114800       push    004811C0                              ;  goodsoft  
  004559FF   .  8D5424 2C         lea     edx, dword ptr [esp+2C]  
  00455A03   .  B3 06             mov     bl, 6  
  00455A05   .  51                push    ecx  
  00455A06   .  52                push    edx  
  00455A07   .  889C24 30010000   mov     byte ptr [esp+130], bl  
  00455A0E   .  E8 258C0000       call    <jmp.&MFC42.#924_operator+>  
  00455A13   .  8D4C24 14         lea     ecx, dword ptr [esp+14]  
  00455A17   .  8D5424 34         lea     edx, dword ptr [esp+34]  
  00455A1B   .  51                push    ecx  
  00455A1C   .  50                push    eax  
  00455A1D   .  52                push    edx  
  00455A1E   .  C68424 30010000 0>mov     byte ptr [esp+130], 7  
  00455A26   .  E8 2B8C0000       call    <jmp.&MFC42.#922_operator+>  
  00455A2B   .  50                push    eax  
  00455A2C   .  8D4C24 30         lea     ecx, dword ptr [esp+30]  
  00455A30   .  C68424 28010000 0>mov     byte ptr [esp+128], 8  
  00455A38   .  E8 C58B0000       call    <jmp.&MFC42.#858_CString::operator=>  
  00455A3D   .  8D4C24 34         lea     ecx, dword ptr [esp+34]  
  00455A41   .  C68424 24010000 0>mov     byte ptr [esp+124], 7  
  00455A49   .  E8 FA8A0000       call    <jmp.&MFC42.#800_CString::~CString>           ;  取注册日期  
  00455A4E   .  8D4C24 28         lea     ecx, dword ptr [esp+28]  
  00455A52   .  889C24 24010000   mov     byte ptr [esp+124], bl  
  00455A59   .  E8 EA8A0000       call    <jmp.&MFC42.#800_CString::~CString>           ;  取邮箱  
  00455A5E   .  8B45 00           mov     eax, dword ptr [ebp]  
  00455A61   .  8BCD             mov     ecx, ebp  
  00455A63   .  FF50 0C          call    dword ptr [eax+C]                           ;  连接字符串  
  00455A66   .  8B4424 2C         mov     eax, dword ptr [esp+2C]                     ;  ASCII  
  "20070218goodsoftkungbim@163.com"  
  00455A6A   .  8B55 00           mov     edx, dword ptr [ebp]  
  00455A6D   .  8B48 F8           mov     ecx, dword ptr [eax-8]  
  00455A70   .  51                push    ecx  
  00455A71   .  50                push    eax  
  00455A72   .  8BCD             mov     ecx, ebp  
  00455A74   .  FF52 04          call    dword ptr [edx+4]  
  00455A77   .  8B45 00           mov     eax, dword ptr [ebp]  
  00455A7A   .  8D4C24 54         lea     ecx, dword ptr [esp+54]  
  00455A7E   .  51                push    ecx  
  00455A7F   .  8BCD             mov     ecx, ebp  
  00455A81   .  FF50 08          call    dword ptr [eax+8]  
  00455A84   .  B9 20000000       mov     ecx, 20  
  00455A89   .  33C0              xor     eax, eax  
  00455A8B   .  8DBC24 99000000   lea     edi, dword ptr [esp+99]  
  00455A92   .  C68424 98000000 0>mov     byte ptr [esp+98], 0  
  00455A9A   .  8D9424 98000000   lea     edx, dword ptr [esp+98]  
  00455AA1   .  F3:AB             rep     stos dword ptr es:[edi]  
  00455AA3   .  52                push    edx  
  00455AA4   .  8D4424 58         lea     eax, dword ptr [esp+58]  
  00455AA8   .  6A 10             push    10  
  00455AAA   .  50                push    eax  
  00455AAB   .  E8 40FDFFFF       call    004557F0                              ;  算法加密 CALL，跟进  
  00455AB0   .  8B5424 24         mov     edx, dword ptr [esp+24]                     ;  假码入栈  
  00455AB4   .  8D8C24 A4000000   lea     ecx, dword ptr [esp+A4]                     ;  真码入栈  
  00455ABB   .  51                push    ecx                                       ; /真码压栈  
  00455ABC   .  52                push    edx                                       ; |假码压栈  
  00455ABD   .  FF15 8CB94600     call    dword ptr [<&MSVCRT._mbscmp>]               ; \\经典比较  
  00455AC3   .  83C4 14           add     esp, 14  
  00455AC6   .  85C0              test    eax, eax                                  ;  比较注册码是否合法  
  00455AC8   .  0F85 F0020000     jnz     00455DBE                              ;  跳则 GAME OVER  
  00455ACE   .  8D4424 20         lea     eax, dword ptr [esp+20]  
  00455AD2   .  8D4C24 30         lea     ecx, dword ptr [esp+30]  
  00455AD6   .  50                push    eax  
  00455AD7   .  68 A4114800       push    004811A4                              ;  update RegSoft set Rname = '  
  00455ADC   .  51                push    ecx  
  00455ADD   .  E8 5C8B0000       call    <jmp.&MFC42.#926_operator+>  
  00455AE2   .  68 98114800       push    00481198                              ;  ', Remail='  
  00455AE7   .  8D5424 3C         lea     edx, dword ptr [esp+3C]  
  00455AEB   .  50                push    eax  
  00455AEC   .  52                push    edx  
  00455AED   .  C68424 30010000 0>mov     byte ptr [esp+130], 9  
  00455AF5   .  E8 3E8B0000       call    <jmp.&MFC42.#924_operator+>  
  00455AFA   .  8D4C24 14         lea     ecx, dword ptr [esp+14]  
  00455AFE   .  8D5424 4C         lea     edx, dword ptr [esp+4C]  
  00455B02   .  51                push    ecx  
  00455B03   .  50                push    eax  
  00455B04   .  52                push    edx  
  00455B05   .  C68424 30010000 0>mov     byte ptr [esp+130], 0A  
  00455B0D   .  E8 448B0000       call    <jmp.&MFC42.#922_operator+>  
  00455B12   .  68 8C114800       push    0048118C                              ;  ', Rdate='  
  00455B17   .  50                push    eax  
  00455B18   .  8D4424 58         lea     eax, dword ptr [esp+58]  
  00455B1C   .  C68424 2C010000 0>mov     byte ptr [esp+12C], 0B  
  00455B24   .  50                push    eax  
  00455B25   .  E8 0E8B0000       call    <jmp.&MFC42.#924_operator+>  
  00455B2A   .  8D4C24 10         lea     ecx, dword ptr [esp+10]  
  00455B2E   .  8D5424 44         lea     edx, dword ptr [esp+44]  
  00455B32   .  51                push    ecx  
  00455B33   .  50                push    eax  
  00455B34   .  52                push    edx  
  00455B35   .  C68424 30010000 0>mov     byte ptr [esp+130], 0C  
  00455B3D   .  E8 148B0000       call    <jmp.&MFC42.#922_operator+>  
  00455B42   .  68 80114800       push    00481180                              ;  ', Rarea='  
  00455B47   .  50                push    eax  
  00455B48   .  8D4424 50         lea     eax, dword ptr [esp+50]  
  00455B4C   .  C68424 2C010000 0>mov     byte ptr [esp+12C], 0D  
  00455B54   .  50                push    eax  
  00455B55   .  E8 DE8A0000       call    <jmp.&MFC42.#924_operator+>  
  00455B5A   .  8D4C24 1C         lea     ecx, dword ptr [esp+1C]  
  00455B5E   .  8D5424 40         lea     edx, dword ptr [esp+40]  
```



```asm
 00455B62   .  51            push    ecx  
 00455B63   .  50            push    eax  
 00455B64   .  52            push    edx  
 00455B65   .  C68424 30010000 0>mov     byte ptr [esp+130], 0E  
 00455B6D   .  E8 E48A0000    call    <jmp.&MFC42.#922\_operator+>  
 00455B72   .  68 74114800    push    00481174                          ;  ', Rpwd='  
 00455B77   .  50            push    eax  
 00455B78   .  8D4424 44      lea     eax, dword ptr [esp+44]  
 00455B7C   .  C68424 2C010000 0>mov     byte ptr [esp+12C], 0F  
 00455B84   .  50            push    eax  
 00455B85   .  E8 AE8A0000    call    <jmp.&MFC42.#924\_operator+>  
 00455B8A   .  8D4C24 18      lea     ecx, dword ptr [esp+18]  
 00455B8E   .  8D5424 34      lea     edx, dword ptr [esp+34]  
 00455B92   .  51            push    ecx  
 00455B93   .  50            push    eax  
 00455B94   .  52            push    edx  
 00455B95   .  C68424 30010000 1>mov     byte ptr [esp+130], 10  
 00455B9D   .  E8 B48A0000    call    <jmp.&MFC42.#922\_operator+>  
 00455BA2   .  68 64114800    push    00481164                          ;  '  where ID =1  
 00455BA7   .  50            push    eax  
 00455BA8   .  8D4424 30      lea     eax, dword ptr [esp+30]  
 00455BAC   .  C68424 2C010000 1>mov     byte ptr [esp+12C], 11  
 00455BB4   .  50            push    eax  
 00455BB5   .  E8 7E8A0000    call    <jmp.&MFC42.#924\_operator+>  
 00455BBA   .  50            push    eax  
 00455BBB   .  8D4C24 28      lea     ecx, dword ptr [esp+28]  
 00455BBF   .  C68424 28010000 1>mov     byte ptr [esp+128], 12  
 00455BC7   .  E8 368A0000    call    <jmp.&MFC42.#858\_CString::operator=>  
 00455BCC   .  8D4C24 28      lea     ecx, dword ptr [esp+28]  
 00455BD0   .  C68424 24010000 1>mov     byte ptr [esp+124], 11  
 00455BD8   .  E8 6B890000    call    <jmp.&MFC42.#800\_CString::~CString>  
 00455BDD   .  8D4C24 34      lea     ecx, dword ptr [esp+34]  
 00455BE1   .  C68424 24010000 1>mov     byte ptr [esp+124], 10  
 00455BE9   .  E8 5A890000    call    <jmp.&MFC42.#800\_CString::~CString>  
 00455BEE   .  8D4C24 3C      lea     ecx, dword ptr [esp+3C]  
 00455BF2   .  C68424 24010000 0>mov     byte ptr [esp+124], 0F  
 00455BFA   .  E8 49890000    call    <jmp.&MFC42.#800\_CString::~CString>  
 00455BFF   .  8D4C24 40      lea     ecx, dword ptr [esp+40]  
 00455C03   .  C68424 24010000 0>mov     byte ptr [esp+124], 0E  
 00455C0B   .  E8 38890000    call    <jmp.&MFC42.#800\_CString::~CString>  
 00455C10   .  8D4C24 48      lea     ecx, dword ptr [esp+48]  
 00455C14   .  C68424 24010000 0>mov     byte ptr [esp+124], 0D  
 00455C1C   .  E8 27890000    call    <jmp.&MFC42.#800\_CString::~CString>  
 00455C21   .  8D4C24 44      lea     ecx, dword ptr [esp+44]  
 00455C25   .  C68424 24010000 0>mov     byte ptr [esp+124], 0C  
 00455C2D   .  E8 16890000    call    <jmp.&MFC42.#800\_CString::~CString>  
 00455C32   .  8D4C24 50      lea     ecx, dword ptr [esp+50]  
 00455C36   .  C68424 24010000 0>mov     byte ptr [esp+124], 0B  
 00455C3E   .  E8 05890000    call    <jmp.&MFC42.#800\_CString::~CString>  
 00455C43   .  8D4C24 4C      lea     ecx, dword ptr [esp+4C]  
 00455C47   .  C68424 24010000 0>mov     byte ptr [esp+124], 0A  
 00455C4F   .  E8 F4880000    call    <jmp.&MFC42.#800\_CString::~CString>  
 00455C54   .  8D4C24 38      lea     ecx, dword ptr [esp+38]  
 00455C58   .  C68424 24010000 0>mov     byte ptr [esp+124], 9  
 00455C60   .  E8 E3880000    call    <jmp.&MFC42.#800\_CString::~CString>  
 00455C65   .  8D4C24 30      lea     ecx, dword ptr [esp+30]  
 00455C69   .  889C24 24010000  mov     byte ptr [esp+124], bl  
 00455C70   .  E8 D3880000    call    <jmp.&MFC42.#800\_CString::~CString>  
 00455C75   .  6A 01          push    1  
 00455C77   .  6A 00          push    0  
 00455C79   .  8D4C24 2C      lea     ecx, dword ptr [esp+2C]  
 00455C7D   .  E8 AEDDFAFF    call    00403A30  
 00455C82   .  51            push    ecx  
 00455C83   .  8BCC           mov     ecx, esp  
 00455C85   .  896424 44      mov     dword ptr [esp+44], esp  
 00455C89   .  50            push    eax  
 00455C8A   .  E8 81DFFAFF    call    00403C10  
 00455C8F   .  8D4C24 3C      lea     ecx, dword ptr [esp+3C]  
 00455C93   .  889C24 30010000  mov     byte ptr [esp+130], bl  
 00455C9A   .  51            push    ecx  
 00455C9B   .  8D8E C0000000  lea     ecx, dword ptr [esi+C0]  
 00455CA1   .  E8 FA39FEFF    call    004396A0  
 00455CA6   .  8BC8           mov     ecx, eax  
 00455CA8   .  E8 43E2FAFF    call    00403EF0  
 00455CAD   .  8B4424 30      mov     eax, dword ptr [esp+30]  
 00455CB1   .  85C0           test    eax, eax  
 00455CB3   .  74 06          je      short 00455CBB  
 00455CB5   .  8B10           mov     edx, dword ptr [eax]  
 00455CB7   .  50            push    eax  
 00455CB8   .  FF52 08        call    dword ptr [edx+8]  
 00455CBB   >  6A 00          push    0  
 00455CBD   .  6A 00          push    0  
 00455CBF   .  68 18114800    push    00481118                          ;  您已经成为我们的正式用户!感谢您的注册,
您将获得免费升级和售后服务!  
 00455CC4   .  8BCE           mov     ecx, esi  
 00455CC6   .  E8 7F890000    call    <jmp.&MFC42.#4224\_CWnd::MessageBoxA>  
 00455CCB   .  68 F0104800    push    004810F0                          ;  您已经成为我们的正式用户!感谢您的注册!  
 00455CD0   .  68 31040000    push    431  
 00455CD5   .  8BCE           mov     ecx, esi  
 00455CD7   .  E8 1A890000    call    <jmp.&MFC42.#3092\_CWnd::GetDlgItem>  
 00455CDC   .  8BC8           mov     ecx, eax  
 00455CDE   .  E8 D7880000    call    <jmp.&MFC42.#6199\_CWnd::SetWindowTextA>  
 00455CE3   .  68 20154800    push    00481520  
 00455CE8   .  68 35040000    push    435  
 00455CED   .  8BCE           mov     ecx, esi  
 00455CEF   .  E8 02890000    call    <jmp.&MFC42.#3092\_CWnd::GetDlgItem>  
 00455CF4   .  8BC8           mov     ecx, eax  
 00455CF6   .  E8 BF880000    call    <jmp.&MFC42.#6199\_CWnd::SetWindowTextA>  
 00455CFB   .  68 20154800    push    00481520  
 00455D00   .  68 36040000    push    436  
 00455D05   .  8BCE           mov     ecx, esi  
 00455D07   .  E8 EA880000    call    <jmp.&MFC42.#3092\_CWnd::GetDlgItem>  
 00455D0C   .  8BC8           mov     ecx, eax  
 00455D0E   .  E8 A7880000    call    <jmp.&MFC42.#6199\_CWnd::SetWindowTextA>  
 00455D13   .  68 20154800    push    00481520  
 00455D18   .  68 37040000    push    437  
 00455D1D   .  8BCE           mov     ecx, esi  
 00455D1F   .  E8 D2880000    call    <jmp.&MFC42.#3092\_CWnd::GetDlgItem>  
 00455D24   .  8BC8           mov     ecx, eax  
 00455D26   .  E8 8F880000    call    <jmp.&MFC42.#6199\_CWnd::SetWindowTextA>  
 00455D2B   .  68 20154800    push    00481520  
 00455D30   .  68 38040000    push    438  
 00455D35   .  8BCE           mov     ecx, esi  
 00455D37   .  E8 BA880000    call    <jmp.&MFC42.#3092\_CWnd::GetDlgItem>  
 00455D3C   .  8BC8           mov     ecx, eax  
 00455D3E   .  E8 77880000    call    <jmp.&MFC42.#6199\_CWnd::SetWindowTextA>  
 00455D43   .  6A 00          push    0  
 00455D45   .  68 2B040000    push    42B  
```



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



`4dd466d0e1e0cc123afbe504da2462ca`
