
过滤引擎由很多层组成。每一层代表了网络协议栈的一个层。分层是一个容器，里面包含了0或多个过滤器，或一个或多个子层。每个分层都有一个唯一标识的UID，不同的层获取的数据不一样，这是Windows内核安全与驱动开发书中的解释

下面是我对层的理解

![img](https://bbs.kanxue.com/upload/attach/202102/838741_XTX28HK3RFKXBZJ.jpg)

在不同的层获取的数据不一样，这个不一样指的是首部，传输层比应用层多个tcp/udp首部，但本质上这个数据的用户数据部分没有改变，只是加了个首部，数据会经过每个层，我看书的时候以为获取不一样的数据是指不同的用户数据，看书的时候一直纠结，这些都能截获什么样的用户数据，看完了都没说，后搜了一下网络协议栈，才恍然大悟，其实用户数据会经过每个层，只是经过时会加上不同的头部，所以不同。所以不同的层就是有不同的首部。

其他的细节部分建议看书。

# 0x4开发日志

## 1打印过滤到的ip并进行拦截测试

|  |  |

| --- | --- |

| 1  2  3  4  5  6 | `/``/``ulRemoteIPAddress 表示远端IP`  `ulRemoteIPAddress` `=`  `inFixedValues``-``>`  `incomingValue[FWPS_FIELD_ALE_FLOW_ESTABLISHED_V4_IP_REMOTE_ADDRESS]`  `.value.uint32;`  `DbgPrint(``"remote ip is %x\n"``, ulRemoteIPAddress);` |

在过滤函数`Wfp_Sample_Established_ClassifyFn_V4`里对获取的ip进行打印

![](https://bbs.kanxue.com/upload/attach/202101/838741_8A4AJSNFZR3D9V8.png)

百度ip为112.80.248.76，获取的数为0x7050f84c,转化一下刚好对应，ip获取成功。

|  |  |

| --- | --- |

| 1  2  3  4 | `if` `(ulRemoteIPAddress` `=` `0x7050F84C``)`  `{`  `classifyOut``-``>actionType` `=` `FWP_ACTION_BLOCK;`  `}` |

先做个简单的拦截测试，`classifyOut->actionType = FWP_ACTION_BLOCK;`就是返回拦截的意思。

![](https://bbs.kanxue.com/upload/attach/202101/838741_CEUYNMTHS3VH7GD.png)

百度刷新一下一直是没反应，就用cmd，ping一下baidu，拦截成功

## 2应用层获取网站输入并进行ip地址转化发给驱动

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7 | `CString` `str``;`  `GetDlgItemText(IDC_EDIT_PORT,` `str``);`  `const char``*` `cstr;`  `char temp[``100``];`  `::wsprintfA(temp,` `"%ls"``, (LPCTSTR)``str``);`  `cstr` `=` `temp;`  `hostent` `*``host` `=` `gethostbyname(cstr);` |

实现定义变量`str`，从文本框中获取文本放入`str`，再把 `char*` 类型的`str`转为`char`类型，再通过字符串使用函数`gethostbyname`获取ip。

![](https://bbs.kanxue.com/upload/attach/202101/838741_8YKPZZG8VCHPJZC.png)

然后我本来想测试一下IP是否成功get到，就用messagebox，显示一下host。host为0，调用`WSAGetLastError`，看一下没有符号表，就用od调了一下，返回值为0000276D，看了一下错误信息，函数就通过字符串获取ip，我还以为那个函数可以单独使用。又加上`WSAStartup`相关代码。

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24 | `ULONG a[``10``]` `=` `{` `0` `};`  `int` `x` `=` `0``;`  `for` `(x;; x``+``+``)`  `{`  `a[x]` `=` `inet_addr(inet_ntoa(``*``((in_addr` `*``)host``-``>h_addr_list[``0``])));`  `if` `(host``-``>h_addr_list[x]` `+` `host``-``>h_length >``=` `host``-``>h_name)`  `{`  `break``;`  `}`  `}`  `HANDLE hFile` `=` `CreateFile(_T(``"\\\\.\\wfp_sample_device"``),GENERIC_ALL,``0``,NULL,OPEN_EXISTING,``0``,NULL);`  `if``( hFile` `=``=` `INVALID_HANDLE_VALUE )`  `{`  `return``;`  `}`  `ST_WFP_NETINFO Info` `=` `{``0``};`  `/``/` `Info.m_uRemotePort` `=` `uPort;`  `for` `(x; x >``=` `0``; x``-``-``)`  `{`  `Info.m_ulRemoteIPAddr` `=` `a[x];`  `DWORD dwNeedSize` `=` `0``;`  `BOOL` `rValue` `=` `DeviceIoControl(hFile, IOCTL_WFP_SAMPLE_ADD_RULE, (LPVOID)&Info, sizeof(Info), NULL,` `0``, &dwNeedSize, NULL);`    `}` |

接下来用个循环把通过函数`inet_addr`把字符串ip转成`ULONG`的值存入数组`ULONG a[10]`，再打开驱动通过`DeviceIoControl`发送数据，for循环把数组每个值都发过去。

## 3驱动获取输入存入链表与过滤函数读取链表判断拦截

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38 | `BOOLEAN AddNetRuleInfo(PVOID pBuf,ULONG uLen)`  `{`  `BOOLEAN bSucc` `=` `FALSE;`  `PST_WFP_NETINFO    pRuleInfo` `=` `NULL;`  `do`  `{`  `PST_WFP_NETINFOLIST pRuleNode` `=` `NULL;`  `KIRQL    OldIRQL` `=` `0``;`  `pRuleInfo` `=` `(PST_WFP_NETINFO)pBuf;`  `if``( pRuleInfo` `=``=` `NULL )`  `{`  `break``;`  `}`  `if``( uLen < sizeof(ST_WFP_NETINFO) )`  `{`  `break``;`  `}`  `pRuleNode` `=` `(PST_WFP_NETINFOLIST)ExAllocatePoolWithTag(NonPagedPool,sizeof(ST_WFP_NETINFOLIST),WFP_TAG);`  `if``( pRuleNode` `=``=` `NULL )`  `{`  `break``;`  `}`  `memset(pRuleNode,``0``,sizeof(ST_WFP_NETINFOLIST));`  `pRuleNode``-``>m_stWfpNetInfo.m_uSrcPort` `=` `pRuleInfo``-``>m_uSrcPort;`  `pRuleNode``-``>m_stWfpNetInfo.m_uRemotePort` `=` `pRuleInfo``-``>m_uRemotePort;`  `pRuleNode``-``>m_stWfpNetInfo.m_ulSrcIPAddr` `=` `pRuleInfo``-``>m_ulSrcIPAddr;`  `pRuleNode``-``>m_stWfpNetInfo.m_ulRemoteIPAddr` `=` `pRuleInfo``-``>m_ulRemoteIPAddr;`  `pRuleNode``-``>m_stWfpNetInfo.m_ulNetWorkType` `=` `pRuleInfo``-``>m_ulNetWorkType;`  `pRuleNode``-``>m_stWfpNetInfo.m_uDirection` `=` `pRuleInfo``-``>m_uDirection;`  `KeAcquireSpinLock(&g_RuleLock,&OldIRQL);`  `InsertHeadList(&g_WfpRuleList,&pRuleNode``-``>m_linkPointer);`  `KeReleaseSpinLock(&g_RuleLock,OldIRQL);`  `bSucc` `=` `TRUE;`  `break``;`  `}`  `while` `(FALSE);`  `return` `bSucc;`  `}` |

应用层通过`DeviceIoControl`发送数据，驱动通过设置`DeviceIoControl`的irp函数进行操作，在`WfpSampleIRPDispatch`函数里调用`AddNetRuleInfo`函数将数据存入全局变量的链表。

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32 | `BOOLEAN IsHitRule(USHORT ulRemoteIPAddress)`  `{`  `BOOLEAN bIsHit` `=` `FALSE;`  `do`  `{`    `KIRQL    OldIRQL` `=` `0``;`  `PLIST_ENTRY    pEntry` `=` `NULL;`  `if``( g_WfpRuleList.Blink` `=``=` `NULL ||`  `g_WfpRuleList.Flink` `=``=` `NULL )`  `{`  `break``;`  `}`    `KeAcquireSpinLock(&g_RuleLock,&OldIRQL);`  `pEntry` `=` `g_WfpRuleList.Flink;`  `while``( pEntry !``=` `&g_WfpRuleList )`  `{`  `PST_WFP_NETINFOLIST pInfo` `=` `CONTAINING_RECORD(pEntry,ST_WFP_NETINFOLIST,m_linkPointer);`  `if` `(ulRemoteIPAddress` `=``=` `pInfo``-``>m_stWfpNetInfo.m_ulRemoteIPAddr)`  `{`  `bIsHit` `=` `TRUE;`  `break``;`  `}`  `pEntry` `=` `pEntry``-``>Flink;`  `}`  `KeReleaseSpinLock(&g_RuleLock,OldIRQL);`  `}`  `while` `(FALSE);`  `return` `bIsHit;`    `}` |

过滤函数`Wfp_Sample_Established_ClassifyFn_V4`里调用`IsHitRule`函数，传入参数`ulRemoteIPAddress`为当前包的ip，通过读取链表中从应用层获取的ip进行一一比对，有目标ip就把`bIsHit = TRUE;`，返回它，在外面判断`bIsHit`的值，是true就拦截。

# 0x5调试日志

***测试时发现没有拦截成功***

## 应用层调试

![](https://bbs.kanxue.com/upload/attach/202101/838741_8CY3WMVES9NPSHN.png)

对`wsprintfA`下断点（其他的函数也一样），断下后返回用户代码，往下找`inet_addr`函数（ip转换函数）

![](https://bbs.kanxue.com/upload/attach/202101/838741_6M25TR4QKS6C83F.png)

断下后第一次od里是112.80.248.75 而ping 的是112.80.248.76，我又重试了一次又是112.80.248.75 。

![](https://bbs.kanxue.com/upload/attach/202101/838741_KKEN2X4JHHEGSYF.png)

然后我又全部重试了一次，这一次od里是112.80.248.76，而ping 的是112.80.248.75，我又ping一下又是112.80.248.76，猜测可能百度有多个ip（个人猜测，有错误欢迎大佬指出）。

应用层ip获取成功了，接下来看数据是否发送成功。

![](https://bbs.kanxue.com/upload/attach/202101/838741_97TDKF89XTEP9GH.png)

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17 | `for` `(x; x >``=` `0``; x``-``-``)`  `{`  `ST_WFP_NETINFO Info` `=` `{` `0` `};`  `Info.m_ulRemoteIPAddr` `=` `a[x];`  `DWORD dwNeedSize` `=` `0``;`  `BOOL` `rValue` `=` `DeviceIoControl(hFile, IOCTL_WFP_SAMPLE_ADD_RULE, (LPVOID)&Info, sizeof(Info), NULL,` `0``, &dwNeedSize, NULL);`  `if` `(rValue` `=``=` `0``)`  `{`  `MessageBox(L``"failed"``);`  `}`  `else`  `{`  `CString temp_value` `=` `_T("");`  `temp_value.``Format``(_T(``"%x"``), a[x]);`  `MessageBox(temp_value);`  `}`  `}` |

`a[x]`是存放ip的`ULONG`数组，`MessageBox(L"failed");`是失败，`MessageBox`一个数值就是发送成功，数值为ip的值，4cf85070可以和ip对上，那就是没问题题，应用层的排查到此为止。

## 驱动层调试

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17 | `while``( pEntry !``=` `&g_WfpRuleList )`  `{`  `PST_WFP_NETINFOLIST pInfo` `=` `CONTAINING_RECORD(pEntry,ST_WFP_NETINFOLIST,m_linkPointer);`  `DbgPrint(``"111is --%x-\n"``, pInfo``-``>m_stWfpNetInfo.m_ulRemoteIPAddr);`  `char` `*``ip1` `=` `ulRemoteIPAddress;`  `char` `*``ip2` `=` `pInfo``-``>m_stWfpNetInfo.m_ulRemoteIPAddr;`    `DbgBreakPoint();`    `if` `(ulRemoteIPAddress` `=``=` `pInfo``-``>m_stWfpNetInfo.m_ulRemoteIPAddr)`  `{`    `bIsHit` `=` `TRUE;`  `break``;`  `}`  `pEntry` `=` `pEntry``-``>Flink;`  `}``602C9824` |

在判断前加个`DbgBreakPoint()`断点，既可以看目标ip是否拦截到，也可以看应用层发的数据是否存入链表。

![](https://bbs.kanxue.com/upload/attach/202101/838741_ZQ4K8T9CM7XFT93.png)

从windbg中可以很明显的看出来了，`ulRemoteIPAddress`是0x24982c60

而`pInfo->m_stWfpNetInfo.m_ulRemoteIPAddr`是 0x602c9824，两个数刚好是反过来的，

总结一下：应用层的数据类型变来变去的过程里内存的存放翻转了一次，导致驱动进行比对时不相同，从而未能拦截。

# 0x6使用展示

![](https://bbs.kanxue.com/upload/attach/202101/838741_TMMPHYD7QATNGZS.png)

在网站框内输入想拦截的网址即可，第一次为未输入网站ping 百度成功，第二次为输入网站后ping百度，拦截成功，网址可输入多个。
