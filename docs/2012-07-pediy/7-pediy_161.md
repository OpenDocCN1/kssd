
# W32Dasm 8.93 Patch 1.0 的搜索功能

> 原文：[`www.pediy.com/kssd/pediy05/pediy50253.htm`](https://www.pediy.com/kssd/pediy05/pediy50253.htm)

看到 `kanxun` 发布的这个搜索版，我测试了一下，所有补丁的动作都在 `w32dsm.dll` 里面，更改了程序的入口使其一开始就 `loadlibary` `w32dsm.dll`，然后 `w32dsm.dll` 会修改原程序相关的指令，主要与弹出搜索 dialog 的 api 是 `createdialogparam`，加载 `w32dsm.dll` 和没有加载，其消息函数的地址，和显示的资源都不一样，这个有搜索 edit，和 button 的 dialog 资源在 `w32dsm.dll` 里面，当而且 `w32dsm.dll` 还有专门处理搜索的消息函数，其他处理函数和原来程序函数功能一致没有什么变化。

我现在将这个搜索消息函数分析一下，给有兴趣把 killer 版 10 的 `w32asm` 也加上这个功能的朋友做参考，希望大家迎难而上，多多实践。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:10002298(C)
|

:100022AD 3D11010000                   cmp eax, 00000111==>比较消息是否是 wm_command
:100022B2 7525                         jne 100022D9===>不是完蛋
:100022B4 8B4510                       mov eax, dword ptr [ebp+10]
:100022B7 663D6318                     cmp ax, 1863＝＝>比较子参量是否点击 search 按钮
:100022BB 7518                         jne 100022D5
:100022BD 6860180000                   push 00001860
:100022C2 68FF550000                   push 000055FF
:100022C7 FF7508                       push [ebp+08]
:100022CA E89F020000                   call 1000256E==>做 search 的功能
:100022CF C9                           leave
:100022D0 C21000                       ret 0010
```

call 到这里

```
:1000256E 55                           push ebp
:1000256F 8BEC                         mov ebp, esp
:10002571 FF750C                       push [ebp+0C]===>identifier of control
:10002574 FF7508                       push [ebp+08]==> handle of dialog box

* Reference To: USER32.GetDlgItem, Ord:0100h

                                                        |

:10002577 E838050000                     Call 10002AB4
:1000257C A310660010                     mov dword ptr [10006610], eax==>这样会得到 search 文本框的 handle
:10002581 6800020000                     push 00000200==>最大字符支持 512 个
:10002586 68EC680010                     push 100068EC===>拷贝你要搜索的字符到这个 buffer
:1000258B FF7510                         push [ebp+10]==>identifier of control
:1000258E FF7508                         push [ebp+08]==>handle of dialog box

* Reference To: USER32.GetDlgItemTextA, Ord:0102h

                                                        |

:10002591 E824050000                     Call 10002ABA＝＝》这个函数大家都知道干什么
:10002596 0BC0                           or eax, eax
:10002598 7506                           jne 100025A0==>判断是否有字符拷贝，没有就不搜索了
:1000259A 33C0                           xor eax, eax
:1000259C C9                             leave
:1000259D C20C00                         ret 000C
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:10002598(C)
|

:100025A0 6A00                           push 00000000
:100025A2 6A00                           push 00000000
:100025A4 688B010000                     push 0000018B===>发送一个 LB_GETCOUNT 消息给下面的 list 框，需要得到有多少个需要比较
:100025A9 FF3510660010                   push dword ptr [10006610]

* Reference To: USER32.SendMessageA, Ord:0210h

                                                        |

:100025AF E830050000                     Call 10002AE4
:100025B4 A30C660010                     mov dword ptr [1000660C], eax==>返回 list 列表的个数到这个地址
:100025B9 6A00                           push 00000000
:100025BB 6A00                           push 00000000
:100025BD 6888010000                     push 00000188==>发送 LB_GETCURSEL 消息给 list 筐,目的是得到现在选中字串的 index
:100025C2 FF3510660010                   push dword ptr [10006610]

* Reference To: USER32.SendMessageA, Ord:0210h

                                                        |

:100025C8 E817050000                     Call 10002AE4
:100025CD 83F8FF                         cmp eax, FFFFFFFF＝＝》比较是否有没有选中 list 框中的某个字串，如果有的话返回的是选中的字串 index,没有选中返回 ffffffff(-1)
:100025D0 7502                           jne 100025D4==>有字窜选中,跳到 100025D4
:100025D2 33C0                           xor eax, eax
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:100025D0(C)
|

:100025D4 A308660010                     mov dword ptr [10006608], eax==>给这个变量清零,这个变量是代表搜索的次数
:100025D9 A3F8650010                     mov dword ptr [100065F8], eax==>给这个变量清零,这个变量其实是表示你需要搜索字串的实际 index
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:10002664(U)
|

:100025DE FF0508660010                   inc dword ptr [10006608]＝＝》搜索的 index 加 1
:100025E4 A108660010                     mov eax, dword ptr [10006608]
:100025E9 3B050C660010                   cmp eax, dword ptr [1000660C]==>和 listist 列表的个数比较，看是否已经全部搜索晚所有的列出的字串
:100025EF 7507                           jne 100025F8==>没有搜索完，继续搜索
:100025F1 33C0                           xor eax, eax
:100025F3 A308660010                     mov dword ptr [10006608], eax==>已经搜索到最后的 list 了，把搜索的 index 清空，准备重头搜索
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:100025EF(C)
|

:100025F8 3B05F8650010                   cmp eax, dword ptr [100065F8]==>比较是否和实际搜索到的 index 相等
:100025FE 7519                           jne 10002619==>不等跳到 10002619
:10002600 6A00                           push 00000000
:10002602 50                             push eax====>相等,搜索到字传的 index
:10002603 6886010000                     push 00000186==>发一个 LB_SETCURSEL 消息给 list 框，然后返回，这样那条蓝条就会停在搜索到的字串上
:10002608 FF3510660010                   push dword ptr [10006610]

* Reference To: USER32.SendMessageA, Ord:0210h

                                                        |

:1000260E E8D1040000                     Call 10002AE4
:10002613 33C0                           xor eax, eax
:10002615 C9                             leave
:10002616 C20C00                         ret 000C===>已经搜索到需要找得字串，返回
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:100025FE(C)比较是否和实际搜索到的 index 不相等，跳过来的
|

:10002619 68EC6A0010                     push 10006AEC＝＝》得到的字串放到这个变量里面
:1000261E 50                             push eax==>这里是现在搜索到那个字串的 index 了
:1000261F 6889010000                     push 00000189==>发送 LB_GETTEXT 消息给 list 框,告诉它我们现在搜索到 eax 值的 index，list 把这个 index 对应的字串放到 10006AEC 这个地址里去，我们等下好比较是否 GetDlgItemTextA 得到搜索框的字串和这个地址的是否相等
:10002624 FF3510660010                   push dword ptr [10006610]

* Reference To: USER32.SendMessageA, Ord:0210h

                                                        |

:1000262A E8B5040000                     Call 10002AE4
:1000262F 0BC0                           or eax, eax==如果成功的话 eax 是拷贝字串的数目
:10002631 7431                           je 10002664＝＝》失败就跳到比较下一条 index
:10002633 68EC680010                     push 100068EC===>这个是我们现在 index 字串的地址
:10002638 68EC6A0010                     push 10006AEC==>这个是 GetDlgItemTextA 得到字串保存的地址
:1000263D E821030000                     call 10002963===>想都不要想就知道这个 call 是比较上面两个字窜是否相等
:10002642 0BC0                           or eax, eax
:10002644 741E                           je 10002664＝＝》不相等跳到比较下一条 index
:10002646 6A00                           push 00000000
:10002648 FF3508660010                   push dword ptr [10006608]===>现在 index 和 GetDlgItemTextA 得到字串相等，把那个蓝条放到这个 index 上
:1000264E 6886010000                     push 00000186＝＝》LB_SETCURSEL
:10002653 FF3510660010                   push dword ptr [10006610]

* Reference To: USER32.SendMessageA, Ord:0210h

                                                        |

:10002659 E886040000                     Call 10002AE4
:1000265E 33C0                           xor eax, eax
:10002660 C9                             leave
:10002661 C20C00                         ret 000C==>找到一个，比较完成，返回
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

```
|:10002631(C), :10002644(C)
|

:10002664 E975FFFFFF                     jmp 100025DE===>跳到把 [10006608] 的值加 1，比较下一个 index 是否相等
```

好了，怎么搜索的原理我已经分析玩了，怎么 diy 这个 pe，我还有有些想法的，到了 killer 的 10 版以后 `creatdialog` 这些功能都在 `w32path` 这个 dll 里面了，如果补丁的话，需要做的事情很多，首先更改资源，增加一个 edit 控件，一个 button，增加捕获这个事件的功能，然后参照上面的程序，做相应的动作，需要的工作量太大了（如果有毅力有横心还是能完成的：）），我现在没有足够的时间来干这个工作，那个朋友有兴趣，可以试试，做不下去的话可以 email 质询我，我尽力帮忙，造福大家的事情就靠你们了，：）

### 前言

看雪提供了一个老外作的 `w32asm`，带搜索这个很实用的功能，不过是在 8.93 版上补丁的，很多 kill 版的实用功能都没有，我用着不习惯。

于是就自己在原来作拖放和滚动版本基础上再加上搜索功能。其实这篇文章没有什么新的东西，也就是把我前面几篇文章提到的技术全部用上，搞了个综合篇：）

说起来容易，做起来就.....

首先分析问题：

#### 第一、从资源的角度上分析



# `w32asm` 搜索功能实现

`w32asm` 一共有 `str`、`dlg`、`menu`、`code`、`data`、`exp`、`imp` 这七个 dialog 需要增加搜索功能。

增加搜索功能就是每个 dialog 需要增加一个 `edit` 控件，一个标题为 `search` 的 `button` 控件（方法见 `diype` 系列前面的文章，我不重复讲相同的东西）。

1. 首先把我这个拖放版的资源释放出来（具体细节与文章内容无关，略去）。
2. 打开资源黑客，找到 `str`、`dlg` 等这 7 个 dialog 的资源，每个 dialog 都增加一个 `edit` 控件（把这个 `edit` 的 `id` 设为 `1568`），再增加一个 `button`（每个 dialog 增加 `button` 的 `id` 都不同，我等一下列个表给大家），然后记下每个 `dialog` 的 `list` 控件的 `id`，最后调整资源界面，让它们好看一点。

## 从编程的角度上去分析

1. 资源添加好了，我们需要让它动起来，要做到和 `w32asm` 结合在一起，就需要找到 `w32asm` 的消息处理的进程，然后更改这个进程、加入我们需要实现的功能。在 Windows 系统里面，每个弹出的 dialog 都有自己的消息处理进程，和程序主消息处理进程不相干。也就是说我们上次作拖放功能的消息进程和作搜索功能的消息进程不是一个进程。现找到消息处理进程地址是 `4981e3`（怎么找消息进程的方法 `diype` 前面的文章有讲述），这 7 个 `dialog` 共享这个消息进程，这样很好，省了很多事情。

```asm
004981E3     . 55             PUSH EBP\===>c++参数传递
004981E4     . 8BEC           MOV EBP,ESP
004981E6     . 83C4 D8        ADD ESP,-28
004981E9     . 53             PUSH EBX
004981EA     . 56             PUSH ESI
004981EB     . 57             PUSH EDI
004981EC     . 8B75 0C        MOV ESI,DWORD PTR SS:[EBP+C]＝＝>[EBP+C]就是消息代码
004981EF     . 8B7D 08        MOV EDI,DWORD PTR SS:[EBP+8]＝＝>[EBP+8]接收消息窗口句柄
004981F2     . B8 E4D54C0     MOV EAX,kW32Dsm\_.004CD5E4
004981F7     . E8 0CFC000     CALL kW32Dsm\_.004A7E08
```

消息处理进程找到了，现在就加入我们的代码，更改如下：

```asm
004981E3     . 55             PUSH EBP
004981E4     . 8BEC           MOV EBP,ESP
004981E6     . 83C4 D8        ADD ESP,-28
004981E9     . 53             PUSH EBX
004981EA     . 56             PUSH ESI
004981EB     . 57             PUSH EDI
004981EC     . E9 9F720100    JMP pll621_9.004AF490＝＝>跳到我们自己的代码段
004981F1        90             NOP
004981F2     > B8 E4D54C00    MOV EAX,pll621_9.004CD5E4
```

`JMP pll621_9.004AF490` 自己的代码到这里：

```asm
004AF490     > 817D 0C 11010000   CMP DWORD PTR SS:[EBP+C],111＝＝>比较消息是否是 wm_command
004AF497     . 75 09              JNZ SHORT pll621_9.004AF4A2＝＝>不是恢复源程序动作，然后返回
004AF499     . E9 32020000        JMP pll621_9.004AF6D0＝＝＝>是就跳到比较子参量
004AF49E        90                 NOP
004AF49F        90                 NOP
004AF4A0        90                 NOP
004AF4A1        90                 NOP
004AF4A2     > 8B75 0C            MOV ESI,DWORD PTR SS:[EBP+C]＝＝>恢复源程序动作
004AF4A5     . 8B7D 08            MOV EDI,DWORD PTR SS:[EBP+8]
004AF4A8     .^E9 458DFEFF        JMP pll621_9.004981F2＝＝>调回原程序执行
```

现在我们就需要知道点击了哪个 dialog 的 `search` 按钮，判别就靠每个 `dialog` 的 `search` 按钮 `id` 是不同，我把我定义的按钮 `id` 和 `dialog`、和 `list` 控件 `id` 的值列表如下：

| dialog | str | dlg | menu | code | data | exp | imp |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **消息进程地址** | `4981e3` | `4981e3` | `4981e3` | `4981e3` | `4981e3` | `4981e3` | `4981e3` |
| **list 控件 id** | `55ff` | `5625` | `5624` | `5602` | `5603` | `55fe` | `55fd` |
| *(16 进制)* | | | | | | | |
| **增加搜索按钮 id** | `621` | `627` | `626` | `625` | `624` | `623` | `622` |
| *(16 进制）* | | | | | | | |
| *(转成 10 进制，因为资源黑客是用 10 进制）* | `1569` | `1575` | `1574` | `1573` | `1572` | `1571` | `1570` |

既然知道了每个 `dialog` 上面添加的 `search` 按钮的 `id` 都不同，那么我们就能为每个 `dialog` 作不同的处理了。

`JMP pll621_9.004AF6D0＝＝＝>是就跳到比较子参量跳到这里：`

```asm
004AF6D0     > 817D 10 21060000   CMP DWORD PTR SS:[EBP+10],621＝＝>比较是否点击了 str 的 search 按钮
004AF6D7     . 75 0A              JNZ SHORT pll621_9.004AF6E3
004AF6D9     . B8 FF550000        MOV EAX,55FF＝＝＝＝＝＝＝＝＝＝＝＝>点击了 str 的 search 按钮，那么保存 str 的 list 控件的 id 到 eax
004AF6DE     .^E9 D0FDFFFF        JMP pll621_9.004AF4B3＝＝>跳到搜索功能实现代码
004AF6E3     > 817D 10 22060000   CMP DWORD PTR SS:[EBP+10],622＝＝>比较是否点击了 imp 的 search 按钮
004AF6EA     . 75 0A              JNZ SHORT pll621_9.004AF6F6
004AF6EC     . B8 FD550000        MOV EAX,55FD＝＝＝＝＝＝＝＝＝＝＝＝>点击了 imp 的 search 按钮，那么保存 str 的 list 控件的 id 到 eax
004AF6F1     .^E9 BDFDFFFF        JMP pll621_9.004AF4B3＝＝>跳到搜索功能实现代码，以下代码功能类似，我就不一一写注释了
004AF6F6     > 817D 10 23060000   CMP DWORD PTR SS:[EBP+10],623
004AF6FD     . 75 0A              JNZ SHORT pll621_9.004AF709
004AF6FF     . B8 FE550000        MOV EAX,55FE
004AF704     .^E9 AAFDFFFF        JMP pll621_9.004AF4B3
004AF709     > 817D 10 24060000   CMP DWORD PTR SS:[EBP+10],624
004AF710     . 75 0A              JNZ SHORT pll621_9.004AF71C
004AF712     . B8 03560000        MOV EAX,5603
004AF717     .^E9 97FDFFFF        JMP pll621_9.004AF4B3
004AF71C     > 817D 10 25060000   CMP DWORD PTR SS:[EBP+10],625
004AF723     . 75 0A              JNZ SHORT pll621_9.004AF72F
004AF725     . B8 02560000        MOV EAX,5602
004AF72A     .^E9 84FDFFFF        JMP pll621_9.004AF4B3
004AF72F     > 817D 10 26060000   CMP DWORD PTR SS:[EBP+10],626
004AF736     . 75 0A              JNZ SHORT pll621_9.004AF742
004AF738     . B8 24560000        MOV EAX,5624
004AF73D     .^E9 71FDFFFF        JMP pll621_9.004AF4B3
004AF742     > 817D 10 27060000   CMP DWORD PTR SS:[EBP+10],627
004AF749     . 75 0A              JNZ SHORT pll621_9.004AF755
004AF74B     . B8 25560000        MOV EAX,5625
004AF750     .^E9 5EFDFFFF        JMP pll621_9.004AF4B3
004AF755     >^E9 48FDFFFF        JMP pll621_9.004AF4A2＝＝＝>没有点击我们添加的 search 按钮,返回原程序
```

这七个 `dialog` 的 `search` 按钮区分代码也写好了，那么下一步就需要实现搜索功能了。

我大致讲一下实现搜索功能原理：

1. 首先得到 `dialog` 里面 `list` 控件的 `handle`，使用 `GetDlgItem` 函数实现。
2. 得到我们 `edit` 控件里面的字符保存到一个地址（也就是我们要搜索的字符），使用 `GetDlgItemtexta` 实现。
3. 通过 `list` 控件的 `handle` 使用 `sendmessage` 函数得到 `list` 控件一共有多少条需要我们搜索。
4. 把我们需要搜索的字符和 `list` 控件里面字符，一条一条比较，如果比较到最后了，就从头比较（这个比较函数，还得自己写，Windows 是没有相关的 `api`）。
5. 如果 `list` 控件里面某一条包含我们需要搜索的字符，那么我们就认为找到了，就把那个兰条设置到找到的这条上。
6. 如果继续寻找就，把从当前位置往下，重复 4、5 部，直到全部搜索完毕，再重头搜索。
7. 所有对 `list` 的控制都靠 `list` 的 `handle` 和 `sendmessage` 函数实现。
8. 具体原理，看我昨天写的那篇对 `diype` 的一点想法文章。
9. 由于原程序没有上述的 `api` 函数、所以自己构造了 `GetDlgItem`、`GetDlgItemtexta`、`sendmessage` 三个函数(构造方法见 `diype3` 那篇文章）。

程序用到的变量声明：

* `4aff00`: `list` 控件的 `handle`
* `4aff04`: `list` 控件 `index` 的总数
* `4aff08`: 现在搜索到那一条 `index`
* `4aff0c`: 实际的 `index`
* `4aff10`: 作比较字符函数，保存原来的返回地址
* `4aff14`: 目的字符比较指针
* `4AFD00`: 源字符地址
* `4afc00`: 目的字符地址

这里是实现搜索功能了的代码：

```asm
004AF4B3     > 50             PUSH EAX                      ; /ControlID＝＝>每个 dialog 的 list 控件的 id 都不同，所以我用 eax 作变量
004AF4B4     . 90             NOP                           ; |
004AF4B5     . 90             NOP                           ; |
004AF4B6     . 90             NOP                           ; |
004AF4B7     . 90             NOP                           ; |
004AF4B8     . FF75 08        PUSH DWORD PTR SS:[EBP+8]     ; |hWnd
004AF4BB     . E8 96FFFFFF    CALL pll621_9.004AF456        ; \GetDlgItem
004AF4C0     . A3 00FF4A00    MOV DWORD PTR DS:[4AFF00],EAX＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝>这样 eax 会得到 list 的 handle
004AF4C5     . 68 00010000    PUSH 100                      ; /Count = 100 (256.)＝＝>最多只能搜索 512 个字符，记住不要超过了，否则非法超作不要找我
004AF4CA     . 68 00FD4A00    PUSH pll621_9.004AFD00        ; |Buffer = pll621_9.004AFD00
004AF4CF     . 68 20060000    PUSH 620                      ; |ControlID = 620 (1568.)
004AF4D4     . FF75 08        PUSH DWORD PTR SS:[EBP+8]     ; |hWnd
004AF4D7     . E8 74FFFFFF    CALL pll621_9.004AF450        ; \GetDlgItemTextA
004AF4DC     . 09C0           OR EAX,EAX
004AF4DE     . 75 05          JNZ SHORT pll621_9.004AF4E5
004AF4E0     . 33C0           XOR EAX,EAX
004AF4E2     .^EB BE          JMP SHORT pll621_9.004AF4A2
004AF4E4        90             NOP
004AF4E5     > 6A 00          PUSH 0                        ; /lParam = 0
004AF4E7     . 6A 00          PUSH 0                        ; |wParam = 0
004AF4E9     . 68 8B010000    PUSH 18B                      ; |Message = LB_GETCOUNT
004AF4EE     . FF35 00FF4A00  PUSH DWORD PTR DS:[4AFF00]    ; |hWnd = NULL
```



004AF4F4     . E8 63FFFFFF         CALL pll621_9.004AF45C       ; \SendMessageA

004AF4F9     . A3 04FF4A00         MOV DWORD PTR DS:[4AFF04],EAX

004AF4FE     . 6A 00               PUSH 0                         ; /lParam = 0

004AF500     . 6A 00               PUSH 0                         ; |wParam = 0

004AF502     . 68 88010000        PUSH 188                       ; |Message = LB_GETCURSEL

004AF507     . FF35 00FF4A00      PUSH DWORD PTR DS:[4AFF00]     ; |hWnd = NULL

004AF50D     . E8 4AFFFFFF        CALL pll621_9.004AF45C       ; \SendMessageA

004AF512     . 83F8 FF            CMP EAX,-1

004AF515     . 75 02              JNZ SHORT pll621_9.004AF519

004AF517     . 33C0               XOR EAX,EAX

004AF519     > A3 08FF4A00        MOV DWORD PTR DS:[4AFF08],EAX

004AF51E     . A3 0CFF4A00        MOV DWORD PTR DS:[4AFF0C],EAX

004AF523     > FF05 08FF4A00     INC DWORD PTR DS:[4AFF08]

004AF529     . A1 08FF4A00        MOV EAX,DWORD PTR DS:[4AFF08]

004AF52E     . 3B05 04FF4A00     CMP EAX,DWORD PTR DS:[4AFF04]

004AF534     . 75 07              JNZ SHORT pll621_9.004AF53D

004AF536     . 33C0               XOR EAX,EAX

004AF538     . A3 08FF4A00        MOV DWORD PTR DS:[4AFF08],EAX

004AF53D     > 3B05 0CFF4A00     CMP EAX,DWORD PTR DS:[4AFF0C]

004AF543     . 75 1A              JNZ SHORT pll621_9.004AF55F

004AF545     . 6A 00               PUSH 0                         ; /lParam = 0

004AF547     . 50                  PUSH EAX                       ; |wParam

004AF548     . 68 86010000        PUSH 186                       ; |Message = LB_SETCURSEL

004AF54D     . FF35 00FF4A00      PUSH DWORD PTR DS:[4AFF00]     ; |hWnd = NULL

004AF553     . E8 04FFFFFF        CALL pll621_9.004AF45C       ; \SendMessageA

004AF558     . 33C0               XOR EAX,EAX

004AF55A     .^E9 43FFFFFF        JMP pll621_9.004AF4A2

004AF55F     > 68 00FC4A00       PUSH pll621_9.004AFC00        ; /lParam = 4AFC00

004AF564     . 50                  PUSH EAX                       ; |wParam

004AF565     . 68 89010000        PUSH 189                       ; |Message = LB_GETTEXT

004AF56A     . FF35 00FF4A00      PUSH DWORD PTR DS:[4AFF00]     ; |hWnd = NULL

004AF570     . E8 E7FEFFFF        CALL pll621_9.004AF45C       ; \SendMessageA

004AF575     . 09C0               OR EAX,EAX

004AF577     . 74 32              JE SHORT pll621_9.004AF5AB

004AF579     . 68 00FD4A00       PUSH pll621_9.004AFD00＝＝＝＝＝＝＝＝＝》传递参数；源字符地址指针

004AF57E     . 68 00FC4A00       PUSH pll621_9.004AFC00＝＝＝＝＝＝＝＝＝》传递参数；目的字符指针

004AF583     . E8 65000000       CALL pll621_9.004AF5ED＝＝＝＝＝＝＝＝＝＝＝＝＝＝》这个 call 就是比较目的字符是否包含源字符

004AF588     . 85C0               TEST EAX,EAX＝＝＝＝＝＝＝＝＝＝＝＝＝》用 eax 作返回值，cracker 们熟悉这样的格式，我也喜欢这样写，呵呵

004AF58A     . 74 1F              JE SHORT pll621_9.004AF5AB

004AF58C     . 6A 00               PUSH 0                         ; /lParam = 0

004AF58E     . FF35 08FF4A00     PUSH DWORD PTR DS:[4AFF08]     ; |wParam = 0

004AF594     . 68 86010000       PUSH 186                       ; |Message = LB_SETCURSEL

004AF599     . FF35 00FF4A00     PUSH DWORD PTR DS:[4AFF00]     ; |hWnd = NULL

004AF59F     . E8 B8FEFFFF       CALL pll621_9.004AF45C       ; \SendMessageA

004AF5A4     . 33C0               XOR EAX,EAX

004AF5A6     .^E9 F7FEFFFF       JMP pll621_9.004AF4A2====>找到一个合适的

004AF5AB     >^E9 73FFFFFF       JMP pll621_9.004AF523＝＝》没有找到，重头开始

以上代码具体流程含义请参照我一点想法的那篇文章

因为老外用`c++`写的字符比较函数，翻译成汇编就又长又罗嗦，完全没有必要，我也没有这么多空间来装`c＋＋`的垃圾代码，还好我的汇编功力还是勉强可以的，所以我就自己用汇编写了

一个字符比较函数如下：

也就是上面代码比较需要`call`的函数

`CALL pll621_9.004AF5ED`＝＝＝＝＝＝＝＝＝＝＝＝＝＝》这个`call`就是比较目的字符是否包含源字符：

`call`到这里

```
004AF5ED      /$ 55                  PUSH EBP
004AF5EE      |. 8BC4                MOV EAX,ESP
004AF5F0      |. 5D                  POP EBP
004AF5F1      |. 60                  PUSHAD
004AF5F2      |. 8BE8               MOV EBP,EAX＝＝》以上都是保证堆栈合理用的
004AF5F4      |. 8B45 04            MOV EAX,DWORD PTR SS:[EBP+4]
004AF5F7      |. A3 10FF4A00        MOV DWORD PTR DS:[4AFF10],EAX
004AF5FC      |. 33C0               XOR EAX,EAX
004AF5FE      |. E8 8D000000       CALL pll621_9.004AF690＝＝＝>这个函数是把两个需要比较的字符串都转换成大写，这样比较就不会区分大小写了
004AF603      |. 90                  NOP
004AF604      |. 90                  NOP
004AF605      |. 90                  NOP
004AF606      |. 90                  NOP
004AF607      |. 90                  NOP
004AF608      |. 90                  NOP
004AF609      |. 90                  NOP
004AF60A      |. 90                  NOP
004AF60B      |. 33C0               XOR EAX,EAX
004AF60D      |. A3 14FF4A00       MOV DWORD PTR DS:[4AFF14],EAX
004AF612      |. 8B75 0C           MOV ESI,DWORD PTR SS:[EBP+C]
004AF615      |. B9 00000000       MOV ECX,0
004AF61A      |. 8B7D 08           MOV EDI,DWORD PTR SS:[EBP+8]
004AF61D      |> 8A06              /MOV AL,BYTE PTR DS:[ESI]
004AF61F      |. 8B0D 14FF4A00    |MOV ECX,DWORD PTR DS:[4AFF14]
004AF625      |. 8A2439           |MOV AH,BYTE PTR DS:[ECX+EDI]
004AF628      |. 80FC 00          |CMP AH,0
004AF62B      |. 74 2D            |JE SHORT pll621_9.004AF65A
004AF62D      |. 3AC4             |CMP AL,AH
004AF62F      |. 75 0F            |JNZ SHORT pll621_9.004AF640
004AF631      |. 47               |INC EDI
004AF632      |. 46               |INC ESI
004AF633      |. 8A06             |MOV AL,BYTE PTR DS:[ESI]
004AF635      |. 3C 00            |CMP AL,0
004AF637      |.⁷⁵ E4             |JNZ SHORT pll621_9.004AF61D
004AF639      |. B8 01000000     |MOV EAX,1
004AF63E      |. EB 2A           |JMP SHORT pll621_9.004AF66A
004AF640      |> FF05 14FF4A00   |INC DWORD PTR DS:[4AFF14]
004AF646      |. 8B75 0C         |MOV ESI,DWORD PTR SS:[EBP+C]
004AF649      |. 8B0D 14FF4A00  |MOV ECX,DWORD PTR DS:[4AFF14]
004AF64F      |. 8B7D 08         |MOV EDI,DWORD PTR SS:[EBP+8]
004AF652      |. 803C39 00      |CMP BYTE PTR DS:[ECX+EDI],0
004AF656      |. 74 02           |JE SHORT pll621_9.004AF65A
004AF658      |.^EB C3           \\JMP SHORT pll621_9.004AF61D
004AF65A      |> 61              POPAD
004AF65B      |. B8 00000000     MOV EAX,0
004AF660      |. 83C4 0C         ADD ESP,0C
004AF663      |. FF35 10FF4A00  PUSH DWORD PTR DS:[4AFF10]
004AF669      |. C3              RETN
004AF66A      |> 61              POPAD
004AF66B      |. B8 01000000     MOV EAX,1
004AF670      |. 83C4 0C         ADD ESP,0C
004AF673      |. FF35 10FF4A00  PUSH DWORD PTR DS:[4AFF10]
004AF679      \\. C3             RETN
```

> 转换大小写函数的代码：

```
004AF690      /$ 60              PUSHAD
004AF691      |. 8B75 08         MOV ESI,DWORD PTR SS:[EBP+8]
004AF694      |> 8A06            /MOV AL,BYTE PTR DS:[ESI]
004AF696      |. 3C 00           |CMP AL,0                       ;  Switch (cases 0..7A)
004AF698      |. 74 14           |JE SHORT pll621_9.004AF6AE
004AF69A      |. 3C 61           |CMP AL,61
004AF69C      |. 73 02           |JNB SHORT pll621_9.004AF6A0
004AF69E      |. EB 0B           |JMP SHORT pll621_9.004AF6AB
004AF6A0      |> 3C 7A           |CMP AL,7A
004AF6A2      |. 77 07           |JA SHORT pll621_9.004AF6AB
```



```
004AF6A4      |. 2C 20             |SUB AL,20                  ;  Cases 61 ('a'),62 ('b'),63 ('c'),64 ('d'),65 ('e'),66 ('f'),67 ('g'),68 ('h'),69 ('i'),6A ('j'),6B ('k'),6C ('l'),6D ('m'),6E ('n'),6F ('o'),70 ('p'),71 ('q'),72 ('r'),73 ('s'),74 ('t')... of
004AF6A6      |. 90               |NOP
004AF6A7      |. 90               |NOP
004AF6A8      |. 90               |NOP
004AF6A9      |. 8806             |MOV BYTE PTR DS:[ESI],AL
004AF6AB      |> 46               |INC ESI                     ;  Default case of switch 004AF696
004AF6AC      |.^EB E6            |\JMP SHORT pll621_9.004AF694
004AF6AE      |> 90               NOP                           ;  Case 0 of switch 004AF696
004AF6AF      |. 8B75 0C          MOV ESI,DWORD PTR SS:[EBP+C]
004AF6B2      |> 8A06            /MOV AL,BYTE PTR DS:[ESI]
004AF6B4      |. 3C 00            |CMP AL,0                     ;  Switch (cases 0..7A)
004AF6B6      |. 74 11            |JE SHORT pll621_9.004AF6C9
004AF6B8      |. 3C 61            |CMP AL,61
004AF6BA      |. 73 02            |JNB SHORT pll621_9.004AF6BE
004AF6BC      |. EB 08            |JMP SHORT pll621_9.004AF6C6
004AF6BE      |> 3C 7A            |CMP AL,7A
004AF6C0      |. 77 04            |JA SHORT pll621_9.004AF6C6
004AF6C2      |. 2C 20            |SUB AL,20                    ;  Cases 61 ('a'),62 ('b'),63 ('c'),64 ('d'),65 ('e'),66 ('f'),67 ('g'),68 ('h'),69 ('i'),6A ('j'),6B ('k'),6C ('l'),6D ('m'),6E ('n'),6F ('o'),70 ('p'),71 ('q'),72 ('r'),73 ('s'),74 ('t')... of
004AF6C4      |. 8806             |MOV BYTE PTR DS:[ESI],AL
004AF6C6      |> 46               |INC ESI                      ;  Default case of switch 004AF6B4
004AF6C7      |.^EB E9            |\JMP SHORT pll621_9.004AF6B2
004AF6C9      |> 61               POPAD                         ;  Case 0 of switch 004AF6B4
004AF6CA      \. C3               RETN
```

以上两段代码的含义请看附录。

不过上面两端代码写的没有什么章法，也没有什么规范，而且从程序员的角度来讲完全是三流的，如果用在单片机这个领域来看，那就更加不行了，单片机的空间是有限的，不能这么乱写重复的代码浪费空间。还好我这里 `w32asm` 程序大，几个字节的区区空间还是有的，所以也懒得规划了，只要能正确运行就 `ok` 了。我想把代码简化的任务就教给兄弟们了，`wscn` 上，重新简化代码就是你的任务了，我只管大方向，不管小细节：）

好了，到现在为止，所有的功能都写完了，不知道比老外写的还有什么不足，主要是我自身的水平太烂，没有什么更好的可以和大家交流的。以后我不会写这种重复技术的文章，除非觉得有什么新的东西可写，因为写重复的东西没有意义。谢谢大家。

你的朋友：`pll621`

### 附录

这里是我写的比较函数，和转换大小写函数的草本，有一点注释，但愿大家能看懂。

这段是比较函数的草本：

```
xor eax,eax
mov [4aff14],eax
mov esi,[ebp+0c]  ; esi 是源字符串地址
mov ecx,0
mov edi,[ebp+08]  ; edi 是目的字符串地址

p3:
mov al,[esi]       ; 给 al 一个源字符

p1:
mov ecx,[4aff14]   ; 把现在比较的位置
mov ah,[edi+ecx]   ; 给一个目的字符
cmp ah,0           ; 比较是不是目的字符到头了
jz exit            ; 到头就退出
cmp al,ah          ; 比较源字符和目的字符
jnz p2             ; 不相等就把目的字符向后退一位，再比较
inc edi            ; 增加目的字符指针
inc esi            ; 增加源字符指针
mov al,[esi]
cmp al,0           ; 判断是否源字符到头了
jnz p3             ; 还没有到头，继续比较
mov eax,1          ; 到头了，证明找到
jmp find

p2:
INC DWORD PTR DS:[4AFF14] ; 目的字符向后退一位
mov esi,[ebp+0c]    ; 重新定位 esi 和 edi
mov ecx,[4aff14]
mov edi,[ebp+08]
cmp byte ptr [edi+ecx],0 ; 目的字符是否已经退到结尾了
jz exit
jmp p3

exit:
popa
mov eax,0
add esp,0c
push [4aff10]
ret

find:
popa
mov eax,1
add esp,0c
push [4aff10]
ret
```

这段是转化大小写函数的草本：

这个操作很简单看看就明白，我写的时候连注释都懒得加。

```
pusha
begain:
mov esi,[ebp+0c]
mov al,[esi]
cmp al,0
jz nextchange
cmp al,61
jae small
jmp next

small:
cmp al,7a
ja next
sub 20
mov [esi],al
next:
inc esi
jmp begain

begain1:
pusha
mov esi,[ebp+08]
mov al,[esi]
cmp al,0
jz exit
cmp al,61
jae small1
jmp next

small1:
cmp al,7a
ja next1
sub 20
mov [esi],al
next1:
inc esi
jmp begain1

exit:
popa
ret
```

`4dd466d0e1e0cc123afbe504da2462ca`
