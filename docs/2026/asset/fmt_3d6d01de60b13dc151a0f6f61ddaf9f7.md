
(3) 蓝框内是第三个参数（PDU Body偏移`0x38`）：`nInt32`是一个四字节的`int`类型。我们可以看到`nInt32`参数的数据并不是紧跟着`wcsString`的数据尾部的，而是存放在了`wcsString`尾部两个字节之后。这是因为PUD Body的参数编码是需要进行数据对齐的，对齐方式是根据数据内容的大小进行1,2,4,8最大8字节的对齐。`nInt32`长度为4个字节，所以需要4字节对齐。所以需要在空了两个字节之后的位置存放。

(4) 金框内是第四个参数（PDU Body偏移`0x40`）：`nInt64`是一个八字节的`int64`类型，它因为需要8字节对齐，所以需要空4个字节之后的位置存放。

至此，所有参数都已经编码完成，这样我们就可以通过连续调用两次`NtAlpcSendWaitReceivePort`函数来自己构造RPC bind和RPC Request完成PRC的调用。**这里要说一点，由于ALPC系列函数微软只作为内部使用，所以其参数并未公开，调用`NtAlpcSendWaitReceivePort`函数的参数和属性的填写我是通过`windbg`调试得到的，由于这并不是本文的重点，所以这里不具体展开讲解。**具体请参考`AlpcDoMyRPCClient`工程源码。

# 三：开始尝试分析定位系统的RPC调用

我们已经对RPC有了基本认识，开始我们的最终目的：1. 确定某个我们感兴趣的系统功能是否为RPC方式调用。2. 分析定位出这个系统功能是怎么调用的，从而我们可以通过代码实现该RPC功能的调用。现在以`Explorer` "以管理员身份运行" 某个EXE这个功能为例子开始分析：

![](upload/attach/202304/560415_HGZJ5VN7VYWR38Y.jpg)

**我的具体思路：**

1. 首先要定位该功能所在的进程，这里我们随便用一个窗口查看工具就可以定位到右键弹出菜单是`Explorer`进程的窗口。

2. 对所有的RPC关键函数下特殊的条件断点（`NdrClientCall3`、`NdrClientCall2`、`NdrAsyncClientCall`、`Ndr64AsyncClientCall`、`NdrDcomAsyncClientCall`、`Ndr64DcomAsyncClientCall`），只要当我们点击 "以管理员身份运行" 按钮时该断点立刻断下，并且我们F5后立刻弹出UAC窗口，则我们就可以大概率定位断下来的的。

3. 我们通过代码模仿我们定位到的系统RPC调用，如果同样能弹出UAC窗口，则反向印证我们分析定位的没有问题。

**开始实战：**

通过工具可定位到该右键菜单为`Explorer`进程，我们直接通过`windbg`附加到`Explorer`进程，并对RPC函数下断。**这时候重点来了：我们会发现`Explorer`进程并不是像我们想象的那样，只有在点击 "以管理员身份运行" 按钮时才会触发我们下的RPC断点，`Explorer`进程无时无刻不在频繁的进行着RPC调用，我们的断点会被频繁的断下。**我的思路就是尽量想办法排除干扰项，在我没有点击 "以管理员身份运行" 按钮时产生的RPC请求一定都是干扰项。我们只关注点击后产生的RPC请求。因此我们需要一个条件断点。

在设置条件断点之前我们需要先了解一下RPC函数：

```
CLIENT_CALL_RETURN RPC_VAR_ENTRY Ndr*ClientCall*(
PMIDL_STUB_DESC pStubDescriptor,
PFORMAT_STRING pFormat,
...
);
```

```
CLIENT_CALL_RETURN RPC_VAR_ENTRY Ndr*ClientCall*(
MIDL_STUBLESS_PROXY_INFO *pProxyInfo,
unsigned long nProcNum,
void *pReturnValue,
...
);
```

所有的RPC函数都可以归类成上边这两种类型，这两种类型函数的第一个参数都是用来描述远端服务器信息的，其中就包含了RPC服务的GUID标识符。第二个参数是表示要调用远端RPC服务器的哪个函数，`pFormat`用于详细解释函数的参数的各种属性，`nProcNum`则表示是RPC服务器导出的第几个函。两种函数他们的参数的目的都一样，只不过参数的类型有所不同。

所以我们需要一个条件断点，这个条件断点有两个功能：1. 根据第一个参数里的GUID和第二个参数来决定哪些是"干扰项"，从而跳过"干扰项"。2. 如果断下来，则打印出：1. RPC服务的GUID。2. `pFormat`或`nProcNum`。

```
bm rpcrt4!Ndr*ClientCall* "
r $t0 = @rcx
r $t1 = @rdx
.if(@rdx > 0x1000)
{
r $t2 = poi(@$t0) + 4
.if( ((dwo(@$t2) == 0xe60c73e6) & (qwo(@$t1) == 0x40000`00006800)) )
{
gc
}
.else
{
.echo Format
db @$t2 l10
dd @$t2 l1
dq rdx l1
}
}
.else
{
r $t2 = poi(poi(@$t0)) + 4
.if(@$t2 == 0x04)
{
gc
}
.else
{
.if( ((dwo(@$t2) == 0xb18fbab6) & (@$t1 == 0x0A)))
{
gc
}
.else
{
.echo nNumber
db @$t2 l10
dd @$t2 l1
r rdx
}
}
}
"
bc 1-2
bc 4-5
bc 7
bc 9-12
bc 16
```

我编写好的可以直接使用的已经排除了大部分的"干扰项"的脚本文件`MyRPCBreakPoint.txt`已经放在了附件里以供直接使用。具体`windbg`断点语法不是本文重点，可以通过微软官网了解，会有详细的说明。上边是断点脚本文件的Demo演示：

第2行和第3行申请了两个伪寄存器`t0` `t1`，用于存放第一个和第二个参数。

第4行 `.if(@rdx > 0x1000)` 用来判断RPC函数类型，如果小于等于`1000`则说明第二个参数是函数序号，如果大于`1000`则说明是函数格式。

第6行和第21行申请了伪寄存器`t2`，从`t0`中取出GUID指针,并将GUID指针放入`t2`中，由于两种函数的参数不一样，一个是`PMIDL_STUB_DESC`类型，一个是`MIDL_STUBLESS_PROXY_INFO`，所以取GUID指针的方式也不一样。

第7行和第28行是重点，如果满足条件则说明是"干扰项"则会`gc`也就是继续运行，否则会打印出：1. RPC服务的GUID。2. `pFormat`函数格式 或者是 `nProcNum`函数序号。

第42行开始是清理除了（`NdrClientCall3`、`NdrClientCall2`、`NdrAsyncClientCall`、`Ndr64AsyncClientCall`、`NdrDcomAsyncClientCall`、`Ndr64DcomAsyncClientCall`）这几个RPC函数之外的函数断点，由于我们下的是批量断点，所以也会误下一部分命中我们名字规则的断点。**请注意，这里每个人的机器肯定是不一样的，所以读者需要根据自身下断点情况来自己清理断点。**

接下来我实际演示一遍如何将"干扰项"放入 `.if` 判断中，从而排除干扰项：

首先`windbg`命令行输入`$$>a<C:\MyRPCBreakPoint.txt`，运行脚本文件，之后在没点击"以管理员身份运行" 时断点也会不停的断下来，每断下来一个，我们就把这个"干扰项"的两个参数保存放入 `.if` 判断中。

![](upload/attach/202304/560415_DSPERDN6FD6UDC4.jpg)

如上图，输入脚本文件路径，然后F5跑起来，`windbg`立刻会断下。脚本输出`Format`表示函数是两种类型的第一种。`0xe60c73e6`是RPC服务GUID的前4个字节，为了简单我的脚本文件并没有完整判断GUID的16个字节，只判断了前4个字节。`0x4000000006800`是函数`PFORMAT_STRING`的前8个字节，为了简单也是只取了前8个字节。将`0xe60c73e6`和`0x4000000006800`放入第7行的 `.if` 判断中，即可排除此"干扰项"。

![](upload/attach/202304/560415_9S8FGJRVY76WZYK.jpg)

脚本输出`nNumber`表示函数是两种类型的第二种，`0xb18fbab6`是RPC服务GUID的前4个字节，`0x0A`是函数序号。将上面的`0xb18fbab6`和`0x0A`放入第28行的 `.if` 判断中，即可排除此"干扰项"。

按照上边的方法即可慢慢排除所有干扰项，直到在点击"以管理员身份运行"按钮前不会有任何RPC函数断下.这样的方法缺点就是比较耗时，但是优点就是方法比较通用，大多数RPC功能都可使用此方法定位，而且可以用来确定一个功能是否是通过RPC调用的。

接下来我们点击右键弹出菜单的"以管理员身份运行"，`windbg`立刻断下：

![](upload/attach/202304/560415_SVRYBB9R8SCWFTD.jpg)

根据我们的脚本输出我们可以看到GUID为`201ef99a`开头：`{201ef99a-7fa0-444c-9399-19ba84f12a1a}`，函数序号为0的的RPC请求断下。查看堆栈信息，我们可以看到是模块名称为`windows_storage.dll`的`AicLaunchAdminProcess`函数调用过来的。再随便翻看一下函数参数，我们发现函数的第6个参数刚好就是我们"以管理员身份运行"的EXE的路径，再看函数名称也很像我们的目标函数。那么这个RPC调用被我们列为重点怀疑对象。

先不急去验证，我们继续F5，直到弹出UAC窗口才算分析结束, `windbg`再次断下：

![](upload/attach/202304/560415_3XBN5GHNCNGBB6Q.jpg)

这次的GUID为`{e1af8308-5d1f-11c9-91a4-08002b14a0fa}`，函数序号为`0x08`，我们查看函数堆栈，发现是上面的`AicLaunchAdminProcess`函数调用`Ndr64AsyncClientcall`函数后其内部又再次调用的RPC请求，这可能会让人有些迷惑，其实这是RPC协议中规定好的一个基础RPC服务，名字叫"Endpoint Mapper"，它的作用就是保存好已经注册的其他的RPC服务的端口号。因为RPC服务在注册时是可以不填写端口号的，这样系统会为该RPC服务生成随机端口号并将其保存在"Endpoint Mapper"中。该RPC服务不是我们本次讨论的重点，这个留给读者作为第一个练手用的RPC服务。

继续F5，UAC窗口成功弹出，这期间一共调用了两次RPC请求，已知第二个RPC请求是"Endpoint Mapper"请求，第一个RPC根据函数名字和参数都有很大的可能性是"以管理员身份运行"按钮对应的RPC请求。

接下来我们需要使用在Github上的一个叫做"RpcView"的开源工具，下载地址是[711K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9h3I4$3k6i4u0X3x3s2S2Q4x3V1k6J5M7r3y4$3K9h3g2%4](elink@40aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9h3I4$3k6i4u0X3x3s2S2Q4x3V1k6J5M7r3y4$3K9h3g2%4)。它可以将目标RPC服务的所有接口导出到IDL文件中，具体使用方法如图：

![](upload/attach/202304/560415_VMAQPMCE52FJ48Y.jpg)

管理员方式运行`RpcView.exe`，然后CTRL+F搜索GUID是`0x201ef99a`开头的RPC服务。右键Decompile，此RPC服务的接口就会以文本的方式被导出。接下来我们根据函数声明和`windbg`调试，就可以依葫芦画瓢的填写好我们的参数：

```
RPC_BINDING_HANDLE UacBinding = 0;
STARTUPINFOW* StartInfo = 0;
WCHAR wcsExePath[] = { L"C:\\Windows\\system32\\notepad.exe" };
WCHAR wcsParam[] = { L"\"C:\\Windows\\system32\\notepad.exe\" aabbcc.txt" };
DWORD dwNumber = 0x11;
DWORD dwNumber2 = 0x04080414;
WCHAR wcsDir[] = L"D:\\data";
WCHAR wcsDesktop[] = { L"WinSta0\\Default" };
Struct_22_t Struct22;
memset(&Struct22, 0, sizeof(Struct_22_t));
Struct22.StructMember8 = 1;
Struct22.StructMember9 = 5;
DWORD64 dwNumber3 = 0;
DWORD dwNumber4 = -1;
PROCESS_INFORMATION ProcInfo;
memset(&ProcInfo, 0, sizeof(PROCESS_INFORMATION));
long nNumberOut = 6;
status = RpcStringBindingComposeW((RPC_WSTR)L"201ef99a-7fa0-444c-9399-19ba84f12a1a", (RPC_WSTR)L"ncalrpc", 0i64, 0i64, 0i64, &StringBinding);
status = RpcBindingFromStringBindingW(StringBinding, &UacBinding);
LaunchAdminProcess(UacBinding,
wcsExePath, wcsParam,
dwNumber, dwNumber2,
wcsDir, wcsDesktop,
&Struct22, dwNumber3, dwNumber4,
(Struct_56_t*)&ProcInfo, &nNumberOut);
```

其中`wcsExePath`是要运行的EXE路径，`wcsParam`是运行参数，`ProcInfo`是`PROCESS_INFORMATION`结构，用于返回进程创建信息。具体调用方式请参考附件中的源码。我们运行测试代码，成功以管理员权限运行了`notepad.exe`程序并传递参数，反向印证了我们的猜测：第一个RPC请求就是"以管理员身份运行"按钮对应的RPC请求。

**Good！我们的工作已经完成了一大半，接下来我们要通过`NTAlpc*`函数实现我们的RPC请求：**

为了方便起见，就不再调试`Explorer`来查看`NTAlpc*`函数参数了，我们直接调试只有一个RPC请求以管理员方式运行`Notepad.exe`的`RPCRunAdminProcess`工程。像第二章节一样老样子，`windbg`打开`RPCRunAdminProcess`工程，`bm ntdll!*alpc*` 对alpc相关函数批量下断点，然后F5:

```
0:000:x86> g
Breakpoint 52 hit
ntdll!NtAlpcConnectPort:
00007ffb`0750dfc0 4c8bd1 mov r10,rcx
0:000> dq rdx
00000000`0037deb0 00000000`002c002a 00000000`009cbdc8
00000000`0037dec0 0000000c`02090000 009c0000`00000001
0:000> du 009cbdc8
00000000`009cbdc8 "\RPC Control\epmapper"
```

`windbg`断下，查看端口号，是"Endpoint Mapper"服务的端口号，上边已经讲过，对于随机端口的RPC服务，要先通过RPC请求"Endpoint Mapper"服务来获取目标RPC服务的端口号。这不是本文的重点，留给读者用来练手。

我们一路F5，跳过和"Endpoint Mapper"相关的Alpc操作，直到再次断在`AlpcConnect`函数：

![](upload/attach/202304/560415_FG9XZ5TBHM2GYBH.jpg)

其端口号是`LRPC-50ddf8142f3561885f`，刚好和我们上面通过`RpcView.exe`查询到的RPC服务GUID对应的端口号一致。

F5，函数断在了`NtAlpcQueryInformation`上，不重要，继续F5，断在函数`NtAlpcSendWaitReceivePort`上，这次是Bind操作，我们第二章已经分析过，跳过，再次F5，再次断在函数`NtAlpcSendWaitReceivePort`上，这次是Request请求：

![](upload/attach/202304/560415_KAFFDUWCU3P9NMH.jpg)

第二章节已经讲过Request请求里面的的参数是如何编码的了，这里我就不一一讲解了，我只说第二章没有遇到的参数类型。如上图，`wcsExePath`和`wcsParam`两个字符串数组前边还有红色框标注的4个字节的数据。我们查看IDL文件对该函数的声明:
