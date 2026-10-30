
![](https://bbs.kanxue.com/upload/attach/202002/531365_dkxg9e5i4br5hoi.png)

放置完成后，直接运行`decoder.exe`，则会进行批量反编译，将`luaJit`下面的所有的字节码文件进行反编译，最后在`data`文件夹的`out`目录中生成相应反编译后的文件。如果出现解密失败，那么就必须要考虑上面提到的问题，是否是版本不匹配，作者是否是修改了`luajit`虚拟机的源代码，导致字节码文件中的一些字节被修改或者替换，从而需要进行修正后在进行解密。该工具引用自看雪文章“浅析`android`手游`lua`脚本的加密与解密”作者所提供。在附件中也会提供相应的下载，解密本题目时也是使用了该工具。

## 3、题目分析和解题思路

### 3.1 题目分析

现在该轮到题目分析了，这个题目放出来两个小时就被id为“风间仁”的选手搞定，如本次比赛组织者“netwind”所说“已经到了在用意念做题”的地步，膜拜一下。我想攻击者这么牛，也是经过很多磨炼和无数的努力才能达到此登峰造极的地步。所谓“台上一分钟，台下十年功”，做任何事情都不容易，但只要努力就肯定能战胜。

下面开始分享一下自己的解题过程，拿到题目直接运行，随便输入序列号，提示`Wrong!`

![](http://bbs.pediy.com/upload/attach/201707/531365_0y5wahsd669ngm3.png)

于是按照常规思路开始上调试器，拖到`IDA`里面静态分析，发现被加壳，开始跟踪分析用普通的`OllyICE`被提示检测到调试器：

![](http://bbs.pediy.com/upload/attach/201707/531365_gbc9qw44js6xwva.png)

于是换上`HawkOD`，以及`strongOD`等插件，反调试被搞定，于是脱壳，试了所有能够尝试的各种脱壳手段，发现不行，于是单步跟着走，发现调用`VirtualAlloc`申请空间，然后利用拷贝数据到申请空间中，最后有利用`VitualProtect`改变代码段的权限，将数据写入到`.txt`代码段中，跟踪到`0x00401000`等位置处被解密出来，以为字节能搞定，跟着程序继续走，但是到后面又跑丢了或者程序异常，搞了半天也没搞定，于是放弃纠缠。换思路，不再纠缠于于脱壳，直接将程序运行起来，然后附加上`HawkOD`，这时一切正常，看了一下`0x00401000`等位置处的相关代码，发现也和之前分析到的解密后代码相同，初步判定解密完成。为了方便了解程序流程，还是从`0x00401000`出开始进行了内存`dump`，拖到`ida`中，发现还是很大部分函数能够解析，但后面分析发现这些`dump`出来的内容对分析程序帮助不大，这种方法在对付一些已经过了反调试，但是无法进行完整脱壳时还有很有用的，在实际分析中，经常会遇到很多壳无法完整的跟踪或者脱掉，那么就等程序运行起来后，在附加上调试器，然后直接从内存中`dump`出代码，拖到`ida`里面查看，很多时候我们只需要分析到程序的核心跳转流程就行了，并并不需要完整的脱壳。

![](http://bbs.pediy.com/upload/attach/201707/531365_rcxw0wizjp2cedg.png)

于是开始在调试器里面分析，大体思路就是利用`ALT+M`在，然后`ctrl+B`在内存中搜索相关的字符串，比如输入的字符串，或者提示`Wrong!`等字符，然后设置硬件内存断点，查看程序访问的位置，经过多次调试分析，总共设置了20多个断点，大概分析到一些关键代码位置，但是刚开始分析了很久还是不明白，由于刚开始不对`Lua`不熟悉，经常跟踪到`lua`的虚拟机中，然后半天被绕晕，不知所云。下面大概截取一些在分析过程中通过设置断点定位到的关键代码位置：

位置1：`0x40103d`，写入`luaJIT`字节码文件，在这里向`esp+0x36`位置写入字节码问文件内容，`esp+0x30`中存放的是字节码文件开始的六个字节，其开始为`\x1B\x4C\x4A`

![](http://bbs.pediy.com/upload/attach/201707/531365_zuajfn23smz9fu0.png)

写入完成后总共有`0x275`个字节的字节码文件，写入完成后内存中的字节码文件内容如下所示：

![](http://bbs.pediy.com/upload/attach/201707/531365_e09f9roxkly31rc.png)

但是由于不熟悉`LuaJIT`字节码文件，依然没看明白是啥东西，尴尬！然后接着分析：

![](https://bbs.kanxue.com/upload/attach/201707/531365_ztitpascyopesn6.png)在`0x4021CF`

在写入字节码文件中，后面的函数分别是完成了`LuaJIT`虚拟机初始化以及`Loal_loadbuffer`加载`luajit`字节码文件等，入上图所示。其中在`0x004021CF`处调用函数`sub_00412270`是加载了`luaJIT`的相关库，并完成了初始化等工作。开始不熟悉，跟进了该函数，走了半天，也正是该函数里面的内容给予了自己的提示，因此遇到不明白的就多跟着程序走，多走几次，总会看出点东西来，程序使用了混淆技术，走了很久终于发现了一些有意思的东西，在`0x42a088`的`Call EDI`会被调用多次，里面进去后会出现很多有意思的字符串。后面经过和源码确认是此处循环多次执行是加载了多个`lua`虚拟机的核心的库。。

![](https://bbs.kanxue.com/upload/attach/201707/531365_kt1rti407j590na.png)

第一次进入后出现了一些关键字符串，如下所示

![](http://bbs.pediy.com/upload/attach/201707/531365_1ncaoqhh33doxx5.png)

有`Lua 5.1 coroutine`，这些信息本来可以作为提示了其可能使用了`Lua` 虚拟机，但是由于不熟悉了，还没是没有识别出来，但是后面经过分析其使用的是`Luajit beta 2.1.0`的虚拟机，这个函数多次被调用，第二次调用相关信息如下：

![](http://bbs.pediy.com/upload/attach/201707/531365_qxn6pdvkgxxndvg.png)

![](http://bbs.pediy.com/upload/attach/201707/531365_rujxl3hm4dtkxp3.png)

有大量的`lua`字符串出现，这次开始觉得有点可疑，应该是使用了什么工程的源代码。这个给一个自己以前分析的经验，一般在程序中看到有很多有意义的字符串，但是又不像是作者自己写的，那多半是用了什么开源库，之前自己分析一个加密软件时，发现其使用了很多`crypt`先关的字符串，利用字符串在`google`进行搜索，确认了是使用了`openssl`库，一般确认了使用什么开源库时，分析就简单了，这是只需要找到源码，然后对应分析知道其调用的函数，最后分析就方便很多了。同样的思路，直接上谷歌，关键词`crackme` 加 `lua`，一搜索，出现的居然是看雪`CTF2016`第二题的信息，初略看了下，知道了`lua`在`crackme`中的应用，但是自己去年没做啊。

![](http://bbs.pediy.com/upload/attach/201707/531365_ko10q72e5nzlwi9.png)

继续搜索，发现了`HighHand`大神关于关于第二题的解题思路（<http://bbs.pediy.com/thread-213697.htm>），里面提到标准的`luac`字节码文件开始为`0x1B,0x6C,0x73,0x11`，只是在那个题目中作者将其修改成了`0x1B,0x6C,0x73,0x11,0x00`；

![](http://bbs.pediy.com/upload/attach/201707/531365_zu2tdg4l4i6ohsr.png)

但是和本题目的开始的几个字节也对不上啊，于是继续分析，在后面`0x42A088`处有多次调用了`call edi`这个函数，后来经过确认是调用的`luaL_openlibs`这个函数里面的，分别加载了相关的模块，下面是调用`luaopen_table`：

![](http://bbs.pediy.com/upload/attach/201707/531365_8mq3o6ddyx4o7zf.png)

调用`luaopen_io`：

![](http://bbs.pediy.com/upload/attach/201707/531365_15ay4naynlfm0e8.png)

调用`luaopen_os`：

![](http://bbs.pediy.com/upload/attach/201707/531365_bjx2uf8oq557au0.png)

调用`luaopen_string`：

![](http://bbs.pediy.com/upload/attach/201707/531365_52tkq9td749x2ty.png)

调用`luaopen_math`

![](http://bbs.pediy.com/upload/attach/201707/531365_a4t4i83sys154kk.png)

调用`luaopen_debug`

![](http://bbs.pediy.com/upload/attach/201707/531365_ulkepg45922ebnn.png)

调用`luaopen_bit`：

![](http://bbs.pediy.com/upload/attach/201707/531365_6x7f3330hpdf2cx.png)

上述过程后来经过下载`luajit beta 2.1,0`的源代码比对确认后确认实际上执行`luaL_openlibs`，这里面通过调用`lj_lib_load`这个函数，分别执行了上述功能。

![](http://bbs.pediy.com/upload/attach/201707/531365_l4evs7wa5q7wboq.png)

![](http://bbs.pediy.com/upload/attach/201707/531365_xac3wyuw3hm97p3.png)

在最后一个执行`luaopen_jit`时出现了`LuaJIT 2.1.0-beta3`字符串，这里才提醒了自己，于是上`google`查选，发现`LuaJIT 2.1.0-beta3`是`lua`虚拟机的另外一个版本，而且起`luaJIt`脚本文件开始确实也是`0x1B,0x4C,0x4A`，越是下载了`luaJIT 2.1.0-beta3`的源码，经过字符串对比分析后才还原确认上前面分析的函数。

下面就是调用最后一个`luaopen_jit`函数。

![](http://bbs.pediy.com/upload/attach/201707/531365_mb6fambayrb70q1.png)

这是在源码该函数，可以和上述匹配起来。

![](http://bbs.pediy.com/upload/attach/201707/531365_k97tr55homo2vgm.png)

由于确认了上述信息，通过了解可知，其肯定会调用`luaL_loadbuffer`动态加载字节码文件，于是进一步确认了`0x004021e0`执行的是：`luaL_loadbuffer`函数。

![](http://bbs.pediy.com/upload/attach/201707/531365_krqme5lqrgxqzha.png)

![](http://bbs.pediy.com/upload/attach/201707/531365_h9x3wco7tu1z5xy.png)

于是从内存中`luajit`字节码文件提取出来并进行反编译，使用的是`luajit-decomp`来对文件进行反编译，放在了附件中可以下载后使用，具体请查看`2.3节`的使用。解密最后得到`lua`脚本如下：
