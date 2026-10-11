
# .net 程序保护方式大观

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-366.htm`](https://www.pediy.com/kssd/pediy08/pediy8-366.htm)

## .net 软件保护方式大观

最近调试一个运行于 .net 2.0 下的软件，发现该软件使用的保护方式很具有代表性，基本囊括了现在 .net 下的所有保护措施。实践证明，这些保护措施就像全真七子，单打独斗功力差了点儿，但结合起来应用还是有一定强度的。下面做以说明，供 .net 开发者参考。

### 1. 加壳

该软件使用 `MaxtoCode` 加壳，该壳会生成本地 `dll` 文件，在运行时通过动态挂钩 .net 内核解密，并且是 `each method` 解密，所以不会在内存中出现完整的 `assembly`，使得传统的内存 `dump` 方法失效。类似的壳还有国外的 `Remotesoft Protector`、`XHEO CodeVeil`、`.Net Reactor`。这类壳本身就提供了反调试功能，再加上壳的本地 `dll` 也被加壳，因此这一步便可以挡住 80% 的逆向分析者。

但是 `rick`(http://rick.cnblogs.com) 已经做出了脱壳机，所以对于这种保护的程序第一步仍是 `dump`。脱壳后，便可利用 `ildasm` 将程序集反编译，将所有的引用本地 `dll` 的代码删除，再次编译后便可正常运行：

```
.method private specialname rtspecialname static 
          void   .cctor() cil managed 
  { 
    // Code size       6 (0x6) 
    .maxstack  8 
    IL_0000:  //call       void InFaceMaxtoCode::Startup() 
    IL_0005:  ret 
  } // end of method xebd220b94e14b2d7::.cctor 
```

为了减肥，也可将 `InFaceMaxtoCode` 的代码删除，这些代码还是占据了一些体积的。

### 2. 混淆

该软件使用了 `Xenocode Postbuild` 进行混淆，`xenocode` 的保护包括了名称混淆和流程混淆。其中名称通常被改为如下形式：`x485ea7930f0abd9a` `xdc0a6303c9bfe8dd`。这样，代码的名称就无法代表原来的意义。

除了名称混淆，该软件还使用了流程混淆，加入了许多垃圾代码，比如：

(1) 永远为“true”或“false”的跳转判断

```
IL_0086:  ldc.i4.0 
IL_0087:  brtrue.s   IL_0032 
IL_0089:  ldc.i4.0 
IL_008a:  brfalse.s  IL_00e6 
```

(2) 将代码分块并添加多个跳转，所以在每个方法尾部会看到如下类似跳转表的代码

```
IL_0296:  br.s       IL_029d 
IL_0298:  br         IL_022c 
IL_029d:  ldc.i4.0 
IL_029e:  brfalse.s  IL_026f 
IL_02a0:  ldc.i4     0x3 
IL_02a5:  brtrue     IL_0227 
IL_02aa:  br         IL_0204 
IL_02af:  br         IL_0032 
```

混淆过的方法在反编译为高级语言时已经非常难以分析，如下：

```
if (Field34_4.Columns.Count != 0) 
    goto label_11; 
bool flag2 = (flag1 - i) > 0xFFFFFFFF; 
if (!flag2) 
    goto label_10; 
while (true) 
{ 
    if (!0) 
        goto label_4; 
    Field34_1.Fill(Field34_4, Field34_2); 
    return true; 
    label_1: 
    flag2 = (flag1 + flag1) > 0xFFFFFFFF; 
    if (!flag2 && true) 
        goto label_7; 
    label_2: 
    Field34_4.Clear(); 
}
```

### 3. 字符串加密

在 `win32` 下用 `wdasm` 查找字符串参考可能是逆向分析的第一步，.net 中也是这样，一些敏感字符串会暴露程序作者的意图。不过现在的混淆程序都提供了源代码加密，比如：

```
IL_0086:  ldstr      "bppkcahlaaolhafmbllmhpcnopjnikaogohoapoopjfpbpmpho" 
+ "daiokainbbhnibgnpbnngclnncjmeddnlddmcepljeihafilhfbmofplfgfmmghhdh" 
IL_008b:  ldc.i4     0x3c9baf9d 
IL_0090:  call       string xb9d8bb5e6df032aa.x7841ead83ad1c299::_bc25ec13a5229081(string,int32) 
```

这段代码解密出的字符是`This is an unregistered copy`，这样便有效地防止了程序在第一时间被定位至关键代码。开发者也可以在自己的程序中自行加入解码方法。

### 4. anti-debug

反调试就是当检测到有解密软件运行时，让程序自动退出。见如下代码：

```
.method private static pinvokeimpl("user32" lasterr winapi) 
          bool   EnumWindows(class CodeLib.x41a6f5dc72fea230/xdd2ea1989d3fc452 xa479061352f99870, 
                              native int x7a5eebe5d933ebbc) cil managed preservesig 
  { 
  } 
IL_0114:  ldloc.2 
IL_0115:  ldstr      "agnhoheigglificjphjjbiakpghkkgokihflahmlahdm"//DeProtector 
IL_011a:  ldc.i4     0x50e67d1c 
IL_011f:  call       string xb9d8bb5e6df032aa.x7841ead83ad1c299::_bc25ec13a5229081(string, 
                                                                     int32) 
IL_0124:  call       string [mscorlib]System.String::Intern(string) 
IL_0129:  callvirt   instance int32 [mscorlib]System.String::IndexOf(string) 
IL_012e:  ldc.i4.m1 
IL_012f:  bgt        IL_009f 
IL_0134:  br         IL_0010 
IL_0139:  br         IL_01e4 
IL_013e:  ldloc.2 
IL_013f:  ldstr      "mkbbemibanpbljgcimnceledjlldllcenljehgafikhfalofclfgpimgkkdhgjkhfjbiejii"//JetBrains dotTrace 
IL_0144:  ldc.i4     0x43ee1162 
IL_0149:  call       string xb9d8bb5e6df032aa.x7841ead83ad1c299::_bc25ec13a5229081(string, 
                                                                     int32) 
IL_014e:  call       string [mscorlib]System.String::Intern(string) 
IL_0153:  callvirt   instance int32 [mscorlib]System.String::IndexOf(string) 
IL_0158:  ldc.i4.m1 
IL_0159:  bgt        IL_009f 
IL_015e:  ldloc.2 
IL_015f:  ldstr      "iofohpmojpdpmmkphobajoia"//WinDbg 
IL_0164:  ldc.i4     0xdd4e591 
IL_0169:  call       string xb9d8bb5e6df032aa.x7841ead83ad1c299::_bc25ec13a5229081(string, 
                                                                     int32) 
IL_016e:  call       string [mscorlib]System.String::Intern(string) 
IL_0173:  callvirt   instance int32 [mscorlib]System.String::IndexOf(string) 
IL_0178:  ldc.i4.m1 
IL_0179:  bgt        IL_009f 
```

程序从 `user32.dll` 中调用 `EnumWindows` 枚举当前所有窗口并取得名称，和解码过的敏感字符串进行对比。虽然这种方法“层次较高”，但仍不失为一种自我保护的手段。再就是当软件要进行敏感操作，比如在注册表中保存信息，从某个文件中读取信息时，也可以使用这些手段。

```
IL_0000:  ldnull 
IL_0001:  ldstr      "Registry Monitor - Sysinternals: www.sysinternals.com" 
IL_0006:  call       native int [CodeLibWin]CodeLib.Win32.WindowsAPI::FindWindow(string, string) 
IL_0101:  ldnull 
IL_0102:  ldstr      "File Monitor - Sysinternals: www.sysinternals.com" 
IL_0107:  call       native int [CodeLibWin]CodeLib.Win32.WindowsAPI::FindWindow(string,string) 
```

利用系统方法也可以进行检测是否有调试器：

```
IL_002d:  call       bool [mscorlib]System.Diagnostics.Debugger::get_IsAttached() 
IL_0032:  ldc.i4.0 
IL_0033:  ceq 
```

### 5. 时间验证

在关键代码处，如果进行单步跟踪调试，则从一句代码到另一句代码会运行较长时间。因此可以预估计一下正常的执行时间，然后进行对比，若超过这个时间界限，则认为被调试。代码如下：

```
IL_22df:  ldloca.s   V_51 
IL_22e1:  ldloc.s    V_14 
IL_22e3:  call       instance valuetype [mscorlib]System.TimeSpan [mscorlib]System.DateTime::Subtract(valuetype [mscorlib]System.DateTime) 
IL_22e8:  stloc.s    V_52 
IL_22ea:  ldloca.s   V_52 
IL_22ec:  call       instance int32 [mscorlib]System.TimeSpan::get_Seconds() 
IL_22f1:  ldc.i4.3 
IL_22f2:  cgt 
```

分别取两次系统时间，算出秒数差，和 3 秒进行对比，若大于 3 秒则跳转。

### 6. 程序员自身的安全意识

无论什么保护手段，都需要程序作者会熟练运用，并且加强自身的安全意识。不过个人觉得该软件的作者有点过了，后期基本没更新软件本身的什么功能，全用于加强保护了。汗！（另：好像还有网络验证，还没看到）

`4dd466d0e1e0cc123afbe504da2462ca`
