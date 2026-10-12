
# Inside of ServiceMessageBox

> 原文：[`www.pediy.com/kssd/pediy10/91633.html`](https://www.pediy.com/kssd/pediy10/91633.html)

`MB_SERVICE_NOTIFICATION` 是 `MessageBox` 函数提供的一个 Type 值，该值允许通过 `CSRSS.exe` 而非进程本身来弹出 `MessageBox`。

当调用 `MessageBox` 时，若 `nType` 参数中包含 `MB_SERVICE_NOTIFICATION` 或 `MB_DEFAULT_DESKTOP_ONLY` 标志，`MessageBox` 将调用 `use32.dll` 内部的一个函数：`ServiceMessageBox`，代码如下：

```
MessageBoxWorker(...)

{

....省略无关代码...

    if (dwStyle & (MB_DEFAULT_DESKTOP_ONLY | MB_SERVICE_NOTIFICATION)) {
        if (pMsgBoxParams->hwndOwner != NULL) {
              SetLastError(ERROR_INVALID_PARAMETER);
             return 0;
        }

        return ServiceMessageBox(pMsgBoxParams->lpszText,
                                 pMsgBoxParams->lpszCaption,
                                 dwStyle & ~MB_SERVICE_NOTIFICATION);
    }
    }
```

`ServiceMessageBox` 根据当前的会话状况不同，调用不同的函数来实现弹框。

在 XP 中，首先会判断当前是否为终端服务器版本（通过 `UserSharedData->SuiteMasks` 是否包含 `VER_SUITE_TERMINAL`）。如果不是服务器版本，则直接调用 `NtRaiseHardError` 来通知 CSRSS。VISTA 则不会走这一步，因为 VISTA 默认会有 1 个以上的会话在运行。

接着，`ServiceMessageBox` 会通过 `NtOpenThreadToken` 函数试图打开当前线程的 Token，此函数仅在当前线程进行了 Token 模拟（Impersonation）时才会返回成功。若此函数返回了被模拟的线程 Token，则通过 `NtQueryInformationToken(TokenSessionId)` 取得此 TOKEN 所在的会话 ID。

随后，会通过 `kernel32!ProcessIdToSessionId` 调用 `NtQueryInformationProcess(ProcessSessionInformation)` 获得当前进程的 `SessionId`（若失败，则从 `NtCurretnTeb()->Peb->SessionId` 获取）。

最后，比较模拟的 Token 的 `SessionId` 是否等于当前进程的 `SessionId`。若不等于，说明当前进程和被模拟的 TOKEN 不属于同一会话，此时 `ServiceMessageBox` 会通过调用 `winsta!WinStationSendMessageW` 来实现弹框。`WinStationSendMessageW` 函数调用 `CSmartSession::ShowMessageBox` 找到当前活动会话中的 CSRSS，并调用 `RpcShowMessageBox` 来通知当前活动会话中的 `csrss` 弹出通知对话框。

若会话相同，则仍旧调用 `NtRaiseHardError` 来通知默认 `ExceptionPort` 的 `csrss`。

通常，系统服务在弹出对话框前都会模拟当前活动会话中的 TOKEN，这样调用 `MessageBox` 弹框时就可以在当前用户上显示出来。

在 VISTA 之前，由于默认会话和服务是在同一个会话中，所以我们一般感受到的服务弹框都是调用 `NtRaiseHardError` 实现的，只有在切换到其他会话时，才会出现调用 `RpcShowMessageBox` 的情况。

但 VISTA 之后，由于服务进程单独享有一个会话（0 号会话），因此所有来自服务的弹框，都会通过 `RpcShowMessageBox` 来显示出来。

至于 CSRSS 如何处理 `MESSAGEBOX` 的 RPC（如 `HandleHandError` 等），早已是众所周知，在此不再赘述，感兴趣的可以参考 WIN2K 源代码。
