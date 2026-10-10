

# 编写 QQ 显 IP 外挂插件及原理分析(1)

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-216.htm`](https://www.pediy.com/kssd/pediy08/pediy8-216.htm)

这是本人第一次在这里发帖，所以请各位多多包涵~  
由于我是大二学生，文章是利用空余时间写的，所以将分几次上传。  
由于接触汇编时间不长，文章中的错误和不足恳请指正，谢谢  
  
由于文章是首先发在我 blog 上的，所以转到此没有经过很好的排版，请各位见谅，如果无法忍受排版问题可以去我 blog 看原文：  
[`www.csksoft.net/blog/post/QQPuginTips\_1.html`](http://www.csksoft.net/blog/post/QQPuginTips_1.html)  
\-------------------------------  
前一段时间研究了下 QQ 目前各种外挂的机理，包括著名的 coralQQ。也大致写了个一样的东西，不过马上要考试了，所以还是等暑假完善吧……反正 8 月份就要来学校。  
  
鉴于目前网上关于这方面的文章少之又少，一般能找到的应该就这下面 3 篇（由于可能涉及版权问题，我链接就不给出了），自己在开发中也走了不少弯路，所以特此把我的开发过程写了下来，给各位做个参考把  
  

引用:    
a.木子版显 IPQQ 的制作教程  
  
b.关于 QQ 外挂 DLL 的加载原理的分析  
  
c.明日帝国(sunwangme)写的我是这样来做破解 qq，做 QQ 外挂的系列  

  
  
在开始我的分析前我简要对上面这些资料作下评价，首先我觉得如果你也想写个类似的外挂插件，他们的文章你是必看的，而且特别是你想真的写出什么有用的东西的话，明日帝国得文章一定要看，而且必须看懂（呵呵，你去看看就明白我的意思）。对于木子版的教程应该说是最早“公开”的资料了，很多人都是看了这个教程开始写自己的外挂的。但是他通过直接修改 QQ 来做显 IP 补丁，可能引起的法律问题不说（如果你只是自娱自乐的话），他不能适应不同版本的 QQ，而且用户也不太能接受直接的修改，而且教程已经不能直接用于目前版本的 QQ 了。  
  
第二个教程是做外挂 DLL 插件必看的，但是他丝毫没涉及显示 IP 的问题，只是简单介绍了 DLL 注入的问题，并对 win9x 环境下手动加载 dll 到进程空间作了分析。但是目前 win9x 已逐渐退出舞台，所以一般只要使用 CreateRemoteThread 即可。  
  
好了，现在就开始我的分析过程。  
  
第一部分：  
  
1.1 主流的外挂插件如何获取 IP 和其他信息的？  
  
 也许你会认为他拦截了底层的 Socket 通讯？当然不至于，但这样肯定是最有效的办法。  
  
让我们换个思路：如果你现在需要和一个 QQ 好友传输文件或者进行语音聊天或者发送了图片或自定义表情。那么 QQ 必须知道对方的 IP 地址和端口信息，这样才能把数据传给对方。  
  
所以，很有可能 QQ 内部已经实现了获取 IP 地址和其他信息的相关函数了。的确如此。这也是木子版 QQ 教程里面提到的办法，调用 QQ 内部的函数。  
  
下面是截至 QQ 组件之一的 CQQApplication.dll 中的汇编代码：（建议先浏览木子版 QQ 的教程）  
  
027832C7     8B45 F0     mov eax,dword ptr ss:\[ebp-10\]  
027832CA     53             push  ebx  
027832CB     68 38558302     push CQQAppli.02835538    ; ASCII dwIP  
027832D0     50                     push eax  
027832D1     8B08                 mov ecx,dword ptr ds:\[eax\]  
027832D3     FF51 18            call dword ptr ds:\[ecx+18\]  
027832D6     8B45 F0           mov eax,dword ptr ss:\[ebp-10\]  
027832D9     53                   push ebx  
027832DA     68 40558302    push CQQAppli.02835540     ; ASCII wPort  
027832DF     50                    push eax  
027832E0     8B08                mov ecx,dword ptr ds:\[eax\]  
027832E2     FF51 14            call dword ptr ds:\[ecx+14\]  
CQQApplication.dll 通俗来说就是负责显示和实现 QQ 聊天窗口的模块。就是那些和 xxx 聊天中的窗口，所以这就是为什么要在其中寻找这样的代码的依据。  
  
从上面的汇编来看，显然是调用了 2 个 thiscall 规范的函数，也就是我们所说的 C++类成员函数。  
  
2 个成员函数的的大致形式是 this->Func(void \*ptr1,char \*cmd,DWORD \*ptr2);其中 cmd 就是上面 dwIP、wPort 这些字符串，而 ptr2 也很容易知道是函数返回值得存储指针。现在关键是要获取 this 指针，也就是 ecx 寄存器的数据和 ptr1 这个神秘指针的数据。  
  
如果你有兴趣反汇编 CoralQQ 中相关的代码，也会发现与上面类似的调用部分。  
  
这里暂时不深入这些函数的作用和那个 cmd 指针的细节，我们先来研究如何获取 this 指针和 ptr2 吧。  
  
注意  
  
027832DF     50                    push eax  
027832E0     8B08                mov ecx,dword ptr ds:\[eax\]  
  
这 2 段代码，也就是说 ptr2 获取了，那么 this 指针也可以得到。所以现在一切的关键就是找出 ptr2 的来历。这样我们就能很轻松的实现显示 ip 了。  
  
2.2 神秘的 ptr2 指针  
  
为了能更快的说明问题，这里就不厚道的引用 CoralQQ.dll 的汇编了~  
  
0056D97F     51                             push ecx  
0056D980     52                             push edx  
0056D981     50                             push eax  
0056D982     FF15 38AB5A00        call dword ptr ds:\[5AAB38\]  ; BasicCtr.GetFriendQQData  
0056D988     8B4424 14                 mov eax,dword ptr ss:\[esp+14\]  
0056D98C     83C4 0C                   add esp,0C  
0056D98F     3BC3                       cmp eax,ebx  
0056D991     0F84 03020000       je CoralQQ.0056DB9A  
0056D997     57                             push edi  
0056D998     895C24 14                mov dword ptr ss:\[esp+14\],ebx  
0056D99C     8D5424 14              lea edx,dword ptr ss:\[esp+14\]  
0056D9A0     52                             push edx  
0056D9A1     68 50BB5900           push CoralQQ.0059BB50  
0056D9A6     C64424 30 01           mov byte ptr ss:\[esp+30\],1  
0056D9AB     8B08                           mov ecx,dword ptr ds:\[eax\]  
0056D9AD     68 C8BA5900        push CoralQQ.0059BAC8   ; ASCII QQUSER\_DYNAMIC\_DATA  
0056D9B2     50                            push eax  
0056D9B3     8B41 54                 mov eax,dword ptr ds:\[ecx+54\]  
0056D9B6     FFD0                    call eax  
0056D9B8     8B4424 14             mov eax,dword ptr ss:\[esp+14\]  
0056D9BC     3BC3                     cmp eax,ebx  
0056D9BE     0F84 F9000000       je CoralQQ.0056DABD  
0056D9C4     8B08                       mov ecx,dword ptr ds:\[eax\]  
0056D9C6     8D5424 1C             lea edx,dword ptr ss:\[esp+1C\]  
0056D9CA     52                       push edx  
0056D9CB     68 ACA15900          push CoralQQ.0059A1AC       ; ASCII wProcotol  
0056D9D0     50                             push eax  
0056D9D1     8B41 30                  mov eax,dword ptr ds:\[ecx+30\]  
0056D9D4     FFD0                      call eax  
0056D9D6     8B4424 14              mov eax,dword ptr ss:\[esp+14\]  
0056D9DA     8B08                      mov ecx,dword ptr ds:\[eax\]  
0056D9DC     8D5424 10            lea edx,dword ptr ss:\[esp+10\]  
0056D9E0     52                          push edx  
0056D9E1     68 94A15900         push CoralQQ.0059A194    ; ASCII dwRecentIP  
0056D9E6     50                         push eax  
0056D9E7     8B41 34               mov eax,dword ptr ds:\[ecx+34\]  
0056D9EA     FFD0                   call eax  
  
以上代码正式 coralQQ 4.5 版获取 IP 信息的片断。我们只需要关注上面的 0056D982 和 0056D9B6 地址的 2 个调用函数。  
  
为什么这样说了，先看下面获取 dwRecentIP 数据的代码，它和上面提到的那个成员函数是属于一个类的（这里没提供出完整代码，你可以自己验证下:-P)。那么这里的 this 指针从哪里来呢？  
  
0056D9DA     8B08                      mov ecx,dword ptr ds:\[eax\]  
  
0056D9D6     8B4424 14              mov eax,dword ptr ss:\[esp+14\]  
  
按照 thiscall 规范，ecx 就保存了 this 指针，上面代码说明 ecx 是来自\[esp+14\]的，我们在往上看：  
  
0056D99C     8D5424 14              lea edx,dword ptr ss:\[esp+14\]  
0056D9A0     52                             push edx  
0056D9A1     68 50BB5900           push CoralQQ.0059BB50  
0056D9A6     C64424 30 01           mov byte ptr ss:\[esp+30\],1  
0056D9AB     8B08                           mov ecx,dword ptr ds:\[eax\]  
0056D9AD     68 C8BA5900        push CoralQQ.0059BAC8   ; ASCII QQUSER\_DYNAMIC\_DATA  
0056D9B2     50                            push eax  
0056D9B3     8B41 54                 mov eax,dword ptr ds:\[ecx+54\]  
0056D9B6     FFD0                    call eax  
  
看到么 0056D99C lea edx,dword ptr ss:\[esp+14\]！！  
  
也就是说 this 指针和 ptr2 时由这个函数获得的，我们暂时以它的一个参数命名：QQUSER\_DYNAMIC\_DATA。  
  
但不幸的是，这个函数同样也是 thiscall 调用规范的，也就说也是需要得到 this 指针……不过不慌：  
  
0056D97F     51                             push ecx  
0056D980     52                             push edx  
0056D981     50                             push eax  
0056D982     FF15 38AB5A00        call dword ptr ds:\[5AAB38\]  ; BasicCtr.GetFriendQQData  
0056D988     8B4424 14                 mov eax,dword ptr ss:\[esp+14\]  
0056D9AB     8B08                           mov ecx,dword ptr ds:\[eax\]  
  
注意上面 2 段代码，QQUSER\_DYNAMIC\_DATA 函数的 this 指针最终是\[esp+14\]，而 esp+14 的数据是  
  
0056D97F     51                             push ecx  
这段代码压入的。所幸的是 GetFriendQQData 是个导出函数（位于 BasicCtrDll.dll），我们看看他的申明：  
  
int GetFriendQQData(struct IQQCore \*,unsigned long,struct IQQData \* \*);  
  
上面这个 push ecx 实际上是压入了参数 struct IQQData \* \*。  
  
所以现在的只要获得 struct IQQCore \*,和第二个神秘参数的含义就能实现显示 IP 的功能了。  
  
\-----------------------------  
下回将说明 IQQCore 的具体含义和如何去获得，以及第二的神秘参数的作用，以及 IQQData 的含义。同时具体介绍如何编写一个具有实际意义的外挂 dll 插件 

上一篇地址:[`www.csksoft.net/blog/post/QQPuginTips\_1.html`](http://www.csksoft.net/blog/post/QQPuginTips_1.html)  
\--------------------------------------------------------  
写教程是需要激情的，所以趁激情还在继续写……  
  
发现自己没有把文章组织好，可能一开始看我写得会觉得头大，没事，最后具体讲编写插件时会好的……抱歉  
  
上次最后说到了从调试的角度来看，如果要调用显示 IP 等信息的 QQ 内部函数，关键问题就是通过调用  
  
1.3 IQQCore 和 Uin  
  
int GetFriendQQData(struct IQQCore \*,unsigned long,struct IQQData \* \*);  
这个函数，从而获得那个 struct IQQData \*指针。但问题就是要调用这个函数必须要提供 2 个参数:IQQCore \*和一个 unsigned long(DWORD)的神秘数据。  
  
在开始正式分析前请各位思考下，如果要你编写一个能显示好友 IP 的函数，你需要先知道什么呢？  
  
至少需要知道要去获取哪个 QQ 好友吧。这个肯定是必须的，否则函数就没有执行的意义了  
  
现在我们跟踪下上面这个 GetFriendQQData 函数。在其入口点下断点。然后小心的把鼠标移动到 QQ 好友列表中某个头像上（需要使用 CoralQQ……有点不厚道）。这时候应该程序就会被断下。  
  
因为正常理鼠标移至好友头像会显示信息卡片，CoralQQ 会在下面显示 IP 信息，所以按照上一篇文章反汇编的代码，GetFriendQQData 必然会调用。我们从堆栈里面找到这个 unsigned long 对应的数据：  
  
0x1A53836  
因为是 DWORD 数据，把它转化为 10 进制看看：27605046  
  
呵呵，这不是我的 QQ 号码么……的确，先前鼠标是移动在我的头像上了。  
  
所以可以猜测这个 unsigned long 就是好友的 QQ 号码。  
  
   
  
经过多次验证，的确如此。  
  
所以，这个神秘的 unsigned long 明确：他是要获取好友信息的号码,顺便补充下，这个 unsigned long 在 QQ 中可是有专门名字的：Uin  
  
接下来就是 struct IQQCore\*，我想他的作用从名字中应该就能猜出大概来。虽然具体他的结构我还没弄清，但可以肯定他好比是 QQ 程序内核信息的指针。而现在最关键的问题是如何去获得它。  
  
如果你研究过 BasicCtrlDll.dll 中导出的函数，你会发现几乎一半的函数的参数都由这个 IQQCore，比如：  
  
int GetCurrentStatus(struct IQQCore \*,int \*)  
  
int GetCurrentUin(struct IQQCore \*,unsigned long \*)  
  
int GetFriendStat(struct IQQCore \*,unsigned long)  
  
那么我们就拦截其中的一部分函数，看看提供给他们的这个 struct IQQCore \*参数具体是什么。  
  
如果你的确这样做了，那么会发现所有的函数，无论在什么时候，这个 struct IQQCore \*的值是确定的唯一的。如果你这个 IQQCore \*指针的地址区域下内存写入断点的话会发现 struct IQQCore \*实际上在 QQ 进行登录初始化时就创建了，以后就不再被修改。  
  
所以现在的问题就很简单了，我们可以暂时不用理会 struct IQQCore \*到底是什么，只要能获取到他就 ok  
  
1.4 如何获取 struct IQQCore \*？  
  
如果只是调试个 QQ，得到 struct IQQCore \*是非常容易的，但记住我们是要编写外挂。所以就是说我们要问：外挂如果通过程序来获取这个指针呢？  
  
最野蛮的办法：把外挂写成一个调试器，模拟手工调试的过程，获得这个 IQQCore \*。ok，我很佩服你这样做，这也是我原先的想法。虽然写这个一个调试器是很简单的，但是他基本上没有实际意义，因为不能应对各种版本的 QQ 程序，而且也就无法再使用 OD 这些调试器来调试你的程序了  
  
下面的方法要感谢明日帝国(sunwangme)写的教程了，可能一开始你看他教程会不知所云，但相信你现在去看他的文章就会很有感触。  
  
上面说过这个 IQQCore 是不会改变的，而且 BasicCtrlDll.dll 导出的那么多函数又偏偏要用到他，为什么不去拦截一个有 IQQCore \*参数的函数来获取这个 IQQCore \*呢？  
  
具体的做法我会在编写插件时说明。可能你会想拦截导出函数（也就是 API）不也是调试器作的事么？其实也有别的办法，这里就是用 API Hook 技术来实现的（建议先了解下 win32 的 hook 技术）。  
  
问题是 API SetWindowsHookEx 是不可能 hook 一个 API 的。所以这里要用比较“底层”的办法：  
  
一个导出函数的入口内存地址可以用 GetProcAddress API 获取，我们只要修改程序的代码，使得他在原先函数入口点执行时跳入我们的函数取执行，然后再跳转回来即可。（实际做法不是这样，在说明如何编写插件时我会说到）  
  
现在问题就是要在 BasicCtrlDll.dll 导出函数中选取一个比较理想的函数取拦截，得到 IQQCore  
  
要拦截的函数应该具有如下特点：  
  
函数形参简单，最好只有 IQQCore \*一个参数  
函数能尽早被调用，这样能及早的获取 IQQCore \*  
函数不能是 thiscall 规范的，也就是说函数必须是全局函数，不是一个成员函数  
为何要这些特点应该都能理解，我对最后一个做下说明，thiscall 规范的函数还需要一个类的 this 指针地址，这会给拦截造成一定麻烦（今后就会遇到这样的情况，以后再讨论）。  
  
最终我们选取的函数是 QQHelperDll.dll 中的一个导出函数：  
  
int IsLogin(struct IQQCore \*);  
很满足我们的要求，而且用 OD 跟踪发现，他在 QQ 登录后就不停的调用，太爽了……  
  
总结一下：要获取 struct IQQCore \*通过拦截 API 获取参数实现，我们拦截的函数选用了 IsLogin()。  
  
1.5 Uin 和 struct IQQData \*  
  
 现在还有 2 个问题要去研究，第一，在调用 GetFriendQQData 时候，Uin（就是那个 unsigned long 参数）如何确定呢？  
  
这个问题要等到我介绍插件编写时在讨论，但可以先做下暗示，我们编写的插件是需要在打开和好友聊天的对话框以及将鼠标移动到好友头像上时，显示对方的 IP 信息，就第一个情况:QQ 的聊天对话框里面不就有好友的 QQ 号码(Uin)么？第二个情况：虽然探出的信息卡片没有号码，但可以猜测 QQ 也是先需要获取相关信息的。  
  
现在我们再说说这个 IQQData \*的作用，看看 BasicCtrlDll.dll 的一些导出函数：  
  
long GetQQDataBuf(struct IQQData \*,char const \*,class CString &)  
  
long GetQQDataStr(struct IQQData \*,char const \*,class CString &)  
  
其中需要 IQQData \*参数，那么我们看看这些函数有什么作用呢？以 GetQQDataStr 在 CoralQQ 中的使用情况为例：  
  
0056DAC4     57                       push edi  
0056DAC5     68 DCBA5900      push CoralQQ.0059BADC   ; ASCII NAME  
0056DACA     52                       push edx  
0056DACB     FF15 44AB5A00  call dword ptr ds:\[5AAB44\]    ; BasicCtr.GetQQDataStr  
0056DAE5     8B4424 24           mov eax,dword ptr ss:\[esp+24\]  
0056DAE9     8D56 34               lea edx,dword ptr ds:\[esi+34\]  
0056DAEC     52                       push edx  
0056DAED     68 ECBA5900      push CoralQQ.0059BAEC   ; ASCII REMARK\_REALNAME  
0056DAF2     50                       push eax  
0056DAF3     FF15 44AB5A00  call dword ptr ds:\[5AAB44\]    ; BasicCtr.GetQQDataStr  
0056DAF9     8B5424 30           mov edx,dword ptr ss:\[esp+30\]  
0056DAFD     8D4E 38            lea ecx,dword ptr ds:\[esi+38\]  
0056DB00     51                       push ecx  
0056DB01     68 FCBA5900      push CoralQQ.0059BAFC    ; ASCII COUNTRY  
0056DB06     52                      push edx  
0056DB07     FF15 44AB5A00    call dword ptr ds:\[5AAB44\]  ; BasicCtr.GetQQDataStr  
0056DB0D     8B4C24 3C          mov ecx,dword ptr ss:\[esp+3C\]  
0056DB11     8D46 3C              lea eax,dword ptr ds:\[esi+3C\]  
0056DB14     50                       push eax  
0056DB15     68 04BB5900       push CoralQQ.0059BB04    ; ASCII PROVINCE  
0056DB1A     51                   push ecx  
0056DB1B     FF15 44AB5A00   call dword ptr ds:\[5AAB44\]  ; BasicCtr.GetQQDataStr  
这些代码是在获取 IP 信息后出现的，猜猜在做什么呢？NAME，COUNTRY，PROVINCE 这些词汇来看，应该是在获取当前要显示 ip 好友的名字、国籍、省份。而具体的执行函数就是 GetQQDataStr。  
  
再结合这个函数的参数来看，IQQData \*,很有可能就是存放着一个用户相关信息的结构  
  
事实也是如此的，到目前为止编写 QQ 外挂插件的条件已经具备 

上一篇地址:[`www.csksoft.net/blog/post/QQ...ips\_2.html.html`](http://www.csksoft.net/blog/post/QQPuginTips_2.html.html)  
前面说了一大堆的原理，现在终于能具体介绍插件的编写了。  
  
2 编写 QQ 显外挂插件  
  
2.1 插件存在形式  
  
这里说的插件是编写一个外部的 dll 文件，至于为何要用 dll 我就不想多说了。QQ 自身不支持插件特性，所以做出显 ip 外挂的可能无非有 3  
  
直接修改 QQ 本身的程序文件->不具备版本无关性，容易引发法律纠纷  
使用外部 exe 进程->必须采用调试进程的办法，效率低且浪费系统资源  
编写 dll 文件让 QQ 加载->效率高，稳定  
好了，现在就不讨论前面 2 种情况了。  
  
2.2 如何加载 dll  
  
dll 文件是不可能自己执行的，必须要又调用它的程序主动去加载。这应该是常识。那么如何让 qq.exe 去“主动”加载我们的插件呢？  
  
同样采用外部插件的 coralQQ，你会发现实际上它包含 2 个文件  
  
coralQQ.exe 和 coralqq.dll  
  
而平时点击第一个文件就是用来加载珊瑚虫版 QQ 的，但你千万别认为这个 exe 文件就是外挂的核心，核心是那个 dll。  
  
CoralQQ.exe 实际上只做了一件事：加载 qq.exe，然后把 coralqq.dll 注入到 qq.exe 的进程中去。  
  
关于如何实现该方法我不多说了，可以参考网上关于珊瑚虫外挂原理的文章（当然实际上它没有介绍原理……）  
  
将 dll 文件注入到程序的方法有 2  
  
1.首先通过 CreateProcess 创建进程，然后模拟 windows 加载一个 dll 的全过程，在加载程序主体  
  
2.采用远程线程植入技术，即使用 CreateRemoteThread  
  
我们采用后者。  
  
2.3 开始具体编写  
  
2.3.1 插件加载程序  
  
编写一个用于加载我们插件 dll 的 exe，好比 coralQQ.exe  
  
其流程伪代码：  
  
int WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPTSTR    lpCmdLine,int nCmdShow){  
  
pinfo=CreateProcess(<加载 qq.exe> );  
  
 if (hProcess){  
  HANDLE processHandle;  
  DWORD Exitcode;  
  processHandle=pinfo.hProcess;  
  HANDLE hThread;  
  char    szLibPath\[MAX\_PATH\];    
  void\*   pLibRemote;  
  DWORD   hLibModule;  
  HMODULE hKernel32 = GetModuleHandle(TEXT(Kernel32));  
  strcpy(szLibPath,<插件名称> );  
  
  pLibRemote = VirtualAllocEx( processHandle, NULL, sizeof(szLibPath),MEM\_COMMIT, PAGE\_READWRITE );  
  WriteProcessMemory( processHandle, pLibRemote, (void\*)szLibPath,sizeof(szLibPath), NULL );  
  hThread = CreateRemoteThread( processHandle, NULL, 0,(LPTHREAD\_START\_ROUTINE)GetProcAddress( hKernel32,LoadLibraryA ),pLibRemote, 0, NULL );  
  WaitForSingleObject( hThread, INFINITE );  
  
  GetExitCodeThread( hThread, &hLibModule );  
  CloseHandle( hThread );  
  VirtualFreeEx( processHandle, pLibRemote, sizeof(szLibPath), MEM\_RELEASE );  
 }  
 else  
 {  
  MessageBox(NULL,TEXT(Err,Cannot Load qq.exe),TEXT(Err!),MB\_OK);  
 }}  
  
该部分的代码实际上是通过 CreateRemoteThread 将 LoadLibraryA(<插件 dll 文件名> );语句注入了 qq.exe 执行。这样就做到了 qq 主动加载我们插件的目的。  
  
2.3.2 编写具体插件  
  
这里部分参考了目子版 qq 的方法。我们拦截 CQQApplication.dll 中对 user32.dll SetForegroundWindow API 的调用来判断 QQ 聊天对话框弹出的时机，以便显示 IP 信息。   
  
主要的思路是：  
  
DllMain 函数:  
  
创建一个多线程（VC 环境推荐采用\_beginthread 这个 crt 函数，否则将无法在新创建的线程中调用 crt 函数)。新创建线程的函数名比如为 workproc，他将在我们 dll 插件被植入 qq.exe 后开始执行  
  
workproc 函数:  
  
主要的功能就是去修改 qq.exe 内部程序，使得我们能够拦截相关的函数，比如以前提到的获取 IQQCore 和 Uin 的函数，以及上面说的 SetForegroundWindow 函数。  
  
所谓 API 函数拦截前一篇文章已经说过了大致的方法，这里我们采用《windows 核心编程》中推荐的 ReplaceIATEntryInOneMod 函数。为了方便，我把函数代码给出：  
  
bool ReplaceIATEntryInOneMod(PCSTR pszCalleeModName,  
        PROC pfnCurrent, PROC pfnNew,HMODULE hmodCaller)  
{  
 ULONG ulSize;  
 PIMAGE\_IMPORT\_DESCRIPTOR pImportDesc = (PIMAGE\_IMPORT\_DESCRIPTOR)  
  ImageDirectoryEntryToData(hmodCaller,TRUE,IMAGE\_DIRECTORY\_ENTRY\_IMPORT,  
  &ulSize);  
 if (pImportDesc == NULL)  
 {  
  return false;  
 }  
  
 for (;pImportDesc->Name;pImportDesc++)  
 {  
  PSTR pszModName = (PSTR)  
   ((PBYTE) hmodCaller + pImportDesc->Name);  
  if (lstrcmpiA(pszModName,pszCalleeModName)==0)  
   break;  
 }  
  
 if (pImportDesc->Name == 0)  
 {  
  return false;  
 }  
 PIMAGE\_THUNK\_DATA pThunk = (PIMAGE\_THUNK\_DATA)  
  ((PBYTE)hmodCaller + pImportDesc->FirstThunk);  
 for (;pThunk->u1.Function;pThunk++)  
 {  
  PROC \*ppfn = (PROC \*)&pThunk->u1.Function;  
  BOOL fFound = (\*ppfn == pfnCurrent);  
  if (fFound)  
  {  
   HANDLE hProcess = GetCurrentProcess();  
   DWORD oldAttr;  
   VirtualProtectEx(hProcess, ppfn, sizeof(pfnNew), PAGE\_READWRITE, &oldAttr);  
   if (WriteProcessMemory(hProcess,ppfn,&pfnNew,sizeof(pfnNew),NULL)==TRUE)  
   {  
    VirtualProtectEx(hProcess, ppfn, sizeof(pfnNew), oldAttr, &oldAttr);  
    return true;  
   }  
   else  
   {  
    VirtualProtectEx(hProcess, ppfn, sizeof(pfnNew), oldAttr, &oldAttr);  
    return false;  
   }  
  
  }  
 }  
 return false;  
}  
  
函数的作用是将所用在 hmodCaller 句柄指向模块中对 pszCalleeModName 指向文件名 dll 中提供的入口地址在 pfnCurrent 的函数的调用，用 pfnNew 指向的函数去替换。也就是说原先要调用 pfnCurrent 函数的，现在就会执行 pfnNew 指向的函数。  
  
要拦截上面说的 SetForegroundWindow，就用如下代码：  
  
  OrgFuncProc = GetProcAddress( GetModuleHandleA(user32),SetForegroundWindow);  
  if (ReplaceIATEntryInOneMod(user32.dll,OrgFuncProc,(PROC)&OnQQWndShow,global\_hCQQAppModule))  
  {     
  
//替换成功  
    }  
  
其中 global\_hCQQAppModule 是通过 GetModuleHandle 获取的 CQQApplication.dll 文件在 qq.exe 中的句柄。上面代码调用成功后，以后 CQQApplication.dll 中代码要调用 SetForegroundWindow 时，实际上将执行我们编写的 OnQQWndShow 函数。  
  
现在我们看 OnQQWndShow 的申明：  
  
extern C  BOOL  APIENTRY OnQQWndShow(HWND hWnd);  
  
为什么要有 extern C 和 APIENTRY （也就是\_\_stdcall)前缀呢？因为原先的 SetForegroundWindow 是采用 stdcall 调用规范的，如果替换函数的调用规范不同，将引发程序崩溃。  
  
同时替换函数的参数也要和原函数一致。  
  
经过这样的替换后，以后每当新弹出一个 qq 聊天窗口，我们的 OnQQWndShow 就会被执行，在其中我们就可以负责获取 ip，并创建一个 edit 或者 static 窗口在 qq 聊天窗口上显示 ip 信息  
  
OnQQWndShow 函数:  
  
前面已经说了它的作用，具体流程如下：  
  
获取对应好友 QQ 号码  
  
获取对应的 IP 信息  
  
在 qq 聊天窗口，也就是函数 hWnd 参数指出的窗口中寻找窗口上的广告框，然后把它销毁，创建一个 edit 或者 static 窗口，显示 ip 信息。  
  
最后调用原先的 SetForegroundWindow 函数（这步很重要，否则正常的代码逻辑会破坏）  
  
\-----------------------  
好了，最近就写了这 3 篇，如果觉得还行的话就期待着接下来的后续把~  
有什么错误希望多多指正 

上一篇地址:[`www.csksoft.net/blog/post/QQPuginTips\_3.html`](http://www.csksoft.net/blog/post/QQPuginTips_3.html)  
上一篇写得有点凌乱了，所以把前面的内容再重写下....  
  
2.编写 QQ 显 IP 插件  
  
2.1 编写加载外挂 dll 的程序  
  
这里采用 VC 作为开发环境，我使用的是 VS2005  
  
首先说下大体的思路：  
  
这里仿造 coralQQ 一样，将真正的外挂代码写入 dll，然后编写一个 exe 文件去加载 qq.exe，并把我们写得 dll 注入。  
  
这一步在前一篇文章中已经给出了具体代码。  
  
2.2 编写插件主体  
  
感觉自己如果一涉及具体代码就回说不来话，所以这次就先谈思路，然后给出部分代码。  
  
dll 要做的事：  
  
将我们前面分析过的关键 API 挂钩，以便得到 IQQCore \*，Uin，以及在 qq 聊天窗口弹出时进行捕获以便显示 IP 信息。  
  
要做到上面的要求，首先就是要在 dll 加载后开始做 API hook 的工作。  
  
不过这件事情不能在 DllMain 里面写，推荐的做法是在 DllMain 里面创建一个新的线程，线程的执行函数我们设为：WorkerProc（VC 编写的话推荐用\_beginthread 创建这个线程，否则无法使用 CRT 函数），为什么不能直接在 DllMain 里面写我稍候分析  
  
然后就是在 WorkerProc 里面写入具体的 hook 代码。  
  
1.拦截 QQHelperDll.dll 中的 IsLogin 函数  
  
前面分析过了，IQQCore \*指针可以通过拦截这个 API 来获取。  
  
这里我们采用的是《windows 核心编程》中推荐的拦截 API hook 的办法(修改入口地址跳转相对他烦了些，而且我们这里只是想做 hook)：通过动态修改 API 调用者模块的 IAT 表，将原先 API 的入口地址替换成我们函数的。具体的替换代码我们就采用书中提供的了的 ReplaceIATEntryInOneMod（见前一篇）。  
  
现在要注意一个问题，我们用来替换原先 API 的函数必须和原函数采用同样的调用规范，这个 IsLogin 本身就是 cdecl 得，所以只要用个形参和返回值一样的函数取替换即可：  
  
给出实现本功能的代码：  
  
typedef int (\*OrgIsLogin)(DWORD ptrIQQCore);  
  
int PokeIsLogin(DWORD ptrIQQCore)  
{  
 global\_ptrIQQCore = ptrIQQCore;  
 return ((OrgIsLogin)(PROC)OrgIsLoginProc)(ptrIQQCore);  
}  
  
////下面代码在 WorkerProc 函数中///  
  
  OrgIsLoginProc = GetProcAddress( GetModuleHandleA(QQHelperDll.dll),?IsLogin@@YAHPAUIQQCore@@@Z);  
  if (ReplaceIATEntryInOneMod(QQHelperDll.dll,OrgIsLoginProc,(PROC)&PokeIsLogin,GetModuleHandleA(qq.exe)))  
  {  
   //替换成功  
  }  
  
//  
  
今后，只要 QQ.exe 调用 IsLogin，我们的 PokeIsLogin 函数就会调用，并把 IQQCore 纪录下来，注意替换的函数要最后去调用原函数，否则就会让程序崩溃。  
  
2.拦截 SetForegroundWindow API  
  
拦截该 API 是因为每次弹出新的 QQ 聊天窗口时，CQQApplication.dll 都会去调用它，以便把聊天窗口至于前景显示。所以拦截 CQQApplication.dll 对他的调用就能在由新的聊天窗口出现时调用我们插件的代码去显示 IP 信息，而不必傻傻的去写循环等待了  
  
这里就要注意 SetForegroundWindow 是采用 stdall 调用规范的，所以别忘在替换函数名前加上\_\_stdcall（或 WINAPI 宏）标记。  
  
该部分的替换代码如下：  
  
typedef BOOL  ( \_\_stdcall \*OrgSetForegrandWindow)(HWND hWnd);  
  
extern C  BOOL  APIENTRY OnQQWndShow(HWND hWnd)  
{  
  
bool trueResult = ((OrgSetForegrandWindow)(PROC)OrgFuncProc)(hWnd);  
 return trueResult;   
  
}  
  
//下面代码在 WorkerProc 函数或由其调用的函数中  
  
OrgFuncProc = GetProcAddress( GetModuleHandleA(user32),SetForegroundWindow);  
  if (ReplaceIATEntryInOneMod(user32.dll,OrgFuncProc,(PROC)&OnQQWndShow,global\_hCQQAppModule))  
  {  
//替换函数入口成功     
  
  }  
  
其中 global\_hCQQAppModule 是 CQQApplication.dll 在 qq.exe 中的模块句柄，可以调用 GetModuleHandleA(CQQApplication.dll)来实现。不过现在存在一个问题  
  
用 od 加载 qq.exe 就会发现实际上 CQQApplication.dll 并不是在 qq.exe 启动后加载的，而是在登录以后。所以如果我们的 dll 插件在被加载之后立刻调用 GetModuleHandleA 的话就会返回 NULL，那么之后的函数替换就不可能实现了。  
  
这里我用了个比较笨的办法，在替换 SetForegroundWindow 前编写循环不断的去掉用 GetModuleHandleA，直到返回非 null，而为了防止频率过快可以在循环中加上 Sleep 函数。（之前也尝试 loadlibrary 提前加载，但一直失败。还有中办法就是从前面的 IsLogin 来判断是否登录了 QQ。但没有试验过。总之如果你有比较好的办法也希望告诉我）  
  
这部分的代码如下：  
  
bool WaitforLogon()  
{  
 while(!bIsDllUnload)  
 {  
  
  global\_hCQQAppModule = GetModuleHandleA(CQQApplication.dll);  
  if (global\_hCQQAppModule) return true;  
  Sleep(300);  
 }  
 return false;  
}  
  
这就是为什么前面说过不能再 dllmain 里面作函数替换的工作了，否则会导致 dllmain 无法退出，从而会锁死整个 qq  
  
3 获取 Uin  
  
现在 IQQCore \*已经获得了，同时只要在 OnQQWndShow 中写入得到用户 IP 信息的代码，再使用 FindWindow 的方法把原先的广告去除，再创建个 Edit 或者 Static 来显示我们的数据即可。  
  
但之前还要得到 Uin，也就是对方好友的 QQ 号。  
  
其实这里有个很笨的办法：QQ 聊天对话框中有对方的号码的：比如“&heaven(27605046)（后面是个性签名）。的确可以用 FindWindow+GetWindowText 获取，然后得到括号里面的数据就好了，但是是否有更简单办法呢？有  
  
这里要感谢明日帝国(sunwangme)的教程，这里就是用他的方法了，具体原理还是大家去他 blog 看吧（google 一下）  
  
大体的方法是：在 QQ 准备显示聊天对话框时，上面“&heaven(27605046)（后面是个性签名）这段文字会调用位于 QQBaseClassInDll.dll 中的 CAllInOneStatusBar::SetUin(unsigned long);这个函数，其中参数就是我们要的 QQ 号了。而且可以保证 CQQApplication.dll 仅仅在要显示对话框前才会调用它。（再次感谢明日帝国）  
  
所以和上面一样，这次拦截 CAllInOneStatusBar::SetUin(unsigned long);  
  
OrgSetUINProc =   
  
GetProcAddress(GetModuleHandleA(QQBaseClassInDll.dll),?SetUin@CAllInOneStatusBar@@QAEX\_J@Z);  
  
  if (ReplaceIATEntryInOneMod(QQBaseClassInDll.dll,OrgSetUINProc,(PROC)&Poke\_GETUIDA,global\_hCQQAllInOne)){  
  
}  
  
今后就会先调用我们的 OrgSetUINProc，其中获取参数即可。  
  
不过要注意 2 个问题，第一这个 QQBaseClassInDll.dll 也不是一开始加载，更不是在登录后加载，而是在第一次要显示 QQ 聊天窗口才加载，不过比较幸运的是可以直接 LoadLibrary 把它先载过来。  
  
还有一个问题就是 SetUin 是 thiscall 规范的……哎，自己对 thiscall 还不是很了解，所以怕出错，那个 OrgSetUINProc 只能用 naked 调用规范了，同时要自己写汇编去完成细节的事：  
  
\_\_declspec( naked ) int Poke\_GETUIDA()  
{  
 \_asm  
 {  
   
   push ecx  
   mov ecx,\[esp+8\]  
   mov dwRecentUINA,ecx  
   pop ecx  
   jmp OrgSetUINProc  
 }  
}  
  
ok，这样一切都完成了  
  
3.编写获取 IP 信息的代码  
  
先整理下上面我们 3 个替换函数的执行顺序：  
  
PokeIsLogin：最好执行，且今后不断被调用  
  
Poke\_GETUIDA：在将要显示 QQ 聊天窗口时执行  
  
OnQQWndShow：在 Poke\_GETUIDA 之后运行。  
  
所以可以保证在 OnQQWndShow 中可以得到我们获取 IP 信息所有必要的参数了。  
  
那么就开始编写获取 IP 的代码：  
  
按照前面分析的，先用 BasicCtr.GetFriendQQData 得到当前好友的 IQQData \*：  
  
 DWORD MainHandle;  
 if (ptrBasicCtr\_GetFriendQQData==NULL || global\_ptrIQQCore==NULL) return 0;  
 \_asm  
 {  
    
  mov edx,dwID  
  mov eax,global\_ptrIQQCore  
  lea ecx,MainHandle  
  push ecx  
  push edx  
  push eax  
        
  call ptrBasicCtr\_GetFriendQQData                                   ; BasicCtr.GetFriendQQData  
  
  add esp,0xC  
  
  
 }  
  
其中 dwID 是 Uin，即用 DWORD 保存的 QQ 号码  
  
global\_ptrIQQCore 就是前面获得的 IQQCore \*  
  
ptrBasicCtr\_GetFriendQQData 是 GetFriendQQData 的入口地址，用 GetProcAddress 得到  
  
最终 MainHandle 将保存 IQQData \*，如果运行失败，会返回 null  
  
接下来就是通过 QQUSER\_DYNAMIC\_DATA（暂时命名）函数来得到动态信息类的指针：  
  
char \*szQQUSER\_DYNAMIC\_DATA = QQUSER\_DYNAMIC\_DATA;   
  
DWORD tmpInfo;  
 DWORD returnVal;  
 tmpInfo = 0xba863a1e;  
 if (mainHandle == NULL)  
 {  
  return NULL;  
 }  
 \_asm  
 {  
  mov eax,mainHandle  
  lea edx,returnVal  
  push edx  
     lea edx,tmpInfo  
  push edx  
  mov ecx,\[eax\]  
  push szQQUSER\_DYNAMIC\_DATA  
  push eax  
  call \[ecx+54h\]  
 }  
  
结果将保存在 returnVal 中，同样，如果执行错误 returnVal=NULL  
  
最后就是去获取 IP 地址了  
  
  
char \*szwProcotol = wProcotol;  
char \*szRecentip=dwRecentIP;  
char \*szwRecentPort=wRecentPort;  
char \*szdwC2CIP=dwC2CIP;  
char \*szwC2CPort=wC2CPort;  
char \*szdwIP=dwIP;  
char \*szwPort=wPort;  
  
bool GetDestIPInfo(DWORD ptrClassHandle,DWORD \*ptrDestIp,DWORD \*ptrDestPort)  
{  
  
   
 if (ptrClassHandle==NULL || ptrDestIp==NULL || ptrDestPort==NULL) return false;  
 //Using Std Info Buffer  
 \_asm   
 {  
  pushad  
  pushf   
  mov eax,ptrClassHandle  
  mov ecx,\[eax\]  
  mov edx,ptrDestIp  
  push edx  
  push szRecentip  
  push eax  
  mov eax,\[ecx+34h\]  
  call eax  
  
  mov eax,ptrClassHandle  
  mov ecx,\[eax\]  
  mov edx,ptrDestPort  
  push edx  
  push szwRecentPort  
  push eax  
  mov eax,\[ecx+30h\]  
  call eax  
  popf  
  popad    
 }  
 (\*ptrDestPort) &= 0xFFFF;  
 if ((\*ptrDestIp) != NULL && (\*ptrDestPort) != NULL) return true;   
   
 \_asm  
 {  
  pushad  
  pushf   
  mov eax,ptrClassHandle  
  mov ecx,\[eax\]  
  mov edx,ptrDestIp  
  push edx  
  push szdwC2CIP  
  push eax  
  mov eax,\[ecx+34h\]  
  call eax  
  mov eax,ptrClassHandle  
  mov ecx,\[eax\]  
  mov edx,ptrDestPort  
  push edx  
  push szwC2CPort  
  push eax  
  mov eax,\[ecx+30h\]  
  call eax  
  popf  
  popad    
 }  
 (\*ptrDestPort) &= 0xFFFF;  
 if ((\*ptrDestIp) != NULL && (\*ptrDestPort) != NULL) return true;   
  
 \_asm  
 {  
  pushad  
  pushf    
  mov eax,ptrClassHandle  
  mov ecx,\[eax\]  
  mov edx,ptrDestIp  
  push edx  
  push szdwIP  
  push eax  
  mov eax,\[ecx+34h\]  
  call eax  
  mov eax,ptrClassHandle  
  mov ecx,\[eax\]  
  mov edx,ptrDestPort  
  push edx  
  push szwPort  
  push eax  
  mov eax,\[ecx+30h\]  
  call eax  
  popf  
  popad  
 }  
 (\*ptrDestPort) &= 0xFFFF;  
 return ((\*ptrDestIp) != NULL && (\*ptrDestPort) != NULL);   
   
}  
  
ptrClassHandle 是前面得到的 returnVal，IP 地址的 DWORD 数据和 WORD 的 port 信息将保存在后 2 个参数指针的地址中。（代码借鉴了 CoralQQ）  
  
到目前为止已大功告成！其他的细节代码就请各位发挥吧  
  
4.扩展  
  
上面并没有介绍如何得到对方 QQ 版本的信息，其实和获取 IP 一样，你在 CQQApplication.dll 中找到 wProcotol 的字符串参考，就会发现它和获取 IP 的代码很像，直接抄下来用就是了  
  
CoralQQ 还能在鼠标移动到好友头像上弹出的信息卡片下方显示 IP，其实机制是一样的，这里就不说了，可以看看明日帝国的教程~  
  
好了，我的教程就到此为止，希望能对大家有用~有什么错误希望能立刻指正