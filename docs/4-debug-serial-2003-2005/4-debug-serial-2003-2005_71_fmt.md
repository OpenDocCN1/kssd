
# BadCopy Pro V3.71.0727

> 原文：`www.pediy.com/kssd/pediy06/pediy6359.htm`

BadCopy Pro V3.71.0727 汉化版破解分析

**软件名称：**

BadCopy Pro V3.71.0727 汉化版

**软件简介：**

BadCopy 软件可以在不需要人工干预的情况下帮您读出 CD-ROM 或磁盘上的坏文件。效果显著，还具有智能修复的功能，最大程度挽回您的损失。新增或加强了以下特别功能：1、优化的算法，保证修复拯救数据更快更全。2、支持文件夹直接复制，您甚至可以直接把整个逻辑盘完整复制到另一个文件夹下。3、可以自由定制 BadCopy 的各项纠错参数，保证能高效，最大限度的挽救您的宝贵数据。4、可以直接对隐含或系统文件进行操作。5、文件操作安全性增强，危险操作先行提示，避免不必要的损失。6、可以直接在 BadCopy 里面运行或打开程序。

**下载地址：**

`http://www.skycn.com/soft/3497.html`

------------------------------------------------------------

**破解作者：**

weiyi75 [Dfcg]

**破解工具：**

`olldbg`, `AspackDie`, `peid`, `hiew`, `dede2.40b`

**破解目的：**

个性用户名，对抗 Antidede 程序。

------------------------------------------------------------

**[破解过程]**

这个汉化版本的使用说明中已经含有注册码，好长啊，没数多少位，填入注册成功。标题是授权 `Id=Brought` 云云，看着好不舒服。这个注册码如果可以找出来，也很不错啊，实在不行就爆掉它。

看注册表中它把注册信息写哪去了，作为破解爱好者不难找到位置在

```
[HKEY_CURRENT_USER\Software\JufSoft\BadCopy]

"ShowNotice"="FALSE"

"Key"="[8655115][ID=BROUGHT TO YOU BY ECLIPSE][JUFSOFT-BADCOPY]"

"KeyName"="ID=BROUGHT TO YOU BY ECLiPSE"
```

看到这里，实时比较估计不好跟踪，到是启动跟踪比较相对容易些。破解思路，删除原先的注册信息，爆破关键点，让程序授权给空用户，然后伪造一份注册信息到注册表中成为"合法用户"。像 `Vopt7.X`, `videofixer3.23` Crack 版和我的方法类似，也不失为一种方法。你可能会问，追出注册码不比爆破好吗，下一版还可以用。是的，但一我跟不出注册码，二，这些软件的注册码作者每个版本的算法都不一样，三，在知识不够的情况下，但需要用这个软件爆破是捷径。启动时读取注册表中的值时，先读 `KeyName`, 再读 `Key` 的键值，读完开始分界。

**详细过程：**

启动时读取注册表的必经之函数拦截非 `ADVAPI32` 中的 `RegQueryValueExA` 莫属。

开始，这里是从未输入过假用户名和假注册码的跟踪过程，即注册表中无注册信息，全都为空。

用 `Od` 载入程序，用插件隐藏 `Od`。右键代码窗口-搜索－当前模块中的名称，就是输入表。找到

`ADVAPI32.RegQueryValueExA` 有两处，选第二处，回车，

参考位于 `Unpacked:CODE` 到 `advapi32.RegQueryValueExA`

| 地址 | 反汇编 | 注释 |
| :--- | :--- | :--- |
| `00407174` | `jmp dword ptr ds:[<&advapi32.RegQueryVal` | `ADVAPI32.RegQueryValueExA` |
| `0046F588` | `call <jmp.&advapi32.RegQueryValueExA>` | |
| `004731CE` | `call <jmp.&advapi32.RegQueryValueExA>` | |
| `0047334B` | `call <jmp.&advapi32.RegQueryValueExA>` | |

找到这些调用，点右键在每个命令中下断点，因为我们暂时不清除哪个调用是关键 Call，跟踪时自然明白。

`F9` 运行程序，时刻观察堆栈窗口中的注册表中的键值。 慢慢按 `F9`

```
0046F588    .  E8 E77BF9FF   call <jmp.&advapi32.RegQueryV>; RegQueryValueExA  第一处断点。

0046F58D    .  85C0          test eax, eax

0046F58F    .  75 4F         jnz short Unpacked.0046F5E0

...........................................................

004731CE   |.  E8 A13FF9FF   call <jmp.&advapi32.RegQueryV>; RegQueryValueExA  第三处中断开始读

004731D3   |.  85C0          test eax, eax                  "KeyName"

004731D5   |.  0F94C3        sete bl

004731D8   |.  8B0424        mov eax, dword ptr ss:[esp]

004731DB   |.  E8 78FBFFFF   call Unpacked.00472D58

..........................................................
```

堆栈窗口。

```
0012FD14   004F2A08  |ValueName = "KeyName"

0012FD18   00000000  |Reserved = NULL  注册表中什么信息也没有输,Keyname 不存在。

0012FD1C   0012FD28  |pValueType = 0012FD28

0012FD20   00000000  |Buffer = NULL

0012FD24   0012FD44  pBufSize = 0012FD44

'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

00407174    $- FF25 84B2B700 jmp dword ptr ds:[<&advapi32.>;  ADVAPI32.RegQueryValueExA  F9

0040717A       8BC0          mov eax, eax

0040717C    $- FF25 80B2B700 jmp dword ptr ds:[<&advapi32.>;  ADVAPI32.RegSetValueExA

00407182       8BC0          mov eax, eax

00407184    $- FF25 F0B3B700 jmp dword ptr ds:[<&kernel32.>;  kernel32.CloseHandle

0040718A       8BC0          mov eax, eax
```

堆栈窗口。

```
0012FD0C   004731D3  /CALL 到 RegQueryValueExA 来自 Unpacked.004731CE

0012FD10   00000090  |hKey = 90

0012FD14   004F2A08  |ValueName = "KeyName"  读取完毕 keyname,无内容啊。

0012FD18   00000000  |Reserved = NULL

0012FD1C   0012FD28  |pValueType = 0012FD28

0012FD20   00000000  |Buffer = NULL

0012FD24   0012FD44  pBufSize = 0012FD44

...........................................................
```

`F9` 几次，到了

```
004731CE   |.  E8 A13FF9FF   call <jmp.&advapi32.RegQueryV>; RegQueryValueExA

004731D3   |.  85C0          test eax, eax

004731D5   |.  0F94C3        sete bl

004731D8   |.  8B0424        mov eax, dword ptr ss:[esp]

004731DB   |.  E8 78FBFFFF   call Unpacked.00472D58
```

堆栈窗口。

```
0012FD10   00000090  |hKey = 90

0012FD14   004F2A38  |ValueName = "Key" 开始读 Key

0012FD18   00000000  |Reserved = NULL    注册表中什么信息也没有输,Key 不存在。

0012FD1C   0012FD28  |pValueType = 0012FD28

0012FD20   00000000  |Buffer = NULL

0012FD24   0012FD44  pBufSize = 0012FD44

0012FD28   0012FD40

0012FD2C   004F2A38  ASCII "Key"

..........................................................

00407174    $- FF25 84B2B700 jmp dword ptr ds:[<&advapi32.>;  ADVAPI32.RegQueryValueExA

0040717A       8BC0          mov eax, eax   呵，到站了，在 F9 就飞了。

0040717C    $- FF25 80B2B700 jmp dword ptr ds:[<&advapi32.>;  ADVAPI32.RegSetValueExA

00407182       8BC0          mov eax, eax

00407184    $- FF25 F0B3B700 jmp dword ptr ds:[<&kernel32.>;  kernel32.CloseHandle

0040718A       8BC0          mov eax, eax

0040718C    $- FF25 ECB3B700 jmp dword ptr ds:[<&kernel32.>;  kernel32.CompareStringA

00407192       8BC0          mov eax, eax

00407194    $- FF25 E8B3B700 jmp dword ptr ds:[<&kernel32.>;  kernel32.CreateDirectoryA

0040719A       8BC0          mov eax, eax
```

堆栈窗口。

```
0012FD0C   004731D3  /CALL 到 RegQueryValueExA 来自 Unpacked.004731CE

0012FD10   00000090  |hKey = 90

0012FD14   004F2A38  |ValueName = "Key"

0012FD18   00000000  |Reserved = NULL   读取完毕 key,任无内容啊，郁闷，程序说的。

0012FD1C   0012FD28  |pValueType = 0012FD28

0012FD20   00000000  |Buffer = NULL

0012FD24   0012FD44  pBufSize = 0012FD44

0012FD28   0012FD40

0012FD2C   004F2A38  ASCII "Key"

.............................................................
```

读取注册表键值完毕，程序即将开始比较结果，现在 `Alt+B` 取消所有断点，从系统领空中回到程序中去，`ADVAPI32` 只负责读取注册表，比较真伪可不是它的工作。

`Ctrl+F9`

```
77DA24F7     C2 1800         retn 18

77DA24FA     397D 1C         cmp dword ptr ss:[ebp+1C], ed>

77DA24FD     0F84 5CF50000   je ADVAPI32.77DB1A5F

77DA2503   ^ E9 29FFFFFF     jmp ADVAPI32.77DA2431

77DA2508     3955 18         cmp dword ptr ss:[ebp+18], ed>

77DA250B   ^ 0F84 A6FEFFFF   je ADVAPI32.77DA23B7

77DA2511     837D 08 01      cmp dword ptr ss:[ebp+8], 1

77DA2515     0F85 DB000000   jnz ADVAPI32.77DA25F6

......................................................
```

`Ctrl+F9`

```
004731EA   .  C3             retn

004731EB       90            nop

004731EC   /$  53            push ebx

004731ED   |.  56            push esi
```

`F8`

回家

```
00473200   |.  84C0          test al, al   这里。

00473202   |.  74 06         je short Unpacked.0047320A

00473204   |.  8B4424 04     mov eax, dword ptr ss:[esp+4]

00473208   |.  EB 03         jmp short Unpacked.0047320D

0047320A   |> 83C8 FF       or eax, FFFFFFFF

0047320D   |>  59            pop ecx

0047320E   |.  5A            pop edx

0047320F   |.  5E            pop esi

00473210   |.  5B            pop ebx

00473211   .  C3             retn

00473254   |.  8BD8          mov ebx, eax

00473256   |.  85DB          test ebx, ebx

00473258   |.  7E 4F         jle short Unpacked.004732A9  放心走，没那么快比较。
```

负杂程序要绕一些弯子，你程序跟踪多了，一眼就可以找到关键点。

....................................................

```
004732A9   |> 8BC7          mov eax, edi

004732AB   |.  E8 B017F9FF  call Unpacked.00404A60

004732B0   |>  5A            pop edx

004732B1   |.  5D            pop ebp

004732B2   |.  5F            pop edi

004732B3   |.  5E            pop esi

004732B4   |.  5B            pop ebx

004732B5   .  C3             retn

004F27B6    .  8B45 CC       mov eax, dword ptr ss:[ebp-34>

004F27B9    .  8D55 F0       lea edx, dword ptr ss:[ebp-10>

004F27BC    .  E8 5F69F1FF   call Unpacked.00409120

004F27C1    .  33C0         xor eax, eax

004F27C3    .  5A           pop edx
```



004F27C4&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;59 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pop&nbsp;&nbsp;ecx &nbsp;&nbsp;

004F27C5&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;59 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pop&nbsp;&nbsp;ecx &nbsp;&nbsp;

004F27C6&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;64:8910 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;fs:\[eax\],&nbsp;&nbsp;edx &nbsp;&nbsp;

004F27C9&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;EB&nbsp;&nbsp;0A &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;jmp&nbsp;&nbsp;short&nbsp;&nbsp;Unpacked.004F27D5 &nbsp;&nbsp;

004F27D5&nbsp;&nbsp;&nbsp;&nbsp;&gt;&nbsp;&nbsp;68&nbsp;&nbsp;442A4F00 &nbsp;&nbsp;push&nbsp;&nbsp;Unpacked.004F2A44 &nbsp;&nbsp;

004F27DA&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;6A&nbsp;&nbsp;00 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;push&nbsp;&nbsp;0 &nbsp;&nbsp;

004F27DC&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;68&nbsp;&nbsp;B8000000 &nbsp;&nbsp;push&nbsp;&nbsp;0B8 &nbsp;&nbsp;

004F27E1&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;B0&nbsp;&nbsp;03 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;al,&nbsp;&nbsp;3 &nbsp;&nbsp;

004F27E3&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;E8&nbsp;&nbsp;EC75F1FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.00409DD4 &nbsp;&nbsp;

004F27E8&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;E8&nbsp;&nbsp;6733F1FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.00405B54 &nbsp;&nbsp;

004F27ED&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;52 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;push&nbsp;&nbsp;edx &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;;&nbsp;/Arg2 &nbsp;&nbsp;

004F27EE&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;50 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;push&nbsp;&nbsp;eax &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|Arg1 &nbsp;&nbsp;

004F27EF&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;8D45&nbsp;&nbsp;C8 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lea&nbsp;&nbsp;eax,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-38\]&gt;&nbsp;;&nbsp;| &nbsp;&nbsp;

004F27F2&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;E8&nbsp;&nbsp;756BF1FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.0040936C &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Unpacked.0040936C &nbsp;&nbsp;

004F27F7&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;FF75&nbsp;&nbsp;C8 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;push&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-38\] &nbsp;&nbsp;

004F27FA&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;68&nbsp;&nbsp;502A4F00 &nbsp;&nbsp;push&nbsp;&nbsp;Unpacked.004F2A50 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ASCII&nbsp;&nbsp;"\]\[" &nbsp;&nbsp;

004F27FF&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;8D55&nbsp;&nbsp;C0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lea&nbsp;&nbsp;edx,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-40\]&gt; &nbsp;&nbsp;

004F2802&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;A1&nbsp;&nbsp;18A2B600 &nbsp;&nbsp;mov&nbsp;&nbsp;eax,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ds:\[B6A218\]&gt; &nbsp;&nbsp;

004F2807&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;E8&nbsp;&nbsp;C466F1FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.00408ED0 &nbsp;&nbsp;

004F280C&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;8B45&nbsp;&nbsp;C0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;eax,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-40\]&gt; &nbsp;&nbsp;

004F280F&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;8D55&nbsp;&nbsp;C4 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lea&nbsp;&nbsp;edx,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-3C\]&gt; &nbsp;&nbsp;

004F2812&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;E8&nbsp;&nbsp;0969F1FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.00409120 &nbsp;&nbsp;

004F2817&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;FF75&nbsp;&nbsp;C4 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;push&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-3C\] &nbsp;&nbsp;

004F281A&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;68&nbsp;&nbsp;5C2A4F00 &nbsp;&nbsp;push&nbsp;&nbsp;Unpacked.004F2A5C &nbsp;&nbsp;

004F281F&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;68&nbsp;&nbsp;682A4F00 &nbsp;&nbsp;push&nbsp;&nbsp;Unpacked.004F2A68 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ASCII&nbsp;&nbsp;"\[JUFSOFT-BADCOPY\]" &nbsp;&nbsp;

004F2824&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;8D45&nbsp;&nbsp;F4 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lea&nbsp;&nbsp;eax,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-C\] &nbsp;&nbsp;

004F2827&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;BA&nbsp;&nbsp;06000000 &nbsp;&nbsp;mov&nbsp;&nbsp;edx,&nbsp;&nbsp;6 &nbsp;&nbsp;

004F282C&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;E8&nbsp;&nbsp;A725F1FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.00404DD8 &nbsp;&nbsp;

004F2831&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;B3&nbsp;&nbsp;01 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;bl,&nbsp;&nbsp;1 &nbsp;&nbsp;

004F2833&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;A1&nbsp;&nbsp;18A2B600 &nbsp;&nbsp;mov&nbsp;&nbsp;eax,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ds:\[B6A218\]&gt; &nbsp;&nbsp;

004F2838&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;E8&nbsp;&nbsp;DB24F1FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.00404D18 &nbsp;&nbsp;

004F283D&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;83F8&nbsp;&nbsp;05 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cmp&nbsp;&nbsp;eax,&nbsp;&nbsp;5 &nbsp;&nbsp;

004F2840&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;0F8E&nbsp;&nbsp;B1000000 &nbsp;&nbsp;jle&nbsp;&nbsp;Unpacked.004F28F7 &nbsp;&nbsp;

004F28F7&nbsp;&nbsp;&nbsp;&nbsp;&gt;&nbsp;&nbsp;8B45&nbsp;&nbsp;F4 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;eax,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-C\] &nbsp;&nbsp; &nbsp;

004F28FA&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;8B55&nbsp;&nbsp;F0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;edx,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-10\]&gt; &nbsp;&nbsp; &nbsp;

004F28FD&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;E8&nbsp;&nbsp;5A25F1FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.00404E5C &nbsp;&nbsp; &nbsp;

004F2902&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;75&nbsp;&nbsp;1C &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;jnz&nbsp;&nbsp;short&nbsp;&nbsp;Unpacked.004F2920 &nbsp;&nbsp; &nbsp;

004F2904&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;8D55&nbsp;&nbsp;BC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lea&nbsp;&nbsp;edx,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-44\]&gt; &nbsp;&nbsp;

004F2907&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;A1&nbsp;&nbsp;18A2B600 &nbsp;&nbsp;mov&nbsp;&nbsp;eax,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ds:\[B6A218\]&gt; &nbsp;&nbsp;

004F290C&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;E8&nbsp;&nbsp;0F68F1FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.00409120 &nbsp;&nbsp;

004F2911&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;837D&nbsp;&nbsp;BC&nbsp;&nbsp;00 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cmp&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-44\],&nbsp;&nbsp;0 &nbsp;&nbsp;

004F2915&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;74&nbsp;&nbsp;09 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;je&nbsp;&nbsp;short&nbsp;&nbsp;Unpacked.004F2920 &nbsp;&nbsp;

004F2917&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;C745&nbsp;&nbsp;FC&nbsp;&nbsp;05270&gt; &nbsp;&nbsp;mov&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-4\],&nbsp;&nbsp;270&gt; &nbsp;&nbsp;

004F291E&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;EB&nbsp;&nbsp;05 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;jmp&nbsp;&nbsp;short&nbsp;&nbsp;Unpacked.004F2925 &nbsp;&nbsp;

004F2981&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;E8&nbsp;&nbsp;A687F1FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.0040B12C &nbsp;&nbsp;

004F2986&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;DD5D&nbsp;&nbsp;B4 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;fstp&nbsp;&nbsp;qword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-4C\] &nbsp;&nbsp;

004F2989&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;9B &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;wait &nbsp;&nbsp;

004F298A&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;E8&nbsp;&nbsp;6189F1FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.0040B2F0 &nbsp;&nbsp;

004F298F&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;DC5D&nbsp;&nbsp;B4 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;fcomp&nbsp;&nbsp;qword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-4C\] &nbsp;&nbsp;

004F2992&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;DFE0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;fstsw&nbsp;&nbsp;ax &nbsp;&nbsp;

004F2994&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;9E &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sahf &nbsp;&nbsp;

004F2995&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;76&nbsp;&nbsp;19 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;jbe&nbsp;&nbsp;short&nbsp;&nbsp;Unpacked.004F29B0 &nbsp;&nbsp;

004F2997&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;8B45&nbsp;&nbsp;F4 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;eax,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-C\] &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Eax&nbsp;&nbsp;中是注册重要信息，自己看。 &nbsp;&nbsp;

004F299A&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;8B55&nbsp;&nbsp;F0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;edx,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-10\] &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们没有输入&nbsp;&nbsp;Key&nbsp;&nbsp;的假键值 &nbsp;&nbsp;

004F299D&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;E8&nbsp;&nbsp;BA24F1FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.00404E5C &nbsp;&nbsp;

004F29A2&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;74&nbsp;&nbsp;0C &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;je&nbsp;&nbsp;short&nbsp;&nbsp;Unpacked.004F29B0 &nbsp;&nbsp;

004F29A4&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;6A&nbsp;&nbsp;0A &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;push&nbsp;&nbsp;0A &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;;&nbsp;/Timeout&nbsp;&nbsp;=&nbsp;&nbsp;10.&nbsp;&nbsp;ms &nbsp;&nbsp;

004F29A6&nbsp;&nbsp;&nbsp;&nbsp;. &nbsp;E8&nbsp;&nbsp;41BCF1FF &nbsp;&nbsp;call&nbsp;&nbsp;&lt;jmp.&amp;kernel32.Sleep&gt; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Sleep &nbsp;&nbsp;

```text
.................................................................... 
```

004EE47D&nbsp;&nbsp;|. &nbsp;83C0&nbsp;&nbsp;0B &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;add&nbsp;&nbsp;eax,&nbsp;&nbsp;0B &nbsp;&nbsp;

004EE480&nbsp;&nbsp;|. &nbsp;3D&nbsp;&nbsp;10270000 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cmp&nbsp;&nbsp;eax,&nbsp;&nbsp;2710 &nbsp;&nbsp;

004EE485 &nbsp;&nbsp;&nbsp;&nbsp;75&nbsp;&nbsp;40 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;jnz&nbsp;&nbsp;short&nbsp;&nbsp;Unpacked.004EE4C7 &nbsp;&nbsp;开始分界了，跳走就授权给你了。爆破点。 &nbsp;&nbsp;

004EE487&nbsp;&nbsp;|. &nbsp;FF75&nbsp;&nbsp;FC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;push&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-4\] &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;必须跳走，飞向明教光明顶。 &nbsp;&nbsp;

004EE48A&nbsp;&nbsp;|. &nbsp;68&nbsp;&nbsp;E8E54E00 &nbsp;&nbsp;push&nbsp;&nbsp;Unpacked.004EE5E8 &nbsp;&nbsp;

004EE48F&nbsp;&nbsp;|. &nbsp;A1&nbsp;&nbsp;7C605000 &nbsp;&nbsp;mov&nbsp;&nbsp;eax,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ds:\[50607C\] &nbsp;&nbsp;

004EE494&nbsp;&nbsp;|. &nbsp;FF30 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;push&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ds:\[eax\] &nbsp;&nbsp;

004EE496&nbsp;&nbsp;|. &nbsp;68&nbsp;&nbsp;FCE54E00 &nbsp;&nbsp;push&nbsp;&nbsp;Unpacked.004EE5FC &nbsp;&nbsp;

004EE49B&nbsp;&nbsp;|. &nbsp;8D45&nbsp;&nbsp;F8 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lea&nbsp;&nbsp;eax,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-8\] &nbsp;&nbsp;

004EE49E&nbsp;&nbsp;|. &nbsp;BA&nbsp;&nbsp;04000000 &nbsp;&nbsp;mov&nbsp;&nbsp;edx,&nbsp;&nbsp;4 &nbsp;&nbsp;

004EE4A3&nbsp;&nbsp;|. &nbsp;E8&nbsp;&nbsp;3069F1FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.00404DD8 &nbsp;&nbsp;

004EE4A8&nbsp;&nbsp;|. &nbsp;8B55&nbsp;&nbsp;F8 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;edx,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-8\] &nbsp;&nbsp;

004EE4AB&nbsp;&nbsp;|. &nbsp;A1&nbsp;&nbsp;E4A1B600 &nbsp;&nbsp;mov&nbsp;&nbsp;eax,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ds:\[B6A1E4\] &nbsp;&nbsp;

004EE4B0&nbsp;&nbsp;|. &nbsp;E8&nbsp;&nbsp;AB23F6FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.00450860 &nbsp;&nbsp;

```text
.................................................................... 
```

004EE4C7&nbsp;&nbsp;|&gt;&nbsp;&nbsp;FF75&nbsp;&nbsp;FC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;push&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-4\] &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;; &nbsp;&nbsp;Unpacked.004EE5C0 &nbsp;&nbsp;

004EE4CA&nbsp;&nbsp;|. &nbsp;68&nbsp;&nbsp;08E64E00 &nbsp;&nbsp;push&nbsp;&nbsp;Unpacked.004EE608 &nbsp;&nbsp;

004EE4CF&nbsp;&nbsp;|. &nbsp;8D55&nbsp;&nbsp;F0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lea&nbsp;&nbsp;edx,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-10\] &nbsp;&nbsp;

004EE4D2&nbsp;&nbsp;|. &nbsp;B8&nbsp;&nbsp;94CF4E00 &nbsp;&nbsp;mov&nbsp;&nbsp;eax,&nbsp;&nbsp;Unpacked.004ECF94 &nbsp;&nbsp;

004EE4D7&nbsp;&nbsp;|. &nbsp;E8&nbsp;&nbsp;D088F1FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.00406DAC &nbsp;&nbsp;

004EE4DC&nbsp;&nbsp;|. &nbsp;FF75&nbsp;&nbsp;F0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;push&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-10\] &nbsp;&nbsp;

004EE4DF&nbsp;&nbsp;|. &nbsp;68&nbsp;&nbsp;FCE54E00 &nbsp;&nbsp;push&nbsp;&nbsp;Unpacked.004EE5FC &nbsp;&nbsp;

004EE4E4&nbsp;&nbsp;|. &nbsp;8D45&nbsp;&nbsp;F4 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lea&nbsp;&nbsp;eax,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-C\] &nbsp;&nbsp;

004EE4E7&nbsp;&nbsp;|. &nbsp;BA&nbsp;&nbsp;04000000 &nbsp;&nbsp;mov&nbsp;&nbsp;edx,&nbsp;&nbsp;4 &nbsp;&nbsp;

004EE4EC&nbsp;&nbsp;|. &nbsp;E8&nbsp;&nbsp;E768F1FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.00404DD8 &nbsp;&nbsp;

004EE4F1&nbsp;&nbsp;|. &nbsp;8B55&nbsp;&nbsp;F4 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;edx,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ss:\[ebp\-C\] &nbsp;&nbsp;

004EE4F4&nbsp;&nbsp;|. &nbsp;A1&nbsp;&nbsp;E4A1B600 &nbsp;&nbsp;mov&nbsp;&nbsp;eax,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;ds:\[B6A1E4\] &nbsp;&nbsp;

004EE4F9&nbsp;&nbsp;|. &nbsp;E8&nbsp;&nbsp;6223F6FF &nbsp;&nbsp;call&nbsp;&nbsp;Unpacked.00450860 &nbsp;&nbsp;

打上评估版本的标记。

## 破解小结

双击 `004EE485` 这行 `jnz short Unpacked.004EE4C7`

直接修改为 `jz short Unpacked.004EE4C7` 编汇编按钮，关闭。

右键代码窗口－复制到可执行程序－全部修正－复制。

然后在新窗口中右键－保存文件，`Badcopyc.exe`。

运行程序，爆破成功，授权人是谁呢？

伪造注册用户信息，`keyname` 可个性话。

```
REGEDIT4

[HKEY_CURRENT_USER\Software\JufSoft\BadCopy]

"ShowNotice"="FALSE"

"Key"="999999999999999999999999999999"

"KeyName"=" 给 weiyi75"
```

保存为 `user.reg`，加入注册表，完工，比正版用户如何。

这篇文章我觉得很像脱壳啊，受 `Fly` 影响了，另外算法，管它是 `rsa` 还是 `brofish` 算法，可不是 `weiyi75` 的事。

谢谢大家耐心看完!

补充，虽然软件叫 `Badcopy`，不过好盘一样可以 `copy`。选择硬盘中的某文件恢复，到最后一步提示你是评估版，无法保存结果，必须在线注册。怒!和我猫腻，我吃定你了。

用 `Dede3.0` 反编译程序，停止响应???你的是不是这样，`Antidede`，谁让 `Dede` 公开代码的呢，看!不过我还有一个必杀 `Dede` 版本----------------------`Dede2.40beta` 版本,根本不运行程序就开始反编译，你如何 `Anti`???超酷!

反编译后如果你有一些汉化的经验就更好，至少应该对 `Dephi` 资源有些概念。

点 `Procedures` 标签，

在 `mainunit` 处点一下，再点 `BNextClick`

```text
***** TRY
|
```

004EE7B2 &nbsp;64FF30 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;push&nbsp;&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;fs:\[eax\] &nbsp;&nbsp;

004EE7B5 &nbsp;648920 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;fs:\[eax\],&nbsp;&nbsp;esp &nbsp;&nbsp;

004EE7B8 &nbsp;8B45FC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;eax,&nbsp;&nbsp;\[ebp\-$04\] &nbsp;&nbsp;

004EE7BB &nbsp;E8F81D0000 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;call&nbsp;&nbsp;&nbsp;004F05B8 &nbsp;&nbsp;

004EE7C0 &nbsp;8D55EC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lea&nbsp;&nbsp;&nbsp;&nbsp;edx,&nbsp;&nbsp;\[ebp\-$14\] &nbsp;&nbsp;

004EE7C3 &nbsp;8B45FC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;eax,&nbsp;&nbsp;\[ebp\-$04\] &nbsp;&nbsp;

`* Possible reference to control 'MainNote':TNotebook`

```text
|
```

004EE7C6 &nbsp;8B805C030000 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;eax,&nbsp;&nbsp;\[eax+$035C\] &nbsp;&nbsp;

004EE7CC &nbsp;E8176BF5FF &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;call&nbsp;&nbsp;&nbsp;004452E8 &nbsp;&nbsp;

004EE7D1 &nbsp;8B45EC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;eax,&nbsp;&nbsp;\[ebp\-$14\] &nbsp;&nbsp;

`* Possible String Reference to: "SaveFile" &nbsp;保存文件`

```text
|
```

004EE7D4 &nbsp;BA74EE4E00 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;edx,&nbsp;&nbsp;$004EEE74 &nbsp;&nbsp;

004EE7D9 &nbsp;E87E66F1FF &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;call&nbsp;&nbsp;&nbsp;00404E5C &nbsp;&nbsp;

004EE7DE &nbsp;0F857D010000 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;jnz&nbsp;&nbsp;&nbsp;004EE961 &nbsp;&nbsp;

004EE7E4 &nbsp;A1BC5D5000 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;eax,&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;\[$505DBC\] &nbsp;&nbsp;

004EE7E9 &nbsp;803800 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cmp&nbsp;&nbsp;&nbsp;&nbsp;byte&nbsp;&nbsp;ptr&nbsp;&nbsp;\[eax\],&nbsp;&nbsp;$00 &nbsp;&nbsp;

004EE7EC &nbsp;750D &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;jnz&nbsp;&nbsp;&nbsp;004EE7FB &nbsp;&nbsp;

004EE7EE &nbsp;8B45FC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;eax,&nbsp;&nbsp;\[ebp\-$04\] &nbsp;&nbsp;

004EE7F1 &nbsp;E85E1C0000 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;call&nbsp;&nbsp;&nbsp;004F0454 &nbsp;&nbsp;

004EE7F6 &nbsp;E90F060000 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;jmp&nbsp;&nbsp;&nbsp;004EEE0A &nbsp;&nbsp;

004EE7FB &nbsp;8D55E4 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lea&nbsp;&nbsp;&nbsp;&nbsp;edx,&nbsp;&nbsp;\[ebp\-$1C\] &nbsp;&nbsp;

004EE7FE &nbsp;8B45FC &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;eax,&nbsp;&nbsp;\[ebp\-$04\] &nbsp;&nbsp;

`* Possible reference to control 'ESaveFile':TEdit`

```text
|
```

004EE801 &nbsp;8B8098040000 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;eax,&nbsp;&nbsp;\[eax+$0498\] &nbsp;&nbsp;

004EE807 &nbsp;E82420F6FF &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;call&nbsp;&nbsp;&nbsp;00450830 &nbsp;&nbsp;

004EE80C &nbsp;8B45E4 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;eax,&nbsp;&nbsp;\[ebp\-$1C\] &nbsp;&nbsp;

004EE80F &nbsp;8D55E8 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lea&nbsp;&nbsp;&nbsp;&nbsp;edx,&nbsp;&nbsp;\[ebp\-$18\] &nbsp;&nbsp;

004EE812 &nbsp;E809A9F1FF &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;call&nbsp;&nbsp;&nbsp;00409120 &nbsp;&nbsp;

004EE817 &nbsp;837DE800 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cmp&nbsp;&nbsp;&nbsp;&nbsp;dword&nbsp;&nbsp;ptr&nbsp;&nbsp;\[ebp\-$18\],&nbsp;&nbsp;+$00 &nbsp;&nbsp;



## 汇编代码分析

```
004EE81B   751A                    jnz     004EE837   

004EE81D   8D55E0                  lea     edx, [ebp-$20]   

004EE820   B89CCF4E00             mov     eax, $004ECF9C   

004EE825   E88285F1FF             call    00406DAC   

004EE82A   8B45E0                  mov     eax, [ebp-$20]   

004EE82D   E8C6B7F5FF             call    00449FF8   

004EE832   E9D3050000             jmp     004EEE0A   

004EE837   E8B4CAF1FF             call    0040B2F0   

004EE83C   D81D80EE4E00           fcomp   dword ptr [$4EEE80]   

004EE842   DFE0                   fstsw   ax   

004EE844   9E                     sahf   

004EE845   7223                   jb      004EE86A   

004EE847   8B157C605000           mov     edx, [$50607C]   

004EE84D   8B12                   mov     edx, [edx]  取标题名字，我的是给 weiyi75   

* Possible String Reference to: "ID="  猫腻点。   

|   

004EE84F   B88CEE4E00             mov     eax, $004EEE8C   和 ID=字符   

004EE854   E8FB67F1FF             call    00405054  比较第一处字符是 ID=开始的吗?   

004EE859   85C0                   test    eax, eax   

004EE85B   750D                   jnz     004EE86A  爆破点，修改让它再次飞向明教光明顶。   

004EE85D   8B45FC                 mov     eax, [ebp-$04]   

004EE860   E8EF1B0000             call    004F0454  评估限制出现   

004EE865   E9A0050000             jmp     004EEE0A   

004EE86A   66BA0500               mov     dx, $0005   

004EE86E   A1ECA1B600             mov     eax, dword ptr [$B6A1EC]   

004EE873   E8C8F6FFFF             call    004EDF40   

004EE878   8D55DC                 lea     edx, [ebp-$24]   

004EE87B   B8A4CF4E00             mov     eax, $004ECFA4   

004EE880   E82785F1FF             call    00406DAC   

004EE885   8B55DC                 mov     edx, [ebp-$24]   

004EE888   A1ECA1B600             mov     eax, dword ptr [$B6A1EC]   

004EE88D   E866F7FFFF             call    004EDFF8
```

------------------------------------------------------------

用 `hiew` 修改 `004EE85B   750D  jnz     004EE86A`   

为   

`004EE85B   740D   jz     004EE86A`   

实在想当正版用户，我们通过分析得知。   

`004F2997    .  8B45 F4       mov eax, dword ptr ss:[ebp-C]      Eax 中是注册重要信息，自己看。`   

再次用 `Od` 拦截看到 `[8655115][ID=给 WEIYI75][JUFSOFT-BADCOPY]` 是个固定格式，中间是授权人名。   

将 `keyname` 给 WEIYI75 转为大写，对本上面的猫腻分析，知道结果如下。   

再次用计事本新建一个 `User.reg`,内容如下。   

```
REGEDIT4   

[HKEY_CURRENT_USERSoftwareJufSoftBadCopy]   

"ShowNotice"="FALSE"   

"Key"="[8655115][ID=给 WEIYI75][JUFSOFT-BADCOPY]"   

"KeyName"="ID=给 weiyi75"
```

这个是仿正版 `reg` 注册文件。   

导入后仍然是评估版本，跟进   

```
004EE473   |.  E8 8066F1FF   call Unpacked.00404AF8   

004EE478   |.  E8 BF410000   call Unpacked.004F263C  F7 跟进去看看   

004EE47D   |.  83C0 0B       add eax, 0B   

004EE480   |.  3D 10270000   cmp eax, 2710   

004EE485       75 40         jnz short Unpacked.004EE4C7   

004F263C    $  55           push ebp  到这里。   

004F263D    .  8BEC          mov ebp, esp   

004F263F    .  B9 09000000   mov ecx, 9   

004F2644    >  6A 00         push 0   

004F2646    .  6A 00         push 0   

004F2648    .  49           dec ecx   

004F2649    .^ 75 F9        jnz short Unpacked.004F264   

.............................................
```

里面比较了 `keyname`,`key`,`version`   

```
004F26C9    .  BA 082A4F00   mov edx, Unpacked.004F2A08        ;  ASCII "KeyName"   

004F26CE    .  8B45 F8       mov eax, dword ptr ss:[ebp-8]   

004F26D1    .  E8 6A0BF8FF   call Unpacked.00473240   

004F275E    .  68 8E274F00   push Unpacked.004F278E   

004F2763    .  64:FF30       push dword ptr fs:[eax]   

004F2766    .  64:8920       mov dword ptr fs:[eax], esp   

004F2769    .  8D4D D0       lea ecx, dword ptr ss:[ebp-30]   

004F276C    .  BA 282A4F00   mov edx, Unpacked.004F2A28        ;  ASCII "Version"   

004F2771    .  8B45 F8       mov eax, dword ptr ss:[ebp-8]   

004F2774    .  E8 C70AF8FF   call Unpacked.00473240   

004F2779    .  8B45 D0       mov eax, dword ptr ss:[ebp-30]   

004F277C    .  8D55 E8       lea edx, dword ptr ss:[ebp-18]   

004F277F    .  E8 9C69F1FF   call Unpacked.00409120   

004F2784    .  33C0          xor eax, eax   

004F2786    .  5A           pop edx   

004F2787    .  59           pop ecx   

004F2788    .  59           pop ecx   

004F2789    .  64:8910      mov dword ptr fs:[eax], edx   

004F279B    .  68 CB274F00   push Unpacked.004F27CB   

004F27A0    .  64:FF30       push dword ptr fs:[eax]   

004F27A3    .  64:8920       mov dword ptr fs:[eax], esp   

004F27A6    .  8D4D CC       lea ecx, dword ptr ss:[ebp-34]   

004F27A9    .  BA 382A4F00   mov edx, Unpacked.004F2A38        ;  ASCII "Key"   

004F27AE    .  8B45 F8       mov eax, dword ptr ss:[ebp-8]   

004F27B1    .  E8 8A0AF8FF   call Unpacked.00473240   

004F2838    .  E8 DB24F1FF   call Unpacked.00404D18  这里不能跳走,应为程序发现几处键值都是空   

004F283D    .  83F8 05       cmp eax, 5           要给你打评估版本标记了。   

004F2840    .  0F8E B1000000 jle Unpacked.004F28F7   

004F2846    .  8B45 E8       mov eax, dword ptr ss:[ebp-18]   

004F2849    .  BA 842A4F00   mov edx, Unpacked.004F2A84        ;  ASCII "3.71.0727"   

004F284E    .  E8 0926F1FF   call Unpacked.00404E5C  这个就是 Version 注册键值。
```

结果如下:   

```
REGEDIT4   

[HKEY_CURRENT_USERSoftwareJufSoftBadCopy]   

"ShowNotice"="FALSE"   

"Key"="[8655115][ID=给 WEIYI75][JUFSOFT-BADCOPY]"   

"KeyName"="ID=给 weiyi75"   

"Version"="3.71.0727"
```

这个是完全仿正版 `reg` 注册文件。   

破解版，必须修改以下两处跳转。   

`004EE485`   

`004EE85B`   

破解版的 `Key` 如下。`key` 和 `keyname` 可任意填。   

```
REGEDIT4   

[HKEY_CURRENT_USERSoftwareJufSoftBadCopy]   

"ShowNotice"="FALSE"   

"Key"="99999999999999999999999999999"   

"KeyName"="给 weiyi75"
```

再次感谢大家看完,请大家运行 `badcopy` 选择媒介或其它驱动器，如 `d` 盘中某文件放到它默认的保存地址体验一下。  

`4dd466d0e1e0cc123afbe504da2462ca`
