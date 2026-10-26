
```
004B9A08  |>  46            |inc     esi

004B9A09  |.  E9 86000000   |jmp     004B9A94

004B9A0E  |>  E8 61FEFFFF   |call    004B9874                        ;  查表函数，放到后面看。a=fun（）

004B9A13  |.  8945 F0       |mov     [local.4], eax

004B9A16  |.  46            |inc     esi

004B9A17  |.  8A06          |mov     al, byte ptr [esi]           ；从psw_1中再取下一个字符，命名为b

004B9A19  |.  E8 56FEFFFF   |call    004B9874                  ； b=fun()

004B9A1E  |.  8BD8          |mov     ebx, eax

004B9A20  |.  46            |inc     esi

004B9A21  |.  8B45 F0       |mov     eax, [local.4]

004B9A24  |.  C1E0 04       |shl     eax, 4                              ;a=a<<4

004B9A27  |.  03D8          |add     ebx, eax                             ;b=b+a;

004B9A29  |.  8B45 E4       |mov     eax, [local.7]

004B9A2C  |.  8B55 EC       |mov     edx, [local.5]                  ;  local5：局部变量。int值，用于从字符串constr中取字符

004B9A2F  |.  0FB64410 FF   |movzx   eax, byte ptr [eax+edx-1]         a= constr[local.5]

004B9A34  |.  33D8          |xor     ebx, eax                              ；b=b^a;

004B9A36  |.  FF45 EC       |inc     [local.5]

004B9A39  |.  8B45 EC       |mov     eax, [local.5]

004B9A3C  |.  3B45 E8       |cmp     eax, [local.6]

004B9A3F  |.  7E 07         |jle     short 004B9A48

004B9A41  |.  C745 EC 01000>|mov     [local.5], 1

004B9A48  |>  8D45 D8       |lea     eax, [local.10]

004B9A4B  |.  8BD3          |mov     edx, ebx

004B9A4D  |.  E8 2AACF4FF   |call    0040467C                        ;  strFormChar(string,char)

004B9A52  |.  8B55 D8       |mov     edx, [local.10]

004B9A55  |.  8D45 E0       |lea     eax, [local.8]

004B9A58  |.  E8 FFACF4FF   |call    0040475C                        ;  StrCat

004B9A5D  |.  BB 64000000   |mov     ebx, 64                         ;  (initial cpu selection)  从该行到 004B9A92不知道有什么用

004B9A62  |>  8D45 DC       |/lea     eax, [local.9]

004B9A65  |.  8B55 E0       ||mov     edx, [local.8]

004B9A68  |.  E8 C7AAF4FF   ||call    00404534                       ;  \* Reference to: System.@LStrLAsg(void;void;void;void);

004B9A6D  |.  B8 0A000000   ||mov     eax, 0A

004B9A72  |.  E8 5995F4FF   ||call    00402FD0                       ;  生成一个随机数，0-10？

004B9A77  |.  8B55 FC       ||mov     edx, [local.1]

004B9A7A  |.  8A5402 FF     ||mov     dl, byte ptr [edx+eax-1]       ;  用户名[k-1]

004B9A7E  |.  8D45 D4       ||lea     eax, [local.11]

004B9A81  |.  E8 F6ABF4FF   ||call    0040467C                       ;  strFormChar(string,char)

004B9A86  |.  8B55 D4       ||mov     edx, [local.11]

004B9A89  |.  8B45 DC       ||mov     eax, [local.9]

004B9A8C  |.  E8 07AEF4FF   ||call    00404898                       ;  \* Reference to: System.@LStrCmp;

004B9A91  |.  4B            ||dec     ebx

004B9A92  |.^ 75 CE         |\jnz     short 004B9A62

004B9A94  |>  836D F4 02    |sub     [local.3], 2                 ；

004B9A98  |.  8B45 E0       |mov     eax, [local.8]

004B9A9B  |.  8A4438 FF     |mov     al, byte ptr [eax+edi-1]

004B9A9F  |.  8B55 FC       |mov     edx, [local.1]

004B9AA2  |.  3A443A FF     |cmp     al, byte ptr [edx+edi-1]        ;  依次比较上面生成的b与用户名中对应位置的字符。

004B9AA6     /75 04         jnz     short 004B9AAC

004B9AA8  |. |33DB          |xor     ebx, ebx

004B9AAA  |. |EB 05         |jmp     short 004B9AB1

004B9AAC  |> \83CB FF       |or      ebx, FFFFFFFF

004B9AAF  |.  EB 24         |jmp     short 004B9AD5

004B9AB1  |>  47            |inc     edi

004B9AB2  |.  837D F4 00    |cmp     [local.3], 0

004B9AB6  |.^ 0F8F 39FFFFFF \jg      004B99F5                        ；没有处理完注册码，循环
```
再看看`fun()`
```
004B9874  /$  55            push    ebp

004B9875  |.  8BEC          mov     ebp, esp

004B9877  |.  6A 00         push    0

004B9879  |.  53            push    ebx

004B987A  |.  8BD8          mov     ebx, eax

004B987C  |.  33C0          xor     eax, eax

004B987E  |.  55            push    ebp

004B987F  |.  68 39994B00   push    004B9939

004B9884  |.  64:FF30       push    dword ptr fs:[eax]

004B9887  |.  64:8920       mov     dword ptr fs:[eax], esp

004B988A  |.  33C0          xor     eax, eax

004B988C  |.  8AC3          mov     al, bl

004B988E  |.  83C0 D0       add     eax, -30                         ;  Switch (cases 30..46)

004B9891  |.  83F8 16       cmp     eax, 16

004B9894  |.  0F87 84000000 ja      004B991E

004B989A  |.  8A80 A7984B00 mov     al, byte ptr [eax+4B98A7]

004B98A0  |.  FF2485 BE984B>jmp     dword ptr [eax\*4+4B98BE]

004B98A7  |.  01            db      01                               ;  分支 004B98BE 索引表

004B98A8  |.  01            db      01

004B98A9  |.  01            db      01

004B98AA  |.  01            db      01

004B98AB  |.  01            db      01

004B98AC  |.  01            db      01

004B98AD  |.  01            db      01

004B98AE  |.  01            db      01

004B98AF  |.  01            db      01

004B98B0  |.  01            db      01

004B98B1  |.  00            db      00

004B98B2  |.  00            db      00

004B98B3  |.  00            db      00

004B98B4  |.  00            db      00

004B98B5  |.  00            db      00

004B98B6  |.  00            db      00

004B98B7  |.  00            db      00

004B98B8  |.  02            db      02

004B98B9  |.  03            db      03

004B98BA  |.  04            db      04

004B98BB  |.  05            db      05

004B98BC  |.  06            db      06

004B98BD  |.  07            db      07

004B98BE  |.  1E994B00      dd      Internet.004B991E                ;  分支表 被用于 004B98A0

004B98C2  |.  DE984B00      dd      Internet.004B98DE

004B98C6  |.  F4984B00      dd      Internet.004B98F4

004B98CA  |.  FB984B00      dd      Internet.004B98FB

004B98CE  |.  02994B00      dd      Internet.004B9902

004B98D2  |.  09994B00      dd      Internet.004B9909

004B98D6  |.  10994B00      dd      Internet.004B9910

004B98DA  |.  17994B00      dd      Internet.004B9917

004B98DE  |>  8D45 FC       lea     eax, [local.1]                   ;  Cases 30 ('0'),31 ('1'),32 ('2'),33 ('3'),34 ('4'),35 ('5'),36 ('6'),37 ('7'),38 ('8'),39 ('9') of switch 004B988E

004B98E1  |.  8BD3          mov     edx, ebx

004B98E3  |.  E8 94ADF4FF   call    0040467C                         ;  System.@LStrFromChar(String;String;Char);

004B98E8  |.  8B45 FC       mov     eax, [local.1]

004B98EB  |.  E8 C0F3F4FF   call    00408CB0                         ;  SysUtils.StrToInt(AnsiString):Integer;

004B98F0  |.  8BD8          mov     ebx, eax

004B98F2  |.  EB 2F         jmp     short 004B9923

004B98F4  |>  BB 0A000000   mov     ebx, 0A                          ;  Case 41 ('A') of switch 004B988E

004B98F9  |.  EB 28         jmp     short 004B9923

004B98FB  |>  BB 0B000000   mov     ebx, 0B                          ;  Case 42 ('B') of switch 004B988E

004B9900  |.  EB 21         jmp     short 004B9923

004B9902  |>  BB 0C000000   mov     ebx, 0C                          ;  Case 43 ('C') of switch 004B988E

004B9907  |.  EB 1A         jmp     short 004B9923

004B9909  |>  BB 0D000000   mov     ebx, 0D                          ;  Case 44 ('D') of switch 004B988E

004B990E  |.  EB 13         jmp     short 004B9923

004B9910  |>  BB 0E000000   mov     ebx, 0E                          ;  Case 45 ('E') of switch 004B988E

004B9915  |.  EB 0C         jmp     short 004B9923

004B9917  |>  BB 0F000000   mov     ebx, 0F                          ;  Case 46 ('F') of switch 004B988E

004B991C  |.  EB 05         jmp     short 004B9923

004B991E  |>  BB 07000000   mov     ebx, 7                           ;  Default case of switch 004B988E

004B9923  |>  33C0          xor     eax, eax

004B9925  |.  5A            pop     edx

004B9926  |.  59            pop     ecx

004B9927  |.  59            pop     ecx

004B9928  |.  64:8910       mov     dword ptr fs:[eax], edx

004B992B  |.  68 40994B00   push    004B9940

004B9930  |>  8D45 FC       lea     eax, [local.1]
```
