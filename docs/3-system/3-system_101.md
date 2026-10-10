

# 编译带 TlS 回调函数的汇编程序

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-718.htm`](https://www.pediy.com/kssd/pediy07/pediy7-718.htm)

在 EliCZ 的主页中很早就放了这样的代码。  
[`www.anticracking.sk/EliCZ/infos/TlsInAsm.zip`](http://www.anticracking.sk/EliCZ/infos/TlsInAsm.zip)  
  
下面的代码在 EliCZ 的代码中略修改了下，用 masm 可以编译通过，最初见于 OD 的 NtGlobalFlag 插件包中，这里我也修改了一下，并加了点注释。。  
  
.386  
.MODEL   flat,stdcall  
OPTION   casemap:none  
  
include windows.inc  
include user32.inc  
include kernel32.inc  
  
includelib user32.lib  
includelib kernel32.lib  
  
.data?  
\_tls\_index dd  ?  
  
OPTION    DOTNAME           
;增加一个名为.tls 的区段              
.tls   SEGMENT                          
\_tls\_start LABEL  DWORD  
 dd    80H    DUP ("slt.")   
\_tls\_end   LABEL  DWORD  
.tls   ENDS  
OPTION    NODOTNAME  
  
.data  
\_\_xl\_a dd TlsCallBack0 ;TlsCallBack1            ;回调函数，可以多个...  
\_\_xl\_z dd 0    ;null terminated list of pointers to callback procedures  
  
MsgCaption      db "Tls test",0  
Msgtls          db "in Tls",0  
msghello        db 'hello word!',0  
  
;tls 目录结构  
TLS\_DIRECTORY    STRUCT                            
 lpTlsDataStart LPDWORD ? ;copy block starting here  
 lpTlsDataEnd   LPDWORD ? ;and ending here + block (size=ZeroFillSize) filled with 0 to  
 lpTlsIndex     LPDWORD ? ;DS:\[FS:\[2CH\]\]+TlsIndex\*4  
 lpTlsCallbacks LPDWORD ? ;pointer to 0 terminated array of pointers to callbacks  
 ZeroFillSize     DWORD ? ;overall size=lpTlsDataEnd-lpTlsDataStart+ZeroFillSize  
 Characteristic   DWORD ? ;reserved  
TLS\_DIRECTORY      ENDS  
  
PUBLIC \_tls\_used ;this name is required and must be PUBLIC!!!!  
\_tls\_used TLS\_DIRECTORY <\_tls\_start, \_tls\_end, \_tls\_index, \_\_xl\_a, 0, ?>  
  
.code  
start:  
        invoke MessageBox,NULL,addr msghello,addr MsgCaption,MB\_OK  
           invoke ExitProcess,NULL  
           ret  
  
OPTION    DOTNAME    
.code .tls       ;;;把这段代码放到 tls 段中，当然你也可以不用这么做。。  
OPTION    NODOTNAME  
;tls section  
;Tls 回调函数  
TlsCallBack0   PROC    hinstImg, fdwReason, lpvReserved  
   invoke MessageBox,NULL,addr Msgtls,addr MsgCaption,MB\_OK  
   mov     dword ptr\[\_\_xl\_a\],0  
   MOV     EAX, TRUE  
   RET  
TlsCallBack0   ENDP  
  
end start  
  
  
广告![](img/710b5589904d0d2bf58a67748fca40e1.jpg)   
大话编程论坛今天开通，欢迎大家前往捧场和灌水。。。  
http://www.ptteam.com 

 4dd466d0e1e0cc123afbe504da2462ca