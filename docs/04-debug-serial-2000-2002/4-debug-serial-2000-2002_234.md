
# BrainsBreaker3.0

> 原文：[`www.pediy.com/kssd/pediy03/forum691.htm`](https://www.pediy.com/kssd/pediy03/forum691.htm)

谁能找出 BrainsBreaker3.0（巨好的拼图游戏）注册码？《论坛精华 2》没搞定！

破这种既好玩又有极大难度的软件才过瘾！

`http://www.brainsbreaker.com/download.htm` 或

`http://ourworld.compuserve.com/homepages/jtrujillo/bbsetup.exe`

==========================================================================

1. Pack：我不填

Name: `zest`

Your ID: `aaaa`

Key: `CCCCCCCCCCCCCCCC`

--------------------------------------------------------------------------

2. 此程序老在 `0167:00452509`～`0167:0045278F` 处循环，循环结束后出注册错误框，

注意 `0167:00452543` `PUSH DWORD [EBP-0C]` 处 `PUSH` 之值，你可下断点

`bpx 452546 do"d*esp"` 看看，我用 `TRW2K122`。

值依次变为 `full`, `entry`, `brainsbreaker`, `summer96`, `spring96`, `winter96`, `contry`。

--------------------------------------------------------------------------

3. 我做如下 3 个断点跟踪：①`bpx 452546 do"d*esp"` ②`bpx 452653` ③`bpx 45274e`

如 `KEY` 个数小于 16 个，正常中断顺序为 ①②①②①②...

如 `KEY` 个数大于等于 16 个，正常中断顺序为 ①②③①②③...

为了便于跟踪，我的 `KEY` 填 16 个 `c`，你或许会曰：`NOP` 掉 `45274e` 此行（`key` 数 < 16 还要 `NOP` 掉 `452653`）不就行了？的确，你注册成功了！上 `WC` 轻松一刻！

--------------------------------------------------------------------------

4. 一泡尿工夫你会发觉你的程序 `DIE` 得很惨！

--------------------------------------------------------------------------

5. 程序注册后（强迫注册）会在 `BBRK.INI` 加入：

`[PACKBrainsBreaker]`

`0=zest`

`1=aaaa`

`2=1076e9867711e9867711e9867711e9867711`

`3=2100`

`4=4e37d9b1`

`5=5533cba4`

注：上 `[PACK***]` 中 `***` 有可能就是 `full`, `entry`, `brainsbreaker`, `summer96`, `spring96`, `winter96`, `contry` 其中之一，细观察断点 ①。我发觉 `[PACKfull]` 会出错，见上第 3 点，`[PACKentry]` 和 `[PACKBrainsBreaker]` 可注册成功！其余 `[PACK***]` 说是成功了，但功能有限制。我爱 `[PACKBrainsBreaker]`，使用后没任何功能限制！

--------------------------------------------------------------------------

6. 每次想玩时得跟踪注册一次才能消除其功能限制，麻烦！谁叫我没 `$19.95` 呢。

你可试做一个 `PATCH` 看看，或用 `SMC` 方法。

--------------------------------------------------------------------------

7. 谁能找到注册码？

==========================================================================

```
0167:00452509 FF36            PUSH    DWORD [ESI]-------->循环开始处！
0167:0045250B 8D4DF0          LEA     ECX,[EBP-10]
0167:0045250E E891990100      CALL    0046BEA4
0167:00452513 8D45F0          LEA     EAX,[EBP-10]
0167:00452516 B9B05C4B00      MOV     ECX,004B5CB0
0167:0045251B 50             PUSH    EAX
0167:0045251C E84D54FDFF     CALL    0042796E
0167:00452521 837DF802       CMP     DWORD [EBP-08],BYTE +02
0167:00452525 8945D0         MOV     [EBP-30],EAX
0167:00452528 6A2C           PUSH    BYTE +2C
0167:0045252A 752A           JNZ     00452556
0167:0045252C E8EA380400     CALL    00495E1B
0167:00452531 8BF0           MOV     ESI,EAX
0167:00452533 59             POP     ECX
0167:00452534 85F6           TEST    ESI,ESI
0167:00452536 7440           JZ      00452578
0167:00452538 FF75E4         PUSH    DWORD [EBP-1C]-------->$$CCCCCCCCCCCCCCCC
0167:0045253B 8BCE           MOV     ECX,ESI
0167:0045253D FF75DC         PUSH    DWORD [EBP-24]-------->aaaa
0167:00452540 FF75C8         PUSH    DWORD [EBP-38]-------->zest
0167:00452543 FF75F4         PUSH    DWORD [EBP-0C]-------->注意变化！
0167:00452546 E8EC55FDFF     CALL    00427B37
0167:0045254B C706847C4A00   MOV     DWORD [ESI],004A7C84
0167:00452551 8975FC         MOV     [EBP-04],ESI
0167:00452554 EB26           JMP     SHORT 0045257C
0167:00452556 E8C0380400     CALL    00495E1B
0167:0045255B 85C0           TEST    EAX,EAX
0167:0045255D 59             POP     ECX
0167:0045255E 7418           JZ      00452578
0167:00452560 FF75E4         PUSH    DWORD [EBP-1C]
0167:00452563 8BC8           MOV     ECX,EAX
0167:00452565 FF75DC         PUSH    DWORD [EBP-24]
0167:00452568 FF75C8         PUSH    DWORD [EBP-38]
0167:0045256B FF75F4         PUSH    DWORD [EBP-0C]
0167:0045256E E8C455FDFF     CALL    00427B37
0167:00452573 8945FC         MOV     [EBP-04],EAX
0167:00452576 EB04           JMP     SHORT 0045257C
0167:00452578 8365FC00       AND     DWORD [EBP-04],BYTE +00
0167:0045257C 8B4DFC         MOV     ECX,[EBP-04]
0167:0045257F 8D951CFFFFFF   LEA     EDX,[EBP+FFFFFF1C]
0167:00452585 52             PUSH    EDX
0167:00452586 8B01           MOV     EAX,[ECX]
0167:00452588 FF5008        CALL    NEAR [EAX+08]-------->此 CALL 估计重要！
0167:0045258B 8BF0           MOV     ESI,EAX\-------->注意 EAX 值变化！
0167:0045258D 6A08           PUSH    BYTE +08
0167:0045258F 59             POP     ECX
0167:00452590 0FB63E         MOVZX   EDI,BYTE [ESI]
0167:00452593 8BC7           MOV     EAX,EDI
0167:00452595 6A01           PUSH    BYTE +01
0167:00452597 99             CDQ
0167:00452598 F7F9           IDIV    ECX
0167:0045259A 8BC7           MOV     EAX,EDI
0167:0045259C 5B             POP     EBX
0167:0045259D C1E803         SHR     EAX,03
0167:004525A0 0FB6443001     MOVZX   EAX,BYTE [EAX+ESI+01]
0167:004525A5 8BCA           MOV     ECX,EDX
0167:004525A7 D3E3           SHL     EBX,CL
0167:004525A9 23D8           AND     EBX,EAX
0167:004525AB 895D08         MOV     [EBP+08],EBX
0167:004525AE 7406           JZ      004525B6
0167:004525B0 6A01           PUSH    BYTE +01
0167:004525B2 5B             POP     EBX
0167:004525B3 895D08         MOV     [EBP+08],EBX
0167:004525B6 8A4604         MOV     AL,[ESI+04]
0167:004525B9 6A08           PUSH    BYTE +08
0167:004525BB 0C04           OR      AL,04
0167:004525BD 59             POP     ECX
0167:004525BE C0E002         SHL     AL,02
0167:004525C1 884604         MOV     [ESI+04],AL
0167:004525C4 8BC7           MOV     EAX,EDI
0167:004525C6 99             CDQ
0167:004525C7 F7F9           IDIV    ECX
0167:004525C9 6A08           PUSH    BYTE +08
0167:004525CB 8BC8           MOV     ECX,EAX
0167:004525CD 8BC7           MOV     EAX,EDI
0167:004525CF 99             CDQ
0167:004525D0 5F             POP     EDI
0167:004525D1 F7FF           IDIV    EDI
0167:004525D3 85DB           TEST    EBX,EBX
0167:004525D5 751B           JNZ     004525F2
0167:004525D7 8D7C3101      LEA     EDI,[ECX+ESI+01]
0167:004525DB 8D4A01         LEA     ECX,[EDX+01]
0167:004525DE B001           MOV     AL,01
0167:004525E0 B301           MOV     BL,01
0167:004525E2 D2E0           SHL     AL,CL
0167:004525E4 8BCA           MOV     ECX,EDX
0167:004525E6 D2E3           SHL     BL,CL
0167:004525E8 F6D0           NOT     AL
0167:004525EA 0A1F           OR      BL,[EDI]
0167:004525EC 22C3           AND     AL,BL
0167:004525EE 8807           MOV     [EDI],AL
0167:004525F0 EB0E           JMP     SHORT 00452600
0167:004525F2 8D443101      LEA     EAX,[ECX+ESI+01]
0167:004525F6 B301           MOV     BL,01
0167:004525F8 8BCA           MOV     ECX,EDX
0167:004525FA D2E3           SHL     BL,CL
0167:004525FC F6D3           NOT     BL
0167:004525FE 2018           AND     [EAX],BL
0167:00452600 0FB63E         MOVZX   EDI,BYTE [ESI]
0167:00452603 8BC7           MOV     EAX,EDI
0167:00452605 6A08           PUSH    BYTE +08
0167:00452607 99             CDQ
0167:00452608 59             POP     ECX
0167:00452609 F7F9           IDIV    ECX
0167:0045260B 8BC8           MOV     ECX,EAX
0167:0045260D 8D4101         LEA     EAX,[ECX+01]
0167:00452610 3D80000000     CMP     EAX,80
0167:00452615 7E02           JNG     00452619
0167:00452617 33C0           XOR     EAX,EAX
```



```
0167:00452619 8A543001       MOV      DL,[EAX+ESI+01]

0167:0045261D 8D443001       LEA      EAX,[EAX+ESI+01]

0167:00452621 80CA04         OR       DL,04

0167:00452624 BBDE100000     MOV      EBX,10DE

0167:00452629 C0EA02         SHR      DL,02

0167:0045262C 8810          MOV      [EAX],DL

0167:0045262E A1E86C4B00     MOV      EAX,[004B6CE8]

0167:00452633 99            CDQ

0167:00452634 F7FB         IDIV     EBX

0167:00452636 85D2         TEST     EDX,EDX

0167:00452638 750E         JNZ      00452648

0167:0045263A 8BC7         MOV      EAX,EDI

0167:0045263C 6A08         PUSH     BYTE +08

0167:0045263E 99            CDQ

0167:0045263F 5F           POP      EDI

0167:00452640 8D4C3102     LEA      ECX,[ECX+ESI+02]

0167:00452644 F7FF        IDIV     EDI

0167:00452646 0811         OR       [ECX],DL

0167:00452648 FF05E86C4B00  INC      DWORD [004B6CE8]

0167:0045264E 8A4601       MOV      AL,[ESI+01]

0167:00452651 A804         TEST     AL,04

0167:00452653 740B         JZ       00452660-------->NOP 掉它!!!!!!

0167:00452655 33C9         XOR      ECX,ECX

0167:00452657 394D08       CMP      [EBP+08],ECX

0167:0045265A 0F94C1       SETZ     CL

0167:0045265D 894D08       MOV      [EBP+08],ECX

0167:00452660 F6D0         NOT      AL

0167:00452662 884601       MOV      [ESI+01],AL

0167:00452665 33C0         XOR      EAX,EAX

0167:00452667 394508       CMP      [EBP+08],EAX

0167:0045266A 0F84F2000000  JZ       NEAR 00452762

0167:00452670 8B4DFC       MOV      ECX,[EBP-04]

0167:00452673 50           PUSH     EAX

0167:00452674 50           PUSH     EAX

0167:00452675 8D45B0       LEA      EAX,[EBP-50]

0167:00452678 8B11         MOV      EDX,[ECX]

0167:0045267A 50           PUSH     EAX

0167:0045267B FF5210       CALL     NEAR [EDX+10]

0167:0045267E 8BF0         MOV      ESI,EAX

0167:00452680 6A08         PUSH     BYTE +08

0167:00452682 59           POP      ECX

0167:00452683 0FB63E       MOVZX    EDI,BYTE [ESI]

0167:00452686 8BC7         MOV      EAX,EDI

0167:00452688 6A01         PUSH     BYTE +01

0167:0045268A 99            CDQ

0167:0045268B F7F9        IDIV     ECX

0167:0045268D 8BC7         MOV      EAX,EDI

0167:0045268F 5B           POP      EBX

0167:00452690 C1E803       SHR      EAX,03

0167:00452693 0FB6443001  MOVZX    EAX,BYTE [EAX+ESI+01]

0167:00452698 8BCA         MOV      ECX,EDX

0167:0045269A D3E3         SHL      EBX,CL

0167:0045269C 23D8         AND      EBX,EAX

0167:0045269E 7403         JZ       004526A3

0167:004526A0 6A01         PUSH     BYTE +01

0167:004526A2 5B           POP      EBX

0167:004526A3 8A4604       MOV      AL,[ESI+04]

0167:004526A6 6A08         PUSH     BYTE +08

0167:004526A8 0C04         OR       AL,04

0167:004526AA 59           POP      ECX

0167:004526AB C0E002       SHL      AL,02

0167:004526AE 884604       MOV      [ESI+04],AL

0167:004526B1 8BC7         MOV      EAX,EDI

0167:004526B3 99            CDQ

0167:004526B4 F7F9        IDIV     ECX

0167:004526B6 6A08         PUSH     BYTE +08

0167:004526B8 8BC8         MOV      ECX,EAX

0167:004526BA 8BC7         MOV      EAX,EDI

0167:004526BC 99            CDQ

0167:004526BD 5F           POP      EDI

0167:004526BE F7FF        IDIV     EDI

0167:004526C0 85DB         TEST     EBX,EBX

0167:004526C2 7521         JNZ      004526E5

0167:004526C4 8D7C3101     LEA      EDI,[ECX+ESI+01]

0167:004526C8 8D4A01       LEA      ECX,[EDX+01]

0167:004526CB B001         MOV      AL,01

0167:004526CD D2E0         SHL      AL,CL

0167:004526CF 8BCA         MOV      ECX,EDX

0167:004526D1 F6D0         NOT      AL

0167:004526D3 88450B       MOV      [EBP+0B],AL

0167:004526D6 B001         MOV      AL,01

0167:004526D8 D2E0         SHL      AL,CL

0167:004526DA 8A4D0B       MOV      CL,[EBP+0B]

0167:004526DD 0A07         OR       AL,[EDI]

0167:004526DF 22C8         AND      CL,AL

0167:004526E1 880F         MOV      [EDI],CL

0167:004526E3 EB14         JMP      SHORT 004526F9

0167:004526E5 8D443101     LEA      EAX,[ECX+ESI+01]

0167:004526E9 B101         MOV      CL,01

0167:004526EB 884D0B       MOV      [EBP+0B],CL

0167:004526EE 8BCA         MOV      ECX,EDX

0167:004526F0 8A550B       MOV      DL,[EBP+0B]

0167:004526F3 D2E2         SHL      DL,CL

0167:004526F5 F6D2         NOT      DL

0167:004526F7 2010         AND      [EAX],DL

0167:004526F9 0FB606       MOVZX    EAX,BYTE [ESI]

0167:004526FC 894508       MOV      [EBP+08],EAX

0167:004526FF 6A08         PUSH     BYTE +08

0167:00452701 99            CDQ

0167:00452702 59           POP      ECX

0167:00452703 F7F9        IDIV     ECX

0167:00452705 8BF8         MOV      EDI,EAX

0167:00452707 8D4701       LEA      EAX,[EDI+01]

0167:0045270A 3D80000000   CMP      EAX,80

0167:0045270F 7E02         JNG      00452713

0167:00452711 33C0         XOR      EAX,EAX

0167:00452713 8A4C3001     MOV      CL,[EAX+ESI+01]

0167:00452717 8D443001     LEA      EAX,[EAX+ESI+01]

0167:0045271B 80C904       OR       CL,04

0167:0045271E C0E902       SHR      CL,02

0167:00452721 8808         MOV      [EAX],CL

0167:00452723 A1E86C4B00   MOV      EAX,[004B6CE8]

0167:00452728 99            CDQ

0167:00452729 B9DE100000   MOV      ECX,10DE

0167:0045272E F7F9        IDIV     ECX

0167:00452730 85D2         TEST     EDX,EDX

0167:00452732 750F         JNZ      00452743

0167:00452734 8B4508       MOV      EAX,[EBP+08]

0167:00452737 6A08         PUSH     BYTE +08

0167:00452739 8D4C3E02     LEA      ECX,[ESI+EDI+02]

0167:0045273D 5F           POP      EDI

0167:0045273E 99            CDQ

0167:0045273F F7FF        IDIV     EDI

0167:00452741 0811         OR       [ECX],DL

0167:00452743 FF05E86C4B00 INC      DWORD [004B6CE8]

0167:00452749 8A4601       MOV      AL,[ESI+01]

0167:0045274C A804         TEST     AL,04

0167:0045274E 7409         JZ       00452759-------->NOP 掉它!!!!!!!!

0167:00452750 33C9         XOR      ECX,ECX

0167:00452752 85DB         TEST     EBX,EBX

0167:00452754 0F94C1       SETZ     CL-------->CL 应为 1

0167:00452757 8BD9         MOV      EBX,ECX-------->赋值

0167:00452759 F6D0         NOT      AL

0167:0045275B 85DB         TEST     EBX,EBX-------->ebx 不应为 0

0167:0045275D 884601       MOV      [ESI+01],AL

0167:00452760 7539         JNZ      0045279B-------->应该让它跳！

0167:00452762 837DFC00     CMP      DWORD [EBP-04],BYTE +00

0167:00452766 7411         JZ       00452779

0167:00452768 8B4DFC       MOV      ECX,[EBP-04]

0167:0045276B E819F3FAFF   CALL     00401A89

0167:00452770 FF75FC       PUSH     DWORD [EBP-04]

0167:00452773 E8002F0400   CALL     00495678

0167:00452778 59           POP      ECX

0167:00452779 8B75D4       MOV      ESI,[EBP-2C]

0167:0045277C 83C604       ADD      ESI,BYTE +04

0167:0045277F 8975D4       MOV      [EBP-2C],ESI

0167:00452782 833E00       CMP      DWORD [ESI],BYTE +00

0167:00452785 740D         JZ       00452794

0167:00452787 8D4DF0       LEA      ECX,[EBP-10]

0167:0045278A E8B5970100   CALL     0046BF44

0167:0045278F E975FDFFFF   JMP      00452509-------->大循环，跳到上面第一行！
```



# BrainsBreaker 破解分析

## 汇编代码片段

```
0167:00452794 33F6             XOR      ESI,ESI
0167:00452796 E9ED010000       JMP      00452988-------->此 JMP 跳过了注册成功的 CALL！
0167:0045279B 33FF             XOR      EDI,EDI
0167:0045279D 397DD0           CMP      [EBP-30],EDI
0167:004527A0 7C11             JL       004527B3
0167:004527A2 FF75D0           PUSH     DWORD [EBP-30]
0167:004527A5 BBB05C4B00       MOV      EBX,004B5CB0
0167:004527AA 8BCB             MOV      ECX,EBX
0167:004527AC E865730000       CALL     00459B16
0167:004527B1 EB05             JMP      SHORT 004527B8
0167:004527B3 BBB05C4B00       MOV      EBX,004B5CB0
0167:004527B8 393D845A4B00     CMP      [004B5A84],EDI
0167:004527BE 7518             JNZ      004527D8
0167:004527C0 6A05             PUSH     BYTE +05
0167:004527C2 C705BC584B000100+MOV     DWORD [004B58BC],01
0167:004527CC E821560000       CALL     00457DF2
0167:004527D1 59               POP      ECX
0167:004527D1 59               POP      ECX
0167:004527D2 893DBC584B00     MOV      [004B58BC],EDI
0167:004527D8 6848010000       PUSH     DWORD 0148
0167:004527DD E8D0E20200       CALL     00480AB2
0167:004527E2 3BC7             CMP      EAX,EDI
0167:004527E4 59               POP      ECX
0167:004527E5 7410             JZ       004527F7
0167:004527E7 FF75DC           PUSH     DWORD [EBP-24]
0167:004527EA 8BC8             MOV      ECX,EAX
0167:004527EC FF75C8           PUSH     DWORD [EBP-38]
0167:004527EF FF75F4           PUSH     DWORD [EBP-0C]
0167:004527F2 E83660FEFF       CALL     0043882D-------->此 CALL 注册成功！
0167:004527F7 8B45EC           MOV      EAX,[EBP-14]
0167:004527FA FF7004           PUSH     DWORD [EAX+04]
0167:004527FD FF1564754A00     CALL     `USER32!UpdateWindow`
0167:00452803 8B75FC           MOV      ESI,[EBP-04]
0167:00452806 8D4508           LEA      EAX,[EBP+08]
0167:00452809 6A01             PUSH     BYTE +01
```

`/\\zest/\\2000.2`

### 半破 BrainsBreaker 3.0

改 1 字节！一点补充。

1. 首先：在 `BBRK.INI` 加入：

   ```ini
   [PACKBrainsBreaker]
   0=zest
   1=aaaa
   2=1076e9867711e9867711e9867711e9867711
   3=2100
   4=4e37d9b1
   5=5533cba4
   ```

   详见上篇找出注册假成功部分！

2. `[HKEY_CLASSES_ROOT\BrainsBreaker_Puzzle\ID]`
   `@="af123835144e2f34a8a56cbb980357a8b43055347f980017"`

   因为注册假成功还会在注册表改为此值。如你已经注册假成功了则不需改动。

   详见上篇怎样注册假成功部分！

3. 改：`BBRK32.EXE`

   ```asm
   0167:00426B85 1BC0             SBB      EAX,EAX
   0167:00426B87 40               INC      EAX
   0167:00426B88 85C0             TEST     EAX,EAX
   0167:00426B8A 0F8504010000     JNZ      NEAR 00426C94

               ^^
               84

   0167:00426B90 8D8500FFFFFF     LEA      EAX,[EBP+FFFFFF00]
   0167:00426B96 8BCB             MOV      ECX,EBX
   0167:00426B98 50               PUSH     EAX
   ```

4. 运行程序已无功能限制！在 `Options` 显示注册成功。
5. 但有文件被改动的 `nag` 和自己做的拼图有限制。
6. 注册码不知怎么找！估计就在附近！没找到对比处，太乱。
7. `00452337`～`00452A4B` 是检测注册码正确与否部分，`004265C4`～`00426D0B` 是文件启动检测是否注册部分，程序段非常相似，我是依此找突破口的。
8. 你可把上 `[PACKBrainsBreaker]` 改 `[PACKFull]`、`[PACKEntry]`、`[PACKCountry]` 等试试，`[PACKFull]` 有问题，`[PACKCountry]` 是 `DEMO`。

`/\\zest/\\2000.2`

## Chapter 3: BrainsBreaker v2.1

**Tools used:** `Softice`, `Wdasm` (or `IDA`)

**Output:** [Brains Breaker v2.1 crack](http://fravia.org/99solu/bbcrack.zip)

Indeed a fun game to play! In fact I spent about 2-3 hours just playing the game before I started to crack it. It seems like some puzzles are only demos? Hmm, must be a nasty bug...but mr. Juan Trujillo Tarradas dont have to worry, we'll fix that for him! We'll just go to the "enter registraion name/serial" window and crack it like any other program... yeah...right...eh..only one problem... where is it?! At times like this I always go to the helpfile to try to find some usefull information, and look what I found:

```
Once your payment is processed, you'll receive an e-mail message containing your
registration code with the instructions to enter it in the program that makes
fully playable all the puzzles (if registered the Full pack) or the puzzles in the Entry set.
```

Ah...The registration window is hidden! How to find it then?

### Step 1 - finding the registration window!

I choosed to disasemble the program, and searched after some well- choosen strings like "registration". I found this hit (after disasming with `Wdasm`, no need to use `IDA` if we dont have to):

```
* Possible StringData Ref from Data Obj ->"Registration info. Program version "
```

Interesting indeed! I browsed down a bit through the code and found alot of even more interesting strings there, including `Pack`, `Name`, `Your ID` and `Key:`. Looks like this could be the place where we enter our registration details. Now we want to know how we execute this code, so we browse upwards through the code. We notice that this code is called from another location:

```
* Referenced by a CALL at Address:
|:0044C881
```

Now we know where this code was executed from. Go up to that location and have a look:

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0044B8CF(C)
|
:0044C852 6A06                    push 00000006
:0044C854 FFB381010000            push dword ptr [ebx+00000181]
:0044C85A E8CF49FDFF              call 0042122E
:0044C85F E9EB0C0000              jmp 0044D54F
:0044C864 6A1C                    push 0000001C
:0044C866 E8723FFDFF              call 004207DD
:0044C86B 898590FEFFFF            mov dword ptr [ebp+FFFFFE90], eax
:0044C871 85C0                    test eax, eax
:0044C873 0F84D60C0000            je 0044D54F
:0044C879 6A00                    push 00000000
:0044C87B FFB590FEFFFF            push dword ptr [ebp+FFFFFE90]
:0044C881 E85FE40100              call 0046ACE5		<-- here is the intersting call
:0044C886 E9C40C0000              jmp 0044D54F
```

Hmm, judging from this code, we'd say that the call to the interesting routine will never be executed because the program will jump at offset `44C85F`, and we dont see any other jump references...Here is a good example of how bad `wdasm` disassembles sometimes. There ARE references to the interesting call, if you use `IDA` you'll see them. The problem is that it referenced through the instruction `jmp dword ptr [4*eax+0044B96B]` and that's why `Wdasm` doesnt find it. Go to offset `44B964`, you'll find this offset using `IDA`. (It is possible to find it using `Wdasm` as well but it takes some more work. As you can see from the code snippet above, we'd like to jump to either offset `44C864` or `44C879` to execute our "enter registraion information" call, so try a search on `44C864` and you'll land in a very intersting place!)

```
:0044B964 FF24856BB94400          jmp dword ptr [4*eax+0044B96B]
:0044B96B C7C34400                DWORD 0044C3C7
:0044B96F 5CC44400                DWORD 0044C45C
:0044B973 CEC44400                DWORD 0044C4CE
:0044B977 C0C84400                DWORD 0044C8C0
:0044B97B 64C84400                DWORD 0044C864   <-- you'll land here if you're using wdasm
:0044B97F C0C84400                DWORD 0044C8C0
```

Ah - great! At offset `44B964`, based on the value in `eax`, a jump is taken. So in order to execute our interesting call, `eax` should hold a value of `4` (`4h*4h = 10h. 44B96Bh+10h = 44B97Bh` and that's the offset where the interesting jump address is stored).



```
:0044B8A2 8B4607                  mov eax, dword ptr [esi+07]   
:0044B8A5 3D50040000              cmp eax, 00000450
:0044B8AA 0F8F99000000            jg 0044B949
:0044B8B0 0F849A0E0000            je 0044C750
...
:0044B949 3D68040000              cmp eax, 00440468
:0044B94E 7F6B                    jg 0044B9BB
:0044B950 0F84EB080000            je 0044C241
:0044B956 05AEFBFFFF              add eax, FFFFFBAE	;Same as "sub eax,452"
:0044B95B 83F813                  cmp eax, 00000013
:0044B95E 0F875C0F0000            ja 0044C8C0
:0044B964 FF24856BB94400          jmp dword ptr [4*eax+0044B96B]
```

现在弄清楚我们需要什么值来执行“enter registation information”调用变得很容易了。首先，它必须大于`450h`，因为我们想要在偏移量`44B8AA`处进行跳转。它必须小于`468h`，因为我们不想在偏移量`44B94E`或`44B950`处进行跳转。从我们的值中减去`452`，如果结果小于或等于`13h`，则执行`44B964`处的跳转。我们已经知道，在执行跳转时我们希望`eax`为`4`，因此我们希望`eax`在偏移量`44B8A2`处为`452+4=456h`，以调出注册窗口。要验证这一理论是否正确，让我们在实践中测试它！启动游戏，并在`cs:44B8A2`处设置断点（确保在设置断点时`brainsbreaker`拥有控制权，否则`CS`不会指向`brainsbreaker`的代码）。现在选择一个菜单项，`softice`会因我们的断点而中断。你会看到一个值被移动到`eax`中（该值取决于我们选择的菜单项）。单步执行过指令，并使用`r`命令编辑`eax`寄存器。将值更改为`456`并让它继续运行...是的！“enter registraion information”窗口弹出了！

### Step 2 - What pack do we want?

`Entry pack`和另一个称为`Full pack`，因此我们可以很容易地猜测我们应该输入“full pack”、“full”或类似内容。要找出我们是否猜对了，让我们在第一个字段中输入“full pack”，并在其他字段中填入一些虚假信息。现在我们继续像平常破解名称/序列保护时那样做，在一个字符串获取函数（`GetDlgItemTextA`、`GetWindowTextA`、`hmemcpy`或类似物，你知道它们：)）上设置断点，或者尝试在Windows消息（`bmsg`）方式上设置断点...做任何你想做的，只要你能中断即可，我选择在Windows消息`wm_gettext`上设置断点。

`bpr`

在`wm_gettext`上设置断点，然后跟踪代码。你会看到大量的`repnz scasb`函数（当`al=0`时，它被大量用于长度检查），以及几个`repz movsb`函数。只需单步执行它们，并在字符串每次被复制时设置一个新的断点。只有当你看到它被其他内容覆盖时，才移除旧的断点。

**4012BD**：

```
:004012BD 0FBE06                  movsx eax, byte ptr [esi]
:004012C0 50                      push eax
:004012C1 E8EE560000              call 004069B4
:004012C6 59                      pop ecx
:004012C7 8BD8                    mov ebx, eax
:004012C9 50                      push eax
:004012CA 0FBE17                  movsx edx, byte ptr [edi]
```

我们首先将输入字符串的第一个字母移动到`eax`中（在我们的例子中是`46h`，即'F'）。接下来几行做的是将该值复制到其他寄存器中（这并不是很有趣），但最后一条指令非常有趣。它从`edi`指向的位置获取一个字节并将其移动到`edx`中。我想知道`edi`指向什么，你不觉得吗？让我们找出来！在`softice`中执行“`d edi`”，现在查看数据窗口。“Entry Full Upgrade...”等等。太好了！我们现在知道（好吧，不是完全知道，但这是一个非常好的猜测）我们应该在第一个编辑框中输入“Full”，因为我们确实想要完全注册版本，不是吗？

### Step 3 - Examining the protection scheme

真正的痛苦开始了，真正的保护方案。因此，从在编辑框中输入一些内容开始，我输入了“Full”、“Cruehead”、“111”、“1230123”。现在我们像上次一样做，但这次我们只对程序对我们的输入序列做什么感兴趣，因此我们选择在`1230123`（在我的例子中）这个字符串上设置断点。故事跟以前一样，大量的长度检查和字符串复制...我知道这很无聊，但坚持下去。过一段时间后，你会到达偏移量`401440`。这是它最终变得有点有趣的地方。

程序所做的是比较序列中的所有字符与“-”，但由于我们的序列没有“-”，我们继续执行而不做任何改变。在更多无趣的长度检查之后，我们再次到达相同的偏移量，但这次程序将序列中的每个字符与“0”进行比较。如果它找到匹配项，零将被移除，并替换为“O”。我们的新代码将是“123O123”。在`softice`中花了一段时间后，我们再次发现自己在这个偏移量处。这次程序检查序列中是否有字符为'1'。如果有，将它们替换为“L”。我们的新代码将是“L23OL23”。在此之后，序列中的每个字符都将与（按顺序，以十六进制）`00`、`3B`、`0D`和`0A`进行比较。我们不必关心这个，因为我们的序列中不会出现这些字节。

**46AC55**和这里是有趣部分开始的地方。



# Reverse Engineering Analysis

## Segment 1: Output Generation Loop

**Assembly Code:**

```asm
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0046ACB3(C)
|
:0046AC4C FF35BC554800            push dword ptr [004855BC]
:0046AC52 8B45E8                  mov eax, dword ptr [ebp-18]
:0046AC55 0FBE10                  movsx edx, byte ptr [eax]	<-- 40="" 43="" 46="" 50="" 52="" 59="" 7503="" 201431="" we'll="" land="" here="" :0046ac58="" push="" edx="" :0046ac59="" e856bdf9ff="" call="" 004069b4="" :0046ac5e="" pop="" ecx="" :0046ac5f="" eax="" :0046ac60="" e8bdbafaff="" 00416722="" :0046ac65="" 8845ef="" mov="" byte="" ptr="" \[ebp-11\],="" al="" :0046ac68="" 33c0="" xor="" eax,="" \*="" referenced="" by="" a="" (u)nconditional="" or="" (c)onditional="" jump="" at="" address:="" |:0046aca5(c)="" |="" :0046ac6a="" 8bc8="" ecx,="" :0046ac6c="" ba01000000="" edx,="" 00000001="" :0046ac71="" d3e2="" shl="" cl="" :0046ac73="" 33c9="" :0046ac75="" 8a4def="" cl,="" \[ebp-11\]="" :0046ac78="" 23d1="" and="" :0046ac7a="" 740e="" je="" 0046ac8a="" :0046ac7c="" 8bcb="" ebx="" :0046ac7e="" b201="" dl,="" 01="" :0046ac80="" d2e2="" :0046ac82="" 8b4df6="" dword="" \[ebp-0a\]="" :0046ac85="" 081431="" \[ecx+esi\],="" dl="" :0046ac88="" eb0e="" jmp="" 0046ac98="" |:0046ac7a(c)="" :0046ac8a="" :0046ac8c="" :0046ac8e="" :0046ac90="" f6d2="" not="" :0046ac92="" :0046ac95="" |:0046ac88(u)="" :0046ac98="" inc="" :0046ac99="" 83fb08="" cmp="" ebx,="" 00000008="" :0046ac9c="" jne="" 0046aca1="" :0046ac9e="" esi="" :0046ac9f="" 33db="" |:0046ac9c(c)="" :0046aca1="" :0046aca2="" 83f805="" 00000005="" :0046aca5="" 7cc3="" jl="" 0046ac6a="" :0046aca7="" ff45f0="" \[ebp-10\]="" :0046acaa="" ff45e8="" \[ebp-18\]="" :0046acad="" 8b45f0="" :0046acb0="" 3b45fc="" \[ebp-04\]="" :0046acb3="" 7c97="" 0046ac4c="" <="" pre="">
```

**Analysis:**

Whoah, quite some code there! But don't worry, we really don't have to understand it (as you will see later), we just need to know where the output is stored. The code here isn't very hard to understand either, but we'll deal with it later. To save some time, I'll tell you now that output will be created at offset `46AC85` and `46AC95`. You will see that after the loop, a few bytes (how many depends on the length of the serial entered) have been generated from our serial and saved at the address that `ecx+esi` points to at offset `46AC85`. You can now remove all breakpoints and put one (breakpoint on memory range - `bpr`) on our newly created bytes (in our case it will be 5 of them). Just to make sure you got the correct spot, the bytes for `"L23OL23"` is `4B,6F,B7,F4,06`. As you will see this string will be copied around a bit, but just follow the same procedure as last time (same procedure as every year, James). After a little while you'll land in the next interesting spot:

## Segment 2: XOR Check Loop

**Assembly Code:**

```asm
:00459B50 8BC1                    mov eax, ecx
:00459B52 99                      cdq
:00459B53 F7FB                    idiv ebx
:00459B55 8B45E4                  mov eax, dword ptr [ebp-1C]
:00459B58 03C2                    add eax, edx
:00459B5A 8A16                    mov dl, byte ptr [esi]	<-- 41="" 46="" 3010="" you'll="" be="" here="" :00459b5c="" xor="" byte="" ptr="" \[eax\],="" dl="" :00459b5e="" inc="" ecx="" :00459b5f="" esi="" \*="" referenced="" by="" a="" (u)nconditional="" or="" (c)onditional="" jump="" at="" address:="" |:00459b4e(u)="" |="" :00459b60="" 8b45fc="" mov="" eax,="" dword="" \[ebp-04\]="" :00459b63="" 2bc3="" sub="" ebx="" :00459b65="" 3bc8="" cmp="" ecx,="" eax="" :00459b67="" 7ce7="" jl="" 00459b50="" <="" pre="">
```

**Analysis:**

Not that hard to understand. One byte will be XOR'd with each of our new bytes, and a few others as well (these are constants). This final byte will have a value of `0E` in our case. You should always be very suspicious when you see an instruction that uses `XOR`. The only exception is when it's followed by two registers of the same kind, because then (as you all probably already know) it just sets the register to zero. Now after this loop we'll get to the first check. If you start stepping through the code you'll soon see that the byte which we're XOR'd with our "registration bytes" will be compared to zero. If it matches, go on, otherwise it's a fake code. Finally we're getting somewhere! We now know that after quite a lot of different manipulations of our serial (remember that we had two different manipulations, the first created a couple of bytes based on our serial, and the other XOR'ed them together along with a few other constant bytes) we shall have a final value of zero! Remember this as we'll use it later, but for now, just change the program so it continues, preferably by changing the final byte from `0E` to `00` (you should always try to patch as little as possible, at least while investigating a protection scheme). One can almost guess that somewhere nearby there will be another (or more) checks, and therefore we start stepping through the next piece of code. This is pretty hard, you have to feel where something interesting happens, but I'll tell you that at offset `459D01` something is happening:

## Segment 3: Validation Logic

**Assembly Code:**

```asm
:00459D01 3B4DEC                  cmp ecx, dword ptr [ebp-14]
:00459D04 7D4F                    jge 00459D55

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00459D53(C)
|
:00459D06 8BC1                    mov eax, ecx
:00459D08 99                      cdq
:00459D09 F7FF                    idiv edi
:00459D0B 8B45E0                  mov eax, dword ptr [ebp-20]
:00459D0E 8A1C10                  mov bl, byte ptr [eax+edx]
:00459D11 0FB745F0                movzx eax, word ptr [ebp-10]
:00459D15 3BC8                    cmp ecx, eax
:00459D17 7D09                    jge 00459D22
:00459D19 0FB7D1                  movzx edx, cx
:00459D1C 8B45F4                  mov eax, dword ptr [ebp-0C]
:00459D1F 321C10                  xor bl, byte ptr [eax+edx]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00459D17(C)
|
:00459D22 0FB7560C                movzx edx, word ptr [esi+0C]
:00459D26 3BCA                    cmp ecx, edx
:00459D28 7D09                    jge 00459D33
:00459D2A 0FB7C1                  movzx eax, cx
:00459D2D 8B5610                  mov edx, dword ptr [esi+10]
:00459D30 321C02                  xor bl, byte ptr [edx+eax]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00459D28(C)
|
:00459D33 0FB74614                movzx eax, word ptr [esi+14]
:00459D37 3BC8                    cmp ecx, eax
:00459D39 7D09                    jge 00459D44
:00459D3B 0FB7D1                  movzx edx, cx
:00459D3E 8B4618                  mov eax, dword ptr [esi+18]
:00459D41 321C10                  xor bl, byte ptr [eax+edx]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00459D39(C)
|
:00459D44 8BC1                    mov eax, ecx
:00459D46 99                      cdq
:00459D47 F7FF                    idiv edi
:00459D49 8B45E0                  mov eax, dword ptr [ebp-20]
:00459D4C 881C10                  mov byte ptr [eax+edx], bl
:00459D4F 41                      inc ecx
:00459D50 3B4DEC                  cmp ecx, dword ptr [ebp-14]
:00459D53 7CB1                    jl 00459D06
```

**Analysis:**

A pretty easy to understand protection here. Take the first char in the `"pack"` field, `xor` it with the first char in the `"Username"` field and finally `xor` this with the first char in the `"ID"` field. Save this byte and then proceed with the second character, the third and so on. I suppose a table will describe it better:

## XOR Operation Table

| Pack string: | 46 | 75 | 6C | 6C | | | | |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Username String: | 43 | 72 | 75 | 65 | 68 | 65 | | |
| ID string: | 31 | 32 | 33 | | | | | |
| Result after XOR: | **34** | **35** | **2A** | **09** | **68** | **65** | | |



"Hey! You only got `6` characters in the table, but the strings can be longer (and one of them in fact is)!" Calm down, That's because something is happening after `6` bytes has been processed. Now the first byte in our "result string" (`34` in our case) will also be included in the scheme. So, `34` will be `xor'ed` to every `7'th` char in the other strings, and in our case the "username" string will be the only one effected, as the others are too small. The result is written in the first position in the "result string", thus removing the old value. After `6` more bytes, it does the same again, thus keeping the size of the result string to `6` bytes. When `20h` (= `32` dec) characters has been processed the routine is done.

Something has to be happening with these `6` bytes (why write that routine otherwise), so lets put a `bpr` on them. You will break in a string comparing routine. So...this string is compared to another string! Very interesting indeed... I wonder what happends if they matches??? let's see! Change one of the strings so it matches the other and let the program run...

**`YEEEEES! Program registered!`**

So, now all we have to do is to make the strings match...just one BIG problem...One would have thought that the string that our "pack, username, ID" string was compared with should be constant...but this is not true! It changes everytime, so there is not a chance to make these strings match eachother! First I thought this was some anti-cracking feature, but that is not true either. I changed the program so it would display a little `messagebox` with the first bytes of the string, and I ran it without `softice` (for the first time in months, `softice` wasnt loaded on my machine :)). No luck, it still changed! One alternative still remains...`patching`!

**`Step 4 - Cracking it!`** First of all I'd like to say that I dont really like `patches` because one never knows what comes with a patch. It could be a nasty `checksum`, or even worse - a delayed `checksum`. That's why I always like to find/calculate a valid `serial` when cracking `name/serial` protections, but when you're out of ideas, I guess `patching` can be justified. But even if we are going to patch, let's patch as little as we have too. We dont have to patch the first `check` (remember? one byte is compared to `zero`...) as we can pretty easily find a `serial` that justifies that. As most crackers are lazy by nature (well, atleast I am :)), lets just write a quick and dirty `bruteforcer`. You can find my source and the executable for it [here](http://fravia.org/99solu/bb.zip). Run it, and we'll get the `serial` `125000`. If our `bruteforcer` does what it's suppose to do we would now get by the first `check` by using this `serial`, so let's test it! Go back to the `register` window, and enter the same information as before, but change the `serial` to `125000`. Wow! It worked! Of course we didnt get the "thanks for registering" message, but we didnt expect that either, because the last `check` is still there. For reasons that I told you about earlier we have to patch that (or spend the next couple of weeks trying to figure out a way how to crack it without patching, but as I said - we're all lazy).

Of course the next question will be *`Where shall we patch?`* Before we can answer that we have to make sure we understand what we want to do. In this case we want two strings to match eachother (or atleast making the program think they do). If we investigate the `comparision` routine we see that the routine compares the number of bytes specified in `edi`. In our case `edi=6` and therefor `6` bytes are compared. If we look at the code right after the comparison routine we will also see that `dl` is set to `1` if the strings matches. If they dont `dl` is set to `zero`. Now we can make a small list of possible `patches`:

```
1)   Action: Set edi to zero so that zero bytes will be compared.
Consequence: The strings wont be compared.
2)   Action: Overwrite the two strings with the same 6 bytes.
Consequence: The strings will match.
3)   Action: Fake the return value so dl will be set to one either way.
Consequence: The program thinks that the strings matches.
4)   Action: Fake a conditional jump
Consequence: The program thinks that the strings matches.
```

There are even more possibilites but these are more than enough. We can choose any of these and the program would, in theory, think our entered `serial` is valid.

Let's try for example to set `edi` to `zero`. First of all lets find where the program sets `edi` to `6`:

```
:00459C43 8B7508                  mov esi, dword ptr [ebp+08]
:00459C46 807E2400                cmp byte ptr [esi+24], 00
:00459C4A 7415                    je 00459C61
:00459C4C BF06000000              mov edi, 00000006
```

Can we change this into "`mov edi,0`" instead? Yes we can but the program would crash later due to a couple of "`IDIV`" instructions that relies on `edi` not beeing zero. So where can we change `edi` to `zero` then? Look at this:

```
:00459CFF 33C9                    xor ecx, ecx
:00459D01 3B4DEC                  cmp ecx, dword ptr [ebp-14]
:00459D04 7D4F                    jge 00459D55
...
```

This is the begining of the code that creates the `6` bytes long string from our entered Name, ID and Pack. This code is completely useless if we change `edi` to `zero` (because `zero` bytes will be compared) so we can saftly patch here. Let's change the code so it looks like this instead:

```
:00459CFF 33FF                    xor edi,edi	;set edi to zero
:00459D01 EB52                    jmp 00459D55	;Jump over the rest
						;of the function
```

Oh yeah! This works very nicely! Patch it using your favourite `hex-editor` and run it.

*`Integrity check:`

`Program seems to be altered from his original contents`

Of course...a `checksum` function as well. How can we crack this then?

In fact, the author has already done half the work for us. Notice how the message is presented - through a `messagebox`! Very good news for us crackers, we just have to put a `bpx` on `MessageBoxA` now in order to land right in the interesting code. So, do just that and trace upward and you'll see this:

```
:00442CC2 E870DBFFFF              call 00440837
:00442CC7 3D22334455              cmp eax, 55443322	;Whoah! What a nice hardcoded value!
:00442CCC 7472                    je 00442D40		;Take this jump if you want to jump
							;over the messagebox!
:00442CCE 6810200100              push 00012010
:00442CD3 FF35843B4800            push dword ptr [00483B84]
:00442CD9 83C4FC                  add esp, FFFFFFFC
:00442CDC 66C70424BE00            mov word ptr [esp], 00BE
:00442CE2 E8C16FFFFF              call 00439CA8
:00442CE7 50                      push eax
:00442CE8 6A00                    push 00000000
:00442CEA E80AEF0300		  call USER32!MessageBoxA
```

You can see directly that something is fuzzy about the "`cmp eax, 55443322`" line. That looks VERY suspicious if you ask me (why not say "Hey Cracker! I'm here!!!" instead). Now we could quickly crack this just by changing the "`je 00442D40`" instruction to "`jmp 00442D40`" but we wont do that. The program might save the value returned in `eax` and use it later on for more `checksum-checking` (this is infact true for this program) so instead we want the above `call` to return `55443322`. So, let's trace the `call`! You'll soon find this piece of code:

```
:00440DAE E8B504FCFF              call 00401268
:00440DB3 83C40C                  add esp, 0000000C
:00440DB6 85C0                    test eax, eax
:00440DB8 752E                    jne 00440DE8
:00440DBA A1803B4800              mov eax, dword ptr [00483B80]
:00440DBF 8B10                    mov edx, dword ptr [eax]
:00440DC1 8B0D243B4800            mov ecx, dword ptr [00483B24]
:00440DC7 8911                    mov dword ptr [ecx], edx
:00440DC9 B822334455              mov eax, 55443322	;We want this!
:00440DCE 50                      push eax
```



Now, this isn't very hard either. As the program is already patched, the call at `440DAE` will return `eax <> 0` and thus the jump will be taken. Btw, that call is our good old "comparison" call. As you see, it's used quite a lot in the program, so it was a good idea not to patch there! Now just NOP away the jump. `55443322` will be stored in `eax` and the checksum routine thinks that everything is alright and `we have a fully functional registered copy of BrainsBreaker!`

### Step 5 - Making a nice crack!

You can skip this part if you want to. This step is not really necessary, but if we were to release a crack for this program, we had to make a nice looking patcher. The problem is that we don't know how to bring up the "enter registration info" window, and thus the user wouldn't have anywhere to enter his name. We can solve this by making our own little "registration window" that asks for a username in our patcher. If you look where `BrainsBreaker` saves the registration information, you will see that all of the reg-info is saved in the `BBRK.INI` file. It will save both plaintext and encrypted values. It will look something like this:

```
[PackFull]            ;What pack we're using
0=Cruehead            ;Username plain text
1=123                 ;ID plain text
2=7E15AB3D1516D6      ;Serial encrypted
3=2100                ;Not sure. I believe it's version number
4=192BEC6D323CF86C    ;Username encrypted
5=6B6BAA              ;ID encrypted
```

Of course I didn't know all of this before I fully examined the program. I started out by putting a breakpoint on `GetPrivateProfileStringA` and when the encrypted serial line was read I put a bpr on that so I would see what was happening to that string. I won't describe every step on how I did it because it's fairly easy. Instead I'll try to describe how it works:

Every two letters in `7E15AB3D1516D6` is in fact one byte. So first of all the string is transformed so it looks like this: `7E 15 AB 3D 15 16 D6`. 7 bytes that is. Now this string is XOR'ed against a four byte long string that was `5A 59 99 08`. This string caught my curiosity because it didn't look like a hardcoded value and neither could I find it in the `.EXE` file, but I continued nevertheless, not worrying about it. The string was XOR'ed like this:

| String from INI file: | `7E` | `15` | `AB` | `3D` | `15` | `16` | `D6` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Four byte string: | `5A` | `59` | `99` | `08` | `5A` | `59` | `99` |
| Result after XOR: | `24` | `4C` | `32` | `35` | `4F` | `4F` | `4F` |

You see? The result is `$L25OOO`. Our entered serial string (somewhat transformed). The same procedure is done with the two other encrypted strings. If the decrypted username matches the username in plain text and if the decrypted ID matches the ID in plain text, then the program executes the registration check with the decrypted serial. The program will gladly accept it because we have patched the last check!

Ok, I sat down one night and coded this crack. It worked great on my computer, but when I tried it on another computer, it simply didn't work! Back to the drawing board again. It turned out that I was right when thinking that the 4 bytes XOR string was strange. After some more examination, I found out that this value was created from a special ID which is stored in the registry. The ID is created the first time `brainsbreaker` is installed and there isn't much point in reversing the algorithm for creating a `BrainsBreaker` ID because it uses the `TlsGetValue` function as source. The complete ID is about 50 bytes long, but only 8 of these bytes are used to create the XOR string (if you're really interested, byte number 5, 10, 8, 13, 17, 19, 22, 38 of the ID is used to make the XOR string). That's why my crack didn't work on the other machine - when I installed `BrainsBreaker` for the first time a different ID was created and thus also a different XOR string. So I sat down another night and re-coded the crack, this time for Windows as we need to access the registry. This time the crack worked like a charm on both machines! Oh joy oh joy!

### Thoughts about BrainsBreaker

This is not important for the crack, but it's nonetheless very interesting. I'm not sure about this but it looks like every puzzle is a stand-alone part of the program. Every puzzle stores a number of things including the name of the author of the puzzle, if the main program has been altered, whether this puzzle is free/demo/registered and a number of other things. My guess is that these values (or flags) are stored directly after the main program checks if it's registered or not. This way every puzzle can check if the main program has been cracked. But we don't have to care about all of this because the crack I presented above takes care of everything :). None of these flags are stored directly in the `.exe` file, instead they are stored in encrypted state. At startup these puzzles are decrypted using different key sources, but the most common seems to be `(c)J. Trujillo T.` I managed to get this info by stumbling across a `xor [edi], al` function when the program was loading the startup screen with all the puzzles. That of course caught my interest and I started to examine it a bit more closely.

A small hint: It's a good idea to search the deadlisting for `MessageBoxA` functions as these can give you a lot of useful information sometimes. Fake a flag or something to make the program show the messagebox. For example, I found out that every puzzle stores if the main program has been altered or not (if it has, a value of `12345` is stored in each of the puzzles) using this method.

But there are some things that remain a mystery for me still. I haven't been able to figure out how to bring up the registration-window yet, and you can find a few very interesting functions if you look at the dead-listing:

```
Addr:00487ABC Ord:  14 (000Eh) Name: __DebuggerHookData
Addr:004035BC Ord:  15 (000Fh) Name: __lockDebuggerData(void)
Addr:004035E4 Ord:  16 (0010h) Name: __unlockDebuggerData(void)
```

Antidebugging functions probably, but I haven't seen any marks of them. Pretty strange...

### Conclusion - BrainsBreaker:

A somewhat complicated protection scheme, but nothing that we haven't seen before. The hidden registration window is a good idea, but it's far too easy to find it using a dead-listing approach.

-->-->-->

破 `BrainsBreaker`3.0[07b]!

`http://www.brainsbreaker.com/download.htm`

非常感谢 1212 提供的 `Cruehead` 的优秀教程。

如有疑问，请参看此教程!

`http://www.instinct.org/fravia/99solu/cruecom.htm`

===========================================================

#### 1、Key 运算检测，最后保证 `[eax]` 为 0，可 BPM 跟踪。

```
0167:00426D93 8BC1            MOV     EAX,ECX
0167:00426D95 99              CDQ
0167:00426D96 F7FF            IDIV    EDI
0167:00426D98 8D041A          LEA     EAX,[EDX+EBX]
0167:00426D9B 8A1431          MOV     DL,[ECX+ESI]
0167:00426D9E 3010            XOR     [EAX],DL    ******
0167:00426DA0 41              INC     ECX
0167:00426DA1 3B4DF8          CMP     ECX,[EBP-08]
0167:00426DA4 7CED            JL      00426D93---->循环 9 次
```

改为：

```
0167:00426D93 8BC1            MOV     EAX,ECX
0167:00426D95 99              CDQ
0167:00426D96 F7FF            IDIV    EDI
0167:00426D98 8D041A          LEA     EAX,[EDX+EBX]
0167:00426D9B 8A10            MOV     DL,[EAX]    ******
0167:00426D9D 90              NOP
0167:00426D9E 3010            XOR     [EAX],DL    ******we shall have a final value of zero
0167:00426DA0 41              INC     ECX
0167:00426DA1 3B4DF8          CMP     ECX,[EBP-08]
0167:00426DA4 7CED            JL      00426D93
```

===========================================================

#### 2、Pack、Name、Your ID 运算检测，对比处可 BPM 跟踪。

```
0167:00427587 33C9            XOR     ECX,ECX
0167:00427589 8BC1            MOV     EAX,ECX
```



0167:0042758B 668B7D0C `MOV DI,[EBP+0C]`

0167:0042758F 99 `CDQ`

0167:00427590 F77DFC `IDIV DWORD [EBP-04]`

0167:00427593 8B45F8 `MOV EAX,[EBP-08]`

0167:00427596 03D0 `ADD EDX,EAX`

0167:00427598 663B7DE4 `CMP DI,[EBP-1C]`

0167:0042759C 8A02 `MOV AL,[EDX]`

0167:0042759E 7306 `JNC 004275A6`

0167:004275A0 8B7DE8 `MOV EDI,[EBP-18]`

0167:004275A3 32040F `XOR AL,[EDI+ECX]`

0167:004275A6 668B7D0C `MOV DI,[EBP+0C]`

0167:004275AA 663B7E0C `CMP DI,[ESI+0C]`

0167:004275AE 7306 `JNC 004275B6`

0167:004275B0 8B7E10 `MOV EDI,[ESI+10]`

0167:004275B3 32040F `XOR AL,[EDI+ECX]`

0167:004275B6 668B7D0C `MOV DI,[EBP+0C]`

0167:004275BA 663B7E14 `CMP DI,[ESI+14]`

0167:004275BE 7306 `JNC 004275C6`

0167:004275C0 8B7E18 `MOV EDI,[ESI+18]`

0167:004275C3 32040F `XOR AL,[EDI+ECX]`

0167:004275C6 FF450C `INC DWORD [EBP+0C]`

0167:004275C9 8802 `MOV [EDX],AL`

0167:004275CB 0FB74D0C `MOVZX ECX,WORD [EBP+0C]`

0167:004275CF 83F920 `CMP ECX,BYTE +20`

0167:004275D2 7CB5 `JL 00427589---->循环 20h 次`

0167:004275D4 FF75F4 `PUSH DWORD [EBP-0C]`

改为：(乱改程序会崩溃!)

0167:00427587 33C9 `XOR ECX,ECX`

0167:00427589 C745FC00000000 `MOV DWORD [EBP-04],00` ******because zero bytes will be compared

0167:00427590 E93F000000 `JMP 004275D4` ******

0167:00427595 F8 `CLC`

0167:00427596 03D0 `ADD EDX,EAX`

0167:00427598 663B7DE4 `CMP DI,[EBP-1C]`

0167:0042759C 8A02 `MOV AL,[EDX]`

0167:0042759E 7306 `JNC 004275A6`

0167:004275A0 8B7DE8 `MOV EDI,[EBP-18]`

0167:004275A3 32040F `XOR AL,[EDI+ECX]`

0167:004275A6 668B7D0C `MOV DI,[EBP+0C]`

0167:004275AA 663B7E0C `CMP DI,[ESI+0C]`

0167:004275AE 7306 `JNC 004275B6`

0167:004275B0 8B7E10 `MOV EDI,[ESI+10]`

0167:004275B3 32040F `XOR AL,[EDI+ECX]`

0167:004275B6 668B7D0C `MOV DI,[EBP+0C]`

0167:004275BA 663B7E14 `CMP DI,[ESI+14]`

0167:004275BE 7306 `JNC 004275C6`

0167:004275C0 8B7E18 `MOV EDI,[ESI+18]`

0167:004275C3 32040F `XOR AL,[EDI+ECX]`

0167:004275C6 FF450C `INC DWORD [EBP+0C]`

0167:004275C9 8802 `MOV [EDX],AL`

0167:004275CB 0FB74D0C `MOVZX ECX,WORD [EBP+0C]`

0167:004275CF 83F920 `CMP ECX,BYTE +20`

0167:004275D2 7CB5 `JL 00427589`

0167:004275D4 FF75F4 `PUSH DWORD [EBP-0C]`

===========================================================

### 3、程序注册检验对比处及是否被修改检验处

\* Referenced by a CALL at Addresses:

`:00405FC3`, `:00420734`, `:004213E0`, `:004225A3`, `:00426DAE`

`:0042763A`, `:00427788`, `:00459AF8`, `:0046EBF0`, `:00470C39`

`:004800B5`, `:00491CC4`

`|:00495EC0 8B44240C mov eax, dword ptr [esp+0C]`

`:00495EC4 85C0 test eax, eax`

`:00495EC6 7444 je 00495F0C`

`:00495EC8 8B542404 mov edx, dword ptr [esp+04]`

`:00495ECC 56 push esi`

`:00495ECD 57 push edi`

`:00495ECE 8BF2 mov esi, edx`

`:00495ED0 8B7C2410 mov edi, dword ptr [esp+10]`

`:00495ED4 0BD7 or edx, edi`

`:00495ED6 83E203 and edx, 00000003`

`:00495ED9 7432 je 00495F0D`

`:00495EDB A901000000 test eax, 00000001`

`:00495EE0 740B je 00495EED`

`:00495EE2 8A0E mov cl, byte ptr [esi]`

`:00495EE4 3A0F cmp cl, byte ptr [edi]`

`:00495EE6 7552 jne 00495F3A`

`:00495EE8 46 inc esi`

`:00495EE9 47 inc edi`

`:00495EEA 48 dec eax`

`:00495EEB 741D je 00495F0A`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:00495EE0(C)`, `:00495F08(C)`

`:00495EED 8A0E mov cl, byte ptr [esi]`

`:00495EEF 8A17 mov dl, byte ptr [edi]`

`:00495EF1 38D1 cmp cl, dl`

`:00495EF3 7545 jne 00495F3A`

`:00495EF5 8A4E01 mov cl, byte ptr [esi+01]`

`:00495EF8 8A5701 mov dl, byte ptr [edi+01]`

`:00495EFB 38D1 cmp cl, dl`

`:00495EFD 753B jne 00495F3A`

`:00495EFF 83C702 add edi, 00000002`

`:00495F02 83C602 add esi, 00000002`

`:00495F05 83E802 sub eax, 00000002`

`:00495F08 75E3 jne 00495EED`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:00495EEB(C)`, `:00495F44(C)`

`:00495F0A 5F pop edi`

`:00495F0B 5E pop esi`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00495EC6(C)`

`:00495F0C C3 ret`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00495ED9(C)`

`:00495F0D 8BC8 mov ecx, eax`

`:00495F0F 83E003 and eax, 00000003`

`:00495F12 C1E902 shr ecx, 02`

`:00495F15 742B je 00495F42`

`:00495F17 F3 repz`

`:00495F18 A7 cmpsd`

`:00495F19 7427 je 00495F42`

`:00495F1B 8B4EFC mov ecx, dword ptr [esi-04]`

`:00495F1E 8B57FC mov edx, dword ptr [edi-04]`

`:00495F21 38D1 cmp cl, dl`

`:00495F23 7510 jne 00495F35`

`:00495F25 38F5 cmp ch, dh`

`:00495F27 750C jne 00495F35`

`:00495F29 C1E910 shr ecx, 10`

`:00495F2C C1EA10 shr edx, 10`

`:00495F2F 38D1 cmp cl, dl`

`:00495F31 7502 jne 00495F35`

`:00495F33 38F5 cmp ch, dh`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:00495F23(C)`, `:00495F27(C)`, `:00495F31(C)`, `:00495F4C(C)`, `:00495F53(C)`

`:00495F66(C)`

`:00495F35 B800000000 mov eax, 00000000`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:00495EE6(C)`, `:00495EF3(C)`, `:00495EFD(C)`

`:00495F3A 1BC0 sbb eax, eax`

`:00495F3C 5F pop edi`

`:00495F3D 83D8FF sbb eax, FFFFFFFF`

`:00495F40 5E pop esi`

`:00495F41 C3 ret`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:00495F15(C)`, `:00495F19(C)`

`:00495F42 85C0 test eax, eax`

`:00495F44 74C4 je 00495F0A`

`:00495F46 8B16 mov edx, dword ptr [esi]`

`:00495F48 8B0F mov ecx, dword ptr [edi]`

`:00495F4A 38CA cmp dl, cl`

`:00495F4C 75E7 jne 00495F35`

`:00495F4E 48 dec eax`

`:00495F4F 7418 je 00495F69`

`:00495F51 38EE cmp dh, ch`

`:00495F53 75E0 jne 00495F35`

`:00495F55 48 dec eax`

`:00495F56 7411 je 00495F69`

`:00495F58 81E10000FF00 and ecx, 00FF0000`

`:00495F5E 81E20000FF00 and edx, 00FF0000`



```
:00495F64 3BD1               cmp edx, ecx   

:00495F66 75CD               jne 00495F35   

:00495F68 48                 dec eax   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

|:00495F4F(C), :00495F56(C)   

|   

:00495F69 5F                 pop edi   

:00495F6A 5E                 pop esi   

:00495F6B C3                 ret   
```

不能乱改，否则崩溃，程序其他部分还会调用此处，此乃程序高明之处。

===========================================================

### 4、程序被修改提示框的去除

我用 `R!SC'sPatcher v1.4` 做 `PATCH`，非常简单，按 `1,2` 改。软件可在 `TOYE` 工具处下载。

我的 `bbrk32.rpp` 如下：

`F=bbrk32.exe:`

`O=cr_bbrk32.exe:`

`P=426d9c/14,31/10,90:`

`P=427589/8b,c1,66,8b,7d,0c,99,f7,7d,fc,8b,45/c7,45,fc,00,00,00,00,e9,3f,00,00,00:`

```
$
```

===========================================================

### 5、运行生成的 `cr_bbrk32.exe`

We have a fully functional `REGISTERED` copy of `BrainsBreaker3.0[07b]!`

===========================================================

```
/zest//2001.3.8  4dd466d0e1e0cc123afbe504da2462ca
```
