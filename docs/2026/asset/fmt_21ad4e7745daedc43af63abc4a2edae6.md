
# 浅谈NT下Ring3无驱进入Ring0的方法

> [原创]浅谈NT下Ring3无驱进入Ring0的方法
> 
> 关键字:NT,Ring0,无驱
> 
> (测试环境:Windows 2000 SP4,Windows XP SP2. Windows 2003 未测试)

在NT下无驱进入Ring0是一个老生常谈的方法了，网上也有一些C代码的例子，我之所以用汇编重写是因为上次在

[原创/探讨]Windows 核心编程研究系列之一(改变进程 PTE)

的帖子中自己没有实验成功(其实已经成功了，只是自己太马虎，竟然还不知道 -\_-b),顺面聊聊PM(保护模式)中的调用门的使用情况。鉴于这些都是可以作为基本功来了解的知识点,所以对此已经熟悉的朋友就可以略过不看 了，当然由于本人水平有限，各位前来“挑挑刺”也是非常欢迎的，呵呵。

下面言归正传，我们知道在NT中进入Ring0的一般方法是通过驱动，我的Windows 核心编程研究系列 文章前两篇都使用了这个方法进入Ring0 完成特定功能。现在我们还可以通过在Ring3下直接写物理内存的方法来进入Ring0，其主要步骤是:

0  以写权限打开物理内存对象；
1  取得 系统 GDT 地址，并转换成物理地址；
2  构造一个调用门；
3  寻找 GDT 中空闲的位置，将 CallGate 植入；
4  Call植入的调用门。

前面已打通主要关节，现在进一步看看细节问题:

## [零]  默认只有 System 用户有写物理内存的权限

administrators 组的用户只有读的权限，但是通过修改用户安全对象中的DACL 可以增加写的权限:

```
_SetPhyMemDACLs  proc  uses ebx edi esi \
        \_hPhymem:HANDLE,\
        \_ptusrname:dword

           local  @dwret:dword
  local  @htoken:HANDLE
  local  @hprocess:HANDLE
  local  @个
  local  @OldDACLs:PACL
  local  @SecurityDescriptor:PSECURITY_DESCRIPTOR
  local  @Access:EXPLICIT_ACCESS

  mov    @dwret,FALSE

  invoke  RtlZeroMemory,addr @NewDACLs,sizeof @NewDACLs
      invoke  RtlZeroMemory,addr @SecurityDescriptor,\
      sizeof  @SecurityDescriptor

  invoke  GetSecurityInfo,\_hPhymem,SE_KERNEL_OBJECT,\
      DACL_SECURITY_INFORMATION,NULL,NULL,\
      addr @OldDACLs,NULL,\
      addr @SecurityDescriptor

  .if  eax != ERROR_SUCCESS
      jmp  SAFE_RET
  .endif

  invoke  RtlZeroMemory,addr @Access,sizeof @Access

  mov    @Access.grfAccessPermissions,SECTION_ALL_ACCESS
  mov    @Access.grfAccessMode,GRANT_ACCESS
  mov    @Access.grfInheritance,NO_INHERITANCE
  mov    @Access.stTRUSTEE.MultipleTrusteeOperation,\
      NO_MULTIPLE_TRUSTEE
  mov    @Access.stTRUSTEE.TrusteeForm,TRUSTEE_IS_NAME
  mov    @Access.stTRUSTEE.TrusteeType,TRUSTEE_IS_USER
  push  \_ptusrname
  pop    @Access.stTRUSTEE.ptstrName

  invoke  GetCurrentProcess
  mov    @hprocess,eax
  invoke  OpenProcessToken,@hprocess,TOKEN_ALL_ACCESS,\
      addr @htoken

  invoke  SetEntriesInAcl,1,addr @Access,\
      @OldDACLs,addr @NewDACLs

  .if  eax != ERROR_SUCCESS
      jmp  SAFE_RET
  .endif

  invoke  SetSecurityInfo,\_hPhymem,SE_KERNEL_OBJECT,\
      DACL_SECURITY_INFORMATION,NULL,NULL,\
      @NewDACLs,NULL

  .if  eax != ERROR_SUCCESS
      jmp  SAFE_RET
  .endif

  mov    @dwret,TRUE

SAFE_RET:

  .if  @NewDACLs != NULL
      invoke  LocalFree,@NewDACLs
      mov  @NewDACLs,NULL
  .endif

  .if  @SecurityDescriptor != NULL
      invoke  LocalFree,@SecurityDescriptor
      mov  @SecurityDescriptor,NULL
  .endif

  mov    eax,@dwret
  ret

_SetPhyMemDACLs    endp
```

## [一] 可以在Ring3下使用`SGDT`指令取得系统GDT表的虚拟地址

这条指令没有被Intel设计成特权0级的指令。据我的观察，在 Windows 2000 SP4 中 GDT 表的基址都是相同的，而且在 虚拟机VMware 5.5 虚拟的 Windows 2000 SP4中执行 `SGDT` 指令后返回的是错误的结果，在虚拟的 Windows XP中也有同样情况，可能是虚拟机的问题，大家如果有条件可以试一下:

```
local  @stGE:GDT_ENTRY

  mov    @dwret,FALSE

  lea    esi,@stGE
  sgdt  fword ptr [esi]

  assume  esi:ptr GDT_ENTRY

  ;xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  ;在 VMware 虚拟环境下用以下两条指令替代
  ;只用于 Windows 2000 SP4
  ;mov  [esi].Base,80036000h
  ;mov  [esi].Limit,03ffh
  ;xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

  mov    eax,[esi].Base
  invoke  @GetPhymemLite,eax
  .if  eax == FALSE
      jmp  quit
  .endif
```

下面就是虚拟地址转换物理地址了，这在Ring0中很简单，直接调用`MmGetPhysicalAddress` 即可，但在Ring3中要另想办法，还好系统直接将 `0x80000000` ? `0xa0000000` 影射到物理0地址开始的位置，所以可以写一个轻量级的`GetPhysicalAddress`来替代 :)

```
@GetPhymemLite  proc  uses esi edi ebx      \_vaddr
  local  @dwret:dword

  mov    @dwret,FALSE

  .if  \_vaddr < 80000000h
      jmp  quit
  .endif

  .if  \_vaddr >= 0a0000000h
      jmp  quit
  .endif

  mov    eax,\_vaddr
  and    eax,01ffff000h    ;or sub eax,80000000h
  mov    @dwret,eax
quit:
  mov    eax,@dwret
  ret

@GetPhymemLite  endp
```

## [二] 调用门在保护模式中可以看成是低特权级代码向高特权级代码转换的一种实现机制

如图1所示（由于本人较懒,所以借用李彦昌先生所著的80x86保护模式系列教程 中的部分截图，希望李先生看到后不要见怪 ^-^）：

![](https://www.kanxue.com/pediy09-574.files/r3tor0.bmp)

图1

要说明的是调用门也可以完成相同特权级的转换。一般门的结构如图2所示:

门描述符  m+7  m+6  m+5  m+4  m+3  m+2  m+1  m+0
Offset(31...16)  Attributes  Selector  Offset(15...0)

门描述符属性  Byte m+5  Byte m+4
BIT7  BIT6  BIT5  BIT4  BIT3  BIT2  BIT1  BIT0  BIT7  BIT6  BIT5  BIT4  BIT3  BIT2  BIT1  BIT0
P  DPL  DT0  TYPE  000  Dword Count

图2

简单的介绍一下各个主要位置的含义：

`Offset` 和 `Selector` 共同组成目的地址的48位全指针，这意味着，如果远CALL指令指向一个调用门，则CALL指令中的偏移被丢弃；
P位置位代表门有效，DPL是门描述符的特权级，后面要设置成3，以便在Ring3中可以访问。TYPE 是门的类型，386调用门是 `0xC` ,
Dword Count 是系统要拷贝的双字参数的个数，后面也将用到。下面是设置CallGate的代码:

```
mov    eax,\_FucAddr
  mov    @CallGate.OffsetL,ax    ;Low Part Addr Of FucAddr
  mov    @CallGate.Selector,8h  ;Ring0 Code Segment
  mov    @CallGate.DCount,1    ;1 Dword
  mov    @CallGate.GType,AT386CGate  ;Must A CallGate

  shr    eax,16
  mov    @CallGate.OffsetH,ax    ;High Part Addr Of FucAddr
```

## [三] 既然可以读些物理内存了，也知道了GDT的物理基地址和长度

所以可以通过将GDT整个读出，然后寻找一块空闲的区域来植入前面设置好的CallGate：

```
;申请一片空间，以便存放读出的GDT
 Invoke  VirtualAlloc,NULL,@tmpGDTLimit,MEM_COMMIT,\
PAGE_READWRITE
  .if  eax == NULL
      jmp  quit
  .endif

  mov    @pmem,eax
  invoke  @ReadPhymem,@tmpGDTPhyBase,@pmem,@tmpGDTLimit,\
      \_hmem

  .if  eax == FALSE
      jmp  quit
  .endif

  mov    esi,@pmem
  mov    ebx,@tmpGDTLimit
  shr    ebx,3
  ;找到第一个GDT描述符中P位没有置位的地址。
mov    ecx,1
  .while  ecx < ebx
      mov  al,byte ptr [esi+ecx*8+5]
      bt  ax,7
    .if  CARRY?

    .else
      jmp  lop0
    .endif
    Inc    ecx
  .endw

  invoke  VirtualFree,@pmem,0,MEM_RELEASE
  jmp    quit

lop0:
  lea    eax,[ecx*8]
  mov    @OffsetGatePos,eax
  add    @PhyGatePos,eax

  mov    esi,@pmem
  add    esi,eax

  invoke  RtlMoveMemory,addr oldgatebuf,esi,8

  ;释放内存空间
  invoke  VirtualFree,@pmem,0,MEM_RELEASE
```

## [四] 现在主要工作基本完成了

剩下的就是设计一个运行在Ring0中的子函数，在这个子函数中我将调用Ring0里面真正的`MmGetPhysicalAddress`来取得实际的物理地址，所以这个函数要有一个输入参数用来传递要转换的虚拟地址，并且还要考虑到如何获取返回的物理地址(`EDX:EAX`)。在网络上的C版本代码中，这是通过定义几个全局变量来传递的，因为没有发生进程切换，所以可以使用原进程中的一些变量。然而我在传递虚拟地址上采用了另一种做法，就是通过实际形参来传递的:

```
Ring0Fuc  proc      ;\_vaddr

    ;手动保存
    push  ebp
    mov    ebp,esp
    sub    esp,4
    mov    eax,[ebp+0ch]
    mov    [ebp-4],eax    ;first local val
    pushad
    pushfd
    cli

    mov    eax,[ebp-4]
    ;调用真正的 MmGetPhysicalAddress.
    invoke  MmGetPhysicalAddress,eax
    mov    phymem_L,eax
    mov    phymem_H,edx

    popfd
    popad
    ;手动还原
    mov    esp,ebp
    pop  ebp
    retf  4

Ring0Fuc  endp
```

最后，通过一个远CALL来调用这个调用门：

```
    lea    edi,FarAddr
    push  \_vaddr
    call  fword ptr [edi]
```

通过亲手编码，可以对调用门、远调用等一些80386+保护模式中的概念在windows的实现中有了进一步的了解，不再像以前那样模棱两可了。看似全部写完了，其实中间还有很多可以挖掘出来扩展说的细节，但我现在已没有精力写了…（ :( ）,还要准备其他东西，结尾就用这个不是结尾的结尾,结尾吧（绕口令?）。:)

比较好的排版请到我的blog中观赏:

http://blog.csdn.net/mydo/

如果斑竹不忙的话，请帮我重排一下这里的版面，乱七八糟的。我排来排去总排不好:(

侯佩|hopy
2006.01.14 17:09 (机场)办公室
