
# Now place this DLL somewhere safe
现在把这个 DLL 放到一个安全的地方

## Dump 基址要求
We now need to have 2 almost identical dumps ` `Same OEP, Header, Good IAT` ` but with different imagebases at the time of dumping.

We already have one from above, but that was made using `LoadDLL`.

现在我们需要 2 个几乎一模一样的 Dump 文件（同样的 OEP，Header,正确的 IAT)，但是转存的时候不能是相同的基址。

我们已经从上面得到了一个，但是她是用 `LoadDLL` 生成的.

We need a dump from the actual target program aswell, in this case `TibiaBot` uses a loader to inject `hook.dll` into the active process `Tibia.exe` ` `The Game this was designed for` `.

So we much load `Tibia.exe` into `OllyDBG` and run it, then goto Debugging Options and Select Break on New Module

我们需要一个最好从现行目标程序的 dump ，在这个案例中 `TibiaBot` 使用一个 loader 来注入 `hook.dll` 到一个激活的进程中 `Tibia.exe` ` `这是特意为这个游戏的` `.

所以我们必须载入 `Tibia.exe` 到 OD 并执行她,然后到 调试设置 ＞ 事件 >中断于新模块 ` `DLL` `

### 捕获注入时的入口点
Once this is done run `Loader.exe` ` `The program that injects the DLL into the process` ` and now `OllyDBG` should break showing the DLL being loaded.

Next to it there is a column called Entry, this shows the entrypoint of the DLL > Go to it

当这些完成后执行 `Loader.exe` ` `程序会把 DLL 注入到进程` ` 现在 OD 应该中断并显示 DLL 载入后的开始处

这有一列调用表,这些显示 DLL 的入口点 > 到那去

Once there place a breakpoint on it and then remove Break on new modules.

然后在那设置一个断点并且删除 "中断于新模块 ` `DLL` `"

Once it has broken at the entry point of the dll then you must follow all the steps from the beginning again until you have the 2 working dumps.

Remember to change the imagebase on the 2nd dumped dll to what it was in memory.

当中断在 DLL 的入口点后你必须再次重复前面所有的步骤直到你有了 2 个运行中的 dumps.

记住要修改两个 Dump 出的 DLL 文件的镜像基址为当时的内存基址

### Armadillo 断点绕过
PLEASE NOTE that `Armadillo` can ignore breakpoints on some targets if the option was enabled, if this is the case then Press `Ctrl+G` and type the API name ` `i.e VirtualProtect` ` and place a normal breakpoint on the retn of that function instead of BP FUNCTION\_NAME

请注意如果选项是钩选的  `Armadillo` 可以在一些目标上忽略断点,

如果发生这种情况可以按 `Ctrl+G` 并输入 API 名 ` `如：VirtualProtect` `，并在函数的 `retn` 处下一个普通的断点来代替 BP 函数名这种方式.

## 使用 ReloX 修复
Once you have your 2 dll's then we need to use a wonderful tool called `ReloX` by `Mackt`.

Load both Dll's into the program:

当你有了 2 个 DLL 后，我们需要一个非常好的 `Mackt` 写的工具叫 `ReloX`

载入 2 个 DLL 到程序中

![](http://bbs.pediy.com/upload/2006/8/image/relox.png)

### 选择段
Click Select sections and remove the ticks from any sections that arent used for code or are part of `Armadillo`.

I only ticked:

`CODE`

`DATA`

`.idata`

`.text`

点 Select sections 并从所有的段中去掉不是代码所使用钩选或者是属于 `Armadillo` 的部分

我只钩选了:

`CODE`

`DATA`

`.idata`

`.text`

Once that is done click Compare:

完成后点击 Compare:

![](http://bbs.pediy.com/upload/2006/8/image/relox2.png)

Now you can click Fix PE Module and choose the same dll you opened first in ReloX

` `At the top of the screen aka Original` `.

现在你可以单击 Fix PE Module 并选择你用 `ReloX` 先打开的相同的 DLL

Now Test...

现在测试

![](http://bbs.pediy.com/upload/2006/8/image/working.png)

Looks like its working fine!

看起来运行的还不错!

### 清理段
Last thing just for neatness you should remove all the unused sections from your file.

I have only these sections left:

最后一件事就是整理，你应该从你的文件中删除所有未使用的段

我仅剩下这些段:

`CODE`

`DATA`

`.idata`

`.rsrc`

`.mackt`

`.reloc`

Well Done!

You have successfully unpacked an `Armadillo` DLL!Greets

`TeamICU`, `TSRh`, `SnD`, `FFF`, `SEC-8`, `MP2K`, `ARTeam`, `AHTeam`, `Revenge`, `{RES}` and all other Teams!Written by: `Whiterat` //ICU

8th March 2006

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [unpacking armadillo 3.xx dll.rar](javascript:void(0);)

  （295.49kb，80次下载）
