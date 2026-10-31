
# [分享]为PE Optimizer添加拖放功能

本人菜鸟一个，写的不好，如有错误请指正，刚学了`pll621`的`diy pe`教学3--`w32Dasm`加滚轮拖放。

今天来练习下，给程序添加拖拽功能，练手软件`PE Optimizer1.4`英文原版脱壳文件，用工具脱壳后汉化了界面，添加了`XP`样式，发现不支持拖拽，就拿来练手了，不好意思了。

我们需要给程序添加函数使之支持拖放，用到的函数有`DragAcceptFiles`、`DragQueryFileA`、`DragFinish`，其中分别为支持拖放、获得拖放文件路径、结束拖放。

首先我们用工具`LordPE`给软件添加所需的三个函数，这三个函数在`SHELL32.DLL`内记下四个函数的`ThunkRVA`值`0001403C`、`00014040`、`00014044`，下面添加`FirstThunk`以便以后调用。

用16进制工具查看资源段末尾有点空间可以存放代码，用`OD`加载来到`C710`对应位置处修改如下：

```
00413F10  - FF25 3C404100   JMP DWORD PTR DS:[<&SHELL32.DragAcceptFi>; shell32.DragAcceptFiles ----JMP DWORD PTR DS:[41403C]

00413F16  - FF25 40404100   JMP DWORD PTR DS:[<&SHELL32.DragQueryFil>; shell32.DragQueryFileA  ----JMP DWORD PTR DS:[414040]

00413F1C  - FF25 44404100   JMP DWORD PTR DS:[<&SHELL32.DragFinish>] ; shell32.DragFinish      ----JMP DWORD PTR DS:[414044]
```

保存，`DragAcceptFiles`是实现拖放功能的函数，此函数有两个参数，一个是是否接受拖放，一个是对象的句柄，接受设置为`1`即可，现在来找句柄`HWND`，右键查找所有模块间的调用--在所有`SendMessageA`上下断点，运行，系统断下

```
00406506  |.  53            PUSH EBX                                 ; /lParam

00406507  |.  6A 00         PUSH 0                                   ; |wParam = 0

00406509  |.  68 80000000   PUSH 80                                  ; |Message = WM\_SETICON

0040650E  |.  A1 E0844000   MOV EAX,DWORD PTR DS:[4084E0]            ; |

00406513  |.  50            PUSH EAX                                 ; |hWnd => 60340

00406514  |.  E8 4FE1FFFF   CALL <JMP.&user32.SendMessageA>          ; \SendMessageA
```

向上来到段首，发现调用来自`004066E6`，我们来到`4066E6`向上看，发现了系统判断消息的地方，好啊正好待会我们获得拖放地址的时候会用到。程序把句柄存到了`4084E0`这里然后调用的，我们以后可以调用这个地址，就选择把`4066E6`修改了转到我们自己的代码处，添加接受拖放。修改为：`JMP 413F2A`，来到`413F2A`修改如下：

```
00413F2A    6A 01           PUSH 1

00413F2C    50              PUSH EAX

00413F2D    E8 DEFFFFFF     CALL 00413F10

00413F32    E8 A925FFFF     CALL 004064E0

00413F37  - E9 B627FFFF     JMP 004066F2
```

保存，现在试试是不是拖放后图标变了，接下来实现获得拖放文件的路径，获得路径后我们要显示出来。

我们先找怎么显示出来，在所有的`SetDlgItemTextA`上下断点，运行程序，点击浏览按钮选择一个文件，程序断下：

```
004053EC  |.  50            PUSH EAX                                 ; /Text

004053ED  |.  68 91010000   PUSH 191                                 ; |ControlID = 191 (401.)

004053F2  |.  A1 E0844000   MOV EAX,DWORD PTR DS:[4084E0]            ; |

004053F7  |.  50            PUSH EAX                                 ; |hWnd => 000905FC ('PE Optimizer by Dr.Golova',class='#32770')

004053F8  |.  E8 73F2FFFF   CALL <JMP.&user32.SetDlgItemTextA>       ; \SetDlgItemTextA
```

向上发现`004053BF MOV DWORD PTR SS:[ESP+1C], 004084E4`，存放了路径，我们记住`4084E4`，我们拖放的文件路径也放到这个。根据程序自身的函数来修改显示出路径，好了需要的东西找齐了，开始动工；来到消息判断处：

```
00406679   .  8B5D 10       MOV EBX,DWORD PTR SS:[EBP+10]

0040667C   .  33F6          XOR ESI,ESI

0040667E   .  8B45 0C       MOV EAX,DWORD PTR SS:[EBP+C]

00406681   .  83E8 02       SUB EAX,2                                ;  Switch (cases 2..111)

00406684   .  74 67         JE SHORT 004066ED
```

运行到`406681`时`EAX`放的是消息的参数，拖放为`233H`，即上一句的`EBP+C`的位置，`EBP+10`是拖放操作的句柄，正是我们要找的`DragQueryFileA`函数所要的句柄，查看数据发现，`EBP+8`为调试程序的句柄，我们也可以调用之前的`4084E0`记得吧。

修改`40667E`跳转到我们自定义修改的位置`413F40`，修改如下：

```
0040667E    - E9 BDD80000   JMP 00413F40

00406683      90            NOP

00406684   .  74 67         JE SHORT 004066ED
```

到`413F40`修改如下：

```
00413F40    8B45 0C         MOV EAX,DWORD PTR SS:[EBP+C]

00413F43    3D 33020000     CMP EAX,233

00413F48    74 0C           JE 00413F56        ;是拖放就跳转

00413F4A    83E8 02         SUB EAX,2

00413F4D  - E9 3227FFFF     JMP 00406684                ;跳回原程序执行
```

下面自定义处理拖放程序：

```
00413F56    68 04010000     PUSH 104

00413F5B    68 E4844000     PUSH 004084E4                        ; 原存放路径的地方

00413F60    6A 00           PUSH 0

00413F62    FF75 10         PUSH DWORD PTR SS:[EBP+10]               ; 拖放的句柄

00413F65    E8 ACFFFFFF     CALL 00413F16       ; 获取拖放文件的路径JMP.&SHELL32.DragQueryFileA

00413F6A    68 E4844000     PUSH 004084E4

00413F6F    68 91010000     PUSH 191                                 ; 控件ID

00413F74    FF75 08         PUSH DWORD PTR SS:[EBP+8]                ; 程序句柄

00413F77    E8 F406FFFF     CALL 00404670       ; 显示路径JMP.&user32.SetDlgItemTextA

00413F7C    FF75 10         PUSH DWORD PTR SS:[EBP+10]

00413F7F    E8 98FFFFFF     CALL 00413F1C       ; 拖放过程结束JMP.&SHELL32.DragFinish

00413F84  - E9 6127FFFF     JMP 004066F2        ;转到消息结束

00413F89    90              NOP
```

好了保存后就可以测试了，注册这么久了，一直没怎么学。请大家多多指教！`Your Friends IVASOX, 2012-10`

![](https://bbs.kanxue.com/upload/attach/201210/235855_slubd6pul90ve95.png)

[PEOptimizer.rar](https://bbs.kanxue.com/attach-download-72267.htm)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [PEOptimizer.rar](javascript:void(0);)（53.73kb，66次下载）
- [PO.png](javascript:void(0);)（13.47kb，269次下载）
