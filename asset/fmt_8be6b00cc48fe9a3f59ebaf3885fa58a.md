
成`"JMP xxxxxx"`的形式，实现hook`"CreateProcessA"`函数

6. 由于`"CreateProcessA"`的入口已改变，但病毒要实现创建线程的功能，所以跳到病毒hook`"CreateProcessA"`函数`sub_403CB0`后必须要把`"CreateProcessA"`的入口恢复

```
CODE:00402DD6             lea     eax, [ebp+NumberOfBytesWritten]

CODE:00402DD9             push    eax         ; lpNumberOfBytesWritten

CODE:00402DDA             push    edi         ; nSize

CODE:00402DDB             mov     eax, [ebp+lpBuffer]

CODE:00402DDE             push    eax         ; lpBuffer  ；原来保存

CODE:00402DDF             push    esi         ; lpBaseAddress ；

CODE:00402DE0             push    ebx         ; hProcess

CODE:00402DE1             call    WriteProcessMemory
```

7. 通过`CreateProcessA`创建进程

```
CODE:00403CD4             push    ebx

CODE:00403CD5             mov     eax, [ebp+arg_20]

......

......

CODE:00403CF2             mov     eax, [ebp+arg_0]

CODE:00403CF5             push    eax

CODE:00403CF6             call    ds:dword_40AFBC      

; dword_40AFBC 为“CreateProcessA”的入口地址
```

8. 实现DLL注入的方法：在游戏进程里申请一块内存并把DLL名写进去，把进程空间中的`LoadLibrary`函数当成线程来执行，输入的参数是前面申请的地址（已把DLL名写进去），这样`LoadLibrary`函数执行到`ret`时，远程线程结束，但DLL也被载入目标进程，这样就可执行DLL中的代码

```
CODE:00402E66             push    eax         ; dwProcessId  ;当前进程

CODE:00402E67             push    0           ; bInheritHandle

CODE:00402E69             push    1F0FFFh     ; dwDesiredAccess

CODE:00402E6E             call    OpenProcess

CODE:00402E73             mov     ebx, eax

CODE:00402E75             push    4           ; flProtect

CODE:00402E77             push    1000h       ; flAllocationType

CODE:00402E7C             push    104h        ; dwSize

CODE:00402E81             push    0           ; lpAddress

CODE:00402E83             push    ebx         ; hProcess

CODE:00402E84             call    VirtualAllocEx

CODE:00402E89             mov     esi, eax

CODE:00402E8B             push    esp         ; lpNumberOfBytesWritten

CODE:00402E8C             push    104h        ; nSize

CODE:00402E91             push    edi         ; lpBuffer

CODE:00402E92             push    esi         ; lpBaseAddress ；申请 的内存空间

CODE:00402E93             push    ebx         ; hProcess

CODE:00402E94             call    WriteProcessMemory

CODE:00402E99             push    offset aLoadlibrarya ; "LoadLibraryA"

CODE:00402E9E             push    offset aKernel32_dll ; "kernel32.dll"

CODE:00402EA3             call    GetModuleHandleA

CODE:00402EA8             push    eax         ; hModule

CODE:00402EA9             call    GetProcAddress ；获取"LoadLibraryA"的地址

CODE:00402EAE             push    esp         ; lpThreadId

CODE:00402EAF             push    0           ; dwCreationFlags

CODE:00402EB1             push    esi         ; lpParameter，前面申请的地 址

CODE:00402EB2             push    eax         ; lpStartAddress，"LoadLibraryA"的地址

CODE:00402EB3             push    0           ; dwStackSize

CODE:00402EB5             push    0           ; lpThreadAttributes

CODE:00402EB7             push    ebx         ; hProcess

CODE:00402EB8             call    CreateRemoteThread
```

### 二. 搜索游戏进程匹配特征码

```
CODE:0040401D             lea     ebx, [eax+1000h]    ；eax

CODE:00404023             lea     esi, [eax+200000h]

CODE:00404029             push    0Ch  ；获取页面或读出内容的最小值

CODE:0040402B             push    ebx  ；地址下限

CODE:0040402C             push    esi  ；地址上限

CODE:0040402D             mov     ecx, offset unk_4060C0；特征码地址

CODE:00404032             xor     edx, edx  ；识别控制

CODE:00404034             mov     eax, [edi]；进程句柄

CODE:00404036             call    findGameAddr

........

........

CODE:0040410D             mov     eax, [ebp+ThreadId]

CODE:00404110             mov     ds:dword_40AFF0, eax

CODE:00404115             push    1           ; dwMilliseconds

CODE:00404117             call    Sleep
```

核心函数就是`findGameAddr`，以`CODE:00404036`的为例说明

1. 首先用`[eax+1000h]`作为搜索的起始地址，循环查找时的下次起始地址就是用`VirtualQueryEx`获得的`MEMORY_BASIC_INFORMATION`结构中的`BaseAddress+RegionSize`

```
CODE:00402F48             mov     eax, [ebp+Buffer.BaseAddress]

CODE:00402F4E             add     eax, [ebp+Buffer.RegionSize]

CODE:00402F54             mov     [ebp+lpBaseAddress], eax

CODE:00402F57             mov     eax, [ebp+lpBaseAddress]

CODE:00402F5A             cmp     eax, [ebp+arg_4]

CODE:00402F5D             jnb     short loc_402F65

CODE:00402F5F             mov     eax, [ebp+arg_4]；  [ebp+arg_4]=

CODE:00402F62             mov     [ebp+lpBaseAddress], eax
```

2. 限制查找地址的范围在`[eax+1000h]`和`[eax+200000h]`之间

```
mov     eax, [ebp+lpBaseAddress]；[ebp+lpBaseAddress]查找的开始地址

cmp     eax, [ebp+arg_0]  ；[ebp+arg_0]=[eax+200000h]

ja      loc_403060    ；跳到结束
```

3. 用`VirtualQueryEx`获得相关的内存信息

```
push    1Ch           ; dwLength

lea     eax, [ebp+Buffer]

push    eax           ; lpBuffer，指向 MEMORY_BASIC_INFORMATION 结构

mov     eax, [ebp+lpBaseAddress]

push    eax           ; lpAddress，查找的开始地址

mov     eax, [ebp+hProcess]

push    eax           ; hProcess

call    VirtualQueryEx
```

4. 检查相同属性的页面的范围是否少于传入的参数`0Ch`，如果小于的话就跳到1.

继续查找

```
CODE:00402F8F             mov     eax, [ebp+Buffer.RegionSize]

CODE:00402F95             mov     [ebp+var_14], eax

CODE:00402F98

CODE:00402F98 loc_402F98:                       ; CODE XREF: 

CODE:00402F98             findGameAddr+121 j

CODE:00402F98                       ; findGameAddr+16F j

CODE:00402F98             mov     eax, [ebp+var_14]

CODE:00402F9B             cmp     eax, [ebp+arg_8]

CODE:00402F9E             jb      short loc_402F48
```

5. 读取内存页面到指定的缓冲区

```
CODE:00402FD5             lea     eax, [ebp+NumberOfBytesRead]

CODE:00402FD8             push    eax         ; lpNumberOfBytesRead

CODE:00402FD9             push    ebx         ; nSize，为 MEMORY_BASIC_INFORMATION 结构中的 RegionSize 和 19000h 两者之间的最小值

CODE:00402FDA             lea     eax, [ebp+var_19438]  ；缓冲区地址

CODE:00402FE0             push    eax         ; lpBuffer

CODE:00402FE1             mov     eax, [ebp+lpBaseAddress]

CODE:00402FE4             push    eax         ; lpBaseAddress

CODE:00402FE5             mov     eax, [ebp+hProcess]

CODE:00402FE8             push    eax         ; hProcess

CODE:00402FE9             call    ReadProcessMemory

CODE:00402FEE             mov     eax, [ebp+NumberOfBytesRead]

CODE:00402FF1             cmp     eax, [ebp+arg_8]  ；读到的实际大

CODE:00402FF4             jb      loc_402F48
```

6. 根据特征码匹配内存，看`[ebp+lpBaseAddress]`是否为对应的内存地址，不是的话跳到1.重新查找，直到查到内存边界`[eax+200000h]`为止

```
CODE:00403022             lea     eax, [ebp+var_41C]  ；[ebp+var_41C]

CODE:00403028

CODE:00403028 loc_403028:                       ; CODE XREF: findGameAddr+151 j

CODE:00403028             lea     ebx, [edi+esi]

CODE:0040302B             mov     bl, [ebp+ebx+var_19438]  ；

CODE:00403032             cmp     bl, [eax]    ；比较数据是否

CODE:00403034             jz      short loc_40303A

CODE:00403036             xor     ecx, ecx

CODE:00403038             jmp     short loc_40303F

CODE:0040303A ; ---------------------------------------------------------------------

CODE:0040303A
```
