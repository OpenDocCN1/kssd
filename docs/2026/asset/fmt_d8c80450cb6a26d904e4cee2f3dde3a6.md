
`cs`=0008 `ss`=0010 `ds`=0023 `es`=0023 `fs`=0030 `gs`=0000 `efl`=00000293

`nt!KeAcquireQueuedSpinLockAtDpcLevel`:

`80541a80` `8b5104` `mov edx,dword ptr [ecx+4]` `ds:0023:f9c2c53c={nt!KiDispatcherLock (80554880)}`

注意：`ECX+4`里面就是`KiDispatcherLock`线程分发自旋锁了。

这个`ECX`是哪里来的呢？找不到`KiDispatcherLock`的指针，说什么都没用，因为我们的理念是与系统共用一个锁来禁止在我们操作期间产生线程切换，所以锁的指针是必须的。

其实获取也很简单：

`804fe528` `64a120000000` `mov eax,dword ptr fs:[00000020h]`

`804fe52e` `8d8818040000` `lea ecx,[eax+418h]`

这里就是了！

但需要说明的是，这个锁的使用，并非简单的置位1或0，这里我模拟Windows写了两段上锁解锁代码，供大家使用，简单的模拟实现，没做什么优化，大家凑合着用吧：

### 多核同步锁

```asm
;=====================================================
;多核同步锁
;=====================================================
\_LockCPU        proc
         assume fs:nothing
         mov eax,dword ptr fs:[00000020h]
         add eax,420h
         mov ecx,eax
         mov edx,dword ptr [ecx+4]
         mov eax,ecx
         xchg eax,dword ptr [edx]
         cmp eax,0
         jne StartTest
         or edx,2
         mov dword ptr [ecx+4],edx
        ExitLock:
         ret
        StartTest:
         or edx,1
         mov dword ptr [ecx+4],edx
         mov dword ptr [eax],ecx
        LoopTest:
         test dword ptr [ecx+4],1
         je ExitLock
         db        0f3h,90h
         jmp LoopTest
\_LockCPU endp
;;------------------------------
\_UnLockCPU        proc
         assume fs:nothing
         mov eax,dword ptr fs:[00000020h]
         add eax,420h
         mov        ecx,eax
         mov edx,dword ptr [ecx]
         mov ecx,dword ptr [ecx+4]
         test edx,edx
         btr ecx,1
         mov dword ptr [eax+4],ecx
         jne UnExit
         xor edx,edx
         push eax
         lock cmpxchg dword ptr [ecx],edx
         pop eax
         jne UnLoopTest
         ret
        UnExit:
         xor dword ptr [edx+4],3
         mov dword ptr [eax],0
         ret
        UnLoopTest:
         mov edx,dword ptr [eax]
         test edx,edx
         jne UnExit
         db        0f3h,90h
         jmp UnLoopTest
\_UnLockCPU endp
```

我做了一个测试，`InlineHOOK`了调用相对频繁的`KeInsertQueueApc`，将`HOOK`与`UnHOOK`代码放到了一个定时器中，每秒种`HOOK`与`UnHOOK`一次，仅仅用上面两段代码加锁，没加任何其它措施，比如提升`IRQL`等，在多核处理器中跑了很长时间后，没发现问题。

当然了，这并不能说明就没有问题，谁用谁测试吧，有什么问题欢迎反馈。

`29dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2#2M7$3g2Z5k6i4u0G2i4K6u0W2j5X3I4G2k6#2)9J5k6i4c8A6j5h3&6&6j5g2)9J5k6h3y4F1i4@1f1K6i4K6R3H3i4K6R3J5`

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
