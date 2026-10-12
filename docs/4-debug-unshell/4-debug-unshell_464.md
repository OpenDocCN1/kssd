
# Boilsoft Video Splitter v5.16&v5.21 脱壳笔记

> 原文：[`www.pediy.com/kssd/pediy11/108210.html`](https://www.pediy.com/kssd/pediy11/108210.html)

英文介绍如下：

Boilsoft Video Splitter is a tool to split, cut or trim a large AVI, MPEG, RM, ASF, WMV, 3GP or MP4 Video file into smaller video clips without re-encode. video splitter software using the included video player, you can easily split AVI, MPEG, RM, ASF, WMV, 3GP or MP4 Video file by time or the selection as needed. Direct stream cut, without re-encode, it is the most efficient and fastest in the market.

该软件功能比较强大，用来剪辑整理电影非常方便。于是想看看它是怎样编程实现的，顺手用 `peid` 查了下壳，`v5.16` 版本为 `ASProtect v1.23 RC1 *`，`v5.21` 版本为 `ASProtect 1.2x - 1.3x [Registered] -> Alexey Solodovnikov`，不管 `peid` 准不准，感觉是个猛壳。而我对脱壳也是一知半解，正好拿来实践一下。

捣鼓了一下，没有什么头绪，网上搜了搜也没有介绍脱该软件的文章，绿色汉化版倒是满天飞，均是出自论坛 `temerata` 前辈之手。我们也不好意思到论坛求助，还是自己搞定吧。

很奇怪，异常法不起作用(两个版本竟然都没有异常),只好手动调试跟踪，误打误撞尽然碰到了 `oep`。

简单总结一下找 `oep` 的方法：

### 方法一：人肉法

1、先下断点 `GetSystemTime`，断下后执行到返回再单步一下来到

```
00C02771    50             PUSH EAX  
00C02772    E8 01E9FFFF    CALL 00C01078    ; JMP 到 kernel32.GetSystemTime  
00C02777    0FB745 F0      MOVZX EAX,WORD PTR SS:[EBP-10] // 来到这里
```

2、搜索二进制串 `3130330D0A`，来到下面：

```
00C3F043    FF05 00000031  INC DWORD PTR DS:[31000000]  
00C3F049    3033           XOR BYTE PTR DS:[EBX],DH  
00C3F04B    0D 0A000000    OR EAX,0A  
00C3F050    8D00           LEA EAX,DWORD PTR DS:[EAX]    //我们关心的是这行，下硬件执行断点  
00C3F052    C3             RETN
```

3、`SHIFT-F9` 后到该处，执行到返回，再单步一下来到下面：

```
01CE0000    81EE 8876462F  SUB ESI,2F467688  
01CE0006    8BF8           MOV EDI,EAX  
01CE0008    E8 07000000    CALL 01CE0014  
01CE000D    A3 A0591EFF    MOV DWORD PTR DS:[FF1E59A0],EAX  
```

然后一直按“`Trace into`”就可以来到下面的 `oep` 了，这就是所谓的人肉法。

```
0044386A    E8 F6B50000    CALL VideoSpl.0044EE65  //oep  
0044386F  ^ E9 78FEFFFF    JMP VideoSpl.004436EC  
00443874    8BFF           MOV EDI,EDI
```

### 方法二：脚本法

```
gpa "GetSystemTime", "kernel32.dll"  
bp $RESULT  
esto  
bc $RESULT  
rtr  
sti  
GMEMI eip, MEMORYOWNER  
mov tmp1, $RESULT    
add tmp1, 1000  
find tmp1, #3130330D0A#     //search ASCII"103"  
mov tmp2, $RESULT  
find tmp2, #8D00C3#         //search "lea eax,[eax]" "ret"  
mov tmp1, $RESULT  
bphws tmp1, "x"  
eob lab0  
eoe lab0  
esto  
lab0:  
bphwc tmp1  
rtr  
sti   
GMEMI eip, MEMORYOWNER  
mov tmp1, $RESULT  
eval "eip < {tmp1}"   
ticnd $RESULT  //"Trace into" 操作，直到条件为真时停止 
```

以上方法受 `volx` 大侠脚本的启发。

在 `od` 中，查找互调调用可以看到这样的调用形式 `004019F6 CALL 01D00000`。必须要把它改回去，即改成如下样式：

`CALL DWORD PTR DS:[<&kernel32.SetFilePointer>]`

用脚本实现的原理就是在代码空间找 `CALL 01D00000` 这样的指令，而后借助壳的算法代码得到真正的 `api` 地址，再根据该地址到 `IAT` 表中搜索，得到存放该 `api` 地址的内存地址，而后在原指令的地方利用得到的这个内存地址重写该行 `call` 指令，难点就是如何利用壳的本身的 `call`。

不幸的是 `volx` 大侠的脚本虽然强大，但并不是万能的。该软件未采用 `vm` 技术但是也不能成功。通过调试发现直接用 `volx` 的脚本脱完再修复 `IAT` 后，双击运行没有反应。最后发现原来是程序中有几处关键跳转跳错了地方。

我用如下脚本解决了这个问题：

```
mov tmp1,imgbase  
add tmp1,10000  
find tmp1, #740c8d8c24f4000000#   //找修改第一处的位置    
mov tmp9, $RESULT  
mov [tmp9], #75#     
find tmp1, #743be921000000#   //找修改第二处的位置    
mov tmp9, $RESULT  
mov [tmp9], #75#
```

最后要注意的是修复 `IAT` 后仍然有 3 个无效指针，这其实是壳的 3 个 `api` `GetRegistrationInformation`、`GetTrialDays`、`CheckKey`，脱壳时重新 `patch` 代码实现。

在调试过程中发现该软件最重要的 `call` 如下：

```
0041527F    50             PUSH EAX  
00415280    51             PUSH ECX  
00415281    57             PUSH EDI  
00415282    56             PUSH ESI  
00415283    E8 76830000    CALL VideoSpl.0041D5FE   ; JMP 到 OFFSET rmutil.#2
```

就是 `RMUtil.dll` 中的`#2` 号功能调用，入口参数是要剪切的文件的 `path`、剪切得到的文件保存的 `path`、以及剪切的起始、结束时间。有了这个 `call`，其实我们自己也可以写个这样功能的小软件。

脱壳后简单修改了一下，截图一张以作留念。附件含脱壳脚本以及脱壳后程序。将该程序放在安装目录中就可以运行。

![](img/b4024bb524440a6837af52b0615db788.jpg)

天易 love  
2010-03-04

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || 脚本和脱壳版.rar  
|
