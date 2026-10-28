
# [旧帖]Nasm制作Http服务器之HelloWorld0.00雪花

这个teapoy是我学习C语言时做的一个支持C语言制作动态网页的Http服务程序，现在想深入学习汇编了，就打算把它用汇编重写一次。今天写了点赶紧拿来申请邀请码了。![](https://bbs.kanxue.com/view/img/face/16.gif)

软件基础：`Nasm` 2.07、`MinGW gcc` 4.4.0、`WindowsXP`

这里我使用了MinGW的`make`和`ld`，连接的是MinGW带的静态库。头文件是自己写的，下面都有。

因为我分的文件比较多，所以我把4个`inc`文件的代码写在一起列在下面了，若有幸拿到邀请码我会把代码整理后在其它版面陆续贴出来。

这篇代码写到用套接字接到浏览器请求并将请求显示在屏幕上。相当于制作http服务器的helloworld吧，不过因为工作原因实在没什么时间塌实下来写点东西，希望能看在汇编语言的面子上给一个邀请码![](https://bbs.kanxue.com/view/img/face/41.gif)

## main.asm

```
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;main.asm

;包含定义

%include "memory.inc"

%include "socket2.inc"

%include "ntkern.inc"

%include "teapoy.inc"

;导入表

extern \_printf

;导出表

global \_start

;配置

SOCKET\_BUFF equ 2000

;正文

segment .data

msg\_war\_port db "警告:端口号%d不正确,修改为默认的80!",10,0

msg\_war\_tryport db "Teapoy:正在尝试打开%d端口...",10,0

msg\_war\_threadmax db "Teapoy:当前线程数已经达到限制。",10,0

msg\_suc\_start db "Teapoy:成功打开%d端口。",10,0

msg\_suc\_exit db "Teapoy:成功退出。",10,0

msg\_err\_sock db "错误:Socket错误(%d)",10,0

msg\_debug db "eax=%08x,ebx=%08x,ecx=%08x,edx=%08x",10,0

segment .bss

sock\_port        resd 1        ;端口

sock\_handle        resd 1        ;句柄

sock\_buff        resd SOCKET\_BUFF;缓存在线程分配时未能及时处理的滞留请求

cur\_sockbuff        resd 1        ;sock\_buff的指针

app\_status        resd 1        ;程序运行状态

app\_threadcount resd 1        ;正在运行的线程数量

thread\_id        resd 1        ;线程id(临时变量)

wsd:        istruc WSAData

                at WSAData.Version,resw 1

                at WSAData.HeightVersion,resd 1

                at WSAData.Descripton,resb 257

                at WSAData.SystemStatus,resb 129

                at WSAData.MaxSockets,resw 1

                at WSAData.MaxUdpDg,resw 1

                at WSAData.VendorInfo,resd 1

        iend

sock\_dr:istruc SOCKADDR\_IN

                at SOCKADDR\_IN.sin\_family,resw 1

                at SOCKADDR\_IN.sin\_port,resw 1

                at SOCKADDR\_IN.sin\_addr,resd 1

                at SOCKADDR\_IN.sin\_zero,resb 8

        iend

fd:        istruc fd\_set

                at fd\_set.fd\_count,resd 1

                at fd\_set.fd\_array,resd FD\_SETSIZE

        iend

zt:        istruc timeval

                at timeval.tv\_sec,resd 1

                at timeval.tv\_usec,resd 1

        iend

segment .text

\_start:

        push ebp

        mov ebp,esp

        ;将堆栈对齐到16字节边界

        and esp,-16

        sub esp,16

        ;WSAStartup(0x0202,&wsd);

        push wsd

        push dword 0x0202

        call \_WSAStartup@8

        ;判断端口是否为0

        cmp dword [sock\_port],0

        jnz .connect

        ;是0就警告，并修改为80

        push dword [sock\_port]

        push msg\_war\_port

        call \_printf

        add esp,8

        mov dword [sock\_port],80

.connect:

        ;初始化sock\_dr

        mov dword [sock\_dr + SOCKADDR\_IN.sin\_family],AF\_INET

        ;sock\_handle = socket(AF\_INET,SOCK\_STREAM,0);

        push dword 0

        push dword SOCK\_STREAM

        push dword AF\_INET

        call \_socket@12

        mov dword [sock\_handle],eax

        ;sock\_dr.sin\_addr = htol(INADDR\_ANY);

        push INADDR\_ANY

        call \_htonl@4

        mov dword [sock\_dr + SOCKADDR\_IN.sin\_addr],eax

.tryport:

        ;sock\_dr.sin\_port = htons(sock\_port);

        push dword [sock\_port]

        call \_htons@4

        mov word [sock\_dr + SOCKADDR\_IN.sin\_port],ax

        ;提示尝试哪个端口

        push dword [sock\_port]

        push msg\_war\_tryport

        call \_printf

        add esp,8

        ;准备下一次尝试的sock\_port

        inc dword [sock\_port]

        ;eax = bind(sock\_handle,(SOCKADDR\*)&sock\_dr,sizeof(SOCKADDR))

        push SOCKADDR\_IN\_size        ;sizeof(SOCKADDR)

        push sock\_dr

        push dword [sock\_handle]

        call \_bind@12

        ;if(eax == SOCKET\_ERROR)

        cmp eax,SOCKET\_ERROR

        jz .tryport

        dec dword [sock\_port]

        ;报告已经打开端口

        push dword [sock\_port]

        push msg\_suc\_start

        call \_printf

        add esp,8

        ;开始监听

        push 10

        push dword [sock\_handle]

        call \_listen@8

        ;开始监听前最后一点点初始化工作

        mov eax,dword [sock\_handle]

        mov dword [fd + fd\_set.fd\_array + 0],eax

        mov dword [cur\_sockbuff],0

        mov dword [app\_status],1

        mov dword [app\_threadcount],0

        mov dword [zt + timeval.tv\_sec],0

        mov dword [zt + timeval.tv\_usec],500

.listen:

        cmp dword [app\_status],1

        jnz .disconect

        ;开始接受连接

        mov dword [fd + fd\_set.fd\_count],1

        ;select(1,&fdmain,NULL,NULL,&\_zt);

        push zt

        push 0

        push 0

        push fd

        push 1

        call \_select@20

        push fd

        push dword [sock\_handle]

        call \_\_\_WSAFDIsSet@8

        cmp eax,0

        jz .listen

        cmp dword [app\_threadcount],THREAD\_LIMIT

        ja .listen

        inc dword [cur\_sockbuff]

        cmp dword [cur\_sockbuff],SOCKET\_BUFF

        jnb .accept

        mov dword [cur\_sockbuff],0

.accept:

        ;accept(sock,(SOCKADDR\*)&socket\_addr,NULL);

        push 0

        push sock\_dr

        push dword[sock\_handle]

        call \_accept@12

        mov edx,sock\_buff

        add edx,dword [cur\_sockbuff]

        mov dword [edx],eax

        push thread\_id

        push 0

        push dword [edx]

        push thread\_response

        push 0

        push 0

        call \_CreateThread@24

        ;警告线程数目

        cmp dword [app\_threadcount],THREAD\_LIMIT

        jnz .listen        ;未达到限制

        push msg\_war\_threadmax

        call \_printf

        add esp,4

        jmp .listen

.disconect:

        push dword [sock\_handle]

        call \_closesocket@4

        call \_WSACleanup@0

        push msg\_suc\_exit

        call \_printf

        ;正常退出程序

        xor eax,eax

        mov esp,ebp

        pop ebp

        ret

thread\_response:

        push ebp

        mov ebp,esp

        push ebx

        push esi

        push edi

        ;原子使线程数+1

        push 1

        push app\_threadcount

        call \_InterlockedExchangeAdd@8

        mov ebx,esp

        sub esp,200        ;开辟200个字节局部内存

        ;申请动态内存并将指针临时存入ebx指向的内存（ebx指向堆栈）

        push REQMEM\_ALL

        call \_malloc

        mov dword [ebx],eax

        push 0

        push eax

        call \_memset

        add esp,12        ;上面程序将malloc和memset的参数连起来写了，所以这里一起清理

        ;recvret = recv(chsock,requestheader,REQUEST\_HEAD\_LIMIT+REQUEST\_BUFF\_LIMIT,0);

        push 0

        push REQMEM\_HEAD + REQMEM\_BUFF

        mov edx,dword [ebx]

        add edx,REQMEM\_GET + REQMEM\_ARGA

        push edx

        push dword [ebp + 8]

        call \_recv@16

        mov edx,dword [ebx]

        add edx,REQMEM\_GET + REQMEM\_ARGA

        push edx

        call \_printf

        add esp,4

        cmp eax,10

        jnb .request

        call \_WSAGetLastError@0

        push eax

        push msg\_err\_sock

        call \_printf

        add esp,8

.request:

        ;显示读到的数据

call debug

        push edx

        call \_printf

        add esp,4

        push dword [ebx]

        call \_free

        ;因为后面已经不需要大量使用堆栈了，这里就不多此一举再平衡堆栈了，等着最后函数返回前的leave自动清理。

        ;add esp,4        ;恢复free的参数占用的堆栈

        ;mov esp,ebx        ;恢复局部内存造成的堆栈不平衡   这一点是不必要的，因为函数末尾已经在清理堆栈了。

.exit:

        push eax

        ;原子使线程数-1

        push -1

        push app\_threadcount

        call \_InterlockedExchangeAdd@8

        pop eax

        pop edi

        pop esi

        pop ebx

        mov esp,ebp        ;leave

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

        push msg\_debug

        call \_printf

        add esp,4

        pop edi

        pop esi

        pop ebx

        mov esp,ebp        ;leave
```
