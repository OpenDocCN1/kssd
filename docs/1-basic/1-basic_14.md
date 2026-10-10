

# 给和我一样在学汇编特别是《王爽汇编》的同志们一点小帮助

> 原文：[`www.pediy.com/kssd/pediy10/75960.html`](https://www.pediy.com/kssd/pediy10/75960.html)

看汇编没几天，整天抱着书看，去机器上写书上要求写的实验程序。可是汇编没有 IDE（我没有找到）每次语法或者逻辑错误后 先要去该代码 然后又在控制台 masm 一遍 link 一遍 debug 一遍。  
我比较笨一个程序要改好多次，不知道同志们有没有类型的经历。去控制台键入编译的命令真的很烦。就算有 doskey 记住后，然后箭头选，敲回车都烦。  
也是我决定偷懒  
首先我用的 masm 的版本是 6.11 masm 和 link 分别为 C:\\masm611\\bin\\masm.exe,C:\\masm611\\bin\\link.exe  
在 bin 目录下有个叫 NEW-VARS.BAT 的批 我把它名字改成 bc.bat(原来的名字太长了） 其作用是设置系统为 asm 的编译环境吧。英语不好，看不完全懂。  
反正是把这个 bat 拿出来改动了下，主要是因为我不知道 path 命令的全部用法，我只知道添加我需要的路径。反正这 bat 的其他部分也基本无害。  
改动后的版本如下：  
  
@echo off  
cls  
SET PATH=C:\\MASM611\\BIN;c:masm611\\bin\\archer;c:\\windows\\system32\\;%PATH%   
SET LIB=C:\\MASM611\\LIB;%LIB%  
SET INCLUDE=C:\\MASM611\\INCLUDE  
SET INIT=C:\\MASM611\\INIT  
SET HELPFILES=C:\\MASM611\\HELP\\\*.HLP  
SET ASMEX=C:\\MASM611\\SAMPLES  
SET TMP=C:\\WINDOWS\\TEMP  
cls  
masm %1  
del \*.bak  
del c:\\masm611\\bin\\archer\\\*.bak  
if exist %1.obj call lr %1  
  
添加了 编译器的路径 archer 那个路径是我自己放代码的地方  
其他 set 命令不是我写的 也不知道什么意思  
后面先用 masm 编译 你需要编译的源文件  
如果没有生成 obj 的话也就是语法错误，就乖乖的直接回去改  
如果生成了 obj 调用另外的一个 lr.bat（我不知道怎么在批处理里面如何在 if 语句后面添加多条命令）并把参数传递过去  
  
下面是 lr.bat 的内容  
@echo on  
link %1.obj;  
@echo off  
del %1.obj  
copy %1.asm c:\\masm611\\bin\\archer\\%1.asm  
debug %1.exe  
  
连接 obj ，然后删除 obj，将这次的 asm 放到我放代码的地方，然后开始开始用 debug 调试 这样如果是我们的逻辑错误后 我们不需手动去删除编译好了的 obj 和连接好的 exe，唯一需要做的是重复用第一个 bat 就行了。  
  
有几点需要说明的是  
1.我是菜鸟 这些东西是我自己看着一本 MS-DOS6.2 写出来的。只是让和我一样的初学者少敲几次键盘，在程序出错的时候不因为不必要的时候弄糟了心情。  
2.我用的 xp 系统，在开始菜单的运行里面键入 cmd 进入控制台后默认的目录是 C:\\Documents and Settings\\Administrator 所以如果你要用这两个 bat 的话，请自己把他们丢在这个目录里。并在这个目录里放入你需要编译的 XXXX.asm 文件。剩下在控制台 用键入 bc XXXX 就行了。 如果程序出错的话，改好了源文件，并保存后，直接在控制台窗口。上箭头+回车就把编译 连接 debug 所有的事情搞定（前提是你每次出错的时候不退出当前控制台，发现逻辑错误的话可以用-quit 退出 debug 程序。反正意思就是让 doskey 记住 bc XXXX 能少按几下键盘就少按几下）  
3.第一次在看雪发帖，有什么不对的多见谅，多包涵。