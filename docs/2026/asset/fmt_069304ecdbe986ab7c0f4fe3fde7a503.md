
## 反汇编片段

```
:1000542C 0F8EDCF7FFFF            jle 10004C0E
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10004C02(C)
|
:10005432 8D4DD4
lea ecx, dword ptr [ebp-2C]
:10005435 C645FC01                mov
[ebp-04], 01
:10005439 E872D4FFFF              call 100028B0

:1000543E 8D4DE4
lea ecx, dword ptr [ebp-1C]
:10005441 C645FC00                mov
[ebp-04], 00
:10005445 E8C3580500              call 1005AD0D

:1000544A 8D4DF0
lea ecx, dword ptr [ebp-10]
:1000544D C745FCFFFFFFFF          mov [ebp-04], FFFFFFFF

:10005454 E8B4580500              call 1005AD0D

:10005459 B001
  mov al, 01
:1000545B 8B4DF4
mov ecx, dword ptr [ebp-0C]
:1000545E 64890D00000000          mov dword ptr fs:[00000000],
ecx
:10005465 5F
  pop edi
:10005466 5E
  pop esi
:10005467 5B
  pop ebx
:10005468 8BE5
  mov esp, ebp
:1000546A 5D
  pop ebp
:1000546B C3
  ret
```

## 破解思路记录

难怪RU，叫我遇到不常破解一定要分析流程。运行的是，这里都有易懂html命令。*^_^*

最后总结几种破解思路:

### 1. 掷石问路法
`This page is not extracted in demo version.`

直接找这此内容，不过win32asm在搜索只能看前顾后。

`This or This page[前]`
`demo version[后]`

找到之后要向上分析。

### 2. 功能分析法
它功能是`pdf to html`
所以着重放在`html`中！
在`win32asm`字串窗找到:

**方法一：**

`<!-- text starts here -->`
`-->`此处到正常状态!
`:1000B304 682CD50710              push 1007D52C`

`"<!-- text starts here -->"`--->关键就是这里了!
`:10004F82 6804CC0710              push 1007CC04`-->从程序运行角度，就要在这里开始向上分析

`:10005190 6804CC0710              push 1007CC04`

**方法二：**

```
<HTML>
<HEAD><TITLE>Blank Page</TITLE></HEAD>
"
```
`:10005875 682CCF0710              push 1007CF2C`--同理，向上看

`:10005926 682CCF0710              push 1007CF2C`

### 3. 逻辑分析法
从逻辑推角度，它应该在处理进程就有一致命判断。

处理`[Processing]`

String Resource ID=`00049`: `"Processing Page %d of %d..."`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
跳到好几处，不是好方法。再找一个比较好的:
`"Page %d of %d - Magellan"`
`^^^^^^^^^^---->`咦！标志这么明白？不管了有前无后打死就算!:)

* Possible StringData Ref from Data Obj ->`"Page %d of %d - Magellan"`

```
|
:10004CB0 6824CD0710              push 1007CD24
:10004CB5 50
  push eax
```
只有一个好地方!
上面就是`Processing Page %d of %d...`下面不久就是`This page is not extracted in demo version.`

最好关键是::`10004CE0 0F84A9050000            je 1000528F`

是否跳？而且这值，就要看:`10004CDD D95DBC`
`    fstp dword ptr [ebp-44]`
这个值又取决于::`10004CDB 3BC1`
`    cmp eax, ecx`
所以试一下把修改为:`cmp eax,eax`
果然成功。

最后方法：就是关羽单枪负会，直接了当！
很久没有写教程，打手软！谁掷东西？什么来的？不好了，嗅鸡蛋！当场嗅晕！!@#$% *\_*/

师傅绝招，闪啊！^_\^

2002-2-18 晚上11:30分钟
