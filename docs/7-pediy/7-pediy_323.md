

# 增加 taskmgr.exe 调用 FileInfo 的功能

> 原文：[`www.pediy.com/kssd/pediy06/pediy6560.htm`](https://www.pediy.com/kssd/pediy06/pediy6560.htm)

前不久写了个小程序：“ProcessList and FI”，他能查看系统的所有进程，并且有效地和 FI 结合在了一起。当面对一些可疑的进程时就可以快速得调用 FI 进行更深一步地分析了，可这个小程序仅仅支持 WIN9X，到了 NT 系统中就不能使用了。再写个 NT 下的进程管理器不如直接改造现有的 taskmgr.exe，因为我个人感觉 taskmgr.exe 还不错！  
  
一：本文所涉及的程序。  
  
    目标程序：taskmgr.exe（Win2000SP4 所带的 Windows TaskManager(ver5.0.2195.6620)）。  
    调用 FI 的程序：CallFI.exe（在 taskmgr.exe 中获取指定进程的路径并依此调用 FileInfo 程序）。  
    所需工具：SoftICE、W32dasm、Exescope、LordPE，UltraEdit。  
  
  
二：分析 taskmgr.exe。  
  
    打开 taskmgr.exe 后选择\[进程\]标签，右键单击任意一个进程后你会发现弹出了一个菜单，我们可以在里面添加一个菜单项用来调用 CallFI.exe 程序。  
      
    在添加菜单项之前先对其菜单消息处理代码进行定位。定位后就可以修改消息处理代码，处理我们新添加的菜单项事件了。可以用 TrackPopupMenuEx()和 SendMessageW()作为断点进行定位。先下 TrackPopupMenuEx()断点，右键单击进程后就会中断，中断后要按 F11，然后选\[结束进程\]，这时又会出现中断，此时再下 SendMessageW()断点，中断后再往下走几步就会找到关键位置了。当然，你也可以直接下 SendMessageW()作为断点，但不如前一种方法方便。  
      
    下面是其菜单消息处理代码的部分内容：  
  
:010085F6 BA5B9C0000              mov edx, 00009C5B      ————9C5B(ID=40027,\[调试\]菜单)  
:010085FB 3BCA                    cmp ecx, edx  
:010085FD 7F50                    jg 0100864F            ————所选 ID>40027 -> JUMP  
:010085FF 7438                    je 01008639            ————跳到\[调试\]事件处理代码  
:01008601 81E9F0030000            sub ecx, 000003F0  
:01008607 7410                    je 01008619  
:01008609 49                      dec ecx  
:0100860A 49                      dec ecx  
:0100860B 0F8492000000            je 010086A3  
:01008611 49                      dec ecx  
:01008612 7425                    je 01008639  
:01008614 E99A000000              jmp 010086B3           ————退出判断  
.  
:01008639 85C0                    test eax, eax  
:01008648 E8B2FDFFFF              call 010083FF    ————\[调试\]事件处理代码  
.  
:0100864F BA5C9C0000              mov edx, 00009C5C  ————\[结束进程 ID\]  
:01008654 3BCA                    cmp ecx, edx  
:01008656 744B                    je 010086A3        ————\[结束进程\]事件处理代码  
.  
:0100866A 81F9A99C0000            cmp ecx, 00009CA9 ————\[结束进程树 ID\]  
:01008670 7541                    jne 010086B3  
:0100867B E852070000              call 01008DD2    ————\[结束进程树\]事件处理代码  
:01008680 EB31                    jmp 010086B3           ————退出判断  
  
  
    可以看到这段代码首先和\[调试\]菜单的 ID 进行比较，只要 ID 比 40027 大，代码就会跳到另外的一个地方再进行判断，直到不满足任何条件退出至：010086B3。这里好象没什么好讲的，只要在退出之前跳到我们的代码就可以了……可究竟选择修改哪里呢？跳到 010086B3 处的代码有很多，例如：01008614，他是在 ID 小于 40027 并且不等于 40027 的情况下跳出去的，此时寄存器 ECX 的值已经不再是所选菜单的 ID 值了。我们要添加的代码首先要判断 ECX 是否为我们所选菜单的 ID 值，难道再 MOV 一次？不好！（其实不判断 ID 也能正常执行，就是代码不严谨，我在后面的修改中就没有 CMP，偷懒……这里还是讲清楚为好。）往下看吧，01008670，他跳到 010086B3 前 ECX 的值还是所选菜单的 ID 值！就选他了！我们新建菜单的 ID 值比 40027 大就可以确保程序能执行到那里。我选的是：40066。  
  
      
三：修改 taskmgr.exe。  
  
    1，添加菜单项：  
    用 Exescope 插入菜单项时提示没有足够的空间，只好修改原来的分隔线了，将 ID 改为：40066，名称也只能两个字符了，我写的是：“FI”。  
  
    2，增加输入函数：  
    新建菜单项\[FI\]的功能是运行 CallFI.exe，这就要利用 ShellExecute()或类似功能的函数。原 taskmgr.exe 中并没有类似函数的调用，所以我们必须增加一个 ShellExecute()。简单介绍一下步骤：  
    用 LordPE 中的\[PE Editor\]打开 taskmgr.exe（记下 ImageBase=01000000）。选\[Directories\]，在 Import Table 处选\[...\]，右键单击任意 DLL，选\[add import\]，分别填上：SHELL32.dll，ShellExecuteA，然后分别点：\[+\]和\[OK\]。在\[Import Table\]中选中刚刚添加进去的 SHELL32.dll，并将右下脚的\[View always FirstThunk\]选中，记下 ShellExecuteA 的 ThunkRVA 值：0001801C。  
  
    3，寻找空白空间并添加数据和代码：  
    空白空间我选的是 offset：560。现在再来看看 ShellExecute()：  
  
    HINSTANCE ShellExecute(  
  
    HWND hwnd,  // handle to parent window  
    LPCTSTR lpOperation,  // pointer to string that specifies operation to perform  
    LPCTSTR lpFile,  // pointer to filename or folder name string  
    LPCTSTR lpParameters,  // pointer to string that specifies executable-file parameters   
    LPCTSTR lpDirectory,  // pointer to string that specifies default directory  
    INT nShowCmd   // whether file is shown when opened  
    );  
  
    如果要运行 CallFI.exe，在 C 中可以这样写：  
    ShellExecute(NULL,"open","F:\\\\tools\\\\cracktools\\\\Fi250\\\\FI\_REG.EXE",NULL, NULL, SW\_SHOWNORMAL);  
  
    ShellExecute()共有 6 个参数，在这里只有两个参数需要存放在空白空间里，即：  
    lpOperation="open"，  
    lpFile="F:\\tools\\cracktools\\Fi250\\CallFI.exe"。//CallFI.exe 在计算机中的位置。  
  
    用 UltraEdit 打开 taskmgr.exe，将上面 44 字节写入 offset：560，并记下他们的地址：  
    lpOperation=560  
    lpFile=568  
  
    这样我们就具备了运行 CallFI.exe 的条件：  
  
    PUSHAAD        60  
    PUSH 01        6A01        ->SW\_SHOWNORMAL  
    PUSH 00        6A00  
    PUSH 00        6A00  
    PUSH 01000568  6868050001  ->"F:\\tools\\cracktools\\Fi250\\FI\_REG.EXE"  
    PUSH 01000560  6860050001  ->"open"  
    PUSH 00        6A00  
    CALL 0001801C  FF151C800101->0101801C(ThunkRVA+ImageBase=0001801C+01000000)  
    POPAD          61  
    JMP 010086B3   E904810000  退出判断  
  
    用 UltraEdit 打开 taskmgr.exe，将上面 31 字节写入 offset：590。  
  
    当 ID 值不满足所有条件时，让程序执行我们的代码，将  
    :01008670 jne 010086B3  
    改为：  
    :01008670 jne 01000590  
  
    用 UltraEdit 搜索 754185C0743D 替换为 0F851A7FFFFF 即可。  
  
    现在你就可以试试与 FileInfo 联手后的进程管理器了，是不是感觉比以前强大了很多呢？  
  
  
四：相关提示。  
  
    文中提到的 CallFI.exe 可从我的个人主页下载，CallFI.exe 须与 FileInfo 程序在同一目录，详见其 Readme 文件。  
  
    我的个人主页：  
    [`timw.yeah.net`](http://timw.yeah.net)  
    [`timw.126.com`](http://timw.126.com)  
  
五：灌水。  
  
    和其他的 PEDIY 文章相比本文实属拙文，写出来当个总结吧。水平有限，还请大家多多指教。  
  
    再过几个小时，2004 雅典奥运会就正式拉开帷幕了，祝所有的运动员都取得好成绩！  
  
    谢谢大家！  
  
  
23:09 2004-08-13

 4dd466d0e1e0cc123afbe504da2462ca