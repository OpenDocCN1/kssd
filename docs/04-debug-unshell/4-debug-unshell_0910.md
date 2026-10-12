
# PESpin V0.70

> 原文：[`www.pediy.com/kssd/pediy06/pediy7004.htm`](https://www.pediy.com/kssd/pediy06/pediy7004.htm)

## 基本信息

- **标题**：PESpin V0.70 简单脱壳方案
- **作者**：weiyi75[Dfcg]
- **邮箱**：weiyi75@sohu.com
- **主页**：Dfcg 官方大本营
- **下载**：点击下载
- **加壳方式**：`PESpin` `0.70` -> `cyberbob`
- **声明**：我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 脱壳内容

这个版本同 `0.3` 版本的变化主要是 IAT，加密方式略微有些变化，加入了 Replace Code for IAT。如果你喜欢追根问底，那么请看 `loveboom` 的脱文，如果喜欢快，准，狠，那么往下看。

## 详细内容

首先希望读者已经阅读过二哥的 `PESpin` `V0.3` 怒而脱之并看懂调试通过，如果 `0.3` 都不会就搞 `0.7`，真服了。用 `PESpin` 加密记事本（全部 Anti 选项），`OD` 选项不忽略特权指令，其余全部忽略。

这个壳是在最后一次特权指令 SEH 异常后就开始处理 IAT。如果问为什么，很简单，到那里后 `dd 4062e4`（IAT 的开始地址）往下看 IAT 还没有写入。然后到 Stolen code。

使用 `OD` 载入全部 Anti 的记事本：

```
0040C087 N> /EB 01             jmp short NOTEPAD.0040C08A //开始就有小花，用花指令插件清除 6 个
0040C089 |68 60E80000          push 0E860
0040C08E   0000               add byte ptr ds:[eax],al
0040C090   8B1C24             mov ebx,dword ptr ss:[esp]
0040C093   83C3 12            add ebx,12
0040C096   812B E8B10600      sub dword ptr ds:[ebx],6B1E8
0040C09C   FE4B FD            dec byte ptr ds:[ebx-3]
0040C09F   822C24 83          sub byte ptr ss:[esp],-7D
0040C0A3   D5 46              aad 46
0040C0A5   000B               add byte ptr ds:[ebx],cl
0040C0A7   E4 74              in al,74
0040C0A9   9E                 sahf
0040C0AA   75 01              jnz short NOTEPAD.0040C0AD
..............................................................

0040C087 N>  90               nop
0040C088     90               nop
0040C089     90               nop
0040C08A     60               pushad  //这里 hr esp，方便找 Stolen code，后面的版本无效，再说。
0040C08B     E8 00000000      call NOTEPAD.0040C090
0040C090     8B1C24           mov ebx,dword ptr ss:[esp]
0040C093     83C3 12          add ebx,12
0040C096     812B E8B10600    sub dword ptr ds:[ebx],6B1E8
0040C09C     FE4B FD          dec byte ptr ds:[ebx-3]
0040C09F     822C24 83        sub byte ptr ss:[esp],-7D
0040C0A3     D5 46            aad 46
0040C0A5     000B             add byte ptr ds:[ebx],cl
0040C0A7     E4 74            in al,74
0040C0A9     9E               sahf
0040C0AA     90               nop
0040C0AB     90               nop
0040C0AC     90               nop
0040C0AD     8173 04 D77AF72F  xor dword ptr ds:[ebx+4],2FF77AD7
0040C0B4     8173 19 770043B7  xor dword ptr ds:[ebx+19],B7430077
..............................................................
```

执行 `Shift+F9` 4 次（其中硬件中断一次）运行到最后一次特权指令 SEH 处：

```
00400181     FFFF             ???                          ; 未知命令
00400183     FFFF             ???                          ; 未知命令
00400185     FFFF             ???                          ; 未知命令
00400187     FFFF             ???                          ; 未知命令
00400189     FFFF             ???                          ; 未知命令
0040018B     FFFF             ???                          ; 未知命令
0040018D     FFFF             ???                          ; 未知命令
0040018F     FFFF             ???                          ; 未知命令
00400191     FFFF             ???                          ; 未知命令
00400193     FFFF             ???                          ; 未知命令
00400195     FFFF             ???                          ; 未知命令
00400197     FFFF             ???                          ; 未知命令
.......................
```

在命令行输入：

```
bp LoadLibraryA
```

按 `Shift+F9` 中断：

```
77E78023 K>  55               push ebp
77E78024     8BEC             mov ebp,esp
77E78026     53               push ebx
77E78027     56               push esi
77E78028     57               push edi
77E78029     68 5480E777      push KERNEL32.77E78054       ; ASCII "twain_32.dll"
77E7802E     FF75 08          push dword ptr ss:[ebp+8]
77E78031     FF15 0413E677    call dword ptr ds:[<&NTDLL._strcmpi>; ntdll._stricmp
77E78037     59               pop ecx
77E78038     85C0             test eax,eax
77E7803A     59               pop ecx
77E7803B     0F84 C7C30100    je KERNEL32.77E94408
................................................................
```

清除断点，`Alt+F9` 返回：

```
0040CE0E     85C0             test eax,eax                   ; SHELL32.#465
0040CE10     0F84 2F070000    je NOTEPAD.0040D545
0040CE16     E8 01000000      call NOTEPAD.0040CE1C
0040CE1B     FF59 50          call far fword ptr ds:[ecx+50]
0040CE1E     51               push ecx
0040CE1F     55               push ebp
0040CE20     810424 0C2B4000  add dword ptr ss:[esp],NOTEPAD.0040>
0040CE27     814424 04 22000000 add dword ptr ss:[esp+4],22
0040CE2F     C3               retn
```

### 标签 1

整个区段搜索写 IAT 指令：

```
mov dword ptr ds:[edx],eax
```

找到这里：

```
0040C98D     8902             mov dword ptr ds:[edx],eax    ; wiat
0040C98F     EB 25            jmp short NOTEPAD.0040C9B6
0040C991     EB 04            jmp short NOTEPAD.0040C997
0040C993     FFEB            jmp far ebx                   ; 非法使用寄存器
```

加密 Call 特征指令：

```
jmp dword ptr ss:[esp-4]
```

依次找到，按 `Ctrl+L` 找下一个：

```
0040CCCC     FF6424 FC        jmp dword ptr ss:[esp-4]
0040CDE0     FF6424 FC        jmp dword ptr ss:[esp-4]
0040CE7B     FF6424 FC        jmp dword ptr ss:[esp-4]
0040CF0C     FF6424 FC        jmp dword ptr ss:[esp-4]
0040CF42     FF6424 FC        jmp dword ptr ss:[esp-4] //这个就是，底下有个非法使用寄存器的花指令命令
0040CF46     FFE8             jmp far eax                 ; 非法使用寄存器
0040CF42    /FF6424 FC        jmp dword ptr ss:[esp-4]     ; NOTEPAD.0040CF47
```

下 `F2` 断点，`F9` 中断清除断点。

这里就可以补丁了。

修改为：

```
0040CF42   ^\E9 46FAFFFF      jmp NOTEPAD.0040C98D
0040CF47     90               nop
```

跟踪一下路线：

```
0040CF42   ^\E9 46FAFFFF      jmp NOTEPAD.0040C98D
0040CF47     90               nop
0040C98D     8902             mov dword ptr ds:[edx],eax    ; wiat
0040C98F     EB 25            jmp short NOTEPAD.0040C9B6
0040C9B6     59               pop ecx                       ; NOTEPAD.0040DB1D
0040C9B7     5F               pop edi
0040C9B8     C3               retn //这里由于堆栈平衡等关系 Over
```

基本思路已经确定，再来看看如何尽量让程序“平衡”。

重启 `OD`，重复上面的过程到标签 1。

整个区段搜索写 IAT 指令：

```
mov dword ptr ds:[edx],eax
```

找到这里：

```
0040C98D     8902             mov dword ptr ds:[edx],eax //F2 下断
0040C98F     EB 25            jmp short NOTEPAD.0040C9B6
0040C991     EB 04            jmp short NOTEPAD.0040C997
0040C993     FFEB            jmp far ebx                   ; 非法使用寄存器
```

`Shift+F9` 中断后看到：

`EAX=76AFA5CD` COMDLG32 库正好是不加密的路线，尾行^-^

```
0040C98F    /EB 25           jmp short NOTEPAD.0040C9B6
0040C9B6     59               pop ecx                       ; NOTEPAD.0040CE3E
0040C9B7     5F               pop edi
0040C9B8     C3               retn //返回到 40CF78，这就好办了。
```

```
0040C9B6     59               pop ecx                       ; NOTEPAD.0040CE3E
0040C9B7     5F               pop edi
0040C9B8     C3               retn //如果这里修改为
0040C9B8    /E9 BB050000      jmp NOTEPAD.0040CF78 //运行立即崩溃
0040C9BD    |90               nop
```

```
0040C9B6     59               pop ecx  //如果选这里修改
0040C9B6    /E9 BD050000      jmp NOTEPAD.0040CF78 //IAT 就全部正确写入。
0040C9BB    |90               nop
0040C9B7     5F               pop edi
0040C9B8     C3               retn //返回到 40CF78，这就好办了。
```

补丁上面两个地址，`F9` 运行到这里。

去花：

```
0040D088     55               push ebp  //到这里就用 Loadpe 纠正镜像大小脱壳。
0040D089     90               nop
```



# 脱壳过程记录与总结

## 1. 代码段分析

### 1.1 初始代码块
```
0040D08A     90                    nop  
0040D08B     90                    nop  
0040D08C     8BEC                 mov ebp,esp  
0040D08E     90                    nop  
0040D08F     90                    nop  
0040D090     90                    nop  
0040D091     83EC 44              sub esp,44  
0040D094     90                    nop  
0040D095     90                    nop  
0040D096     90                    nop  
0040D097     56                    push esi  
0040D098     90                    nop  
0040D099     90                    nop  
0040D09A     90                    nop  
0040D09B     FF15 19E34000        call dword ptr ds:[40E319]          ; NOTEPAD.004063E4  
0040D09B     FF15 19E34000        call dword ptr ds:[40E319]   //这里实际使访问壳代码,具体往后看。  

[40E319]  0085048A 确定区域壳位置  

 地址=00850000  

 大小=00002000 (8192.)  
```

### 1.2 注意这里
```
004063E4     CC                    int3      Replace Code for IAT，这就是 CC?不是 Arm 才有吗?但是解码错误，待会再处理。  
```

### 1.3 后续代码片段
```
004063E5   ^ 71 E6               jno short NOTEPAD.004063CD  
004063E7   ^ 77 F4               ja short NOTEPAD.004063DD  
004063E9     B7 E6               mov bh,0E6  
004063EB   ^ 77 97               ja short NOTEPAD.00406384  
004063ED     9B                    wait  
004063EE     E7 77                out 77,eax  
004063F0     A3 CCE67700          mov dword ptr ds:[77E6CC],eax  
004063F5     0000                 add byte ptr ds:[eax],al  
004063F7     00D6                 add dh,dl  
004063F9     AA                    stos byte ptr es:[edi]  
004063FA     5F                    pop edi  
004063FB     77 08                ja short NOTEPAD.00406405  
```

### 1.4 更多代码分析
```
0040D0A1     90                    nop  
0040D0A2     90                    nop  
0040D0A3     90                    nop  
0040D0A4     8BF0                 mov esi,eax  
0040D0A6     90                    nop  
0040D0A7     90                    nop  
0040D0A8     90                    nop  
0040D0A9     8A00                 mov al,byte ptr ds:[eax]  
0040D0AB     90                    nop  
0040D0AC     90                    nop  
0040D0AD     90                    nop  
0040D0AE     3C 22                cmp al,22  
0040D0B0     90                    nop  
0040D0B1     90                    nop  
0040D0B2     90                    nop  
0040D0B3   - E9 2740FFFF          jmp NOTEPAD.004010DF  
```

## 2. 使用工具修复 Import Table (IAT)

### 2.1 运行 ImportREC
运行`ImportREC`，选择这个进程。把`OEP`改为`D088`,`RVA` `62e0`,`Size` `248`。

虽然这是已知的，但也可以以理服人。

随便找个
```
0040D09B     FF15 19E34000        call dword ptr ds:[40E319]          ; NOTEPAD.004063E4  
```

命令行`dd 4063e4`，向上找`IAT`的起点，向下找结束位置，或者找到`IAT`起点，`Size`填`1000`的懒人用大面积剪刀法，多余的垃圾全部剪掉。

点获取输入表，指针全部有效，由于`CC`问题先不能用现在的文件脱壳，得脱一个"干净"的文件出来。

### 2.2 再次开启 OllyDbg (OD)
再开一个`OD`
```
0040D088     55                    push ebp  //直接用 Esp 定律在这里脱壳。  
0040D089     EB 01                jmp short NOTEPAD.0040D08C  
0040D08B     B4 8B                mov ah,8B  
0040D08D     EC                    in al,dx  
0040D08E     EB 01                jmp short NOTEPAD.0040D091  
0040D090   ^ E2 83                loopd short NOTEPAD.0040D015  
0040D092     EC                    in al,dx  
0040D093     44                    inc esp  
0040D094     EB 01                jmp short NOTEPAD.0040D097  
0040D096     F756 EB              not dword ptr ds:[esi-15]  
0040D099     011CFF               add dword ptr ds:[edi+edi*8],ebx  
0040D09C     15 19E34000          adc eax,NOTEPAD.0040E319  
0040D0A1     EB 01                jmp short NOTEPAD.0040D0A4  
0040D0A3     C6                    ???                                ; 未知命令  
0040D0A4     8BF0                 mov esi,eax  
0040D0A6     EB 01                jmp short NOTEPAD.0040D0A9  
```

现在`IR`可以修复这个文件了。

### 2.3 区域脱壳，解决 Replace Code for IAT
然后是
```
0040D09B     FF15 19E34000        call dword ptr ds:[40E319]          ; NOTEPAD.004063E4  

0040D09B     FF15 19E34000        call dword ptr ds:[40E319]   //这里实际是访问壳代码  

[40E319]  0085048A 确定区域壳位置  

 地址=00850000  

 大小=00002000 (8192.)  
```

单独运行加密程序，区域脱壳

`地址` `00850000`

`大小` `00002000`

`PE`编辑器编辑脱壳修复文件，选择从磁盘载入区域脱壳文件，然后编辑刚加入的区段，虚拟地址修正为`00450000`，重建`PE`选项只保留验证`PE`即可，然后正常运行。

## 3. 脱壳总结

修复实际是以下几个步骤：

1. `Esp`定律直接到`Stolen code`脱壳，预先区域脱壳`Replace Code for IAT`
2. 用`PATCH IAT`的方式修复脱壳文件
3. 组合修复文件和区域脱壳
4. 为了大家看明白原理，文章是类似`Arm`乱序的方式写的，有些乱。

--------------------------------------------------------------------------------

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢

`[PESpin 0.71 -> cyberbob]`  

`signature = EB 01 68 60 E8 00 00 00 00 8B 1C 24 83 C3 12 81 2B E8 B1 06 00 FE 4B FD 82 2C 24 83 D5 46 00 0B E4 74 9E`

`ep_only = true`
`4dd466d0e1e0cc123afbe504da2462ca`
