
```
KdPrint(("OldParseProcedure addrs is %08X\n",OldParseProcedure));  

KdPrint(("addrs is %08X\n",addrs));  

//这里最好检查一下 OldParseProcedure ，我真的是太懒了。  

__asm  
  {  
    cli;  
    mov eax, cr0;  
    and eax, not 10000h;  
    mov cr0, eax;  
  }  

pType->TypeInfo.ParseProcedure = NewParseProcedure;//hook  
  __asm  
  {  
    mov eax, cr0;  
    or eax, 10000h;  
    mov cr0, eax;  
    sti;  
  }  

 Status = ZwClose(hFile);  

  return Status;  

}  

NTSTATUS DriverEntry(PDRIVER_OBJECT DriverObject,PUNICODE_STRING RegistryPath)  

{  
  NTSTATUS Status = STATUS_SUCCESS;  
  Status=Hook();  
  return Status;  

}  
```

![](img/8bfcb520dcd38d15a87b3b6894fee165.jpg)

![](img/a3509246108d0c27ec6cad30216c8ac4.jpg)
