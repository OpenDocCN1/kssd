
# 《送别》直接 IO 版

> 原文：[`www.pediy.com/kssd/pediy10/89660.html`](https://www.pediy.com/kssd/pediy10/89660.html)

很早以前就想写个这样的东西,不过自己是音痴,不会读乐谱.![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")  

上次在看雪上看到 玩命大牛写的用 `BEEP` 做的《送别》之后,摘抄了曲子,自己实现了直接 IO 版本.![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")  

## 硬件原理详解

有几个小细节需要解释:  

- 驱动喇叭的端口:`0x61`,`0x40`-`0x43`.  
- `0x40`-`0x42` 是 3 个计数器,`0x40` 控制系统时钟,`0x41` 控制 `DRAM` 刷新的,`0x42` 就是留给一般应用,比如扬声器,我们要用到的就是 `0x42`.  
- `0x43` 是这 3 个计数器的控制端口,下面详细解释.  
- 这些端口都是 8 位操作的,所以每次读写只能操作 8 位数据.  

我在程序中给 `0x43 OUT` 的是 `0xB6`,即 `10110100`.  

- 第 0 位是 0,表示写入的是二进制数,  
- 第 1-3 位代表计数器的运行方式,`010` 代表以方式 2 运行,即分频器方式.  
- 第 4-5 位代表给 `0x40`-`0x42` 端口读写数据的方式,`11` 代表先读低 8 位,再读高 8 位.  
- 第 6-7 位代表选择的计数器,`10` 代表选择选择计数器 2,即 `0x42` 端口.  

再来解释 `0x61` 端口.  

计数器 2 的时钟信号一个 `1193180Hz` 的输入,计数器 2 的另外一个输入是 `0x61` 端口的第 0 位,当第 0 位为 1 时,计数器 2 启动.  

另外,计数器 2 的输出端和 `0x61` 端口的第 1 位经过与门之后输出到喇叭,以此来驱动喇叭.  

所以要让喇叭启动,就把 `0x61` 端口的低两位置 1 就可以了,要停止就置 0.  

详细的看代码.  

解释完了,希望不是废话.![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")  

## 代码实现

下面看代码.  

代码:

```c
#define ONE_MyBeep 600 
#define HALF_MyBeep 300 

#define NOTE_1 440 
#define NOTE_2 495 
#define NOTE_3 550 
#define NOTE_4 587 
#define NOTE_5 660 
#define NOTE_6 733 
#define NOTE_7 825 

UCHAR In_8(PUCHAR Port) 
{ 
UCHAR Value; 
__asm 
{ 
mov edx,Port 
in al,dx 
mov Value,al 
nop 
nop 
} 
return Value; 
} 

void Out_8(PUCHAR Port,UCHAR Value) 
{ 
__asm 
{ 
mov edx,Port 
mov al,Value 
out dx,al 
nop 
nop 
} 
} 

void MyBeep(USHORT freq,ULONG duration) 
{ 
LARGE_INTEGER timeout; 

UCHAR stop; 

USHORT timer=1193180/freq; 

UCHAR start; 
start=In_8((PUCHAR)0x61); 
start=start|3; 

timeout=RtlConvertLongToLargeInteger(-10000*duration); 

/*0x61 端口低两位置 1，启动喇叭*/ 
Out_8((PUCHAR)0x61,start); 

/*计数器控制端口 0x43，置位为 10110100*/ 
Out_8((PUCHAR)0x43,0xB4); 

/*先写低八位*/ 
Out_8((PUCHAR)0x42,timer&0xF); 

/*再写高八位*/ 
Out_8((PUCHAR)0x42,(timer>>8)&0xF); 

/*喇叭持续时间*/ 
KeDelayExecutionThread(KernelMode,FALSE,&timeout); 

/*0x61 端口低两位置 0，停止喇叭*/ 
stop=In_8((PUCHAR)0x61); 
stop=stop&0xFC; 
Out_8((PUCHAR)0x61,stop); 

} 

void PlayMusic(void) 
{ 
MyBeep(NOTE_5, ONE_MyBeep); 
MyBeep(NOTE_3, HALF_MyBeep); 
MyBeep(NOTE_5, HALF_MyBeep); 
MyBeep(NOTE_1*2, ONE_MyBeep*2); 

MyBeep(NOTE_6, ONE_MyBeep); 
MyBeep(NOTE_1*2, ONE_MyBeep); 
MyBeep(NOTE_5, ONE_MyBeep*2); 

MyBeep(NOTE_5, ONE_MyBeep); 
MyBeep(NOTE_1, HALF_MyBeep); 
MyBeep(NOTE_2, HALF_MyBeep); 
MyBeep(NOTE_3, ONE_MyBeep); 
MyBeep(NOTE_2, HALF_MyBeep); 
MyBeep(NOTE_1, HALF_MyBeep); 
MyBeep(NOTE_2, ONE_MyBeep*4); 

MyBeep(NOTE_5, ONE_MyBeep); 
MyBeep(NOTE_3, HALF_MyBeep); 
MyBeep(NOTE_5, HALF_MyBeep); 
MyBeep(NOTE_1*2, HALF_MyBeep*3); 
MyBeep(NOTE_7, HALF_MyBeep); 
MyBeep(NOTE_6, ONE_MyBeep); 
MyBeep(NOTE_1*2, ONE_MyBeep); 
MyBeep(NOTE_5, ONE_MyBeep*2); 

MyBeep(NOTE_5, ONE_MyBeep); 
MyBeep(NOTE_2, HALF_MyBeep); 
MyBeep(NOTE_3, HALF_MyBeep); 
MyBeep(NOTE_4, HALF_MyBeep*3); 
MyBeep(NOTE_7/2, HALF_MyBeep); 
MyBeep(NOTE_1, ONE_MyBeep*4); 

MyBeep(NOTE_6, ONE_MyBeep); 
MyBeep(NOTE_1*2, ONE_MyBeep); 
MyBeep(NOTE_1*2, ONE_MyBeep*2); 

MyBeep(NOTE_7, ONE_MyBeep); 
MyBeep(NOTE_6, HALF_MyBeep); 
MyBeep(NOTE_7, HALF_MyBeep); 
MyBeep(NOTE_1*2, ONE_MyBeep*2); 

MyBeep(NOTE_6, HALF_MyBeep); 
MyBeep(NOTE_7, HALF_MyBeep); 
MyBeep(NOTE_1*2, HALF_MyBeep); 
MyBeep(NOTE_6, HALF_MyBeep); 
MyBeep(NOTE_6, HALF_MyBeep); 
MyBeep(NOTE_5, HALF_MyBeep); 
MyBeep(NOTE_3, HALF_MyBeep); 
MyBeep(NOTE_1, HALF_MyBeep); 
MyBeep(NOTE_2, ONE_MyBeep*4); 

MyBeep(NOTE_5, ONE_MyBeep); 
MyBeep(NOTE_3, HALF_MyBeep); 
MyBeep(NOTE_5, HALF_MyBeep); 
MyBeep(NOTE_1*2, HALF_MyBeep*3); 
MyBeep(NOTE_7, HALF_MyBeep); 
MyBeep(NOTE_6, ONE_MyBeep); 
MyBeep(NOTE_1*2, ONE_MyBeep); 
MyBeep(NOTE_5, ONE_MyBeep*2); 

MyBeep(NOTE_5, ONE_MyBeep); 
MyBeep(NOTE_2, HALF_MyBeep); 
MyBeep(NOTE_3, HALF_MyBeep); 
MyBeep(NOTE_4, HALF_MyBeep*3); 
MyBeep(NOTE_7/2, HALF_MyBeep); 
MyBeep(NOTE_1, ONE_MyBeep*3); 

} 
```

## 附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `PlayMusic.rar`  
|
