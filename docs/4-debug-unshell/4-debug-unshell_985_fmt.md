
# SecuROM 脱壳分析，纯属陋文，为抛砖引玉而作

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-548.htm`](https://www.pediy.com/kssd/pediy09/pediy09-548.htm)

> 此文参考各位达人们的文章及其研究经验总结完善而成。

## 序言

SecuROM 以前一直想弄下，但是一直没机会，最近偶获 `WE2007` 版本，且使用 `OD` 可方便调试，故兴趣大增并分析之。

## 定位 OEP

要开工，得先找到 `OEP`，`OD` 加载，突然想到以前论坛达人们的偷懒一法，直接中断 `GetVersionExA` 测试下。

中断后可从堆栈得返回地址，所以写了个脚本，查找在 `004xxxxx` 这个范围返回的 `CALL` 地址。

故查到第一个接近目标的地址：

```text
0022FEA8   0041A483  /CALL 到 GetVersionExA 来自  0041A47D
0022FEAC   0022FEB0  \pVersionInformation = 0022FEB0
```

跳到返回地址往上看，爽，这个不就像 `VC` 的 `OEP` 么。

```text
0041A45D   6A 60           PUSH 60
0041A45F    68 68F3B700    PUSH 00B7F368
0041A464    E8 73340000    CALL 0041D8DC
0041A469    BF 94000000    MOV EDI,94
0041A46E    8BC7           MOV EAX,EDI
0041A470    E8 BB340000    CALL 0041D930
0041A475    8965 E8        MOV DWORD PTR SS:[EBP-18],ESP
0041A478    8BF4           MOV ESI,ESP
0041A47A    893E           MOV DWORD PTR DS:[ESI],EDI
0041A47C    56             PUSH ESI
0041A47D    FF15 04E2B700  CALL DWORD PTR DS:[<&KERNEL32.GetVersion>; kernel32.GetVersionExA
```

所以，这里可以 `DUMP` 一个比较干净的 `EXE`，但是不是现在，因为还有很多工作以后要接触到。

到这个时候，看了下 `OD` 里面的段，还真有点多，不知道申请那么多内存来做什么，好我们接着看下他要用这些内存作什么。

跟到下面这个地址，大概就有点点明白了：

```text
0041DB96  - FF25 14D04A04  JMP DWORD PTR DS:[44AD014]               ;  04517DD0
0041DB9C    CC             INT3
```

看，他通过这个 `JMP` 到内存段去执行代码，看来这里的代码是被 `SecuROM` 抽掉了。我查了下 `JMP DWORD PTR DS:[]` 这样的有几十个，任意看了 1, 2 个，都是跳到一个公共地方先加密，然后再执行解密后的正确代码。这些代码存放的内存空间都是单独的，看来都是 `VirtualAlloc` 出来的呢，记录下，等下专门跟下 `VirtualAlloc`。

仔细看了下执行解密的代码：

```text
040D531C    FF70 09        PUSH DWORD PTR DS:[EAX+9]
040D531F    FF33           PUSH DWORD PTR DS:[EBX]
040D5321    E8 FDFcFFFF   CALL 040D5023
........

03D800CF  - FF25 E8A62404  JMP DWORD PTR DS:[424A6E8]               ; kernel32.GetSystemInfo
```

仔细看了下，他取系统信息作为解密 `KEY`，对抽取代码进行还原，执行一次后代码就赤裸裸的存在了，就不细管这里的解密，后面我再讲。

## 分析 VirtualAlloc 调用

好，重新装载启动，中断 `VirtualAlloc` 看下。

恩，发现被壳调用了，呵呵，高兴，去看看他申请来做什么，我走啊走啊，晕倒，竟然分配的空间用来做 `SOFTICE`, `OD` 检查的，因为我的能跑起来，呵呵，不看了略过。

恩，继续看下面的，嘿嘿，发现申请空间的地址，恩，奇怪，他竟然是指定 `ADDR` 申请的，我观察了，发现一个小细节：

```text
03E2B557    0F31          RDTSC
03E2B559    A8 01         TEST AL,1
03E2B55B    74 05         JE SHORT we2007.03E2B562

.......

03E2B712  - FF25 55794704  JMP DWORD PTR DS:[<&KERNEL32.VirtualAlloc>]     ; kernel32.VirtualAlloc
03E2B718    6C             INS BYTE PTR ES:[EDI],DX                   ; I/O 命令
03E2B719    97             XCHG EAX,EDI
03E2B71A  - E9 B605AD15    JMP 198FBCD5
03E2B71F    EF             OUT DX,EAX                                 ; I/O 命令
03E2B720    09C0           OR EAX,EAX
03E2B722  ^ 0F84 2FFEFFFF  JE we2007.03E2B557
```

呵呵，他竟然用 `RDTSC` 取得一个值，简单的换算下，作为申请空间的固定地址，目的明显是为了得到随机的空间地址。

## 代码解密过程

下面简单说下过程，下面我说个方法可以略过这里，为了凑点字数，加上代码解密的过程。

将本地数据 `COPY` 到分配的空间：

```text
03E1EAE7    F3:A4         REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
```

进行第一次解密码：

```text
03D7E63F    3008          XOR BYTE PTR DS:[EAX],CL
03D7E641    8A00          MOV AL,BYTE PTR DS:[EAX]
```

进行第二次解密码，得到初始数据：

```text
04242993    8A49 01       MOV CL,BYTE PTR DS:[ECX+1]
04242996    47             INC EDI
04242997    880F          MOV BYTE PTR DS:[EDI],CL
04242999    3B5D D4       CMP EBX,DWORD PTR SS:[EBP-2C]
```

这里，就将代码还原了，后来才知道，这个不是被抽掉的代码，是 `VM` 的代码还原到内存中去。这里 `VirtualAlloc` 的值可以被修改成自己的空间里面，这样就方便将零散的 `VM` 代码集中。如果不这样的话，估计有几十个这样的零散空间，够你手动还原的了。

下面是修改 `VM` 代码到自己空间的 `OD` 脚本：

```text
data:
  var virtualaddr
  var virtuallen
  var fixmemaddr
  var fixmemlen
  var incmemlen

start:
  Pause
  mov incmemlen,0
  mov fixmemlen,7000000      //申请自己的存空间
  alloc fixmemlen           
  mov fixmemaddr, $RESULT
  
  bp 7C809AB9      //VirtualAllocEx

code:
  eob  code        //bpx
  esto
  cmp eip, 7C809AB9
  je   fixmem
  
  jmp  code

end:
  MSG "脚本暂停"
  Pause
  ret

fixmem:   
  cmp eax,0
  je  code
  mov virtualaddr,[esp+8]
  mov virtuallen, [esp+c]
  mov eax,fixmemaddr
  add incmemlen,  virtuallen
  cmp incmemlen,  fixmemlen
  jae err
  mov eax,  fixmemaddr
  add fixmemaddr, virtuallen
  jmp code

err:
  MSG "内存越界"
  Pause
  ret
```

这样，可以将 `VM` 申请空间都统一到一个内存空间里面，比一个一个 `DUMP` 下来再附加在 `EXE` 上方便多了，这样只附加一个即可。

## 处理被抽掉的代码

之前发现被抽掉的代码空间我们还没找到，中断 `VirtualAlloc` 看下，发现处理这段代码的位置。

将本地数据 `COPY` 到分配的空间：

```text
03FB39B3    F3:A5           REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]
03FB39B5    FF2495 CC3AFB03 JMP DWORD PTR DS:[EDX*4+3FB3ACC]
```

解密出原始的抽掉的代码：

```text
040B1BEE    3008          XOR BYTE PTR DS:[EAX],CL
040B1BF0    B8 D338FBFF   MOV EAX,FFFB38D3
......
040B1A59    3008          XOR BYTE PTR DS:[EAX],CL
040B1A5B    B8 A987FDFF   MOV EAX,FFFD87A9
```

对还原出来的数据，根据内存地址的不同，而修正 `CALL` 和 `PUSH`：

```text
040B202E    0118          ADD DWORD PTR DS:[EAX],EBX
040B2030    8D6424 FC     LEA ESP,DWORD PTR SS:[ESP-4]
......
040B2278    2918          SUB DWORD PTR DS:[EAX],EBX
040B227A    8D6424 FC     LEA ESP,DWORD PTR SS:[ESP-4]
```

对修正后的数据，进行 `SYSTEMINFO` 和 `GetCurrentProcessId` 加密，为了保证代码唯一性：

```text
040B2893    300C03        XOR BYTE PTR DS:[EBX+EAX],CL
040B2896    83C7 01       ADD EDI,1
040B2899    83E7 07       AND EDI,7
```

这样被抽掉的代码处理地方就发现，这里有点问题还没搞明白，这里的 `VirtualAlloc` 不能替换成自己申请的空间，要报错。所以暂时只能硬 `DUMP` 下来附加。

## PE 修复

好，`OEP` 和代码处理的地方基本就是这些，现在开始进行 `PE` 修复了。

先是用脚本将 `VM` 的空间都指向自己的空间里面，然后中断到 `OEP` 处，这个时候 `DUMP` 出 `EXE`。由于被抽掉的代码我没办法结合到一个空间，所以就只能全段 `DUMP` 下来，然后将所有的段拼接起来，然后修正 `OEP`。只不过 `EXE` 文件有点大，哈哈，丢人啊。

代码段就到这里 `OK` 了，开始修复 `IAT`。我看了下，这个 `EXE` 的 `IAT` 竟然是干净的，不用修复，爽，只要将 `IAT` 从指向壳的 `IAT` 修正到原来 `EXE` 的 `IAT` 地址即可，这个很简单，这样 `IAT` 就修正完毕。

恩，将修正了段和 `IAT` 的用 `OD` 加载，晕倒，错误还满多的，还看不到画面就直接下课了，只能继续检查错误。

因为被抽掉的代码要用 `GetCurrentProcessId` 解密，因为 `GetCurrentProcessId` 每次启动不一样，所以的修复。不然被抽掉的还原不正确的，所以要报错阿。

修正由于 `GetCurrentThreadId` 出错的地方，直接将址修改成你 `DUMP EXE` 时候那个那个 `ID` 即可：

```text
040D5049    FF15 B1794704  CALL DWORD PTR DS:[<&KERNEL32.GetCurrent>; kernel32.GetCurrentProcessId
```

恩，继续用 `OD` 装载修正后的 `EXE`，`OK`，这次跑的稍微远点了，还是在某处报错了，跟进去看下：

```text
04024A8A  50           PUSH EAX
04024A8B  FF15 78253D04 CALL DWORD PTR DS:[43D2578]            ; we2007.04029F80
04024A91  35 809F0204  XOR EAX,4029F80
04024A96  3305 78253D04 XOR EAX,DWORD PTR DS:[43D2578]         ; we2007.04029F80
04024A9C  E9 8A000000  JMP we2007.04024B2B
04024AA1  50           PUSH EAX
04024AA2  FF15 7C253D04 CALL DWORD PTR DS:[43D257C]            ; we2007.04029F90
04024AA8  35 909F0204  XOR EAX,4029F90
04024AAD  3305 7C253D04 XOR EAX,DWORD PTR DS:[43D257C]         ; we2007.04029F90
04024AB3  EB 76        JMP SHORT we2007.04024B2B
04024AB5  50           PUSH EAX
04024AB6  FF15 80253D04 CALL DWORD PTR DS:[43D2580]            ; we2007.04029FA0
04024ABC  35 A09F0204  XOR EAX,4029FA0
04024AC1  3305 80253D04 XOR EAX,DWORD PTR DS:[43D2580]         ; we2007.04029FA0
04024AC7  EB 62        JMP SHORT we2007.04024B2B
04024AC9  50           PUSH EAX
04024ACA  FF15 84253D04 CALL DWORD PTR DS:[43D2584]            ; we2007.04029FD0
04024AD0  35 D09F0204  XOR EAX,4029FD0
04024AD5  3305 84253D04 XOR EAX,DWORD PTR DS:[43D2584]         ; we2007.04029FD0
04024ADB  EB 4E        JMP SHORT we2007.04024B2B
```



```
04024ADD   50                PUSH EAX  
04024ADE   FF15 88253D04     CALL DWORD PTR DS:[43D2588]               ; we2007.04029FF0  
04024AE4   35 F09F0204       XOR EAX,4029FF0  
04024AE9   3305 88253D04     XOR EAX,DWORD PTR DS:[43D2588]           ; we2007.04029FF0  
04024AEF   EB 3A             JMP SHORT we2007.04024B2B  
04024AF1   50                PUSH EAX  
04024AF2   FF15 8C253D04     CALL DWORD PTR DS:[43D258C]               ; we2007.0402A050  
04024AF8   35 50A00204       XOR EAX,402A050  
04024AFD   3305 8C253D04     XOR EAX,DWORD PTR DS:[43D258C]           ; we2007.0402A050  
04024B03   EB 26             JMP SHORT we2007.04024B2B  
04024B05   50                PUSH EAX  
04024B06   FF15 90253D04     CALL DWORD PTR DS:[43D2590]               ; we2007.0402A060  
04024B0C   35 60A00204       XOR EAX,402A060  
04024B11   3305 90253D04     XOR EAX,DWORD PTR DS:[43D2590]           ; we2007.0402A060  
04024B17   EB 12             JMP SHORT we2007.04024B2B  
04024B19   50                PUSH EAX  
04024B1A   FF15 94253D04     CALL DWORD PTR DS:[43D2594]               ; we2007.0402A0D0  
04024B20   35 D0A00204       XOR EAX,402A0D0  
04024B25   3305 94253D04     XOR EAX,DWORD PTR DS:[43D2594]           ; we2007.0402A0D0  
04024B2B   3345 0C           XOR EAX,DWORD PTR SS:[EBP+C]  
04024B2E   5D                POP EBP  
04024B2F   C3                RETN  
```

这里集合了好几种检查机制,抽 2 个说明下

```
04029F80   FF15 482F2D04     CALL DWORD PTR DS:[42D2F48]               ; kernel32.GetCurrentProcessId  
04029F86   034424 04         ADD EAX,DWORD PTR SS:[ESP+4]  
04029F8A   C2 0400           RETN 4  
```

```
04029F90   FF15 4C2F2D04     CALL DWORD PTR DS:[42D2F4C]               ; kernel32.GetVersion (0A280105)  
04029F96   2B4424 04         SUB EAX,DWORD PTR SS:[ESP+4]  
```

这个分别进入前 2 个 CALL 后的情况

很明显第一个又用 `GetCurrentProcessId` 来对压入的值进行变换,如果 `GetCurrentProcessId` 不一样得到不一样的值,就导致运算出错

第二个也是用 `GetVersion` 参与运算,呵呵,这里就一次将几个判断都修复,直接付 `DUMP exe` 时候,进程当前环境值即可

用 `OD` 加载继续跑,同样出错,呵呵,郁闷了,继续检查,发现这个地方

```
0049017C    8B0D 4C9F4A04    MOV ECX,DWORD PTR DS:[44A9F4C]           ; 3th_17_d.05075B5D  
00490182    1B0D 69164C04    SBB ECX,DWORD PTR DS:[44C1669]           ; 3th_17_d.044AECC8  
```

看到没有,他在效验刚才被我修改的代码段,呵呵,这里处理办法很多,你可以跳开检查的段,或者直接付值.

嘿嘿,这次用 `OD` 加载没报告错误了,嘿嘿,运行起来了,爽

看到游戏画面,只是踢球的时候报了错,非法了,郁闷,继续检查.

```
0047E1FF    8B3D 149F4A04    MOV EDI,DWORD PTR DS:[44A9F14]           ; 3th_17_d.009D1DA5  
0047E205    1B3D 68014000    SBB EDI,DWORD PTR DS:[400168]  
0047E20B    9D               POPFD  
```

你看他在做什么,他在检查 `PE` 头的数据,呵呵,知道他在检查什么,就方便了,我这里处理的办法就是,将他的 `PE` 头保留,然后自己在他老的 `PE` 头

再加上自己的头信息,呵呵,`OK` 就搞定了.

这次可以正常游戏了,至少本机没出现任何其他问题.对于跨机器的 `VM` 中的修正,下次再讲下吧,累了~

`4dd466d0e1e0cc123afbe504da2462ca`
