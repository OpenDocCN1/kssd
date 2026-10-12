
# aspack 2.1

> 原文：[`www.pediy.com/kssd/pediy01/forum032.htm`](https://www.pediy.com/kssd/pediy01/forum032.htm)

工具：`softice`, `procdump`, `SoftICE Backdoor Keeper`, `frogsice 0.43`, `icedump`

`SoftICE Backdoor Keeper` 到这边抓

`http://protools.exit.de/files/debuggers/ex-sibdk1.zip`

他说这个应该是适合所有的 `softice` 版本，我用 `softice 4.05` 测试没问题

### 前言

这个档案也是用 `asprotect` 保护的

此版的 `asprotect` 会检测 `softice`，且还会用 `int 3` 去检测

只用 `frogsice` 是无效的，因此要配合 `SoftICE Backdoor Keeper`

把 `ex-sibdk1` 解开，放到 `softice` 的目录，运行他，他会去改 `winice.exe`

改完了再重开机，就可以下 `bpint 3` 了

若没有运行这个，下 `bpint 3` 是拦不到 `int 3` 的

还有一点，这个程序，未进入 `OEP` 之前

地址每次运行都会变，所以我写的地址，当然叶丝谕你的不一样

这个要注意的

ok, go!!

### 1. 先用 procdump 观看一下 aspack

`size of image` 是 `0007A000`

`image og base` 是 `00400000`

看一下 `section`

`.idata` 的 `visual offset` 是 `46000`

这个应该是放 `import table` 的地方

### 2. 运行 frogsice, 还有 icedump

按 `ctrl-d` 进入 `softice`，下 `bpint 3`

就是要拦截 `int 3`，因为有运行过

运行 `aspack2.1`

`aspack2.1` 会检测 `sice` 和 `ntice`

`sice` 要去欺骗他，`ntice` 不要欺骗他，不然会当机

到了拦截 `int 3` 时，就是这边，下 `e eip 0`

故意让他生成例外。在下 `bpm 446000`

为什么要下 `446000`？是因为 `400000+46000 = 446000`

```
014F:004D2AD5 CC                INT     3 ;  光标停在这边
014F:004D2AD6 EB02              JMP     004D2ADA
014F:004D2AD8 E9025DEB02        JMP     033887DF
014F:004D2ADD E9FF33C05A        JMP     5B0D5EE1
014F:004D2AE2 59                POP     ECX
014F:004D2AE3 59                POP     ECX
014F:004D2AE4 648910           MOV     FS:[EAX],EDX
014F:004D2AE7 EB0A             JMP     004D2AF3
014F:004D2AE9 E9121AFFFF       JMP     004C4500
014F:004D2AEE E8791BFFFF       CALL    004C466C
014F:004D2AF3 8A154C8F4D00     MOV     DL,[004D8F4C]
014F:004D2AF9 8B45F4           MOV     EAX,[EBP-0C]
```

### 3. 继续调试

下完了 `bpm 446000`，不要按 `F10` 或 `F8`，直接按 `F5`

按四次 `F5`，仔细看一下 `ds:446000` 的下方，此时 `import table` 应该是

正确的 `import table` 了，可以 `dump` 出来了

用 `icedump` 来 `dump`，下 `pagein d 446000 2000 c:\temp\aspack.import`

名字可以自定

再按 2 次 `F12`，按第一次 `F12` 会等约 10~20 秒才回到 `softice`

我也不知道为什么

按完了两次，会到这边

```
014F:004BEDB4 33C0             XOR     EAX,EAX
014F:004BEDB6 5A               POP     EDX
014F:004BEDB7 59               POP     ECX
014F:004BEDB8 59               POP     ECX
014F:004BEDB9 648910          MOV     FS:[EAX],EDX
014F:004BEDBC EB13            JMP     004BEDD1 ; 若过期了,当到了这边可下
      r eip 004BEE50 (当然,我的地址一定和你的地址不一样)
014F:004BEDBE E96914FFFF      JMP     004B022C
014F:004BEDC3 66B80D00        MOV     AX,000D
014F:004BEDC7 E870F1FFFF      CALL    004BDF3C
014F:004BEDCC E8C715FFFF      CALL    004B0398
014F:004BEDD1 33C0            XOR     EAX,EAX
014F:004BEDD3 55              PUSH    EBP
014F:004BEDD4 68F1ED4B00     PUSH    004BEDF1
014F:004BEDD9 64FF30        PUSH    DWORD PTR FS:[EAX]
014F:004BEDDC 648920        MOV     FS:[EAX],ESP
014F:004BEDDF 8B4508        MOV     EAX,[EBP+08]
014F:004BEDE2 E80DFCFFFF    CALL    004BE9F4 ; 会跑出过期的消息
014F:004BEDE7 33C0           XOR     EAX,EAX
014F:004BEDE9 5A             POP     EDX
014F:004BEDEA 59             POP     ECX
014F:004BEDEB 59             POP     ECX
014F:004BEDEC 648910        MOV     FS:[EAX],EDX
014F:004BEDEF EB13           JMP     004BEE04
014F:004BEDF1 E93614FFFF     JMP     004B022C
014F:004BEDF6 66B81100       MOV     AX,0011
014F:004BEDFA E83DF1FFFF     CALL    004BDF3C
014F:004BEDFF E89415FFFF     CALL    004B0398
014F:004BEE04 33C0           XOR     EAX,EAX
014F:004BEE06 55             PUSH    EBP
014F:004BEE07 683DEE4B00     PUSH    004BEE3D
014F:004BEE0C 64FF30        PUSH    DWORD PTR FS:[EAX]
014F:004BEE0F 648920        MOV     FS:[EAX],ESP
014F:004BEE12 B83CDF4B00     MOV     EAX,004BDF3C
014F:004BEE17 50             PUSH    EAX
014F:004BEE18 8B4508         MOV     EAX,[EBP+08]
014F:004BEE1B 8B4810         MOV     ECX,[EAX+10]
014F:004BEE1E 8B4508         MOV     EAX,[EBP+08]
014F:004BEE21 8B5004         MOV     EDX,[EAX+04]
014F:004BEE24 8B4508         MOV     EAX,[EBP+08]
014F:004BEE27 8B00           MOV     EAX,[EAX]
014F:004BEE29 E80AF6FFFF     CALL    004BE438
014F:004BEE2E A2305D4C00     MOV     [004C5D30],AL
014F:004BEE33 33C0           XOR     EAX,EAX
014F:004BEE35 5A             POP     EDX
014F:004BEE36 59             POP     ECX
014F:004BEE37 59             POP     ECX
014F:004BEE38 648910        MOV     FS:[EAX],EDX
014F:004BEE3B EB13           JMP     004BEE50
014F:004BEE3D E9EA13FFFF     JMP     004B022C
014F:004BEE42 66B80E00       MOV     AX,000E
014F:004BEE46 E8F1F0FFFF     CALL    004BDF3C
014F:004BEE4B E84815FFFF     CALL    004B0398
014F:004BEE50 8B4508         MOV     EAX,[EBP+08] SS:006AFE14=00465C57
014F:004BEE53 8D4818         LEA     ECX,[EAX+18]
014F:004BEE56 8B4508         MOV     EAX,[EBP+08]
014F:004BEE59 8B10           MOV     EDX,[EAX]
014F:004BEE5B 8B4508         MOV     EAX,[EBP+08]
014F:004BEE5E 8B401C         MOV     EAX,[EAX+1C]
014F:004BEE61 E85EF6FFFF     CALL    004BE4C4 ; 按 F8,追进去
014F:004BEE66 5F             POP     EDI
014F:004BEE67 5E             POP     ESI
014F:004BEE68 5B             POP     EBX
014F:004BEE69 59             POP     ECX
014F:004BEE6A 59             POP     ECX
```

在 `cs:014F:004BEE61 CALL 004BE4C4`

追进去之后，就可以看到 `POPAD`

```
PUSH EAX ; 此时 eax = 442238
CALL xxxxxxxx ; 按 F8 追进去
```

所以 `OEP` 是 `442238`

看要用 `icedump` 或是 `procdump` 来 `dump` 都可以

`dump` 完了之后

把刚刚的 `import table` 的信息覆盖到不正确的 `import table`

用 `procdump` 去改刚刚 `dump` 出来的 `exe`

改 `Entry point = 42238`，`Import table` 的 `RVA = 46000`

若是用 `icedump` `dump` 的，

还要多一个步骤

把所有的 `section` 的

`PSize = VSize`

`offset = RVA`

改完了，应该就没问题了，也没有过期了，检测 `softice` 了
