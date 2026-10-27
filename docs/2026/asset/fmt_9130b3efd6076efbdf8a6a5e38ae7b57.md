
# 一个典型的破狗例子，送给PPP307做参考，同时宣布我回家修炼，愿BCG和CCG发扬光大，并保留我的荣誉地位

(12千字)

前几天PPP307送给我一个路桥CAD软件(`XCHT14`)，说实话，软件编制的不是很好，但是他体现了初学者的crack思路，和忽视的方面。在这里把他补上，总共改动4个字节就收工了。

主程序：`newxcht.arx`

读狗主控程序：`TDMD.vxd`（不用我说，大家也知道是什么狗了）

工具：`softICE4.0`、`w32dasm`

反编译后可以找到以下字串，不过ppp307为了表示自己的恒心和毅力，把它都改掉了。

由以下代码可以知道，调用`10006520`的共17处。但是我们没有必要path每一处代码，只要看程序从这个`call`返回了什么就行了。通常我们知道早期的狗都是返回一个标志。而这个程序非常聪明的返回了一个标志并且携带了一个种子。

```
* Referenced by a CALL at Addresses:
|:100084C9  , :1000AABC  , :1000B091  , :1000BD46  , :1000C704
|:1000E15A  , :1000E8DA  , :100151FC  , :1001A680  , :1001A7AC
|:1001AA71  , :1001B5BB  , :1001DF45  , :10021FE2  , :10022CC1
|:10023074  , :10023154
|
:10006520 83EC40                sub esp, 00000040
:10006523 B907000000            mov ecx, 00000007
:10006528 56                    push esi
:10006529 57                    push edi
* Possible StringData Ref from Data Obj ->"我要解密！ 我要解密！！！ 1974"
            |
:1000652A BE84190310            mov esi, 10031984
:1000652F 8D7C2408              lea edi, dword ptr [esp+08]
:10006533 F3                    repz
:10006534 A5                    movsd
:10006535 66A5                  movsw
:10006537 A4                    movsb
* Possible StringData Ref from Data Obj ->"我要解密！ 我要解密！！！ 1972"
            |
:10006538 BE64190310            mov esi, 10031964
:1000653D 8D7C2428              lea edi, dword ptr [esp+28]
:10006541 B907000000            mov ecx, 00000007
:10006546 F3                    repz
:10006547 A5                    movsd
:10006548 66A5                  movsw
:1000654A A4                    movsb
:1000654B C705E4500310E9F40000  mov dword ptr [100350E4], 0000F4E9
:10006555 C705EC50031000000000  mov dword ptr [100350EC], 00000000
:1000655F C705E85003101E000000  mov dword ptr [100350E8], 0000001E
:10006569 8D442408              lea eax, dword ptr [esp+08]
:1000656D A3E0500310            mov dword ptr [100350E0], eax
:10006572 E8C9D3FFFF            call 10003940------------->进入狗调用
:10006577 85C0                  test eax, eax
:10006579 740D                  je 10006588
------------------------------------------------------------------
:10006572 EB38                  jmp 100065AC
:10006574 D3FF                  sar edi, cl
:10006576 FF85C0740D68          inc dword ptr [ebp+680D74C0]
:1000657C 50                    push eax
------------------------------------------------------------------ 我改动的地方
* Possible StringData Ref from Data Obj ->"
Dog reading error:"
            |
:1000657B 6850190310            push 10031950
* Reference To: ACAD.acrx_abort, Ord:0D5Dh
            |
:10006580 E8D1D30100            Call 10023956
:10006585 83C404                add esp, 00000004
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10006579(C)
|
:10006588 8D442408              lea eax, dword ptr [esp+08]
:1000658C 8D4C2428              lea ecx, dword ptr [esp+28]
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:100065AA(C)
|
:10006590 8A10                  mov dl, byte ptr [eax]
:10006592 3A11                  cmp dl, byte ptr [ecx]
:10006594 751A                  jne 100065B0
:10006596 0AD2                  or dl, dl
:10006598 7412                  je 100065AC
:1000659A 8A5001                mov dl, byte ptr [eax+01]
:1000659D 3A5101                cmp dl, byte ptr [ecx+01]
:100065A0 750E                  jne 100065B0
:100065A2 83C002                add eax, 00000002
:100065A5 83C102                add ecx, 00000002
:100065A8 0AD2                  or dl, dl
:100065AA 75E4                  jne 10006590
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10006598(C)
|
:100065AC 33C0                  xor eax, eax
:100065AE EB05                  jmp 100065B5
-------------------------------------------------------------------
:100065AC 33C0                  xor eax, eax          我改动的地方
:100065AE EB47                  jmp 100065F7
-------------------------------------------------------------------
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:10006594(C), :100065A0(C)
|
:100065B0 1BC0                  sbb eax, eax
:100065B2 83D8FF                sbb eax, FFFFFFFF
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:100065AE(U)
|
:100065B5 85C0                  test eax, eax
:100065B7 740D                  je 100065C6
* Possible StringData Ref from Data Obj ->"
error"
            |
:100065B9 6848190310            push 10031948
* Reference To: ACAD.acrx_abort, Ord:0D5Dh
            |
:100065BE E893D30100            Call 10023956
:100065C3 83C404                add esp, 00000004
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:100065B7(C)
|
:100065C6 E815C4FFFF            call 100029E0
:100065CB 8BD0                  mov edx, eax
:100065CD 8B7C244C              mov edi, dword ptr [esp+4C]
:100065D1 B9FFFFFFFF            mov ecx, FFFFFFFF
:100065D6 2BC0                  sub eax, eax
:100065D8 893DE0500310          mov dword ptr [100350E0], edi
:100065DE F2                    repnz
:100065DF AE                    scasb
:100065E0 F7D1                  not ecx
:100065E2 49                    dec ecx
:100065E3 85D2                  test edx, edx
:100065E5 890DE8500310          mov dword ptr [100350E8], ecx
:100065EB 7505                  jne 100065F2
:100065ED E82ECDFFFF            call 10003320
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:100065EB(C)
|
:100065F2 A148160310            mov eax, dword ptr [10031648]
:100065F7 5F                    pop edi
:100065F8 5E                    pop esi
:100065F9 83C440                add esp, 00000040
:100065FC C3                    ret
```

之所以把代码贴的比较长，不过是为了教学方便而已。可以比较清晰看到程序流程。

下面是对 `call 10006520` 的一个主要调用：

```
* Possible StringData Ref from Data Obj ->"s_layer"
            |
:1000B08C 680C130310              push 1003130C
:1000B091 E88AB4FFFF              call 10006520------------------->判断狗
:1000B096 89442414                mov dword ptr [esp+14], eax----->
:1000B09A 83C404                  add esp, 00000004
* Reference To: ACAD.ads_getargs, Ord:0D93h
            |
:1000B09D E890880100              Call 10023932
:1000B0A2 8BD0                    mov edx, eax
:1000B0A4 85D2                    test edx, edx
:1000B0A6 7442                    je 1000B0EA
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000B0E8(C)
|
:1000B0A8 0FBF6A04                movsx ebp, word ptr [edx+04]
:1000B0AC 81FD8D130000            cmp ebp, 0000138D
:1000B0B2 7524                    jne 1000B0D8
:1000B0B4 8B7A06                  mov edi, dword ptr [edx+06]
:1000B0B7 B9FFFFFFFF              mov ecx, FFFFFFFF
:1000B0BC 2BC0                    sub eax, eax
:1000B0BE F2                      repnz
:1000B0BF AE                      scasb
:1000B0C0 F7D1                    not ecx
:1000B0C2 2BF9                    sub edi, ecx
:1000B0C4 8BC1                    mov eax, ecx
:1000B0C6 C1E902                  shr ecx, 02
:1000B0C9 8BF7                    mov esi, edi
:1000B0CB 8D7C2414                lea edi, dword ptr [esp+14]
:1000B0CF F3                      repz
:1000B0D0 A5                      movsd
:1000B0D1 8BC8                    mov ecx, eax
:1000B0D3 83E103                  and ecx, 00000003
:1000B0D6 F3                      repz
:1000B0D7 A4                      movsb
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000B0B2(C)
|
:1000B0D8 81FD8B130000            cmp ebp, 0000138B
:1000B0DE 7504                    jne 1000B0E4
:1000B0E0 0FBF5A06                movsx ebx, word ptr [edx+06]
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000B0DE(C)
|
:1000B0E4 8B12                    mov edx, dword ptr [edx]
:1000B0E6 85D2                    test edx, edx
:1000B0E8 75BE                    jne 1000B0A8
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000B0A6(C)
|
:1000B0EA 8D442414                lea eax, dword ptr [esp+14]
:1000B0EE 6A00                    push 00000000
:1000B0F0 50                      push eax
* Possible StringData Ref from Data Obj ->"layer"
            |
:1000B0F1 6848170310              push 10031748
* Reference To: ACAD.ads_tblsearch, Ord:0DF6h
            |
:1000B0F6 E843880100              Call 1002393E
:1000B0FB 83C40C                  add esp, 0000000C
:1000B0FE 85C0                    test eax, eax
:1000B100 6888130000              push 00001388
:1000B105 6878160310              push 10031678
:1000B10A 688D130000              push 0000138D
:1000B10F 7542                    jne 1000B153
:1000B111 8D442420                lea eax, dword ptr [esp+20]
:1000B115 6878160310              push 10031678
```
