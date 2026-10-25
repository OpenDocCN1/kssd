
# leapftp2.7.4

> 原文：[`www.pediy.com/kssd/pediy06/pediy6521.htm`](https://www.pediy.com/kssd/pediy06/pediy6521.htm)

闲着没事，从网上拉下来分析了一下，发现这个软件注册算法很有趣。他一共有两种注册码算法。

第一种注册码和用户名无关，可以随便输入。运算手法只是将你输入假码经过运算，得出真正的注册码，格式为：

`XXXX-XXXX-XXXX-XXXX`。

第二种注册码是将用户名运算出一个注册码，然后再和一个`214065`合在一起，也就是楼主所说的在内存中看到的不变的数字。格式为：

`214065-XXXXXXXXX` \\\ 注：这里的`XXXXXXXXX`是将用户名运算出来的，不一定是九位。

以下是我粗粗的分析了一下，不对之处请大家指教。

```
:0048742A 8B45F8                mov eax, dword ptr [ebp-08]              \\中断在这里，此时 EAX 值为输入假码。
:0048742D 8D55FC                lea edx, dword ptr [ebp-04]
:00487430 E87B16F8FF            call 00408AB0                            \\这个 Call 虽然可疑，但不是关键点。
:00487435 80BBF402000000        cmp byte ptr [ebx+000002F4], 00
:0048743C 740E                  je 0048744C
:0048743E 8B55FC                mov edx, dword ptr [ebp-04]
:00487441 8BC3                  mov eax, ebx
:00487443 E888030000            call 004877D0                            \\第一个关健 Call，算出第一种注册码，格式为：XXXX-XXXX-XXXX-XXXX。
:00487448 84C0                  test al, al
:0048744A 7526                  jne 00487472                             \\一开始已经说过，这个软件有两种注册码算法，所以如果第一个注册码是对的就直接跳到注册成功处，如果第一个注册码是错的话这里将不跳继续算出第二种注册码。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0048743C(C)
|

:0048744C 8B83F0020000          mov eax, dword ptr [ebx+000002F0]        \\第一次注册码错的话在这里继续运算。
:00487452 50                    push eax
:00487453 8D55F4                lea edx, dword ptr [ebp-0C]
:00487456 8B83D0020000          mov eax, dword ptr [ebx+000002D0]
:0048745C E867C5FAFF            call 004339C8
:00487461 8B55F4                mov edx, dword ptr [ebp-0C]
:00487464 8B4DFC                mov ecx, dword ptr [ebp-04]
:00487467 8BC3                  mov eax, ebx
:00487469 E8BA010000            call 00487628                            \\第二个关健 Call，算出第二种注册码，格式为：214065-XXXXXXXXX。
:0048746E 84C0                  test al, al
:00487470 7462                  je 004874D4                              \\若第二种注册码也是错的话，这里将跳到出错点。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0048744A(C)
|

:00487472 8D55F0                lea edx, dword ptr [ebp-10]
:00487475 8B83E4020000          mov eax, dword ptr [ebx+000002E4]
:0048747B E848C5FAFF            call 004339C8
:00487480 8B45F0                mov eax, dword ptr [ebp-10]
:00487483 50                    push eax
:00487484 8D55EC                lea edx, dword ptr [ebp-14]
:00487487 8B83D0020000          mov eax, dword ptr [ebx+000002D0]
:0048748D E836C5FAFF            call 004339C8
:00487492 8B4DEC                mov ecx, dword ptr [ebp-14]
:00487495 8B93EC020000          mov edx, dword ptr [ebx+000002EC]
:0048749B 8BC3                  mov eax, ebx
:0048749D E8AE040000            call 00487950

* Possible StringData Ref from Code Obj ->"感谢你的注册！"
                                  |
:004874A2 B820754800            mov eax, 00487520
:004874A7 E87834FDFF            call 0045A924
:004874AC C7833402000001000000  mov dword ptr [ebx+00000234], 00000001
:004874B6 8D55E8                lea edx, dword ptr [ebp-18]
:004874B9 8B83D0020000          mov eax, dword ptr [ebx+000002D0]
:004874BF E804C5FAFF            call 004339C8
:004874C4 8B55E8                mov edx, dword ptr [ebp-18]
:004874C7 8D83E8020000          lea eax, dword ptr [ebx+000002E8]
:004874CD E806C8F7FF            call 00403CD8
:004874D2 EB15                  jmp 004874E9

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00487470(C)
|

:004874D4 6A00                  push 00000000
:004874D6 668B0D3C754800        mov cx, word ptr [0048753C]
:004874DD B201                  mov dl, 01

* Possible StringData Ref from Code Obj ->"你输入的许可密匙是不正确的. 要确保准确, "
                                  ->"你应该直接总你的购买确认 E-Mail "
                                  ->"中复制并粘贴序列号. 如果你继续操作后碰到麻烦, "
                                  ->"请联系 support@leapware.com."
                                  |
:004874DF B848754800            mov eax, 00487548
:004874E4 E84333FDFF            call 0045A82C                             \\跳到这里已经完了。
```

这里是第一个关健 Call

```
:004877D0 55                    push ebp
:004877D1 8BEC                  mov ebp, esp
:004877D3 83C4F4                add esp, FFFFFFFF4
:004877D6 53                    push ebx
:004877D7 56                    push esi
:004877D8 57                    push edi
:004877D9 8955FC                mov dword ptr [ebp-04], edx
:004877DC 8B45FC                mov eax, dword ptr [ebp-04]
:004877DF E8D4C8F7FF            call 004040B8
:004877E4 33C0                  xor eax, eax
:004877E6 55                    push ebp
:004877E7 683D794800            push 0048793D
:004877EC 64FF30                push dword ptr fs:[eax]
:004877EF 648920                mov dword ptr fs:[eax], esp
:004877F2 C645FB00              mov [ebp-05], 00
:004877F6 8B45FC                mov eax, dword ptr [ebp-04]
:004877F9 E806C7F7FF            call 00403F04                             \\这个 Call 得到注册码位数
:004877FE 83F813                cmp eax, 00000013
:00487801 0F8520010000          jne 00487927                              \\若注册码位数不等于十六进制 13 位，就跳出这个 Call 进行第二种注册码运算。
:00487807 8B45FC                mov eax, dword ptr [ebp-04]
:0048780A 8078042D              cmp byte ptr [eax+04], 2D               \\注册码第 5 位必须是 ASLL 码“-"
:0048780E 0F8513010000          jne 00487927
:00487814 8B45FC                mov eax, dword ptr [ebp-04]
:00487817 8078092D              cmp byte ptr [eax+09], 2D               \\注册码第 10 位必须是 ASLL 码“-”
:0048781B 0F8506010000          jne 00487927
:00487821 8B45FC                mov eax, dword ptr [ebp-04]
:00487824 80780E2D              cmp byte ptr [eax+0E], 2D               \\注册码第 15 位必须是 ASLL 码“-”
```

从这里开始是将注册码的第一位至十四位运算出 4 个值分别保存。我这里就不注释了。有兴趣的朋友自己分析分析。

```
:00487828 0F85F9000000          jne 00487927
:0048782E 33F6                  xor esi, esi
:00487830 33FF                  xor edi, edi
:00487832 33C0                  xor eax, eax
:00487834 8945F4                mov dword ptr [ebp-0C], eax
:00487837 BB01000000            mov ebx, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004878C2(C)
|

:0048783C 8BC3                  mov eax, ebx
:0048783E 2503000080            and eax, 80000003
:00487843 7905                  jns 0048784A
:00487845 48                    dec eax
:00487846 83C8FC                or eax, FFFFFFFC
:00487849 40                    inc eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00487843(C)
|

:0048784A 85C0                  test eax, eax
:0048784C 7516                  jne 00487864
:0048784E 8B45FC                mov eax, dword ptr [ebp-04]
:00487851 8A4418FF              mov al, byte ptr [eax+ebx-01]
:00487855 E84EFFFFFF            call 004877A8
:0048785A 84C0                  test al, al
:0048785C 0F84C5000000          je 00487927
:00487862 EB22                  jmp 00487886

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0048784C(C)
|

:00487864 8BC3                  mov eax, ebx
:00487866 B905000000            mov ecx, 00000005
:0048786B 99                    cdq
:0048786C F7F9                  idiv ecx
:0048786E 85D2                  test edx, edx
:00487870 7414                  je 00487886
:00487872 8B45FC                mov eax, dword ptr [ebp-04]
:00487875 8A4418FF              mov al, byte ptr [eax+ebx-01]
:00487879 E83EFFFFFF            call 004877BC
:0048787E 84C0                  test al, al
```



```
:00487880 0F84A1000000            je 00487927
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00487862(U), :00487870(C)

|

```
:00487886 8B45FC                 mov eax, dword ptr [ebp-04]
:00487889 8A4418FF               mov al, byte ptr [eax+ebx-01]
:0048788D 3C2D                   cmp al, 2D
:0048788F 742D                   je 004878BE
:00487891 83FB05                 cmp ebx, 00000005
:00487894 7D0C                   jge 004878A2
:00487896 8B55FC                 mov edx, dword ptr [ebp-04]
:00487899 25FF000000             and eax, 000000FF
:0048789E 03F0                   add esi, eax
:004878A0 EB1C                   jmp 004878BE
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00487894(C)

|

```
:004878A2 83FB0A                 cmp ebx, 0000000A
:004878A5 7D0C                   jge 004878B3
:004878A7 8B55FC                 mov edx, dword ptr [ebp-04]
:004878AA 25FF000000             and eax, 000000FF
:004878AF 03F8                   add edi, eax
:004878B1 EB0B                   jmp 004878BE
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004878A5(C)

|

```
:004878B3 8B55FC                 mov edx, dword ptr [ebp-04]
:004878B6 25FF000000             and eax, 000000FF
:004878BB 0145F4                 add dword ptr [ebp-0C], eax
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0048788F(C), :004878A0(U), :004878B1(U)

|

```
:004878BE 43                     inc ebx
:004878BF 83FB0F                 cmp ebx, 0000000F
:004878C2 0F8574FFFFFF           jne 0048783C
:004878C8 8D0C37                 lea ecx, dword ptr [edi+esi]
:004878CB 034DF4                 add ecx, dword ptr [ebp-0C]
:004878CE 8BC6                   mov eax, esi
```

从这里开始再将上面运算出的 4 个值运算出最后四位注册码。

```
:004878D0 BB1A000000             mov ebx, 0000001A
:004878D5 99                     cdq
:004878D6 F7FB                   idiv ebx
:004878D8 83C241                 add edx, 00000041
:004878DB 8B45FC                 mov eax, dword ptr [ebp-04]
:004878DE 3A500F                 cmp dl, byte ptr [eax+0F]
:004878E1 7544                   jne 00487927
:004878E3 8BC7                   mov eax, edi
:004878E5 BB1A000000             mov ebx, 0000001A
:004878EA 99                     cdq
:004878EB F7FB                   idiv ebx
:004878ED 83C241                 add edx, 00000041
:004878F0 8B45FC                 mov eax, dword ptr [ebp-04]
:004878F3 3A5010                 cmp dl, byte ptr [eax+10]
:004878F6 752F                   jne 00487927
:004878F8 8B45F4                 mov eax, dword ptr [ebp-0C]
:004878FB BB1A000000             mov ebx, 0000001A
:00487900 99                     cdq
:00487901 F7FB                   idiv ebx
:00487903 83C241                 add edx, 00000041
:00487906 8B45FC                 mov eax, dword ptr [ebp-04]
:00487909 3A5011                 cmp dl, byte ptr [eax+11]
:0048790C 7519                   jne 00487927
:0048790E 8BC1                   mov eax, ecx
:00487910 B91A000000             mov ecx, 0000001A
:00487915 99                     cdq
:00487916 F7F9                   idiv ecx
:00487918 83C241                 add edx, 00000041
:0048791B 8B45FC                 mov eax, dword ptr [ebp-04]
:0048791E 3A5012                 cmp dl, byte ptr [eax+12]
:00487921 7504                   jne 00487927
:00487923 C645FB01               mov [ebp-05], 01
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00487801(C), :0048780E(C), :0048781B(C), :00487828(C), :0048785C(C)

|:00487880(C), :004878E1(C), :004878F6(C), :0048790C(C), :00487921(C)

|

```
:00487927 33C0                   xor eax, eax
:00487929 5A                     pop edx
:0048792A 59                     pop ecx
:0048792B 59                     pop ecx
:0048792C 648910                 mov dword ptr fs:[eax], edx
:0048792F 6844794800             push 00487944
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00487942(U)

|

```
:00487934 8D45FC                 lea eax, dword ptr [ebp-04]
:00487937 E848C3F7FF             call 00403C84
:0048793C C3                     ret
:0048793D E9DABDF7FF             jmp 0040371C
:00487942 EBF0                   jmp 00487934
:00487944 8A45FB                 mov al, byte ptr [ebp-05]
:00487947 5F                     pop edi
:00487948 5E                     pop esi
:00487949 5B                     pop ebx
:0048794A 8BE5                   mov esp, ebp
:0048794C 5D                     pop ebp
:0048794D C3                     ret
```

这里是第二个关健 Call，也是楼主所找到的关健 Call。

```
:00487628 55                     push ebp
:00487629 8BEC                   mov ebp, esp
:0048762B 83C4DC                 add esp, FFFFFFFFDC
:0048762E 53                     push ebx
:0048762F 33DB                   xor ebx, ebx
:00487631 895DDC                 mov dword ptr [ebp-24], ebx
:00487634 895DE0                 mov dword ptr [ebp-20], ebx
:00487637 895DEC                 mov dword ptr [ebp-14], ebx
:0048763A 894DF8                 mov dword ptr [ebp-08], ecx
:0048763D 8955FC                 mov dword ptr [ebp-04], edx      \\初始化内存。
:00487640 8B45FC                 mov eax, dword ptr [ebp-04]
:00487643 E870CAF7FF             call 004040B8
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004875CF(C)

|

```
:00487648 8B45F8                 mov eax, dword ptr [ebp-08]
:0048764B E868CAF7FF             call 004040B8
:00487650 8B4508                 mov eax, dword ptr [ebp+08]
:00487653 E860CAF7FF             call 004040B8
:00487658 33C0                   xor eax, eax
:0048765A 55                     push ebp
:0048765B 688B774800             push 0048778B
:00487660 64FF30                 push dword ptr fs:[eax]
:00487663 648920                 mov dword ptr fs:[eax], esp
:00487666 33C0                   xor eax, eax
:00487668 8945F0                 mov dword ptr [ebp-10], eax
:0048766B 8945F4                 mov dword ptr [ebp-0C], eax
:0048766E 8B45FC                 mov eax, dword ptr [ebp-04]
:00487671 E88EC8F7FF             call 00403F04
:00487676 8BD0                   mov edx, eax
:00487678 85D2                   test edx, edx
:0048767A 7E33                   jle 004876AF
:0048767C B801000000             mov eax, 00000001
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004876AD(C)

|

```
:00487681 8B4DFC                 mov ecx, dword ptr [ebp-04]      \\取得用户名
:00487684 0FB64C01FF             movzx ecx, byte ptr [ecx+eax-01]
:00487689 0FAFC8                 imul ecx, eax
:0048768C 8BD9                   mov ebx, ecx
:0048768E C1E104                 shl ecx, 04
:00487691 2BCB                   sub ecx, ebx
:00487693 894DE8                 mov dword ptr [ebp-18], ecx
:00487696 DB45E8                 fild dword ptr [ebp-18]
:00487699 DC45F0                 fadd qword ptr [ebp-10]
:0048769C 8D0C80                 lea ecx, dword ptr [eax+4*eax]
:0048769F 894DE4                 mov dword ptr [ebp-1C], ecx
:004876A2 DB45E4                 fild dword ptr [ebp-1C]
:004876A5 DEC1                   faddp st(1), st(0)
:004876A7 DD5DF0                 fstp qword ptr [ebp-10]
:004876AA 9B                     wait
:004876AB 40                     inc eax
:004876AC 4A                     dec edx
:004876AD 75D2                   jne 00487681
```

将用户名分别取出循环运算出一个值，最后保存到浮点寄存器中。



### 无条件或条件跳转引用 (0048767A)

*   **引用自**：`:0048767A` (条件跳转 C)

```asm
:004876AF 8B4508          mov eax, dword ptr [ebp+08]  ; 取出注册码常数 214065。
:004876B2 E84D16F8FF      call 00408D04
:004876B7 8945E8          mov dword ptr [ebp-18], eax  ; 这里开始再将注册常数和刚才用户名运算到的值再用浮点数算出正确注册码。
:004876BA DB45E8          fild dword ptr [ebp-18]
:004876BD DD45F0          fld qword ptr [ebp-10]
:004876C0 DC4DF0          fmul qword ptr [ebp-10]
:004876C3 DEC1           faddp st(1), st(0)
:004876C5 DD5DF0          fstp qword ptr [ebp-10]
:004876C8 9B             wait
:004876C9 DD45F0          fld qword ptr [ebp-10]
:004876CC 83C4F4         add esp, FFFFFFFF4
:004876CF DB3C24         fstp tbyte ptr [esp]
:004876D2 9B             wait
:004876D3 8D45EC         lea eax, dword ptr [ebp-14]  ; 到这里已经运算出正确的注册码。保存在浮点寄存器中。
:004876D6 E85525F8FF     call 00409C30
:004876DB 8D45E0         lea eax, dword ptr [ebp-20]
:004876DE 50             push eax
:004876DF 8B55F8         mov edx, dword ptr [ebp-08]
:004876E2 B8A4774800     mov eax, 004877A4
:004876E7 E804CBF7FF     call 004041F0
:004876EC 8BC8           mov ecx, eax
:004876EE 49             dec ecx
:004876EF BA01000000     mov edx, 00000001
:004876F4 8B45F8         mov eax, dword ptr [ebp-08]
:004876F7 E810CAF7FF     call 0040410C
:004876FC 8B45E0         mov eax, dword ptr [ebp-20]
:004876FF 8B5508         mov edx, dword ptr [ebp+08]
:00487702 E80DC9F7FF     call 00404014  ; 这个 Call 比较注册码的正错。
:00487707 7548          jne 00487751  ; 若返回 0 则说明你的注册码是错的。
:00487709 8D45DC         lea eax, dword ptr [ebp-24]
:0048770C 50             push eax
:0048770D 8B55F8         mov edx, dword ptr [ebp-08]
:00487710 B8A4774800     mov eax, 004877A4
:00487715 E8D6CAF7FF     call 004041F0
:0048771A 50             push eax
:0048771B 8B45F8         mov eax, dword ptr [ebp-08]
:0048771E E8E1C7F7FF     call 00403F04
:00487723 5A             pop edx
:00487724 2BC2           sub eax, edx
:00487726 50             push eax
:00487727 8B55F8         mov edx, dword ptr [ebp-08]
:0048772A B8A4774800     mov eax, 004877A4
:0048772F E8BCCAF7FF     call 004041F0
:00487734 8BD0           mov edx, eax
:00487736 42             inc edx
:00487737 8B45F8         mov eax, dword ptr [ebp-08]
:0048773A 59             pop ecx
:0048773B E8CCC9F7FF     call 0040410C
:00487740 8B45DC         mov eax, dword ptr [ebp-24]
:00487743 8B55EC         mov edx, dword ptr [ebp-14]
:00487746 E8C9C8F7FF     call 00404014
:0048774B 7504          jne 00487751
:0048774D B301           mov bl, 01
:0048774F EB02          jmp 00487753
```

### 无条件或条件跳转引用 (00487751)

*   **引用自**：`:00487707` (条件跳转 C), `:0048774B` (条件跳转 C)

```asm
:00487751 33DB          xor ebx, ebx  ; 跳到这，ebx 清 0，程序已经完了,
```

### 无条件或条件跳转引用 (00487753)

*   **引用自**：`:0048774F` (无条件跳转 U)

```asm
:00487753 33C0          xor eax, eax
:00487755 5A            pop edx
:00487756 59            pop ecx
:00487757 59            pop ecx
:00487758 648910        mov dword ptr fs:[eax], edx
:0048775B 6892774800    push 00487792
```

### 无条件跳转引用 (00487760)

*   **引用自**：`:00487790` (无条件跳转 U)

```asm
:00487760 8D45DC        lea eax, dword ptr [ebp-24]
:00487763 BA02000000    mov edx, 00000002
:00487768 E83BC5F7FF    call 00403CA8
:0048776D 8D45EC        lea eax, dword ptr [ebp-14]
:00487770 E80FC5F7FF    call 00403C84
:00487775 8D45F8        lea eax, dword ptr [ebp-08]
:00487778 BA02000000    mov edx, 00000002
:0048777D E826C5F7FF    call 00403CA8
:00487782 8D4508        lea eax, dword ptr [ebp+08]
:00487785 E8FAC4F7FF    call 00403C84
:0048778A C3            ret
```

### 注册码示例

最后给出两个正确注册码供大家研究：

#### 第一种

*   **用户名**：小虾
*   **注册码**：`ABC1-FG1I-K1NM-NDTJ`

#### 第二种

*   **用户名**：小虾
*   **注册码**：`214065-811039690 4dd466d0e1e0cc123afbe504da2462ca`
