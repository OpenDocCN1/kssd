
# 加壳时处理 IAT

> 原文：[`www.pediy.com/kssd/pediy06/pediy6480.htm`](https://www.pediy.com/kssd/pediy06/pediy6480.htm)

加壳时处理 IAT

`kongfoo/2004.4.3`

避免壳运行时处理 IAT 能被跳过的弊端。达到原程序没有一刻是完好的的目的。

加壳时处理 IAT 要的是找到文件中 IAT 所在位置和大小。

VC 和 DELPHI 程序不同。

## 一、VC notepad.exe 分析

- `PE header(eifanew)`在 `e8` 处。
- `IAT RVA` 在 `eifanew+d8` 处，`1000`
- `IAT Size` 在 `eifanew+dc` 处，`324`，这 2 个值是系统参考值，系统在装入文件时就会参考这 2 个值。
- 物理位置上的 IAT 放在第一个节。即 `400` 处。
- PE 头接着就是第一个节，所以取 `SizeOfHeader` 值就可以定位到第一个节。
- `SizeOfHeader` 在 `eifanew+54` 处，`400`。

现在有了 IAT 的物理位置和大小。

## 二、DELPHI project1.exe 分析

- `eifanew+d8` 和 `eifanew+dc` 处都为 `0`。
- `IAT RVA` 在 `eifanew+80` 处，`54000`，即目录表的 `IMPORT_DIR.VirtualAddress`
- `IAT Size` 在 `eifanew+84` 处，`1f4`，即目录表的 `IMPORT_DIR.Size`
- 这 2 个值也是系统参考值，系统在装入文件时参考这 2 个值，把 IAT 放进去。
- 跳转表在第一个节开始。

## 三、系统装入 IAT 工作流程猜测

1. 先在 `eifanew+d8` 和 `eifanew+dc` 取 `IAT RVA` 和 `IAT Size`，值非空即将第一个节 `IAT Size` 个字节装入 `IAT RVA` 指向的地址。
2. 若取得的 `IAT RVA` 和 `IAT Size` 为 `0` 即取 `eifanew+80` 和 `eifanew+84` 的 `IMPORT_DIR.VirtualAddress` 和 `IMPORT_DIR.Size` 值，将这 2 个值作为 `IAT RVA` 和 `IAT Size` 参考值。
3. 再去 `BaseOfData` 取各 API 的字串和 DLL 字串，根据字串取 API 地址，再把地址放到 `IAT RVA` 中。

## 四、加壳时加密 IAT

要在加壳时加密 IAT，对 VC 的程序很简单。对 DELPHI 复杂很多。

1. 取 `IMPORT_DIR` 值，
2. 再用取各 API 地址，把地址写回第一个节，
3. 再把 `IAT RVA` 和 `IAT Size` 的值改成 `IMPORT_DIR` 的值。

这样就完成了 DELPHI 程序 IAT 向 VC 程序的转变。再用对 VC 程序 IAT 加密的方法对其进行加密。

但有一难点，原来的第一个节没办法处理。所以加壳时加密 IAT 只能针对个别程序。

## 五、其它

DELPHI 这种对 API 地址处理的方式也有其好处，跨平台。

`4dd466d0e1e0cc123afbe504da2462ca`
