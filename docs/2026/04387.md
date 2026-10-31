
# 我的第一篇破文,献给看学学院! 高手免进! (2千字)

**软件名称：** `某国产软件 V.32`

**简介：** 一个很不错的象棋软件

**破解目的：** 解掉它的时间限制(未注册只能用30天)实现上只是一个Nag窗口，主要是解除其只能下50步的功能限制

**所需工具：** `Ollydbg`、`win32api.chm`

**说明：** 我乃一只菜鸟，本文只作技术交流，文中若有不当之处，还请“大虾米”们多多指教，谢谢！

`open` “文件.exe”，按`Ctrl+F8`运行。进入提示画面。

```
0042DEF1  |. E8 11E10000    CALL 0043C007
```

在`Ollydbg`中看到光标停在这行上，按下`F2`设置断点

```
0042DEF1  |. E8 11E10000
CALL 0043C007
```

重新用`Ollydbg`的`Debug`下的`Restore`装入程序，再按`Ctrl+F8`运行，光标再次停在了这行上，按下`F2`取消断点，再按6次`F7`后，再按`Ctrl+F8`

```
004463FF
|. FF50 50        CALL [DWORD DS:EAX+50]
```

这次停在了这里，按下`F2`设置断点，重新用`Ollydbg`的`Debug`下的`Restore`装入程序，再按`Ctrl+F8`运行，

```
004463FF  |. FF50 50
CALL [DWORD DS:EAX+50]
```

一样停要这里，按`F7`跟进,再按`Ctrl+F8` 继续

```
00418AA4
E8 67610200    CALL 0043EC10
      <-进入NAG窗口
00418AA9    83F8 01
CMP EAX,1
<-比较选择  继续与退出
00418AAC    74 2D
JE  00418ADB
<-继续就跳00418ADB
```

这次停在这里,仔细研究发现这里就是NAG窗口的`Call`，

改：

在 `00418AA4    E8 67610200    CALL 0043EC10` 行处直接按空格输入`NOP`,用`Shift`加光标选择红色行，点鼠标右键`Copy to executable file` 在新窗口的阴影行点右键 `save files` 保存

在 `00418AAC    74 2D` `JE  00418ADB` 行处直接按空格输入`JMP 00418ADB` ,用`Shift`加光标选择红色行，点鼠标右键`Copy to executable file` 在新窗口的阴影行点右键 `save files` 保存

时间限制已经解除，但试用版只能下50步棋。想了几天都不知道从什么地方下手解除，原来查看`win32api.chm`函数手册发现这个捕获鼠标的`SetCapture`函数，再次用`Ollydbg`运行，先下几步，在`Ollydbg`的`CPU`窗口中`Search for name(label)` 在`USER32.SetCapture`行点右键 `set breakpoint on every reference` 然后移动棋字，就可以跟到这里，要多跟几次。

```
0041CD3A  |. 8B5424 10      MOV EDX,[DWORD SS:ESP+10]
0041CD3E  |. 8BCE          MOV ECX,ESI
0041CD40  |. 52            PUSH EDX
0041CD41
|. 55            PUSH EBP
0041CD42  |.
C746 40 010000>MOV [DWORD DS:ESI+40],1
0041CD49  |. E8 12D0FFFF
CALL 00419D60
0041CD4E  |. 50
PUSH EAX
0041CD4F  |. 8BCE
MOV ECX,ESI
0041CD51  |. E8 5AFEFFFF    CALL 0041CBB0
0041CD56  |. 8B46 1C        MOV EAX,[DWORD DS:ESI+1C]
0041CD59  |. 50            PUSH EAX
0041CD5A  |. E9 BC000000    JMP 0041CE1B
0041CD5F
|> 83B8 8C110000 >CMP [DWORD DS:EAX+118C],64        <-
来回50步=100  100=64H
0041CD66  |. 0F8D 27010000  JGE 0041CE93
<- 等于 64 就跳
0041CD6C  |. 8B4E 74        MOV ECX,[DWORD
DS:ESI+74]
0041CD6F  |. 8BC3          MOV EAX,EBX
0041CD71  |. 2BC1          SUB EAX,ECX
0041CD73  |. 99            CDQ
```

改：

在 `0041CD66  |. 0F8D 27010000  JGE 0041CE93` 行处直接按空格输入`NOP`,用`Shift`加光标选择红色行，点鼠标右键`Copy to executable file` 在新窗口的阴影行点右键 `save files` 保存

好了，搞定收工!本次破解到此结束,因为是第一次写东西,有什么不对的地方请大家指出!谢谢了!

`OICQ:2806369`
