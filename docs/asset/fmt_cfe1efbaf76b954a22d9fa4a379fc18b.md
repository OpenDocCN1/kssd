
# 反汇编代码片段：调试器检测与地址解析

## 数据段：设备名称定义

以下为全局变量及数据库声明，用于定义调试器检测所需的设备名称：

```
SDPI:0048289F a_GlobalFilemon db '\\.\Global\FILEMON', 0
SDPI:004828B2 a_Regmon db '\\.\REGMON', 0
SDPI:004828BD a_GlobalRegmon db '\\.\Global\REGMON', 0
SDPI:004828CF a_Filevxd_vxd db '\\.\FILEVXD.VXD', 0
SDPI:004828DF a_Regvxd_vxd db '\\.\REGVXD.VXD', 0
SDPI:004828EE a_Trw db '\\.\TRW', 0
SDPI:004828F6 a_Trwdebug db '\\.\TRWDEBUG', 0
SDPI:00482903 a_Icedump db '\\.\ICEDUMP', 0
SDPI:0048290F a_Frogsice db '\\.\FROGSICE', 0
SDPI:0048291C a_Iceext db '\\.\IceExt', 0
SDPI:00482927 a_Rvtracerdevice0 db '\\.\RvtracerDevice0', 0
```

## 子过程：CFCHKDBG_48293B

该子过程主要利用 `CreateFile` 函数检测是否存在调试器。注释指出这是一种常见的反调试手段。

### 局部变量声明

```
SDPI:0048293B var_7C = dword ptr -7Ch
SDPI:0048293B var_60 = dword ptr -60h
SDPI:0048293B var_5C = dword ptr -5Ch
SDPI:0048293B var_44 = dword ptr -44h
SDPI:0048293B var_40 = dword ptr -40h
SDPI:0048293B var_3C = dword ptr -3Ch
SDPI:0048293B var_38 = dword ptr -38h
SDPI:0048293B var_34 = dword ptr -34h
SDPI:0048293B var_30 = dword ptr -30h
SDPI:0048293B var_2C = dword ptr -2Ch
SDPI:0048293B var_28 = dword ptr -28h
SDPI:0048293B var_24 = dword ptr -24h
SDPI:0048293B var_20 = dword ptr -20h
SDPI:0048293B var_1C = dword ptr -1Ch
SDPI:0048293B var_18 = dword ptr -18h
SDPI:0048293B var_14 = dword ptr -14h
SDPI:0048293B var_10 = dword ptr -10h
SDPI:0048293B var_C = dword ptr -0Ch
SDPI:0048293B var_8 = dword ptr -8
SDPI:0048293B var_4 = dword ptr -4
```

### 代码逻辑

首先分配栈空间并保存寄存器，随后获取一系列调试相关符号的绝对地址，最后进入循环结构进行调试器检测。

```
SDPI:0048293B sub esp, 44h
SDPI:0048293E push ebx
SDPI:0048293F push ebp
SDPI:00482940 push esi
SDPI:00482941 push edi
SDPI:00482942 mov ebx, ecx

; 获取 strSICE 的实际地址
SDPI:00482944 push offset strSICE
SDPI:00482949 call GetAbsAddress

; 获取 NTICE 的实际地址
SDPI:0048294E push offset NTICE
SDPI:00482953 mov [esp+58h+var_40], eax
SDPI:00482957 call GetAbsAddress

; 获取 SIWDEBUG 的实际地址
SDPI:0048295C push offset SIWDEBUG
SDPI:00482961 mov [esp+58h+var_3C], eax
SDPI:00482965 call GetAbsAddress

; 获取 SIWVID 的实际地址
SDPI:0048296A push offset SIWVID
SDPI:0048296F mov [esp+58h+var_38], eax
SDPI:00482973 call GetAbsAddress

; 获取 FILEMON 的实际地址
SDPI:00482978 push offset FILEMON
SDPI:0048297D mov [esp+58h+var_34], eax
SDPI:00482981 call GetAbsAddress

; 获取 GlobalFILEMON 的实际地址
SDPI:00482986 push offset GlobalFILEMON
SDPI:0048298B mov [esp+58h+var_30], eax
SDPI:0048298F call GetAbsAddress

; 获取 REGMON 的实际地址
SDPI:00482994 push offset REGMON
SDPI:00482999 mov [esp+58h+var_2C], eax
SDPI:0048299D call GetAbsAddress

; 获取 GlobalREGMON 的实际地址
SDPI:004829A2 push offset GlobalREGMON
SDPI:004829A7 mov [esp+58h+var_28], eax
SDPI:004829AB call GetAbsAddress

; 获取 FILEVXD_VXD 的实际地址
SDPI:004829B0 push offset FILEVXD_VXD
SDPI:004829B5 mov [esp+58h+var_24], eax
SDPI:004829B9 call GetAbsAddress

; 获取 REGVXD_VXD 的实际地址
SDPI:004829BE push offset REGVXD_VXD
SDPI:004829C3 mov [esp+58h+var_20], eax
SDPI:004829C7 call GetAbsAddress

; 获取 TRW 的实际地址
SDPI:004829CC push offset TRW
SDPI:004829D1 mov [esp+58h+var_1C], eax
SDPI:004829D5 call GetAbsAddress

; 获取 TRWDEBUG 的实际地址
SDPI:004829DA push offset TRWDEBUG
SDPI:004829DF mov [esp+58h+var_18], eax
SDPI:004829E3 call GetAbsAddress

; 获取 ICEDUMP 的实际地址
SDPI:004829E8 push offset ICEDUMP
SDPI:004829ED mov [esp+58h+var_14], eax
SDPI:004829F1 call GetAbsAddress

; 获取 FROGSICE 的实际地址
SDPI:004829F6 push offset FROGSICE
SDPI:004829FB mov [esp+58h+var_10], eax
SDPI:004829FF call GetAbsAddress

; 获取 IceExt 的实际地址
SDPI:00482A04 push offset IceExt
SDPI:00482A09 mov [esp+58h+var_C], eax
SDPI:00482A0D call GetAbsAddress

; 获取 RvtracerDevice0 的实际地址
SDPI:00482A12 push offset RvtracerDevice0
SDPI:00482A17 mov [esp+58h+var_8], eax
SDPI:00482A1B call GetAbsAddress

; 初始化循环变量
SDPI:00482A20 mov [esp+54h+var_4], eax
SDPI:00482A24 mov [esp+54h+var_44], 6
SDPI:00482A2C mov ebp, 3

loc_482A31:
SDPI:00482A31 xor edi, edi
SDPI:00482A33 lea esi, [esp+54h+var_40]

loc_482A37:
SDPI:00482A37 mov eax, [esi]

; 调用 CreateFileA 尝试打开设备句柄
SDPI:00482A39 push 0               ; hTemplateFile
SDPI:00482A3B push 80h             ; dwFlagsAndAttributes
SDPI:00482A40 push ebp             ; dwCreationDisposition
SDPI:00482A41 push 0               ; lpSecurityAttributes
SDPI:00482A43 push 1               ; dwShareMode
SDPI:00482A45 push 80000000h       ; dwDesiredAccess
SDPI:00482A4A push eax             ; lpFileName
SDPI:00482A4B call CreateFileA

; 检查返回句柄是否有效
SDPI:00482A50 cmp eax, 0FFFFFFFFh
SDPI:00482A53 jz short loc_482A6D

; 条件跳转逻辑
SDPI:00482A55 cmp edi, 4
SDPI:00482A58 jge short loc_482A65
SDPI:00482A5A cmp edi, 7
SDPI:00482A5D jle short loc_482A65

; 写入标记
SDPI:00482A5F mov [ebx+31Ch], ebp

loc_482A65:
; 关闭句柄
SDPI:00482A65 push eax             ; hObject
SDPI:00482A66 call CloseHandle
SDPI:00482A6B jmp short loc_482A7D
```
