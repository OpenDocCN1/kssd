
```assembly
00421453  |.  8D45 C8       lea     eax, [ebp-38]
00421456  |.  BA 02000000   mov     edx, 2
0042145B  |.  E8 F81B0100   call    00433058
00421460  |.  FF4D BC       dec     dword ptr [ebp-44]
00421463  |.  8D45 CC       lea     eax, [ebp-34]
00421466  |.  BA 02000000   mov     edx, 2
0042146B  |.  E8 E81B0100   call    00433058
00421470  |.  6A 00         push    0
00421472  |.  B9 90854400   mov     ecx, 00448590            ;  注意
00421477  |.  BA 7C854400   mov     edx, 0044857C            ;  注册成功，欢迎使用.
0042147C  |.  A1 303B4500   mov     eax, [<&vcl60.Forms::App>
00421481  |.  8B00          mov     eax, [eax]
00421483  |.  E8 9C1A0100   call    00432F24
00421488  |.  FF4D BC       dec     dword ptr [ebp-44]
0042148B  |.  8D45 F8       lea     eax, [ebp-8]
0042148E  |.  BA 02000000   mov     edx, 2
00421493  |.  E8 C01B0100   call    00433058
00421498  |.  FF4D BC       dec     dword ptr [ebp-44]
0042149B  |.  8D45 FC       lea     eax, [ebp-4]
0042149E  |.  BA 02000000   mov     edx, 2
004214A3  |.  E8 B01B0100   call    00433058
004214A8  |.  8B4D A0       mov     ecx, [ebp-60]
004214AB  |.  64:890D 00000>mov     fs:[0], ecx
004214B2  |>  8BE5          mov     esp, ebp
004214B4  |.  5D            pop     ebp
004214B5  \.  C3            retn
```

### 跟进关键call 4210A4 处代码如下

```assembly
00431038  /$  55            push    ebp                       ;  继续F8↓
00431039  |.  8BEC          mov     ebp, esp
0043103B  |.  81C4 60FFFFFF add     esp, -0A0
00431041  |.  56            push    esi
00431042  |.  57            push    edi
00431043  |.  BE 9CEA4400   mov     esi, 0044EA9C
00431048  |.  8D7D 80       lea     edi, [ebp-80]
0043104B  |.  B9 1E000000   mov     ecx, 1E                   ;  将“1E=30”送入ECX
00431050  |.  F3:A5         rep     movs dword ptr es:[edi], >;  ecx=0000001E (十进制 30.)
00431052  |.  FF75 0C       push    dword ptr [ebp+C]         ; /ASCII "WD-WCAM96574947"
00431055  |.  8D85 60FFFFFF lea     eax, [ebp-A0]             ; |
0043105B  |.  50            push    eax                       ; |dest
0043105C  |.  E8 2D650100   call    <jmp.&CC3260MT.\_strcpy>   ; \\_strcpy
00431061  |.  83C4 08       add     esp, 8
00431064  |.  FF75 0C       push    dword ptr [ebp+C]         ; /S = "WD-WCAM96574947"
00431067  |.  E8 28650100   call    <jmp.&CC3260MT.\_strlen>   ; \\_strlen
0043106C  |.  59            pop     ecx                       ;  弹出生成码
0043106D  |.  8945 FC       mov     [ebp-4], eax              ;  EAX=F 即为15 送入[EBP-4]
00431070  |.  33D2          xor     edx, edx                  ;  EDX清零
00431072  |.  8955 F8       mov     [ebp-8], edx              ;  EDX值送入[EBP-8]
00431075  |.  8B4D F8       mov     ecx, [ebp-8]              ;  [EBP-8]的值则送入ECX
00431078  |.  3B4D FC       cmp     ecx, [ebp-4]              ;  与ECX值作比较，生成码是否大于ECX
0043107B  |.  7D 54         jge     short 004310D1            ;  大于则跳
0043107D  |>  8B45 F8       /mov     eax, [ebp-8]
00431080  |.  8A5485 80     |mov     dl, [ebp+eax\*4-80]       ;  [EBP+EAX\*4-80]的值送入dl
00431084  |.  8B4D F8       |mov     ecx, [ebp-8]
00431087  |.  30940D 60FFFF>|xor     [ebp+ecx-A0], dl         ;  dl xor 生成码（W）的ASCII码第一位的16进制数---依次取生成码的ASCII码来与dl不同的值运算，DL的值依次为“352742123356972”
0043108E  |.  8B45 F8       |mov     eax, [ebp-8]
00431091  |.  0FBE9405 60FF>|movsx   edx, byte ptr [ebp+eax-A>;  xor 得来的值54再转换为ASCII码（T）
00431099  |.  83FA 30       |cmp     edx, 30                  ;  比较EDX的值是否大于30 也就是ASCII码（0）
0043109C  |.  7D 0D         |jge     short 004310AB           ;  大于则跳
0043109E  |.  8B4D F8       |mov     ecx, [ebp-8]
004310A1  |.  C6840D 60FFFF>|mov     byte ptr [ebp+ecx-A0], 3>
004310A9  |.  EB 1B         |jmp     short 004310C6
004310AB  |>  8B45 F8       |mov     eax, [ebp-8]
004310AE  |.  0FBE9405 60FF>|movsx   edx, byte ptr [ebp+eax-A>;  ASCII码 （T）送入EDX
004310B6  |.  83FA 7A       |cmp     edx, 7A                  ;  EDX与7A做比较
004310B9  |.  7E 0B         |jle     short 004310C6           ;  小于则跳
004310BB  |.  8B4D F8       |mov     ecx, [ebp-8]
004310BE  |.  C6840D 60FFFF>|mov     byte ptr [ebp+ecx-A0], 7>
004310C6  |>  FF45 F8       |inc     dword ptr [ebp-8]        ;  +1
004310C9  |.  8B45 F8       |mov     eax, [ebp-8]
004310CC  |.  3B45 FC       |cmp     eax, [ebp-4]
004310CF  |.^ 7C AC         \jl      short 0043107D           ;  没运算完，继续
004310D1  |>  8D95 60FFFFFF lea     edx, [ebp-A0]             ;  [ebp-A0]的值偏移到EDX
004310D7  |.  52            push    edx                       ; /出现真码，压入栈
004310D8  |.  8B4D 08       mov     ecx, [ebp+8]              ; |
004310DB  |.  81C1 84000000 add     ecx, 84                   ; |
004310E1  |.  51            push    ecx                       ; |dest
004310E2  |.  E8 A7640100   call    <jmp.&CC3260MT.\_strcpy>   ; \\_strcpy
004310E7  |.  83C4 08       add     esp, 8
004310EA  |.  5F            pop     edi
004310EB  |.  5E            pop     esi
004310EC  |.  8BE5          mov     esp, ebp
004310EE  |.  5D            pop     ebp
004310EF  \.  C3            retn
```

## 【算法总结】

1、生成码最长不允许超过15位

2、依次取生成码的ASCII码转换为16进制数后，分别与不同的`DL`（`352742123356972`）值`XOR`，得到的值再转换为ASCII码

机器码：`WD-WCAM96574947`

运算符：`352742123356972`

两者之间与对应的转换数运算：

```text
W = 57 XOR 3 = 54 ---> T
D = 44 XOR 5 = 41 ---> A
- = 2D XOR 2 = 2F ---> /  ------------->与30作比较，小于30 直接用30替换！ 所以得 - = 2D XOR 2 = 2F < 30 =0
W = 57 XOR 7 = 50 ---> P
C = 43 XOR 4 = 47 ---> G
A = 41 XOR 2 = 43 ---> C
M = 4D XOR 1 = 4C ---> L
9 = 39 XOR 2 = 3B ---> ;
6 = 36 XOR 3 = 35 ---> 5
5 = 35 XOR 3 = 36 ---> 6
7 = 37 XOR 5 = 32 ---> 2
4 = 34 XOR 6 = 32 ---> 2
9 = 39 XOR 9 = 30 ---> 0
4 = 34 XOR 7 = 33 ---> 3
7 = 37 XOR 2 = 35 ---> 5
```

3、其中如果生成码小于ASCII码0 的话，那就直接用0来作替代（与30作比较，小于30 直接用30（0x30=48=0）替换！）

4、如果生成码中有哪个大雨ASCII码 Z的，那就直接用Z来作替代（与7A作比较，大于7A 直接用7A（0x7A=122=Z）替换！）

5、最后得出的正确注册码为：`TA0PGCL;5622035`

还有测试一台机器的：

生成码: `B1AR3MHE`

注册码：`A4CU7OIG`

```text
B = 42 XOR 3 = 41 ---> A
1 = 31 XOR 5 = 34 ---> 4
A = 41 XOR 2 = 43 ---> C
R = 52 XOR 7 = 55 ---> U
3 = 33 XOR 4 = 37 ---> 7
M = 4D XOR 2 = 4F ---> O
H = 48 XOR 1 = 49 ---> I
E = 45 XOR 2 = 47 ---> G
```

## 【版权声明】

本文纯属技术交流, 转载请注明作者信息并保持文章的完整, 谢谢!

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
