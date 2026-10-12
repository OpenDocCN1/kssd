
```
00406EE8  - FF25 74B2B700   JMP DWORD PTR DS:[B7B274]   
00406EEE    8BC0            MOV EAX,EAX   
00406EF0  - FF25 70B2B700   JMP DWORD PTR DS:[B7B270]   
00406EF6    8BC0            MOV EAX,EAX   
00406EF8  - FF25 6CB2B700   JMP DWORD PTR DS:[B7B26C]   
00406EFE    8BC0            MOV EAX,EAX   
00406F00    50              PUSH EAX   
00406F01    6A 40           PUSH 40   
00406F03    E8 E0FFFFFF     CALL BadCopy.00406EE8   
00406F08    C3              RETN   

....................................................   
01511C64    55              PUSH EBP //F8 返回壳中。   
01511C65    8BEC            MOV EBP,ESP   
01511C67    8B45 08         MOV EAX,DWORD PTR SS:[EBP+8]   
01511C6A    85C0            TEST EAX,EAX   
01511C6C    75 13           JNZ SHORT 01511C81   
01511C6E    813D A47A5101 0>CMP DWORD PTR DS:[1517AA4],400000        ; ASCII "MZP"   
01511C78    75 07           JNZ SHORT 01511C81   
01511C7A    A1 A47A5101     MOV EAX,DWORD PTR DS:[1517AA4]   
01511C7F    EB 06           JMP SHORT 01511C87   
01511C81    50              PUSH EAX   
01511C82    E8 3135FFFF     CALL 015051B8                            ; JMP to kernel32.GetModuleHandleA   
01511C87    5D              POP EBP   
01511C88    C2 0400         RETN 4 //Ctrl+F9 返回。   

......................................................   
00406FB5    A3 14775000     MOV DWORD PTR DS:[507714],EAX            ; BadCopy.00400000   
00406FBA    A1 14775000     MOV EAX,DWORD PTR DS:[507714]   
00406FBF    A3 B4105000     MOV DWORD PTR DS:[5010B4],EAX   
00406FC4    33C0            XOR EAX,EAX   
00406FC6    A3 B8105000     MOV DWORD PTR DS:[5010B8],EAX   
00406FCB    33C0            XOR EAX,EAX   
00406FCD    A3 BC105000     MOV DWORD PTR DS:[5010BC],EAX   
00406FD2    E8 C1FFFFFF     CALL BadCopy.00406F98   
00406FD7    BA B0105000     MOV EDX,BadCopy.005010B0   
00406FDC    8BC3            MOV EAX,EBX       //这里 EBX 就是我们要的 EAX 的数值。00500638 记下来。   
00406FDE    E8 65D7FFFF     CALL BadCopy.00404748   
00406FE3    5B              POP EBX   
00406FE4    C3              RETN   

......................................................   
00500997    0000            ADD BYTE PTR DS:[EAX],AL  确定真 Oep 为 00500998   
00500999    0000            ADD BYTE PTR DS:[EAX],AL   
0050099B    0000            ADD BYTE PTR DS:[EAX],AL   
0050099D    0000            ADD BYTE PTR DS:[EAX],AL   
0050099F    0000            ADD BYTE PTR DS:[EAX],AL   
005009A1    0000            ADD BYTE PTR DS:[EAX],AL   
005009A3    E8 FC65F0FF     CALL BadCopy.00406FA4   
005009A8    B0 01           MOV AL,1          //临时 Oep,滚动条往上看。   
005009AA    84C0            TEST AL,AL   
005009AC    74 59           JE SHORT BadCopy.00500A07   
005009AE    A1 D8615000     MOV EAX,DWORD PTR DS:[5061D8]   
005009B3    8B00            MOV EAX,DWORD PTR DS:[EAX]   
005009B5    E8 FE0CF7FF     CALL BadCopy.004716B8   
005009BA    A1 D8615000     MOV EAX,DWORD PTR DS:[5061D8]   
005009BF    8B00            MOV EAX,DWORD PTR DS:[EAX]   
005009C1    BA 140A5000     MOV EDX,BadCopy.00500A14                ; ASCII "BadCopy Pro"   
005009C6    E8 F908F7FF     CALL BadCopy.004712C4   
005009CB    8B0D D45D5000   MOV ECX,DWORD PTR DS:[505DD4]           ; BadCopy.00B6A1F8   
005009D1    A1 D8615000     MOV EAX,DWORD PTR DS:[5061D8]   
005009D6    8B00            MOV EAX,DWORD PTR DS:[EAX]   
005009D8    8B15 D4C24E00   MOV EDX,DWORD PTR DS:[4EC2D4]           ; BadCopy.004EC320   
005009DE    E8 ED0CF7FF     CALL BadCopy.004716D0   

......................................................   
```

现在我们已经得到了

1. 脱壳文件
2. 被抽 11 个字节代码

```
PUSH EBP
MOV EBP,ESP
ADD ESP,-10
MOV EAX,00500638
```

剩下 IAT 修复，关闭 `OD`，打开加壳程序，呵呵 OEP 填 `1000`, `ImportREC` 把 Size 认为 `000000B4` 太小了，改为 `1000`, 点获得输入表。显示无效，先用等级 1 方式跟踪，然后用 `Asprtect1.22` 插件再减轻一下压力。还剩 563 个指针没修复。

现在的问题是,这 563 个指针是有用的吗? `ImpRec 1.6` 会告诉你

点击 `show invalid` 并右键单击选择其中的一个，让我们来修复它

```
0077b234<- 右键单击选择 DeasmHexView 查看:
```

`ImpRec 1.6` 提示你,`read error!` 好,够了,只要有这个提示的一律可以安全的 `Cut`。

如果这个指针有效,里面会显示汇编指令。

找了一会,发现 `0077B3A0` 处的指针是唯一一个没识别的指针。

右键单击选择 `DeasmHexView` 查看,究竟它是 `Kernel32.dll` 中的哪个函数呢,按 `Page Down` 两次,看到

```
01511D2E    call 01505158    // = kernel32.dll/00B3/FindResourceA
```

第一个看到的 Api 函数就是要修复的指针。

呵呵,修复它吧,双击 `0077B3A0` 这行,填入 Api 函数 `FindResourceA`。

拿起剪刀，将剩下 562 个垃圾指针全部 `Cut`, IAT 全部修复，爽啊。修正 Oep 为 `100998` 最后修复文件,最后 `Od` 载入脱壳修复的程序，补上 11 个字节代码，保存为一个文件，正常运行。

### 破解总结

发现 `Dephi` 语言的 `Asprotect1.23Rc4` 的壳开始用 `Stolen Code`，并且有变形指令，反脱壳代码，最麻烦,`tc ebp==12fff0` 还不错吧。作者没有使用 `Asprotect` 内置的 `RSA` 算法保护，而采用自己的注册算法，任然是 `RSA1024`, 破解参考二哥=`weiyi75`=`David` 的再次分析坏盘拷贝破文, 感谢你耐心看完。

--------------------------------------------------------------------------------

### 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
