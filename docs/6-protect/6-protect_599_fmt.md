
# VMP1.7 讲解【破解 vmp 程序的关键点】

> 原文：[`www.pediy.com/kssd/pediy11/106890.html`](https://www.pediy.com/kssd/pediy11/106890.html)

有点标题党,其实贴的是海风月影的帖子 `http://bbs.pediy.com/showthread.php?t=82618`

之后的学习笔记而已.通过分析代码,发觉似乎理解了一点.喜不自禁.

故写下来与大家分享一下和日后回顾一下![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")

并感谢大牛们的分析,希望能继续爆料![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")

本笔记所使用的 `demo` 是由 `VMP1.7` 加的 `notepad.exe` 的 `EP`.

主要内容为:分析以 `VMP` 处,调用 `IsDebuggerPresent`,判断返回值后走哪条流程..

由于 `VMP1.7` `Handle` 垃圾指令太多,故而写了个 `OD` 小插件 `F7` 分析 `Handle` 转义为对应 `x86` 指令.

所以看到的会有点不同了![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")

先上 `IsDebuggerPresent` 返回 `1` 的代码,代码乱且多,请看注释的关键代码![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")

```
vm_call_IsDebuggerPresent

call 的返回值为:00000001     ;返回值为 1

vm_popDword reg[14],数据为:00000000
vm_popDword reg[2C],数据为:7C800000
vm_popDword reg[28],数据为:00000001 ;保存 EAX
vm_popDword reg[08],数据为:0007FF88
vm_popDword reg[3C],数据为:7C800000
vm_popDword reg[04],数据为:000A0608
vm_popDword reg[34],数据为:0000022A
vm_popDword reg[30],数据为:0007FF74
vm_popDword reg[24],数据为:0007FF98
vm_popDword reg[0C],数据为:00000206
vm_popDword reg[20],数据为:0103A148
vm_popDword reg[00],数据为:4E89BA21
vm_pushDword,数据为:11426946 ;后面的跳转地址 1
vm_pushDword,数据为:114217BD ;后面的跳转地址 2
vm_push_esp
vm_pushbyte reg[28],数据为:01 ;压入 AL 的值
vm_pushbyte reg[28],数据为:01 ;压入 AL 的值
vm_nna_byte,数据为:FE ;第一处 not_not_and  (~al) & ~(al)
vm_popDword reg[18],数据为:00000282
vm_push_esp
vm_check,数据为:00FE ;这个虽然叫 check 指令,但其实就是 mov BYTE ptr ds:[esp],BYTE ptr ds:[esp+4] ,所以这里也压入的是上面的 NNA 过后的 FE
vm_nna_byte,数据为:01 ;第二处 not_not_and,由于是残留的,所以相当帖子中说的三次 not_not_and 了, 也就是说上面的代码就等价于 test al,al 操作了.
vm_popDword reg[00],数据为:00000202 ;保存 eflags,用它来判断是否该跳转的哈
vm_popbyte reg[28],数据为:01
vm_pushword,数据为:0004 ;两个候选答案,04 00
vm_pushDword reg[00],数据为:00000202 ;然后又压入 eflags
vm_push_esp
vm_pushDword,数据为:00000202 ;mov DWORD ptr ds:[esp],DWORD ptr ds:[esp+4]
vm_nna_Dword,数据为:FFFFFDFD ;第一处 NNA,即 (~eflags) & (~eflags)
vm_popDword reg[18],数据为:00000282
vm_pushDword,数据为:FFFFFFBF ;这个是事先 NNA 过的数据,NOT 一下 FFFFFFBF 就为 40 了
vm_nna_Dword,数据为:00000000 ;则这里是第三次了,相当于 test eflags,40 ,也就是判断 ZF 啦
vm_popDword reg[20],数据为:00000246
vm_operation_Dword [esp+4],数据为:00000000 ;这里把上面的结果处理了一下,如果为 0,转换后的结果为 0,如果不为 0,就转换为 4 了,两个地址跳表嘛.
vm_popDword reg[20],数据为:00000246
vm_add_Dword [esp+4],[esp],数据为:0007F790 ;这里就算下要跳向地址的地址,后面就是取出 0007F790 的内容,然后解密,改变 esi 的值,jump 过去..
vm_popDword reg[38],数据为:00000206
vm_pushDword,数据为:114217BD
vm_popDword reg[0C],数据为:114217BD
vm_popDword reg[20],数据为:114217BD
vm_popDword reg[38],数据为:11426946
vm_pushDword reg[0C],数据为:114217BD
vm_push_esp
vm_pushDword,数据为:114217BD
vm_popDword reg[18],数据为:114217BD
vm_pushDword reg[18],数据为:114217BD
vm_nna_Dword,数据为:EEBDE842
vm_popDword reg[20],数据为:00000286
vm_pushDword,数据为:EFBEE2A6
vm_nna_Dword,数据为:10401519
vm_popDword reg[10],数据为:00000202
vm_pushDword,数据为:10411D59
vm_pushDword reg[18],数据为:114217BD
vm_nna_Dword,数据为:EEBCE002
vm_popDword reg[38],数据为:00000282
vm_nna_Dword,数据为:01030AE4 ;解密出地址
vm_popDword reg[1C],数据为:00000206
vm_popDword reg[38],数据为:01030AE4
vm_pushDword reg[10],数据为:00000202
vm_pushDword reg[20],数据为:00000286
vm_pushDword reg[08],数据为:0007FF88
vm_pushDword reg[34],数据为:0000022A
vm_pushDword reg[2C],数据为:7C800000
vm_pushDword reg[24],数据为:0007FF98
vm_pushDword reg[30],数据为:0007FF74,
vm_pushDword reg[04],数据为:000A0608
vm_pushDword reg[00],数据为:00000202
vm_pushDword reg[24],数据为:0007FF98
vm_pushDword reg[28],数据为:00000001
vm_pushDword reg[14],数据为:00000000
vm_pushDword reg[38],数据为:01030AE4
vm_jump,数据为:01030AE4 ;跳过去,流程改变..
```

总结下上面的,

`VMP` 跳转之前,会先在栈里存放着两个加密过后的地址数据.

一个为条件成立跳的,一个为不成立跳的.

经过三次 `not_not_and al,al` 之后,

然后再 `not_not_and eflags,40` 来判断 `ZF` 位是否被置位.

也就相当于我们熟悉的两句代码

```
test al,al
je xxx
```

好了,就到这里了,然后贴下 `IsDebuggerPresent` 返回 `0` 的流程,对照的看看![](img/6821a6175013046370981b6035d43ab4.jpg "使眼色")

```
vm_call_IsDebuggerPresent

call 的返回值为:00000000

vm_popDword reg[28],数据为:00000000
vm_pushDword,数据为:11426946
vm_pushDword,数据为:114217BD
vm_nna_byte,数据为:FF
vm_nna_byte,数据为:00
vm_popDword reg[00],数据为:00000246 ;保存 eflags,很明显看出 ZF 位是被置位了
vm_popbyte reg[28],数据为:00
vm_pushword,数据为:0004
vm_pushDword reg[00],数据为:00000246
vm_push_esp
vm_pushDword,数据为:00000246
vm_nna_Dword,数据为:FFFFFDB9
vm_popDword reg[18],数据为:00000282
vm_pushDword,数据为:FFFFFFBF
vm_nna_Dword,数据为:00000040
vm_popDword reg[20],数据为:00000202
vm_operation_Dword [esp+4],数据为:00000004 ;这里计算的结果就为 4 了,也就是跳到第二个地址去
vm_popDword reg[20],数据为:00000202
vm_add_Dword [esp+4],[esp],数据为:0007F794 ;0007F794 上段代码此处为 0007F790
vm_pushDword,数据为:11426946 ;取出 0007F794 加密后地址数据
vm_nna_Dword,数据为:0103741F ;解密出要跳转的地址
vm_jump,数据为:0103741F ;jump
vm_popDword reg[2C],数据为:00000000,pcode 地址为:0103741F

Log file closed
```

`NOTEPAD.vmp.rar`
