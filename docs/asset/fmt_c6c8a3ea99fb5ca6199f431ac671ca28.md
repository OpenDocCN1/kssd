
# 彻底改掉进程名

> 原文：[`www.pediy.com/kssd/pediy10/96427.html`](https://www.pediy.com/kssd/pediy10/96427.html)

写了个改进程名的东西，跟大家分享！技术含量不高，大牛飘过。

先总结一下，一个进程的名字有可能从以下部位获取（参考小伟同学的《伪造进程初探》一文）：

一、`EPROCESS` 中：

1. `EPROCESS-->ImageFileName`（很常用，冰刃获取进程名的地方）
2. `EPROCESS-->SeAuditProcessCreationInfo->ImageFileName`（任务管理器获取进程名的地方，`NtQueryInformationProcess` 就是从这里获取进程名的）
3. `EPROCESS->SectionObject->Segment->ControlArea->FileObject->FileName`（`RKU` 获取进程名的方法）
4. `VAD`（记录用户空间内存分配情况的数据结构，里面当然有进程的 `exe` 模块）

二、`PEB` 中：

1. `PEB-->ProcessParameters-->ImagePathName`
2. `PEB-->ProcessParameters-->CommandLine`
3. `PEB-->ProcessParameters-->WindowTitle`（这个地方比较奇怪，如果双击的是 `exe` 的快捷方式，则记录的是快捷方式的路径，还是一并改掉的好）
4. `PEB-->LDR-->InLoadOrderModuleList->第一个结构->FullDllName`
5. `PEB-->LDR-->InLoadOrderModuleList->第一个结构->BaseDllName`
6. `PEB-->LDR-->InMemoryOrderModuleList->第一个结构->FullDllName`（此处的 `BaseDllName` 貌似为 `NULL`，就不管它了）
   - `PEB-->LDR-->InInitializationOrderModuleList` 这个表里貌似没有 `exe` 模块，也不管它了

把这些地方都改掉即可彻底改掉进程名（如果不够彻底，谢谢补充！）。

示例代码如下（示例代码中以 `winmine.exe` 做测试，平台为 `XP SP3`。代码中硬编码很多，通用性不强。）：

代码:

```c
//Fypher 
//http://hi.baidu.com/nmn714

VOID ChangeName(ULONG pProcess){
    ULONG peb,ProcessParameters,ldr;
    ULONG InLoadOrderModuleList;
    ULONG InMemoryOrderModuleList;
    ULONG tmp;

    KAPC_STATE kapc;
    PUCHAR str;
    PWCHAR wstr;
    
    //get PEB
    peb=*(PULONG)(pProcess + 0x1b0);
    
    KeStackAttachProcess((PEPROCESS)pProcess,&kapc);
    __try{
        ProcessParameters = *(PULONG)(peb + 0x010);

        //ImagePathName
        FindAndChangeUni(ProcessParameters+0x038);

        //CommandLine
        FindAndChangeUni(ProcessParameters+0x040);

        //WindowTitle
        FindAndChangeUni(ProcessParameters+0x070);

        //Ldr
        ldr = *(PULONG)(peb + 0x00c);

        //InLoadOrderModuleList->FullDllName
        InLoadOrderModuleList = *(PULONG)(ldr+0x00c);
        FindAndChangeUni(InLoadOrderModuleList+0x024);

        //InLoadOrderModuleList->BaseDllName
        FindAndChangeUni(InLoadOrderModuleList+0x02c);

        //InMemoryOrderModuleList->FullDllName
        InMemoryOrderModuleList = *(PULONG)(ldr+0x014);
        FindAndChangeUni(InMemoryOrderModuleList+0x024);
    }__except(1){
        KdPrint(("exception occured!"));
    }
    KeUnstackDetachProcess (&kapc);

    //EPROCESS-->ImageFileName
    FindAndChangeA(pProcess+0x174,16);

    //EPROCESS-->SeAuditProcessCreationInfo->ImageFileName
    FindAndChangeUni(*(PULONG)(pProcess + 0x1F4));

    //EPROCESS->SectionObject->Segment->ControlArea->FileObject->FileName
    //should use MmIsAddressValid to verify
    tmp=*(PULONG)(pProcess+0x138);
    tmp=*(PULONG)(tmp+0x14);
    tmp=*(PULONG)tmp;
    tmp=*(PULONG)(tmp+0x024);
    FindAndChangeUni(tmp+0x030);
    
    //VAD
    //should use MmIsAddressValid to verify
    tmp=*(PULONG)(pProcess+0x11c);
    tmp=*(PULONG)(tmp+0x10);
    tmp=*(PULONG)(tmp+0x018);
    tmp=*(PULONG)(tmp+0x024);
    FindAndChangeUni(tmp+0x030);
}
```

其中，`FindAndChangeUni` 和 `FindAndChangeA` 的作用是在一个字符串（`UNICODE_STRING` 或 `CHAR`）中定位“`winmine.exe`”并改成"winxxoo.exe"。代码如下:

代码:

```c
//Fypher 
//http://hi.baidu.com/nmn714

VOID FindAndChangeUni(ULONG strAddr){
    PUNICODE_STRING uniStr = (PUNICODE_STRING)strAddr;
    ULONG len = uniStr->Length / 2;
    ULONG maxLen = uniStr->MaximumLength / 2;
    PWCHAR str = uniStr->Buffer;
    ULONG i=0;

    if(!str || len<11|| maxLen<11 )
        return;

    for(i=0;i<= len - 11;++i){
        if(!_wcsnicmp(str+i,L"winmine.exe",11))
            break;
    }

    if(i>len - 11)
        return;
    
    _asm{
        cli
        mov eax, cr0 
        and eax, not 0x10000 
        mov cr0, eax
    }
    //str 可能是 PEB 中的，故 try 之
    __try{
        str[i+3]=L'x';
        str[i+4]=L'x';
        str[i+5]=L'o';
        str[i+6]=L'o';
    }__except(1){
    }     
    _asm{
        mov eax, cr0
        or eax,0x10000
        mov cr0,eax
        sti
    }
}

VOID FindAndChangeA(ULONG strAddr,ULONG len){
    PUCHAR str = (PUCHAR)strAddr;
    ULONG i=0;

    if(!str || len<11 )
        return;

    for(i=0;i<= len - 11;++i){
        if(!_strnicmp(str+i,"winmine.exe",11))
            break;
    }

    if(i>len - 11)
        return;
    
    _asm{
        cli
        mov eax, cr0 
        and eax, not 0x10000 
        mov cr0, eax
    }
    //str 可能是 PEB 中的，故 try 之
    __try{
        str[i+3]='x';
        str[i+4]='x';
        str[i+5]='o';
        str[i+6]='o';
    }__except(1){
    }     
    
    _asm{
        mov eax, cr0
        or eax,0x10000
        mov cr0,eax
        sti
    }
}
```

截图效果：

![](img/bb77cc9db180f3ebcfd006d2900fb548.jpg)

参考：《伪造进程初探》小伟同学（膜拜一下）
