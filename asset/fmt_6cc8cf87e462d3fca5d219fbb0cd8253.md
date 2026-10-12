
00 00 90 8D 50 10 89 10 89 50 04 90 8B 85 80 00 00 00 8B 30 8B 85 84 00 00 00 8B 38 90 AD 83 F8

00 74 3D FF E0 90 90 90 90 90 90 90 90 90 90 60 E8 00 00 00 00 5D 81 E5 00 00 FF FF 89 55 54

61 FF 15 18 90 2C 01 C3 90 90 90 90 90 90 90 90 8B 45 54 AB EB C7 90 90 90 90 90 90 90 90 90 90

8B 45 20 89 45 60 8B 45 24 89 45 64 90 90 90 90 8B 85 80 00 00 00 8B 30 8B 85 84 00 00 00 8B 18

90 AD 8B 13 83 F8 00 74 6C 50 52 90 90 90 90 8B 7D 60 8B 4D 64 C1 E9 02 8B C2 F2 AF 90 83 F9 00
75 3D 90 A9 00 00 00 C0 74 17 25 00 00 FF FF 05 00 00 01 00 2D 00 00 01 00 66 81 38 4D 5A 75 F4
90 83 7D 58 00 74 10 3B 45 58 74 0B 90 89 45 58 33 C0 AB 83 45 64 04 89 45 58 8B C2 AB 83 45 64
04 90 83 EF 04 5A 58 66 C7 00 FF 25 89 78 02 83 C3 04 EB 8D 90 33 C0 83 C7 04 AB 83 45 58 04 61
EB FE 90 90 90

### 3.2) 修复用的 Script

```
// filename: ospr.osc

var addr          // 定义变量

start:
  eob label_break // 中断发生时，跳到 label_break
  run             // == F9

label_break:
  cmp eip,015100A3    // 每次运行这个值都要重新修改
  jne label_exit
  mov addr,esp
  sub addr,4
  mov [addr],1540210  // 回到 CODE2 中去
  jmp start

label_exit:
  ret               // 退出 script
```

### 3.3) 例子下载

`http://www.crystaloffice.com/download.html`

## 4.0 心得与感受

### 4.1) 感觉现在用 `ASPR V2.X` 加密的软件越来越多了.

### 4.2) 有些版本的 `ASPR` 会在 `INT 3` 的 `SEH` 中清调试寄存器, 所以在 `INT 3` 之前设的硬件断点会失效.

### 4.3) 有些版本的 `ASPR` 在 `CALL 12000000` 后, 并不改写 `CALL 12000000`, 并且出现 `API Address` 的 `CODE` 也与本文中的 `Patch2` 处的 `CODE` 不一样; 但用 `Ctrl+F11` 跟到 `POPF` 处, 在 `Run Trace` 中总有个地方 `API Address` 会出现在某一寄存器中.

### 4.4) `ASPR V2.X` 真的好难脱, 即使没有 `Stolen code`

`4dd466d0e1e0cc123afbe504da2462ca`
