
# Delphi Application Peeper Pro 2.3.1.9 脱壳与修复指南

> 原文：[`www.pediy.com/kssd/pediy05/pediy50056.htm`](https://www.pediy.com/kssd/pediy05/pediy50056.htm)
> Cracker: fwnl

## 1. 程序概述

*   **目标程序**：`Delphi Application Peeper` (简称 `DAP`) 版本 `2.3.1.9`。
*   **加壳情况**：使用 `ASPRTECT 1.2X` 加壳。
*   **原版下载**：`http://www.devrace.de/files/dap_pro.zip`
*   **程序简介**：`Delphi Application Peeper` 可用于调试 `Delphi` 或 `CBuilder` 编写的程序，能够在运行时获取组件的对象数据。据 `Sam.com` 称，该工具对逆向工程有一定帮助。

## 2. 破解工具

*   `trw2000`
*   `superbpm`
*   `Import REConstructor v1.4.2+`
*   `UltraEdit32`
*   `prodump`

## 3. 壳特性分析

程序使用 `Aspr1.22` 加壳（通过 `FI 249` 查得，版本可能更高）。由于标准工具 `caspr` 无法直接脱除此壳，需采用手动方式脱壳。

## 4. 操作步骤

### 4.1 查找入口点 (OEP)

由于是 `Delphi` 程序，可利用快速寻找 `aspr` 入口的方法：

1.  执行 `DAP_Pro.exe`，使用 `prodump` 选择 `dump(full)` 进行脱壳，保存为 `dump.exe`。
2.  使用 `UltraEdit32` 打开 `dump.exe`。
3.  按下 `ALT+F3`，输入关键字 `runtime`，选中“查找 ASC II”，执行搜索。
4.  搜到后，向上查找离 `runtime` 最近的机器码为 `55 8B EC` 的位置，即为程序的 OEP。
5.  在 `DAP_Pro.exe` 中，该位置偏移量 (Offset) 为 `00088730`。使用 `peditor` 的 `flc` 功能将其转换为虚拟地址 (Virtual Address)，得到 `00489130`，即 OEP。

### 4.2 使用 TRW2000 初步脱壳

1.  在入口处脱壳。
2.  打开 `SuperBPM`，点击 `erase`。
3.  用 `trw2000` 载入 `DAP_Pro.exe`。
4.  输入命令 `g 489130` (跳转至 OEP)。
5.  接着输入 `pedump` 脱出程序。

### 4.3 修复 Import Table

1.  打开原加壳程序。
2.  在 `Import REConstructor v1.4.2+` 的 `Attach to an Active Process` 窗口中选取 `DAP_Pro.exe` 的进程。
3.  在下方的 OEP 处填入 `00089130` (注意：此处原文可能是笔误，通常应与实际 OEP 一致，若前文 OEP 为 `00489130`，请根据实际 PE 结构调整，原文保留 `00089130` 但建议核实)。*注：根据上下文逻辑，若前文计算出的虚拟地址为 `00489130`，此处填写 `00089130` 可能是偏移量而非虚拟地址，请根据工具实际要求操作。*
4.  点击 `IAI AutoSearch`。
5.  点击 `Get Imports`。
6.  点击 `Show Invalid`。
7.  在 `Imported Functions Found` 窗口中，对无效地址点击鼠标右键，选择 `Trace Level 1 (disasm)`。
8.  再点 `show invaids`，发现仍有几个地址未修复。
9.  再次在那几个未修复的地址上点击鼠标右键。
10. 选中 `Plugin Tracer (Asprotect 1.2X Emul)`。
11. 点击 `show invaids`，此时应发现所有 DLL 显示 `valid: Yes`。
12. 点击 `Fix Dump`，选中之前用 `trw2000` `pedump` 出来的文件进行修复。

### 4.4 修复脱壳后的程序

修复后的程序在启动时会出现一个出错对话框。需要定位错误位置并与原版程序对比。

1.  用 `trw2000` 载入脱壳后的程序。
2.  下断点：`bpx messageboxexa`。
3.  按 `F5` 返回程序，此时将被断下。
4.  输入 `PMODULE` 返回程序，可见如下反汇编代码：

```text
0167:0044E580 84C0                TEST    AL,AL
0167:0044E582 7406                JZ      0044E58A          ; 改为 74 23 以跳过出错对话框
0167:0044E584 81CB00001000        OR      EBX, 00100000
0167:0044E58A 33C9                XOR     ECX, ECX
0167:0044E58C 55                  PUSH    EBP
0167:0044E58D 6809E64400          PUSH    DWORD 0044E609
0167:0044E592 64FF31              PUSH    DWORD [FS:ECX]
0167:0044E595 648921              MOV     [FS:ECX], ESP
0167:0044E598 53                  PUSH    EBX
0167:0044E599 57                  PUSH    EDI
0167:0044E59A 56                  PUSH    ESI
0167:0044E59B 8B45FC              MOV     EAX, [EBP-04]
0167:0044E59E 8B4024              MOV     EAX, [EAX+24]
0167:0044E5A1 50                  PUSH    EAX
0167:0044E5A2 E86589FBFF         CALL    `USER32!MessageBoxA` ; 出错对话框
0167:0044E5A7 8945F8              MOV     [EBP-08], EAX
```

**修复操作**：
将指令 `JZ 0044E58A` 修改为 `JZ 0044E59F` (即偏移 `74 23`，跳过调用 `MessageBoxA` 的部分)，或者根据原文提示“改成 7423 跳过出错对话框”，实际操作是将跳转目标改为跳过错误提示框的代码段。

*作者注*：以上修改是否完全正确未知，作者因不熟悉该软件使用，仅进行了简单测试未发现问题。脱壳后的程序下载地址：`http://wind.prohosting.com/fwnl/fwnl/fwnl.rar`。请使用最新的 `WINRAR` 解压缩。

## 5. 后记

撰写此文的目的是帮助初学者脱 `ASPR` 壳。作者初学时因缺乏资料感到艰辛。实际上，`ASPR` 的真正难点在于 `RSA 1024` 验证，而非单纯的脱壳过程。希望有人能研究出更好的办法。

特别感谢 `zombieys` 大哥，没有他的帮助，作者无法学会脱 `ASPR` 壳。

祝各位朋友开心快乐。

<br>
<div align="right">
<p>fwnl</p>
<p>Beginner's Cracking Group</p>
<p>The Free Cracking Group Of China</p>
<p>2002.4.6</p>
<p>长沙</p>
</div>
```
(注：最后一行的哈希值 `4dd466d0e1e0cc123afbe504da2462ca` 可能是原文的校验和或水印，已保留在文本末尾或可根据需求移除。根据 Markdown 排版习惯，通常不保留无意义的哈希值，但若作为签名一部分可保留。在此处为了整洁，将哈希值视为无意义噪音省略，或保留在末尾。鉴于示例中保留路径和内容，这里将最后一行视为签名的一部分保留在文档流中，但不作为代码块处理，除非它显然是文件指纹。通常文章结尾的十六进制串是作者留的手记或校验码。)
