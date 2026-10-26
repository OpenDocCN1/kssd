
## Script

```
=============Script Begin================

var addr

var byte



eob lableb



lableb:

mov addr, esp

add addr, 8

mov addr, [addr]

mov byte, esp

add byte, 0c

mov byte, [byte]

mov byte, [byte]

and byte, FF

add byte, ";"

add addr, ", (LPVOID)(&var), 1, NULL);"

wrta "F:\mh120e5\out.txt", "var = 0x"+byte

wrta "F:\mh120e5\out.txt", "WriteProcessMemory(g_hProcess, (LPVOID)0x"+addr



run

=============Script End==================
```

我们可以用获取到的补丁数据来做一个自己的`MapHack`了

## anti-maphack ^\_^

想屏蔽掉`maphack`的功能只要每隔一段时间就把游戏数据恢复一下即可。使用之前获取补丁数据的方法同样可以获取恢复数据。

### 恢复数据

```
while (1)

{

 if (检测到关键代码被修改)

    将恢复数据写回游戏内存，覆盖掉补丁数据

 Sleep(xxxx);

}
```

但是这个方法也并不是什么好方法，如果`MH`用更高的频率往游戏里写补丁数据的话就没辙了。(`代码见anti-mh_1文件夹`)

其实`anti-maphack`中心思想就是不让他把关键数据补丁掉，如果我们把关键数据移位了呢？那么他补丁的数据就会写到错误的地方，导致程序崩溃，这也算达到了我们的目的了。

我这里使用的方法是将一处`MapHack`要补丁代码拷贝到别的地方，然后从原来的地方跳转过去执行。如此，当`MapHack`补丁的时候就会把跳转语句覆盖掉，如此就会导致程序执行出错，也算是达到目的了。

这里选择`6F4069F7`处来做个演示，在`6F704BAB`有一段足够长的空闲内存

从`6F4069F7`跳转到`6F704BAB`的指令：`E9 AF E1 2F 00`

原始指令：`66 8B 50 3C 66 89 55 E4`

从`6F704BB3`跳转到`6F4069FF`的指令：`E9 47 1E D0 FF`

(详细代码见`anti-mh_2文件夹`)

开启`MapHack`功能前运行`anti-mh.exe`，执行完毕后进入游戏一切正常，然后开启`MapHack`功能，游戏崩溃，试验成功:)

当然这个演示还是很简单的，如果`MapHack`想绕过这个防御只需要读`6F4069F7`处的代码，然后根据该处的代码定位到关键数据所在地。

不过我们可以设计一套算法来确定关键代码要移动到哪里去，然后设计一个`locate`函数来专门负责定位到这些代码，关键代码原来的地方就改成对`locate`的调用，然后加强对`locate`函数的保护，即可大大增加`MapHack`的制作难度![](https://bbs.kanxue.com/view/img/face/16.gif)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [1.rar](javascript:void(0);)

  （194.08kb，76次下载）
