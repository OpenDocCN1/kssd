
# Bannermaker

> 原文：[`www.pediy.com/kssd/pediy01/forum044.htm`](https://www.pediy.com/kssd/pediy01/forum044.htm)

Bannermaker

介绍:此软件在<<软件世界>>光盘上,制作网页横幅广告及按钮软件

设中断`bpx hmemcpy`

按`F12`若干次(若按`F12`后光标行后几行便出现`ret`就接着按)

找到`call ****`

`JNZ ****`

按`F8`,进入`CALL`

按`F10`若干次便找到此段程序

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00403F3B(C)

|

:00403F3F 52                             push edx
:00403F40 C1EA02                         shr edx, 02
:00403F43 7426                           je 00403F6B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00403F61(C)

|

:00403F45 8B0E                           mov ecx, dword ptr [esi]<-----输入注册码
:00403F47 8B1F                           mov ebx, dword ptr [edi]<-----实际注册码
:00403F49 39D9                           cmp ecx, ebx
:00403F4B 7558                           jne 00403FA5
:00403F4D 4A                             dec edx
:00403F4E 7415                           je 00403F65
:00403F50 8B4E04                         mov ecx, dword ptr [esi+04]
:00403F53 8B5F04                         mov ebx, dword ptr [edi+04]
:00403F56 39D9                           cmp ecx, ebx
:00403F58 754B                           jne 00403FA5
:00403F5A 83C608                         add esi, 00000008
:00403F5D 83C708                         add edi, 00000008
:00403F60 4A                             dec edx
:00403F61 75E2                           jne 00403F45
:00403F63 EB06                           jmp 00403F6B
```

本想追踪注册码的生成,没想到是固定的注册码,而与用户名无关

用户名:任意

注册码:6212104512252A
