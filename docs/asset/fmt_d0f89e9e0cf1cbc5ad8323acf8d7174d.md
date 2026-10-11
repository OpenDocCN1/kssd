
```
*//----------------------------------------*
};
*//----------------------------------------------------------------*
```

## 4.3 Some notes regarding creating a new PE file

### 4.3 创建新`PE`文件时应该注意的地方

*   按`SectionAlignment`对齐每个段的`VirtualAddress`和`VirtualSize`：

    ```
    image_section_header[i]->VirtualAddress =
        PEAlign(image_section_header[i]->VirtualAddress,
        image_nt_headers->OptionalHeader.SectionAlignment);

    image_section_header[i]->Misc.VirtualSize =
        PEAlign(image_section_header[i]->Misc.VirtualSize,
        image_nt_headers->OptionalHeader.SectionAlignment);
    ```

*   按`FileAlignment`对齐每个段的`PointerToRawData`和`SizeOfRawData`：

    ```
    image_section_header[i]->PointerToRawData =
        PEAlign(image_section_header[i]->PointerToRawData,
        image_nt_headers->OptionalHeader.FileAlignment);

    image_section_header[i]->SizeOfRawData =
        PEAlign(image_section_header[i]->SizeOfRawData,
        image_nt_headers->OptionalHeader.FileAlignment);
    ```

*   通过最后一个段的虚拟大小和虚拟地址更正`SizeOfImage`：

    ```
    image_nt_headers->OptionalHeader.SizeOfImage =
        image_section_header[LastSection]->VirtualAddress +
        image_section_header[LastSection]->Misc.VirtualSize;
    ```

*   将Bound Import Directory头部设置为零，因为该目录对执行`PE`文件不重要：

    ```
    image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT].VirtualAddress = 0;
    image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT].Size = 0;
    ```

## 4.4 Some notes regarding linking this VC Project

### 4.4 关于连接这个`VC`项目时应该注意的地方

*   将`Linker->General->Enable Incremental Linking`设置为`No (/INCREMENTAL:NO)`。

    ![Incremental Linking Setting](img/959671faa3c91797d56141b9ba187ffa.jpg)

通过下图，你应该能理解增量连接与非增量连接之间的区别：

![Incremental vs No-incremental Linking](img/b2e9e23c898ce04a75962005b4b00483.jpg)

为了得到`DynLoader()`的虚拟地址，我们在增量连接里可以得到`JMP pemaker.DynLoader`的虚拟地址，但是在非增量连接里，只能通过下面的代码得到真正的虚拟地址：

```
DWORD dwVA = (DWORD) DynLoader;
```

当你设法通过`CPECryptor::ReturnToBytePtr()`找出`DynLoader()`（即`Loader`）的开始和结束部分时，在增量连接里的这个设置显得尤为重要。

```
void* CPECryptor::ReturnToBytePtr(void* FuncName, DWORD findstr)
{
    void* tmpd;
    __asm
    {
        mov eax, FuncName
        jmp df

hjg:    inc eax
df:     mov ebx, [eax]
        cmp ebx, findstr
        jnz hjg
        mov tmpd, eax
    }
    return tmpd;
}
```

## 5 Store Important Data and Reach Original OEP

### 5 保存重要的数据并找到原始的`OEP`

为了找到`OEP`的虚拟地址，我们立即保存原始的`OEP`和`Image Base`。我在`DynLoader()`的尾部预留了一块空地来保存它们，参考[DynLoader Step 2](http://www.codeproject.com/system/inject2exe.asp#DynLoaderStep2#DynLoaderStep2)。

### `PE Maker - Step 2`

*   [Download source files - 58.3 Kb](http://www.codeproject.com/system/inject2exe/pemaker2.zip)

### `DynLoader Step 2`

```
__stdcall void DynLoader()
{
    _asm
    {
        *//----------------------------------*
        DWORD_TYPE(DYN_LOADER_START_MAGIC)
        *//----------------------------------*
    Main_0:
        PUSHAD
        *// get base ebp*
        CALL Main_1
    Main_1:
        POP EBP
        SUB EBP, OFFSET Main_1
        MOV EAX, DWORD PTR [EBP+_RO_dwImageBase]
        ADD EAX, DWORD PTR [EBP+_RO_dwOrgEntryPoint]
        PUSH EAX
        RETN *// >> JMP to Original OEP*

        *//----------------------------------*
        DWORD_TYPE(DYN_LOADER_START_DATA1)
        *//----------------------------------*
    _RO_dwImageBase:              DWORD_TYPE(0xCCCCCCCC)
    _RO_dwOrgEntryPoint:          DWORD_TYPE(0xCCCCCCCC)

        *//----------------------------------*
        DWORD_TYPE(DYN_LOADER_END_MAGIC)
        *----------------------------------*
    }
}
```

新函数`CPECryptor::CopyData1()`将把`Image Base`值和`OEP`值复制到`loader`里空闲空间的8字节里。

### 5.1 Restore the first Registers Context

#### 5.1 恢复最初寄存器的上下文

恢复线程的原始上下文很重要。不过，我们在[DynLoader Step 2](http://www.codeproject.com/system/inject2exe.asp#DynLoaderStep2#DynLoaderStep2)的源码里并没有这样做。我们可以修改`DynLoader()`的源码，重新得到最初的上下文。

```
__stdcall void DynLoader()
{
    _asm
    {
        *//----------------------------------*
        DWORD_TYPE(DYN_LOADER_START_MAGIC)
        *----------------------------------*
    Main_0:
        PUSHAD *// Save the registers context in stack*
        CALL Main_1
    Main_1:
        POP EBP *// Get Base EBP*
        SUB EBP, OFFSET Main_1
        MOV EAX, DWORD PTR [EBP+_RO_dwImageBase]
        ADD EAX, DWORD PTR [EBP+_RO_dwOrgEntryPoint]
        MOV DWORD PTR [ESP+1Ch], EAX *// pStack.Eax <- EAX*
        POPAD *// Restore the first registers context from stack*
        PUSH EAX
        XOR EAX, EAX
        RETN *// >> JMP to Original OEP*

        *----------------------------------*
        DWORD_TYPE(DYN_LOADER_START_DATA1)
        *----------------------------------*
    _RO_dwImageBase:              DWORD_TYPE(0xCCCCCCCC)
    _RO_dwOrgEntryPoint:          DWORD_TYPE(0xCCCCCCCC)

        *----------------------------------*
        DWORD_TYPE(DYN_LOADER_END_MAGIC)
        *----------------------------------*
    }
}
```

### 5.2 Restore the Original Stack

#### 5.2 恢复原始的栈

我们也可以在原始`OEP`的基础上，设置栈的起始值+0x34来恢复原来的栈，但它不是非常重要。不过，在下面的代码中，我通过一个小技巧找到`OEP`并重新修饰栈来完成`loader`代码。你可以用[OllyDbg](http://www.ollydbg.de/)或`SoftICE`观察具体的实现。

```
__stdcall void DynLoader()
{
    _asm
    {
        *----------------------------------*
        DWORD_TYPE(DYN_LOADER_START_MAGIC)
        *----------------------------------*
    Main_0:
        PUSHAD *// Save the registers context in stack*
        CALL Main_1
    Main_1:
        POP EBP
        SUB EBP, OFFSET Main_1
        MOV EAX, DWORD PTR [EBP+_RO_dwImageBase]
        ADD EAX, DWORD PTR [EBP+_RO_dwOrgEntryPoint]
        MOV DWORD PTR [ESP+54h], EAX *// pStack.Eip <- EAX*
        POPAD *// Restore the first registers context from stack*
        CALL _OEP_Jump
        DWORD_TYPE(0xCCCCCCCC)
    _OEP_Jump:
        PUSH EBP
        MOV EBP, ESP
        MOV EAX, DWORD PTR [ESP+3Ch] *// EAX <- pStack.Eip*
        MOV DWORD PTR [ESP+4h], EAX  *// _OEP_Jump RETURN pointer <- EAX*
        XOR EAX, EAX
        LEAVE
        RETN
    }
}
```
