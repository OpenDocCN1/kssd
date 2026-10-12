
# 研读 Reflector 的保护原理心得（二）

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-342.htm`](https://www.pediy.com/kssd/pediy07/pediy7-342.htm)

**【标题】** 研读 Reflector 的保护原理心得（二）

**【作者】** henryouly

**【目的】** 研究 .NET 的软件保护技术

**【工具】** `Reflector`, `ildasm`, `OD`, `ultraedit`

**【研究软件】** .NET Reflector 4.1.80.0

**【软件性质】** 国外免费软件

**【下载】** `http://www.aisto.com/roeder/dotnet`

**【声明】** 以单纯技术探讨为目的，转载时请保留原作信息

经过两天的研读，终于把 `Reflector` 的核心程序集的秘密完全揭开了，废话少说，我们直接来看破解过程。

## 初步尝试

上一篇文章说过，可以有两种解密思路，一种是用 `Reflector` 导出 `c#` 源代码，然后直接用 `c#` 编写程序。很快就发现这种方法是不可行的。由于导出的代码，存在大量函数名与变量名重名的情况，使得手动区分并改名的办法难以进行下去。特别是 `int` 各种类型存在隐式转换的问题，使得某些情况下根本无法区分。因此在源码层次上做逆向的思路行不通。

于是考虑第二种思路，直接加入 `IL` 代码。用 `ildasm` 打开，导出……非法操作了！！上网查找发现也有不少人碰到过这种问题，这是混淆器的另外一种功能，利用 `ildasm` 的 `bug` 可以使得 `ildasm` `crack` 掉，无法完整导出文件。可是没有人提出解决办法。

## 进一步分析

偶然发现 `ildasm` 是 `vc7` 编译的，于是我想起了老本行，打破这种僵局的唯一办法就是自己把 `ildasm` 的 `bug` `fix` 掉。用 `OD` `attach` 上 `ildasm` 线程（`ildasm` 是启动后再自己创建一个线程运行的，不知道这样做的道理是什么，可能是想防止 `debug` 吧），运行，停在异常处，跟踪堆栈来到这里：

```
00421892   |.  FF91 D0000000 call dword ptr ds:[ecx+D0]
00421898   |.  0FB645 C4     movzx eax,byte ptr ss:[ebp-3C]
0042189C   |.  8D48 FE       lea ecx,dword ptr ds:[eax-2]                ;  Switch (cases 2..12)，这里 ecx 变成 0xffffffff
0042189F   |.  83F9 10       cmp ecx,10
004218A2   |.  0F87 87010000 ja ildasm.00421A2F            ;跳到 Default case
004218A8   |.  FF248D 8E1A42>jmp dword ptr ds:[ecx*4+421A8E]
004218AF  |>  0FB645 CC     movzx eax,byte ptr ss:[ebp-34]               ;  Cases 4,5 of switch 0042189C
004218B3   |.  50            push eax
004218B4   |.  68 3C6E4000   push ildasm.00406E3C                 ;  ASCII " = int8(0x%02X)"
004218B9   |.  E9 0C010000   jmp ildasm.004219CA
004218BE  |>  0FB745 CC     movzx eax,word ptr ss:[ebp-34]               ;  Cases 6,7 of switch 0042189C
004218C2   |.  50            push eax
004218C3   |.  68 286E4000   push ildasm.00406E28                 ;  ASCII " = int16(0x%04X)"
004218C8   |.  E9 FD000000   jmp ildasm.004219CA
……省略……
00421A2F  |>  57            push edi                                ;  Default case of switch 0042189C
00421A30   |.  FF75 D4       push dword ptr ss:[ebp-2C]                 ; /<%d>
00421A33   |.  8B3D 24134000 mov edi,dword ptr ds:[<&MSVCR71.sprintf>>; |MSVCR71.sprintf
00421A39   |.  50            push eax                                ; |<%02X>
00421A3A   |.  68 4C6D4000   push ildasm.00406D4C                 ; |format = " /* ILLEGAL CONSTANT type:0x%02X, size:%d bytes, blob: "
00421A3F   |.  56            push esi                                ; |s
00421A40   |.  FFD7          call edi                                ; \sprintf
00421A42   |.  83C4 10       add esp,10
00421A45   |.  03F0          add esi,eax
00421A47   |.  837D CC 00    cmp dword ptr ss:[ebp-34],0      ;buffer 的长度，发现是个很大的数
00421A4B      74 1B         je short ildasm.00421A68                 ; 爆破，改成 jmp
00421A4D   |.  68 244D4000   push ildasm.00404D24                 ; /format = "("
00421A52   |.  56            push esi                                ; |s
00421A53   |.  FFD7          call edi                                ; \sprintf
00421A55   |.  59            pop ecx
00421A56   |.  59            pop ecx
00421A57   |.  FF75 10       push dword ptr ss:[ebp+10]                 ; /Arg4
00421A5A   |.  FF75 D4       push dword ptr ss:[ebp-2C]                 ; |Arg3
00421A5D   |.  FF75 CC       push dword ptr ss:[ebp-34]                 ; |Arg2
00421A60   |.  53            push ebx                                ; |Arg1
00421A61   |.  E8 99FCFFFF   call ildasm.004216FF                 ; 输出 buffer 的内容
00421A66   |.  EB 0A         jmp short ildasm.00421A72
00421A68  |>  68 446D4000   push ildasm.00406D44                 ;  ASCII "NULL"
00421A6D   |.  56            push esi
00421A6E   |.  FFD7          call edi
00421A70   |.  59            pop ecx
00421A71   |.  59            pop ecx
00421A72  |>  68 406D4000   push ildasm.00406D40                 ; /src = " */"
00421A77   |.  53            push ebx                                ; |dest
00421A78   |.  E8 F1AB0000   call <jmp.&MSVCR71.strcat>               ; \strcat
00421A7D   |.  59            pop ecx
00421A7E   |.  59            pop ecx
00421A7F   |.  5F            pop edi
00421A80  |>  8B4D FC       mov ecx,dword ptr ss:[ebp-4]
00421A83   |.  5E            pop esi
00421A84   |.  5B            pop ebx
00421A85   |.  E8 97AB0000   call ildasm.0042C621
00421A8A   |.  C9            leave
00421A8B   \.  C2 0C00       retn 0C
```

`00421A2F` 这段是用于默认情况的处理，如果遇到 `ildasm` 不认识的标识位，就直接把缓冲的内容全部输出到 `il` 文件当中。正是这里使得 `ildasm` 发生越界访问错误。简单爆破一下，跳过缓冲区内容的输出。

`00421A4B 74 1B je short ildasm.00421A68`

改成

`00421A4B EB 1B jmp short ildasm.00421A68`

现在 `ildasm` 可以正常产生 `Reflector.il` 文件了

再把有问题的那段 `il` `code` 简单修复一下

```
  .class auto ansi sealed nested private _1
         extends [mscorlib]System.Enum
  {
    .field public specialname rtspecialname int32 value__
    .field public static literal specialname rtspecialname valuetype _3/_1 _Deleted /* ILLEGAL CONSTANT type:0x01, size:415192 bytes, blob: NULL */
    .field public static literal specialname rtspecialname valuetype _3/_1 _Deleted /* ILLEGAL CONSTANT type:0x01, size:415192 bytes, blob: NULL */
    .field public static literal specialname rtspecialname valuetype _3/_1 _Deleted /* ILLEGAL CONSTANT type:0x01, size:415192 bytes, blob: NULL */
    .field public static literal specialname rtspecialname valuetype _3/_1 _Deleted /* ILLEGAL CONSTANT type:0x01, size:415192 bytes, blob: NULL */
    .field public static literal specialname rtspecialname valuetype _3/_1 _Deleted /* ILLEGAL CONSTANT type:0x01, size:415192 bytes, blob: NULL */
    .field public static literal specialname rtspecialname valuetype _3/_1 _Deleted /* ILLEGAL CONSTANT type:0x01, size:415192 bytes, blob: NULL */
    .field public static literal specialname rtspecialname valuetype _3/_1 _Deleted /* ILLEGAL CONSTANT type:0x01, size:415192 bytes, blob: NULL */
    .field public static literal specialname rtspecialname valuetype _3/_1 _Deleted /* ILLEGAL CONSTANT type:0x01, size:415192 bytes, blob: NULL */
    .field public static literal specialname rtspecialname valuetype _3/_1 _Deleted /* ILLEGAL CONSTANT type:0x01, size:415192 bytes, blob: NULL */
    .field public static literal specialname rtspecialname valuetype _3/_1 _Deleted /* ILLEGAL CONSTANT type:0x01, size:415192 bytes, blob: NULL */
    .field public static literal specialname rtspecialname valuetype _3/_1 _Deleted /* ILLEGAL CONSTANT type:0x01, size:415192 bytes, blob: NULL */
  } // end of class _1
```

修改为：

```
  .class auto ansi sealed nested private _1
         extends [mscorlib]System.Enum
  {
    .field public specialname rtspecialname int32 value__
    .field public static literal valuetype _3/_1 enum0 = int32 (0x00000000)
    .field public static literal valuetype _3/_1 enum1 = int32 (0x00000001)
    .field public static literal valuetype _3/_1 enum2 = int32 (0x00000002)
    .field public static literal valuetype _3/_1 enum3 = int32 (0x00000003)
    .field public static literal valuetype _3/_1 enum4 = int32 (0x00000004)
    .field public static literal valuetype _3/_1 enum5 = int32 (0x00000005)
    .field public static literal valuetype _3/_1 enum6 = int32 (0x00000006)
    .field public static literal valuetype _3/_1 enum7 = int32 (0x00000007)
    .field public static literal valuetype _3/_1 enum8 = int32 (0x00000008)
    .field public static literal valuetype _3/_1 enum9 = int32 (0x00000009)
    .field public static literal valuetype _3/_1 enum10 = int32 (0x0000000a)
```



`}` // end of class `_1`

去掉`.publickey`，重新编译成功，哈哈，马上试试能不能运行。

**【有这么简单吗？】**

没有！`FileLoadException` 无情的把答案告诉你。是不是我做错了哪一步？于是我决定先确定是哪个函数抛出的 `FileLoadException`。

于是我修改了 il 文件，插入两句：

```
newobj instance void [mscorlib]System.InvalidOperationException::.ctor()

throw
```

这两句 IL 相当于 `throw new InvalidOperationException()`，把它尝试插入到代码中，如果程序抛 `InvalidOperationException`，说明抛 `FileLoadException` 的语句在插入语句之后。经过一些分析调试，确认了 `FileLoadException` 是在这里抛出的：

```
public Application(IWindowManager windowManager)

{

     this._1 = base.GetType().Assembly;

     Type type1 = this._1.GetType("Reflector.ApplicationManager");

     if (type1 == null)

     {

          AppDomain.CurrentDomain.AssemblyResolve += new ResolveEventHandler(this._1);

          using (Stream stream1 = this._1.GetManifestResourceStream("Reflector.resources")) {

               ……省略

               this._1 = Assembly.Load(_2._1());    //没有抛出 exception

               type1 = this._1.GetType("Reflector.ApplicationManager", true);

          }

     }

     object[] objArray1 = new object[1] { windowManager };

     this._1 = (IServiceProvider) Activator.CreateInstance(type1, objArray1);//抛出 FileLoadException

}
```

也就是说 `Assembly` 是可以正常加载的，解密过程没有错（否则执行 `Assembly.Load` 的时候有 `BadImageFormatException` 抛出），`FileLoadException` 的原因基本确定为，从 resource 中加载的 `Assembly` 引用了 `Reflector.exe`，.NET 的机制检查出 `Reflector.exe` 的强名字已经被窜改，所以拒绝运行。

于是再修改 `Reflector.il`，在 `Reflector.Application..ctor` 中增加一段

```
        IL_00c4:  callvirt   instance unsigned int8[] _4::_1()

        //开始添加我们的代码

        stloc.s temp

        ldstr "C:\\out.dll"

        ldc.i4.2

        ldc.i4.2

        newobj     instance void [mscorlib]System.IO.FileStream::.ctor(string,

                                                       valuetype [mscorlib]System.IO.FileMode,

                                                       valuetype [mscorlib]System.IO.FileAccess)

        stloc.s fs

        ldloc.s fs

        ldloc.s temp

        ldc.i4.0

        ldloc.s temp

        ldlen

        conv.i4

        callvirt instance void [mscorlib]System.IO.Stream::Write(unsigned int8[], int32, int32)

        ldloc.s fs

        callvirt instance void [mscorlib]System.IO.Stream::Close()

        ldloc.s temp

        //添加结束

        IL_00c9:  call       class [mscorlib]System.Reflection.Assembly [mscorlib]System.Reflection.Assembly::Load(unsigned int8[])
```

同时增加上面那段代码用到的两个本地变量

```
     .locals init (class [mscorlib]System.Type V_0,

          class [mscorlib]System.IO.Stream V_1,

          int32 V_2,

          unsigned int8[] V_3,

          unsigned int8 V_4,

          unsigned int8 V_5,

          int32 V_6,

          class _3 V_7,

          class _4 V_8,

          object[] V_9,

          unsigned int8[] temp,//增加

          class [mscorlib]System.IO.FileStream fs)//增加
```

编译后用 `Reflector` 看到的代码为

```
             _3 _1 = new _3(new MemoryStream(buffer1));

             _1.MoveNext();

             _4 _2 = (_4) _1.Current;

             byte[] buffer2 = _2._1();  //修改过的代码，原来是 this._1 = Assembly.Load(_2._1());

             FileStream stream2 = new FileStream(@"C:\out.dll", FileMode.Create, FileAccess.Write); //添加的代码

             stream2.Write(buffer2, 0, buffer2.Length); //添加的代码

             stream2.Close();//添加的代码

             this._1 = Assembly.Load(buffer2);//修改过的代码

             type1 = this._1.GetType("Reflector.ApplicationManager", true);
```

运行，还是 `FileLoadException`，不过这正是我们期待的（如果是其他 `Exception` 就说明我加入的代码有问题），赶快用 `Reflector` 打开 `c:\\out.dll`，呵呵，核心代码提取出来了（虽然里面还是加了混淆），外壳已经成功脱掉。

**【总结】**

`Reflector` 外壳自身的保护方法：

1.  加入了反 `ildasm`，利用 `ildasm` 的溢出漏洞来防止 `ildasm` 导出
2.  用了强名字，文件中有 `.publickey` 信息，签名用的 `private key` 只有作者才有，文件被修改后签名信息必然会改变。
3.  混淆，并且构造大量不同类型、重名的类和变量，并用同一个命名空间，使得无法通过修改命名空间的字符串来进行区分两者，导出的源码也无法直接编译。

`Reflector` 外壳对核心部分的保护方法：

1.  把核心部分加密后作为 resource 保存，并利用 `Assembly.Load` 将解密后的核心部分动态加载
2.  利用 .NET 的机制，当核心部分检测到外壳的强名字无效时，拒绝运行

**破解方法：**

修复 `ildasm` 的 bug 后，添加代码把解密后的内容另存到文件中，获得完整的核心 assembly。
