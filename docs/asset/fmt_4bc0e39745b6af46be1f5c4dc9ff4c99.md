
## 汇编代码分析与逆向逻辑

以下是反汇编代码片段，包含数据定义与控制流逻辑：

```assembly
01004AD0   . 00                     db 00                              ; 这里是是否为我们伪造消息的状态，1 为是，0 为否
01004AD1     00                     db 00
01004AD2   . 00000000               dd 00000000
01004AD6   . 00000000               dd 00000000                       ; 当前的列数
01004ADA   . 00000000               dd 00000000                       ; 当前的行数
01004ADE   > C1E3 05                shl ebx,5
01004AE1   .^ EB D4                jmp short winmine.01004AB7
01004AE3   > 803D D04A0001 00      cmp byte ptr ds:[1004AD0],0        ; 跳到这里，先判断是否为我们伪造消息状态
01004AEA   .^ 75 F2                jnz short winmine.01004ADE          ; 是就跳走，不处理下面的
01004AEC   . C605 D04A0001 01      mov byte ptr ds:[1004AD0],1        ; 标志为伪造消息状态
01004AF3   . 3E:8B5D 08            mov ebx,dword ptr ds:[ebp+8]
01004AF7   . 891D D24A0001         mov dword ptr ds:[1004AD2],ebx
01004AFD   >/ 8B35 D64A0001        mov esi,dword ptr ds:[1004AD6]      ; 取出列数
01004B03   .| 46                    inc esi                              ; ＋1
01004B04   .| 8935 D64A0001         mov dword ptr ds:[1004AD6],esi      ; 写回去
01004B0A   .| 3B35 34530001         cmp esi,dword ptr ds:[1005334]      ; 比较是否出界
01004B10   .| 7F 68                 jg short winmine.01004B7A           ; 出界代表结束了，跳走
01004B12   .| C1E6 04               shl esi,4                           ; 计算坐标的逆运算
01004B15   .| 83EE 03               sub esi,3
01004B18   >|/8B1D DA4A0001         mov ebx,dword ptr ds:[1004ADA]      ; 取出行数
01004B1E   .||43                   inc ebx                              ; ＋1
01004B1F   .||891D DA4A0001         mov dword ptr ds:[1004ADA],ebx      ; 写回去
01004B25   .||3B1D 38530001        cmp ebx,dword ptr ds:[1005338]      ; 比较是否出界
01004B2B   .||7F 41                jg short winmine.01004B6E           ; 出界就跳走
01004B2D   .||33FF                xor edi,edi                           ; edi 为参数 lParam
01004B2F   .||03FB                add edi,ebx                           ; 坐标参数的逆运算
01004B31   .||C1E7 04             shl edi,4
01004B34   .||83C7 28             add edi,28
01004B37   .||C1E7 10             shl edi,10
01004B3A   .||03FE                add edi,esi
01004B3C   .||57                  push edi                              ; /lParam
01004B3D   .||6A 01               push 1                                ; |wParam
01004B3F   .||68 01020000         push 201                              ; |msg=左键按下
01004B44   .||FF35 D24A0001       push dword ptr ds:[1004AD2]           ; |hWnd
01004B4A   .||E8 7AD0FFFF         call winmine.01001BC9                 ; \\call WndProc
01004B4F   .||C705 40510001 01000000 mov dword ptr ds:[1005140],1       ; 左键弹起标志
01004B59   .||57                  push edi                              ; /lParam
01004B5A   .||6A 00               push 0                                ; |wParam
01004B5C   .||68 02020000         push 202                              ; |msg=左键弹起
01004B61   .||FF35 D24A0001       push dword ptr ds:[1004AD2]           ; |hWnd
01004B67   .||E8 5DD0FFFF         call winmine.01001BC9                 ; \\call WndProc
01004B6C   .|\\EB AA             jmp short winmine.01004B18            ; 行数小循环
01004B6E   >| C705 DA4A0001 00000000 mov dword ptr ds:[1004ADA],0       ; 行数从 0 开始
01004B78   .\\ EB 83             jmp short winmine.01004AFD            ; 列数大循环
01004B7A   > C705 D64A0001 00000000 mov dword ptr ds:[1004AD6],0        ; 列数从 0 开始
01004B84   . C605 D04A0001 00    mov byte ptr ds:[1004AD0],0           ; 恢复伪造消息标志
01004B8B   .^ E9 D1FEFFFF        jmp winmine.01004A61                  ; 出门咯^\_^
```

### 伪造鼠标单击消息的逻辑

这里为了让计算机帮我们单击，我们伪造了鼠标单击的消息。

```assembly
01004B3C   .||57                  push edi                              ; /lParam
01004B3D   .||6A 01               push 1                                ; |wParam
01004B3F   .||68 01020000         push 201                              ; |msg=左键按下
01004B44   .||FF35 D24A0001       push dword ptr ds:[1004AD2]           ; |hWnd
01004B4A   .||E8 7AD0FFFF         call winmine.01001BC9                 ; \\call WndProc
01004B4F   .||C705 40510001 01000000 mov dword ptr ds:[1005140],1       ; 左键弹起标志
01004B59   .||57                  push edi                              ; /lParam
01004B5A   .||6A 00               push 0                                ; |wParam
01004B5C   .||68 02020000         push 202                              ; |msg=左键弹起
01004B61   .||FF35 D24A0001       push dword ptr ds:[1004AD2]           ; |hWnd
01004B67   .||E8 5DD0FFFF         call winmine.01001BC9                 ; \\call WndProc
```

然后用了两个循环，分别遍历行数和列数，做到每个格子都让鼠标点一遍。但是，我们的这个处理是在消息过程里分出来的，所以要有个标志，就是消息是否是我们发出的。如果是的话就不执行上面这段伪造代码，所以在开始有个标志位：

```assembly
01004AE3   > 803D D04A0001 00      cmp byte ptr ds:[1004AD0],0        ; 跳到这里，先判断是否为我们伪造消息状态
01004AEA   .^ 75 F2                jnz short winmine.01004ADE          ; 是就跳走，不处理下面的伪造消息
```

### 坐标的逆运算

还有一个就是坐标的逆运算。原来的运算是：

```assembly
01004A82   > 8B45 14               mov eax,dword ptr ss:[ebp+14]       ; 这里开始是抄写源程序分解坐标的方法
01004A85   . C1E8 10               shr eax,10                          ; 取高位
01004A88   . 83E8 27               sub eax,27                          ; －39（边界）
01004A8B   . C1F8 04               sar eax,4                           ; ÷16
01004A8E   . 50                    push eax                            ; 结果为行
01004A8F   . 0FB745 14             movzx eax,word ptr ss:[ebp+14]    ; 低位
01004A93   . 83C0 04               add eax,4                           ; ＋4
01004A96   . C1F8 04               sar eax,4                           ; ÷16
01004A99   . 50                    push eax                            ; 列
```

高位－39，再除以 16 结果为行；低位＋4 再除以 16，结果为列。所以我们可以逆运算一下：

- 行数×16，再加上 40（为的是将鼠标点在格子里，所以要小于 39，小一点就可以了，否则点不到的）
- 列数×16，再减取 3（道理和上面一样）

然后再组装起来，行数放在高位（前两个字节），列数放在低位（后两个字节）：

```text
00 00   00 00
行数     列数
```

最后保存一下就可以了。在代码上点右键，复制到可执行文件－》全部修正，就行了。

### 整理添加的代码

整理一下自己添上去的代码：

```assembly
01004A56   > \\8B55 0C             mov edx,dword ptr ss:[ebp+C]
01004A59   . 8B4D 14               mov ecx,dword ptr ss:[ebp+14]       ; 这两行重写一遍也无所谓，跳到下一行也没关系
01004A5C   . 53                    push ebx
01004A5D   . 56                    push esi
01004A5E   . 57                    push edi                              ; 这三行保存寄存器
01004A5F   . EB 07                 jmp short winmine.01004A68            ; 跳到我们的作弊代码
01004A61   > 33DB                  xor ebx,ebx                           ; 这里是出口，跳回去，清空 ebx（原来的代码）
01004A63   .^ E9 72D1FFFF          jmp winmine.01001BDA                  ; 跳回去
01004A68   > 81FA 01020000         cmp edx,201                             ; 左键按下？
```
