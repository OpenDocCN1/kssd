
# w32Dasm 加滚轮拖放

> 原文：[`www.pediy.com/kssd/pediy05/pediy50239.htm`](https://www.pediy.com/kssd/pediy05/pediy50239.htm)

diy 你的 pe 完结篇

上两篇我教给大家如何 diy pe，由于种种原因，都是没有太实际的例子。今天我就讲述一个实际的例子。

我想看雪论坛的人没有不用 w32asm 的吧，这个实用的工具大家都爱它。但是 w32asm 也有些不完善的地方。

首先它不支持滚轮鼠标的滚动（当然是在你没有用辅助鼠标软件的情况下），然后是不支持文件的拖放

你打开 w32asm 后拖个文件进去，鼠标是个禁止拖放的图标。想实现这些功能么，那就带上你的老虎钳（`trw`）

，扳手（`hiew`）我们出发修理机床（w32asm，我修理的 w32asm 是 killer 修改过的 w32asm10，原文件用 `pecompack` 压缩过，

自己脱壳修改），（声明：我的系统是 98，2000 下我没有测试。看完我这篇文章的有兴趣的人，可以在 2000 下试试）

好，我们先做第一个功能吧加入鼠标滚轮功能（这时候我的想法是假如我有 w32asm 的源程序的话多好啊，没有？那只有从反编译的 pe 文件干活了，

感觉像在修里煤气管道漏气）

## 第一步：分析问题

（每个 diy 者都应该养成这个习惯，不要上来就 `bpx` 断点乱下一通）

不支持滚轮是什么原因造成的？

windows 是个消息系统，w32asm 不支持滚轮是因为它接受到滚轮的消息但是根本不处理它，

我们的目标就是找到 w32asm 处理 windows 消息的地方，然后加入处理滚轮消息，如果你问我怎么处理滚轮消息，很简单，

我们把滚轮的消息转化成按键的消息，w32asm 不是可以按上下键来滚动么，我们把滚轮的上下滚动的消息转化为上下按键

的消息，然后其他的事情交给 w32asm 自己去处理就 ok，好了修理思路定好了，开始动工

## 第二步：找到 win32asm 处理消息的地方

怎么找处理消息，我在上一篇文章已经讲过了，这里我就不在重复了。

找到处理消息的地方如下

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045E471(U)

|

```
:0045261E 8B4604                mov eax, dword ptr [esi+04] =====>【这里是函数参数的传递，esi+04 就是 msg 的 unit】
:00452621 3D21010000            cmp eax, 00000121======>这里 eax 就是 windows 消息的代码
:00452626 7F41                  jg 00452669
:00452628 0F84A30D0000          je 004533D1
:0045262E 3D11010000            cmp eax, 00000111===>看过我上篇文章的人就熟悉这个吧，就是 wm_command
:00452633 7F1B                  jg 00452650     *处理消息的地方很长，因为 w32asm 有很多消息要处理*
:00452635 7463                  je 0045269A     *限于篇幅，我只贴关键地方，有兴趣的朋友可以自己*
:00452637 2DA0000000            sub eax, 000000A0     *反汇编原文件观察            *
:0045263C 0F84530D0000          je 00453395     *                         *
:00452642 83E860                sub eax, 00000060
:00452645 0F8493000000          je 004526DE
:0045264B E92F120000            jmp 0045387F
```

好我们下断点 `bpx 45261e`

这时候你会发现不断的中断停在这个地方，这很正常，这里是 w32asm 处理消息的核心部分，每一个给 w32asm 的消息都经过这里过滤。

频繁的 windows 消息当然就会频繁的中断了：）。这个没有关系，每次中断我们就 `f5`，然后在按 `f5` 的时候的间隔就滚动鼠标的滚轮，看看

滚轮的消息是多少（注：这个在 windows 编程手册里可以查到 `wm_mousewhell` 消息的代码是 20A，我这样来的目的是因为我不知道其 waram 的

子参数，也就是如何判断滚轮向上滚动和向下滚动的子参数是如何定义的，而且手头上没有类似的滚轮程序，我只有自己测试了，这个方法

是比较笨了一点，但是我们想人家郭靖都是大智若愚，安慰一下自己），当看到 `esi+04` 的值为 20a 的时候我们 `d esi+08`（一般子参量都是这个

位置，windows 消息本身就是这么定义的），比较向上滚动和向下滚动 `esi+08` 地址有什么不同，我这里看到的是当向上滚动的时候 `esi+0a` 是 7800

当向下滚动的时候 `esi+0a` 是 88ff，我就初步确定向上滚动和向下滚动的判断了。知道了滚轮的滚动，我们还要测试一下按键的子参数，按键的消息

是 `wm_keydown` 100, `wm_keyup` 101, `wm_char` 102 这里 w32asm 只处理 `wm_keydown` 的消息（看 w32asm 消息处理的地方，我没有贴出来，有兴趣可以自己反

编译自己往 45264B 下面看），好既然只处理 `wm_keydown` 的消息，我们还是老办法，`f5` 加上在间隔的时候按下向上，向下的按键，然后观察 `esi+08` 的地址

的子参量，我这里当按向上键 `esi+08` 是 26，向下键 `esi+08` 是 28，好现在我们知道了所有要知道的东西了，开始用扳手去修理机床吧

## 实现滚轮功能

首先找到在程序里空出来的地方

把原来的

```
:00452621 3D21010000            cmp eax, 00000121
```

改为

```
00452621: E90ACD0500           jmp       .0004AF330 =====>跳到我们自己的消息处理补丁上
```

|

自己的消息处理补丁：

```
004AF330: 3D0A020000           cmp       eax,00000020A ;"  =====>比较消息是否是滚轮
"
004AF335: 7415                 je        .0004AF34C  ----- (1)=====是的话转到滚轮处理
004AF337: 3D33020000           cmp       eax,000000233 ;"  3"=====比较消息是否是拖放文件（这个是补丁第二个功能用的，在下面有阐述）
004AF33C: 7400                 je        .0004AF33E  ----- (2)======>是的话就转到（拖放文件的处理）
004AF33E: 90                   nop===========空出这么多 nop 是因为我怕等一下补丁 je        .0004AF33E 以后会变成长跳转所以预留这些字节
004AF33F: 90                   nop
004AF340: 90                   nop
004AF341: 90                   nop
004AF342: 3D21010000           cmp       eax,000000121 ;"  !"=====恢复原程序动作
004AF347: E9DA32FAFF           jmp       .000452626  ----- (3)=====跳回原程序地方
004AF34C: 668B460A             mov       ax,[esi][0A]=====取出滚轮的子参数
004AF350: 663DFF00             cmp       ax,000FF ;"  "=====比较是向上滚还是向下滚
004AF354: 720A                 jb        .0004AF360  ----- (4)
004AF356: B828000000            mov       eax,000000028 ;"  ("=====向下滚动，改动消息子参数为按键下
004AF35B: 894608                mov       [esi][08],eax
004AF35E: EB08                  jmps      .0004AF368  ----- (5)
004AF360: B826000000            mov       eax,000000026 ;"  &"=====向上滚动，改动消息子参数为按键上
004AF365: 894608                mov       [esi][08],eax
004AF368: B800010000            mov       eax,000000100 ;"  "=====改动消息为按键消息
004AF36D: 894604                mov       [esi][04],eax
004AF370: EBD0                  jmps      .0004AF342  ----- (6)=====跳回原程序
```

好到现在为止，我们测试看看，哈哈，果然滚动了，成功！什么？你说滚动的太慢！）◎）◎※￥）※）（◎，如果嫌慢的话，自己把按键消息子参数改成 `pgdn`。和 `pgup` 一次滚动一页，快了吧。什么？你说能不能不要这么快，一次滚动 3 行，或者 5 行，最好加个定义窗口想滚动几行就几行？我要晕倒了，

大哥，我是改动 pe 文件，不是改源程序。如果你想这样的话，我告诉你思路，你自己做.首先自己在 w32asm 的 menu 里面加个定义滚轮滚动的子选项，我的第二篇

文章有说怎么做的，然后加入点击这个子选项的消息处理，让点击这个子选项的时候弹出一个 dialog，可以用 `CreateDialog` 的 api 函数，当然你的先做好这个 dialog 的资源，然后加入这个 dialog 的的消息函数，在 dialog 里面做个 edit 的控件和一个 button 控件当点击 button 的时候就把 edit 里面的数值保存到一个地址

，然后你的滚轮判断的地方读入这个地址的值，根据这个值判断用 `sendmessage` 函数向 w32asm 发送多少个按键消息，1 就发 1 个，50 就发 50 个，这样你乐意滚动多少行就多少行，还可以自定义：）。我是很累，不做了，那个大哥不满意我做的就自己就做一个想滚动几行就几行的吧，记得做好了发一个给我用用就行了!好了

到现在为止我们的滚轮版已经做好了。休息一下眼睛，我们要开始大动干戈，开始做拖放版了（如果是初学者就不要往下看了，讲述的专业比较多）

## 实现拖放功能

现在继续我们的 diy 之旅

打开 w32asm 试试拖放一个文件，呵呵，鼠标是禁止拖放的图标，证明是 w32asm 是根本不支持拖放，说白了也就是不处理拖放的消息。

好，既然我们要是这个 w32asm 支持拖放，首先我们先了解一下拖放的知识，windows 是个图形界面系统，各种各样的程序都是基于图形界面的，

这个界面是我们也可以称为窗口，窗口很多属性，比如大家都了解的 `enablewindow` 就是设置窗口的属性的，不过这个函数是设置窗口是否可用的

属性。现在我们需要的是窗口是否接受拖放的属性，我告诉大家，这个函数就是 `DragAcceptFiles`，它是 `shell32.dll` 的函数，与拖放有关的函数还有 `DragFinish` 和 `DragQueryFile`，我们要补丁也就要用到这个三个函数，

首先我们看看 `DragAcceptFiles` 这个函数是设置窗口是否能接受拖放的消息，也就是 windows 是否发送 `WM_DROPFILES` 消息给这个窗口。使用的方法是 api 手册上查到为：

```
VOID DragAcceptFiles(
    HWND hWnd,    //  handle to the registering window 注册窗口的 hwnd
    BOOL fAccept  // acceptance option  是否接受拖放的消息
);
```

知道了 `DragAcceptFiles` 的用法后.我的思路是把 w32asm 的所有的窗口都加上一个

`invoke DragAcceptFiles,hwnd,TRUE`

当然这个是在汇编里实现，但是在已经连接好的 pe 文件里如何实现

`invoke DragAcceptFiles,hDlg,TRUE` 呢？

要达到这个目的

第一：首先我们需要 pe 文件的 import 表（注：如果不懂 import 表的可以自己先学习一下 pe 文件的格式，不难，只是有点烦而已）有 `DragAcceptFiles` 这个引入函数

第二：然后就是我们需要在 pe 文件中的到窗口的 `hwnd`

达到以上条件以后

只要在 pe 空白空间用以下代码就可实现

```
push 1
push hwnd
call DragAcceptFiles
```

我们现在先来达到第一个条件

拿出我脱壳的 w32asm(注：我自己脱壳的有两个版本，第一个是在入口点 dump 完全的 pe，然后手动修复 import，这个版本很遗憾，只能在 98 下运行，不能在 2000 下运行，原因是需要修补的 import 太多可能有遗漏，或者我也不知道，第二个版本，是情狼大哥在入口点用 `trw2000` 的 `pedump` 命令 dump 出来版本，这个版本能跨平台，但是也遗憾，用 pe 编辑器看不到任何的 import 表）

为了能跨平台，我决定用第二个版本开始改造，由于没有任何的 import 表，我们根本不知道是否包含有 `DragAcceptFiles` 这个函数，用 `stud_pe` 打开自己做的第一个版本（也就是修复了 import 表的那个版本），点击 `function` 按钮，观察 import 的函数信息发现其函数有 `kernell32`、`gdi32.dll`

等，没有包含 `shell32.dll`，也就是说，不可能有 `DragAcceptFiles` 这个我们需要的函数了，没有函数，我们又想用，那就只有自己手动构造了，三个相关的函数构造起来，还是比较容易的，不过我还是教大家怎么使用 `lordpe` 构造吧（用 `stud_pe` 和 `peeditor` 都差不多，手动也行，如果来个二三十个函数的话，我看手动的可就吃亏了）

### 构造函数篇：

用 `lordpe` 打开第二个脱壳版本，点击 `directories` 按钮，然后点击 `importtabl` 按钮，只能看到一个 `kernell32.dll`，在上面点击右键，选择 `add import`，弹出窗口，在 `dll` 填入 `SHELL32.dll`，`api` 填入 `DragAcceptFiles` 点击那个加号的按钮看到 `DragAcceptFiles` 已经添加进去了然后用这个方法添加 `DragFinish` 和 `DragQueryFile` 函数添加完成后点击 `ok`，看到 import 表多了一个 `shell32` 的 `dll` 了吧，里面有我们添加的三个函数,记下这个三个函数的 `thunkrva` 值（以后要用到），我这里如下:

```
ThunkRva      ThunkOffset     ThunkValue      Hint      ApiName

0015803B      0015803B        0015800C        0000      DragAcceptFiles
0015803F      0015803F        0015801E        0000      DragQueryFile
00158043      00158043        0015802E        0000      DragFinish
```

其实你也可以自己用 `winhex` 构造这个三个函数，不难，看看相关的 pe 格式文档就能做到，自己构造的有个好处就是结构比较分明，看起来比较舒服，软件构造的是重新定位过 import 所以有点混乱，不过你想偷懒就用软件构造吧，这个方便

构造完了函数,我们的给它一个 firshthunk，否则在程序里面怎么 call 那里呢？这里就用到了 `thunkrva` 的值了，现在工具换成了 `hiew`,用 `hiew` 打开脱壳后的 w32asm 我们来到 `004AF29C`

如下：

```
004AF29C: FF25244B4D00         jmp       d,[004D4B24]
004AF2A2: FF252C4B4D00         jmp       d,[004D4B2C]
```



`004AF2A8: FF25304B4D00 jmp d,[004D4B30]`

`004AF2AE: FF25344B4D00 jmp d,[004D4B34]`

这些 `jmp` 都是定位 api 函数的，每个 `jmp` 代表程序要用到 api 函数地址

`004AF2B4: 0000 add [eax],al *****************************`

`004AF2B6: 0000 add [eax],al *你问，怎么找到这个地方的，很简单，随便下个程序要用到的 api 函*`

`004AF2B8: 0000 add [eax],al *数，看看其 call 的是那个地址，就能找到这个地方      *`

`004AF2BA: 0000 add [eax],al *****************************`

我们现在就要加入 `DragAcceptFiles`、`DragQueryFile`、`DragFinish` 三个函数的 `jmp`，

`jmp` 到那里,其实就是 `jmp` 那个 `ThunkRva+imagebase` 的值, `imagebase` 值是 `4000000`

所以我们要在 `004AF2B4` 处填入：

`004AF2B4：jmp d,[0055803B] *55803b=15803B(ThunkRva)+4000000(imagebase)以下一样计算`

`    jmp d,[0055803f]`

`    jmp d,[00558043]`

添完后如下：

`004AF29C: FF25244B4D00 jmp d,[004D4B24]`

`004AF2A2: FF252C4B4D00 jmp d,[004D4B2C]`

`004AF2A8: FF25304B4D00 jmp d,[004D4B30]`

`004AF2AE: FF25344B4D00 jmp d,[004D4B34]`

`004AF2B4: FF253B805500 jmp DragAcceptFiles ;SHELL32.dl==>看到我们构造的函数了么，真是 happy 啊`

`004AF2BA: FF253F805500 jmp DragQueryFile ;SHELL32.dll`

`004AF2C0: FF2543805500 jmp DragFinish ;SHELL32.dll`

`004AF2C6: 0000 add [eax],al`

`004AF2C8: 0000 add [eax],al`

现在记下 `004AF2B4`、`004AF2BA`、`004AF2C0` 这三个值. 以后 `call 004AF2B4` 就是 `call    DragAcceptFiles` 了，调用 `DragQueryFile` 也就是 `call 004AF2BA` 了，以此类推

函数构造完成第一个条件也就满足了，开始我们第二个条件 `hwnd` 的寻找吧

### 寻找 `hwnd` 打补丁篇：

如何找到 `hwnd` 值呢，对 windows 编程熟悉的人(你不熟悉就慢慢学，总有一天会熟悉的）知道 `GetWindowRect` 函数也需要一个 `hwnd` 值，我们截获

`getwindowsrect` 函数，保存这个 `hwnd`，再使用这个 `hwnd` 来 `DragAcceptFiles` 不就 ok，好确定思路，打开 `trw` 然后 `bpx getwindowsrect`

启动 `w32asm` 发现其在这里调用 `getwindowsrect`：

`0167:00492941 52        PUSH    EDX=========>这个是一个 rect 的 struct 的 point，也就是一个矩阵结构的指针`

`0167:00492942 FF700C        PUSH    DWORD [EAX+0C]===>这里就是我们需要的 hwnd`

`0167:00492945 E810C90100      CALL    `USER32!GetWindowRect===>调用这个函数了也就是 call      .0004AF25A`

`0167:0049294A EB1B          JMP     SHORT 00492967    `

`0167:0049294C 8B582C          MOV     EBX,[EAX+2C]`

程序启动调用了很多次 `GetWindowRect` 函数，每次 `hwnd` 都不同，这是因为 `w32asm` 窗口分为好几个，`edit` 窗口，`menu` 窗口，`toolbar`，状态栏等等

我们不管它，把每个窗口的 `DragAcceptFiles` 值都设置为 `ture` 就行了，管它有多少个呢，有多少个，就 `ture` 多少个，个个都能接受拖放，想往

那里拖就往那里拖

把这句：

`0167:00492945 E810C90100      CALL    `USER32!GetWindowRect`

改为：

`00492945: E996C90100                  jmp       .0004AF2E0 -----这里是我们自己的补丁程序了`

### 自己补丁的地方：

`004AF2E0: 58                        pop       eax－－－－》取出压入堆栈的 hwnd`

`004AF2E1: A3C6F24A00                  mov       [004AF2C6],eax－－－》保存到一个地址，我选择的是 004AF2C6`

`004AF2E6: 50                        push      eax－－－－》压 hwnd 入堆栈`

`004AF2E7: E86EFFFFFF                  call      .0004AF25A -----》 CALL    `USER32!GetWindowRect`

`004AF2EC: 6A01                      push      001＝＝＝传递 ture`

`004AF2EE: FF35C6F24A00                  push      d,[004AF2C6]＝＝》传递 hwnd`

`004AF2F4: E8BBFFFFFF                  call      DragAcceptFiles ;SHELL32.dll＝＝＝》也就是 call 004AF2B4 我们在上面构造的地方`

`004AF2F9: E96936FEFF                  jmp      .000492967 -----》回到程序原来的地方`

现在打开补丁完的程序，拖个文件试试，是不是图标已经变了，你说，变是变了,怎么没有反应啊，还没有加入事件怎么会有反应？真是的，路要

一步一步的走。不要太急

现在开始我们加入事件的过程了，首先找到接受消息的地方，加入判断拖放消息的代码，上面已经说过了在

`:0045261E 8B4604                  mov eax, dword ptr [esi+04]＝＝＝》【这里是函数参数的传递，esi+04 就是 msg 的 unit】`

`:00452621 3D21010000              cmp eax, 00000121======>这里 eax 就是 windows 消息的代码`

而且

`:00452621 3D21010000              cmp eax, 00000121`

已经改为：

`00452621: E90ACD0500                  jmp      .0004AF330`

也就是消息处理已经在我们自己的补丁地方了

`004AF330: 3D0A020000                  cmp      eax,00000020A ;" ＝＝》比较消息是否是滚轮`

`"`

`004AF335: 7415                      je      .0004AF34C ----- (1)＝＝是的话转到滚轮处理`

`004AF337: 3D33020000                  cmp      eax,000000233 ;" 3"＝＝》比较消息是否是拖放文件`

`004AF33C: 7400                      je      .0004AF33E ----- (2)＝＝》是的话就转到（拖放文件的处理）`

好现在我们可以把 `004AF33C: 7400                      je      .0004AF33E` 转到我们自己拖放处理的地方了

现在程序也能判断拖放消息，我们就需要根据这个消息做相关的动作了，也就是需要做打开这个拖放文件并且反汇编它。

学习到这里，大家是不是已经比较有思路了，首先我们必须得到拖放文件的文件名，这就需要 `DragQueryFile` 函数了

The `DragQueryFile` function retrieves the filenames of dropped files.

```
UINT DragQueryFile(
    HDROP hDrop,    // handle to structure  for dropped files＝＝＝》拖放的 hdrop
    UINT iFile,    //  index of file to query＝＝》为 0 就可以了
    LPTSTR lpszFile,    //  buffer for returned filename＝＝》放文件名的地址
    UINT cch      //  size of buffer for filename＝＝》buffer 的尺寸
    );
```

我们看看这个函数需要什么参数，现在需要找的就是 `hdrop` 这个类似于 windows 窗口的 `hwnd` 用来唯一标识拖放的，这个也就是 `msg` 的子参数

`:0045261E 8B4604                  mov eax, dword ptr [esi+04]＝＝＝》【这里是函数参数的传递，esi+04 就是 msg 的 unit】`

`esi+04` 是消息 `unit`，那么这个子参数就在 `esi+08` 处

好了相关的参数都搞定了，下一步就是看看 `w32asm` 怎么打开一个根据文件名打开文件的动作的；从那里下手呢？从 `disassembler` 的 `open file` 下手？这个会弹出一个框子出来，不好，那就从 `recent files` 下手，`rencent files` 可以打开你最近打开的文件，好我们看看它是怎么打开的

（跟踪这个动作很复杂，我要详细的说，可能打一天的字也打不完，我打字到现在已经很累了，大家就将就一下，我仅仅讲跟踪的关键地方）

在消息判断（消息是 `111 wm_command`）的地方下断点，然后点击 `recent files` 的按钮，发现其子参量为 `bd02` 然后又调用自己,这时候的子参量变成

`0c5f`,当你点击 `open file` 时候发现子参量也是 `0c5f`，看来是在点击 `recent files` 文件的时候会 `post` 一个 `0c5f` 消息回来，那么 `rencent` 里面的最近打开的文件是存在那个地方呢？经过跟踪可以发现是存在 `c:\windows\w32dasm8.ini` 文件里面，

windows 用来打开 `ini` 文件的 `api` 是 `GetPrivateProfileString`，下 `bpx GetPrivateProfileString` 这个断点，然后点击 `recent` 里面的最近打开的文件程序中断在这里：（我们就来分析这里是什么意思）

`:100411F0`

`:`

`:1004310C 55                        push ebp`

`:1004310D 8BEC                      mov ebp, esp＝＝＝》这是 vc 编程传递参数的形式，用 vc 都是这样，和编译器有关系`

`:1004310F 53                        push ebx＝＝》这里传递的参数是窗口的 hwnd`

`:10043110 56                        push esi`

`:10043111 8B450C                      mov eax, dword ptr [ebp+0C]＝＝》这里是消息的子参数，这里是最近五个文件的 id`

`:10043114 2DBD020000                  sub eax, 000002BD`

`:10043119 D1E0                      shl eax, 1`

`* Possible StringData Ref from Data Obj ->"12345"`

`|`

`:1004311B BB3B5A0410                  mov ebx, 10045A3B`

`:10043120 03D8                      add ebx, eax＝＝＝》把 eax 做转换变成了 table 表，就知道是点击了那个文件，[ebx]会变成 1-5 之间`

`* Possible StringData Ref from Data Obj ->"W32dasm8.ini"`

`|`

`:10043122 684A5A0410                  push 10045A4A==>参数，文件名`

`:10043127 6880000000                  push 00000080==>buffer size`

`:1004312C 68575A0410                  push 10045A57==>读入内容地址指针`

`* Possible StringData Ref from Data Obj ->"NULL"`

`|`

`:10043131 68455A0410                  push 10045A45＝＝》缺省字符串的指针`

`:10043136 53                        push ebx\==>key name`

`* Possible StringData Ref from Data Obj ->"RECENT"`

`|`

`:10043137 68345A0410                  push 10045A34===>段名`

`* Reference To: KERNEL32.GetPrivateProfileStringA, Ord:0125h`

`|`

`:1004313C E8AB0B0000                  Call 10043CEC==>得到 key name 下的内容`

`:10043141 A3305A0410                  mov dword ptr [10045A30], eax\==>返回的到字节`

`:10043146 68575A0410                  push 10045A57`

`* Possible StringData Ref from Data Obj ->"NULL"`

`|`

`:1004314B 68455A0410                  push 10045A45`

`* Reference To: KERNEL32.lstrcmpiA, Ord:02D9h`

`|`

`:10043150 E8F70B0000                  Call 10043D4C==>比较上面两个字符串(这个比较的作用是看是不是要调整最近打开的文件的顺序）`

`:10043155 0BC0                      or eax, eax`



:10043157 745E                  `je 100431B7`   

:10043159 BF575A0410           `mov edi, 10045A57`   

:1004315E 8B0D305A0410         `mov ecx, dword ptr [10045A30]`   

:10043164 8BF7                 `mov esi, edi`   

:10043166 03F9                 `add edi, ecx`   

:10043168 FD                   `std`   

:10043169 B05C                 `mov al, 5C`   

:1004316B F2                   `repnz`   

:1004316C AE                   `scasb`   

:1004316D 85C9                 `test ecx, ecx`   

:1004316F 741C                 `je 1004318D`   

:10043171 83C702               `add edi, 00000002`   

:10043174 C647FF00             `mov [edi-01], 00`   

:10043178 3BFE                 `cmp edi, esi`   

:1004317A 7411                 `je 1004318D`

> 这一段的作用是把 `d:\\1\\hiew.exe` 变成 `d:\\1` 的形式，为 `SetCurrentDirectoryA` 做准备   

:1004317C 56                   `push esi`

> `esi` 的地址就是 `X:\\XXX\\` 的形式了   

\* Reference To: `KERNEL32.SetCurrentDirectoryA`, Ord:023Eh > 设置现在的路径为 `X:\\XXX\\`   

:1004317D E8A60B0000           `Call 10043D28`   

:10043182 57                   `push edi`   

:10043183 68575A0410           `push 10045A57`   

\* Reference To: `KERNEL32.lstrcpyA`, Ord:02DCh   

:10043188 E8C50B0000           `Call 10043D52`   

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:1004316F(C), :1004317A(C)`   

:1004318D 68575A0410           `push 10045A57`   

:10043192 68515E0410           `push 10045E51`   

\* Reference To: `KERNEL32.lstrcpyA`, Ord:02DCh   

:10043197 E8B60B0000           `Call 10043D52`

> 上面一段的作用是调整最近文件打开的顺序   

:1004319C C605505E041002       `mov byte ptr [10045E50], 02`

> 这个可是标志位哦，如果为零的话，那就会弹出那个打开文件的窗口   

:100431A3 6A00                 `push 00000000`   

\* Possible Ref to Menu: `MenuID_00CB`, Item: "Open File to Disassemble.."   

:100431A5 680C5F0000           `push 00005F0C`   

:100431AA 6811010000           `push 00000111`   

:100431AF FF7508               `push [ebp+08]`

> 窗口的 `hwnd`   

\* Reference To: `USER32.PostMessageA`, Ord:01DBh   

:100431B2 E8E70A0000           `Call 10043C9E`

> 我果然没有猜错，就是 post 一个点击 open file 消息给自己   

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:10043157(C)`   

:100431B7 5E                   `pop esi`   

:100431B8 5B                   `pop ebx`   

:100431B9 C9                   `leave`   

:100431BA C20800               `ret 0008`   

这段代码我已经分析完毕，现在我们要做的事情就是在得到 `DragQueryFile` 的消息后，做一样的处理动作，然后 post 一个 open file 消息给自己，就达到目的了。关键的地方是如何得到 `PostMessageA` 的 `hwnd` 呢？这个 `hwnd` 是 w32asm menu 的 id，windows 程序员都知道 `setmenu` 的函数吧，我们下断点 `bpx setmenu`，看到原程序是在这里 `setmenu`

:0049CD86 8BF7                 `mov esi, edi`   

:0049CD88 56                   `push esi`

> 旧 `hwnd`   

:0049CD89 8B0B                 `mov ecx, dword ptr [ebx]`   

:0049CD8B FF710C               `push [ecx+0C]`

> 新 `hwnd`   

\* Reference To: `user32.SetMenu`, Ord:0221h   

:0049CD8E E865230100           `Call 004AF0F8`   

:0049CD93 85C0                 `test eax, eax`   

我们要做的事情就是保存这个新的 `hwnd`，以便 `PostMessageA` 的时候用到   

原代码：

:0049CD8E E865230100           `Call 004AF0F8`   

改为：

```
0049CD8E: E97D250100           jmp       .0004AF310  -----到我们自己保存 hwnd 的代码
```

自己保存 `hwnd` 的代码：

```
004AF310: 58                   pop       eax       > 取出 hwnd
004AF311: A3CAF24A00           mov       [004AF2CA],eax    > 保存在 [004AF2CA]，以便用到的时候在这里取回
004AF316: 50                   push      eax
004AF317: E8DCFDFFFF           call      .0004AF0F8  -----call setmenu
004AF31C: E972DAFEFF           jmp       .00049CD93  -----回到程序原来地方
004AF321: 0000                 add       [eax],al
004AF323: 0000                 add       [eax],al
004AF325: 0000                 add       [eax],al
```

现在 `hwnd` 也有了，而且 `DragQueryFile` 会返回打开文件的字节，完全和 `GetPrivateProfileStringA` 返回的一样，也就是在原来 `GetPrivateProfileStringA` 处改为 `DragQueryFile` 就 ok，那么其他的 `PostMessageA`、`lstrcpyA`、`SetCurrentDirectoryA`、`lstrcmpiA` 的 call 到那里去找呢，如果能看到 import 表就直接从 import 表找，但是这个脱壳版的无 import 表真是麻烦，没有办法，下断点一个一个看，先来 `bpx postmessagea`

原程序中断在

:00453402 E85DBD0500           `Call 004AF164`

> 这个就是 call `postmessagea`

其他几个函数如下：

*   `postmessagea` `:00453402 E85DBD0500` `Call 004AF164`
*   `lstrcpyA` `:00451C49 E80CD00500` `Call 004AEC5A`
*   `SetCurrentDirectoryA` `:0043E679 E8B8050700` `Call 004AEC36`
*   `lstrcmpiA` `:0043E468 E815070700` `Call 004AEB82`

好了现在需要的函数地址也有了，我们开始补丁

```
004AF330: 3D0A020000           cmp       eax,00000020A ;"  > 比较消息是否是滚轮 "
004AF335: 7415                 je        .0004AF34C  ----- (1) 是的话转到滚轮处理
004AF337: 3D33020000           cmp       eax,000000233 ;" 3"  > 比较消息是否是拖放文件
004AF33C: 7400                 je        .0004AF33E  ----- (2) 是的话就转到（拖放文件的处理）
```

先把

```
004AF33C: 7400                 je        .0004AF33E  ----- (2) 是的话就转到（拖放文件的处理）
```

改成：

```
004AF33C: 7442                 je        .0004AF380  -----调到处理拖放的子程序中
```

处理拖放的子程序：

> 这段程序就是参照原来处理程序写的

```
004AF380: FF7608               push      d,[esi][08]
004AF383: 8F05CEF24A00         pop       d,[004AF2CE]  > 保存拖放的 hDrop 到 [004AF2CE]
004AF389: 6800010000           push      000000100 ;"  "  > 缓冲区为 100 所以不要拖动路径长度大于 256 的文件哦，可能要出错
004AF38E: 68575A0410           push      010045A57 ;"ZW"  > 保存文件名称的地址
004AF393: 6A00                 push      000  > UINT iFile
004AF395: FF35CEF24A00         push      d,[004AF2CE]  > push hDrop
004AF39B: E81AFFFFFF           call      DragQueryFile ;SHELL32.dll  > 得到拖动的文件名
004AF3A0: A3305A0410           mov       [10045A30],eax  > 返回的字节数保存，以下都是模仿原来程序的动作，上面已经有分析，我就不详细的说明了
004AF3A5: 68575A0410           push      010045A57 ;"ZW"
004AF3AA: 68455A0410           push      010045A45 ;"ZE"
004AF3AF: E8CEF7FFFF           call      .0004AEB82  ----- (2) call lstrcmpiA
004AF3B4: 0BC0                 or        eax,eax
004AF3B6: 0F846A32FAFF         je        .000452626  ----- (3) 如果为零就直接退出，原动作是直接 ret，我们就直接返回
004AF3BC: 6660                 pusha     > 保存所有的寄存器，因为下面要用到 esi 等，所以现在先保存之
004AF3BE: BF575A0410           mov       edi,010045A57 ;"ZW"
004AF3C3: 8B0D305A0410         mov       ecx,[10045A30]
004AF3C9: 8BF7                 mov       esi,edi
004AF3CB: 03F9                 add       edi,ecx
004AF3CD: FD                   std
004AF3CE: B05C                 mov       al,05C ;"\\"
004AF3D0: F2AE                 repne     scasb
004AF3D2: 85C9                 test      ecx,ecx
004AF3D4: 741D                 je        .0004AF3F3  ----- (4)
004AF3D6: 83C702               add       edi,002 ;""
004AF3D9: C647FF00             mov       b,[edi][-01],000 ;" "
004AF3DD: 3BFE                 cmp       edi,esi
004AF3DF: 7412                 je        .0004AF3F3  ----- (1)
004AF3E1: 56                   push      esi
004AF3E2: E84FF8FFFF           call      .0004AEC36  ----- (2) 就是 call SetCurrentDirectoryA
004AF3E7: 57                   push      edi
004AF3E8: 68575A0410           push      010045A57 ;"ZW"
004AF3ED: 90                   nop
004AF3EE: E867F8FFFF           call      .0004AEC5A  ----- (3) 就是 call lstrcpyA
004AF3F3: 68575A0410           push      010045A57 ;"ZW"
004AF3F8: 68515E0410           push      010045E51 ;"^Q"
```



`004AF3FD: E858F8FFFF` &emsp; `call` `.0004AEC5A` &emsp; `(4)`就是 `call lstrcpyA`

`004AF402: C605505E041002` &emsp; `mov` `b,[10045E50],002` ;`""`

`004AF409: 6A00` &emsp; `push` `000`

`004AF40B: 680C5F0000` &emsp; `push` `000005F0C` ;`" \_`

`004AF410: 6811010000` &emsp; `push` `000000111` ;`" "`

`004AF415: A1CAF24A00` &emsp; `mov` `eax,[004AF2CA]` ＝＝＝＞这个就是我们在 `setmenu` 保存的 `hwnd`

`004AF41A: 50` &emsp; `push` `eax`

`004AF41B: E844FDFFFF` &emsp; `call` `.0004AF164` &emsp; `(5)`就是 `call postmessagea`

`004AF420: FF35CEF24A00` &emsp; `push` `d,[004AF2CE]` ＝＝＝＞这个是保存的 `hDrop`，

`004AF426: E895FeFFFF` &emsp; `call` `DragFinish` ;`SHELL32.dll` ＝＝＝＞拖放处理完后要释放 `hDrop`，调用 `DragFinish`

`004AF42B: 6661` &emsp; `popa` &emsp; 恢复堆栈

`004AF42D: E9F431FAFF` &emsp; `jmp` `.000452626` &emsp; `(7)`跳回原处理地方

好了到现在为止我们 `diy` 加强版的 `w32asm` 已经出来了，大家享受一下劳动成果吧，如果有兴趣的话，还可给 `w32asm` 做很多改造，使之更加适合与我们自己的工作习惯。我已经把方法和原理都教给大家了，有谁做出更加 `power` 的版本，记得给我一份哦，

此程序在 `98` 下测试成功，`2000` 下我想应该不会成功，因为 `2000` 下的拖放函数不同，有兴趣的人可以在里面加上 `getversion` 函数判断是 `98` 还是 `2000`

如果是 `2000` 下用另外一套函数实现。我自己很少在 `2000` 下使用 `w32asm` 所以我就不做了：（，就当留给大家的作业吧

后记：写到这里已经是两手发酸，希望这片文章成为 `diy pe` 的经典教程，如果看雪出一本 `diy pe` 的书是不是考虑选上我这篇文章：）

&emsp;&emsp;&emsp;&emsp;你的朋友：`pll621`

&emsp;&emsp;&emsp;&emsp;`2002.8.24`

又对滚轮功能改动了一番。这时滚动的速度趋于正常化，不再那么慢了。你只要更改其中的一个字节，就能改变滚动速度。不过程序代码比原来大了 `2` 个字节，呵呵。

**基本思路：**

将原来老大的按向下箭头的消息重定向改为了实实在在的 `WM\_MOUSEWHEEL`，对窗口发送了 `WM\_VSCROLL` 消息，即纵向滚动条滚动的消息。查 `MSDN`，找到 `WM\_VSCROLL` 和 `WM\_MOUSEWHEEL` 的相关参数：

1、`WM\_VSCROLL`:

```c
WM_VSCROLL nScrollCode = (int)LOWORD(wParam);
nPos = (short int)HIWORD(wParam);
hwndScrollBar = (HWND) lParam;
```

第一个参数与我们的程序关系密切，作一下说明：

`nScrollCode`:

- `SB\_BOTTOM` Scrolls to the lower right
- `SB\_ENDSCROLL` Ends scroll
- `SB\_LINEDOWN` Scrolls one line down
- `SB\_LINEUP` Scrolls one line up
- `SB\_PAGEDOWN` Scrolls one page down
- `SB\_PAGEUP` Scrolls one page up
- `SB\_THUMBPOSITION` The user has dragged the scroll box (thumb) and released the mouse button. The `nPos` parameter indicates the position of the scroll box at the end of the drag operation.
- `SB\_THUMBTRACK` The user is dragging the scroll box. This message is sent repeatedly until the user releases the mouse button. The `nPos` parameter indicates the position that the scroll box has been dragged to.
- `SB\_TOP` Scrolls to the upper left

其中与我们相关的常量的值是这样定义的，在 `windows.inc` 里面有：

```asm
WM_VSCROLL                equ 115h
SB_LINEUP                 equ 0
SB_LINEDOWN               equ 1
```

2、`WM\_MOUSEWHEEL`

```c
fwKeys = LOWORD(wParam);
zDelta = HIWORD(wParam);
xPos = LOWORD(lParam);
yPos = HIWORD(lParam);
```

其中我们最关心的是第二个参数，即 `zDelta`，它是一个 `DWORD` 参数的高 `16` 位，在 `C` 程序里面是 `word ptr [esi+0Ah]`，它表示了鼠标滚动的距离。这个参数是这样说明的：

`zDelta`

Indicates that the mouse wheel was pressed, expressed in multiples or divisions of `WHEEL\_DELTA`, which is `120`. (笔者注：`120` 即 `78H`，如果正向滚动为 `78H`，反向滚动为 `-78H`，在机器里面以补码表示为 `10000h-78h=0ff88h`，这就是老大跟踪得到的两个值。)

```asm
:004AF34C 60                pushad                  ;保存所有的寄存器
:004AF34D 668B460A          mov ax, word ptr [esi+0A] ;取出 zDelta 值
:004AF351 98                cwde                    ;符号位扩展 ax 到 eax
:004AF352 99                cdq                     ;再将符号扩展 eax 到 edx，这里及 xor eax,edx  sub eax,edx 三句是为了求 zDelta 的绝对值。具体原理自己想吧。
:004AF353 33C2              xor eax, edx
:004AF355 2BC2              sub eax, edx
:004AF357 F7DA              neg edx                 ;edx 作为 WM_VSCROLL 里面的 nScrollCode+10000h*nPos，这里 nPos 可为 0，实际就是 nScrollCode。如果滚动方向为正，edx 为 0，即 SB_LINEUP,否则为 not ffffffff=1，即 SB_LINEDOWN。

;以下的循环按 zDelta 对 78h 的倍数值，重复向窗口发送 WM_VSCROLL 消息：

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AF36F(C)
|
:004AF359 52                push edx                ;先保存 nScrollCode 值，以免调用 PostMessage 时把 edx 破坏。
:004AF35A 50                push eax                ;同样道理，保存 zDelta 值。
:004AF35B 6A00              push 00000000           ;hwndScrollBar，为 0
:004AF35D 52                push edx                ;nScrollCode
:004AF35E 6815010000        push 00000115           ;WM_VSCROLL 消息
:004AF363 FF36              push dword ptr [esi]    ;窗口的 Handle。在 C 里是这样的，不要问为什么：)
:004AF365 E83449B90F        call 10043C9E           ;PostMessage 函数。
:004AF36A 58                pop eax                 ;以下两句弹出保存的两个重要值
:004AF36B 5A                pop edx
:004AF36C 83E878            sub eax, 00000078       ;减去一个标准位移量。如果你嫌它还是太慢，改 78h 为更小的一个数值。不过最好是 78h 的子因数。这样舒服些。
:004AF36F 77E8              ja 004AF359             ;没减完则重复发送下一个 WM_VSCROLL 消息。
:004AF371 61                popad
:004AF372 EBCE              jmp 004AF342            ;转入原程序流程。
```

=================================================================================================

**老大原文：**

`004AF34C: 668B460A` &emsp; `mov` `ax,[esi][0A]` ＝＝＝＞取出滚轮的子参数

`004AF350: 663DFF00` &emsp; `cmp` `ax,000FF` ;`" "` ＝＝＝＞比较是向上滚还是向下滚

`004AF354: 720A` &emsp; `jb` `.0004AF360` &emsp; `(4)`

`004AF356: B828000000` &emsp; `mov` `eax,000000028` ;`" ("` ＝＝＝＞向下滚动，改动消息子参数为按键下

`004AF35B: 894608` &emsp; `mov` `[esi][08],eax`

`004AF35E: EB08` &emsp; `jmps` `.0004AF368` &emsp; `(5)`

`004AF360: B826000000` &emsp; `mov` `eax,000000026` ;`" &"` ＝＝＝＞向上滚动，改动消息子参数为按键上

`004AF365: 894608` &emsp; `mov` `[esi][08],eax`

`004AF368: B800010000` &emsp; `mov` `eax,000000100` ;`" "` ＝＝＝＞改动消息为按键消息

`004AF36D: 894604` &emsp; `mov` `[esi][04],eax`

`004AF370: EBD0` &emsp; `jmps` `.0004AF342` &emsp; `(6)` ＝＝＝＞跳回原程序

=================================================================================================

`4dd466d0e1e0cc123afbe504da2462ca`
