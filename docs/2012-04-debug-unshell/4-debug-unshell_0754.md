# Script: MoleBox 2.x Unpacker

> 原文：[`www.pediy.com/kssd/pediy11/106136.html`](https://www.pediy.com/kssd/pediy11/106136.html)

## 文章元信息

- 【文章标题】: `Script: MoleBox 2.x Unpacker`
- 【文章作者】: `CCDebuger`
- 【下载地址】: 自己搜索下载

## 详细过程

这个脚本去年就写好了。当时是基于 `MoleBox` 的新版本写的，后来在测试旧版的时候发现旧版和新版在解包打包的文件时顺序有所不同，新版是把主程序运行时要用到的文件在执行主程序前先解包出来，再解包主程序运行，而旧版是先解出主程序再解打包文件。当然这里说的打包文件不包括进入主程序后运行各项功能再调用的打包文件。本来准备修改一下脚本再支持旧版的。不过当时去玩别的东西了，就扔在那打算过几天再写的。这一过就是大半年，这次从 `MoleBox` 网站下了个 `MoleBox9x` 啥的（就是以前的 `MoleBox Pro` 版本），准备重写一下脚本的。不过看了一下以前的代码，虽然有点注释，但我也不记得以前写的时候是啥思路了。真要改的话还是太费劲了。算了，反正大家现在大部分都用的新版吧？我就不改了，谁有兴趣谁改一下支持旧版吧![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")。以后有空再玩玩 `MoleBox` 网站上那个 `Ultra` 版本。下面对脚本作一下说明吧，主要是针对新版的：

> 1. 脚本支持 `MoleBox` 目前的新版，如 `MoleBox` 网站上的 `MoleBox9x` 版本。旧版（这地方还不太好定义）支持不完整，只支持解包在主程序运行前的文件及主程序，其它的不管。
>
> 2. 脚本类似于脱壳机，会自动解包 `MoleBox` 打包程序的主程序及里面的打包文件，解包后的打包文件和打包前的原始文件基本一样，解包后的主程序名为 `UN_XXXX`（这里的 `XXXX` 就是原加壳的程序名），解包后的程序直接就可以使用。
>
> 3. 脚本自动生成 `FileList.txt` 和 `FileRemainList.txt` 这两个文件，`FileList.txt` 里保存的是所有打包文件的文件名，`FileRemainList.txt` 保存的是运行完脚本后没解包出来的文件名。如果这个文件是空的，那恭喜你，文件都解出来了。
>
> 4. 脚本运行完了可以看看 `OD` 的日志窗口，了解一些信息。

作为此次原打算修改脚本活动的副产品，`MoleBox` 网站上最新版本的 `MoleBox9x.v2.3682` 惨遭蹂躏，这里顺便贴一下 `XXOO` 的记录![](img/962324edb48d57effae0c0571850a66b.jpg "酷")：

### 操作步骤

1. 运行我写的脱壳脚本 `MoleBox 2.x Unpacker.txt`，把除了 `mbox2_bootupDbgLt` 和 `mbox2_bootupLt` 外的所有文件先解出来（剩余没解出来的文件可以通过查看脚本自动生成的 `FileRemainList.txt` 来了解）。

2. 重新用 `OD` 载入程序，忽略所有异常，`F9` 一直跑到注册窗口出现。这个程序在进入主界面的时候会检测调试器，省点力气，直接用海风的 `StrongOD`，把内核模式直接选上，隐藏调试器即可。

3. 随便输入注册名和注册码，点注册的时候会弹出一个注册码错误的窗口。现在在 `OD` 中设断 `bp MessageBoxA`，重新点那个注册按钮，会被断下。现在根据堆栈返回到相关位置（注意别在前面跑完脚本到注册窗口的时候设断拦那个注册错误对话框，拦不下来的）：

    程序检查注册码是在捆绑的 `RegisterPro.dll` 中。断下时的堆栈情况：

    ```
    0012F6DC   00C61484  /CALL 到 MessageBoxA 来自 MBX@A6_2.00C6147E
    0012F6E0   006707AA  |hOwner = 006707AA ('Register MoleBox',class='#32770')
    0012F6E4   00C77068  |Text = "Incorrect registration code"
    0012F6E8   00C77054  |Title = "Registration error"
    0012F6EC   00000010  \Style = MB_OK|MB_ICONHAND|MB_APPLMODAL
    ```

    右键点击 `0012F6DC` 这第一条指令，选择“反汇编窗口中跟随”：

    ```
    00C613F0    51             PUSHECX
    00C613F1    E8 B60A0000    CALL 00C61EAC    ; 注册算法，直接在这里设个硬件执行断点，重新点注册断下后跟进去看看
    00C613F6    83C4 0C        ADDESP,0C
    00C613F9    25 FF000000    ANDEAX,0FF
    00C613FE    85C0           TESTEAX,EAX; 这里正确注册的话，EAX 是 1，如果是 0，下面一跳就完了
    00C61400    74 6C          JE SHORT 00C6146E    ; 这里不要改，就算改成 NOP 或改跳转，说你成功注册，但确定后主程序还是会退出
    00C61402    8D55 D4        LEAEDX,DWORDPTRSS:[EBP-2C]
    00C61405    52             PUSHEDX
    00C61406    8D4D B0        LEAECX,DWORDPTRSS:[EBP-50]
    00C61409    E8 02040000    CALL 00C61810
    00C6140E    8D45 D0        LEAEAX,DWORDPTRSS:[EBP-30]
    00C61411    50             PUSHEAX
    00C61412    8D4D AC        LEAECX,DWORDPTRSS:[EBP-54]
    00C61415    E8 F6030000    CALL 00C61810
    00C6141A    8D4D D8        LEAECX,DWORDPTRSS:[EBP-28]
    00C6141D    51             PUSHECX
    00C6141E    8D4D A8        LEAECX,DWORDPTRSS:[EBP-58]
    00C61421    E8 EA030000    CALL 00C61810
    00C61426    8D55 B0        LEAEDX,DWORDPTRSS:[EBP-50]
    00C61429    52             PUSHEDX
    00C6142A    8D45 AC        LEAEAX,DWORDPTRSS:[EBP-54]
    00C6142D    50             PUSHEAX
    00C6142E    8D4D A8        LEAECX,DWORDPTRSS:[EBP-58]
    00C61431    51             PUSHECX
    00C61432    E8 36100000    CALL 00C6246D
    00C61437    83C4 0C        ADDESP,0C
    00C6143A    25 FF000000    ANDEAX,0FF
    00C6143F    85C0           TESTEAX,EAX
    00C61441    74 29          JE SHORT 00C6146C
    00C61443    6A 40          PUSH 40
    00C61445    68 2070C700    PUSH 00C77020    ; ASCII "Registration success"
    00C6144A    68 3870C700    PUSH 00C77038    ; ASCII "Thank you for registering!"
    00C6144F    8B55 08        MOVEDX,DWORDPTRSS:[EBP+8]
    00C61452    52             PUSHEDX
    00C61453    FF15 0061C600 CALLDWORDPTRDS:[C66100]   ; user32.MessageBoxA
    00C61459    C605 E672C700 0>MOVBYTEPTRDS:[C772E6],1
    00C61460    6A 00          PUSH 0
    00C61462    8B45 08        MOVEAX,DWORDPTRSS:[EBP+8]
    00C61465    50             PUSHEAX
    00C61466    FF15 FC60C600 CALLDWORDPTRDS:[C660FC]   ; user32.EndDialog
    00C6146C    EB 16          JMP SHORT 00C61484
    00C6146E    6A 10          PUSH 10
    00C61470    68 5470C700    PUSH 00C77054    ; ASCII "Registration error"
    00C61475    68 6870C700    PUSH 00C77068    ; ASCII "Incorrect registration code"
    00C6147A    8B4D 08        MOVECX,DWORDPTRSS:[EBP+8]
    00C6147D    51             PUSHECX
    00C6147E    FF15 0061C600 CALLDWORDPTRDS:[C66100]   ; user32.MessageBoxA
    00C61484    EB 08          JMP SHORT 00C6148E   ; 跟随到这
    00C61486    6A 40          PUSH 40
    00C61488    FF15 0461C600 CALLDWORDPTRDS:[C66104]   ; user32.MessageBeep
    ```

跟进 `00C613F1` 处的那个注册算法 `CALL`。我这里的目的主要是解出剩下的两个文件，不关注算法，只要找到能让程序正确启动的方法。找个以前的注册码 `4CFTRYAB-D4REQS8C-44DFF988-AB9EY8BR` 作为试验用的注册码：

```
00C61EAC    55             PUSHEBP; 注册算法函数入口
00C61EAD    8BEC           MOVEBP,ESP
00C61EAF    8B45 10        MOVEAX,DWORDPTRSS:[EBP+10]
00C61EB2    50             PUSHEAX
00C61EB3    8B4D 0C        MOVECX,DWORDPTRSS:[EBP+C]
00C61EB6    51             PUSHECX
00C61EB7    8B55 08        MOVEDX,DWORDPTRSS:[EBP+8]
00C61EBA    52             PUSHEDX
00C61EBB    E8 49000000    CALL 00C61F09
00C61EC0    83C4 0C        ADDESP,0C
00C61EC3    25 FF000000    ANDEAX,0FF
00C61EC8    85C0           TESTEAX,EAX
00C61ECA    74 39          JE SHORT 00C61F05

省略若干代码...

00C620C1    83C4 10        ADDESP,10
00C620C4    8945 BC        MOVDWORDPTRSS:[EBP-44],EAX
00C620C7    8B0D 4070C600  MOVECX,DWORDPTRDS:[C67040]
00C620CD    8D148D A43BC700 LEAEDX,DWORDPTRDS:[ECX*4+C73BA4]
00C620D4    3955 BC        CMPDWORDPTRSS:[EBP-44],EDX
00C620D7    0F84 8B000000  JE 00C62168
00C620DD    6A 10          PUSH 10
00C620DF    8B45 BC        MOVEAX,DWORDPTRSS:[EBP-44]
00C620E2    8B08           MOVECX,DWORDPTRDS:[EAX]
00C620E4    51             PUSHECX; 真正的注册码仅转换后的数据
00C620E5    8D55 AC        LEAEDX,DWORDPTRSS:[EBP-54]
00C620E8    52             PUSHEDX; 我们输入的注册码经转换后的数据。到这在寄存器窗口中把 ECX 的数据在数据窗口中跟随，把前面 10h 的数据二进制复制，粘贴到 EDX 所在地址内存中替换原数据，这样下面的比较就一样了
00C620E9    E8 62120000    CALL 00C63350    ; JMP 到 msvcrt.memcmp
00C620EE    83C4 0C        ADDESP,0C
00C620F1    85C0           TESTEAX,EAX
00C620F3    75 73          JNZ SHORT 00C62168
00C620F5    33C0           XOREAX,EAX
00C620F7    85C0           TESTEAX,EAX
00C620F9    ^ 75 FA        JNZ SHORT 00C620F5

省略若干代码...

00C621BF    6A 10          PUSH 10
00C621C1    8D8D 4CFEFFFF  LEAECX,DWORDPTRSS:[EBP-1B4]
00C621C7    51             PUSHECX
00C621C8    68 F072C700    PUSH 00C772F0    ; 这里还有个内存比较。不过如果前面改了的话，这里就不用改了
00C621CD    E8 7E110000    CALL 00C63350    ; JMP 到 msvcrt.memcmp
00C621D2    83C4 0C        ADDESP,0C
00C621D5    F7D8           NEGEAX
00C621D7    1BC0           SBBEAX,EAX
00C621D9    40             INCEAX
```

经过上面的修改后程序就能正常运行了，也显示是注册版。现在就可以按以前那种通过设断 `BP GetSystemTimeAsFileTime` 来找到运行时文件的方法，然后找文件来打包，解出前面因为有注册框，脚本没跑出来的两个文件。这两个文件解出来后原程序所有的东西我们都解出来了。解出来的主程序也没啥暗桩，现在针对脱壳版本，你要写注册机还是爆破，那就由你玩了。



## 破解：

只要在`RegisterPro.dll`中把前面指出的那个内存比较的地方改一下就行了：

```text
00CE20DD  .  6A 10              PUSH 10                                     ; /n = 10 (16.)
00CE20DF  .  8B45 BC           MOV  EAX,DWORD PTR SS:[EBP-44]             ; |
00CE20E2  .  8B08              MOV  ECX,DWORD PTR DS:[EAX]               ; |把后面的一句提到这里来，改成 LEA EDX,DWORD PTR SS:[EBP-54]
00CE20E4  .  51                PUSH ECX; |这里改成 MOV ECX,EDX
00CE20E5  .  8D55 AC           LEA  EDX,DWORD PTR SS:[EBP-54]             ; |这里改成 PUSH ECX
00CE20E8  .  52                PUSH EDX; |s1
00CE20E9  .  E8 62120000      CALL <JMP.&MSVCRT.memcmp>                 ; \memcmp
```

经过这样改后，可以随意输入注册码注册。

附件是脚本及`MoleBox9x.v2.3682`脱壳破解后的文件。

---

### 【版权声明】

本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢！

### 上传的附件

| 文件描述 | 文件名 |
| :--- | :--- |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | `MoleBox 2.x Unpacker.rar` |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | `MoleBox9x.v2.3682.Cracked.rar` |
