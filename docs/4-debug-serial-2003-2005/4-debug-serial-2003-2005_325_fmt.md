
# 仓管王 V12.7 单机版

> 原文：[`www.pediy.com/kssd/pediy05/pediy50657.htm`](https://www.pediy.com/kssd/pediy05/pediy50657.htm)

下载页面：[`www.shd.com.cn/software/download.asp?id=28`](http://www.shd.com.cn/software/download.asp?id=28)

软件大小：4399 KB

软件语言：简体中文

软件类别：国产软件 / 共享版 / 商业贸易

应用平台：Win9x/NT/2000/XP

加入时间：2003-04

下载次数：15571

推荐等级：\*\*\*\*

开发商：[`www.shd.com.cn/`](http://www.shd.com.cn/)

## 软件简介

完全满足企事业、行政单位的仓库、财产、物资管理的要求。可选择金额、数量记帐法。除设有基本的入库单、出库单、调拨单、报废单、盘点单外，尚有功能强大的帐单导入、单据修改、单据撤销、单据审批、分类统计等全自动的统计功能，是同类产品中功能最强、价格最低的优秀产品。

## 软件限制

功能限制

## 作者声明

初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

## 破解工具

`TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`AspackDie`、`W32Dasm 9.0` 白金版

---

## 过 程

朋友刚刚装了宽带，却离我这儿有几公里远，我过去用 2 块雪糕 2 瓶啤酒外加“下不为例”的“保证”，终于蹭了会儿宽带用。哎，没宽带的人就是受“压迫”呀。呵呵，其实朋友是好心的，不同意我把所有休息娱乐的时间都用在破解上。

好了，言归正传吧。有朋友问这个东东的算法，我看到 `lordor` 兄有篇 `V12.6` 的笔记，于是照着算了一下，谁知告诉我“注册号错误”，我不得不亮出我的兵刃了。^O^^O^ 分析完了发现竟然比 `V12.6` 还简单。

`CG2000.exe` 是 `ASPack 2.11` 壳，用 `AspackDie` 脱之，1.21M->4.72M。`Delphi 6.0` 编写。

软件分为 3 个版本，就以“标准版”来分析吧。

系列号：`223064214258`

试炼码：`1234-5678-9012-3456`

---

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0065255F(C)

|

```
:0065256B 6A24                  push 00000024
:0065256D 683C286500            push 0065283C
```

\* Possible StringData Ref from Code Obj ->"  您确认接受以上所声明的内容吗？ "

|

```
:00652572 6844286500            push 00652844
:00652577 8BC3                  mov eax, ebx
:00652579 E8AA98E4FF            call 0049BE28
:0065257E 50                    push eax
```

\* Reference To: `user32.MessageBoxA`, Ord:0000h

|

```
:0065257F E89C5FDBFF            Call 00408520
:00652584 83F807                cmp eax, 00000007
:00652587 7505                  jne 0065258E
:00652589 E81EC5DBFF            call 0040EAAC
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00652587(C)

|

```
:0065258E A188276B00            mov eax, dword ptr [006B2788]
:00652593 8B00                  mov eax, dword ptr [eax]
:00652595 FF80A8050000          inc dword ptr [eax+000005A8]
:0065259B 8D55F8                lea edx, dword ptr [ebp-08]
:0065259E 8B834C030000          mov eax, dword ptr [ebx+0000034C]
:006525A4 E8572FE4FF            call 00495500
:006525A9 FF75F8                push [ebp-08]
:006525AC 6870286500            push 00652870
:006525B1 8D55F4                lea edx, dword ptr [ebp-0C]
:006525B4 8B8350030000          mov eax, dword ptr [ebx+00000350]
:006525BA E8412FE4FF            call 00495500
:006525BF FF75F4                push [ebp-0C]
:006525C2 6870286500            push 00652870
:006525C7 8D55F0                lea edx, dword ptr [ebp-10]
:006525CA 8B8354030000          mov eax, dword ptr [ebx+00000354]
:006525D0 E82B2FE4FF            call 00495500
:006525D5 FF75F0                push [ebp-10]
:006525D8 6870286500            push 00652870
:006525DD 8D55EC                lea edx, dword ptr [ebp-14]
:006525E0 8B8358030000          mov eax, dword ptr [ebx+00000358]
:006525E6 E8152FE4FF            call 00495500
:006525EB FF75EC                push [ebp-14]
:006525EE 8D45FC                lea eax, dword ptr [ebp-04]
:006525F1 BA07000000            mov edx, 00000007
:006525F6 E8292CDBFF            call 00405224
:006525FB 8B45FC                mov eax, dword ptr [ebp-04]
```
\* EAX=1234-5678-9012-3456  试炼码

```
:006525FE 50                    push eax
:006525FF 6A00                  push 00000000
:00652601 8D55E8                lea edx, dword ptr [ebp-18]
:00652604 8B8348030000          mov eax, dword ptr [ebx+00000348]
:0065260A E8F12EE4FF            call 00495500
:0065260F 8B4DE8                mov ecx, dword ptr [ebp-18]
```
\* ECX=223064214258   系列号

```
:00652612 66BA0100              mov dx, 0001
:00652616 B87C286500            mov eax, 0065287C
```
\* EAX=33     不知道是否是固定值。

```
:0065261B E82C930400            call 0069B94C
```
\* 关键 CALL！进入！

```
:00652620 84C0                  test al, al
:00652622 7530                  jne 00652654
```
\* 不跳则 OVER！

```
:00652624 6A10                  push 00000010
```

\* Possible StringData Ref from Code Obj ->"错误"

|

```
:00652626 B980286500            mov ecx, 00652880
```

\* Possible StringData Ref from Code Obj ->"  注 册 号 错 误!   "

\* BAD BOY！

```
:0065262B BA88286500            mov edx, 00652888
:00652630 A1D42D6B00            mov eax, dword ptr [006B2DD4]
:00652635 8B00                  mov eax, dword ptr [eax]
:00652637 E84C45E3FF            call 00486B88
:0065263C A190646B00            mov eax, dword ptr [006B6490]
:00652641 8B804C030000          mov eax, dword ptr [eax+0000034C]
:00652647 8B10                  mov edx, dword ptr [eax]
:00652649 FF92C0000000          call dword ptr [edx+000000C0]
:0065264F E9C0010000            jmp 00652814
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00652622(C)

|

```
:00652654 A1DC316B00            mov eax, dword ptr [006B31DC]
:00652659 8B00                  mov eax, dword ptr [eax]
:0065265B 8BB0AC000000          mov esi, dword ptr [eax+000000AC]
:00652661 8BC6                  mov eax, esi
:00652663 E804D5DEFF            call 0043FB6C
:00652668 8BC6                  mov eax, esi
:0065266A E875FBDEFF            call 004421E4
:0065266F 8BC6                  mov eax, esi
:00652671 E87AFFDEFF            call 004425F0
```

\* Possible StringData Ref from Code Obj ->"ZCZC"

|

```
:00652676 BAA8286500            mov edx, 006528A8
:0065267B 8BC6                  mov eax, esi
:0065267D E89EE7DEFF            call 00440E20
:00652682 B201                  mov dl, 01
:00652684 8B08                  mov ecx, dword ptr [eax]
:00652686 FF9194000000          call dword ptr [ecx+00000094]
```

\* Possible StringData Ref from Code Obj ->"YHDM1"

|

```
:0065268C BAB8286500            mov edx, 006528B8
:00652691 8BC6                  mov eax, esi
:00652693 E888E7DEFF            call 00440E20
:00652698 33D2                  xor edx, edx
:0065269A 8B08                  mov ecx, dword ptr [eax]
:0065269C FF91B0000000          call dword ptr [ecx+000000B0]
:006526A2 BAC8286500            mov edx, 006528C8
:006526A7 8BC6                  mov eax, esi
:006526A9 E872E7DEFF            call 00440E20
:006526AE B201                  mov dl, 01
:006526B0 8B08                  mov ecx, dword ptr [eax]
:006526B2 FF9194000000          call dword ptr [ecx+00000094]
:006526B8 8D55E4                lea edx, dword ptr [ebp-1C]
:006526BB 8B8348030000          mov eax, dword ptr [ebx+00000348]
:006526C1 E83A2EE4FF            call 00495500
:006526C6 8B45E4                mov eax, dword ptr [ebp-1C]
:006526C9 50                    push eax
:006526CA BAD4286500            mov edx, 006528D4
:006526CF 8BC6                  mov eax, esi
:006526D1 E84AE7DEFF            call 00440E20
:006526D6 5A                    pop edx
:006526D7 8B08                  mov ecx, dword ptr [eax]
:006526D9 FF91B0000000          call dword ptr [ecx+000000B0]
:006526DF 8D55DC                lea edx, dword ptr [ebp-24]
:006526E2 A190646B00            mov eax, dword ptr [006B6490]
```



```
:006526E7 8B804C030000    mov eax, dword ptr [eax+0000034C]  

:006526ED E80E2EE4FF        call 00495500  

:006526F2 FF75DC             push [ebp-24]  

:006526F5 6870286500        push 00652870  

:006526FA 8D55D8             lea edx, dword ptr [ebp-28]  

:006526FD A190646B00        mov eax, dword ptr [006B6490]  

:00652702 8B8050030000    mov eax, dword ptr [eax+00000350]  

:00652708 E8F32DE4FF        call 00495500  

:0065270D FF75D8             push [ebp-28]  

:00652710 6870286500        push 00652870  

:00652715 8D55D4             lea edx, dword ptr [ebp-2C]  

:00652718 A190646B00        mov eax, dword ptr [006B6490]  

:0065271D 8B8054030000    mov eax, dword ptr [eax+00000354]  

:00652723 E8D82DE4FF        call 00495500  

:00652728 FF75D4             push [ebp-2C]  

:0065272B 6870286500        push 00652870  

:00652730 8D55D0             lea edx, dword ptr [ebp-30]  

:00652733 A190646B00        mov eax, dword ptr [006B6490]  

:00652738 8B8058030000    mov eax, dword ptr [eax+00000358]  

:0065273E E8BD2DE4FF        call 00495500  

:00652743 FF75D0             push [ebp-30]  

:00652746 8D45E0             lea eax, dword ptr [ebp-20]  

:00652749 BA07000000        mov edx, 00000007  

:0065274E E8D12ADBFF        call 00405224  

:00652753 8B45E0             mov eax, dword ptr [ebp-20]  

:00652756 50                 push eax  

:00652757 BAE0286500        mov edx, 006528E0  

:0065275C 8BC6               mov eax, esi  

:0065275E E8BDE6DEFF        call 00440E20  

:00652763 5A                 pop edx  

:00652764 8B08               mov ecx, dword ptr [eax]  

:00652766 FF91B0000000    call dword ptr [ecx+000000B0]  

:0065276C 8BC6               mov eax, esi  

:0065276E 8B10               mov edx, dword ptr [eax]  

:00652770 FF9248020000    call dword ptr [edx+00000248]  

:00652776 8BC6               mov eax, esi  

:00652778 E8FBD3DEFF        call 0043FB78  

:0065277D 6A24               push 00000024
```

`* Possible StringData Ref from Code Obj -> "恭喜您!"`  

`|`  

```
:0065277F B9E4286500        mov ecx, 006528E4
```

`* Possible StringData Ref from Code Obj -> "  注 册 成 功 "`  

`====>呵呵，胜利女神！`

```
:00652784 BAEC286500        mov edx, 006528EC  

:00652789 A1D42D6B00        mov eax, dword ptr [006B2DD4]  

:0065278E 8B00               mov eax, dword ptr [eax]  

:00652790 E8F343E3FF        call 00486B88  

:00652795 83F806             cmp eax, 00000006  

:00652798 755D               jne 006527F7  

:0065279A E85D800400        call 0069A7FC  

:0065279F A188276B00        mov eax, dword ptr [006B2788]  

:006527A4 8B00               mov eax, dword ptr [eax]  

:006527A6 C6807B06000001    mov byte ptr [eax+0000067B], 01  

:006527AD A1942C6B00        mov eax, dword ptr [006B2C94]  

:006527B2 833800             cmp dword ptr [eax], 00000000  

:006527B5 751C               jne 006527D3  

:006527B7 8B0DD42D6B00      mov ecx, dword ptr [006B2DD4]  

:006527BD 8B09               mov ecx, dword ptr [ecx]  

:006527BF B201               mov dl, 01
```

`* Possible StringData Ref from Code Obj -> "@蒊"`  

`|`  

```
:006527C1 A154AE6400        mov eax, dword ptr [0064AE54]  

:006527C6 E85DCAE2FF        call 0047F228  

:006527CB 8B15942C6B00      mov edx, dword ptr [006B2C94]  

:006527D1 8902               mov dword ptr [edx], eax
```

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`  

`|:006527B5(C)`  

`|`  

```
:006527D3 A1942C6B00        mov eax, dword ptr [006B2C94]  

:006527D8 8B00               mov eax, dword ptr [eax]  

:006527DA 8B10               mov edx, dword ptr [eax]  

:006527DC FF92E8000000    call dword ptr [edx+000000E8]  

:006527E2 A1942C6B00        mov eax, dword ptr [006B2C94]  

:006527E7 8B00               mov eax, dword ptr [eax]  

:006527E9 E80A0DE3FF        call 004834F8  

:006527EE A1942C6B00        mov eax, dword ptr [006B2C94]  

:006527F3 33D2               xor edx, edx  

:006527F5 8910               mov dword ptr [eax], edx
```

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`  

`|:00652798(C)`  

`|`  

```
:006527F7 A190646B00        mov eax, dword ptr [006B6490]  

:006527FC E8BB0AE3FF        call 004832BC  

:00652801 A188276B00        mov eax, dword ptr [006B2788]  

:00652806 8B00               mov eax, dword ptr [eax]  

:00652808 C6809905000001    mov byte ptr [eax+00000599], 01  

:0065280F E80CAA0400        call 0069D220
```

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`  

`|:0065264F(U)`  

`|`  

```
:00652814 33C0               xor eax, eax  

:00652816 5A                 pop edx  

:00652817 59                 pop ecx  

:00652818 59                 pop ecx  

:00652819 648910             mov dword ptr fs:[eax], edx  

:0065281C 6836286500        push 00652836
```

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`  

`|:00652834(U)`  

`|`  

```
:00652821 8D45D0             lea eax, dword ptr [ebp-30]  

:00652824 BA0C000000        mov edx, 0000000C  

:00652829 E8A226DBFF        call 00404ED0  

:0065282E C3                 ret
```

—————————————————————————————————

进入关键 `CALL`：`0065261B call 0069B94C`

`* Referenced by a CALL at Addresses:`  

`|:0065261B   , :00652F4A   , :00653442   , :0069BD99`  

`|`  

```
:0069B94C 55                 push ebp  

:0069B94D 8BEC               mov ebp, esp  

:0069B94F 51                 push ecx  

:0069B950 B906000000        mov ecx, 00000006
```

`====>下面几个大、小循环有点晕人呀。跳下去看就明白了。`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`  

`|:0069B95A(C)`  

`|`  

```
:0069B955 6A00               push 00000000  

:0069B957 6A00               push 00000000  

:0069B959 49                 dec ecx  

:0069B95A 75F9               jne 0069B955  

:0069B95C 51                 push ecx  

:0069B95D 874DFC             xchg dword ptr [ebp-04], ecx  

:0069B960 53                 push ebx  

:0069B961 56                 push esi  

:0069B962 57                 push edi  

:0069B963 894DF8             mov dword ptr [ebp-08], ecx  

:0069B966 8BFA               mov edi, edx  

:0069B968 8945FC             mov dword ptr [ebp-04], eax  

:0069B96B 8B45FC             mov eax, dword ptr [ebp-04]  

:0069B96E E8D999D6FF        call 0040534C  

:0069B973 8B45F8             mov eax, dword ptr [ebp-08]  

:0069B976 E8D199D6FF        call 0040534C  

:0069B97B 8B450C             mov eax, dword ptr [ebp+0C]  

:0069B97E E8C999D6FF        call 0040534C  

:0069B983 33C0               xor eax, eax  

:0069B985 55                 push ebp  

:0069B986 6833BC6900        push 0069BC33  

:0069B98B 64FF30             push dword ptr fs:[eax]  

:0069B98E 648920             mov dword ptr fs:[eax], esp  

:0069B991 C645F700           mov [ebp-09], 00  

:0069B995 33C0               xor eax, eax  

:0069B997 55                 push ebp  

:0069B998 68F9BB6900        push 0069BBF9  

:0069B99D 64FF30             push dword ptr fs:[eax]  

:0069B9A0 648920             mov dword ptr fs:[eax], esp  

:0069B9A3 66BE0100           mov si, 0001
```

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`  

`|:0069BA80(C)`  

`|`



# 汇编代码段：0069B9A7 至 0069BB3B

```
:0069B9A7 8D45EC          lea eax, dword ptr [ebp-14h]
:0069B9AA 50              push eax
:0069B9AB 0FB7D6          movzx edx, si
:0069B9AE B901000000      mov ecx, 00000001
:0069B9B3 8B450C          mov eax, dword ptr [ebp+0C]
:0069B9B6 E8019AD6FF      call 004053BC
:0069B9BB 8B45EC          mov eax, dword ptr [ebp-14h]
:0069B9BE BA50BC6900      mov edx, 0069BC50
:0069B9C3 E8E098D6FF      call 004052A8
:0069B9C8 7512            jne 0069B9DC

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :0069B9C8(C)

:0069B9DC 8B45EC          mov eax, dword ptr [ebp-14h]
:0069B9DF BA68BC6900      mov edx, 0069BC68
:0069B9E4 E8BF98D6FF      call 004052A8
:0069B9E9 750F            jne 0069B9FA

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :0069B9E9(C)

:0069B9FA 8B45EC          mov eax, dword ptr [ebp-14h]
:0069B9FD BA80BC6900      mov edx, 0069BC80
:0069BA02 E8A198D6FF      call 004052A8
:0069BA07 750F            jne 0069BA18

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :0069BA07(C)

:0069BA18 8B45EC          mov eax, dword ptr [ebp-14h]
:0069BA1B BA98BC6900      mov edx, 0069BC98
:0069BA20 E88398D6FF      call 004052A8
:0069BA25 750F            jne 0069BA36

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :0069BA25(C)

:0069BA36 8B45EC          mov eax, dword ptr [ebp-14h]
:0069BA39 BAB0BC6900      mov edx, 0069BCB0
:0069BA3E E86598D6FF      call 004052A8
:0069BA43 750F            jne 0069BA54

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :0069BA43(C)

:0069BA54 8B45EC          mov eax, dword ptr [ebp-14h]
:0069BA57 BAC8BC6900      mov edx, 0069BCC8
:0069BA5C E84798D6FF      call 004052A8
:0069BA61 750D            jne 0069BA70

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
| :0069B9D7(U), :0069B9F8(U), :0069BA16(U), :0069BA34(U), :0069BA52(U)
| :0069BA61(C)

:0069BA70 8D45E8          lea eax, dword ptr [ebp-18h]
:0069BA73 8B55EC          mov edx, dword ptr [ebp-14h]
:0069BA76 E8F196D6FF      call 0040516C
:0069BA7B 46              inc esi
:0069BA7C 6683FE14        cmp si, 0014
:0069BA80 0F8521FFFFFF    jne 0069B9A7
:0069BA86 8D45F0          lea eax, dword ptr [ebp-10h]
:0069BA89 E81E94D6FF      call 00404EAC
:0069BA8E 8D45EC          lea eax, dword ptr [ebp-14h]
:0069BA91 E81694D6FF      call 00404EAC
:0069BA96 66BE0100        mov si, 0001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :0069BAF3(C)

:0069BA9A 66BB0100        mov bx, 0001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :0069BAEC(C)

:0069BA9E 8D45E0          lea eax, dword ptr [ebp-20h]
:0069BAA1 50              push eax
:0069BAA2 0FB7C6          movzx eax, si
:0069BAA5 8D1480          lea edx, dword ptr [eax+4*eax]
:0069BAA8 83EA04          sub edx, 00000004
:0069BAAB 0FB7C3          movzx eax, bx
:0069BAAE 48              dec eax
:0069BAAF 03D0            add edx, eax
:0069BAB1 B901000000      mov ecx, 00000001
:0069BAB6 8B45E8          mov eax, dword ptr [ebp-18h]
:0069BAB9 E8FE98D6FF      call 004053BC
:0069BABE 8B45E0          mov eax, dword ptr [ebp-20h]
:0069BAC1 E896EED6FF      call 0040A95C
:0069BAC6 50              push eax
:0069BAC7 B809000000      mov eax, 00000009
                      ==== > EAX=9
:0069BACC 5A              pop edx
:0069BACD 2BC2            sub eax, edx
                      ==== > EAX=9 依次减去试炼码的数字值
:0069BACF 99              cdq
:0069BAD0 33C2            xor eax, edx
:0069BAD2 2BC2            sub eax, edx
:0069BAD4 8D55E4          lea edx, dword ptr [ebp-1Ch]
:0069BAD7 E8E0EDD6FF      call 0040A8BC
:0069BADC 8B55E4          mov edx, dword ptr [ebp-1Ch]
:0069BADF 8D45F0          lea eax, dword ptr [ebp-10h]
:0069BAE2 E88596D6FF      call 0040516C
:0069BAE7 43              inc ebx
:0069BAE8 6683FB05        cmp bx, 0005
:0069BAEC 75B0            jne 0069BA9E
:0069BAEE 46              inc esi
:0069BAEF 6683FE05        cmp si, 0005
:0069BAF3 75A5            jne 0069BA9A
                      ==== > 这几个大小循环有点让人花眼，其实只是相当于取试炼码的数字，
                      然后依次直接用 9 去减，1234-5678-9012-3456 得出 8765432109876543 呵呵，我再这儿浪费了 20 分钟呀。
:0069BAF5 66FFCF          dec di
:0069BAF8 740C            je 0069BB06
:0069BAFA 4F              dec edi
:0069BAFB 6683EF02        sub di, 0002
:0069BAFF 727E            jb 0069BB7F
:0069BB01 E9E9000000      jmp 0069BBEF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :0069BAF8(C)

:0069BB06 8D45DC          lea eax, dword ptr [ebp-24h]
:0069BB09 50              push eax
:0069BB0A B90C000000      mov ecx, 0000000C
:0069BB0F BA01000000      mov edx, 00000001
:0069BB14 8B45F0          mov eax, dword ptr [ebp-10h]
                      ==== > EAX=8765432109876543 试炼码运算的结果。
:0069BB17 E8A098D6FF      call 004053BC
                      ==== > 取 8765432109876543 的前 12 位
:0069BB1C 8B45DC          mov eax, dword ptr [ebp-24h]
                      ==== > EAX=876543210987
:0069BB1F 8B55F8          mov edx, dword ptr [ebp-08h]
                      ==== > EDX=223064214258 系列号
:0069BB22 E88197D6FF      call 004052A8
                      ==== > 比较前 12 位是否和系列号相等？！
:0069BB27 0F85C2000000    jne 0069BBEF
                      ==== > 跳则 OVER！
:0069BB2D 8D45D8          lea eax, dword ptr [ebp-28h]
:0069BB30 50              push eax
:0069BB31 B902000000      mov ecx, 00000002
:0069BB36 BA0D000000      mov edx, 0000000D
:0069BB3B 8B45F0          mov eax, dword ptr [ebp-10h]
```



# 逆向分析

:0069BB3E `E87998D6FF`          `call 004053BC`
==> EAX=`8765432109876543`

:0069BB43 `8B45D8`             `mov eax, dword ptr [ebp-28]`
==> 取 `8765432109876543` 的 13、14 位

:0069BB46 `8B55FC`             `mov edx, dword ptr [ebp-04]`
==> EAX=`65`

:0069BB49 `E85A97D6FF`        `call 004052A8`
==> 比较第 13、14 位是否和 `33` 相等？

:0069BB4E `0F859B000000`      `jne 0069BBEF`
==> 跳则 OVER！

:0069BB54 `8D45D4`            `lea eax, dword ptr [ebp-2C]`
:0069BB57 `50`                `push eax`
:0069BB58 `B902000000`        `mov ecx, 00000002`
:0069BB5D `BA0F000000`        `mov edx, 0000000F`
:0069BB62 `8B45F0`            `mov eax, dword ptr [ebp-10]`
==> EAX=`8765432109876543`

:0069BB65 `E85298D6FF`        `call 004053BC`
==> 取 `8765432109876543` 的最后 2 位

:0069BB6A `8B45D4`            `mov eax, dword ptr [ebp-2C]`
==> EAX=`43`

:0069BB6D `BAE0BC6900`        `mov edx, 0069BCE0`
==> EDX=`28` （不确定是否是固定数）

:0069BB72 `E83197D6FF`        `call 004052A8`
==> 比较最后 2 位是否和 `28` 相等？

:0069BB77 `7576`              `jne 0069BBEF`
==> 跳则 OVER！

:0069BB79 `C645F701`          `mov [ebp-09], 01`
==> 置 1 则 OK！

:0069BB7D `EB70`              `jmp 0069BBEF`

> * Referenced by a (U)nconditional or (C)onditional Jump at Address:
> | :0069BAFF(C)

:0069BB7F `8D45D0`            `lea eax, dword ptr [ebp-30]`
:0069BB82 `50`                `push eax`
:0069BB83 `B90C000000`        `mov ecx, 0000000C`
:0069BB88 `BA01000000`        `mov edx, 00000001`
:0069BB8D `8B45F0`            `mov eax, dword ptr [ebp-10]`
:0069BB90 `E82798D6FF`        `call 004053BC`
:0069BB95 `8B45D0`            `mov eax, dword ptr [ebp-30]`
:0069BB98 `8B55F8`            `mov edx, dword ptr [ebp-08]`
:0069BB9B `E80897D6FF`        `call 004052A8`
:0069BBA0 `754D`              `jne 0069BBEF`
:0069BBA2 `8D45CC`            `lea eax, dword ptr [ebp-34]`
:0069BBA5 `50`                `push eax`
:0069BBA6 `B902000000`        `mov ecx, 00000002`
:0069BBAB `BA0D000000`        `mov edx, 0000000D`
:0069BBB0 `8B45F0`            `mov eax, dword ptr [ebp-10]`
:0069BBB3 `E80498D6FF`        `call 004053BC`
:0069BBB8 `8B45CC`            `mov eax, dword ptr [ebp-34]`
:0069BBBB `8B55FC`            `mov edx, dword ptr [ebp-04]`
:0069BBBE `E8E596D6FF`        `call 004052A8`
:0069BBC3 `752A`              `jne 0069BBEF`
:0069BBC5 `8D45C8`            `lea eax, dword ptr [ebp-38]`
:0069BBC8 `50`                `push eax`
:0069BBC9 `B902000000`        `mov ecx, 00000002`
:0069BBCE `BA0F000000`        `mov edx, 0000000F`
:0069BBD3 `8B45F0`            `mov eax, dword ptr [ebp-10]`
:0069BBD6 `E8E197D6FF`        `call 004053BC`
:0069BBDB `8B45C8`            `mov eax, dword ptr [ebp-38]`
:0069BBDE `E879EDD6FF`        `call 0040A95C`
:0069BBE3 `0FB75508`          `movzx edx, word ptr [ebp+08]`
:0069BBE7 `3BC2`              `cmp eax, edx`
:0069BBE9 `7504`              `jne 0069BBEF`
:0069BBEB `C645F701`          `mov [ebp-09], 01`

> * Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
> | :0069BB01(U), :0069BB27(C), :0069BB4E(C), :0069BB77(C), :0069BB7D(U)
> | :0069BBA0(C), :0069BBC3(C), :0069BBE9(C)

:0069BBEF `33C0`              `xor eax, eax`
:0069BBF1 `5A`                `pop edx`
:0069BBF2 `59`                `pop ecx`
:0069BBF3 `59`                `pop ecx`
:0069BBF4 `648910`            `mov dword ptr fs:[eax], edx`
:0069BBF7 `EB0A`              `jmp 0069BC03`
:0069BBF9 `E9BE88D6FF`        `jmp 004044BC`
:0069BBFE `E8E58CD6FF`        `call 004048E8`

> * Referenced by a (U)nconditional or (C)onditional Jump at Address:
> | :0069BBF7(U)

:0069BC03 `33C0`              `xor eax, eax`
==> 清 0 则 OVER！

:0069BC05 `5A`                `pop edx`
:0069BC06 `59`                `pop ecx`
:0069BC07 `59`                `pop ecx`
:0069BC08 `648910`            `mov dword ptr fs:[eax], edx`
:0069BC0B `683ABC6900`        `push 0069BC3A`

> * Referenced by a (U)nconditional or (C)onditional Jump at Address:
> | :0069BC38(U)

:0069BC10 `8D45C8`            `lea eax, dword ptr [ebp-38]`
:0069BC13 `BA0B000000`        `mov edx, 0000000B`
:0069BC18 `E8B392D6FF`        `call 00404ED0`
:0069BC1D `8D45F8`            `lea eax, dword ptr [ebp-08]`
:0069BC20 `BA02000000`        `mov edx, 00000002`
:0069BC25 `E8A692D6FF`        `call 00404ED0`
:0069BC2A `8D450C`            `lea eax, dword ptr [ebp+0C]`
:0069BC2D `E87A92D6FF`        `call 00404EAC`
:0069BC32 `C3`                `ret`

:0069BC33 `E9388BD6FF`        `jmp 00404770`
:0069BC38 `EBD6`              `jmp 0069BC10`
:0069BC3A `8A45F7`            `mov al, byte ptr [ebp-09]`
==> `[ebp-09]` 的值入 `AL`

:0069BC3D `5F`                `pop edi`
:0069BC3E `5E`                `pop esi`
:0069BC3F `5B`                `pop ebx`
:0069BC40 `8BE5`              `mov esp, ebp`
:0069BC42 `5D`                `pop ebp`
:0069BC43 `C20800`            `ret 0008`

## 算法总结

算法很简单。求逆如下：

1. 注册码的格式如：`1234-5678-9012-3456`，去除 `-` 后共实际输入 16 位。
2. 前 12 位应是 `9` 逐位系列号所得的数字，`223064214258` -> `776935785741`。
3. 第 13、14 位是 `9` 逐位 - `3`、`3` = `66`。
4. 第 15、16 位是 `9` 逐位 - `2`、`8` = `71`。

重新组合起来就是：`7769-3578-5741-6671`。

呵呵，不知道 `33` 和 `28` 是否是固定值了。哪位朋友做了麻烦告诉我一声呀。

## 完美爆破

```asm
0069BC3A 8A45F7             mov al, byte ptr [ebp-09]
```

改为：

```asm
B00190                   mov al, 01      补一个 NOP
```

呵呵，让其永远返回 1，岂有不 OK 的？！程序已然自动保存好注册信息了！

## 注册信息保存

主程序目录下的 `/DATA` 文件夹下的 `demo.DB` 文件中。

## 整理

系列号：`223064214258`

注册码：`7769-3578-5741-6671`

---

```text
        ,     \_/      
      /| \_.-~/        \_     ,       青春都一饷
      ( /~   /           \~-.\ |
      `\\  /_/             \   ~\ )       忍把浮名
  _-~~~-.)  )\_\_/;;,.      \_  //
  /'\_,\   --~   \ ~~~-  ,;;\_\_(  (.-~~~-.       换了破解轻狂
 `~ \_( ,_..--\ (     ,;'' /   ~--  /.\`\\
  /~~//'   /' \`~\        ) /--..\_, )_  `~
  "  `~"  "      `"      /~'\`\   `\~~\ 

            "     "   "~'  ""

        Cracked By 巢水工作坊——fly [OCN][FCG]
                2003-04-27  4:00  4dd466d0e1e0cc123afbe504da2462ca
```
