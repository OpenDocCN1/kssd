
# 通过散列获取函数地址

> 原文：[`www.pediy.com/kssd/pediy10/97141.html`](https://www.pediy.com/kssd/pediy10/97141.html)

## 介绍

本文不静态引入任何库文件，动态获取`kernel32`地址。

然后查找`kernel32`的输出表，计算输出函数的散列值与输入的值比较，找到后查找该函数的序号，最后查找函数地址数组，找到函数地址（这个方法我以前见过，可是现在忘了，于是花了一天时间写了这个代码，这样的代码应该在`shellcode`中能见到）。

## 程序演示

用`LoadLibrary`加载`user32.dll`，然后调用`MessageBoxA`。这两个函数的输入字符串都是通过堆栈传递的。

汇编很蹩脚大家凑合着看，特别是开始部分（查找`kernel32.dll`地址）是早上写的，看了几个小时的书终于会写函数了，所以下午写的代码还像个样子。

如果用`ollyice`调试可能会出现异常的情况，可能是由于手工编写的原因，下个断点都比较困难，而且`ollyice`反汇编出来的代码也不稳定，这应该是`ollyice`的问题。

## 汇编代码

```asm
.386
.model flat,stdcall
option casemap:none

Jisuan proto :DWORD
GetAddr proto :DWORD,:DWORD

.code

start:
  ;xor eax,eax
  ;ret
  MOV EAX,DWORD PTR [ESP]         ;取得程序返回地址 只要这个地址在 kernel32.dll 中都可以找到 kerner32.dll 的基地址
  AND EAX,0FFFF0000H              ;模块的加载地址 64K 对齐
  F:
  ;.IF  word ptr [eax] != 'MZ'相反了
  ;NOP
  ;NOP
  ;.ENDIF
  CMP WORD PTR [EAX],5A4DH;MZ
  JNZ NEXT
  MOV EDX,DWORD PTR [EAX+3CH]
  ADD EDX,EAX
  CMP WORD PTR [EDX],4550H;PE
  JE  FINDED
  NEXT:
  DEC EAX
  XOR AX,AX
  CMP EAX,70000000H     
  JA F
  FINDED:           ;找到 kernel32 的地址
  MOV EBX,EAX;EBX=KERNEL.DLL 地址 EDX 是 PE 地址              ;;;;;;;;;  
  MOV EAX,[EDX+78H]         
  ADD EAX,EBX;EAX = IMAGE_EXPORT_DIRECTORY BaseAddress  ;
  MOV ECX,[EAX+10H];[EAX+10] = nBase       ;测试用
  MOV EDX,[EAX+20H];AddressOfNames       ; 
  MOV EAX,[EBX+EDX];            ;   
  LEA EAX,[EBX+EAX];           ;;;;;;;;;
  ;INVOKE Jisuan,EAX
  INVOKE GetAddr,EBX,0AADF0F1H;LoadLibraryA hash 后的值
  PUSH 00006C6CH
  PUSH 642E3233H
  PUSH 52455355H
  MOV  EBX,ESP
  PUSH EBX
  CALL EAX
  ADD ESP,0CH
  INVOKE GetAddr,EAX,078A5C51H;MessageBoxA hash 值
  PUSH 00006C6CH
  PUSH 642E3233H
  PUSH 52455355H
  MOV  EBX,ESP
  PUSH 0
  PUSH EBX
  PUSH EBX
  PUSH 0
  CALL EAX
  ADD ESP,0CH
  NOP
  NOP
  NOP
  RET

;;;;;;;;;;;;;;;;;;;;;;;
;;;;Jisuan 用于计算指定字符串的 hash 值
;;;;只有一个字符串地址参数 返回 hash 值
;;;;;;;;;;;;;;;;;;;;;;;
Jisuan proc pFunName:DWORD;USES EBX,ECX 
  ;LOCAL f:BYTE
  PUSH EBX
  PUSH ECX
  PUSH EDX
  XOR EBX,EBX
  ;MOV BL,BYTE PTR [pFunName] 直接用这句是不行滴
  MOV ECX,pFunName
  MOV BL,BYTE PTR [ECX]
  XOR EAX,EAX
  ;XOR ECX,ECX
  .WHILE BL
    SHL EAX,4
    ADD EAX,EBX
    XOR EDX,EDX
    MOV EDX,EAX
    AND EDX,0F0000000H
    .IF EDX
      PUSH EDX;保护 EDX 值
      SHR EDX,24
      XOR EAX,EDX
      POP EDX
    .endif
    NOT EDX
    AND EAX,EDX
    INC ECX
    MOV BL,BYTE PTR [ECX]
  .endw
  POP EDX
  POP ECX
  POP EBX
  ret
Jisuan endp

;;;;;;;;;;;;;;;
;BaseAddr 模块的加载地址
;ID 函数名 hash 后的数值
;;;;;;;;;;;;;;;
GetAddr proc uses EBX ECX EDX, BaseAddr:DWORD,ID:DWORD
  LOCAL AddressOfFunction:DWORD,AddressOfNames:DWORD,AddressOfNameOrdinals:DWORD,nBase:DWORD,Num:DWORD
  ;MOV EAX,BaseAddr
  MOV EBX,BaseAddr
  MOV EAX,[EBX+3CH]
  ADD EAX,EBX;获取 PEHEADER ADDRESS
  MOV EAX,[EAX+78H]
  ADD EAX,EBX
  MOV EDX,[EAX+10H]
  MOV nBase,EDX
  MOV EDX,[EAX+18H]
  MOV Num,EDX;函数名字总数
  MOV EDX,[EAX+20H]
  MOV AddressOfNames,EDX
  MOV EDX,[EAX+1CH]
  MOV AddressOfFunction,EDX
  MOV EDX,[EAX+24H]
  MOV AddressOfNameOrdinals,EDX
  ;MOV nBase,[EAX+10H];nBase
  ;MOV AddressOfNames,[EAX+20H];AddressOfname
  XOR ECX,ECX
  ;LEA EDX,[EBX+AddressOfNames]不能这样
  MOV EDX,AddressOfNames
  ADD EDX,EBX
  .WHILE ECX < Num
    MOV EAX,[EDX+ECX*4]
    ADD EAX,EBX
    INVOKE Jisuan,EAX
    .IF EAX == ID
      .break
    .endif
    INC ECX
  .endw
  MOV EDX,AddressOfNameOrdinals
  ADD EDX,EBX
  ;.WHILE ECX
  ;  LEA EDX,[EDX+2H]
  ;  DEC ECX
  ;.endw
  XOR EAX,EAX
  MOV AX,[EDX+ECX*2];取得 Ordinal
  MOV EDX,AddressOfFunction
  ADD EDX,EBX
  MOV EAX,[EDX+EAX*4];从 AddressOfFunction 数组查出地址
  ADD EAX,EBX
  ;XOR EAX,EAX
  ret
GetAddr endp

END start
```

上面计算散列的函数可以用以下代码实现：

```c
#include <stdio.h>

//#define M 0xffffffff

int ELFhash(char *key);

int main(int argc, char* argv[])
{
  printf("Abs:%8x",ELFhash("LoadLibraryA"));
  printf("\n");
  printf("MessageBoxA:%8x",ELFhash("MessageBoxA"));
  return 0;
}

int ELFhash(char *key)
{
  unsigned long h=0;
  while(*key)
  {
    h=(h<<4)+*key++;
    unsigned long g = h& 0xf0000000L;
    if(g) h^=g>>24;
    h &=~g;
  }
  return h;
}
```

## 附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | first.rar |
| --- | --- |
