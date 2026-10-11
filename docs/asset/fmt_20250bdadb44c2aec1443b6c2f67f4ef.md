
### 调试器输出

```
kd> 

eax=61caf508 ebx=00000000 ecx=e1dfb800 edx=00000000 esi=61caf508 edi=fb6e6bd0 

eip=804b398f esp=fb6e6af8 ebp=fb6e6b08 iopl=0 nv up ei ng nz na pe nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000282 

nt!ExLockHandleTableEntry+0x24: 

804b398f 8b4d0c mov ecx,[ebp+0xc] ss:0010:fb6e6b14=e1dfb8f0 

kd> 

eax=61caf508 ebx=00000000 ecx=e1dfb8f0 edx=00000000 esi=61caf508 edi=fb6e6bd0 

eip=804b3992 esp=fb6e6af8 ebp=fb6e6b08 iopl=0 nv up ei ng nz na pe nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000282 

nt!ExLockHandleTableEntry+0x27: 

804b3992 8b55fc mov edx,[ebp-0x4] ss:0010:fb6e6b04=e1caf508 

kd> 

eax=61caf508 ebx=00000000 ecx=e1dfb8f0 edx=e1caf508 esi=61caf508 edi=fb6e6bd0 

eip=804b3995 esp=fb6e6af8 ebp=fb6e6b08 iopl=0 nv up ei ng nz na pe nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000282 

nt!ExLockHandleTableEntry+0x2a: 

804b3995 0fb111 cmpxchg [ecx],edx ds:0023:e1dfb8f0=61caf508 

kd> 

Breakpoint 1 hit 

eax=61caf508 ebx=00000000 ecx=e1dfb8f0 edx=e1caf508 esi=61caf508 edi=fb6e6bd0 

eip=804b3998 esp=fb6e6af8 ebp=fb6e6b08 iopl=0 nv up ei pl zr na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000246 

nt!ExLockHandleTableEntry+0x2d: 

804b3998 3bc6 cmp eax,esi 

kd> 

eax=61caf508 ebx=00000000 ecx=e1dfb8f0 edx=e1caf508 esi=61caf508 edi=fb6e6bd0 

eip=804b399a esp=fb6e6af8 ebp=fb6e6b08 iopl=0 nv up ei pl zr na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000246 

nt!ExLockHandleTableEntry+0x2f: 

804b399a 0f858ddb0000 jne nt!ExLockHandleTableEntry+0x31 (804c152d) 

kd> 

eax=61caf508 ebx=00000000 ecx=e1dfb8f0 edx=e1caf508 esi=61caf508 edi=fb6e6bd0 

eip=804b39a0 esp=fb6e6af8 ebp=fb6e6b08 iopl=0 nv up ei pl zr na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000246 

nt!ExLockHandleTableEntry+0x60: 

804b39a0 b001 mov al,0x1 

kd> 

eax=61caf501 ebx=00000000 ecx=e1dfb8f0 edx=e1caf508 esi=61caf508 edi=fb6e6bd0 

eip=804b39a2 esp=fb6e6af8 ebp=fb6e6b08 iopl=0 nv up ei pl zr na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000246 

nt!ExLockHandleTableEntry+0x62: 

804b39a2 5e pop esi 

kd> 

eax=61caf501 ebx=00000000 ecx=e1dfb8f0 edx=e1caf508 esi=e1dfb8f0 edi=fb6e6bd0 

eip=804b39a3 esp=fb6e6afc ebp=fb6e6b08 iopl=0 nv up ei pl zr na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000246 

nt!ExLockHandleTableEntry+0x63: 

804b39a3 5b pop ebx 

kd> 

eax=61caf501 ebx=00000078 ecx=e1dfb8f0 edx=e1caf508 esi=e1dfb8f0 edi=fb6e6bd0 

eip=804b39a4 esp=fb6e6b00 ebp=fb6e6b08 iopl=0 nv up ei pl zr na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000246 

nt!ExLockHandleTableEntry+0x64: 

804b39a4 c9 leave 

kd> 

eax=61caf501 ebx=00000078 ecx=e1dfb8f0 edx=e1caf508 esi=e1dfb8f0 edi=fb6e6bd0 

eip=804b39a5 esp=fb6e6b0c ebp=fb6e6bc4 iopl=0 nv up ei pl zr na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000246 

nt!ExLockHandleTableEntry+0x65: 

804b39a5 c20800 ret 0x8 

kd> 

eax=61caf501 ebx=00000078 ecx=e1dfb8f0 edx=e1caf508 esi=e1dfb8f0 edi=fb6e6bd0 

eip=804b3a17 esp=fb6e6b18 ebp=fb6e6bc4 iopl=0 nv up ei pl zr na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000246 

nt!ExMapHandleToPointer+0x1e: 

804b3a17 f6d8 neg al 

kd> 

eax=61caf5ff ebx=00000078 ecx=e1dfb8f0 edx=e1caf508 esi=e1dfb8f0 edi=fb6e6bd0 

eip=804b3a19 esp=fb6e6b18 ebp=fb6e6bc4 iopl=0 nv up ei ng nz ac po cy 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000297 

nt!ExMapHandleToPointer+0x20: 

804b3a19 1bc0 sbb eax,eax 

kd> 

eax=ffffffff ebx=00000078 ecx=e1dfb8f0 edx=e1caf508 esi=e1dfb8f0 edi=fb6e6bd0 

eip=804b3a1b esp=fb6e6b18 ebp=fb6e6bc4 iopl=0 nv up ei ng nz ac po cy 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000297 

nt!ExMapHandleToPointer+0x22: 

804b3a1b 23c6 and eax,esi 

kd> 

eax=e1dfb8f0 ebx=00000078 ecx=e1dfb8f0 edx=e1caf508 esi=e1dfb8f0 edi=fb6e6bd0 

eip=804b3a1d esp=fb6e6b18 ebp=fb6e6bc4 iopl=0 nv up ei ng nz na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000286 

nt!ExMapHandleToPointer+0x24: 

804b3a1d 5e pop esi 

kd> 

eax=e1dfb8f0 ebx=00000078 ecx=e1dfb8f0 edx=e1caf508 esi=00000000 edi=fb6e6bd0 

eip=804b3a1e esp=fb6e6b1c ebp=fb6e6bc4 iopl=0 nv up ei ng nz na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000286 

nt!ExMapHandleToPointer+0x25: 

804b3a1e c20800 ret 0x8 
```

### 句柄与对象信息

```
kd> !handle 78 ff fcc654e0

processor number 0 

PROCESS fcc654e0 SessionId: 0 Cid: 03ac Peb: 7ffdf000 ParentCid: 02e0 

DirBase: 03cc6000 ObjectTable: fcc59988 TableSize: 16. 

Image: NOTEPAD.EXE 

Handle Table at e1dfb000 with 16 Entries in use 

0078: Object: e1caf520 GrantedAccess: 000f003f (Locked) 

Object: e1caf520 Type: (fcebc160) Key 

ObjectHeader: e1caf508 

HandleCount: 1 PointerCount: 1 

Directory Object: 00000000 Name: \\REGISTRY\\USER 
```

我们得到了什么呢？我们可以看到我们进入了句柄表中，并找到了一些奇怪的数字。那个奇怪的数字而后被对象头 `e1caf50` 替换，真正的对象在 `ObjectHeader+18h` 处，`e1caf508+18=e1caf520`。

```
kd> !object e1caf520

Object: e1caf520 Type: (fcebc160) Key 

ObjectHeader: e1caf508 

HandleCount: 1 PointerCount: 1 

Directory Object: 00000000 Name: \\REGISTRY\\USER 
```

哪个计算能得到这个值呢？很简单，`or eax ,0x80000000`。我们来试试其它的对象，比如 `7cdee148`，加上 `18h`，然后使用 `!object` 命令。

```
kd> !object fcdee160

Object: fcdee160 Type: (fcebf420) Event 

ObjectHeader: fcdee148 

HandleCount: 1 PointerCount: 1 
```

这很简单，不是吗？

### 堆信息

我们可以在内核态下显示某个进程的堆信息。把上下文设置为对应进程，然后打印出 `PEB` 信息。`PEB` 会显示堆的个数和一个使用的堆列表，每个堆都有一个 `VirtualAlloc` 链表。我们当然也可以在用户态下得到这些消息。最大的不同就是你可能不能打印出所有信息，如果它被页换出的话。但是你并不需要自己去完成这些工作，还是和第三篇那样在内核态下使用 `!heap`，设置上下文就可以了。

### Pools

内核驱动一般都使用分页和非分页内存。你可以使用 `!pool` 命令来确定它的大小，tag 信息等等。如果你开发过驱动，你就会知道，在内核中分配的内存是来自于全局的，而不是对于某个进程而言的。因此，如果你破坏了内存，你也会使其他驱动崩溃。当一个驱动分配内存时，它们可以指定一个 `pool tag`，你就可以根据这个来确定是哪个驱动分配了内存。你可以还使用 `!poolfind *` 来找到你想要的那块内存。网上也有一些关于这方面的资料。

下面我们来看看对一个对象使用 `!pool` 命令。

```
kd> !pool e1caf520

e1caf000 size: 60 previous size: 0 (Allocated) NtFd 

e1caf060 size: 20 previous size: 60 (Free) .... 

e1caf080 size: 40 previous size: 20 (Allocated) NtFs 

e1caf0c0 size: 20 previous size: 40 (Free) Key 

e1caf0e0 size: 20 previous size: 20 (Allocated) CMVI 

e1caf100 size: 40 previous size: 20 (Allocated) CMVa 

e1caf140 size: 20 previous size: 40 (Free) CMVa 

e1caf160 size: 40 previous size: 20 (Lookaside) Key (Protected) 

e1caf1a0 size: 20 previous size: 40 (Allocated) Ntf0 

e1caf1c0 size: 40 previous size: 20 (Allocated) IoNm 

e1caf200 size: 20 previous size: 40 (Free) Gla8 

e1caf220 size: 40 previous size: 20 (Allocated) CMVa 

e1caf260 size: 40 previous size: 40 (Allocated) CMVa 

e1caf2a0 size: 20 previous size: 40 (Free) Key 

e1caf2c0 size: 60 previous size: 20 (Allocated) Ntfo 

e1caf320 size: 40 previous size: 60 (Allocated) CMNb (Protected) 

e1caf360 size: 40 previous size: 40 (Allocated) CMIn (Protected) 

e1caf3a0 size: a0 previous size: 40 (Allocated) SeSd 

e1caf440 size: 20 previous size: a0 (Free) Usqm 

e1caf460 size: 40 previous size: 20 (Allocated) CMVa 

e1caf4a0 size: 40 previous size: 40 (Allocated) CMVa 
```
