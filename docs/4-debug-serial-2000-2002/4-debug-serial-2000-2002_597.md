
# 炒股理财

> 原文：[`www.pediy.com/kssd/pediy03/forum763.htm`](https://www.pediy.com/kssd/pediy03/forum763.htm)

**炒股理财算法分析**

作者：`THK`

目的：学习找注册码

关于脱壳：用 `caspr1.012` 脱壳，去掉自检错误 `messagebox`。

解除 `ANTI-W32DAASM` 功能（见 `http://001.com.cn/forum/toye/11626.html`，方法与文中程序一的方法一模一样）

```
016F:0041CA86  XOR      EDI,EDI
016F:0041CA88  PUSH     BYTE +04
016F:0041CA8A  LEA      EAX,[EBP-74]
016F:0041CA8D  PUSH     EAX
016F:0041CA8E  PUSH     ESI
016F:0041CA8F  CALL     0041C794     //将输入的假码变换，内容见下。
016F:0041CA94  ADD      ESP,BYTE +0C
016F:0041CA97  INC      EDI
016F:0041CA98  CMP      EDI,BYTE +03
016F:0041CA9B  JL       0041CA88     //循环 3 次
016F:0041CA9D  XOR      DWORD [EBP-74],81079516 //变换结果于 81079516 异或
016F:0041CAA4  MOV      EDX,[EBP-74]
016F:0041CAA7  LEA      EAX,[EBP-04]
016F:0041CAAA  ADD      [EBP-78],EDX
016F:0041CAAD  CALL     0055C4F0
016F:0041CAB2  SUB      [EBP-78],EAX
016F:0041CAB5  MOV      EAX,ESI
016F:0041CAB7  CALL     0041C428     //由用户名和硬盘 ID 号生成一个数
016F:0041CABC  SUB      [EBP-78],EAX //减异或结果
016F:0041CABF  CMP      DWORD [EBP-78],BYTE +00 //为 0 吗
016F:0041CAC3  JNZ      NEAR 0041CC89    //跳就死
016F:0041CAC9  MOV      EDX,80000002     //下面还有一处比较和这里基本一样
```

\-----------------------------------------------------------------------

在 `0041CA8F` 处按 `F8` 来到这里

```
016F:0041C794  PUSH     EBP
016F:0041C795  MOV      EBP,ESP
016F:0041C797  ADD      ESP,FFFFFDFC
016F:0041C79D  PUSH     EBX
016F:0041C79E  PUSH     ESI
016F:0041C79F  MOV      ECX,[EBP+10]
016F:0041C7A2  XOR      EAX,EAX
016F:0041C7A4  LEA      ESI,[EBP+FFFFFDFC]
016F:0041C7AA  MOV      EDX,[EBP+0C]
016F:0041C7AD  CMP      ECX,EAX
016F:0041C7AF  JNG      0041C7DD
016F:0041C7B1  MOV      BL,[EDX]
016F:0041C7B3  MOV      [EBP-01],BL
016F:0041C7B6  LEA      EBX,[ECX-01]
016F:0041C7B9  CMP      EAX,EBX
016F:0041C7BB  JNZ      0041C7C7
016F:0041C7BD  MOV      EBX,[EBP+0C]
016F:0041C7C0  MOV      BL,[EBX]
016F:0041C7C2  MOV      [EBP-02],BL
016F:0041C7C5  JMP      SHORT 0041C7CD
016F:0041C7C7  MOV      BL,[EDX+01]
016F:0041C7CA  MOV      [EBP-02],BL
016F:0041C7CD  SHL      WORD [EBP-02],1  //运算核心
016F:0041C7D1  MOV      BL,[EBP-01]
016F:0041C7D4  MOV      [ESI],BL
016F:0041C7D6  INC      EAX
016F:0041C7D7  INC      ESI
016F:0041C7D8  INC      EDX
016F:0041C7D9  CMP      ECX,EAX
016F:0041C7DB  JG       0041C7B1
016F:0041C7DD  MOV      BYTE [EBP+EAX+FFFFFDFC],00
016F:0041C7E5  XOR      EAX,EAX
016F:0041C7E7  MOV      EDX,[EBP+0C]
016F:0041C7EA  MOV      ESI,EDX
016F:0041C7EC  LEA      EDX,[EBP+FFFFFDFC]
016F:0041C7F2  CMP      ECX,EAX
016F:0041C7F4  JNG      0041C801
016F:0041C7F6  MOV      BL,[EDX]
016F:0041C7F8  MOV      [ESI],BL
016F:0041C7FA  INC      EAX
016F:0041C7FB  INC      ESI
016F:0041C7FC  INC      EDX
016F:0041C7FD  CMP      ECX,EAX
016F:0041C7FF  JG       0041C7F6
016F:0041C801  LEA      EAX,[EBP+FFFFFDFC]
016F:0041C807  POP      ESI
016F:0041C808  POP      EBX
016F:0041C809  MOV      ESP,EBP
016F:0041C80B  POP      EBP
```

\--------------------------------------------------------------------

### 运算过程

假码：`12345678` //必须是 8 位

```
7856    5634    3412    1278
左移 1 位 f0      ac      68      24      //只取高 8 位
结果：2468acf0

f0ac    ac68    6824    24f0
左移 1 位 e1      58      d0      49
结果 : 49d058e1

e158    58d0    d049    49e1
左移 1 位 c2      b1      a0      93
结果 : 93a0b1c2
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*那位大客帮帮忙写出注册机\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

这里先谢了！！！

### 对于炒股理财 `1.65` 版注册码

1. 注册码为 8 位，`0....9`, `a...f` 取值，`(1)(2)(3)(4)(5)(6)(7)(8)`
2. 取 `(1)(2)` 数字，`shl 1`，如是 `(3)(4)`,... `(7)(8)` `shl 1`。这样三次得到一组数字（高端 8 位+低端 8 位）
3. 将低端 8 位数字与 `81079516` 异或 (`bpx cs:41f2b1`)
4. 利用用户名及机器码（好象与硬盘序号相关）得到 8 位数字，如果二者相等，注册成功 (`bpx cs:41f200`) `4dd466d0e1e0cc123afbe504da2462ca`
