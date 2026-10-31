
# [原创]反模拟类游戏外挂

## 一、游戏外挂分类

游戏外挂有模拟类、内部Call功能调用、脱机类外挂。模拟类外挂通过模拟键盘鼠标的按键信息达到挂机的目的。内部Call功能挂往游戏进程注入dll并调用游戏功能函数，此类外挂需分析清楚主要功能函数。脱机类外挂需要还原游戏客户端协议明文，见<http://bbs.pediy.com/showthread.php?t=178255>，通过发送协议封包操纵游戏。

按键精灵、按键游侠、许多游戏辅助都属于模拟类外挂。下图是按键精灵的模拟技术的选择：

![](https://bbs.kanxue.com/upload/attach/201309/597552_2z8e81ijvqhd78e.png)

## 二、模拟类外挂检测方法

以下分别介绍上面四种模拟技术的检测方法：

### 1. 普通模式、增强模式

普通模式调用`SendMessage`、`PostMessage`，增强模式调用`keybd_event`，`mouse_event`、`SendInput`发送键盘鼠标消息。`keybd_event`，`mouse_event`是调用`SendInput`。只要HOOK内核层的`NtUserPostThreadMessage`、`NtUserPostMessage`、`NtUserMessageCall`、`NtUserSendInput`这几个API，并在其中判断发送的目的地址是否为我们要保护的进程就行了。附件中代码使用Hook `KiFastCallEntry`的方式Hook `NtUserPostThreadMessage`、`NtUserPostMessage`、`NtUserMessageCall`、`NtUserSendInput`这几个内核API。

### 2. 软件模式

比较常用的方式，很多游戏辅助用的就是这种方式。因为是操作类驱动中的函数，所以能同时兼容PS/2和USB的键盘鼠标模拟，被按键精灵、游戏辅助广泛使用。它直接调用`KeyboardClassServiceCallback`、`MouseClassServiceCallback`。这两个函数是Windows键盘鼠标端口驱动(`Kbdhid.sys`、`i8042prt.sys`、`mouclass.sys`、`mouhid.sys`)在获得键盘鼠标硬件输入后往上传递给类驱动(`Kbdclass.sys`、`mouclass.sys`)的两个回调函数。

如何模拟参考：看雪文章：<http://bbs.pediy.com/showthread.php?t=101653>。

检测方法为：HOOK键盘鼠标类驱动中`KeyboardClassServiceCallback`和`MouseClassServiceCallback`回调，进入到自己的函数中时栈回溯(`ebp+4`)判断调用地址是否在系统的键盘鼠标端口驱动中，若不在说明是直接调用的，也就是按键精灵模拟操作的。

若是键盘鼠标类驱动和端口驱动中有层过滤驱动(`eaps2kbd`)，则需回溯多个栈帧(`ebp+n`)查找系统的键盘鼠标端口驱动。附件中代码使用Call Hook替换键盘鼠标回调。`kbdclass.sys`驱动中`KeyboardClassServiceCallback`函数的反汇编代码如下：

```
:00011192 ; int __stdcall KeyboardClassServiceCallback(size_t, void *, int, int)
.text:00011192 _KeyboardClassServiceCallback@16 proc near
.text:00011192 var_8           = dword ptr -8
.text:00011192 var_4           = dword ptr -4
.text:00011192 arg_0           = dword ptr  8
.text:00011192 arg_4           = dword ptr  0Ch
.text:00011192 arg_8           = dword ptr  10h
.text:00011192 arg_C           = dword ptr  14h
.text:00011192
.text:00011192                 mov     edi, edi
.text:00011194                 push    ebp
.text:00011195                 mov     ebp, esp
.text:00011197                 push    ecx
.text:00011198                 push    ecx
.text:00011199                 mov     eax, [ebp+arg_0]  //DeviceObject
.text:0001119C                 and     [ebp+arg_0], 0
.text:000111A0                 push    ebx
.text:000111A1                 mov     ebx, [ebp+arg_8]  //InputDataEnd
.text:000111A4                 sub     ebx, [ebp+arg_4]   //ebx为键盘输入的数量
.text:000111A7                 push    esi
.text:000111A8                 mov     esi, [eax+28h]
.text:000111AB                 mov     eax, [ebp+arg_C]
.text:000111AE                 and     dword ptr [eax], 0
.text:000111B1                 push    edi
.text:000111B2                 push    4               ; Flags
.text:000111B4          call    PoSetSystemState(x)  //此函数将被Call Hook替换
```

替换后的键盘回调函数代码如下：

```
__declspec(naked)void MyKeyboardClassServiceCallback()
{
	_asm
	{
		pop   g_OrigRetAddr //调用被Call Hook替换的函数前,平衡堆载
		call  g_ulkbdNeedCallAddr //xp是PoSetSystemState函数
		push  eax
		mov   eax,[ebp+4h] //判断调用Callback的函数的地址是否在端口驱动中

		cmp   eax,g_ulKbdPs2PortStart
		jae	  ps2StartOk			   //先判断是否为ps2端口驱动调用,
		jmp   short isUsbKbd		   //再判断是否为usb端口驱动调用
ps2StartOk:
		cmp   eax,g_ulKbdPs2PortEnd
		jbe   over

isUsbKbd:
		cmp   eax,g_ulKbdUsbPortStart
		jae	  usbStartOk
		jmp   short isFilter
usbStartOk:
		cmp   eax,g_ulKbdUsbPortEnd
		jbe   over

isFilter:              //检查是否在过滤驱动中,允许过滤驱动调用kbdCallBack(比如eaps2kbd)
		push  ebx
		push  ecx
		push  edx
		mov   ecx,20   //从ebp+18h(跳过4个传入的参数)往下检测20个堆载帧
		mov   al,0
		mov   ebx,14h
conCheck:
		add   ebx,4
		mov   edx,[ebp+ebx]
		cmp   edx, MmSystemRangeStart  //低于内核地址(一般是0x80000000)
		jb    fOver
		cmp   edx,g_ulKbdPs2PortStart //有过滤驱动
		jae	  _ps2StartOk
		jmp   short _isUsbKbd
_ps2StartOk:
		cmp   edx,g_ulKbdPs2PortEnd
		setbe al        //满足则置al为1
		jbe   fOver

_isUsbKbd:
		cmp   edx,g_ulKbdUsbPortStart
		jae	  _usbStartOk
		jmp   short fLoop
_usbStartOk:
		cmp   edx,g_ulKbdUsbPortEnd
		setbe al        //满足则置al为1
		jbe   fOver

fLoop:
		Loop  conCheck

fOver:
		pop   edx
		pop   ecx
		pop   ebx
		test  al,1
		jnz   over

		mov   eax,[ebp+10h]  //fake
		sub   eax,[ebp+0Ch]
		cmp   eax,ebx		//真实Callback函数调用第一个函数前会把输入的数量赋值给ebx
		jne   cmpEcx		//(其它也要判断下,以防微软代码小变动,winxp使用ebx,win7使用eax)
		mov   ebx,0			//把需要伪造的数量置为0
		jmp   short doLast
cmpEcx:
		cmp   eax,ecx
		jne   cmpEdx
		mov   ecx,0
		jmp   short doLast
cmpEdx:
		cmp   eax,edx
		jne   cmpEsi
		mov   edx,0
		jmp   short doLast
cmpEsi:
		cmp   eax,esi
		jne   cmpEdi
		mov   esi,0
		jmp   short doLast
cmpEdi:
		cmp   eax,edi
		jne   cmpArg1
		mov   edi,0
		jmp   short doLast
cmpArg1:                     //编译器有时会用传入的参数保存临时变量
		cmp   eax,[ebp+8h]   //参数2和3为InputDataStart和InputDataEnd,
		jne   cmpArg4
		mov   [ebp+8],0
		jmp   short doLast
cmpArg4:
		cmp   eax,[ebp+14h]
		jne   doLast
		mov   [ebp+14h],0
		jmp   short doLast
doLast:
		mov   [ebp+8],0		  //DeviceObject置为NULL
		mov   eax,[ebp+0Ch]  //设置InputDataEnd=InputDataStart 键盘输入数量置为0
		mov   [ebp+10h],eax
		mov   m_bFake,1
over:
		pop   eax
		push  g_OrigRetAddr  //返回到真实kbdCallBack Call后的地址
		ret
	}
}
```

### 3. 硬件模式

模拟挂使用`WRITE_PORT_UCHAR`、往`0x60`、`0x64`端口直接发送键盘数据。使用`READ_PORT_UCHAR`读取`0x64`端口状态(实际上是调用`in`、`out`输入输出汇编指令)。检测方法为hook `int 0x93`中断，在中断中获取按键，并要求i8048(键盘控制器)重发上次的按键，若两次不相等则说明是模拟挂在操作键盘。因为直接使用`WRITE_PORT_UCHAR`发送的按键是不会在i8048中保存上次按键的。有关i8048详细资料参考：4a3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4y4h3k6W2M7X3g2^5i4K6u0r3K9i4c8W2L8g2)9J5c8U0j5K6x3e0t1H3x3e0p5@1j5U0g2T1x3r3q4W2k6e0f1#2k6e0f1K6j5U0p5I4k6q4!0q4x3#2)9^5x3q4)9^5x3R3`.`.

以下是检测硬件模拟中操作i8048的代码：

```
BOOL __stdcall AntiKeyboardMock()
{
UCHAR sch=0, ch= 0,vch= 0,temp= 0,comand = 0;
    KIRQL oldIrql, currentIrql;
currentIrql = KeGetCurrentIrql();
if (currentIrql < DISPATCH_LEVEL) //防止多核CPU同时执行
	KeAcquireSpinLock(&KbdLock,&oldIrql);

	_asm
	{
		push eax
		push ebx
cmp  bKbdWriteBacked,1  //鼠标和键盘情况不一样,鼠标发送4个包可能就一次中断,
		mov  bKbdWriteBacked,0  //所以写回不会引起中断,而键盘会
		jz   short retB

		in   al,64h
		test al,00001000b     //若是命令直接返回给原中断
		jnz  retB
		mov al,0x60        // 禁止鼠标键盘接口
		call RealSend8042Cmd
		mov al,0x77
		call Write8042Data
		mov  bl,0
		call Read8042Data
		cmp  bl,1              //判断是否读到数据
		jb   short overDo        //没有数据
		mov  sch,al
		mov  dl,al			   //保存到寄存器,加快处理
		call Flush8042
		mov al,0xFE			   //通过i8042间接要求i8048重发扫描码
		call Write8042Data     //发送给i8048(键盘控制器)不用发送0xD4,直接发给i8048
		mov bl,2	           //表示请求重传,需要等待的时间稍微长些
		call Read8042Data	   //获得重发的扫描码
		mov vch,al             //无需判断是否要求重传命令i8048没接收正确,因为不好判断,
							   //i8048有可能上次发送的是要求重传命令
		cmp dl,al
		jz  writeback
		add al,80h			   //重发的扫描码若等于之前获得扫描码+80h(键按下),认为不是伪造的
		cmp dl,al
		jz  writeback
		mov m_bFake,1
writeback:
		mov al,0xD2
		call RealSend8042Cmd
		mov al,dl
		call Write8042Data //写回数据。写回数据时会再次引起中断,中断后判断原因
		mov  bKbdWriteBacked,1 //,若是写回则返回,若是新数据则处理
overDo:
		mov al,0x60        // 打开鼠标键盘接口
		call RealSend8042Cmd
		mov al,0x47
		call Write8042Data
retB:
		pop ebx
		pop eax
	}
	if (currentIrql < DISPATCH_LEVEL)
		KeReleaseSpinLock(&KbdLock,oldIrql);
	return TRUE;
}
```

附件中`TestProtect`为被保护的测试程序，它会调用dll `bProtect`，由`bProtect`加载驱动，键盘鼠标回调函数由`inpout32.dll`传入。

![](https://bbs.kanxue.com/upload/attach/201309/597552_82hij7q68zio3cu.png)

![](https://bbs.kanxue.com/upload/attach/201309/597552_tymju76gz41kqrs.png)

![](https://bbs.kanxue.com/upload/attach/201309/597552_km1aermaf2soyvm.png)

> [`AntiRabot.rar`](https://bbs.kanxue.com/attach-download-82927.htm)

> [`反模拟类游戏外挂.doc`](https://bbs.kanxue.com/attach-download-82933.htm)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`1.PNG`](javascript:void(0);)

  （69.22kb，10次下载）

- [`2.PNG`](javascript:void(0);)

  （204.82kb，17次下载）

- [`3.PNG`](javascript:void(0);)

  （192.92kb，11次下载）

- [`4.PNG`](javascript:void(0);)

  （223.22kb，14次下载）

- [`AntiRabot.rar`](javascript:void(0);)

  （401.23kb，705次下载）

- [`反模拟类游戏外挂.doc`](javascript:void(0);)

  （640.50kb，666次下载）
