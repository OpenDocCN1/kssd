
# SoftSENTRY V3.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6419.htm`](https://www.pediy.com/kssd/pediy06/pediy6419.htm)

SoftSENTRY V3.0 脱壳——SoftSENTRY V3.0 电子商务版

下载页面： `http://secrecy.ayinfo.ha.cn/soft/44.htm`

软件大小： `2.40M`

软件语言： 英文

软件类别： 国外软件 / 零售版 / 序列号日期加密

运行环境： `Win9x/NT/2000/XP/`

软件更新： `2003-5-22 17:11:31`

软件添加： 洋白菜

下载次数： `222`

软件评级： `*****`

【软件简介】：一套不错的加密软件，可以自己定义加密算法、界面和时间次数限制。不过就是加密强度现在看来已经太低了。不过一些情况下用它还是很不错的！

【软件限制】：必须注册，否则拒绝运行

【作者声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`Ollydbg1.09`、`PEiD`、`LordPE`、`ImportREC`

### 过程

SoftSENTRY V2.11 可以用 `Crkss211.com` 自动脱壳，白菜乐园下载的 `ProcDump32` 支持 SoftSENTRY V3.0

《加密与解密》第二版 `P462` 有 SoftSENTRY 脱壳的论述，但是如果程序采取没有注册码就无法运行的保护方案，按书上的方法就无法跟踪到 OEP 了。下面就以 SoftSENTRY V3.0 电子商务版自身简单看看这种类型的脱壳。

`Sentry32.exe` 用 `FI` 看是 `PE-softSENTRY v3.0`，呵呵，自己保护自己。

## 一、用 `Ollydbg` 脱壳

```
00515270        55            push ebp
                                        ==== >进入 OD 后断在这！

00515271        8BEC          mov ebp,esp
00515273        83EC 78      sub esp,78
00515276        53            push ebx
00515277        56            push esi
00515278        57            push edi
00515279        E9 B0060000   jmp SENTRY32.0051592E
                                        ==== >跳 典型的 SoftSENTRY 壳入口 :-)

0051592C       /EB 05         jmp short SENTRY32.00515933
0051592E      ^|E9 3BFAFFFF   jmp SENTRY32.0051536E
                                        ==== >跳

0051536E        C745 E4 00000000 mov dword ptr ss:[ebp-1C],0
00515375        8D45 B8       lea eax,dword ptr ss:[ebp-48]
00515378        50            push eax
00515379        FF15 30015200 call dword ptr ds:[<&KERNEL32.GetStartupInfoA>
0051537F        8B4D E4       mov ecx,dword ptr ss:[ebp-1C]
00515382        83E1 01       and ecx,1
00515385        85C9          test ecx,ecx
00515387        74 0E         je short SENTRY32.00515397
00515389        8B55 E8       mov edx,dword ptr ss:[ebp-18]
0051538C        81E2 FFFF0000 and edx,0FFFF
00515392        8955 88       mov dword ptr ss:[ebp-78],edx
00515395        EB 07         jmp short SENTRY32.0051539E
                                        ==== >跳

0051539E        8B45 88       mov eax,dword ptr ss:[ebp-78]
005153A1        8945 14       mov dword ptr ss:[ebp+14],eax
005153A4        6A 00         push 0
005153A6        FF15 40015200 call dword ptr ds:[<&KERNEL32.GetModuleHandle>
005153AC        8945 08       mov dword ptr ss:[ebp+8],eax
005153AF        C745 0C 00000000 mov dword ptr ss:[ebp+C],0
005153B6        FF15 1C015200 call dword ptr ds:[<&KERNEL32.GetCommandLineA>
005153BC        8945 10       mov dword ptr ss:[ebp+10],eax
005153BF        8B4D 08       mov ecx,dword ptr ss:[ebp+8]
005153C2        894D AC       mov dword ptr ss:[ebp-54],ecx
005153C5        66:C705 10FE5100 >mov word ptr ds:[51FE10],0
005153CE        66:C705 08FB5100 >mov word ptr ds:[51FB08],0
005153D7        837D 0C 00    cmp dword ptr ss:[ebp+C],0
005153DB        75 13         jnz short SENTRY32.005153F0
005153DD        8B4D 08       mov ecx,dword ptr ss:[ebp+8]
005153E0        E8 EB100000   call SENTRY32.005164D0
005153E5        85C0          test eax,eax
005153E7        75 07         jnz short SENTRY32.005153F0
                                        ==== >跳

005153F0        68 04010000   push 104
005153F5        68 7CFC5100   push SENTRY32.0051FC7C
005153FA        8B55 08       mov edx,dword ptr ss:[ebp+8]
005153FD        52            push edx
005153FE        FF15 20015200 call dword ptr ds:[<&KERNEL32.GetModuleFileNa>
00515404        85C0          test eax,eax
00515406        75 07         jnz short SENTRY32.0051540F
                                        ==== >跳

0051540F        8B55 14       mov edx,dword ptr ss:[ebp+14]
00515412        8B4D 08       mov ecx,dword ptr ss:[ebp+8]
00515415        E8 16110000   call SENTRY32.00516530
0051541A        85C0          test eax,eax
0051541C        75 1B         jnz short SENTRY32.00515439
                                        ==== >跳

00515439        C745 B4 01000000 mov dword ptr ss:[ebp-4C],1
00515440        8B45 10       mov eax,dword ptr ss:[ebp+10]
00515443        A3 1CFC5100   mov dword ptr ds:[51FC1C],eax
00515448        E8 C32E0000   call SENTRY32.00518310
0051544D        85C0          test eax,eax
0051544F       /0F84 28010000 je SENTRY32.0051557D
                                        ==== >这里不能跳！所以修改为 NOP :-)
                                        ==== >这步关键！否则此类保护是无法跟踪到 OEP 的！

00515455       |66:C705 10FE5100 >mov word ptr ds:[51FE10],1
0051545E       |C705 F8FD5100 010>mov dword ptr ds:[51FDF8],1
00515468       |8B0D 14FE5100 mov ecx,dword ptr ds:[51FE14]
0051546E       |83E1 03       and ecx,3
00515471       |85C9          test ecx,ecx
00515473       |75 1C         jnz short SENTRY32.00515491
00515475       |8B15 14FE5100 mov edx,dword ptr ds:[51FE14]
0051547B       |83CA 03       or edx,3
0051547E       |8915 14FE5100 mov dword ptr ds:[51FE14],edx
00515484       |A1 80FD5100   mov eax,dword ptr ds:[51FD80]
00515489       |83F0 03       xor eax,3
0051548C       |A3 80FD5100   mov dword ptr ds:[51FD80],eax
00515491       |8B0D 14FE5100 mov ecx,dword ptr ds:[51FE14]
00515497       |83E1 70       and ecx,70
0051549A       |85C9          test ecx,ecx
0051549C       |75 1C         jnz short SENTRY32.005154BA
0051549E       |8B15 14FE5100 mov edx,dword ptr ds:[51FE14]
005154A4       |83CA 70       or edx,70
005154A7       |8915 14FE5100 mov dword ptr ds:[51FE14],edx
005154AD       |A1 80FD5100   mov eax,dword ptr ds:[51FD80]
005154B2       |83F0 70       xor eax,70
005154B5       |A3 80FD5100   mov dword ptr ds:[51FD80],eax
005154BA       |8B0D 14FE5100 mov ecx,dword ptr ds:[51FE14]
005154C0       |81E1 000A0000 and ecx,0A00
005154C6       |85C9          test ecx,ecx
005154C8       |75 1E         jnz short SENTRY32.005154E8
005154CA       |8B15 14FE5100 mov edx,dword ptr ds:[51FE14]
005154D0       |80CE 0A       or dh,0A
005154D3       |8915 14FE5100 mov dword ptr ds:[51FE14],edx
005154D9       |A1 80FD5100   mov eax,dword ptr ds:[51FD80]
005154DE       |35 000A0000   xor eax,0A00
005154E3       |A3 80FD5100   mov dword ptr ds:[51FD80],eax
005154E8       |8B0D 14FE5100 mov ecx,dword ptr ds:[51FE14]
005154EE       |81E1 00E00000 and ecx,0E000
005154F4       |85C9          test ecx,ecx
005154F6       |75 1E         jnz short SENTRY32.00515516
                                        ==== >跳

005154F8       |8B15 14FE5100 mov edx,dword ptr ds:[51FE14]
005154FE       |80CE E0       or dh,0E0
00515501       |8915 14FE5100 mov dword ptr ds:[51FE14],edx
00515507       |A1 80FD5100   mov eax,dword ptr ds:[51FD80]
0051550C       |35 00E00000   xor eax,0E000
00515511       |A3 80FD5100   mov dword ptr ds:[51FD80],eax
00515516       |8B0D 14FE5100 mov ecx,dword ptr ds:[51FE14]
0051551C       |81E1 00000600 and ecx,60000
00515522       |85C9          test ecx,ecx
00515524       |75 21         jnz short SENTRY32.00515547
                                        ==== >跳
```



```
00515526       |8B15 14FE5100     mov edx,dword ptr ds:[51FE14]

0051552C       |81CA 00000600     or edx,60000

00515532       |8915 14FE5100     mov dword ptr ds:[51FE14],edx

00515538       |A1 80FD5100       mov eax,dword ptr ds:[51FD80]

0051553D       |35 00000600       xor eax,60000

00515542       |A3 80FD5100       mov dword ptr ds:[51FD80],eax

00515547       |8B0D 14FE5100     mov ecx,dword ptr ds:[51FE14]

0051554D       |81E1 00002000     and ecx,200000

00515553       |85C9              test ecx,ecx

00515555       |75 21             jnz short SENTRY32.00515578

00515557       |8B15 14FE5100     mov edx,dword ptr ds:[51FE14]

0051555D       |81CA 00002000     or edx,200000

00515563       |8915 14FE5100     mov dword ptr ds:[51FE14],edx

00515569       |A1 80FD5100       mov eax,dword ptr ds:[51FD80]

0051556E       |35 00002000       xor eax,200000

00515573       |A3 80FD5100       mov dword ptr ds:[51FD80],eax

00515578       |E9 1C030000       jmp SENTRY32.00515899

                          ==== >OK，从这里跳下去！       :-)

005155D6        833D 50C25100 00  cmp dword ptr ds:[51C250],0

005155DD        74 16             je short SENTRY32.005155F5

005155DF        8B4D B4           mov ecx,dword ptr ss:[ebp-4C]

005155E2        E8 F92D0000       call SENTRY32.005183E0

                          ==== >这里是索要注册码的地方！跳过去了！

00515899        8D55 AC           lea edx,dword ptr ss:[ebp-54]

0051589C        8D4D B0           lea ecx,dword ptr ss:[ebp-50]

0051589F        E8 1C010000       call SENTRY32.005159C0

005158A4        8945 FC           mov dword ptr ss:[ebp-4],eax

005158A7        6A 00             push 0

005158A9        6A 00             push 0

005158AB        6A 10             push 10

005158AD        A1 38FC5100       mov eax,dword ptr ds:[51FC38]

005158B2        50                push eax

005158B3        FF15 08025200     call dword ptr ds:[<&USER32.SendMessageA>]

005158B9        833D 0CFE5100 02  cmp dword ptr ds:[51FE0C],2

005158C0        74 4F             je short SENTRY32.00515911

005158C2        837D B4 01        cmp dword ptr ss:[ebp-4C],1

005158C6        75 49             jnz short SENTRY32.00515911

005158C8        33C9              xor ecx,ecx

005158CA        66:8B0D 10FE5100  mov cx,word ptr ds:[51FE10]

005158D1        85C9              test ecx,ecx

005158D3        74 3C             je short SENTRY32.00515911

005158D5        33D2              xor edx,edx

005158D7        66:8B15 74FC5100  mov dx,word ptr ds:[51FC74]

005158DE        81FA 05800000     cmp edx,8005

005158E4        74 2B             je short SENTRY32.00515911

005158E6        8B45 08           mov eax,dword ptr ss:[ebp+8]

                          ==== >EAX=00400000            基地址

005158E9        50                push eax

005158EA        68 88C25100       push SENTRY32.0051C288     ; ASCII "sSENTRYWndClass"

005158EF        FF15 98015200     call dword ptr ds:[<&USER32.UnregisterClassA>>

                          ==== >注意：USER32.UnregisterClassA 可以看作一个标志吧？

005158F5        33C9              xor ecx,ecx

005158F7        66:8B0D 98C25100  mov cx,word ptr ds:[51C298]

005158FE        85C9              test ecx,ecx

00515900        74 0F             je short SENTRY32.00515911

00515902        8B55 AC           mov edx,dword ptr ss:[ebp-54]

00515905        52                push edx

00515906        8D55 B0           lea edx,dword ptr ss:[ebp-50]

00515909        8B4D FC           mov ecx,dword ptr ss:[ebp-4]

0051590C        E8 2F000000       call SENTRY32.00515940

                          ==== >F7 进去！别跑飞了！      :-)

00515911        837D B0 00        cmp dword ptr ss:[ebp-50],0

00515915        74 08             je short SENTRY32.0051591F

00515917        8B4D B0           mov ecx,dword ptr ss:[ebp-50]

0051591A        E8 213B0000       call SENTRY32.00519440

0051591F        8B45 B4           mov eax,dword ptr ss:[ebp-4C]

00515922        50                push eax

00515923        FF15 24015200     call dword ptr ds:[<&KERNEL32.ExitProcess>]

                          ==== >到这里就 OVER 了！
```

进入：`0051590C` `Call` `SENTRY32.00515940`

```
00515940        A1 80FD5100       mov eax,dword ptr ds:[51FD80]

00515945        53                push ebx

00515946        55                push ebp

00515947        56                push esi

00515948        8B71 06           mov esi,dword ptr ds:[ecx+6]

0051594B        57                push edi

0051594C        8BFA              mov edi,edx

0051594E        8B51 02           mov edx,dword ptr ds:[ecx+2]

00515951        33D0              xor edx,eax

00515953        8BC2              mov eax,edx

00515955        8951 02           mov dword ptr ds:[ecx+2],edx

                          ==== >《加密与解密》上说此处不可单步走，须 F4 到下面的 00515958

00515958        8B51 0A           mov edx,dword ptr ds:[ecx+A]

                          ==== >但我此处用 F8 单步过来了，没有异常

0051595B        33F0              xor esi,eax

0051595D        8B4424 14         mov eax,dword ptr ss:[esp+14]

00515961        33F2              xor esi,edx

00515963        03F0              add esi,eax

                          ==== >ESI=000166AF + 00400000=004166AF   这就是 OEP 值  :-)

00515965        33D2              xor edx,edx

00515967        8BC6              mov eax,esi

00515969        8B59 06           mov ebx,dword ptr ds:[ecx+6]

0051596C        8B28              mov ebp,dword ptr ds:[eax]

0051596E        33EB              xor ebp,ebx

00515970        42                inc edx

00515971        8928              mov dword ptr ds:[eax],ebp

00515973        8B59 0A           mov ebx,dword ptr ds:[ecx+A]

00515976        8B68 04           mov ebp,dword ptr ds:[eax+4]

00515979        83C0 04           add eax,4

0051597C        33EB              xor ebp,ebx

0051597E        42                inc edx

0051597F        8928              mov dword ptr ds:[eax],ebp

00515981        83C0 04           add eax,4

00515984        83FA 14           cmp edx,14

00515987      ^ 7C E0             jl short SENTRY32.00515969

                          ==== >F4 下去，不在这儿耽误时间

00515989        8B0F              mov ecx,dword ptr ds:[edi]

0051598B        E8 B03A0000       call SENTRY32.00519440

00515990        C707 00000000     mov dword ptr ds:[edi],0

00515996        66:833D 10FE5100 >cmp word ptr ds:[51FE10],0

0051599E        74 0C             je short SENTRY32.005159AC

005159A0        66:833D 9AC25100 >cmp word ptr ds:[51C29A],0

005159A8        74 02             je short SENTRY32.005159AC

005159AA        FFD6              call esi                  ; SENTRY32.004166AF

                          ==== >F7 进去！飞向光明的地方！  :-)

005159AC        5F                pop edi

005159AD        5E                pop esi

005159AE        5D                pop ebp

005159AF        5B                pop ebx

005159B0        C2 0400           retn 4
```

进入：`005159AA` `call` `esi`

```
004166AF          55              db 55

                          ==== >在这儿用 LordPE 完全 DUMP 这个进程

004166B0          8B              db 8B

004166B1          EC              db EC

004166B2          6A              db 6A

004166B3          FF              db FF

004166B4          68              db 68

004166B5          48644400        dd SENTRY32.00446448
```

`F9` 运行程序，运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `000166AF`，点 `IT AutoSearch`。



# 点“Get Import”，FixDump，正常运行！
1.12M ->1.16M 用 `FileScan` 优化后是 1.13M

## 二、闲话 SoftSENTRY 保护程序的破解
`SoftSENTRY` 的保护是比较脆弱的。无论是壳还是算法上都比现在的“猛“壳逊色许多。

1. **脱壳法**：上面即是这种方法。脱壳后即取消一切限制了。这种方法应该说是比较完美的。
2. **爆破法**：把 `SoftSENTRY` 限制的地方跳过。如上面的 `0051544F` 修改为 `NOP`，即可完美爆破。
3. **追算法**：虽然有的注册码会很长，但算法都是简单的。详情可看我以前发的相关帖子。

## 三、附：白菜乐园下载的 ProcDump32 的 SoftSentry 脚本

```
P1E=SoftSentry 2.11

P1F=SoftSentry 3.0

[SoftSentry 2.11]

L1=OBJR

L2=LOOK 0F,85,2E,00,00,00

L3=REPL 90,90,90,90,90,90

L4=OBJR

L5=LOOK 0F,84,1D,00,00,00

L6=REPL 90,90,90,90,90,90

L7=OBJR

L8=LOOK FF,D7,6A,00

L9=BP

LA=STEP

OPTL1=00000000

OPTL2=01010001

OPTL3=01010001

OPTL4=00030000

OPTL5=00000000

[SoftSentry 3.0]

L1=OBJR

L2=LOOK 0F,84,28,01,00,00

L3=REPL 0F,85

L4=OBJR

L5=LOOK FF,D6,5F,5E,5D

L6=BP

L7=STEP

OPTL1=00000000

OPTL2=01010001

OPTL3=01010001

OPTL4=00030000

OPTL5=00000000
```

```
             ,     \_/
         /| \_.-~/          \_     ,        青春都一饷
        ( /~   /             ~-.\_ |
        \`\\  \_/                   ~ )          忍把浮名
   _-~~~-.)  )\_\_/;;,.             \_  //'
  /'\_,   --~     ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂
 \`~ \_( ,\_..-- (     ,;'' /    ~--   /.\_\`
  /~~//'   /' \`~          ) /--..\_, )\_  \`~
  "  \`~"  "       \`"      /~'\`    \`\\~~

                   "     "   "~'  ""
               Cracked By 巢水工作坊——fly [OCN][FCG]
                     2003-09-20  16:16
```

`4dd466d0e1e0cc123afbe504da2462ca`
