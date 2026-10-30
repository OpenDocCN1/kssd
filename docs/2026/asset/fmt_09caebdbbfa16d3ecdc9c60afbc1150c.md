
```
004129D1    8D85 CCFDFFFF   lea     eax, dword ptr [ebp-0x234]

004129D7    50              push    eax

004129D8    8B8D C0FDFFFF   mov     ecx, dword ptr [ebp-0x240]

004129DE    51              push    ecx

004129DF    E8 7EE8FFFF     call    <kernel32!Process32FirstW>

004129E4    8985 B4FDFFFF   mov     dword ptr [ebp-0x24C], eax

004129EA    83BD B4FDFFFF 0>/cmp     dword ptr [ebp-0x24C], 0x0

004129F1    0F84 94010000   |je      00412B8B

004129F7    8BF4            |mov     esi, esp

004129F9    8D85 F0FDFFFF   |lea     eax, dword ptr [ebp-0x210]

004129FF    50              |push    eax

00412A00    FF15 A0C14100   |call    dword ptr [<&ucrtbased.wcslen>]                 ; ucrtbase.wcslen

00412A06    83C4 04         |add     esp, 0x4

00412A09    3BF4            |cmp     esi, esp

00412A0B    E8 6CE7FFFF     |call    <CheckEsp>

00412A10    83C0 01         |add     eax, 0x1

00412A13    8BF4            |mov     esi, esp

00412A15    50              |push    eax

00412A16    8D8D F0FDFFFF   |lea     ecx, dword ptr [ebp-0x210]

00412A1C    51              |push    ecx

00412A1D    FF15 9CC14100   |call    dword ptr [<&ucrtbased._wcslwr_s>]              ; ucrtbase._wcslwr_s

00412A23    83C4 08         |add     esp, 0x8

00412A26    3BF4            |cmp     esi, esp

00412A28    E8 4FE7FFFF     |call    <CheckEsp>

00412A2D    8BF4            |mov     esi, esp

00412A2F    68 848B4100     |push    00418B84                                        ; UNICODE "ollyice.exe"

00412A34    8D85 F0FDFFFF   |lea     eax, dword ptr [ebp-0x210]

00412A3A    50              |push    eax

00412A3B    FF15 A4C14100   |call    dword ptr [<&ucrtbased.wcscmp>]                 ; ucrtbase.wcscmp

00412A41    83C4 08         |add     esp, 0x8

00412A44    3BF4            |cmp     esi, esp

00412A46    E8 31E7FFFF     |call    <CheckEsp>

00412A4B    85C0            |test    eax, eax

00412A4D    75 1E           |jnz     short 00412A6D

00412A4F    68 A08B4100     |push    00418BA0                                        ; ///////WARNING///////\n

00412A54    E8 9EE9FFFF     |call    <printf>

00412A59    83C4 04         |add     esp, 0x4

00412A5C    8BF4            |mov     esi, esp

00412A5E    6A 00           |push    0x0

00412A60    FF15 94C14100   |call    dword ptr [<&ucrtbased.exit>]                   ; ucrtbase.exit

00412A66    3BF4            |cmp     esi, esp

00412A68    E8 0FE7FFFF     |call    <CheckEsp>

00412A6D    8BF4            |mov     esi, esp

00412A6F    68 BC8B4100     |push    00418BBC                                        ; UNICODE "ollydbg.exe"

00412A74    8D85 F0FDFFFF   |lea     eax, dword ptr [ebp-0x210]

00412A7A    50              |push    eax

00412A7B    FF15 A4C14100   |call    dword ptr [<&ucrtbased.wcscmp>]                 ; ucrtbase.wcscmp

00412A81    83C4 08         |add     esp, 0x8

00412A84    3BF4            |cmp     esi, esp

00412A86    E8 F1E6FFFF     |call    <CheckEsp>

00412A8B    85C0            |test    eax, eax

00412A8D    75 1E           |jnz     short 00412AAD

00412A8F    68 D88B4100     |push    00418BD8                                        ; ///////\nWARNING\n///////\n

00412A94    E8 5EE9FFFF     |call    <printf>

00412A99    83C4 04         |add     esp, 0x4

00412A9C    8BF4            |mov     esi, esp

00412A9E    6A 00           |push    0x0

00412AA0    FF15 94C14100   |call    dword ptr [<&ucrtbased.exit>]                   ; ucrtbase.exit

00412AA6    3BF4            |cmp     esi, esp

00412AA8    E8 CFE6FFFF     |call    <CheckEsp>

00412AAD    8BF4            |mov     esi, esp

00412AAF    68 F48B4100     |push    00418BF4                                        ; UNICODE "peid.exe"

00412AB4    8D85 F0FDFFFF   |lea     eax, dword ptr [ebp-0x210]

00412ABA    50              |push    eax

00412ABB    FF15 A4C14100   |call    dword ptr [<&ucrtbased.wcscmp>]                 ; ucrtbase.wcscmp

00412AC1    83C4 08         |add     esp, 0x8

00412AC4    3BF4            |cmp     esi, esp

00412AC6    E8 B1E6FFFF     |call    <CheckEsp>

00412ACB    85C0            |test    eax, eax

00412ACD    75 1E           |jnz     short 00412AED

00412ACF    68 D88B4100     |push    00418BD8                                        ; ///////\nWARNING\n///////\n

00412AD4    E8 1EE9FFFF     |call    <printf>

00412AD9    83C4 04         |add     esp, 0x4

00412ADC    8BF4            |mov     esi, esp

00412ADE    6A 00           |push    0x0

00412AE0    FF15 94C14100   |call    dword ptr [<&ucrtbased.exit>]                   ; ucrtbase.exit

00412AE6    3BF4            |cmp     esi, esp

00412AE8    E8 8FE6FFFF     |call    <CheckEsp>

00412AED    8BF4            |mov     esi, esp

00412AEF    68 108C4100     |push    00418C10                                        ; UNICODE "ida.exe"

00412AF4    8D85 F0FDFFFF   |lea     eax, dword ptr [ebp-0x210]

00412AFA    50              |push    eax

00412AFB    FF15 A4C14100   |call    dword ptr [<&ucrtbased.wcscmp>]                 ; ucrtbase.wcscmp

00412B01    83C4 08         |add     esp, 0x8

00412B04    3BF4            |cmp     esi, esp

00412B06    E8 71E6FFFF     |call    <CheckEsp>

00412B0B    85C0            |test    eax, eax

00412B0D    75 1E           |jnz     short 00412B2D

00412B0F    68 D88B4100     |push    00418BD8                                        ; ///////\nWARNING\n///////\n

00412B14    E8 DEE8FFFF     |call    <printf>

00412B19    83C4 04         |add     esp, 0x4

00412B1C    8BF4            |mov     esi, esp

00412B1E    6A 00           |push    0x0

00412B20    FF15 94C14100   |call    dword ptr [<&ucrtbased.exit>]                   ; ucrtbase.exit

00412B26    3BF4            |cmp     esi, esp

00412B28    E8 4FE6FFFF     |call    <CheckEsp>

00412B2D    8BF4            |mov     esi, esp

00412B2F    68 208C4100     |push    00418C20                                        ; UNICODE "idaq.exe"

00412B34    8D85 F0FDFFFF   |lea     eax, dword ptr [ebp-0x210]

00412B3A    50              |push    eax

00412B3B    FF15 A4C14100   |call    dword ptr [<&ucrtbased.wcscmp>]                 ; ucrtbase.wcscmp

00412B41    83C4 08         |add     esp, 0x8

00412B44    3BF4            |cmp     esi, esp

00412B46    E8 31E6FFFF     |call    <CheckEsp>

00412B4B    85C0            |test    eax, eax

00412B4D    75 1E           |jnz     short 00412B6D

00412B4F    68 D88B4100     |push    00418BD8                                        ; ///////\nWARNING\n///////\n

00412B54    E8 9EE8FFFF     |call    <printf>

00412B59    83C4 04         |add     esp, 0x4

00412B5C    8BF4            |mov     esi, esp

00412B5E    6A 00           |push    0x0

00412B60    FF15 94C14100   |call    dword ptr [<&ucrtbased.exit>]                   ; ucrtbase.exit

00412B66    3BF4            |cmp     esi, esp

00412B68    E8 0FE6FFFF     |call    <CheckEsp>

00412B6D    8D85 CCFDFFFF   |lea     eax, dword ptr [ebp-0x234]

00412B73    50              |push    eax

00412B74    8B8D C0FDFFFF   |mov     ecx, dword ptr [ebp-0x240]

00412B7A    51              |push    ecx

00412B7B    E8 A1E6FFFF     |call    00411221

00412B80    8985 B4FDFFFF   |mov     dword ptr [ebp-0x24C], eax

00412B86  ^ E9 5FFEFFFF     \jmp     004129EA
```

## 1x2 分析函数 `00411186`

这段函数其实就是`IsDebuggerPresent()`。检测了`PEB`的`BeingDebugged`，若`BeingDebugged`为0则未被调试，否则被调试。

函数代码如下:

![这里写图片描述](https://bbs.kanxue.com/upload/attach/202002/759802_26CU9UMQQY698HY.jpg)

## 1x3 一次性全部拆解

这两个检测函数都位于`TLS`回调函数内，因此删掉即可。

![这里写图片描述](https://bbs.kanxue.com/upload/attach/202002/759802_P52VQ3JKATSZ7FV.jpg)

这样系统就不会去执行`TLS`回调函数了，而是直接执行入口点处的代码了。

## 2x0 分析第一个输入

`OD`来到`main`函数`00412E20`处，单步调试一阵发现第一个算法函数 `00411316`

因为这个函数的代码很长，所以使用`IDA`来分析。

`IDA`来到`00411316`处，使用`F5`插件查看反编译好后的C代码。

首先将注册码的顺序颠倒(`123456789` -> `987654321`)

![这里写图片描述](https://bbs.kanxue.com/upload/attach/202002/759802_FZQ6FEHMNBTAMY7.jpg)

然后将循环次数作为密匙，加密颠倒后的`Key`

![这里写图片描述](https://bbs.kanxue.com/upload/attach/202002/759802_Q8QEBWU6MQFZHEV.jpg)

最后和一串密文比较，若全部相等则通过第一个`Key`的验证。
