
# 逆向一个直接IO硬盘的驱动

【文章标题】: 逆向一个直接IO硬盘的驱动

【文章作者】: prince

【作者邮箱】: cracker\_prince@163.com

【作者QQ号】: 812937

【软件名称】: SectorOperator.sys

【下载地址】: http://bbs.pediy.com/showthread.php?s=&threadid=30708

【加壳方式】: 无

【编写语言】: ---

【使用工具】: IDA 5.0

【操作平台】: WINDOWS XP

【软件介绍】: 利用驱动直接IO硬盘

【作者声明】: 水平有限，了解不多，仅作参考。

---

## 前言

对直接IO硬盘比较感兴趣，找到了这个程序看了看，作者的办法有限制，只能正确操作第1-63个sector，原因未知，希望哪位大侠可以指点一下。现在的硬盘已经逐渐突破了几个级别容量的限制，所以新的协议应该也可以利用来正确IO硬盘，大家可以研究研究。本人纯属入门级选手，所以各位看到有什么解释的不妥或者直接让您喷饭地方，还请不吝赐教，谢谢。

## 详细过程

IDA载入驱动，可以很清楚地找到关键部分，其他地方见源码：

---

```text
.text:004010F9 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
.text:004010F9
.text:004010F9 ; Attributes: bp-based frame
.text:004010F9
.text:004010F9 ; int \_\_stdcall sub\_4010F9(int,PIRP Irp)
.text:004010F9 sub\_4010F9      proc near               ; DATA XREF: start+24o
.text:004010F9
.text:004010F9 var\_438         = dword ptr -438h
.text:004010F9 var\_434         = byte ptr -434h
.text:004010F9 var\_430         = byte ptr -430h
.text:004010F9 var\_42C         = dword ptr -42Ch
.text:004010F9 var\_22C         = dword ptr -22Ch
.text:004010F9 var\_228         = byte ptr -228h
.text:004010F9 var\_224         = byte ptr -224h
.text:004010F9 var\_220         = dword ptr -220h
.text:004010F9 var\_20          = dword ptr -20h
.text:004010F9 var\_1C          = dword ptr -1Ch
.text:004010F9 var\_18          = dword ptr -18h
.text:004010F9 var\_14          = dword ptr -14h
.text:004010F9 var\_10          = dword ptr -10h
.text:004010F9 var\_C           = dword ptr -0Ch
.text:004010F9 var\_8           = dword ptr -8
.text:004010F9 var\_4           = dword ptr -4
.text:004010F9 Irp             = dword ptr  0Ch
.text:004010F9
.text:004010F9                 push    ebp
.text:004010FA                 mov     ebp, esp
.text:004010FC                 sub     esp, 438h
.text:00401102                 push    ebx
.text:00401103                 push    esi
.text:00401104                 push    edi
.text:00401105                 mov     [ebp+var\_18], 0C0000010h ; var\_18 = STATUS\_INVALID\_DEVICE\_REQUEST (0xC0000010)
.text:0040110C                 mov     eax, [ebp+Irp]
.text:0040110F                 mov     ecx, [eax+60h]  ; ecx = IrpStack
.text:0040110F                                         ; from wdm.h:
.text:0040110F                                         ; #define IoGetCurrentIrpStackLocation( Irp ) ( (Irp)->Tail.Overlay.CurrentStackLocation )
.text:00401112                 mov     [ebp+var\_8], ecx ; var\_8 = IrpStack
.text:00401115                 mov     edx, [ebp+var\_8]
.text:00401118                 mov     eax, [edx+0Ch]  ; eax = IrpStack->Parameters.DeviceIoControl.IoControlCode
.text:0040111B                 mov     [ebp+var\_4], eax ; var\_4 = dwControlCode
.text:0040111E                 mov     ecx, [ebp+Irp]
.text:00401121                 mov     edx, [ecx+0Ch]  ; edx = Irp->AssociatedIrp.SystemBuffer
.text:00401124                 mov     [ebp+var\_1C], edx ; var\_1C = Irp->AssociatedIrp.SystemBuffer
.text:00401127                 mov     eax, [ebp+var\_8]
.text:0040112A                 mov     ecx, [eax+8]    ; ecx = IrpStack->Parameters.DeviceIoControl.InputBufferLength
.text:0040112D                 mov     [ebp+var\_14], ecx ; var\_14 =
.text:0040112D                                         ; IrpStack->Parameters.DeviceIoControl.InputBufferLength
.text:00401130                 mov     edx, [ebp+var\_8]
.text:00401133                 mov     eax, [edx+4]    ; eax =
.text:00401133                                         ; IrpStack->Parameters.DeviceIoControl.OutputBufferLength
.text:00401136                 mov     [ebp+var\_C], eax ; var\_C =
.text:00401136                                         ; IrpStack->Parameters.DeviceIoControl.OutputBufferLength
.text:00401139                 mov     byte ptr [ebp+var\_10], 1 ; var\_10 = 1
.text:00401139                                         ; 读写操作标志
.text:0040113D                 mov     ecx, [ebp+var\_4] ; ecx = dwControlCode
.text:00401140                 mov     [ebp+var\_438], ecx ; var\_438 = dwControlCode
.text:00401146                 cmp     [ebp+var\_438], 2220C0h ; dwControlCode == 0x2220c0 ? （写操作）
.text:00401150                 jz      short loc\_401167 ; 相等则跳走
.text:00401150
.text:00401152                 cmp     [ebp+var\_438], 2220C4h ; dwControlCode == 0x2220c4 ? （读操作）
.text:0040115C                 jz      loc\_401216      ; 相等则跳走
.text:0040115C
.text:00401162                 jmp     loc\_4012C7      ; 直接跳走返回
.text:00401162
.text:00401167 ; ---------------------------------------------------------------------------
.text:00401167
.text:00401167 loc\_401167:                             ; CODE XREF: sub\_4010F9+57j
.text:00401167                 cmp     [ebp+var\_14], 202h ; InputBufferLength == 0x202 ?
.text:0040116E                 jnz     loc\_401211      ; 不相等则跳走返回
.text:0040116E
.text:00401174                 cmp     [ebp+var\_C], 0  ; OutputBufferLength == 0 ?
.text:00401178                 jnz     loc\_401211      ; 不相等则跳走返回
.text:00401178
.text:0040117E                 mov     edx, [ebp+var\_1C]
.text:00401181                 mov     [ebp+var\_20], edx ; var\_20 = Irp->AssociatedIrp.SystemBuffer
.text:00401184                 mov     eax, [ebp+var\_20]
.text:00401187                 mov     cl, [eax]
.text:00401189                 mov     [ebp+var\_224], cl ; var\_224 = SystemBuffer中的数据
.text:0040118F                 mov     edx, [ebp+var\_20]
.text:00401192                 movsx   eax, byte ptr [edx+1] ; 下面的写操作部分不写注释了，参见读过程
.text:00401196                 neg     eax
.text:00401198                 sbb     eax, eax
.text:0040119A                 and     eax, 10h
.text:0040119D                 add     eax, 0A0h
.text:004011A2                 mov     [ebp+var\_228], al
.text:004011A8                 mov     esi, [ebp+var\_20]
.text:004011AB                 add     esi, 2
.text:004011AE                 mov     ecx, 80h
.text:004011B3                 lea     edi, [ebp+var\_220]
.text:004011B9                 rep movsd
.text:004011BB                 mov     dx, 1F6h
.text:004011BF                 mov     al, [ebp+var\_228]
.text:004011C5                 out     dx, al          ; AT hard disk controller:
.text:004011C5                                         ; Drive & Head.
.text:004011C5                                         ; Read/Write: bits indicate head, drive for operation
.text:004011C6                 mov     dx, 1F2h
.text:004011CA                 mov     al, 1
.text:004011CC                 out     dx, al          ; AT hard disk controller:
.text:004011CC                                         ; Sector count.
.text:004011CC                                         ; Read/Write count of sectors for operation
.text:004011CD                 mov     dx, 1F3h
.text:004011D1                 mov     al, [ebp+var\_224]
.text:004011D7                 out     dx, al          ; AT hard disk controller:
.text:004011D7                                         ; Sector number.
.text:004011D7                                         ; Read/Write current/starting logical sector number
.text:004011D8                 mov     dx, 1F4h
.text:004011DC                 mov     al, 0
.text:004011DE                 out     dx, al          ; AT hard disk controller:
.text:004011DE                                         ; Cylinder high (bits 0-1 are bits 8-9 of 10-bit cylinder number)
.text:004011DF                 mov     dx, 1F5h
.text:004011E3                 out     dx, al          ; AT hard disk controller:
.text:004011E3                                         ; Cylinder low (bits 0-7 of 10-bit cylinder number)
```
