
```asm
0048E0D2  mov     dword ptr [esp+1C], 1

0048E0DA  lea     esi, dword ptr [esp+48]

0048E0DE  mov     eax, ebx

0048E0E0  /mov     dl, byte ptr [eax]

0048E0E2  |mov     bl, byte ptr [esi]

0048E0E4  |mov     cl, dl

0048E0E6  |cmp     dl, bl

0048E0E8  |jnz     short 0048E108

0048E0EA  |test    cl, cl

0048E0EC  |je      short 0048E104

0048E0EE  |mov     dl, byte ptr [eax+1]

0048E0F1  |mov     bl, byte ptr [esi+1]

0048E0F4  |mov     cl, dl

0048E0F6  |cmp     dl, bl

0048E0F8  |jnz     short 0048E108

0048E0FA  |add     eax, 2

0048E0FD  |add     esi, 2

0048E100  |test    cl, cl

0048E102  \jnz     short 0048E0E0

0048E104  xor     eax, eax

0048E106  jmp     short 0048E10D

0048E108  sbb     eax, eax

0048E10A  sbb     eax, -1

0048E10D  test    eax, eax

0048E10F  jnz     0048E395                                        ; 若此时变换后的RegCode不等于变换后的SerialNo则报错

0048E115  mov     eax, dword ptr [esp+1C]

0048E119  test    eax, eax

0048E11B  je      0048E395                                        ; 若此时SerialNo中的字母个数为0则报错

0048E121  mov     eax, dword ptr [esp+14]

0048E125  cmp     byte ptr [eax], 0                                ; 注册名为空？

0048E128  jnz     0048E1B4

; 48E12E到48E1B3提醒您输入注册名后退出

; ... ...

0048E1B4  lea     edx, dword ptr [ebp+178]

0048E1BA  push    0

0048E1BC  push    edx

0048E1BD  mov     ecx, ebp

0048E1BF  mov     byte ptr [ebp+39C], 1

0048E1C6  call    005608A0

0048E1CB  lea     eax, dword ptr [ebp+1B0]

0048E1D1  lea     ecx, dword ptr [esp+3C]

0048E1D5  push    eax

0048E1D6  call    <jmp.&MFC42.#537\_CString::CString>

0048E1DB  mov     esi, eax

0048E1DD  lea     eax, dword ptr [ebp+17C]

0048E1E3  lea     ecx, dword ptr [esp+38]

0048E1E7  push    eax

0048E1E8  mov     byte ptr [esp+30C], 13

0048E1F0  call    <jmp.&MFC42.#537\_CString::CString>

0048E1F5  mov     edi, eax

0048E1F7  push    005CCC9C                                        ; Serial Number:

0048E1FC  lea     eax, dword ptr [esp+14]

0048E200  push    006A5894                                        ; 提醒您保存注册信息的CString对象

0048E205  push    eax

0048E206  mov     byte ptr [esp+314], 14

0048E20E  call    <jmp.&MFC42.#924\_operator+>

0048E213  push    edi

0048E214  lea     ecx, dword ptr [esp+20]

0048E218  push    eax

0048E219  push    ecx

0048E21A  mov     byte ptr [esp+314], 15

0048E222  call    <jmp.&MFC42.#922\_operator+>

0048E227  push    005CCC88                                        ;  ASCII 0A,"Registration:  "

0048E22C  lea     edx, dword ptr [esp+30]

0048E230  push    eax

0048E231  push    edx

0048E232  mov     byte ptr [esp+314], 16

0048E23A  call    <jmp.&MFC42.#924\_operator+>

0048E23F  push    esi

0048E240  push    eax

0048E241  lea     eax, dword ptr [esp+30]

0048E245  mov     byte ptr [esp+310], 17

0048E24D  push    eax

0048E24E  call    <jmp.&MFC42.#922\_operator+>

0048E253  lea     ecx, dword ptr [esp+2C]

0048E257  mov     byte ptr [esp+308], 1D

0048E25F  call    <jmp.&MFC42.#800\_CString::~CString>

0048E264  lea     ecx, dword ptr [esp+1C]

0048E268  mov     byte ptr [esp+308], 1C

0048E270  call    <jmp.&MFC42.#800\_CString::~CString>

0048E275  lea     ecx, dword ptr [esp+10]

0048E279  mov     byte ptr [esp+308], 1B

0048E281  call    <jmp.&MFC42.#800\_CString::~CString>

0048E286  lea     ecx, dword ptr [esp+38]

0048E28A  mov     byte ptr [esp+308], 1A

0048E292  call    <jmp.&MFC42.#800\_CString::~CString>

0048E297  lea     ecx, dword ptr [esp+3C]

0048E29B  mov     byte ptr [esp+308], 19

0048E2A3  call    <jmp.&MFC42.#800\_CString::~CString>

0048E2A8  mov     ecx, dword ptr [esp+28]

0048E2AC  push    0

0048E2AE  push    0

0048E2B0  push    ecx

0048E2B1  call    <jmp.&MFC42.#1200\_AfxMessageBox>

0048E2B6  mov     edx, dword ptr [esp+310]

0048E2BD  push    0

0048E2BF  push    edx

0048E2C0  mov     ecx, ebp

0048E2C2  call    0048E8B0

0048E2C7  mov     esi, 1

0048E2CC  mov     byte ptr [esp+308], 4

0048E2D4  mov     dword ptr [esp+10], esi

0048E2D8  lea     ecx, dword ptr [esp+28]

0048E2DC  call    <jmp.&MFC42.#800\_CString::~CString>

0048E2E1  mov     dword ptr [6A5850], esi

0048E2E7  mov     al, byte ptr [6A5928]

0048E2EC  test    al, al

0048E2EE  je      0048E5FD                                        ; [6A5928]=0则正常退出

0048E2F4  mov     eax, dword ptr [esp+14]

0048E2F8  cmp     byte ptr [eax], 0

0048E2FB  jnz     0048E54D

0048E301  mov     al, byte ptr [ebp+39C]

0048E307  test    al, al

0048E309  je      0048E54D

; 48E30F到48E394提醒输入注册名后退出

; ... ...

; 48E395到48E470、48E471到48E54C提醒您注册信息无效后退出

; ... ...

0048E54D  cmp     byte ptr ss:[ebp+39C],1

0048E554  je      SITESPIN.0048E5FD

0048E55A  mov     eax,dword ptr ss:[esp+30]

0048E55E  test    eax,eax

0048E560  jnz     SITESPIN.0048E5FD

; 48E566到48E5FC提醒您试用期已过后退出

; ... ...

0048E5FD  mov     eax, dword ptr [esp+10]

0048E601  xor     edx, edx

0048E603  test    eax, eax

0048E605  setge   dl

0048E608  mov     esi, edx

0048E60A  lea     ecx, dword ptr [esp+18]

0048E60E  mov     byte ptr [esp+308], 3

0048E616  call    <jmp.&MFC42.#800\_CString::~CString>

0048E61B  lea     ecx, dword ptr [esp+9C]

0048E622  mov     byte ptr [esp+308], 2

0048E62A  call    00493430

0048E62F  lea     ecx, dword ptr [esp+20]

0048E633  mov     byte ptr [esp+308], 0

0048E63B  call    <jmp.&MFC42.#800\_CString::~CString>

0048E640  lea     ecx, dword ptr [esp+24]

0048E644  mov     dword ptr [esp+308], -1

0048E64F  call    <jmp.&MFC42.#800\_CString::~CString>

0048E654  mov     eax, esi

0048E656  mov     ecx, dword ptr [esp+300]

0048E65D  pop     edi

0048E65E  pop     esi

0048E65F  pop     ebp

0048E660  mov     dword ptr fs:[0], ecx

0048E667  pop     ebx

0048E668  add     esp, 2FC

0048E66E  retn    4

005CB4F8  07 08 02 0F 06 07 09 0C 04 0A 03 03 09 04 05 06 07 02 00 08 06 11 0D 02
```

好了，通过前面的分析我们可以得出以下结论：

1. 有效的注册码不能是`EXTEND`、`RESET`，甚至其第七位开始的部分不能等同于默认序列号的数字部分；
2. 按照代码分析中的变换算法，对`Serial Number`和`Registration Code`分别进行变换的结果应该相同；
3. 在满足条件2的前提下，若`Serial Number`的前几个字符是`VS2`、`SLS2`、`RSS2`、`EDS2`、`RRS2`，则其第5位始的数值应从0到5000000；若`Serial Number`的前几个字符是`IWS`，则其后面的数值应从300000到300350；若`Serial Number`的前几个字符是`BYWSP`，则其后面的数值应从2097到2125。

通过跟踪发现，程序会在注册表的`HKEY_LOCAL_MACHINE\Software\CLASSES`下创建一个名为`Ssiomv2`的主键来记录注册信息。

设计注册机的时候，关键是`Registration Code`的变换算法的理解。我设计的注册机用起来很好。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [Splash.JPG](javascript:void(0);)
  （38.80kb，89次下载）
- [RegErr.JPG](javascript:void(0);)
  （52.66kb，90次下载）
