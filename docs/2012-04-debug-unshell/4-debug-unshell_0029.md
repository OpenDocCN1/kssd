
# ESP 脱壳 个人小节

> 原文：[`www.pediy.com/kssd/pediy06/pediy6696.htm`](https://www.pediy.com/kssd/pediy06/pediy6696.htm)

我学 ESP 定律也有 0.5 天了

总结了点东西想给大家分享～～～～～

## 关于 ESP 脱壳找 EOP

ESP 定律：就是加载程序后 `F8` 单步运行第一个变化的 `ESP` 值，右击寄存器上 `ESP` 在转存中跟随，在内存地址上选中前四个，右击，断点，硬件访问，`word` `F9`，中断，....返回。

(`D 12ffc0` 选四个下硬件写入 `dword`) 我的其它小记

### 1. `Aspcak` 方法：

`ESP`+`6175`(Sb) `POPAD` `JMP`

### 2. `UPX` 方法：

`S 0000` The First `JMP`

变种

`ESP`+`S` (`60E9`)

```
0040EA0E    60              PUSHAD
0040EA0F  - E9 B826FFFF     JMP chap702.004010CC
```

### 3. `PECompact` `1.68` - `1.84`

`ESP`

Or 注意第一个 `PUSH XXXX`   `XXXX`+基址(`400000`)=`EOP`

### 4. `EZIP` `1.0` 方法：

`ESP`

### 5. `JDPack` `1.x` / `JDProtect` `0.9`

`ESP`+`S`(`6150`)

```
0040E3F8    894424 1C       MOV DWORD PTR SS:[ESP+1C],EAX
0040E3FC    61              POPAD
0040E3FD    50              PUSH EAX
0040E3FE    C3              RETN
0040E3FF    23E8            AND EBP,EAX
```

### 6. `PE Pack` `1.0`

`ESP`+`S` (`61ff`)

```
0040D26F    61              POPAD
0040D270    FFE0            JMP EAX
```

### 7. `WWPack32` `1.x`

`ESP`

### 8. `PEDiminisher` `0.1`

`ESP`+`S`(`FFE0`) or `S`(`JMP EAX`)

```
0041708D    5D              POP EBP
0041708E    5F              POP EDI
0041708F    5E              POP ESI
00417090    5A              POP EDX
00417091    59              POP ECX
00417092    5B              POP EBX
00417093  - FFE0            JMP EAX
```

### 9. `DxPACK` `V0.86`

`ESP`+`S`(`JMP EAX or 61ffe0`)

```
0040D163    61              POPAD
0040D164  - FFE0            JMP EAX
```

### 10. `PKLITE32` `1.1`

`F8` 5 次来到 `EOP`

### 11. `32Lite` `0.03a`

`ESP` 往下找到

```
    PUSH EAX
       50c3 or PUSH EAX
       003780F4    50              PUSH EAX
       003780F5    C3              RETN
```

来到

```
       0041C53C    FF96 84B50100   CALL DWORD PTR DS:[ESI+1B584]
       0041C542    61              POPAD
       0041C543  - E9 0848FFFF     JMP QEDITOR.00410D50
```

注意：先用 `LoadPE` 转存 `Olldbg` 加载的那个加壳文件，退出，运行加壳后的文件，`RecImport` 修正 `EOP` 修复抓取文件到 `DUMP` 的那个文件。

### 12. `VGCrypt` `PE Encryptor` `V0.75`

`ESP` 安 `F9`(断点开始)，地址-1＝`EOP`

### 13. `PC Shrinker` `0.71`

`ESP`

```
00142BA8    BA CC104000     MOV EDX,4010CC
00142BAD    FFE2            JMP EDX
```

### 14. `Petite` `2.2`

1. `D 12ffa0`
   3 下 `F9`
   来到 `EOP`=地址-1

2. `ESP` 两下 同上

### 15. `EXE Stealth` `2.72`

`ESP`+`S`()

```
0040D49F 8B9D B62F4000 MOV EBX,DWORD PTR SS:[EBP+402FB6]
0040D4A5 039D BA2F4000 ADD EBX,DWORD PTR SS:[EBP+402FBA]
```

将堆栈中值 `10cc`+`400000`=`4010cc` 得出记事本的 `Oep`. 加壳软件的 `oep`.

```
0040D4AB C1CB 07 ROR EBX,7
```

注明：

例子： `ESP`+`S` (`60E9`)

后面 `S` 是搜索的意思，里面为搜索内容，可能是 `Ctrl+F` 的也可能是 `Ctrl+B`

反正大家是聪明人 一看就知道了～·

呵呵～

Cater 华夏

2004.08.30

`4dd466d0e1e0cc123afbe504da2462ca`
