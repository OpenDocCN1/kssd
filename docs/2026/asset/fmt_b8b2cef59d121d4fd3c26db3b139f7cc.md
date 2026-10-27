
`ntdll!ZwReadFile`:

```
776262dc b811010000      mov     eax,111h//系统调用号

776262e1 ba0003fe7f      mov     edx,offset SharedUserData!SystemCallStub (7ffe0300)//这个7ffe03000地址固定的 系统初始化 查看是否支持快速系统调用 支持 这个地址里面存的就是KiFastSystemCall 否则就是KiInitSystemCall(int 0x2e模拟中断)的地址 这些函数都存在于用户空间ntdll.dll(和内核ntdll不一样)中 这个dll 对于每个进程 地址不变 一直在内存中

7ffe03000 和0x7fe0000 一个是r3的地址 一个是内核的地址 两个64kb地址映射到同一个物理内存

776262e6 ff12            call    dword ptr [edx]

776262e8 c22400          ret     24h

776262eb 90              nop
```

`ntdll!KiFastSystemCall`:

```
776270d0 8bd4            mov     edx,esp//进入KiFastSystemCall之前 已经被调用方压入了各种参数 最后压入的是返回地址 当前esp指向它 在书中 被压入堆栈的参数区域叫参数块  处理完后 调用sysexit 出栈ret 返回

776270d2 0f34            sysenter
```

`Sysenter`进入内核后 的总入口是`KiFastCallEntry`

和`int2e`的区别:快速系统调用的`sysenter` 堆栈地址保存在`edx` 返回地址保存在`SharedUserData->SystemCallReturn`指向`KiFastSystemCallRet`地址

`nt!KiFastCallEntry`:

```
83e888e0 b923000000      mov     ecx,23h

83e888e5 6a30            push    30h

83e888e7 0fa1            pop     fs//fs指向kpcr

83e888e9 8ed9            mov     ds,cx//指向用户空间数据段

83e888eb 8ec1            mov     es,cx

83e888ed 648b0d40000000  mov     ecx,dword ptr fs:[40h]//从kpcr获取TSS段的起点

83e888f4 8b6104          mov     esp,dword ptr [ecx+4]//从TSS获取系统空间段的指针

83e888f7 6a23            push    23h//压栈r3的数据段选择子(模仿中断自陷 异常进入内核的指令 会自动堆栈上创建一个框架 这个框架结构一样 所以这里虽然是快速系统调用 但是也是进入内核 模仿自陷 中断 异常进入内核的函数入口样子 照猫画虎 但是注意int2e的内核入口函数没有照猫画虎 因为int 2e本身就是中断 所以cpu进入内核后 堆栈上已经画出老虎了

)

83e888f9 52              push    edx//压栈r3的esp

83e888fa 9c              pushfd//压栈r3的eflags

83e888fb 6a02            push    2

83e888fd 83c208          add     edx,8//跳过用户堆栈的参数块

83e88900 9d              popfd//r0的eflags 所有标志都为0 中断关闭

83e88901 804c240102      or      byte ptr [esp+1],2

83e88906 6a1b            push    1Bh//模仿自陷中断异常的push cs eip

83e88908 ff350403dfff    push    dword ptr ds:[0FFDF0304h]//KiFastSystemCallRet地址

83e8890e 6a00            push    0//这里以下见①注释

83e88910 55              push    ebp

83e88911 53              push    ebx

83e88912 56              push    esi

83e88913 57              push    edi

83e88914 648b1d1c000000  mov     ebx,dword ptr fs:[1Ch]

83e8891b 6a3b            push    3Bh

83e8891d 8bb324010000    mov     esi,dword ptr [ebx+124h]

83e88923 ff33            push    dword ptr [ebx]

83e88925 c703ffffffff    mov     dword ptr [ebx],0FFFFFFFFh

83e8892b 8b6e28          mov     ebp,dword ptr [esi+28h]

83e8892e 6a01            push    1

83e88930 83ec48          sub     esp,48h

83e88933 81ed9c020000    sub     ebp,29Ch

83e88939 c6863a01000001  mov     byte ptr [esi+13Ah],1

83e88940 3bec            cmp     ebp,esp

83e88942 7597            jne     nt!KiFastCallEntry2+0x49 (83e888db)

83e88944 83652c00        and     dword ptr [ebp+2Ch],0

83e88948 f64603df        test    byte ptr [esi+3],0DFh

83e8894c 89ae28010000    mov     dword ptr [esi+128h],ebp

83e88952 0f8538feffff    jne     nt!Dr_FastCallDrSave (83e88790)

83e88958 8b5d60          mov     ebx,dword ptr [ebp+60h]

83e8895b 8b7d68          mov     edi,dword ptr [ebp+68h]

83e8895e 89550c          mov     dword ptr [ebp+0Ch],edx

83e88961 c74508000ddbba  mov     dword ptr [ebp+8],0BADB0D00h

83e88968 895d00          mov     dword ptr [ebp],ebx

83e8896b 897d04          mov     dword ptr [ebp+4],edi

83e8896e fb              sti

83e8896f 8bf8            mov     edi,eax

83e88971 c1ef08          shr     edi,8

83e88974 83e710          and     edi,10h

83e88977 8bcf            mov     ecx,edi

83e88979 03bebc000000    add     edi,dword ptr [esi+0BCh]

83e8897f 8bd8            mov     ebx,eax

83e88981 25ff0f0000      and     eax,0FFFh

83e88986 3b4708          cmp     eax,dword ptr [edi+8]

83e88989 0f8333fdffff    jae     nt!KiBBTUnexpectedRange (83e886c2)

83e8898f 83f910          cmp     ecx,10h

83e88992 751a            jne     nt!KiSystemServiceAccessTeb+0x12 (83e889ae)

83e88994 8b8e88000000    mov     ecx,dword ptr [esi+88h]

83e8899a 33f6            xor     esi,esi
```

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

![](https://bbs.kanxue.com/upload/attach/201704/620086_0wr7uzmd0z743zl.png)

![](https://bbs.kanxue.com/upload/attach/201704/620086_b0i2j39j1aqdd2n.png)

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++以下是`int 2e` 讲解  参考总结++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

`ntdll!ZwReadFile`:

```
    push ebp

mov ebp,esp

Mov eax,111h//系统调用号

Lea edx,[8+ebp]//指向参数块

Int 2e//不支持syscall/sysenter的cpu 只能用这个了

Pop ebp

    ret     9h
```

### 2.2系统调用的内核入口`KiSystemService()`

上面讲到老的cpu是通过`int 2e` 模拟进入内核 而不是新cpu 直接支持快速系统调用

那么`KiSystemService()`就是`int 2e`的处理函数 进入这个函数之前 cpu会自动读取`TR`寄存器 找到`TSS`段 读取里面的`ss esp` 就是内核堆栈地址了 完后往这个地址保存用户空间的堆栈 `eflags cs eip` 具体开头已经说了 这里就不废话了

`nt!KiSystemService`:

```
83e8880e 6a00            push    0   //①是一种类似TrapFrame 进入这个函数之前 cpu会自动压入各种ss esp eip等等 你可以理解为一种上下文 这里push 0 是因为函数最后要把他的值存入eax当做状态码返回 另外就是操作系统 把这个Frame 定义了一个结构 然而异常发生后 cpu会自动压入一个错误码 中断和自陷都没有 所以为了通用 这个结构里面不管是异常还是中断进入内核 结构第一个元素都是0 占个位置的意思

83e88810 55              push    ebp//保存栈帧

83e88811 53              push    ebx//函数下面要用到ebx 所以这里先保存一下

83e88812 56              push    esi//函数下面 要用到esi保存kthread 所以这先保存

83e88813 57              push    edi//函数下面 要用edi引用调用号 所以这也要保存

83e88814 0fa0            push    fs//内核fs指向kpcr 用户层指向TEB 所以这里提前保存 因为进了内核 fs要改变了

83e88816 bb30000000      mov     ebx,30h//r0的fs指向kpcr fs和cs一样都是段选择子 所以这六30h是GDT的kpcr的索引
```

表②:

![](https://bbs.kanxue.com/upload/attach/201704/620086_tuyf43ghdorifc7.png)
