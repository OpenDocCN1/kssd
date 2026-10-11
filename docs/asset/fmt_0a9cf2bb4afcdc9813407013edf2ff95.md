
```
004C8BC2  JNZ     SHORT unpack.004C8BA5  循环，这个循环是把试验码的各位相加：1+2+3+4+5+6+-----+1=48，值存放在 `ESI`   

004C8BC4  MOV     EAX, [LOCAL.2]  把上面取出的 6 位试验码：357864 入 `EAX`   

004C8BC7  MOV     EDX, unpack.004C8C80  固定字串值 472008 入 `EDX`   

004C8BCC  CALL    unpack.00404E60   

004C8BD1  JB      SHORT unpack.004C8BE6  比较：357864 和 472008，低于就跳走!一跳就失败!   

004C8BD3  MOV     EAX, [LOCAL.2]  把上面取出的 6 位试验码：357864 入 `EAX`   

004C8BD6  MOV     EDX, unpack.004C8C90  固定字串值 500000 入 `EDX`   

004C8BDB  CALL    unpack.00404E60   

004C8BE0  JA      SHORT unpack.004C8BE6  比较：357864 和 500000，高于就跳走!一跳就失败!   

004C8BE2  MOV     BL, 1  不跳到这里，赋 `BL` 的值为 1   

004C8BE4  JMP     SHORT unpack.004C8BE8  无条件跳!   

004C8BE6  XOR     EBX, EBX  跳到这里 `EBX` 清 0   

004C8BE8  TEST    BL, BL  004C8BE2 处跳来，测试 `BL` 的值!   

004C8BEA  JE      SHORT unpack.004C8C3C  为 0 就跳，跳就失败!不跳往下!   

004C8BEC  XOR     EBX, EBX  `EBX` 清 0   

004C8BEE  CMP     ESI, 39  比较 `ESI` 与 39，`ESI` 的值是：48   

004C8BF1  JL      SHORT unpack.004C8BFA  小于就跳走   

004C8BF3  CMP     ESI, 39  比较 `ESI` 与 39   

004C8BF6  JG      SHORT unpack.004C8BFA  大于就跳走!相等呢？不跳!就成功   

004C8BF8  MOV     BL, 1  赋 `BL` 的值为 1   

004C8BFA  CMP     ESI, 47  同上!   

004C8BFD  JL      SHORT unpack.004C8C06   

004C8BFF  CMP     ESI, 48  同上!   

004C8C02  JG      SHORT unpack.004C8C06  大于就跳，等于 48 就不跳!就成功   

004C8C04  MOV     BL, 1   

004C8C06  CMP     ESI, 64   

004C8C09  JL      SHORT unpack.004C8C12   

004C8C0B  CMP     ESI, 65   

004C8C0E  JG      SHORT unpack.004C8C12  等于 65 就成功!   

004C8C10  MOV     BL, 1   

004C8C12  CMP     ESI, 6F   

004C8C15  JL      SHORT unpack.004C8C1E   

004C8C17  CMP     ESI, 6F   

004C8C1A  JG      SHORT unpack.004C8C1E  等于 6F 就成功!   

004C8C1C  MOV     BL, 1   

004C8C1E  CMP     ESI, 7D   

004C8C21  JL      SHORT unpack.004C8C2A   

004C8C23  CMP     ESI, 7D   

004C8C26  JG      SHORT unpack.004C8C2A  等于 7D 就成功   

004C8C28  MOV     BL, 1   

004C8C2A  CMP     ESI, 88   

004C8C30  JL      SHORT unpack.004C8C3C   

004C8C32  CMP     ESI, 88   

004C8C38  JG      SHORT unpack.004C8C3C  等于 88 就成功!   

004C8C3A  MOV     BL, 1  赋 `BL` 的值为 1   

004C8C3C  XOR     EAX, EAX   

004C8C3E  POP     EDX          

004C8C3F  POP     ECX          

004C8C40  POP     ECX          

004C8C41  MOV     DWORD PTR FS:[EAX], EDX       

004C8C44  PUSH    unpack.004C8C5E   

004C8C49  LEA     EAX, [LOCAL.5]   

004C8C4C  MOV     EDX, 5   

004C8C51  CALL    unpack.00404A88   

004C8C56  RETN
```

### 算法分析

这个软件注册成功必须满足下列条件：

第一：注册码必须是 16 位!

第二：从注册码中取出 6 位，必须不能小于 472008，不能大于 500000，注册码的 6 位取法：取注册码的最后 1 位，与 3 除，由除的余数来控制从第几位取，如最后一位是 1，则余数为 1，1+2=3，就从第 3 位取，如是 2，则余数是 2,2+2=4，从第 4 位取，以下类推，然后每隔一位取 1 位，取完 6 位为止。

第三：把注册码的各位相加，设值为 `X`，则 `X` 必须等于：39、48、65、6F、7D、88 中的任何一个就成功!

给几个可用的注册码：`1243858867111101`、`8488999999999999`

`4dd466d0e1e0cc123afbe504da2462ca`
