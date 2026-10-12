
# 乾坤大挪移 `VprotectDemo.2.0.9` 脱壳之处理 `antidump`

> 原文：[`www.pediy.com/kssd/pediy12/141405.html`](https://www.pediy.com/kssd/pediy12/141405.html)

最近比较忙不会像上次放国庆那样有时间弄了，`VprotectDemo.2.0.9` 的内存随机校检变得非常非常的强悍，不能像上次 `Vprotect Demo 2.0.4 patch` 虚拟机那样简单了。

关于脱壳请参考:`http://www.unpack.cn/thread-70263-1-1.html` 的脚本。

如果程序 SDK 了代码并加了壳，脚本脱完壳修复好 `IAT` 之后，程序在本机和本机的虚拟机跑是没有任何问题的，一但换了机器程序就跑不起来了这可能就是 `VprotectDemo.2.0.9 anti dump`，下面主要介绍如何绕过 `crc32` 内存校验并 patch `CPUID` 来处理 `anti dump` 的。

## 一.乾坤大挪移之 patch `CRCHandler` 绕过 `crc32` 的内存校验

1. `LordPE` 载入修复好 `IAT` 后的程序,依次选择区段 -> 一般选择 `RawSize` 最大的区段可能就是虚拟机区段了,如果不确认区段请在程序中找,保存区段到磁盘。
   然后选择从磁盘中载入我们刚才保存的区段,这样子就在程序的区段后面添加了一个镜像区段。计算好一个偏移地址：`AD5000 = 010C8000(新添加的镜像区段虚拟地址) -005F3000(虚拟机起始虚拟地址)`，注意是 `VA` 不是 `RAV`。
2. `OD` 载入修复好 `IAT` 后的程序,在反汇编窗口中打开有虚拟机代码的区段,查找所有命令:`MOV ESI,DWORD PTR DS:[EDI+30]` ,确定之后弹出 8 个 `crc32 handler`,这 8 个 handler 代码的功能一模一样，再往上拉不要超过 `retn` 指令找到 `MOV EAX,DWORD PTR SS:[EBP]` 就是 `CRC32Handler` 开始的地方了，分别找到所有的 `crc32 handler` 起始的位置。然后将剩下的 7 个起始位置修改为 `JMP` 到其中一个 `crc32 handler` 起始位置，这样做是好集中 patch。
3. 在其中的 `crc32 handler` 开始位置修改代码如下:

   二进制代码:

   ```
   8B 45 00 05 DE C0 00 00 C1 C0 0E F7 D0 8B C8 8B 45 04 05 DE C0 00 00 C1 C0 0E F7 D0 05 00 70 CB
   00 8B D0 8D 6D 08 B8 FF FF FF FF 53 56 8B 77 30 8A 1A 32 D8 81 E3 FF 00 00 00 8B 1C 9E C1 E8 08
   33 C3 42 49 85 C9 75 E8 5E 5B F7 D0 F7 D5 8D 6D 04 F7 D5 F7 D0 C1 C8 0E 2D DE C0 00 00 89 45 00
   C3
   ```

   二进制粘贴后代码:

   修改代码为 `AD5000` 上面计算的偏移，让虚拟机的 `CRC32Handler` 代码去校验镜像区段去吧(镜像区段的代码和数据千万改不得,你懂的)，这样就可以过 `CRC` 内存校验了，此时虚拟机就像提线的木偶任你 patch 了。^\_^代码:

   ```
   00931C87    8B45 00         MOV     EAX,DWORD PTR SS:[EBP]
   00931C8A    05 DEC00000     ADD     EAX,0C0DE
   00931C8F    C1C0 0E         ROL     EAX,0E
   00931C92    F7D0            NOT     EAX
   00931C94    8BC8            MOV     ECX,EAX
   00931C96    8B45 04         MOV     EAX,DWORD PTR SS:[EBP+4]
   00931C99    05 DEC00000     ADD     EAX,0C0DE
   00931C9E    C1C0 0E         ROL     EAX,0E
   00931CA1    F7D0            NOT     EAX
   00931CA3    05 0070CB00     ADD     EAX,00CB7000     //此处修改为上面计算的偏移 AD5000。
   00931CA8    8BD0            MOV     EDX,EAX
   00931CAA    8D6D 08         LEA     EBP,DWORD PTR SS:[EBP+8]
   00931CAD    B8 FFFFFFFF     MOV     EAX,-1
   00931CB2    53              PUSH    EBX
   00931CB3    56              PUSH    ESI
   00931CB4    8B77 30         MOV     ESI,DWORD PTR DS:[EDI+30]
   00931CB7    8A1A            MOV     BL,BYTE PTR DS:[EDX]
   00931CB9    32D8            XOR     BL,AL
   00931CBB    81E3 FF000000   AND     EBX,0FF
   00931CC1    8B1C9E          MOV     EBX,DWORD PTR DS:[ESI+EBX*4]
   00931CC4    C1E8 08         SHR     EAX,8
   00931CC7    33C3            XOR     EAX,EBX
   00931CC9    42              INC     EDX
   00931CCA    49              DEC     ECX
   00931CCB    85C9            TEST    ECX,ECX
   00931CCD   ^ 75 E8           JNZ     SHORT 00931CB7
   00931CCF    5E              POP     ESI
   00931CD0    5B              POP     EBX
   00931CD1    F7D0            NOT     EAX
   00931CD3    F7D5            NOT     EBP
   00931CD5    8D6D 04         LEA     EBP,DWORD PTR SS:[EBP+4]
   00931CD8    F7D5            NOT     EBP
   00931CDA    F7D0            NOT     EAX
   00931CDC    C1C8 0E         ROR     EAX,0E
   00931CDF    2D DEC00000     SUB     EAX,0C0DE
   00931CE4    8945 00         MOV     DWORD PTR SS:[EBP],EAX
   00931CE7    C3              RETN
   ```

## 二.Patch `CPUID`

1. 接着上面在虚拟机区段的反汇编窗口中搜索 -> 查找所有命令:`cpuid` ，确定之后会弹出很多的 `CPUID`，一个一个的点击查看吧，带花的肯定不是。一般是 4 个相邻的 `CPUID` 指令，找到之后抓取脱壳机器的 `CPUID 0` 和 `CPUID 1` 我的是:代码:

   ```
   CPUID 0
   EAX 0000000B
   ECX 6C65746E
   EDX 49656E69
   EBX 756E6547

   CPUID 1
   EAX 00020655
   ECX 029AE3FF
   EDX BFEBFBFF
   EBX 04100800
   ```

2. 找到一个空的代码区段，补 `IAT` 的区段时留下了有大把的空闲的地址哦。

   二进制代码:代码:

   ```
   83 F8 00 74 15 B8 55 06 02 00 B9 FF E3 9A 02 BA FF FB EB BF BB 00 08 10 04 C3 B8 0B 00 00 00 B9
   6E 74 65 6C BA 69 6E 65 49 BB 47 65 6E 75 C3 90
   ```

   粘贴后的代码如下:

   将下面的 `EAX`,`ECX`,`EDX`,`EBX` 代码修改成你脱壳机器的 `CPUID`。代码:

   ```
   00931E0D    83F8 00         CMP     EAX,0
   00931E10    74 15           JE      SHORT 00931E27
   00931E12    B8 55060200     MOV     EAX,20655
   00931E17    B9 FFE39A02     MOV     ECX,29AE3FF
   00931E1C    BA FFFBEBBF     MOV     EDX,BFEBFBFF
   00931E21    BB 00081004     MOV     EBX,4100800
   00931E26    C3              RETN
   00931E27    B8 0B000000     MOV     EAX,0B
   00931E2C    B9 6E74656C     MOV     ECX,6C65746E
   00931E31    BA 696E6549     MOV     EDX,49656E69
   00931E36    BB 47656E75     MOV     EBX,756E6547
   00931E3B    C3              RETN
   00931E3C    90              NOP
   ```

3. 将四个 `CPUID` 的代码 call 到上面代码的地址`00931E0D`即可。

最后保存 Patch `CRC32Handler` 和 Patch `CPUID` 的代码，怎么保存你懂的。

目前跨平台测试机器是两台笔记本:`winxp 32bit sp3` 和 `win7 64bit home base` 都能正常运行，当然没有处理 `anti dump` 是不能跨机器运行的。

处理 `anti dump` 录像 115 网盘下载：

`http://115.com/file/e6r0subs#`
