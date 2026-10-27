
```
005B1D79  |.  48  |vAdd4           DWORD v1213 = 108B302C + v46

005B1D45  |.  67  |vReadMemDs4     DWORD m104 = DWORD DS:[24 + v1213]

005B1CF2  |.  C7  |vNand4          DWORD v1214 = m104 - 0A5E8562E

005B1CD7  |.  11  |vPushReg4 vR9   DWORD v1215 = v1214 : (Setnz(SubFlag(Check(48DB8B, 2E8), 48859C66)) | GetBytes(v1214, 0, 1))//检查检查表是否被修改

005B1CBF  |.  1D  |vPushReg4 vR12  DWORD v1216 = 2E8			//要检查的区域的数量*0C

005B1CBB  |.  25  |vPushReg4 vR14  DWORD v1217 = 48DB8B			//检查表地址

005B1CB9  |.  09  |vPushReg4 vR7   DWORD v1218 = 2E8

005B1CB2  |.  6F  |vAdd4           DWORD v1219 = 0EF74CFD4 + v1213

005B1CB0  |.  21  |vPushReg4 vR13

005B1CAF  |>  0B  |/vPopReg4 vR8

005B1C7F  |.  8C  ||vReadMemDs4    DWORD m105 = DWORD DS:[v1217]	//要检查的偏移

005B1C56  |.  73  ||vNand4         DWORD v1220 = 0 - (~(m105 - 47DFC7B1) + 349CDF75)//解密偏移

005B1C1F  |.  2B  ||vReadMemSs4    DWORD m106 = DWORD SS:[Jnz(SubFlag(v1220, 0FFFFFFFF)) + 0FFFFF800]

005B1BF1  |.  48  ||vAdd4          DWORD v1219 = 0EF74CFD4 + (108B302C + v1219)

005B1BED  |.  94  ||vJmp_005BED37  if (v1220 == 0FFFFFFFF) goto VMPCrack.005B2AFF//偏移等于-1跳过检查

005B1BC9  |.  F3  ||vPopReg4 vR2

005B1BB0  |.  8C  ||vReadMemDs4    DWORD m107 = DWORD DS:[4 + v1217]	//要检查的长度

005B1B49  |.  0C  ||vShrd          DWORD v1221 = (5812DF7B ^ ~m107 + 1 >>< 0E) >>< 0F//解密长度

005B1B03  |.  8C  ||vReadMemDs4    DWORD m108 = DWORD DS:[v1217 + 8]	////校验值必须等于这个数

005B1A84  |.  2B  ||vReadMemSs4    DWORD m109 = DWORD SS:[Je(SubFlag(v1216, 0C)) + 0FFFFF800]

005B1A65  |.  05  ||vPushReg4 vR6  DWORD v1215 = GetBytes(v1215, 1, 3) : (GetBytes(v1215, 0, 1) | Setnz(SubFlag((0 - (Check(v1220 + 400000, v1221) - 9B59C608) ^ 0D2F92494) <<> 0C, m108)))//检查

005B1A63  |.  01  ||vPushReg4 vR5  DWORD v1216 = v1216 - 0C

005B1A5F  |.  F9  ||vPushReg4 vR3  DWORD v1217 = v1217 + 0C

005B1A5D  |.  FD  ||vPushReg4 vR4  DWORD v1218 = v1221

005B1A56  |.  6F  ||vAdd4          DWORD v1219 = 0EF74CFD4 + (108B302C + v1219)

005B1A52  |.^ 06  |\vJmp_005BED37  if (v1216 != 0) goto VMPCrack.005B1CAF

005B2AFF  |>  0B  |vPopReg4 vR8

005B2A91  |.  93  |vReadMemSs4     DWORD m110 = DWORD SS:[Jb(SubFlag(v1219 + 108B302C, 400000)) + 0FFFFF800]

005B2A72  |.  F5  |vPushReg4 vR2   DWORD v18 = v1216

005B2A70  |.  FD  |vPushReg4 vR4   DWORD v17 = v1218

005B2A6A  |.  F9  |vPushReg4 vR3   DWORD v1222 = v1215

005B2A63  |.  48  |vAdd4           DWORD v1223 = 0EF74CFD4 + (108B302C + v1219)

005B2A5F  |.  06  |vJmp_005BED37   if (v1219 + 108B302C < 400000) goto VMPCrack.005B7218//检查刚才分配的内存是否小于400000

005ABF90  |.  0B  |vPopReg4 vR8

005ABF22  |.  2B  |vReadMemSs4     DWORD m111 = DWORD SS:[Jb(SubFlag(v1223 + 108B302C, 5BB9EE)) + 0FFFFF800]

005ABEF3  |.  48  |vAdd4           DWORD v1223 = 108B302C + v1223 + 0EF74CFD4

005ABEEF  |.  94  |vJmp_005BED37   if (v1223 + 108B302C >= 5BB9EE) goto VMPCrack.005B7218//检查刚才分配的内存是否大于等于5BB9EE

005B7275  |.  17  |vPopReg4 vR11

005B7222  |.  1D  |vPushReg4 vR12  DWORD v1222 = GetBytes(v1222, 1, 3) : 1		//地址在文件范围内v1222设为1

005B721B  |.  6F  |vAdd4           DWORD v1223 = 108B302C + v1223 + 0EF74CFD4

005B7219  |.  19  |vPushReg4 vR11

005B7218  |>  F7  |vPopReg4 vR3

005B71AE  |.  93  |vReadMemSs4     DWORD m112 = DWORD SS:[Je(SubFlag(GetBytes(v1222, 0, 1), 0)) + 0FFFFF800]

005B7180  |.  6F  |vAdd4           DWORD v351 = 108B302C + v1223 + 0EF74CFD4

005B717C  |.  94  |vJmp_005BED37   if (GetBytes(v1222, 0, 1) == 0) goto VMPCrack.004E6A0E//等于0转到正常流程，否则转到发现文件改变
```

又检查一遍内存，方法跟刚才差不多，这回循环索引是检查表大小，每次减去一个表项的长度（`0C`），一共是`2E8/0C=3E`个。然后又检查了刚才分配的内存（就是初始数据中的初始化检查）是否大于等于`400000`而且小于`5BB9EE`，也就是在当前文件的范围内，估计是防止脱壳后把它写到文件中。然后是

```
004E6A0E  |> \EF  vPopReg4 vR1

004E69BD  |.  8F  vWriteMemDs4   DWORD DS:[38 + (108B302C + v351)] = v1414; DWORD v1414 = 0FA5463A1

004E69B6  |.  8F  vWriteMemDs4   DWORD DS:[108B302C + v351 + 18] = v1415; DWORD v1415 = 0AA4BF36A

004E698E  |.  8F  vWriteMemDs4   DWORD DS:[108B302C + v351 + 20] = v1416; DWORD v1416 = 4C37F4AB

004E6978  |.  BF  vWriteMemDs4   DWORD DS:[44 + (108B302C + v351)] = v1417; DWORD v1417 = 3371EDD3

004E6966  |.  8F  vWriteMemDs4   DWORD DS:[108B302C + v351 + 10] = v1418; DWORD v1418 = 0BE40E302

004E6956  |.  20  vPopVEsp       vESP = v21 + 0C

004E6920  |.  F1  vPushReg4 vR1

004E691F  |>  1B  vPopReg4 vR12

004E68E9  |.  47  vPopfd         EFL = 0FFFFF700 & Cross(nonentity, v0)

004E68CD  |.  94  vJmp_005BED37  goto VMPCrack.0044DB51

0044DB51  |>  00  DB 00
```

这就结束了，`44DB51`就是真正的入口，由于入口也被虚拟了，这里直接用一个转移转到入口执行。因为分析时代码还没有解压，无法继续分析，插件标记为指令块没有被初始化。现在走到这里已经解压完成了，分析一下看看，数据窗口跟随`44DB51`，右键菜单选择分析虚拟程序，虚拟机填`5BED37`，就是`vJmp`后面的那个数。提示分析错误，看看记录

反汇编虚拟程序时出现错误：`VZhilingXinxi` 未知指令信息无效

提示未知指令的信息无效，不能自动分析。因为代码在不同的虚拟机，刚才分析模块入口点时这些虚拟机和转到他们的`vJmp`还没有解压，这些`vJmp`没有被识别，被当作了未知指令。现在再到模块入口点分析虚拟机，这回出来了，一共找到6个虚拟机，一个是壳自己使用的，就是刚才的那个，其他5个是真正执行虚拟代码的。现在再分析`44DB51`，已经可以正确分析了，不过这里的代码都是编译器生成的，没什么有用的东西，只是记一下调用的api以后修复输入表时用。

知道原理后可以开始脱壳了，等程序运行起来后用`OllyDump`插件保存下来，不重建输入表，入口先不用填，以后再修改。先修复资源，VMP拦截了`LdrFindResource_U`和`LdrAccessResource`，在使用时生成资源，由于这次分析主要针对虚拟机，先不研究这个问题了。这个程序只用了一个资源，就是显示的对话框，把资源找到自己添加进去就可以了。在`DialogBoxParamA`设个断点（别设到第一个字节，会被查），中断后往下跟，走到`FindResourceExA`看看参数，`id`是`0x65`（101），记下来。再往下到`LoadResource`，先看看长度，因为和`SizeofResource`参数相同，直接设置`EIP`到这里，返回`EA`。重启程序到`LoadResource`，把返回值开始`EA`字节内容记录下来，这就是要找的资源。用vc新建一个对话框资源，`id`设为101，选择打开二进制数据，把内容填进去。vc的资源编辑器不能粘贴16进制数据，还是先保存后再编辑吧，先随便填一些内容到`EA`字节，然后另存为`.res`文件，在od中选菜单查看-文件加载刚才的文件，把内容填进去后保存。用Resource Hacker载入`dump`下来的文件，选择添加一个新资源，把`.res`添加进去。

添加两个要用到的输入函数，还有一个新的段，保存修复代码。用Explorer Suite（或其他的PE编辑工具）打开文件，使用其中的引入添加器添加`GetProcAddress`和`VirtualProtect`，选择创建新节，会自动添加一个段。重新载入文件，在节头里面把虚拟大小改成`1000`（4KB），准备保存修复代码。

然后是修复输入表，我没想到什么好的办法，只能模拟VMP填写的过程，在程序执行前添加一段代码来修复。输入函数太多了，手动的话很麻烦，因为VMP的加密只是加上一个常量，我做了一个插件来修复。但是虚拟机内调用的函数加密方法不同，有随机算法，反正数量不多，手动修复吧。准备记录下要修复的函数，先在解压完成后准备填写输入表时设一个断点

```
005B656D  |.^\06  ||vJmp\_005BED37  goto VMPCrack.005B13EC
```

中断后在取输入函数地址返回时设一个条件记录断点

```
005AD2F9  |.  5E 02  ||vCall 2        v37 = Call(5AB3FD); <VMPCrack.输入函数地址>
005AD2F7  |.  13     ||vPopReg4 vR10  DWORD \_t1290 = v37        //在这里设断点
```

注意要先把显示模式设为全部指令，否则可能不会显示，断点说明为`apiAddr`，插件根据这个取地址，表达式为`[vESP]`，只记录表达式，不暂停程序。再设一个`vWriteMemDs4`的指令条件记录断点，也是只记录不暂停，记录把加密后的输入函数地址写入内存，说明设为`writeMem`，表达式设为`[vESP];[vESP+4]`。在填写输入表结束时设一个断点

```
005A8B1B  |.  5E 03  |vCall 3  v1196 = Call(415690); VMPCrack.00415690
```

把`od`的log设置为保存到文件，然后执行直到中断，生成了一个记录文件。用`od`启动`dump`的文件，在输入表修复插件里选加载记录文件，会出现两个窗口，显示普通调用和虚拟机内调用的输入函数，可能有一些壳自己使用的，先不用管。选修复输入表，在刚才添加输入函数时创建的段里找一个位置，`5C3100`，这样普通调用的输入表就修复完成了。看看添加的内容，标出了两个调用要替换成对应的api

```
005C311C      E8 00000000    CALL dump.005C3121                             ; 替换成LoadLibraryA

005C3127      E8 00000000    CALL dump.005C312C                             ; 替换成GetProcAddress
```

看看输入函数地址，下面找个位置填上

```
005C3D30    - FF25 0C255B00  JMP DWORD PTR DS:[<&KERNEL32.LoadLibraryA>]    ; kernel32.LoadLibraryA

005C3D36    - FF25 A7305C00  JMP DWORD PTR DS:[<&kernel32.GetProcAddress>]  ; kernel32.GetProcAddress
```

把刚才的调用改成

```
005C311C      E8 0F0C0000    CALL <JMP.&KERNEL32.LoadLibraryA>              ; 替换成LoadLibraryA

005C3127      E8 0A0C0000    CALL <JMP.&kernel32.GetProcAddress>            ; 替换成GetProcAddress
```

然后把整个段保存到文件。

插件只能修复普通调用，虚拟机内调用要手动修复。插件显示的输入表中有很多没用的内容，这里只修复程序会用到的，看看原来分析过的程序中都调用了哪些函数

`user32.GetDlgItem`  
`user32.GetWindowTextLengthA`  
`user32.MessageBoxA`  
`user32.GetWindowTextA`  
`kernel32.GetVersion`  
`kernel32.GetCommandLineA`  
`kernel32.GetStartupInfoA`  
`kernel32.GetModuleHandleA`

只有这些是需要的，随便找一个

```
user32.dll  GetDlgItem  00000000  0044499C  005B798F
```

指令地址是`005B798F`，看看代码

```
005B79AF  |.  36  ||vShrd         DWORD v429 = (0E6188CDD ^ v37 >>< 2) >>< 18

005B798F  |.  BF  ||vWriteMemDs4  DWORD DS:[44499C] = v430; EXIT DWORD v430 = v429
```

加密算法已经很明显了，添加下面的汇编代码来修复
