
```text
                        push ebp
                        mov ebp,esp
                        mov eax,g_KiInsertQueueApc
                        add eax,5
                        jmp eax
        }
}

//处理函数
//apc--kthread--apc_state--eprocess

VOID FASTCALL DetourMyKiInsertQueueApc(IN PKAPC Apc,IN KPRIORITY Increment)
{
        ULONG thread;
        ULONG process;

        /*OriginalKiInsertQueueApc(Apc,Increment);*/
        if(MmIsAddressValid((PULONG)((ULONG)Apc+0x008)))    //地址验证 KAPC结构+008--->kthread
                thread=*((PULONG)((ULONG)Apc+0x008));
        else
                return ;

        if(MmIsAddressValid((PULONG)((ULONG)thread+0x044))) //kthread+30-->KAPC_STATE+10-->eprocess
                process=*((PULONG)((ULONG)thread+0x044));
        else
                return ;

        if(MmIsAddressValid((PULONG)((ULONG)process+0x174)))  //eprocess+174---->进程名字
        {
                if((_stricmp((char *)((ULONG)process+0x174),"notepad.exe")==0)&&(Increment==2))
                {
                        return ;
                }
                else
                        OriginalKiInsertQueueApc(Apc,Increment);
        }
        else
                return;
}

//卸载函数
VOID UnHookKiInsertQueueApc()
{
        KIRQL Irql;
        WPOFF();
        Irql = KeRaiseIrqlToDpcLevel();

        //inline hook函数
        RtlCopyMemory ( (unsigned char*)g_KiInsertQueueApc, OriginalBytes, 5);
        // 恢复写保护，降低IRQL
        KeLowerIrql(Irql);
        WPON();
}

VOID OnUnload( IN PDRIVER_OBJECT DriverObject )
{
        DbgPrint("My Driver Unloaded!");
        UnHookKiInsertQueueApc();
}

NTSTATUS DriverEntry( IN PDRIVER_OBJECT theDriverObject, IN PUNICODE_STRING theRegistryPath )
{
        DbgPrint("My Driver Loaded!");
        // TODO!!
        theDriverObject->DriverUnload = OnUnload;
        HookKiInsertQueueApc();
        return STATUS_SUCCESS;
}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [1.JPG](javascript:void(0);)（18.88kb，252次下载）
- [2.JPG](javascript:void(0);)（20.91kb，251次下载）
