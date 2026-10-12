
# The Bat! 1.39 by:冰毒

> 原文：[`www.pediy.com/kssd/pediy01/forum022.htm`](https://www.pediy.com/kssd/pediy01/forum022.htm)

## 1. 主程序的脱壳

```
014F:00990B35  8B10                 MOV     EDX,[EAX]
014F:00990B37  8B4508               MOV     EAX,[EBP+08]
014F:00990B3A  035018               ADD     EDX,[EAX+18]
014F:00990B3D  8B4508               MOV     EAX,[EBP+08]
014F:00990B40  8B401C               MOV     EAX,[EAX+1C]
014F:00990B43  E880F9FFFF           CALL    009904C8 <-这里! F8 进入
014F:009904C8  89C4                 MOV     ESP,EAX
014F:009904CA  89D0                 MOV     EAX,EDX
014F:009904CC  8B1D34569900         MOV     EBX,[00995634]
014F:009904D2  89041C               MOV     [EBX+ESP],EAX
014F:009904D5  61                     POPAD
014F:009904D6  50                     PUSH    EAX <-记下 EAX 的值(61C528)
014F:009904D7  C3                     RET     <-这里用 Procdump 脱壳
```

## 2. 获得完整的.idata section

如上得到的脱壳后的程序，在你改过 `EIP` 后仍无法运行。还有工作要做，你得用 `Icedump`。

```
014F:009909FF  8B4508               MOV     EAX,[EBP+08]
014F:00990A02  8D4824               LEA     ECX,[EAX+24]
014F:00990A05  8B4508               MOV     EAX,[EBP+08]
014F:00990A08  8B500C               MOV     EDX,[EAX+0C]
014F:00990A0B  8B4508               MOV     EAX,[EBP+08]
014F:00990A0E  8B4008               MOV     EAX,[EAX+08]
014F:00990A11  E8FAF6FFFF           CALL    00990110
014F:00990A16  33C0                 XOR     EAX,EAX <-这里!
014F:00990A18  5A                     POP     EDX
014F:00990A19  59                     POP     ECX
014F:00990A1A  59                     POP     ECX
014F:00990A1B  648910               MOV     FS:[EAX],EDX
014F:00990A1E  EB13                 JMP     00990A33
```

在 `00990A16` 行，`Pagein D 62e000 3000 c:\thebat.bin`

## 3. 运行 Hex 编辑程序

用完整的`.idata`替换掉脱壳后主程序中`.idata`部分。Job done.
