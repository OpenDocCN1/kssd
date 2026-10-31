
# [原创]关于E语言写的使用硬盘序列号加密的程序通用切入点

- 【文章标题】关于E语言写的使用硬盘序列号加密的程序通用切入点
- 【文章作者】hcbajiao(蓝雨[PCG])
- 【作者主页】[f9eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0W2&6j5$3&6Q4x3X3g2U0L8$3@1`.](https://bbs.kanxue.com/elink@252K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0W2&6j5$3&6Q4x3X3g2U0L8$3@1`.)
- 【作者邮箱】bj\_008@163.com
- 【所属组织】[PCG], <http://poje.kmip.net:81QQ群成员>
- 【软件名称】冒险岛加强外挂
- 【使用工具】`OD`，`PEID`
- 【软件限制】注册码
- 【破解平台】win xp sp2
- 【保护方式】UPX, Aspack
- 【开发语言】易语言
- 【本文写于】2005-04-27
- 【文章声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 破解分析

先用`PEID`查有壳，ASPack 2.12 -> Alexey Solodovnikov [Overlay]，载入`OD`，手动解决，

```asm
430001  pushad                        OD载入程序停在这里
430002  call 冒险岛加.0043000A        F7追入
430007  jmp 45A004F7
43000C  push ebp
43000D  retn
```

到这里

```asm
43000A  pop ebp
43000B  inc ebp
43000C  push ebp
43000D  retn
```

返回到这里

```asm
430008  jmp short 冒险岛加.0043000E
43000A  pop ebp
43000B  inc ebp
43000C  push ebp
43000D  retn
```

```asm
43000E  call 冒险岛加.00430014         F7追入
430013  jmp short 冒险岛加.00430072
430015  mov ebx,-13
```

到这里

```asm
430014  pop ebp
430015  mov ebx,-13
43001A  add ebx,ebp
43001C  sub ebx,30000
430022  cmp dword ptr ss:[ebp+422],0
430029  mov dword ptr ss:[ebp+422],ebx
43002F  jnz 冒险岛加.0043039A
430035  lea eax,dword ptr ss:[ebp+42E]
43003B  push eax
43003C  call dword ptr ss:[ebp+F4D]
430042  mov dword ptr ss:[ebp+426],eax
430048  mov edi,eax
43004A  lea ebx,dword ptr ss:[ebp+5E]
43004D  push ebx
43004E  push eax
43004F  call dword ptr ss:[ebp+F49]
```

向下找`POPAD`指令，在这里

```asm
4303AF  popad                        在这里按F4
4303B0  jnz short 冒险岛加.004303BA
4303B2  mov eax,1
4303B7  retn 0C
4303BA  push 0
4303BF  retn
```

按F4后指令变成这样

```asm
4303AF  popad
4303B0  jnz short 冒险岛加.004303BA   跳
4303B2  mov eax,1
4303B7  retn 0C
4303BA  push 冒险岛加.0042EADF        这里压入第二层壳的入口
4303BF  retn                          返回到第二层壳的入口
```

第二层壳的入口是这样

```asm
42EADF  nop
42EAE0  popad
42EAE1  mov esi,冒险岛加.0041B000
42EAE6  lea edi,dword ptr ds:[esi+FFFE6000]
42EAEC  add word ptr ds:[edi+2E704],5
42EAF5  push edi
42EAF6  or ebp,FFFFFFFF
42EAF9  jmp short 冒险岛加.0042EB0A
```

向下找`PUSHAD`指令，在这里

```asm
42EC36  pushad                   在这里按F4
42EC37  jmp 冒险岛加.00403831    跳到真正的入口403831
42EC3C  add byte ptr ds:[eax],al
42EC3E  add byte ptr ds:[eax],al
42EC40  add byte ptr ds:[eax],al
42EC42  add byte ptr ds:[eax],al
42EC44  add byte ptr ds:[eax],al
```

真正的入口代码是这样

```asm
403831  push ebp                  停在这里,现在用LOADPE选完全脱壳DUMP内存中的文件
403832  mov ebp,esp
403834  push -1
403836  push 冒险岛加.004062F0
40383B  push 冒险岛加.00404CA4
403840  mov eax,dword ptr fs:[0]
403846  push eax
403847  mov dword ptr fs:[0],esp
40384E  sub esp,58
403851  push ebx
403852  push esi
403853  push edi
403854  mov dword ptr ss:[ebp-18],esp
```

`DUMP`好文件然后用`ImportREC`修复导入表，`OEP`填`00003831`。

这样就脱掉了，但是脱壳的过程中源文件的覆盖数据被丢弃了，要补回去，不然程序不能运行！

用`HEXWORKSHOP`将源文件物理地址`15800`后的数据全部拷贝到内存，然后贴到脱壳后文件的最后就可以了！

`OD`重新载入，`F9`运行，`ALT+E`，找到这个可执行模块，项目 5

- 基数=`00E50000`
- 大小=`00116000` (1138688.)
- 入口=`00EE96B4` `krnln.<ModuleEntryPoint>`
- 名称=`krnln`
- 文件版本=1, 0, 0, 1
- 路径=`C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\E\_80003\krnln.fnr`

双击，使用`OD`插件 Ultra string Reference 的 `Find ASCII` 功能，找到`\.\PhysicalDrive0`双击，到这里：

```asm
E5DC46   68 CC7FF200    push krnln.00F27FCC           ; ASCII "\\.\PhysicalDrive0"
E5DC4B   FF15 00D4F000  call dword ptr ds:[<&KERNEL32.Create>; kernel32.CreateFileA
E5DC51   8BF0           mov esi,eax
E5DC53   83FE FF        cmp esi,-1
E5DC56   0F84 C0000000  je krnln.00E5DD1C
E5DC5C   57             push edi
E5DC5D   B9 06000000    mov ecx,6
E5DC62   33C0           xor eax,eax
E5DC64   8D7C24 14      lea edi,dword ptr ss:[esp+14]
```

在`1000DC46`上按F2下断。

现在关闭外挂程序，重新载入。

`F9`运行程序后，断在这里了。

`F8`走回到主程序里：

```asm
422F68   83C4 10           add esp,10
422F6B   C745 F0 00000000  mov dword ptr ss:[ebp-10],0
422F72   68 00000000       push 0
422F77   BB C4060000       mov ebx,6C4
422F7C   E8 D2310000       call 冒险岛加.00426153
422F81   83C4 04           add esp,4
422F84   68 01000380       push 80000301
422F89   6A 00             push 0
422F8B   50                push eax
422F8C   68 01000000       push 1
422F91   BB 68010000       mov ebx,168
422F96   E8 B8310000       call 冒险岛加.00426153
422F9B   83C4 10           add esp,10            这里看到机器码：1347656527
422F9E   8945 E8           mov dword ptr ss:[ebp-18],eax
422FA1   68 04000080       push 80000004
422FA6   6A 00             push 0
```

向下走，这里

```asm
4236A6   83C4 04           add esp,4
4236A9   8B5D F0           mov ebx,dword ptr ss:[ebp-10]
4236AC   85DB              test ebx,ebx
4236AE   74 09             je short 冒险岛加.004236B9
4236B0   53                push ebx
4236B1   E8 7F2A0000       call 冒险岛加.00426135
4236B6   83C4 04           add esp,4
4236B9   8B45 D8           mov eax,dword ptr ss:[ebp-28]
4236BC   8945 F0           mov dword ptr ss:[ebp-10],eax
4236BF   8B45 F0           mov eax,dword ptr ss:[ebp-10]
4236C2   85C0              test eax,eax
4236C4   74 15             je short 冒险岛加.004236DB
4236C6   50                push eax              这里到看到注册码：DF65CDC2
4236C7   8BD8              mov ebx,eax
4236C9   E8 0AF7FFFF       call 冒险岛加.00422DD8
4236CE   40                inc eax
4236CF   50                push eax
4236D0   E8 8A2A0000       call 冒险岛加.0042615F
```

内存注册机：

- 地址：`4236C6`
- 次数：1
- 第一字节：`50`
- 指令长度：1
- 内存方式寄存器`EAX`

完工

---

## 总结

这个方法对E语言写的使用硬盘序列号加密的程序应该有通用性。

好了，就写到这里，大家一起多研究研究。。。。。。

---

## 版权说明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

特此感谢`Phoenix`及`PCG`成员的帮助和支持![](https://bbs.kanxue.com/view/img/face/27.gif) ![](https://bbs.kanxue.com/view/img/face/27.gif) ![](https://bbs.kanxue.com/view/img/face/27.gif) ![](https://bbs.kanxue.com/view/img/face/27.gif)

附软件原版可以自己试试

地址：[128K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0W2&6j5$3&6Q4x3X3g2U0L8$3#2Q4x3V1k6S2N6s2c8S2j5$3S2E0k6h3&6@1i4K6u0W2M7r3S2H3i4K6y4r3j5h3W2V1i4K6y4p5y4K6g2Q4x3U0k6U0K9r3g2U0K9$3W2V1i4K6y4p5x3K6b7&6x3r3c8Q4x3U0k6V1L8%4N6F1L8r3!0S2k6q4)9K6c8o6p5`.](https://bbs.kanxue.com/elink@b63K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0W2&6j5$3&6Q4x3X3g2U0L8$3#2Q4x3V1k6S2N6s2c8S2j5$3S2E0k6h3&6@1i4K6u0W2M7r3S2H3i4K6y4r3j5h3W2V1i4K6y4p5y4K6g2Q4x3U0k6U0K9r3g2U0K9$3W2V1i4K6y4p5x3K6b7&6x3r3c8Q4x3U0k6V1L8%4N6F1L8r3!0S2k6q4)9K6c8o6p5`.)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
