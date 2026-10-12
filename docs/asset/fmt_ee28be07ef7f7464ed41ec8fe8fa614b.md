
```c
//--------------------------------------------------------------------------
// 进程信息

PId = PsGetCurrentProcessId();

ProcessName = GetCurrentProcessFileName(); // 获得当前进程完整路径

RtlInitUnicodeString(&uProcessPath, ProcessName);
RtlUnicodeStringToAnsiString(&aProcessPath, &uProcessPath, TRUE);
strcpy(aProcessName, aProcessPath.Buffer);

if (!(RtlCompareUnicodeString(pustr, uRealName, TRUE)))
{
    {
        ptr = (ULONG *)(RequestMessage->MessageData);

        for (i = 0; i < RequestMessage->ActualMessageLength / sizeof(ULONG); i++)
        {
            DbgPrint("%x ", ptr[i]); // 输出数据
        }
        // if(ptr[0]==0x01&&ptr[1]==0x1f0241) // 有点问题

        if (ptr[1] == 0x1f0241)
        {
            DbgPrint("进程%s 开启服务", aProcessName); // 输出数据
            strcpy(LastCalled_Path, aProcessName);
            LastCalled_Pid = PId;
        }
    }
}

return RealZwRequestWaitReplyPort(PortHandle, RequestMessage, ReplyMessage);

}

else
    return RealZwRequestWaitReplyPort(PortHandle, RequestMessage, ReplyMessage);
}
```

结果：

![](img/80c776e429c84ff0dad397bb87a45bbb.jpg)

![](img/95edc59fed54a51ae12dd9d7a48184c6.jpg)
