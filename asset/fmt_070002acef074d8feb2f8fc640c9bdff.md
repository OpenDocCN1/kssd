
```
      jmp                 loc_4                       ; 若16页均可读,则跳过下面的anti;-)

loc_7:
      sub                 eax,00000010                 ; 退到最后1个可读页(的最后16 bytes)
      push               eax
      push               00000010
      push               eax
      call                [ebp+07981030]               ; 这16 bytes是否可读?
      ; 第6段
      mov                 ebx,eax
      pop                eax
      or                   ebx,ebx
      jnz                 loc_4                          ; 不能读则跳过
      ; 4个DWORD全为FEEEFEEE则设置标记
      cmp                 dword ptr [eax],FEEEFEEE
      jnz                 loc_4
      cmp                 dword ptr [eax+00000004],FEEEFEEE
      jnz                 loc_4
      cmp                 dword ptr [eax+00000008],FEEEFEEE
      jnz                 loc_4
      cmp                 dword ptr [eax+0000000C],FEEEFEEE
      jnz                 loc_4
      mov                 eax,eax
      cmp                 dword ptr [ebp+0792299D],00000000
      jnz                 loc_8
      cmp                 dword ptr [ebp+07920499],00000000
      jz                  loc_9
loc_8:
      push               eax
      push               ebx
      mov                 eax,eax
      mov                 eax,000004D1
      lea                 ebx,[ebp+0794F939]
      call                ebx
      ; 第7段
      pop                ebx
      pop                eax
      mov                 eax,eax
loc_9:
      mov                 dword ptr [ebp+07922CB1],00000001   ; debugger detected
loc_4:
      mov                 eax,eax
      cmp                 dword ptr [ebp+0792299D],00000000
      jnz                 loc_10
      cmp                 dword ptr [ebp+07920499],00000000
      jz                  loc_11
loc_10:
      push               eax
      push               ebx
      mov                 eax,eax
      mov                 eax,000004D1
      mov                 [ebp+07920651],eax
      lea                 ebx,[ebp+0794FC33]
      call                ebx
      ; 第8段
      pop                ebx
      pop                eax
loc_11:
      cmp                 dword ptr [ebp+07920499],00000000
      jz                  loc_12
      push               eax
      push               ebx
      mov                 eax,000004D1
      mov                 [ebp+07920651],eax
      lea                 eax,[ebp+0794F8D4]
      call                ebx
      ; 第9段
      pop                ebx
      pop                eax
loc_12:
      mov                 eax,eax
      mov                 eax,[ebp+0798102C]
      xor                 eax,35F9E374
      add                 eax,ebp
      jmp                 eax
```

这样的检测，我的`OllyDbg`本来应该能通过的，问题在`WinXP SP`自己。`SP2`为防止溢出攻击，对`PEB`的地址作了随机化处理，如果用来隐藏`OllyDbg`的代码使用了`7FFDF000`的硬编码地址，就会对错误的地址进行数据清理。

如果清理代码是注入的，直接从`fs:[30h]`取，不会有问题。如果是在被调试进程之外用`WriteProcessMemory`实现的，需要用`GetThreadSelectorEntry`获取`fs`段的地址。

## Virtual Machine Anti-Dump

这里的讨论不适用于`1.8.x.x`。在写这篇文章的时候，`Themida`升级到`1.9.4.0`，专门提到增强了`Anti-Dump`，所以估计也对付不了`1940`了。

原程序中一段被`VM`保护的代码，开始执行时首先就是`AntiDump`。我看的这个程序，用于`AntiDump`的`PCODE`约`9500`行。`AntiDump`原始代码就是变形代码。

脱壳后运行，出现内存访问异常，异常出现在`VM`内，试图访问`8D8C0`的数据。下面是部分还原代码:

```
……
      push     dword ptr [ebp+0792090D]  ; [11372F38]= 8D8C0h
      push     ebp
      push     ebx
      mov      ebx,501B7E96
      mov      ebp,C605CE17
      add      ebp,ebx
      pop      ebx
      add      [esp+4],ebp
      pop      ebp
      mov      eax,[esp]
      add      esp,00000004
      sub      eax,16214CAD
      push     000001901
      mov      [esp],ecx
      push     esi
      mov      esi,0A09435A             ;  Magic Number为硬编码值
      sub      esi,C18ED54D             ;  487a6e0d
      mov      ecx,esi
      pop      esi
      push     esi
      mov      esi,ecx
      mov      edx,esi
      pop      esi
      pop      ecx
      cmp      [eax],edx         ; Anti-Dump
      jz        l_1142D17E
```

这里`eax`为`8D8C0`，这里的`dword`必须等于硬编码值`487A6E0D`。这是壳代码分配的内存，`dump`出来后自然没有。这个地址保存在壳代码区段内:

![](img/ad432ee136c2ea2a1b49da7e75128dd9.jpg)

知道为什么就简单了 J。

还出现了另外一种。我看的是个`DLL`，却检测了宿主`EXE`的`MZ`头。至于`EXE`文件是否存在这种`AntiDump`，我没有验证。

这段代码直接看了一下，没有还原代码了。从`11374BA8`获取`EXE`的加载地址。

![](img/df8cdfa311a49e5c5b165e592c4c405a.jpg)

检测`MZ+0Ch`的`word`。

```
07311   1144CAA8  00000000  15  PUSH16      [R2]
07328   1144CABF  000000DC  A2  PUSH16      0600
07339   1144CACF  0000008F  4F  CMP16       [esp+2],[esp]
07358   1144CAE9  000034E8  79  POP32       [addr_ctx.eflag]
07360   1144CAEC  00000000  3E  JZ
07361   1144CAF1  00000C3F  87  SetDst      1144CE3B  ; Anti-Dump
```

即`MZ+0Ch`应为`600h`。正常的文件这里为`FFFF`。

![](img/62e3a572b5f0dff26c4b99c141deb47a.jpg)

`AntiDump`我就碰到这 2 种，不清楚还有没有别的。

`Themida`的`CISC`指令集`VM`相比之下比较简单，实战中也出现较少，倒是可以用来学学`VMJ`。

## 致谢

`www.pediy.com`

`www.unpack.cn`

`4dd466d0e1e0cc123afbe504da2462ca`
