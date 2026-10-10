

# MacroClip 2000.2.7

> 原文：[`www.pediy.com/kssd/pediy04/kanxue094.htm`](https://www.pediy.com/kssd/pediy04/kanxue094.htm)

软件破解初体验之 MacroClip 2000.2.7 代码修改破解   
  
  
  
破解对象： MacroClip   
  
Version: 2000.2.7   
Platform: Windows95/98/NT/2000.   
  
URL：http://www.gentee.com/mclip/mclip.exe   
  
破解工具： Hiew 6.76 注册版   
          Caspr 1.10 GUI 版本   
          FI 2.45 注册版   
          W32DSM 8.93 增强版   
      外加 大脑和手、纸、笔。   
  
一、破解分析：   
  
1、首先检测软件是否加壳：使用 FI 查看得知，软件使用 aspack 1.00 加壳，所以使用 Caspr 脱掉程序壳。   
2、对拿到的脱壳后的文件进行 W32DSM 反汇编，查看代码。通过查看“串式参考”查看，无法得到出错信息，   
  只是能知道本程序是采用 keyfile 来做注册的。没有明显的语句提示来提供破解参考。   
3、运行程序知道程序有 30 天限制，同时显示“Unregistered”字样。   
  
二、破解点选择：   
  
1、首先可以使用 trw2000 来跟踪，下断点 bpx getsystemtime,bpx localtime 等。   
2、根据 30 天的限制，查找汇编语句中带有 0000001E 的比较语句。   
  
  这里我们选择第 2 种方法。   
  
三、代码分析和查找修改点   
  
  程序根据 30 天的限制来判断是否是注册版，使用的是标志位判断，标志位不对就是肥注册版，所以我们要把他判断注册标志位的分支全部修改。让它认为程序是注册版，而不会去限制程序的使用时间和功能限制。   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004097AF(U)   
|   
:004097C4 8B0D0C2B4200            mov ecx, dword ptr \[00422B0C\]   
:004097CA 83C901                  or ecx, 00000001   
:004097CD 890D0C2B4200            mov dword ptr \[00422B0C\], ecx   
:004097D3 8B4DFC                  mov ecx, dword ptr \[ebp\-04\]   
:004097D6 E8C5FEFFFF              call 004096A0   
:004097DB 833D802A420000          cmp dword ptr \[00422A80\], 00000000   
:004097E2 7511                    jne 004097F5  //-------------------->标志位的判断点   
:004097E4 833D842A420000          cmp dword ptr \[00422A84\], 00000000   
:004097EB 7508                    jne 004097F5   
:004097ED 8B4DFC                  mov ecx, dword ptr \[ebp\-04\]   
:004097F0 E81C6E0000              call 00410611   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00409782(C), :004097E2(C), :004097EB(C)   
|   
:004097F5 8BE5                    mov esp, ebp   
:004097F7 5D                      pop ebp   
:004097F8 C3                      ret   
.......<中间省略代码大部分>......................   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040A9A1(C)   
|   
:0040A9FC 8BE5                    mov esp, ebp   
:0040A9FE 5D                      pop ebp   
:0040A9FF C3                      ret   
  
  
  
\* Referenced by a CALL at Address:   
|:0040AD27    
|   
:0040AA00 55                      push ebp   
:0040AA01 8BEC                    mov ebp, esp   
:0040AA03 83EC10                  sub esp, 00000010   
:0040AA06 894DF0                  mov dword ptr \[ebp\-10\], ecx   
:0040AA09 8B45F0                  mov eax, dword ptr \[ebp\-10\]   
:0040AA0C 8B88AA000000            mov ecx, dword ptr \[eax+000000AA\]   
:0040AA12 8B516A                  mov edx, dword ptr \[ecx+6A\]   
:0040AA15 8B4A52                  mov ecx, dword ptr \[edx+52\]   
:0040AA18 E82D4E0000              call 0040F84A   
:0040AA1D 8945F4                  mov dword ptr \[ebp\-0C\], eax   
:0040AA20 8B45F0                  mov eax, dword ptr \[ebp\-10\]   
:0040AA23 8B88AE000000            mov ecx, dword ptr \[eax+000000AE\]   
:0040AA29 8B516A                  mov edx, dword ptr \[ecx+6A\]   
:0040AA2C 8B4A52                  mov ecx, dword ptr \[edx+52\]   
:0040AA2F E8164E0000              call 0040F84A   
:0040AA34 8945FC                  mov dword ptr \[ebp\-04\], eax   
:0040AA37 8B45F0                  mov eax, dword ptr \[ebp\-10\]   
:0040AA3A 8B88A6000000            mov ecx, dword ptr \[eax+000000A6\]   
:0040AA40 8B5172                  mov edx, dword ptr \[ecx+72\]   
:0040AA43 C1EA03                  shr edx, 03   
:0040AA46 83E201                  and edx, 00000001   
:0040AA49 85D2                    test edx, edx   
:0040AA4B 742A                    je 0040AA77   
:0040AA4D 837DF403                cmp dword ptr \[ebp\-0C\], 00000003   
:0040AA51 7C06                    jl 0040AA59   
:0040AA53 837DF41E                cmp dword ptr \[ebp\-0C\], 0000001E   
:0040AA57 7E1C                    jle 0040AA75  //--------------------->分析重点   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040AA51(C)   
|   
:0040AA59 8B45F0                  mov eax, dword ptr \[ebp\-10\]   
:0040AA5C 8B88AA000000            mov ecx, dword ptr \[eax+000000AA\]   
:0040AA62 51                      push ecx   
:0040AA63 BA36010000              mov edx, 00000136   
:0040AA68 8B4DF0                  mov ecx, dword ptr \[ebp\-10\]   
:0040AA6B E8D45A0000              call 00410544   
:0040AA70 E9EB010000              jmp 0040AC60   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
  
  通过对含有 0000001E 的语句分析得知 cmp dword ptr \[00422A80\], 00000000 是标志位的判断，所以分析与此有关的代码段。下面是要修改的代码：   
  
####################################################################   
  
★Part One★   
  
####################################################################   
  
\* Possible StringData Ref from Data Obj ->"?   
                                  |   
:00405E3D B9B0274200              mov ecx, 004227B0   
:00405E42 E8F5B60000              call 0041153C   
:00405E47 A390314200              mov dword ptr \[00423190\], eax   
:00405E4C 6A00                    push 00000000   
:00405E4E 686A434000              push 0040436A   
:00405E53 BA10000000              mov edx, 00000010   
:00405E58 8B0D90314200            mov ecx, dword ptr \[00423190\]   
:00405E5E E8FAA60000              call 0041055D   
:00405E63 E8521B0000              call 004079BA   
:00405E68 6A00                    push 00000000   
:00405E6A BACE000000              mov edx, 000000CE   
:00405E6F 8B0D202E4200            mov ecx, dword ptr \[00422E20\]   
:00405E75 E8CAA60000              call 00410544   
:00405E7A 833D802A420000          cmp dword ptr \[00422A80\], 00000000   
:00405E81 7542                    jne 00405EC5 //----------->这里要跳转。75-->74   
  
\* Possible StringData Ref from Data Obj ->""   
                                  |   
:00405E83 68A8214200              push 004221A8   
  
\* Possible StringData Ref from Data Obj ->"?   
                                  |   
:00405E88 BA60274200              mov edx, 00422760   
:00405E8D 8B0D202E4200            mov ecx, dword ptr \[00422E20\]   
:00405E93 E809A70000              call 004105A1   
:00405E98 8985E4FEFFFF            mov dword ptr \[ebp+FFFFFEE4\], eax   
:00405E9E 8B85E4FEFFFF            mov eax, dword ptr \[ebp+FFFFFEE4\]   
:00405EA4 8B4842                  mov ecx, dword ptr \[eax+42\]   
:00405EA7 83C940                  or ecx, 00000040   
:00405EAA 8B95E4FEFFFF            mov edx, dword ptr \[ebp+FFFFFEE4\]   
:00405EB0 894A42                  mov dword ptr \[edx+42\], ecx   
:00405EB3 6A00                    push 00000000   
:00405EB5 BACC000000              mov edx, 000000CC   
:00405EBA 8B8DE4FEFFFF            mov ecx, dword ptr \[ebp+FFFFFEE4\]   
:00405EC0 E87FA60000              call 00410544   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
  
####################################################################   
  
★Part Two★   
  
####################################################################   
  
  
\* Possible StringData Ref from Data Obj ->"jj@˙鯝"   
                                  |   
:00406F75 BA30274200              mov edx, 00422730   
:00406F7A 8B8D4CFFFFFF            mov ecx, dword ptr \[ebp+FFFFFF4C\]   
:00406F80 E81C960000              call 004105A1   
:00406F85 8BC8                    mov ecx, eax   
:00406F87 8BD6                    mov edx, esi   
:00406F89 E8B6950000              call 00410544   
:00406F8E 833D802A420000          cmp dword ptr \[00422A80\], 00000000   
:00406F95 0F85EB000000            jne 00407086  //----------->这里要跳转。75-->74   
  
:00406F9B C7458437000000          mov \[ebp\-7C\], 00000037   
:00406FA2 8D8578FFFFFF            lea eax, dword ptr \[ebp+FFFFFF78\]   
:00406FA8 50                      push eax   
  
\* Possible StringData Ref from Data Obj ->"jj@˙鯝"   
                                  |   
:00406FA9 BA30274200              mov edx, 00422730   
:00406FAE 8B0D782C4200            mov ecx, dword ptr \[00422C78\]   
:00406FB4 E8E8950000              call 004105A1   
:00406FB9 8945F8                  mov dword ptr \[ebp\-08\], eax   
:00406FBC 6A00                    push 00000000   
:00406FBE BACD000000              mov edx, 000000CD   
:00406FC3 8B4DF8                  mov ecx, dword ptr \[ebp\-08\]   
:00406FC6 E879950000              call 00410544   
  
\* Possible StringData Ref from Data Obj ->""   
                                  |   
:00406FCB 68A8214200              push 004221A8   
  
\* Possible StringData Ref from Data Obj ->"?   
                                  |   
####################################################################   
  
★Part Three★   
  
####################################################################   
  
  
:00407674 833800                  cmp dword ptr \[eax\], 00000000   
:00407677 740A                    je 00407683   
:00407679 B901000000              mov ecx, 00000001   
:0040767E E8A9BBFFFF              call 0040322C   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00407677(C)   
|   
:00407683 833D802A420000          cmp dword ptr \[00422A80\], 00000000   
:0040768A 753C                    jne 004076C8  //----------->这里要跳转。75-->74   
  
:0040768C 833D842A420000          cmp dword ptr \[00422A84\], 00000000   
:00407693 741E                    je 004076B3   
:00407695 8B15842A4200            mov edx, dword ptr \[00422A84\]   
:0040769B 52                      push edx   
:0040769C B9A2010000              mov ecx, 000001A2   
:004076A1 E84A3C0000              call 0040B2F0   
:004076A6 50                      push eax   
:004076A7 6A08                    push 00000008   
:004076A9 E87899FFFF              call 00401026   
:004076AE 83C40C                  add esp, 0000000C   
:004076B1 EB15                    jmp 004076C8   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00407693(C)   
|   
:004076B3 B9A3010000              mov ecx, 000001A3   
:004076B8 E8333C0000              call 0040B2F0   
:004076BD 50                      push eax   
:004076BE 6A01                    push 00000001   
:004076C0 E86199FFFF              call 00401026   
:004076C5 83C408                  add esp, 00000008   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
  
  
####################################################################   
  
★Part Four★   
  
####################################################################   
  
  
  
:0040976A BA77270000              mov edx, 00002777   
:0040976F A1082B4200              mov eax, dword ptr \[00422B08\]   
:00409774 8B0C85602E4200          mov ecx, dword ptr \[4\*eax+00422E60\]   
:0040977B E8496D0000              call 004104C9   
:00409780 85C0                    test eax, eax   
:00409782 7571                    jne 004097F5   
:00409784 833D802A420000          cmp dword ptr \[00422A80\], 00000000   
:0040978B 7524                    jne 004097B1  //----------->这里要跳转。75-->74   
  
:0040978D 833D842A420000          cmp dword ptr \[00422A84\], 00000000   
:00409794 751B                    jne 004097B1   
:00409796 B9A3010000              mov ecx, 000001A3   
:0040979B E8501B0000              call 0040B2F0   
:004097A0 8BD0                    mov edx, eax   
  
\* Possible StringData Ref from Data Obj ->"ZZ\`鐱 KA"   
                                  |   
:004097A2 B9D0264200              mov ecx, 004226D0   
:004097A7 E8907D0000              call 0041153C   
:004097AC 8945FC                  mov dword ptr \[ebp\-04\], eax   
:004097AF EB13                    jmp 004097C4   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:0040978B(C), :00409794(C)   
|   
:004097B1 8B0D082B4200            mov ecx, dword ptr \[00422B08\]   
:004097B7 8B148D602E4200          mov edx, dword ptr \[4\*ecx+00422E60\]   
:004097BE 8B4261                  mov eax, dword ptr \[edx+61\]   
:004097C1 8945FC                  mov dword ptr \[ebp\-04\], eax   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004097AF(U)   
  
####################################################################   
  
★Part Five★   
  
####################################################################   
  
  
|:004097AF(U)   
|   
:004097C4 8B0D0C2B4200            mov ecx, dword ptr \[00422B0C\]   
:004097CA 83C901                  or ecx, 00000001   
:004097CD 890D0C2B4200            mov dword ptr \[00422B0C\], ecx   
:004097D3 8B4DFC                  mov ecx, dword ptr \[ebp\-04\]   
:004097D6 E8C5FEFFFF              call 004096A0   
:004097DB 833D802A420000          cmp dword ptr \[00422A80\], 00000000   
:004097E2 7511                    jne 004097F5  //----------->这里要跳转。75-->74   
  
:004097E4 833D842A420000          cmp dword ptr \[00422A84\], 00000000   
:004097EB 7508                    jne 004097F5   
:004097ED 8B4DFC                  mov ecx, dword ptr \[ebp\-04\]   
:004097F0 E81C6E0000              call 00410611   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00409782(C), :004097E2(C), :004097EB(C)   
|   
:004097F5 8BE5                    mov esp, ebp   
:004097F7 5D                      pop ebp   
:004097F8 C3                      ret   
  
  
  
\* Referenced by a CALL at Addresses:   
|:004098D5  , :004098E2  , :004098EC  , :00409902    
|   
:004097F9 55                      push ebp   
:004097FA 8BEC                    mov ebp, esp   
  
####################################################################   
  
★Part Six★   
  
####################################################################   
  
  
  
:0040BE75 8B15F0344200            mov edx, dword ptr \[004234F0\]   
:0040BE7B 6BD203                  imul edx, 00000003   
:0040BE7E 89953CFFFFFF            mov dword ptr \[ebp+FFFFFF3C\], edx   
:0040BE84 C78530FFFFFF0D000000    mov dword ptr \[ebp+FFFFFF30\], 0000000D   
:0040BE8E 8B854CFFFFFF            mov eax, dword ptr \[ebp+FFFFFF4C\]   
:0040BE94 24F3                    and al, F3   
:0040BE96 89854CFFFFFF            mov dword ptr \[ebp+FFFFFF4C\], eax   
:0040BE9C 8B8D4CFFFFFF            mov ecx, dword ptr \[ebp+FFFFFF4C\]   
:0040BEA2 83C940                  or ecx, 00000040   
:0040BEA5 898D4CFFFFFF            mov dword ptr \[ebp+FFFFFF4C\], ecx   
:0040BEAB 833D802A420000          cmp dword ptr \[00422A80\], 00000000   
:0040BEB2 740E                    je 0040BEC2  //----------->这里要跳转。75-->74   
  
:0040BEB4 8B15802A4200            mov edx, dword ptr \[00422A80\]   
:0040BEBA 8995DCFEFFFF            mov dword ptr \[ebp+FFFFFEDC\], edx   
:0040BEC0 EB0A                    jmp 0040BECC   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040BEB2(C)   
|   
:0040BEC2 C785DCFEFFFFA02A4200    mov dword ptr \[ebp+FFFFFEDC\], 00422AA0   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040BEC0(U)   
|   
:0040BECC 8B85DCFEFFFF            mov eax, dword ptr \[ebp+FFFFFEDC\]   
  
####################################################################   
  
★Part End★   
  
####################################################################   
  
四、修改程序代码   
  
    用 Hiew6.76 注册版选择主文件，然后使用 F4 选择 Dcode 模式，分别查找上面代码中要修改的地方的 offset 值（可以在 W32DSM 中光标双击代码行，看 W32DSM 窗口的状态栏，里面的@OffSet \*\*\*\*\*\*\*\*\*h,中的\*\*\*\*\*\*\*就是要的值）。分别修改就完成了破解。   
  
五、软件汉化   
  
    拷贝 Langage 目录中的 English.ln 为 Chinese.ln，然后汉化翻译其中的语句即可在程序菜单中选择中文。   
  
六、最终破解汉化文件下载   
  
    借用 CrackerABC\[BCG\]老兄的 FTP，谢谢！   
  
    http://sffs.china.com/soft/hy-macroclip2000.2.2.zip   
  
    欢迎测试和交流，初写破解，请大客多指教！小妹这厢有礼了，^\_^   
  
  
                                    翠微池儿   
  
                                      2001 年 10 月 9 日  4dd466d0e1e0cc123afbe504da2462ca