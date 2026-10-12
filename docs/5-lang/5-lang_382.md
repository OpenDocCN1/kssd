
# 独裁者 v1.26

> 原文：[`www.pediy.com/kssd/pediy05/pediy50405.htm`](https://www.pediy.com/kssd/pediy05/pediy50405.htm)

VB 黑客程序的暴破(修改)一例

好久没有制造垃圾了，昨晚和网友拼火，无意中找到个有趣的东东名曰：独裁者 `Autocrat` `DDos` client 版本：`v1.26.59`。为了让它更有趣，就产生了这篇破破文。。。

- 软件名称：独裁者 `Autocrat` `DDos` client
- 软件版本：`v1.26.59`
- 运行平台：`windows98`/`2000`/`xp`
- 软件作用：差点忘了说，看名字就知道了，这是在网络上进行 `DDos` 攻击用的，真的很有趣。。。呵呵。。。
- 破解目的：软件的主机列表框中，最多只允许添加十台主机（威力不够），老大大一个文本框，多浪费，所以目的是解除最多只能添加十台主机的限制。
- 声明：本文的目的只是想说明正确设断对暴破 VB 程序的重要性，如果哪里说错了，请各位指出。
- 下载地址：`ftp://eyy.vicp.net/bluearc/Client.rar`

哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪 BEGIN 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪

1. 用 `peid` `0.8` 打开程序文件：`Client.exe`。显示：`VB6.0`，说明没加壳。。。暗喜。。。

2. 用 `w32dasmgold` 反汇编。

3. 在 `w32dasmgold` 的界面使用串式参考查看，唉，真失望。果然是乱七八糟的一片。。。乱码迷人眼。。。面对茫茫码海，字符串参考又看不出什么名堂。。。到哪找我要的东东呀。。。僵尸。。。哦不，是僵局。

4. 经过了一番思索，初步列出了破解思路：在程序中添加主机时，如果超过十台，则程序会弹出“已经超过主机上限，禁止添加！”的消息框。所以应该道先从弹出消息框的地方着手。。。想到这里。。。我立即用 `CTRL+D` 呼出等候多时的 `Softice`。。。。

5. 设断：`bpx messageboxa` ..... 打开主程序--添加主机---直到弹出消息框。。。可是却什么也拦不到。。。随后我又试了有关拦截消息框和对话框的几个常用断点，包括 `messageboxexa`、`messageboxindirecta`、`dialogbox` 等。。。都是一无所获。。。正在纳闷时，我突然想起：这不是 VB 程序吗？VB 程序不是有几个专用断点的嘛。。。想到这里，我立即呼出 `Softice`，设断：`bpx msvbvm60!rtcmsgbox`。按下“添加”，没有拦住，但是当我再次按下“添加”时。。。程序被挡，`bc*` ... 当我企图按 `F12` 回到程序的领空时，“当！”。。。“已经超过主机上限，禁止添加！”。。。。不过别急，此时你按下对话框上的确定键后又自动回到了 `Softice` 的控制之中。。。按 `CTRL+` 向上箭头。。。回到了下面的地方：

```
。。。。。。。
0167:004109E9 LEA       EAX,[EBP-4C]                                                                                
0167:004109EC PUSH      EDX                                                                                       
0167:004109ED PUSH      EAX                                                                                       
0167:004109EE LEA       ECX,[EBP-3C]                                                                                
0167:004109F1 PUSH      30                                                                                    
0167:004109F3 PUSH      ECX                                                                                       
0167:004109F4 CALL      [MSVBVM60!rtcMsgBox]               注意这里！  “已经超过主机上限，
0167:004109FA LEA       EDX,[EBP-6C]                                                                            
0167:004109FD LEA       EAX,[EBP-5C]                                                                                
0167:00410A00 PUSH      EDX                                                                                       
0167:00410A01 LEA       ECX,[EBP-4C]                                                                            
0167:00410A04 PUSH      EAX                                                                                       
0167:00410A05 LEA       EDX,[EBP-3C]                                                                            
0167:00410A08 PUSH      ECX                                                                                       
0167:00410A09 PUSH      EDX                                                                                       
0167:00410A0A PUSH      04                                                                                        
0167:00410A0B CALL      EBX                                                                                       
```

呵呵，真是天助。这里正是我想找的地方。。。 `0167:004109F4 CALL [MSVBVM60!rtcMsgBox]` 这里就是弹出限制提示的地方，现在要做的就是找到调用这段子程序的地方，然后。。。呵呵。。。

6. 用 `w32dasmgold` 打开程序文件，使用转到代码位置功能，来到 `004109F4` 的地方。

```
。。。。。。。。。。。。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:00410470(C)                                                                                                        
* Reference To: MSVBVM60.__vbaVarDup, Ord:0000h

:00410988 8B3558114000            mov esi, dword ptr [00401158]
:0041098E B904000280             mov ecx, 80020004
:00410993 894D9C                 mov dword ptr [ebp-64], ecx
:00410996 B80A000000             mov eax, 0000000A
:0041099B 894DAC                 mov dword ptr [ebp-54], ecx
:0041099E 8D9544FFFFFF           lea edx, dword ptr [ebp+FFFFFF44]
:004109A4 8D4DB4                 lea ecx, dword ptr [ebp-4C]
:004109A7 894594                 mov dword ptr [ebp-6C], eax
:004109AA 8945A4                 mov dword ptr [ebp-5C], eax

* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h

:004109F4 FF1574104000           Call dword ptr [00401074]-------------------就这里了，nop 掉不会弹出。
:004109FA 8D5594                 lea edx, dword ptr [ebp-6C]                                                       
:004109FD 8D45A4                 lea eax, dword ptr [ebp-5C]
:00410A00 52                     push edx
:00410A01 8D4DB4                 lea ecx, dword ptr [ebp-4C]
:00410A04 50                     push eax
:00410A05 8D55C4                 lea edx, dword ptr [ebp-3C]
:00410A08 51                     push ecx
:00410A09 52                     push edx
:00410A0A 6A04                   push 00000004
```

跳到 `004109F4` 后，再使用查找文本功能，向上查找 `00410470`，就来到了调用这段子程序的地方：

```
。。。。。。。。

* Reference To: MSVBVM60.__vbaFreeObj, Ord:0000h

:00410467 FF1594114000           Call dword ptr [00401194]
:0041046D 663BF3                 cmp si, bx        <-----------------程序文本框内信息的行数是否等于 10
:00410470 0F8412050000           je 00410988 <-----------------等于则弹出限制提示。
```

看来这里就是关键之处了，马上用 `hiew` 打开程序文件，改 `00410470` `0F8412050000 je 00410988` 为 `00410470` `0F8012050000 j0 00410988`。（为什么要将上面的 `00410470 0F8412050000` 改为 `00410470 0F8012050000` 而不是 `00410470 0F8512050000` 或是 nop 掉呢？这里有几个小问题，1、如果改为 `00410470 0F8512050000`，即文本行数不等于 10 则跳转到弹出限制的地方，那么如果程序文本框的 `IPf` 地址的行数只要小于或不等于 10 行都要报错。2、如果改为 nop, 那么要好多 90，并且这样改的话肯定出错，为什么出错我也不知道，请高手指点。3、改为 `00410470 0F8012050000` 呢？即溢出转移。也许朋友们会问了，这不是风马牛不相关吗？是啊，笨人想笨办法嘛。。。这里就是要让它不相关，让它永远都不可能转到 `004109F4 FF1574104000 Call dword ptr [00401074]` 的地方去），按 `F9` 存盘。打开程序试运行，按下添加主机按钮，已经不会弹出“已经超过主机上限，禁止添加！”的提示了，不过先别高兴的太早，当你往弹出的输入框里随便输入 `IP` 地址按确定时，你会发现，虽然程序没有报错，但文本框中的 `IP` 地址行数还是 10 行，仍然不能添加。此程序的破解工作一度陷入僵尸，哦不，僵局。

7. 又经过了一番思索，想到了一此东西：程序检验行数是否已经达到十行，如果到了你还要添加的话则先弹出错误提示框，然后不进行操作。想到这里，立即对可能的情况作了一些模拟，其中最有可能的情况 VB 简单模拟应该是：

```
Dim b
b = 文本框中用户实际添加的文本行数

Private Sub 添加按钮_Click()
    if b = 10 then ' 判断是否达到上限 10
        msgbox " 已经超过主机上限，禁止添加！"
        exit sub ' 什么也不做
    else
        b = b + 1 ' 这里是代替一下，意思是允许向文本框继续写入 IP 地址，具体实现与本案无关
    end if
End Sub
```

分析：假如真是这样的话，那么在程序作出判断时是一定会产生内存拷贝动作的。。。

8. 打开主程序，点击添加按钮，输入任意 `IP`，然后按 `CTRL+D` 呼出 `Softice`，下万能断点 `bpx hmemcpy` ----- `F5` ----- 单击确定。程序马上被拦，我不知耐心地按了多少次 `F12` 后来到了下面的地方（即程序领空）：

```
。。。。。。。。。。。。。。。
0167:00410507 PUSH      EDX                                                                                       
0167:00410508 LEA       ECX,[EBP-3C]                                                                                
0167:0041050B PUSH      EAX                                                                                       
0167:0041050C PUSH      ECX                                                                                       
0167:0041050D CALL      [MSVBVM60!rtcInputBox]                                                                    
0167:00410513 MOV       ESI,[MSVBVM60!__vbaStrMove]                                                                
0167:00410519 MOV       EDX,EAX                                                                                   
0167:0041051B LEA       ECX,[EBP-18]                                                                                
0167:0041051E CALL      ESI                                                                                       
```

9. 一路 `F10` 一路看。。。不知不觉走了下来，突然我眼前一亮，不就是这里吗？

```
。。。。。。。。。。。。。
0167:004105BD LEA       ECX,[EBP-28]                                                                                
0167:004105C0 MOV       [EBP-011C],AX                                                                                
0167:004105C7 CALL      [MSVBVM60!__vbaFreeObj]                                                                       
0167:004105CD CMP       WORD PTR [EBP-011C],00                                                                        
0167:004105D5 JE        004108B0                                                                    
0167:004105DB LEA       EDX,[EBP-00AC]                                                                                
0167:004105E1 LEA       ECX,[EBP-3C]                                                                                
0167:004105E4 MOV       DWORD PTR [EBP-00A4],00407FC4                                                                    
0167:004105EE MOV       DWORD PTR [EBP-00AC],00000008                                                                    
0167:004105F8 CALL      EBX
```

不用多说了吧。。。马上用 `hiew` 打开程序文件，改 `0167:004105D5 JO 004108B0`。这里也是和前面一样，只能将等于转移改为溢出转移即将 `0F84D5020000` 改为 `0F80D5020000`。

看来这个推断来八九不离十。。。呵呵。。。

10. 整个世界清静了。。。不信你试试。

11. 总结：经过此次破解经过，得出了以下结论：只要解得其法，VB 程序就一点也不可怕 :)

12. 谢谢观看

哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪 THE END 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪

~~~~~转载请保持完整性~~~~~

作者：飞龙狗狗 `email:bluearc_arc@hotmail.com` `QQ:40116000`

主页：`http://www.bleuarc.net`

这是一个 `vb6` native 编译程序。用 `wdasm` 反汇编结合 `vbde` 辅助定位，知道：

自：`00410390` 开始的代码为“添加”按钮 `_Click` 事件。



```
...

:004103D5 33DB                          xor ebx, ebx              //ebx==0

...

...

:00410411 899DF0FEFFFF                  mov dword ptr [ebp-110], ebx

// 在现代编译器生成的代码中，dword ptr [ebp-110]指的是一局部变量。

// 后文可以知道其实际指的就是 `list1.listcount`

// 此处是使此变量清零。

:00410417 FF92DC030000                  call dword ptr [edx+000003DC]

:0041041D 50                            push eax

:0041041E 8D45D8                        lea eax, dword ptr [ebp-28]

:00410421 50                            push eax

* Reference To: `MSVBVM60.__vbaObjSet`, Ord:0000h

                                   |

:00410422 FF156C104000                  Call dword ptr [0040106C]

:00410428 8BF0                          mov esi, eax

:0041042A 8D95F0FEFFFF                  lea edx, dword ptr [ebp-110]

:00410430 52                            push edx

// push `List1.listcount`

:00410431 56                            push esi

:00410432 8B0E                          mov ecx, dword ptr [esi]

:00410434 FF91D8000000                  call dword ptr [ecx+000000D8]

// call get `list1.listcount`

:0041043A 3BC3                          cmp eax, ebx

:0041043C DBE2                          fclex

:0041043E 7D12                          jge 00410452

:00410440 68D8000000                    push 000000D8

* Possible StringData Ref from Code Obj ->"O?檉??"

                                   |

:00410445 68C47E4000                    push 00407EC4

// 指向 `UUID{33AD4F11-6699-11CF-B70C-00AA0060D393}`，即 `listbox1`

:0041044A 56                            push esi

:0041044B 50                            push eax

* Reference To: `MSVBVM60.__vbaHresultCheckObj`, Ord:0000h

                                   |

:0041044C FF1554104000                  Call dword ptr [00401054]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041043E(C)

|

:00410452 33C0                          xor eax, eax

:00410454 6683BDF0FEFFFF09              cmp word ptr [ebp-110], 0009

// cmp `list1.listcount`, 09

:0041045C 8D4DD8                        lea ecx, dword ptr [ebp-28]

:0041045F 0F9EC0                        setle al

// Is less ?

:00410462 F7D8                          neg eax

:00410464 668BF0                        mov si, ax

// 结果传递给 si，比较

* Reference To: `MSVBVM60.__vbaFreeObj`, Ord:0000h

                                   |

:00410467 FF1594114000                  Call dword ptr [00401194]

:0041046D 663BF3                        cmp si, bx

:00410470 0F8412050000                  je 00410988
```

所以，我喜欢改 `:0041042A 8D95F0FEFFFF lea edx, dword ptr [ebp-110]`

为 `:0041042A 8D95F2FEFFFF lea edx, dword ptr [ebp-10E]`

...

...

同样，还有一处同样对比，也是同样改法。

```
:00410563 8D8DF0FEFFFF                  lea ecx, dword ptr [ebp+FFFFFEF0]

:00410569 51                            push ecx

:0041056A 50                            push eax

:0041056B 8985ECFEFFFF                  mov dword ptr [ebp+FFFFFEEC], eax

:00410571 FF92D8000000                  call dword ptr [edx+000000D8]

...

:004105AE 6683BDF0FEFFFF09              cmp word ptr [ebp+FFFFFEF0], 0009

:004105B6 0F9EC1                        setle cl

:004105B9 F7D9                          neg ecx
```

`4dd466d0e1e0cc123afbe504da2462ca`
