
# Krypton 0.5

> 原文：[`www.pediy.com/kssd/pediy06/pediy6387.htm`](https://www.pediy.com/kssd/pediy06/pediy6387.htm)

用 `Ollydbg` 快速手脱 Krypton 0.5 加壳程序——Krypton 主程序 等   

下载页面： `http://secrecy.ayinfo.ha.cn/soft/51.htm`   

软件大小： 821K   

软件语言： 英文   

软件类别： 国外软件 / 免费版 / 加壳软件   

运行环境： `Win9x/NT/2000/XP/`   

软件更新： 2003-5-22 17:37:14   

软件添加： 洋白菜   

下载次数： 588   

软件评级： *****

【软件简介】：很不错的一款免费加密保护工具，他的加密强度也不错。

【作者声明】：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`Ollydbg1.09`、`PEiD`、`LordPE`、`ImportREC V1.6`

—————————————————————————————————

【脱壳过程】：

Krypton 0.3 和 0.4 都有大侠写过脱壳教程，却没有看见写 0.5 的。 飞叶流枫 大侠写过脱壳机 `Krykiller0.5`，佩服！偶等脱壳菜鸟还是希望有手脱笔记呀。偶失败了 N 次终于脱了几个 Krypton 0.5 加壳的实验程序，笔记共享。

调试前先设置一下 `Ollydbg`。打开：`Ollydbg`——>选项——>调试设置——>异常

把“忽略在 `KERNEL32` 中的内存访问异常”、“`INT3` 中断”、“单步中断” 这 3 个选项选上。

OK，让偶开始“战斗”吧！:-)

—————————————————————————————————

### 一、Krypton 0.5 主程序：`Krypton.exe`

```
004A4000     54                   push esp
                             ==== > 进入 OD 后断在这！

004A4001     E8 00000000          call Krypton.004A4006
```

————————————————————————

`F9` 运行，程序会在异常处中断。

```
003A03E1     8918                 mov dword ptr ds:[eax],ebx   ; Krypton.00450600
                             ==== > 第 1 次异常
003A03E3     EB 40                jmp short 003A0425
```

`Shift+F9` 通过异常，2 次程序运行。好了，Try Again，按 1 次 `Shift+F9`，停下来。

弹出数十个对话框，一一确定之。

————————————————————————

```
003ACBC8     DF59 9C              fistp word ptr ds:[ecx-64]
                             ==== > 第 2 次异常在这儿！         :-)
                             ==== > 看看堆栈区的第二条地址是：003ACCFB（SE 句柄） 设断
003ACBCB     83C1 E7              add ecx,-19
003ACBCE     9D                   popfd
003ACBCF     FFE1                 jmp ecx
```

————————————————————————

在 `003ACCFB` 处设断后，`Shift+F9` 运行，程序会中断在 `003ACCFB`

```
003ACCFB     64:67:A1 0000        mov eax,dword ptr fs:[0]
                             ==== > 堆栈区的第二条地址         下断点！
```

…… …… 省 略 ……  …… 用 `F7` 走 …… ……

```
003ACD98   ^ 72 F0                jb short 003ACD8A
003ACD8A     64:67:8926 0000      mov dword ptr fs:[0],esp
                             ==== > 这里注意了！马上又要有异常了！
                             ==== > 在这里看看堆栈的第 2 个地址是：003AE440  下断！
003ACD90     EB 3B                jmp short 003ACDCD
```

…… …… 省 略 ……  …… 用 `F7` 走 …… ……

```
003ACE0C    7B F0                jpo short 003ACDFE
003ACDFE     CC                   int3
                             ==== > 注意：INT3 引起异常！
                             ==== > F9，程序断在 003AE440
```

————————————————————————

```
003AE440     64:67:A1 0000        mov eax,dword ptr fs:[0]
                             ==== > 在 003ACD8A 处看到的 SE 句柄
003AE445     8B20                 mov esp,dword ptr ds:[eax]
003AE447     64:67:8F06 0000      pop dword ptr fs:[0]
003AE44D     E8 00000000          call 003AE452
003AE452     5D                   pop ebp
003AE453     81ED 9D5D4100        sub ebp,415D9D
003AE459     8CC9                 mov cx,cs
003AE45B     EB 42                jmp short 003AE49F
003AE49F     51                   push ecx
003AE4A0     E8 F0FFFFFF          call 003AE495
003AE495     59                   pop ecx            ; 003AE4A5
003AE496     9C                   pushfd
003AE497     83C1 E7              add ecx,-19
003AE49A     9D                   popfd
003AE49B     FFE1                 jmp ecx
                             ==== > 跳至 003AE48C
003AE48C     59                   pop ecx
003AE48D     7A F4                jpe short 003AE483
003AE48F     EB 01                jmp short 003AE492
003AE492    7B EF                jpo short 003AE483
003AE483     32C9                 xor cl,cl
003AE485     EB 4A                jmp short 003AE4D1
003AE4D1     51                   push ecx
003AE4D2     E8 F0FFFFFF          call 003AE4C7
003AE4C7     59                   pop ecx            ; 003AE4D7
003AE4C8     9C                   pushfd
003AE4C9     83C1 E7              add ecx,-19
003AE4CC     9D                   popfd
003AE4CD     FFE1                 jmp ecx
                             ==== > 跳至 003AE4BE
003AE4BE     59                   pop ecx
003AE4BF   ^ 74 ED                je short 003AE4AE
003AE4AE    /E3 02                jecxz short 003AE4B2
003AE4B0    |EB 05               jmp short 003AE4B7
003AE4B2    E9 5B050000          jmp 003AEA12
003AEA12    /EB 45               jmp short 003AEA59
003AEA59   ^EB F4                jmp short 003AEA4F
003AEA4F     51                   push ecx
003AEA50   ^ EB F4                jmp short 003AEA46
003AEA46     59                   pop ecx
003AEA47   ^ 79 EF                jns short 003AEA38
003AEA38     80BD DCC44100 FF     cmp byte ptr ss:[ebp+41C4DC],0FF
003AEA3F     EB 42                jmp short 003AEA83
003AEA83   ^EB F4                jmp short 003AEA79
003AEA79     51                   push ecx
003AEA7A   ^ EB F4                jmp short 003AEA70
003AEA70     59                   pop ecx
003AEA71     7A F0                jpe short 003AEA63
003AEA63    /0F84 D0000000        je 003AEB39
                             ==== > 注意：如果是用 Krypton 0.5 加壳的程序此处不跳！则下面不远处就会跳到 OEP。但是 Krypton 主程序却从此处跳下去，在下面进行了 4 个大循环，每个循环是 11265 次！可惜偶的键盘呀。汗水的收获是点滴的经验。CTR+F 在“整个区段”查找命令：xor ebp,112233   轻松跳出几万圈循环。
```

```
003B38DB     81F5 33221100        xor ebp,112233
                             ==== > 下断！F9 运行程序被断下！
003B38E1     EB 43                jmp short 003B3926
```

…… …… 省 略 …… …… 用 `F7` 走 …… ……

```
003B3943   ⁷⁵ F0                jnz short 003B3935
003B3935     50                   push eax           ;Krypton.00401000
                             ==== > EAX=00401000    这就是 OEP 值       :-)
003B3936     EB 3B                jmp short 003B3973
003B3973     51                   push ecx   
003B3974     E8 F0FFFFFF          call 003B3969
003B3969     59                   pop ecx   
003B396A     9C                   pushfd
003B396B     83C1 E7              add ecx,-19
003B396E     9D                   popfd
003B396F     FFE1                 jmp ecx
003B3960     59                   pop ecx   
003B3961     72 F5                jb short 003B3958
003B3963     EB 01                jmp short 003B3966
003B3966     73 F0                jnb short 003B3958
003B3958     C3                   retn
                             ==== > 飞向 光明之巅！ :-) 终于跟到 OEP 了！
```

————————————————————————

```
00401000     6A 00                push 0
                             ==== > 在这儿用 LordPE 完全 DUMP 这个进程  :-)
00401002     E8 962A0000          call Krypton.00403A9D
00401007     A3 B5634000          mov dword ptr ds:[4063B5],eax
0040100C     68 2C684000          push Krypton.0040682C 
00401011     E8 7B2A0000          call Krypton.00403A91
00401016     E8 882A0000          call Krypton.00403AA3
```

————————————————————————

晕，无法用 `Import REConstructor v1.6` 修复输入表！偶是没办法啦。但是脱壳后的程序可以运行！

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

兄弟们或许看急了，看到现在满脑子都是跳转，比蜗牛的速度也快不了多少呀？呵呵，别急呀，偶调试的时候记录的代码是上面的 N 倍！:-(  现在偶就来演示 快速手脱 Krypton 0.5 加壳的程序 :-) 。当然，最快的脱壳方式是用 飞叶流枫 大侠的 `Krykiller0.5` 脱壳机啦。偶用 `Krykiller0.5` 试脱几个加壳的程序正常，但是脱 Krypton 0.5 主程序后发现脱壳后的主程序功能不正常了。不清楚是偶操作的原因还是怎么了。 下面开始啦！ :-)

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

### 二、`WinXP` 的 `spider.exe`  蜘蛛纸牌  去你的 `WINDOWSsystem32` 目录下找

偶实在是找不到能够成功加 Krypton 壳的常用程序了，只好找几个不常用的。只选了“K-Protect on API”加壳，否则无法运行。Krypton 挺挑剔，一般 `VC++`和汇编写的程序比较容易加壳成功。

————————————————————————

```
010C9000     54                   push esp
                             ==== > 进入 OD 后断在这！
010C9001     E8 00000000          call spider.010C9006
```

`F9` 运行，程序会在异常处中断。

```
003003E1     8918                 mov dword ptr ds:[eax],ebx
                             ==== > 第 1 次异常
```



`Shift+F9` 通过异常，2 次程序运行。好了，`Try Again`，按 1 次 `Shift+F9`，停下来。

弹出数十个对话框，一一确定之。

```
0030CBC8      DF59 9C           fistp word ptr ds:[ecx-64]

                             ==== > 第 2 次异常在这儿！         :-)
0030CBCB      83C1 E7          add ecx,-19
0030CBCE      9D              popfd
0030CBCF      FFE1           jmp ecx
```

————————————————————————

`OK`，现在不需要跳来跳去的跟踪了！`CTR+F` 在“整个区段”查找命令：`jmp edx`

```
0030EB08      6240 C5         bound eax,qword ptr ds:[eax-3B]
0030EB0B      2BDF           sub ebx,edi
0030EB0D      F9            stc
0030EB0E      1C D3         sbb al,0D3
0030EB10      41            inc ecx
0030EB11      A8 6A         test al,6A
0030EB13   - FFE2          jmp edx    ; spider.01008726

                             ==== > 找到这里！Very Good！F2 此处下断！
```

`Shift+F9` 跳过第 2 次异常，砰——程序在 `0030EB13` 处断下！`F8` 过去看看吧 :-)

```
01008726      6A 60         push 60

                             ==== > 偶来到这里！这里就是 OEP 啦！:-)
                             ==== > 在这儿用 LordPE 完全 DUMP 这个进程
01008728      68 60160001   push spider.01001660
0100872D      E8 BE100000   call spider.010097F0
```

运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `00008726`, 点 `IT AutoSearch`，点“`Get Import`”，函数无效，右键选“`Krypton0.4-0.5` 插件”追踪，`OK`，全部修复！`FixDump`，正常运行！`600K` -> `1.37M` ！太大了，用 `LordPE` 打开区段把 `krypton` 和 `!_!!_` 删除，然后再用 `LordPE` 重建 `PE`，`1.37M`->`522K` 仅比原程序多了 `1K`！`COOL`

——————————————————————————————

### 三、`WinXP` 的 `regedt32.exe` 也是注册表编辑器 去你的 `WINDOWSsystem32` 目录下找

```
01046000      54            push esp

                             ==== > 进入 OD 后断在这！
01046001      E8 00000000   call regedt32.01046006
```

`F9` 运行，程序会在异常处中断。

```
002C03E1      8918          mov dword ptr ds:[eax],ebx   ; regedt32.01001200

                             ==== > 第 1 次异常
```

`Shift+F9` 通过异常，2 次程序运行。好了，`Try Again`，按 1 次 `Shift+F9`，停下来。

弹出数十个对话框，一一确定之。

```
002CCBC8      DF59 9C       fistp word ptr ds:[ecx-64]

                             ==== > 第 2 次异常在这儿！         :-)
```

`OK`，闪电寻找 `OEP`！`CTR+F` 在“整个区段”查找命令：`jmp edx`

```
002CEB08      6240 C5       bound eax,qword ptr ds:[eax-3B]
002CEB0B      2BDF         sub ebx,edi
002CEB0D      F9           stc
002CEB0E      1C D3        sbb al,0D3
002CEB10      41           inc ecx
002CEB11      A8 6A        test al,6A
002CEB13      FFE2         jmp edx    ; regedt32.01001088

                             ==== > 找到这里！Very Good！F2 此处下断！
```

`Shift+F9` 跳过第 2 次异常，砰——程序在 `002CEB13` 处断下！`F8` 过去看看吧 :-)

```
01001088      55           push ebp

                             ==== > 偶来到这里！这里就是 OEP 啦！:-)
                             ==== > 在这儿用 LordPE 完全 DUMP 这个进程
01001089      8BEC         mov ebp,esp
0100108B      83EC 44      sub esp,44
0100108E      56           push esi
0100108F      FF15 08100001 call dword ptr ds:[1001008]
```

修复输入表、优化 `PE` 文件 方法 同上！

—————————————————————————————————

### 四、`Win98` 的 `Notepad` 不好意思，偶的 `98` 下的记事本可以在 `XP` 下运行 :-)

```
00450000      54           push esp

                             ==== > 进入 OD 后断在这！
00450001      E8 00000000  call NOTEPAD.00450006
```

`F9` 运行，程序会在异常处中断。

```
004703E1      8918         mov dword ptr ds:[eax],ebx  ; Notepad.00407000

                             ==== > 第 1 个异常
```

`Shift+F9` 通过异常，2 次程序运行。好了，`Try Again`，按 1 次 `Shift+F9`，停下来。

```
0047CBC8      DF59 9C      fistp word ptr ds:[ecx-64]

                             ==== > 第 2 次异常在这儿！         :-)
```

`OK`，闪电寻找 `OEP`！`CTR+F` 在“整个区段”查找命令：`jmp edx`

```
0047EB08      6240 C5      bound eax,qword ptr ds:[eax-3B]
0047EB0B      2BDF        sub ebx,edi
0047EB0D      F9          stc
0047EB0E      1C D3       sbb al,0D3
0047EB10      41          inc ecx
0047EB11      A8 6A       test al,6A
0047EB13   - FFE2        jmp edx      ; Notepad.004010CC

                             ==== > 找到这里！Very Good！F2 此处下断！
```

`Shift+F9` 跳过第 2 次异常，砰——程序在 `0047EB13` 处断下！`F8` 过去看看吧 :-)

```
004010CC       55         db 55

                             ==== > 偶来到这里！这里就是 OEP 啦！:-)
                             ==== > 在这儿用 LordPE 完全 DUMP 这个进程
```

修复输入表、优化 `PE` 文件 方法 同上！

——————————————————————————————

【`Krypton 0.5` 加壳程序脱壳总结】：

1、按前面的方法设置 `Ollydbg` 的“忽略异常”的选项

2、在持续运行前 1 次的异常处（一般是第 2 次）搜索“`jmp edx`”命令！找到如下代码处：

```
0047EB08      6240 C5     bound eax,qword ptr ds:[eax-3B]
0047EB0B      2BDF       sub ebx,edi
0047EB0D      F9         stc
0047EB0E      1C D3      sbb al,0D3
0047EB10      41         inc ecx
0047EB11      A8 6A      test al,6A
0047EB13   - FFE2      jmp edx   //此处设断点！
```

3、`Shift+F9` 跳过异常，程序在 `jmp edx` 处断下

4、`F8` 过去，就到了 `OEP` 的地方！拿出你的 `LordPE` 脱壳吧！

5、修复输入表方法见上！ 快不快？ 呵呵 :-)

注：这个方法适用于用 `Krypton 0.5` 加壳的程序，`Krypton` 主程序的脱壳方法见上。

错误之处欢迎大家指正！偶是脱壳菜鸟，虚心学习。

——————————————————————————————

```
        ,      \_/  
      /| \_.-~/        \_     ,        青春都一饷
     ( /~   /          ~-.|   
     \`\\  \_/             ~ )          忍把浮名
  _-~~~-.)  )\_\_/;;,.      _  //’
  /'\_,   --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂
 \`~ \_( ,\_..-- (     ,;'' /    ~--   /.\_\`  
  /~~//'   /' \`~        ) /--..\_, )\_  \`~
  "  \`~"  "     \`"      /~'\`    \`\\~~  

               "     "   "~'  ""
          Cracked By 巢水工作坊——fly [OCN][FCG]
                    2003-10-04  02:40
```

* * *

**标 题**: 讨论

*   **作 者**：`CoTaYoMa`
    
    **时 间**: `2003` 年 `10` 月 `04` 日 `11:39`
    
    **详细信息**:
    
    > `<tr>` | `fxyang` 写道: | `tr>` `<tr>` | 我在 `OD` 中在 `OEP` 中直接 `dump` 下来就可以运行，不需要修复，加壳的 `regedt32.exe` 也可以用同样的方法。
    >
    > 另外 `Krypton 0.5` 的主程序我也用同样的方法脱出来了。 | `tr>`
    >
    > 
    > 
    > 嘿嘿,这样用 `OD` 直接 `DUMP` 出来 `Krypton 0.5` 的主程序,可以运行?不会吧........主程序可是有 `K-LOCK` 的啊.在它的说明书这样写道:
    >
    > ; -------------------------------------------------------------------------
    >
    > `Programmer code for manual K-Execution :`
    >
    > ; -------------------------------------------------------------------------
    >
    > `db 0ebh,0eh`
    >
    > `db 'KDES'`
    >
    > `db 00,00,00,00,00`
    >
    > `db 00,00,00,00,00`
    >
    > ;这里放上你的代码
    >
    > `db 0ebh,0eh`
    >
    > `db 'KDEE'`
    >
    > `db 00,00,00,00,00`
    >
    > `db 00,00,00,00,00`
    >
    > The `KDES` code indicates the beginning of the `K-Execution`, it means that
    >
    > this part once executed will be deleted.
    >
    > The `KDEE` code must be placed at the end of the routine which shall be
    >
    > included into the `K-Execution`.
    >
    > ; -------------------------------------------------------------------------
    >
    > `db 0ebh,0eh`
    >
    > `db 'KEES'`
    >
    > `db 00,00,00,00,00`
    >
    > `db 00,00,00,00,00`
    >
    > ;这里放上你的代码
    >
    > `db 0ebh,0eh`
    >
    > `db 'KEEE'`
    >
    > `db 00,00,00,00,00`
    >
    > `db 00,00,00,00,00`
    >
    > The `KEES` code indicates the beginning of the `K-Execution`, it means that
    >
    > this part of the code once executed will NOT be deleted (it can be used
    >
    > more than once then).
    >
    > 当你在入口点 `DUMP` 的时候,程序代码还没有完全解压缩完啊!我想这样 `DUMP` 出来的程序一定不能运行吧.就算是引入表修复了也是不能运行的.当程序运行到上面的 `K-EXE` 代码的时候,它会跳到壳中去再次解压缩,所以我说要如果不是自己写一个程序来解压缩代码的话,是不可能脱壳的.
    >
    > 
    > 
    > `4dd466d0e1e0cc123afbe504da2462ca`
