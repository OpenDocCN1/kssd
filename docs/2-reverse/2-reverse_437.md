

# 关于“IDA Pro 5.4 中文帮助手册.chm”在 IDA 打开文件时按 F1 出现错误的解决方法

> 原文：[`www.pediy.com/kssd/pediy10/92503.html`](https://www.pediy.com/kssd/pediy10/92503.html)

关于“IDA Pro 5.4 中文帮助手册.chm”在 IDA 打开文件时按 F1 出现错误的解决方法  
   
首先感谢“看雪”提供的中文翻译版帮助文件，同时也感谢各位翻译人员的辛勤劳动。  
中文版的帮助有助于迅速普及国人学习 IDA 的使用。一个小遗憾就是在 IDA 打开数据文件  
时按快捷键 F1 出现“HH\_HELPCONTEXT called without a \[MAP\] section”错误信息。  
现在提供该错误的解决办法。  
   
所用的工具是 Far Helpware。其实问题很简单，就是在用 FAR 的项目编辑器(HH Project Editor)打开项目  
文件时的“Other Sections”选项下定义\[MAP\]和\[ALIAS\]两个节。我们首先要学习一下\[MAP\]和\[ALIAS\]的  
语法，在 Far 的帮助文件(Help)内搜索“\[MAP\]”, 可找到如下信息：  
   
\===============================================================================================================================  
\[MAP\] \[ALIAS\]  
The Map and Alias sections must be set if you want to create a Context ID to URL mapping. Used when the application opens help using the HH\_HELP\_CONTEXT help command. ie. For opening help using an HelpContext / integer ID.  
   
EG. HtmlHelp(0, "c:\\app\\help\\Help.chm", HH\_HELP\_CONTEXT, 1001);  
   
The \[MAP\] section defines the IDs.   
The \[ALIAS\] section defines the mapping between ID and help topic.  
   
Example MAP and ALIAS section  
\[ALIAS\]  
IDH\_HomePage=default.htm  
IDH\_TestTopic1=htmlfiles\\testtopic1.htm  
IDH\_TestTopic2=mychm::htmlfiles\\testtopic2.htm  
   
\[MAP\]  
#define IDH\_HomePage 1000 ;comment  
#define IDH\_TestTopic1 1001  
#define IDH\_TestTopic2 1002  
   
Example MAP and ALIAS section using #include statements;Use the #include statement to include an external header file at compile time.  
   
\[ALIAS\]  
#include myhelp.ali  
   
\[MAP\]  
#include myhelp.h  
   
where myhelp.ali is a text file containing  
   
IDH\_HomePage=default.htm  
IDH\_TestTopic1=htmlfiles\\testtopic1.htm  
IDH\_TestTopic2=mychm::htmlfiles\\testtopic2.htm  
   
and myhelp.h is a text file containing  
   
#define IDH\_HomePage 1000  
#define IDH\_TestTopic1 1001  
#define IDH\_TestTopic2 1002  
\===============================================================================================================================  
   
内容翻译如下：  
\===============================================================================================================================  
要想创建上下文 ID 与 URL 关联信息，就必须设置 Map 和 Alias 节，这两个节用与应用程序在打开帮助文件时使用"HH\_HELP\_CONTEXT"帮助命令的情况。也就是说使用一个  
"帮助上下文"与"ID 号"关联对（ID 号为一个整数类型的数值）来打开帮助信息。  
   
例如命令：  
HtmlHelp(0, "c:\\app\\help\\Help.chm", HH\_HELP\_CONTEXT, 1001); // 帮助上下文为“HH\_HELP\_CONTEXT”，“ID”为 1001。  
\[MAP\]节定义 ID 值。  
\[ALIAS\]节定义 ID 与帮助主题的映射关系。  
   
直接定义 MAP 与 ALIAS 节的例子：  
\[ALIAS\]  
IDH\_HomePage=default.htm  
IDH\_TestTopic1=htmlfiles\\testtopic1.htm  
IDH\_TestTopic2=mychm::htmlfiles\\testtopic2.htm  
   
\[MAP\]  
#define IDH\_HomePage 1000 ;comment  
#define IDH\_TestTopic1 1001  
#define IDH\_TestTopic2 1002  
   
此外还可以用类似 C 语言的#include 语句在编译帮助文件时包含外部定义的头文件来定义\[MAP\]和\[ALIAS\]节，相应的例子如下：  
\[ALIAS\]  
#include myhelp.ali  
   
\[MAP\]  
#include myhelp.h  
   
其中 myhelp.ali 为一个纯文本文件，其内容如下：  
IDH\_HomePage=default.htm  
IDH\_TestTopic1=htmlfiles\\testtopic1.htm  
IDH\_TestTopic2=mychm::htmlfiles\\testtopic2.htm  
   
而 myhelp.h 也是一个纯文本文件，其内容如下：  
#define IDH\_HomePage 1000  
#define IDH\_TestTopic1 1001  
#define IDH\_TestTopic2 1002  
\===============================================================================================================================  
   
下面的问题就简单了, 只要在 Other Sections 的"\[ALISAS\]"和"\[MAP\]"填入相应信息即可。因为涉及的文件有 756 个之多，所以我们采用包含头文件的方法，  
即用#include 语句来定义相关信息。  
   
idahelp.ali 文件的内容如下（因篇幅省略表示）：  
\--------------------------------------  
IDH\_1=IDA Pro 中文帮助手册说明.htm  
IDH\_2=index.htm  
IDH\_4=4.htm  
IDH\_5=5.htm  
IDH\_6=6.htm  
IDH\_7=7.htm  
IDH\_8=8.htm  
......  
......  
IDH\_1523=1523.htm  
\--------------------------------------  
   
idahelp.h 文件的内容如下（因篇幅省略表示）：  
\--------------------------------------  
#define IDH\_1 2  
#define IDH\_2 2  
#define IDH\_4 4  
#define IDH\_5 5  
#define IDH\_6 6  
#define IDH\_7 7  
#define IDH\_8 8  
......  
......  
#define IDH\_1523 1523  
\--------------------------------------  
   
按如下步骤操作：  
1) 将 idahelp.ali 和 idahelp.h 文件放在与 idahelp.hpp（帮助项目助文件）相同的文件夹下；  
2) 在 FAR 项目编辑器(HH Project Editor)主界面下 Other Options 选项向的中间栏内选中“\[ALIAS\]”，  
在右侧单击“Include File...”并选中 idahelp.ali 文件；  
3) 在 FAR 项目编辑器(HH Project Editor)主界面下 Other Options 选项向的中间栏内选中“\[MAP\]”，  
在右侧单击“Include File...”并选中 idahelp.h 文件；  
4) 保存帮助项目助文件 idahelp.hpp，然后编译帮助文件；  
5) 将编译好的文件复制到 IDA 的安装位置（注意一定要先备份英文的 idahelp.chm 文件，或先将其更名为 idahelp\_en.chm），  
　 再将中文版的帮助文件更名为 idahelp.chm，此时大功告成。  
   
另外发现一个小窍门：.chm 文件中的“#IVB”就是编译结果，因此也可以将“#IVB”文件直接放入.chm 文件中。  
我使用的是 Total Commander 的 chm 插件将其直接放入 chm 内的。请大家在试验之前一定要做好备份工作。但这个方法  
不是十分可靠，有时可能会显示空页（“无法显示网页”），但可以点击其它主题。  
   
还有一个问题，用 IDA 5.2 的 F1 打开帮助文件时，显示的是“WindowOpen”主题，可能 IDA5.2 调用的命令是  
HtmlHelp(0, "IdaHelp.chm", HH\_HELP\_CONTEXT, 585);   
经查 585.htm 正是该主题。如果要想显示 index.htm 的话，应将其更名为 585.htm。但原来的 585.htm 可能与其他文件有关联，  
因此，要把所有的关联修正好后才行。当然还可以分析 idag.exe 将那个 585 改为 2（我这里将 index.htm 的 ID 定义为 2）。  
   
   
附件内含“idahelp.ali”、“idahelp.h”、“idahelp.hpp”、“#IVB”和搞定的中文版帮助文件 idahelp\_cn.chm 共 5 个文件。  
  
更正后的附件见 14 楼，对此造成的不便表示歉意。