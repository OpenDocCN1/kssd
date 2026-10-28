
![](https://bbs.kanxue.com/upload/attach/201509/694261_6orvy2u4fj2xrph.jpg)

## 框架流程

简单来讲流程如下：在驱动初始化时会通过解析 PE 来获取两份纯净的服务函数表（`SSDT` 和 `ShadowSSDT`）。过滤函数表没有任何数据结构，只是一个函数指针数组，与服务号对应：如果该服务号有过滤函数，则数组元素非空；如果没有过滤函数，则数组元素值填 `0`。派发例程通过判断该值是否为 `0` 来判断有无过滤函数。如果有过滤函数则调用过滤函数，如果没有过滤函数则调用纯净的服务函数。如下图：

```text
系统调用 --> kifastcallentry --> 2345dispatch --> 判断服务表是 SSDT 还是 ShadowSSDT --> 根据服务号和对应表找到相应的过滤函数以及对应的纯净 NT 函数 --> 进行派发调用：如果过滤函数非空则调用过滤函数，否则调用纯净 Nt 服务
```

至此，“框架”就到此结束。

### 单独处理的 6 个调用

其中该驱动对以下 6 个调用单独处理，但是代码逻辑还是跟框架一样，所以应该是后期做优化没去掉该处代码：

- `84h` `NtPlugPlayControl`
- `3f` `NewNtDeleteKey`
- `41` `NewNtDeleteValueKey`
- `f7` `NewNtSetValueKey`
- `c0` `NewNtRenameKey`
- `b1` `NewNtQueryValueKey`

## 为何是有毒的 hook“框架”

细看一下派发流程：

1. 不管过不过滤，最终都会调用纯净服务函数，穿透 `SSDT` hook。
2. 如果其他软件是通过 `SSDT` hook 返回地址来找挂钩点的话，装了 2345 直接导致找不到挂钩点，因为调用不会走到代理函数里面去（可以通过 `rdsmr` 或者 `idt 0x2e` 去寻找）。
3. 接管 NT 派发调用，即真正的调用服务函数不会在 NT 的原始派发代码里，而是 `2345port.sys` 写死在自己的代码里（即跳过 NT 本身的 `call ebx`，在自己的代码里 `call ebx`（高版本系统是 `call edx`））。而 360、Q管、金山、百度等大致钩子位置相近或者一致，但都是在系统原有派发前面挂钩（即 `call ebx` 之前挂钩）。他们的挂钩属于寄存器 hook，即修改 `ebx`（服务函数）的值，最终让 NT 服务派发器去调用。恰巧 2345 在接管派发流程时，不会重走 NT 的服务派发流程（这里的派发是指调用服务函数，并不是全部的派发代码），直接在自己的代码里派发。这样，就造成除 2345 之外的所有杀软需要钩子的功能全部失效。

所以这套服务派发逻辑，毒就毒在没有考虑兼容性，比如多杀软共存（感觉像是故意的~）。

## 以 XP 为例的过滤函数表

其过滤函数表如下：

### SSDT

| 服务号 | 服务名称 |
| ---: | --- |
| 2 | `fake_NtAccessCheckAndAuditAlarm` |
| 11 | `fake_NtAdjustPrivilegesToken` |
| 17 | `fake_NtAllocateVirtualMemory` |
| 19 | `fake_NewNtAssignProcessToJobObject` |
| 25 | `fake_NtClose` |
| 31 | `fake_NtConnectPort` |
| 37 | `fake_NtCreateFile` |
| 41 | `fake_NewNtCreateKey` |
| 43 | `fake_NtCreateMutant` |
| 45 | `fake_NtCreatePagingFile` |
| 48 | `fake_NtCreateProcessEx` |
| 50 | `fake_NtCreateSection` |
| 52 | `fake_NtCreateSymbolicLinkObject` |
| 53 | `fake_NtCreateThread` |
| 57 | `fake_NtDebugActiveProcess` |
| 62 | `fake_NtDeleteFile` |
| 63 | `fake_NtDeleteKey` |
| 65 | `fake_NtDeleteValueKey` |
| 66 | `fake_NtDeviceIoControlFile` |
| 67 | `fake_ZwDisplayString` |
| 68 | `fake_NtDuplicateObject` |
| 71 | `fake_NtEnumerateKey` |
| 73 | `fake_NtEnumerateValueKey` |
| 83 | `fake_ZwFreeVirtualMemory` |
| 84 | `fake_NtFsControlFile` |
| 93 | `fake_NtInitiatePowerAction` |
| 97 | `fake_NtLoadDriver` |
| 105 | `fake_NtMakeTemporaryObject` |
| 116 | `fake_NtOpenFile` |
| 119 | `fake_NtOpenKey` |
| 122 | `fake_NtOpenProcess` |
| 125 | `fake_NtOpenSection` |
| 127 | `fake_NtOpenSymbolicLinkObject` |
| 128 | `fake_NtOpenThread` |
| 132 | `fake_NtPlugPlayControl` |
| 137 | `fake_NtProtectVirtualMemory` |
| 160 | `fake_NtQueryKey` |
| 177 | `fake_NtQueryValueKey` |
| 178 | `fake_NtQueryVirtualMemory` |
| 180 | `fake_NtQueueApcThread` |
| 182 | `fake_NewNtRaiseHardError` |
| 192 | `fake_NtRenameKey` |
| 193 | `fake_NtReplaceKey` |
| 200 | `fake_NtRequestWaitReplyPort` |
| 204 | `fake_NtRestoreKey` |
| 210 | `fake_NtSecureConnectPort` |
| 213 | `fake_NtSetContextThread` |
| 224 | `fake_NtSetInformationFile` |
| 228 | `fake_NtSetInformationProcess` |
| 237 | `fake_NtSetSecurityObject` |
| 240 | `fake_NtSetSystemInformation` |
| 241 | `fake_NtSetSystemPowerState` |
| 242 | `fake_NtSetSystemTime` |
| 244 | `fake_NtSetTimer` |
| 247 | `fake_NtSetValueKey` |
| 249 | `fake_NtShutdownSystem` |
| 253 | `fake_NtSuspendProcess` |
| 254 | `fake_NtSuspendThread` |
| 255 | `fake_NtSystemDebugControl` |
| 256 | `fake_NtTerminateJobObject` |
| 257 | `fake_NtTerminateProcess` |
| 258 | `fake_NtTerminateThread` |
| 259 | `fake_NtTestAlert` |
| 262 | `fake_NtUnloadDriver` |
| 267 | `fake_NtUnmapViewOfSection` |
| 274 | `fake_NtWriteFile` |
| 277 | `fake_NtWriteVirtualMemory` |

### ShadowSSDT

| 服务号 | 服务名称 |
| ---: | --- |
| 310 | `fake_NtUserBlockInput` |
| 319 | `fake_NtUserCallHwndParamLock` |
| 330 | `fake_NtUserClipCursor` |
| 355 | `fake_NtUserDestroyWindow` |
| 383 | `fake_NtUserGetAsyncKeyState` |
| 414 | `fake_NtUserGetKeyboardState` |
| 416 | `fake_NtUserGetKeyState` |
| 427 | `fake_NtUserGetRawInputBuffer` |
| 428 | `fake_NtUserGetRawInputData` |
| 460 | `fake_NtUserMessageCall` |
| 465 | `fake_NtUserMoveWindow` |
| 475 | `fake_NtUserPostMessage` |
| 476 | `fake_NtUserPostThreadMessage` |
| 502 | `fake_NtUserSendInput` |
| 517 | `fake_NtUserSetImeInfoEx` |
| 529 | `fake_NtUserSetParent` |
| 544 | `fake_NtUserSetWindowLong` |
| 545 | `fake_NtUserSetWindowPlacement` |
| 546 | `fake_NtUserSetWindowPos` |
| 547 | `fake_NtUserSetWindowRgn` |
| 549 | `fake_NtUserSetWindowsHookEx` |
| 552 | `fake_NtUserSetWinEventHook` |
| 555 | `fake_NtUserShowWindow` |
| 556 | `fake_NtUserShowWindowAsync` |
| 559 | `fake_NtUserSystemParametersInfo` |
| 581 | `fake_NtUserSetLayeredWindowAttributes` |

写的比较匆忙，如有问题欢迎指正~

图如果看不清，请下载文档查看，或者点击图片放大。

> [2345hook分析.doc](https://bbs.kanxue.com/attach-download-100218.htm)

驱动文件

> [2345driver.rar](https://bbs.kanxue.com/attach-download-100221.htm)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [1.jpg](javascript:void(0);)（71.38kb，65次下载）
- [2.jpg](javascript:void(0);)（61.16kb，19次下载）
- [3.jpg](javascript:void(0);)（55.30kb，19次下载）
- [4.jpg](javascript:void(0);)（61.30kb，5次下载）
- [5.jpg](javascript:void(0);)（45.63kb，17次下载）
- [6.jpg](javascript:void(0);)（69.71kb，13次下载）
- [2345hook分析.doc](javascript:void(0);)（2.77MB，176次下载）
- [2345driver.rar](javascript:void(0);)（67.36kb，87次下载）
