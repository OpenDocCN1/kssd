

# TLS 小玩法

> 原文：[`www.pediy.com/kssd/pediy10/99134.html`](https://www.pediy.com/kssd/pediy10/99134.html)

一般的，我们用 OllyDbg 调试的时候，都是设置断在 OEP（改第一个字节为 0xcc）处，而 TLS 可以在 OEP 前被执行，  
所以就可以玩玩了，就是把第一个字节再改过来，继续跑，不过应该没多大实用价值的，不过还是要分享下![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")  
直接上代码，TLS 模板来自玩命大哥的[《【成果 6.3】软件保护壳专题 - 处理 TLS 表》](http://bbs.pediy.com/showthread.php?t=67627)  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
;加链接参数：/section:.text,RWE  
.386  
.model   flat,stdcall  
option   casemap:none  
  
include windows.inc  
include user32.inc  
include kernel32.inc  
  
includelib user32.lib  
includelib kernel32.lib  
  
.data?  
dwTLS\_Index dd  ?  
  
OPTION    DOTNAME         
.tls  SEGMENT                          
TLS\_Start LABEL  DWORD  
 dd    050h    dup ("slt.")  
TLS\_End   LABEL  DWORD  
.tls   ENDS  
OPTION    NODOTNAME  
  
.data  
TLS\_CallBackStart   dd TlsCallBack0  
TLS\_CallBackEnd     dd 0  
  
g\_szTitle           db "Hello TLS",0  
g\_szInTls           db "我在 TLS 里",0  
g\_szInNormal        db "我在正常代码内",0  
PUBLIC \_tls\_used  
\_tls\_used IMAGE\_TLS\_DIRECTORY <TLS\_Start, TLS\_End, dwTLS\_Index, TLS\_CallBackStart, 0, ?>  
  
.code  
Start:  
    db 055h, 08Bh, 0ECh, 06Ah, 0FFh, 068h, 0D0h, 070h, 040h, 000h  
    db 068h, 08Ch, 032h, 040h, 000h, 064h, 0A1h, 000h, 000h, 000h  
    db 000h, 050h, 064h, 089h, 025h, 000h, 000h, 000h, 000h, 083h  
    db 0ECh, 058h, 053h, 056h, 057h, 0B8h  ;随便抠过来 VC6.0 入口的一段(24h)  
    invoke MessageBox, NULL, addr g\_szInNormal, addr g\_szTitle, MB\_OK  
    invoke ExitProcess, 1  
    ret  
  
;; TLS 的回调函数  
TlsCallBack0 proc Dllhandle : LPVOID, dwReason : DWORD, lpvReserved : LPVOID  
   mov eax, dwReason  
   cmp eax, DLL\_PROCESS\_ATTACH  
   jnz ExitTlsCallBack0  
   invoke MessageBox, NULL, addr g\_szInTls, addr g\_szTitle, MB\_OK  
    ;mov BYTE   ptr\[Start\],06Ah     ;改回 OEP 继续跑，不被调试器断在入口  
  ; cmp BYTE ptr\[Start\],0CCh      ;判断是否被下断和调试  
  ; je Bye  
    mov WORD ptr\[Start\],22EBh  ;直接 jmp 无用代码  
   mov dword ptr\[TLS\_Start\],0    
   xor eax, eax  
   inc eax  
ExitTlsCallBack0:     
   ret  
Bye:  
   invoke ExitProcess,0  
TlsCallBack0   ENDP  
  
end Start  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*