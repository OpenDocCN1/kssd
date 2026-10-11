
# Armadillo V2.20 脱壳教程

> 原文：[`www.pediy.com/kssd/pediy06/pediy6401.htm`](https://www.pediy.com/kssd/pediy06/pediy6401.htm)

使用 `TRW` 快速手脱 `Armadillo V2.20` 加壳程序——适用于 Win98 的 `Calc.exe`、`Notepad.exe` 等。

## 软件简介

记事本可能是被大家使用最多的实验品了。Win98 的计算器在 XP 下也可以运行。

## 作者声明

初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

## 调试环境

`Win98SE`、`TRW2000` 娃娃修改版、`PEiD`、`LordPE`、`ImportREC`、`W32Dasm 9.0` 白金版

---

## 脱壳过程

`Armadillo V2.20` 已是很旧的版本了，但对于我等脱壳菜鸟来说依旧是有学习价值的。我们是幸运的，有那么多的大侠把自己的心得无私的奉献出来，感谢这些心胸宽广的大侠们！这次我主要是学习了 `leo_cyl1` 大侠的《关于 Armadillo 1.8x-2.x 的 anti-debug&加壳原理初步探讨和脱壳方法》。

### Armadillo V2.20 反跟踪

1. 调用 `creatfilea` 检测 `SoftICE`。
2. 调用 `IsDebuggerPresent` 检测 windows 调试器是否存在，如 `Ollydbg`。
3. `INT3`，在程序中使用一个伪装的中断来阻止程序的运行。

所以，为了简单，我在 98 下用 `trw2000` 调试，下 `faults off` 就可以避开这些反跟踪了。

> 注：以下方法仅仅针对用 `Armadillo V2.20` 版加壳的部分程序，不适用于 `Armadillo` 主程序。

---

### 一、Win98 的 `Calc.exe`

**下断步骤：**
`BPX Sleep` 断下后，`F5` 三次程序运行，所以再次 `LOAD`，`F5` 二次，`BD` 暂停断点，`F11` 返回程序领空。

```asm
* Reference To: KERNEL32.Sleep, Ord:0296h
|
:00406F59 FF1560104100          Call dword ptr [00411060]
:00406F5F 8B15E8324100          mov edx, dword ptr [004132E8]
                                ====>F11 返回到这！
:00406F65 3315F0324100          xor edx, dword ptr [004132F0]
:00406F6B 3315EC324100          xor edx, dword ptr [004132EC]
:00406F71 3315F4324100          xor edx, dword ptr [004132F4]
:00406F77 89951CF8FFFF          mov dword ptr [ebp+FFFFF81C], edx
                                ====>EDX=010119E0            这就是 OEP 值  :-)
:00406F7D 8D8564F7FFFF          lea eax, dword ptr [ebp+FFFFF764]
:00406F83 50                    push eax
:00406F84 8B0DE4684100          mov ecx, dword ptr [004168E4]
:00406F8A 51                    push ecx
* Reference To: KERNEL32.SetThreadContext, Ord:0283h
|
:00406F8B FF1590104100          Call dword ptr [00411090]
```

**脱壳步骤：**
下断：`BPX 010119E0`，`F5` 运行，断下！脱壳：`PDEDUMP 1.EXE`，`BC` 取消断点，`F5` 运行程序。

运行 `ImportREC`，选择 `CALC.TMP0` 进程。把 `OEP` 改为 `000119E0`，点 `IT AutoSearch`，点“`Get Import`”，`FixDump`，正常运行！
结果：`217K -> 96K`

---

### 二、Win98 的记事本

**下断步骤：**
`BPX Sleep` 断下后，`F5` 二次程序运行，所以再次 `LOAD`，`F5` 一次，`F11` 返回程序领空。

```asm
* Reference To: KERNEL32.Sleep, Ord:0296h
|
:00406F59 FF1560104100          Call dword ptr [00411060]
:00406F5F 8B15E8324100          mov edx, dword ptr [004132E8]
                                ====>F11 返回到这！
:00406F65 3315F0324100          xor edx, dword ptr [004132F0]
:00406F6B 3315EC324100          xor edx, dword ptr [004132EC]
:00406F71 3315F4324100          xor edx, dword ptr [004132F4]
:00406F77 89951CF8FFFF          mov dword ptr [ebp+FFFFF81C], edx
                                ====>EDX=004010CC            这就是 OEP 值  :-)
:00406F7D 8D8564F7FFFF          lea eax, dword ptr [ebp+FFFFF764]
:00406F83 50                    push eax
:00406F84 8B0DE4684100          mov ecx, dword ptr [004168E4]
:00406F8A 51                    push ecx
* Reference To: KERNEL32.SetThreadContext, Ord:0283h
|
:00406F8B FF1590104100          Call dword ptr [00411090]
```

**脱壳步骤：**
下断：`BPX 004010CC`，`F5` 运行，断下！脱壳：`PDEDUMP 2.EXE`，`BC` 取消断点，`F5` 运行程序。

运行 `ImportREC`，选择 `NOTEPAD.TMP0` 进程。把 `OEP` 改为 `000010CC`，点 `IT AutoSearch`，点“`Get Import`”，`FixDump`，正常运行！
结果：`208K -> 56K`

---

### 三、`WDCT4.exe`

下载地址：`http://bbs.tmc2.com.cn/non-cgi/usr/14/14_157.rar`

这是前几日 `东方龙` 兄弟提到的软件。`315K`。

相关页面：`http://tongtian.net/pediybbs/viewtopic.php?p=4018#`

**下断步骤：**
`BPX Sleep` 断下后，`F5` 九次程序运行，所以再次 `LOAD`，`F5` 八次，`F11` 返回程序领空。

```asm
* Reference To: KERNEL32.Sleep, Ord:0296h
|
:00406F59 FF1560104100          Call dword ptr [00411060]
:00406F5F 8B15E8324100          mov edx, dword ptr [004132E8]
                                ====>F11 返回到这！
:00406F65 3315F0324100          xor edx, dword ptr [004132F0]
:00406F6B 3315EC324100          xor edx, dword ptr [004132EC]
:00406F71 3315F4324100          xor edx, dword ptr [004132F4]
:00406F77 89951CF8FFFF          mov dword ptr [ebp+FFFFF81C], edx
                                ====>EDX=0043E7C4            这就是 OEP 值  :-)
:00406F7D 8D8564F7FFFF          lea eax, dword ptr [ebp+FFFFF764]
:00406F83 50                    push eax
:00406F84 8B0DE4684100          mov ecx, dword ptr [004168E4]
:00406F8A 51                    push ecx
* Reference To: KERNEL32.SetThreadContext, Ord:0283h
|
:00406F8B FF1590104100          Call dword ptr [00411090]
```

**脱壳步骤：**
下断：`BPX 0043E7C4`，`F5` 运行，断下！脱壳：`PDEDUMP 3.EXE`，`BC` 取消断点，`F5` 运行程序。

运行 `ImportREC`，选择 `NOTEPAD.TMP0` 进程。把 `OEP` 改为 `0003E7C4`，点 `IT AutoSearch`，点“`Get Import`”，`FixDump`，正常运行！
结果：`376K -> 380K`

---

## 总结

用 `TRW` 载入加 `Armadillo V2.20` 壳的程序，`FULTS OFF`，下断 `BPX Sleep`，看看 `F5` 几次程序运行。重新 `LOAD`，下断 `BPX Sleep`，少按 1 次 `F5`，`F11` 返回程序领空，这时可以看见下面有几个 `xor e*x, dword ptr [004*****]`，异或的结果就是 `OEP` 值了。`BD`，暂停断点。`BPX OEP`，断下后 `PEDUMP` 脱壳，`BC` 取消断点，`F5` 运行程序。

运行 `ImportREC`，选择 `****.TMP0` 进程。修改 `OEP`，点 `IT AutoSearch`，点“`Get Import`”，`FixDump`，正常运行！

---

```text
  ,     _/
 /| .-~/
( /~   /
\`\\  /_
 -~~-.)  )__/.
/'\,   --~  ~~
\`~ _ ,_.
  /~~//'  /`
  " `~"  "
             "
             "~'  ""
Cracked By 巢水工作坊——fly [OCN][FCG]
2003-10-02  17:30
```
