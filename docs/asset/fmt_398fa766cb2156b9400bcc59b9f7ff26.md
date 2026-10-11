
# \[转载\] - Vmware 的后门

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-669.htm`](https://www.pediy.com/kssd/pediy07/pediy7-669.htm)

## Vmware 的后门 (1) - \[技术\]

Vmware 的后门

### 后门简介

Vmware 后门是 `vmware` 和 `vmware tools` 通信的一个接口。例如，`vmware-checkvm` 程序就是利用这个后门检测自己是否运行在 `vmware` 里。

这个后门开在 `IO` 端口 `0x5658`。利用这个后门时，必需：

* `EAX = 0x564D5868 ( "VMXh" )`
* `EBX` 为参数，一般不用。
* `ECX` 低 16 位为功能号。其实是一个函数数组的索引。`Vmware` 调用对应的函数处理后门请求。 这个函数数组共有 36 个元素，但某些没有定义。`ECX` 的高 16 位为功能参数。
* `EDX = 0x5658 ( "VX" )`，为 `IO` 端口号。

通过读端口 `(in)` 命令调用后门。

### 后门详细描述

下表列出后门的各个功能。

0. 未定义
1. `getMhz` 得到 CPU 速率
2. `APM` 函数族
3. `getDiskGeo`
4. `getPtrLocation`
5. `setPtrLocation`
6. 得到宿主机剪贴板数据长度
7. 读宿主机剪贴板数据
8. 设置宿主机剪贴板数据长度
9. 向宿主机剪贴板写数据
10. 得到 `vmware` 版本
11. 取设备信息
12. 连接或断开设备
13. 取 `GUI` 配置信息
14. 设置 `GUI` 配置信息
15. 取宿主机屏幕分辨率
16. 未定义
17. 未定义
18. `osNotFound, vmware` 提示插入引导盘
19. `GetBiosUUID`
20. 取虚拟机内存大小
21. 未定义
22. `OS2` 系统用到的一个函数
23. `getTime`，取宿主机时间
24. `stopCatchup`
25. 未定义
26. 未定义
27. 未定义
28. `initScsiIoprom`
29. 未定义
30. `Message`，通道函数族
31. `rsvd0`
32. `rsvd1`
33. `rsvd2`
34. `ACPID` 函数
35. 未定义

### 应用

`vmcall.s`

`vmcall.s` 为 `vm.c` 提供 `vmcall(uint32_t out[4],int cmd,uint32_t param)` 函数，实现调用后门功能

```
        .text
        .align 2
.globl vmcall
        .type   vmcall,@function

vmcall:
        pushl   %ebp
        movl    %esp, %ebp
        movl 0x8(%ebp),%eax
        push %edi
        push %ebx
        push %ecx
        push %edx
        mov %eax,%edi
        mov $0x564d5868,%edx
        mov %edx,%eax
        mov 0xc(%ebp),%edx
        mov 0x10(%ebp),%ebx
        mov %edx,%ecx
        mov $0x5658,%edx
        in (%dx),%eax
        mov %eax,0x0(%edi)
        mov %ebx,0x4(%edi)
        mov %ecx,0x8(%edi)
        pop %edx
[原文出处] - [`www.blogbus.com/blogbus/blog...p;pg=8&cat=`](http://www.blogbus.com/blogbus/blog/index.php?blogid=4982&pg=8&cat=) 

pop %ecx
        pop %ebx
        pop %edi
        leave
        ret
```

`vm.c`

`vm.c` 为主程序。命令行参数为后门功能号

```
#include <stdio.h>

#include <signal.h>

#include <stdint.h>

extern uint32_t vmcall(uint32_t buf[4],int func,uint32_t arg);

void segfault(int seg)
{
        fprintf(stderr,"vmcall failed\n");
        _exit(1);
}

int main(int argc,char **argv)
{
        int i;
        uint32_t buf[4];
        int cmd;
        if (argc == 1)
        {
                puts("arg.......");
                return 1;
        }
        cmd = atoi(argv[1]);
        signal(SIGSEGV,segfault);
        memset(buf,0,sizeof(buf));
        vmcall(buf,cmd,0);
        printf("%x: %x-%x-%x-%x\n",cmd,buf[0],buf[1],buf[2],buf[3]);
        return 0;
}
```

### 举例

编译： `gcc –g –o vm vm.c vmcall.s`

1.  1 号调用取 CPU 速率

    `$ ./vm 1`

    `1: 69f-0-1-0`

    `69f` 十进制 `1695`，CPU 速率为 `1695Mhz`

2.  10 号调用取 `vmware` 版本，也用来判断是否运行在 `vmware` 里。

    `$ ./vm 10`

    `a: 6-564d5868-4-0`

    `6` 是 `vmware` 版本号。注意这和 `about` 里看到的不同。`4` 表示是 `vmware workstation`，其它可能取值有：

    * `2` `ESX Server`
    * `1` `Express`
    * `3` `GSX Server`

3.  15 号调用取宿主机屏幕分辨率

    `$ ./vm 15`

    `f: 4000300-0-f-0`

    分辨率是 `0x400 * 0x300`，即 `1024 * 768`

4.  28 号调用 `vmware` 的实现有问题，导致我的 `vmware workstation 4.0.0.4460` 立刻崩溃。原因 `Vmware` 试图去读 `0x14` 的内存地址，不过因为没有用户输入，无法利用。

### 附录：分析工具和环境

`Vmware 4.0.0.4460` 和 `vmware 4.0.5.6030`。宿主机 `Windows 2003 Standard`。

`GuestOS RedHat 8 (Kernel 2.6.2, gcc 3.2)`。

`IDApro` 用于静态分析，`OllyDBG` 用于动态调试。

静态分析 `vmware-checkvm` 程序发现这个后门。

静态分析 + 动态调试得到各功能信息。

```
; ********************************************************************
; *            检测 VMMVARE              *
; ********************************************************************

MAGIC           equ 564D5868h           ;VMXh
CMD_PARAM       equ 0
CMD_NUM         equ 0Ah
PORT_NUM        equ 5658h               ;VX          

CHK_VMMVARE:
pushad
assume esi:ptr EXCEPTION_RECORD,edi:ptr CONTEXT
xor  esi, esi
lea  eax,[esp-8]
xchg  eax, dword ptr fs:[esi]
call  @F
CHK_VMMVARE_SEH  PROC C uses ebx esi edi pExcept,pFrame,pContext,pDispatch
        xor     eax,eax
        mov     esi,pExcept
        mov     edi,pContext
        test    [esi].ExceptionFlags,7
        jne     _continue_search
        cmp     [esi].ExceptionCode,STATUS_PRIVILEGED_INSTRUCTION
        jne     _continue_search
        inc     [edi].regEip    ;eip++
        jmp     _pure_exit
_continue_search:
        inc  eax
_pure_exit:
        ret
CHK_VMMVARE_SEH    ENDP
@@:
        push    eax
        MOV     EAX,MAGIC
        MOV     EBX,CMD_PARAM
        MOV     ECX,CMD_NUM
        MOV     DX,PORT_NUM
        IN      EAX,DX
        XOR     EAX,EAX
        ;unistall SEH frame
        pop     fs:[eax]
        add     esp,4
        CMP     EBX,MAGIC
        JNZ     @F
        INC     EAX
      @@:
    test  eax, eax
    .if  !zero?
      jmp @import_bute_exit; exit
    .endif
    assume esi:nothing,edi:nothing
popad
CHK_VMMVARE_END:
ret
```

4dd466d0e1e0cc123afbe504da2462ca
