
# 调试器检测与反调试分析

## 初步观察

在走了 `N` 步，跳过了 `N` 条伪指令后，终于 `EBPSTACK` 里出现了一点貌似有意思的东西。

```
0013F78C   0013FFE0  .    ; Pointer to next SEH record  
0013F790   00436302  cC.  ; SE handler  
0013F794   0013FF98  .  
0013F798   230A53C4  S.#  
```

获取 `2` 个常量：

```
0013F6A0   00004647  GF..  
0013F784   4A4D0000  
```

接下来进入 `VM_EXIT`，我照常来一个 `F9`。

程序停下来了，当前的各种数据如下：

```
004394FA    9D              POPFD     ；int3 异常  
004394FB    60              PUSHAD  
```

寄存器状态：
- `EAX` `00000000`
- `ECX` `0000E39D`
- `EDX` `BFEBFBFF`
- `EBX` `00000000`
- `ESP` `0013F788`
- `EBP` `0013FF98`
- `ESI` `00004647`
- `EDI` `00004A4D`

内存数据：
```
0013F78C   0013FFE0  .    ; Pointer to next SEH record  
0013F790   00436302  cC.  ; SE handler  
```

还在检测 `Softice`，真不知道如果我不装它，能够少多少麻烦！看了半天资料也没看出要怎么过，直接在 `SEH` 下断，飞过去了，貌似没有什么问题，不管它继续走。

也不知道上面有没有检测了什么东西，不管了，赶时间呀，一定要尽快跟完整个程序，反正都是检测调试器部分，要是检测到，早爆发了，没检测掉的，错过了也不管了。

接下来就是堆栈跑了老远的距离呀，然后连着来 `VM_EXIT`，慢慢的退下去了。

`FS` 寄存器又登场了：

`VM_FS:[EBPSTACK]` ；`EBPSTACK=00000000`

清除了前面设置的 `SEH`。继续刷过一大堆的伪指令。

`VM_EBPSTACKdw_MEMdw` ；获取的是 `0043D289` 的数据。然后解码出下一个函数：

```
 0013F794   7C92DCAE  |    ; ntdll.NtSetInformationThread  
```

接 `CC` 码检测。

## 理论背景：NtSetInformationThread

先来一段理论：

`NtSetInformationThread()` 的参数列表如下。要实现这一功能，`ThreadHideFromDebugger(0x11)` 被当作 `ThreadInformationClass` 参数传递，`ThreadHandle` 通常设为当前线程的句柄(`0xFFFFFFFE`)：

```c
NTSTATUS NTAPI NtSetInformationThread(
HANDLE                        ThreadHandle,
THREAD_INFORMATION_CLASS      ThreadInformaitonClass,
PVOID                         ThreadInformation,
ULONG                         ThreadInformationLength
);
```

`ThreadHideFromDebugger` 内部设置内核结构 `ETHREAD16` 的 `HideThreadFromDebugger` 成员。一旦这个成员设置以后，主要用来向调试器发送事件的内核函数 `_DbgkpSendApiMessage()` 将不再被调用。

### 示例

调用 `NtSetInformationThread()` 的一个典型示例：

```asm
push          0                               ;InformationLength  
push          NULL                            ;ThreadInformation  
push          ThreadHideFromDebugger          ;0x11  
push          0xfffffffe                      ;GetCurrentThread()  
call          [NtSetInformationThread]  
```

对应汇编执行流：

```
0013F67C   7C92DCAE  |    ; ntdll.NtSetInformationThread  
0013F680   0043EEB7  C.    ; RETURN from NOTEPAD.0043D111 to NOTEPAD.0043EEB7  

0013F684   FFFFFFFE  

0013F688   00000011  ...  

0013F68C   00000000  ....  

0013F690   00000000  ....  
```

不解释。。想想怎么过吧。

`GetCurrentProcess` 返回值是 `-1`。`GetCurrentThread` 返回值是 `-2`。这是个假的句柄，仅仅用来代表当前线程，还有当前进程用 `-1` 表示。

把 `FFFFFFFE` 改为 `FFFFFFFD`，然后把返回值 `EAX=C0000008` 改为 `00000000`。

程序回到 `VM` 中执行，解码出的下一个有用的东西是：

接着出来的东西也要贴下来。怎么说也有点用处了：

```
 0013F788   00437DF0  }C.   ; <&KERNEL32.GetModuleFileNameA>  
 0013F78C         00400000  ..@.  ;  
 0013F790   0013FB98  .    ; PTR to UNICODE "kernel32.dll"  
 0013F794   00000104  ..  
```

`VM_EBPSTACKdw_MEMdw` ；取出 `GetModuleFileNameA` 函数的地址

```
 0013F788           7C80B56F  o|    ; kernel32.GetModuleFileNameA  
 0013F78C         00400000  ..@.  ; OFFSET NOTEPAD.B  
 0013F790   0013FB98  .    ; PTR to UNICODE "kernel32.dll"  
 0013F794   00000104  ..  
```

`VM_APIJMP_EBPSTACK`

```c
DWORD GetModuleFileName(
    HMODULE hModule,  // handle to module to find filename for  
    LPTSTR lpFilename,  // pointer to buffer for module path  
    DWORD nSize   // size of buffer, in characters  
   );  
```

参数也贴贴。

获得了路径：

```
 0013F68C   0013FB98  .    ; ASCII "D:\\JKylin\\VMProtect Ultimate2.04\\NotePad\\NOTEPAD.exe"  
```

`EAX` 中返回长度 `00000034`。

`VM` 接着就检测一下返回的是不是 `0`，看看有没有执行成功。

你以为检测完了，实际还没有完，我们还在路上。

## CreateFileA 检测

下一个函数是它：

```
 0013F69C   7C801A28  (|   ; kernel32.CreateFileA  
 0013F778   7C801A28  (|   ; kernel32.CreateFileA  
 0013F77C   0013FB98  .    ; ASCII "D:\\JKylin\\VMProtect Ultimate2.04\\NotePad\\NOTEPAD.exe"  
 0013F780   80000000  ...  
 0013F784   00000003  ...  
 0013F788   00000000  ....  
 0013F78C   00000003  ...  
 0013F790   00000080  ...  
 0013F794   00000000  ....  
```

`VM_APIJMP_EBPSTACK`

打开自身来看看有没有调试器。

```c
HANDLE CreateFile(
    LPCTSTR lpFileName,        // pointer to name of the file  
    DWORD dwDesiredAccess,     // access (read-write) mode  
    DWORD dwShareMode,        // share mode  
    LPSECURITY_ATTRIBUTES lpSecurityAttributes,  // pointer to security attributes  
    DWORD dwCreationDistribution,     // how to create  
    DWORD dwFlagsAndAttributes,     // file attributes  
    HANDLE hTemplateFile       // handle to file with attributes to copy   
   );  
```

我决定玩玩这样，我在路径里放上了 `win98` 的原始记事本，命名为 `NOTEPAC.exe`。然后修改函数的参数为 `NOTEPAC.exe`。

返回了 `0000006C`，看看 `VMP` 到底只是比较一下就过了，还是他要做其他的事情。

`VMP` 检测返回值是否为 `0`，肯定不为洒，我已经改了。

果然 `VMP` 没有那么轻易放过这个句柄，早知道我应该放上加壳记事本的复本，算了，到后面改跳转好了。

## GetFileSize 检测

下一个函数是：`GetFileSize`。

接着我们就来到了这里：

```
 0013F78C   7C810B17  |   ; kernel32.GetFileSize  
 0013F790   0000006C  l...  
 0013F794   00000000  ....  
```

```c
DWORD GetFileSize(
    HANDLE hFile,  // handle of file to get size of
    LPDWORD lpFileSizeHigh   // address of high-order word for file size
   );  
```

还是贴贴函数参数比较人性化，`^\_^`，可惜前面的没有贴。

得到的大小是 `EAX=0000D000`。

毫无疑问，我相信 `VMP` 肯定已经提前计算好了文件值，现在要看看是不是一样了。

`VM_EBPSTACKdw_MEMdw` ；估摸着这里得到的应该就加密的原程序大小了

```
  0013F794   A5B56CFF  l  
```

解密解密：

```
  0013F794   F48003FF  
```

`0043DE51` 地址，出来一个新的伪指令：

```
  0013F78C   03FF  
  0013F790   03FFF480  
  0013F794   000AF480  ..  
```

相关指令：

```
0043DE5D    8B45 00         MOV EAX,DWORD PTR SS:[EBP]               ; *
0043DE69    8B55 04         MOV EDX,DWORD PTR SS:[EBP+4]             ; *
0043DE6E    8A4D 08         MOV CL,BYTE PTR SS:[EBP+8]               ; *
0043DE7A    0FADD0          SHRD EAX,EDX,CL                          ; *
0043D38F    8945 04         MOV DWORD PTR SS:[EBP+4],EAX             ; *
```

非常清晰，这个伪指令就是读取 `EBPSTACK` 的数据，然后移位，我把它命名为 `VM_SHRw_EBPSTACK`。

`F48003FF` 解密成了 `FFFD2000`。

继续解密解密，貌似这里贴的太细了。早知道直接贴结果：

```
0013F794   0002E000  ..    ;这个应该是结果了  
```

果然接下来就是比较大小是否一致，`EBPSTACK` 和 `EDISTACK` 中暂存的所有 `0000D000` 都修改为 `0002E000`。

```
  0013F790   0002E000  ..      ；这里是修改的  
  0013F794   0002E000  ..  
```

检测是否相同，跳转地址为 `004259B3`。

## CreateFileMappingA 检测

下面解密出场的函数是：`CreateFileMappingA`

```
  0013F77C   7C80950A  .|    ; kernel32.CreateFileMappingA  
  0013F780   0000006C  l...  
  0013F784   00000000  ....  
  0013F788   00000002  ...  
  0013F78C   00000000  ....  
  0013F790   00000000  ....  
  0013F794   00000000  ....  
```

```c
HANDLE CreateFileMapping(
    HANDLE hFile,            // handle to file to map  
    LPSECURITY_ATTRIBUTES lpFileMappingAttributes,  // optional security attributes  
    DWORD flProtect,         // protection for mapping object  
    DWORD dwMaximumSizeHigh,        // high-order 32 bits of object size   
    DWORD dwMaximumSizeLow,        // low-order 32 bits of object size   
    LPCTSTR lpName           // name of file-mapping object  
   );  
```

这个玩笑开大了，刚才真的应该放一个加壳记事本的复本呀。

函数调用完后，`EAX` 返回的句柄是 `00000070`。

`VM` 检测返回值是否为 `0`。

很多套路都是一样的，每个函数解密后必定要看看是否为 `0`，接着 `CC` 码检测，每个函数完成后，都是检测返回值，然后跳转到相应的位置，然后又做点事，又跳转到另外一个地方。

这次的跳转地址是：`00427D2A`，其实根本没有用处，随便写写记记。`⊙⊙b` 汗。

我用 `0D2.0` 无插件又装着 `SoftIce`，开着程序就不敢关闭呀。重装一次壳程序，就要重新手动跑到原来的地方。超级麻烦。

大量的放入函数参数，都是以明码常量放入的，这么多参数不可能 `VMP` 每个都去加密洒。函数倒是都是加密的洒。

## MapViewOfFile 检测

废话少说，下一个解密函数是：`MapViewOfFile`

```
  0013F780   7C80B9A5  |     ; kernel32.MapViewOfFile  
  0013F784   00000070  p...  
  0013F788   00000004  ...  
  0013F78C   00000000  ....  
  0013F790   00000000  ....  
  0013F794   00000000  ....  
```

```c
LPVOID MapViewOfFile(
    HANDLE hFileMappingObject,    // file-mapping object to map into address space   
    DWORD dwDesiredAccess,    // access mode  
    DWORD dwFileOffsetHigh,    // high-order 32 bits of file offset  
    DWORD dwFileOffsetLow,    // low-order 32 bits of file offset  
    DWORD dwNumberOfBytesToMap   // number of bytes to map  
   );  
```

函数执行完毕，返回的映射地址是 `00A00000`，不管三七二十一先在数据窗口里面跟踪着 `00A00000`，同时开了 `PEID` 把加壳记事本的程序也打开，`Hex Viewer` 打开，先做点准备工作，如果它要搞什么比较之类的东西我也好看着点，谁叫我当时放了不同的程序给它打开。

`VM` 照例先检测一下返回值是否为 `0` 并跳转：`0043BE55`。

哗哗哗的跳过了一些内容,后面发现悲剧了。

`VM` 算了 `120`，然后加到了映射地址。现在 `00A00000+120=00A00120`，我们还是来理理 `PE` 文件格式，看看 `120` 是 `PE` 中的什么地方。

`0043CE89`，哦哦，有新的伪指令出现。里面的重要的一条指令就是：
