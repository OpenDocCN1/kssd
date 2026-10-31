
# [原创][翻译]规避技术： 拦截

备注

原文地址：[82cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3q4K6K9h3!0F1M7#2)9J5k6h3y4Z5k6h3y4C8M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3y4Z5L8X3W2I4N6h3g2K6i4K6u0r3K9r3!0G2K9%4y4Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@26dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3q4K6K9h3!0F1M7#2)9J5k6h3y4Z5k6h3y4C8M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3y4Z5L8X3W2I4N6h3g2K6i4K6u0r3K9r3!0G2K9%4y4Q4x3X3g2Z5N6r3#2D9)

原文标题：Evasions: Hooks

更新日期：2021年6月1日

此文后期：根据自身所学进行内容扩充

因自身技术有限，只能尽自身所能翻译国外技术文章，供大家学习，若有不当或可完善的地方，希望可以指出，用于共同完善这篇文章。

![](https://bbs.kanxue.com/upload/attach/202106/767964_Z745FYXMFHTHMS3.jpg)

目录

- 拦截检测方法
  - 1. 检查系统功能中是否设置了拦截
  - 2. 通过鼠标拦截检查用户的点击中是否存在特定字符串
    - 2.1. Windows Vista+
- 识别标志
- 反制措施
- 归功于

## 拦截检测方法

这里描述的技术利用拦截来检测用户的存在，或者作为检查是否安装了一些不寻常的主机操作系统拦截的手段。

### 1. 检查系统功能中是否设置了拦截

恶意软件在特定地址读取内存，以检查Windows API函数是否被拦截。

这种方法是基于这样一个事实，即仿真环境最有可能钩住这些功能，以便能够在伪装期间收集数据和统计数据。

要检查的常用函数:

- `ReadFile`
- `DeleteFile`
- `CreateProcessA/W`

通过以下函数读取内存:

- `ReadProcessMemory`
- `NtReadVirtualMemory`

然后可以使用不同的算法进行检查：

- 将前两个字节与`\x8B\xFF`（`mov edi，edi`）进行比较 — `kernel32`函数的典型出现方式。
- 将前N个字节与`\xCC`-软件断点（`int 3`）进行比较，该断点未直接与拦截连接，但仍有可疑行为。
- 将前N个字节与`\xE9`（调用）或`\xEB`（`jmp`指令）进行比较-用于重定向执行的典型指令。
- 正在检查执行重定向的`push/ret`组合

  等等

要计算每一种可能的比较是非常困难的，所以应用程序行为中不正常的一般迹象是读取操作系统库所在的内存。如果更准确地说：读取 "有趣 "的函数所在的内存。

这篇文章解释了如何检测用户模式下的钩子并移除它们。下面的代码样本取自这篇[文章](https://bbs.kanxue.com/elink@fcdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5H3P5o6l9H3M7$3g2U0i4K6u0W2L8%4u0Y4i4K6u0r3N6q4)9J5c8X3c8W2k6X3g2S2N6r3W2F1k6#2)9J5k6s2g2K6k6i4u0D9j5h3&6V1i4K6u0V1K9r3!0G2K9%4y4Q4x3X3c8X3N6q4)9J5k6r3u0A6N6r3c8W2k6X3g2F1k6r3g2J5i4K6u0r3x3e0t1@1z5e0j5`.)。

检测拦截的样本：

```
HOOK_TYPE IsHooked(LPCVOID lpFuncAddress, DWORD_PTR *dwAddressOffset) {

    LPCBYTE lpBytePtr = (LPCBYTE)lpFuncAddress;



    if (lpBytePtr[0] == 0xE9) {

        *dwAddressOffset = 1;

        return HOOK_RELATIVE;    // E9 jmp is relative.

    } else if (lpBytePtr[0] == 0x68 &&  lpBytePtr[5] == 0xC3) {

        *dwAddressOffset = 1;

        return HOOK_ABOLSUTE;    // push/ret is absolute.

    }



    return HOOK_NONE;            // No hook.

}



LPVOID lpFunction = ...;

DWORD_PTR dwOffset = 0;

LPVOID dwHookAddress = 0;



HOOK_TYPE ht = IsHooked(lpFunction, &dwOffset);

if (ht == HOOK_ABSOLUTE) {

    // 1. Get the pointer to the address (lpFunction + dwOffset)

    // 2. Cast it to a DWORD pointer

    // 3. Dereference it to get the DWORD value

    // 4. Cast it to a pointer

    dwHookAddress = (LPVOID)(*(LPDWORD)((LPBYTE)lpFunction + dwOffset));

} else if (ht == HOOK_RELATIVE) {

    // 1. Get the pointer to the address (lpFunction + dwOffset)

    // 2. Cast it to an INT pointer

    // 3. Dereference it to get the INT value (this can be negative)

    INT nJumpSize = (*(PINT)((LPBYTE)lpFunction  + dwOffset);

    // 4. E9 jmp starts from the address AFTER the jmp instruction

    DWORD_PTR dwRelativeAddress = (DWORD_PTR)((LPBYTE)lpFunction + dwOffset + 4));

    // 5. Add the relative address and jump size

    dwHookAddress = (LPVOID)(dwRelativeAddress + nJumpSize);

}
```

解除拦截函数的样本：

```
// Parse the PE headers.

PIMAGE_DOS_HEADER pidh = (PIMAGE_DOS_HEADER)lpMapping;

PIMAGE_NT_HEADERS pinh = (PIMAGE_NT_HEADERS)((DWORD_PTR)lpMapping + pidh->e_lfanew);



// Walk the section headers and find the .text section.

for (WORD i = 0; i < pinh->FileHeader.NumberOfSections; i++) {

    PIMAGE_SECTION_HEADER pish = (PIMAGE_SECTION_HEADER)((DWORD_PTR)IMAGE_FIRST_SECTION(pinh) + 

                                 ((DWORD_PTR)IMAGE_SIZEOF_SECTION_HEADER * i));

    if (!strcmp(pish->Name, ".text")) {

        // Deprotect the module's memory region for write permissions.

        DWORD flProtect = ProtectMemory(

            (LPVOID)((DWORD_PTR)hModule + (DWORD_PTR)pish->VirtualAddress),    // Address to protect.

            pish->Misc.VirtualSize,                        // Size to protect.

            PAGE_EXECUTE_READWRITE                         // Desired protection.

        );



        // Replace the hooked module's .text section with the newly mapped module's.

        memcpy(

            (LPVOID)((DWORD_PTR)hModule + (DWORD_PTR)pish->VirtualAddress),

            (LPVOID)((DWORD_PTR)lpMapping + (DWORD_PTR)pish->VirtualAddress),

            pish->Misc.VirtualSize

        );



        // Reprotect the module's memory region.

        flProtect = ProtectMemory(

            (LPVOID)((DWORD_PTR)hModule + (DWORD_PTR)pish->VirtualAddress),    // Address to protect.

            pish->Misc.VirtualSize,                        // Size to protect.

            flProtect                                      // Revert to old protection.

        );

    }

}
```

### 2. 通过鼠标拦截检查用户的点击中是否存在特定字符串

这项技术由这个[链接](https://bbs.kanxue.com/elink@9b5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3K9i4u0W2k6i4W2W2i4K6u0W2j5$3!0E0i4K6u0r3j5$3!0F1N6r3g2F1N6q4)9J5c8X3c8S2L8g2)9J5c8X3k6A6M7X3g2W2P5h3g2Q4x3X3c8%4N6%4N6Q4x3V1k6U0N6i4u0J5k6h3&6@1i4K6u0V1N6r3S2J5k6h3q4@1M7#2)9J5c8Y4m8V1k6Y4y4Q4x3V1k6H3k6W2)9J5c8X3k6A6L8r3g2Q4x3V1k6X3K9i4u0W2k6i4W2W2i4K6u0V1K9r3!0@1i4K6u0V1K9$3&6A6N6X3g2K6i4K6u0V1N6r3S2J5L8%4g2Y4K9q4)9J5k6r3u0#2N6s2c8W2M7W2)9J5k6i4m8V1k6R3`.`.)描述（第4页，第7页）。

恶意软件设置鼠标拦截来检测点击（或更多）如果它发生。如果是这样的话，恶意软件会将主机视为一个常规的主机，即最终用户在屏幕后面，而不是虚拟环境。如果没有检测到鼠标点击，那么很可能是一个虚拟环境。

使用的函数：

- `SetWindowsHookExA/W` (`WH_MOUSE_LL`, ...)
- `GetAsyncKeyState`

代码样本 (`SetWindowsHookExA`)：

```
HHOOK g_hhkMouseHook = NULL;



LRESULT CALLBACK mouseHookProc(int nCode, WPARAM wParam, LPARAM lParam)

{

  switch (wParam)

  {

  case WM_MOUSEMOVE:

    // ...

    break;

  case WM_NCLBUTTONDOWN:

    // ...

    break;

  case WM_LBUTTONUP:

    UnhookWindowsHookEx(g_hhkMouseHook);

    CallMaliciousCode();

    ExitProcess(0);

  }

  return CallNextHookEx(g_hhkMouseHook, nCode, wParam, lParam);

}



g_hhkMouseHook = SetWindowsHookEx(WH_MOUSE_LL, mouseHookProc, GetModuleHandleA(NULL), NULL);
```

代码样本 (`GetAsyncKeyState`)：

```
std::thread t([]()

{

  int count = 0;

  while (true)

  {

    if (GetAsyncKeyState(VK_LBUTTON) || GetAsyncKeyState(VK_RBUTTON) || GetAsyncKeyState(VK_MBUTTON))

    {

      if (++count == 2)

        break;

    }

    Sleep(100);

  }

  CallMaliciousCode();

});

t.join();
```

## 识别标志

没有为这个规避组提供识别标志，因为很难在以某种规避技术为目标的代码和 "合法使用 "的代码之间做出区别。

## 反制措施

- 与函数拦截检查：设置内核模式钩子；第二个解决方案是使用堆栈路由来实现函数拦截。
- 与通过拦截进行的鼠标点击检查相比：使用鼠标移动伪装模块。

## 归功于

归功于[0x00sec.org](https://bbs.kanxue.com/elink@4c1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5H3P5o6l9H3M7$3g2U0i4K6u0W2L8%4u0Y4i4K6u0r3)论坛的用户dtm。

由于Check Point名为InviZzzible的工具采用了模块化的代码结构，因此需要更多的空间来展示这个工具的代码样本，以达到相同的目的。这就是为什么我们决定在整个百科全书中使用其他伟大的开源项目作为例子。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)

[#其他](https://bbs.kanxue.com/forum-41-1-138.htm)
