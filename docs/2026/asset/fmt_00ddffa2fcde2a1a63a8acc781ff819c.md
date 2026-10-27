
# [原创]游戏保护大放送之梦幻诛仙保护

本人不保证此办法现在还生效！蓝屏死机于本人无关！

完美保护里面有个反调试比较BT，不过多开更bt。（梦幻诛仙为例）

第一，先说一些ssdt hook和inline hook的解决办法，也就是 hook `Kifastcallentry` 来搞。

第二，附加卡死情况的解决。完美驱动会获取`FastMutex`而不释放，导致卡死情况。

发一小段代码

```
ULONG ExReleaseFastMutex_addr,ReExReleaseFastMutex_addr;
char g_ExReleaseFastMutex[8];

VOID GetExReleaseFastMutexAddr()
{
    ULONG  Address=0;
    ULONG temp_addr;
    char* temp_cr=0;

    Address = (ULONG)_KeServiceDescriptorTable->NotUse1.ServiceTableBase + 269 * 4;

    temp_addr = *(ULONG*)Address;
    temp_addr=temp_addr+0x19A;
    temp_cr=(char*)temp_addr;
    ExReleaseFastMutex_addr=*((ULONG*)&temp_cr[2]);
    ExReleaseFastMutex_addr=*(ULONG*)ExReleaseFastMutex_addr;
}

VOID __declspec(naked)MyExReleaseFastMutex()
{
    __asm
    {
        cmp        ecx,DbgkpProcessDebugPortMutexaddr
        jnz        ONMYFAST
        mov        ecx,pMyDbgkpProcessDebugPortMutex

ONMYFAST:
        xor     eax, eax
        mov     al, [ecx + 0x1c]
        jmp        [ReExReleaseFastMutex_addr]
    }
}

VOID Patch_ExReleaseFastMutex()
{
    KIRQL oldIrql;
    unsigned char newcode[] = { 0xE9, 0x44, 0x33, 0x22, 0x11};

    GetExReleaseFastMutexAddr();

    ReExReleaseFastMutex_addr=ExReleaseFastMutex_addr+5;

    *( (ULONG *)(&newcode[1]) ) =(ULONG)MyExReleaseFastMutex-ExReleaseFastMutex_addr-5;

    WPOFF();
    oldIrql = KeRaiseIrqlToDpcLevel();

    memcpy(g_ExReleaseFastMutex,(VOID*)ExReleaseFastMutex_addr,5);
    memcpy((VOID*)ExReleaseFastMutex_addr,newcode,5);

    KeLowerIrql(oldIrql);
    WPON();
}

VOID UnPatch_ExReleaseFastMutex()
{
    KIRQL oldIrql;

    WPOFF();
    oldIrql = KeRaiseIrqlToDpcLevel();

    memcpy((VOID*)ExReleaseFastMutex_addr,g_ExReleaseFastMutex,5);

    KeLowerIrql(oldIrql);
    WPON();
}
```

第三，`int 1` `int 3`处理。以前的版本完美驱动有2个监视线程，你只要hook创建线程的内核函数就可以把他干掉。最后更新后就不能干了，干掉后游戏启动不了。所有只能找到一些关键点来patch。如果不patch的话就不能恢复IDT，会重启。

第四，ring3和ring0的通信。应用层和驱动层是通过`int 0x22`来通信的，`int 0x23`没有怎么分析。自己要接管`int 0x22`的通信，也就是把他的替换成我们的代码。

第五，游戏多开。这个比过保护都困难。梦诛好像只能2开。他多开是这样检测的，在驱动里有8个字节的空间，其实是2个结构体。结构体原型如下：

```
typedef struct WMGAMEOPEN
{
    ULONG PID;   //游戏进程
    PEPROCESS eProcess;   //游戏进程的EPROCESS
} WMGameOpen;
```

每启动一个游戏驱动里就会记录一条，总共能记录2条，也就只能2开。而且游戏会定时和驱动通讯来验证游戏进程的`pid`等是否吻合。如果这2个记录里没有这个游戏的`pid`，那就等着重启吧。

最后丫丫一下他的驱动，他公共有2个驱动一个只是一个loader，没有什么作用，另一个才是真真管用的。`pp_data0`.。。。不过pe被破坏了。

下面见代码。
