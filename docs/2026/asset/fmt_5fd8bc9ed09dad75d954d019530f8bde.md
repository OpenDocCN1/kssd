
```
.text:00010700 LDR R0, [R11,#var_10] ; load into r0 the file descriptor

.text:00010704 BL fgetc

.text:00010708 STR R0, [R11,#var_18  ; save r0 into the local variable var_18

after we have the function feof

.text:0001070C LDR R0, [R11,#var_10] ; load into r0 the file descriptor

.text:00010710 BL feof

.text:00010714 MOV R3, R0            ; mov the reterun value into r3

.text:00010718 CMP R3, #0            ; compare r3 with 0

.text:0001071C BEQ loc_10750         ; associated with the stream is not set (r3=0) branch to loc_10750
```

**如果 `r3` 等于 0**，则来到 `loc_10750`：

```
.text:00010750 loc_10750 ; CODE XREF: ptrace_capt+D0#j

.text:00010750 SUB R3, R11, #-var_1C  ; r3 = address of var_1C

.text:00010754 LDR R0, [R11,#var_18]  ; r0 ← *(r11+var_18)

.text:00010758 LDR R1, [R11,#var_8]   ; r1 ← *(r11+var_8)

.text:0001075C MOV R2, R3             ; r2 = r3

.text:00010760 BL sub0
```

`var_18` 是读取的字符，`var_8` 是循环索引，`sub0` 的调用则为 `sub0(var_18, var_8, &var_1C);`。

看 `sub0` 函数：

![](https://bbs.kanxue.com/upload/attach/201708/663974_chq5t7823o4tgsx.png)

C 代码：

```
if(var_C==0 || var_C==2)

{

  //loc_1060C

  *var_10=var_8|0x55;

}

else

{

  //loc_10620

  *var_10=var_8^0x69 | var_8<<3;

}
```

`sub0` 返回时继续执行，`var_1C` 保存返回值：

```
.text:00010764 LDR R3, [R11,#var_1C]

.text:00010768 LDR R2, [R11,#var_C]

.text:0001076C ADD R3, R2, R3

.text:00010770 STR R3, [R11,#var_C]

.text:00010774 LDR R3, [R11,#var_8]

.text:00010778 ADD R3, R3, #1

.text:0001077C STR R3, [R11,#var_8]

.text:00010780 B loc_10700
```

这段用伪码表示即：

```
var_C = var_1C + var_C;

var_8++; //increment the index
```

**如果 `r3` 不等于 0**，则来到：

```
.text:00010720 LDR R3, [R11,#var_C]

.text:00010724 LDR R2, =0x997

.text:00010728 CMP R3, R2

.text:0001072C BNE loc_10740

.text:00010730 LDR R3, =flag

.text:00010734 MOV R2, #1

.text:00010738 STR R2, [R3]

.text:0001073C B loc_10784

.text:00010740 loc_10740 ; CODE XREF: ptrace_capt+E0#j

.text:00010740 LDR R3, =flag

.text:00010744 MOV R2, #2

.text:00010748 STR R2, [R3]

.text:0001074C B loc_10784
```

C 代码表示：

```
if (var_C==0x997)

{

  flag=1;

}

else

{

  //loc_10740

  flag=2;

}
```

终于找到了 `flag` 赋值的地方，而我们需要其值为 1。

新建 `password.raw` 文件，写入：

```
# vim password.raw

bbbbb
```

我用 `vim` 的设置删掉了换行：

```
:set noendofline binary
```

运行：

![](https://bbs.kanxue.com/upload/attach/201708/663974_sa9p08johywgmtg.png)

现在用 `gdb` 运行去掉反调试：`# gdb ./3b`

![](https://bbs.kanxue.com/upload/attach/201708/663974_0no02pnz1sohevk.png)

在 `0x10678` 处下断，修改 `r3` 的值：

![](https://bbs.kanxue.com/upload/attach/201708/663974_riwlw7mvcz715um.png)

继续往下分析，我现在要使 `var_C=0x997`，`flag` 才能赋值为 1。现在文件里的内容是：

![](https://bbs.kanxue.com/upload/attach/201708/663974_yp2x56x9y10475m.png)

要修改第五个字节使得 `var_C=0x997`，就要知道第4次循环时 `var_C` 的值。

在 `0x10774` 下断点，

![](https://bbs.kanxue.com/upload/attach/201708/663974_u9r4n50jkp4wqu7.png)

此时循环索引为 3（第4次），`var_C` 的值为 `0x724`，现在要改掉第 5 个字节的值，我用了 Python 来计算：

```
num = 0x997-0x724

for c in range (0x20,0x7f):

  ref = c^0x69 | (c<<3)

  if (ref==num):

    print "The number is " + hex(c)

print "End!"
```

运行：

```
# python antDgbAlgho.py

The number is 0x4a

End!
```

得到了第五个字节的值，修改它：

```
# vim password.raw

bbbbJ
```

记得删掉换行 `:set noendofline binary`。

运行：

![](https://bbs.kanxue.com/upload/attach/201708/663974_80v4fd2yala48lq.png)

“Good” 就被打印了。

原文链接：[b21K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6I4N6h3g2I4N6h3g2J5L8#2)9J5k6h3!0J5k6#2)9J5c8U0t1H3x3e0N6Q4x3V1j5H3y4#2)9J5c8X3q4J5L8g2)9J5k6r3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9J5k6r3W2G2N6q4)9J5k6r3g2H3K9i4y4G2k6r3g2Q4x3X3b7I4i4K6u0r3](https://bbs.kanxue.com/elink@43aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6I4N6h3g2I4N6h3g2J5L8#2)9J5k6h3!0J5k6#2)9J5c8U0t1H3x3e0N6Q4x3V1j5H3y4#2)9J5c8X3q4J5L8g2)9J5k6r3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9J5k6r3W2G2N6q4)9J5k6r3g2H3K9i4y4G2k6r3g2Q4x3X3b7I4i4K6u0r3)

本文由 看雪翻译小组 kiyaa 编译

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
