
# 网页特效小精灵

> 原文：[`www.pediy.com/kssd/pediy04/kanxue111.htm`](https://www.pediy.com/kssd/pediy04/kanxue111.htm)

软件功能：看名字就知道

下载网址：`http://chinabds.126.com`

功能限制：只能在 2001.10－2001.12 之间使用。

破解工具：`Language 2000`，`UnPecompact`，`Trw2000`，`W32dasm`，`Hiew`

文章作者：`eCool`

整理时间：2001.10.10 （转载请保持原文完整）

## 破解过程

1. 用 `Language 2000` 看看，发现是加了 `PeCompact` 壳，马上用 `UnPecompact` 解壳，接着用 `Language 2000` 发现是 `VB6` 写的，头大，：（

2. 用 `W32dasm` 看看能否找到过期提示，结果一无所获，看来只好用 `Trw2000` 试试了。

3. 用 `bpx msvbvm60!\_\_.rtcGetYear` 设断，运行程序。

4. 程序被拦下：

```
\* Reference To: MSVBVM60.rtcGetYear, Ord:0229h                      <------获取年份

:004209AC FF1540104000           Call dword ptr [00401040]
:004209B2 8D559C                 lea edx, dword ptr [ebp-64]
:004209B5 C78564FFFFFFD2070000   mov dword ptr [ebp+FFFFFF64], 000007D2
:004209BF 52                     push edx
:004209C0 C7855CFFFFFF02800000   mov dword ptr [ebp+FFFFFF5C], 00008002
:004209CA FFD3                   call ebx
:004209CC 8D459C                 lea eax, dword ptr [ebp-64]
:004209CF 8D4D8C                 lea ecx, dword ptr [ebp-74]
:004209D2 50                     push eax
:004209D3 51                     push ecx

\* Reference To: MSVBVM60.rtcGetMonthOfYear, Ord:0221h             <---------获取月

:004209D4 FF1550114000           Call dword ptr [00401150]
:004209DA 8D55BC                 lea edx, dword ptr [ebp-44]
:004209DD 8D855CFFFFFF           lea eax, dword ptr [ebp+FFFFFF5C]
:004209E3 52                     push edx
:004209E4 8D4DAC                 lea ecx, dword ptr [ebp-54]
:004209E7 50                     push eax
:004209E8 51                     push ecx
:004209E9 C78554FFFFFF0A000000   mov dword ptr [ebp+FFFFFF54], 0000000A
:004209F3 C7854CFFFFFF02800000   mov dword ptr [ebp+FFFFFF4C], 00008002

\* Reference To: MSVBVM60.\_\_vbaVarCmpEq, Ord:0000h               <---------比较是否为 2001 年

:004209FD FF152C114000           Call dword ptr [0040112C]
:00420A03 50                     push eax
:00420A04 8D558C                 lea edx, dword ptr [ebp-74]
:00420A07 8D854CFFFFFF           lea eax, dword ptr [ebp+FFFFFF4C]
:00420A0D 52                     push edx
:00420A0E 8D8D7CFFFFFF           lea ecx, dword ptr [ebp+FFFFFF7C]
:00420A14 50                     push eax
:00420A15 51                     push ecx

\* Reference To: MSVBVM60.\_\_vbaVarCmpLt, Ord:0000h               <---------比较是否在 10 月到 12 月中

:00420A16 FF1518114000           Call dword ptr [00401118]
:00420A1C 8D956CFFFFFF           lea edx, dword ptr [ebp+FFFFFF6C]
:00420A22 50                     push eax
:00420A23 52                     push edx

\* Reference To: MSVBVM60.\_\_vbaVarOr, Ord:0000h

:00420A24 FF159C104000           Call dword ptr [0040109C]
:00420A2A 50                     push eax

\* Reference To: MSVBVM60.\_\_vbaBoolVarNull, Ord:0000h

:00420A2B FF1578104000           Call dword ptr [00401078]
:00420A31 668BD8                 mov bx, ax
:00420A34 8D458C                 lea eax, dword ptr [ebp-74]
:00420A37 8D4D9C                 lea ecx, dword ptr [ebp-64]
:00420A3A 50                     push eax
:00420A3B 8D55BC                 lea edx, dword ptr [ebp-44]
:00420A3E 51                     push ecx
:00420A3F 8D45CC                 lea eax, dword ptr [ebp-34]
:00420A42 52                     push edx
:00420A43 50                     push eax
:00420A44 6A04                   push 00000004

\* Reference To: MSVBVM60.\_\_vbaFreeVarList, Ord:0000h

:00420A46 FF151C104000           Call dword ptr [0040101C]
:00420A4C 83C414                 add esp, 00000014
:00420A4F 663BDF                 cmp bx, di          <--------比较标志位
:00420A52 0F8495000000           je 00420AED          <--------为 0 则跳，关键
                                                  否则 Over，改为 Jmp

\* Reference To: MSVBVM60.\_\_vbaVarDup, Ord:0000h

:00420A58 8B3534114000           mov esi, dword ptr [00401134]
:00420A5E B90A000000             mov ecx, 0000000A
:00420A63 B804000280             mov eax, 80020004
:00420A68 894D9C                 mov dword ptr [ebp-64], ecx
:00420A6B 894DAC                 mov dword ptr [ebp-54], ecx
:00420A6E BB08000000             mov ebx, 00000008
:00420A73 8D954CFFFFFF           lea edx, dword ptr [ebp+FFFFFF4C]
:00420A79 8D4DBC                 lea ecx, dword ptr [ebp-44]
:00420A7C 8945A4                 mov dword ptr [ebp-5C], eax
:00420A7F 8945B4                 mov dword ptr [ebp-4C], eax
:00420A82 C78554FFFFFF68DE4100   mov dword ptr [ebp+FFFFFF54], 0041DE68
:00420A8C 899D4CFFFFFF           mov dword ptr [ebp+FFFFFF4C], ebx
:00420A92 FFD6                   call esi
:00420A94 8D955CFFFFFF           lea edx, dword ptr [ebp+FFFFFF5C]
:00420A9A 8D4DCC                 lea ecx, dword ptr [ebp-34]
:00420A9D C78564FFFFFF28DE4100   mov dword ptr [ebp+FFFFFF64], 0041DE28
:00420AA7 899D5CFFFFFF           mov dword ptr [ebp+FFFFFF5C], ebx
:00420AAD FFD6                   call esi
:00420AAF 8D4D9C                 lea ecx, dword ptr [ebp-64]
:00420AB2 8D55AC                 lea edx, dword ptr [ebp-54]
:00420AB5 51                     push ecx
:00420AB6 8D45BC                 lea eax, dword ptr [ebp-44]
:00420AB9 52                     push edx
:00420ABA 50                     push eax
:00420ABB 8D4DCC                 lea ecx, dword ptr [ebp-34]
:00420ABE 6A10                   push 00000010
:00420AC0 51                     push ecx

\* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h               <---------这里是过期对话框，
                                                  所以往上看。

:00420AC1 FF1560104000           Call dword ptr [00401060]
:00420AC7 8D559C                 lea edx, dword ptr [ebp-64]
:00420ACA 8D45AC                 lea eax, dword ptr [ebp-54]
:00420ACD 52                     push edx
:00420ACE 8D4DBC                 lea ecx, dword ptr [ebp-44]
:00420AD1 50                     push eax
:00420AD2 8D55CC                 lea edx, dword ptr [ebp-34]
:00420AD5 51                     push ecx
:00420AD6 52                     push edx
:00420AD7 6A04                   push 00000004
```

5. 用 `Hiew` 将 `00420A52` 处的 `0F8495000000` 改为 `EB8495000000`，运行程序，`Why?`

   程序出错，？？有 `CRC`？，`No`，因为我脱壳后运行都没有出错啊，所以判断是：

   ```
   :00420A4F 663BDF                 cmp bx, di
   ```

   这里的标志位应该影响了以后的运行，所以，呵呵，改为 `cmp di,di`，哈哈，搞定。

## 总结

只要将 `00420A4F` 处的 `663BDF` 改为 `663BFF`，就将可以了，再不用担心过期了。 `4dd466d0e1e0cc123afbe504da2462ca`
