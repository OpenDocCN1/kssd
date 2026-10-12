
# GetTcpTable.asm

> 原文：[`www.pediy.com/kssd/pediy12/131669.html`](https://www.pediy.com/kssd/pediy12/131669.html)

```asm
.386
.model flat,stdcall
option casemap:none

include kernel32.inc
includelib kernel32.lib

include ws2_32.inc
includelib WS2_32.lib

include iphlpapi.inc
includelib iphlpapi.lib

.data? ;为节省空间这些变量不定义在.code 里面。
hstdout dd ?
hstdin dd ?
x dd ?
y dd ?
pTcpTable dd ? ;要定义在.data?段，不然运行出错。输出不了字符。
dwSize DWORD ?
dwRetVal DWORD ?
buffer1 DB 20 DUP (?)
bufferl db 8+4 dup (?),?;0

.code
correy db "made by correy",0
entry db 13,10,0
;showformat db "%-20s%-20s-20s-20s",0

MIB_TCP_STATE_CLOSED db "MIB_TCP_STATE_CLOSED",0
MIB_TCP_STATE_LISTEN db "MIB_TCP_STATE_LISTEN",0
MIB_TCP_STATE_SYN_SENT db "MIB_TCP_STATE_SYN_SENT",0
MIB_TCP_STATE_SYN_RCVD db "MIB_TCP_STATE_SYN_RCVD",0
MIB_TCP_STATE_ESTAB db "MIB_TCP_STATE_ESTAB",0
MIB_TCP_STATE_FIN_WAIT1 db "MIB_TCP_STATE_FIN_WAIT1",0
MIB_TCP_STATE_FIN_WAIT2 db "MIB_TCP_STATE_FIN_WAIT2",0
MIB_TCP_STATE_CLOSE_WAIT db "MIB_TCP_STATE_CLOSE_WAIT",0
MIB_TCP_STATE_CLOSING db "MIB_TCP_STATE_CLOSING",0
MIB_TCP_STATE_LAST_ACK db "MIB_TCP_STATE_LAST_ACK",0
MIB_TCP_STATE_TIME_WAIT db "MIB_TCP_STATE_TIME_WAIT",0
MIB_TCP_STATE_DELETE_TCB db "MIB_TCP_STATE_DELETE_TCB",0

showeax proc ;坚决拒绝使用 wsprintf 等类似函数。
  pushad ;坚决不使用 msvcrt.dll 等运行时库。
  mov ecx,8 ;坚决杜绝.net 与.frameset 等环境。
  xor esi,esi ;坚决杜绝 java 虚拟机。
  againp: ;此函数在驱动及 ring0 中也可以运行哟！
  dec ecx
  rol eax,4
  push eax
  and eax,0Fh
  cmp eax,9
  jg big
  add eax,30h
  mov [bufferl+esi],al
  pop eax
  inc esi
  cmp ecx,0
  jne againp
  je showp
  big:
  add eax,37h
  mov [bufferl+esi],al
  pop eax
  inc esi
  cmp ecx,0
  jne againp
  showp:
  popad
ret
showeax endp

exbuffer proc ;可能端口的高位在前，所以才有此转换函数。
push eax ;其实用交换指令也可以完成。
mov ax,word ptr bufferl+4
mov bx,word ptr bufferl+6
mov word ptr bufferl + 6,ax
mov word ptr bufferl + 4,bx
pop eax
ret
exbuffer endp

sztitle db "Active (tcp) Connections (like netstat command)",13,10,0
la db "Local Address",0,0,0,0,0,0,0,0;补充一些数据，以防止垃圾的显示。
fa db "Foreign Address",0,0,0,0,0,0,0;补充空格为最好，这样重定向是不会显示乱码。
port db "port",0,0,0,0,0,0,0,0,0,0,0
state db "state",0

notice db "注释：端口为 16 位，为四个 16 进制数。",13,10,
        "说明：端口是以 16 进制显示的。",13,10,
        "不足之处，敬请指点：",13,10,
        "QQ:112426112",13,10,
        "Email:leguanyuan at 126 dot com",13,10,
        "Homepage:http://correy.webs.com",13,10,
        "按 enter 键退出！",13,10,0

start:
invoke GetStdHandle,-10
mov hstdin,eax
invoke GetStdHandle,-11
mov hstdout,eax
invoke SetConsoleTitle,addr correy
invoke SetConsoleScreenBufferSize,hstdout,01000064h;高字是高度，低字是宽度。
invoke WriteFile,hstdout,addr sztitle,sizeof sztitle-1,addr x,0
invoke WriteFile,hstdout,addr la,20,addr x,0
invoke WriteFile,hstdout,addr port,12+1,addr x,0 ;这为何要加一呢？不加一不对齐。
invoke WriteFile,hstdout,addr fa,20,addr x,0
invoke WriteFile,hstdout,addr port,12+1,addr x,0
invoke WriteFile,hstdout,addr state,sizeof state,addr x,0
invoke WriteFile,hstdout,addr entry,2,addr x,0

invoke GetTcpTable,addr pTcpTable,addr dwSize,1
invoke GetProcessHeap
invoke HeapAlloc,eax,0,dwSize
mov pTcpTable,eax
invoke GetTcpTable,addr pTcpTable,addr dwSize,1

lea esi,pTcpTable
add esi,4

again:
  .if dword ptr [esi+4]==0 ;本地地址是零，就跳转到下一个.
    jmp next ;在这里面发现有的端口大于 65536.
  .endif ;还有的状态没有显示，是程序错误，还是有其他状态，请去掉这两个过滤试试看。
  .if dword ptr [esi+12]==0;外面的地址是零，就跳转到下一个.
    jmp next ;其实这里用两行指令就办到了。
  .endif
  invoke RtlZeroMemory,addr buffer1,20
  invoke inet_ntoa,dword ptr [esi+4]
  invoke lstrcpy,addr buffer1,eax
  invoke WriteFile,hstdout,addr buffer1,20,addr x,0;x 前不加取地址符，也能运行，但重定向到文件，只能重定向一个命令。
  invoke RtlZeroMemory,addr bufferl,sizeof bufferl
  mov eax,dword ptr [esi+8]
  call showeax
  call exbuffer
  invoke WriteFile,hstdout,offset bufferl,sizeof bufferl,addr x,0
  invoke RtlZeroMemory,addr buffer1,20
  invoke inet_ntoa,dword ptr [esi+12]
  invoke lstrcpy,addr buffer1,eax
  invoke WriteFile,hstdout,addr buffer1,20,addr x,0
  invoke RtlZeroMemory,addr bufferl,sizeof bufferl
  mov eax,dword ptr [esi+16]
  call showeax
  call exbuffer
  invoke WriteFile,hstdout,addr bufferl,sizeof bufferl,addr x,0
  mov eax,dword ptr [esi]
  .if eax==1
    invoke WriteFile,hstdout,addr MIB_TCP_STATE_CLOSED,sizeof MIB_TCP_STATE_CLOSED,addr x,0
  .elseif eax== 2
    invoke WriteFile,hstdout,addr MIB_TCP_STATE_LISTEN,sizeof MIB_TCP_STATE_LISTEN,addr x,0
  .elseif eax== 3
    invoke WriteFile,hstdout,addr MIB_TCP_STATE_SYN_SENT,sizeof MIB_TCP_STATE_SYN_SENT,addr x,0
  .elseif eax== 4
    invoke WriteFile,hstdout,addr MIB_TCP_STATE_SYN_RCVD,sizeof MIB_TCP_STATE_SYN_RCVD,addr x,0
  .elseif eax== 5
    invoke WriteFile,hstdout,addr MIB_TCP_STATE_ESTAB,sizeof MIB_TCP_STATE_ESTAB,addr x,0
  .elseif eax== 6
    invoke WriteFile,hstdout,addr MIB_TCP_STATE_FIN_WAIT1,sizeof MIB_TCP_STATE_FIN_WAIT1,addr x,0
  .elseif eax== 7
    invoke WriteFile,hstdout,addr MIB_TCP_STATE_FIN_WAIT2,sizeof MIB_TCP_STATE_FIN_WAIT2,addr x,0
  .elseif eax== 8
    invoke WriteFile,hstdout,addr MIB_TCP_STATE_CLOSE_WAIT,sizeof MIB_TCP_STATE_CLOSE_WAIT,addr x,0
  .elseif eax== 9
    invoke WriteFile,hstdout,addr MIB_TCP_STATE_CLOSING,sizeof MIB_TCP_STATE_CLOSING,addr x,0
  .elseif eax== 10
    invoke WriteFile,hstdout,addr MIB_TCP_STATE_LAST_ACK,sizeof MIB_TCP_STATE_LAST_ACK,addr x,0
  .elseif eax== 11
    invoke WriteFile,hstdout,addr MIB_TCP_STATE_TIME_WAIT,sizeof MIB_TCP_STATE_TIME_WAIT,addr x,0
  .elseif eax== 12
    invoke WriteFile,hstdout,addr MIB_TCP_STATE_DELETE_TCB,sizeof MIB_TCP_STATE_DELETE_TCB,addr x,0
  .endif
  invoke WriteFile,hstdout,addr entry,2,addr x,0
  next:
  add esi,20
  dec dword ptr pTcpTable
  cmp dword ptr pTcpTable,0
  jne again

invoke WriteFile,hstdout,addr notice,sizeof notice-1,addr x,0
invoke ReadFile,hstdin,addr buffer1,sizeof buffer1,addr x,0
invoke ExitProcess,0
end start

;made at 2011.03.23
;当然还有 GetUdpTable 等更多附加的函数。
```

### 附件

`![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg)`

`4.rar`
