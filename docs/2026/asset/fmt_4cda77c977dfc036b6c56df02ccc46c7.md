
# Windows NT/2000下不用驱动的Ring0代码实现

Windows NT/2000下不用驱动的Ring0代码实现

WebCrazy(<http://webcrazy.yeah.net/>)

大家知道，Windows NT/2000为实现其可靠性，严格将系统划分为内核模式与用户模式，在i386系统中分别对应CPU的`Ring0`与`Ring3`级别。`Ring0`下，可以执行特权级指令，对任何I/O设备都有访问权等等。要实现从用户态进入核心态，即从`Ring 3`进入`Ring 0`必须借助CPU的某种门机制，如中断门、调用门等。而Windows NT/2000提供用户态执行系统服务(`Ring 0`例程)的此类机制即`System Service`的`int 2eh`中断服务等，严格的参数检查，只能严格的执行Windows NT/2000提供的服务，而如果想执行用户提供的`Ring 0`代码(指运行在`Ring 0`权限的代码)，常规方法似乎只有编写设备驱动程序。本文将介绍一种在用户态不借助任何驱动程序执行`Ring0`代码的方法。

Windows NT/2000将设备驱动程序调入内核区域(常见的位于地址`0x80000000`上)，由`DPL`为0的`GDT`项8，即`cs`为8时实现`Ring 0`权限。本文通过在系统中构造一个指向我们的代码的调用门(`CallGate`)，实现`Ring0`代码。基于这个思路，为实现这个目的主要是构造自己的`CallGate`。`CallGate`由系统中叫`Global Descriptor Table(GDT)`的全局表指定。`GDT`地址可由i386指令`sgdt`获得(`sgdt`不是特权级指令，普通`Ring3`程序均可执行)。`GDT`地址在Windows NT/2000保存于`KPCR(Processor Control Region)`结构中(见《再谈Windows NT/2000环境切换》)。`GDT`中的`CallGate`是如下的格式：

```c
typedef struct
{
    unsigned short  offset_0_15;
    unsigned short  selector;
    unsigned char    param_count : 4;
    unsigned char    some_bits   : 4;
    unsigned char    type        : 4;
    unsigned char    app_system  : 1;
    unsigned char    dpl         : 2;
    unsigned char    present     : 1;
    unsigned short  offset_16_31;
} CALLGATE_DESCRIPTOR;
```

`GDT`位于内核区域，一般用户态的程序是不可能对这段内存区域有直接的访问权。幸运的是Windows NT/2000提供了一个叫`PhysicalMemory`的`Section`内核对象位于`\Device`的路径下。顾名思义，通过这个`Section`对象可以对物理内存进行操作。用`objdir.exe`对这个对象分析如下：

```
C:\NTDDK\bin>objdir /D \Device
PhysicalMemory
    Section
    DACL -
       Ace[ 0] - Grant - 0xf001f - NT AUTHORITY\SYSTEM
                         Inherit:
                         Access: 0x001F  and  ( D RCtl WOwn WDacl )
       Ace[ 1] - Grant - 0x2000d - BUILTIN\Administrators
                         Inherit:
                         Access: 0x000D  and  ( RCtl )
```

从dump出的这个对象`DACL`的`Ace`可以看出默认情况下只有`SYSTEM`用户才有对这个对象的读写权限，即对物理内存有读写能力，而`Administrator`只有读权限，普通用户根本就没有权限。不过如果我们有`Administrator`权限就可以通过`GetSecurityInfo`、`SetEntriesInAcl`与`SetSecurityInfo`这些`API`来修改这个对象的`ACE`。这也是我提供的代码需要`Administrator`的原因。实现的代码如下：

```c
VOID SetPhyscialMemorySectionCanBeWrited(HANDLE hSection)
{
   PACL pDacl=NULL;
   PACL pNewDacl=NULL;
   PSECURITY_DESCRIPTOR pSD=NULL;
   DWORD dwRes;
   EXPLICIT_ACCESS ea;
   if(dwRes=GetSecurityInfo(hSection,SE_KERNEL_OBJECT,DACL_SECURITY_INFORMATION,
              NULL,NULL,&pDacl,NULL,&pSD)!=ERROR_SUCCESS)
      {
         printf( "GetSecurityInfo Error %u\n", dwRes );
         goto CleanUp;
      }
   ZeroMemory(&ea, sizeof(EXPLICIT_ACCESS));
   ea.grfAccessPermissions = SECTION_MAP_WRITE;
   ea.grfAccessMode = GRANT_ACCESS;
   ea.grfInheritance= NO_INHERITANCE;
   ea.Trustee.TrusteeForm = TRUSTEE_IS_NAME;
   ea.Trustee.TrusteeType = TRUSTEE_IS_USER;
   ea.Trustee.ptstrName = "CURRENT_USER";
   if(dwRes=SetEntriesInAcl(1,&ea,pDacl,&pNewDacl)!=ERROR_SUCCESS)
      {
         printf( "SetEntriesInAcl %u\n", dwRes );
         goto CleanUp;
      }
   if(dwRes=SetSecurityInfo(hSection,SE_KERNEL_OBJECT,DACL_SECURITY_INFORMATION,NULL,NULL,pNewDacl,NULL)!=ERROR_SUCCESS)
      {
         printf("SetSecurityInfo %u\n",dwRes);
         goto CleanUp;
      }
CleanUp:
   if(pSD)
      LocalFree(pSD);
   if(pNewDacl)
      LocalFree(pSD);
}
```

这段代码对给定`HANDLE`的对象增加了如下的`ACE`:

```
PhysicalMemory
    Section
    DACL -
       Ace[ 0] - Grant - 0x2 - WEBCRAZY\Administrator
                         Inherit:
                         Access: 0x0002    //SECTION_MAP_WRITE
```

这样我们在有`Administrator`权限的条件下就有了对物理内存的读写能力。但若要修改`GDT`表实现`Ring 0`代码。我们将面临着另一个难题，因为`sgdt`指令获得的`GDT`地址是虚拟地址(线性地址)，我们只有知道`GDT`表的物理地址后才能通过`\Device\PhysicalMemory`对象修改`GDT`表，这就牵涉到了线性地址转化成物理地址的问题。我们先来看一看Windows NT/2000是如何实现这个的：

```
kd> u nt!MmGetPhysicalAddress l 30
ntoskrnl!MmGetPhysicalAddress:
801374e0 56               push    esi
801374e1 8b742408         mov     esi,[esp+0x8]
801374e5 33d2             xor     edx,edx
801374e7 81fe00000080     cmp     esi,0x80000000
801374ed 722c             jb    ntoskrnl!MmGetPhysicalAddress+0x2b (8013751b)
801374ef 81fe000000a0     cmp     esi,0xa0000000
801374f5 7324             jnb   ntoskrnl!MmGetPhysicalAddress+0x2b (8013751b)
801374f7 39153ce71780     cmp     [ntoskrnl!MmKseg2Frame (8017e73c)],edx
801374fd 741c             jz    ntoskrnl!MmGetPhysicalAddress+0x2b (8013751b)
801374ff 8bc6             mov     eax,esi
80137501 c1e80c           shr     eax,0xc
80137504 25ffff0100       and     eax,0x1ffff
80137509 6a0c             push    0xc
8013750b 59               pop     ecx
8013750c e8d3a7fcff       call    ntoskrnl!_allshl (80101ce4)
80137511 81e6ff0f0000     and     esi,0xfff
80137517 03c6             add     eax,esi
80137519 eb17             jmp   ntoskrnl!MmGetPhysicalAddress+0x57 (80137532)
8013751b 8bc6             mov     eax,esi
8013751d c1e80a           shr     eax,0xa
80137520 25fcff3f00       and     eax,0x3ffffc
80137525 2d00000040       sub     eax,0x40000000
8013752a 8b00             mov     eax,[eax]
8013752c a801             test    al,0x1
8013752e 7506             jnz   ntoskrnl!MmGetPhysicalAddress+0x44 (80137536)
80137530 33c0             xor     eax,eax
80137532 5e               pop     esi
80137533 c20400           ret     0x4
```

从这段汇编代码可看出如果线性地址在`0x80000000`与`0xa0000000`范围内，只是简单的进行移位操作(位于`801374ff-80137519`指令间)，并未查页表。我想Microsoft这样安排肯定是出于执行效率的考虑。这也为我们指明了一线曙光，因为`GDT`表在Windows NT/2000中一般情况下均位于这个区域(我不知道/3GB开关的Windows NT/2000是不是这种情况)。

经过这样的分析，我们就可以只通过用户态程序修改`GDT`表了。而增加一个`CallGate`就不是我可以介绍的了，找本Intel手册自己看一看了。具体实现代码如下：

```c
typedef struct gdtr {
    short Limit;
    short BaseLow;
    short BaseHigh;
} Gdtr_t, *PGdtr_t;

ULONG MiniMmGetPhysicalAddress(ULONG virtualaddress)
{
    if(virtualaddress<0x80000000||virtualaddress>=0xA0000000)
       return 0;
    return virtualaddress&0x1FFFF000;
}

BOOL ExecRing0Proc(ULONG Entry,ULONG seglen)
{
   Gdtr_t gdt;
   __asm sgdt gdt;
   ULONG mapAddr=MiniMmGetPhysicalAddress(gdt.BaseHigh<<16U|gdt.BaseLow);
   if(!mapAddr) return 0;
   HANDLE   hSection=NULL;
   NTSTATUS status;
   OBJECT_ATTRIBUTES        objectAttributes;
   UNICODE_STRING objName;
   CALLGATE_DESCRIPTOR *cg;
   status = STATUS_SUCCESS;
   RtlInitUnicodeString(&objName,L"\\Device\\PhysicalMemory");
   InitializeObjectAttributes(&objectAttributes,
                              &objName,
                              OBJ_CASE_INSENSITIVE | OBJ_KERNEL_HANDLE,
                              NULL,
                             (PSECURITY_DESCRIPTOR) NULL);
   status = ZwOpenSection(&hSection,SECTION_MAP_READ|SECTION_MAP_WRITE,&objectAttributes);
   if(status == STATUS_ACCESS_DENIED){
      status = ZwOpenSection(&hSection,READ_CONTROL|WRITE_DAC,&objectAttributes);
      SetPhyscialMemorySectionCanBeWrited(hSection);
      ZwClose(hSection);
      status =ZwOpenSection(&hSection,SECTION_MAP_WRITE|SECTION_MAP_WRITE,&objectAttributes);
   }
   if(status != STATUS_SUCCESS)
     {
        printf("Error Open PhysicalMemory Section Object,Status:%08X\n",status);
        return 0;
     }
   PVOID BaseAddress;
   BaseAddress=MapViewOfFile(hSection,
                 FILE_MAP_READ|FILE_MAP_WRITE,
                 0,
                 mapAddr,    //low part
                 (gdt.Limit+1));
   if(!BaseAddress)
      {
         printf("Error MapViewOfFile:");
         PrintWin32Error(GetLastError());
         return 0;
      }
   BOOL setcg=FALSE;
```
