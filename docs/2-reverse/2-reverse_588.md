

# NewPlay 4.01

> 原文：[`www.pediy.com/kssd/pediy05/pediy50381.htm`](https://www.pediy.com/kssd/pediy05/pediy50381.htm)

第一次发文章。很浅。不过还详细。仅供大家一笑。有错漏之处请指教为谢！   
  
去年（好象很遥远，其实是 2 个月前）我下载了一个多媒体播放软件 NewPlay 4.01，看看介绍好象不错，但 NewPlay 每次启动后就自动将系统设置为静音，菜单里的“静音”选项一点不起作用，而且一调节音量系统也自动没了声音。程序出了这么大的问题还放出来？给作者发封 e-mail 抱怨（当然写的很客气的啦）：   
  
“昨天我在上海热线的下载频道上看到您的软件（第一次）介绍，觉得功能比 winamp 好，所以下载一个试用。装好后启动就没了声音，发现 tray 里面的音量被静掉了，我没在意，消了静音，放了几个 avi、wma、mp3、realplay 文件后觉得还不错，随后我关掉了 NewPlay，在资源管理器双击一个 mp3 文件，它启动了 NewPlay 播放，却没声音，我向 tray 里一看，又被静音了。然后我消静音，能听见声音了，于是我调节 NewPlay 上的音量和平衡,无论动哪一个，都会立刻静音。我向您发了 e-mail 咨询。……”   
  
才半天时间工夫，就收到了作者的回复（好快）：   
“您好！   
    您的问题我们非常重视。……（嘿嘿，客气话）   
    我们之所以对你所提出的这个“静音问题”比较关注，是因为我们的一位国外用户也曾经提到过类似问题。看来问题是的确存在的。只是“静音问题”看来的确很奇怪。因为我们在很多不同的操作系统上都做过了非常仔细的测试，当然包括了不同版本的 Windows XP，只是都没有出现问题。但到目前为止，加上你，总共有两位用户向我们反应过，这说明有若干 XP 版本可能存在问题。……”   
  
好家伙，只有两个用户存在问题，我就是其中一个，怎么这么不幸!@#$%^&\*   
  
看来作者是没机会解决了（问题无法重复），很巧那几天我心情稍微好一点，就帮他看看吧。   
  
启动 Softice，加载主程序 NewPlay.exe，先从调节静音的菜单出招：   
bpx CheckMenuItem   
  
很快找到了这段子程序：   
:00435424 53                      push ebx   
:00435425 56                      push esi   
:00435426 57                      push edi   
:00435427 8BDA                    mov ebx, edx   
:00435429 8BF0                    mov esi, eax   
:0043542B 3A5E2C                  cmp bl, byte ptr \[esi+2C\]   
:0043542E 7442                    je 00435472   
:00435430 885E2C                  mov byte ptr \[esi+2C\], bl   
:00435433 8B7E58                  mov edi, dword ptr \[esi+58\]   
:00435436 85FF                    test edi, edi   
:00435438 7427                    je 00435461   
:0043543A F6462002                test \[esi+20\], 02   
:0043543E 7521                    jne 00435461   
:00435440 33C0                    xor eax, eax   
:00435442 8AC3                    mov al, bl   
:00435444 8B0485D4074A00          mov eax, dword ptr \[4\*eax+004A07D4\]   
:0043544B 83C800                  or eax, 00000000   
:0043544E 50                      push eax   
:0043544F 0FB74644                movzx eax, word ptr \[esi+44\]   
:00435453 50                      push eax   
:00435454 8BC7                    mov eax, edi   
:00435456 E871F4FFFF              call 004348CC   
:0043545B 50                      push eax   
  
\* Reference To: user32.CheckMenuItem, Ord:0000h   
                                  |   
:0043545C E86F16FDFF              Call 00406AD0   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00435438(C), :0043543E(C)   
|   
:00435461 84DB                    test bl, bl   
:00435463 740D                    je 00435472   
:00435465 807E2F00                cmp byte ptr \[esi+2F\], 00   
:00435469 7407                    je 00435472   
:0043546B 8BC6                    mov eax, esi   
:0043546D E86AFFFFFF              call 004353DC   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:0043542E(C), :00435463(C), :00435469(C)   
|   
:00435472 5F                      pop edi   
:00435473 5E                      pop esi   
:00435474 5B                      pop ebx   
:00435475 C3                      ret   
  
测试一下就知道，这段子程序的入口参数：DL=1 的话，就将“静音”菜单打个勾，DL=0 则将去掉勾。   
  
这的、是一段公用代码，对于“静音”选项来说，由 00490DBF 调用的：   
  
:00490DA9 833DD88E4B0001          cmp dword ptr \[004B8ED8\], 00000001   
:00490DB0 7514                    jne 00490DC6   
:00490DB2 A1808E4B00              mov eax, dword ptr \[004B8E80\]   
:00490DB7 8B80A4030000            mov eax, dword ptr \[eax+000003A4\]   
:00490DBD B201                    mov dl, 01   
:00490DBF E86046FAFF              call 00435424   
:00490DC4 5A                      pop edx   
:00490DC5 C3                      ret   
  
  
这里有个分支，在于那句比较代码：   
cmp dword ptr \[004B8ED8\], 00000001   
而   
:00490DBD B201                    mov dl, 01   
:00490DBF E86046FAFF              call 00435424   
就是用 DL=1 来调用菜单 check 的子程序了。   
如此看来，只要地址\[004B8ED8\]为 1 的话，菜单的“静音”选项就要打勾，为 0 的话就否，这是个 boolean 信号变量。   
下一步就是要找到为什么\[004B8ED8\]数值会为 1，这正是问题的核心所在。   
  
有了\[004B8ED8\]这个信号，就逆着往上追……   
  
:004778C7 83BDB4FEFFFF00          cmp dword ptr \[ebp+FFFFFEB4\], 00000000   
:004778CE 740B                    je 004778DB   
:004778D0 8B4510                  mov eax, dword ptr \[ebp+10\]   
:004778D3 C70001000000            mov dword ptr \[eax\], 00000001   
:004778D9 EB07                    jmp 004778E2   
  
动态追踪中发现，这里 eax=004B8ED8，正好是设置地址\[004B8ED8\]为 1 嘛，再看看前面的条件：   
cmp dword ptr \[ebp+FFFFFEB4\], 00000000   
看来是因为\[ebp+FFFFFEB4\]等于 0 的缘故了。看看\[ebp+FFFFFEB4\]的数值：0012F370。   
目标改变，需要追踪\[0012F370\]地址了……   
来招乾坤大挪移：bpm 12F370   
  
于是我们来到了   
  
:0047782E 33C0                    xor eax, eax   
:00477830 8945E4                  mov dword ptr \[ebp\-1C\], eax   
:00477833 C745E804000000          mov \[ebp\-18\], 00000004   
:0047783A 8D85B4FEFFFF            lea eax, dword ptr \[ebp+FFFFFEB4\]   
:00477840 8945EC                  mov dword ptr \[ebp\-14\], eax   
:00477843 6A00                    push 00000000   
:00477845 8D45D8                  lea eax, dword ptr \[ebp\-28\]   
:00477848 50                      push eax   
:00477849 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
:0047784C 8B4030                  mov eax, dword ptr \[eax+30\]   
:0047784F 50                      push eax   
  
\* Reference To: winmm.mixerGetControlDetailsA, Ord:0000h   
                                  |   
:00477850 E873F3FFFF              Call 00476BC8   
:00477855 85C0                    test eax, eax   
:00477857 0F8585000000            jne 004778E2   
:0047785D 817B0801000350          cmp dword ptr \[ebx+08\], 50030001   
:00477864 753B                    jne 004778A1   
:00477866 8B4518                  mov eax, dword ptr \[ebp+18\]   
:00477869 8338FF                  cmp dword ptr \[eax\], FFFFFFFF   
:0047786C 7533                    jne 004778A1   
:0047786E F7430C00000080          test \[ebx+0C\], 80000000   
:00477875 8B450C                  mov eax, dword ptr \[ebp+0C\]   
:00477878 0F9700                  seta byte ptr \[eax\]   
:0047787B 8B450C                  mov eax, dword ptr \[ebp+0C\]   
:0047787E 803800                  cmp byte ptr \[eax\], 00   
:00477881 755F                    jne 004778E2   
:00477883 8B4518                  mov eax, dword ptr \[ebp+18\]   
:00477886 8B95B4FEFFFF            mov edx, dword ptr \[ebp+FFFFFEB4\]   
:0047788C 8910                    mov dword ptr \[eax\], edx   
:0047788E 837DE001                cmp dword ptr \[ebp\-20\], 00000001   
:00477892 764E                    jbe 004778E2   
:00477894 8B4514                  mov eax, dword ptr \[ebp+14\]   
:00477897 8B95B8FEFFFF            mov edx, dword ptr \[ebp+FFFFFEB8\]   
:0047789D 8910                    mov dword ptr \[eax\], edx   
:0047789F EB41                    jmp 004778E2   
  
呵呵，程序调用了 win 美眉库的 mixerGetControlDetail 来取得当前音量设置，如果是静音，就将这个可爱的信号\[0012F370\]置 0。   
终于找到菜单的“静音”选项似乎看起来不起作用的原因了——其实还是有动作，不过因为设置失败，所以一直不能改变状态。   
所以事情还远远没有结束，我们还没找到为何设置取消“静音”无效的原因。   
  
很容易想到，程序可能会调用 win 美眉库的 mixerSetControlDetail 来设置音量，那就找找 mm 吧。   
bpx winmm.mixerSetControlDetail   
  
gogogo!   
:00477B94 6A00                    push 00000000   
:00477B96 8D45D4                  lea eax, dword ptr \[ebp\-2C\]   
:00477B99 50                      push eax   
:00477B9A 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
:00477B9D 8B4030                  mov eax, dword ptr \[eax+30\]   
:00477BA0 50                      push eax   
  
\* Reference To: winmm.mixerSetControlDetails, Ord:0000h   
                                  |   
:00477BA1 E852F0FFFF              Call 00476BF8   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00477AD9(C), :00477AEE(C), :00477AFD(C)   
|   
:00477BA6 46                      inc esi   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00477AC7(U)   
|   
:00477BA7 807DEF00                cmp byte ptr \[ebp\-11\], 00   
:00477BAB 7406                    je 00477BB3   
:00477BAD 807DEE00                cmp byte ptr \[ebp\-12\], 00   
:00477BB1 750F                    jne 00477BC2   
  
看来这是设置音量、静音的代码了。需要追踪一下函数参数，才会明白为什么设置取消静音无效了。   
参数颇为复杂，我是弄不懂的，看了老半天的 API 帮助，才知道那个 cbDetailSize 和 paDetail 指针还有点用，看看：   
cbDetailSize=4，paDetail=12F8EC，好啊，就守株待兔，看看\[12F8EC\]用了什么值设置音量。   
另外 cChannels=2，说明是设置了两个声道的。   
左声道=0，右声道=00000C78。   
奇怪，为什么作者不把右声道设置为 0 呢？在其它系统中音量调节既然是正常的，那只有一种解释：系统中有一个是主声道，可能是左、可能是右，只要将此主声道音量设置为 0 时，系统认为取消两个声道的静音了。那么，我的系统中的主声道应该是右。   
好了，改改参数试试吧。将右声道的值改为 0，运行之后，果然“静音”被取消掉了。   
  
再回溯调用程序，左声道\*paDetail 的值是从这里设置的：   
:00477B87 8B4508                  mov eax, dword ptr \[ebp+08\]   
:00477B8A 8985B4FEFFFF            mov dword ptr \[ebp+FFFFFEB4\], eax   
但找不到设置右声道的代码，看来是作者遗漏掉了。   
  
修正这个问题可以令 cCannel=1，不过这段代码设置音量也在用，此方法不行。那就将左右两声道同时都设置为 0，就万无一失了！   
  
好了，问题解决了，回信~~~~~~   
  
“您好！   
这个（静音）问题我已经解决了。从二进制代码分析颇为困难，费了两天时间，刚刚才弄出来。   
问题出在对静音的设置上。文件定位：NewPlay.exe。   
  
NewPlay.exe 里面有一个过程，类似这样（名字是我自己取的，和源程序中的名字不同。省略号代表其它参数）：   
procedure ChangeVolumn(mute: interger......);   
第一个参数是静音标志，当 mute=1 的时候就静音，mute=0 的时候就取消静音。   
这个过程的功能比较复杂，可以设置音量大小、平衡左右声道、是否静音。我们只关心静音问题，看看静音是如何设置的。   
……   
Channel:=2;   
……（设置其它参数，略）   
left:=mute;   
MIXSETCONTROLDETAILS(...);   
……   
非常明显，程序假设左声道（left）是主音道，设置主音道的 mute 后，次音道（右音道 right）被自动 mute，所以不用设置。   
不幸的是，我的系统右声道（right）才是主音道，所以对左声道设置 mute 对系统毫无影响。   
解决上面的问题就必须这样写：   
left:=mute;   
right:=mute;   
MIXSETCONTROLDETAILS(...);   
两个音道都设置 mute 值才万无一失。   
我改了 NewPlay.exe 文件，本想插一句 right:=mute;进去，无奈代码太紧凑，插不进，想改 Channel 值，好象这个值别的功能也在用，没办法，只好改成适应我的系统：   
right:=mute;   
MIXSETCONTROLDETAILS(...);   
改过的 NewPlay.exe 在我的系统中运行完全正常，但估计在你的系统中会出现静音问题：）   
我要吃晚饭了，饥寒交迫中。”   
  
这次收到了作者的感谢：   
"　没想到你能帮我们把这个困扰我们多时的问题给解决了。   
    实在是我自己的疏忽，写程序时太想当然了。现在依照你的思路   
问题已经解决。:-)   
    谢谢。"   
另外作者答应：   
“你可以永远得到 NewPlay 的注册码，即便是以后改变了注册方式。”   
  
以前破解别人的软件，收山后，现在总算做了一件好事。   
作者也很快推出了 4.02 升级版，正是目前上海热线上提供下载的版本。 

我的一次 Debug（续）：音量调节函数浅析   
  
这篇文字希望能回答一些朋友所询问的关于音量调节的细节问题。上篇文章中我只贴出了最关键的设置代码，也没有进一步解释。   
另外，宽河朋友跟踪一个类似出问题的软件后，也贴了一段代码。我看了后觉得有点眼熟耶~~~随后比较 NewPlay 4.01 相关代码，竟然一模一样：）这是怎么回事，我不太明白……   
解释代码是件困难的事情，还是用程序来描述吧。下面用了“类 Delphi”来描述，因为源程序可能是 Delphi 写的，但由于编译优化，有些只好用寄存器描述，goto 语句还不能避免。有些习惯还保留，如 16 进制还是用 h 后缀。不当的地方请多多指教为谢！   
最后，我在测试时发现，上篇似乎将左声道和右声道写反了:( 尽管这对找静音原因不是一个问题，却不希望误导，特此提出。   
  
\==================================================   
  
这个函数（过程）设置音量，包括静音。分析一下，可以知道参数一共有 5 个，栈中 push 了 3 个，寄存器中放了 2 个（eax, ecx）。下面要进一步解释和了解这些参数的意义。为了叙述方便，把栈参数称 param1、param2、param3，寄存器参数称 regeax、regecx。   
  
先把变量的相关资料总结在下面供查询：   
  
参数：   
regeax 为指针，通过它取一个变量作为参数调用 sub\_00477588   
regecx 为一个 32 位 integer，为一个正整数，或 FFFFFFFF。在 NewPlay 中总是 FFFFFFFF。   
param1 为静音与否 Mute（=1 静音，=0 取消静音）   
param2 是左声道值 leftValue   
param3 是右声道值 rightValue   
  
局部变量：   
\[ebp\-4\]=regeax。4pEax   
\[ebp\-8\]=regecx。8Ecx   
\[ebp\-9\]byte 型的 boolean。9bResult;函数的返回值   
\[ebp\-10h\]指针，通过 sub\_00477588 返回的值。10p   
\[ebp\-11\]byte 类型的 boolean，如果 leftValue=-1 就等于为 true，否则 0。11bNoRightValue   
\[ebp\-12\]byte 类型的 boolean，如果 Mute=-1 就等于为 true，否则 0。12bNoMute   
  
局部变量之 MIXERCONTROLDETAILS 结构   
\[ebp\-18\]=paDetails;（这个关键指针指向\[ebp\-150h\]）   
\[ebp\-1c\]=cbDetails;   
\[ebp\-20\]=hwndOwner or cMultipleItems;   
\[ebp\-24\]=cChannels;   
\[ebp\-28\]=dwControlID;   
\[ebp\-2c\]=cbStuct;   
\[ebp\-2c\]=&MIXERCONTROLDETAILS;   
\[ebp\-150\]=右声道数值(right)。或是否 mute(此时要 cChannels=1)   
\[ebp\-14c\]=左声道数值(left)   
  
\[ebp\-2c\]到\[ebp\-0D4\]为一个 32 位整数数组。d4IntArray\[1..2Ah\]。其中\[ebp\-0B8\]（B8int）是 cChannel 的默认值 2   
  
\==================================================   
  
下面是程序分析，先逐行把汇编写成“类 Delphi”语言，再把“类 Delphi”代码合并起来，脉络就清晰多了   
  
\* Referenced by a CALL at Addresses:   
|:004920CD  , :0049A35D  , :0049A3C5  , :0049A407  , :0049A48C   
|:0049A5E9  , :0049A745  , :0049CCFC   
|   
:00477A24 55                      push ebp   
:00477A25 8BEC                    mov ebp, esp   
:00477A27 81C4B0FEFFFF            add esp, FFFFFEB0   
  
上面三句大概是 Borland 的习惯吧。经过这样处理，栈具有一定的布局：   
\[ebp+8\]为从栈传的第一个参数的位置，\[ebp+c\]为栈传的第二个参数的位置，类推。   
\[ebp\-4\]为第一个局部变量的位置（如果这个局部变量是 32 位的话），类推。   
  
........——locVar2—locVar1—EBPvalue（EBP 指向这里）—RetAddr—1stParam—2ndParam——......栈顶（ESP 指向这里）   
                    局部变量区              EBP 值                          返回地址  参数区                                      栈顶   
  
  
:00477A2D 53                      push ebx   
:00477A2E 56                      push esi   
:00477A2F 57                      push edi   
压栈   
  
:00477A30 894DF8                  mov dword ptr \[ebp\-08\], ecx   
:00477A33 8945FC                  mov dword ptr \[ebp\-04\], eax   
:00477A36 C645F700                mov \[ebp\-09\], 00   
  
8Ecx:=ecx;   
4pEax:=eax;   
9bResult:=false;   
  
:00477A3A 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
:00477A3D 8B4028                  mov eax, dword ptr \[eax+28\]   
:00477A40 E843FBFFFF              call 00477588   
:00477A45 8945F0                  mov dword ptr \[ebp\-10\], eax   
:00477A48 837DF000                cmp dword ptr \[ebp\-10\], 00000000   
:00477A4C 0F848D020000            je 00477CDF   
  
10p:=sub\_00477588( ^(4pEax+28h) );   
if 10p=nil then   
  goto lblExit;   
  
:00477A52 837DF8FF                cmp dword ptr \[ebp\-08\], FFFFFFFF   
:00477A56 7520                    jne 00477A78   
:00477A58 8B45F0                  mov eax, dword ptr \[ebp\-10\]   
:00477A5B 8BB8B0000000            mov edi, dword ptr \[eax+000000B0\]   
:00477A61 8B45F0                  mov eax, dword ptr \[ebp\-10\]   
:00477A64 57                      push edi   
:00477A65 8D7008                  lea esi, dword ptr \[eax+08\]   
:00477A68 8DBD2CFFFFFF            lea edi, dword ptr \[ebp+FFFFFF2C\]   
:00477A6E B92A000000              mov ecx, 0000002A   
:00477A73 F3                      repz   
:00477A74 A5                      movsd   
:00477A75 5F                      pop edi   
:00477A76 EB31                    jmp 00477AA9   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00477A56(C)   
|   
:00477A78 8B45F0                  mov eax, dword ptr \[ebp\-10\]   
:00477A7B 8B80B4000000            mov eax, dword ptr \[eax+000000B4\]   
:00477A81 8B55F8                  mov edx, dword ptr \[ebp\-08\]   
:00477A84 E817F9FFFF              call 004773A0   
:00477A89 85C0                    test eax, eax   
:00477A8B 741A                    je 00477AA7   
:00477A8D 8BB8B0000000            mov edi, dword ptr \[eax+000000B0\]   
:00477A93 57                      push edi   
:00477A94 8D7008                  lea esi, dword ptr \[eax+08\]   
:00477A97 8DBD2CFFFFFF            lea edi, dword ptr \[ebp+FFFFFF2C\]   
:00477A9D B92A000000              mov ecx, 0000002A   
:00477AA2 F3                      repz   
:00477AA3 A5                      movsd   
:00477AA4 5F                      pop edi   
:00477AA5 EB02                    jmp 00477AA9   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00477A8B(C)   
|   
:00477AA7 33FF                    xor edi, edi   
  
if 8Ecx=FFFFFFFFh then   
begin   
  for i:=1 to 2Ah do   
    d4IntArray\[i\]:=^((10p+7)+i);   
  edi:=^(10p+B0h);   
end   
else   
begin   
  eax:=sub\_004773A0( ^(10p+B4h),8Ecx );   
  if eax<>0 then   
  begin   
    for i:=1 to 2Ah do   
      d4IntArray\[i\]:=^((eax+7)+i));   
    edi:=^(eax+B0h);   
  end   
  else   
    edi:=0;   
end;   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00477A76(U), :00477AA5(U)   
|   
:00477AA9 85FF                    test edi, edi   
:00477AAB 0F842E020000            je 00477CDF   
  
if edi\=0 then   
  goto lblExit;   
  
:00477AB1 33F6                    xor esi, esi   
:00477AB3 837D10FF                cmp dword ptr \[ebp+10\], FFFFFFFF   
:00477AB7 0F9445EF                sete byte ptr \[ebp\-11\]   
:00477ABB 837D08FF                cmp dword ptr \[ebp+08\], FFFFFFFF   
:00477ABF 0F9445EE                sete byte ptr \[ebp\-12\]   
:00477AC3 C645F701                mov \[ebp\-09\], 01   
:00477AC7 E9DB000000              jmp 00477BA7   
  
esi:=0;   
if rightValue=FFFFFFFFh then   
  11bNoRightValue:=true;   
if Mute=FFFFFFFFh then   
  12bNoMute:=true;   
9bResult:=true;   
goto lbl00477BA7;   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00477BBC(C)   
|   
:00477ACC 8BD6                    mov edx, esi   
:00477ACE 8BC7                    mov eax, edi   
:00477AD0 E8EBF6FFFF              call 004771C0   
:00477AD5 8BD8                    mov ebx, eax   
:00477AD7 85DB                    test ebx, ebx   
:00477AD9 0F84C7000000            je 00477BA6   
  
lbl00477AAC:   
ebx:=sub\_004771C0(esi,edi);//返回值 ebx 是个指针，\[ebx+4\]是 dwControlID，\[ebx+0Ch\]为 cChannel    
if ebx<>0 then   
begin   
  
:00477ADF 8B4308                  mov eax, dword ptr \[ebx+08\]   
:00477AE2 3D01000350              cmp eax, 50030001   
:00477AE7 740B                    je 00477AF4   
:00477AE9 3D02000120              cmp eax, 20010002   
:00477AEE 0F85B2000000            jne 00477BA6   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00477AE7(C)   
|   
:00477AF4 8B430C                  mov eax, dword ptr \[ebx+0C\]   
:00477AF7 83E002                  and eax, 00000002   
:00477AFA 83F802                  cmp eax, 00000002   
:00477AFD 0F84A3000000            je 00477BA6   
:00477B03 C745D418000000          mov \[ebp\-2C\], 00000018   
:00477B0A 8B4304                  mov eax, dword ptr \[ebx+04\]   
:00477B0D 8945D8                  mov dword ptr \[ebp\-28\], eax   
:00477B10 F7430C01000000          test \[ebx+0C\], 00000001   
:00477B17 7609                    jbe 00477B22   
:00477B19 C745DC01000000          mov \[ebp\-24\], 00000001   
:00477B20 EB09                    jmp 00477B2B   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00477B17(C)   
|   
:00477B22 8B8548FFFFFF            mov eax, dword ptr \[ebp+FFFFFF48\]   
:00477B28 8945DC                  mov dword ptr \[ebp\-24\], eax   
  
  case ^(ebx+8) of   
    20010002,5003001:   
    begin   
      eax:=^(ebx+Ch);   
      if eax\=00000002h then   
        goto lbl00477BA6;   
      cbStuct:=18h;   
      dwControlID:=^(ebx+4h);   
      if ^(ebx+Ch)>1 then   
        cChannels:=1   
      else   
        cChannels:=B8int;//=2    
      end;   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00477B20(U)   
|   
:00477B2B 33C0                    xor eax, eax   
:00477B2D 8945E0                  mov dword ptr \[ebp\-20\], eax   
:00477B30 C745E404000000          mov \[ebp\-1C\], 00000004   
:00477B37 8D85B0FEFFFF            lea eax, dword ptr \[ebp+FFFFFEB0\]   
:00477B3D 8945E8                  mov dword ptr \[ebp\-18\], eax   
  
      hwndOwner:=0;   
      cbDetails:=4;   
      paDetails:=@\[ebp\-150h\];   
  
:00477B40 817B0801000350          cmp dword ptr \[ebx+08\], 50030001   
:00477B47 752F                    jne 00477B78   
:00477B49 807DEF00                cmp byte ptr \[ebp\-11\], 00   
:00477B4D 7529                    jne 00477B78   
:00477B4F 8B4510                  mov eax, dword ptr \[ebp+10\]   
:00477B52 8985B0FEFFFF            mov dword ptr \[ebp+FFFFFEB0\], eax   
:00477B58 837D0CFF                cmp dword ptr \[ebp+0C\], FFFFFFFF   
:00477B5C 750B                    jne 00477B69   
:00477B5E 8B4510                  mov eax, dword ptr \[ebp+10\]   
:00477B61 8985B4FEFFFF            mov dword ptr \[ebp+FFFFFEB4\], eax   
:00477B67 EB09                    jmp 00477B72   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00477B5C(C)   
|   
:00477B69 8B450C                  mov eax, dword ptr \[ebp+0C\]   
:00477B6C 8985B4FEFFFF            mov dword ptr \[ebp+FFFFFEB4\], eax   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00477B67(U)   
|   
:00477B72 C645EF01                mov \[ebp\-11\], 01   
:00477B76 EB1C                    jmp 00477B94   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00477B47(C), :00477B4D(C)   
|   
:00477B78 817B0802000120          cmp dword ptr \[ebx+08\], 20010002   
:00477B7F 7513                    jne 00477B94   
:00477B81 807DEE00                cmp byte ptr \[ebp\-12\], 00   
:00477B85 750D                    jne 00477B94   
:00477B87 8B4508                  mov eax, dword ptr \[ebp+08\]   
:00477B8A 8985B4FEFFFF            mov dword ptr \[ebp+FFFFFEB0\], eax   
:00477B90 C645EE01                mov \[ebp\-12\], 01   
  
      if \[ebx+8\]=50030001 then//50030001 设置音量    
      begin   
        if not 11bNoRightValue then//如果右声道数据有定义(<>FFFFFFFFh)   
        begin   
          right:=rightValue;   
          if leftValue=FFFFFFFFh then   
            left:=rihtValue;   
          else   
            left:=leftValue;   
          11bNoRightValue:=true;   
        end   
      end   
      else   
        begin   
        if \[ebx+8\]=20010002 then//20010002 设置静音    
        begin   
          if not 12bNoMute then   
          begin   
            right:=Mute;   
            12bNoMute:=true;   
          end;   
        end;   
      end;   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00477B76(U), :00477B7F(C), :00477B85(C)   
|   
:00477B94 6A00                    push 00000000   
:00477B96 8D45D4                  lea eax, dword ptr \[ebp\-2C\]   
:00477B99 50                      push eax   
:00477B9A 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
:00477B9D 8B4030                  mov eax, dword ptr \[eax+30\]   
:00477BA0 50                      push eax   
  
\* Reference To: winmm.mixerSetControlDetails, Ord:0000h   
                                  |   
:00477BA1 E852F0FFFF              Call 00476BF8   
  
      mixerSetControlDetails(^(4pEax+30h),@MIXERCONTROLDETAILS,0);   
    end;   
  else   
    begin   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00477AD9(C), :00477AEE(C), :00477AFD(C)   
|   
:00477BA6 46                      inc esi   
  
lbl00477BA6:   
      esi:=esi+1;   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00477AC7(U)   
|   
:00477BA7 807DEF00                cmp byte ptr \[ebp\-11\], 00   
:00477BAB 7406                    je 00477BB3   
:00477BAD 807DEE00                cmp byte ptr \[ebp\-12\], 00   
:00477BB1 750F                    jne 00477BC2   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00477BAB(C)   
|   
:00477BB3 8BC7                    mov eax, edi   
:00477BB5 E812F6FFFF              call 004771CC   
:00477BBA 3BF0                    cmp esi, eax   
:00477BBC 0F8C0AFFFFFF            jl 00477ACC   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00477BB1(C)   
|   
:00477BC2 807DEE00                cmp byte ptr \[ebp\-12\], 00   
:00477BC6 0F8513010000            jne 00477CDF   
:00477BCC 837DF8FF                cmp dword ptr \[ebp\-08\], FFFFFFFF   
:00477BD0 0F8409010000            je 00477CDF   
  
lbl00477BA7:   
      if 11NoRightValue then   
        if not 12bNoMute then   
          goto lb00477BC2;   
      if esi<sub\_004771CC(edi) then   
        goto lbl00477ACC;   
lbl00477BC2:   
      if not 12bNoMute then   
        goto lblExit;   
      if 8Ecx=FFFFFFFFh then   
        goto lblExit;   
  
:00477BD6 8B45F0                  mov eax, dword ptr \[ebp\-10\]   
:00477BD9 8BB8B0000000            mov edi, dword ptr \[eax+000000B0\]   
:00477BDF 8B45F0                  mov eax, dword ptr \[ebp\-10\]   
:00477BE2 57                      push edi   
:00477BE3 8D7008                  lea esi, dword ptr \[eax+08\]   
:00477BE6 8DBD2CFFFFFF            lea edi, dword ptr \[ebp+FFFFFF2C\]   
:00477BEC B92A000000              mov ecx, 0000002A   
:00477BF1 F3                      repz   
:00477BF2 A5                      movsd   
:00477BF3 5F                      pop edi   
:00477BF4 85FF                    test edi, edi   
:00477BF6 0F84E3000000            je 00477CDF   
:00477BFC 33F6                    xor esi, esi   
:00477BFE E9C7000000              jmp 00477CCA   
  
      for i:=1 to 2Ah do   
        d4IntArray\[i\]:=^((10p+7)+i));   
      edi:=^(10p+B0h);   
      if edi\=0 then   
        goto lblExit;   
      esi:=0   
      goto lbl00477CCA;   
    end;   
  end;   
end;   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00477CD9(C)   
|   
:00477C03 8BD6                    mov edx, esi   
:00477C05 8BC7                    mov eax, edi   
:00477C07 E8B4F5FFFF              call 004771C0   
:00477C0C 8BD8                    mov ebx, eax   
:00477C0E 8B4308                  mov eax, dword ptr \[ebx+08\]   
:00477C11 3D01000171              cmp eax, 71010001   
:00477C16 740B                    je 00477C23   
:00477C18 3D01000170              cmp eax, 70010001   
:00477C1D 0F85A6000000            jne 00477CC9   
  
lbl00477C03;   
ebx:=sub\_4771C0(esi,edi);   
case ^(ebx+8) of   
  7101001,70010001:   
  begin   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00477C16(C)   
|   
:00477C23 C745D418000000          mov \[ebp\-2C\], 00000018   
:00477C2A 8B4304                  mov eax, dword ptr \[ebx+04\]   
:00477C2D 8945D8                  mov dword ptr \[ebp\-28\], eax   
:00477C30 8B430C                  mov eax, dword ptr \[ebx+0C\]   
:00477C33 F7C001000000            test eax, 00000001   
:00477C39 7609                    jbe 00477C44   
:00477C3B C745DC01000000          mov \[ebp\-24\], 00000001   
:00477C42 EB09                    jmp 00477C4D   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00477C39(C)   
|   
:00477C44 8B9548FFFFFF            mov edx, dword ptr \[ebp+FFFFFF48\]   
:00477C4A 8955DC                  mov dword ptr \[ebp\-24\], edx   
  
    cbStuct:=18h;   
    dwControlID:=^(ebx+4h);   
    eax:=^(ebx+Ch);   
    if eax\>1 then   
      cChannels:=1   
    else   
      cChannels:=B8int;   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00477C42(U)   
|   
:00477C4D 83E002                  and eax, 00000002   
:00477C50 83F802                  cmp eax, 00000002   
:00477C53 7508                    jne 00477C5D   
:00477C55 8B4310                  mov eax, dword ptr \[ebx+10\]   
:00477C58 8945E0                  mov dword ptr \[ebp\-20\], eax   
:00477C5B EB05                    jmp 00477C62   

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
> |:00477C53(C)   
> |   
> :00477C5D 33C0                    xor eax, eax   
> :00477C5F 8945E0                  mov dword ptr \[ebp\-20\], eax   
>   
>     and eax,00000002h;   
>     if eax\=00000002h then   
>       hwndOwner:=^(ebx+10h)   
>     else   
>       hwndOwner:=0;   
>     end;   
>   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
> |:00477C5B(U)   
> |   
> :00477C62 C745E404000000          mov \[ebp\-1C\], 00000004   
> :00477C69 8D85B0FEFFFF            lea eax, dword ptr \[ebp+FFFFFEB0\]   
> :00477C6F 8945E8                  mov dword ptr \[ebp\-18\], eax   
> :00477C72 C645EE01                mov \[ebp\-12\], 01   
>   
>     cbDetails:=4;   
>     paDetails:=@(ebp\-150h);   
>     12bNoMute:=true;   
>   
> :00477C76 6A00                    push 00000000   
> :00477C78 8D45D4                  lea eax, dword ptr \[ebp\-2C\]   
> :00477C7B 50                      push eax   
> :00477C7C 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
> :00477C7F 8B4030                  mov eax, dword ptr \[eax+30\]   
> :00477C82 50                      push eax   
>   
> \* Reference To: winmm.mixerGetControlDetailsA, Ord:0000h   
>                                   |   
> :00477C83 E840EFFFFF              Call 00476BC8   
>   
>     mixerGetControlDetails(^(4pEax+30h),@MIXERCONTROLDETAILS,0);   
>   
> :00477C88 817B0801000170          cmp dword ptr \[ebx+08\], 70010001   
> :00477C8F 7519                    jne 00477CAA   
> :00477C91 8B5310                  mov edx, dword ptr \[ebx+10\]   
> :00477C94 4A                      dec edx   
> :00477C95 85D2                    test edx, edx   
> :00477C97 7C11                    jl 00477CAA   
> :00477C99 42                      inc edx   
> :00477C9A 8D85B0FEFFFF            lea eax, dword ptr \[ebp+FFFFFEB0\]   
>   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
> |:00477CA8(C)   
> |   
> :00477CA0 33C9                    xor ecx, ecx   
> :00477CA2 8908                    mov dword ptr \[eax\], ecx   
> :00477CA4 83C004                  add eax, 00000004   
> :00477CA7 4A                      dec edx   
> :00477CA8 75F6                    jne 00477CA0   
>   
>     if ^(ebx+8)=70010001 then   
>     begin   
>       edx:=^(ebx+10);   
>       edx:=edx\-1;   
>       if edx\=0 then   
>       begin   
>         edx:=edx+1;   
>         eax:=ebp\-150h;   
>         for i:=edx to 0 do   
>         begin   
>           @eax:=0;   
>           eax:=eax+4;   
>         end;   
>       end;   
>     end;   
>   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
> |:00477C8F(C), :00477C97(C)   
> |   
> :00477CAA 8B45F8                  mov eax, dword ptr \[ebp\-08\]   
> :00477CAD 8B5508                  mov edx, dword ptr \[ebp+08\]   
> :00477CB0 899485B0FEFFFF          mov dword ptr \[ebp+4\*eax\-00000150\], edx   
>   
>     ^(ebp+8Ecx\*4-150h):=Mute;//在 8Ecx 不为 FFFFFFFFh 的时候才会进入这里    
>   
> :00477CB7 6A00                    push 00000000   
> :00477CB9 8D45D4                  lea eax, dword ptr \[ebp\-2C\]   
> :00477CBC 50                      push eax   
> :00477CBD 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
> :00477CC0 8B4030                  mov eax, dword ptr \[eax+30\]   
> :00477CC3 50                      push eax   
>   
> \* Reference To: winmm.mixerSetControlDetails, Ord:0000h   
>                                   |   
> :00477CC4 E82FEFFFFF              Call 00476BF8   
>   
>     mixerGetControlDetails(^(4pEax+30h),@MIXERCONTROLDETAILS,0);   
>   
>   end;   
> else   
>   begin   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
> |:00477C1D(C)   
> |   
> :00477CC9 46                      inc esi   
>   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
> |:00477BFE(U)   
> |   
> :00477CCA 807DEE00                cmp byte ptr \[ebp\-12\], 00   
> :00477CCE 750F                    jne 00477CDF   
> :00477CD0 8BC7                    mov eax, edi   
> :00477CD2 E8F5F4FFFF              call 004771CC   
> :00477CD7 3BF0                    cmp esi, eax   
> :00477CD9 0F8C24FFFFFF            jl 00477C03   
>   
>     esi:=esi+1;   
> lbl0077CCA:   
>     if not 12bNoMute then   
>       if esi<sub\_004771CC(edi) then   
>         goto lbl00477C03;   
>   end;   
> end;   
>   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
> |:00477A4C(C), :00477AAB(C), :00477BC6(C), :00477BD0(C), :00477BF6(C)   
> |:00477CCE(C)   
> |   
>   
> :00477CDF 8A45F7                  mov al, byte ptr \[ebp\-09\]   
> :00477CE2 5F                      pop edi   
> :00477CE3 5E                      pop esi   
> :00477CE4 5B                      pop ebx   
> :00477CE5 8BE5                    mov esp, ebp   
> :00477CE7 5D                      pop ebp   
> :00477CE8 C20C00                  ret 000C   
>   
> lblExit:   
>   result:=9bResult;   
>   
> \==================================================   
>   
> 把上面的代码合并起来，我们得到下面一段程序，这样看似乎更清晰一些：）   
>   
> 8Ecx:=ecx;   
> 4pEax:=eax;   
> 9bResult:=false;   
>   
> 10p:=sub\_00477588( ^(4pEax+28h) );//返回指针 10p   
> if 10p=nil then//失败返回   
>   goto lblExit;   
>   
> if 8Ecx=FFFFFFFFh then   
> begin   
>   for i:=1 to 2Ah do   
>     d4IntArray\[i\]:=^((10p+7)+i);//拷贝数据    
>   edi:=^(10p+B0h);   
> end   
> else   
> begin   
>   eax:=sub\_004773A0( ^(10p+B4h),8Ecx );//返回指针 eax   
>   if eax<>0 then   
>   begin   
>     for i:=1 to 2Ah do   
>       d4IntArray\[i\]:=^((eax+7)+i));//拷贝数据    
>     edi:=^(eax+B0h);   
>   end   
>   else   
>     edi:=0;   
> end;   
>   
> if edi\=0 then//失败返回   
>   goto lblExit;   
>   
> esi:=0;   
> if rightValue=FFFFFFFFh then   
>   11bNoRightValue:=true;   
> if Mute=FFFFFFFFh then   
>   12bNoMute:=true;   
> 9bResult:=true;   
> goto lbl00477BA7;//转移到后面去执行，然后又跳回来执行下面的代码   
>   
> lbl00477AAC:   
> ebx:=sub\_004771C0(esi,edi);//返回值 ebx 是个指针，\[ebx+4\]是 dwControlID，\[ebx+8\]是设置类型，\[ebx+0Ch\]为 cChannels    
> if ebx<>0 then   
> begin   
>   case ^(ebx+8) of//本机返回的三个数值分别是：20010004，50030001，20010002    
>     20010002,5003001:   
>     begin   
>       eax:=^(ebx+Ch);//NewPlay 在这里返回^(ebx+Ch)=0，表示使用默认值 2   
>       if eax\=00000002h then   
>         goto lbl00477BA6;   
>       cbStuct:=18h;//设置 MIXERCONTROLDETAILS 结构   
>       dwControlID:=^(ebx+4h);   
>       if ^(ebx+Ch)>1 then   
>         cChannels:=1   
>       else   
>         cChannels:=B8int;//=2   
>       end;   
>       hwndOwner:=0;   
>       cbDetails:=4;   
>       paDetails:=@\[ebp\-150h\];   
>       if \[ebx+8\]=50030001 then//50030001 设置音量   
>       begin   
>         if not 11bNoRightValue then//如果右声道数据有定义(<>FFFFFFFFh)   
>         begin   
>           right:=rightValue;   
>           if leftValue=FFFFFFFFh then//如果左声道没有定义，用右声道数据设置   
>             left:=rihtValue;   
>           else   
>             left:=leftValue;   
>           11bNoRightValue:=true;   
>         end   
>       end   
>       else   
>         begin   
>         if \[ebx+8\]=20010002 then//20010002 设置静音   
>         begin   
>           if not 12bNoMute then   
>           begin   
>             right:=Mute;//当 cChannels=2 时，这里是静音问题的原因，只设置了一个声道。如果设置两个声道或者 cChannels=1 就正常    
>             12bNoMute:=true;   
>           end;   
>         end;   
>       end;   
>       mixerSetControlDetails(^(4pEax+30h),@MIXERCONTROLDETAILS,0);//设置音量或静音    
>     end;   
>   else   
>     begin   
> lbl00477BA6:   
>       esi:=esi+1;   
> lbl00477BA7:   
>       if 11NoRightValue then//如果右声道参数无效，或者已经设置过（上面设置后就会令 11NoRightValue=true）   
>         if 12bNoMute then//如果静音参数无效，此时无可用参数，失败返回。或者已经设置过（上面设置后就会令 12bNoMute=true），此时成功返回    
>           goto lb00477BC2;   
>       if esi<sub\_004771CC(edi) then//本机此函数返回 3   
>         goto lbl00477ACC;   
> lbl00477BC2:   
>       if 12bNoMute then   
>         goto lblExit;   
>       if 8Ecx=FFFFFFFFh then   
>         goto lblExit;   
>       for i:=1 to 2Ah do   
>         d4IntArray\[i\]:=^((10p+7)+i));   
>       edi:=^(10p+B0h);   
>       if edi\=0 then   
>         goto lblExit;   
>       esi:=0   
>       goto lbl00477CCA;   
>     end;   
>   end;   
> end;   
>   
> ////////////////////////////////////////下面一些设置相关的代码在 8Ecx<>FFFFFFFFh 时执行，但本机上，参数 ecx 始终是 FFFFFFFFh，所以没有执行的机会    
> lbl00477C03;   
> ebx:=sub\_4771C0(esi,edi);   
> case ^(ebx+8) of   
>   7101001,70010001:   
>   begin   
>     cbStuct:=18h;   
>     dwControlID:=^(ebx+4h);   
>     eax:=^(ebx+Ch);   
>     if eax\>1 then   
>       cChannels:=1   
>     else   
>       cChannels:=B8int;   
>     and eax,00000002h;   
>     if eax\=00000002h then   
>       hwndOwner:=^(ebx+10h)   
>     else   
>       hwndOwner:=0;   
>     end;   
>     cbDetails:=4;   
>     paDetails:=@(ebp\-150h);   
>     12bNoMute:=true;   
>     mixerGetControlDetails(^(4pEax+30h),@MIXERCONTROLDETAILS,0);//取设置    
>     if ^(ebx+8)=70010001 then   
>     begin   
>       edx:=^(ebx+10);   
>       edx:=edx\-1;   
>       if edx\=0 then   
>       begin   
>         edx:=edx+1;   
>         eax:=ebp\-150h;   
>         for i:=edx to 0 do   
>         begin   
>           @eax:=0;   
>           eax:=eax+4;   
>         end;   
>       end;   
>     end;   
>     ^(ebp+8Ecx\*4-150h):=Mute;//在 8Ecx 不为 FFFFFFFFh 的时候才会进入这里    
>     mixerGetControlDetails(^(4pEax+30h),@MIXERCONTROLDETAILS,0);//设置    
>   end;   
> else   
>   begin   
>     esi:=esi+1;   
> lbl0077CCA:   
>     if not 12bNoMute then   
>       if esi<sub\_004771CC(edi) then   
>         goto lbl00477C03;   
>   end;   
> end;   
>   
> lblExit:   
>   result:=9bResult;   
>   
> \==================================================   
>   
> 可以看到，代码的流程是先设置音量（包括平衡），再设置静音。   
>   
> NewPlay4.01 的静音问题出在这里：   
>   
> :00477B78 817B0802000120          cmp dword ptr \[ebx+08\], 20010002   
> :00477B7F 7513                    jne 00477B94   
> :00477B81 807DEE00                cmp byte ptr \[ebp\-12\], 00   
> :00477B85 750D                    jne 00477B94   
> :00477B87 8B4508                  mov eax, dword ptr \[ebp+08\]   
> :00477B8A 8985B4FEFFFF            mov dword ptr \[ebp+FFFFFEB0\], eax   
> :00477B90 C645EE01                mov \[ebp\-12\], 01   
>   
>         if \[ebx+8\]=20010002 then//20010002 设置静音    
>         begin   
>           if not 12bNoMute then   
>           begin   
>             right:=Mute;   
>             12bNoMute:=true;   
>           end;   
>         end;   
>   
> 现在我反汇编了 NewPlay 4.02 版，已经修正为如下代码：   
>   
> :00477E58 817B0802000120          cmp dword ptr \[ebx+08\], 20010002   
> :00477E5F 751C                    jne 00477E7D   
> :00477E61 807DEE00                cmp byte ptr \[ebp\-12\], 00   
> :00477E65 7516                    jne 00477E7D   
> :00477E67 8B4508                  mov eax, dword ptr \[ebp+08\]   
> :00477E6A 8985B0FEFFFF            mov dword ptr \[ebp+FFFFFEB0\], eax   
> :00477E70 8B4508                  mov eax, dword ptr \[ebp+08\]   
> :00477E73 8985B4FEFFFF            mov dword ptr \[ebp+FFFFFEB4\], eax   
> :00477E79 C645EE01                mov \[ebp\-12\], 01   
>   
>         if \[ebx+8\]=20010002 then//20010002 设置静音   
>         begin   
>           if not 12bNoMute then   
>           begin   
>             right:=Mute;   
>             left:=Mute;   
>             12bNoMute:=true;   
>           end;   
>         end;   
>   
> 我想这可能有助于理解 fm99 朋友的问题 “用什么方法可以插入一句“本想插一句 right:=mute;进去” （抱歉的是，上文的 left 和 right 写反了，4.02 版是插了一句 left:=mute 进去）。  4dd466d0e1e0cc123afbe504da2462ca