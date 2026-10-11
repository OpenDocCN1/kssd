
```
// finally, make the interrupt point to our template code  
__asm cli  
idt_entries[count].LowOffset = (unsigned short)entry_ptr;  
idt_entries[count].HiOffset = (unsigned short)((unsigned long)entry_ptr >> 16);  
__asm sti  
}  
DbgPrint("Hooking Interrupt complete");  
return STATUS_SUCCESS;  
}
```

后面再附上一个`Greg Hoglund`的例子，请大家欣赏。

上传的附件

| `![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg)` || `bhwin_keysniff.rar`  
 |
