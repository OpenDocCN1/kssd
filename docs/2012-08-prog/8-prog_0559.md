
# 获取函数机器码大小

> 原文：[`www.pediy.com/kssd/pediy10/101137.html`](https://www.pediy.com/kssd/pediy10/101137.html)

主题思想：分析 `jmp` `jcc` 等转移类指令，需找最远的指令地址；寻找 `ret` 指令；如果当前指令为 `ret` 指令并且为最远指令则函数到达结束位置。

## 代码

```
GetProcSize  proc  uses esi ebx edi pProc:DWORD
;eax    指令长度
;ebx    当前指令
;ecx    函数地址
;esi    当前指令地址,也是当前最远的那条指令地址

  mov    esi,pProc
  mov   edi,esi
   
  invoke  GetCodeSize,esi
  .while  eax
    .if  eax == 2 && ( (byte ptr[esi] > 70H && byte ptr[esi] < 7FH) || byte ptr[esi] == 0EBH )
      movsx  ebx,byte ptr[esi+1]
    .elseif  eax == 5 && byte ptr[esi] == 0E9H
      mov   ebx,[esi+1]
    .elseif eax == 6 && byte ptr[esi] == 0FH && byte ptr[esi+1] > 80H && byte ptr[esi+1] < 8FH
      mov   ebx,[esi+2]
    .else
      .if  (byte ptr[esi] == 0C2H || byte ptr[esi] == 0C3H || byte ptr[esi] == 0CAH || byte ptr[esi] == 0CBH) && esi == edi
        lea   eax,[esi+eax]
        sub    eax,pProc
        ret
      .else
        xor    ebx,ebx
      .endif
    .endif
    add    esi,eax
    test  ebx,ebx
    .if  !sign?
      add   ebx,esi
      .if  ebx > edi
        mov   edi,ebx
      .endif
    .endif
    .if  esi > edi
      mov   edi,esi
    .endif
    invoke  GetCodeSize,esi
  .endw
  xor    eax,eax
  ret
GetProcSize endp
```

## 备注

1、`GetCodeSize` 为 `lde32` 长度反汇编引擎。

2、此代码有局限性，不能解析 `jmp [xx]` 指令；由后面 `jmp` 到 `ret` 指令的函数可能不能获取完整大小。但是由编译器生成的大部分都是可以解析的。

    希望大家给点改善的意见，使劲拍砖。
