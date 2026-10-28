
```asm
mov y3,-32

mov y4,-32

ret

\_choose endp

;;----------------------------------------\_brush----------------------------------

\_brush proc p1

invoke GetDC,hwnd

mov hdc,eax

invoke SelectObject,hdc,p1

mov eax,x1

add eax,equwallsize

mov ebx,y1

add ebx,equwallsize

invoke Rectangle,hdc,x1,y1,eax,ebx

mov eax,x2

add eax,equwallsize

mov ebx,y2

add ebx,equwallsize

invoke Rectangle,hdc,x2,y2,eax,ebx

mov eax,x3

add eax,equwallsize

mov ebx,y3

add ebx,equwallsize

invoke Rectangle,hdc,x3,y3,eax,ebx

mov eax,x4

add eax,equwallsize

mov ebx,y4

add ebx,equwallsize

invoke Rectangle,hdc,x4,y4,eax,ebx

invoke ReleaseDC,hwnd,hdc

ret

\_brush endp

;;;;;------------------------------\_check-----------------------------------

\_check proc p1,p2,p3

mov eax,p2

add eax,128

shr eax,5

mov xline,eax

shl eax,2

lea ebx,xx

add ebx,eax

mov eax,p1

add eax,32

shr eax,5

cmp p3,0

jnz nocheck

bt dword ptr [ebx],eax

jc @f

mov eax,0

ret

@@:

mov eax,1

ret

nocheck:

btr dword ptr [ebx],eax

cmp dword ptr [ebx],0

jnz l1

inc number0

mov eax,p2

cmp eax,base1

js l1

mov base1,eax

l1:

ret

\_check endp

end start
```

（看到主题我连名字都打出来了，想必我是个爱炫耀的人。这，确实是事实。我心还想您在心里轻轻顶下我。哈哈）

高手一般有源码就够了，如果你是的话，下面的分析，不妨先不看。

先观察下游戏，也就是下面几个步骤：

1. 方块随机出现
2. 方块下滑，期间人顺时针转动；
3. 方块下滑到不到下滑了，也就停了
4. 到第 1 个步骤了

下面到开始写程序了：

### 第一：设计界面

要作图的窗口，最好的就是风格`WS_PUPOPWINDOW`的窗口了。其它的也行。不过这个程序貌似没多实际价值。（有的话，我肯定不在多话了。早卖钱了，不过对拿来练习实在是个好程序），所以有些地方不是很完美，望见谅。

代码就是这个了：

这个实在说不多少；

### 第二：方块随机出

也就小段`_init`和小段`_choose`

在纸上，画画那些方块也就出了这些代码。

至于哪个取随机数那几句代码：

```asm
invoke GetTickCount

mov ebx,5

xor edx ,edx

div ebx
```

。是常用的，最好记熟。

```asm
pushf

mov eax,2046

mov ecx,20

lea edi,xx

cld

rep stosd

mov dword ptr [edi],0

popf
```

上面这段要说说：其中数列`XX`记录的是方块的位置

`2046` 也就`111，111,111,10（2）`

如此一来，我们玩游戏时能就看见的一行10个方格也就对应中间的十个‘1’了

哪个还有第一位的’0‘是什么意思呢？

其实第11位也是’0‘的。这样一来，一个数也就记录了12个位置，也就围墙也占一位

当然也你也可以有10个数来记录。这要有方格下滑和变化时稍稍改下就是了

回到我的程序，这样做，我就可以忽略边界了

’1‘表示没方格，’0‘，表示有方格，当一行满方块时，对应的那个数也就0了，这样好判断

说实在我就看好亮点才用这数列。

### 第三：到这，方块应该随机出来了

还有问题的话，就回头一步步检查，说实在我的程序都是修改出的。

在方块出来后，在我的程序里也就有手动控制就先处理手动控制，再自动下滑：向左，向右，向下，写一二次，应该可以写出。就是按UP键时有些麻烦。

首先看段数据：

```asm
up1 dd 0,0,0,0,-32,0,32,64
     dd -32,0,32,64,0,0,0,0
     dd 0,0,0,0,-32,0,32,64
     dd -32,0,32,64,0,0,0,0
up2 dd 0,-32,0,32,-32,0,0,0
     dd 32,0,0,0,0,-32,0,32
     dd 0,32,0,-32,32,0,0,0
     dd -32,0,0,0,0,32,0,-32
up3 dd -32,-32,-32,0,-64,-32,0,0
     dd 32,0,-32,-32,-32,-32,-32,0
     dd 0,0,0,-32,32,0,-32,-32
     dd -64,-32,0,0,0,0,0,-32
up4 dd 0,0,0,-32,-64,-32,0,0
     dd 32,0,-32,-32,0,0,0,-32
     dd -32,-32,-32,0,32,0,-32,-32
     dd -64,-32,0,0,-32,-32,-32,0
```

这是除田字方块外的四种方块的四种状态数据，是对应方块中心点（`BASEX`,`BASEY`）来说，相加就可以了。

（这个，我也办法，要统一来循环的话，也分来分去，实在是惭愧，哪位牛人能统一的话，欢迎向我’炫耀’）

每一行对应是`X1`,`X2`,`X3`,`X4`,`Y1`,`Y2`,`Y3`,`Y4`

写代码时要注意下基址

### 第四：到方块落定了

方法`_check`有两个功能：一是检查位置有无方块，二是设置位置有广场

由参数 `P3` 决定；

最后也就一行满了，一面就向一下一行，数据`XX`用`REP MOVSD`就行了，接着描绘方块，又到新方块随机出来 完了

哪个高人低人，只要是有想法的，可以发个邮件给我：522165473@qq.com

（最好是好的源代码，当然是问题也可以）。

下面就程序了

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [w.rar](javascript:void(0);)

  （1.94kb，68次下载）
