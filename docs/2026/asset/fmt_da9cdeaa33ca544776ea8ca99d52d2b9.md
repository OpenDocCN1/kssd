
```
CallHook(nCallAddr,(int)MyKiAttachProcess);

DbgPrint("nCallAddr=%x\n",nCallAddr);

}

VOID UnHookKeAttchProcess()
{
        CallHook(HookKeAttachProcessAddr,(int)KiAttachProcessAddr);
}

#endif
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
