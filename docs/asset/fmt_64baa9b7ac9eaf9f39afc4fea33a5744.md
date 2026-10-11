
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
