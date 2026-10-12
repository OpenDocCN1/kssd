
```
:00662840 8D45B4             lea eax, dword ptr [ebp-4C]
:00662843 50                 push eax
:00662844 8D45C4             lea eax, dword ptr [ebp-3C]
:00662847 50                 push eax
:00662848 53                 push ebx
* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h
:00662849 E8B85ADAFF         Call 00408306
:0066284E 83C40C             add esp, 0000000C
:00662851 FF75D8             push [ebp-28]
* Reference To: MSVBVM60.rtcAnsivalueBstr, Ord:0204h
:00662854 E8615BDAFF         Call 004083BA
:00662859 660345E4           add ax, word ptr [ebp-1C]
    ①、    ==>AX=57  +  00=57
    ②、    ==>AX=41  +  55=96
    ③、    ==>AX=59  +  94=ED
    …… ……省略…… ……
    ⑨、    ==>AX=54  + 266=2BA
:0066285D 6A01               push 00000001
:0066285F 0F80B7000000       jo 0066291C
:00662865 662BC3             sub ax, bx
    ①、    ==>AX=57  -  02=55
    ②、    ==>AX=96  -  02=94
    ③、    ==>AX=ED  -  02=EB
    …… ……省略…… ……
    ⑨、    ==>AX=2BA -  02=2B8
:00662868 0F80AE000000       jo 0066291C
:0066286E 8945E4             mov dword ptr [ebp-1C], eax
    >[ebp-1C]=EAX
:00662871 58                 pop eax
:00662872 6603C7             add ax, di
:00662875 0F80A1000000       jo 0066291C
:0066287B 8BF8               mov edi, eax
:0066287D EB80               jmp 006627FF
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00662803(C)
:0066287F 668B45E4           mov ax, word ptr [ebp-1C]
    ==>AX=2B8     上面的累加运算之和
:00662883 66B91A00           mov cx, 001A
:00662887 6699               cwd
:00662889 66F7F9             idiv cx
    ==>DX=2B8 % 1A=14
:0066288C 6683C241           add dx, 0041
    ==>DX=14 + 41=55
:00662890 0F8086000000       jo 0066291C
:00662896 0FBFC2             movsx eax, dx
:00662899 50                 push eax
:0066289A 8D45C4             lea eax, dword ptr [ebp-3C]
:0066289D 50                 push eax
* Reference To: MSVBVM60.rtcVarBstrFromAnsi, Ord:0260h
:0066289E E8295BDAFF         Call 004083CC
    ==>把上面所得的 45 转变成字符
:006628A3 8D45C4             lea eax, dword ptr [ebp-3C]
:006628A6 50                 push eax
* Reference To: MSVBVM60.__vbaStrVarMove, Ord:0000h
:006628A7 E8CC5ADAFF         Call 00408378
:006628AC 8BD0               mov edx, eax
    ==>EDX=U          这就是 K1 的最后 1 位
————————————————
注：生成程序显示的序列号的最后 1 位：
对 IELKLKHID 运算得出：27F % 1A=F    F + 41=50  即：P
—————————————————————————————————
```

进入算法 `CALL②`：`00662AFA` `call dword ptr [eax+000000D8]`

我改变 `E` 盘序列号测试了几次，发现后段注册码是相同的，或许这是根据单位名来运算的。但是在我机子上安装后的程序单位名是无法改变的，有朋友做的话麻烦验证一下！

…… ……省略…… ……

```
:00662669 50                 push eax
:0066266A 895DA8             mov dword ptr [ebp-58], ebx
* Reference To: MSVBVM60.rtcMidCharVar, Ord:0278h
:0066266D E8425DDAFF         Call 004083B4
    ==>依次取 E8 96 73 4F 46 55 1A 4E 6C 51 F8 53 20 00
:00662672 8D45B8             lea eax, dword ptr [ebp-48]
:00662675 50                 push eax
* Reference To: MSVBVM60.__vbaStrVarMove, Ord:0000h
:00662676 E8FD5CDAFF         Call 00408378
:0066267B 8BD0               mov edx, eax
:0066267D 8D4DDC             lea ecx, dword ptr [ebp-24]
* Reference To: MSVBVM60.__vbaStrMove, Ord:0000h
:00662680 E8BD5CDAFF         Call 00408342
:00662685 8D45B8             lea eax, dword ptr [ebp-48]
:00662688 50                 push eax
:00662689 8D45C8             lea eax, dword ptr [ebp-38]
:0066268C 50                 push eax
:0066268D 56                 push esi
* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h
:0066268E E8735CDAFF         Call 00408306
:00662693 8B45E0             mov eax, dword ptr [ebp-20]
:00662696 83C40C             add esp, 0000000C
:00662699 8945B0             mov dword ptr [ebp-50], eax
:0066269C C745A808000000     mov [ebp-58], 00000008
:006626A3 FF75DC             push [ebp-24]
* Reference To: MSVBVM60.rtcAnsivalueBstr, Ord:0204h
:006626A6 E80F5DDAFF         Call 004083BA
:006626AB 8BC8               mov ecx, eax
* Reference To: MSVBVM60.__vbaI2Abs, Ord:0000h
:006626AD E87E5FDAFF         Call 00408630
:006626B2 6699               cwd
:006626B4 66B91A00           mov cx, 001A
:006626B8 66F7F9             idiv cx
:006626BB 6683C241           add dx, 0041
:006626BF 0F80BF000000       jo 00662784
    ==>上面是求模、相加运算
:006626C5 0FBFC2             movsx eax, dx
:006626C8 50                 push eax
:006626C9 8D45C8             lea eax, dword ptr [ebp-38]
:006626CC 50                 push eax
* Reference To: MSVBVM60.rtcVarBstrFromAnsi, Ord:0260h
:006626CD E8FA5CDAFF         Call 004083CC
    ==>依次把上面所得的 HEX 值转变为所对应的字符
:006626D2 8D45A8             lea eax, dword ptr [ebp-58]
:006626D5 50                 push eax
:006626D6 8D45C8             lea eax, dword ptr [ebp-38]
:006626D9 50                 push eax
:006626DA 8D45B8             lea eax, dword ptr [ebp-48]
:006626DD 50                 push eax
* Reference To: MSVBVM60.__vbaVarCat, Ord:0000h
:006626DE E8E15BDAFF         Call 004082C4
    ==>依次连接所得的字符
:006626E3 50                 push eax
* Reference To: MSVBVM60.__vbaStrVarMove, Ord:0000h
:006626E4 E88F5CDAFF         Call 00408378
:006626E9 8BD0               mov edx, eax
    ==>最后得出：EDX=CNSZNOG   这就是后段注册码 K2
:006626EB 8D4DE0             lea ecx, dword ptr [ebp-20]
* Reference To: MSVBVM60.__vbaStrMove, Ord:0000h
:006626EE E84F5CDAFF         Call 00408342
:006626F3 8D45B8             lea eax, dword ptr [ebp-48]
:006626F6 50                 push eax
:006626F7 8D45C8             lea eax, dword ptr [ebp-38]
:006626FA 50                 push eax
:006626FB 56                 push esi
* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h
:006626FC E8055CDAFF         Call 00408306
:00662701 83C40C             add esp, 0000000C
:00662704 6A01               push 00000001
:00662706 58                 pop eax
:00662707 6603C7             add ax, di
:0066270A 7078               jo 00662784
```
