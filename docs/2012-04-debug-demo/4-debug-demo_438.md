
# 飓风加密的 WORD 文档提取 [之] 千里之堤亏于快捷键

> 原文：[`www.pediy.com/kssd/pediy11/120149.html`](https://www.pediy.com/kssd/pediy11/120149.html)

翻看论坛原来存档帖子的时候，发现了自己这个宝贵的 ID，原来在 04 年的时候就注册了，无奈那时候菜的一塌糊涂。珍惜 ID 发个原创帖，希望水贴也能转正吧。六年无长进，汗颜呀。

看到有位坛友求飓风加密的 WORD 文档，破解的话，已经有很多牛牛研究的一塌糊涂了。以至于无人回答，`http://bbs.pediy.com/showthread.php?t=119974`

所以小菜就下载下来，分析了下。破解的过程很简单，都是老路。记录下吧。也免的自己忘记

引用: `OD` 载入。`F9` 运行

随便输入试炼码 `111111111111`

点击确定，然后 `F12` 暂停。去堆栈查看调用

```
Call stack of main thread, item 3

 Address=0012F5A8

 Stack=0045A306

 Procedure / arguments=1212.0045D450

 Called from=1212.0045A301

 Frame=0012F604
```

双击来到

```
0045A2F6  .  8990 4C020000  MOV DWORD PTR DS:[EAX+24C],EDX
0045A2FC  |>  A1 E0BB4900   /MOV EAX,DWORD PTR DS:[49BBE0]
0045A301  .  E8 4A310000   |CALL 0045D450
0045A306  .  A1 E0BB4900   |MOV EAX,DWORD PTR DS:[49BBE0]
0045A30B  .  80B8 9C000000>|CMP BYTE PTR DS:[EAX+9C],0
0045A312  .  74 0F         |JE SHORT 0045A323
```

`Ctrl+A` 分析下，然后搜索字符串。

有几个字符串是比较重要的

1. 阅读授权不合法

双击进去，向上翻到段首附近下断点

```
00494AF7   .  55             PUSH EBP                F2 下断
00494AF8   .  68 04514900   PUSH 00495104
00494AFD   .  64:FF30       PUSH DWORD PTR FS:[EAX]
00494B00   .  64:8920       MOV DWORD PTR FS:[EAX],ESP
00494B03   .  8D45 DC       LEA EAX,DWORD PTR SS:[EBP-24]
00494B06   .  E8 5DFCF6FF   CALL 00404768
00494B0B   .  8D95 0CFEFFFF LEA EDX,DWORD PTR SS:[EBP-1F4]
00494B11   .  8B83 FC020000 MOV EAX,DWORD PTR DS:[EBX+2FC]
.......
省略数行
.......
00494BA3   .  83F8 04       CMP EAX,4
00494BA6   .  0F85 0A010000 JNZ 00494CB6               ; JNZ 需 NOP 掉
00494BAC   .  8D8D F8FDFFFF LEA ECX,DWORD PTR SS:[EBP-208]
00494BB2   .  33D2          XOR EDX,EDX
00494BB4   .  8BC6          MOV EAX,ESI
00494BB6   .  8B38          MOV EDI,DWORD PTR DS:[EAX]
00494BB8   .  FF57 0C       CALL DWORD PTR DS:[EDI+C]        ; 将这个 Call NOP 掉
00494BBB   .  8B95 F8FDFFFF MOV EDX,DWORD PTR SS:[EBP-208]
00494BC1   .  B8 A4BD4900   MOV EAX,0049BDA4
00494BC6   .  E8 F1FBF6FF   CALL 004047BC
00494BCB   .  8D4D E8       LEA ECX,DWORD PTR SS:[EBP-18]
00494BCE   .  BA 01000000   MOV EDX,1
00494BD3   .  8BC6          MOV EAX,ESI
00494BD5   .  8B38          MOV EDI,DWORD PTR DS:[EAX]
00494BD7   .  FF57 0C       CALL DWORD PTR DS:[EDI+C]        ; 将这个 Call NOP 掉
00494BDA   .  8D4D E0       LEA ECX,DWORD PTR SS:[EBP-20]
00494BDD   .  BA 02000000   MOV EDX,2
00494BE2   .  8BC6          MOV EAX,ESI
00494BE4   .  8B38          MOV EDI,DWORD PTR DS:[EAX]
00494BE6   .  FF57 0C       CALL DWORD PTR DS:[EDI+C]        ; 将这个 Call NOP 掉
00494BE9   .  8D4D E4       LEA ECX,DWORD PTR SS:[EBP-1C]
00494BEC   .  BA 03000000   MOV EDX,3
00494BF1   .  8BC6          MOV EAX,ESI
00494BF3   .  8B38          MOV EDI,DWORD PTR DS:[EAX]
00494BF5   .  FF57 0C       CALL DWORD PTR DS:[EDI+C]        ; 将这个 Call NOP 掉
00494BF8   .  8D4D DC       LEA ECX,DWORD PTR SS:[EBP-24]
00494BFB   .  BA 04000000   MOV EDX,4
.......
省略数行
.......
00494C9D   .  8B55 DC       MOV EDX,DWORD PTR SS:[EBP-24]
00494CA0   .  E8 CFFEF6FF   CALL 00404B74
00494CA5     74 1E         JE SHORT 00494CC5               ; 改为 jmp
00494CA7   .  B8 28514900   MOV EAX,00495128
```

2. 注意这个 `yao1`，`yao2`，和 `yao3`

`yao1` 附近

双击进去，到汇编窗口。向下拉找到关键比较

```
00494E58   .  E8 7FEDFFFF   CALL 00493BDC
00494E5D   .  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]    ; F2 下断
00494E60   .  8B55 EC       MOV EDX,DWORD PTR SS:[EBP-14]    ；将 EDX 中的值修改为 EAX 中的值
00494E63   .  E8 0CFDF6FF   CALL 00404B74
00494E68   .  74 0F         JE SHORT 00494E79               ; 关键跳转
```

`Ctrl+B` 搜索二进制代码

```
8B 45 F0 8B 55 EC
```

可以还搜索到一处，`F2` 下断

```
0049505D   .  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]    ; F2 下断
00495060   .  8B55 EC       MOV EDX,DWORD PTR SS:[EBP-14]    ；将 EDX 中的值修改为 EAX 中的值
00495063   .  E8 0CFBF6FF   CALL 00404B74
00495068   .  74 0C         JE SHORT 00495076
```

3. 去 `yao3` 那里看看，还少这最后一个断点。

```
0049533E  |.  8D45 DC       LEA EAX,DWORD PTR SS:[EBP-24]
00495341  |.  8D55 F8       LEA EDX,DWORD PTR SS:[EBP-8]
00495344  |.  E8 93E8FFFF   CALL 00493BDC
00495349  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]      ; F2 下断
0049534C  |.  8B55 F8       MOV EDX,DWORD PTR SS:[EBP-8]    ；将 EDX 中的值修改为 EAX 中的值
0049534F  |.  E8 20F8F6FF   CALL 00404B74
00495354  |.  0F95C3        SETNE BL
00495357  |.  84DB          TEST BL,BL
00495359  |.  74 11         JE SHORT 0049536C
```

至此，`WORD` 文档已经能浏览了，但是这东西不能保存，也没办法打印。可恶的是它还 `Anti OD` 中的 `Ctrl+C` 复制，但是 `Shift+C` 复制二进制代码还是可以的，无奈下。所有的代码都是直接保存到文件中然后拷贝出来的。`LoadPE Dump` 也检测，但是好在只是强行关闭，界面还是出来了。在想怎么保存这个文档的时候，无意中触碰到了 `F1` 按键，呵呵，看到帮助文档里。于是乎豁然开朗，点击 `F12`，然后快速输入一个文件名，小 `p` `![](img/6821a6175013046370981b6035d43ab4.jpg "使眼色")` 回车。去我的文档里找找。果然保存。小技巧一个，希望大家不是被我的标题忽悠进来的，水文一篇留作纪念。
