
```
0040D039    813424 C4147002  xor dword ptr ss:[esp],27014C4
0040D040    C3               retn
```

随便算出一个来看看，确认一下。好了，现在是写一些东西来修复它。

有3种选择：1.在空白处写一段程序修复；2.脚本修复；3.用Luo写的`ollydbg` machine来修复。

我用了第三种，新东西嘛，总是要多尝试一下，程序如下：

```
mov reg01,0x40635c
lp:
invoke ReadMemLong,reg01,4---先取出要跳转的指针
;invoke PrintNum,reg00,16----这些是我最初拿来验证数据是否正确的
mov reg02,reg00--------------存一下，下面还要用，寄存器多就是好
inc reg00--------------------上面的代码中可以看出来，xor的前一个数在被指向地址+1处
invoke ReadMemLong,reg00,4---读取
;invoke PrintNum,reg00,16
mov reg10,reg00
add reg02,8------------------xor后一个数在+8处
invoke ReadMemLong,reg02,4---读取
xor reg00,reg10--------------xor，我们要的api地址
;invoke PrintNum,reg00,16
invoke WriteMemLong,reg01,reg00,4---写入到IAT中
add reg01,4
cmp reg01,0x4063f4-----------懒得算到底有多长，直接用地址一比较算了
jb lp
```

还有第二部分的无效函数和这个一样：

```
mov reg01,0x406414
lp:
invoke ReadMemLong,reg01,4
;invoke PrintNum,reg00,16
mov reg02,reg00
inc reg00
invoke ReadMemLong,reg00,4
;invoke PrintNum,reg00,16
mov reg10,reg00
add reg02,8
invoke ReadMemLong,reg02,4
xor reg00,reg10
;invoke PrintNum,reg00,16
invoke WriteMemLong,reg01,reg00,4
add reg01,4
cmp reg01,0x406504
jb lp
```

经过这样以后，只有一个没有被修复（其实不能说没有被修复，只是他修复以后还是指在自己的程序中，跟踪一下就好了，是`MessageBoxA`）。

这样就全部完了。

谢谢看完。
