
## 6. 继续分析 `AfdTransmitFile`

这里为了节省篇幅，动态跟踪的图就不放了，按照上面的步骤你可以自行查看

![](upload/attach/202205/930159_C29D6U33KA94DWR.png)

![](upload/attach/202205/930159_PV8AZMK6EX9WGSY.png)

简单解释一下，上图中，我把 `esi` 假设一个结构体（根据结构体逆向经验可知），目前来看这个结构体的大小为 `0xC`

```c
struct unknow{
int flag; //标志一类的东西
ULONG length; //长度
void VirtualAddress; //虚拟地址
PMDL mdl; //mdl结构的指针
}
```

然后返回去看一下，上图中有一个乘法的部分，因为 `esi = 0 = *(tpinfo+0x28)`，所以最后相当于 `0 + *(tpinfo+0x20)`

如果把乘法那两条指令反编译，会得到一个寻址公式 `*(tpinfo+0x20) + *(tpinfo+0x28) *18`，简化一下 `p + a * 18`

可以得到这是一个数组寻址公式，p作为数组首地址，a作为index，`0x18`作为每一个元素的大小

再接着下面的分析，寻址出来的数据刚好被我们当作 `unknow` 结构体来解释

所以这里可得，`*(tpinfo+0x20)` 是作为一个结构体数组的首地址

再看 `*(tpinfo+0x28)`，第一次被使用它的值为 `0`，寻址完后，它的值变为 `1`，预示着后面会寻址下一个 `unknow` 结构体，这也符合我们的推测

既然 `p` 与 `a` 都确定了下来，那么 `0x18` 没跑，肯定是作为一个 `size` 的定位，且这个 `size` 就是 `unknow` 结构体的 `size`

然后再往上看，`*(tpinfo+0x38) = 0x1000`，且这个值是被全局变量传过来的，`AfdTransmitIoLength`，传输 `IO` 长度，不清楚什么意思

现在就可根据分析，来大致描述出 `tpinfo` 与 `unknow` 的结构

```c
struct tpinfo {
0x20 void unknowArrary; //unknow结构体数组首地址
0x28 DWORD index; //unknow结构体数组的索引
0x38 DWORD TransmitIoLength; //传输IO长度
}
struct unknow {
0x0 int flag; //标志一类的东西
0x4 ULONG length; //长度
0x8 void VirtualAddress; //虚拟地址
0xC void mdl; //mdl结构的指针
0x10 int Reserved1; //没使用
0x14 int Reserved2; //没使用
}
```

注：外文 `PDF` 中称这个结构体为 `TpInfoElement` 当然名字并不重要

那么现在还有一个问题，如果说 `tpinfo+0x28` 的值作为索引，那它寻址完后加 `1` 是不是不合理，可能会产生溢出，此时猜测我们对它的分析还是有些偏差，或者还有没注意到的成员

此时往上看，在 `AfdAllocateTpInfo` 中调用了 `AfdInitializeTpInfo(TpInfo, AfdDefaultTpInfoElementCount, AfdTdiStackSize, 1);`

这里 `AfdDefaultTpInfoElementCount` 的值为 `3`，可以推测它初始了 `3` 个 `unknow` 结构体

但是根据我们上面留存的数据比对，数组的首地址为 `0x89654158`，数据中并没有 `3` 的字样，且 `0x28` 偏移也就是我们推测的 `index` 为 `0`

那么我们根据微软的习惯，大胆推测，这个 `0x28` 偏移为索引计数，在寻址后自加 `1`，以便寻址下一个 `unknow` 结构，`3` 没有出现在结构体中是因为微软使用了它常用的数组判断，结尾时放置同样大小的 `0` 来确定结尾，这样就可以解释的通 `0x28` 处的 `index` 的作用

当然，这里我没有动态去查看它的结尾是否为 `0`，推测而已

接下来继续解释 `MDL` 相关的问题

以下为 `MDL` 相关的定义和参数的意义与返回值

```c
PMDL IoAllocateMdl(
[in, optional] __drv_aliasesMem PVOID VirtualAddress,
[in] ULONG Length,
[in] BOOLEAN SecondaryBuffer,
[in] BOOLEAN ChargeQuota,
[in, out, optional] PIRP Irp
);
/*
VirtualAddress - 将要映射的起始虚拟地址。
Length - 虚拟地址空间需要映射的长度，以字节为单位。
SecondaryBuffer - 此参数仅当指定了IRP时有效。如果SecondaryBuffer为TRUE，那么将链接MDL到IRP的MDL链表中(通过MDL->next)，否则将申请到的MDL替换掉原来IRP的MDL（替换Irp->MdlAddress）
ChargeQuota - 指定MDL的配额控制。注意，在WRK1.2中此参数被忽略。
Irp - 指定MDL服务的IRP，为可选的参数。
return ： 返回申请的MDL，创建MDL失败返回NULL。
如果是因为配额导致的MDL创建失败，由调用者负责捕捉这个异常。
*/
typedef struct _MDL {
struct _MDL *Next; //指向下一个MDL的指针
CSHORT Size; //整个MDL的长度 包含后面的数组
CSHORT MdlFlags; //MDL的标记值
struct _EPROCESS *Process; //所属进程的EPROCESS
PVOID MappedSystemVa; //该缓冲区映射在系统的地址VA
PVOID StartVa; //虚拟地址 4K对齐
ULONG ByteCount; //缓冲区长度
ULONG ByteOffset; //虚拟地址的偏移
} MDL, *PMDL;
//StartVa + ByteOffset = 该段缓冲在Process进程空间中的虚拟地址
```

`MmProbeAndLockPages` 主要用来将分页内存锁定，防止交换，它的第二个参数为 `1`，表示用户态地址，第一个参数为 `mdl`，第三个参数为访问权限

这里动态跟踪一下，查看 `mdl` 结构的信息

```
1: kd> dt _MDL 87dd8a20
ntdll!_MDL
+0x000 Next : (null)
+0x004 Size : 0n1500
+0x006 MdlFlags : 0n0
+0x008 Process : (null)
+0x00c MappedSystemVa : (null)
+0x010 StartVa : 0x13371000 Void
+0x014 ByteCount : 0x16ecca
+0x018 ByteOffset : 0x337
1: kd> dd 0x13371000
13371000 ???????? ???????? ???????? ????????
13371010 ???????? ???????? ???????? ????????
```

虚拟地址为问号，说明这是无效内存，此时想起来 `POC` 中的一些数据，整理一下逆向过程中得到的数据

```c
DWORD targetSize = 0x310;
DWORD virtualAddress = 0x13371337;
DWORD mdlSize = (0x4000 * (targetSize - 0x30) / 8) - 0xFFF - (virtualAddress & 0xFFF);
static DWORD inbuf1[100];
memset(inbuf1, 0, sizeof(inbuf1));
inbuf1[6] = virtualAddress;
inbuf1[7] = mdlSize;
inbuf1[10] = 1;
```

```c
tmpbuf + 0x18 = VirtualAddress; // 对应 inbuf1[6]
tmpbuf + 0x1C = Length; // 对应 inbuf1[7]
tmpbuf + 0x28 = 0x1; //最开始是1 经过一个or运算变为了0x11 对应inbuf1[10]
```

`size` 为什么是这样计算，目前还不清楚，先记下这些数据，留着备用

可以看到 `poc` 指定了一个地址 `0x13371337`，刚好对应 `mdl` 中的 `StartVA + Offset`，大小经过计算对应 `0x16ecca`

好的，下面调用了锁定内存的函数，因为这是一个无效内存，则必定会触发异常

我们提前来到异常处理函数的位置下断，`2C376` 为 `try` 开始的地址，`IDA` 有提示

![](upload/attach/202205/930159_SHHK9Y8Q7EAG8FU.png)

![](upload/attach/202205/930159_ZXY62YHZPBZM85A.png)

进入到这以后，会进行判断 `v19` 是否为 `0`，最开始初始化为 `0` 了，所以跟着跳转，看到调用 `AfdReturnTpInfo`，且 `AfdReturnTpInfo` 的两个参数分别为 `tpinfo flag`

## 7. 逆向分析 `AfdReturnTpInfo`

首先函数进来以后，把参数 `tpinfo` 给到 `esi`，之后无条件跳转

![](upload/attach/202205/930159_ENTQ9BYSVPWSYK9.png)

这部分我没有动态调试，根据我之前留存的数据即可查找到对应的值（还好我有先见之明qaq）

![](upload/attach/202205/930159_M5EJDA3YN9YVGD3.png)

这部分主要是判断后释放掉了 `unknow` 中的 `mdl` 指针

![](upload/attach/202205/930159_P3DDF35U68Z2NKK.png)

这部分主要是释放了 `tpinfo`，其实也不叫释放，根据前面的相关分析可知，这个函数会把内存块挂入 `Lookaside` 的单向链表

并且释放的内存块如果没达到 `Lookaside` 结构中的深度的限度值，则这个内存块的数据是不会改变的

且经过前面的分析，`afd` 使用了刚初始化 `Lookaside` 结构，所以第一次申请时链表为空，调用了结构指定的申请内存例程，如果在 `Lookaside` 中申请内存的时候，链表不为空，则可以直接申请到被释放的那块内存

也就是说，现在要想触发双重释放漏洞，则必须从 `Lookaside` 中取出这个块内存再次触发异常，调用 `AfdReturnTpInfo` 执行相同的流程来触发 `Crash`

好的，接下来分析 `POC` 是怎样实现这个过程的

# 0x120C3 处理例程分析

根据前面的调用堆栈可以看出执行流程，派遣函数第一个调用的函数为 `AfdTransmitPackets`，所以 `IDA` 启动！

## 1. 逆向分析 `AfdTransmitPackets`

`AfdTransmitPackets` 的参数为 `pIrp` 与 `pIoStackLocation`

![](upload/attach/202205/930159_USK5EVFY68QJMEN.png)

![](upload/attach/202205/930159_FNUPHSUNPGQKSYP.png)

![](upload/attach/202205/930159_T6PA89NH9PSUW58.png)

这里可以看到 `POC` 中的 `1` 与 `0AAAAAAA` 已经拷贝过来了

```
static DWORD inbuf2[100];
memset(inbuf2, 0, sizeof(inbuf2));
inbuf2[0] = 1;
inbuf2[1] = 0x0AAAAAAA;
1: kd> dd ebp-78
902afa74 00000001 0aaaaaaa 00000000 00000000
```

![](upload/attach/202205/930159_ZHYGZA5BAH7WY94.png)

接下来调用了 `AfdTliGetTpInfo`，注意它的参数，`ecx = 0x0AAAAAAA` 7个 `A`

好的，现在已经得到了 `poc` 中第二次调用的数据的用处，总结一下 `poc` 的工作流程

```
InputBufferLength >= 0x10
InputBuffer & 0x3 == 0
InputBuffer < 0x7FFF0000
*(InputBuffer + 0xC) & 0xFFFFFFC8 == 0
*(InputBuffer + 0xC) & 0x30 != 0x30
*InputBuffer != 0
*(InputBuffer + 0x4) != 0
*(InputBuffer + 0x4) <= 0x0AAAAAAA
如果InputBuffer满足这些条件则调用
AfdTliGetTpInfo(0x0AAAAAAA)
```

接下来再让我们看一下 `AfdTliGetTpInfo` 中有什么玄机

## 2. 再次分析 `AfdTliGetTpInfo`

注：有一大段没保存 我心态裂了...但还是坚持重新写了

![](upload/attach/202205/930159_3USPTWV8VSPZ549.png)

这里我们可以看到与第一次调用不同的流程为，这次进入了 `if` 中，由于 `edi` 是由用户可控参数设置，所以这里会再次申请内存

申请的内容为 `esi` 个 `unknow` 结构体，大小为 `0x18`，这也印证了前面的判断 `unknow` 结构体大小为 `0x18`

这里总申请的内存大小为 `0x0AAAAAAA * 0x18 = 0xFFFFFFF0`，明显在 `32` 位程序中是不可能的，所以这里会触发异常

异常流程为图中箭头指向的位置，可以看到同样的剧情，调用 `AfdReturnTpInfo`

注意此时的情况，此时已经执行了 `ExAllocateFromNPagedLookasideList`，申请到了 `tpinfo` 结构，也就是之前释放的那个

为什么这么说呢，因为根据我们之前的分析，第一次申请 `tpinfo` 时 `LookasideList` 为空，之后释放到 `Lookaside` 中，所以第二个申请可以申请到这个同样的内存

但是这个内存中的 `tpinfo->unknow.mdl` 的值 还是释放后的值，所以再次进入 `AfdReturnTpInfo` 进行同样的数据，同样的流程，会再次释放这个 `mdl` 指针，造成 `double free`

# 漏洞原理总结

1. 第一次 `DeviceIoControl`， `IoControlCode = 0x1207F`

`afd.sys` 调用 `AfdTransmitFile`，`AfdTransmitFile` 首先会调用 `AfdTliGetTpInfo` 获得一个 `TpInfo` 结构体，然后根据用户的输入，去申请 `mdl`，`mdl` 申请后，由于用户的输入，造成了锁定 `mdl` 的时候出现异常，进入 `AfdReturnToInfo` 释放了 `mdl` 指针，此时为第一次 `free`。之后又因为 `LookasideList` 的相关操作，导致了这块内存可以重复使用，并且里面的数据都没有被改变，所以可以尝试进行第二次 `free`。

2. 第二次 `DeviceIoControl`, `IoControlCode = 0x120C3`

`afd.sys` 调用 `AfdTransmitPackets`，`AfdTransmitPackets` 内部调用 `AfdTliGetTpInfo` 获得一个 `TpInfo` 结构体，而这块内存正好就是从 `Lookaside` 中申请的，之后因为用户控制了 `AfdTliGetTpInfo` 的参数，导致申请过大的内存，使程序进入异常处理流程，调用 `AfdReturnToInfo`，并且参数正是第一次调用时的 `tpinfo`，所以流程会跟第一次调用一样，释放 `mdl`，此时为第二次 `free`，也就是著名的 `double free`

3. `POC` 中的 `socket` 句柄

根据分析可知，`poc` 工作流程中有很多次对文件对象 `Fscontext` 域的判断，分别有 `1AFD AFD2 AFD1`，这里可以看出，都是跟驱动 `afd` 名字相关的值，所以可以推测，`poc` 中使用 `connect` 后的 `socket` 句柄，不使用符号链接的句柄，正是出于此原因，由此可见外国人对内核的了解是特别深的

好的，到此 `POC` 的工作流程与漏洞原理分析完毕，接下来思考一下如何把 `double free` 转为 `use after free`

# 漏洞利用思路与分析

## 流程

这一段参考了 `Vsbat` 的文章，据他文章所述，他也是参考外文的 `PDF`

1. 调用 `DeviceIoControl`， `IoControlCode = 0x1207F`, 造成一次 `MDL free`

2. 创建某个对象，使得这个对象恰好占据刚才被 `free` 掉的空间，至此转化 `double-free` 为 `use-after-free` 问题

3. 调用 `DeviceIoControl`, `IoControlCode =0x120C3`, 走入重复释放流程，释放掉刚才新申请的对象

4. 被释放的对象应该被设置为可控数据，也就是伪造一个对象，此时我们并没有释放这个对象，我们可以继续操作这个对象

5. 尝试调用能够操作此对象的函数，让函数通过操作我们刚刚覆盖的可控数据，实现一个内核内存写操作，这个写操作最理想的就是任意地址写任意内容，这样我们就可以覆写 `HalDispatchTable` 的某个单元为我们 `ShellCode` 的地址，这样就可以劫持一个内核函数调用

6. 用户层触发刚刚被 `Hook` 的 `HalDispatchTable` 函数，使得内核执行 `shellcode`，进行提权

## 方法

此时就出现了一个问题，我们应该用什么样的对象来覆盖这块内存，这个对象应该有两个要求

1. 这个对象的大小要等于第一次被释放的内存的大小

2. 这个对象应该有这样一个操作函数，这个函数能够操作我们的恶意数据，使得我们间接实现任意地址写任意内容

对于 `1` 来说，大小不是问题，因为第一次申请的 `mdl` 的 `va` 与 `length` 都是我们可以控制的

对于 `2` 来说，这个对象去哪里找，外文 `pdf` 中提到了 `WorkerFactory`

此时又出现了问题

1. `WorkerFactory` 对象的大小是多少（`NtCreateWorkerFactory`）

2. 如何覆盖掉被释放的数据，根据最开始的蓝屏分析，被释放的地址为非分页地址，而用户层没办法分配非分页地址，所以我们不得不找到一个函数来在内核申请非分页内存并把 `shllcode` copy 到这里，去哪找这个函数？（外文 `pdf` 中提到了 `NtQueryEaFile`）
