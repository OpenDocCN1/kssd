
对于`TRAMPOLINE`的构建是一块相对复杂的内容。可以看到结构体的前两个成员分别是原函数地址和我们想要到达的自定义函数地址，第三个成员`MemorySlot`，记录的是原函数地址开头的五个字节，以及跳转回原函数的`jmp`指令，`OldPos`和`NewPos`则分别保存了在构建`MemorySlot`的过程中，已经读取的目标函数偏移字节数，和已经写入的`Detour`函数字节数。

构建`MemorySlot`是通过一个`do_while`循环来实现的，首先它需要保存好被覆盖的那些字节数，所以在原函数已经读取的字节数少于`jmp`+四字节`offset`，一共五字节之前，`MemorySlot`要先备份下原来的指令，一旦原函数已经读取的字节数超过或者等于覆盖所需要的五字节时，也就不必再备份原来函数基地址处的指令了，因为仅仅只是覆盖五字节的内容：

```
do
{
    HDE hde;
    UINT CopyCodeLength;
    LPVOID CopyCodeData;

    //对于出现的相对偏移地址，在跳板中都要给出新的相对地址
    /*
    74CA8B80 8B FF                mov         edi,edi
    74CA8B82 55                   push        ebp
    74CA8B83 8B EC                mov         ebp,esp
    74CA8B85 6A 00                push        0
    74CA8B87 FF 75 14             push        dword ptr [ebp+14h]
    74CA8B8A FF 75 10             push        dword ptr [ebp+10h]
    74CA8B8D FF 75 0C             push        dword ptr [ebp+0Ch]
    74CA8B90 FF 75 08             push        dword ptr [ebp+8]
    74CA8B93 E8 F8 FC FF FF       call        _MessageBoxExW@20 (74CA8890h)
    */

    ULONG_PTR OldInstance = (ULONG_PTR)Trampoline->TargetFunctionAddress + OldPos;
    ULONG_PTR NewInstance = (ULONG_PTR)Trampoline->MemorySlot + NewPos;

    //指令长度
    CopyCodeLength = HDE_DISASM((LPVOID)OldInstance, &hde);
    if(hde.flags & F_ERROR)
        return FALSE;

    CopyCodeData = (LPVOID)OldInstance;//第一次时，CopyCodeData是MessageBox入口地址

    if(OldPos >= sizeof(JMP_REL))
    {
        // The trampoline function is long enough.
        // Complete the function with the jump to the target function.
        #if defined(_M_X64) || defined(__x86_64__)
        jmp.address = pOldInst;
        #else
        //OldInstance = 74CA8B85
        //目标 = 源 + Offset + 5
        //Offset = 目标 - (源 + 5)
        jmp.Operand = (UINT32)(OldInstance - (NewInstance + sizeof(jmp)));  //计算跳转到目标的偏移
        #endif
        CopyCodeData = &jmp;
        CopyCodeLength = sizeof(jmp);
        IsLoop = TRUE;
    }
    //......此处省略部分源码，突出重点

    Trampoline->OldIPs[Trampoline->Index] = OldPos;
    Trampoline->NewIPs[Trampoline->Index] = NewPos;
    Trampoline->Index++;

    // Avoid using memcpy to reduce the footprint.
    #ifndef _MSC_VER
    memcpy((LPBYTE)ct->pTrampoline + newPos, pCopySrc, copySize);
    #else
    __movsb((LPBYTE)Trampoline->MemorySlot + NewPos, (const unsigned char*)CopyCodeData, CopyCodeLength);
    #endif

    NewPos += CopyCodeLength;
    OldPos += hde.len;
} while(!IsLoop);
```

在这段`do_while`循环里，通过反汇编可以清晰地看到`MemorySlot`一步步的构建，在测试代码中我Hook的是`MessageBox`函数，所以在看`MemorySlot`的构造过程之前，先来看一看反汇编下原始的`MessageBox`的机器指令：

对应着`MessageBox`的机器指令，来看`MemorySlot`在`do_while`循环中的构造过程：

局部变量窗口找到`MemorySlot`的地址：

![](https://bbs.kanxue.com/upload/attach/202002/756811_grs8xp7oyd7w2ac.png)

先是申请了32字节的结构体大小：

![](https://bbs.kanxue.com/upload/attach/202002/756811_w4t7uwwl9iqwsg0.png)
