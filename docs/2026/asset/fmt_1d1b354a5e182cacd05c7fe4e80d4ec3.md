
3. `ExceptionList->Next` 如果不是 `0xFFFFFFFF`，获取下一个 `ExceptionList`，重复第2步

## 寻找VEH VCH异常的函数地址

我创建一个 demo

```
#include "stdafx.h"
#include <windows.h>

DWORD d;

LONG WINAPI VCHCallBack(PEXCEPTION_POINTERS ExceptionInfo)
{
    ExceptionInfo->ContextRecord->Eax = (DWORD)&d;
    return EXCEPTION_CONTINUE_EXECUTION;
}

LONG WINAPI VEHCallBack(PEXCEPTION_POINTERS ExceptionInfo)
{
    ExceptionInfo->ContextRecord->Eax = (DWORD)&d;
    return EXCEPTION_CONTINUE_EXECUTION;
}

int main()
{
    AddVectoredExceptionHandler(0, VEHCallBack);
    AddVectoredContinueHandler(0, VCHCallBack);

    __asm
    {
        mov eax,0
        mov [eax],0
    }
    getchar();
    return 0;
}
```

用 OD 附加查看 `main` 函数：

![VEH VCH调用](https://bbs.kanxue.com/upload/attach/202002/793792_P8WAS84H9YG2TJZ.png)

我们可以看到添加 VEH、VCH 调用的是

- `RtlAddVectoredExceptionHandler`
- `RtlAddVectoredContinueHandler`

用 IDA 打开 `ntdll.dll` 查看这两个函数

![VCH](https://bbs.kanxue.com/upload/attach/201804/793792_FPVYWVMQA2DC4RT.png)

![VCH](https://bbs.kanxue.com/upload/attach/201804/793792_WMDHNJDAB7P4N77.png)

这两个函数最终都调用了 `RtlpAddVectoredHandler`

这个函数有 3 个参数，其中第 3 个参数代表是 VEH 还是 VCH

可以看到 VEH 时 `RtlpAddVectoredHandler` 的第3个参数为 0

VCH 时 `RtlpAddVectoredHandler` 的第3个参数为 1

继续用 IDA 查看 `RtlpAddVectoredHandler` 函数

```
text:7DED3621 ; int __stdcall RtlpAddVectoredHandler(int Arg_First, PVOID Pointer, int)
.text:7DED3621 _RtlpAddVectoredHandler@12 proc near    ; CODE XREF: RtlAddVectoredContinueHandler(x,x)+D↓p
.text:7DED3621                                         ; RtlAddVectoredExceptionHandler(x,x)+D↓p
.text:7DED3621
.text:7DED3621 Arg_First       = dword ptr  8
.text:7DED3621 Pointer         = dword ptr  0Ch
.text:7DED3621 arg_Type        = dword ptr  10h
.text:7DED3621
.text:7DED3621 ; FUNCTION CHUNK AT .text:7DEE7414 SIZE 00000012 BYTES
.text:7DED3621
.text:7DED3621                 mov     edi, edi
.text:7DED3623                 push    ebp
.text:7DED3624                 mov     ebp, esp
.text:7DED3626                 mov     eax, large fs:18h ; 获取TEB首地址
.text:7DED362C                 mov     eax, [eax+30h]  ; 获取PEB地址
.text:7DED362F                 push    esi
.text:7DED3630                 push    10h             ; Size
.text:7DED3632                 push    0               ; Flags
.text:7DED3634                 push    dword ptr [eax+18h] ; HeapHandle
.text:7DED3637                 call    _RtlAllocateHeap@12 ; 申请10h堆栈空间
.text:7DED363C                 mov     esi, eax
.text:7DED363E                 test    esi, esi
.text:7DED3640                 jz      short loc_7DED36A4 ; 判断地址空间是否申请成功
.text:7DED3642                 push    ebx             ; 保存寄存器
.text:7DED3643                 push    edi
.text:7DED3644                 push    [ebp+Pointer]   ; Pointer
.text:7DED3647                 mov     dword ptr [esi+8], 1 ; 填充申请的第3个字段
.text:7DED364E                 call    _RtlEncodePointer@4 ; RtlEncodePointer(x)
.text:7DED3653                 mov     ebx, [ebp+arg_Type] ; 当Arg_Type 是0是VEH 是1时是VCH
.text:7DED3656                 imul    ebx, 0Ch
.text:7DED3659                 add     ebx, offset _LdrpVectorHandlerList ; arg_type*0cH+_LdrpVectorHandlerList 指向了存储的地方
.text:7DED365F                 push    ebx
.text:7DED3660                 mov     [esi+0Ch], eax
.text:7DED3663                 lea     edi, [ebx+4]
.text:7DED3666                 call    _RtlAcquireSRWLockExclusive@4 ; RtlAcquireSRWLockExclusive(x)
.text:7DED366B                 cmp     [edi], edi
.text:7DED366D                 jnz     short loc_7DED3686 ; 判断First参数
.text:7DED366F                 mov     ecx, large fs:18h ; 获取TEB首地址
.text:7DED3676                 mov     eax, [ebp+arg_Type]
.text:7DED3679                 mov     ecx, [ecx+30h]  ; 获取PEB
.text:7DED367C                 add     eax, 2
.text:7DED367F                 add     ecx, 28h        ; 指向PEB.EnvironmentUpdateCount
.text:7DED3682                 lock bts [ecx], eax
.text:7DED3686
.text:7DED3686 loc_7DED3686:                           ; CODE XREF: RtlpAddVectoredHandler(x,x,x)+4C↑j
.text:7DED3686                 cmp     [ebp+Arg_First], 0 ; 判断First参数
.text:7DED368A                 jz      loc_7DEE7414
.text:7DED3690                 mov     eax, [edi]      ; 如果为1
.text:7DED3692                 mov     [esi], eax
.text:7DED3694                 mov     [esi+4], edi
.text:7DED3697                 mov     [eax+4], esi
.text:7DED369A                 mov     [edi], esi
.text:7DED369C
.text:7DED369C loc_7DED369C:                           ; CODE XREF: RtlpAddVectoredHandler(x,x,x)+13E00↓j
.text:7DED369C                 push    ebx
.text:7DED369D                 call    _RtlReleaseSRWLockExclusive@4 ; RtlReleaseSRWLockExclusive(x)
.text:7DED36A2                 pop     edi
.text:7DED36A3                 pop     ebx
.text:7DED36A4
.text:7DED36A4 loc_7DED36A4:                           ; CODE XREF: RtlpAddVectoredHandler(x,x,x)+1F↑j
.text:7DED36A4                 mov     eax, esi
.text:7DED36A6                 pop     esi
.text:7DED36A7                 pop     ebp
.text:7DED36A8                 retn    0Ch
.text:7DED36A8 _RtlpAddVectoredHandler@12 endp
```

可以看到 `_LdrpVectorHandlerList` 指向了存放 VEH 和 VCH 的地方。其中每一个 VEH 和 VCH 会用一个 `10h` 大小的空间保存。

`_LdrpVectorHandlerList` 在 `ntdll` 是一个全局参数

- `_LdrpVectorHandlerList`：指向了 VEH
- `_LdrpVectorHandlerList+C`：指向了 VCH

经过分析 `_LdrpVectorHandlerList` 全局变量的结构类型为

```
typedef struct _LIST_ENTRY  //这是一个环形链表
{
    struct _LIST_ENTRY *Flink;//指向下一个
    struct _LIST_ENTRY *Blink;//指向前一个
}LIST_ENTRY,*PLIST_ENTRY;

struct _LVHL  //_LdrpVectorHandlerList
{
    struct _VEH
    {
        DWORD  VEH_LockCount;//这个参数不重要
        LIST_ENTRY VEH_listTop;
    };
    struct _VCH
    {
        DWORD  VCH_LockCount;//这个参数不重要
        LIST_ENTRY VCH_listTop;
    };
//申请10h堆空间保存VEH和VCH的结构

struct SaveVector
{
    LIST_ENTRY List;
    DWORD      isUse;
    DWORD      EncodeVectoredHandler;//经过加密的回调函数地址

}

}
```

还需要注意的是 VEH 和 VCH 的回调函数指针是经过 `RtlEncodePointer` 加密的，所以要获取真实的函数指针地址需要经过 `RtlDecodePointer` 解密。

描述总是浅显的,看图

![链表](https://bbs.kanxue.com/upload/attach/201804/793792_DZ2TUETS3KB5U5Q.png)

总结:

1. 在 `ntdll` 中找到 `RtlpAddVectoredHandler` 函数
2. 获取 `_LdrpVectorHandlerList`（`add ebx, offset _LdrpVectorHandlerList`;这句汇编代码获取）
3. 根据链表关系寻找 VEH 和 VCH 的回调函数地址
4. 用 `RtlDecodePointer` 解密

## 寻找UEF异常的函数地址

UEF函数地址的寻找和寻找VEH和VCH是一样的,而且UEF更加简单

### demo

```
#include "stdafx.h"
#include <windows.h>

DWORD d;

LONG WINAPI    UEFCallBack(PEXCEPTION_POINTERS ExceptionInfo)
{
    ExceptionInfo->ContextRecord->Eax = (DWORD)&d;
    return EXCEPTION_CONTINUE_EXECUTION;
}

int main()
{
    SetUnhandledExceptionFilter(UEFCallBack);

    __asm
    {
        mov eax,0
        mov [eax],0
    }
    getchar();
    return 0;
}
```

**用OD附加**

![UEF](https://bbs.kanxue.com/upload/attach/201804/793792_4U6HS5DTHQWDBXV.png)

用 IDA 打开 `Kernel32.dll` 找到 `SetUnhandledExceptionFilter`

完整代码:
