
| `\_EH\_prolog3/\_EH\_epilog3` | C++ EH | + | - | - |
| `\_EH\_prolog3\_catch/\_EH\_epilog3` | C++ EH | + | - | + |
| `\_EH\_prolog3\_GS/\_EH\_epilog3\_GS` | C++ EH | + | + | - |
| `\_EH\_prolog3\_catch\_GS/\_EH\_epilog3\_catch\_GS` | C++ EH | + | + | + |

### SEH2

显然，在过去它用于 MSVC 1.XX 编译器（由 `crtdll.dll` 导出）。可能会在一些老的 NT 程序中碰到它。

...

```
    Saved edi
    Saved esi
    Saved ebx
    Next SEH frame
    Current SEH handler (\_\_except\_handler2)
    Pointer to the scopetable
    Try level
    Saved ebp (of this function)
    Exception pointers
    Local variables
    Saved ESP
    Local variables
    Callee EBP
    Return address
    Function arguments
```

...

### Appendix I: SEH 样例

让我们思考下面的反汇编代码。

```
func1           proc near

_excCode        = dword ptr -28h
buf             = byte ptr -24h
_saved_esp      = dword ptr -18h
_exception_info = dword ptr -14h
_next           = dword ptr -10h
_handler        = dword ptr -0Ch
_scopetable     = dword ptr -8
_trylevel       = dword ptr -4
str             = dword ptr  8

 push    ebp
 mov     ebp, esp
 push    -1
 push    offset _func1_scopetable
 push    offset _except_handler3
 mov     eax, large fs:0
 push    eax
 mov     large fs:0, esp
 add     esp, -18h
 push    ebx
 push    esi
 push    edi
 ; --- end of prolog ---

 mov     [ebp+_trylevel], 0 ;trylevel -1 -> 0: beginning of try block 0
 mov     [ebp+_trylevel], 1 ;trylevel 0 -> 1: beginning of try block 1
 mov     large dword ptr ds:123, 456
 mov     [ebp+_trylevel], 0 ;trylevel 1 -> 0: end of try block 1
 jmp     short _endoftry1
_func1_filter1:              ; __except() filter of try block 1
 mov     ecx, [ebp+_exception_info]
 mov     edx, [ecx+EXCEPTION_POINTERS.ExceptionRecord]
 mov     eax, [edx+EXCEPTION_RECORD.ExceptionCode]
 mov     [ebp+_excCode], eax
 mov     ecx, [ebp+_excCode]
 xor     eax, eax
 cmp     ecx, EXCEPTION_ACCESS_VIOLATION
 setz    al
 retn
_func1_handler1:             ; beginning of handler for try block 1
 mov     esp, [ebp+_saved_esp]
 push    offset aAccessViolatio ; "Access violation"
 call    _printf
 add     esp, 4
 mov     [ebp+_trylevel], 0 ;trylevel 1 -> 0: end of try block 1
_endoftry1:
 mov     edx, [ebp+str]
 push    edx
 lea     eax, [ebp+buf]
 push    eax
 call    _strcpy
 add     esp, 8
 mov     [ebp+_trylevel], -1 ; trylevel 0 -> -1: end of try block 0
 call    _func1_handler0     ; execute __finally of try block 0
 jmp     short _endoftry0
_func1_handler0:             ; __finally handler of try block 0
 push    offset aInFinally ; "in finally"
 call    _puts
 add     esp, 4
 retn
_endoftry0:
 ; --- epilog ---
 mov     ecx, [ebp+_next]
 mov     large fs:0, ecx
 pop     edi
 pop     esi
 pop     ebx
 mov     esp, ebp
 pop     ebp
 retn

func1           endp

_func1_scopetable
 ;try block 0
 dd -1                    ;EnclosingLevel
 dd 0                     ;FilterFunc
 dd offset _func1_handler0 ;HandlerFunc
 ;try block 1
 dd 0                     ;EnclosingLevel
 dd offset _func1_filter1  ;FilterFunc
 dd offset _func1_handler1 ;HandlerFunc
```

`Try` 块 0 没有 filter，因此它的 handler 是一个 `__finally` 块。`Try` 块 1 的 `EnclosingLevel` 是 0，所以它被置于 try 块 0 内部。考虑到这些，我们就可以试着重构出函数的结构：

```
    void func1 (char* str)
    {
      char buf[12];
      __try // try block 0
      {
         __try // try block 1
         {
           *(int*)123=456;
         }
         __except(GetExceptCode() == EXCEPTION_ACCESS_VIOLATION)
         {
            printf("Access violation");
         }
         strcpy(buf,str);
      }
      __finally
      {
         puts("in finally");
      }
    }
```

### Appendix II: C++异常样例

```
func1           proc near

_a1             = dword ptr -24h
_exc            = dword ptr -20h
e               = dword ptr -1Ch
a2              = dword ptr -18h
a1              = dword ptr -14h
_saved_esp      = dword ptr -10h
_next           = dword ptr -0Ch
_handler        = dword ptr -8
_state          = dword ptr -4

 push    ebp
 mov     ebp, esp
 push    0FFFFFFFFh
 push    offset func1_ehhandler
 mov     eax, large fs:0
 push    eax
 mov     large fs:0, esp
 push    ecx
 sub     esp, 14h
 push    ebx
 push    esi
 push    edi
 mov     [ebp+_saved_esp], esp
 ; --- end of prolog ---

 lea     ecx, [ebp+a1]
 call    A::A(void)
 mov     [ebp+_state], 0          ; state -1 -> 0: a1 constructed
 mov     [ebp+a1], 1             ; a1.m1 = 1
 mov     byte ptr [ebp+_state], 1 ; state 0 -> 1: try {
 lea     ecx, [ebp+a2]
 call    A::A(void)
 mov     [ebp+_a1], eax
 mov     byte ptr [ebp+_state], 2 ; state 2: a2 constructed
 mov     [ebp+a2], 2              ; a2.m1 = 2
 mov     eax, [ebp+a1]
 cmp     eax, [ebp+a2]            ; a1.m1 == a2.m1?
 jnz     short loc_40109F
 mov     [ebp+_exc], offset aAbc  ; _exc = "abc"
 push    offset __TI1?PAD         ; char *
 lea     ecx, [ebp+_exc]
 push    ecx
 call    _CxxThrowException       ; throw "abc";
loc_40109F:
 mov     byte ptr [ebp+_state], 1 ; state 2 -> 1: destruct a2
 lea     ecx, [ebp+a2]
 call    A::~A(void)
 jmp     short func1_try0end

; catch (char * e)
func1_try0handler_pchar:
 mov     edx, [ebp+e]
 push    edx
 push    offset aCaughtS ; "Caught %s\\n"
 call    ds:printf
 add     esp, 8
 mov     eax, offset func1_try0end
 retn

; catch (...)
func1_try0handler_ellipsis:
 push    offset aCaught___ ; "Caught ...\\n"
 call    ds:printf
 add     esp, 4
 mov     eax, offset func1_try0end
 retn

func1_try0end:
 mov     [ebp+_state], 0          ; state 1 -> 0: }//try
 push    offset aAfterTry ; "after try\\n"
 call    ds:printf
 add     esp, 4
 mov     [ebp+_state], -1         ; state 0 -> -1: destruct a1
 lea     ecx, [ebp+a1]
 call    A::~A(void)
 ; --- epilog ---
 mov     ecx, [ebp+_next]
 mov     large fs:0, ecx
 pop     edi
 pop     esi
 pop     ebx
 mov     esp, ebp
 pop     ebp
 retn

func1           endp

func1_ehhandler proc near
 mov     eax, offset func1_funcinfo
 jmp     __CxxFrameHandler
func1_ehhandler endp

func1_funcinfo
 dd 19930520h             ; magicNumber
 dd 4                     ; maxState
 dd offset func1_unwindmap ; pUnwindMap
 dd 1                     ; nTryBlocks
 dd offset func1_trymap   ; pTryBlockMap
 dd 0                     ; nIPMapEntries
 dd 0                     ; pIPtoStateMap
 dd 0                     ; pESTypeList

func1_unwindmap
 dd -1
 dd offset func1_unwind_1tobase ; action
 dd 0                      ; toState
 dd 0                      ; action
 dd 1                      ; toState
 dd offset func1_unwind_2to1 ; action
 dd 0                      ; toState
 dd 0                      ; action

func1_trymap
 dd 1                     ; tryLow
 dd 2                     ; tryHigh
 dd 3                     ; catchHigh
 dd 2                     ; nCatches
 dd offset func1_tryhandlers_0 ; pHandlerArray
 dd 0

func1_tryhandlers_0
dd 0                      ; adjectives
dd offset char * `RTTI Type Descriptor' ; pType
dd -1Ch                   ; dispCatchObj
dd offset func1_try0handler_pchar ; addressOfHandler
dd 0                      ; adjectives
dd 0                      ; pType
dd 0                      ; dispCatchObj
dd offset func1_try0handler_ellipsis ; addressOfHandler

func1_unwind_1tobase proc near
a1 = byte ptr -14h
 lea     ecx, [ebp+a1]
 call    A::~A(void)
 retn
func1_unwind_1tobase endp
```
