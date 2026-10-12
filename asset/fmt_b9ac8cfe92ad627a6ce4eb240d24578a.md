
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
