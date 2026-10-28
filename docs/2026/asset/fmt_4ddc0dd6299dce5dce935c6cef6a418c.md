
# [原创]《送别》直接IO版

很早以前就想写个这样的东西,不过自己是音痴,不会读乐谱.![](https://bbs.kanxue.com/view/img/face/16.gif)

上次在看雪上看到 玩命大牛写的用BEEP做的《送别》之后,摘抄了曲子,自己实现了直接IO版本.![](https://bbs.kanxue.com/view/img/face/28.gif)

有几个小细节需要解释:

驱动喇叭的端口:`0x61`,`0x40`-`0x43`.

`0x40`-`0x42`是3个计数器,`0x40`控制系统时钟,`0x41`控制DRAM刷新的,`0x42`就是留给一般应用,比如扬声器,我们要用到的就是`0x42`.

`0x43`是这3个计数器的控制端口,下面详细解释.

这些端口都是8位操作的,所以每次读写只能操作8位数据.

我在程序中给`0x43` `OUT` 的是`0xB6`,即`10110100`.

第0位是0,表示写入的是二进制数,

第1-3位代表计数器的运行方式,`010`代表以方式2运行,即分频器方式.

第4-5位代表给`0x40`-`0x42`端口读写数据的方式,`11`代表先读低8位,再读高8位.

第6-7位代表选择的计数器,`10`代表选择选择计数器2,即`0x42`端口.

再来解释`0x61`端口.

计数器2的时钟信号一个`1193180Hz`的输入,计数器2的另外一个输入是`0x61`端口的第0位,当第0位为1时,计数器2启动.

另外,计数器2的输出端和`0x61`端口的第1位经过与门之后输出到喇叭,以此来驱动喇叭.

所以要让喇叭启动,就把`0x61`端口的低两位置1就可以了,要停止就置0.

详细的看代码.

解释完了,希望不是废话.![](https://bbs.kanxue.com/view/img/face/16.gif)

下面看代码.

```
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





/*0x61端口低两位置1，启动喇叭*/ 

Out_8((PUCHAR)0x61,start); 



/*计数器控制端口0x43，置位为10110100*/ 

Out_8((PUCHAR)0x43,0xB4); 



/*先写低八位*/ 

Out_8((PUCHAR)0x42,timer&0xF); 



/*再写高八位*/ 

Out_8((PUCHAR)0x42,(timer>>8)&0xF); 



/*喇叭持续时间*/ 

KeDelayExecutionThread(KernelMode,FALSE,&timeout); 





/*0x61端口低两位置0，停止喇叭*/ 

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

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [PlayMusic.rar](javascript:void(0);)

  （2.47kb，197次下载）
