
```
{
    if (NULL != DeviceObject)
    {
        IoDeleteDevice(DeviceObject);
    }

    return status;
}

RtlZeroMemory(Buffe_0_1_2, 0, 256);
GetKernelBase();
HookCode = ExAllocatePoolWithTag(NonPagedPool, 100, 0x123456);
RtlCopyMemory(HookCode, (ULONG)InlineHook + 8, 0x62);
return STATUS_SUCCESS;
}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [Demo.rar](javascript:void(0);)
  （3.29kb，218次下载）
