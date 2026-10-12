
# EditPlus 2.10a

> 原文：[`www.pediy.com/kssd/pediy03/forum634.htm`](https://www.pediy.com/kssd/pediy03/forum634.htm)

## 如何去除 EditPlus 2.10a 的 NAG

本文仅供初学者参考，因为本人也是初学者。

用 `ExeScope` 可以发现，NAG 不是以窗口资源的形式存在的，因此用 `DASM` 反汇编查找窗口 ID 来找出产生 NAG 的地方的方法是无用的，以前 2.00 版倒是可以的，只是先去壳，好像是 `petite` 加的壳吧。

既然 `DASM` 派不上用场，那只好用调试工具硬追，关键是找到开窗的 API 函数。

可以注意到 `editplus` 的 NAG 是一个非模式窗口（非模式窗口这个概念大家应该明白吧，有些 `windows` 下编程能力的人都应该知道，它是相对于模式窗口而言的，不必关闭此窗口就可以继续动用程序的其它功能，如文本编辑器中的“查找字符对话框”等等），经过几次试探，找到一个可用的断点：

`bpx createdialogindirectparama`

所需工具：`trw200` 或 `soft-ice`，一个十六进制编辑器（我用的是 `Ultra Edit 8.0`）。

### 操作步骤

1. 用 `trw2000` 载入 `editplus.exe`，下断点，`bpx createdialogindirectparama`。
2. 将被拦下四次，第四次拦下时，下命令，`pmodule` 返回，看到如下代码：

```
015F:004A9653  MOV       EAX,[EAX+1C]
015F:004A9656  PUSH      EBX
015F:004A9657  PUSH      DWORD 004A9224
015F:004A965C  PUSH      EAX
015F:004A965D  PUSH      DWORD [EBP+08]
015F:004A9660  PUSH      DWORD [EBP+10]
015F:004A9663  CALL      `USER32!CreateDialogIndirectParamA`     ;跳出 NAG
015F:004A9669  LEA       ECX,[EBP-1C]     ;我们停在这里
015F:004A966C  MOV       [EBP-18],EAX
015F:004A966F  MOV       [EBP-04],BL
015F:004A9672  CALL      004AA836
015F:004A9677  JMP       SHORT 004A9696
015F:004A9679  MOV       ECX,[EBP-2C]
015F:004A967C  CALL      004AC642
015F:004A9681  MOV       EAX,[EBP-24]
015F:004A9684  OR        DWORD [EAX+2C],BYTE -01
```

3. 返回到调用此 NAG 的地方，返回好几个 `ret`，还要被它吓住哦!

来到这儿：

```
015F:0042D7ED  CALL      0042F570
015F:0042D7F2  JMP       0042D9DD
015F:0042D7F7  CMP       EAX,BYTE +6B
015F:0042D7FA  JNZ       0042D813        ;此处一定要跳，改成 JMP 就可以了
015F:0042D7FC  PUSH      EAX
015F:0042D7FD  MOV       EAX,[ESI+1C]
015F:0042D800  PUSH      EAX
015F:0042D801  CALL      `USER32!KillTimer`
015F:0042D807  MOV       ECX,ESI
015F:0042D809  CALL      0042D210        ;这个 CALL 弹出 NAG
015F:0042D80E  JMP       0042D9DD
015F:0042D813  CMP       EAX,BYTE +6D
015F:0042D816  JNZ       0042D88F
015F:0042D818  MOV       ECX,[ESI+1C]
015F:0042D81B  PUSH      EAX
015F:0042D81C  PUSH      ECX
```

4. 总结：在 `editplus.exe` 中，用十六进制编辑器进行修改：

   Find: `83 F8 6B 75 17 50 8B 46 1C`

   ```
           ^^
       EB
   ```

   是不是很容易:)

by `mjing`, 2000/2/16,

E-mail: `mjing@wx88.net`
