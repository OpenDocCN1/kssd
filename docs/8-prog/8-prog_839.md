
# Windows XP 下直接写屏试验

> 原文：[`www.pediy.com/kssd/pediy11/123048.html`](https://www.pediy.com/kssd/pediy11/123048.html)

在 `dos` 下可以通过直接填充显示缓冲区，实现快速绘图，在 `windows` 下能否实现呢？

`windows` 下一般用户程序运行在 `ring3` 下，没法直接读写物理内存，因此只能通过运行在 `ring0` 下的驱动来实现。

`dos` 下，图形模式下显示缓冲区的地址是 `0A000H` 段，长度只有 `64k`，太小了，跨段操作很麻烦。

在 `windows` 下，显卡的显示缓冲区地址是多少？我们打开设备管理器看看，我的机器上显卡型号是 `GeForce 7050`，它占用的资源是：内存范围 `FD000000-FDFFFFFF` `D0000000-DFFFFFFF`

`FC000000-FCFFFFFF` `000A0000-000BFFFF`，看来 `dos` 下 `0a000` 段缓冲区还能用，但太小了，每次操作 `64k` 很慢，况且也不知道怎么在保护模式下换页，放弃。

剩下的三段中哪一段是和当前屏幕对应的呢？经过试验，`D0000000-DFFFFFFF` 这段就是，是连续的 `256M`，不需要换页。

下面的工作就是写一个小驱动，直接操作这段内存，让屏幕上显示一小幅 `bmp` 图片。

试验了两天，已经在 `1024*768` `32` 位真彩色模式下实现这个功能，以下是小驱动的源码：

```
; 填充显示缓冲区，显示一幅 bmp 图片的小驱动
; 作者：盛玉增 （www.aogosoft.com 电子管）

; 2010 年 10 月 20 日 WinXP masm32v8 kmdkit1.8 下调试通过。

; @echo off
; goto make

.386

.model flat, stdcall

option casemap:none

include \\masm32\\include\\w2k\\ntstatus.inc
include \\masm32\\include\\w2k\\ntddk.inc
include \\masm32\\include\\w2k\\ntoskrnl.inc

includelib \\masm32\\lib\\w2k\\ntoskrnl.lib

.code

DriverEntry proc pDriverObject:PDRIVER_OBJECT, pusRegistryPath:PUNICODE_STRING

    pushad

    invoke MmMapIoSpace,0d0000000h,0,4*1024*1024,MmNonCached ;物理地址映射为线性地址,长度 4M，返回值在 eax
              cmp eax,0   ;eax==0,失败
              jz   ta_1
              cli
              mov edi,eax    ;eax 是显示缓冲区的首地址
              mov ecx,320    ;共 320 行，每行 240 个像素
              mov esi,offset bmp_2 ;bmp 文件中的像素数据开始地址
    ta_2:      push ecx
              mov ecx,960   ;每像素 4 个字节，240*4=960,图片一行送显示缓冲区
              rep movsb
              add edi,1024*4-960  ;1024*768 显示模式
                                  ;320*240 的 bmp 图片，每行占用 960 个字节
              pop ecx
              loop ta_2
              sti

    ta_1:

        popad

  mov eax, STATUS_DEVICE_CONFIGURATION_ERROR

  ret

DriverEntry endp;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

.data

  ; 以下为 bmp 格式的 320*240 的 bmp 图片文件数据 长度 307254 bytes

bmp_1  db 66,77,54,176,4,0,0,0,0,0,54,0,0,0,40,0    ;bmp 文件头占用 54 个字节
  db 0,0,240,0,0,0,64,1,0,0,1,0,32,0,0,0
  db 0,0,0,176,4,0,0,0,0,0,0,0,0,0,0,0
  db 0,0,0,0,0,0

bmp_2 db 229,211,248,0,131,125,242,0,74,103      ;bmp_2 为像素数据开始位置，每个像素占用 4 个字节,顺序为蓝、绿、红、保留，保留字节都为 0
  db 248,0,45,88,251,0,40,66,255,0,48,71,247,0,63,103
  db 251,0,67,127,251,0,69,106,252,0,42,68,252,0,31,57
  db 255,0,41,76,250,0,44,86,247,0,41,74,254,0,32,57
  db 255,0,30,53,253,0,21,55,245,0,39,78,255,0,51,102
  db 249,0,43,100,233,0,28,74,239,0,27,63,255,0,24,60
; ...数据省略

end DriverEntry

:make

set drv=bmp1

\\masm32\\bin\\ml /nologo /c /coff %drv%.bat

\\masm32\\bin\\link /nologo /driver /base:0x10000 /align:32 /out:%drv%.sys /subsystem:native %drv%.obj

del %drv%.obj

echo.

pause

end DriverEntry

:make

set drv=bmp1

\\masm32\\bin\\ml /nologo /c /coff %drv%.bat

\\masm32\\bin\\link /nologo /driver /base:0x10000 /align:32 /out:%drv%.sys /subsystem:native %drv%.obj

del %drv%.obj

echo.

pause
```

图片在屏幕的左上角显示，发现图像是上下颠倒的。原来 `bmp` 文件中的数据是从下向上保存的，并不是从左上角到右下角，而是从左下角开始，逐行到最上一行。

为了显示美观，我的图片是事先颠倒过的。

（顺便说一句，图片的作者是云门光影，一个摄影爱好者，版权所有，图片不得用于其它目的，哈哈）

加载这个小驱动用了 `kmdkit` 中的 `beeper` 中的小程序 `scp.exe`，稍微改了一下。也可以用其它加载驱动的程序，注册这个驱动后，`start` 就能显示图片。

加载驱动的小程序源码如下：

```
; 加载驱动程序的代码 电子管（qq5611409） 2010 年 10 月 20 日在 winxp masm32v8 kmdkit1.8 下调试通过

.386

.model flat, stdcall

option casemap:none

include \\masm32\\include\\windows.inc
include \\masm32\\include\\kernel32.inc
include \\masm32\\include\\user32.inc
include \\masm32\\include\\advapi32.inc

includelib \\masm32\\lib\\kernel32.lib
includelib \\masm32\\lib\\user32.lib
includelib \\masm32\\lib\\advapi32.lib

include \\masm32\\Macros\\Strings.mac

.code

start proc

local hSCManager:HANDLE

local hService:HANDLE

local acDriverPath[MAX_PATH]:CHAR

  invoke OpenSCManager, NULL, NULL, SC_MANAGER_CREATE_SERVICE
  .if eax != NULL
    mov hSCManager, eax
    push eax
    invoke GetFullPathName, $CTA0("bmp1.sys"), sizeof acDriverPath, addr
acDriverPath, esp
      pop eax
    ; Register driver in SCM active database
    invoke CreateService, hSCManager, $CTA0("bemp1"), $CTA0("Nice Melody bmp1"),

\
        SERVICE_START + DELETE, SERVICE_KERNEL_DRIVER,
SERVICE_DEMAND_START, \
        SERVICE_ERROR_IGNORE, addr acDriverPath, NULL, NULL, NULL,

NULL, NULL
    .if eax != NULL
      mov hService, eax
      invoke StartService, hService, 0, NULL
      invoke DeleteService, hService
      invoke CloseServiceHandle, hService
    .else
      invoke MessageBox, NULL, $CTA0("注册驱动失败."), NULL, MB_ICONSTOP
    .endif
    invoke CloseServiceHandle, hSCManager
  .else
    invoke MessageBox, NULL, $CTA0("Can't connect to Service Control Manager."),

\
              NULL, MB_ICONSTOP
  .endif
  invoke ExitProcess, 0

start endp

end start
```

这里是编译后的可执行文件，运行时 `bmp1.exe` 必须和 `bmp1.sys` 在同一个目录下。显示器必须设置成 `1024*768` `32` 位真彩色模式，其它模式可能花屏，刷新一下就可以恢复原样。

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `Bmp1.rar`

8 楼的朋友真是急性子，本人写的代码太乱，本想把代码整理一下。还是算了吧，自然美吧：-）

下面是所有源码和可执行文件，还有个加载驱动的小工具。

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `Bmp2.rar`
