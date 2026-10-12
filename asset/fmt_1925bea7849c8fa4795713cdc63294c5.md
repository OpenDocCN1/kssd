
# 有声有色 3.10

> 原文：[`www.pediy.com/kssd/pediy02/forum260.htm`](https://www.pediy.com/kssd/pediy02/forum260.htm)

今天我 cracking 了有声有色 3.10， 下载地点 `http://www.newhua.com.cn/down/you310.zip`, 它是用 `aspack 2.1` 加的壳，我已成功地将它 `unpack`, `crack down`, 但我想现在想写一个 `patch`, 一个直接在它的原文件 `MusRea.exe` 上写上 `patch` 代码，即在 `.Aspack section` 最后的无代码部分加上我的 `patch` 代码，但似乎不成功，望高手能赐教：

分析过程如下：

在 `.Aspack section` 一节, 最后要返回原程序 `entry point` 前的反汇编代码如下：

```
:004CC4F3 61                 popad
:004CC4F4 7508               jne 004CC4FE
* Possible Reference to String Resource ID=00001: "櫃橠 DDD 櫃櫃 DDA 櫃櫃橠 D 櫃櫃橠 D 櫃櫃橠 D 櫃櫃煓 D 櫉櫃閂檻 D?鶛?
                                                  |
:004CC4F6 B801000000         mov eax, 00000001
:004CC4FB C20C00             ret 000C
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004CC4F4(C)
|
:004CC4FE 6800000000         push 00000000  <==将此处改为 E99D1F0000
:004CC503 C3                 ret    <==从此返回原程序 entry point
```

我的思路将 `:004CC4FE` 处代码改为 `jmp 004ce434`, `:004ce434` 为 `.Aspack section` 一节无代码处，在此处我写入我的 `patch` 代码，即:

```
mov dword ptr [0048A4E1], 90E98002
push 00000000
ret
```

HEX 值：`C705E1A448000280E9906800000000C3`,

这种思路我在老外的 `tutor` 里看到过，但我的测试未通过，请高人指点。

附：我写的关于如何破解有声有色 3.10 的 `tutor`

```
Target file : MusRea.exe
Packer: Aspack 2.1
Compiler: Delphi 5.0
cracking tutorial:
1.unpacking target file with your favorite tools , eg. softice or trw, I prefer to use UnAspack 1.0.9.1
2.Using ProcDump change the Section Characteristics for the CODE section to 0xE0000020. (you must master PE file format, or you don't understand it. :-()
3.Disassemble the target file
4.anlysis the source code:
:0048A4D4 8B8318050000         mov eax, dword ptr [ebx+00000518]
:0048A4DA 83B8380100001E       cmp dword ptr [eax+00000138], 0000001E  <== see if you have  used 30 times
:0048A4E1 0F8498000000         je 0048A57F
:0048A4E7 8B831C050000         mov eax, dword ptr [ebx+0000051C]
:0048A4ED 8B10                 mov edx, dword ptr [eax]
:0048A4EF FF92B4000000         call dword ptr [edx+000000B4]
:0048A4F5 3C01                 cmp al, 01
:0048A4F7 0F8482000000         je 0048A57F
:0048A4FD 8B8324050000         mov eax, dword ptr [ebx+00000524]
:0048A503 8B10                 mov edx, dword ptr [eax]
:0048A505 FF92B4000000         call dword ptr [edx+000000B4]
:0048A50B 3C01                 cmp al, 01
:0048A50D 7470                 je 0048A57F
:0048A50F 8B8328050000         mov eax, dword ptr [ebx+00000528]
:0048A515 8B10                 mov edx, dword ptr [eax]
:0048A517 FF92B4000000         call dword ptr [edx+000000B4]
:0048A51D 3C01                 cmp al, 01
:0048A51F 745E                 je 0048A57F
:0048A521 8B832C050000         mov eax, dword ptr [ebx+0000052C]
:0048A527 8B10                 mov edx, dword ptr [eax]
:0048A529 FF92B4000000         call dword ptr [edx+000000B4]
:0048A52F 3C01                 cmp al, 01
:0048A531 744C                 je 0048A57F
:0048A533 8B8330050000         mov eax, dword ptr [ebx+00000530]
:0048A539 8B10                 mov edx, dword ptr [eax]
:0048A53B FF92B4000000         call dword ptr [edx+000000B4]
:0048A541 3C01                 cmp al, 01
:0048A543 743A                 je 0048A57F
:0048A545 8B8330050000         mov eax, dword ptr [ebx+00000530]
:0048A54B 8B10                 mov edx, dword ptr [eax]
:0048A54D FF92B4000000         call dword ptr [edx+000000B4]
:0048A553 3C01                 cmp al, 01
:0048A555 7428                 je 0048A57F
:0048A557 8B8334050000         mov eax, dword ptr [ebx+00000534]
:0048A55D 8B10                 mov edx, dword ptr [eax]
:0048A55F FF92B4000000         call dword ptr [edx+000000B4]
:0048A565 3C01                 cmp al, 01
:0048A567 7416                 je 0048A57F
:0048A569 8B833C050000         mov eax, dword ptr [ebx+0000053C]
:0048A56F 8B10                 mov edx, dword ptr [eax]
:0048A571 FF92B4000000         call dword ptr [edx+000000B4]
:0048A577 3C01                 cmp al, 01
:0048A579 0F85E2010000         jne 0048A761  <==if everything OK,  you should jump to 0048A761 from it.
   Through the analysis, you can change the code at :0048A4DA to jmp 0048A761, I think you know how to modify HEX value, do it yourself. Do you understand? let me know.
5.run the target file again, see what happen? Bingo!  The limit use 30 times is removed.
DO NOT USE THIS TUTOR FOR COMMERICAL PURPOSE, IF YOU LIKE THIS PROGRAM, PLEASE PAY THE AUTHOR'S HARD WORK, THE REGISTER FEE IS ONLY TEN YUAN.
```

## 如何 patch 有声有色 3.10

难得今天心情比较好, 准备学习 `SEH`; 拿有声有色 3.10 开刀( 有人已经破解了, 不必再去找 `CODE` ).

1. 用 `PRODUMP` 载入主程序 `MUSREA.EXE`, 知道基地址为 `00400000`,为了找程序入口点, 用上 `D.BOY` 的冲击波 2000, 找到为 `0048A410`. 所以 `offset` = `0048a410-00400000` = `0008a410`. 用 `ULTRAEDIT` 搜索 `10 a4 08`, 找到惟一地址 `0004c49d`. 因为我准备让她先运行我的 `PATCH CODE`, 所以准备在 `000003d0` 处修补,于是在 `0004c49d` 处将 `10 a4 08` 改为 `d0 03 00`.

2. `patch code`.

   首先感谢十三少与 `dREAMtHEATER` 的工作, 我知道要在

   ```
   cs:0048A4E1 0F8498000000      je  0048A57F 改为
                       eb74               jmp 0048A561
                       9090               nop
                       9090               nop
   ```

   与我见到的 `[PC/MFD]`写的文章\*\*\*\* `iNLiNE pATCHiNG A pROGRAM pACKED WiTH ASProtect - by Predator` 不同, 我发现要这样修改.

   在 `000003d0` 处: 用 `hiew` 写上如下代码

   ```
   000003d0:  0000                   我发现跳到这里后不能直接
                                           写 patch,必须过渡一下.
   000003d2:  66c705e1a44800eb74   patch code 1
   000003db:  66c705e3a448009090   patch code 2
   000003e4:  66c705e5a448009090   patch code 3
   000003ed:  6810e44800           放入真实的入口点
   000003f2:  c3                   返回
   ```

   `ok, job done`.

   完成了自己的第一个 `SEH` 作品.

   同时今天自己也是第一次破解 `VB6` 程序--`Dr salamn's powertools` 成功的日子.

   ```
   000003d2:  66c705e1a44800e9eb  patch code 1
         000003db:  66c705e3a448000200  patch code 2
         000003e4:  66c705e5a448000090  patch code 3
   ```

   `4dd466d0e1e0cc123afbe504da2462ca`
