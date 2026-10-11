
# Uedit32 8.0

> 原文：[`www.pediy.com/kssd/pediy03/forum980.htm`](https://www.pediy.com/kssd/pediy03/forum980.htm)

Uedit32 8.0 破解过程

工具:

`Sice4.5` `W32dasm`

如见采用 `key` 文件保护方式,用 `bpx createfilea do " d esp\->4 "` 下断点,看到打开的是文件 `Uedit32.reg` 时下断点 `bpx readfile`.跟踪度写的数据到下面程序。

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00468D47(C)
:00468D57 E8AAAA0500            call 004C3806
:00468D5C 8B7004                mov esi, dword ptr [eax+04]
:00468D5F E8A2AA0500            call 004C3806
:00468D64 8B4004                mov eax, dword ptr [eax+04]
:00468D67 05E8030000            add eax, 000003E8        // eax 指向输入的用户名
:00468D6C 50                    push eax                  // 下断点 bpx eax
:00468D6D E80E910200            call 00491E80
:00468D72 59                    pop ecx
:00468D73 C68430E803000020      mov byte ptr [eax+esi+000003E8], 20
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:00468CA2(C), :00468CFB(C)
:00468D7B 8D4DE4                lea ecx, dword ptr [ebp-1C]
:00468D7E 885DFC                mov byte ptr [ebp-04], bl
:00468D81 E8942E0400            call 004ABC1A
:00468D86 834DFCFF              or dword ptr [ebp-04], FFFFFFFF
:00468D8A 8D4D10                lea ecx, dword ptr [ebp+10]
:00468D8D E8391D0400            call 004AAACB
:00468D92 33C0                  xor eax, eax
:00468D94 EB12                  jmp 00468DA8
```

按 `F5` 运行拦到后按 `F12` 从函数中返回

```
* Possible Reference to String Resource ID=00006: "Load Macro"
:00413847 6A06                  push 00000006
:00413849 8065BF00              and byte ptr [ebp-41], 00
:0041384D 80A547FFFFFF00        and byte ptr [ebp+FFFFFF47], 00
:00413854 59                    pop ecx
:00413855 8D8570FFFFFF          lea eax, dword ptr [ebp+FFFFFF70]
:0041385B 50                    push eax
:0041385C 8DBD70FFFFFF          lea edi, dword ptr [ebp+FFFFFF70]
:00413862 FF750C                push [ebp+0C]
:00413865 C68537FFFFFF30        mov byte ptr [ebp+FFFFFF37], 30
:0041386C F3                    repz
:0041386D A5                    movsd
:0041386E 80658700              and byte ptr [ebp-79], 00
:00413872 C68577FFFFFF30        mov byte ptr [ebp+FFFFFF77], 30
:00413879 C6857CFFFFFF30        mov byte ptr [ebp+FFFFFF7C], 30
:00413880 C6458630              mov [ebp-7A], 30
:00413884 C6853CFFFFFF30        mov byte ptr [ebp+FFFFFF3C], 30
:0041388B C68546FFFFFF30        mov byte ptr [ebp+FFFFFF46], 30
:00413892 E8A9FD0700            call 00493640
:00413897 59                    pop ecx                   // 返回到这里
:00413898 59                    pop ecx
:00413899 5E                    pop esi
:0041389A 85C0                  test eax, eax
:0041389C 7523                  jne 004138C1              //一定不等，跳转
:0041389E 8D8570FFFFFF          lea eax, dword ptr [ebp+FFFFFF70]
:004138A4 50                    push eax
:004138A5 FF750C                push [ebp+0C]
:004138A8 E893FD0700            call 00493640
:004138AD 59                    pop ecx
:004138AE 85C0                  test eax, eax
:004138B0 59                    pop ecx
:004138B1 7554                  jne 00413907
:004138B3 FF750C                push [ebp+0C]
:004138B6 E8C5E50700            call 00491E80
:004138BB 83F80C                cmp eax, 0000000C
:004138BE 59                    pop ecx
:004138BF 7446                  je 00413907
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0041389C(C)
:004138C1 8D45B0                lea eax, dword ptr [ebp-50]   //比较的值，不知是什么
:004138C4 50                    push eax
:004138C5 8D8570FFFFFF          lea eax, dword ptr [ebp+FFFFFF70] //输入的注册码
:004138CB 50                    push eax
:004138CC E86FFD0700            call 00493640                 //作比较
:004138D1 59                    pop ecx
:004138D2 85C0                  test eax, eax                 //eax=1，比较不等
:004138D4 59                    pop ecx
:004138D5 7429                  je 00413900                   //步跳转
:004138D7 8D8530FFFFFF          lea eax, dword ptr [ebp+FFFFFF30] //真正的注册码
:004138DD 50                    push eax
:004138DE 8D8570FFFFFF          lea eax, dword ptr [ebp+FFFFFF70] //输入的注册码
:004138E4 50                    push eax
:004138E5 E856FD0700            call 00493640                 //比较
:004138EA 59                    pop ecx
:004138EB 85C0                  test eax, eax                 //如果相等 eax=0
:004138ED 59                    pop ecx
:004138EE 7410                  je 00413900                   //跳转就 OK 了！
:004138F0 C705C04E500001000000  mov dword ptr [00504EC0], 00000001
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:00413659(C), :00413662(C), :0041367A(C)
:004138FA 33C0                  xor eax, eax
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0041390A(U)
:004138FC 5F                    pop edi
:004138FD 5B                    pop ebx
:004138FE C9                    leave
:004138FF C3                    ret
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:004138D5(C), :004138EE(C)
:00413900 8325C04E500000        and dword ptr [00504EC0], 00000000
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:004138B1(C), :004138BF(C)
:00413907 6A01                  push 00000001
:00413909 58                    pop eax
:0041390A EBF0                  jmp 004138FC
```

我的用户名和注册码是：

`Ultra Edit8.0` `Name:` `floatsnow` `Sn:` `M2V3R-Q0N1J-08Z8W-G9B30`

程序退出时对注册码的其中三位进行比较，下断点 `bpx deletefilea` 拦住后按 `F12` 返回主程序向上着跳转。程序如下：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00468EAB(C)
:00468EB4 66813DF88E5000D007    cmp word ptr [00508EF8], 07D0
:00468EBD 0F8656010000          jbe 00469019
:00468EC3 66833DFA8E500001      cmp word ptr [00508EFA], 0001
:00468ECB 0F8648010000          jbe 00469019
:00468ED1 68D84D5000            push 00504DD8
:00468ED6 8D4DE4                lea ecx, dword ptr [ebp-1C]
:00468ED9 E862190400            call 004AA840
:00468EDE FF75E4                push [ebp-1C]
:00468EE1 895DFC                mov dword ptr [ebp-04], ebx
:00468EE4 E8978F0200            call 00491E80
:00468EE9 A1BC4E5000            mov eax, dword ptr [00504EBC]
:00468EEE FF35CC4D5000          push dword ptr [00504DCC]
:00468EF4 8945F0                mov dword ptr [ebp-10], eax
:00468EF7 E8848F0200            call 00491E80
:00468EFC 59                    pop ecx
:00468EFD 83F80F                cmp eax, 0000000F
:00468F00 59                    pop ecx
:00468F01 0F8206010000          jb 0046900D
:00468F07 391D9C8E5000          cmp dword ptr [00508E9C], ebx
:00468F0D 0F85FA000000          jne 0046900D
:00468F13 391DA08E5000          cmp dword ptr [00508EA0], ebx
:00468F19 0F84EE000000          je 0046900D
:00468F1F 0FB645F0              movzx eax, byte ptr [ebp-10]
```

```
* Possible Reference to String Resource ID=00025: "Dos Command"
:00468F23 6A19                  push 00000019
:00468F25 8B3DCC4D5000          mov edi, dword ptr [00504DCC]
:00468F2B 99                    cdq
:00468F2C 59                    pop ecx
:00468F2D F7F9                  idiv ecx
:00468F2F 0FBE4716              movsx eax, byte ptr [edi+16]  //取注册码最后一位
:00468F33 83C241                add edx, 00000041
```



:00468F36 `3BC2`                    `cmp eax, edx` //和真正的值进行比较   

:00468F38 `7530`                    `jne 00468F6A` //不等就跳，删除文件   

:00468F3A `0FB645F0`                `movzx eax, byte ptr [ebp\-10]`   

\* Possible Reference to String Resource ID=00009: "   

This copy of UltraEdit-32 is licensed to :   

 "   

                               |   

:00468F3E `6A09`                    `push 00000009`   

:00468F40 `99`                       `cdq`   

:00468F41 `59`                       `pop ecx`   

:00468F42 `F7F9`                    `idiv ecx`   

:00468F44 `0FBE4707`                `movsx eax, byte ptr [edi+07]` //取注册码第 7 位   

:00468F48 `83C230`                  `add edx, 00000030`   

:00468F4B `3BC2`                    `cmp eax, edx`  //比较   

:00468F4D `751B`                    `jne 00468F6A`  //不等跳到删除文件   

:00468F4F `0FB645F0`                `movzx eax, byte ptr [ebp\-10]`   

:00468F53 `8A4F0C`                  `mov cl, byte ptr [edi+0C]` //取注册码第 12 位   

\* Possible Reference to String Resource ID=00013: "Mod: "   

                               |   

:00468F56 `6A0D`                    `push 0000000D`   

:00468F58 `99`                       `cdq`   

:00468F59 `5F`                       `pop edi`   

:00468F5A `F7FF`                    `idiv edi`   

:00468F5C `0FBEC1`                  `movsx eax, cl`  //注册码第 12 位给 eax   

:00468F5F `83C241`                  `add edx, 00000041`   

:00468F62 `3BC2`                    `cmp eax, edx`  //比较   

:00468F64 `0F84A3000000`            `je 0046900D`  //不等继续执行，删除文件   

      //相等则跳转   

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

|:00468F38(C), :00468F4D(C)   

|   

:00468F6A `891DA08E5000`            `mov dword ptr [00508EA0], ebx`   

:00468F70 `E891A80500`              `call 004C3806`   

:00468F75 `8B4004`                  `mov eax, dword ptr [eax+04]`   

:00468F78 `8D4DF0`                  `lea ecx, dword ptr [ebp\-10]`   

:00468F7B `FFB08C000000`            `push dword ptr [eax+0000008C]`   

:00468F81 `E8B31B0400`              `call 004AAB39`   

:00468F86 `8B45F0`                  `mov eax, dword ptr [ebp\-10]`   

:00468F89 `8D4DF0`                  `lea ecx, dword ptr [ebp\-10]`   

:00468F8C `C645FC01`                `mov [ebp\-04], 01`   

:00468F90 `8B40F8`                  `mov eax, dword ptr [eax\-08]`   

:00468F93 `83C0FD`                  `add eax, FFFFFFFD`   

:00468F96 `50`                       `push eax`   

:00468F97 `8D45E0`                  `lea eax, dword ptr [ebp\-20]`   

:00468F9A `50`                       `push eax`   

:00468F9B `E8309B0300`              `call 004A2AD0`   

\* Possible StringData Ref from Data Obj ->"REG"   

                               |   

:00468FA0 `68E0B24F00`              `push 004FB2E0`   

:00468FA5 `50`                       `push eax`   

:00468FA6 `8D45DC`                  `lea eax, dword ptr [ebp\-24]`   

:00468FA9 `C645FC02`                `mov [ebp\-04], 02`   

:00468FAD `50`                       `push eax`   

:00468FAE `E8611D0400`              `call 004AAD14`   

:00468FB3 `50`                       `push eax`   

:00468FB4 `8D4DF0`                  `lea ecx, dword ptr [ebp\-10]`   

:00468FB7 `C645FC03`                `mov [ebp\-04], 03`   

:00468FBB `E8F81B0400`              `call 004AABB8`   

:00468FC0 `8D4DDC`                  `lea ecx, dword ptr [ebp\-24]`   

:00468FC3 `C645FC02`                `mov [ebp\-04], 02`   

:00468FC7 `E8FF1A0400`              `call 004AAACB`   

:00468FCC `8D4DE0`                  `lea ecx, dword ptr [ebp\-20]`   

:00468FCF `C645FC01`                `mov [ebp\-04], 01`   

:00468FD3 `E8F31A0400`              `call 004AAACB`   

\* Possible Reference to Dialog: DialogID\_0080   

                               |   

\* Possible Reference to Dialog: DialogID\_006E, CONTROL\_ID:0080, "Details for registration can be found in"   

                               |   

\* Possible Reference to String Resource ID=00128: "Lines containing find string:"   

                               |   

:00468FD8 `6880000000`              `push 00000080`   

:00468FDD `FF75F0`                  `push [ebp\-10]`   

\* Reference To: `KERNEL32.SetFileAttributesA`, Ord:0268h   

                               |   

:00468FE0 `FF1544434D00`            Call dword ptr [004D4344]   

:00468FE6 `FF75F0`                  `push [ebp\-10]`   

\* Reference To: `KERNEL32.DeleteFileA`, Ord:0057h   

// 调用 `deletefilea` 删除文件 `Uedit32.reg`   

:00468FE9 `FF1520434D00`            Call dword ptr [004D4320]   

:00468FEF `A1D48E5000`              `mov eax, dword ptr [00508ED4]`   

:00468FF4 `8D4DF0`                  `lea ecx, dword ptr [ebp\-10]`   

:00468FF7 `83C0D2`                  `add eax, FFFFFFD2`   

:00468FFA `885DFC`                  `mov byte ptr [ebp\-04], bl`   

:00468FFD `A3E08E5000`              `mov dword ptr [00508EE0], eax`   

:00469002 `899E820D0000`            `mov dword ptr [esi+00000D82], ebx`   

:00469008 `E8BE1A0400`              `call 004AAACB`   

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

|:00468F01(C), :00468F0D(C), :00468F19(C), :00468F64(C)   

|   

:0046900D `834DFCFF`                `or dword ptr [ebp\-04], FFFFFFFF`   

:00469011 `8D4DE4`                  `lea ecx, dword ptr [ebp\-1C]`   

:00469014 `E8B21A0400`              `call 004AAACB`   

破解完成,整理注册码为:   

```
Ultra Edit8.0  Name: floatsnow Sn: M2V3R-Q0N1J-E8Z8W-G9B3A
```

千万不要用我的呀!!!!!!!  `4dd466d0e1e0cc123afbe504da2462ca`
