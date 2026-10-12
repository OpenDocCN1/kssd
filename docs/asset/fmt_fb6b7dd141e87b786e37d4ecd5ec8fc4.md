
# 逆向一个直接 IO 硬盘的驱动

> **原文**：[`www.pediy.com/kssd/pediy09/pediy09-80.htm`](https://www.pediy.com/kssd/pediy09/pediy09-80.htm)
>
> **【文章标题】**: 逆向一个直接 IO 硬盘的驱动  
> **【文章作者】**: prince  
> **【作者邮箱】**: `cracker_prince@163.com`  
> **【作者 QQ 号】**: 812937  
> **【软件名称】**: `SectorOperator.sys`  
> **【下载地址】**: `http://bbs.pediy.com/showthread.php?s=&threadid=30708`  
> **【加壳方式】**: 无  
> **【编写语言】**: ---  
> **【使用工具】**: `IDA 5.0`  
> **【操作平台】**: `WINDOWS XP`  
> **【软件介绍】**: 利用驱动直接 IO 硬盘  
> **【作者声明】**: 水平有限，了解不多，仅作参考。  
>
> **前言**
>
> 对直接 IO 硬盘比较感兴趣，找到了这个程序看了看，作者的办法有限制，只能正确操作第 1-63 个 sector，原因未知，希望哪位大侠可以指点一下。现在的硬盘已经逐渐突破了几个级别容量的限制，所以新的协议应该也可以利用来正确 IO 硬盘，大家可以研究研究。本人纯属入门级选手，所以各位看到有什么解释的不妥或者直接让您喷饭地方，还请不吝赐教，谢谢。

**详细过程**

`IDA` 载入驱动，可以很清楚地找到关键部分，其他地方见源码：

```
.text:004010F9  ; *************** S U B R O U T I N E ******************************
.text:004010F9

.text:004010F9  ; Attributes: bp-based frame
.text:004010F9

.text:004010F9  ; int __stdcall sub_4010F9(int,PIRP Irp)
.text:004010F9 sub_4010F9      proc near                ; DATA XREF: start+24o
.text:004010F9

.text:004010F9 var_438         = dword ptr -438h
.text:004010F9 var_434         = byte ptr  -434h
.text:004010F9 var_430         = byte ptr  -430h
.text:004010F9 var_42C         = dword ptr -42Ch
.text:004010F9 var_22C         = dword ptr -22Ch
.text:004010F9 var_228         = byte ptr  -228h
.text:004010F9 var_224         = byte ptr  -224h
.text:004010F9 var_220         = dword ptr -220h
.text:004010F9 var_20          = dword ptr -20h
.text:004010F9 var_1C          = dword ptr -1Ch
.text:004010F9 var_18          = dword ptr -18h
.text:004010F9 var_14          = dword ptr -14h
.text:004010F9 var_10          = dword ptr -10h
.text:004010F9 var_C           = dword ptr -0Ch
.text:004010F9 var_8           = dword ptr -8
.text:004010F9 var_4           = dword ptr -4
.text:004010F9 Irp             = dword ptr  0Ch
.text:004010F9

.text:004010F9               push    ebp
.text:004010FA               mov     ebp, esp
.text:004010FC               sub     esp, 438h
.text:00401102               push    ebx
.text:00401103               push    esi
.text:00401104               push    edi
.text:00401105               mov     [ebp+var_18], 0C0000010h ; var_18 = STATUS_INVALID_DEVICE_REQUEST (0xC0000010)
.text:0040110C               mov     eax, [ebp+Irp]
.text:0040110F               mov     ecx, [eax+60h]  ; ecx = IrpStack
.text:0040110F                                ; from wdm.h:
.text:0040110F                                ; #define IoGetCurrentIrpStackLocation( Irp ) ( (Irp)->Tail.Overlay.CurrentStackLocation )
.text:00401112               mov     [ebp+var_8], ecx ; var_8 = IrpStack
.text:00401115               mov     edx, [ebp+var_8]
.text:00401118               mov     eax, [edx+0Ch]  ; eax = IrpStack->Parameters.DeviceIoControl.IoControlCode
.text:0040111B               mov     [ebp+var_4], eax ; var_4 = dwControlCode
.text:0040111E               mov     ecx, [ebp+Irp]
.text:00401121               mov     edx, [ecx+0Ch]  ; edx = Irp->AssociatedIrp.SystemBuffer
.text:00401124               mov     [ebp+var_1C], edx ; var_1C = Irp->AssociatedIrp.SystemBuffer
.text:00401127               mov     eax, [ebp+var_8]
.text:0040112A               mov     ecx, [eax+8]    ; ecx = IrpStack->Parameters.DeviceIoControl.InputBufferLength
.text:0040112D               mov     [ebp+var_14], ecx ; var_14 = 
.text:0040112D                                ; IrpStack->Parameters.DeviceIoControl.InputBufferLength
.text:00401130               mov     edx, [ebp+var_8]
.text:00401133               mov     eax, [edx+4]    ; eax = 
.text:00401133                                ; IrpStack->Parameters.DeviceIoControl.OutputBufferLength
.text:00401136               mov     [ebp+var_C], eax ; var_C = 
.text:00401136                                ; IrpStack->Parameters.DeviceIoControl.OutputBufferLength
.text:00401139               mov     byte ptr [ebp+var_10], 1 ; var_10 = 1
.text:00401139                                ; 读写操作标志
.text:0040113D               mov     ecx, [ebp+var_4] ; ecx = dwControlCode
.text:00401140               mov     [ebp+var_438], ecx ; var_438 = dwControlCode
.text:00401146               cmp     [ebp+var_438], 2220C0h ; dwControlCode == 0x2220c0 ? （写操作）
.text:00401150               jz      short loc_401167 ; 相等则跳走
.text:00401150
.text:00401152               cmp     [ebp+var_438], 2220C4h ; dwControlCode == 0x2220c4 ? （读操作）
.text:0040115C               jz      loc_401216      ; 相等则跳走
.text:0040115C
.text:00401162               jmp     loc_4012C7      ; 直接跳走返回
.text:00401162
.text:00401167 ; ----------------------------------------------------------------
.text:00401167
.text:00401167 loc_401167:                 ; CODE XREF: sub_4010F9+57j
.text:00401167               cmp     [ebp+var_14], 202h ; InputBufferLength == 0x202 ?
.text:0040116E               jnz     loc_401211      ; 不相等则跳走返回
.text:0040116E
.text:00401174               cmp     [ebp+var_C], 0   ; OutputBufferLength == 0 ?
.text:00401178               jnz     loc_401211      ; 不相等则跳走返回
.text:00401178
.text:0040117E               mov     edx, [ebp+var_1C]
.text:00401181               mov     [ebp+var_20], edx ; var_20 = Irp->AssociatedIrp.SystemBuffer
.text:00401184               mov     eax, [ebp+var_20]
.text:00401187               mov     cl, [eax]
.text:00401189               mov     [ebp+var_224], cl ; var_224 = SystemBuffer 中的数据
.text:0040118F               mov     edx, [ebp+var_20]
.text:00401192               movsx   eax, byte ptr [edx+1] ; 下面的写操作部分不写注释了，参见读过程
.text:00401196               neg     eax
.text:00401198               sbb     eax, eax
.text:0040119A               and     eax, 10h
.text:0040119D               add     eax, 0A0h
.text:004011A2               mov     [ebp+var_228], al
.text:004011A8               mov     esi, [ebp+var_20]
.text:004011AB               add     esi, 2
.text:004011AE               mov     ecx, 80h
.text:004011B3               lea     edi, [ebp+var_220]
.text:004011B9               rep movsd
.text:004011BB               mov     dx, 1F6h
.text:004011BF               mov     al, [ebp+var_228]
.text:004011C5               out     dx, al           ; AT hard disk controller:
.text:004011C5                                ; Drive & Head.
.text:004011C5                                ; Read/Write: bits indicate head, drive for operation
.text:004011C6               mov     dx, 1F2h
.text:004011CA               mov     al, 1
.text:004011CC               out     dx, al           ; AT hard disk controller:
.text:004011CC                                ; Sector count.
.text:004011CC                                ; Read/Write count of sectors for operation
.text:004011CD               mov     dx, 1F3h
.text:004011D1               mov     al, [ebp+var_224]
.text:004011D7               out     dx, al           ; AT hard disk controller:
```
