

# Shielden2.04 主程序脱壳分析

> 原文：[`www.pediy.com/kssd/pediy12/130783.html`](https://www.pediy.com/kssd/pediy12/130783.html)

编辑一下 怎么变成求职格式了 ![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")  
Shielden 主程序使用 Safengine NetLicensor v2.0.4.0 保护.以下分析可能对 SE 其他版本也同理.  
  
1.Anti:  
S 兄贴了篇 Bypass anti 分析,原理很透彻了.我补充一下.有点嗦,大侠们请跳过.![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")  
  SE 反调试的几个选项全部是 ANTI,可以直接干掉线程,我看了一下主程序,后面也没发现会出问题,当然我没有完整的加一次壳来测试,加壳过程会不会出问题,也无法保证.一开始的时候可以改 CreateThread,直接返回,到 OEP 的时候再取消修改,否则程序里的线程也无法创建.比如 Shielden 主程序拖放试练品进入而创建的 PE 分析线程.如果不取消,将会看到 Shielden 界面灰色没反应.  
  干掉诸如检测 REGMON 等等线程后,主线程里依然有不少 ANTI.体现在对 DRX 的积极检测.壳会检测 TlsValue 与 DR0~DR3 之和是否相当,会 mov r32,\[r32\]通过设置 r32 为 DRX 来构造 80000004 异常,S 兄的文章提到过了.当发现不相等或是没出异常,那么壳就获知当前状态为被调试.对第一条 S 兄采取的办法是在 GetThreadContext 的时候清掉 DRX,并清掉 TlsValue.对第二条则自己构造了一个异常.实际上壳在许多地方会进行第二条比较,比较的地址也不同,所以 S 兄给出的方法可以顺利到达 OEP.但是一旦断下之后,在诸如 IAT 调用之类的与壳通信的地方,又会被壳检测到.因为可能这里出现了另外一条 mov r32,\[r32\].而且 r32 是随机的,不同试练品里不同.DR0~DR3 这 4 个 DWORD,不同的试练品也是不同的.  
  解决办法有两个:  
1>.OD 在 F2 下断并断下的时候,会无情的清掉 DRX.所以还是改改 OD 这个执拗习惯吧.  
修改 OD:  

代码:

```
<0042EA22>

  mov ecx,dword ptr ds:\[4D8D70\]
  mov esi,400
  mov eax,dword ptr ds:\[4D8D8C\]
  xor eax,eax
  mov edx,dword ptr ds:\[4D8DA8\]
  mov edx,4D8D78
  mov ecx,dword ptr ds:\[4D8DC4\]
  nop
  nop
  nop
  nop
  nop
  nop
  nop
  nop
  nop
  nop
  nop
  nop

```

这么改当然简单了,如此一来当中断的时候,OD 将不再清 DRX 了.不过单步也可能会出问题.比如 PUSHFD 这样指令单步就飞了.如果不满意,那就判断 4 个 DRX 是否都为 0 即可.  
代码:

```
<0042EA22>
  jmp 004AF645

<004AF645>

  pushfd
  cmp dword ptr ds:\[4D8D70\],0
  jnz @L00000001
  cmp dword ptr ds:\[4D8D8C\],0
  jnz @L00000001
  cmp dword ptr ds:\[4D8DA8\],0
  jnz @L00000001
  cmp dword ptr ds:\[4D8DC4\],0
  jnz @L00000001
  mov ecx,dword ptr ds:\[4D8D70\]
  mov esi,400
  mov eax,dword ptr ds:\[4D8D8C\]
  xor eax,eax
  mov edx,dword ptr ds:\[4D8DA8\]
  mov edx,4D8D78
  mov ecx,dword ptr ds:\[4D8DC4\]
  jmp @L00000002

@L00000001:
  mov ecx,dword ptr ds:\[4D8D70\]
  mov esi,400
  mov dword ptr ds:\[ebx+4\],ecx
  mov eax,dword ptr ds:\[4D8D8C\]
  mov dword ptr ds:\[ebx+8\],eax
  xor eax,eax
  mov edx,dword ptr ds:\[4D8DA8\]
  mov dword ptr ds:\[ebx+C\],edx
  mov edx,4D8D78
  mov ecx,dword ptr ds:\[4D8DC4\]
  mov dword ptr ds:\[ebx+10\],ecx

@L00000002:
  popfd
  jmp 0042EA51

```

代码我不优化了,时间宝贵.  
这样以后,可以 F2 而不用担心被 ANTI 了.当然,依然面临被内存校验到.这里用 S 兄的方法,搜索  
代码:

```
9C810C24000100009D0F31

```

,将 rdtsc 给 NOP 掉即可.  
  
2>.使用方法 1,将可以随意调试而不用担心被 SE 的"检测调试事件"给 ANTI 到了.这里避免了对 G(S)etThreadContext 的 Hook 以免不便(不同程序的 mov r32,\[r32\]代码不同导致 HOOK  
  
麻烦)以及可能的各种崩溃(本机 WIN7,HOOK 之后会在不明之处挂掉,其他系统未测试.).  
这里依然没有解决硬断以及单步的问题.单步不用说了,和谐单步方法多种多样,躲不完的.对于硬断,SE 作者这里犯了一条大忌.  
作者把"检测调试事件"给写进了同一个函数里,而且所有需要检测调试的地方,全部调用的是同一个函数.所以只要一个字节,所有的检测都将无效.  
对 Shielden2.04 主程序而言,只需将 006C0874 改为 C3 即可.在看到 SE 中对乱序中的 JCC 都给展开了虚假分支执行相同代码之后,再看到这里,有点说不出的奇怪感觉.  
可以看出,使用方法 2 也可以避免修改 OD 了.更不用 HOOK API 了.  
  
二.修复输入表:  
不知道从什么时候起,导入表已经没有"表"的概念了.尤其在加密壳里面,更是千方百计把 API 都拆散,避免放在一起方便自己也方便脱壳者.所以现在的脱壳技术也不执着于重建一张完整的表了,只需要在程序调用 API 的时候,给他一个正确的 API 让他调用就是了.而 SE 的 API 加密很有特色.  
  
1>.一个完整的 API 调用,加壳后分为以下几个过程:  
  
1.通过 GetModuleHandleA 获得当前 API 所在的 DLL 基址.(Dll 名在加壳时写入)  
2.将 API 名称入栈(CALL +X),把 1 中取到的基址入栈(PUSH EAX)  
3.调用同一个 ANTI 函数(前面说的大忌.)  
4.GetProcAddress 获得 API 地址(这里是单独的函数,其中的 GetProcAddress 部分好像是自己写的,不是用的 API,这里也方便进行后续的模拟操作.并且这里也不排除有 ANTI.)  
5.将获取的地址写入到前面的执行入口附近,并几个字节,构造成  
代码:

```
push api
pushfd
inc \[esp+8\] ;6 字节的调用有这句,5 字节的则没有
popfd
retn

```

如此一来,避免了再次调用同一 API 时,执行前面 1,2,3,4 这些重复的操作,避免降低效率.无疑这是此壳的一个亮点.  
这里还有一种情况是针对 mov r32,api 这种情况的 API 调用.壳会构造如下结构:  
代码:

```
006D41A3    E8 0F000000     call 006D41B7                            ; 006D41B7
006D41A8    90              nop
006D41A9    90              nop
006D41AA    90              nop
006D41AB    90              nop
006D41AC    8B0D A8416D00   mov ecx,dword ptr \[6D41A8\]

```

中间的 4 个 NOP 是壳填入的 API 地址,通过下面的 mov r32,\[const\]操作,把地址放到寄存器中并返回.在执行前修改入口流程,直接改成 JMP 跳过中间的过程,跳到这个结构中,同样避免了上述所说的效率问题.  
  
2>.模拟  
SE 是为数不多的在 WIN7 下依然会模拟 API 的壳(我见过的唯一一个,也可能是我孤陋寡闻).当然在 WIN7 下模拟得比 XP 下要含蓄许多.所以调试 SE 可以的话还是在 WIN7 下调吧.省下不少事情.  
SE 的模拟我看到的有两处,第一处是对壳用到的函数以及用户函数进行模拟.第二处是对壳自己引入的几个 DLL 中的 API 进行模拟.  
第二处比较有意思,壳会把无壳试练品中用到的 DLL,都放到加壳后的程序里,并挑几个 API 列进来.时间有限,我没有一一印证,猜测这里可能与前面的获取 DLL 基址用到的 GetModuleHandleA 有关.如果壳不事先把所有 DLL 装载进去,那么 GetModuleHandleA 也就无法获取基址了.也可能跟系统有关系,避免特殊情况下的锁死.至于作者为什么如此构造导入表而不是用 LoadLibraryA,是否如此考虑,也不得而知了.  
通过跟踪得知,壳会获取 API,并通过一个函数来比较是否需要模拟,需要则返回模拟的地址,否则则返回真实地址.  
对 Shielden2.04 主程序而言,第一种模拟,壳的比较函数在  
代码:

```
0075F52C    E8 AFCFF9FF     call 006FC4E0                            ; 006FC4E0

```

执行完毕后,EAX 可能出现模拟 API 地址,而 EDI 中则为真实 API 地址.其他试练品中可能在 ESI 中出现真实 API 地址.  
所以可以 HOOK 下一条指令,跳到空地后,mov eax,edi,再跳回来即可.  
第二种模拟:  
代码:

```
0073F1BC    FF56 8B         call dword ptr \[esi-75\]
0073F1BF    90              nop
0073F1C0    8B06            mov eax,dword ptr \[esi\]
0073F1C2    5E              pop esi
0073F1C3    E8 39D1FBFF     call 006FC301                            ; 006FC301
0073F1C8    8BF8            mov edi,eax
0073F1CA    3B3E            cmp edi,dword ptr \[esi\]
0073F1CC    74 3F           je short 0073F20D                        ; 0073F20D

```

0073F1C3 返回 EAX 中可能是模拟 API,\[ESI\]中放的是真实 API,这里只需要把此处 NOP 或者把 006FC301 改成 RETN 即可.为避免壳其他地方模拟,显然后一种方法为佳.上述第一种模拟  
  
也可以试试直接把 CALL 改成 MOV EAX,EDI.  
我没测试,写这个的时候胡思乱想的.o(∩\_∩)o   
  
至此模拟和 ANTI 以及自校验都过了,可以脱壳了.  
  
3>.脱壳  
找个 OEP 附近的 API 下断,本例为 GetSystemTimeAsFileTime.断下后设置 EIP 为  
代码:

```
0050852E > $  E8 4D950000   call 00511A80                            ;  00511A80

```

第一个 API  
代码:

```
00511AB7    E8 0F635800     call 00A97DCB                            ; 00A97DCB

```

这里被执行过了,壳已经改成 PUSH API 了,所以要重开个 OD,再适当的时候下断,记录下原始代码,贴回去,避免被初始化而无法跨平台.  
看下这个 CALL.  
代码:

```
00A9806F    FF90 2A639484   call dword ptr \[eax+8494632A\]

```

这里实际调用的是  
代码:

```
006AD168->GetModuleHandleA 

```

在刚才 NOP 的地方写如下 PATCH  
代码:

```
00A97DCC    FF35 B4A01101   push dword ptr \[111A0B4\]                 ; kernel32.GetModuleHandleA
00A97DD2    8F05 68D16A00   pop dword ptr \[6AD168\]                   ; kernel32.GetModuleHandleA
00A97DD8    90              nop
00A97DD9    90              nop
00A97DDA    90              nop
00A97DDB    90              nop
00A97DDC    90              nop

```

这里  
代码:

```
00A97DCC    FF35 B4A01101   push dword ptr \[111A0B4\]                 ; kernel32.GetModuleHandleA

```

要记录 1 下  
  
代码:

```
00A98180    E8 EF86C2FF     call 006C0874                            ; 006C0874
00A98185    E8 DC67C2FF     call 006BE966                            ; 006BE966

```

这里就是上面说的 1>\_3 和 1>\_4 了.把 6C0874 改为 RETN 过掉 ANTI.  
006BE966 这里主要目的是获取 API 地址,而这个函数里可能有其他不和谐的东西,所以还是帮他重写下的好.  
改为:  
代码:

```
006BE966    8B0424          mov eax,dword ptr \[esp\]
006BE969    A3 7EE96B00     mov dword ptr \[6BE97E\],eax
006BE96E    83C4 04         add esp,4
006BE971    FF15 B0A01101   call dword ptr \[111A0B0\]                 ; kernel32.GetProcAddress
006BE977    FF35 7EE96B00   push dword ptr \[6BE97E\]
006BE97D    C3              retn

```

这样就是单纯的取 API 地址了.  
同样的  
代码:

```
006BE971    FF15 B0A01101   call dword ptr \[111A0B0\]                 ; kernel32.GetProcAddress

```

我们也要记录一下.  
在壳自己的导入表下面添加 GetModuleHandleA 以及 GetProcAddress.  
Dump.  
Importrec 重建壳原来的导入表.  
找到添加的两个 API 的地址,更改之前记录的两处即可.这样就借助壳自己的取 API 过程来完成了 API 填充了.  
  
4.资源修复:  
对资源处理比较优秀的有:  
1:PEP(HOOK 资源相关的许多 API 到壳自己的资源函数里)  
2:TTP(将资源块整个移动到高地址并修改各自索引)  
3:ZP(将图标,版本信息提出来单独造一个资源,在壳的 Shell 里把原始资源地址和大小写回 PE 头)  
  
SE 的资源保护仅仅从强度来说介于 PEP 和 TTP 之间.保留了完整的资源树,资源片则明文分散在内存中.但也同时避免了 PEP 的资源保护较低的效率.  
修复方法:可以从树中读地址,然后从内存 DUMP 贴回去.  
  
  
附一个 Shielden2.04 脱壳主程序.可以跑起来.加壳的时候会挂在 RtlAllocateHeap.可能是壳里 Create 了 Heap,某个地方再跑出来调戏人.明天还要上班,不看了,该睡觉觉了.  
  
[`u.115.com/file/f76dfecc3f#`](http://u.115.com/file/f76dfecc3f#)  
Shielden2.04\_uNpacked\_bY\_KisSy.rar