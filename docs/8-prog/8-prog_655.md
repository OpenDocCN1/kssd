
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



```asm
.text:004011D7                  ; Sector number.  
.text:004011D7                  ; Read/Write current/starting logical sector number  
.text:004011D8                  mov     dx, 1F4h  
.text:004011DC                  mov     al, 0  
.text:004011DE                  out     dx, al          ; AT hard disk controller:  
.text:004011DE                  ; Cylinder high (bits 0-1 are bits 8-9 of 10-bit cylinder number)  
.text:004011DF                  mov     dx, 1F5h  
.text:004011E3                  out     dx, al          ; AT hard disk controller:  
.text:004011E3                  ; Cylinder low (bits 0-7 of 10-bit cylinder number)  
.text:004011E4                  mov     dx, 1F7h  
.text:004011E8                  mov     al, 30h  
.text:004011EA                  out     dx, al          ; AT hard disk  
.text:004011EA                  ; command register:  
.text:004011EA                  ; 1?H = Restore to cylinder 0  
.text:004011EA                  ; 7?H = Seek to cylinder  
.text:004011EA                  ; 2?H = Read sector  
.text:004011EA                  ; 3xH = Write sector  
.text:004011EA                  ; 50H = Format track  
.text:004011EA                  ; 4xH = verify read  
.text:004011EA                  ; 90H = diagnose  
.text:004011EA                  ; 91H = set parameters for drive  
.text:004011EA  
.text:004011EB  
.text:004011EB loc_4011EB:                  ; CODE XREF: sub_4010F9+F5j  
.text:004011EB                  in      al, dx          ; AT hard disk  
.text:004011EB                  ; status register bits:  
.text:004011EB                  ; 0: 1=prev cmd error  
.text:004011EB                  ; 2: Corrected data  
.text:004011EB                  ; 3: Data Request. Buffer is busy  
.text:004011EB                  ; 4: Seek completed  
.text:004011EB                  ; 5: Write fault  
.text:004011EB                  ; 6: Drive ready (unless bit 4=0)  
.text:004011EB                  ; 7: Busy  
.text:004011EC                  test    al, 8  
.text:004011EE                  jz      short loc_4011EB  
.text:004011EE  
.text:004011F0                  xor     ecx, ecx  
.text:004011F2                  mov     cx, 100h  
.text:004011F6                  lea     esi, [ebp+var_220]  
.text:004011FC                  mov     dx, 1F0h  
.text:00401200                  cli  
.text:00401201                  cld  
.text:00401202                  rep outsw  
.text:00401205                  sti  
.text:00401206                  mov     [ebp+var_18], 0 ; status = NT_SUCCESS (0)  
.text:0040120D                  mov     byte ptr [ebp+var_10], 0 ; var_10 = 0  
.text:0040120D                  ; 表示是写操作  
.text:0040120D  
.text:00401211  
.text:00401211 loc_401211:                  ; CODE XREF: sub_4010F9+75j  
.text:00401211                  ; sub_4010F9+7Fj  
.text:00401211                  jmp     loc_4012C7  
.text:00401211  
.text:00401216 ; -------------------------------------------------------------------------  
.text:00401216  
.text:00401216 loc_401216:                  ; CODE XREF: sub_4010F9+63j  
.text:00401216                  cmp     [ebp+var_14], 2 ; InputBufferLength == 2 ?  
.text:0040121A                  jnz     loc_4012C7      ; 不相等则跳走返回  
.text:0040121A  
.text:00401220                  cmp     [ebp+var_C], 200h ; OutputBufferLength == 0x200 ?  
.text:00401227                  jnz     loc_4012C7      ; 不相等则跳走返回  
.text:00401227  
.text:0040122D                  mov     ecx, [ebp+var_1C] ; ecx = Irp->AssociatedIrp.SystemBuffer  
.text:00401230                  mov     [ebp+var_22C], ecx ; var_22C = Irp->AssociatedIrp.SystemBuffer  
.text:00401236                  mov     edx, [ebp+var_22C] ; edx = Irp->AssociatedIrp.SystemBuffer   (InputBuffer)  
.text:0040123C                  mov     al, [edx]        ; edx = *InputBuffer (要读取的扇区号码)  
.text:0040123E                  mov     [ebp+var_430], al ; var_430 = 目标扇区号  
.text:00401244                  mov     ecx, [ebp+var_22C]  
.text:0040124A                  movsx   edx, byte ptr [ecx+1] ; 取 InputBuffer 的第 2 个字节 （硬盘号？）  
.text:0040124E                  neg     edx             ; 取 edx 补码  
.text:00401250                  sbb     edx, edx        ; 带借位减法  
.text:00401252                  and     edx, 10h        ; edx &= 0x10  
.text:00401255                  add     edx, 0A0h       ; edx += 0xA0  
.text:0040125B                  mov     [ebp+var_434], dl ;  
.text:00401261                  mov     al, [ebp+var_434]  
.text:00401267                  mov     dx, 1F6h        ; 指定 0x1f6 端口  
.text:0040126B                  out     dx, al          ; 将计算结果写入 0x1f6 端口中  
.text:0040126B                  ;  
.text:0040126B                  ; 1f6h----低四位是起始 LBA 的 bit24-bit27，  
.text:0040126B                  ; 第五位是设备选择，0 是 master，1 是 slave.  
.text:0040126B                  ; 我们读的是 master，应设为 0，第七位设为 1 表示使用 LBA，  
.text:0040126B                  ; 现在硬盘都使用 LBA，应置起来。其他两个 bit 要求为 1  
.text:0040126B                  ;  
.text:0040126B                  ; AT hard disk controller:  
.text:0040126B                  ; Drive & Head.  
.text:0040126B                  ; Read/Write: bits indicate head, drive for operation  
.text:0040126B                  ;  
.text:0040126B                  ;  
.text:0040126C                  mov     al, 1           ; al = 1  
.text:0040126E                  mov     dx, 1F2h        ; 指定 0x1f2 端口  
.text:00401272                  out     dx, al          ; 1f2h----操作的扇区个数  
.text:00401272                  ;  
.text:00401272                  ; AT hard disk controller:  
.text:00401272                  ; Sector count.  
.text:00401272                  ; Read/Write count of sectors for operation  
.text:00401272                  ;  
.text:00401272                  ;  
.text:00401273                  mov     al, [ebp+var_430] ; al = 目标扇区号  
.text:00401279                  mov     dx, 1F3h        ; 1f3h----起始 LBA 的 bit0-bit7  
.text:0040127D                  out     dx, al          ; 设置目标扇区号  
.text:0040127D                  ;  
.text:0040127D                  ; AT hard disk controller:  
.text:0040127D                  ; Sector number.  
.text:0040127D                  ; Read/Write current/starting logical sector number  
.text:0040127D                  ;  
.text:0040127D                  ;  
.text:0040127E                  xor     al, al          ; al = 0  
.text:00401280                  mov     dx, 1F4h        ; 1f4h----起始 LBA 的 bit8-bit15  
.text:00401284                  out     dx, al          ; AT hard disk controller:  
```



```
.text:00401284                         ; Cylinder high (bits 0-1 are bits 8-9 of 10-bit cylinder number)

.text:00401284                         ;

.text:00401284                         ;

.text:00401285                 mov     dx, 1F5h        ; 1f5h----起始 LBA 的 bit16-bit23

.text:00401289                 out     dx, al          ; AT hard disk controller:

.text:00401289                         ; Cylinder low (bits 0-7 of 10-bit cylinder number)

.text:00401289                         ;

.text:00401289                         ;

.text:0040128A                 mov     al, 20h         ; al = 0x20

.text:0040128C                 mov     dx, 1F7h        ; 1F7 disk 0 status

.text:00401290                 out     dx, al          ; AT hard disk

.text:00401290                         ; command register:

.text:00401290                         ; 1?H = Restore to cylinder 0

.text:00401290                         ; 7?H = Seek to cylinder

.text:00401290                         ; 2?H = Read sector

.text:00401290                         ; 3xH = Write sector

.text:00401290                         ; 50H = Format track

.text:00401290                         ; 4xH = verify read

.text:00401290                         ; 90H = diagnose

.text:00401290                         ; 91H = set parameters for drive

.text:00401290

.text:00401291

.text:00401291 loc_401291:                       ; CODE XREF: sub_4010F9+19Bj

.text:00401291                 in      al, dx          ; 读取 0x1f7 端口数据

.text:00401291                         ;

.text:00401291                         ; AT hard disk

.text:00401291                         ; status register bits:

.text:00401291                         ; 0: 1=prev cmd error

.text:00401291                         ; 2: Corrected data

.text:00401291                         ; 3: Data Request. Buffer is busy

.text:00401291                         ; 4: Seek completed

.text:00401291                         ; 5: Write fault

.text:00401291                         ; 6: Drive ready (unless bit 4=0)

.text:00401291                         ; 7: Busy

.text:00401292                 test    al, 8           ; 是否完成？

.text:00401294                 jz      short loc_401291

.text:00401294

.text:00401296                 xor     ecx, ecx        ; ecx = 0

.text:00401298                 mov     cx, 100h        ; cx = 0x100

.text:0040129C                 mov     dx, 1F0h        ; 1F0--存有结果数据

.text:004012A0                 lea     edi, [ebp+var_42C]

.text:004012A6                 cli                    ; 关中断

.text:004012A7                 cld                    ; 清除方向标志

.text:004012A7                         ;

.text:004012A8                 rep insw

.text:004012A8                         ;

.text:004012AB                 sti                    ; 开中断

.text:004012AC                 mov     ecx, 80h        ; ecx = 0x80

.text:004012B1                 lea     esi, [ebp+var_42C] ; esi 指向读出的数据

.text:004012B7                 mov     edi, [ebp+var_1C] ; var_1C = Irp->AssociatedIrp.SystemBuffer

.text:004012BA                 rep movsd               ; 将数据拷贝到 SystemBuffer 中

.text:004012BC                 mov     [ebp+var_18], 0 ; status = NT_SUCCESS (0)

.text:004012C3                 mov     byte ptr [ebp+var_10], 1 ; var_10 = 1

.text:004012C3                         ; 表示是读操作

.text:004012C3

.text:004012C7

.text:004012C7 loc_4012C7:                       ; CODE XREF: sub_4010F9+69j

.text:004012C7                         ; sub_4010F9:loc_401211j

.text:004012C7                         ; sub_4010F9+121j

.text:004012C7                         ; sub_4010F9+12Ej

.text:004012C7                 cmp     [ebp+var_18], 0 ; 判断操作是否成功？

.text:004012CB                 jnz     short loc_4012E4 ; 不成功则跳走

.text:004012CB

.text:004012CD                 mov     eax, [ebp+var_10] ; eax = var_10

.text:004012CD                         ; 根据读写操作设置 Information 长度

.text:004012D0                 and     eax, 0FFh

.text:004012D5                 neg     eax

.text:004012D7                 sbb     eax, eax

.text:004012D9                 and     eax, 200h

.text:004012DE                 mov     ecx, [ebp+Irp]

.text:004012E1                 mov     [ecx+1Ch], eax  ; 指定拷贝到 User buffer 的 buffer size

.text:004012E1                         ; Irp->IoStatus.Information =

.text:004012E1

.text:004012E4

.text:004012E4 loc_4012E4:                       ; CODE XREF: sub_4010F9+1D2j

.text:004012E4                 mov     edx, [ebp+Irp]

.text:004012E7                 mov     eax, [ebp+var_18]

.text:004012EA                 mov     [edx+18h], eax  ; Irp->IoStatus.Status = status

.text:004012ED                 xor     dl, dl          ; PriorityBoost

.text:004012EF                 mov     ecx, [ebp+Irp]  ; Irp

.text:004012F2                 call    ds:IofCompleteRequest ; complete Irp

.text:004012F8                 mov     eax, [ebp+var_18] ; 返回 status

.text:004012FB                 pop     edi

.text:004012FC                 pop     esi

.text:004012FD                 pop     ebx

.text:004012FE                 mov     esp, ebp

.text:00401300                 pop     ebp

.text:00401301                 retn    8

.text:00401301

.text:00401301 sub_4010F9      endp

```

---

**【总结】**

以上是很清晰的`0x1f0` - `0x1f7`的端口操作过程，未列出的端口或者想了解更详细的信息请自己`Google`吧。

源代码见附件。

---

**【致谢】**

感谢看雪论坛、一蓑烟雨、`DebugMan`论坛和驱动开发网，还有众多的大侠。

`2007`年`03`月`05`日`18:48` by `prince`

`4dd466d0e1e0cc123afbe504da2462ca`
