
`printf` `proto` `c:VARARG`

.data

`msg1` `db` `"%d",0`

`msg2` `db` `0ah,0dh,0`

.code

`start:`  

  `call` `main`  
  `ret`  

`main` `proc`  

  `local` `i:dword` ;定义了一个循环变量，就是数组的下标  
  `local` `a[10]:byte` ;定义了一个 10 个字节的数组  
  `local` `b[10]:dword` ;定义了一个 10 个双字的数组  
  `local` `p` `:byte` ;定义了一个字节变量，这里可以把它看成是字节指针  
  `local` `p1` `:dword` ;定义了一个双字变量，这里可以把它看成是双字指针  
  `mov` `i,0`  
`again:`   
  `cmp` `i,9`  
  `mov` `ebx,i`  
  `mov` `eax,i`  
  `ja` `out_again`  
  `mov` `byte ptr a[ebx],al` ;这句就是用下标访问数组了，和下面的比较下，想想为什么  
  `inc` `i`  
  `jmp` `again` ;这一段代码主要是往 a 数组里面依次存入 0，1，2，。。。9  
`out_again:`  
  `mov` `i,0`  
`again1:`  
  `cmp` `i,9`  
  `mov` `eax,i`  
  `mov` `ebx,i`  
  `ja` `print_arry`  
  `mov` `b[ebx*4],eax` ;根据下标 i 访问数组，和上面那句比较下看看  
  `inc` `i`  
  `jmp` `again1`  
`print_arry:`  
  `mov` `i,0`  
`loop_print_arry:`  
  `cmp` `i,9`  
  `ja` `out_print_arry`  
  `mov` `ebx,i`  
  `mov` `al,byte ptr a[ebx]`  
  `movzx` `eax,al`  
  `push` `eax`  
  `push` `offset msg1`  
  `call` `printf`  
  `add` `esp,8`  
  `inc` `i`  
  `jmp` `loop_print_arry` ;循环打印字节数组  
`out_print_arry:`  
  `push` `offset msg2`  
  `call` `printf`  
  `add` `esp,4` ;这里打印回车换行  
  `mov` `i,0`  
`loop_print_arry1:`  
  `cmp` `i,9`  
  `mov` `ebx,i`  
  `ja` `out_this_program`  
  `mov` `eax,dword ptr b[ebx*4]`  
  `push` `eax`  
  `push` `offset msg1`  
  `call` `printf`  
  `add` `esp,8`  
  `inc` `i`  
  `jmp` `loop_print_arry1` ;循环打印双字节数组  
`out_this_program:`  
  `push` `offset msg2`  
  `call` `printf`  
  `add` `esp,4` ;这里打印回车换行  
  `ret`  
  `main` `endp`  

`end` `start`  

呵呵，你可以和前面的代码比较看看，如果你看不出第一个程序的问题，最好的是你把第一个代码编译运行，自己调试着看看，保你收获多多。

下面看看用指针试试：看例子 `ch6-2`：

### 例 ch6-2：用指针输出数组的全部元素

代码如下：

```
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
.data
msg1 db "%d",0
msg2 db 0ah,0dh,0
.code
start:
 call main
 ret
main proc
 local i:dword ;定义了一个循环变量，就是数组的下标
 local a[10]:byte ;定义了一个 10 个字节的数组
 local b[10]:dword ;定义了一个 10 个双字的数组
 local p :dword ;定义了一个双字变量，这里可以把它看成是字节指针,至于怎么会这样，请看下面代码
 local p1 :dword ;定义了一个双字变量，这里可以把它看成是双字指针
 mov i,0
again:
 cmp i,9
 mov ebx,i
 mov eax,i
 ja out_again
 mov byte ptr a[ebx],al ;这句就是用下标访问数组了，和下面的比较下，想想为什么
 inc i
 jmp again ;这一段代码主要是往 a 数组里面依次存入 0，1，2，。。。9
out_again:
 mov i,0
again1:
 cmp i,9
 mov eax,i
 mov ebx,i
 ja print_arry
 mov b[ebx*4],eax ;根据下标 i 访问数组，和上面那句比较下看看
 inc i
 jmp again1
print_arry:
 lea eax,a
 mov p,eax
 add eax,10
loop_print_arry:
 cmp p,eax
 ja out_print_arry
 mov bl,byte ptr p ;这里就是把双字节指针当成字节用
 movzx ebx,bl
 push ebx
 push offset msg1
 call printf
 add esp,8
 inc p
 jmp loop_print_arry ;循环打印字节数组
out_print_arry:
 push offset msg2
 call printf
 add esp,4 ;这里打印回车换行
 lea eax,b
 mov p1,eax
 add eax,40
loop_print_arry1:
 cmp p1,eax
 ja out_this_program
 mov ebx,dword ptr p
 push ebx
 push offset msg1
 call printf
 add esp,8
 add p,4
 jmp loop_print_arry1 ;循环打印双字节数组
out_this_program:
 push offset msg2
 call printf
 add esp,4 ;这里打印回车换行
 ret
 main endp
end start
```

但是我们得到了很奇怪的结果，没办法啦，只有调试咯，经调试发现 `mov bl,byte ptr p` 这句访问数组元素根本访问不到，因为 `ch6_2!main+0x3f`:

```
00401045 8d45f2 lea eax,[ebp-0Eh]
0:000>
eax=0013ffae ebx=0000000a ecx=0013ffb0 edx=7c90e514 esi=00000000 edi=00000000
eip=00401048 esp=0013ff7c ebp=0013ffbc iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000202
ch6_2!main+0x42:
00401048 8945c4 mov dword ptr [ebp-3Ch],eax ss:0023:0013ff80=852c3fe0
0:000>
eax=0013ffae ebx=0000000a ecx=0013ffb0 edx=7c90e514 esi=00000000 edi=00000000
eip=0040104b esp=0013ff7c ebp=0013ffbc iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000202
```

因为 `a` 的首地址是 `0013ffae`，当执行这句 `mov bl,byte ptr p` 时，你可以看到调试器是这样的：

```
00401048 8945c4 mov dword ptr [ebp-3Ch],eax ss:0023:0013ff80=852c3fe0
```

哈哈，它访问了 `0013ff80`里面的元素了，唉，怎么办呢？

我想只能用第一个那种寄存器间接寻址方式了，呵呵，又回到第一种访问方式了。

发现第二个错误就是这个程序调用了 `printf` 之后，就改变了 `eax`，`ecx`，`ebx` 的值，反正就是我们这里用到的寄存器都被它改了。然后修改后的程序如下：

```
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
.data
msg1 db "%d",0
msg2 db 0ah,0dh,0
.code
start:
 call main
 ret
main proc
 local i:dword ;定义了一个循环变量，就是数组的下标
 local a[10]:byte ;定义了一个 10 个字节的数组
 local b[10]:dword ;定义了一个 10 个双字的数组
 local p :dword ;定义了一个双字变量，这里可以把它看成是字节指针,至于怎么会这样，请看下面代码
 local p1 :dword ;定义了一个双字变量，这里可以把它看成是双字指针
 mov i,0
again:
 cmp i,9
 mov ebx,i
 mov eax,i
 ja out_again
 mov byte ptr a[ebx],al ;这句就是用下标访问数组了，和下面的比较下，想想为什么
 inc i
 jmp again ;这一段代码主要是往 a 数组里面依次存入 0，1，2，。。。9
out_again:
 mov i,0
again1:
 cmp i,9
 mov eax,i
 mov ebx,i
 ja print_arry
 mov b[ebx*4],eax ;根据下标 i 访问数组，和上面那句比较下看看
 inc i
 jmp again1
print_arry:
 lea eax,a
 mov p,eax
 add eax,10
loop_print_arry:
 push eax
 cmp p,eax
 jae out_print_arry
 mov ecx,p
 mov bl,byte ptr [ecx] ;这里就是把双字节指针当成字节用
 movzx ebx,bl
 push ebx
 push offset msg1
 call printf
 add esp,8
 inc p
 pop eax
 jmp loop_print_arry ;循环打印字节数组
out_print_arry:
 push offset msg2
 call printf
 add esp,4 ;这里打印回车换行
 lea eax,b
 mov p1,eax
 add eax,40
loop_print_arry1:
 push eax ;由于下面的 ptintf 会改变我们这里要用到的寄存器，我用他们的时候就先保存
 cmp p1,eax
 jae out_this_program
 mov ecx,p1
 mov ebx,dword ptr [ecx]
 push ebx
 push offset msg1
 call printf
 add esp,8
 add p1,4
 pop eax ;弹出保存的 eax 值
 jmp loop_print_arry1 ;循环打印双字节数组
out_this_program:
 push offset msg2
 call printf
 add esp,4 ;这里打印回车换行
 ret
 main endp
end start
```

下面是运行结果：

```
C:\w\b\ch6\ch6-2>ch6-2
0123456789
0123456789
```

呵呵，终于是对的了。

下面再来看看多重数组：

先看个最简单的例子 `ch6-3`：输出一个二维数组，例如：

然后运行完程序在屏幕上打印如下：

然后写好的程序如下：

```
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
.data
a dd 1,2,3
 dd 4,5,6 ;定义了一个两行三列的整数数组
msg1 db "array a :",0ah,0dh,0
msg2 db "%8d",0
msg3 db 0ah,0dh,0
.code
start:
 call main
 ret
main proc
 local i:dword
 local j:dword ;定义了两个循环变量
 push offset msg1
 call printf
 add esp,4 ;打印消息
 mov i,0
 mov j,0 ;初始化两个循环变量
loop_1:
 cmp i,1
 ja out_loop_1
loop_2:
 cmp j,2
 ja out_loop_2
 mov eax,3 ;首先这个数组每行有三个元素，所以要用当前的行数乘以 3
 mul i
 add eax,j ;然后再加上当前的列数，由于这个数组数四字节的，所以还要
 push a[eax*4] ;乘以 4，你才能正确的访问每个数组元素
 push offset msg2
 call printf
 add esp,8 ;内循环就是依次打印每行的数组元素
 inc j
 jmp loop_2
out_loop_2:
 inc i
 push offset msg3
 call printf
 add esp,4
 jmp loop_1 ;这里是跳转到外循环，外循环是控制打印多少行
out_loop_1:
 ret
 main endp
end start
```

但是结果是这样的：

```
C:\w\b\ch6\ch6-3>ch6-3
array a :
       1       2       3
```

很明显是错的哦，你能在上面的那段程序里看出哪里错了么？呵呵，我反正没看出什么名堂来，经过调试之后发现，再进入第二次循环之前忘记把循环变量赋初值了！！！那我就在这里改了下：

`ja out_loop_1`

`mov j,0`

再调试运行，结果如下：

```
C:\w\b\ch6\ch6-3>ch6-3
array a :
       1       2       3
       4       5       6
```

呵呵，是正确的，最后正确的代码如下：

```
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
.data
a dd 1,2,3
 dd 4,5,6 ;定义了一个两行三列的整数数组
```
