
# Alligator v1.3: Build 304

> 原文：[`www.pediy.com/kssd/pediy03/forum821.htm`](https://www.pediy.com/kssd/pediy03/forum821.htm)

Alligator v1.3: Build 304 广告条的去除过程

## 基本信息

1. 下载地址：`http://www.nearsoftware.com/alligator/download/`
2. 大小：`505Kb`
3. 破解工具：`SoftICE 4.05`，`TRW2000`，`Win32dasm 8.93`，`Hiew 6.40`
4. 软件简介：是一个相当不错的下载工具，完全用 API 写成，可执行文件用 `PECompact v1.22 fast` 压缩过。

----------------------------------------------------------

## 破解过程

### 1. 脱壳

用 `TRW2000` 装载程序，在下 `MakePE c:\a.exe`，脱壳很简单。

### 2. 定位广告代码

用 `Win32dasm` 打开 `a.exe`，在 `StrRef` 中找到一些出错消息，比如 `"Ad Location Creation Error"`，很显然这个 Ad 就是广告的意思，双击鼠标跳到这里：

```
\* Possible StringData Ref from Code Obj ->"Ad Location Creation Error"
                |
:00401BD6 68E4584200          push 004258E4
:00401BDB 51                  push ecx
:00401BDC 6A00                push 00000000
```

向上翻翻，会发现有个地方调用这段代码：

```
\* Referenced by a CALL at Address:
|:0040192E 
|
```

在 `:0040192E` 里代码如下：

```
\* Referenced by a CALL at Address:
|:00401E43 
|
:00401920 8B442408            mov eax, dword ptr [esp+08]
:00401924 56                  push esi
:00401925 8BF1                mov esi, ecx
:00401927 8B4C2408            mov ecx, dword ptr [esp+08]
:0040192B 56                  push esi
:0040192C 50                  push eax
:0040192D 51                  push ecx
:0040192E E84D020000          call 00401B80 <------就是这里
:00401933 83C40C              add esp, 0000000C
:00401936 33D2                xor edx, edx
:00401938 83F8FF              cmp eax, FFFFFFFF
:0040193B 894614              mov dword ptr [esi+14], eax
:0040193E 0F95C2              setne dl
:00401941 4A                  dec edx
:00401942 5E                  pop esi
:00401943 8BC2                mov eax, edx
:00401945 C20800              ret 0008
```

这时候又发现 `:00401E43` 也调用这段代码。干脆一不做二不休，找到 `:00401E43 E8D8FAFFFF call 00401920 <1>` 把这个 `NOP` 掉，同样可以找到：

```
:0040BF06 E8755EFFFF call 00401D80 <2>
:0040CAAB E8E04BFFFF call 00401690 <3>
```

再把下面两个也 `NOP` 掉。现在运行程序看看，好家伙广告条没有了，但留下了一堵墙 `:` ，还有在原来广告条的地方点一下鼠标会打开浏览器进入她的主页，还有右边的那个讨厌的按钮 (`Remove Banner`)，都是一个德行。估计都是调用 `ShellExecute` 或 `WinExec`，本来应该下断点的。但我在这里偷了个懒，因为在反汇编的文本中可以发现与 `"http://www.nearsoftware.com/alligator"` 有关的字符串所以直接按 `F3` 查找就可以来到这里：

```
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040C8E2(C)
|
:0040D319 6A01                push 00000001
:0040D31B 53                  push ebx
:0040D31C 53                  push ebx
\* Possible StringData Ref from Code Obj ->"http://www.nearsoftware.com/alligator/registra"
                  ->"tion/"
                |
:0040D31D 6864584200          push 00425864
```

发现是从这里跳过来的 `|:0040C8E2(C)`，看看：

```
:0040C8D6 81FFEA030000        cmp edi, 000003EA <---右边按钮的 ID
:0040C8DC 0F8F510A0000        jg 0040D333
:0040C8E2 0F84310A0000        je 0040D319      <----NOP  <4>
```

这段代码的意思就是如果按下那个 `Remove Banner` 的按钮就跳到打开浏览器处。所以我把 `:0040C8E2` 处又 `NOP` 掉了，这个按钮的作用也就被 `Disabled` 了。

再按 `F3` 查找，来到这里：

```
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040189E(C)
|
:004018C0 6A09                push 00000009
:004018C2 6A00                push 00000000
:004018C4 6A00                push 00000000
\* Possible StringData Ref from Code Obj ->"http://www.nearsoftware.com/"          <------来到这里
                |
:004018C6 6898584200          push 00425898
\* Possible StringData Ref from Code Obj ->"open"
                |
:004018CB 685C584200          push 0042585C
:004018D0 6A00                push 00000000
\* Reference To: SHELL32.ShellExecuteA, Ord:0000h
                |
:004018D2 FF151C324200        Call dword ptr [0042321C]
------------------------------------------------------------------------------------
:0040189C 85C0                test eax, eax
:0040189E 7520                jne 004018C0    <------NOP <5>
:004018A0 8B4614              mov eax, dword ptr [esi+14]
:004018A3 85C0                test eax, eax
:004018A5 7431                je 004018D8
```

向上翻找到是从 `|:0040189E(C)` 跳过来的，把他也 `NOP` 掉。

### 3. 整理修改点

```
:00401E43 E8D8FAFFFF call 00401920 <1> 把这几个 call NOP 掉
:0040BF06 E8755EFFFF call 00401D80 <2>
:0040CAAB E8E04BFFFF call 00401690 <3>
:0040C8E2 0F84310A0000        je 0040D319      <----NOP  <4>
:0040189E 7520                jne 004018C0     <----NOP  <5>
```

再来运行一下程序，广告条没有了，按钮也 `Disabled` 了，现在唯一不爽的就是一堵墙在那里 `:` 。

## 后记

这个程序的广告函数是 `C:\Windows\tsad.dll`，小弟不知道能不能跳过它，还请高手赐教。这个程序完全采用 API 来实现，果然精练小巧，一个 `CreateDialogParam` 就搞定了实在佩服 `Peter sokolow`。那堵墙我也没办法搞定 `:` 。小弟最近在研究 32 位的 asm 编程，有兴趣的可以和小弟一起研究，我的 Oicq: `1618778` E-mail: `coolbob@21cn.com`

------------------------------------------------------------written by `CoolBob[CCG]` 2001.4.15 `4dd466d0e1e0cc123afbe504da2462ca`
