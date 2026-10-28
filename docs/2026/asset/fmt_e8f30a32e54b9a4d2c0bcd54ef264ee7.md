
```
{
    nop
    nop
    nop
    nop
    nop
    nop
    nop
    nop
    nop
    nop
    nop
    nop
    nop
    nop
    nop
    nop
    nop
    nop
    mov eax,NtCreateSection //将原函数地址送入eax
    add eax,7 //eax加7，用来跳过我们的HOOK
    push eax //将eax压入
    ret //跳回原函数
}

}

//用来生成跳转代码的函数
void makejmp(LPVOID Fun1,LPVOID Fun2,LPVOID jmp)
{
    BYTE *data=(BYTE *)jmp;
    long dFun1=(long)Fun1;
    long dFun2=(long)Fun2;
    DWORD H;
    data[0]=0xe9;
    _asm
    {
        mov eax,dFun1
        mov edx,dFun2
        sub edx,eax
        sub edx,5
        mov H,edx
    }
    memcpy(&data[1],(void *)&H,4);
}

VOID Unload(
    IN PDRIVER_OBJECT DriverObject
)
{
    KIRQL oldIrql;
    while(!NT_SUCCESS(ZwYieldExecution()))//取得CPU时间，如果取得失败，就一直获取，直到成功
    {
    }
    if(pMdl!=NULL)//检查我们是否已经进行过HOOK，//如果我们进行过HOOK 就还原
    {
        oldIrql = KeRaiseIrqlToDpcLevel();
        memcpy(WriteAddress,OldNtCreateSection,7);
        KeLowerIrql(oldIrql);
        IoFreeMdlForAddress(WriteAddress,pMdl);
    }
    IoDeleteDevice(DriverObject->DeviceObject);//删除设备
    pMdl=NULL;
    WriteAddress=NULL;
}

PVOID NTAPI GetJmpAddress(PVOID Fun,BOOL *Call_Code)
{
    PVOID Return=NULL;
    BYTE *data=(BYTE *)Fun;
    DWORD Old=0;
    if(data[0]==0xe9)
    {
        *Call_Code=0;
    }else if (data[0]==0xe8)
    {
        *Call_Code=1;
    }else
    {
        *Call_Code=2;
        return NULL;
    }
    memcpy((void *)&Old,&data[1],4);
    _asm
    {
        mov eax,Old
        mov edx,Fun
        add eax,edx
        add eax,5
        mov Return,eax
    }
    return Return;
}
```

这个驱动只支持单核心CPU，多核心CPU请自行修改（将所有CPU都提升到DPC级别以后再进行HOOK）。驱动加载流程是看了WIN2K的代码来讲的，WIN2K的代码很长，所以就简单的讲了一下

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`NtCreateSection.rar`](javascript:void(0);)（160.20kb，286次下载）
