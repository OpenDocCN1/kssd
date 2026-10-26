
# 监狱(Quod)1.0完全破解（附送作弊指南）

我也不知道自己消失了多久，反正自从上次硬盘挂掉，装系统时一时疏忽，直接装了`XP`而且全盘`NTFS`，就很少再碰破解了，只会在`98`下用`TRW`拦着`hmemcpy`的我，除了偶尔给同学解解需要用的软件，就再也没有研究过破解。几年来，`BCG`一个月一次的作业根本没做，想必早已被除名，说来真是惭愧，对不起`BCG`的大大，对不起朋友们。

前一阵子，在天空软件站偶然看到了`SouSuite`这个游戏，想起来我的第一篇破文，那篇`01`年`9`月`8`号发表的，非常幼稚的“`SolSuite v8.0`破解手记”，看雪老大出于鼓励新人，把它加了精，谢谢你。把`SouSuite`最新版本下载回来后，破了它，软件强度变大了，但我却发现自己的思路也清晰了。是啊，这么多年来，虽然已经很少研究破解，但是这中间学到的东西，不仅是对破解，更是对整个人生，都是非常有益的。

去天空软件站下载了我第一个破解的软件，一个棋类小游戏，监狱(`Quod`)，当时的我，按着教程上面说的，用`TRW`载入，`bpx hmemcpy`，然后一步一步的跟（说是跟，其实是按），每走一步，都要`d eax,d ebx`…直到把所有寄存器都看完（别笑），终于看到真假注册码比较的时候，那时的心情，现在已经体会不到，也难以言表。

这次当然不止是要跟出注册码而已，先把以前欠的补回来一点吧。

## 破解信息

**破解：** `波导终结者`

**被破软件：** 监狱(`Quod`) `V1.0`

**下载地址：** `79cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4C8P5h3y4F1i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0X3N6q4)9J5c8U0j5J5y4o6g2Q4x3X3g2Z5N6r3#2D9`

**工具：** `OD`，`W32DASM`，`FI`，脱壳工具若干

监狱是一个棋类游戏，把四个进攻棋子连成一个正方形则即得胜利，每方有`20`个进攻棋子和`6`个防守棋子。不过感觉和五子棋相似，先走有很大优势……

先用`FI`看看加的什么壳，`aspack 2.001`。用主页的`Aspackdie`脱，却得修复。试了几个脱壳工具都如此，我又不会修复，无奈之下，用主页的通用脱壳工具`File Scanner`脱了下，竟然完美脱下来了，哈。

脱完壳，用`W32DASM`反汇编，很容易找到关键处

```asm
:0045F2F3 90                      nop
:0045F2F4 55                      push ebp
:0045F2F5 8BEC                    mov ebp, esp
:0045F2F7 33C9                    xor ecx, ecx
:0045F2F9 51                      push ecx
:0045F2FA 51                      push ecx
:0045F2FB 51                      push ecx
:0045F2FC 51                      push ecx
:0045F2FD 53                      push ebx
:0045F2FE 8BD8                    mov ebx, eax
:0045F300 33C0                    xor eax, eax
:0045F302 55                      push ebp
:0045F303 68ECF34500              push 0045F3EC
:0045F308 64FF30                  push dword ptr fs:[eax]
:0045F30B 648920                  mov dword ptr fs:[eax], esp
:0045F30E 8D55F8                  lea edx, dword ptr [ebp-08]
:0045F311 8B8300030000            mov eax, dword ptr [ebx+00000300]
:0045F317 E8989EFDFF              call 004391B4
:0045F31C 8B45F8                  mov eax, dword ptr [ebp-08]
:0045F31F 50                      push eax
:0045F320 8D55F0                  lea edx, dword ptr [ebp-10]
:0045F323 8B83FC020000            mov eax, dword ptr [ebx+000002FC]
:0045F329 E8869EFDFF              call 004391B4
:0045F32E 8B55F0                  mov edx, dword ptr [ebp-10]
:0045F331 8D4DF4                  lea ecx, dword ptr [ebp-0C]
:0045F334 8BC3                    mov eax, ebx
:0045F336 E8BD010000              call 0045F4F8
:0045F33B 8B55F4                  mov edx, dword ptr [ebp-0C]
:0045F33E 58                      pop eax
:0045F33F E8F854FAFF              call 0040483C
:0045F344 7576                    jne 0045F3BC
```
很容易看出，这个跳转决定着注册成功或失败，于是从上面下断点

```asm
:0045F346 B201                    mov dl, 01
:0045F348 A1E8EA4500              mov eax, dword ptr [0045EAE8]
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045F2DE(C)
|
:0045F34D E896F8FFFF              call 0045EBE8
:0045F352 8945FC                  mov dword ptr [ebp-04], eax
:0045F355 33C0                    xor eax, eax
:0045F357 55                      push ebp
:0045F358 68B5F34500              push 0045F3B5
:0045F35D 64FF30                  push dword ptr fs:[eax]
:0045F360 648920                  mov dword ptr fs:[eax], esp
:0045F363 B101                    mov cl, 01
* Possible StringData Ref from Code Obj ->"Software\XDZHAN\Quod"
                                  |
:0045F365 BA00F44500              mov edx, 0045F400
:0045F36A 8B45FC                  mov eax, dword ptr [ebp-04]
:0045F36D E87AF9FFFF              call 0045ECEC
* Possible StringData Ref from Code Obj ->"Real Programmers Use Pascal!"
                                  |
:0045F372 B920F44500              mov ecx, 0045F420
* Possible StringData Ref from Code Obj ->"Key"
                                  |
:0045F377 BA48F44500              mov edx, 0045F448
:0045F37C 8B45FC                  mov eax, dword ptr [ebp-04]
:0045F37F E804FBFFFF              call 0045EE88
* Possible StringData Ref from Code Obj ->"软件注册成功,谢谢您的支持!"
                                  |
:0045F384 B854F44500              mov eax, 0045F454
:0045F389 E82A39FDFF              call 00432CB8
:0045F38E A1E83E4600              mov eax, dword ptr [00463EE8]
:0045F393 8B00                    mov eax, dword ptr [eax]
* Possible StringData Ref from Code Obj ->"监狱(Quod)―注册版"
                                  |
:0045F395 BA78F44500              mov edx, 0045F478
:0045F39A E8459EFDFF              call 004391E4
:0045F39F 33C0                    xor eax, eax
:0045F3A1 5A                      pop edx
:0045F3A2 59                      pop ecx
:0045F3A3 59                      pop ecx
:0045F3A4 648910                  mov dword ptr fs:[eax], edx
:0045F3A7 68C6F34500              push 0045F3C6
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045F3BA(U)
|
:0045F3AC 8B45FC                  mov eax, dword ptr [ebp-04]
:0045F3AF E82043FAFF              call 004036D4
:0045F3B4 C3                      ret
:0045F3B5 E9AE4AFAFF              jmp 00403E68
:0045F3BA EBF0                    jmp 0045F3AC
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045F344(C)
|
* Possible StringData Ref from Code Obj ->"注册码不正确,无法注册!"
                                  |
:0045F3BC B894F44500              mov eax, 0045F494
:0045F3C1 E8F238FDFF              call 00432CB8
:0045F3C6 33C0                    xor eax, eax
:0045F3C8 5A                      pop edx
:0045F3C9 59                      pop ecx
:0045F3CA 59                      pop ecx
:0045F3CB 648910                  mov dword ptr fs:[eax], edx
:0045F3CE 68F3F34500              push 0045F3F3
```

## 跟踪过程

在此行拦断，开始跟踪

```asm
0045F302   .  55            PUSH EBP
0045F303   .  68 ECF34500   PUSH QUOD.0045F3EC
0045F308   .  64:FF30       PUSH DWORD PTR FS:[EAX]
0045F30B   .  64:8920       MOV DWORD PTR FS:[EAX],ESP
0045F30E   .  8D55 F8       LEA EDX,DWORD PTR SS:[EBP-8]
0045F311   .  8B83 00030000 MOV EAX,DWORD PTR DS:[EBX+300]
0045F317   .  E8 989EFDFF   CALL QUOD.004391B4
0045F31C   .  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]             ;  假码->eax
0045F31F   .  50            PUSH EAX
0045F320   .  8D55 F0       LEA EDX,DWORD PTR SS:[EBP-10]
0045F323   .  8B83 FC020000 MOV EAX,DWORD PTR DS:[EBX+2FC]
0045F329   .  E8 869EFDFF   CALL QUOD.004391B4
0045F32E   .  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]            ;  用户名->edx
0045F331   .  8D4D F4       LEA ECX,DWORD PTR SS:[EBP-C]
0045F334   .  8BC3          MOV EAX,EBX
0045F336   .  E8 BD010000   CALL QUOD.0045F4F8                       ;  ①下面一行就出真码了，所以这个CALL肯定是运算注册码的关键，跟进去
0045F33B   .  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]             ;  [EBP-C]为真码，真码->edx
0045F33E   .  58            POP EAX                                  ;  假码->eax
0045F33F   .  E8 F854FAFF   CALL QUOD.0040483C                       ;  这个CALL就是进去比较真假注册码了，略过
0045F344   .  75 76         JNZ SHORT QUOD.0045F3BC                  ;  注册码不对就跳
0045F346   .  B2 01         MOV DL,1                                 ;  注册对了之后，下面开始写注册表项
0045F348   .  A1 E8EA4500   MOV EAX,DWORD PTR DS:[45EAE8]
0045F34D   .  E8 96F8FFFF   CALL QUOD.0045EBE8
0045F352   .  8945 FC       MOV DWORD PTR SS:[EBP-4],EAX
0045F355   .  33C0          XOR EAX,EAX
0045F357   .  55            PUSH EBP
0045F358   .  68 B5F34500   PUSH QUOD.0045F3B5
0045F35D   .  64:FF30       PUSH DWORD PTR FS:[EAX]
0045F360   .  64:8920       MOV DWORD PTR FS:[EAX],ESP
0045F363   .  B1 01         MOV CL,1
0045F365   .  BA 00F44500   MOV EDX,QUOD.0045F400                    ;  software\xdzhan\quod
0045F36A   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0045F36D   .  E8 7AF9FFFF   CALL QUOD.0045ECEC
0045F372   .  B9 20F44500   MOV ECX,QUOD.0045F420                    ;  real programmers use pascal!
0045F377   .  BA 48F44500   MOV EDX,QUOD.0045F448                    ;  key
```
