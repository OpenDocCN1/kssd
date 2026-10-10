

# Anti-RORDbg

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-831.htm`](https://www.pediy.com/kssd/pediy08/pediy8-831.htm)

RORDbg 对于一些新壳的确是不错的选择，可惜我的机器太慢，但也可以给一个小程序加壳来分析壳的大致走向。  
  
由于是虚拟执行，所以异常是调试器自己识别出来的，不是 CPU 报告，这里就有一个小漏洞可以利用，就是 RaiseException。  
  
RORDbg 某个版本之后会识别对此函数的调用，视为异常，于是去找 SEH，但如果 RaiseException 之后没有异常，没有 SEH，程序就会卡住，呆若木鸡。  
  
看一个例子，并跟踪一下就会明白了，这个例子可以在 RORDbg 中跑 123 条指令后停下来。  
  
                        ; fuck RORDbg 0.25  
                        ; only a little trick to fool it  
  
                        .386  
                        .model  flat, stdcall  
                        option  casemap :none  
  
include                 windows.inc  
include                 kernel32.inc  
  
includelib              kernel32.lib  
  
@pushsz                 macro   sz  
                        local   \_\_0  
                        call    \_\_0  
                        db      sz, 0  
\_\_0:  
                        endm  
  
                        .code  
  
start:                  @pushsz <"kernel32.dll">  
                        call    GetModuleHandleA  
  
                        @pushsz <"OutputDebugStringA">  
                        push    eax  
                        call    GetProcAddress            
  
                        @pushsz <"i am the rordbg fucker">  
                        push    offset ExitProcess  
                        push    dword ptr \[eax+1\]               ; stolen push xxx  
                        add     eax, 5  
                        jmp     eax       
;  
;7C859B5C > $  68 34020000   PUSH    234  
;7C859B61   .  68 889E857C   PUSH    7C859E88  
;7C859B66   .  E8 6089FAFF   CALL    7C8024CB  
;7C859B6B   .  A1 CC36887C   MOV     EAX, \[7C8836CC\]  
;7C859B70   .  8945 E4       MOV     \[EBP-1C\], EAX  
;7C859B73   .  8B4D 08       MOV     ECX, \[EBP+8\]  
;7C859B76   .  898D C4FDFFFF MOV     \[EBP-23C\], ECX  
;7C859B7C   .  8365 FC 00    AND     DWORD PTR \[EBP-4\], 0  
;7C859B80   .  8BC1          MOV     EAX, ECX  
;7C859B82   .  8D70 01       LEA     ESI, \[EAX+1\]  
;7C859B85   >  8A10          MOV     DL, \[EAX\]  
;7C859B87   .  40            INC     EAX  
;7C859B88   .  84D2          TEST    DL, DL  
;7C859B8A   .^ 75 F9         JNZ     SHORT 7C859B85  
;7C859B8C   .  2BC6          SUB     EAX, ESI  
;7C859B8E   .  40            INC     EAX  
;7C859B8F   .  8985 BCFDFFFF MOV     \[EBP-244\], EAX  
;7C859B95   .  898D C0FDFFFF MOV     \[EBP-240\], ECX  
;7C859B9B   .  8D85 BCFDFFFF LEA     EAX, \[EBP-244\]  
;7C859BA1   .  50            PUSH    EAX                              ; /pArguments  
;7C859BA2   .  6A 02         PUSH    2                                ; |nArguments = 2  
;7C859BA4   .  6A 00         PUSH    0                                ; |ExceptionFlags = EXCEPTION\_CONTINUABLE  
;7C859BA6   .  68 06000140   PUSH    40010006                         ; |ExceptionCode = 40010006  
;7C859BAB   .  E8 314FFCFF   CALL    RaiseException                   ; \\RaiseException  
;7C859BB0   .  E9 52020000   JMP     7C859E07  
;  
  
                        end     start  
  
Firewall 多多改进你的工具，真得很不错。 

 4dd466d0e1e0cc123afbe504da2462ca