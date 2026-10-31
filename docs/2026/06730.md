
# `vfp&exe`加密程序破解实录 (1千字)

`vfp&exe`加密程序破解实录

`vfp&exe.exe` v1.8.5 演示版

用`vfp6`编译一应用程序`myvfp.exe`，运行`vfp&exe`对`myvfp.exe`进行加密。运行加密后的`myvfp.exe`，出现输入密码窗口（如加密时选择了密码项）和一个演示版的提示窗口。

去除`vfp&exe`加的壳：

用`trw2000`调入加密后的`myvfp.exe`，下断点`bpx 401760`（`vfp6`编译的文件入口处），按几次f5。出现输入密码窗口时输入密码，当出现演示版提示窗口时按确定，再按f5键。

A！记下`trw2000`横线上的当前文件名，`suspend`回到windows。

运行`ollydbg`，选file菜单下的attach子菜单，在弹出的窗口中选A！处记下的文件名后击attach钮。

右击左上方窗口，选go to -->address，输入`401760`，击ok钮。右击弹出的窗口，选copy to exxcutable file。右击弹出的窗口，选backup-->save data to file，选一个存放文件的路径和文件名（`unmyvfp.exe`），击打开钮。

运行`prodump.exe`，选菜单编辑pe文件，将文件`unmyvfp.exe`的入口地址由`48e8`改为`1760`后存盘退出。

（或者：运行`winhex`改文件入口偏移。调入`unmyvfp.exe`，取地址`3CH`处的值，为`B8H`，计算`B8H+28H=E0H`，改`E0H`处的`E848H`为`6017`------程序入口处。）

运行`winhex`，寻找`00FEF2`，删除第二次找到的地方的`FF`处开始到文件结束的所有数据后存盘退出。

用`unfoxall`对`unmyvfp.exe`就可进行反编译了！！！

此法完全用于学习，不得用于非法用途！

好了搞定这个东西耗费了我两天的时间,眼睛受不了了.....

lpyxt 写于2001年8月16日17时20分

email:liupeiyuxt@km169.net
