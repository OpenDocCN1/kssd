
# pb 程序调试初探

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-142.htm`](https://www.pediy.com/kssd/pediy09/pediy09-142.htm)

> 软件名称：职称英语学习帮手（综合类）  
> 破解工具：OD，shudepb，pbkill，winhex  
> 破解目的：自用  
> 
> 最近要考职称英语，下载了个职称英语的软件来学习。  
> 未注册的版本只能阅读每个部分第一篇，可惜我的银子不够，只好拿它开刀。  
> 打开程序目录，顿时傻眼：PB 程序。关于 PB 程序的调试可以说网上几乎还没有，可见非常困难。  
> 没有反编译工具几乎是不可能完成的任务。而即使是反编译出来，如果编程功力不够，也很难看出注册方法和写注册机。  
> 而爆破关键跳转目前在网上我还没有搜到跳转的机器码是什么。  
> 调试过程的那个辛苦啊，我只想说：破解＝运气＋毅力。真的，运气可能是第一位的。  
> 
> 先找工具反编译。试了 `pbkill`，只得到一堆 `if...else...end if` 之类的，一句代码也没有出来（可能是 `pbkill` 试用版吧？）  
> 在网上又搜到另一个反编译工具 `shudepb`，使用确实很好，支持全局搜索，大大方便找关键代码。  
> 全局搜索提示："对不起，你尚未注册，只能阅读每一部分的第一篇文章！欢迎注册!"，找到以下代码：

```
//Has been Shielded.
index = PARENT.ddlb_text.finditem(PARENT.ddlb_text.text,1)
index ++
IF index > 1 AND gb_register = FALSE THEN //3
  PARENT.ddlb_text.text = PARENT.is_etitle
  messagebox("提示","对不起，你尚未注册，只能阅读每一部分的第一篇文章！欢迎注册!")
  THIS.setfocus()
  RETURN
END IF //3
ls_text = PARENT.ddlb_text.text(index)
IF ls_text = "" THEN //10
  index --
  PARENT.ddlb_text.text = PARENT.ddlb_text.text(index)
  RETURN
END IF //10
PARENT.ddlb_text.text = PARENT.ddlb_text.text(index)
PARENT.ddlb_text.triggerevent(selectionchanged!)
RETURN
```

> 这个就是点击“下篇”按钮时的过程，判断是否大于第一题和是否未注册，两者条件都满足时跳出 NAG。  
> `gb_register` 是个全局变量，是否注册的标志。再次搜索 `gb_register` 可以找到很多赋值语句，说明程序多处进行校验。  
> 注册过程就不贴了，感觉还是比较复杂，我没看明白，只好想到用爆破的方法。于是艰苦的调试过程开始了。  
> 
> 众所周知，PB 程序类似于 VB 的 pcode，是解释执行的，调试时在 `pbvm80.dll` 中转，  
> 与 pcode 又有很大不同，你几乎找不到程序在什么地方进行比较，什么地方根据比较结果进行跳转（至少没有人公开过）。  
> 用 OD 载入程序，运行，点“下篇”弹出 NAG，回到 OD，F12 暂停，Alt+F9 运行，回到程序点确定，程序中断，  
> 不断 F8 单步执行（按住不放好了），最终发现程序在下面一段反复执行：

```
10CEEB52  MOV ECX,DWORD PTR SS:[EBP-20]
10CEEB55  XOR EAX,EAX
10CEEB57  ADD ECX,EDX
10CEEB59  MOV AX,WORD PTR DS:[ECX]
10CEEB5C  LEA EAX,DWORD PTR DS:[EAX+EAX*2]
10CEEB5F  SHL EAX,2
10CEEB62  MOV EBX,DWORD PTR DS:[EAX+10DF2B84]
10CEEB68  LEA EDX,DWORD PTR DS:[EDX+EBX*2+2]
10CEEB6C  MOV DWORD PTR DS:[ESI+14],EDX
10CEEB6F  MOV EDX,DWORD PTR SS:[EBP-24]
10CEEB72  PUSH EDX
10CEEB73  LEA EDX,DWORD PTR SS:[EBP-E8]
10CEEB79  PUSH EDX
10CEEB7A  PUSH EDI
10CEEB7B  PUSH ECX
10CEEB7C  PUSH ESI
10CEEB7D  CALL DWORD PTR DS:[EAX+10DF2B80]          --->关键，程序解释入口
10CEEB83  MOV DWORD PTR SS:[EBP-4],EAX
10CEEB86  MOV EAX,DWORD PTR DS:[EDI+24C]
10CEEB8C  ADD ESP,14
10CEEB8F  TEST EAX,EAX
10CEEB91  JE SHORT PBVM80.10CEEBAA
10CEEB93  PUSH EDI
10CEEB94  MOV DWORD PTR SS:[EBP-4],1
10CEEB9B  CALL PBVM80.10CEE640
10CEEBA0  ADD ESP,4
10CEEBA3  MOV DWORD PTR SS:[EBP-18],EAX
10CEEBA6  TEST EAX,EAX
10CEEBA8  JE SHORT PBVM80.10CEEBBF
10CEEBAA  MOV EAX,DWORD PTR DS:[ESI+38]
10CEEBAD  TEST EAX,EAX
10CEEBAF  JE SHORT PBVM80.10CEEBB8
10CEEBB1  MOV DWORD PTR SS:[EBP-4],0
10CEEBB8  MOV EAX,DWORD PTR DS:[ESI+18]
10CEEBBB  TEST EAX,EAX
10CEEBBD  JE SHORT PBVM80.10CEEB41
```

> 根据对 pcode 的了解，猜测 `10CEEB7D CALL DWORD PTR DS:[EAX+10DF2B80]` 即是程序解释的入口。  
> 不断跟踪发现解释原理是这样的：根据 `EAX` 值的不同，程序 call 到不同位置执行不同任务，  
> 如读取程序的代码，然后解释执行程序等。  
> 猜测当 `EAX=2AC` 或 `0C` 或 `18` 时，会读取原程序代码，等于其它一些数值时会执行原程序。  
> 不能直接在 `10CEEB7D` 处下断，因为一返回程序就会中断。  
> 于是对“下篇”按钮下 `WM_lbuttonup` 消息断点，点击后中断，返回 OD，在 `10CEEB7D` 处下断，F9 运行。  
> 当 `EAX` 不等于 `2AC` 或 `0C` 或 `18` 时跟 F7 进去看看。经过 n 次（未统计，估计>200），  
> 终于在 `EAX=93C` 时找到了需要的代码，跟进去程序为：

```
10D967B0  MOV ECX,DWORD PTR SS:[ESP+C]
10D967B4  PUSH EBX
10D967B5  PUSH ESI
10D967B6  PUSH EDI
10D967B7  MOV ESI,DWORD PTR DS:[ECX+10E]
10D967BD  XOR EBX,EBX
10D967BF  LEA EDX,DWORD PTR DS:[ESI-38]
10D967C2  MOV DWORD PTR DS:[ECX+10E],EDX
10D967C8 >MOV EDI,DWORD PTR DS:[ESI-1C]
10D967CB  MOV EAX,DWORD PTR DS:[EDX]
10D967CD  CMP EAX,EDI                       --->对应 index > 1 的比较
10D967CF  MOV EAX,1
10D967D4  SETG BL                                --->对应 index > 1 的比较
10D967D7  MOV WORD PTR DS:[EDX],BX
10D967DA  MOV BL,BYTE PTR DS:[EDX+4]
10D967DD  TEST AL,BL
10D967DF  JNZ SHORT PBVM80.10D967EA
10D967E1  TEST BYTE PTR DS:[ESI-18],AL
10D967E4  JNZ SHORT PBVM80.10D967EA
10D967E6  XOR EDX,EDX
10D967E8  JMP SHORT PBVM80.10D967EC
10D967EA  MOV EDX,EAX
10D967EC  MOV ESI,DWORD PTR DS:[ECX+10E]
10D967F2  OR DH,5
10D967F5  POP EDI
10D967F6  MOV WORD PTR DS:[ESI+4],DX
10D967FA  MOV EDX,DWORD PTR DS:[ECX+10E]
10D96800  POP ESI
10D96801  POP EBX
10D96802  MOV WORD PTR DS:[EDX+6],7
10D96808  MOV EDX,DWORD PTR DS:[ECX+10E]
10D9680E  MOV WORD PTR DS:[EDX+1A],0
10D96814  MOV EDX,DWORD PTR DS:[ECX+10E]
10D9681A  ADD EDX,1C
10D9681D  MOV DWORD PTR DS:[ECX+10E],EDX
10D96823  RETN
```

> 继续跟踪，当 `EAX=804` 时，跟进得到如下代码：

```
10D95580  MOV ECX,DWORD PTR SS:[ESP+C]
10D95584  PUSH EBX
10D95585  PUSH ESI
10D95586  XOR EBX,EBX
10D95588  MOV ESI,DWORD PTR DS:[ECX+10E]
10D9558E  LEA EDX,DWORD PTR DS:[ESI-38]
10D95591  MOV DWORD PTR DS:[ECX+10E],EDX
10D95597  MOV AX,WORD PTR DS:[EDX]
10D9559A >CMP AX,WORD PTR DS:[ESI-1C]     --->对应 gb_register = FALSE 的比较
10D9559E  MOV EAX,1
10D955A3  SETE BL                      --->对应 gb_register = FALSE 的比较
10D955A6  MOV WORD PTR DS:[EDX],BX
10D955A9  MOV BL,BYTE PTR DS:[EDX+4]
10D955AC  TEST AL,BL
10D955AE  JNZ SHORT PBVM80.10D955B9
10D955B0  TEST BYTE PTR DS:[ESI-18],AL
10D955B3  JNZ SHORT PBVM80.10D955B9
10D955B5  XOR EDX,EDX
10D955B7  JMP SHORT PBVM80.10D955BB
10D955B9  MOV EDX,EAX
10D955BB  MOV ESI,DWORD PTR DS:[ECX+10E]
10D955C1  OR DH,5
10D955C4  MOV WORD PTR DS:[ESI+4],DX
10D955C8  MOV EDX,DWORD PTR DS:[ECX+10E]
10D955CE  POP ESI
10D955CF  POP EBX
10D955D0  MOV WORD PTR DS:[EDX+6],7
10D955D6  MOV EDX,DWORD PTR DS:[ECX+10E]
10D955DC  MOV WORD PTR DS:[EDX+1A],0
10D955E2  MOV EDX,DWORD PTR DS:[ECX+10E]
10D955E8  ADD EDX,1C
10D955EB  MOV DWORD PTR DS:[ECX+10E],EDX
10D955F1  RETN
```

> 找到比较代码还是不能爆破的，因为这些代码是在 `pbvm80.dll` 中，程序的正确运行是需要它的。  
> 还要找到原程序的代码并修改。这个可没有现成经验。  
> 我的修改思路是将 `index > 1` 改成 `index > 99999`（随便你修改），（`gb_register` 是变量，不容易修改的）  
> 在 `10D967C8 >MOV EDI,DWORD PTR DS:[ESI-1C]` 处观察 `ESI-1C` 的值为 `A09420`，对 `A09420` 下硬件写入断点，  
> 运行时会中断很多次，在最后中断在 `10D9559A >CMP AX,WORD PTR DS:[ESI-1C]` 的前一次看，  
> 得到程序读入 `1F30658` 处的值（即 0），在数据窗口中显示为：  
> `01000000390001000300F40931000100000039000100010006`，用 Winhex 打开程序，搜索，  
> 找到地址：`0013AC27` 处，修改 `0100` 为 `FFFF`，保存。  
> 再次运行程序，点下篇，没有 NAG 了吧？  
> 实际程序还没有爆破完全，但是功能限制已经取消了。  
> 
> 小结一下：  
> `10CEEB7D CALL DWORD PTR DS:[EAX+10DF2B80]` 是 PB 程序解释入口，`EAX` 值不同代表不同功能  
> 我猜测 `EAX=93C` 时比较是否大于，`EAX=804` 时比较逻辑值是否相等。  
> 其它的值代表的含义希望大家继续跟踪贴出来分享。  
> 
> 调试程序好累，写文章更累。真的需要很好地耐心。休息一下，明天继续爆破注册。那个稍微容易点。

今天讲一下爆破注册这个程序。

反编译显示程序调用 `sapiyy.dll`、`RegClient.dll` 进行注册。在 OD 中查看 `sapiyy.dll` 的函数名称，发现很多与注册有关的函数：

| 名称位于 sapiyy | 地址 | 名称 | 注释 |
| :--- | :--- | :--- | :--- |
| 01ADA1AC | RegClient.CheckRegister | | |
| 01AD2860 | DoCheckRegister | 检查是否注册 | |
| 01AD22C0 | _DoCheckRegister2@16 | | |
| 01AD2DC0 | DoRegister | | |
| 01AD2CF0 | DoUnRegister | | |
| 01ADA1B8 | RegClient.GetErrorString | | |
| 01AD1780 | gethdserial | 获取硬件 ID | |
| 01AD1A80 | getsourestring1024 | | |
| 01AD1D40 | getsourestring128 | | |
| 01AD1BE0 | getsourestring256 | | |
| 01AD1920 | getsourestring512 | 好像是加密注册码的逆向处理 | |
| 01ADA1B4 | RegClient.GetTokenString | | |
| 01ADA17C | MSVCRT._mbscmp | | |
| 01ADA1A8 | RegClient.Register | | |
| 01AD17D0 | secstring | 注册码处理函数 | |
| 01ADA1B0 | RegClient.UnRegister | | |

在 `gethdserial` 函数上下断，F9 运行，中断后 F12 返回，程序回到这里（在 `pbvm80.dll` 中）：

```
10BBB5E3  POP ESI
10BBB5E4  POP EDI
```



### 调用外部函数入口分析

`10BBB5E5  CALL DWORD PTR SS:[EBP+8]`  
`10BBB5E8  CMP ESP,DWORD PTR SS:[EBP-8]`  ------>停在这  
`10BBB5EB  JE SHORT PBVM80.10BBB5F7`

明显 `10BBB5E5  CALL DWORD PTR SS:[EBP+8]` 这句就是调用函数。下断，`F9` 运行。

发现它会 `call` 到不同地址。原来这就是 `pb` 程序调用外部函数的入口。

如果想跟踪注册算法，在这里下断应该可以得到一些信息，因为程序调用外部函数处理注册码的。

但是我对追注册码实在没有信心，我只想找到爆破的地方。

如果程序调用外部函数判断是否注册并返回 `bool` 值，那我们修改返回值不就 `OK`?

下断后继续运行，每次中断看看 `Call` 的外部函数名称，觉得是判断注册的就跟进去，如果是处理注册码的就继续运行。

### `DoCheckRegister` 函数分析

终于在一次下断后来到 `DoCheckRegister` 函数。跟进去看：

```
01AD2860  >SUB  ESP,1C4
01AD2866    PUSH  ESI
01AD2867    PUSH  EDI
01AD2868    MOV   EDI,DWORD PTR SS:[ESP+1D8]
```

省略部分代码

```
01AD28D8    PUSH  EAX
01AD28D9    PUSH  ECX
01AD28DA    PUSH  sapiyy.01ADDC30
01AD28DF    PUSH  sapiyy.01ADDC30
01AD28E4    PUSH  sapiyy.01ADC3BC                      ; ASCII "I0qOWoaJ584Lz/CUMSi3qN4anWMV6Zox"
01AD28E9    PUSH  sapiyy.01ADC398                      ; ASCII "gX93+hAOo1HQnq1enogxlpL2kp0gUgF7"
01AD28EE    PUSH  EDX
01AD28EF    PUSH  sapiyy.01ADC294                      ; ASCII "smskb"
01AD28F4    CALL  <JMP.&RegClient.CheckRegister>
01AD28F9    CMP   EAX,9
01AD28FC    JNZ   SHORT sapiyy.01AD290B
01AD28FE    POP   EDI
01AD28FF    XOR   EAX,EAX
01AD2901    POP   ESI
01AD2902    ADD   ESP,1C4
01AD2908    RETN  10
```

省略部分代码

```
01AD295E    PUSH  EAX
01AD295F    PUSH  ECX
01AD2960    PUSH  sapiyy.01ADDC30
01AD2965    PUSH  sapiyy.01ADDC30
01AD296A    PUSH  sapiyy.01ADC3BC                      ; ASCII "I0qOWoaJ584Lz/CUMSi3qN4anWMV6Zox"
01AD296F    PUSH  sapiyy.01ADC398                      ; ASCII "gX93+hAOo1HQnq1enogxlpL2kp0gUgF7"
01AD2974    PUSH  EDX
01AD2975    PUSH  sapiyy.01ADC294                      ; ASCII "smskb"
01AD297A    CALL  <JMP.&RegClient.CheckRegister>
01AD297F    MOV   EDX,EAX
01AD2981    CMP   EDX,ESI
01AD2983    JE    SHORT sapiyy.01AD29EF
01AD2985    CMP   EDX,1F
01AD2988    JE    sapiyy.01AD2CD9
01AD298E    CMP   EDX,25
01AD2991    JE    sapiyy.01AD2CD9
01AD2997    CMP   EDX,26
01AD299A    JE    sapiyy.01AD2CD9
01AD29A0    CMP   EDX,27
01AD29A3    JE    sapiyy.01AD2CD9
01AD29A9    CMP   EDX,28
01AD29AC    JE    sapiyy.01AD2CD9
01AD29B2    CMP   EDX,29
01AD29B5    JE    sapiyy.01AD2CD9
01AD29BB    CMP   EDX,2A
01AD29BE    JE    sapiyy.01AD2CD9
01AD29C4    MOV   ECX,20
01AD29C9    XOR   EAX,EAX
01AD29CB    LEA   EDI,DWORD PTR SS:[ESP+C8]
01AD29D2    REP   STOS  DWORD PTR ES:[EDI]
01AD29D4    LEA   EAX,DWORD PTR SS:[ESP+C8]
01AD29DB    PUSH  EAX
01AD29DC    PUSH  EDX
01AD29DD    CALL  <JMP.&RegClient.GetErrorString>
01AD29E2    POP   EDI
01AD29E3    XOR   EAX,EAX
01AD29E5    POP   ESI
01AD29E6    ADD   ESP,1C4
01AD29EC    RETN  10
```

看到的那一串古怪字符明显与注册码有关。看 `ret` 前几个代码，发现：

```
01AD29DD  CALL  <JMP.&RegClient.GetErrorString>
01AD29E2  POP   EDI
01AD29E3  XOR   EAX,EAX
```

应该是注册不成功的置 `0` 标志位。修改 `EAX` 为 `1`，运行，程序不再有注册提示，按钮变成了“注销注册”。成功！

### 小结

`pb` 程序本身调试是很困难的，但是如果调用外部程序注册调试起来将会轻松得多，因为调用地址是固定的，只要在调用处下断，可以清楚看到用到哪些函数进行运算注册。

写这篇文章没有其它目的，纯粹是为了方便自己记忆，没有太多技术上的东西，`pb` 与 `vb` `pcode` 相似，但不同之处有太多，难度大很多。

一段代码执行起来跟踪要很久，下面这三行就花了我将近 1 个小时才找关键，真的需要运气。

```
index = PARENT.ddlb_text.finditem(PARENT.ddlb_text.text,1)
index ++
IF index > 1 AND gb_register = FALSE THEN //3
```

`pb` 程序没有直接读取原程序，而是将原程序的代码先复制到临时地址，然后不断读取临时地址并执行。

所以调试时找到临时地址还是不够的，还要找到原程序的地址才能爆破，很多机器码如 `jz`、`jnz` 等我也不知道是什么。

毕竟网上有关 `pb` 的调试太少了，各位老大可能都是直接反编译写出注册机的。我没有这个本事，只有寻求爆破了。

真的希望各位同仁能多写些 `pb` 调试的文章，把经验放给大家共享，让 `pb` 程序的破解不再是难题。
