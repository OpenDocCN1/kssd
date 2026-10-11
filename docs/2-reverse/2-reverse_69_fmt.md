
# 关于《OllyDBG 入门系列（五）－消息断点及 RUN 跟踪》的补充

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-182.htm`](https://www.pediy.com/kssd/pediy09/pediy09-182.htm)
>
> 作者: hklzt
>
> 时间: 2007-06-06,16:50
>
> 联系: QQ:87784858

看完了《OllyDBG 入门系列(五)-消息断点及 RUN 跟踪》之后感觉如何？会否有如下问题：

1. 是否觉得不知道在哪下断？
2. 为什么要这样子下断？
3. 如何确定断下来后的位置就是正确的？

好，就本着这几个问题来分析一遍。

首先，先回顾下 Windows 的消息机制。要点：所有要处理的消息必然会由程序自己处理，不处理的消息都交由 Windows 处理。Windows 的消息处理函数的格式，如下：

```c
LRESULT CALLBACK WindowProc(
    HWND hwnd,  // handle of window
    UINT uMsg,  // message identifier
    WPARAM wParam,  // first message parameter
    LPARAM lParam   // second message parameter
);
```

其中 `uMsg` 就是关键，它代表消息的类型，如：`WM_COMMAND`, `WM_GETTEXT` 等。记好哦。

下面，我们来用实例讲解。

![](img/82702640fa02786250999fc26e0a097b.jpg)

这个 CrackMe 是用对话框做的（你是怎么知道的？）。

1. 用 OllyDbg 加载，`Ctrl + N`，找到函数：`USER32.DialogBoxParamA`，右击->“在每个参考上设置断点”。
2. `F9`，运行。看，被拦了下来，如图：

![](img/d991037b208fb56346b2a7b80b63e0c7.jpg)

3. 其中 `DlgProc` 的内容，就是我们需要找的东东。这个地址是消息处理函数的入口点。现在来解释为什么要在窗口那才有消息断点，看图，

![](img/11b03d5e8dd5f585528b0fada8b4068c.jpg)

要下消息断点，首先得找到具有 Windows 消息处理函数格式的函数，然后，再根据栈的数据来判断消息，如果符合下断的消息，那么，OllyDbg 就会拦下来（还会觉得不知道在哪下消息断点了吗？知道该如何下消息断点了吧？）。很明显有一点，这个消息断点的功能是有限的，比如，要拦主窗口中的菜单消息呢？所以，消息断点的功能还是有限的。如何扩展？扯远了，下面再讲。

4. `Ctrl + G` 来到 `cycle.0041029`，我们来到了消息处理函数了，但是，OllyDbg 并未识别出这个函数是消息处理函数。所以，在 `cycle.0041029` 处，右击->“分析”->“假定参数”，如图：

![](img/ac9b215c746e0e164c45498765a2a046.jpg)

弹出一个对话框，选择“`WinProc(hWnd,msg,wParam,lParam)`”。

![](img/19523814b2b64bb043b2df911671b10c.jpg)

点击“应用”后，如图。

![](img/2a018ece35952eec179979c7b3e8bf25.jpg)

5. 在 `cycle.0041029` 处，右击->“断点”->“在 `WinProc` 上的消息断点”(平时是不是没见过这个菜单呀？呵呵)。

![](img/e2092977bcb6ee16eea2f42ad8785357.jpg)

6. 在“消息”列表框中，选择你要下断的消息（`Alt+B`，删除以前的断点，以免影响心情）。

![](img/d06af0c70f28ddffd27cb240547066ba.jpg)

7. `F9`，运行，程序运行起来了。这时没什么事情发生，当你在里面点了一个文本框之后（点它，是想让它获得 `Focus`(焦点)，以便能够输入数据），事情就发生了。现在没办法在文本框里输入注册码，也没办法点击按钮。这是怎么回事呢？仔细看一下 `Stack`(栈)的那个窗口，噢！原来被文本框的通知 `EN_SETFOCUS` 搞坏了！停留在 `EN_SETFOCUS` 和 `EN_KILLFOCUS` 两个消息之间了。

8. 好，现在来扩展消息断点（消息断点是否是条件断点的特殊例子？），即使用条件断点（卖弄了一下，呵呵）。

![](img/9ecd11fa7d081ec3ad3b5ab8e01f36fe.jpg)

看你需求，条件和条件记录，在这里是没什么区别的，因为不需要记录的内容。

9. 按 `Shift+F2`，输入 `MSG ==WM_COMMAND && [ESP+C]==66`（这个 66 是怎么来的？这个就是那个 Check 按钮的 ID，意思就是“当收到 `WM_COMMAND`,且是由 ID 为 0x66 发出的时候中断”），确定，`F9`，运行。

10. 现在，输入 `Name` 和 `Serial` 之后，点”Check”按钮。

11. 这次中断，位置上似乎没有变化，但是，明白了消息处理机制，应该知道这次中断的不同吧？（这次流程会流到注册算法那哦）

```asm
00401029  >/.  >enter   0, 0                            ;  解码为 <WinProc>
0040102D  |.  >push    ebx
0040102E  |.  >push    edi
0040102F  |.  >push    esi
00401030  |.  >cmp     [arg.2], 110
00401037  |.  >je      short cycle.0040105E
00401039  |.  >cmp     [arg.2], 111  //111=WM_COMMAND
00401040  |.  >je      short cycle.00401082 //肯定在这里跳（为什么？）
00401042  |.  >cmp     [arg.2], 10
00401046  |.  >je      short cycle.00401057
00401048  |.  >cmp     [arg.2], 2
0040104C  |.  >je      short cycle.00401057
0040104E  |.  >xor     eax, eax
```

12. 来到 `cycle.00401082`，

```asm
00401082  |> \>cmp     [arg.3], 67
00401086  |.  >jnz     short cycle.0040108D  //这里肯定会跳
00401088  |.  >call    cycle.00401151
0040108D  |>  >cmp     [arg.3], 66          
00401091  |.  >jnz     short cycle.00401098   //这里肯定不会跳（又是为什么呢？）
00401093  |.  >call    cycle.0040109C          //关键，
00401098  |>  >xor     eax, eax
```

13. 蓝色部分，已经在 CCDebuger 那篇文章分析过了，这里就不分析了，我用红色字体来标明重点。

```asm
0040109C  /$  >mov     dword ptr ds:[402182], FEDCBA98
004010A6  |.  >push    11                             ; /Count = 11 (17.)
004010A8  |.  >push    cycle.00402171                 ; |Buffer = cycle.00402171
004010AD  |.  >push    3E9                            ; |ControlID = 3E9 (1001.)
004010B2  |.  >push    [arg.1]                        ; |hWnd
004010B5  |.  >call    <jmp.&USER32.GetDlgItemTextA>  ; \\GetDlgItemTextA（Serial）
004010BA  |.  >or      eax, eax
004010BC  |.  >je      short cycle.0040111F
004010BE  |.  >push    11                             ; /Count = 11 (17.)
004010C0  |.  >push    cycle.00402160                 ; |Buffer = cycle.00402160
004010C5  |.  >push    3E8                            ; |ControlID = 3E8 (1000.)
004010CA  |.  >push    [arg.1]                        ; |hWnd
004010CD  |.  >call    <jmp.&USER32.GetDlgItemTextA>  ; \\GetDlgItemTextA（name）
004010D2  |.  >or      eax, eax
004010D4  |.  >je      short cycle.0040111F
004010D6  |.  >mov     ecx, 10
004010DB  |.  >sub     ecx, eax
004010DD  |.  >mov     esi, cycle.00402160
004010E2  |.  >mov     edi, esi
004010E4  |.  >add     edi, eax
004010E6  |.  >cld
004010E7  |.  >rep     movs byte ptr es:[edi], byte ptr>
004010E9  |.  >xor     ecx, ecx
004010EB  |.  >mov     esi, cycle.00402171
004010F0  |>  />inc     ecx
004010F1  |.  >|lods    byte ptr ds:[esi]
004010F2  |.  >|or      al, al
004010F4  |.  >|je      short cycle.00401100
004010F6  |.  >|cmp     al, 7E
004010F8  |.  >|jg      short cycle.00401100
004010FA  |.  >|cmp     al, 30
004010FC  |.  >|jb      short cycle.00401100
004010FE  |.^  >\jmp     short cycle.004010F0
00401100  |>  >cmp     ecx, 11       
00401103  |.  >jnz     short cycle.0040111F   //判断长度是否为 16 个有效字符，即 16 个字节，不是则跳
00401105  |.  >call    cycle.004011F1         //算法
0040110A  |.  >mov     ecx, 0FF01
0040110F  |.  >push    ecx
00401110  |.  >call    cycle.00401190       //算法
00401115  |.  >cmp     ecx, 1       
00401118  |.  >je      short cycle.00401120 //需要跳
0040111A  |>  >call    cycle.00401166
0040111F  |>  >retn
00401120  |>  >mov     eax, dword ptr ds:[402168]
00401125  |.  >mov     ebx, dword ptr ds:[40216C]
0040112B  |.  >xor     eax, ebx
0040112D  |.  >xor     eax, dword ptr ds:[402182]
00401133  |.  >or      eax, 40404040
00401138  |.  >and     eax, 77777777
0040113D  |.  >xor     eax, dword ptr ds:[402179]
00401143  |.  >xor     eax, dword ptr ds:[40217D]
00401149  |.^  >jnz     short cycle.0040111A  //不可以跳
0040114B  |.  >call    cycle.0040117B       //提示注册成功！
00401150  \.  >retn
```

终于写完了，现在来回顾下刚开始的问题：

1. 是否觉得不知道在哪下断？
   答：在 Windows 消息处理函数的入口处下消息断点。
2. 为什么要这样子下断？
   答：可能 OllyDbg 是根据栈的数据和函数原型来匹配，所以，一般来说，匹配条件都会是`[Esp + XX] ==XXXXX`
3. 如何确定断下来后的位置就是正确的？
   答：这里是根据编程的思路以及 Windows 的消息处理机制来定位的，理论与实战相结合。

最后，总结下，由于 windows 是消息驱动的，很大一部分都是通过消息来完成的，所以，有很大一部分可以通过对消息下断来达到目的，但是，如何下消息断点？从大体上讲，是这样子的：1、找出消息循环处理的函数 2、在消息循环处理函数的入口处设断

写到这里，废话一下。赞扬 CCDebuger 的太多了，但是，在赞扬的同时，不知道大家有没仔细消化人家的成果？呵呵，其实，我也没有，因为我看不太懂，所以，还是照着自己的思路走。写这篇文章的目的是为了帮一位朋友，他想下消息断点，但是，不知道如何下，我就把 CCDebuger 的这那篇消息断点给他，可是，还是没解决，后来，自己也动了一下手，确实，对于 `WM_COMMAND` 消息来说，OD 肯定会不停的拦下来的，所以，单纯的消息断点就行不通了，所以，再结合 Run 跟踪来记录下，刚好能解决问题，也就产生了 CCDebuger 的那篇文章（猜的，呵呵）。

最后，帮忙纠正下错误：

引用：

> 写到这准备跟踪算法时，才发现这个 crackme 还是挺复杂的，具体算法我就不写了，实在没那么多时间详细跟踪。有兴趣的可以跟一下，注册码是 17 位，用户名采用复制的方式扩展到 16 位，如我输入“CCDebuger”，扩展后就是“CCDebugerCCDebug”。大致是先取扩展后用户名的前 8 位和注册码的前 8 位，把用户名的前四位和后四位分别与注册码的前四位和后四位进行运算，算完后再把扩展后用户名的后 8 位和注册码的后 8 位分两部分，再与前面用户名和注册码的前 8 位计算后的值进行异或计算，最后结果等于 0 就成功。注册码的第 17 位我尚未发现有何用处。对于新手来说，可能这个 crackme 的难度大了一点。没关系，我们主要是学习 OllyDBG 的使用，方法掌握就可以了。

关于“位”的概念，“位”，是指二进制位，在这里，一个字节等于 8 位，一个字符等于一个字节。“注册码是 17 位”，应改成“注册码是 17 个字节”，如果你跟踪分析过，你可以发现，这样子还是不对的，最后应该是“注册码是 16 个字节”，

`cmp ecx, 11`，这里的 `11` 是 16 进制，即十进制数：17。从代码中可以看出这个十进制数 17，还得减 1 才是字符串的真实长度，所以，应该改成“注册码是 16 个字节”，后面的“位”，需要改成“字节”。第一次看的时候，没注意看，都给蒙了。
