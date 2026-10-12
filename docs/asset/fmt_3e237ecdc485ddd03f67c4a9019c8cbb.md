
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
