
# Calendar Builder 3.3a 破解

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-83.htm`](https://www.pediy.com/kssd/pediy07/pediy7-83.htm)

**【破解作者】** layper  
**【作者邮箱】** layper2002@yahoo.com.cn  
**【使用工具】** `peid`, `od`  
**【破解平台】** `Win9x`/`NT`/`2000`/`XP`  
**【软件名称】** Calendar Builder 3.3a  
**【下载地址】** `http://www.onlinedown.net/soft/3467.htm`  
**【软件简介】** 非常棒的日历制作软件，程序内建多种的日历模式样板，如每月、每年、每周等等，它支持中文字形，可加入文字及喜欢的图片，更可将每个节日放上喜欢的 ICON 来表示，相当不错，设计完并可将它打出来使用。  
**【软件大小】** 1617KB  
**【加壳方式】** 无  
**【破解声明】** 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 【破解内容】

这个是老外的软件，在破解这个软件时，由于英语不好给本人带来了很大麻烦。

用 `OD` 查壳，无，用 `Borland Delphi 2.0` 写的。破解之前，先分析一下有用的信息，注册码先前要有`"RKS-"`，错误会跳出一个窗口，`INI` 文件处有用户名。

刚开始，我下断就是按照这两个地方进行的。用 `bp MessageBoxExA` 拦不下，又换 `GetPrivateProfileString`、`GetPrivateProfileInt`、`WritePrivateProfileString`、`WritePrivateProfileInt` 还是拦下了，但不知道是哪个，又换 `WriteFile`、`CreateFile` 还是找不到关键断点，没办法，还是另想招数了，换查找字符串，最初用字符串插件，吗呀，全是英文，看得我头皮发麻，TMD，一狠心，右键-搜索-字符参考，在字符参考这里每个命令设置断点，老子就不信这样我还找不到。接着就不断的 `F9`、`F2` 的拼命运行、取消断点(别看这个方法笨，对付老外的软件还是蛮有用的 :) )到 `0042F433` 处停住，`ASCII "BUTTON"`，点击 `enter serial number` 继续，到 `0042D86F` 处出现注册对话框，名字输入`"layper"`，注册码输入`"RKS-1357924680"`，确定，(向下翻动堆栈窗口，呵呵，看见用户名和假注册码)

继续 `F2`、`F9`，到达 `004B0B4F` 处时，呵呵，堆栈框跳出了

```
0012F188   0012F1C0  指针到下一个 SEH 记录
0012F18C   004B0B7C  SE 句柄
0012F190   0012F1B4
0012F194   00B22070
0012F198   0012F288
0012F19C   00000000
0012F1A0   00B227BC  ASCII "920611" ；与 RKS-连接起来就是注册码
0012F1A4   000007CD
0012F1A8   00B22748  ASCII "LAYPER"
0012F1AC   00AB292C  ASCII "CB"
0012F1B0   00B2275C  ASCII "layper"
```

如果你只是想要一个注册码就此打住不用往下看了，`004B0B4F` 不取消断点，继续 `F9`、`F2`，不久又回到这里，看堆栈

```
0012F188   0012F1C0  指针到下一个 SEH 记录
0012F18C   004B0B7C  SE 句柄
0012F190   0012F1B4
0012F194   00B22070
0012F198   0012F288
0012F19C   00000000
0012F1A0   00B22748  ASCII "3683845" ；与“RKS-”连接起来又是注册码
0012F1A4   00001F37
0012F1A8   00B227BC  ASCII "LAYPER"
0012F1AC   00AB292C  ASCII "CB"
0012F1B0   00B2275C  ASCII "layper"
```

看来 `004B0B4F` 处有猫腻，算法部分应该就是这附近了，向上 `004B0AC0` 下断，右键-搜索-字符参考，清除所有断点，`F9` 运行重新下断

```asm
004B0AC0  /$  55             PUSH EBP
004B0AC1  |.  8BEC           MOV EBP,ESP
004B0AC3  |.  83C4 EC        ADD ESP,-14
004B0AC6  |.  53             PUSH EBX
004B0AC7  |.  56             PUSH ESI
004B0AC8  |.  57             PUSH EDI
004B0AC9  |.  33DB           XOR EBX,EBX                  ; ebx 清 0
004B0ACB  |.  895D EC        MOV DWORD PTR SS:[EBP-14],EBX    ; [ebp-14]清 0
004B0ACE  |.  895D F4        MOV DWORD PTR SS:[EBP-C],EBX     ; [ebp-c]清 0
004B0AD1  |.  8BD9           MOV EBX,ECX                  ; ecx 清 0
004B0AD3  |.  8955 F8        MOV DWORD PTR SS:[EBP-8],EDX
004B0AD6  |.  8945 FC        MOV DWORD PTR SS:[EBP-4],EAX
004B0AD9  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
004B0ADC  |.  E8 2B2EF5FF    CALL Cb.0040390C
004B0AE1  |.  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
004B0AE4  |.  E8 232EF5FF    CALL Cb.0040390C
004B0AE9  |.  33C0           XOR EAX,EAX
004B0AEB  |.  55             PUSH EBP
004B0AEC  |.  68 7C0B4B00    PUSH Cb.004B0B7C
004B0AF1  |.  64:FF30        PUSH DWORD PTR FS:[EAX]
004B0AF4  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP
004B0AF7  |.  33F6           XOR ESI,ESI
004B0AF9  |.  8BD3           MOV EDX,EBX
004B0AFB  |.  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
004B0AFE  |.  E8 CDF6FFFF    CALL Cb.004B01D0
004B0B03  |.  8945 F0        MOV DWORD PTR SS:[EBP-10],EAX
004B0B06  |.  8D55 F4        LEA EDX,DWORD PTR SS:[EBP-C]
004B0B09  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]       ; layper
004B0B0C  |.  E8 9752F5FF    CALL Cb.00405DA8
004B0B11  |.  8B45 F4        MOV EAX,DWORD PTR SS:[EBP-C]       ; LAYPER
004B0B14  |.  E8 3F2CF5FF    CALL Cb.00403758
004B0B19  |.  8BD0           MOV EDX,EAX                ; EDX=EAX=用户名长度
004B0B1B  |.  85D2           TEST EDX,EDX               ; 用户名是否为空
004B0B1D  |.  7E 20          JLE SHORT Cb.004B0B3F      ; 空则跳走
004B0B1F  |.  B9 01000000    MOV ECX,1                  ; ECX=1
004B0B24  |> 8B45 F4        /MOV EAX,DWORD PTR SS:[EBP-C]    ; EAX=LAYPER
004B0B27  |.  8A4408 FF      |MOV AL,BYTE PTR DS:[EAX+ECX-1]   ; 大写用户名逐位入 AL
004B0B2B  |.  3C 20          |CMP AL,20               ; 是否为空格
004B0B2D  |.  74 0C          |JE SHORT Cb.004B0B3B
004B0B2F  |.  25 FF000000    |AND EAX,0FF              ; 留下 AL，即大写用户名字符
004B0B34  |.  0FAF45 F0      |IMUL EAX,DWORD PTR SS:[EBP-10]     ; EAX 乘于[EBP-10]，注意[EBP-10]
004B0B38  |.  48             |DEC EAX                 ; 上一步结果减 1
004B0B39  |.  03F0           |ADD ESI,EAX             ; 上一次循环结果与 EAX 相加，ESI 存放计算结果
004B0B3B  |> 41             |INC ECX                  ; ECX 加 1
004B0B3C  |.  4A             |DEC EDX                 ; EDX 减 1，ECX 为计数器
004B0B3D  |.^ 75 E5          \JNZ SHORT Cb.004B0B24
004B0B3F  |> 8D55 EC        LEA EDX,DWORD PTR SS:[EBP-14]
004B0B42  |.  8BC6           MOV EAX,ESI                ; 计算结果放入 EAX
004B0B44  |.  E8 7B55F5FF    CALL Cb.004060C4                 ; 这个 CALL 把上面结果转为十进制
004B0B49  |.  8B4D EC        MOV ECX,DWORD PTR SS:[EBP-14]    ; 换算结果放入 ECX
004B0B4C  |.  8B45 08        MOV EAX,DWORD PTR SS:[EBP+8]
004B0B4F  |.  BA 940B4B00    MOV EDX,Cb.004B0B94                ; ASCII "RKS-"
004B0B54  |.  E8 4B2CF5FF    CALL Cb.004037A4
004B0B59  |.  33C0           XOR EAX,EAX
004B0B5B  |.  5A             POP EDX
004B0B5C  |.  59             POP ECX
004B0B5D  |.  59             POP ECX
004B0B5E  |.  64:8910        MOV DWORD PTR FS:[EAX],EDX
004B0B61  |.  68 830B4B00    PUSH Cb.004B0B83
004B0B66  |> 8D45 EC        LEA EAX,DWORD PTR SS:[EBP-14]
004B0B69  |.  E8 762AF5FF    CALL Cb.004035E4
004B0B6E  |.  8D45 F4        LEA EAX,DWORD PTR SS:[EBP-C]
004B0B71  |.  BA 03000000    MOV EDX,3
004B0B76  |.  E8 892AF5FF    CALL Cb.00403604
004B0B7B  \\.  C3             RETN
```

`004B0B34` 处的`[EBP-10]`分别为

`07cd\1f37\6fa\5ad\aea\99f\cd1\f17\115f\11b4\1749\1530\1b1f\1771\1b54\17d9\1b54\17d9\1b1f\1771\2081\1fb9\231f\0000\0998\03fe\1c17\1eb8\18a1\0e15\0905\245d\18a0\18a0\18a2\0000\0905\245d\18a1\0e15\`

其中 `07cd` 与 `1f37` 算出的结果为有效

## 算法总结

把用户名转为大写后各个字母相加在乘于 `07CD` 或 `1F37`，得之结果加上用户名长度再转化为十六进制，在与`"RKS-"`连接既是注册码了上面是算法部分，要把这个软件破解过程写完整（比如`[EBP-10]`为什么只是 `07CD` 和 `1F37`）需要很长的篇幅，限于篇幅关系，就写这么多了

---

**【版权声明】** 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
