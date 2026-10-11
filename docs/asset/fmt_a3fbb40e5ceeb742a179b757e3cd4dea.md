
```
00343C63  90                NOP
00343C64  90                NOP
00343C65  90                NOP
```

在代码窗口内按`Ctrl+G`，输入`343EB2`，看到这两行代码：

```
00343EB2  8B9D E1364000     MOV EBX,DWORD PTR SS:[EBP+4036E1]
00343EB8  039D E5364000     ADD EBX,DWORD PTR SS:[EBP+4036E5]
```

把它们修改为：

```
00343EB2  8907              MOV DWORD PTR DS:[EDI],EAX             ；把 API 函数地址添入原始 IAT
00343EB4  90                NOP                                     ；这个是 IMPORT BY NAME 的
00343EB5  EB 5E             JMP SHORT 00343F15                      ；跳过第二层重定位
00343EB7  90                NOP
```

还要改一下这里：

```
00343F1D  83C7 20           ADD EDI,20
```

改为：

```
00343F1D  83C7 04           ADD EDI,4                                ；调整 IAT 指针
```

接下来要注意下面的陷阱：

```
00344124  64:FF35 30000000  PUSH DWORD PTR FS:[30]                   ；NT
0034412B  58                  POP EAX                                 ；pointer to PEB
0034412C  0FB658 02           MOVZX EBX,BYTE PTR DS:[EAX+2]
00344130  0ADB                OR BL,BL                                ；检测应用程序级 debugger，用 OD 跟踪一定要注意这里
00344132  0F85 9F130000       JNZ 003454D7                            ；这里是本壳中唯一对 OllyDBG 有威胁的地方
00344138  EB 2A               JMP SHORT 00344164
```

用`F4`执行到`344130`这行，把`BL`寄存器清零，

下面对`HOOK_CALL`代码进行修改，直接解密，修复原始代码，关键代码是下面几行：

```
00344255  0385 0D324000       ADD EAX,DWORD PTR SS:[EBP+40320D]
0034425B  2B85 29324000       SUB EAX,DWORD PTR SS:[EBP+403229]
00344261  8BDE                MOV EBX,ESI
00344263  2BD8                SUB EBX,EAX
00344265  8958 FC             MOV DWORD PTR DS:[EAX-4],EBX
00344268  66:C740 FA 90E8     MOV WORD PTR DS:[EAX-6],0E890
```

把它们修改为：

```
00344255  8B5F 04             MOV EBX,DWORD PTR DS:[EDI+4]            ；这几行解密算法来自 3445B5 处的分析
00344258  81C3 CA0D0BF6       ADD EBX,F60B0DCA                        ；CRC_KEY,来自 3443B2 处的计算结果
0034425E  2BD8                SUB EBX,EAX                             ；
00344260  F7D3                NOT EBX                                 ；
00344262  C1C3 10             ROL EBX,10                              ；
00344265  8958 FC             MOV DWORD PTR DS:[EAX-4],EBX            ；
00344268  66:C740 FA FF25     MOV WORD PTR DS:[EAX-6],25FF            ；恢复为原始的 JMP DWORD PTR [XXXXXXXX]
```

接下来有个`DELPHI`的`MAINFORM`必须要处理一下，因为现在它位于壳所分配的空间内，我们`DUMP`时会丢失掉，脱壳后就会出问题。

```
003442A4  8B85 59324000       MOV EAX,DWORD PTR SS:[EBP+403259]       ；这里是对 DELPHI 的 MAINFORM 的处理
003442AA  0BC0                OR EAX,EAX                              ；其实就是把 MAINFORM 从原代码区搬到了壳里
003442AC  74 3F               JE SHORT 003442ED                       ；只需调整一下首指针就行了
003442AE  8DB5 2E164000       LEA ESI,DWORD PTR SS:[EBP+40162E]
003442B4  03F0                ADD ESI,EAX                             ；
003442B6  8B1E                MOV EBX,DWORD PTR DS:[ESI]              ；MAINFORM 的原始参考 RVA
003442B8  039D 0D324000       ADD EBX,DWORD PTR SS:[EBP+40320D]       ；BASE ADDRESS
003442BE  C706 00000000       MOV DWORD PTR DS:[ESI],0
003442C4  83C6 04             ADD ESI,4                               ；MAINFORM 的现在的地址
003442C7  8933                MOV DWORD PTR DS:[EBX],ESI              ；把 MAINFORM 的现在的地址添到其参考位置
```

当运行到这里时，`ESI=348306`，从`ESI`往后看数据区，容易看出一直到`34921C`都应该是`FORM`的数据，长度为`0F17`字节，

我们再从`EBX=4F96C4`往后看数据区，到`4FAA36`处的有一个`A6`，再往下直到`4FB94C`都是`0`，`A6`与`ESI`处数据的第一字节吻合，

长度也都是`0F17`字节，可以判断出`4FAA36`处就应该是`MAINFORM`的原始位置，因此做如下修改：

```
003442C7  BF 36AA4F00         MOV EDI,4FAA36                          ；FORM 的原始地址
003442CC  893B                MOV DWORD PTR DS:[EBX],EDI              ；添入 FORM 的参考指针
003442CE  B9 170F0000         MOV ECX,0F17                            ；FORM 长度
003442D3  F3:A4               REP MOVSB                               ；移回原始位置
003442D5  EB 16               JMP SHORT 003442ED
003442D7  90                NOP
003442D8  90                NOP
003442D9  90                NOP
003442DA  90                NOP
```

接下来把`ANTI DUMP`处理掉，`2000`下是这里：

```
00344311  64:FF35 30000000    PUSH DWORD PTR FS:[30]
00344318  58                  POP EAX
00344319  85C0                TEST EAX,EAX
0034431B  78 0F               JS SHORT 0034432C
0034431D  8B40 0C             MOV EAX,DWORD PTR DS:[EAX+C]
00344320  8B40 0C             MOV EAX,DWORD PTR DS:[EAX+C]
00344323  C740 20 00100000    MOV DWORD PTR DS:[EAX+20],1000          ；ANTI DUMP，改为 NOP
0034432A  EB 39               JMP SHORT 00344365
```

从下面这里我们`F7`就来到`OEP`了：

```
0034442A  C3                  RETN                                    ；TO OEP
```

看看`OEP`处的代码：

```
0050CB6C  FFD7                CALL EDI
0050CB6E  58                  POP EAX
```

把这两行代码改成：

```
0050CB6C  55                  PUSH EBP
0050CB6D  8BEC                MOV EBP,ESP
```

到此就算大功告成了。

后面工作大家就可以按个自习惯做了，我是用`OD`的`PLUGINS`的`OllyDump`先生成`DUMP.EXE`

然后运行`Import REConstructor v1.6F`，`Attach`到我们的进程上

`OEP`：`0010CB6C`

`RVA`：`0018B1B8` `SIZE`：`000007E0`

去掉`Add new section`选项，输入`RVA`：`0018B998`，这样生成的文件更精练一些，不过要算好空间，有把握才行

点`Get Imports`按钮，一切`OK`

再`Fix Dump`，完全搞定了。

要想做锦上添花，还可以减肥，把壳所在的最后一个`SECTION`去掉，需要手工修复以下`TLS`，还要把被搬到壳中的几个资源

放到合适的位置上，这个我就懒得叙述了。

```
#################################################################################################

后记：

#################################################################################################
```

我还是不要装糊涂的好，脱到最后，我也知道壳是谁做的了，里面有一句信息：`i am xxxxx,do not unpack me,please.`

不好意思了，望兄弟不要介意。

```
#################################################################################################

iPB/heXer

03.09.04  4dd466d0e1e0cc123afbe504da2462ca
```
