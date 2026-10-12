
# Nasm 制作 Http 服务器之 HelloWorld

> 原文：[`www.pediy.com/kssd/pediy12/98644.html`](https://www.pediy.com/kssd/pediy12/98644.html)

这个 `teapoy` 是我学习 C 语言时做的一个支持 C 语言制作动态网页的 Http 服务程序，现在想深入学习汇编了，就打算把它用汇编重写一次。今天写了点赶紧拿来申请邀请码了。![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")

软件基础：`Nasm 2.07`、`MinGW gcc 4.4.0`、`WindowsXP`

这里我使用了 `MinGW` 的 `make` 和 `ld`，连接的是 `MinGW` 带的静态库。头文件是自己写的，下面都有。

因为我分的文件比较多，所以我把 `4` 个 `inc` 文件的代码写在一起列在下面了，若有幸拿到邀请码我会把代码整理后在其它版面陆续贴出来。

这篇代码写到用套接字接到浏览器请求并将请求显示在屏幕上。相当于制作 http 服务器的 helloworld 吧，不过因为工作原因实在没什么时间塌实下来写点东西，希望能看在汇编语言的面子上给一个邀请码![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")

```nasm
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
main.asm
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

;包含定义
%include "memory.inc"
%include "socket2.inc"
%include "ntkern.inc"
%include "teapoy.inc"

;导入表
extern _printf

;导出表
global _start

;配置
SOCKET_BUFF equ 2000

;正文
segment .data
msg_war_port db "警告:端口号%d 不正确,修改为默认的 80!",10,0
msg_war_tryport db "Teapoy:正在尝试打开%d 端口...",10,0
msg_war_threadmax db "Teapoy:当前线程数已经达到限制。",10,0
msg_suc_start db "Teapoy:成功打开%d 端口。",10,0
msg_suc_exit db "Teapoy:成功退出。",10,0
msg_err_sock db "错误:Socket 错误(%d)",10,0
msg_debug db "eax=%08x,ebx=%08x,ecx=%08x,edx=%08x",10,0

segment .bss
sock_port resd 1 ;端口
sock_handle resd 1 ;句柄
sock_buff resd SOCKET_BUFF;缓存在线程分配时未能及时处理的滞留请求
cur_sockbuff resd 1 ;sock_buff 的指针
app_status resd 1 ;程序运行状态
app_threadcount resd 1 ;正在运行的线程数量
thread_id resd 1 ;线程 id(临时变量)

wsd: istruc WSAData
    at WSAData.Version,resw 1
    at WSAData.HeightVersion,resd 1
    at WSAData.Descripton,resb 257
    at WSAData.SystemStatus,resb 129
    at WSAData.MaxSockets,resw 1
    at WSAData.MaxUdpDg,resw 1
    at WSAData.VendorInfo,resd 1
  iend

sock_dr:istruc SOCKADDR_IN
    at SOCKADDR_IN.sin_family,resw 1
    at SOCKADDR_IN.sin_port,resw 1
    at SOCKADDR_IN.sin_addr,resd 1
    at SOCKADDR_IN.sin_zero,resb 8
  iend

fd: istruc fd_set
    at fd_set.fd_count,resd 1
    at fd_set.fd_array,resd FD_SETSIZE
  iend

zt: istruc timeval
    at timeval.tv_sec,resd 1
    at timeval.tv_usec,resd 1
  iend

segment .text

_start:
  push ebp
  mov ebp,esp
  ;将堆栈对齐到 16 字节边界
  and esp,-16
  sub esp,16

  ;WSAStartup(0x0202,&wsd);
  push wsd
  push dword 0x0202
  call _WSAStartup@8

  ;判断端口是否为 0
  cmp dword [sock_port],0
  jnz .connect

  ;是 0 就警告，并修改为 80
  push dword [sock_port]
  push msg_war_port
  call _printf
  add esp,8
  mov dword [sock_port],80

.connect:
  ;初始化 sock_dr
  mov dword [sock_dr + SOCKADDR_IN.sin_family],AF_INET

  ;sock_handle = socket(AF_INET,SOCK_STREAM,0);
  push dword 0
  push dword SOCK_STREAM
  push dword AF_INET
  call _socket@12
  mov dword [sock_handle],eax

  ;sock_dr.sin_addr = htol(INADDR_ANY);
  push INADDR_ANY
  call _htonl@4
  mov dword [sock_dr + SOCKADDR_IN.sin_addr],eax

.tryport:
  ;sock_dr.sin_port = htons(sock_port);
  push dword [sock_port]
  call _htons@4
  mov word [sock_dr + SOCKADDR_IN.sin_port],ax

  ;提示尝试哪个端口
  push dword [sock_port]
  push msg_war_tryport
  call _printf
  add esp,8

  ;准备下一次尝试的 sock_port
  inc dword [sock_port]

  ;eax = bind(sock_handle,(SOCKADDR*)&sock_dr,sizeof(SOCKADDR))
  push SOCKADDR_IN_size ;sizeof(SOCKADDR)
  push sock_dr
  push dword [sock_handle]
  call _bind@12

  ;if(eax == SOCKET_ERROR)
  cmp eax,SOCKET_ERROR
  jz .tryport
  dec dword [sock_port]

  ;报告已经打开端口
  push dword [sock_port]
  push msg_suc_start
  call _printf
  add esp,8

  ;开始监听
  push 10
  push dword [sock_handle]
  call _listen@8

  ;开始监听前最后一点点初始化工作
  mov eax,dword [sock_handle]
  mov dword [fd + fd_set.fd_array + 0],eax
  mov dword [cur_sockbuff],0
  mov dword [app_status],1
  mov dword [app_threadcount],0
  mov dword [zt + timeval.tv_sec],0
  mov dword [zt + timeval.tv_usec],500

.listen:
  cmp dword [app_status],1
  jnz .disconect

  ;开始接受连接
  mov dword [fd + fd_set.fd_count],1

  ;select(1,&fdmain,NULL,NULL,&_zt);
  push zt
  push 0
  push 0
  push fd
  push 1
  call _select@20
  push fd
  push dword [sock_handle]
  call ___WSAFDIsSet@8
  cmp eax,0
  jz .listen

  cmp dword [app_threadcount],THREAD_LIMIT
  ja .listen

  inc dword [cur_sockbuff]
  cmp dword [cur_sockbuff],SOCKET_BUFF
  jnb .accept
  mov dword [cur_sockbuff],0

.accept:
  ;accept(sock,(SOCKADDR*)&socket_addr,NULL);
  push 0
  push sock_dr
  push dword[sock_handle]
  call _accept@12
  mov edx,sock_buff
  add edx,dword [cur_sockbuff]
  mov dword [edx],eax

  push thread_id
  push 0
  push dword [edx]
  push thread_response
  push 0
  push 0
  call _CreateThread@24

  ;警告线程数目
  cmp dword [app_threadcount],THREAD_LIMIT
  jnz .listen ;未达到限制
  push msg_war_threadmax
  call _printf
  add esp,4
  jmp .listen

.disconect:
  push dword [sock_handle]
  call _closesocket@4
  call _WSACleanup@0
  push msg_suc_exit
  call _printf

  ;正常退出程序
  xor eax,eax
  mov esp,ebp
  pop ebp
  ret

thread_response:
  push ebp
  mov ebp,esp
  push ebx
  push esi
  push edi

  ;原子使线程数+1
  push 1
  push app_threadcount
  call _InterlockedExchangeAdd@8

  mov ebx,esp
  sub esp,200 ;开辟 200 个字节局部内存

  ;申请动态内存并将指针临时存入 ebx 指向的内存（ebx 指向堆栈）
  push REQMEM_ALL
  call _malloc
  mov dword [ebx],eax

  push 0
  push eax
  call _memset
  add esp,12 ;上面程序将 malloc 和 memset 的参数连起来写了，所以这里一起清理

  ;recvret = recv(chsock,requestheader,REQUEST_HEAD_LIMIT+REQUEST_BUFF_LIMIT,0);
  push 0
  push REQMEM_HEAD + REQMEM_BUFF
  mov edx,dword [ebx]
  add edx,REQMEM_GET + REQMEM_ARGA
  push edx
  push dword [ebp + 8]
  call _recv@16
  mov edx,dword [ebx]
  add edx,REQMEM_GET + REQMEM_ARGA
  push edx
  call _printf
  add esp,4

  cmp eax,10
  jnb .request
  call _WSAGetLastError@0
  push eax
  push msg_err_sock
  call _printf
  add esp,8

.request:
  ;显示读到的数据
  call debug
  push edx
  call _printf
  add esp,4

  push dword [ebx]
  call _free

  ;因为后面已经不需要大量使用堆栈了，这里就不多此一举再平衡堆栈了，等着最后函数返回前的 leave 自动清理。
  ;add esp,4 ;恢复 free 的参数占用的堆栈
  ;mov esp,ebx ;恢复局部内存造成的堆栈不平衡  这一点是不必要的，因为函数末尾已经在清理堆栈了。

.exit:
  push eax
  ;原子使线程数-1
  push -1
  push app_threadcount
  call _InterlockedExchangeAdd@8
  pop eax
  pop edi
  pop esi
  pop ebx
  mov esp,ebp ;leave
  pop ebp
  ret

debug:
  push ebp
  mov ebp,esp
  push ebx
  push esi
  push edi
  push eax
  push ebx
  push ecx
  push edx
  push msg_debug
  call _printf
  add esp,4
  pop edi
  pop esi
  pop ebx
  mov esp,ebp ;leave
  pop ebp
  ret
```

```nasm
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
inc 文件合集
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

extern _memset
extern _memcpy
extern _malloc
extern _free
extern _strcat
extern _strcpy
extern _strstr
extern _strcmp
extern _strncmp
extern _stricmp
extern _strnicmp

FD_SETSIZE equ 64
INVALID_SOCKET equ ~0
SOCKET_ERROR equ -1
SOCK_STREAM equ 1
SOCK_DGRAM equ 2
SOCK_RAW equ 3
SOCK_RDM equ 4
SOCK_SEQPACKET equ 5
TCP_NODELAY equ 0x0001

AF_UNSPEC equ 0
AF_UNIX equ 1
AF_INET equ 2
AF_IMPLINK equ 3
AF_PUP equ 4
AF_CHAOS equ 5
AF_IPX equ 6
AF_NS equ 6
AF_ISO equ 7
AF_OSI equ AF_ISO
AF_ECMA equ 8
AF_DATAKIT equ 9
```



# 网络协议常量

- `AF_CCITT` equ 10
- `AF_SNA` equ 11
- `AF_DECnet` equ 12
- `AF_DLI` equ 13
- `AF_LAT` equ 14
- `AF_HYLINK` equ 15
- `AF_APPLETALK` equ 16
- `AF_NETBIOS` equ 17
- `AF_VOICEVIEW` equ 18
- `AF_FIREFOX` equ 19
- `AF_UNKNOWN1` equ 20
- `AF_BAN` equ 21
- `AF_ATM` equ 22
- `AF_INET6` equ 23
- `AF_CLUSTER` equ 24
- `AF_12844` equ 25
- `AF_IRDA` equ 26

# IP 地址类常量

- `IN_CLASSA_NET` equ 0xff000000
- `IN_CLASSA_NSHIFT` equ 24
- `IN_CLASSA_HOST` equ 0x00ffffff
- `IN_CLASSA_MAX` equ 128
- `IN_CLASSB_NET` equ 0xffff0000
- `IN_CLASSB_NSHIFT` equ 16
- `IN_CLASSB_HOST` equ 0x0000ffff
- `IN_CLASSB_MAX` equ 65536
- `IN_CLASSC_NET` equ 0xffffff00
- `IN_CLASSC_NSHIFT` equ 8
- `IN_CLASSC_HOST` equ 0xff
- `INADDR_ANY` equ 0
- `INADDR_LOOPBACK` equ 0x7f000001
- `INADDR_BROADCAST` equ 0xffffffff
- `INADDR_NONE` equ 0xffffffff

# 结构体定义

```asm
struc WSAData
  .Version:  resw 1    ;WORD
  .HeightVersion:  resd 1    ;WORD
  .Descripton:  resb 257  ;char[WSADESCRIPTION_LEN+1];
  .SystemStatus:  resb 129  ;char[WSASYS_STATUS_LEN+1];
  .MaxSockets:  resw 1    ;unsigned short
  .MaxUdpDg:  resw 1    ;unsigned short
  .VendorInfo:  resd 1    ;char*
  alignb 2
endstruc

struc SOCKADDR_IN
  .sin_family  resw 1    ;short
  .sin_port  resw 1    ;u_short
  .sin_addr  resd 1    ;struct in_addr
  .sin_zero  resb 8    ;char[8]
endstruc

struc fd_set
  .fd_count  resd 1    ;u_int
  .fd_array  resd FD_SETSIZE  ;SOCKET [FD_SETSIZE]
endstruc

struc timeval
  .tv_sec    resd 1    ;long
  .tv_usec  resd 1    ;long
endstruc
```

# 外部函数声明

```asm
extern _WSAStartup@8
extern _socket@12
extern _closesocket@4
extern _WSACleanup@0
extern _htonl@4
extern _htons@4
extern _bind@12
extern _listen@8
extern _select@20
extern ___WSAFDIsSet@8
extern _accept@12
extern _recv@16
extern _WSAGetLastError@0
;int PASCAL ___WSAFDIsSet(SOCKET,fd_set*);
extern _CreateThread@24
extern _InterlockedExchangeAdd@8
```

# 内存分配常量

- `REQMEM_GET` equ 1024
- `REQMEM_ARGA` equ 3*1024
- `REQMEM_HEAD` equ 4096
- `REQMEM_BUFF` equ 4096
- `REQMEM_POOL` equ 4096+32
- `REQMEM_POST` equ 64*4096
- `REQMEM_RESPONSE` equ 128*4096
- `REQMEM_ALL` equ `REQMEM_GET` + `REQMEM_ARGA` + `REQMEM_HEAD` + `REQMEM_BUFF` + `REQMEM_POOL` + `REQMEM_POST` + `REQMEM_RESPONSE`
- `THREAD_LIMIT` equ 1000

# Makefile 配置

```makefile
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;makefile

Project := Teapoy
Version := 2.1.1

ld := ld
as := nasm
ar := ar

ldflags := -L "%MinGW%/lib" -lcrtdll -lkernel32 -lwsock32
asflags := -fwin32
arflags :=

Root := E:\\guobin\\Slet\\Teapoy3
Bin :=$(Root)\\Bin
Src :=$(Root)\\Code
Inc :=$(Root)\\Include
Lib :=$(Root)\\Library
Doc :=$(Root)\\Document
Cfg :=$(Root)\\Config
Tmp :=$(Root)\\Temp

asmfiles := $(shell dir $(Src)\\*.asm /B)
depend_asm := $(asmfiles:%.asm=$(Tmp)/%.o)

$(Tmp)/%.o:$(Src)/%.asm
  @$(as) -o $@ $< $(asflags) -I"$(Inc)/" -I"$(Inc)/sys/"
  @echo 已经根据$^创建了$@

.PHONY:go clean

$(Bin)/$(Project).exe: $(depend_asm)
  @$(ld) -o $@ $^ $(ldflags) -e _start
  @echo 已经根据$^创建了$@

go:$(Bin)/$(Project).exe
  @$(Bin)/$(Project).exe

clean:
  @del $(Tmp)\\*.o /q
```
