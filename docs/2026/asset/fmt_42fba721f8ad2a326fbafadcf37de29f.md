
```
12=$.obj,O,$B\LCC.EXE /g2 /O /unused /c /errout=error.txt /I"$I",$.cpp
13=$.exe,O,$B\LCCLNK.EXE /SUBSYSTEM WINDOWS,$.obj,$.res
14=0,0,,$.exe
15=rsrc.obj,O,$D\CVTRES.EXE,rsrc.res
16=\*.obj,O,$B\LCC.EXE /g2 /O /unused /c /errout=error.txt /I"$I",\*.cpp
17=0,0,"$E\OllyDbg",$.exe
```

//这里是默认设置，就是你不在`RadASM`中建立工程，只用`RadASM`打开源文件执行编译时的配置。这里和下面一段可说是最重要的部分了，请注意看，我一条条的讲解，尽量说的清楚点。先说一下可能用到的几个特殊字符的含义：

`$`：从`RadASM`编辑器的最顶部标题栏中获取的不带扩展名的文件名。如打开`test`工程下的名为`test.cpp`的文件，这里的`$`就相当于`test`，就等于你不带扩展名的文件名称。下面会详细说明。

`|`：这个符号将在实际编译工程时替换为“,”(逗号)。如`$D\RC.EXE /v,rsrc.rc|test.rc`实际编译时就会是这样(假定`RadASM`装在E盘根目录下)：`E:\RadASM\Addins\RC.EXE /v rsrc.rc,test.rc`

`\*`：这个符号在编译模块(moudle)时使用。

`MenuMake=1,1,1,1,1,1,1,0,0,0` 这一条的意思是在构建菜单上可用的子菜单项目。我们来看一下上面的`Project`段中的`MenuMake`：

`MenuMake=编译资源脚本,编译,连接,构建,构建并运行,运行,在调试器中运行,全部构建并运行,编译模块`

我们数一下，每个逗号分隔的算一个项目，共有9个。再看`MenuMake=1,1,1,1,1,1,1,0,0,0`共有9个。这里个数无所谓，1是指对应的子菜单激活，0是不激活。这样`MenuMake=1,1,1,1,1,1,1,0,0,0`等同于

`MenuMake=编译资源脚本,编译,连接,构建,构建并运行,运行,在调试器中运行`共7个子菜单激活。

`MenuMake=1,1,1,1,1,1,1,0,0`下面的1到17是编译选项，其中1到7是编译为`Release`(发行版)时的设置，11到17是指编译为`Debug`(调试版)时的设置。这里只说1到7的含义，11到17与1到7的类似。首先说一下各个序号的含义：`1`是指资源编译选项；`2`是指编译选项；`3`是指连接选项；`4`是指运行编译好的程序；`5`是用于`CVTRES`的，转换`res`文件到`obj`文件；`6`是编译模块选项；`7`是指在调试器中运行。1到7的含义如下：

`1=rsrc.res,O,$D\RC.EXE /v,rsrc.rc`

//这条用于编译资源，用逗号分隔每个项。`rsrc.res`是指在执行编译前如果有这个文件就先删掉。`O`(注意这里不是`0`，是大写的字母`O`)是指在`RadASM`的输出窗口中显示所有执行命令行时的信息。如果这里是大写字母`C`，则是指在控制台中显示信息，不在输出窗口中显示。其它两个参数`OT`和`CT`与`O`和`C`含义类似。`$D\RC.EXE /v`是指执行`RadASM`的`Addins`目录下的`RC.EXE`来编译后面的`rsrc.rc`到`rsrc.res`。`/v`是`RC.EXE`的命令行选项。`LCC`自带有一个资源编译程序，我原来是这样配置的：`1=$.res,O,$B\LRC.EXE /v,$.rc`这里的`$`是指从`RadASM`编辑器的最顶部标题栏中获取的不带扩展名的文件名。如我打开一个工程中的文件，`RadASM`的顶部标题显示如下：

`RadASM - dialog - [E:\RadASM\Lcc\projects\dialog\dialog.cpp]`

此时`$`取的内容就是`dialog.cpp`中的`dialog`(注意：在`RadASM`中新建一个工程时有两个选项：一个是工程名称，一个是工程说明。打开一个工程时若不打开任何文件的话，在`RadASM`的顶部显示的是工程说明的内容，但此时你进行编译的话`RadASM`取的是工程名称。如我建的工程名称是`dialog`，工程说明是对话框示例，打开这个工程不打开任何文件时`RadASM`顶部显示的就是`RadASM - 对话框示例`，但在编译工程时`$`还是用的工程名称`dialog`)。这样如果编译上面的示例工程的话，`1=$.res,O,$B\LRC.EXE /v,$.rc`就相当于这样(假定你的`LCC`编译器的目录放在放在E盘的根目录下，如：`E:\LCC`，这里的`$B`是从上面的`[Paths]`段中来的，你配置好了的话`$B`就等同于`E:\LCC\Bin`目录)：

`1=dialog.res,O,E:\LCC\Bin\LRC.EXE /v,dialog.rc`

为什么没再用`LCC`原来自带的资源编译器呢？主要是我配置好后测试时，我明明在对话框中选的是宋体，9号，可编译后的程序按钮上显示的不知是什么字体，而且还是个斜体。搞得我还以为那里弄错了，查了半天。后来看`LCC`的用户手册，才知道它的资源编译器还不支持字体，说正在改进中。为此，我就放弃了它的资源编译器，还是用微软的吧。这里注意要把`RC.EXE`复制到你`RadASM`的`Addins`目录下。如果你是用我做的汉化版的话，这一步可省略。

`2=$.obj,O,$B\LCC.EXE /O /unused /c /errout=error.txt /I"$I",$.cpp`

//这条用于编译程序，`$.obj`、`O`两个与上面讲的类似，也是先删除你所做工程名字`.obj`，在输出窗口中输出信息。`$B\LCC.EXE /O /unused /c /errout=error.txt /I"$I"`是编译选项，后面的`/I"$I"`是查找`include`目录用的。`$I`就是`LCC`下的`include`目录，如`LCC`装在E盘根目录的话(下面我再讲的时候都默认你的`RadASM`和`LCC`都装在E盘根目录下)，`$I`就代表`E:\LCC\include`目录。其实上面这句在下面具体定义的工程项中还可以这样写：

`2=3,O,$B\LCC.EXE /O /unused /c /errout=error.txt /I"$I",2`

这里与上面那句不同的地方就是把`$.obj`换成了`3`，`$.cpp`换成了`2`。为什么可以这样换？这是根据`[MakeFiles]`段中定义的文件类型对应的序号来的。看一下`[MakeFiles]`段，其中`2=.cpp`、`3=.obj`这里的替换后的`2`和`3`就是对应你工程名的`cpp`和`obj`文件，与`$.cpp`和`.obj`功能相同。

`3=$.exe,O,$B\LCCLNK.EXE /s /SUBSYSTEM WINDOWS,$.obj,$.res`

//这条用于连接程序。各条对应情况与前面的两条类似。假设你象我一样测试的是`E:\RadASM\Lcc\Projects\dialog\dialog`下的`dialog`工程，`$B\LCCLNK.EXE /s /SUBSYSTEM WINDOWS,$.obj,$.res`执行后效果是这样：

`E:\Lcc\Bin\LCCLNK.EXE /s /SUBSYSTEM WINDOWS "dialog.obj" "dialog.res"`

同样你也可以在具体项目定义中把`$.exe`对应换成`5`，`$.res`对应换成`4`。

`4=0,0,,$.exe`

//这条命令是运行编译好的程序。这里的`0,0`(注意是数字`0`)，照抄别的就可以了。只是如果编译的是控制台程序的话，为防止运行时看不到效果，可这样写：`4=0,0,$D\RunStub,$.exe`这里的`RunStub.exe`是我上次发现在`RadASM`中运行dos程序看不到效果，就跟`RadASM`的作者说了一下，他提供的这个小工具，现在已包含在`masm`的`Projects`目录中了。把它复制到`Addins`目录即可。另外你也可以用`LCC`的`Bin`目录下的一个`rundos.exe`小工具，不但支持按任意键退出，还可显示执行时间，推荐。可以在控制台工程中这样配置：`4=0,0,$B\rundos.exe,$.exe`

`5=rsrc.obj,O,$D\CVTRES.EXE,rsrc.res`

//这条命令是用`CVTRES.EXE`转换`.res`文件到`.obj`文件的。我把第三项修改为`$D\CVTRES.EXE`是因为`CVTRES.EXE`是微软的程序，`LCC`中没有。我把这个程序复制到了`Addins`目录。如果你的`Addins`目录中没有这个文件的话请把这个文件从`masm32`的`bin`目录下复制过来。

`6=\*.obj,O,$B\LCC.EXE /O /unused /c /errout=error.txt /I"$I",\*.cpp`

//这条命令是用于编译模块的，其中的`\*.cpp`中的“`\*`”只在编译模块(module)时使用。一般可用于编译你添加的其它源文件。

`7=0,0,"$E\OllyDbg",$.exe`

//这条命令是用调试器打开你编译好的程序。照抄即可。

```
[Win32 App]

Files=1,1,1,0,0,0,0,0
Folders=1,0,1
MenuMake=1,1,1,1,1,1,1,0,0,0

;x=FileToDelete/CheckExistsOnExit,
;(O)utput/(C)onsole/0,Command,
;MakeFile1[,MakeFile2[,MakeFile3...]]

1=4,O,$D\RC.EXE /v,1
2=3,O,$B\LCC.EXE /O /unused /c /errout=error.txt /I"$I",2
3=5,O,$B\LCCLNK.EXE /s /SUBSYSTEM WINDOWS /o "$5",3,4
4=0,0,,5
5=rsrc.obj,O,$D\CVTRES.EXE,rsrc.res
6=\*.obj,O,$B\LCC.EXE /O /unused /c /errout=error.txt /I"$I",\*.cpp
7=0,0,"$E\OllyDbg",5
11=4,O,$D\RC.EXE /v,1
12=3,O,$B\LCC.EXE /g2 /O /unused /c /errout=error.txt /I"$I",2
13=5,O,$B\LCCLNK.EXE /SUBSYSTEM WINDOWS /o "$5",3,4
14=0,0,,5
15=rsrc.obj,O,$D\CVTRES.EXE,rsrc.res
16=\*.obj,O,$B\LCC.EXE /g2 /O /unused /c /errout=error.txt /I"$I",\*.cpp
17=0,0,"$E\OllyDbg",5
```

//这个段就是具体工程的配置。`[Win32 App]`就是你在`[Project]`的`Type`中定义的工程项目。配置方法可以参照上面的`[MakeDefNoProject]`段的设置，再对具体工程进行修改。下面的具体各个段我就不列出来了，具体内容可以看一下我汉化的`RadASM 2.2.0.5b`中的`lcc.ini`文件。

```
[MenuMake]

1=编译资源脚本(&C),372,M,1
2=编译(&A),116,M,2
3=连接(&L),1652,M,3
4=构建(&B),1396,M,2,3
5=-,0,M,
6=构建并运行(&G),628,M,2,3,4
7=-,0,M,
8=运行(&R),884,M,4
9=在调试器中运行(&D),580,M,7
10=-,0,M,
11=全部构建并运行(&A),1140,M,1,2,3,4
12=-,0,M,
13=编译模块(&M),0,M,6
Ver=100
```

//这里就是你在`RadASM`的构建菜单上看到的子菜单，可以照抄后修改。其中的如`5=-,0,M,`是在子菜单之间加一条分隔线。

```
[Color]

...
```

//这个段是用于在`RadASM`中设置语法高亮及背景色等颜色的，具体内容我不列了，可在颜色及关键字选项中设置。可先照抄一个再在`RadASM`中修改。

```
[KeyWords]

...
```

//这个段是设置语法高亮的关键字。照抄后自己手工修改。也可在`RadASM`的颜色及关键字选项中设置。

```
[ReallyRad]

...
```

//`ReallyRad`插件配置，可在`ReallyRad`插件的选项中修改。照抄。

```
[RadHelp]

...
```

//`RadHelp`插件设置，用于帮助文件。照抄后自己修改，也可在`RadHelp`插件中修改。

```
[MenuMacro]

...
```

//宏菜单，照抄，可在`RadASM`的选项中设置。

到这里已经全部配置完了，现在就是测试。先把配置好的文件(我这里是`lcc.ini`)放到`RadASM`目录下，在`RadASM`的选项->编程语言中添加一下。再在`RadASM`的目录下新建一个文件夹，我们这里命名为`Lcc`，按照别的编译器文件夹的结构依次建立好子文件夹。建好文件夹后在`RadASM`的选项中把路径设置好，现在我们就可以建立一个工程来测试了。不知大家注意到没有，我在编译选项中是这样设置的：`$B\LCC.EXE /O /unused /c /errout=error.txt /I"$I"`。这个`/errout=error.txt`选项是把出现的错误和警告写入到`error.txt`文件中。原来我是没添加这个选项的，但在`RadASM`中编译一个没有任何错误的工程时输出窗口中的编译进度条一直显示进程，没法完成。改为控制台输出则可以。但控制台输出的话我将看不到错误信息，后来加了这个`/errout=error.txt`参数才解决了这个问题。我这里想说的就是出现问题时先不要急，再研究研究命令行选项，可能就可以解决了。测试中还有一个问题就是我要是把连接选项设成这样：

`3=5,O,$B\LCCLNK.EXE /s /SUBSYSTEM WINDOWS /o "$5",3,4`

如果我编译的程序中根本没有资源的话将会出错。这样只好建两个项目，一个是有资源的，一个是无资源的。有资源的就用上面这条连接选项，无资源的用这条连接选项：

`3=5,O,$B\LCCLNK.EXE /s /SUBSYSTEM WINDOWS /o "$5",3`

这样就行了。现在没有错误的工程我们都编译成功了，再来测试一下源码中有错误的工程编译时的情况。我把源程序中这句`return FALSE;`后面的分号删掉，变成这样：`return FALSE`再编译一下，在输出窗口中显示错误如下：

```
1 error, 0 warnings

Error dialog.cpp: 30 Syntax error; missing semicolon before `}'
```

好，我在第二行的`Error dialog.cpp`上双击一下，看看能不能跳到出错的位置。很不幸，出现一个错误：“无法打开文件`E:\RadASM\Lcc\Projects\dialog\`”为什么？没办法，请出`OllyDBG`来调试一下`RadASM`。先用`spy`查找一下输出窗口的句柄，再在`OllyDBG`中设个`203` `WM_LBUTTONDBLCLK`的消息断点(这个断点的意思就是如果在输出窗口中双击鼠标左键时中断)，先在`RadASM`中打开其它编译器中可以正常跳转到出错行的工程，双击输出窗口中的错误，被`OllyDBG`拦下。分析一下，再打开我们自己新建编译器中的工程，双击出错行再进行分析。具体分析过程我不写了，最后得出的结论是由`RadASM`目录下的`RAEdit.dll`控制错误跳转，它只认这种格式的错误：

`dialog.cpp(31) : error C2143: syntax error : missing ';' before '}'`

就是说格式为：源文件名(出错行号):出错信息。而`LCC`的错误信息格式为：`Error 源文件名: 出错行号 出错信息`

`RadASM`把前面的`Error`作为源文件名，当然打不开文件。这时怎么办呢？一个方法就是自己写一个插件专门处理新建编译器的出错信息跳转，另一种方法就是给`RadASM`的作者写封信，让他来帮助解决。第一种太烦，我不干，我选第二种。只要让`RadASM`的作者修改几条代码就行了。另外别忘了把你配置好的文件发给他(这里要注意把配置文件中的中文都改成英文，编辑器中的选项都改为对应英文环境的，我们也要国际化嘛^\_^)，让他集成到`RadASM`中去。

提示：根据`RadASM`的最新版本`2.2.0.6`的说明，现在可以把`1=4,O,$D\RC.EXE /v,1`这样的编译配置改为`1=4,O,$B\LRC.EXE /v,1`，作者在对话框编辑器中提供了一个选项“限定字体信息”可以保证用`LRC.EXE`编译的资源字体显示正常。`2.2.0.6`版已完全支持`LCC`。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
