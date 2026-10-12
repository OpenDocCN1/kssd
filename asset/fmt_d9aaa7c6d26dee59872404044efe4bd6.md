
# PEDIY 技术之新思路（一）：简单实现记事本自动保存

> 原文：[`www.pediy.com/kssd/pediy10/85318.html`](https://www.pediy.com/kssd/pediy10/85318.html)

## 文章信息

*   **【文章标题】**: PEDIY 技术之新思路（一）简单实现记事本自动保存
*   **【简单介绍】**: 在不改动记事本原代码的前提下，另类实现给记事本添加自动保存的功能（为什么要拿这个开刀，因为它简单，作为开篇比较好，前几天 `newjueqi` 写过关于这个功能实现的 `pediy` 文章，相信大家也比较熟悉），我为什么说它是个思路呢，是这样的，我觉得很多 `pediy` 的实现大多都是在原来代码区块里面改来改去，操作繁杂不说，空间也不一定够，而且很容易出错，所以这种改动的功能放到新的区块里面会更方便，而且编译产生二进制文件，在一次写进去，比直接修改代码来得容易和方便了。而且它至少还有两个好处：
    1.  就是如果程序加了壳，这个时候你肯定不能直接改动代码，但是用这种方法却可以（你可以试下给记事本加个壳，把入口改一下，再按如下方法进行处理）。
    2.  经过如下方法处理后的程序，`peid` 什么都查不出来了，或者会误判。
*   **【文章作者】**: `moonife`
*   **【读者对象】**: 都看看吧
*   **【文章难度】**: 3/10（我自己定义的）
*   **【作者邮箱】**: `moonife@163.com`
*   **【作者 QQ 号】**: 765496322
*   **【下载地址】**: 附件
*   **【编写语言】**: `win32` `asm`
*   **【使用工具】**: `AddSection` (我自己刚写的，附件里有)
*   **【操作平台】**: `xp-sp3`

### 作者说明

我有一个不知道是好不好的习惯，就是任何事都想要改进一下，可以更快捷和方便的完成，当然很多事也就只是想想，并没有实现的能力和条件，不过现在这个我算是成功实现了吧，所以就拿出来分享一下，希望可以给各位产生点价值。说真的，我都不知道如何去很好的描述这种思路，所以还是来具体的实现过程吧，各位自己体会了！有不足之处，还恳请各位大虾指正！谢谢了

## 实现过程

### 一、实现过程简介

1.  编写实现功能的代码，产生功能代码编译后的数据文件
2.  给记事本增加一个新的区块，并把数据文件写入，并更改记事本入口地址为新区块开头，执行完功能功能代码后返回原入口地址

### 二、具体操作过程

#### 1. 编写实现功能的代码，产生功能代码编译后的数据文件

下面是实现此功能的 `win32` `asm` 代码并给出比较详细的注译

```asm
; MakeData.asm
;************************************************************
.586
.model flat,stdcall
option casemap:none
;************************************************************
include windows.inc
include user32.inc
include kernel32.inc
includelib user32.lib
includelib kernel32.lib
include  xlib.inc ;这个头文件来自 forgot & heXer 两位大牛的，主要是利用 PEB 原理搜索 API 和初始化库，
;为了更方便，我把罗云彬大哥的仿 invoke 指令宏定义加进去,具体使用见后面的 AutoSave.asm
;因为篇幅关系，这个头文件就不贴了，附件里有
;************************************************************
.data?
hOutFile dd ?
BytesWritten dd ?
.const
szCaption db 'Info',0
szContext db 'success',0
szOutFileName  db 'notepad.data',0
;************************************************************
.code
include   AutoSave.asm  ;这里把实现功能代码引入，编译并产生数据文件，下面的应该都看懂，就不讲了
start:
invoke  CreateFile,offset szOutFileName,GENERIC_READ or GENERIC_WRITE,\
         FILE_SHARE_READ,\
         NULL,CREATE_ALWAYS,\
         FILE_ATTRIBUTE_NORMAL,NULL
   mov  hOutFile,eax
invoke  WriteFile,hOutFile,offset REMOTE_CODE_START,REMOTE_CODE_LENGTH,addr BytesWritten,NULL
invoke  MessageBox,0,offset szContext,offset szCaption,MB_OK
invoke ExitProcess,0
end start
;************************************************************
;AutoSave.asm
REMOTE_CODE_START  equ this byte
;************************Infect code start************************
call  @F
    @@:
    pop  ebx
    sub  ebx,offset @B    ;relocation
initX   XFL_MAX, "user32"     ;引用 xlib.inc 里面的宏操作，功能是加载 user32.dll 库和初始化
lea eax,szContextR[ebx]
lea ecx,szCaptionR[ebx]
invokex   MessageBoxA,0,eax,ecx,MB_YESNO     ;功能开启选择对话框，注意这里是 invokex 哦，它不会进行参数检查，如果 API 进行字符处理，
;要加 A 或 W
.if eax==IDNO
push Ret_Entry[ebx]          ;返回原入口地址
   ret
.else
lea edi, [ebx + offset AutoSaveThread]
invokex CreateThread,NULL,0,edi,NULL,NULL,NULL ;产生一个线程，这个比较关键，关闭记事本后才退出，你应该想到它要干什么了吧
push Ret_Entry[ebx]
   ret
.endif

;>>>>>>>>>>>>>>>>线程定义>>>>>>>>>>>>>>>>>>>>>>>>>>>>
AutoSaveThread  proc  uses ebx edi esi ecx
 call  @F
    @@:
    pop  ebx
  sub  ebx,offset @B         ;特别注意这里也需要重定位
    initX XFL_MAX, "user32"
   .while 1                  ;无限循环
lea edi, [ebx + offset szClassNmae]
   invokex FindWindowA,edi,NULL    ;查找记事本的窗口获得句柄
   .if eax
   mov ecx,eax
    xor eax,eax
    mov ax,3                  ;保持菜单项的 ID，用资源编辑工具可以获得
invokex SendMessageA,ecx,WM_COMMAND,eax,0 ;给保存菜单项发送消息
.endif
    invokex  Sleep, 10000               ;约多长时间循环一次，你可以按需改动
   .endw
    mov eax,TRUE
    ret
    AutoSaveThread  endp

;>>>>>>>>>>>>>>>>这里定义用到的数据>>>>>>>>>>>>>>>>>>>>>>>>>>>>
szContextR db '是否开启自动保存？',0
szCaptionR db 'AutoSave',0
szClassNmae  db  'Notepad',0   ;记事本主窗口的类名
Ret_Entry dd  0100739Dh       ;记事本的原入口地址，用 lordpe 等 pe 工具就可以得到
;************************Infect code end************************
REMOTE_CODE_END    equ this byte
REMOTE_CODE_LENGTH  equ offset REMOTE_CODE_END - offset REMOTE_CODE_START
```

#### 2. 给记事本增加一个新的区块，并把数据文件写入，并更改记事本入口地址为新区块开头，执行完功能功能代码后返回原入口地址

运行附件里面的 `AddSection` ，目标文件选择记事本程序，数据文件选择第一步中产生的 `notepad.data` 文件，并选上写入数据文件和更新入口地址，填写区块名字，然后生成，会在目标文件目录下生成 `xxx@.exe` 文件，就 ok 了；如下图：

![](img/ebf2f288c4be42cd9be8b853489de65a.jpg)

> **the end**

### Ps

不足之处还请给我指正一下，在这里要特别感谢 `forgot` & `heXer` & 罗云彬 等无数大牛的无私奉献，再次感谢看雪学院！应该会有下一篇的吧，或者哪位用这种思路实现更多更好 `PEDIY`

### 附件文件清单

`AutoSave.asm`, `MakeData.asm`, `xlib.inc`, `AddSection.exe`, （最终生成的）`notepad@.exe`，还有本文档。

写于：2009-4-2

### 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `NewPediy.rar` |
| :--- | :--- |
