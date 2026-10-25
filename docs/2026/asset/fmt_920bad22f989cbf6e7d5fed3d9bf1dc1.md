
`1777:2D75 8B560C`

`MOV DX,[BP+0C]0`

`1777:2D78 8B4E06`

`MOV CX,[BP+06]` 机器码长度

`1777:2D7B`

`C47E0E LES DI,[BP+0E]`

`1777:2D7E`

`8B7608 MOV SI,[BP+08]`

`1777:2D81`

`92 XCHG AX,DX`

`1777:2D82`

`33D2 XOR DX,DX`

`1777:2D84`

`85C0 TEST AX,AX`

`1777:2D86`

`7402 JZ 2D8A`

`1777:2D88`

`F7F1 DIV CX`

`1777:2D8A`

`93 XCHG AX,BX` 交换

`1777:2D8B`

`F7F1 DIV CX` 除以机器码长度

`1777:2D8D`

`92 XCHG AX,DX` 商和余数相交换

`1777:2D8E`

`87D3 XCHG DX,BX`

`1777:2D90`

`0430 ADD AL,30` 余数加30

`1777:2D92`

`3C39 CMP AL,39` 是否大于39

`1777:2D94`

`7603 JNA 2D99`

`1777:2D96`

`024604 ADD AL,[BP+04]`

`1777:2D99`

`4E DEC SI`

`1777:2D9A`

`7407 JZ 2DA3`

`1777:2D9C`

`AA STOSB` 存入前面注册码的末尾。

`1777:2D9D`

`8BC2 MOV AX,DX`

`1777:2D9F`

`0BC3 OR AX,BX`

`1777:2DA1`

`75DE JNZ 2D81` 没算完就继续

`1777:2DA3`

`8BC7 MOV AX,DI` 串指针

`1777:2DA5`

`2B460E SUB AX,[BP+0E]` 计算新加入的机器码的长度

`1777:2DA8`

`5F POP DI`

`1777:2DA9`

`5E POP SI`

`1777:2DAA`

`C9 LEAVE`

如果输入的注册名为：`123456`

那么它们的ASCII码的总和＋1＝

`13B`

1、`13B/A` 商是1余数是5，
2、`1F/A` 商是3余数是1
3、`3/A` 商是0余数是3

也就是最后几位的注册码是`513`

与上面合起来就是`MGW@EEINNK513`

最后是把刚才计算的注册码`513`重新排序，其排序很简单。

代码如下：

```
2DC1：MOV AH，[ES:BX] 最后一位
2DC4：MOV
AL，[ES:SI] 首码
2DC7：MOV [ES:SI]，AH
2DCA：MOV [ES:BX]，AL AH，AL 位置互换。
2DCD
INC SI
2DCE：DEC BX
2DCF：LOOP 2DC1 没结束就循环。
```

也就是`513`的重新排列后就是`315`。

那么真正的注册码就是`MGW@EEINNK315`

****************************************

`name:123456`

`code:MGW@EEINNK315`

另注册码位数少于13位要用"`-`"补足如`MGW@EEINNK-31`
