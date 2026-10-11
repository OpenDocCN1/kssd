
# 傲世三国

> 原文：[`www.pediy.com/kssd/pediy03/forum512.htm`](https://www.pediy.com/kssd/pediy03/forum512.htm)

也谈《傲世三国》的暴力破解法

前几天在论坛中看到了一位网友写的关于《傲世三国》的破解手记，正好之前我也刚刚把这个游戏买回家，马不停蹄地把它破了，趁今天有空把破解过程写出来，请各位大虾指点。

安装完游戏，取出光盘，然后执行，会跳出一个提示框，如有光盘，按确定，如无，则只能按取消。将 `EXE` 文件反汇编，查找 `GetDriveTypea`，来到如下代码：

```
* Reference To: KERNEL32.GetDriveTypeA, Ord:0104h

:0048663B FF1524226100 Call dword ptr [00612224]
:00486641 83F805 cmp eax, 00000005
:00486644 0F850C020000 jne 00486856 ----注意这条指令！
:0048664A 8A442414 mov al, byte ptr [esp+14]
:0048664E 84C0 test al, al
:00486650 7525 jne 00486677
:00486652 8D7C2410 lea edi, dword ptr [esp+10]
:00486656 83C9FF or ecx, FFFFFFFF
:00486659 33C0 xor eax, eax
:0048665B 8D542414 lea edx, dword ptr [esp+14]
:0048665F F2 repnz
:00486660 AE scasb
:00486661 F7D1 not ecx
:00486663 2BF9 sub edi, ecx
:00486665 8BC1 mov eax, ecx
:00486667 8BF7 mov esi, edi
:00486669 8BFA mov edi, edx
:0048666B C1E902 shr ecx, 02
:0048666E F3 repz
:0048666F A5 movsd
:00486670 8BC8 mov ecx, eax
:00486672 83E103 and ecx, 00000003
:00486675 F3 repz
:00486676 A4 movsb

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00486650(C)
|
:00486677 8D7C2410 lea edi, dword ptr [esp+10]
:0048667B 83C9FF or ecx, FFFFFFFF
:0048667E 33C0 xor eax, eax
:00486680 8D542418 lea edx, dword ptr [esp+18]
:00486684 F2 repnz
:00486685 AE scasb
:00486686 F7D1 not ecx
:00486688 2BF9 sub edi, ecx
:0048668A 8BC1 mov eax, ecx
:0048668C 8BF7 mov esi, edi
:0048668E 8BFA mov edi, edx
:00486690 8D542418 lea edx, dword ptr [esp+18]
:00486694 C1E902 shr ecx, 02
:00486697 F3 repz
:00486698 A5 movsd
:00486699 8BC8 mov ecx, eax
:0048669B 33C0 xor eax, eax
:0048669D 83E103 and ecx, 00000003
:004866A0 F3 repz
:004866A1 A4 movsb

* Possible StringData Ref from Data Obj ->"\\autorun.inf"
|
:004866A2 BF28F86300 mov edi, 0063F828
:004866A7 83C9FF or ecx, FFFFFFFF
:004866AA F2 repnz
:004866AB AE scasb
:004866AC F7D1 not ecx
:004866AE 2BF9 sub edi, ecx
:004866B0 8BF7 mov esi, edi
:004866B2 8BFA mov edi, edx
:004866B4 8BD1 mov edx, ecx
:004866B6 83C9FF or ecx, FFFFFFFF
:004866B9 F2 repnz
:004866BA AE scasb
:004866BB 8BCA mov ecx, edx
:004866BD 4F dec edi
:004866BE C1E902 shr ecx, 02
:004866C1 F3 repz
:004866C2 A5 movsd
:004866C3 8BCA mov ecx, edx
:004866C5 8D442418 lea eax, dword ptr [esp+18]
:004866C9 83E103 and ecx, 00000003
:004866CC 50 push eax
:004866CD F3 repz
:004866CE A4 movsb
:004866CF 8D8C2420010000 lea ecx, dword ptr [esp+00000120]
:004866D6 6804010000 push 00000104
:004866DB 51 push ecx

* Possible StringData Ref from Data Obj ->"NODISK"
|
:004866DC 6820F86300 push 0063F820

* Possible StringData Ref from Data Obj ->"ObjectKey"
|
:004866E1 6814F86300 push 0063F814

* Possible StringData Ref from Data Obj ->"UI"
|
:004866E6 6810F86300 push 0063F810
:004866EB FFD5 call ebp

* Possible StringData Ref from Data Obj ->"SOFTWARE\\Object Software (Beijng) "
->"Co., Ltd."
|
:004866ED BEE4F76300 mov esi, 0063F7E4
:004866F2 8D84241C010000 lea eax, dword ptr [esp+0000011C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00486717(C)
|
:004866F9 8A10 mov dl, byte ptr [eax]
:004866FB 8ACA mov cl, dl
:004866FD 3A16 cmp dl, byte ptr [esi]
:004866FF 751C jne 0048671D
:00486701 84C9 test cl, cl
:00486703 7414 je 00486719
:00486705 8A5001 mov dl, byte ptr [eax+01]
:00486708 8ACA mov cl, dl
:0048670A 3A5601 cmp dl, byte ptr [esi+01]
:0048670D 750E jne 0048671D
:0048670F 83C002 add eax, 00000002
:00486712 83C602 add esi, 00000002
:00486715 84C9 test cl, cl
:00486717 75E0 jne 004866F9

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00486703(C)
|
:00486719 33C0 xor eax, eax
:0048671B EB05 jmp 00486722

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004866FF(C), :0048670D(C)
|
:0048671D 1BC0 sbb eax, eax
:0048671F 83D8FF sbb eax, FFFFFFFF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048671B(U)
|
:00486722 85C0 test eax, eax
:00486724 0F852C010000 jne 00486856
:0048672A 8D442418 lea eax, dword ptr [esp+18]
:0048672E 8D8C241C010000 lea ecx, dword ptr [esp+0000011C]
:00486735 50 push eax
:00486736 6804010000 push 00000104
:0048673B 51 push ecx

* Possible StringData Ref from Data Obj ->"NODISK"
|
:0048673C 6820F86300 push 0063F820

* Possible StringData Ref from Data Obj ->"BtnNum"
|
:00486741 68DCF76300 push 0063F7DC

* Possible StringData Ref from Data Obj ->"UI"
|
:00486746 6810F86300 push 0063F810
:0048674B FFD5 call ebp

* Possible StringData Ref from Data Obj ->"11"
|
:0048674D BED8F76300 mov esi, 0063F7D8
:00486752 8D84241C010000 lea eax, dword ptr [esp+0000011C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00486777(C)
|
:00486759 8A10 mov dl, byte ptr [eax]
:0048675B 8ACA mov cl, dl
:0048675D 3A16 cmp dl, byte ptr [esi]
:0048675F 751C jne 0048677D
:00486761 84C9 test cl, cl
:00486763 7414 je 00486779
:00486765 8A5001 mov dl, byte ptr [eax+01]
:00486768 8ACA mov cl, dl
:0048676A 3A5601 cmp dl, byte ptr [esi+01]
:0048676D 750E jne 0048677D
:0048676F 83C002 add eax, 00000002
:00486772 83C602 add esi, 00000002
:00486775 84C9 test cl, cl
:00486777 75E0 jne 00486759

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00486763(C)
|
:00486779 33C0 xor eax, eax
:0048677B EB05 jmp 00486782

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0048675F(C), :0048676D(C)
|
:0048677D 1BC0 sbb eax, eax
:0048677F 83D8FF sbb eax, FFFFFFFF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048677B(U)
|
:00486782 85C0 test eax, eax
:00486784 0F85CC000000 jne 00486856
:0048678A 83C9FF or ecx, FFFFFFFF
:0048678D 8D7C2410 lea edi, dword ptr [esp+10]
:00486791 F2 repnz
:00486792 AE scasb
:00486793 F7D1 not ecx
:00486795 2BF9 sub edi, ecx
:00486797 8D542418 lea edx, dword ptr [esp+18]
:0048679B 8BC1 mov eax, ecx
:0048679D 8BF7 mov esi, edi
:0048679F C1E902 shr ecx, 02
:004867A2 8BFA mov edi, edx
:004867A4 8D542418 lea edx, dword ptr [esp+18]
:004867A8 F3 repz
:004867A9 A5 movsd
:004867AA 8BC8 mov ecx, eax
:004867AC 33C0 xor eax, eax
:004867AE 83E103 and ecx, 00000003

* Possible StringData Ref from Data Obj ->"rb"
|
:004867B1 6878F46300 push 0063F478
:004867B6 F3 repz
:004867B7 A4 movsb
:004867B8 83C9FF or ecx, FFFFFFFF

* Possible StringData Ref from Data Obj ->"\\\\"
|
:004867BB BFD4F76300 mov edi, 0063F7D4
:004867C0 F2 repnz
:004867C1 AE scasb
:004867C2 F7D1 not ecx
:004867C4 2BF9 sub edi, ecx
:004867C6 8BF7 mov esi, edi
:004867C8 8BFA mov edi, edx
:004867CA 8BD1 mov edx, ecx
:004867CC 83C9FF or ecx, FFFFFFFF
:004867CF F2 repnz
:004867D0 AE scasb
:004867D1 8BCA mov ecx, edx
:004867D3 4F dec edi
:004867D4 C1E902 shr ecx, 02
:004867D7 F3 repz
:004867D8 A5 movsd
:004867D9 8BCA mov ecx, edx
:004867DB A184358800 mov eax, dword ptr [00883584]
:004867E0 83E103 and ecx, 00000003
:004867E3 8D54241C lea edx, dword ptr [esp+1C]
:004867E7 F3 repz
:004867E8 A4 movsb
:004867E9 8B3C85E0156400 mov edi, dword ptr [4*eax+006415E0]
:004867F0 83C9FF or ecx, FFFFFFFF
:004867F3 33C0 xor eax, eax
```



```
:004867F5 F2 repnz   

:004867F6 AE scasb   

:004867F7 F7D1 not ecx   

:004867F9 2BF9 sub edi, ecx   

:004867FB 8BF7 mov esi, edi   

:004867FD 8BFA mov edi, edx   

:004867FF 8BD1 mov edx, ecx   

:00486801 83C9FF or ecx, FFFFFFFF   

:00486804 F2 repnz   

:00486805 AE scasb   

:00486806 8BCA mov ecx, edx   

:00486808 4F dec edi   

:00486809 C1E902 shr ecx, 02   

:0048680C F3 repz   

:0048680D A5 movsd   

:0048680E 8BCA mov ecx, edx   

:00486810 8D54241C lea edx, dword ptr [esp+1C]   

:00486814 83E103 and ecx, 00000003   

:00486817 F3 repz   

:00486818 A4 movsb   

* Possible StringData Ref from Data Obj ->"\readme.txt"   
```

```
:00486819 BFC8F76300 mov edi, 0063F7C8   

:0048681E 83C9FF or ecx, FFFFFFFF   

:00486821 F2 repnz   

:00486822 AE scasb   

:00486823 F7D1 not ecx   

:00486825 2BF9 sub edi, ecx   

:00486827 8BF7 mov esi, edi   

:00486829 8BFA mov edi, edx   

:0048682B 8BD1 mov edx, ecx   

:0048682D 83C9FF or ecx, FFFFFFFF   

:00486830 F2 repnz   

:00486831 AE scasb   

:00486832 8BCA mov ecx, edx   

:00486834 4F dec edi   

:00486835 C1E902 shr ecx, 02   

:00486838 F3 repz   

:00486839 A5 movsd   

:0048683A 8BCA mov ecx, edx   

:0048683C 8D44241C lea eax, dword ptr [esp+1C]   

:00486840 83E103 and ecx, 00000003   

:00486843 50 push eax   

:00486844 F3 repz   

:00486845 A4 movsb   

:00486846 E822901600 call 005EF86D   

:0048684B 83C408 add esp, 00000008   

:0048684E 85C0 test eax, eax   

:00486850 0F8588000000 jne 004868DE ----注意这里！   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
```

| `:00486644(C)`, `:00486724(C)`, `:00486784(C)`

```
:00486856 8A442410 mov al, byte ptr [esp+10]   

:0048685A FEC0 inc al   

:0048685C 3C5A cmp al, 5A   

:0048685E 88442410 mov byte ptr [esp+10], al   

:00486862 0F8ECEFDFFFF jle 00486636   

:00486868 8B83C40A0000 mov eax, dword ptr [ebx+00000AC4]   

:0048686E 85C0 test eax, eax   

:00486870 0F8505010000 jne 0048697B   

:00486876 8A442414 mov al, byte ptr [esp+14]   

:0048687A 84C0 test al, al   

:0048687C 0F84D0000000 je 00486952   

:00486882 8D7C2414 lea edi, dword ptr [esp+14]   

:00486886 83C9FF or ecx, FFFFFFFF   

:00486889 33C0 xor eax, eax   

:0048688B 8D93C4090000 lea edx, dword ptr [ebx+000009C4]   

:00486891 F2 repnz   

:00486892 AE scasb   

:00486893 F7D1 not ecx   

:00486895 2BF9 sub edi, ecx   

:00486897 8BC1 mov eax, ecx   

:00486899 8BF7 mov esi, edi   

:0048689B 8BFA mov edi, edx   

:0048689D C1E902 shr ecx, 02   

:004868A0 F3 repz   

:004868A1 A5 movsd   

:004868A2 8BC8 mov ecx, eax   

:004868A4 33C0 xor eax, eax   

:004868A6 83E103 and ecx, 00000003   

:004868A9 F3 repz   

:004868AA A4 movsb   

* Possible StringData Ref from Data Obj ->"\\"   
```

```
:004868AB BFD4F76300 mov edi, 0063F7D4   

:004868B0 83C9FF or ecx, FFFFFFFF   

:004868B3 F2 repnz   

:004868B4 AE scasb   

:004868B5 F7D1 not ecx   

:004868B7 2BF9 sub edi, ecx   

:004868B9 8BF7 mov esi, edi   

:004868BB 8BD9 mov ebx, ecx   

:004868BD 8BFA mov edi, edx   

:004868BF 83C9FF or ecx, FFFFFFFF   

:004868C2 F2 repnz   

:004868C3 AE scasb   

:004868C4 8BCB mov ecx, ebx   

:004868C6 4F dec edi   

:004868C7 C1E902 shr ecx, 02   

:004868CA F3 repz   

:004868CB A5 movsd   

:004868CC 8BCB mov ecx, ebx   

:004868CE 83E10   

.   

.   

.   

:004868DE:----------   

:--------:mov eax,00000001 置光驱检测正确标志！   

：-------：ret (这里我已记不清楚，希望能看懂）   
```

通过分析，只要程序能走到`004868DE`就能通过`CD`检测，好，用`TRW`载入游戏，下`BPX` `getdrivetypea`，便会来到上述代码，发现`cs:00486644` `0F850C020000` `jne` `00486856` 要跳转，一跳就完了，改！不让它跳，没用，反正走不到`cs:004868DE`，搞不好还要死机。睁大眼睛再看，`004868DE`是由`cs:00486850`跳过来的，如能走到这儿也行，想来想去，再看看`00486644:jne` `00486856` 这条指令，反正要跳，就索性让它跳到`00486850`，这不就可以了，于是把`0F850C020000`改为`0F8506020000`，

`OK`，程序很老实地走到了`mov eax,00000001`这条指令，通过了光驱检测，别急，按`F10`再往下走，没几行便会来到一个`Call`，`F10`一带过这个`CALL`，就会跳出一个很恐怖的提示框：程序执行了非法操作，必须立即关闭，怎么办？祭出杀手锏，把它

`NOP`掉，行了，下面再没有陷阱了，我试着玩了一下，没出现问题。其实我知道，这样改是很危险的，容易死机，不过这个游戏可以。

去除游戏的`CD`保护有多种方法，我这属于蛮干！主要是想和大家交流一下，欢迎赐教！

bwkpjq

E-Mail:`bwkpjq@pub.sz.jsinfo.net`

`4dd466d0e1e0cc123afbe504da2462ca`
