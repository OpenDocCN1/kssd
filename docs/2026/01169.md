
# 解析`__alloca_probe`

逆向vc编译的程序，经常会看到这样的代码出现在函数头部：

```asm
mov     eax, xxxxh
call    __alloca_probe
```

`xxxxh`是个立即数，一般大于`1000h`,即十进制数4096。

这段代码经常出现在函数头`prolog`之后，如果有异常结构，会出现在`SEH`或`EH`之后。

示例1：

```asm
push    ebp
mov     ebp, esp
mov     eax, 8080h
call    __alloca_probe
```

示例2：

```asm
push    ebp
mov     ebp, esp
and     esp, 0FFFFFFF8h
push    0FFFFFFFFh
push    offset SEH_4A7AA0
mov     eax, large fs:0
push    eax
mov     large fs:0, esp
push    ecx
mov     eax, 8080h
call    __alloca_probe
```

事实上`__alloca_probe`是一个在`stack`上分配大块内存空间函数，功能同`SUB ESP, xxxxh`一致。函数本身由编译器提供，编译的时候`vc`会根据实际情况（`stack`上申请大块空间，一般大于一个内存页大小）插入到函数体，为函数在`stack`上提供私有变量空间，分配的空间大小由`fastcall`调用方式的`EAX`寄存器传入。

函数源码位于`vs\Vcxx\crt\src\intel`目录下的`chkstk.asm`文件。代码很短，就列在这里：

```asm
        page    ,132
        title   chkstk - C stack checking routine

;***
;chkstk.asm - C stack checking routine
;
;       Copyright (c) Microsoft Corporation. All rights reserved.
;
;Purpose:
;       Provides support for automatic stack checking in C procedures
;       when stack checking is enabled.
;
;*******************************************************************************

.xlist
        include cruntime.inc
.list

; size of a page of memory
_PAGESIZE_      equ     1000h

        CODESEG
page

;***
;_chkstk - check stack upon procedure entry
;
;Purpose:
;       Provide stack checking on procedure entry. Method is to simply probe
;       each page of memory required for the stack in descending order. This
;       causes the necessary pages of memory to be allocated via the guard
;       page scheme, if possible. In the event of failure, the OS raises the
;       _XCPT_UNABLE_TO_GROW_STACK exception.
;
;       NOTE:  Currently, the (EAX < _PAGESIZE_) code path falls through
;       to the "lastpage" label of the (EAX >= _PAGESIZE_) code path.  This
;       is small; a minor speed optimization would be to special case
;       this up top.  This would avoid the painful save/restore of
;       ecx and would shorten the code path by 4-6 instructions.
;
;Entry:
;       EAX = size of local frame
;
;Exit:
;       ESP = new stackframe, if successful
;
;Uses:
;       EAX
;
;Exceptions:
;       _XCPT_GUARD_PAGE_VIOLATION - May be raised on a page probe. NEVER TRAP
;                                    THIS!!!! It is used by the OS to grow the
;                                    stack on demand.
;       _XCPT_UNABLE_TO_GROW_STACK - The stack cannot be grown. More precisely,
;                                    the attempt by the OS memory manager to
;                                    allocate another guard page in response
;                                    to a _XCPT_GUARD_PAGE_VIOLATION has
;                                    failed.
;
;*******************************************************************************

public _alloca_probe
_chkstk proc

_alloca_probe    =  _chkstk

        cmp     eax, _PAGESIZE_         ; more than one page?
        jae     short probesetup        ;   yes, go setup probe loop
                                        ;   no
        neg     eax                     ; compute new stack pointer in eax
        add     eax,esp
        add     eax,4
        test    dword ptr [eax],eax     ; probe it
        xchg    eax,esp
        mov     eax,dword ptr [eax]
        push    eax
        ret

probesetup:
        push    ecx                     ; save ecx
        lea     ecx,[esp] + 8           ; compute new stack pointer in ecx
                                        ; correct for return address and
                                        ; saved ecx

probepages:
        sub     ecx,_PAGESIZE_          ; yes, move down a page
        sub     eax,_PAGESIZE_          ; adjust request and...
        test    dword ptr [ecx],eax     ; ...probe it
        cmp     eax,_PAGESIZE_          ; more than one page requested?
        jae     short probepages        ; no

lastpage:
        sub     ecx,eax                 ; move stack down by eax
        mov     eax,esp                 ; save current tos and do a...
        test    dword ptr [ecx],eax     ; ...probe in case a page was crossed
        mov     esp,ecx                 ; set the new stack pointer
        mov     ecx,dword ptr [eax]     ; recover ecx
        mov     eax,dword ptr [eax + 4] ; recover return address
        push    eax                     ; prepare return address
                                        ; ...probe in case a page was crossed
        ret

_chkstk endp
        end
```

看得出，`_alloca_probe`事实上是`_chkstk`的别名。C约定会在编译之后的函数名前加一个下划线`_`,因此函数名就变成`__alloca_probe`。看看函数名，`alloca`是`allocate`缩写，而`probe`是探查、探测的意思，连在一起，应该明白函数名的含义了吧。`chkstk`当然就是`checkstack`了。而别名`_alloca_probe`更能体现函数功能。

源代码每一行都有注释，代码含义很好理解。再看IDA反汇编结果：

```asm
.text:004AC130 ; =============== S U B R O U T I N E =======================================

.text:004AC130

.text:004AC130 ; Attributes: library function

.text:004AC130

.text:004AC130 __alloca_probe  proc near               ; CODE XREF: sub_416D50+1Dp

.text:004AC130                                         ; sub_4307A0+1Dp ...

.text:004AC130

.text:004AC130 arg_0           = byte ptr  4

.text:004AC130

.text:004AC130                 cmp     eax, 1000h

.text:004AC135                 jnb     short probesetup

.text:004AC137                 neg     eax

.text:004AC139                 add     eax, esp

.text:004AC13B                 add     eax, 4

.text:004AC13E                 test    [eax], eax

.text:004AC140                 xchg    eax, esp

.text:004AC141                 mov     eax, [eax]

.text:004AC143                 push    eax

.text:004AC144                 retn

.text:004AC145 ; ---------------------------------------------------------------------------

.text:004AC145

.text:004AC145 probesetup:                             ; CODE XREF: __alloca_probe+5j

.text:004AC145                 push    ecx

.text:004AC146                 lea     ecx, [esp+4+arg_0]

.text:004AC14A

.text:004AC14A probepages:                             ; CODE XREF: __alloca_probe+2Cj

.text:004AC14A                 sub     ecx, 1000h

.text:004AC150                 sub     eax, 1000h

.text:004AC155                 test    [ecx], eax

.text:004AC157                 cmp     eax, 1000h

.text:004AC15C                 jnb     short probepages

.text:004AC15E

.text:004AC15E lastpage:

.text:004AC15E                 sub     ecx, eax

.text:004AC160                 mov     eax, esp

.text:004AC162                 test    [ecx], eax

.text:004AC164                 mov     esp, ecx

.text:004AC166                 mov     ecx, [eax]

.text:004AC168                 mov     eax, [eax+4]

.text:004AC16B                 push    eax

.text:004AC16C                 retn

.text:004AC16C __alloca_probe  endp
```

有源代码，有注释，这个就不用多说了。只是IDA给出的结果有点儿出入，`__alloca_probe`只有一个参数，就是`EAX`，而 `arg_0 = byte ptr  4`是不存在的， `lea     ecx, [esp+4+arg_0]`应该是`lea     ecx, [esp+8]`。

总结一下，`__alloca_probe`函数就是为函数在`stack`上申请大块空间做私有变量或缓存，由于内存分页机制，申请超过一页的内存空间，有可能在函数运行是没有提交到物理内存，调用这个函数，函数会从`stack`的高地址向低地址一页一页地探测，也就是函数中的几个测试语句，

```asm
test    [eax], eax
test    [ecx], eax
```

当函数执行到这些代码时，如果此时`eax`或`ecx`所指向的虚拟内存地址单元没有提交到物理内存，就会产生一个页异常，系统捕获到页异常，将物理内存提交到虚拟内存。函数执行完毕，所有申请的内存空间都已提交到物理内存。如果申请的空间致使线程栈空间分配溢出，系统产生`stack overflow exception` 。函数返回时 `esp = esp - eax`.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
