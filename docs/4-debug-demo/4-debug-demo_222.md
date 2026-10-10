

# ACDSEE 3.0

> 原文：[`www.pediy.com/kssd/pediy03/forum718.htm`](https://www.pediy.com/kssd/pediy03/forum718.htm)

\[题好文一半\] 如何破解 ACDSEE 3.0   
\[始作文者\]  夜月   
\[人最重要的是找准自己的位置\] 菜鸟   
\[工欲善其事，必先利其器\]   
trw2000 1.22   
\[人为刀俎，我为鱼肉\]   
ACDSEE 3.0   
\[天下文章一大抄\]   
嗯？错了，错了，   
\[文章本天成，妙手偶得之\]   
嘿嘿.....这才像“破解四大才子”之一的夜月的风格吗....   
“你说什么”？   
“破解四大才子”呀！   
于是，美好的事物被撕裂给大家看到了.....   
  
一堆 CRACKER 冲了过来，拿着夜月从来没见过的 SICE 把夜月分成了   
60%的脂肪，30%的蛋白质，10%的“不明存在物”，简称 UEO（Unknown Exit Object）....   
  
从此，“破解四大才子”老大的板凳一直空闲,据说以前上面座着一个很帅的小伙......   
  
    ACDSEE 是干什么用的，相信各位不用我再说了吧。以前 2.4 版的时候，可以用序列号的方式注册，于是，   
注册码就满天飞。后来，出了 3.0 版，不能再用注册码注册了，可是后来，又出了个“零售版”，真是......   
不知道软件的作者看到他的软件如此流行，作何感想？小弟今天下午闲着无事，就来砍砍它的时间限制，写成此文，献给大家，希望能博得大家一笑。   
  
    时间限制的破解，肯定是暴力破解。   
    ACDSEE 的保护方式，肯定是 ASPACK 之流。   
    对付 ASPACK，肯定是 TRW2000 的绝招：LOAD+PNEWSEC+MAKEPE   
    大功告成！看到 NEWPE.EXE 了吗？   
    高兴乎？兴奋乎？奋起直追时间限制乎？   
  
    问：偷懒一派的一个典型特点是什么？   
    答：碰到软件先看《论坛精华》。有 1 就看 1，有 2 就看 2，将来有 3 了，自然就看 3。   
    再问：偷懒一派的另一个典型特点是什么？   
    再答：看时容易做时难，跟踪无力破解残。 未到核心力已尽，退出 TR 泪不干。   
      
    由于《论坛精华 2》中的文章是 E 文的，固有下面一段：   
    最近比较烦，比较烦，比较烦。    总觉得 E 文有一点难看。   
    我想我还是不习惯，从方方正正到大洋彼岸。   
    最近比较烦，比较烦，比较烦。    总觉得方法有一些弊端。   
    我问自己说怎么办？ 脑袋说： 基本上，这个很难。   
    麻烦，我很麻烦； 麻烦，我很麻烦；   
    我冥思苦想。   
  
    作家说：没有做不成的文章。   
    军人说：没有打不赢的战役。   
    CRACKER 们说：没有破不了的软件。   
    毛主席总结说：世上无难事，只要肯登攀。   
  
    夜月连忙附和：PATCH 不难打，只要有跳转。PC 烧成灰，CODE 不难追。   
                  我追---------------   
  
    于是有了（别人）追出来的时间断点：GETSYSTEMTIME 的出场。   
      
    断点一找对，事半功也倍。断点找不到，直接一傻冒。   
  
    作者可能以为：ASPACK 的保护，体贴又周到。   
    于是在时间的判断上就........   
  
    我一直追，感觉不算融洽。   
    在代码丛林中，也渐渐感到心在变化。   
    都是追到深处才由它，看不看懂都算了吧。   
    破解从来都是稀里糊涂的生涯。   
    不要再为了它挣扎，不要再为它左牵右挂。   
    算了吧，都随它！   
    
    追出来的代码如下：   
015F:004984DE  CALL    \`KERNEL32!GetSystemTime\`   
015F:004984E4  LEA      ECX,\[ESP+18\]   
015F:004984E8  LEA      EDX,\[ESP+20\]   
015F:004984EC  PUSH    ECX   
015F:004984ED  PUSH    EDX   
015F:004984EE  CALL    \`KERNEL32!SystemTimeToFileTime\`   
015F:004984F4  LEA      EAX,\[ESP+18\]   
015F:004984F8  PUSH    ESI   
015F:004984F9  PUSH    EAX   
015F:004984FA  CALL    \`KERNEL32!CompareFileTime\`   
015F:00498500  INC      EAX   
015F:00498501  POP      ESI   
015F:00498502  NEG      EAX   
015F:00498504  SBB      EAX,EAX   
015F:00498506  NEG      EAX   
015F:00498508  DEC      EAX   
015F:00498509  ADD      ESP,BYTE +7C   
015F:0049850C  RET      04   
      乱七八糟，气煞我也。一个跳转都没有，让我如何 PATCH？      
      代码若是不懂时，何不换条道路？   
      还记得鲁迅的话吗？   
      破解时本没有路，F（N）（N=5，7，9，12。。。。）按的多了，代码中就有了路。   
      于是 F12 过去后.........   
    
      是你给我一片天，是你给了我一片天，就算全世界代码我都不懂，走到你的身旁就看到春天。   
  
      小女孩问：春天是什么样子的？   
      夜月很自豪地 U+>（重定向符）了一下，   
      告诉小女孩：春天就是这个样子的：   
015F:004982BA  PUSH    EAX   
015F:004982BB  CALL    004983E0   
015F:004982C0  TEST    EAX,EAX          改为:inc eax    
015F:004982C2  JNZ      NEAR 004983CD    改为:jmp 4983cd   
015F:004982C8  MOV      CL,\[004E8A30\]   
015F:004982CE  PUSH    EDI   
        
      论坛的大客看过来，看过来，看过来，菜鸟的世界很精彩，请不要装作不理不睬。   
      破解菜鸟的悲哀，  说出来, 谁明白, 求求你发篇教程过来，教教我，让我乐开怀。   
      我左看右看，上看下看，原来每个程序都不简单。   
      我左看右看，上看下看，程序们的判断可真奇怪。   
      嘿！真奇怪........   
  
      看到这一句了么？   
015F:004982C0  TEST    EAX,EAX   
      当我在这里，把 EAX 无中生有地由 0 改成 1 之后，   
      当至尊宝在观音面前手舞足蹈，叽叽喳喳之后，   
      说了一句让我记忆犹新的经典：   
      整个世界清静了...........   
      是的,   
      整个世界清静了。   
  
\[狗尾叙貂\]   
      曾经有一个珍贵的注册码摆在我的面前，但是我没有珍惜，   
      直到看到看雪的教学才追悔莫及。   
      如果上天能再给我一次机会，   
      我会对那个注册码说：我要你！   
      如果要给注册期加上一个限制的话，   
      我希望是--------一万年！   
      
      远处传来刘德华的歌声：   
      注册一万年，注册经得起考验。   
      穿越了判断的防线，出现成功的地面！

将 ACDSee V3.1 Build 921 试用版变为注册版   
  
              by liangs  2001/3/18   
  
说明：软件可从《电脑报 2000 年合订本》配套光盘中获得。   
  
  前两天买了套《电脑报 2000 年合订本》，装了一下这个软件觉得还不错，比原来   
的 2.4 版功能强劲了不少。它的限制主要是时间限制，破掉它的时间限制很简单，但   
程序启动后标题栏显示的 Trial Version 让人看到很不舒服，我们试着拿掉它看看。   
  
(1)首先来搞定时间问题。过期后弹出 Trial Period Expired 的提示框，用 W32Dasm   
反汇编后，我们查找 Expired 的字样，我们可以找到如下的地址：   
  
\* Reference To: InTouchClient.?TrialPeriod\_GetDaysLeft@InTouchClient@@QAEHXZ, Ord:005Dh   
                                  |   
:00404596 FF1564824D00            Call dword ptr \[004D8264\]   
:0040459C 85C0                    test eax, eax   
:0040459E 7F16                    jg 004045B6   
  
\* Reference To: InTouchClient.?g\_itc@@3VInTouchClient@@A, Ord:0069h   
                                  |   
:004045A0 8B0D30824D00            mov ecx, dword ptr \[004D8230\]   
:004045A6 6A00                    push 00000000   
  
\* Reference To: InTouchClient.?TrialMode\_ShowExpiredBox@InTouchClient@@QAEHPAUHWND\_\_@@@Z,   
Ord:005Bh   
                                  |   
:004045A8 FF155C824D00            Call dword ptr \[004D825C\]   
:004045AE 85C0                    test eax, eax   
:004045B0 7521                    jne 004045D3   
:004045B2 83C408                  add esp, 00000008   
:004045B5 C3                      ret   
  
  上面的 TrialMode\_ShowExpiredBox 字样已经很能说明问题了，其它地方也能找到 Expired   
的字样，但是一看就知道不是，当然如果你看不出来的话，你就下 bpx 00xxxxxx 试试就知道了，   
根本就不会在那儿中断下来。   
  
很明显只要将 0040459E 处的 jg 004045B6 改为 Jmp 004045B6 就可以搞定时间问题了。   
  
(2)接着要搞定的是标题栏显示的 Trial Version 字样。我们查找 Trial Version 的字样，我们可   
以找到如下两处地址：   
  
第一处地址：   
  
:004172D0 B900434F00              mov ecx, 004F4300   
:004172D5 E816320000              call 0041A4F0   
:004172DA 85C0                    test eax, eax   
:004172DC 7529                    jne 00417307   
:004172DE 8D8C2414010000          lea ecx, dword ptr \[esp+00000114\]   
:004172E5 6A50                    push 00000050   
:004172E7 51                      push ecx   
  
\* Reference To: KERNEL32.lstrlenA, Ord:0335h   
                                  |   
:004172E8 FF1550834D00            Call dword ptr \[004D8350\]   
:004172EE 8D940418010000          lea edx, dword ptr \[esp+eax+00000118\]   
:004172F5 A124434F00              mov eax, dword ptr \[004F4324\]   
:004172FA 52                      push edx   
  
\* Possible Reference to String Resource ID=00195: " Trial Version"   
                                                  ^^^^^^^^^^^^^^^^^      
                                  |   
:004172FB 68C3000000              push 000000C3   
:00417300 50                      push eax   
  
\------------------------------------------------------------------------   
第二处地址：   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00465A0E(U)   
|   
:00465A36 B900434F00              mov ecx, 004F4300   
:00465A3B E8B04AFBFF              call 0041A4F0   
:00465A40 85C0                    test eax, eax   
:00465A42 7520                    jne 00465A64   
:00465A44 8D442408                lea eax, dword ptr \[esp+08\]   
:00465A48 6A50                    push 00000050   
:00465A4A 50                      push eax   
:00465A4B FFD6                    call esi   
:00465A4D 8B1524434F00            mov edx, dword ptr \[004F4324\]   
:00465A53 8D4C040C                lea ecx, dword ptr \[esp+eax+0C\]   
:00465A57 51                      push ecx   
  
\* Possible Reference to String Resource ID=00195: " Trial Version"   
                                                  ^^^^^^^^^^^^^^^^^   
                                  |   
:00465A58 68C3000000              push 000000C3   
:00465A5D 52                      push edx   
  
  你注意看一下就可以发现，在判断是否是"Trial Version"的时候，在前面共同都   
调用了 call 0041A4F0 来判断 eax 的值，如果 eax 到时候为 0 就铁定是" Trial Version"   
我们进 call 0041A4F0 的里面去看看：   
  
\* Referenced by a CALL at Addresses:   
|:004044F2  , :0040468F  , :0040488B  , :004049FE  , :004087DC    
|:004088D3  , :00412A6F  , :004172D5  , :0041837B  , :00418499    
|:0043D2B6  , :0043D3AB  , :0043D4B9  , :0043D5A5  , :0043D6E6    
|:0045A549  , :00465A3B    
|   
:0041A4F0 33C0                    xor eax, eax   
:0041A4F2 C3                      ret   
:0041A4F3 90                      nop   
:0041A4F4 90                      nop   
:0041A4F5 90                      nop   
:0041A4F6 90                      nop   
:0041A4F7 90                      nop   
  
  我们发现这里只有两行代码就是使 eax 为 0,而且还有很多地方要调用这个 Call,如果这里   
eax 返回的是 1 的话，我们的目的就完成了，好在 0041A4F3 后面有好几个 nop 可以给我们用。   
将上面代码简单修改如下就可以了:   
  
:0041A4F0 33C0                    xor eax, eax   
:0041A4F2 40                      inc eax   
:0041A4F3 c3                      ret   
  
  现在再重新启动 ACDSee 看看，是不是已经变成注册版了，还有一点不尽入人意的地方就是   
打开 About ACDSee...菜单时，里面的"The software is license to:"和"license number:"   
还是空的，要让它在这显示你的名字也很简单，用 RegMon 检测软件对注册表的读写，我们   
可以找到如下的地方，写个注册文件运行一下就可以了，至此 ACDSee V3.1 就彻底搞定了。:->   
  
REGEDIT4   
  
\[HKEY\_LOCAL\_MACHINE\\Software\\ACD Systems\\ACDInTouch\]   
"UserName"="liangs"   
"CompanyName"="ZTE"   
"UserEMail"="liangs@wx88.net"   
  
\[HKEY\_LOCAL\_MACHINE\\Software\\ACD Systems\\ACDSee\]   
"LicenseNumber"="ZTE389"   
  
\[HKEY\_CURRENT\_USER\\Software\\ACD Systems\\ACDInTouch\]   
"UserName"="liangs"

 4dd466d0e1e0cc123afbe504da2462ca