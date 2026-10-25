
# Reversing MFC App Modified (通用 查找 MFC 消息事件) Update...

> 原文：[`www.pediy.com/kssd/pediy10/76990.html`](https://www.pediy.com/kssd/pediy10/76990.html)

I've read Externalist's article (Reversing MFC Applications) about searching for Button Event of any MFC applications and gained a lot yesterday. This article introduce the whole process and related principles in details and provide a script in the end. It's no doubt that this script is convenient for us and I wanna give my thanks to the author again.

However, I have to admit that the whole process is not easy to master because of complex operations. You know, the author used Resource Hacker, Spy Window, OD and finally IDA to analyze and reserve the target in the article. If we follow those steps we'll waste much time definitely especially using IDA to reserve (if our target is not so small). Additionally, these complex operations reduce the adaptation dramatically too.

Actually, it's not necessary to use those comprehensive tools and all we only need is an `OD` of any version. And as you konw that all we should do is modifying one place manually. To achieve our goal, I've modified the script written by Externalist and done a tutorial to tell you how to use it. Enjoy!

If you have any questions, please refer to this article Reversing MFC Applications.

Download:

[`www.tuts4you.com/download.php?view.2509`](http://www.tuts4you.com/download.php?view.2509)

The steps:

1. Using `OD` to open our target and load our script (you'll find a conditional breakpoint in the Breakpoint Window after this step)
2. Press `F9` to run our target and then click "View---Windows" to find Button ID and Window Handle.
3. Modify the conditional breakpoint manually.
4. Press the related button and choose "Resume" to execute our script continously when the breakpoint effect.
5. All done.

代码:

```
/\*
*OllyScript  -  Script to set a Conditional Breakpoint MFC Messages
Author  : Externalist
Modified: Playboysen
Supports : mfc42,mfc71,mfc80,mfc90*
\*/ 

var Dll\_CodeBase,Condition,BP\_Loc,temp
var nMsg,nID,hWnd


/\* Edit these lines \*/
/\* Make sure you put a ‘0’ prefix if the hex value starts with an Alphabet \*/
//--------------------------------------------//
MOV hWnd,"123"  // Window Handle
MOV nMsg,"111"  // WM\_COMMAND
MOV nID,"1"  // Resource ID
//--------------------------------------------//


MOV Condition,"(\[esp+4\]=="+nMsg+") && (\[ebp+8\]=="+nID+") && (\[edi+20\]==0"+hWnd+")"

GMA "mfc42",CODEBASE
CMP $RESULT,0
JNZ FIND\_OPCODE
GMA "mfc42",CODEBASE
CMP $RESULT,0
JNZ FIND\_OPCODE
GMA "mfc42",CODEBASE
CMP $RESULT,0
JNZ FIND\_OPCODE
GMA "mfc42",CODEBASE
...

```

附件中：

* The script and a target.rar 是我们的脚本和一个小练习
* Reversing MFC App Modified.rar 是我做的一个视频教程

脚本更新：

1. 操作更加人性化；
2. 查找到消息事件地址时自动下断并且给出提示；

这个脚本原作出自国外的一个大牛 Externalist，我只是为了方便和扩大脚本的易用性而将其加以修改或者说是改进，为了交流的方便，文章和教程都是用英语写的，请谅解

上传的附件

| | | |
|---|---|---|
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `Reversing MFC App Modified.rar` | [请到论坛下载该视频文件：`bbs.pediy.com/showthread.php?t=76990`](http://bbs.pediy.com/showthread.php?t=76990%20) |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `The script and a target.rar` | |
