
# Asprotect2.X 辅助修复 stolen code 的一个小脚本

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-548.htm`](https://www.pediy.com/kssd/pediy08/pediy8-548.htm)

引用`shoooo`的一句话:"找到了这些点，写脚本也好，写代码恢复也好，修复就不难了.",既然`wak`给我们找出来了那些点了,那我就做点容易的事,写个脚本吧!

初学写脚本,功力有限啊,好些命令是边看帮助边写的,大家湊合着看吧,抛块砖头，希望能引出玉来。

## 准备工作:

### 一、找几个点，设断:

**A：跳转类型断点（记为`he1)`：这个点判断`wak`所讲的**

- `0`：`call`变形
- `1`：`jmp`变形
- `2`:`jXX`变形
- `3`：`cmp x,y jxx z`变形

**B：跳转地址断点(记为`he2)：`**

- 这个点判断执行`CALL XXXXXX`后，跳往的地址，按`wak`的方法是不同的类型有不同的断点，我找了一个共同跳往的点`jmp [esp-4]`,这个点四种类型最后都到这里，其中`[esp-4]`是跳去的地址。

### 二、运行`Volx`脚本，到达`stloen OEP`之后，申请一块内存或找空白的地方写一`PATCH`,开始扫描代码段中`Call xxxxxx`的地址。

```
00EB0008    BA 0000E200     MOV EDX,0E20000                     //stolen 代码段开始地址           
00EB000D    803A E8         CMP BYTE PTR DS:[EDX],0E8
00EB0010    75 14           JNZ SHORT 00EB0026
00EB0012    8B42 01         MOV EAX,DWORD PTR DS:[EDX+1]
00EB0015    03C2            ADD EAX,EDX
00EB0017    05 05000000     ADD EAX,5
00EB001C    3D 0000E900     CMP EAX,0E90000                     //判断是否是 call xxxxxx 变形
00EB0021    75 03           JNZ SHORT 00EB0026
00EB0023  - EB FE           JMP SHORT 00EB0023                  //是则中断，设为普通断点:cc1
00EB0025    90              NOP
00EB0026    42              INC EDX                             //跳回执行，设为:return
00EB0027    81FA 0050E200   CMP EDX,0E25000
00EB002D  ^ 72 DE          JB SHORT 00EB000D
00EB002F  - EB FE          JMP SHORT 00EB002F                  //结束扫描，设为普通断点:cc3
00 00 00 00 00 00 00 00 BA 00 00 E2 00 80 3A E8 75 14 8B 42 01 03 C2 05 05 00 00 00 3D 00 00 E9
00 75 03 EB FE 90 42 81 FA 00 50 E2 00 72 DE EB FE 90
```

因为前两种类型的容易修复，就直接修复了，后两种类型的需要手动修复，所以在`00eb0100`处保存一下后两种类型跳转的地址.

### 三、在`00EB0008`处新建`EIP`,然后运行脚本：

```
var jmpcall
var temp
var temp1
var jmpadds
var eaxsave
//需要按实际修改的变量
var count
var cc1
var cc2
var he1
var he2
var return

mov count,0eb0100              //存放需要手动修复代码地址
mov cc1,00eb0023               //普通断点:cc1
mov cc2,00eb002f               //普通断点:cc2
mov he1,00c488da               //跳转类型断点（记为 he1)
mov he2,0ea00ba                //跳转地址断点(记为 he2)
mov return,00eb0026            //返回 PATCH 继续扫描的地址，记为 return

start:
run

l1:
cmp eip,cc1                    //是否是 CALL  XXXXXXX 变形代码地址,是则进入执行
jne l2
mov jmpcall,edx
mov eip,jmpcall
log jmpcall
jmp start

l2:
cmp eip,he1                    //判断四种跳转类型的地址
jne r3
mov eaxsave,eax
jmp start

l3:
cmp eip,he2                    //JMP [ESP-4]跳转地址
jne l5
mov jmpadds,[esp-4]
cmp eaxsave,0                  //比较四种代码变形类型
je r0
cmp eaxsave,1
je r1
cmp eaxsave,2
je r2
cmp eaxsave,3
je r3

l4:
mov edx,jmpcall
mov eip,return                 //跳回继续执行扫描
jmp start

r0:
mov [jmpcall],0E8
mov temp,jmpadds
sub temp,jmpcall
sub temp,5
mov temp1,jmpcall
inc temp1
mov [temp1],temp
eval "类型 0:{jmpadds}"
cmt jmpcall,$RESULT
jmp l4

r1:
mov [jmpcall],0E9
mov temp,jmpadds
sub temp,jmpcall
sub temp,5
mov temp1,jmpcall
inc temp1
mov [temp1],temp
eval "类型 1:{jmpadds}"
cmt jmpcall,$RESULT
jmp l4

r2:
mov [count],jmpcall
add count,4
eval "类型 2:{jmpadds}"
cmt jmpcall,$RESULT
jmp l4

r3:
mov [count],jmpcall
add count,4
eval "类型 3:{jmpadds}"
cmt jmpcall,$RESULT
jmp l4

l5:
cmp eip,cc2                    //是否结束?
jne error
msg "代码修复完成"
ret

error:
log count
log jmpcall
log temp
log jmpadds
log eaxsave
msg "错误中止!"
ret
```

运行脚本后，从`eb0100`处存放着类型 3 和类型 4 的跳转，可以写代码进行二次修复,我就不再献丑了。

菜鸟文章，高手略过。

`thanks`:

- `shoooo`
- `syscom`
- `wak`
- `Volx`
- .........

升级了一下,`Ver:1.02`

这回不用`PATCH`了,直接修改脚本里的几个值直接运行就行了![](img/77a5f7cfd3ec840c98be3aecc80df838.jpg)

附件下载：`asprotect2.x`修复代码 1.03

`4dd466d0e1e0cc123afbe504da2462ca`
