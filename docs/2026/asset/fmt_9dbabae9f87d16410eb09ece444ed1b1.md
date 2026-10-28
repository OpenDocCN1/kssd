
```cpp
#include "detours.h"
int inline_hooks(PVOID pfnOlds[], PVOID pfnNews[])
{
    int i=0;
    DetourRestoreAfterWith();
    DetourTransactionBegin();
    DetourUpdateThread(GetCurrentThread());
    for(i=0; pfnNews[i]!=NULL ;i++)
        DetourAttach(&pfnOlds[i], pfnNews[i]);
    DetourTransactionCommit();
    return i;
}
```

### LoadDLL

上述hook代码编译成的载体是DLL，我们还需要把此DLL注入目标到exe中，接管某些函数改变其功能。

有三种常用方法：

1. 在exe的导入表中静态添加DLL

2. `code cave`进行`LoadLibrayA`DLL

3. `VirtualAllocEx`、`WriteProcessMemory`和`CreateRemoteThread`来动态注入DLL。

代码如下，详见我的github[`injectdll.py`](https://bbs.kanxue.com/elink@856K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6k6N6i4u0A6f1$3W2*7N6h3E0#2i4K6u0r3c8$3q4D9k6$3q4E0k6g2u0W2N6X3g2J5M7$3g2Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7$3y4J5K9i4m8@1i4K6u0r3N6$3W2F1k6r3!0%4M7#2)9J5c8X3W2F1K9X3g2U0N6r3c8D9L8q4)9J5k6i4m8&6), [`win_hook,c`](https://bbs.kanxue.com/elink@4c5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6k6N6i4u0A6f1$3W2*7N6h3E0#2i4K6u0r3c8$3q4D9k6$3q4E0k6g2u0W2N6X3g2J5M7$3g2Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7$3y4J5K9i4m8@1i4K6u0r3N6$3W2F1k6r3!0%4M7#2)9J5c8Y4N6A6L8W2)9#2k6X3S2G2L8$3E0Q4x3X3g2U0)

```python
import lief
def injectdll(exepath, dllpath, outpath="out.exe"):
    # can not be ASLR
    binary_exe = lief.parse(exepath)
    binary_dll = lief.parse(dllpath)

    dllname = os.path.basename(dllpath)
    dll_imp = binary_exe.add_library(dllname)
    print("the import dll in " + exepath)
    for imp in binary_exe.imports:
        print(imp.name)

    for exp_func in binary_dll.exported_functions:
        dll_imp.add_entry(exp_func.name)
        print(dllname + ", func "+exp_func.name + " added!")

    # disable ASLR
    exe_oph = binary_exe.optional_header;
    exe_oph.remove(lief.PE.DLL_CHARACTERISTICS.DYNAMIC_BASE)

    builder = lief.PE.Builder(binary_exe)
    builder.build_imports(True).patch_imports(True)
    builder.build()
    builder.write(outpath)
```

```cpp
BOOL inject_dll(HANDLE hProcess, LPCSTR dllname)
{
    LPVOID param_addr = VirtualAllocEx(hProcess, 0, 0x100, MEM_COMMIT, PAGE_READWRITE);
    SIZE_T count;
    if (param_addr == NULL) return FALSE;
    WriteProcessMemory(hProcess, param_addr, dllname, strlen(dllname)+1, &count);

    HMODULE kernel = GetModuleHandleA("Kernel32");
    FARPROC pfnLoadlibraryA = GetProcAddress(kernel, "LoadLibraryA");
    HANDLE threadHandle = CreateRemoteThread(hProcess, NULL, NULL, (LPTHREAD_START_ROUTINE)pfnLoadlibraryA, param_addr, NULL, NULL);

    if (threadHandle == NULL) return FALSE;
    WaitForSingleObject(threadHandle, -1);
    VirtualFreeEx(hProcess, param_addr, 0x100, MEM_COMMIT);

    return TRUE;
}
```

## 0x5 内联汇编与C编写动态汉化程序

到此，主要问题我们都搞清楚了，现在可以愉快地编写动态汉化程序了。动态汉化程序主要包括下面几个部分：

- `Inlinehook`处汇编环境与C语言函数的对接, **注意`cdecl`和`stdcall`，汇编调用C函数要自己保存寄存器**

- 维护日文文本与汉化文本的对应关系，文本偏移的定位等数据。并且用二分法等算法来查找替换文本等。

- 编码和字体的hook，以使其适配汉语gb2312编码等(如`CreateFontIndirectA`，改变charset）

### 文本显示Inline hook

这里采取的`__declspec(naked)`形式进行内联汇编，进行获取当前文本指针、计算在文件中的偏移、调用相应的C函数查找字符串、替换汉化文本等操作。此处为了方便使用了一些全局变量，以`g_`前缀开头。

```cpp
void* g_base = (void*)0x400000; // app base addr
void* g_showtext = (void*)0x442760; // replaced text buffer
PMJO_NODE g_mjos=NULL, g_cur_mjo=NULL; // pointer to index structure
char g_textbuf[2048] = {0}; // for showing replaced text

__declspec(naked) void showtext_hook() // replace text to chs, inline hook code
{
    __asm{
        pushad
        mov ecx, g_base
        add ecx, 0xdc350
        mov ecx, dword ptr ds:[ecx] ;mjo struct
        push ecx ;because the function might change the register
        push ecx ;mjo_name
        call search_mjo_ftexts
        pop ecx ;restore ecx for mjo struct
        lea eax, [ecx+29h*4]
        mov eax, dword ptr ds:[eax] ;mjo_addr_base
        mov ebx, g_base
        add ebx, 5D1A58h - 400000h
        mov ebx, dword ptr ds:[ebx]
        mov ebx, dword ptr ds:[ebx] ;mjo_addr_cur

        inc ebx
        loop1: ; do while
        dec ebx
        cmp byte ptr[ebx], 0
        jne loop1

        loop2:
        dec ebx
        cmp byte ptr[ebx], 0
        jne loop2
        inc ebx

        sub ebx, eax
        push ebx
        call find_mjo_chstext
        lea esi, g_textbuf
        cmp byte ptr [esi], 0 ; if g_textbuf is empty, just use origin buffer
        je leave
        mov edi, g_base
        add edi, 52CCE4h - 400000h
        mov edi, dword ptr ds:[edi] ;text_addr

        replace_text:
        mov al, byte ptr [esi]
        mov byte ptr [edi], al
        test al, al
        jz leave
        inc esi
        inc edi
        jmp replace_text

        leave:
        popad
        jmp dword ptr ds:[g_showtext]
    }
}

void install_text_hook()
{
    // inline hook for replace text
    PVOID pfnOlds[3] = {g_base+0x42820, g_base+0x7EE00, NULL};
    PVOID pfnNews[3] = {showtext_hook, is_twobyte, NULL};
    printf("Before inline hooks\n");
    for(int i=0;i<sizeof(pfnOlds)/sizeof(PVOID)-1;i++)
    {
        printf("%d, %lx -> %lx\n", i, (unsigned long)pfnOlds[i], (unsigned long)pfnNews[i]);
    }
    inline_hooks(pfnOlds, pfnNews);
    g_showtext = pfnOlds[0];
    printf("After inline hooks\n");
    for(int i=0;i<sizeof(pfnOlds)/sizeof(PVOID)-1;i++)
    {
        printf("%d, %lx -> %lx\n", i, (unsigned long)pfnOlds[i], (unsigned long)pfnNews[i]);
    }
}
```

### 查找对应中文文本

此处用双向链表数据结构来存储文件名与文本项索引，`g_mjos`全局变量来指向索引链表，`g_cur_mjo`指向当前文本索引位置。当游戏加载脚本时会查询当前链表中是否已经加载过，可以避免重复加载造成的内存泄露。`PFTEXTS`数据结构详见我的通用汉化文本格式，[`binary_text.h`](https://bbs.kanxue.com/elink@e34K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6k6N6i4u0A6f1$3W2*7N6h3E0#2i4K6u0r3c8$3q4D9k6$3q4E0k6g2u0W2N6X3g2J5M7$3g2Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7$3y4J5K9i4m8@1i4K6u0r3j5X3W2F1j5i4u0&6i4K6g2X3N6r3g2^5N6q4)9J5k6h3R3`.)

由于我们用的是日文和中文对照文本，因此文件用的`utf-8`格式存储，动态替换汉化文本要转换为`gb2312`格式。
