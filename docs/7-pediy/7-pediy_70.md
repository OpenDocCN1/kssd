

# IPTools 1.10

> 原文：[`www.pediy.com/kssd/pediy03/forum614.htm`](https://www.pediy.com/kssd/pediy03/forum614.htm)

IPTools 1.10 破解   
URL http://newhua.infosail.com/IPTools.htm   
工具：w32dsm 8.93 gold,VC...   
  
4 个月没有破解了，今天碰到了这个不错的 IP 工具，还是 delphi 写的，网上没有该版本的破解，就破它吧！   
w32dsm 反汇编后觉得很简单，一下子就查到了"Sorry, but Name or Registration...."   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00497171(C), :0049718B(C), :004971C6(C), :004971EE(C)   
|   
  
\* Possible StringData Ref from Code Obj ->"Sorry, but Name or Registration "   
                                        ->"number is wrong !"   
                                  |   
:00497340 B89C744900              mov eax, 0049749C   
:00497345 E866C3FAFF              call 004436B0   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00497109(C), :0049733E(U)   
  
看见上面的 4 个出口，找到相关的跳转 nop 掉，Thank You for registering 出来了。   
  
重新打开程序，看见了"Program was corrupted!!"   
  
继续搜索"Program was corrupted!!"，原来只有一个跳转碍事，   
:004B3237 8B1594984B00            mov edx, dword ptr \[004B9894\]   
:004B323D 3B82B4000000            cmp eax, dword ptr \[edx+000000B4\]   
:004B3243 740F                    je 004B3254          //改为 jne   
\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   
\* Possible StringData Ref from Code Obj ->"Program was corrupted !"   
                                  |   
:004B3245 B85C354B00              mov eax, 004B355C   
:004B324A E86104F9FF              call 004436B0   
  
进去之后发现还是没有注册成功，出现一个非整数错误而程序退出，作者可真够狠毒，   
用了一次非法注册号就永远取消使用权？用 regmon 跟踪了一下启动过程，分析后发现   
关键点在 HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Devices\\0010 下面的两   
个键值，真正的注册信息放在这里，但这些内容是经过加密运算后写入的注册码，   
而相同注册码每次产生的键值是不同的。删掉这两个键值，可以进入程序了，但显示为未注册。   
  
在注册表 HKCU\\Software\\Microsoft\\KS-SOFT\\IP-Tools 中发现了这个键值，顾名思义是显示注册对话框的，   
我起初以为只要每次启动前将该键值置 0，就不会显示那个讨厌的 nag 了，于是用 BCB 做了个 loader，   
但发现依然不成功，似乎不是单单由这个键值决定的，而是由多个键值（比如:LastRunTime）   
等等计算决定是否显示 nag，这时已经陷入修改注册表的迷途，索性将所有可能相关的键值都放进   
loader 中，果然收效显著，我甚至以为破解成功了，可我启动程序到达 20 次左右时，又出现了 nag，昏倒！   
  
唉，还是回来看看 w32dsm 的结果吧，进而开始分析其启动过程的代码，很容易找到了 ShowRegDlg   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004B2B32(C)   
|   
:004B2B43 A118994B00              mov eax, dword ptr \[004B9918\]   
:004B2B48 833800                  cmp dword ptr \[eax\], 00000000   
:004B2B4B 0F8586000000            jne 004B2BD7  //就是这里判断是否显示 nag   
\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   
:004B2B51 E8AEFFF4FF              call 00402B04   
  
\* Possible StringData Ref from Code Obj ->"ShowRegDlg"   
                                  |   
:004B2B56 BA582D4B00              mov edx, 004B2D58   
:004B2B5B 8BC7                    mov eax, edi   
:004B2B5D E87A21F8FF              call 00434CDC   
:004B2B62 84C0                    test al, al   
:004B2B64 740F                    je 004B2B75   
  
只要把 0F8586000000,中的 85 改成 84，nag 便不再弹出了，我反复试验了 2000 次（注册表失败综合症？），   
当然不是我自己用鼠标反复的打开关闭，写个程序即可，我只是在旁边看它究竟有没有出现 nag 而已。   
的确是搞定了。   
  
可当我使用其中的 ping 等功能时，竟然弹出这样一个对话框：   
"98% - you use illegal registration key or cracked version of IP-Tools....",原来使用过程   
中还有校验？由于我是爆破，会导致这个异常的产生，其实后来我发现，某些不完全破解的注册码也   
会产生这个问题。   
看了看 w32dsm 的结果，发现这个对话框的弹出是在异常处理模块中的，即 delphi 的   
try   
  X := Y/Z;   
except   
  on EZeroDivide do HandleZeroDivide;   
end;   
语句，这种判断注册有效否的方法我可是头一次见到，写了个简单的 delphi try...except 程序反汇编了   
一下也没找到任何突破口，因为无法知道这个异常是个什么异常，是自己抛出的还是系统产生的。   
  
想用 delphi/bcb 的 CpuView 看看，结果根本无法调进 IPTools.exe 文件，提示我说 dll 加载错误，转而使用 VC 载入，   
竟然成功了，看来 delphi/bcb 这方面的功能的确有待加强，自己编译生成的东西无法调试，vc 却可以，   
加了个断点在这一句上，   
:004B2B4B 0F8586000000            jne 004B2BD7  //就是这里判断是否显示 nag   
随手把 85 改成 84,原因是想避过 nag，可进入程序后点了 ping 功能一下，竟然没有出现那个异常！爽！   
有句话怎么说来着：“老天疼憨人”！，动态修改代码没有问题，我怎么过去没想到？？？   
可惜我的 softice 4.05 for nt 装上后 NT 中鼠标总是飞来飞去，否则早就发现突破口了！   
  
写个 rpp 文件如下，做个 loader，OK!   
T=3:   
F=IP\_Tools.exe: ;   
P=004b2b4b/0F,85/0F,84:   
  
趁热打铁一番，查了查产生先前提到的那个非整数错误的 Call：   
:004B29C8 E8EF6DFDFF              call 004897BC    
  
只要把里面的 E8EF 改为 EB03 即可躲过，省着错填注册号后每次去删那两个键值了。   
  
这样，最终的 RPP 文件如下：   
T=3:   
F=IP\_Tools.exe: ;   
P=004b2b4b/0F,85/0F,84:   
P=004B29C8/E8,EF/EB,03:   
  
这样做出的 loader 后所有的功能可用，NAG 不再弹出，只是看看 license 中还是显示未注册，   
功力的确不够，忘大小侠们多指点。   
  
                                                  傲世男儿   
                                                    China Crack Group   
                                                      01-2-11 02:23 下午  4dd466d0e1e0cc123afbe504da2462ca