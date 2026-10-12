
# MagicWin v1.3

> 原文：[`www.pediy.com/kssd/pediy05/pediy50291.htm`](https://www.pediy.com/kssd/pediy05/pediy50291.htm)

首先祝各位学习、工作愉快。

`MagicWin v1.3` 是一个内码转换工具。它可是一个老工具了，我现在也不用它了，在它从我的机器消失之前，我把它的注册算法分析出来。我的目的只是为了学习。

你可设中断 `BPX HMEMCPY`

```
36B7:0502 6A01                 PUSH      BYTE +01
36B7:0504 16                   PUSH      SS
36B7:0505 8D86C8FE             LEA       AX, [BP+FEC8]
36B7:0509 50                   PUSH      AX
36B7:050A 16                   PUSH      SS
36B7:050B 8D46C8               LEA       AX, [BP-38]
36B7:050E 50                   PUSH      AX
36B7:050F 16                   PUSH      SS
36B7:0510 8D46E4               LEA       AX, [BP-1C]
36B7:0513 50                   PUSH      AX
36B7:0514 9AAB030737           CALL      03AB 按 F8 进入
36B7:0519 83C40E               ADD       SP,BYTE +0E
36B7:051C 3D0100               CMP       AX,01 01 是成功标志
36B7:051F 7558                 JNZ       0579
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
:0038.03BD 9AFFFF0000             call KERNEL.LSTRLEN 计算机器码长度
:0038.03C2 50                     push ax 机器码长度入栈
:0038.03C3 66FF760A               push word ptr [bp+0A]
:0038.03C7 9AFFFF0000             call KERNEL.LSTRLEN 计算假码长度
:0038.03CC 5A                     pop dx 机器码长度出栈
:0038.03CD 3BD0                   cmp dx, ax 二者进行比较
:0038.03CF 7E25                   jle 03F6 机器码长度低于假码长度就跳
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
:0038.03F6 C45E0A                 les bx, [bp+0A]假码地址
:0038.03F9 26803F00               cmp byte ptr es:[bx], 00 比较是否为空
:0038.03FD 7503                   jne 0402 不为空就跳
:0038.03FF E9C801                 jmp 05CA
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
:0038.0412 8D867AFF               lea ax, [bp+FF7A]机器码地址
:0038.0416 50                     push ax
:0038.0417 9AFFFF0000             call KERNEL.LSTRLEN 计算机器码长度
:0038.041C 8BF8                   mov di, ax 机器码长度
:0038.041E BA0300                 mov dx, 0003 首先取机器码的第四位
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

注：机器码的首地址为`[bp+FF7A]`

```
:0038.0421 8D867DFF               lea ax, [bp+FF7D]机器码的第四位地址
:0038.0425 8BF0                   mov si, ax
:0038.0427 EB4C                   jmp 0475
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
:0038.0475 36803C00               cmp byte ptr ss:[si], 00 比较是否为空
:0038.0479 75AE                   jne 0429 不为空就跳
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

此部分代码是重新排列机器码的位置。

```
:0038.0429 8BC2                   mov ax, dx 第 DX 位
:0038.042B 050400                 add ax, 0004 与 4 相加
:0038.042E 3BC7                   cmp ax, di 是否大于机器码的长度
:0038.0430 7D1E                   jge 0450 大于等于就跳
:0038.0432 368A0C                 mov cl , ss:[si]取第 DX 位机器码
:0038.0435 8D867EFF               lea ax, [bp+FF7E]第 DX＋1 位机器码地址
:0038.0439 8BDA                   mov bx, dx 第 DX 位
:0038.043B 03D8                   add bx, ax
:0038.043D 368A07                 mov al , ss:[bx]第 DX＋1＋DX 位机器码
:0038.0440 368804                 mov ss:[si], al   
:0038.0443 8D867EFF               lea ax, [bp+FF7E]第 DX＋1 位机器码地址
:0038.0447 8BDA                   mov bx, dx
:0038.0449 03D8                   add bx, ax
:0038.044B 36880F                 mov ss:[bx], cl   
:0038.044E EB23                   jmp 0473
:0038.0450 8BC2                   mov ax, dx
:0038.0452 40                     inc ax
:0038.0453 3BC7                   cmp ax, di
:0038.0455 7D1C                   jge 0473
:0038.0457 368A0C                 mov cl , ss:[si]上次循环的串地址
:0038.045A 8D867BFF               lea ax, [bp+FF7B]机器码地址
:0038.045E 8BDA                   mov bx, dx
:0038.0460 03D8                   add bx, ax
:0038.0462 368A07                 mov al , ss:[bx]
:0038.0465 368804                 mov ss:[si], al   
:0038.0468 8D867BFF               lea ax, [bp+FF7B]
:0038.046C 8BDA                   mov bx, dx
:0038.046E 03D8                   add bx, ax
:0038.0470 36880F                 mov ss:[bx], cl   
:0038.0473 46                     inc si 加 1
:0038.0474 42                     inc dx 加 1
:0038.0475 36803C00               cmp byte ptr ss:[si], 00
:0038.0479 75AE                   jne 0429 没循环完毕就跳
:0038.047B BA0300                 mov dx, 0003 初始化机器码序号为 3
:0038.047E 3BD7                   cmp dx, di　DI 是机器码长度
:0038.0480 7D19                   jge 049B 大于等于就跳
:0038.0482 8D867AFF               lea ax, [bp+FF7A]机器码首地址
:0038.0486 8BDA                   mov bx, dx
:0038.0488 03D8                   add bx, ax　BX 也就是机器码的第 BX 位
:0038.048A 8BF3                   mov si, bx
:0038.048C 368A07                 mov al , ss:[bx]
:0038.048F 02C2                   add al , dl　DL 就是所取码的序号，AL 是第在 DX 位机器码的 ASCII 码，二者相加。
:0038.0491 0410                   add al, 10 上面的结果再加 10
:0038.0493 368804                 mov ss:[si], al 结果存入原位
:0038.0496 42                     inc dx 序号加 1
:0038.0497 3BD7                   cmp dx, di　DI 是机器码长度。
:0038.0499 7CE7                   jl 0482 低于就跳，依次循环。
:0038.049B 66FF760E               push word ptr [bp+0E]
:0038.049F 9AFFFF0000             call KERNEL.LSTRLEN
:0038.04A4 8BC8                   mov cx, ax
:0038.04A6 660FBFC1               movsx eax, ecx
:0038.04AA 668946FA               mov [bp-06], eax
:0038.04AE 33D2                   xor dx, dx
:0038.04B0 8B760E                 mov si, [bp+0E]
:0038.04B3 3BD1                   cmp dx, cx
:0038.04B5 7D15                   jge 04CC
:0038.04B7 8E4610                 mov es, [bp+10]
:0038.04BA 268A04                 mov al , es:[si]
:0038.04BD 98                     cbw
:0038.04BE 660FBFC0               movsx eax, eax
:0038.04C2 660146FA               add [bp-06], eax
:0038.04C6 46                     inc si
:0038.04C7 42                     inc dx
:0038.04C8 3BD1                   cmp dx, cx
:0038.04CA 7CEB                   jl 04B7
```

机器码的格式为：如 `MGW3762-10`(每台机器都不一样，但都是 10 位)

按上面的步骤处理如下：

1.  首先是 3 与-互换位置，也就是 `MGW-762310`
2.  取 7 与 1 互换位置，也就是 `MGW－162370`
3.  再取 6 与 0 互换位置，也就是 `MGW－102376`
4.  取 3 与 2 互换位置，也就是 `MGW－103276`
5.  取 2 与 7 互换位置，也就是 `MGW－103726`
6.  取 2 与 6 互换位置，也就是 `MGW－103762`
7.  然后`"-"`的 ASCII 码＋3＋10＝40(也就是`@`),存入原位。依次类推。最后结果：

    `MGW@EEINNK` 这是前十位注册码。

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
:0038.0473 46                     inc si 加 1
:0038.0474 42                     inc dx 加 1
:0038.0475 36803C00               cmp byte ptr ss:[si], 00 为空
:0038.0479 75AE                   jne 0429 不为空继续循环
:0038.047B BA0300                 mov dx, 0003
:0038.047E 3BD7                   cmp dx, di
:0038.0480 7D19                   jge 049B
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
:0038.049B 66FF760E               push word ptr [bp+0E]
:0038.049F 9AFFFF0000             call KERNEL.LSTRLEN 此处是取注册名长度
:0038.04A4 8BC8                   mov cx, ax 长度存入 CX
:0038.04A6 660FBFC1               movsx eax, ecx
:0038.04AA 668946FA               mov [bp-06], eax
:0038.04AE 33D2                   xor dx, dx 清零
:0038.04B0 8B760E                 mov si, [bp+0E]注册名首址
:0038.04B3 3BD1                   cmp dx, cx　CX 是注册名的长度
:0038.04B5 7D15                   jge 04CC
:0038.04B7 8E4610                 mov es, [bp+10]
:0038.04BA 268A04                 mov al , es:[si]首先取注册名的第 SI 位
:0038.04BD 98                     cbw
:0038.04BE 660FBFC0               movsx eax, eax
:0038.04C2 660146FA               add [bp-06], eax 第 SI 位注册名的 ASCII 码与其[bp-06]相加。
:0038.04C6 46                     inc si 注册名串地址加 1
:0038.04C7 42                     inc dx 序号加 1
:0038.04C8 3BD1                   cmp dx, cx 是否循环完毕
:0038.04CA 7CEB                   jl 04B7 否就继续计算。
:0038.04CC 66FF76FA               push word ptr [bp-06]计算之和入栈
:0038.04D0 1E                     push ds
:0038.04D1 684D1E                 push 1E4D
```



# 机器码处理与注册码生成

## 汇编指令解析

以下汇编代码段用于计算机器码的末尾地址并调用注册码生成函数：

```
:0038.04D4 8D867AFF       lea ax, [bp+FF7A] ; 机器码首址
:0038.04D8 8BD7           mov dx, di ; DI 是机器码的长度
:0038.04DA 03D0           add dx, ax ; 机器码末尾地址
:0038.04DC 16             push ss
:0038.04DD 52             push dx
:0038.04DE 9AFFFF0000     call USER._WSPRINTF ; 此处调用是用上面的结果计算最后几位的注册码并把其进行排序。可以按 F8 进入
:0038.04E3 83C40C         add sp, 000C
:0038.04E6 16             push ss
```

## 注册码计算逻辑

**计算最后几位的注册码并把其进行排序**

核心算法代码如下，主要逻辑包括：获取姓名之和、机器码长度，通过除法取余数并转换为字符，最后将结果存入注册码末尾。

```
1777:2D67 837E0400       CMP     WORD [BP+04], BYTE +00
1777:2D6B 7402           JZ      2D6F
1777:2D6D B007           MOV     AL, 07
1777:2D6F 884604         MOV     [BP+04], AL
1777:2D72 8B5E0A         MOV     BX, [BP+0A] ; 姓名之和
1777:2D75 8B560C         MOV     DX, [BP+0C] ; 0
1777:2D78 8B4E06         MOV     CX, [BP+06] ; 机器码长度
1777:2D7B C47E0E         LES     DI, [BP+0E]
1777:2D7E 8B7608         MOV     SI, [BP+08]
1777:2D81 92             XCHG    AX, DX
1777:2D82 33D2           XOR     DX, DX
1777:2D84 85C0           TEST    AX, AX
1777:2D86 7402           JZ      2D8A
1777:2D88 F7F1           DIV     CX
1777:2D8A 93             XCHG    AX, BX ; 交换
1777:2D8B F7F1           DIV     CX ; 除以机器码长度
1777:2D8D 92             XCHG    AX, DX ; 商和余数相交换
1777:2D8E 87D3           XCHG    DX, BX
1777:2D90 0430           ADD     AL, 30 ; 余数加 30
1777:2D92 3C39           CMP     AL, 39 ; 是否大于 39
1777:2D94 7603           JNA     2D99
1777:2D96 024604         ADD     AL, [BP+04]
1777:2D99 4E             DEC     SI
1777:2D9A 7407           JZ      2DA3
1777:2D9C AA             STOSB   ; 存入前面注册码的末尾
1777:2D9D 8BC2           MOV     AX, DX
1777:2D9F 0BC3           OR      AX, BX
1777:2DA1 75DE           JNZ     2D81 ; 没算完就继续
1777:2DA3 8BC7           MOV     AX, DI ; 串指针
1777:2DA5 2B460E         SUB     AX, [BP+0E] ; 计算新加入的机器码的长度
1777:2DA8 5F             POP     DI
1777:2DA9 5E             POP     SI
1777:2DAA C9             LEAVE
```

### 示例计算

如果输入的注册名为 `123456`：

1.  它们的 ASCII 码的总和 + 1 = `13B`
2.  计算过程：
    *   `13B` / `A`：商是 1，余数是 5
    *   `1F` / `A`：商是 3，余数是 1
    *   `3` / `A`：商是 0，余数是 3
3.  也就是最后几位的注册码是 `513`
4.  与上面合起来就是 `MGW@EEINNK513`

### 注册码排序

最后是把刚才计算的注册码 `513` 重新排序，其排序很简单。代码如下：

```
2DC1: MOV AH, [ES:BX] ; 最后一位
2DC4: MOV AL, [ES:SI] ; 首码
2DC7: MOV [ES:SI], AH
2DCA: MOV [ES:BX], AL ; AH, AL 位置互换
2DCD: INC SI
2DCE: DEC BX
2DCF: LOOP 2DC1 ; 没结束就循环
```

即 `513` 重新排列后就是 `315`。

**最终的注册码就是 `MGW@EEINNK315`**

## 结果

```
name: 123456
code: MGW@EEINNK315
```

注意：另注册码位数少于 13 位要用 `-` 补足，如 `MGW@EEINNK-31`

`4dd466d0e1e0cc123afbe504da2462ca`
