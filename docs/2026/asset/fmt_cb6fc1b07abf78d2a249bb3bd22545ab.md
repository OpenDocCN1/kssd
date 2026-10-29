
# Windows 之 CRT的检测内存泄露

## 使用方法

使用方法非常简单，首先定义宏`_CRTDBG_MAP_ALLOC`，然后包含头文件`crtdbg.h`，最后在`main`函数结尾调用`_CrtDumpMemoryLeaks`统计内存申请和释放的情况。相关例子如下，编译的时候需要在Debug模式：

```c++
#define _CRTDBG_MAP_ALLOC
#include <stdlib.h>
#include <crtdbg.h>
#include <iostream>
int main()
{
std::cout << "Hello World!\n";
int* x = (int*)malloc(sizeof(int));
*x = 7;
printf("%d\n", *x);
x = (int*)calloc(3, sizeof(int));
x[0] = 7;
x[1] = 77;
x[2] = 777;
printf("%d %d %d\n", x[0], x[1], x[2]);
_CrtSetReportMode(_CRT_WARN, _CRTDBG_MODE_DEBUG);
_CrtDumpMemoryLeaks();
}
```

运行结果如下:

```c++
Detected memory leaks!
Dumping objects ->
main.cpp(16) : {163} normal block at 0x000002882AE17740, 12 bytes long.
Data: < M > 07 00 00 00 4D 00 00 00 09 03 00 00
main.cpp(10) : {162} normal block at 0x000002882AE148C0, 4 bytes long.
Data: < > 07 00 00 00
Object dump complete.
// main.cpp(x) 表示在main.cpp的x行申请的内存没有被释放
```

## CRT 检测的原理

在安装`Visual Studio`之后，Windows CRT的源码已经被存放在`C:\Program Files (x86)\Windows Kits\10\Source\`，这个目录下面有多个sdk的版本，我选择的是`19041`。

### 内存的申请

在C++编程语言中，内存申请对应的关键字是`new`或`malloc`，其实`new`最后调用的也是`malloc`函数，对应源代码文件是`debug_heap.cpp`。在包含相关头文件之后，`malloc`函数的调用栈为：`malloc` -> `_malloc_dbg` -> `heap_alloc_dbg` -> `heap_alloc_dbg_internal`。`heap_alloc_dbg_internal`函数分析如下：

1. 获取临界区，保证当前只有一个线程进入.

```c++
__acrt_lock(__acrt_heap_lock);
extern "C" void __cdecl __acrt_lock(_In_ __acrt_lock_id _Lock)
{
EnterCriticalSection(&__acrt_lock_table[_Lock]);
}
```

2. 执行自定义`malloc`的回调函数.

```c++
if (_crtBreakAlloc != -1 && request_number == _crtBreakAlloc)
{
_CrtDbgBreak();
}
if (_pfnAllocHook && !_pfnAllocHook(
_HOOK_ALLOC,
nullptr,
size,
block_use,
request_number,
reinterpret_cast<unsigned char const*>(file_name),
line_number))
{
if (file_name)
_RPTN(_CRT_WARN, "Client hook allocation failure at file %hs line %d.\n", file_name, line_number);
else
_RPT0(_CRT_WARN, "Client hook allocation failure.\n");
__leave;
}
```

`_pfnAllocHook`有一个默认的回调函数，也允许程序员自己定义，回调函数原型如下:

```c++
typedef int (__cdecl *_CRT_ALLOC_HOOK)(
int const allocation_type,
void* const data,
size_t const size,
int const block_use,
long const request,
unsigned char const* const file_name,
int const line_number
);
```

设置回调函数的接口为`_CrtSetAllocHook`.

3. 调用Windows API分配内存，不过需要多分配一些冗余内存，记录一些信息，用于管理`malloc`分配的内存。

管理的数据结构如下：

```c++
struct _CrtMemBlockHeader
{
_CrtMemBlockHeader* _block_header_next; //
_CrtMemBlockHeader* _block_header_prev; // 双向链表，访问该双向链表的全局变量为__acrt_first_block
char const* _file_name; // 调用malloc的文件名
int _line_number; // 调用malloc的行数
int _block_use; // 内存类型，所有内存类型如下
/*
#define _FREE_BLOCK 0 // 内存释放
#define _NORMAL_BLOCK 1 // 内存申请
#define _CRT_BLOCK 2 // 标注CRT库申请的内存
#define _IGNORE_BLOCK 3 // 此类内存不进行管理
#define _CLIENT_BLOCK 4 // 暂时没找到用法
#define _MAX_BLOCKS 5 // 暂时没找到用法
*/
size_t _data_size; // malloc分配的大小
long _request_number; // 记录分配内存的序号，每次分配内存自增1
unsigned char _gap[no_mans_land_size]; // 标记
// Followed by:
// unsigned char _data[_data_size]; // malloc返回的内存
// unsigned char _another_gap[no_mans_land_size]; // 标记
};
```

结构中成员`_gap`填充了`no_mans_land_size`(4)个`0xFD`，在释放内存时检测写内存时是否出现溢出(上溢)。该结构后续的内容是`malloc`返回的内存，内存中被填充了`0xCD`。最后内存`_another_gap`也是填充了`no_mans_land_size`(4)个`0xFD`，在释放内存时检测写内存时是否出现溢出(下溢)。

### 内存的扩容

在C++编程语言中，内存扩容的关键字为`realloc`，对应的源文件是`realloc.cpp`，`realloc`函数的调用栈为：`realloc` -> `_realloc_dbg` -> `realloc_dbg_nolock`。该函数的函数原型如下：

```c++
static void * __cdecl realloc_dbg_nolock(
void* const block,
size_t* const new_size,
int const block_use,
char const* const file_name,
int const line_number,
bool const reallocation_is_allowed
) throw()
```

1. 检查`block`和`new_size`的情况.

```c++
if (!block) // block为nullptr，蜕变为malloc(size)
{
return _malloc_dbg(*new_size, block_use, file_name, line_number);
}
if (reallocation_is_allowed && *new_size == 0) // *new_size为0，则蜕变为free(block)
{
_free_dbg(block, block_use);
return nullptr;
}
```

2. 调用`_pfnAllocHook`回调函数，参数`allocation_type`为`_HOOK_REALLOC`。

```c++
if (_pfnAllocHook && !_pfnAllocHook(
_HOOK_REALLOC,
block,
*new_size,
block_use,
request_number,
reinterpret_cast<unsigned char const*>(file_name),
line_number))
{
if (file_name)
_RPTN(_CRT_WARN, "Client hook re-allocation failure at file %hs line %d.\n", file_name, line_number);
else
_RPT0(_CRT_WARN, "Client hook re-allocation failure.\n");
return nullptr;
}
```

3. 对`block`进行一系列检查

```c++
is_block_an_aligned_allocation(block) // 检查block是否被_aligned_malloc分配的，若是，则返回nullptr.
_ASSERTE(_CrtIsValidHeapPointer(block)); // 保证block内存属于进程堆
检查 -> 堆的类型是否是_IGNORE_BLOCK
检查 -> block的header的_data_size是否被破坏
检查 -> *new_size 是否过大
// Ensure the new requested size is not too large:
if (*new_size > static_cast<size_t>(_HEAP_MAXREQ - no_mans_land_size - sizeof(_CrtMemBlockHeader)))
{
errno = ENOMEM;
return nullptr;
}
```

4. 分配一个新的`_CrtMemBlockHeader`结构

```c++
size_t const new_internal_size{sizeof(_CrtMemBlockHeader) + *new_size + no_mans_land_size};
_CrtMemBlockHeader* new_head{nullptr};
new_head = static_cast<_CrtMemBlockHeader*>(_realloc_base(old_head, new_internal_size));
// _realloc_base中调用HeapReAlloc函数重新分配
```

5. 对新分配的内存初始化

```c++
// If the block grew, fill the "extension" with the land fill value:
if (*new_size > new_head->_data_size) // *new_size 新的内存大于原来的
{
memset(new_block + new_head->_data_size, clean_land_fill, *new_size - new_head->_data_size);
}
// Fill in the gap after the client block:
memset(new_block + *new_size, no_mans_land_fill, no_mans_land_size); // 填充向下溢出的标记
```

6. 将新的`header`链接到双向链表中

```c++
// 删除原来的元素
new_head->_block_header_prev->_block_header_next = new_head->_block_header_next;
new_head->_block_header_prev->_block_header_next = new_head->_block_header_next;
// 替换__acrt_first_block指向的元素
__acrt_first_block->_block_header_prev = new_head;
new_head->_block_header_next = __acrt_first_block;
new_head->_block_header_prev = nullptr;
__acrt_first_block = new_head;
```

### 内存的释放

在C++编程语言中，内存释放对应的关键字是`delete`或`free`，`delete`操作符最后调用到`free`函数，对应的源文件是`debug_heap.cpp`。

`free`函数的调用栈为：`free` -> `_free_dbg` -> `free_dbg_nolock`，`_free_dbg`函数会获取临界区然后调用`free_dbg_nolock`。`free_dbg_nolock`函数分析过程如下：

1. 释放内存时`block_use`为`_FREE_BLOCK`，若此时`block_use`为`_NORMAL_BLOCK`，且`block`由`_aligned_malloc`分配，不进行内存释放。

```c++
// Check to ensure that the block was not allocated by _aligned routines
if (block_use == _NORMAL_BLOCK && is_block_an_aligned_allocation(block))
{
// We don't know (yet) where (file, linenum) block was allocated
_RPTN(_CRT_ERROR, "The Block at 0x%p was allocated by aligned routines, use _aligned_free()", block);
errno = EINVAL;
return;
}
```

2. 调用`_pfnAllocHook`，只不过`allocation_type`换成了`_HOOK_FREE`。

```c++
// Forced failure handling
if (_pfnAllocHook && !_pfnAllocHook(_HOOK_FREE, block, 0, block_use, 0, nullptr, 0))
{
_RPT0(_CRT_WARN, "Client hook free failure.\n");
return;
}
```

3. 进行一系列检查

```c++
_ASSERTE(_CrtIsValidHeapPointer(block)); // 保证block是从进程堆申请的
_ASSERTE(is_block_type_valid(header->_block_use)); // 保证block的堆类型是正常的
_ASSERTE(header->_block_use == block_use || header->_block_use == _CRT_BLOCK && block_use == _NORMAL_BLOCK);
// 检查之前的标记是否被破坏，被破坏意味着存在内存溢出
check_bytes(header->_gap, no_mans_land_fill, no_mans_land_size)
check_bytes(block_from_header(header) + header->_data_size, no_mans_land_fill, no_mans_land_size)
```

4. 在双向链表中，删除`block`元素，并释放内存.
