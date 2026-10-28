
```
    test eax, eax

    jz FAIL

    invoke ResumeThread,eax

FAIL:

    mov eax, hProcess

    .if eax

        invoke CloseHandle,eax

    .endif

    mov eax, lpMem

    .if eax

        invoke VirtualFree, eax, 4096, MEM_RELEASE

    .endif

    mov eax, lpMem2

    .if eax

        invoke VirtualFree, eax, 4096, MEM_RELEASE

    .endif

    ret

RemoteInjectBody endp

start:

    invoke RemoteInjectBody

  invoke ExitProcess, 0

end start
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
