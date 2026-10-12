
的代码执行后弹出来的，此时不管是成功提示还是错误提示对我们来说都已太晚了。不信，你可以用`SOFT ICE`看一看`GDT`就知道我所言不虚！如果`Start=1`，系统不会提示错误而运行`mywdm.sys`。

好了，既然`GDT`中已经有了我们的调用门，那么意味着我们写的应用程序可以自由地在`RING3`和`RING0`之间切换，但如何使用这个调用门还是有讲究的，它的使用方法是：

```
...
call 3e3:00000000 ;机器码 9A 00 00 00 00 E3 03
;此时已经进入 RING0，CS=3E8，跟着要切换堆栈
mov eax,esp
mov esp,[esp+4]
push eax
;这里加入你要在 RING0 里执行的代码
;准备返回 RING3
pop esp
push offset ring3
retf
ring3:
;此时回到 RING3
...
```

为什么要切换堆栈？为什么要建立我们自己的 0 级 32 位代码段？直接使用操作系统的 0 级 32 位代码段（选择子=8）不可以吗？在这里我要说明一下：`WINDOWS 2000/XP/2003`对自己保护得非常好，就算普通应用程序能进入`RING0`，但还是在用户区（代码和数据的地址都小于`80000000H`），所以`WINDOWS 2000/XP/2003`会认为是非法进入`RING0`的，就有可能产生页故障，表现是调用子程序或访问内存时就会触发页故障，解决的方法是切换堆栈；不用操作系统的 0 级 32 位代码段（选择子为 8），用我们建立的 0 级 32 位代码段（选择子=`3E8`）。

本文例子生成的`mywdm.sys`大小是 1536 字节，如果你对`PE`文件很了解，可以手工给`mywdm.sys`减肥，但要注意的是减肥后要把正确的`CHECKSUM`值填入`PE`头的`CHECKSUM`字段中，否则操作系统是不会装入`mywdm.sys`执行的。我就把`mywdm.sys`减肥到只有 368 字节（只有 DOS 头，`PE`头+节表，`60H`字节的重定位表和代码，应该是世界上最小的驱动程序了）。完全可以把这个只有 368 字节的驱动程序包含在自己的程序中，在需要是把它还原到`SYSTEM32\DRIVERS`目录里。

计算`CHECKSUM`的方法是：把`PE`文件的所有字用`ADC`相加，然后得到的结果再和文件大小`ADC`相加，这个结果就是`CHECKSUM`值。网上也可以找到计算`CHECKSUM`的工具。

`RING3`测试程序如下：

```
.386
.model flat, stdcall
option casemap:none
include \masm32\include\windows.inc
include \masm32\include\kernel32.inc
include \masm32\include\user32.inc
includelib \masm32\lib\user32.lib
includelib \masm32\lib\kernel32.lib
.data
szExceptionCaused db Exception Caused - could not switch to ring 0,0
szError    db Error,0
MsgCaption      db Test,0
MsgBoxText      db cr0=%8x,0
tmp db 50 dup(90)
Callgt dd 0
       dw 3e3h
.code
ExceptCallBack PROC
invoke    MessageBoxA, 0, addr szExceptionCaused,addr szError, 0
invoke ExitProcess, -1
ret
ExceptCallBack ENDP
start:
push  offset ExceptCallBack
call   SetUnhandledExceptionFilter
call    fword ptr [Callgt]             ;use callgate to Ring0!
Ring0:
mov eax,esp   ;save ring0 esp
mov esp,[esp+4];->ring3 esp
push eax
mov eax,cr0
pop esp   ;restore ring0 esp
push offset Ring3
retf
Ring3:
invoke wsprintfA,addr tmp,addr MsgBoxText,eax
invoke MessageBox, NULL,addr tmp, addr MsgCaption, MB_OK
Exit:
invoke ExitProcess,NULL
end start
```

嘿嘿，大家以后就可以方便的写 2K/XP/2003 下的`RING0`病毒了。

Trackback: `http://tb.blog.csdn.net/TrackBack.aspx?PostId=337881`

`[点击此处收藏本文]`   jarodlau 发表于 2005 年 04 月 06 日 05:44:00

Trackback: `http://tb.blog.csdn.net/TrackBack.aspx?PostId=890537`
