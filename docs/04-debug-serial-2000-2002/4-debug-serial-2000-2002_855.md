
# QQ 密码侦探终结者

> 原文：[`www.pediy.com/kssd/pediy04/kanxue056.htm`](https://www.pediy.com/kssd/pediy04/kanxue056.htm)

## 基本信息

我的第一个作品！

作者：`free`

作案目标：QQ 密码侦探终结者(`unqqspyz`)

作案工具：`trw2000`

## 过程

### 一、运行程序，填好注册码

```
用户名:free
注册码:12345678
```

启动 `trw`

*   `Ctrl+N` 唤出 `trw`
*   `bpx hmemcpy` 设断
*   `g`
*   按“立即注册”，程序被拦截!
*   `bc *` 取消断点
*   `PMODULE` 来到 `unqqspyz` 上空
*   按 7 次 `f12` 后(第八次出现错误提示),程序结束于:`00425966`

### 二、开始追踪

按 `F10` 24 次来到：`0044CAD6`

```asm
0167:0044CAA2  MOV       EAX,[EBX+02E8]
0167:0044CAA8  CALL      00425938
0167:0044CAAD  MOV       EAX,[EBP-10]
0167:0044CAB0  PUSH      EAX
0167:0044CAB1  LEA       EAX,[EBP-14]
0167:0044CAB4  PUSH      EAX
0167:0044CAB5  LEA       EDX,[EBP-18]
0167:0044CAB8  MOV       EAX,[EBX+0300]
0167:0044CABE  CALL      00425938
0167:0044CAC3  MOV       EDX,[EBP-18]
0167:0044CAC6  MOV       ECX,0044CD0C
0167:0044CACB  MOV       EAX,EBX
0167:0044CACD  CALL      0044C8C0
0167:0044CAD2  MOV       EDX,[EBP-14]
0167:0044CAD5  POP       EAX       ------->在此处 d edx 可得到真正的注册码
0167:0044CAD6  CALL      00403D40  ------>此处为关键 CALL,F8 追入来到下面！
```

```asm
0167:00403D40  PUSH      EBX
0167:00403D41  PUSH      ESI
0167:00403D42  PUSH      EDI
0167:00403D43  MOV       ESI,EAX
0167:00403D45  MOV       EDI,EDX
0167:00403D47  CMP       EAX,EDX     ------>比较注册码！
```

在此处 `d EAX` 得假注册码(`12345678`)，`d EDX` 得真注册码(`SmxaUT09YVd4dmRtVj`)！

```asm
0167:00403D49  JZ        NEAR 00403DDE
0167:00403D4F  TEST      ESI,ESI
0167:00403D51  JZ        00403DBB
0167:00403D53  TEST      EDI,EDI
0167:00403D55  JZ        00403DC2
0167:00403D57  MOV       EAX,[ESI-04]
0167:00403D5A  MOV       EDX,[EDI-04]
0167:00403D5D  SUB       EAX,EDX
```

## 收工

注册名：`free`  注册码：`SmxaUT09YVd4dmRtVj`

由于菜弟初学破解不足之处，请各位高兄多指点！
