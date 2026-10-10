

# 用 PEBrowse 对.Net 程序进行动态调试

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-236.htm`](https://www.pediy.com/kssd/pediy08/pediy8-236.htm)

用 PEBrowse 对.Net 程序进行动态调试   
tankaiha\[NE365\]@www.vxer.cn   
2006.04.25   
  
    本文主要讲了通过在.Net 平台下动态调试应用程序来破解程序的一些思路和方法，研究的对象是 Code Library .Net(Access)，版本号 13.8.2294.11793。   
一、    调试工具的选择   
我通常在.Net 平台下使用的动态调试方法有两种：一是用 ildasm 将程序反汇编，然后进行必要修改(如去除强名称)，再用 ilasm 汇编为带 debug 信息的，然后用 Framework SDK 自带的 GuiDebug 进行调试。但是这种方法我用的还不多，因为我机上没有 1.1 版本的 SDK，而现在更多的程序是 1.1 的，所以我更喜欢用第二种：用第三方调试软件(类似 win32 下的 Ollydbg)来直接调试，这里使用的是 PEBrowse Professional Interactive。具体使用方法结合例子在后面再说。   
  
二、    静态分析   
    拿到一个程序我们首先要做的就是静态分析，看看有没有敏感信息，以便找到关键代码。用 Reflector 打开后，发现程序已经混淆，代码直接是看不到的。现在的.Net 程序基本都做过混淆，在名称混淆方面做的比较变态的是把名称混淆为不可打印字符，这样很多反汇编程序出来的代码都会出错，即使不出错也分不清谁跟谁了！只能用 ildasm 直接反汇编成 il 代码了。之后打开 CodeLib.il，发现流程也混淆了，跟踪一个函数发现两三句指令就是一个跳转，非常不爽。   
    先来看看有没有什么明文的敏感信息吧，通过搜索 il 文件，我们发现了“User”“Keycode”两个关键字段，代表什么显而易见，就是开机要我们输入的用户名和密码。然后不管你输入对错都是弹出“感谢注册”的对话框，这种形式通常注册信息被保存在文件或者注册表中，果然，我们在文件中发现了“Software\\VB and VBA Program Settings\\fishCodeLib”字段，这是信息保存的位置。   
    但是还有一些字符串没有搜到，比如第一个窗口中的“This is an unresigered copy”。仔细看一下，代码中有很多 bytearray，比如：   
ldstr      bytearray (46 D8 57 DF 55 E6 5C ED 06 F4 4C FB 53 02 FD 08   
                  3B 10 45 17 F4 1D 46 25 3C 2C 3D 33 2D 3A 2C 41   
                  2B 48 32 4F 30 56 1E 5D 28 64 18 6B 14 72 CD 78   
                  0D 80 16 87 14 8E 1A 95 CC 9B )   
ldc.i4     0x6e66d7f2   
call string xed3d7768b7546353.x7846ebd83ad1c299::\_bc22ed13a5229081(string, int32)   
    看来字符串都被加了密。写个 il 程序解码，代码就用程序中的\_bc22ed13a5229081(string,int32)，具体代码见附件。解码后得到这段字符串正是“This is an unresigered copy”。同理我们可以将其它的相关字符串统统解码。   
  
三、    静态流程分析   
首先在 il 文件中静态跟一下流程，但是经过混淆的代码看起来实在是痛苦，鼠标的滚轮没少受折磨。搜一下文件中的“Keycode”，发现太多地方使用了，没有办法一下确定哪一处是关键点。（这么多关键点，看来爆破是没折了！）但至少我们发现一个关键信息：   
bool CodeLib.x83a1ac2c48984168::x6ae105ff7a55905e   
是判断是否注册的关键变量，根据吗，看下面的代码：   
  
  
\--------------------------------------------------------------------------------  
   

>  
> 
> ```
> 代码:
> 
> * * *
> 
>   .method private instance void  xb4fe74a3788695e0(object x897f7d97ff45d640, 
>            class \[mscorlib\]System.EventArgs x92a51111cd18ca81) cil managed 
>   { 
>     // Code size       122 (0x7a) 
>     .maxstack  7 
>     .locals init (class CodeLib.xf9b7e3b2c792f083 V\_0) 
>     IL\_0000:  newobj     instance void CodeLib.xf9b7e3b2c792f083::.ctor() 
>     IL\_0005:  stloc.0 
>     IL\_0006:  ldsfld     bool CodeLib.x83a1ac2c48984168::x6ae105ff7a55905e //注意这句 
>     IL\_000b:  brfalse.s  IL\_0017  //然后这里跳转 
> 
>     IL\_000d:  ldloc.0 
>     IL\_000e:  callvirt   instance class \[System.Windows.Forms\]System.Windows.Forms.Label CodeLib.xf9b7e3b2c792f083::get\_x6bc866479ed84b6d() 
>     IL\_0013:  br.s       IL\_0043 
> 
>     IL\_0015:  br.s       IL\_0072 
> 
>     IL\_0017:  ldloc.0 
>     IL\_0018:  callvirt   instance class \[System.Windows.Forms\]System.Windows.Forms.Label CodeLib.xf9b7e3b2c792f083::get\_x6bc866479ed84b6d() 
>     IL\_001d:  ldstr      bytearray (AF 06 CD 0D CF 14 78 1B A7 22 B7 29 B6 30 B5 37   // ......x..".).0.7 
>                                     BC 3E BA 45 A8 4C B2 53 A2 5A 9E 61 58 68 )       // .>.E.L.S.Z.aXh 
>     IL\_0022:  ldc.i4     0x5c4c0661 
>     //解码后的信息 
>     //Not Registered! 
>     IL\_0027:  call       string xed3d7768b7546353.x7846ebd83ad1c299::\_bc22ed13a5229081(string, 
>                                                                                        int32) 
>     IL\_002c:  call       string \[mscorlib\]System.String::Intern(string) 
>     IL\_0031:  callvirt   instance void \[System.Windows.Forms\]System.Windows.Forms.Control::set\_Text(string) 
>     IL\_0036:  ldloc.0 
>     IL\_0037:  callvirt   instance class \[System.Windows.Forms\]System.Windows.Forms.Label CodeLib.xf9b7e3b2c792f083::get\_x6bc866479ed84b6d() 
>     IL\_003c:  call       valuetype \[System.Drawing\]System.Drawing.Color \[System.Drawing\]System.Drawing.Color::get\_Red() 
>     IL\_0041:  br.s       IL\_006d 
> 
>     IL\_0043:  ldstr      "Licensed to " 
>     IL\_0048:  ldsfld     string CodeLib.x83a1ac2c48984168::xa07a59ecbea7b301 
>     IL\_004d:  ldstr      "Settings" 
>     IL\_0052:  ldstr      "User" 
>     IL\_0057:  ldstr      "" 
>     IL\_005c:  call       string \[Microsoft.VisualBasic\]Microsoft.VisualBasic.Interaction::GetSetting(string, 
>                                                                                                      string, 
>                                                                                                      string, 
>                                                                                                      string) 
>     IL\_0061:  call       string \[mscorlib\]System.String::Concat(string, 
>                                                                 string) 
>     IL\_0066:  callvirt   instance void \[System.Windows.Forms\]System.Windows.Forms.Control::set\_Text(string) 
>     IL\_006b:  br.s       IL\_0015 
> …… 
> 
> * * *
> 
> 
> ```

   
  
    这段代码明显是关于窗口根据判断是否注册而选择显示“Licensed to XXXX”还是未注册。   
  
四、    动态调试   
先来明确一下动态调试的原则：一是以注册表操作为突破口，Microsoft.VisualBasic.Interaction::GetSetting，当不知道输入用户名和密码后程序在哪里进行计算，下在这个上面是最好的，类似于 win32 的 API 断点。再就是，要是上面那个 bool 变量作为关键点判断是否是进行注册码的验证。   
如果运行一下程序会发现，如果未注册，程序会弹出注册窗口，如果注册肯定不会有此窗口，因此判断程序初始会有代码进行注册码验证。为了找到这段代码，我们先在注册表的 User 字段和 Keycode 字段填上用户名和密码。   
打开 PEBrowse，载入 CodeLib.exe，可这里 Microsoft.VisualBasic.dll 并没有载入（应该是第一次使用时载入），怎么办？用菜单 DebugBreak on Module Initialization，在每一个载入模块初始化时中断。几次 F5 之后，左边的模块列表中多了 microsoft.visualbasic.dll 一项。如图 1。   
   ![](img/d5de5afe51ff98bc235c87bb579196f6.jpg) 图 1   
这时，我们可以在 Microsoft.VisualBasic.Interaction::GetSetting 这个方法上下断，方法是在左边列表的 vb 那项上单击右键，选择 Search From，然后输入“GetSetting”，列表便会自动展开到这一项。然后在该项上下断点，如图 2。   
   ![](img/fe2e1fbce4d41f5c57e82a64cee0c369.jpg) 图 2　   
从列表视图中可以看到刚才的断点是否已经添加，如图 3，第一项就是。   
![](img/f30c0be0c6b4d5cba08a54e398e3ffad.jpg)   
 图 3   
接下来便可以运行，等待程序中断了。PEBrowse 的界面实在是不爽，调试时我通常开三个窗口：一个反汇编（asm）指令窗口，这是调试时最基本的；一个 il 反汇编指令窗口，便于观察当前程序的执行流程和功能；一个寄存器窗口。   
取消刚才的 Module 初始化断点，F5 运行，不一会程序就中断在了 GetSetting 的入口处。向下翻，然后在函数最后的 ret 上选择“run to selection”，因为我们对怎么从注册表取信息并不感兴趣，而是要看它取得了什么信息。寄存器窗口中都是保存各种函数信息的地址，双击可以查看（这点 PEBrowse 还是比较方便的）。返回后，双击 eax，如图 4，第一次取的是密码（我在注册表中输入的就是“1111111…”）：   
  ![](img/5d6100fda5fd53216c438ff6d639b7f5.jpg)  图 4   
  
  
F10 步进（也可以用 F12，不过后者是一次跳好几句，要注意看 il 反汇编窗口，别跳过头），回到了调用该方法的程序段，标题栏显示现在是在 x4859a6930f0abd9a::.ctor（注意 PEBrowse 的反汇编形式，上面是 il 指令，下面是 JIT 形成的本机指令）。接下来是检查注册码长度是否为 0，Microsoft.VisualBasic.CompilerServices.StringType::StrCmp()。接下来在我的机器里会报错，没关键，F5 继续运行。运行到   
  ; IL\_0EC5: ldstr "User"   
  ; IL\_0ECA: ldstr ""   
  ; IL\_0ECF: call  Microsoft.VisualBasic.Interaction::GetSetting()   
时，就已经取得了用户名了，查看方法同上。接着走来到下面：   
  ; IL\_1A98: ldarg.0   
  ; IL\_1A99: ldarg.0   
  ; IL\_1A9A: ldfld x859f040f6b146ec7   
  ; IL\_1A9F: callvirt  CodeLib.x4859a6930f0abd9a::xd065633e7dc3dd02()   
    查看 eax，发现上面提到的“FishCodeLib”已经被编码为了“)f(i\*s&h^c%o$d#l!i~b ”，注意最后还有个空格。   
  ; IL\_1A41: ldloc.2   
  ; IL\_1A42: ldarg.0   
  ; IL\_1A43: ldfld x859f040f6b146ec7   
  ; IL\_1A48: ldarg.0   
  ; IL\_1A49: br.s IL\_1A13   
  ; IL\_1A4B: bne.un.s IL\_1AB6   
下面是把“)f(i\*s&h^c%o$d#l!i~b ”和用户名“tankaiha”连接起来：   
  ; IL\_1A13: ldfld x9ba04a28429d23be   
  ; IL\_1A18: ldarg.0   
  ; IL\_1A19: ldfld x859f040f6b146ec7   
  ; IL\_1A1E: call  System.String::Concat()   
再下面就是我们最关键的了：   
  ; IL\_1A23: ldloc.s 0x0A   
  ; IL\_1A25: call  CodeLib.x83a1ac2c48984168::x720d40bd855b5147()   
将连接起来的字符串进行运算，得到了一串编码“D8tOZUJQ4qU7hBEt6fDaOQ==”。   
  
  ; IL\_1A2A: ldc.i4.0   
  ; IL\_1A2B: call  Microsoft.VisualBasic.CompilerServices.StringType::StrCmp()   
  ; IL\_1A30: ldc.i4.0   
  ; IL\_1A31: br.s IL\_1A4B   
和注册码进行比较。和注册码进行比较，那这段代码是不是就是真正的注册码呢？答案是 YES。其实往后走几步，如果比较正常的话，会有这一句：   
  ; IL\_1AB9: ldc.i4.1   
  ; IL\_1ABA: stsfld x6ae105ff7a55905e   
将我们刚才说的关键的 bool 变量置为真。OK，运行一下，输入 tankaiha 和 D8tOZUJQ4qU7hBEt6fDaOQ==，果然，关于窗口已经改变，注册成功了。见图 5   
  
 ![](img/fef2cb9d76fb8476b2372f53f8e0fbeb.jpg)   
  
  
  
  
    本文主要结合一个具体的软件讲了 PEBrowse 的使用，后续的工作还可以做，比如做出注册机等。偶是没有那个精力了！.Net 下的东东，我研究的很肤浅，所以有什么建议，有啥子 BUG，大家尽管拍砖吧！   
  
(decode.il 中是 bytearray 的解码程序示例，使用了 linhanshi 发布的 xacc.ide，呵呵，效果 8 错！) 

 4dd466d0e1e0cc123afbe504da2462ca