
跟踪`CreateFileW`函数，发现这样一段代码：

```
7C8109B0   CALL DWORD PTR DS:[<&ntdll.NtCreateFile>]
```

到这里又一个新东西出现了，即`ZwCreateFile`和`NtCreateFile`，这两个函数由`NTDLL.DLL`导出。反汇编看一下`ZwCreateFile`函数的代码：

```
.text:7C92D0AE      mov     eax, 25h        ;
.text:7C92D0B3      mov     edx, 7FFE0300h
.text:7C92D0B8      call    dword ptr [edx]
.text:7C92D0BA      retn    2Ch
```

`25h`是`ntoskrnl.exe`中`NtCreateFile`的服务ID，十进制是`37`；`7FFE0300`是参数起始地址。

下面是`NTDLL.DLL`中`NtCreateFile`反汇编代码：

```
.text:7C92D0AE   mov     eax, 25h        ; NtCreateFile
.text:7C92D0B3   mov     edx, 7FFE0300h
.text:7C92D0B8   call    dword ptr [edx]
.text:7C92D0BA   retn    2Ch
```

不难看出，`Ntdll`中`Zw`与`Nt`函数的实现是完全一样的。至于为什么要出现两套名字不同、实现方式完全一样的函数，这两种函数，姑且可以这样表示：

```
Typedef Zw Nt   O(∩_∩)O
```

上面讨论的就是所谓的native API，这是用户模式下最底层的函数了。但是为什么会有这两种函数？接下来就进入windows内核大世界探索这个问题。

`Ntdll.dll`通过软件中断`int 2Eh`进入`ntoskrnl.exe`，就是通过中断门切换CPU特权级。CPU特权级共有4个级别，分别是`ring0`，`ring1`，`ring2`，`ring3`。windows只使用其中两个`ring0`和`ring3`特权。`ring0`拥有最高权限，执行任意代码，windows的核心就在这个特权级运行；`ring3`是最低权限。`int 2eh`使cpu从`ring3`级切换到`ring0`级，接着上面的代码，从用户模式进入内核模式，只传递了两个参数，一个是系统服务ID，一个是参数地址。

服务ID有什么用呢？其实在Windows NT中默认存在两个系统服务调度表，它们对应了两类不同的系统服务。这两个系统服务调度表分别是：`KeServiceDescriptorTable`。WDBG看一下：

```
kd> dd KeServiceDescriptorTable

80554020  80502bbc 00000000 0000011c 80503030
80554030  00000000 00000000 00000000 00000000
80554040  00000000 00000000 00000000 00000000
80554050  00000000 00000000 00000000 00000000
80554060  00002710 bf80c391 00000000 00000000
80554070  f8974a80 81f17ba8 81e4a5f0 806e2f40
80554080  00000000 00000000 00000000 00000000
80554090  c8c5ad40 01cb6858 00000000 00000000
```

`KeServiceDescriptorTable`中就只有第一项有数据，其他都是0。其中`80502bbc`就是`KeServiceDescriptorTable.ntoskrnl.ServiceTableBase`，服务函数个数为`0x11c`个，十进制表示为`284`。再看看`80502bbc`地址里是什么：

```
kd> dd 80502bbc

80502bbc  8059a9ba 805e7e36 805eb694 805e7e68
80502bcc  805eb6ce 805e7e9e 805eb712 805eb756
80502bdc  8060cea8 8060dbec 805e3234 805e2e8c
80502bec  805cbe66 805cbe16 8060d4ce 805ac62e
80502bfc  8060cae6 8059ee30 805a6a70 805cd944
80502c0c  80500838 8060e12a 8056cd48 8053603e
80502c1c  8060617e 805b2cba 805ebbce 8061af14
80502c2c  805f00e2 8059b0a8 8061b168 8059a95a
```

上面这些就是系统服务函数的地址了。当我们在`ring3`调用`CreateFile`时，进入`sysenter`的ID是`0x25H`(XP SP3)，然后系统查`KeServiceDescriptorTable`，大概公式是：`KeServiceDescriptorTable.ntoskrnl.ServiceTableBase(80502bbc) + 0x25H * 4`。看一下：

```
kd> dd 80502bbc+25H*4

80502c50  8056f2ee 8056eccc 805cc908 805cc640
80502c60  8061b344 8056f3fc 8060e868 8056f328
80502c70  805a1e18 8059b476 805c84a0 805c83ea
80502c80  8060ec88 805a175c 8060c204 805ba614
80502c90  805c8288 8060e138 805f048a 8059b49a
80502ca0  8063adbc 8063af0c 8060db3c 8060d35e
80502cb0  8060e12a 8056ce8e 8061b7e0 805ebcda
80502cc0  8061b9b0 8056f4b4 8060a178 805b48ce
```

其中`80502c50`指向`8056f2ee`，这个就是`NtCreateFile`的真正地址，跟进看：

```
kd> u 8056f2ee L7

nt!NtCreateFile:

8056f2ee 8bff            mov     edi,edi
8056f2f0 55              push    ebp
8056f2f1 8bec            mov     ebp,esp
8056f2f3 33c0            xor     eax,eax
8056f2f5 50              push    eax
8056f2f6 50              push    eax
8056f2f7 50              push    eax
```

此结构定义如下：

```
typedef struct ServiceDescriptorTable
{
   PVOID ServiceTableBase;  //服务基址
   PVOID ServiceCounterTable(0);//服务调用次数
   unsigned int NumberOfServices;//服务个数
   PVOID ParamTableBase;//统服务参数字节数表的基地址
 }SYSTEM_DESCRIPTOR_TABLE,*PSYSTEM_DESCRIPTOR_TABLE;
```

这个表由`ntoskrnl.exe`导出。另外系统中存在另一个服务描述表`KeServiceDescriptorTableShadow`，其指在GUI线程中使用，`user32.dll`和`gdi32.dll`两个库依赖此结构，定义如下：

```
typedef struct KeServiceDescriptorTableShadow
{
   SYSTEM_SERVICE_TABLE ntoskrnel; //ntoskrnl.exe 的服务函数
   SYSTEM_SERVICE_TABLE win32k; //win32k.sys 的服务函数
   SYSTEM_SERVICE_TABLE NotUsed1;
   SYSTEM_SERVICE_TABLE NotUsed2;
}SYSTEM_DESCRIPTOR_TABLE,*PSYSTEM_DESCRIPTOR_TABLE;
```

这个表未导出，不过确实存在于`Win32.sys`中。由于涉及GUI线程，不在本文谈论范围，略过……

原理知道了，接下来讲解刚才留下的问题，为什么会有这两种函数？在内核中叶存在一套与NATIVE API类似的函数，`Zw*`和`Nt*`，下面做一下对比：

```
kd> u NT! ZWCREATEFILE 

nt!ZwCreateFile:

804ff08c b825000000      mov     eax,25h
804ff091 8d542404        lea     edx,[esp+4]
804ff095 9c              pushfd
804ff096 6a08            push    8
```

又一次出现`25h`，这和`ntdll`中的`ZwCreateFile`还有`NtCreateFile`又一样了，重温一下：

```
.text:7C92D0AE   mov     eax, 25h        ; NtCreateFile \\  ZwCreateFile
.text:7C92D0B3   mov     edx, 7FFE0300h
.text:7C92D0B8   call    dword ptr [edx]
.text:7C92D0BA   retn    2Ch
```

`Ntoskrnl`中的`Nt`函数：

```
kd> u nt! ntcreatefile L14

nt!NtCreateFile:

8056f2ee 8bff            mov     edi,edi
8056f2f0 55              push    ebp
8056f2f1 8bec            mov     ebp,esp
8056f2f3 33c0            xor     eax,eax
8056f2f5 50              push    eax
8056f2f6 50              push    eax
8056f2f7 50              push    eax
8056f2f8 ff7530          push    dword ptr [ebp+30h]
8056f2fb ff752c          push    dword ptr [ebp+2Ch]
8056f2fe ff7528          push    dword ptr [ebp+28h]
8056f301 ff7524          push    dword ptr [ebp+24h]
8056f304 ff7520          push    dword ptr [ebp+20h]
8056f307 ff751c          push    dword ptr [ebp+1Ch]
8056f30a ff7518          push    dword ptr [ebp+18h]
8056f30d ff7514          push    dword ptr [ebp+14h]
8056f310 ff7510          push    dword ptr [ebp+10h]
8056f313 ff750c          push    dword ptr [ebp+0Ch]
8056f316 ff7508          push    dword ptr [ebp+8]
8056f319 e860d8ffff      call    nt!IoCreateFile (8056cb7e)
8056f31e 5d              pop     ebp
```

通过对比，我们就可以刚才那个问题了：`Zw*`函数集合将从用户模式转入内核模式，而`Nt*`符号直接指向的代码会在模式切换后被执行。这个理解了，继续工作。

上面的代码有一句`8056f319 e860d8ffff      call    nt!IoCreateFile (8056cb7e)`，看来要继续跟进：

```
kd> u nt!IoCreateFile L40

nt!IoCreateFile:

8056cb7e 8bff            mov     edi,edi
8056cb80 55              push    ebp
8056cb81 8bec            mov     ebp,esp
8056cb83 83ec0c          sub     esp,0Ch
8056cb86 53              push    ebx
8056cb87 56              push    esi
8056cb88 33f6            xor     esi,esi
8056cb8a 8975fc          mov     dword ptr [ebp-4],esi
8056cb8d 8b1db80b5580    mov     ebx,dword ptr [nt!KeI386MachineType+0x3634 (80550bb8)]
8056cb93 f6c301          test    bl,1
8056cb96 7443            je      nt!IoCreateFile+0x5d (8056cbdb)
8056cb98 8b45fc          mov     eax,dword ptr [ebp-4]
8056cb9b ff45fc          inc     dword ptr [ebp-4]
8056cb9e 85c0            test    eax,eax
8056cba0 7439            je      nt!IoCreateFile+0x5d (8056cbdb)
8056cba2 8b45fc          mov     eax,dword ptr [ebp-4]
8056cba5 6aff            push    0FFFFFFFFh
8056cba7 99              cdq
8056cba8 68f0d8ffff      push    0FFFFD8F0h
8056cbad 52              push    edx
8056cbae 50              push    eax
8056cbaf e8ac9efcff      call    nt!allmul (80536a60)
8056cbb4 817dfce8030000  cmp     dword ptr [ebp-4],3E8h
8056cbbb 8945f4          mov     dword ptr [ebp-0Ch],eax
8056cbbe 8955f8          mov     dword ptr [ebp-8],edx
8056cbc1 7e07            jle     nt!IoCreateFile+0x4c (8056cbca)
8056cbc3 c745fce8030000  mov     dword ptr [ebp-4],3E8h
8056cbca 8d45f4          lea     eax,[ebp-0Ch]
8056cbcd 50              push    eax
8056cbce 56              push    esi
8056cbcf 56              push    esi
8056cbd0 e811daf8ff      call    nt!KeDelayExecutionThread (804fa5e6)
8056cbd5 8b1db80b5580    mov     ebx,dword ptr [nt!KeI386MachineType+0x3634 (80550bb8)]
8056cbdb 56              push    esi
8056cbdc 56              push    esi
8056cbdd ff753c          push    dword ptr [ebp+3Ch]
8056cbe0 ff7538          push    dword ptr [ebp+38h]
8056cbe3 ff7534          push    dword ptr [ebp+34h]
8056cbe6 ff7530          push    dword ptr [ebp+30h]
8056cbe9 ff752c          push    dword ptr [ebp+2Ch]
```
