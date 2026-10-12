
# 第一阶段第二题我的笨方法+处理的文件(看雪金山 2007 逆向分析挑战赛)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-254.htm`](https://www.pediy.com/kssd/pediy09/pediy09-254.htm)

> 第一阶段第二题我的笨方法+处理的文件(看雪金山 2007 逆向分析挑战赛)  
> by aker  
> 16:29 2007-8-26  
> 使用工具：`winhex`,`stud_pe`,`w32asm`，`od`

如果对 pe 文件格式一点都不知道的，可以先看看 `www.pediy.com` 的资料区的 pe 格式，我也是现学现卖的:P

另外对 windows 消息循环机制要有一定的了解。

开始下载后很快就合并成功并出现窗口了，就放下了，感觉应该很容易出来的。今天在论坛上发现提交的人较少，想是不是很难啊，而且我也上不了网了，所以自己动手做了下，发现确实挺烦得，但是我估计我的方法非常笨拙，也有可能是平时这方面作的少，就当是抛砖引玉吧;)希望看到大牛的手笔，特别是 shoooo,ccfer 等好快的牛，呵呵呵。我找导入表是就这样用眼睛看得，不知道是不是可以写程序判断之类。入口点是合并后用 `w32asm` 看得。另外如果文件要重新对齐和重建引入表，估计就更加麻烦了。大家有好方法都介绍下拉。

附件是我合并好并修改好的文件。

## 1.文件重建

先从其他文件剥下来一个 pe 文件头，命名为 `pehead.dat`。方法随便，我用的是 `winhex` 从一个文件前面剥下一个 4k 的文件头。方法是在文件头 ALT+1，在要选尾部 ALT+2，然后再编辑里面另存为 phead.dat。

然后按照下面的顺序。

文件大小

```
rav    -name    size
0000h  -phead   1000h
1000h  -text    6000h
7000h  -rdata   1000h
8000h  -data    3000h
```

同样用 `winhex` 工具菜单的合并，合并文件，得到 `build.exe`。

打开 `stud_PE`,修改每个区块的位置和大小。这个不要说了吧。

在 `text` 中找入口点，用 `w32asm` 打开刚刚重建的文件，发现有 `GetCommandLine` 的函数的起点 `1527h`，在 `stud_pe` 中填入。

在 `rdata` 中找到 `import table`，看看，刚好应该是这个段的起始位置应该 `7000h`，所以就不用重建了。
`import table`，位置为 `7618h`，大小为 `30`，修改 `IMAGE_DIRECTORY_ENTRY_IMPORT`，基本完成重建。

点击看一下，说是非法可执行文件。郁闷，再看看，哈忘了文件大小，文件大小变了，修改为 `b000h`，okay,保存。测试，出来了那个 pediy 的界面。

另外这个地方发现校验和好像不需要改，好像和资料上说的不一样，谁指点我一下。

还有问题就是如果我不知道到底哪个文件放导入表该怎么办呢？就是怎么判断哪个区段的文件是何种文件。

小小总结，文件添加区段后要修改区段值，文件大小，区段属性等
修改入口点的话自己要记得跳回来。

## 2.增加菜单

看到窗口类注册的时候：一大段代码，其中关键的有这几个：

```
004011F5   |.  BE 94804000         mov esi,bf1.00408094                         ;  ASCII "pediy.com"
        ........
00401248   |.  8D45 F4             lea eax,dword ptr ss:[ebp-C]
0040124B   |.  8945 D0             mov dword ptr ss:[ebp-30],eax
0040124E   |.  8D45 B0             lea eax,dword ptr ss:[ebp-50]
00401251   |.  BF A0804000         mov edi,bf1.004080A0                         ;  ASCII "pediy.com"
00401256   |.  50                  push eax                                     ; /pWndClass
00401257   |.  897D D4             mov dword ptr ss:[ebp-2C],edi                ; |
0040125A   |.  FF15 0C714000       call dword ptr ds:[<&USER32.RegisterClassA>] ; \RegisterClassA
```

发现注册的窗口类已经有了一个默认的菜单名字叫 `pediy.com`，这样添加菜单就很简单了。

手工写一个菜单资源，rc 编译为 `.res` 文件，我不知道 `rsrc` 目录的格式是什么样子的，我就直接把这个和一个其他的代码一起编译了，然后剥出来，名为 `rsrc.dat`，因为对齐是 4k，所以文件大小有 4k。

```
pediy.com MENU
{
       POPUP "&Help"
       {
          MENUITEM "&About",         1
       }
}
```

在 `stud_pe` 中添加一个新节 `rsrc`，大小为 `1000h`,填充为刚刚的文件，然后在 `stud_pe` 中手工修改数据目录中的 `resource table`,指向 `B000h`,这是对于我的文件而言的。文件大小我填写的 `90h`。

好了，这样重新打开文件就发现菜单在上面了。现在的问题就是如何让程序响应我们的点击呢？

## 3.响应消息

在论坛混的大概都知道 window 窗体程序是消息驱动的。我们点击菜单，鼠标滑动等都会产生对应的 windows 消息，消息先存放到系统队列，然后由系统派送到对应的产生消息的窗体。(个人理解:P,不对请一定要告诉我)。写 windows 程序的时候很多消息都是系统自己处理的，我们只处理我们需要的消息，其他都有系统缺省处理。我们要处理的消息由 `wndproc` 指派。

好了，知道这个，我们就可以知道我们点击菜单上的 about 的时候，肯定会有对应的消息发送到窗体了，只不过窗体可能没有处理机制而已。菜单消息的大类是 `WM_COMMAND` 消息，然后小类可以是我们自己制定的消息号，我们在上面定义菜单的时候，把 about 的消息标志定义为 `1`。我们现在要做的就是处理 `WM_COMMAND`,也要处理里面的一号消息。

```
   MENUITEM "&About",   1
```

看看窗体消息处理的汇编代码，在下面好长一段，首先看看 `wndproc` 的原形：

```
LRESULT CALLBACK MainWndProc( HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam )
```

第二个参数为 `msg` 消息号，后面两个为对应消息的参数，第一个为窗体句柄。

所以代码中的 `[ebp+c]` 就是消息号。

这个代码可能是优化过吧，比较的方法和正常编写出来的不一样，比如正常的比较都是 `cmp eax,111` 之类的为对 `wm_command` 的处理，但是这个里面很奇怪，比较是用的减，然后判断标志位是否为 0 这样的，就是 `dec` 或者 `sub`，然后 `je`。所以不容易看得清楚。我手工加了几个消息注释，`WM_DESTROY`（`2h`）, `WM_SIZE`（`5h`），`WM_PAINT`（`15h`）。可以看到，没有 `WM_COMMAND`，但是我们现在需要处理菜单呀，怎么办呢？

```
004012D5   /.  55                  push ebp
004012D6   |.  8BEC                mov ebp,esp
004012D8   |.  83EC 40             sub esp,40
004012DB   |.  8B45 0C             mov eax,dword ptr ss:[ebp+C]
004012DE   |.  48                  dec eax                                      ;  Switch (cases 2..F)
004012DF   |.  48                  dec eax
004012E0   |.  74 68               je short b1.0040134A                         ;  WM_DESTROY
004012E2   |.  83E8 03             sub eax,3
004012E5   |.  74 4D               je short b1.00401334                         ;  WM_SIZE
004012E7   |.  83E8 0A             sub eax,0A
004012EA   |.  74 14               je short b1.00401300                         ;  WM_PAINT
004012EC   |.  FF75 14             push dword ptr ss:[ebp+14]                   ; /lParam; Default case of switch 004012DE
004012EF   |.  FF75 10             push dword ptr ss:[ebp+10]                   ; |wParam
004012F2   |.  FF75 0C             push dword ptr ss:[ebp+C]                    ; |Message
004012F5   |.  FF75 08             push dword ptr ss:[ebp+8]                    ; |hWnd
004012F8   |.  FF15 F4704000       call dword ptr ds:[<&USER32.DefWindowProcA>] ; \DefWindowProcA
004012FE   |.  EB 54               jmp short b1.00401354
00401300   |>  8D45 C0             lea eax,dword ptr ss:[ebp-40]                ;  Case F of switch 004012DE
00401303   |.  50                  push eax                                     ; /pPaintstruct
00401304   |.  FF75 08             push dword ptr ss:[ebp+8]                    ; |hWnd
00401307   |.  FF15 F8704000       call dword ptr ds:[<&USER32.BeginPaint>]     ; \BeginPaint
0040130D   |.  FF35 B0A94000       push dword ptr ds:[40A9B0]
00401313   |.  FF35 B4A94000       push dword ptr ds:[40A9B4]
00401319   |.  50                  push eax
0040131A   |.  FF75 08             push dword ptr ss:[ebp+8]
0040131D   |.  E8 36000000         call b1.00401358
00401322   |.  83C4 10             add esp,10
00401325   |.  8D45 C0             lea eax,dword ptr ss:[ebp-40]
00401328   |.  50                  push eax                                     ; /pPaintstruct
00401329   |.  FF75 08             push dword ptr ss:[ebp+8]                    ; |hWnd
0040132C   |.  FF15 FC704000       call dword ptr ds:[<&USER32.EndPaint>]       ; \EndPaint
00401332   |.  EB 1E               jmp short b1.00401352
00401334   |>  8B45 14             mov eax,dword ptr ss:[ebp+14]                ;  Case 5 of switch 004012DE
00401337   |.  0FB7C8              movzx ecx,ax
0040133A   |.  C1E8 10             shr eax,10
0040133D   |.  890D B4A94000       mov dword ptr ds:[40A9B4],ecx
00401343   |.  A3 B0A94000         mov dword ptr ds:[40A9B0],eax
00401348   |.  EB 08               jmp short b1.00401352
0040134A   |>  6A 00               push 0                                       ; /ExitCode = 0; Case 2 of switch 004012DE
0040134C   |.  FF15 00714000       call dword ptr ds:[<&USER32.PostQuitMessage>]; \PostQuitMessage
00401352   |>  33C0                xor eax,eax
00401354   |>  C9                  leave
00401355   \.  C2 1000             retn 10
```

我考虑的是自己添加消息，但是上面的空间很满，前后都没有空闲地方了。

怎么办呢，真急人啊;)

我考虑的是在下面代码处跳出去，跳到一个空闲的地方，处理了 `WM_COMMAND`，然后再回来，为什么选择这个地方呢？

因为这个地方消息号刚传进来，自己控制的话比较清楚。反正总要在 `DefWindowProcA` 前处理掉自己的消息才是;



### 代码修改示例

`004012DE`   |.  48                  dec eax                                      ;  Switch (cases 2..F)
`004012DF`   |.  48                  dec eax
`004012E0`   |.  74 68               je short b1.`0040134A`                         ;  WM_DESTROY
`004012E2`   |.  83E8 03             sub eax,3
`004012E5`   |.  74 4D               je short b1.`00401334`                         ;  WM_SIZE
`004012E7`   |.  83E8 0A             sub eax,0A
`004012EA`   |.  74 14               je short b1.`00401300`                         ;  WM_PAINT
////////////////////////
 // 我修改的代码如下
`004012DE`    . /E9 91580000         jmp build.`00406B74`
`004012E3`      |90                  nop
`004012E4`      |90                  nop
`004012E5`    . /74 4D               je short build.`00401334`     ///////此处就一样了

可以看到，我就是简单的远跳出去了，把原来的减和一个近程跳转覆盖掉了，但是我们在后面必须恢复该操作。

我找的空闲空间是整个代码段之后的空地，地址是 `00406B74`。下面看看代码跳出去后恢复原来操作和跳回来代码。

看到，我们跳到地头后，首先比较是不是 `111h`，就是看是不是 `WM_COMMAND`，看 `winuser.h`。

如果不是 `jnz short build.00406BBC` 就跳出去处理原来的操作了。`build.00406BBC` 处可以看到就是原来的操作，`dec eax`。。。。

不过该处需要自己手工把跳回去的地址整理好，有点麻烦,不过我的笨方法就这样了;)

代码:

```
00406B74    > \3D 11010000         cmp eax,111                                  ;  Switch (cases 2..111)
00406B79    .  75 41               jnz short build.00406BBC
00406B7B    .  8B45 10             mov eax,dword ptr ss:[ebp+10]                ;  Case 111 (WM_COMMAND) of switch 00406B74
00406B7E    .  66:83F8 01          cmp ax,1
00406B82    .  75 41               jnz short build.00406BC5      //.........省一个关键代码...........过会说
00406BBC    >  48                  dec eax
00406BBD    .  48                  dec eax
00406BBE    .  74 0A               je short build.00406BCA
00406BC0    .  83E8 03             sub eax,3
00406BC3    .  74 0A               je short build.00406BCF
00406BC5    >^ E9 1DA7FFFF         jmp build.004012E7                           ;  Default case of switch 00406B74
00406BCA    >^ E9 7BA7FFFF         jmp build.0040134A                           ;  Case 2 (WM_DESTROY) of switch 00406B74
00406BCF    >^ E9 60A7FFFF         jmp build.00401334                           ;  Case 5 (WM_SIZE) of switch 00406B74
```

好了，上面才讲到如何跳转出去，再跳转回来。

下面就是要考虑怎么样跳一个消息框出来了。你肯定想，不就直接 `push` 参数，`call messagebox` 摆，对了，你是对的。但是有两个问题要考虑，第一个就是数据的问题。压栈的时候你的数据该怎么搞，因为要符合要求才是啊，我就是手工在代码段后面添了

代码:

```
pediy      看雪论坛.珠海金山 2007 逆向分析挑战赛   http://www.pediy.com
```

两段小数据，当然，需要加字符串的结束符 `0` 和回车换行符。

`call MessageBox` 的时候，你知道具体地址不？因为导入表中没有这个呢。

我采用的是 `LoadLibraryA`，`GetProcAddress` 做的，看代码吧，可能还清楚些。

下面是自己手工写的代码。

代码:

```
00406B74    > \3D 11010000         cmp eax,111                                  ;  Switch (cases 2..111)
00406B79    .  75 41               jnz short build.00406BBC
00406B7B    .  8B45 10             mov eax,dword ptr ss:[ebp+10]                ;  Case 111 (WM_COMMAND) of switch 00406B74
00406B7E    .  66:83F8 01          cmp ax,1
00406B82    .  75 41               jnz short build.00406BC5
///////////////// 下面是找函数的关键代码 /////////////////
00406B84    .  68 B8754000         push build.004075B8                          ; /FileName = "user32.dll"
00406B89    .  FF15 6C704000       call dword ptr ds:[<&KERNEL32.LoadLibraryA>] ; \LoadLibraryA
00406B8F    .  8BF8                mov edi,eax
00406B91    .  8B35 A0704000       mov esi,dword ptr ds:[<&KERNEL32.GetProcAddr>;  kernel32.GetProcAddress
00406B97    .  68 AC754000         push build.004075AC                          ; /ProcNameOrOrdinal = "MessageBoxA"
00406B9C    .  57                  push edi                                     ; |hModule
00406B9D    .  FFD6                call esi                                     ; \GetProcAddress
00406B9F    .  8BF8                mov edi,eax
00406BA1    .  68 40000000         push 40
00406BA6    .  68 E06B4000         push build.00406BE0                          ;  ASCII "pediy"
00406BAB    .  68 E76B4000         push build.00406BE7
00406BB0    .  68 00000000         push 0
00406BB5    .  FFD7                call edi
///////////////// 上面是找函数的关键代码 /////////////////
00406BB7    .^ E9 96A7FFFF         jmp build.00401352
00406BBC    >  48                  dec eax
00406BBD    .  48                  dec eax
00406BBE    .  74 0A               je short build.00406BCA
00406BC0    .  83E8 03             sub eax,3
00406BC3    .  74 0A               je short build.00406BCF
00406BC5    >^ E9 1DA7FFFF         jmp build.004012E7                           ;  Default case of switch 00406B74
00406BCA    >^ E9 7BA7FFFF         jmp build.0040134A                           ;  Case 2 (WM_DESTROY) of switch 00406B74
00406BCF    >^ E9 60A7FFFF         jmp build.00401334                           ;  Case 5 (WM_SIZE) of switch 00406B74
```

总结：以前看过一点点 `pe` 的知识，也很少写 `windows` 窗体程序，真是什么什么到用时方恨少啊，发现自己根本就是不懂，呵呵呵呵。

另外估计我的方法是特别笨的方法，希望抛砖引玉，大家说说到底有什么速成的方法咯。

说老实话，我觉得这道题目可能考基础知识点多点，考耐心，比如合并文件自己可以用工具，但是要修改里面的数据的时候，感觉很麻烦，上一道题目考思维能力，解题能力。

觉得这里改比较好：

将原窗口消息处理函数里这段代码的 `call ds:DefWindowProcA`

```
.text:004012EC                 push    [ebp+arg_C]
.text:004012EF                 push    [ebp+arg_8]
.text:004012F2                 push    [ebp+arg_4]
.text:004012F5                 push    [ebp+arg_0]
.text:004012F8                 call    ds:DefWindowProcA
.text:004012FE                 jmp     short locret_401354
```

改成 `call MyWndProc`,函数代码如下:

```
LRESULT CALLBACK MyWndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
  if ( message == WM_COMMAND && wParam == 0x68)
  {
    MessageBox(NULL,"看雪论坛.珠海金山 2007 逆向分析挑战赛\\r\\nhttp://www.pediy.com","pediy",0x40);
    return 0;
  }

  return DefWindowProc(hWnd, message, wParam, lParam);
}
```

还能多出一个空余字节 :)

第一阶段第二题我的笨方法+处理的文件(看雪金山 2007 逆向分析挑战赛)

by aker

16:29 2007-8-26

使用工具：`winhex`,`stud_PE`,`w32asm`，`od`

如果对 `pe` 文件格式一点都不知道的，可以先看看 `www.pediy.com` 的资料区的 `pe` 格式，我也是现学现卖的:P

另外对 `windows` 消息循环机制要有一定的了解。

开始下载后很快就合并成功并出现窗口了，就放下了，感觉应该很容易出来的。今天在论坛上发现提交的人较少，想是不是很难啊，而且我也上不了网了，所以自己动手做了下，发现确实挺烦得，但是我估计我的方法非常笨拙，也有可能是平时这方面作的少，就当是抛砖引玉吧;)希望看到大牛的手笔，特别是 `shoooo`,`ccfer` 等好快的牛，呵呵呵。我找导入表是就这样用眼睛看得，不知道是不是可以写程序判断之类。入口点是合并后用 `w32asm` 看得。另外如果文件要重新对齐和重建引入表，估计就更加麻烦了。大家有好方法都介绍下拉。

附件是我合并好并修改好的文件。

### 1.文件重建

先从其他文件剥下来一个 `pe` 文件头，命名为 `pehead.dat`。方法随便，我用的是 `winhex` 从一个文件前面剥下一个 `4k` 的文件头。方法是在文件头 `ALT+1`，在要选尾部 `ALT+2`，然后再编辑里面另存为 `phead.dat.`

然后按照下面的顺序。

文件大小

代码:

```
rav  - name  size
0000h  - phead  1000h
1000h  - text  6000h
7000h  - rdata  1000h
8000h  - data  3000h
```

同样用 `winhex` 工具菜单的合并，合并文件，得到 `build.exe`。

打开 `stud_PE`,修改每个区块的位置和大小。这个不要说了吧。

在 `text` 中找入口点，用 `w32asm` 打开刚刚重建的文件，发现有 `GetCommandLine` 的函数的起点 `1527h`，在 `stud_pe` 中填入。

在 `rdata` 中找到 `import table`，看看，刚好应该是这个段的起始位置应该 `7000h`，所以就不用重建了。
`import table`，位置为 `7618h`，大小为 `30`，修改 `IMAGE_DIRECTORY_ENTRY_IMPORT` ，基本完成重建。
点击看一下，说是非法可执行文件。郁闷，再看看，哈忘了文件大小，文件大小变了，修改为 `b000h`，okay,保存。测试，出来了那个 `pediy` 的界面。
另外这个地方发现校验和好像不需要改，好像和资料上说的不一样，谁指点我一下。
还有问题就是如果我不知道到底哪个文件放导入表该怎么办呢？就是怎么判断哪个区段的文件是何种文件。
小小总结，文件添加区段后要修改区段值，文件大小，区段属性等
修改入口点的话自己要记得跳回来。

### 2.增加菜单

看到窗口类注册的时候：一大段代码，其中关键的有这几个：
代码:




`004011F5   |.  BE 94804000         mov esi,bf1.00408094                         ;  ASCII "pediy.com"   ........   00401248   |.  8D45 F4             lea eax,dword ptr ss:[ebp-C]   0040124B   |.  8945 D0             mov dword ptr ss:[ebp-30],eax   0040124E   |.  8D45 B0             lea eax,dword ptr ss:[ebp-50]   00401251   |.  BF A0804000         mov edi,bf1.004080A0                         ;  ASCII "pediy.com"   00401256   |.  50                  push eax                                     ; /pWndClass   00401257   |.  897D D4             mov dword ptr ss:[ebp-2C],edi                ; |   0040125A   |.  FF15 0C714000       call dword ptr ds:[<&USER32.RegisterClassA>] ; \RegisterClassA`

* * *

发现注册的窗口类已经有了一个默认的菜单名字叫 `pediy.com`，这样添加菜单就很简单了。

手工写一个菜单资源，rc 编译为 `.res` 文件，我不知道 `rsrc` 目录的格式是什么样子的，我就直接把这个和一个其他的代码一起编译了，然后剥出来，名为 `rsrc.dat`，因为对齐是 `4k`，所以文件大小有 `4k`。

代码:

* * *

```
pediy.com MENU   {      POPUP "&Help"      {         MENUITEM "&About",         1      }   }
```

* * *

在 `stud_pe` 中添加一个新节 `rsrc`，大小为 `1000h`,填充为刚刚的文件，然后在 `stud_pe` 中手工修改数据目录中的 `resource table`,指向 `B000h`,这是对于我的文件而言的。文件大小我填写的 `90h`。

好了，这样重新打开文件就发现菜单在上面了。现在的问题就是如何让程序响应我们的点击呢？

### 3.响应消息

在论坛混的大概都知道 window 窗体程序是消息驱动的。我们点击菜单，鼠标滑动等都会产生对应的 windows 消息，消息先存放到系统队列，然后由系统派送到对应的产生消息的窗体。(个人理解:P,不对请一定要告诉我)。写 windows 程序的时候很多消息都是系统自己处理的，我们只处理我们需要的消息，其他都有系统缺省处理。我们要处理的消息由 `wndproc` 指派。

好了，知道这个，我们就可以知道我们点击菜单上的 about 的时候，肯定会有对应的消息发送到窗体了，只不过窗体可能没有处理机制而已。菜单消息的大类是 `WM_COMMAND` 消息，然后小类可以是我们自己制定的消息号，我们在上面定义菜单的时候，把 about 的消息标志定义为 `1`。我们现在要做的就是处理 `WM_COMMAND`,也要处理里面的一号消息。

代码:

* * *

```
   MENUITEM "&About",   1
```

* * *

看看窗体消息处理的汇编代码，在下面好长一段，首先看看 `wndproc` 的原形：

代码:

* * *

```
LRESULT CALLBACK MainWndProc( HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam )
```

* * *

第二个参数为 `msg` 消息号，后面两个为对应消息的参数，第一个为窗体句柄。

所以代码中的 `[ebp+c]` 就是消息号。

这个代码可能是优化过吧，比较的方法和正常编写出来的不一样，比如正常的比较都是 `cmp eax,111` 之类的为对 `wm_command` 的处理，但是这个里面很奇怪，比较是用的减，然后判断标志位是否为 0 这样的，就是 `dec` 或者 `sub`，然后 `je`。所以不容易看得清楚。我手工加了几个消息注释，`WM_DESTROY`（`2h`）, `WM_SIZE`（`5h`），`WM_PAINT`（`15h`）.可以看到，没有 `WM_COMMAND`，但是我们现在需要处理菜单呀，怎么办呢？

代码:

* * *

```
004012D5   /.  55                  push ebp   004012D6   |.  8BEC                mov ebp,esp   004012D8   |.  83EC 40             sub esp,40   004012DB   |.  8B45 0C             mov eax,dword ptr ss:[ebp+C]   004012DE   |.  48                  dec eax                                      ;  Switch (cases 2..F)   004012DF   |.  48                  dec eax   004012E0   |.  74 68               je short b1.0040134A                         ;  WM_DESTROY   004012E2   |.  83E8 03             sub eax,3   004012E5   |.  74 4D               je short b1.00401334                         ;  WM_SIZE   004012E7   |.  83E8 0A             sub eax,0A   004012EA   |.  74 14               je short b1.00401300                         ;  WM_PAINT   004012EC   |.  FF75 14             push dword ptr ss:[ebp+14]                   ; /lParam; Default case of switch 004012DE   004012EF   |.  FF75 10             push dword ptr ss:[ebp+10]                   ; |wParam   004012F2   |.  FF75 0C             push dword ptr ss:[ebp+C]                    ; |Message   004012F5   |.  FF75 08             push dword ptr ss:[ebp+8]                    ; |hWnd   004012F8   |.  FF15 F4704000       call dword ptr ds:[<&USER32.DefWindowProcA>] ; \DefWindowProcA   004012FE   |.  EB 54               jmp short b1.00401354   00401300   |>  8D45 C0             lea eax,dword ptr ss:[ebp-40]                ;  Case F of switch 004012DE   00401303   |.  50                  push eax                                     ; /pPaintstruct   00401304   |.  FF75 08             push dword ptr ss:[ebp+8]                    ; |hWnd   00401307   |.  FF15 F8704000       call dword ptr ds:[<&USER32.BeginPaint>]     ; \BeginPaint   0040130D   |.  FF35 B0A94000       push dword ptr ds:[40A9B0]   00401313   |.  FF35 B4A94000       push dword ptr ds:[40A9B4]   00401319   |.  50                  push eax   0040131A   |.  FF75 08             push dword ptr ss:[ebp+8]   0040131D   |.  E8 36000000         call b1.00401358   00401322   |.  83C4 10             add esp,10   00401325   |.  8D45 C0             lea eax,dword ptr ss:[ebp-40]   00401328   |.  50                  push eax                                     ; /pPaintstruct   00401329   |.  FF75 08             push dword ptr ss:[ebp+8]                    ; |hWnd   0040132C   |.  FF15 FC704000       call dword ptr ds:[<&USER32.EndPaint>]       ; \EndPaint   00401332   |.  EB 1E               jmp short b1.00401352   00401334   |>  8B45 14             mov eax,dword ptr ss:[ebp+14]                ;  Case 5 of switch 004012DE   00401337   |.  0FB7C8              movzx ecx,ax   0040133A   |.  C1E8 10             shr eax,10   0040133D   |.  890D B4A94000       mov dword ptr ds:[40A9B4],ecx   00401343   |.  A3 B0A94000         mov dword ptr ds:[40A9B0],eax   00401348   |.  EB 08               jmp short b1.00401352   0040134A   |>  6A 00               push 0                                       ; /ExitCode = 0; Case 2 of switch 004012DE   0040134C   |.  FF15 00714000       call dword ptr ds:[<&USER32.PostQuitMessage>>; \PostQuitMessage   00401352   |>  33C0                xor eax,eax   00401354   |>  C9                  leave   00401355   \.  C2 1000             retn 10
```

* * *

我考虑的是自己添加消息，但是上面的空间很满，前后都没有空闲地方了。

怎么办呢，真急人啊;)

我考虑的是在下面代码处跳出去，跳到一个空闲的地方，处理了 `WM_COMMAND`，然后再回来，为什么选择这个地方呢？

因为这个地方消息号刚传进来，自己控制的话比较清楚。反正总要在 `DefWindowProcA` 前处理掉自己的消息才是;)

代码:

* * *

```
004012DE   |.  48                  dec eax                                      ;  Switch (cases 2..F)   004012DF   |.  48                  dec eax   004012E0   |.  74 68               je short b1.0040134A                         ;  WM_DESTROY   004012E2   |.  83E8 03             sub eax,3   004012E5   |.  74 4D               je short b1.00401334                         ;  WM_SIZE   004012E7   |.  83E8 0A             sub eax,0A   004012EA   |.  74 14               je short b1.00401300                         ;  WM_PAINT   //////////////////////////   // 我修改的代码如下   004012DE    . /E9 91580000         jmp build.00406B74   004012E3      |90                  nop   004012E4      |90                  nop   004012E5    . /74 4D               je short build.00401334     ///////此处就一样了
```

* * *

可以看到，我就是简单的远跳出去了，把原来的减和一个近程跳转覆盖掉了，但是我们在后面必须恢复该操作。

我找的空闲空间是整个代码段之后的空地，地址是 `00406B74`。下面看看代码跳出去后恢复原来操作和跳回来代码。

看到，我们跳到地头后，首先比较是不是 `111h`，就是看是不是 `WM_COMMAND`，看 `winuser.h`。

如果不是 `jnz short build.00406BBC` 就跳出去处理原来的操作了。`build.00406BBC` 处可以看到就是原来的操作，`dec eax`。。。。

不过该处需要自己手工把跳回去的地址整理好，有点麻烦,不过我的笨方法就这样了;)

代码:

* * *

```
00406B74    > \3D 11010000         cmp eax,111                                  ;  Switch (cases 2..111)   00406B79    .  75 41               jnz short build.00406BBC   00406B7B    .  8B45 10             mov eax,dword ptr ss:[ebp+10]                ;  Case 111 (WM_COMMAND) of switch 00406B74   00406B7E    .  66:83F8 01          cmp ax,1   00406B82    .  75 41               jnz short build.00406BC5      //.........省一个关键代码...........过会说      00406BBC    >  48                  dec eax   00406BBD    .  48                  dec eax   00406BBE    .  74 0A               je short build.00406BCA   00406BC0    .  83E8 03             sub eax,3   00406BC3    .  74 0A               je short build.00406BCF   00406BC5    >^ E9 1DA7FFFF         jmp build.004012E7                           ;  Default case of switch 00406B74   00406BCA    >^ E9 7BA7FFFF         jmp build.0040134A                           ;  Case 2 (WM_DESTROY) of switch 00406B74   00406BCF    >^ E9 60A7FFFF         jmp build.00401334                           ;  Case 5 (WM_SIZE) of switch 00406B74
```

* * *

好了，上面才讲到如何跳转出去，再跳转回来。



下面就是要考虑怎么样跳一个消息框出来了。你肯定想，不就直接 push 参数，call `messagebox` 摆，对了，你是对的。但是有两个问题要考虑，第一个就是数据的问题。压栈的时候你的数据该怎么搞，因为要符合要求才是啊，我就是手工在代码段后面添了

代码:

***
`pediy      看雪论坛.珠海金山 2007 逆向分析挑战赛   http://www.pediy.com`
***

两段小数据，当然，需要加字符串的结束符 `0` 和回车换行符。

call `MessageBox` 的时候，你知道具体地址不？因为导入表中没有这个呢。

我采用的是 `LoadLibraryA`，`GetProcAddress` 做的，看代码吧，可能还清楚些。

下面是自己手工写的代码。

代码:

***
```
00406B74    > \3D 11010000         cmp eax,111                                  ;  Switch (cases 2..111)   
00406B79    .  75 41               jnz short build.00406BBC   
00406B7B    .  8B45 10             mov eax,dword ptr ss:[ebp+10]                ;  Case 111 (WM_COMMAND) of switch 00406B74   
00406B7E    .  66:83F8 01          cmp ax,1   
00406B82    .  75 41               jnz short build.00406BC5   
/////////////// 下面是找函数的关键代码 //////////////////   
00406B84    .  68 B8754000         push build.004075B8                          ; /FileName = "user32.dll"   
00406B89    .  FF15 6C704000       call dword ptr ds:[<&KERNEL32.LoadLibraryA>] ; \LoadLibraryA   
00406B8F    .  8BF8                mov edi,eax   
00406B91    .  8B35 A0704000       mov esi,dword ptr ds:<&KERNEL32.GetProcAddr>;  kernel32.GetProcAddress   
00406B97    .  68 AC754000         push build.004075AC                          ; /ProcNameOrOrdinal = "MessageBoxA"   
00406B9C    .  57                  push edi                                     ; |hModule   
00406B9D    .  FFD6                call esi                                     ; \GetProcAddress   
00406B9F    .  8BF8                mov edi,eax   
00406BA1    .  68 40000000         push 40   
00406BA6    .  68 E06B4000         push build.00406BE0                          ;  ASCII "pediy"   
00406BAB    .  68 E76B4000         push build.00406BE7   
00406BB0    .  68 00000000         push 0   
00406BB5    .  FFD7                call edi   
/////////////// 上面是找函数的关键代码 //////////////////   
00406BB7    .^ E9 96A7FFFF         jmp build.00401352   
00406BBC    >  48                  dec eax   
00406BBD    .  48                  dec eax   
00406BBE    .  74 0A               je short build.00406BCA   
00406BC0    .  83E8 03             sub eax,3   
00406BC3    .  74 0A               je short build.00406BCF   
00406BC5    >^ E9 1DA7FFFF         jmp build.004012E7                           ;  Default case of switch 00406B74   
00406BCA    >^ E9 7BA7FFFF         jmp build.0040134A                           ;  Case 2 (WM_DESTROY) of switch 00406B74   
00406BCF    >^ E9 60A7FFFF         jmp build.00401334                           ;  Case 5 (WM_SIZE) of switch 00406B74
```
***

总结：以前看过一点点 pe 的知识，也很少写 windows 窗体程序，真是什么什么到用时方恨少啊，发现自己根本就是不懂，呵呵呵呵。

另外估计我的方法是特别笨的方法，希望抛砖引玉，大家说说到底有什么速成的方法咯。

说老实话，我觉得这道题目可能考基础知识点多点，考耐心，比如合并文件自己可以用工具，但是要修改里面的数据的时候，感觉很麻烦，上一道题目考思维能力，解题能力。

一直知道自己对 pe 结构了解不足，做这个题就表现的很彻底，拼接 exe 的时间用了好几个小时，然后想把菜单加到资源里也没有成功，可喜的是最后还是折腾出来了。下面是简要的分析过程：

从导入表入手，很明显存放在`_rdata` 里面，在偏移 `0x061C` 的地方找到

```
00000618h: B4 76 00 00 00 00 00 00 00 00 00 00 AC 77 00 00 ; ............
```

对应的是 `KERNEL32.DLL`，因此这个文件的位置偏移应该是 `0x7000`。

根据各个文件的大小，可以知道其顺序是：

Header：`0x1000`
`_text` ：`0x6000`
`_rdata`：`0x7000`
`_data` ：`0x8000`

将他们连接起来，然后随便 copy 一个 exe 的头进去，修改如下：

Number of Sections = `3`
Size of Code Section = `0x6000`
Entry Point RVA = `0x1000`(临时)
Section Alignment = `0x1000`
File Alignment = `0x1000`
Import Table Address = `0x7618`(关键)
Import Table Size = `0x50`

保存以后用 OD 载入，查找函数 `GetVersion`(多半是 VC 的程序，就猜它)，得到入口点，修改

Entry Point RVA = `0x1527`(修正)

exe 拼接好了，然后开始 diy。在创建窗口以后跳到自己的空间

```
00401282   .  FF15 10714000          CALL    NEAR DWORD PTR [<&USER32.CreateW>; \\CreateWindowExA  
00401288   .- E9 73770000            JMP     00408A00
```

先用高级语言写了个样子：
```
hMenu := CreateMenu;
hPopup := CreateMenu;
AppendMenu(hPopup, MF_POPUP, 555, '&About');
AppendMenu(hMenu, MF_POPUP, hPopup, '&Help');
SetMenu(Handle, hMenu);
```

然后在 OD 里面直接修改如下：

```
00408A00    60                       PUSHAD  
00408A01    8BF8                     MOV     EDI, EAX  
00408A03    68 F5894000              PUSH    004089F5                         ; ASCII "user32.dll"  
00408A08    FF15 6C704000            CALL    NEAR DWORD PTR [<&KERNEL32.LoadL>; kernel32.LoadLibraryA  
00408A0E    A3 BC894000              MOV     DWORD PTR [4089BC], EAX  
00408A13    68 EA894000              PUSH    004089EA                         ; ASCII "CreateMenu"  
00408A18    FF35 BC894000            PUSH    DWORD PTR [4089BC]  
00408A1E    FF15 A0704000            CALL    NEAR DWORD PTR [<&KERNEL32.GetPr>; kernel32.GetProcAddress  
00408A24    A3 B0894000              MOV     DWORD PTR [4089B0], EAX  
00408A29    68 DE894000              PUSH    004089DE                         ; ASCII "AppendMenuA"  
00408A2E    FF35 BC894000            PUSH    DWORD PTR [4089BC]  
00408A34    FF15 A0704000            CALL    NEAR DWORD PTR [<&KERNEL32.GetPr>; kernel32.GetProcAddress  
00408A3A    A3 B4894000              MOV     DWORD PTR [4089B4], EAX  
00408A3F    68 D6894000              PUSH    004089D6                         ; ASCII "SetMenu"  
00408A44    FF35 BC894000            PUSH    DWORD PTR [4089BC]  
00408A4A    FF15 A0704000            CALL    NEAR DWORD PTR [<&KERNEL32.GetPr>; kernel32.GetProcAddress  
00408A50    A3 B8894000              MOV     DWORD PTR [4089B8], EAX  
00408A55    FF15 B0894000            CALL    NEAR DWORD PTR [4089B0]  
00408A5B    8BD8                     MOV     EBX, EAX  
00408A5D    FF15 B0894000            CALL    NEAR DWORD PTR [4089B0]  
00408A63    8BF0                     MOV     ESI, EAX  
00408A65    68 CF894000              PUSH    004089CF                         ; ASCII "&About"  
00408A6A    68 55050000              PUSH    555                              ; 菜单的 ID  
00408A6F    6A 10                    PUSH    10  
00408A71    56                       PUSH    ESI  
00408A72    FF15 B4894000            CALL    NEAR DWORD PTR [4089B4]  
00408A78    68 C8894000              PUSH    004089C8                         ; ASCII "&Help"  
00408A7D    56                       PUSH    ESI  
00408A7E    6A 10                    PUSH    10  
00408A80    53                       PUSH    EBX  
00408A81    FF15 B4894000            CALL    NEAR DWORD PTR [4089B4]  
00408A87    53                       PUSH    EBX  
00408A88    57                       PUSH    EDI  
00408A89    FF15 B8894000            CALL    NEAR DWORD PTR [4089B8]  
00408A8F    61                       POPAD  
00408A90    FF75 14                  PUSH    DWORD PTR [EBP+14]  
00408A93    8BF8                     MOV     EDI, EAX                         ; 恢复破坏的代码  
00408A95  - E9 F387FFFF              JMP     0040128D                         ; 跳回原来空间
```

处理菜单的事件在消息循环里修改：

```
004012DB   .  8B45 0C                MOV     EAX, DWORD PTR [EBP+C]  
004012DE   .  48                     DEC     EAX  
004012DF   .  48                     DEC     EAX  
004012E0   .  74 68                   JE      SHORT 0040134A  
004012E2   .- E9 19780000            JMP     00408B00
```

跳到自己的空间里，判断一下是不是 `WM_COMMAND` 消息，

```
00408B00    2D 0F010000              SUB     EAX, 10F  
00408B05    74 11                    JE      SHORT 00408B18  
00408B07    05 0C010000              ADD     EAX, 10C  
00408B0C  - 0F84 2288FFFF            JE      00401334  
00408B12  - E9 D087FFFF              JMP     004012E7  
00408B17    90                       NOP
```

然后判断一下是不是菜单 `555` 触发的，如果是就弹出对话框，

```
00408B18    817D 10 55050000         CMP     DWORD PTR [EBP+10], 555  
00408B1F    75 18                    JNZ     SHORT 00408B39  
00408B21    90                       NOP
```



> `00408B22    68 40200100              PUSH    12040`  
> `00408B27    68 C08A4000              PUSH    00408AC0                         ; ASCII "pediy"`  
> `00408B2C    68 C68A4000              PUSH    00408AC6`  
> `00408B31    E8 8ECEFFFF              CALL    004059C4`  
> `00408B36    83C4 0C                  ADD     ESP, 0C`  
> `00408B39  - E9 1488FFFF              JMP     00401352`  

注意的是 exe 本身就提供了 `MessageBoxA` 的调用，可以直接用函数 `4059C4`，参数是：

消息内容: `string`;  
窗口标题: `string`;  
窗口 ICON: `Integer`;  

调用完后配平堆栈，返回消息循环处，收工。

这题比上题难多了，要不是有看雪精华、`exescope`、`lordpe`、`winhex`、`ue`、`计算器`、`delphi`、`vc`、`msdn`、`baidu`、`google` 等的大力协助，几乎无法完成。

觉得这里改比较好：

将原窗口消息处理函数里这段代码的 `call ds:DefWindowProcA`

```
.text:004012EC                 push    [ebp+arg_C]
.text:004012EF                 push    [ebp+arg_8]
.text:004012F2                 push    [ebp+arg_4]
.text:004012F5                 push    [ebp+arg_0]
.text:004012F8                 call    ds:DefWindowProcA
.text:004012FE                 jmp     short locret_401354
```

改成 `call MyWndProc`,函数代码如下:

```
LRESULT CALLBACK MyWndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
  if ( message == WM_COMMAND && wParam == 0x68)
  {
    MessageBox(NULL,"看雪论坛.珠海金山 2007 逆向分析挑战赛\r\nhttp://www.pediy.com","pediy",0x40);  
    return 0;  
  }

  return DefWindowProc(hWnd, message, wParam, lParam);
}
```

还能多出一个空余字节 :)

偶也来凑下热闹,相同的东东不贴了,只贴上偶 Patch 的代码吧:

找一空地写入下面的代码:

```
00406B80      817C24 08 11010000    cmp     dword ptr [esp+8], 111          ;WM_COMMAND
00406B88    ^ 0F85 47A7FFFF         jnz     004012D5
00406B8E      66:837C24 0C 68       cmp     word ptr [esp+C], 68            ;About 命令 ID,资源文件自定义时的值
00406B94    ^ 0F85 3BA7FFFF         jnz     004012D5
00406B9A      60                    pushad
00406B9B      6A 40                 push    40
00406B9D      68 8C804000           push    0040808C                         ;  ASCII "pediy"
00406BA2      68 C06B4000           push    00406BC0
00406BA7      E8 18EEFFFF           call    004059C4                          ;延时载入的 MessageBox ,应该就是 VC 静态连接的 AfxMessageBox 吧
00406BAC      83C4 0C               add     esp, 0C
00406BAF      61                    popad
00406BB0      33C0                  xor     eax, eax
00406BB2      C2 1000               retn    10
```

```
00406BC0  BF B4 D1 A9 C2 DB CC B3 2E D6 E9 BA A3 BD F0  .看雪论坛.珠海金
00406BCF  C9 BD 32 30 30 37 C4 E6 CF F2 B7 D6 CE F6 CC F4  山 2007 逆向分析挑
00406BDF  D5 BD C8 FC 0D 0A 68 74 74 70 3A 2F 2F 77 77 77  战赛..http://www
00406BEF  2E 70 65 64 69 79 2E 63 6F 6D 00                 .pediy.com.
```

并改 `RegisterClass` 窗口消息处理入口地址:

`0040121A  |.  C745 B4 D5124>mov     dword ptr [ebp-4C], 004012D5           ;原消息处理入口地址`

为:

`0040121A      C745 B4 806B4000      mov     dword ptr [ebp-4C], 00406B80`

`DefWindowProc + AfxMessageBox + wc.lpszMenuName = 王道`

偶也偶的贴出来献丑一下，写的很烂

代码:

* * *

```
00406BB6   > \FF15 98B04000 CALL DWORD PTR DS:[<&user32.CreateMenu>]                                       
00406BBC   .  8BF0          MOV ESI,EAX  
00406BBE   .  68 AF6B4000   PUSH bithaha.00406BAF                                                          
00406BC3   .  68 00010000   PUSH 100                                                                        
00406BC8   .  6A 00         PUSH 0                                                                          
00406BCA   .  50            PUSH EAX                                                                          
00406BCB   .  FF15 9CB04000 CALL DWORD PTR DS:[<&user32.AppendMenuA>]                                       
00406BD1   .  FF15 98B04000 CALL DWORD PTR DS:[<&user32.CreateMenu>]                                       
00406BD7   .  68 A96B4000   PUSH bithaha.00406BA9                                                          
00406BDC   .  56            PUSH ESI                                                                       
00406BDD   .  8BF0          MOV ESI,EAX                                                                    
00406BDF   .  6A 10         PUSH 10                                                                       
00406BE1   .  50            PUSH EAX                                                                        
00406BE2   .  FF15 9CB04000 CALL DWORD PTR DS:[<&user32.AppendMenuA>]                                      
00406BE8   .  6A 00         PUSH 0  
00406BEA   .  53            PUSH EBX  
00406BEB   .  56            PUSH ESI  
00406BEC   .  B8 00000080   MOV EAX,80000000  
00406BF1   .  33F6          XOR ESI,ESI  
00406BF3   .^ E9 79A6FFFF   JMP bithaha.00401271//返回到 createwindow 前  
00406BF8   >  3D 11010000   CMP EAX,111  
00406BFD   .  75 2D         JNZ SHORT bithaha.00406C2C  
00406BFF   .  90            NOP  
00406C00   .  90            NOP  
00406C01   .  90            NOP  
00406C02   .  90            NOP  
00406C03   .  56            PUSH ESI  
00406C04   .  8B75 10       MOV ESI,DWORD PTR SS:[EBP+10]  
00406C07   .  81FE 00010000 CMP ESI,100  
00406C0D   .  5E            POP ESI  
00406C0E   .  75 1C         JNZ SHORT bithaha.00406C2C  
00406C10   .  90            NOP  
00406C11   .  90            NOP  
00406C12   .  90            NOP  
00406C13   .  6A 00         PUSH 0                                                                          
00406C15   .  68 976B4000   PUSH bithaha.00406B97                                                        
00406C1A   .  68 746B4000   PUSH bithaha.00406B74                                                          
00406C1F   .  6A 00         PUSH 0                                                                          
00406C21   .  FF15 A0B04000 CALL DWORD PTR DS:[<&user32.MessageBoxA>]                                    
00406C27   .^ E9 28A7FFFF   JMP bithaha.00401354  
00406C2C   >  48            DEC EAX  
00406C2D   .  48            DEC EAX  
00406C2E   .^ 0F84 16A7FFFF JE bithaha.0040134A  
00406C34   .  83E8 03       SUB EAX,3  
00406C37   .^ E9 A9A6FFFF   JMP bithaha.004012E5//返回到窗口回调函数
```

* * *

我的处理 Pe 文件代码，唉入口点通过 Peid 的特征码找的，可是就是那个菜单没做出来，差啊。。。

```
unit PEHandler;
```

```
interface

uses
  Classes, Forms, sysutils, windows;

type

  {PE 导出表声明}
  PImageExportDirectory = ^TImageExportDirectory;
  TImageExportDirectory = packed record
    Characteristics: DWORD;
    TimeDateStamp: DWORD;
    MajorVersion: WORD;
    MinorVersion: WORD;
    Name: DWORD;
    Base: DWORD;
    NumberOfFunctions: DWORD;
    NumberOfNames: DWORD;
    AddressOfFunctions: DWORD;
    AddressOfNames: DWORD;
    AddressOfNameOrdinals: DWORD;
  end;

  PImportByName=^TImportByName;
  TImportByName =Packed  record
    ProcedureHint: word;
    ProcedureName: array[0..1]of char;
  end;

  PImageImportDescriptor = ^TImageImportDescriptor;
  TImageImportDescriptor = packed record
    OriginalFirstThunk: DWord;
    TimeDateStamp  : DWord;
    ForwarderChain : DWord;
    DLLName        : DWord;
    FirstThunk     : DWord;
  end;

  PImageThunkData=^TImageThunkData;
  TImageThunkData = record
  case integer of
    1:( ForwarderString : DWord; );
    2:( Function_       : DWord; );
    3:( Ordinal         : DWord; );
    4:( AddressOfData   : DWord; );
  end;
  PImageBaseRelocation=^TImageBaseRelocation;
  TImageBaseRelocation=Packed Record
     VirtualAddress: Dword;
     SizeOfBlock: Dword;
     TypeOffset: array[0..1] of Word;  
  end;

  TPESection = record    //自定义
    ObjectName: string;
    Address: PChar;
```



```pascal
PhysicalSize: Integer;

// VirtualSize: Integer;
// Characteristics: Cardinal;
PointerToRawData: Integer;
end;

TNameOrID = (niName, niID);

TPEImport = record
  NameOrID: TNameOrID;
  Name: string;
  ID: Integer;
//  PAddress: PChar; {指向导入表函数用于执行此函数}
end;

TPEImports = record {记录一个 Dll 文件所调用的函数个数}
  DLLName: string;
  Entries: array of TPEImport; {函数数据}
end;

TPEExport = record
  Name: string;
  RelativeID: Integer;
  ID: Integer;
  Address: DWORD;{相对地址}
end;


TPEHandler = class(TThread)
private
  { Private declarations }
  FPE, FTmp: TMemoryStream;
  FSectionFiles: array[0..3] of string;
  FSectionSizes: array[0..3] of integer;
  FImportCount: integer;
  FImportVA: integer;
  FIAT: integer;
  FIATSize: integer;
  procedure HandlerImport;
  function GetEntryPoint: integer;
  procedure GetPEHead;
  Procedure FillSection;
  procedure ChangPEHead;
  procedure ShowInfo(Info: string);
protected
  procedure Execute; override;
end;

implementation

uses uMain;

{ TPEHandler }

procedure TPEHandler.ChangPEHead;
var
  ImageDosHeader:     TImageDosHeader;
  ImageNtHeaders:     TImageNtHeaders;
  ImageSectionHeader: TImageSectionHeader;
  Sections: array of TImageSectionHeader;
  i, RVA, EP: integer;
begin
  FPe.Seek(0, soBeginning);
  FPE.ReadBuffer(ImageDosHeader, sizeof(ImageDosHeader));
  FPE.Position := ImageDosHeader._lfanew;
  FPE.ReadBuffer(ImageNtHeaders,sizeof(TImageNtHeaders));
//////////////////////////////////////////////////////////
//修改如下：

//File Header
  ImageNtHeaders.FileHeader.NumberOfSections := 4;

  //OptionalHeader
  ImageNtHeaders.OptionalHeader.SizeOfCode := FSectionSizes[0];
  ImageNtHeaders.OptionalHeader.SizeOfImage := FPE.Size;
  //ImageNtHeaders.OptionalHeader.SizeOfImage := FPE.Size - $1000;

  EP := GetEntryPoint;
  ImageNtHeaders.OptionalHeader.AddressOfEntryPoint := EP;
  ShowInfo('PE entry Point: 0x' + IntToHex(EP, 8));

  //修改输入表
  HandlerImport;

  for i := 0 to 15 do
  begin
    ImageNtHeaders.OptionalHeader.DataDirectory[i].VirtualAddress := 0;
    ImageNtHeaders.OptionalHeader.DataDirectory[i].Size := 0;
  end;

  //输入表
  ImageNtHeaders.OptionalHeader.DataDirectory[1].VirtualAddress := FImportVA;
  ImageNtHeaders.OptionalHeader.DataDirectory[1].Size := (FImportCount + 1) *
    sizeof(TImageImportDescriptor);

  //输入地址表
  ImageNtHeaders.OptionalHeader.DataDirectory[12].VirtualAddress := FIAT;
  ImageNtHeaders.OptionalHeader.DataDirectory[12].Size := $124;

////////////////////////////////////////////////////////////////////////////

  //写 改好的信息
  FPE.Position := ImageDosHeader._lfanew;
  FPE.WriteBuffer(ImageNtHeaders,sizeof(TImageNtHeaders));

  SetLength(Sections, 4);

  RVA := $1000;
  for i := 0 to 3 do
  begin
    Move(PChar(FSectionFiles[i])^,Sections[i].Name, length(FSectionFiles[i]));
    Sections[i].Misc.VirtualSize := FSectionSizes[i];
    Sections[i].VirtualAddress := RVA;
    RVA := RVA + FSectionSizes[i];
    Sections[i].SizeOfRawData := FSectionSizes[i];
    Sections[i].PointerToRawData := Sections[i].VirtualAddress;
    Sections[i].PointerToRelocations := 0;
    Sections[i].PointerToLinenumbers := 0;
    Sections[i].NumberOfRelocations := 0;
    Sections[i].NumberOfLinenumbers := 0;
    case i of
    0: Sections[i].Characteristics := $C0000060;
    1: Sections[i].Characteristics := $C0000040;
    2: Sections[i].Characteristics := $C00000C0;
    3:
    begin
      Sections[i].Characteristics := $C00000C0;
      //Sections[i].VirtualAddress := $18000;
    end;

    end;
    FPE.WriteBuffer(Sections[i],sizeof(TImageSectionHeader));
  end;

  Fillchar(ImageSectionHeader, sizeof(TImageSectionHeader), 0);
  FPE.WriteBuffer(ImageSectionHeader,sizeof(TImageSectionHeader));

  ShowInfo('change Pe Head');
end;

procedure TPEHandler.Execute;
begin
  FPE := TMemoryStream.Create;
  try

    FSectionFiles[0] := '_text';
    FSectionFiles[1] := '_rdata';
    FSectionFiles[2] := '_data';
    FSectionFiles[3] := '_rsrc';

    FTmp := TMemoryStream.Create;
    try
      GetPEHead;
      FillSection;
      ChangPEHead;
      FPE.SaveToFile('New.exe');
    finally
      FTmp.Free;
    end;

  finally
    FPE.Free;
  end;

end;

procedure TPEHandler.FillSection;
var
  rmod, i, wcount: integer;

begin
  for i := 0 to 3 do
  begin
    FTmp.LoadFromFile(FSectionFiles[i]);
    rMod := FTmp.Size mod $1000;
    if rMod <> 0 then
    begin
      FTmp.Size := FTmp.Size + $1000 - rMod;
    end;
    FSectionSizes[i] := FTmp.Size;

    FPe.Seek(0, soEnd);
    wcount := FPe.Write(PChar(FTmp.Memory)^, FTmp.Size);
    ShowInfo('Fill Section ' + FSectionFiles[i] + ', size: 0x'
      + IntToHex(WCount, 8));
  end;

end;

function TPEHandler.GetEntryPoint: integer;
const
  VCEP: array[0..5] of byte = ($55, $8B, $EC, $6A, $FF, $68);
var
  rByte: byte;
  i: integer;
  bFind: Boolean;
begin
  Result := 0;
  FPe.Position := $1000;
  while FPe.Position < $1000 + FSectionSizes[0] do
  begin
    bFind := true;
    for i := 0 to 5 do
    begin
      FPe.ReadBuffer(rByte, 1);
      if rByte <> VCEP[i] then
      begin
        FPe.Position := FPe.Position - i;
        bFind := false;
        Break;
      end;
    end;
    if bfind then
    begin
      Result := FPe.Position - 6;
      exit;
    end;

  end;
end;

procedure TPEHandler.GetPEHead;
begin
  FTmp.LoadFromFile('1.exe');
  FPe.Size := $1000;
  FPe.Seek(0, soBeginning);
  FTmp.Size := $1000;
  FPe.LoadFromStream(FTmp);
  ShowInfo('Get Pe Head from: ''1.exe''');

end;

procedure TPEHandler.HandlerImport;
const
  LastRVA: integer = $0654;
var
  iRVA, SectionVA, LastVA, DllVA: integer;
  ImageImportDescriptor: TImageImportDescriptor;
  DLLName: array[1..32] of char;
  DllCount: integer;
begin
  SectionVA := FSectionSizes[0]  + $1000;
  LastVA := SectionVA + LastRVA;
  FImportCount := 0;
  FIAT := $0FFFFFFF;
  while True do
  begin
    FPE.Position := LastVA - (FImportCount + 1) * sizeof(TImageImportDescriptor);
    FPe.ReadBuffer(ImageImportDescriptor, sizeof(TImageImportDescriptor));
    DllVA := ImageImportDescriptor.DLLName;
    if (Integer(DllVA) > sectionVA + FSectionSizes[1]) or
    (Integer(DllVA) < sectionVA) then Break;

    if FIAT > ImageImportDescriptor.FirstThunk then
      FIAT := ImageImportDescriptor.FirstThunk;

    ShowInfo('IAT: 0x' + IntToHex(ImageImportDescriptor.FirstThunk, 8));

    FPE.Position := DLLVA;
    FPE.ReadBuffer(DLLName, 32);
    ShowInfo('Import Dll Name:' + DllName);
    Inc(FImportCount);
  end;
  ShowInfo('Import Count:' + IntToStr(FImportCount));
```



`FImportVA := SectionVA + LastRVA - FImportCount * sizeof(TImageImportDescriptor);`

`ShowInfo('Import VA: 0x' + IntToHex(FImportVA, 8));`

`end;`

`procedure TPEHandler.ShowInfo(Info: string);`

`begin`

`frmMain.lbInfo.Items.Add(Info);`

`Application.ProcessMessages;`

`end;`

`end.`

这道题用 Delphi 我想可以这样做：

写一个 DLL，在 pediy 的窗口创建后 `loadlibary` 进去。

在程序里创建 `TMainMenu` 类，然后添加 Help 和 About，在用 `SetMenu(MainMenu1.Handle);` 来把菜单挂上去。

然后做一个消息钩子把窗口的消息钩住，拦截 `WM_COMMAND` 消息，再在 Hook 程序内 `MessageBox` 就可以了。

当时我是用直接写汇编来做的：

分别在 `CreateWindowEx` 和 `DispatchMessage` 那里作跳转，然后对应的两段代码如下：

代码:

```
00406C00   > \FF15 10714000    CALL DWORD PTR DS:[<&USER32.CreateWindow>; \CreateWindowExA
00406C06   .  60               PUSHAD
00406C07   .  A3 F0B04000      MOV DWORD PTR DS:[40B0F0],EAX
00406C0C   .  68 B8754000      PUSH pediy02.004075B8                    ; /FileName = "user32.dll"
00406C11   .  FF15 6C704000    CALL DWORD PTR DS:[<&KERNEL32.LoadLibrar>; \LoadLibraryA
00406C17   .  8BF8             MOV EDI,EAX
00406C19   .  8B35 A0704000    MOV ESI,DWORD PTR DS:[<&KERNEL32.GetProc>; k kernel32.GetProcAddress
00406C1F   .  68 10B04000      PUSH pediy02.0040B010                    ; /ProcNameOrOrdinal = "CreateMenu"
00406C24   .  57               PUSH EDI                                 ; |hModule
00406C25   .  FFD6             CALL ESI                                 ; \GetProcAddress
00406C27   .  A3 80B04000      MOV DWORD PTR DS:[40B080],EAX
00406C2C   .  68 20B04000      PUSH pediy02.0040B020                    ; /ProcNameOrOrdinal = "InsertMenuItemA"
00406C31   .  57               PUSH EDI                                 ; |hModule
00406C32   .  FFD6             CALL ESI                                 ; \GetProcAddress
00406C34   .  A3 84B04000      MOV DWORD PTR DS:[40B084],EAX
00406C39   .  68 50B04000      PUSH pediy02.0040B050                    ; /ProcNameOrOrdinal = "SetMenu"
00406C3E   .  57               PUSH EDI                                 ; |hModule
00406C3F   .  FFD6             CALL ESI                                 ; \GetProcAddress
00406C41   .  A3 88B04000      MOV DWORD PTR DS:[40B088],EAX
00406C46   .  68 60B04000      PUSH pediy02.0040B060                    ; /ProcNameOrOrdinal = "MessageBoxA"
00406C4B   .  57               PUSH EDI                                 ; |hModule
00406C4C   .  FFD6             CALL ESI                                 ; \GetProcAddress
00406C4E   .  A3 8CB04000      MOV DWORD PTR DS:[40B08C],EAX
00406C90   .  FF15 80B04000    CALL DWORD PTR DS:[40B080]               ; USER32.CreateMenu
00406C96   .  A3 90B04000      MOV DWORD PTR DS:[40B090],EAX
00406C9B   .  68 00B14000      PUSH pediy02.0040B100
00406CA0   .  68 FF000000      PUSH 0FF
00406CA5   .  6A 00            PUSH 0
00406CA7   .  50               PUSH EAX
00406CA8   .  FF15 84B04000    CALL DWORD PTR DS:[40B084]               ; USER32.InsertMenuItemA
00406CAE   .  A1 90B04000      MOV EAX,DWORD PTR DS:[40B090]
00406CB3   .  A3 44B14000      MOV DWORD PTR DS:[40B144],EAX
00406CB8   .  FF15 80B04000    CALL DWORD PTR DS:[40B080]               ; USER32.CreateMenu
00406CBE   .  A3 94B04000      MOV DWORD PTR DS:[40B094],EAX
00406CC3   .  68 30B14000      PUSH pediy02.0040B130
00406CC8   .  68 FF000000      PUSH 0FF
00406CCD   .  6A 00            PUSH 0
00406CCF   .  50               PUSH EAX
00406CD0   .  FF15 84B04000    CALL DWORD PTR DS:[40B084]               ; USER32.InsertMenuItemA
00406CD6   .  FF35 94B04000    PUSH DWORD PTR DS:[40B094]
00406CDC   .  FF35 F0B04000    PUSH DWORD PTR DS:[40B0F0]
00406CE2   .  FF15 88B04000    CALL DWORD PTR DS:[40B088]               ; USER32.SetMenu
00406CE8   .  61               POPAD
00406CE9   .^ E9 9AA5FFFF      JMP pediy02.00401288
```

代码:

```
00406D00   > \8178 04 11010000 CMP DWORD PTR DS:[EAX+4],111
00406D07   .  75 77            JNZ SHORT pediy02.00406D80
00406D09   .  6A 40            PUSH 40
00406D0B   .  68 8C804000      PUSH pediy02.0040808C                    ; ASCII "pediy"
00406D10   .  68 00B24000      PUSH pediy02.0040B200
00406D15   .  FF35 F0B04000    PUSH DWORD PTR DS:[40B0F0]
00406D1B   .  FF15 8CB04000    CALL DWORD PTR DS:[40B08C]               ; USER32.MessageBoxA
00406D80   > \FF15 24714000    CALL DWORD PTR DS:[<&USER32.DispatchMess>; \DispatchMessageA
00406D86   .^ E9 37A5FFFF      JMP pediy02.004012C2
```

4dd466d0e1e0cc123afbe504da2462ca
