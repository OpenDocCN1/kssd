

# 浅谈伪装壳 Hide PE v1.1 的实现机理

> 原文：[`www.pediy.com/kssd/pediy10/66654.html`](https://www.pediy.com/kssd/pediy10/66654.html)

  
  
    早上到论坛发现 cxlrb 兄发布了一个伪装壳，下来测试下效果 加了 DEL 的测试不成功 加 VC 6.0 测试 OK 既然可以加壳成功 那我们来分析一下为什么有些失败呢 我们先来分析下这个伪装壳是如何实现的   
  
Hide PE v1.1 Cracked by Flashback  
  
  
该伪装壳有有两个选项 我们先来选 Asp 来实现伪装（试验品为 VC 写的一程序）   
  
![](img/e004816de9abf1179f37a3913c914dd5.jpg)  
  

代码:

```
伪装后代码如下：

00433B45 >/$  68 01404900   PUSH HideWiza.00494001                   ;  (Initial CPU selection)
00433B4A  |.  E8 01000000   CALL HideWiza.00433B50                   ;  F7
00433B4F  \\.  C3            RETN                                     ;  这里就可以返回
00433B50   $  C3            RETN

返回到这里：

00494001    BA 453B4300     MOV EDX,HideWiza.<模块入口点>             ;  将 OEP 送 EDX 准备 SMC
00494006    B8 558BEC6A     MOV EAX,6AEC8B55
0049400B    8902            MOV DWORD PTR DS:\[EDX\],EAX
0049400D    83C2 04         ADD EDX,4
00494010    B8 FF6818EE     MOV EAX,EE1868FF
00494015    8902            MOV DWORD PTR DS:\[EDX\],EAX
00494017    83C2 04         ADD EDX,4
0049401A    B8 45006880     MOV EAX,80680045
0049401F    8902            MOV DWORD PTR DS:\[EDX\],EAX
00494021    83C2 F8         ADD EDX,-8
00494024  - FFE2            JMP EDX                                  ;  OEP 

```

入口点被做了修改 OEP 还是很好找到的  由于伪装壳壳不对 IAT 处理 Dump 下来也 OK  只是扰乱了数据 OD 中使用 ASCII 搜索将失效 :lol  
  
我们再仔细分析下程序是如何对 OEP 的修改 仔细观察地址 就会发现其实是做了一个 SMC 而已代码:

```
00433B45 >/$  55                   PUSH EBP                                 ;  OEP
00433B46  |?  8BEC                 MOV EBP,ESP
00433B48  |?  6A FF                PUSH -1
00433B4A  |.  68 18EE4500          PUSH HideWiza.0045EE18                   
00433B4F  \\.  68 808F4300          PUSH HideWiza.00438F80                   

对照上文中的 SMC 代码

00433B45 >55 8B EC 6A FF 68 18 EE 45 00 68 80              Ujh.h.

```

入口点现将 SMC 函数的入口压栈 然后 retn 返回到该函数 再函数里实现 OEP 原始数据的 SMC 写入 最后 JMP EDX(EDX 保存原程序的 OEP) 原来伪装就是这样来实现的  
  
那程序是再哪里添加了 ASP 的特征码呢？我们查看一下 fly 大哥制作的 PEsign  
  
\[ASProtect V2.X Registered -> Alexey Solodovnikov   \* Sign.By.fly\]  
signature = 68 01 ?? ?? ?? E8 01 00 00 00 C3 C3  
ep\_only = true  
  
而这个代码 正是程序被修改后的 OEP 入口点的代码：代码:

```
00433B45 >/$  68 01404900          PUSH HideWiza.00494001                   ;  (Initial CPU selection)
00433B4A  |.  E8 01000000          CALL HideWiza.00433B50                   ;  F7
00433B4F  \\.  C3                   RETN                                     ;  这里就可以返回
00433B50   $  C3                   RETN

正好和 ASP 的特征码吻合：68 01 40 49 00 E8 01 00 00 00 C3 C3 

```

根据特征码 Hide PE 在生成自己的 Patch 代码时就有一个局限 即 Patch 的代码地址要为 00XXXX01 即地址低位要为 01 或许就是这里导致对某些程序 Hide 后无法运行吧。  
  
我们再来验证看一下 看一下 Del 加伪装壳无法运行的问题所在:代码:

```
以 MP3 音频录音机为例：

004E6D58 > $  68 01305C00          PUSH MP3\_Audi.005C3001
004E6D5D      E8                   DB E8
004E6D5E      01                   DB 01
004E6D5F      00                   DB 00
004E6D60   .  0000C3C3             DD C3C30000

我们按照上方我们分析作者伪装 ASP 的意图来格式化代码：

004E6D58 > $  68 01305C00          PUSH MP3\_Audi.005C3001
004E6D5D      E8 01000000          CALL MP3\_Audi.004E6D63            ； 问题在这里 
004E6D62      C3                   RETN
004E6D63      C3                   RETN

返回到来到这里：

005C3001    1000                   ADC BYTE PTR DS:\[EAX\],AL
005C3003    0000                   ADD BYTE PTR DS:\[EAX\],AL
005C3005    0202                   ADD AL,BYTE PTR DS:\[EDX\]
005C3007    0030                   ADD BYTE PTR DS:\[EAX\],DH
005C3009    8210 BF                ADC BYTE PTR DS:\[EAX\],-41
005C300C    06                     PUSH ES
005C300D    092A                   OR DWORD PTR DS:\[EDX\],EBP
005C300F    8648 86                XCHG BYTE PTR DS:\[EAX-7A\],CL
005C3012    F7                     ???                                      ; 未知命令
005C3013    0D 010702A0            OR EAX,A0020701
005C3018    8210 B0                ADC BYTE PTR DS:\[EAX\],-50
005C301B    3082 10AC0201          XOR BYTE PTR DS:\[EDX+102AC10\],AL
005C3021    0131                   ADD DWORD PTR DS:\[ECX\],ESI
005C3023    0B30                   OR ESI,DWORD PTR DS:\[EAX\]
005C3025    0906                   OR DWORD PTR DS:\[ESI\],EAX
005C3027    05 2B0E0302            ADD EAX,2030E2B

```

我们发现 他 Patch 代码的地址有代码或数据 导致程序运行异常  我们既然了解了伪装的机理 那我们就自己来实现一下   
  
先 COPY 原程序 OEP 的代码：004E6D58 >55 8B EC 83 C4 F0 53 B8 50 67 4E 00              UgN.瑁  
  
然后找到一个以地址低位为 01 的空白区段地址   
  
004E6F01      00                   DB 00 代码:

```
OEP 修改如下：

004E6D58 >    68 016F4E00          PUSH MP3Recor.004E6F01
004E6D5D      E8 01000000          CALL MP3Recor.004E6D63
004E6D62      C3                   RETN
004E6D63      C3                   RETN

然后我们写 SMC 数据：

004E6F01      BA 586D4E00          MOV EDX,MP3Recor.<模块入口点>
004E6F06      B8 558BEC83          MOV EAX,83EC8B55
004E6F0B      8902                 MOV DWORD PTR DS:\[EDX\],EAX
004E6F0D      83C2 04              ADD EDX,4
004E6F10      B8 C4F053B8          MOV EAX,B853F0C4
004E6F15      8902                 MOV DWORD PTR DS:\[EDX\],EAX
004E6F17      83C2 04              ADD EDX,4
004E6F1A      B8 50674E00          MOV EAX,MP3Recor.004E6750
004E6F1F      8902                 MOV DWORD PTR DS:\[EDX\],EAX
004E6F21      83C2 F8              ADD EDX,-8
004E6F24      FFE2                 JMP EDX  

```

保存文件后 仍旧无法运行 提示代码段不允许写入 我们修改代码段的属性为可写：  
  
![](img/d645a445f67c9a91101ddd28f9aff7a4.jpg)  
  
OK 这下跑起来了 PEiD 测壳：ASProtect V2.X Registered -> Alexey Solodovnikov   \* Sign.By.fly \[Overlay\] \*  
  
只是对程序被伪装后为何有些成功有些失败感兴趣而已 无他 Only Game ~~