
# SoftSENTRY V3.0脱壳――SoftSENTRY V3.0 电子商务版

- 标 题：SoftSENTRY V3.0脱壳——SoftSENTRY V3.0 电子商务版
- 发信人：fly
- 时 间：2003年9月21日 12:21

## 详细信息

SoftSENTRY V3.0脱壳――SoftSENTRY V3.0 电子商务版

- 下载页面：`http://secrecy.ayinfo.ha.cn/soft/44.htm`
- 软件大小：2.40M
- 软件语言：英文
- 软件类别：国外软件 / 零售版 / 序列号日期加密
- 运行环境：Win9x/NT/2000/XP/
- 软件更新：2003-5-22 17:11:31
- 软件添加：洋白菜
- 下载次数：222
- 软件评级：\*\*\*\*

## 软件简介

一套不错的加密软件，可以自己定义加密算法、界面和时间次数限制。不过就是加密强度现在看来已经太低了。不过一些情况下用它还是很不错的！

## 软件限制

必须注册，否则拒绝运行

## 作者声明

初学Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

## 破解工具

`Ollydbg1.09`、`PEiD`、`LordPE`、`ImportREC`

---

## 过程

SoftSENTRY V2.11可以用`Crkss211.com`自动脱壳，白菜乐园下载的`ProcDump32`支持SoftSENTRY V3.0

《加密与解密》第二版 P462 有SoftSENTRY脱壳的论述，但是如果程序采取没有注册码就无法运行的保护方案，按书上的方法就无法跟踪到OEP了。下面就以 SoftSENTRY V3.0 电子商务版 自身简单看看这种类型的脱壳。

`Sentry32.exe` 用`FI`看是 `PE-softSENTRY v3.0`，呵呵，自己保护自己。

---

### 一、用 `Ollydbg` 脱壳

```
00515270        55                push ebp

                                  ====>进入OD后断在这！

00515271        8BEC              mov ebp,esp

00515273        83EC 78           sub esp,78

00515276        53                push ebx

00515277        56                push esi

00515278        57                push edi

00515279        E9 B0060000       jmp SENTRY32.0051592E

                                  ====>跳    典型的SoftSENTRY壳入口  :-)

0051592C       /EB 05             jmp short SENTRY32.00515933

0051592E      ^|E9 3BFAFFFF       jmp SENTRY32.0051536E

                                  ====>跳

0051536E        C745 E4 00000000  mov dword ptr ss:[ebp-1C],0

00515375        8D45 B8           lea eax,dword ptr ss:[ebp-48]

00515378        50                push eax

00515379        FF15 30015200     call dword ptr ds:[<&KERNEL32.GetStartupInfoA>

0051537F        8B4D E4           mov ecx,dword ptr ss:[ebp-1C]

00515382        83E1 01           and ecx,1

00515385        85C9              test ecx,ecx

00515387        74 0E             je short SENTRY32.00515397

00515389        8B55 E8           mov edx,dword ptr ss:[ebp-18]

0051538C        81E2 FFFF0000     and edx,0FFFF

00515392        8955 88           mov dword ptr ss:[ebp-78],edx

00515395        EB 07             jmp short SENTRY32.0051539E

                                  ====>跳

0051539E        8B45 88           mov eax,dword ptr ss:[ebp-78]

005153A1        8945 14           mov dword ptr ss:[ebp+14],eax

005153A4        6A 00             push 0

005153A6        FF15 40015200     call dword ptr ds:[<&KERNEL32.GetModuleHandle>

005153AC        8945 08           mov dword ptr ss:[ebp+8],eax

005153AF        C745 0C 00000000  mov dword ptr ss:[ebp+C],0

005153B6        FF15 1C015200     call dword ptr ds:[<&KERNEL32.GetCommandLineA>

005153BC        8945 10           mov dword ptr ss:[ebp+10],eax

005153BF        8B4D 08           mov ecx,dword ptr ss:[ebp+8]

005153C2        894D AC           mov dword ptr ss:[ebp-54],ecx

005153C5        66:C705 10FE5100 >mov word ptr ds:[51FE10],0

005153CE        66:C705 08FB5100 >mov word ptr ds:[51FB08],0

005153D7        837D 0C 00        cmp dword ptr ss:[ebp+C],0

005153DB        75 13             jnz short SENTRY32.005153F0

005153DD        8B4D 08           mov ecx,dword ptr ss:[ebp+8]

005153E0        E8 EB100000       call SENTRY32.005164D0

005153E5        85C0              test eax,eax

005153E7        75 07             jnz short SENTRY32.005153F0

                                  ====>跳

005153F0        68 04010000       push 104

005153F5        68 7CFC5100       push SENTRY32.0051FC7C

005153FA        8B55 08           mov edx,dword ptr ss:[ebp+8]

005153FD        52                push edx

005153FE        FF15 20015200     call dword ptr ds:[<&KERNEL32.GetModuleFileNa>

00515404        85C0              test eax,eax

00515406        75 07             jnz short SENTRY32.0051540F

                                  ====>跳

0051540F        8B55 14           mov edx,dword ptr ss:[ebp+14]

00515412        8B4D 08           mov ecx,dword ptr ss:[ebp+8]

00515415        E8 16110000       call SENTRY32.00516530

0051541A        85C0              test eax,eax

0051541C        75 1B             jnz short SENTRY32.00515439

                                  ====>跳

00515439        C745 B4 01000000  mov dword ptr ss:[ebp-4C],1

00515440        8B45 10           mov eax,dword ptr ss:[ebp+10]

00515443        A3 1CFC5100       mov dword ptr ds:[51FC1C],eax

00515448        E8 C32E0000       call SENTRY32.00518310

0051544D        85C0              test eax,eax

0051544F       /0F84 28010000     je SENTRY32.0051557D

                                  ====>这里不能跳！所以修改为NOP          :-)

                                  ====>这步关键！否则此类保护是无法跟踪到OEP的！

00515455       |66:C705 10FE5100 >mov word ptr ds:[51FE10],1

0051545E       |C705 F8FD5100 010>mov dword ptr ds:[51FDF8],1

00515468       |8B0D 14FE5100     mov ecx,dword ptr ds:[51FE14]

0051546E       |83E1 03           and ecx,3

00515471       |85C9              test ecx,ecx

00515473       |75 1C             jnz short SENTRY32.00515491

00515475       |8B15 14FE5100     mov edx,dword ptr ds:[51FE14]

0051547B       |83CA 03           or edx,3

0051547E       |8915 14FE5100     mov dword ptr ds:[51FE14],edx

00515484       |A1 80FD5100       mov eax,dword ptr ds:[51FD80]

00515489       |83F0 03           xor eax,3

0051548C       |A3 80FD5100       mov dword ptr ds:[51FD80],eax

00515491       |8B0D 14FE5100     mov ecx,dword ptr ds:[51FE14]

00515497       |83E1 70           and ecx,70

0051549A       |85C9              test ecx,ecx

0051549C       |75 1C             jnz short SENTRY32.005154BA

0051549E       |8B15 14FE5100     mov edx,dword ptr ds:[51FE14]

005154A4       |83CA 70           or edx,70

005154A7       |8915 14FE5100     mov dword ptr ds:[51FE14],edx

005154AD       |A1 80FD5100       mov eax,dword ptr ds:[51FD80]

005154B2       |83F0 70           xor eax,70

005154B5       |A3 80FD5100       mov dword ptr ds:[51FD80],eax

005154BA       |8B0D 14FE5100     mov ecx,dword ptr ds:[51FE14]

005154C0       |81E1 000A0000     and ecx,0A00

005154C6       |85C9              test ecx,ecx

005154C8       |75 1E             jnz short SENTRY32.005154E8

005154CA       |8B15 14FE5100     mov edx,dword ptr ds:[51FE14]

005154D0       |80CE 0A           or dh,0A

005154D3       |8915 14FE5100     mov dword ptr ds:[51FE14],edx

005154D9       |A1 80FD5100       mov eax,dword ptr ds:[51FD80]

005154DE       |35 000A0000       xor eax,0A00

005154E3       |A3 80FD5100       mov dword ptr ds:[51FD80],eax

005154E8       |8B0D 14FE5100     mov ecx,dword ptr ds:[51FE14]

005154EE       |81E1 00E00000     and ecx,0E000

005154F4       |85C9              test ecx,ecx

005154F6       |75 1E             jnz short SENTRY32.00515516

                                  ====>跳

005154F8       |8B15 14FE5100     mov edx,dword ptr ds:[51FE14]

005154FE       |80CE E0           or dh,0E0

00515501       |8915 14FE5100     mov dword ptr ds:[51FE14],edx

00515507       |A1 80FD5100       mov eax,dword ptr ds:[51FD80]

0051550C       |35 00E00000       xor eax,0E000

00515511       |A3 80FD5100       mov dword ptr ds:[51FD80],eax

00515516       |8B0D 14FE5100     mov ecx,dword ptr ds:[51FE14]

0051551C       |81E1 00000600     and ecx,60000

00515522       |85C9              test ecx,ecx

00515524       |75 21             jnz short SENTRY32.00515547

                                  ====>跳

00515526       |8B15 14FE5100     mov edx,dword ptr ds:[51FE14]

0051552C       |81CA 00000600     or edx,60000

00515532       |8915 14FE5100     mov dword ptr ds:[51FE14],edx

00515538       |A1 80FD5100       mov eax,dword ptr ds:[51FD80]

0051553D       |35 00000600       xor eax,60000

00515542       |A3 80FD5100       mov dword ptr ds:[51FD80],eax

00515547       |8B0D 14FE5100     mov ecx,dword ptr ds:[51FE14]

0051554D       |81E1 00002000     and ecx,200000

00515553       |85C9              test ecx,ecx
```
