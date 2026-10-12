
# 使用 dll 进行验证的注册方式

> 原文：[`www.pediy.com/kssd/pediy05/pediy50357.htm`](https://www.pediy.com/kssd/pediy05/pediy50357.htm)

## 暴破之瞒天过海

分析某软件的注册验证，发现它是通过调用一个 `dll` 来判断 `keyfile` 内容是否正确，是否在使用期之内。这个子程序写得比较复杂，当然如果想找算法还是能找得到 :)。如 `keyfile` 正确，则返回 `eax=0`，并且内存某位置的 8 个字节应该为固定的一个值。这样，分析 `dll` 文件，发现它仅有这一个导出的函数，那么仿造这样一个 `dll` 是完全可行的。内容简单如下：

```asm
.386   
.model flat, stdcall   
option casemap: none   
include \masm32\include\windows.inc   
include \masm32\include\user32.inc   
include \masm32\include\kernel32.inc   
includelib \masm32\lib\user32.lib   
includelib \masm32\lib\kernel32.lib   

.data   
.RADIX 16   
SomeThing db 26, 1d, 28, 15, 34, 5d, 08, 4b, 10, 1b, 00   

.code   
.RADIX 16   

DllEntry proc hInstDLL:HINSTANCE, reason:DWORD, reserved1:DWORD   
mov eax, TRUE   
ret   
DllEntry Endp   

;---------------------------------------------------------------------------------------------------   
;反击 1。   
;----------------------------------------------------------------------------------------------------   

XXXX proc   
push ecx   
push esi   
push edi   
mov edi, dword ptr [esp+14] ；待赋值内存位置   
lea esi, SomeThing   
push 0b   
pop ecx   
rep movsb   
pop edi   
pop esi   
pop ecx   
xor eax, eax ；正确的返回值   
ret 0c ；入栈有三个参数，保持 stack 平衡   

XXXX endp   
End DllEntry 4dd466d0e1e0cc123afbe504da2462ca
```
