
在查看一些库之前，让我们先看两个例子 - x86平台下的手动Hook和`NtSetProcessInformation`回调。

### 手动钩取（x86）

使用[Windows API Hooking](https://bbs.kanxue.com/elink@b64K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3N6%4N6%4i4K6u0W2K9i4u0W2k6q4)9J5k6i4c8W2j5h3#2Q4x3V1k6G2k6X3k6W2L8Y4y4A6N6X3g2Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9J5c8X3y4G2k6r3g2Q4x3X3c8A6L8X3A6W2j5%4c8A6L8$3&6Q4x3X3c8H3M7X3!0U0k6i4y4K6i4K6u0V1K9h3&6B7k6h3y4@1K9h3!0F1i4K6u0r3K9r3!0%4i4K6u0V1N6r3!0Q4x3X3c8Z5L8$3!0C8i4K6u0V1N6$3W2F1k6r3!0%4M7#2)9J5k6r3q4H3K9g2)9J5k6s2g2K6K9h3&6Y4i4K6u0V1j5#2)9J5b7W2)9J5b7R3`.`.)作为x86示例（更容易演示），我们可以将代码调整为类似以下的形式：

```
#include <windows.h>
#include <stdio.h>
#define BYTES_REQUIRED 6
int __stdcall HookedMessageBoxA(HWND hWnd, LPCSTR lpText, LPCSTR lpCaption, UINT uType)
{
printf("\n[ HOOKED MESSAGEBOXA ]\n");
printf("-> Arguments:\n");
printf("  1. lpText: %s\n", lpText);
printf("  2. lpCaption: %s\n", lpCaption);
printf("  3. uType: %ld\n", uType);
return 1;
}

void PrintHexA(char* data, int sz)
{
printf("  -> ");
for (int i = 0; i < sz; i++)
{
printf("\\x%02hhX", data[i]);
}

printf("\n");
}

int main()
{

SIZE_T lpNumberOfBytesRead = 0;
HMODULE hModule = nullptr;
FARPROC pMessageBoxAFunc = nullptr;
char pMessageBoxABytes[BYTES_REQUIRED] = {};

void* pHookedMessageBoxFunc = &HookedMessageBoxA;

hModule = LoadLibraryA("user32.dll");
if (!hModule)
{
return -1;
}

pMessageBoxAFunc = GetProcAddress(hModule, "MessageBoxA");

printf("-> Original MessageBoxA: 0x%p\n", pMessageBoxAFunc);

if (ReadProcessMemory(GetCurrentProcess(), pMessageBoxAFunc, pMessageBoxABytes, BYTES_REQUIRED, &lpNumberOfBytesRead) == FALSE)
{
printf("[!] ReadProcessMemory: %ld\n", GetLastError());
return -1;
}

printf("-> MessageBoxA Hex:\n");

PrintHexA(pMessageBoxABytes, BYTES_REQUIRED);

printf("-> Hooked MessageBoxA: 0x%p\n", pHookedMessageBoxFunc);

char patch[BYTES_REQUIRED] = { 0 };
memcpy_s(patch, 1, "\x68", 1);
memcpy_s(patch + 1, 4, &pHookedMessageBoxFunc, 4);
memcpy_s(patch + 5, 1, "\xC3", 1);

printf("-> Patch Hex:\n");
PrintHexA(patch, BYTES_REQUIRED);

if (WriteProcessMemory(GetCurrentProcess(), (LPVOID)pMessageBoxAFunc, patch, sizeof(patch), &lpNumberOfBytesRead) == FALSE)
{
printf("[!] WriteProcessMemory: %ld\n", GetLastError());
return -1;
}

MessageBoxA(NULL, "AAAAA", "BBBBB", MB_OK);

return 0;
}
```

让我们逐步解释这段代码...

首先，`MessageBoxA`函数位于`User32.dll`库中，因此我们需要加载该库：

```
hModule = LoadLibraryA("user32.dll");
if (!hModule)
{
return -1;
}
```

接下来，我们需要获取`USER32!MessageBoxA`函数的地址：

```
pMessageBoxAFunc = GetProcAddress(hModule, "MessageBoxA");
```

有了这个地址，现在可以读取函数的字节：

```
if (ReadProcessMemory(GetCurrentProcess(), pMessageBoxAFunc, pMessageBoxABytes, BYTES_REQUIRED, &lpNumberOfBytesRead) == FALSE)
{
printf("[!] ReadProcessMemory: %ld\n", GetLastError());
return -1;
}
```

这将读取函数调用的前6个字节，稍后将更新这些字节以执行对新函数的`push`，从而实现`jmp`操作。

字节码:

```
\x8B\xFF\x55\x8B\xEC\x83
```

现在，需要构建补丁。可以按以下方式完成：

```
char patch[BYTES_REQUIRED] = { 0 };
memcpy_s(patch, 1, "\x68", 1);
memcpy_s(patch + 1, 4, &pHookedMessageBoxFunc, 4);
memcpy_s(patch + 5, 1, "\xC3", 1);
```

由此产生的十六进制表示为：

```
\x68\x12\x12\xBD\x00\xC3
```

使用[defuse.ca](https://bbs.kanxue.com/elink@747K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3k6r3g2X3N6i4y4W2i4K6u0W2j5$3q4Q4x3V1k6G2L8X3I4A6L8X3g2Q4x3X3c8^5z5o6k6Q4x3X3c8S2M7%4y4W2L8h3u0D9k6i4u0Q4x3X3g2Z5N6r3@1`.)进行反汇编，上述代码可以翻译为汇编语言：

```
0: 68 12 12 bd 00 push 0xbd1212
5: c3 ret
```

请注意，推送的`0x00BD1212`是我们要跳转到的函数的地址，而不是调用`USER32!MessageBoxA`：

```
void* pHookedMessageBoxFunc = &HookedMessageBoxA;
```

此时，补丁已经准备好了。它将使用一个推送指令替换前6个字节，以跳转到新地址。

接下来要做的是实际写入这个新地址：

```
if (WriteProcessMemory(GetCurrentProcess(), (LPVOID)pMessageBoxAFunc, patch, sizeof(patch), &lpNumberOfBytesRead) == FALSE)
{
printf("[!] WriteProcessMemory: %ld\n", GetLastError());
return -1;
}
```

然后，在反汇编中可以看到：

```
00BB1212 jmp HookedMessageBoxA (0BB1A80h)
```

添加了一个`jmp`指令，跳转到新的函数。允许运行后，将调用被Hook的函数，并打印出参数：

```
int __stdcall HookedMessageBoxA(HWND hWnd, LPCSTR lpText, LPCSTR lpCaption, UINT uType)
{
printf("\n[ HOOKED MESSAGEBOXA ]\n");
printf("-> Arguments:\n");
printf("  1. lpText: %s\n", lpText);
printf("  2. lpCaption: %s\n", lpCaption);
printf("  3. uType: %ld\n", uType);
return 1;
}
```

运行结果如下：

 ![图片描述](https://bbs.kanxue.com/upload/attach/202308/876202_4BY5RHTCBR4MFEK.png)

### NtSetProcessInformation回调

在[REcon 2015年的"Hooking Nirvana"（Alex Ionescu）](https://bbs.kanxue.com/elink@bb2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3N6%4N6%4i4K6u0W2P5h3!0#2N6s2g2T1k6g2)9J5k6h3y4G2L8g2)9J5c8Y4N6S2N6r3y4Z5i4K6y4r3N6W2)9K6c8s2m8t1P5g2N6&6d9o6R3H3y4s2S2q4)中，[Alex Ionescu](https://bbs.kanxue.com/elink@7bfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3N6s2N6A6N6s2c8W2M7W2)9J5k6h3y4G2L8g2)9J5c8X3q4A6L8$3&6W2M7$3y4#2)展示了如何使用带有`ProcessInstrumentationCallback`标志的[`NtSetProcessInformation`](https://bbs.kanxue.com/elink@e4aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8Q4x3@1q4Q4x3V1k6Q4x3V1k6#2L8X3c8G2j5%4g2E0k6h3&6@1k6h3c8Q4x3X3g2F1N6r3W2F1N6r3g2J5L8X3q4D9M7#2)9J5k6h3&6W2N6q4)9J5c8X3W2F1k6r3g2^5i4K6u0W2K9s2c8E0L8q4)9K6c8Y4m8S2k6$3g2Q4x3@1c8g2M7$3g2J5e0h3!0V1k6g2)9J5y4e0u0r3g2h3&6V1L8$3y4#2L8h3g2F1N6r3g2V1i4K6t1#2x3U0m8r3N6h3&6U0N6r3W2G2L8Y4y4Q4x3U0f1J5c8V1&6f1i4K6t1#2x3U0m8a6j5X3A6W2j5%4c8K6i4K6t1#2x3V1k6b7M7X3!0U0k6i4y4K6i4K6t1#2x3V1k6z5N6q4y4W2N6p5W2F1k6X3!0J5L8h3q4@1K9h3!0F1f1s2u0G2j5$3g2K6M7#2)9J5k6h3S2@1L8h3H3`.)来进行进程仪器化。在这次演讲中，Alex演示了通过回调函数进行hook的方法。

设置回调非常简单：
