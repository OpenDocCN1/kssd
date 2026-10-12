
# Vopt99 v4.31

> 原文：[`www.pediy.com/kssd/pediy03/forum638.htm`](https://www.pediy.com/kssd/pediy03/forum638.htm)

Vopt99 v4.31 暴力破解实录(仅供初学者参考)

所需工具：`Softice` 或 `trw2000`，`DASM` 黄金版，`UltraEdit`

`VOPT99` 是一个非常好的磁盘整理软件，我一用就喜欢上它了。

可是，它有 30 天限制，要注册，本人没有`$$$`，只好对不住它了。

`Vopt99` 是用 `VB5` 写的(我向来看见 `VB` 的东东就头晕)，尝试找出注册码，但是，它的注册码保护得很好，我功力又尚浅(成为大侠是没有什么希望了:<)，好了，言归正传。先用 `DASM` 将它反编译，以什么为突破口呢？主界面上不是有这么一句话：“30 day trial:”，一找，嘿，还真找到了：

```text
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004318B7(C)
|
:004318D6 C78560FFFFFF00000000    mov dword ptr [ebp+FFFFFF60], 00000000

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004318D4(U)
|
:004318E0 8B55B4                  mov edx, dword ptr [ebp-4C]
:004318E3 895580                  mov dword ptr [ebp-80], edx

* Possible StringData Ref from Code Obj ->"330 day trial: "
                                    |
:004318E6 68DCF94000              push 0040F9DC
:004318EB 668B45CC                mov ax, word ptr [ebp-34]
:004318EF 50                      push eax

* Reference To: MSVBVM50.__vbaStrI2, Ord:0000h
                                    |
:004318F0 FF159CD24500            Call dword ptr [0045D29C]
:004318F6 8BD0                    mov edx, eax
:004318F8 8D4DC4                  lea ecx, dword ptr [ebp-3C]

* Reference To: MSVBVM50.__vbaStrMove, Ord:0000h
                                    |
:004318FB FF15C4D44500            Call dword ptr [0045D4C4]
:00431901 50                      push eax

* Reference To: MSVBVM50.__vbaStrCat, Ord:0000h
                                    |
:00431902 FF1504D34500            Call dword ptr [0045D304]
:00431908 8BD0                    mov edx, eax
:0043190A 8D4DC0                  lea ecx, dword ptr [ebp-40]
```

分析一下，如果试用期未过期的话，`004318E3` 处的代码是应该被执行到的，如果过期了则此部分代码不会被执行。因此向上查看跳转代码：

```text
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043181E(C)
|
:0043183D C78564FFFFFF00000000    mov dword ptr [ebp+FFFFFF64], 00000000

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043183B(U)
|
:00431847 8D4DB8                  lea ecx, dword ptr [ebp-48]

* Reference To: MSVBVM50.__vbaFreeObj, Ord:0000h
                                    |
:0043184A FF15F4D44500            Call dword ptr [0045D4F4]
:00431850 C745FC25000000          mov [ebp-04], 00000025
:00431857 66837DCC00              cmp word ptr [ebp-34], 0000
:0043185C 7E07                    jle 00431865       ;此处只要不跳就可以继续试用了
:0043185E 66837DCC2D              cmp word ptr [ebp-34], 002D
:00431863 7E15                    jle 0043187A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043185C(C)
|
:00431865 C745FC26000000          mov [ebp-04], 00000026
:0043186C 66C70576904500FFFF      mov word ptr [00459076], FFFF
:00431875 E920010000              jmp 0043199A       ;此处跳到了显示主界面和要求输入注册码

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00431863(C)
|
:0043187A C745FC28000000          mov [ebp-04], 00000028
:00431881 8B4D08                  mov ecx, dword ptr [ebp+08]
:00431884 8B11                    mov edx, dword ptr [ecx]
:00431886 8B4508                  mov eax, dword ptr [ebp+08]
:00431889 50                      push eax
:0043188A FF9218030000            call dword ptr [edx+00000318]
:00431890 50                      push eax
:00431891 8D4DB8                  lea ecx, dword ptr [ebp-48]
:00431894 51                      push ecx

* Reference To: MSVBVM50.__vbaObjSet, Ord:0000h
                                    |
:00431895 FF1538D34500            Call dword ptr [0045D338]
:0043189B 894588                  mov dword ptr [ebp-78], eax
:0043189E 8D55B4                  lea edx, dword ptr [ebp-4C]
:004318A1 52                      push edx
:004318A2 6A01                    push 00000001
:004318A4 8B4588                  mov eax, dword ptr [ebp-78]
:004318A7 8B08                    mov ecx, dword ptr [eax]
:004318A9 8B5588                  mov edx, dword ptr [ebp-78]
:004318AC 52                      push edx
:004318AD FF5140                  call [ecx+40]
:004318B0 894584                  mov dword ptr [ebp-7C], eax
:004318B3 837D8400                cmp dword ptr [ebp-7C], 00000000
:004318B7 7D1D                    jge 004318D6
:004318B9 6A40                    push 00000040
:004318BB 68BC154100              push 004115BC
:004318C0 8B4588                  mov eax, dword ptr [ebp-78]
:004318C3 50                      push eax
:004318C4 8B4D84                  mov ecx, dword ptr [ebp-7C]
:004318C7 51                      push ecx

* Reference To: MSVBVM50.__vbaHresultCheckObj, Ord:0000h
                                    |
:004318C8 FF1518D34500            Call dword ptr [0045D318]
:004318CE 898560FFFFFF            mov dword ptr [ebp+FFFFFF60], eax
:004318D4 EB0A                    jmp 004318E0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004318B7(C)
|
:004318D6 C78560FFFFFF00000000    mov dword ptr [ebp+FFFFFF60], 00000000

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004318D4(U)
|
:004318E0 8B55B4                  mov edx, dword ptr [ebp-4C]
:004318E3 895580                  mov dword ptr [ebp-80], edx

* Possible StringData Ref from Code Obj ->"330 day trial: "
                                    |
:004318E6 68DCF94000              push 0040F9DC
:004318EB 668B45CC                mov ax, word ptr [ebp-34]
:004318EF 50                      push eax
```

如果你对这些跳转判断不出的话，可以在跳转上设断，逐一试试，就可以了，我也是试出来的。

## 总结

用十六进制编辑器打开 `vopt99.exe`，找：

```text
66 83 7D CC 00 7E 07
        ^^ ^^
        90 90
```

这样就完成了。运行后，主界面上显示您的试用期还-XX 天，没关系啦，试用一下功能，一切正常。

2001.2.19 by `mjing`，E-mail: `mjing@wx88.net`
