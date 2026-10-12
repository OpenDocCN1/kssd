
# Offline Explorer 1.3

> 原文：[`www.pediy.com/kssd/pediy03/forum291.htm`](https://www.pediy.com/kssd/pediy03/forum291.htm)

软件下载：

```
ftp://ftp.bj.software.chinese.com/software/soft_offbrowse/poe13273sr3-f.exe
```

### 一、运行 trw

### 二、填上注册信息（先任意填）

`name: yubing`

`code: 78787878`

### 三、操作 trw

1. 按 `Ctrl + N` 进入 `trw`。
2. 输入断点命令：
   ```
   :bpx hmemcpy
   ```
3. 输入运行命令：
   ```
   :g
   ```
   （按 `Enter` 返回）。
4. 按“确定”被中断后，按 `F12` 进入如下代码：

```
015F:004BF72A  LEA      EDX,[EBP-08]
015F:004BF72D  MOV      EAX,[EBX+02E8]
015F:004BF733  CALL     00433938
015F:004BF738  LEA      ECX,[EBP-08]
015F:004BF73B  LEA      EDX,[EBP-04]
015F:004BF73E  MOV      EAX,[004E4C9C]
015F:004BF743  MOV      EAX,[EAX]
015F:004BF745  CALL     004D1940   //注册比较部分，按 F8 追入
015F:004BF74A  TEST     AL,AL      //检测标志 al
015F:004BF74C  JZ       004BF7CC   //跳则失败(NO JUMP)
015F:004BF74E  MOV      EAX,[004E4C9C]
015F:004BF753  MOV      EAX,[EAX]
015F:004BF755  MOV      BYTE [EAX+064C],01
```

### 四、追踪注册比较

如上所述，注册比较部分追入后，按 `F10` 到以下代码：

```
015F:004D1DD2  CALL     00403E70
015F:004D1DD7  CMP      EBX,EAX
015F:004D1DD9  JNG      004D1DA7
015F:004D1DDB  MOV      EDX,06D363C1
015F:004D1DE0  MOV      EAX,[EBP-04]
015F:004D1DE3  CALL     0047C580
015F:004D1DE8  LEA      EDX,[EBP-08]
015F:004D1DEB  CALL     004D1324
015F:004D1DF0  MOV      EDX,[EBP-08]  //d edx 正确的注册码
015F:004D1DF3  MOV      EAX,[ESI]      //d eax 你输入的注册码
015F:004D1DF5  CALL     00403F80       //两注册码的比较
015F:004D1DFA  SETZ     BL
015F:004D1DFD  XOR      EAX,EAX
015F:004D1DFF  POP      EDX
015F:004D1E00  POP      ECX
```

当找到正确的注册码后，用笔抄下，就可以注册了。

---
十三少
China Cracking Group
2000.09.04
`4dd466d0e1e0cc123afbe504da2462ca`
