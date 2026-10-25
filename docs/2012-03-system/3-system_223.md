
# `seh` 的另一种用法

> 原文：[`www.pediy.com/kssd/pediy05/pediy50152.htm`](https://www.pediy.com/kssd/pediy05/pediy50152.htm)

;在前面你可能已经熟悉了用 `seh` 修改 `context` 的 `eip` 实现跳转的方法
;这里我们来试试另一种方法,如果只是想在发生异常的时候转移,那更加简单
;理解的关键是在 `xHandler` 中,系统自动又为处理代码建立了异常处理
;所以,要废除掉!然后不交由系统处理,我们自己利用 `seh` 作一下跳板
;你可能已经在某些壳里面或病毒里面看到这种用法了...
;实际上我们也可以用这种方法来用于自己的目的,很方便...
;9X 下测试通过...我还要出去...god save me!

```
.586
.model flat, stdcall
option casemap :none ; case sensitive
include c:\hd\hd.h
include c:\hd\mac.h
;~~~~~~~~~~~~~~~~~~~protos
;~~~~~~~~~~~~~~~~~~~~~~~~~
ASSUME fs:NOTHING
    .CODE
__Start:
    xor    edi,edi
        lea    eax,[esp-8]
        xchg    eax,fs:[edi]
        push    offset xhandler
        push    eax
        mov    eax,[edi]              ;test for
        pop    fs:[edi]
        pop    eax
_xit:
        invoke    ExitProcess,0
safePlace:
        invoke    MessageBox,0,CTEXT("come Here!"),CTEXT("Test"),0
        jmp    _xit
xhandler:
        xor    edi,edi
        mov    eax,fs:[edi]
        mov    esp,[eax]
        pop    fs:[edi]               ;resume stack
        ;pop    eax                   ;a little redundant
        ;push    offset safePlace
        mov    [esp],offset safePlace ;why not act like this?
        ret                            ; to return
END    __Start  4dd466d0e1e0cc123afbe504da2462ca
```
