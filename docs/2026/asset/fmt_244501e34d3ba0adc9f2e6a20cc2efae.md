
```csharp
[DllImport("kernel32", EntryPoint="GetModuleHandleA", CharSet=CharSet.Ansi, SetLastError=true, ExactSpelling=true)]

private static extern int //这个函数名字是”\r\n”，下面的名字均是这种情况导致换行

([MarshalAs(UnmanagedType.VBByRefStr)] ref string lpModuleName);

[DllImport("MShare.dll", EntryPoint="EC1DB9C1620C48588C4701045B242FA9", CharSet=CharSet.Ansi, SetLastError=true, ExactSpelling=true)]

private static extern bool //同上

(int a);

[DllImport("MShare.dll", EntryPoint="F1B0C9B05CF2496c8873B60602A22743", CharSet=CharSet.Ansi, SetLastError=true, ExactSpelling=true)]

private static extern bool

(int a, int b, int c, int d, int e);
```

// Fields
```csharp
private static bool
;

private static int
;
```

```csharp
internal static $$struct0x6000001-1 $$method0x6000001-1 = { 0x35, 0x46, 0x42, 0x32, 0x38, 0x31, 0x46, 0x36 };
```

// Nested Types
```csharp
[StructLayout(LayoutKind.Explicit, Size=8, Pack=1)]
private struct $$struct0x6000001-1

{
}
```

}

```csharp
internal class Class1
{
 // Methods
 private static void
();
 [DllImport("kernel32", EntryPoint="GetModuleHandleA", CharSet=CharSet.Ansi, SetLastError=true, ExactSpelling=true)]
 private static extern int
([MarshalAs(UnmanagedType.VBByRefStr)] ref string lpModuleName);
 [DllImport("MShare.dll", EntryPoint="EC1DB9C1620C48588C4701045B242FA9", CharSet=CharSet.Ansi, SetLastError=true, ExactSpelling=true)]
 private static extern bool
(int a);
 [DllImport("MShare.dll", EntryPoint="F1B0C9B05CF2496c8873B60602A22743", CharSet=CharSet.Ansi, SetLastError=true, ExactSpelling=true)]
 private static extern bool
(int a, int b, int c, int d, int e);
 public Class1();
 [STAThread]
 private static void Main();
 // Fields
 private static bool
;
 private static bool
;
 private static int
;
}
```

```csharp
[STAThread]
private static void Main()
{
 if (Class1.
== 0)
 {
 Class1.
= Class1.
(ref Assembly.GetExecutingAssembly().Location);
 }
 Class1.
(Class1.
);
 Class1.
(Class1.
, 0x2e, 2, 0, 1);
 Class1.
();
}
```

其中`method0x6000001-1`应该就是加密后的函数体了。`MaxtoCode`思想上和`Remotesoft`是类似的，不过实现手法上有所不同。`MaxtoCode`的`Mshare.dll`文件是`MaxtoCode`作者预先写好的，不包含任何被加密软件的功能，仅仅是确定解密的算法并进行解密（据作者的介绍，企业版共有7种不同的加密算法或变体）。被抽取的功能（即`stolen byte`）是经过加密变换后用静态的方式直接保存在`assembly`当中。于是，`MaxtoCode`的加密强度在于分析出函数体的解密算法的困难性。

对这两种加密办法，目前我并未找到方便的破解办法（但并不代表一定没有，也许只是我不知道）。`.NET`框架并不提供把内存中的`Assembly`内容转换到本地`exe/dll`的办法。如果是`.NET`的`exe`文件，还可以通过`OD`进行调试（当然此时`IL code`已经经过`CLR`自动编译，变成本地代码了），如果是`ASP.NET`的`dll`文件，似乎就无法从`exe`调用进去，只能通过静态分析加密软件产生的对应`dll`文件。也许`.NET`的动态跟踪技术发展成熟后，就能出现破解这类本地代码编译的保护壳的通用办法。

### 后记

在`JasonNET`兄的提醒下，发现`Remotesoft Protector`把类的内容保存在`rscoree.dll`，这个说法太武断了。经过进一步分析，确实很有可能类的内容还是保存在原`dll`中，依据如下

1）查看`ISNet.WebUI.WebGrid.dll`的节区表，发现比正常编译出来的`dll`多了`.rdata`一节，大小为100多k
2）用`ildasm`反编译后，再用`ilasm`重新编译，文件大小明显减少，显然`.NET`的`ildasm`直接忽略处理`.rdata`的内容
3）查看`rscoree.dll`，所有模块间调用，发现`ImageRvaToVa`，`ImageNtHeader`，`VirtualAlloc`等`API`
4）`__abstract.__abstract`当中有不明含义的加密字段以及大量和程序结构有关的`Hashtable`

各种依据表明，被抽调函数内容很可能依然保存在`IsNet.WebUI.WebGrid.dll`当中，特别进行加入此后记，加以更正。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
