
```text
.text:00602218 push 0 ; lpWindowName
.text:0060221A push offset off_602264 ; lpClassName
.text:0060221F call FindWindowW
.text:00602224 mov esi, eax
.text:00602226 test esi, esi
.text:00602228 jz short loc_60225E
.text:0060222A push 0 ; lpWindowName
.text:0060222C push offset off_602264 ; lpClassName
.text:00602231 call FindWindowW（查找游戏窗口）
.text:00602236 mov esi, eax
.text:00602238 push 3F0001h ; lParam
.text:0060223D push 74h ; wParam
.text:0060223F push 100h ; Msg
.text:00602244 push esi ; hWnd
.text:00602245 call PostMessageW （模拟按键）
.text:0060224A push 0C03F0001h ; lParam
.text:0060224F push 74h ; wParam
.text:00602251 push 101h ; Msg
.text:00602256 push esi ; hWnd
.text:00602257 call PostMessageW （模拟松开）
.text:0060225C mov bl, 1
.text:0060225E
.text:0060225E loc_60225E: ; CODE XREF: sub_602214+14↑j
.text:0060225E mov eax, ebx
.text:00602260 pop esi
.text:00602261 pop ebx
.text:00602262 retn
.text:00602262 sub_602214 endp
```

以上代码即为登陆器模拟按键操作，通知游戏进程进行截图。

**验证：**

![](upload/attach/202101/916753_J5JNDTVERE5DHA3.png)

发现有两处调用此处代码，把这两处屏蔽掉，屏蔽代码的方式有很多，`nop`掉可以，或者`Jmp`掉，我用`ret`把这两处都屏蔽掉的，这样做有个好处，只改这个函数内的第一个代码就可以实现两处调用全部屏蔽，而且保证原来的寄存器数据安全，因为`nop`和`Jmp`可能会涉及到数据安全问题，导致程序发生异常。（不知道这样想对不对）

下面是修改后的代码：

```text
t:00602214 locret_602214: ; CODE XREF: .text:006027B9↓p
.text:00602214 ; sub_60338C+2F↓p
.text:00602214 retn 原先是push ebp
.text:00602215 ; ---------------------------------------------------------------------------
.text:00602215 push esi
.text:00602216 xor ebx, ebx
.text:00602218 push 0 ; lpWindowName
.text:0060221A push offset off_602264 ; lpClassName
.text:0060221F call FindWindowW
.text:00602224 mov esi, eax
.text:00602226 test esi, esi
.text:00602228 jz short loc_60225E
.text:0060222A push 0 ; lpWindowName
.text:0060222C push offset off_602264 ; lpClassName
.text:00602231 call FindWindowW
.text:00602236 mov esi, eax
```

这样，运行登陆器，发现不会截图了。就这样，破解登陆器的截图反作弊就基本实现了第一步；但是这样有个问题，就是很容易被服务器发现的，因为服务器没有截图，是会被怀疑作弊的。

那怎样实现反反作弊呢？这个问题就有意思了。其实相关代码找到了，实现反反作弊也是完全可能的，这里我只说一下自己的思路，其实就是在通知截图之前先把作弊器关掉就可以，截图完成后再开启，就完全能够躲避反作弊的。但是我个人不建议这样做，毕竟文明游戏才是正道。尤其对于为数不多的还有人玩的经典游戏，作弊就更没必要了。

## 5. 登陆器是多线程的，这样设计有个好处

```text
loc_406B9C:
mov ds:byte_670059, 1
mov eax, [ebp+lpThreadId]
push eax ; lpThreadId
mov eax, [ebp+dwCreationFlags]
push eax ; dwCreationFlags
push ebx ; lpParameter
mov eax, offset unknown_libname_50 ; BDS 2005-2007 and Delphi6-7 Visual Component Library
push eax ; lpStartAddress
mov eax, [ebp+dwStackSize]
push eax ; dwStackSize
mov eax, [ebp+lpThreadAttributes]
push eax ; lpThreadAttributes
call CreateThread
mov esi, eax
test esi, esi
jnz short loc_406BD1
```

---

![](upload/attach/202101/916753_RCSV2EB3EHHF3A6.png)

这是几处dephi线程调用的入口，跟踪发现，登陆器的玩家信息（界面右边的窗口显示区域）是独立线程操作的，截图和上传功能也是独立线程操作的，但是在对线程下断点时，发现登陆器容易发生线程调用异常错误，估计是线程间通信被中断导致的句柄错误（不知道对不对）。

## 感悟

- 代码定位如果一个思路不行，就多换几个思路，在可能性最高的地方下断点进行拦截分析，能够节省不少时间；
- 每一种编译平台对代码的处理方式均有自己的方式，尤其是Dephi;
- 实践和理论结合起来，就能成长的更快；
- 多来看雪论坛，学到更多知识；
- 因为我是小白，文中定有不少漏洞和错误，希望大佬给出指正；
- 感谢平台。

最后，我有个问题：我这算不算入行了啊？

## 前言

1. 我是出于对反作弊代码实现的好奇，和想知道这款软件中有无恶意代码，所以想跟踪一下；同时声明本次破解并无恶意，仅仅是作为一次尝试和练习的过程。
2. 我是真正的`0`基础开始学习，并无编程经验，也非IT魔界人士，只是对计算机安全感兴趣，看过不少相关资料的书籍；也经常浏览看雪论坛中各位大神的技术分享，收获颇丰，本次就自己斗胆尝试了一下破解的过程。
3. 当然，这次破解我选择的破解对象也比较简单，如果您是高手的话完全可以路过的。下面是这款游戏（Cs1.`5`）反作弊登陆器的界面：

![](https://bbs.kanxue.com/upload/attach/202101/916753_PEGJHTRTPW4PH9K.png)

当然这是一款比较老的游戏，如今也只有少数痴迷、热爱的朋友在玩。（是不是暴露年龄了？）

## 工具

使用的几款工具：

- `PEiD`;
- `IDA pro 7`；逆向神器，静态分析读代码，特别是交叉引用分析，对代码分析作用很大；
- `WinDbg Preview`;动态分析，异常捕获分析，动态和静态联合分析对我这次逆向启发很大；
- `DeDeDark`（本次没用上，具体是破解程序使用了针对这款逆向工具的反跟踪，还是DD软件的原因目前还不太清楚；我个人觉得前者的可能性高，因为在跟踪过程中发现很多“`int 3`”的调试陷阱代码，具体是不是限于个人水平没有弄清楚，这里请大佬指正。）

## 思路和步骤

### 1. 判断软件是基于什么编译系统开发的，并且有无壳的保护

1. 用`PEiD`查看，发现是`Dephi`开发的，这里就不截图了。我对于`Dephi`比较陌生，可以说是完完全全的小白，以为什么平台在底层看来其实都无所谓，尤其是从汇编看，所以开始有点盲目的开心和自信了。同时发现，这款反作弊的登陆器居然没有壳；这是令我高兴的，高兴的是，事情又简单了一步，（其实大错特错，这完全属于小白的盲目自信和无知的傻开心）；但又有点小失望，因为我还是想试一下破壳的，可惜这次没遇到机会。
2. 其实`PEiD`仅仅能起到参考作用，因为很多软件的反调试完全可以迷惑`PEiD`的，就是提示是`Dephi`并不一定真的就是，所以不要认真了。反正代码跟踪就是真么真真假假的。（个人体会）

### 2. 熟悉反作弊登陆器的功能，判断反作弊的机制是什么

1. 通过几次使用登陆器确定游戏过程中，反作弊登陆器有截图，并上传服务器的过程，所以推测基本就是依靠截图判断玩家是否使用了作弊器。如下截图：

    ![](https://bbs.kanxue.com/upload/attach/202101/916753_MUEXCXU2SFHCVJ8.png)

    ![](https://bbs.kanxue.com/upload/attach/202101/916753_GHCVYF9AHU5K7YA.png)

2. 如果玩家使用了透视作弊，截图是能发现的。熟悉这款游戏的朋友可能问，如果是自动瞄准呢？如果作弊器隐蔽程度很高的话，侦测起来就难了（大佬请提供一下思路）；所以这次，主要是破解软件的截图功能。

3. 了解反作弊机制的目的就是为了方便之后进行代码定位。既然登陆器使用的是截图，那就清楚了，只要跟踪到截图功能的代码就可以了。其实，这样的思路没错，但可能忽视了一个关键问题，那就是截图究竟是登陆器实现的，还是游戏软件自身实现的？我在这个问题上绕了很多弯路，这个问题表面看起来简单，但对于我这小白来说确实是个不小的挑战；因为，登陆器和游戏是两个软件，系统里看是两个进程，这就涉及到两个进程间的通信机制，这个问题就难了。于是，就边做边学，啃了网上很多Windows平台的进程通信方面的资料，跟踪了几处代码，发现都不是，这着实令人头大了。

### 3. 登陆器是如何启动游戏进程的

首先，登陆器启动游戏进程，需要一次进程调用，Window平台启动进程函数`WinExec`、`ShellExecute`、`CreateProcess`，因此依次定位这几处代码的调用，判断登陆器是如何启动游戏进程的。

![](https://bbs.kanxue.com/upload/attach/202101/916753_VXPGKEERSJPVTMH.png)

#### 有4处`ShellExecute`的调用：

```text
sub_5FAADC：

push ebp
push offset loc_5FAB2A
push dword ptr fs:[eax]
mov fs:[eax], esp
push 5 ; nShowCmd
push 0 ; lpDirectory
push 0 ; lpParameters
mov eax, [ebp+var_4]
call sub_407954
push eax ; lpFile
push offset Operation ; "open"
push 0 ; hwnd
call ShellExecuteW （此处并非游戏启动）
xor eax, eax
pop edx
pop ecx
pop ecx
mov fs:[eax], edx

sub_609B30：

mov ecx, [ebp+var_4]
mov edx, offset aOld ; "old"
call sub_407BC0
mov eax, [ebp+var_5C]
call sub_407954
push eax ; lpParameters
lea edx, [ebp+var_64]
mov eax, [ebx]
call unknown_libname_1018 ; BDS 2005-2007 and Delphi6-7 Visual Component Library
mov eax, [ebp+var_64] ; System::AnsiString
lea edx, [ebp+var_60]
call @Sysutils@ExtractFilePath$qqrx17System@AnsiString ; Sysutils::ExtractFilePath(System::AnsiString)
lea eax, [ebp+var_60]
mov edx, [ebp+var_4]
call sub_407B68
mov eax, [ebp+var_60]
call sub_407954
push eax ; lpFile
push offset aOpen_0 ; "Open"
push 0 ; hwnd
call ShellExecuteW （此处也不是）
mov eax, [ebx] ; this
call @Forms@TApplication@Terminate$qqrv ; Forms::TApplication::Terminate(void)
xor eax, eax
pop edx
pop ecx
; int __fastcall TForm1_btn5Click(Ibcustomdataset::TIBDataSet *this)
_TForm1_btn5Click proc near
push ebx
mov ebx, eax
push 1 ; nShowCmd
push 0 ; lpDirectory
push 0 ; lpParameters
push offset File ; "6e4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2H3x3e0x3^5i4K6u0W2j5$3!0E0i4K6u0r3"
push 0 ; lpOperation
mov eax, ebx ; this
call @Ibcustomdataset@TIBDataSet@PSGetParams$qqrv ; Ibcustomdataset::TIBDataSet::PSGetParams(void)
push eax ; hwnd
call ShellExecuteW （此处是检查IP代码处，会打开浏览器自行检查机器IP，登陆器查看本机IP功能的实现）
pop ebx
retn
_TForm1_btn5Click endp

TForm1_btn6Click
push offset aIndexHtml ; "_index.html"
lea eax, [ebp+var_4]
mov edx, 6
call sub_407C48
push 1 ; nShowCmd
push 0 ; lpDirectory
push 0 ; lpParameters
mov eax, [ebp+var_4]
call sub_407954
push eax ; lpFile
push 0 ; lpOperation
mov eax, ebx ; this
call @Ibcustomdataset@TIBDataSet@PSGetParams$qqrv ; Ibcustomdataset::TIBDataSet::PSGetParams(void)
push eax ; hwnd
call ShellExecuteW（查看服务器截图，会打开浏览器）
xor eax, eax
pop edx
pop ecx
```

由此可见，登陆器至少还与系统进程、浏览器进程有协作，并非游戏进程一个，这从登陆器界面中也可以判断出来，如打开系统文件目录设置游戏启动路径，查看IP和截图。

`ShellExecuteW` 定位并未找到启动游戏进程代码入口，因此换`CreateProcess`，果然发现，一处父类的点击调用（通过IDA文字提示能找到很多程序本身的信息线索），应该判断是游戏的启动入口。

![](https://bbs.kanxue.com/upload/attach/202101/916753_NZF5GNDZGRDRFEJ.png)

```text
text:0060C82E loc_60C82E: ; CODE XREF: _TForm1_btn_StrarClick+549↑j
.text:0060C82E call unknown_libname_413 ; BDS 2005-2007 and Delphi6-7 Visual Component Library
.text:0060C833 call sub_456914
.text:0060C838 push offset off_60CD5C ; lpName
.text:0060C83D push 0 ; int
.text:0060C83F push 0 ; lpMutexAttributes
.text:0060C841 call @Windows@CreateMutexW$qqsp20_SECURITY_ATTRIBUTESipb ; Windows::CreateMutexW(_SECURITY_ATTRIBUTES *,int,wchar_t* )（判断游戏进程是否已经启动，如果启动会终止，保证一个游戏进程的运行）
.text:0060C846 lea eax, [ebp+ProcessInformation]
```
