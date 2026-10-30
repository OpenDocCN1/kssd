
```
Breakpoint 0 hit

eax=00000001 ebx=40000358 ecx=76fc316f edx=096f2d34 esi=09759d70 [COLOR="red"]edi=40000358[/COLOR]

eip=69da06be esp=0446eeec ebp=0446eef8 iopl=0         nv up ei pl zr na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246

msado15!CRecordGroup::AllocateHRowRange+0x5e:

69da06be 85ff            [COLOR="red"]test    edi,edi[/COLOR]

0:005> p

eax=00000001 ebx=40000358 ecx=76fc316f edx=096f2d34 esi=09759d70 edi=40000358

eip=69da06c0 esp=0446eeec ebp=0446eef8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

msado15!CRecordGroup::AllocateHRowRange+0x60:

69da06c0 0f8e34380200    jle     msado15!CRecordGroup::AllocateHRowRange+0x62 (69dc3efa) [br=0]

0:005> p

eax=00000001 ebx=40000358 ecx=76fc316f edx=096f2d34 esi=09759d70 edi=40000358

eip=69da06c6 esp=0446eeec ebp=0446eef8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

msado15!CRecordGroup::AllocateHRowRange+0x64:

69da06c6 8bc7           [COLOR="red"] mov     eax,edi[/COLOR]	/[B]/ eax = edi = 0x40000358，即CacheSize[/B]

0:005> p

eax=40000358 ebx=40000358 ecx=76fc316f edx=096f2d34 esi=09759d70 edi=40000358

eip=69da06c8 esp=0446eeec ebp=0446eef8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

msado15!CRecordGroup::AllocateHRowRange+0x66:

69da06c8 8b3dfc10d969    mov     edi,dword ptr [msado15!_imp__MpHeapAlloc 

(69d910fc)] ds:0023:69d910fc={MSDART!MpHeapAlloc (72de9730)}

0:005> p

eax=40000358 ebx=40000358 ecx=76fc316f edx=096f2d34 esi=09759d70 edi=72de9730

eip=69da06ce esp=0446eeec ebp=0446eef8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

msado15!CRecordGroup::AllocateHRowRange+0x6c:

69da06ce 89460c          mov     dword ptr [esi+0Ch],eax ds:0023:09759d7c=00000000

0:005> p

eax=40000358 ebx=40000358 ecx=76fc316f edx=096f2d34 esi=09759d70 edi=72de9730

eip=69da06d1 esp=0446eeec ebp=0446eef8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

msado15!CRecordGroup::AllocateHRowRange+0x6f:

69da06d1 8b0d10f0e569    mov     ecx,dword ptr [msado15!g_hHeapHandle (69e5f010)] ds:0023:69e5f010=096f0000

0:005> p

eax=40000358 ebx=40000358 ecx=096f0000 edx=096f2d34 esi=09759d70 edi=72de9730

eip=69da06d7 esp=0446eeec ebp=0446eef8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

msado15!CRecordGroup::AllocateHRowRange+0x75:

69da06d7 8d048504000000 [B][COLOR="red"] lea     eax,[eax*4+4]	// 分配的堆块大小= 0x40000358*4+4 = 0xd64，这里eax的最大值是0xFFFFFFFF，经eax*4+4后变成0x100000D64> 0xFFFFFFFF造成整数溢出，结果溢出后等于0xD64。如果我们把CacheSize设置成0x3FFFFFFF，那么后面分配的堆块就是0了。[/COLOR][/B]

0:005> p

eax=00000d64 ebx=40000358 ecx=096f0000 edx=096f2d34 esi=09759d70 edi=72de9730

eip=69da06de esp=0446eeec ebp=0446eef8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

msado15!CRecordGroup::AllocateHRowRange+0x7c:

69da06de 50              [COLOR="red"][B]push    eax	// 堆块大小=0xd64[/B][/COLOR]

0:005> p

eax=00000d64 ebx=40000358 ecx=096f0000 edx=096f2d34 esi=09759d70 edi=72de9730

eip=69da06df esp=0446eee8 ebp=0446eef8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

msado15!CRecordGroup::AllocateHRowRange+0x7d:

69da06df 680000a000      push    0A00000h

0:005> p

eax=00000d64 ebx=40000358 ecx=096f0000 edx=096f2d34 esi=09759d70 edi=72de9730

eip=69da06e4 esp=0446eee4 ebp=0446eef8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

msado15!CRecordGroup::AllocateHRowRange+0x82:

69da06e4 51              push    ecx

0:005> p

eax=00000d64 ebx=40000358 ecx=096f0000 edx=096f2d34 esi=09759d70 edi=72de9730

eip=69da06e5 esp=0446eee0 ebp=0446eef8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

msado15!CRecordGroup::AllocateHRowRange+0x83:

69da06e5 ffd7           [COLOR="Red"] call    edi {MSDART!MpHeapAlloc (72de9730)}[/COLOR]
```

## 技巧四：基于条件记录断点的漏洞分析技巧

以 `CVE-2012-0774` Adobe Reader TrueType 字体整数溢出漏洞为例，该漏洞主要是在解析 TTF 字体中的虚拟指令导致的溢出，为了确定导致溢出的是哪条虚拟指令，就可以通过设置条件记录断点来实现。

打开 poc 触发崩溃后，通过栈回溯定位漏洞函数，此处标记为 `VulFunction`，它就是用于索引虚拟指令处理函数的。

![](https://bbs.kanxue.com/upload/attach/201403/124373_ce9h0okxlf08yeh.png)

通过快捷键 Shift + F4 对 `VulFunction` 下条件记录断点，记录每次调用 `VulFunction` 时对应的虚拟指令索引号，

![](https://bbs.kanxue.com/upload/attach/201403/124373_aam6hlcdlngsswx.png)

设置好后重新加载 `AcroRd32.exe` 运行（不要关闭调试器否则前面设置的断点可能失效），打开 `poc.pdf` 运行后查看日志窗口：

![](https://bbs.kanxue.com/upload/attach/201403/124373_2blt3mfn4lzg2ar.png)

导致漏洞的虚拟指令索引号为 `0x26`，通过苹果官方提供的指令集b9fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2M7s2m8D9k6g2)9J5k6h3y4G2L8g2)9J5c8X3k6G2L8Y4c8K6i4K6u0r3N6s2c8J5k6h3k6E0j5h3&6Q4x3V1k6d9e0e0l9#2i4K6u0r3b7$3S2S2M7o6g2Q4x3X3g2Z5N6r3#2D9i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4K6S2r3i4@1q4r3i4@1f1@1i4@1u0n7i4@1p5#2i4@1f1$3i4K6W2r3i4@1p5#2i4@1f1#2i4K6R3^5i4@1t1H3i4@1f1#2i4@1q4p5i4K6V1%4i4@1f1^5i4K6S2m8i4K6R3J5i4@1f1%4i4@1p5H3i4K6R3I4x3s2R3J5y4W2!0q4y4g2!0m8c8W2!0n7z5g2!0q4y4g2!0n7b7g2)9&6y4q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4z5q4)9&6z5g2)9&6b7g2!0q4y4W2)9^5b7W2)9&6c8W2!0q4y4W2)9^5b7#2)9^5y4#2!0q4y4q4!0n7b7W2!0m8y4q4!0q4y4q4!0n7z5q4!0n7b7f1#2u0e0V1c8q4h3q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4W2!0m8c8q4!0m8x3#2!0q4y4W2)9&6z5q4!0m8c8W2!0q4z5q4!0m8c8W2!0m8y4g2!0q4y4W2)9&6c8q4!0m8x3g2!0q4y4W2)9^5b7#2)9^5y4#2!0q4y4q4!0n7b7W2!0m8y4q4!0q4y4g2!0m8c8W2!0n7b7#2!0q4z5q4)9^5y4#2!0n7y4q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4W2!0n7b7g2!0m8x3W2!0q4y4g2)9^5y4#2!0n7b7g2!0q4x3#2)9^5x3q4)9^5x3R3`.`.

## 技巧五：基于 JS 日志的漏洞分析技巧

有时在调试 IE 漏洞时，我们需要样本中的 JS 代码的执行先后情况，通过添加一些数学函数，比如 `math.atan2`、`math.asin` 等，还可以使用其它函数，然后通过对 `jscript!Js::Math::Atan`（或者其它函数）下断点来输出 log，以方便分析者观察执行情况。比如 poc 中有如下 js，其中的 `math.atan2` 是我们添加的：

```
Math.atan2(0xbabe, "[*] Creating object button...");

	var obj = document.createElement("button");

	Math.atan2(0xbabe, "[*] Assigning data to title...");

	obj.title = data.substring(0,0x40000-0x58);

	Math.atan2(0xbabe, "[*] Let's AppendChild");

	div_container.appendChild(obj);
```

通过设置以下断点：

```
bu jscript!JsAtan2 ".printf \"%mu\", poi(poi(poi(esp+14)+8)+8);.echo;g"
```

运行后在 `windbg` 上的输出窗口就会看到：

```
[*] Creating object button...

[*] Assigning data to title...

[*] Let's AppendChild
```

## 技巧六：通过虚拟机快照来固定堆地址

最早听到这个方法，是 instruder 在 QQ 群里面提到的。由于在分析漏洞时，尤其是堆漏洞时，每个重新加载运行时，分配的堆地址都是固定，无论是分析还是写文档，都不太利用于我们分析和描述。因此我们可以先将程序调试已经完成堆分配的某个地址，然后将其保存为虚拟机快照，等我们需要再重新开始调试时，可通过恢复先前保存的快照来重新调试，那么此时的堆地址跟之前分析的地址都是固定的。

## 技巧七：监控堆分配释放动作来分析 UAF、double free 漏洞

以之前分配 `libpng` 某个 double free 漏洞的分析为例，主要是通过对释放函数 `free` 进行监控并输出记录来分析的。程序崩溃时的栈回溯如下：

![](https://bbs.kanxue.com/upload/attach/201403/124373_h4opb7tpebceqbg.png)

重新用 `windbg` 加载，通过监控堆块的释放过程，可以发现它在对其中某堆块进行了双重释放，先下断：

```
bu 3440D279 ".if(1){.echo EnterVulnFunc;gc}"

bu 6e264b6c ".if(1){.echo Free heap block; dd esp l4;gc}"
```

输出结果：

```
EnterVulnFunc

Free heap block

0011bc5c  [B]3441e2a2[COLOR="red"]138f0020[/COLOR][/B] 3b906313 10027b64

Free heap block

0011bc5c  [B]3441dc6c [COLOR="red"]138f0020[/COLOR][/B] 3b906313 10027b64

(1508.e84): Access violation - code c0000005 (first chance)

First chance exceptions are reported before any exception handling.

This exception may be expected and handled.

eax=138f0018 ebx=138f0020 ecx=6e287a7e edx=10028a70 esi=008a0000 edi=00000000

eip=77691f88 esp=0011bbe8 ebp=0011bbf8 iopl=0         nv up ei pl zr na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010246

ntdll!RtlFreeHeap+0x3a:

77691f88 80780705        cmp     byte ptr [eax+7],5         ds:0023:138f001f=??
```
