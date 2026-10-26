
```
20001F58    8D8424 4C020000 lea     eax, dword ptr [esp+24C]
20001F5F    52              push    edx
20001F60    68 F0000000     push    0F0                              ; push f0改push 400
20001F65    50              push    eax
20001F66    FF15 EC240020   call    dword ptr [200024EC]             ; psapi.EnumProcesses
20001F6C    85C0            test    eax, eax
```

基本上就行了的

新上传一个版本，写了一个`LordPe_Fix.dll`来进行fix。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [lordpe_fix.rar](javascript:void(0);)

  （83.97kb，695次下载）
