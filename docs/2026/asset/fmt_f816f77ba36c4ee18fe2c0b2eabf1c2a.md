
到这里，该外挂主程序的功能就分析完成了，点退出后，该外挂会删除掉该程序创建的内核服务。

从目前的行为上来说，该程序应该是快外挂而不是木马。那么接下来我们来分析下`G-a-m-e-D-K.dat`，看看它被加载后主要都做了些什么。

我们把它载入`IDA`，前面的`DriverEntry`并没做什么，都是些正常流程，如创建设备对象，和设备名称等，这些就不看了，我们直接来到关键函数处：

```
.text:000105C8 sub_105C8       proc near               ; CODE XREF: sub_106F8:loc_10779 p

.text:000105C8                 mov     eax, ds:KeServiceDescriptorTable ;取得SSDT表的地址

.text:000105CD                 push    esi

.text:000105CE                 mov     esi, [eax]      ; 把SSDT表中的函数地址给esi

.text:000105D0                 add     esi, 0ACh       ; 找到序号为0xAC的函数

.text:000105D6                 mov     eax, [esi]      ; 取得该函数地址并放到eax中

.text:000105D8                 mov     dword_1090C, eax ; 把函数地址保存到变量中

.text:000105DD                 call    sub_104EA       ; 该函数是通过 Cr0寄存器修改内存读写属性

.text:000105E2                 call    ds:KeRaiseIrqlToDpcLevel ; 提升Irql等级

.text:000105E8                 mov     cl, al          ; NewIrql

.text:000105EA                 mov     dword ptr [esi], offset sub_10574 ; Hook SSDT,把自己的函数替换SSDT表中的函数

.text:000105F0                 call    ds:KfLowerIrql

.text:000105F6                 sti                     ; 通过Cr0寄存器恢复内存读写属性

.text:000105F7                 push    eax

.text:000105F8                 mov     eax, dword_10908

.text:000105FD                 mov     cr0, eax

.text:00010600                 pop     eax

.text:00010601                 xor     eax, eax

.text:00010603                 inc     eax

.text:00010604                 pop     esi

.text:00010605                 retn

.text:00010605 sub_105C8       endp
```

通过对以上代码的分析，可以得出，该外挂`Hook`了`SSDT`，至于`Hook`了那个内核函数，由于操作系统的版本不同，`SSDT`表中内核函数的序号也不同，所以我找了个干净的虚拟机，启动外挂，然后用工具查看，原来该外挂是`hook`了`NtCreateMutant`。

至于该外挂用于替换`NtCreaeteMutant`的函数做了什么，其实也很简单，我这就不分析了，有兴趣的朋友可以自己看看。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [桌面.rar](javascript:void(0);)

  （16.13kb，194次下载）
