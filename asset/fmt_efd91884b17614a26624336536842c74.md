
# ASProtect 1.23RC4

> 原文：[`www.pediy.com/kssd/pediy06/pediy6736.htm`](https://www.pediy.com/kssd/pediy06/pediy6736.htm)

**【破解作者】** 小虾（战神\[DFCG\])
**【使用工具】** `Ollydbg`, `LordPE`, `AsprDbgr`, `RecImport`
**【官方主页】** `http://www.chinadfcg.com`
**【破解平台】** `NT/2000/XP/Win98` 下不推荐
**【软件名称】** `notepad.exe`
**【下载地址】** [本地下载](http://bbs.pediy.com/upload/files/NOTEPADy.rar_65686.rar) 或 [`bbs.pediy.com/showthread.php?s=&threadid=2744`](http://bbs.pediy.com/showthread.php?s=&threadid=2744)
**【软件简介】** `win98` 下的记事本
**【软件大小】** `52K`
**【加壳方式】** `ASProtect` `1.23` `RC4`
**【破解声明】** 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

--------------------------------------------------------------------------------

## 破解内容

以前一直用 `98` 系统，以致一直到现在也只会脱一些简单的压缩壳，这一次为了学习脱猛壳下决心安装了双系统（`win98` OR `winXP`)^_^!，这是我的第一篇脱文，`ASProtect` `1.23RC4` 脱文。关于 `ASProtect` `1.23RC4` 脱文坛子 `Fly` 前辈等大侠已经写了很多，我只是将我脱壳过程及一点心得写出来供像我一样的菜鸟学习，写得有些罗嗦，高手就不用看了。在此我要感谢 `Fly` 前辈，没有他的精彩文章和他的无私指点，就不会有我今天的脱文。`OK`，我们开始吧^O^。

首先用 `OD` 载入目标程序，用 `OD` 隐藏插件隐藏 `OD`，并在调试选项中除了内存异常不要选之外，其余全选上。

```
代码:
00401000 N>  68 01D04000         push NOTEPADy.0040D001　//载入后停在这里。
00401005     E8 01000000         call NOTEPADy.0040100B
0040100A     C3                  retn
0040100B     C3                  retn
0040100C     72 79               jb short NOTEPADy.00401087
0040100E     BD 1E8EF67C         mov ebp,7CF68E1E
00401013     871E                xchg dword ptr ds:[esi],ebx
00401015     89BD 723363B2       mov dword ptr ss:[ebp+B2633372],e>
0040101B     B1 39               mov cl,39
```

按 `Shift+F9` 来到最后一次异常（大约 27、28 次），

```
代码:
00C539EC     3100                xor dword ptr ds:[eax],eax　//最后一次异常
00C539EE     64:8F05 00000000    pop dword ptr fs:[0]
00C539F5     58                  pop eax
00C539F6     833D B07EC500 00    cmp dword ptr ds:[C57EB0],0
00C539FD     74 14               je short 00C53A13
00C539FF     6A 0C               push 0C
00C53A01     B9 B07EC500         mov ecx,0C57EB0
00C53A06     8D45 F8             lea eax,dword ptr ss:[ebp\-8]
00C53A09     BA 04000000         mov edx,4
00C53A0E     E8 2DD1FFFF         call 00C50B40
00C53A13     FF75 FC            push dword ptr ss:[ebp\-4]
00C53A16     FF75 F8            push dword ptr ss:[ebp\-8]
00C53A19     8B45 F4            mov eax,dword ptr ss:[ebp\-C]
00C53A1C     8338 00            cmp dword ptr ds:[eax],0
00C53A1F     74 02              je short 00C53A23
00C53A21     FF30               push dword ptr ds:[eax]
00C53A23     FF75 F0            push dword ptr ss:[ebp\-10]
00C53A26     FF75 EC            push dword ptr ss:[ebp\-14]
00C53A29     C3                 retn　//我们在这里 F2 设一个断点，Shift+F9 运行，中断在这里。
```

现在我们停下来看一看堆栈，

```
代码:
0012FF5C  00C6392C
0012FF60  00400000  NOTEPADy.00400000
0012FF64  F370A663
0012FF68  0012FFA4　//注意这个值。
0012FF6C  00C40000
```

我们在 `0012FF68` 处下硬件访问断点，按 `F9` 运行，中断在这里。

```
代码:
00C656D4   /EB 44                jmp short 00C6571A　//硬件中断在这里。
00C656D6   |EB 01                jmp short 00C656D9
00C656D8   |9A 51579CFC BF00     call far 00BF:FC9C5751
00C656DF   |0000                 add byte ptr ds:[eax],al
00C656E1   |00B9 00000000        add byte ptr ds:[ecx],bh
00C656E7   |F3:AA               rep stos byte ptr es:[edi]　//在这里 F2 设一个断点。
00C656E9   |9D                  popfd
00C656EA   |5F                  pop edi
00C656EB   |59                  pop ecx
00C656EC   |C3                  retn
```

`F8` 后程序将跳到这里，到了这里程序代码已经解开，可以用 `LordPE` 纠正一下文件大小后完整 `Dump` 下程序。接着我们再进行区域脱壳：`00C60000`，大小＝`00008000`，也就是脱出上面的部分壳处理代码段。至此我们已经将 `ASProtect` `1.23RC4` 壳脱出来，后面的不管他了（其实是不知接着要怎样干下去）^O^。

```
代码:
00C6571A     03C3                add eax,ebx                      ; NOTEPADy.00400000
00C6571C     BB AC000000        mov ebx,0AC　//注意这个值，等下修复程序我们用得上。
00C65721     0BDB              or ebx,ebx
00C65723     75 07             jnz short 00C6572C
00C65725     894424 1C         mov dword ptr ss:[esp+1C],eax
00C65729     61                popad
00C6572A     50                push eax
00C6572B     C3                retn
00C6572C     E8 00000000        call 00C65731　//00C6572C 动态地址必须记住，等一下用得着，这个地
址是动态生成的，以你机器的地址为准。
00C65731     5D                pop ebp
00C65732     81ED 4DE14B00      sub ebp,4BE14D
00C65738     8D85 F2E04B00      lea eax,dword ptr ss:[ebp+4BE0F2]
00C6573E     8D8D 94E14B00      lea ecx,dword ptr ss:[ebp+4BE194]
00C65744     03CB              add ecx,ebx
00C65746     8941 01           mov dword ptr ds:[ecx+1],eax
00C65749     8D85 36E14B00      lea eax,dword ptr ss:[ebp+4BE136]
00C6574F     8D8D FAE04B00      lea ecx,dword ptr ss:[ebp+4BE0FA]
00C65755     8901              mov dword ptr ds:[ecx],eax
00C65757     B8 5E140000        mov eax,145E
00C6575C     8D8D FFE04B00      lea ecx,dword ptr ss:[ebp+4BE0FF]
00C65762     8901              mov dword ptr ds:[ecx],eax
00C65764     8D8D 94E14B00      lea ecx,dword ptr ss:[ebp+4BE194]
00C6576A     8D85 94F34B00      lea eax,dword ptr ss:[ebp+4BF394]
00C65770     51                push ecx
00C65771     50                push eax
00C65772     E8 76FFFFFF        call 00C656ED
00C65777     61                popad
00C65778     EB 02             jmp short 00C6577C
```

现在我们来“组装”一下 `dumped.exe`。先用 `LordPE` 打开 `dumped.exe`，然后从磁盘载入刚才区域脱壳的 `Region00C60000-00C68000.dmp` 区段，修改其 `Voffset=00860000`（`00C60000`－`00400000`＝`00860000`），只保留 `LordPE` 的“验证 `PE`”选项，最后重建 `PE`。`Dump` 完成！^O^，不过这时程序还不能运行，因为我们还没有修复 `IAT` 和入口点及参数。现在我们接着来修复 `IAT`，修复 `IAT` 我用 `AsprDbr`，因为简单。启动 `AsprDbgr`，加载未脱壳的程序，一路按确定键，直到目标程序启动。

![](img/c36bdbc7f2ecd9eb289cb907567107fb.jpg)

```
代码:
\=================================================================
AsprDbgr v1.0beta <:P> Made by me... Manko.

 iEP=401000 <G:\\Documents and Settings\\whx\\桌面\\NOTEPADy.EXE>

   IAT Start: 4062E4　//这个是 IAT 开始位置
         End: 406524　//结束地址
      Length: 240　　//IAT 大小
     IATentry 40639C = C51C64 resolved as GetModuleHandleA
     IATentry 4063E4 = C51CD8 resoLved as GetCommandLineA
   6 invalid entries erased.
 Dip-Table at adress: C57AB4
   0 0 0 0 0 0 0 0 0 0 0 0 0 0
 Last SEH passed. <C530EE> Searching for signatures. Singlestepping to OEP!
   Call + OEP-jump-setup at: C63AD4 < Gode: E8000000 5D81ED >
   Mutated, stolen byets at: C63B20 < Code: EB02CD20 F2EB019A >
   Erase of stolen bytes at: C63A83 < Code: 9CFCBFC2 3AC600B9 >
     Repz ... found. Skipping erase of stolen bytes. ;>
  possible <temp)OEP: 4010D3 <Reached from preOEP: C63A94>
```

启动 `RecImport`，`OEP`：`00860000`，`RAV`：`000062E4`，大小：`240`，搜索 `IAT`，找到的 `IAT` 全部有效，修复 `Dump` 文件。`IAT` 修复完毕。最后我们只用 `OD` 加载脱壳修复后的程序，将入口前两句修改成以下语句，保存文件即可，最后运行脱壳程序，启动正常整个脱壳过程完成。^_^

```
代码:
00C60000 d>  BB AC000000        mov ebx,0AC　//还记得这个值吗？
00C60005     E9 CA3A0000        jmp dumped\_.00C6572C　//跳到解壳地址。
00C6000A     0000               add byte ptr ds:[eax],al
00C6000C     0000               add byte ptr ds:[eax],al
00C6000E     0000               add byte ptr ds:[eax],al
00C60010     0000               add byte ptr ds:[eax],al
```

--------------------------------------------------------------------------------

## 破解总结

懒得写了...

作者：小虾（战神\[DFCG\])

2004 年 7 月 12 日

--------------------------------------------------------------------------------

## 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! `4dd466d0e1e0cc123afbe504da2462ca`
