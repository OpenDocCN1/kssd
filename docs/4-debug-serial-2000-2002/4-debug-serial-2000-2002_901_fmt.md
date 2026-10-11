
# yonc v1.12

> 原文：[`www.pediy.com/kssd/pediy04/kanxue046.htm`](https://www.pediy.com/kssd/pediy04/kanxue046.htm)

下载：`http://newhua.ruyi.com/down/yonc112.exe`

这个程序的保护很有意思，作者放了不少陷阱在里面，用 `W32Dasm` 你可以搜索到下面这个字符串

* Possible StringData Ref from Data Obj -> "DQJXN-ASNYZ-GQI-FNB"

如果你用它来注册的话，程序会告诉你这个注册码已经无效。

不同错误的注册码，会有不同的错误提示，比如：这个注册码是旧版本的注册码，是以后版本的注册码，是无效注册码，

是过期注册码，等等。

注册码比较过程中所查的表，也很有意思，我把它列在下面了，有兴趣可以看看

好了，言归正传：

```
\======================================================================================================

\* Reference To: USER32.SendMessageA, Ord:0214h

                      |
:004287D7 FF1560C24500              Call dword ptr [0045C260]
:004287DD 8D8C2498000000           lea ecx, dword ptr [esp+00000098]
:004287E4 E8272A0100               call 0043B210<--------------------读取注册码
:004287E9 85C0                     test eax, eax
:004287EB 0F8455010000             je 00428946
:004287F1 8B542434                 mov edx, dword ptr [esp+34]<------假注册码
:004287F5 8D4C2414                 lea ecx, dword ptr [esp+14]
:004287F9 52                       push edx
:004287FA E801EAFFFF               call 00427200<--------------------(1)核心，跟入
:004287FF 8D4C2414                 lea ecx, dword ptr [esp+14]
:00428803 C684245405000005         mov byte ptr [esp+00000554], 05
:0042880B E8A0EAFFFF               call 004272B0<--------------------al<-[ecx+08]
:00428810 25FF000000               and eax, 000000FF
:00428815 8D4C2414                 lea ecx, dword ptr [esp+14]
:00428819 8BD8                     mov ebx, eax
:0042881B E880EAFFFF               call 004272A0<--------------------eax<-[ecx+0c]
:00428820 85DB                     test ebx, ebx
:00428822 8BE8                     mov ebp, eax
:00428824 7431                     je 00428857
:00428826 8D4C2414                 lea ecx, dword ptr [esp+14]
:0042882A E841EAFFFF               call 00427270<-------------------(2)判断,跟入
:0042882F 8B4E68                   mov ecx, dword ptr [esi+68]
:00428832 25FF000000               and eax, 000000FF
:00428837 3BC8                     cmp ecx, eax
:00428839 7E07                     jle 00428842
:0042883B BD04000000               mov ebp, 00000004
:00428840 EB19                     jmp 0042885B

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00428839(C)
|
:00428842 8D4C2414                 lea ecx, dword ptr [esp+14]
:00428846 E805EAFFFF               call 00427250<------------------(3)判断,跟入
:0042884B 384614                   cmp byte ptr [esi+14], al
:0042884E 7407                     je 00428857
:00428850 BD05000000               mov ebp, 00000005
:00428855 EB04                     jmp 0042885B

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00428824(C), :0042884E(C)
|
:00428857 85ED                     test ebp, ebp
:00428859 743A                     je 00428895<----------------------跳吧，不要怕,
                                   不跳就没机会了

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00428840(U), :00428855(U)
|
:0042885B 8D842484000000           lea eax, dword ptr [esp+00000084]
:00428862 55                       push ebp
:00428863 50                       push eax
:00428864 E857010000               call 004289C0
:00428869 8B00                     mov eax, dword ptr [eax]
:0042886B 8B7618                   mov esi, dword ptr [esi+18]
:0042886E 50                       push eax
:0042886F 56                       push esi
:00428870 57                       push edi
:00428871 C684246805000009         mov byte ptr [esp+00000568], 09
:00428879 E8A2710100               call 0043FA20<---------------------出错信息,到这里就玩完了

\===================================================================================

\* Referenced by a CALL at Addresses:<--------------------------- call (1)
|:004287FA  , :00428FA7
|
:00427200 8B542404                 mov edx, dword ptr [esp+04]<-------假注册码
:00427204 56                       push esi
:00427205 8BF1                     mov esi, ecx
:00427207 33C0                     xor eax, eax
:00427209 52                       push edx
:0042720A 884608                   mov byte ptr [esi+08], al
:0042720D 89460C                   mov dword ptr [esi+0C], eax
:00427210 8901                     mov dword ptr [ecx], eax
:00427212 894104                   mov dword ptr [ecx+04], eax
:00427215 E806070000               call 00427920<----------------(1.1)
:0042721A 8BC6                     mov eax, esi
:0042721C 5E                       pop esi
:0042721D C20400                   ret 0004

\==================================================================================

\* Referenced by a CALL at Addresses:<--------------------------- call (1.1)
|:00427215  , :00427849
|
:00427920 6AFF                     push FFFFFFFF
:00427922 68E8884500               push 004588E8
:00427927 64A100000000             mov eax, dword ptr fs:[00000000]
:0042792D 50                       push eax
:0042792E 64892500000000           mov dword ptr fs:[00000000], esp
:00427935 83EC2C                   sub esp, 0000002C
:00427938 53                       push ebx
:00427939 55                       push ebp
:0042793A 56                       push esi
:0042793B 33DB                     xor ebx, ebx
:0042793D 57                       push edi
:0042793E 8BF1                     mov esi, ecx
:00427940 53                       push ebx
:00427941 6AFF                     push FFFFFFFF
:00427943 8D4C241C                 lea ecx, dword ptr [esp+1C]
:00427947 895C241C                 mov dword ptr [esp+1C], ebx
:0042794B 895C2420                 mov dword ptr [esp+20], ebx
:0042794F 895C2424                 mov dword ptr [esp+24], ebx
:00427953 C744242860000000         mov [esp+28], 00000060
:0042795B 895C242C                 mov dword ptr [esp+2C], ebx
:0042795F E85C690000               call 0042E2C0
:00427964 8B6C244C                 mov ebp, dword ptr [esp+4C]<-------假注册码
:00427968 895C2444                 mov dword ptr [esp+44], ebx
:0042796C 3BEB                     cmp ebp, ebx
:0042796E 7417                     je 00427987
:00427970 8BFD                     mov edi, ebp<---------------------开始计算注册码长度
:00427972 83C9FF                   or ecx, FFFFFFFF
:00427975 33C0                     xor eax, eax
:00427977 55                       push ebp
:00427978 F2                       repnz
:00427979 AE                       scasb
:0042797A F7D1                     not ecx
:0042797C 49                       dec ecx
:0042797D 51                       push ecx<------------------------- 并入栈
:0042797E 8D4C241C                 lea ecx, dword ptr [esp+1C]
:00427982 E839690000               call 0042E2C0

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042796E(C)
|
:00427987 8D442414                 lea eax, dword ptr [esp+14]<-------假注册码
:0042798B 8D4C2428                 lea ecx, dword ptr [esp+28]
:0042798F BF01000000               mov edi, 00000001
:00427994 50                       push eax
:00427995 51                       push ecx
:00427996 897C244C                 mov dword ptr [esp+4C], edi
:0042799A E8C1FBFFFF               call 00427560<---------------------(1.1.1)查表，跟入
:0042799F 83C408                   add esp, 00000008
:004279A2 8D4C2414                 lea ecx, dword ptr [esp+14]<-------假注册码
:004279A6 C644244403               mov [esp+44], 03
:004279AB E8E0680000               call 0042E290
```



```
:004279B0 395C2428                 cmp dword ptr [esp+28], ebx

:004279B4 7445                    je 004279FB

:004279B6 837C242C08              cmp dword ptr [esp+2C], 00000008

:004279BB 753E                    jne 004279FB

:004279BD 6820ED4600              push 0046ED20

:004279C2 6A5A                    push 0000005A

* Possible StringData Ref from Data Obj ->"w:\zaphod\strings.hpp"

                                          |

:004279C4 6878624600              push 00466278

* Possible StringData Ref from Data Obj ->"i>=0 && i<this->Len"

                                          |

:004279C9 6898824600              push 00468298

:004279CE 57                      push edi

:004279CF E83C660000              call 0042E010

:004279D4 8B44243C                mov eax, dword ptr [esp+3C]

:004279D8 83C414                  add esp, 00000014

:004279DB 8D4C2412                lea ecx, dword ptr [esp+12]

:004279DF 8B10                    mov edx, dword ptr [eax]

:004279E1 51                      push ecx

:004279E2 8916                    mov dword ptr [esi], edx

:004279E4 8B4004                  mov eax, dword ptr [eax+04]

:004279E7 8D542415                lea edx, dword ptr [esp+15]

:004279EB 8BCE                    mov ecx, esi

:004279ED 52                      push edx

:004279EE 894604                  mov dword ptr [esi+04], eax

:004279F1 E8CAF8FFFF              call 004272C0<---------------------(1.1.2)比较，跟入

:004279F6 89460C                  mov dword ptr [esi+0C], eax<-------0

:004279F9 EB07                    jmp 00427A02

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004279B4(C), :004279BB(C)

|

:004279FB C7460C07000000          mov [esi+0C], 00000007

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004279F9(U)

|

:00427A02 395E0C                  cmp dword ptr [esi+0C], ebx

:00427A05 7570                    jne 00427A77

:00427A07 8B06                    mov eax, dword ptr [esi]<----------0x67452310

:00427A09 8A4C2411                mov cl, byte ptr [esp+11]<---------0x4a

:00427A0D C1E818                  shr eax, 18<-----------------------0x67

:00427A10 3AC8                    cmp cl, al<------------------------0x67 和 0x4a 比较

:00427A12 7517                    jne 00427A2B

:00427A14 8B4E04                  mov ecx, dword ptr [esi+04]<-------0xefcdab89

:00427A17 C1E90C                  shr ecx, 0C<-----------------------0xefcda

:00427A1A 81E1FF0F0000            and ecx, 00000FFF<-----------------0xcda

:00427A20 66394C2412              cmp word ptr [esp+12], cx<---------0xcda 和 0x999 比较

:00427A25 7504                    jne 00427A2B

:00427A27 8BC7                    mov eax, edi

:00427A29 EB02                    jmp 00427A2D

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00427A12(C), :00427A25(C)

|

:00427A2B 33C0                    xor eax, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00427A29(U)

|

:00427A2D 3AC3                    cmp al, bl

:00427A2F 884608                  mov byte ptr [esi+08], al

:00427A32 7503                    jne 00427A37

:00427A34 897E0C                  mov dword ptr [esi+0C], edi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00427A32(C)

|

:00427A37 395E0C                  cmp dword ptr [esi+0C], ebx

:00427A3A 753B                    jne 00427A77

:00427A3C 55                      push ebp

:00427A3D E83E050000              call 00427F80

:00427A42 83C404                  add esp, 00000004

:00427A45 85C0                    test eax, eax

:00427A47 740A                    je 00427A53

:00427A49 885E08                  mov byte ptr [esi+08], bl

:00427A4C C7460C08000000          mov [esi+0C], 00000008

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00427A47(C)

|

:00427A53 395E0C                  cmp dword ptr [esi+0C], ebx

:00427A56 751F                    jne 00427A77

:00427A58 8BCE                    mov ecx, esi

:00427A5A E831F8FFFF              call 00427290

:00427A5F 50                      push eax

:00427A60 55                      push ebp

:00427A61 E86A050000              call 00427FD0

:00427A66 83C408                  add esp, 00000008

:00427A69 85C0                    test eax, eax

:00427A6B 740A                    je 00427A77

:00427A6D 885E08                  mov byte ptr [esi+08], bl

:00427A70 C7460C07000000          mov [esi+0C], 00000007

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00427A05(C), :00427A3A(C), :00427A56(C), :00427A6B(C)

|

:00427A77 8D4C2428                lea ecx, dword ptr [esp+28]

:00427A7B C7442444FFFFFFFF        mov [esp+44], FFFFFFFF

:00427A83 E808680000              call 0042E290

:00427A88 8B4C243C                mov ecx, dword ptr [esp+3C]

:00427A8C 5F                      pop edi

:00427A8D 5E                      pop esi

:00427A8E 5D                      pop ebp

:00427A8F 5B                      pop ebx

:00427A90 64890D00000000          mov dword ptr fs:[00000000], ecx

:00427A97 83C438                  add esp, 00000038

:00427A9A C20400                  ret 0004

============================================================================================

* Referenced by a CALL at Address:<--------------------------------call (1.1.1)

|:0042799A

|

:00427560 6AFF                    push FFFFFFFF

:00427562 683F884500              push 0045883F

:00427567 64A100000000            mov eax, dword ptr fs:[00000000]

:0042756D 50                      push eax

:0042756E 64892500000000          mov dword ptr fs:[00000000], esp

:00427575 83EC24                  sub esp, 00000024

:00427578 53                      push ebx

:00427579 55                      push ebp

:0042757A B820ED4600              mov eax, 0046ED20

:0042757F 33ED                    xor ebp, ebp

:00427581 56                      push esi

:00427582 57                      push edi

:00427583 85C0                    test eax, eax

:00427585 896C241C                mov dword ptr [esp+1C], ebp

:00427589 C644241300              mov [esp+13], 00

:0042758E 896C2414                mov dword ptr [esp+14], ebp

:00427592 896C2420                mov dword ptr [esp+20], ebp

:00427596 896C2424                mov dword ptr [esp+24], ebp

:0042759A 896C2428                mov dword ptr [esp+28], ebp

:0042759E C744242C60000000        mov [esp+2C], 00000060

:004275A6 896C2430                mov dword ptr [esp+30], ebp

:004275AA 740B                    je 004275B7

:004275AC 50                      push eax

:004275AD 55                      push ebp

:004275AE 8D4C2428                lea ecx, dword ptr [esp+28]

:004275B2 E8096D0000              call 0042E2C0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004275AA(C)

|

:004275B7 C744243C01000000        mov [esp+3C], 00000001

:004275BF 33FF                    xor edi, edi

:004275C1 896C2418                mov dword ptr [esp+18], ebp

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0042767B(U), :004276AC(U)

|

:004275C5 8B4C2448                mov ecx, dword ptr [esp+48]<-------假注册码

:004275C9 8B01                    mov eax, dword ptr [ecx]

:004275CB 3BC5                    cmp eax, ebp

:004275CD 7405                    je 004275D4

:004275CF 8B4904                  mov ecx, dword ptr [ecx+04]<-------假注册码长度

:004275D2 EB02                    jmp 004275D6
```



*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   |:004275CD(C)
*   |:004275D4
    ```
    :004275D4 33C9    xor ecx, ecx
    ```
*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   |:004275D2(U)
*   |:004275D6
    ```
    :004275D6 3BF9    cmp edi, ecx
    ```
*   |:004275D8
    ```
    :004275D8 0F8DD3000000 jnl 004276B1
    :004275DE 33C9    xor ecx, ecx
    :004275E0 8A0C38  mov cl, byte ptr [eax+edi] <-------取一个字
    :004275E3 51      push ecx
    :004275E4 E8B30F0200 call 0044859C<--------------------判断大小写
    :004275E9 8AD8    mov bl, al
    :004275EB 83C404  add esp, 00000004
    :004275EE 80FB20  cmp bl, 20<-----------------------判断是否空格
    :004275F1 0F84B4000000 je 004276AB
    :004275F7 80FB2D  cmp bl, 2D<-----------------------'-'
    :004275FA 0F84AB000000 je 004276AB
    :00427600 80FB0D  cmp bl, 0D
    :00427603 0F84A2000000 je 004276AB
    :00427609 80FB0A  cmp bl, 0A
    :0042760C 0F8499000000 je 004276AB<----------------------是的话,计数器加 1,不比较
    :00427612 8B442414 mov eax, dword ptr [esp+14]
    :00427616 B905000000 mov ecx, 00000005
    ```
*   |:0042761B
    ```
    :0042761B 99      cdq
    :0042761C F7F9    idiv ecx
    :0042761E 8B442414 mov eax, dword ptr [esp+14]
    :00427622 40      inc eax
    :00427623 89442414 mov dword ptr [esp+14], eax
    :00427627 33C0    xor eax, eax
    :00427629 8B3495F4814600 mov esi, dword ptr [4*edx+004681F4]<----查表
    :00427630 8A0E    mov cl, byte ptr [esi]
    :00427632 84C9    test cl, cl
    :00427634 740D    je 00427643
    ```
*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   |:00427641(C)
*   |:00427636
    ```
    :00427636 3ACB    cmp cl, bl<-----------------------------比较
    :00427638 7409    je 00427643
    :0042763A 8A4C3001 mov cl, byte ptr [eax+esi+01]<----------不等的话，取表中
    :0042763E 40      inc eax
    :0042763F 84C9    test cl, cl
    :00427641 75F3    jne 00427636
    ```
*   Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
    *   |:00427634(C), :00427638(C)
*   |:00427643
    ```
    :00427643 803C3000 cmp byte ptr [eax+esi], 00<------------查完表,还没有相等的
    :00427647 7502    jne 0042764B
    :00427649 33C0    xor eax, eax
    ```
*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   |:00427647(C)
*   |:0042764B
    ```
    :0042764B 8B4C2418 mov ecx, dword ptr [esp+18]
    :0042764F 8BD1    mov edx, ecx
    :00427651 81E201000080 and edx, 80000001<--------------------判断是奇位数，还是偶位数
    :00427657 7905    jns 0042765E
    ```
*   |:00427A61
    ```
    00427A61 E86A050000 call 00427FD0
    :00427A66 83C408  add esp, 00000008
    :00427A69 85C0    test eax, eax
    :00427A6B 740A    je 00427A77
    :00427A6D 885E08  mov byte ptr [esi+08], bl
    :00427A70 C7460C07000000 mov [esi+0C], 00000007
    ```
*   Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
    *   |:00427A05(C), :00427A3A(C), :00427A56(C), :00427A6B(C)
*   |:00427A77
    ```
    :00427A77 8D4C2428 lea ecx, dword ptr [esp+28]
    :00427A7B C7442444FFFFFFFF mov [esp+44], FFFFFFFF
    :00427A83 E808680000 call 0042E290
    :00427A88 8B4C243C mov ecx, dword ptr [esp+3C]
    :00427A8C 5F      pop edi
    :00427A8D 5E      pop esi
    :00427A8E 5D      pop ebp
    :00427A8F 5B      pop ebx
    :00427A90 64890D00000000 mov dword ptr fs:[00000000], ecx
    :00427A97 83C438  add esp, 00000038
    :00427A9A C20400  ret 0004
    ```

============================================================================================

*   Referenced by a CALL at Address:<--------------------------------call (1.1.1)
    *   |:0042799A
*   |:00427560
    ```
    :00427560 6AFF      push FFFFFFFF
    :00427562 683F884500 push 0045883F
    :00427567 64A100000000 mov eax, dword ptr fs:[00000000]
    :0042756D 50        push eax
    :0042756E 64892500000000 mov dword ptr fs:[00000000], esp
    :00427575 83EC24    sub esp, 00000024
    :00427578 53        push ebx
    :00427579 55        push ebp
    :0042757A B820ED4600 mov eax, 0046ED20
    :0042757F 33ED      xor ebp, ebp
    :00427581 56        push esi
    :00427582 57        push edi
    :00427583 85C0      test eax, eax
    :00427585 896C241C mov dword ptr [esp+1C], ebp
    :00427589 C644241300 mov [esp+13], 00
    :0042758E 896C2414 mov dword ptr [esp+14], ebp
    :00427592 896C2420 mov dword ptr [esp+20], ebp
    :00427596 896C2424 mov dword ptr [esp+24], ebp
    :0042759A 896C2428 mov dword ptr [esp+28], ebp
    :0042759E C744242C60000000 mov [esp+2C], 00000060
    :004275A6 896C2430 mov dword ptr [esp+30], ebp
    :004275AA 740B      je 004275B7
    ```
*   |:004275AC
    ```
    :004275AC 50        push eax
    :004275AD 55        push ebp
    :004275AE 8D4C2428 lea ecx, dword ptr [esp+28]
    :004275B2 E8096D0000 call 0042E2C0
    ```
*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   |:004275AA(C)
*   |:004275B7
    ```
    :004275B7 C744243C01000000 mov [esp+3C], 00000001
    :004275BF 33FF      xor edi, edi
    :004275C1 896C2418 mov dword ptr [esp+18], ebp
    ```
*   Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
    *   |:0042767B(U), :004276AC(U)
*   |:004275C5
    ```
    :004275C5 8B4C2448 mov ecx, dword ptr [esp+48]<-------假注册码
    :004275C9 8B01      mov eax, dword ptr [ecx]
    :004275CB 3BC5      cmp eax, ebp
    :004275CD 7405      je 004275D4
    :004275CF 8B4904    mov ecx, dword ptr [ecx+04]<-------假注册码长度
    :004275D2 EB02      jmp 004275D6
    ```
*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   |:004275CD(C)
*   |:004275D4
    ```
    :004275D4 33C9      xor ecx, ecx
    ```
*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   |:004275D2(U)
*   |:004275D6
    ```
    :004275D6 3BF9      cmp edi, ecx
    :004275D8 0F8DD3000000 jnl 004276B1
    :004275DE 33C9      xor ecx, ecx
    :004275E0 8A0C38    mov cl, byte ptr [eax+edi]<-------取一个字
    :004275E3 51        push ecx
    :004275E4 E8B30F0200 call 0044859C<--------------------判断大小写
    :004275E9 8AD8      mov bl, al
    :004275EB 83C404    add esp, 00000004
    :004275EE 80FB20    cmp bl, 20<-----------------------判断是否空格
    :004275F1 0F84B4000000 je 004276AB
    :004275F7 80FB2D    cmp bl, 2D<-----------------------'-'
    :004275FA 0F84AB000000 je 004276AB
    :00427600 80FB0D    cmp bl, 0D
    :00427603 0F84A2000000 je 004276AB
    :00427609 80FB0A    cmp bl, 0A
    :0042760C 0F8499000000 je 004276AB<----------------------是的话,计数器加 1,不比较
    :00427612 8B442414 mov eax, dword ptr [esp+14]
    :00427616 B905000000 mov ecx, 00000005
    :0042761B 99        cdq
    :0042761C F7F9      idiv ecx
    :0042761E 8B442414 mov eax, dword ptr [esp+14]
    :00427622 40        inc eax
    :00427623 89442414 mov dword ptr [esp+14], eax
    :00427627 33C0      xor eax, eax
    :00427629 8B3495F4814600 mov esi, dword ptr [4*edx+004681F4]<----查表
    :00427630 8A0E      mov cl, byte ptr [esi]
    :00427632 84C9      test cl, cl
    :00427634 740D      je 00427643
    ```
*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   |:00427641(C)
*   |:00427636
    ```
    :00427636 3ACB      cmp cl, bl<-----------------------------比较
    :00427638 7409      je 00427643
    :0042763A 8A4C3001 mov cl, byte ptr [eax+esi+01]<----------不等的话，取表中
    :0042763E 40        inc eax
    :0042763F 84C9      test cl, cl
    :00427641 75F3      jne 00427636
    ```
*   Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
    *   |:00427634(C), :00427638(C)
*   |:00427643
    ```
    :00427643 803C3000 cmp byte ptr [eax+esi], 00<------------查完表,还没有相等的
    :00427647 7502      jne 0042764B
    :00427649 33C0      xor eax, eax
    ```
*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   |:00427647(C)
*   |:0042764B
    ```
    :0042764B 8B4C2418 mov ecx, dword ptr [esp+18]
    :0042764F 8BD1      mov edx, ecx
    :00427651 81E201000080 and edx, 80000001<--------------------判断是奇位数，还是偶位数
    :00427657 7905      jns 0042765E
    ```



:00427659 `4A` `dec edx`

:0042765A `83CAFE` `or edx, FFFFFFFE`

:0042765D `42` `inc edx`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

\| :00427657(C)

\|

:0042765E `41` `inc ecx`

:0042765F `85D2` `test edx, edx`

:00427661 `894C2418` `mov dword ptr [esp+18], ecx`

:00427665 `7519` `jne 00427680`

:00427667 `250F000080` `and eax, 8000000F`

:0042766C `7905` `jns 00427673`

:0042766E `48` `dec eax`

:0042766F `83C8F0` `or eax, FFFFFFF0`

:00427672 `40` `inc eax`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

\| :0042766C(C)

\|

:00427673 `C0E004` `shl al, 04`

:00427676 `88442413` `mov byte ptr [esp+13], al`

:0042767A `47` `inc edi`

:0042767B `E945FFFFFF` `jmp 004275C5` <-----------------------回去,取下一个字

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

\| :00427665(C)

\|

:00427680 `250F000080` `and eax, 8000000F`

:00427685 `7905` `jns 0042768C`

:00427687 `48` `dec eax`

:00427688 `83C8F0` `or eax, FFFFFFF0`

:0042768B `40` `inc eax`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

\| :00427685(C)

\|

:0042768C `8A5C2413` `mov bl, byte ptr [esp+13]`

:00427690 `6A01` `push 00000001`

:00427692 `0AD8` `or bl, al`

:00427694 `8D442417` `lea eax, dword ptr [esp+17]`

:00427698 `50` `push eax`

:00427699 `8D4C2428` `lea ecx, dword ptr [esp+28]`

:0042769D `885C241B` `mov byte ptr [esp+1B], bl`

:004276A1 `E87A710000` `call 0042E820` <---------------------注册码在表中的序号

:004276A6 `C644241300` `mov [esp+13], 00`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

\| :004275F1(C), :004275FA(C), :00427603(C), :0042760C(C)

\|

:004276AB `47` `inc edi` <----------------------------计数器加 1

:004276AC `E914FFFFFF` `jmp 004275C5` <-----------------------回去,取下一个字

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

\| :004275D8(C)

\|

:004276B1 `8B742444` `mov esi, dword ptr [esp+44]`

:004276B5 `8B442420` `mov eax, dword ptr [esp+20]`

:004276B9 `8B4C242C` `mov ecx, dword ptr [esp+2C]`

:004276BD `8B542430` `mov edx, dword ptr [esp+30]`

:004276C1 `3BC5` `cmp eax, ebp`

:004276C3 `892E` `mov dword ptr [esi], ebp`

:004276C5 `896E04` `mov dword ptr [esi+04], ebp`

:004276C8 `896E08` `mov dword ptr [esi+08], ebp`

:004276CB `894E0C` `mov dword ptr [esi+0C], ecx`

:004276CE `895610` `mov dword ptr [esi+10], edx`

:004276D1 `740D` `je 004276E0`

:004276D3 `50` `push eax`

:004276D4 `8B442428` `mov eax, dword ptr [esp+28]`

:004276D8 `50` `push eax`

:004276D9 `8BCE` `mov ecx, esi`

:004276DB `E8E06B0000` `call 0042E2C0`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

\| :004276D1(C)

\|

:004276E0 `C744241C01000000` `mov [esp+1C], 00000001`

:004276E8 `8D4C2420` `lea ecx, dword ptr [esp+20]`

:004276EC `C644243C00` `mov [esp+3C], 00`

:004276F1 `E89A6B0000` `call 0042E290`

:004276F6 `8B4C2434` `mov ecx, dword ptr [esp+34]`

:004276FA `8BC6` `mov eax, esi`

:004276FC `5F` `pop edi`

:004276FD `5E` `pop esi`

:004276FE `5D` `pop ebp`

:004276FF `5B` `pop ebx`

:00427700 `64890D00000000` `mov dword ptr fs:[00000000], ecx`

:00427707 `83C430` `add esp, 00000030`

:0042770A `C3` `ret`

\========================================================================================

\* Referenced by a CALL at Address:<------------------------------call (1.1.2)

\| :004279F1

\|

:004272C0 `83EC1C` `sub esp, 0000001C`

:004272C3 `53` `push ebx`

:004272C4 `55` `push ebp`

:004272C5 `56` `push esi` <---------------------转换后的注册码(序号)

:004272C6 `57` `push edi`

:004272C7 `8B39` `mov edi, dword ptr [ecx]` <-----0x67452310

:004272C9 `33C0` `xor eax, eax`

:004272CB `8BD7` `mov edx, edi` <-----------------0x67452310

:004272CD `89442410` `mov dword ptr [esp+10], eax` <------0

:004272D1 `83E20F` `and edx, 0000000F` <------------取第 2 位

:004272D4 `C74424140D000000` `mov [esp+14], 0000000D` <----|

:004272DC `83FA01` `cmp edx, 00000001` |

:004272DF `C744241803000000` `mov [esp+18], 00000003` <----|注意这几个数字

:004272E7 `C744241C01000000` `mov [esp+1C], 00000001` <----|

:004272EF `C744242009000000` `mov [esp+20], 00000009` <----|

:004272F7 `C744242405000000` `mov [esp+24], 00000005` <----|

:004272FF `89442428` `mov dword ptr [esp+28], eax`

:00427303 `736F` `jnb 00427374` <---------------------第 2 位要为 0

:00427305 `8B7104` `mov esi, dword ptr [ecx+04]` <------0xefcdab89

:00427308 `8D1452` `lea edx, dword ptr [edx+2*edx]`

:0042730B `8BEE` `mov ebp, esi`

:0042730D `83E50F` `and ebp, 0000000F` <----------------取第 10 位'9'

:00427310 `8D5CD414` `lea ebx, dword ptr [esp+8*edx+14]`

:00427314 `7519` `jne 0042732F`

:00427316 `F7C6000E0000` `test esi, 00000E00`

:0042731C `7511` `jne 0042732F`

:0042731E `F7C7F0FFFF00` `test edi, 00FFFFF0`

:00427324 `7509` `jne 0042732F`

:00427326 `33D2` `xor edx, edx`

:00427328 `B907000000` `mov ecx, 00000007`

:0042732D `EB4C` `jmp 0042737B`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

\| :00427314(C), :0042731C(C), :00427324(C)

\|

:0042732F `8B4B10` `mov ecx, dword ptr [ebx+10]` <------0x00000005 用上了

:00427332 `8BD6` `mov edx, esi` <---------------------0xefcdab89

:00427334 `C1EA04` `shr edx, 04` <----------------------0xefcdab89>>4==0x0efcdab8

:00427337 `8BC6` `mov eax, esi` <---------------------0xefcdab89

:00427339 `83E21F` `and edx, 0000001F` <----------------0x0efcdab8&0x1f==0x18

:0042733C `C1E818` `shr eax, 18` <----------------------0xefcdab89>>18==0xef

:0042733F `D3E2` `shl edx, cl` <----------------------0x18<<0x5==0x300

:00427341 `8B4B0C` `mov ecx, dword ptr [ebx+0C]` <------0x00000009 用上了

:00427344 `83E00F` `and eax, 0000000F` <----------------取第 16 位'f'

:00427347 `C1EE09` `shr esi, 09` <----------------------0xefcdab89>>9==0x77e6d5

:0042734A `D3E0` `shl eax, cl` <----------------------第 16 位 0xf<<9==0x1e00

:0042734C `8B4B04` `mov ecx, dword ptr [ebx+04]` <------0x00000003 用上了

:0042734F `83E607` `and esi, 00000007` <----------------0x77e6d5&0x7==0x5

:00427352 `C1EF04` `shr edi, 04` <----------------------0x67452310>>4==0x06745231

:00427355 `D3E6` `shl esi, cl` <----------------------0x5<<0x3==0x28



### 汇编代码段 1

`:00427357` `8B4B08` `mov ecx, dword ptr [ebx+08]` `<------0x00000001 用上了`

`:0042735A` `81E7FFFF0F00` `and edi, 000FFFFF` `<----------------0x06745231&0xfffff==0x45231`

`:00427360` `D3E7` `shl edi, cl` `<----------------------0x45231<<0x1==0x8a462`

`:00427362` `8B0B` `mov ecx, dword ptr [ebx]` `<---------0x0000000d 用上了`

`:00427364` `33D0` `xor edx, eax` `<---------------------0x300⁰x1e00==0x1d00`

`:00427366` `D3E5` `shl ebp, cl` `<----------------------第 10 位 0x9<<0xd==0x12000`

`:00427368` `8B4C2410` `mov ecx, dword ptr [esp+10]` `<------0`

`:0042736C` `33D6` `xor edx, esi` `<---------------------0x1d00⁰x28==0x1d28`

`:0042736E` `33D7` `xor edx, edi` `<---------------------0x1d28⁰x8a462==0x8b94a`

`:00427370` `33D5` `xor edx, ebp` `<---------------------0x8b94a⁰x12000==0x9994a`

`:00427372` `EB07` `jmp 0042737B`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00427303(C)` |

`:00427374` `B903000000` `mov ecx, 00000003`

`:00427379` `33D2` `xor edx, edx`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| `:0042732D(U), :00427372(U)` |

`:0042737B` `8B442430` `mov eax, dword ptr [esp+30]`

`:0042737F` `5F` `pop edi`

`:00427380` `5E` `pop esi`

`:00427381` `5D` `pop ebp`

`:00427382` `8810` `mov byte ptr [eax], dl` `<-----------0x4a`

`:00427384` `8B442428` `mov eax, dword ptr [esp+28]`

`:00427388` `C1EA08` `shr edx, 08` `<----------------------0x9994a>>0x8==0x999`

`:0042738B` `81E2FF0F0000` `and edx, 00000FFF`

`:00427391` `5B` `pop ebx`

`:00427392` `668910` `mov word ptr [eax], dx` `<-----------0x999`

`:00427395` `8BC1` `mov eax, ecx`

`:00427397` `83C41C` `add esp, 0000001C`

`:0042739A` `C20800` `ret 0008`

===================================================================================================

### 汇编代码段 2

* Referenced by a CALL at Addresses: `<-------------------------------CALL (2)`

| `:00427B01` , `:0042882A` , `:0042911D` |

`:00427270` `8B4104` `mov eax, dword ptr [ecx+04]`

`:00427273` `C1E818` `shr eax, 18`

`:00427276` `83E00F` `and eax, 0000000F` `<--------------取最后 1 位,if`

`:00427279` `C3` `ret` `要为 1`

===================================================================================================

### 汇编代码段 3

* Referenced by a CALL at Addresses: `<-------------------------------CALL (3)`

| `:00427B0F` , `:00428846` , `:0042913C` |

`:00427250` `8B4104` `mov eax, dword ptr [ecx+04]`

`:00427253` `83E00F` `and eax, 0000000F`

`:00427256` `83C041` `add eax, 00000041` `<-------------i9+0x41==0x4e`

`:00427259` `C3` `ret` `i9==0xd`

===================================================================================================

### 查表数据

文中程序所查的表：

| | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | A | B | C | D | E | F |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 0 | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P |
| 1 | Q | R | S | T | U | V | W | X | Y | A | B | C | D | E | F | G | <-----------有没有发现少了点什么
| 2 | H | I | J | K | L | M | N | O | P | Q | R | S | T | U | V | W |
| 3 | X | Y | Z | A | B | C | D | E | F | G | H | I | J | K | L | M |
| 4 | N | O | P | Q | R | S | T | U | V | W | X | Y | Z | A | B | C |

### 注册码推导

注册码：`i0 i1 i2 i3 i4` - `i5 i6 i7 i8 i9` - `ia ib ic` - `id ie if` <-------共 16 位

注册码: `B Q J A R - F W O F W - K C T - K B P` <--------原来输入的注册码

序号: `1 0 2 3 4 5 6 7 8 9 a b c d e f` <--------这样比较容易看出是那一位

已知： `i1==0`, `i9==0xd`, `if==1`

假设： `i0==1`, `i2==1`, `i3==1`, `i4==1`, `i5==1`, `i8==0`, `ib==0`, `ie==0`

则：
(1) `0xi4i5i2i3i0<<1==0xi4i5i2i3i0*2==0x11111*0x2==0x22222`
(2) `0xif<<0x9==0xif*0x200==0x1*0x200==0x200`
(3) `i9<<0xd==0xi9*0x2000==0xd*0x2000==0x1a000`
(4) `0xi8<<0x5==0xi8*0x20==0x0`
(5) `(0xieificidiaibi8i9>>0x9)&0x7==>(0xib/2)&0x7==0`
(6) `(((0x200 xor 0) xor 0) xor 0x22222) xor 0x1a000==0xicidiai6i7==0x38022`

所以:

注册码: `i0 i1 i2 i3 i4 - i5 i6 i7 i8 i9 - ia ib ic - id ie if`

序号: `1 0 1 1 1 1 2 2 0 d 0 0 3 8 0 1`

注册码: `B Q I Y O - B S J X A - A Q K - F N B`

### 签名

```
        ,;~;,
          /\__
        ( /
        (()      //)
        | \\ ,,;\'
        __ _( )m=(lancelot(================--------
       /' ' '()/~' '.(, |
      ,;( )||   | ~
    ,;' \ /-(.;, )
         ) /  /
       //       ||
       )\      )\  兰斯洛特[CCG][FCG]
                 2001.09.18
         4dd466d0e1e0cc123afbe504da2462ca
```
