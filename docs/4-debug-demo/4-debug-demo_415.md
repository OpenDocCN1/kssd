

# TOM 对弈围棋客户端

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-116.htm`](https://www.pediy.com/kssd/pediy08/pediy8-116.htm)

【文章标题】: TOM 对弈围棋客户端的去“形势判断”功能限制  
【作    者】: laoqian\[FCG\]  
【邮    箱】: -  
【主    页】: www.fcgchina.com  
【QQ    号】: -  
【软件名称】: TOMweiqi.exe  
【下载地址】: http://weiqi.tom.com/down/TomWeiqi.exe  
【加壳方式】: 无  
【编写语言】: VC  
【工    具】: OllyDbg1.10 fly, W32DSM  
【操作平台】: windows 系列  
【软件介绍】: TOM 对弈围棋客户端  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
       狗年就不打狗了，我们来一个简单的爆破吧！  
       本文只适合初学者，高手略过！  
       前两天在这里看到一篇【XX 围棋客户端的去功能限制】的文章，搞的是新浪围棋，看了有些启发。但是我不用新浪围棋，主要是里面的  
  
人比较赖，我发现 TOM 对弈围棋里面的人都比较君子！  
    遂想到 TOM 围棋也有限制，不妨拿来看看。  
       TOM 对弈围棋其他也没什么，只有其中的“形势判断”极为有用，可以省去你自己数目判断的麻烦（——不过对于高手有些多余甚至降  
  
低水平之嫌了）！  
    但他有限制，主要是下面 2 个：  
      
    1.在押分对局中使用加倍卡或者在对局中进行最多押分时（比如，一般的 9 段对局为 5000 万）押分时，可以使用形势分析功能。  
      此限制，就是在观看高手对弈和棋谱时，无法在试下情况下使用“形势判断”。  
    2.根据双方协议，本局不能使用形势分析功能。此限制，就是在对弈时，双方比赛前协议不能使用“形势判断”，无法使用“形势判断”  
  
。  
      
    由于分析得知，客户端是有代码的，但是他的出错提示都是加过密的，反汇编是看不到关键点的。那我们就来爆破他实现，工作很简单，  
  
我只是说一下如何入手。其实对于新手破解时最困难的是如何入手，如何最恰当的下断点！  
      
    用 OllyDbg1.10 打开 LiveBaduk.exe，中断在入口，因为没有加壳，因此直接调试，忽略所有异常，取消所有断点。  
      
    F9 运行，登录帐号，进入大厅！  
    由于是联网调试，如果我们选择和其他人对弈时调试，肯定会有调试中断甚至掉线，会影响他人的，还连累自己的分数！那我们就选择观  
  
看棋局进入，再调试，即使被踢出来也无所谓。  
    进入某对弈房间观棋，此时切换到 od，下断点 bp MessageBoxA ，再切换回程序来（btw，od 对函数的大小写敏感，开始没注意老是断点设  
  
置不成功，唉）  
    点击“形势判断”按钮，假设房主的协议是 “本局不能使用形势分析功能”，此时会中断，出现提示窗口的，但是我当时进的房间是可以  
  
使用，没有中断！不要紧，那我们点击“试下”按钮，进入试下窗口，此时再点击“形势判断”按钮，中断！如下：  
      
      
    77D504EA U>  8BFF                 mov edi,edi    ; 我们的 bp MessageBoxA 断点，停在这  
    77D504EC     55                   push ebp       ;F8 一路走  
    77D504ED     8BEC                 mov ebp,esp  
    77D504EF     833D BC04D777 00     cmp dword ptr ds:\[77D704BC\],0  
    77D504F6     74 24                je short USER32.77D5051C  
    77D504F8     64:A1 18000000       mov eax,dword ptr fs:\[18\]  
    77D504FE     6A 00                push 0  
    77D50500     FF70 24              push dword ptr ds:\[eax+24\]  
    77D50503     68 240BD777          push USER32.77D70B24  
    77D50508     FF15 C812D177        call dword ptr ds:\[<&KERNEL32.InterlockedCompa>  
    77D5050E     85C0                 test eax,eax  
    77D50510     75 0A                jnz short USER32.77D5051C  
    77D50512     C705 200BD777 010000>mov dword ptr ds:\[77D70B20\],1  
    77D5051C     6A 00                push 0  
    77D5051E     FF75 14              push dword ptr ss:\[ebp+14\]  
    77D50521     FF75 10              push dword ptr ss:\[ebp+10\]  
    77D50524     FF75 0C              push dword ptr ss:\[ebp+C\]  
    77D50527     FF75 08              push dword ptr ss:\[ebp+8\]  
    77D5052A     E8 2D000000          call USER32.MessageBoxExA  ;F8 一路走  
    77D5052F     5D                   pop ebp  
    77D50530     C2 1000              retn 10    ;返回到下面  
     
     
  .......  
    6C14C8D7     FF7424 10            push dword ptr ss:\[esp+10\]  
    6C14C8DB     50                   push eax  
    6C14C8DC     FF7424 10            push dword ptr ss:\[esp+10\]  
    6C14C8E0     51                   push ecx                   ;F8 一路走到这，看堆栈窗口如下提示  
    6C14C8E1     FF15 D0B5186C        call dword ptr ds:\[<&USER32.MessageBoxA>\]      ;返回到这里  
    6C14C8E7     5E                   pop esi  
    6C14C8E8     C2 0C00              retn 0C      ;返回到 jmp.&MFC42.#4224  
    6C14C8EB     8B4E 20              mov ecx,dword ptr ds:\[esi+20\]  
    6C14C8EE   ^ EB E7                jmp short MFC42.6C14C8D7  
    6C14C8F0     E8 4F49FAFF          call MFC42.#2864  
    6C14C8F5   ^ E9 EE8AFAFF          jmp MFC42.6C0F53E8  
      
    0012CD94   000A09FE  |hOwner = 000A09FE ('摆 718 房间的棋',class='#32770')  
    0012CD98   016157F0  |Text = "在押分对局中使用加倍卡  
      
    或者在对局中进行最多押分时（比如，一般的 9 段对局为 5000 万）押分时，  
      
    可以使用形势分析功能。"  
    0012CD9C   00D04F50  |Title = "Tom 对弈"  
    0012CDA0   00000030  \\Style = MB\_OK|MB\_ICONEXCLAMATION|MB\_APPLMODAL  
    0012CDA4   0483A2D0  
    0012CDA8   00438FC4  返回到 LiveBadu.00438FC4 来自 <jmp.&MFC42.#4224>  
    
    
    00438F60     8B4E 74              mov ecx,dword ptr ds:\[esi+74\]  
    00438F63     8D4424 08            lea eax,dword ptr ss:\[esp+8\]  
    00438F67     50                   push eax  
    00438F68     81C1 14050000        add ecx,514  
    00438F6E     E8 8D7AFDFF          call LiveBadu.00410A00  
    00438F73     8B0D 08065600        mov ecx,dword ptr ds:\[560608\]  
    00438F79     8B81 E0060000        mov eax,dword ptr ds:\[ecx+6E0\]  
    00438F7F     85C0                 test eax,eax  
    00438F81     75 52                jnz short LiveBadu.00438FD5     ; jmp，这里就是爆破点  
    00438F83     8B46 68              mov eax,dword ptr ds:\[esi+68\]  
    00438F86     85C0                 test eax,eax  
    00438F88     75 4B                jnz short LiveBadu.00438FD5    ; jmp，这里也是爆破点  
    00438F8A     8D4C24 10            lea ecx,dword ptr ss:\[esp+10\]  
    00438F8E     E8 05270D00          call <jmp.&MFC42.#540>  
    00438F93     68 04030000          push 304  
    00438F98     8D4C24 14            lea ecx,dword ptr ss:\[esp+14\]  
    00438F9C     C74424 38 01000000   mov dword ptr ss:\[esp+38\],1  
    00438FA4     E8 A5280D00          call <jmp.&MFC42.#4160>  
    00438FA9     8B15 00065600        mov edx,dword ptr ds:\[560600\]   ; LiveBadu.0055FB78  
    00438FAF     6A 30                push 30  
    00438FB1     8BCE                 mov ecx,esi  
    00438FB3     8B82 34020000        mov eax,dword ptr ds:\[edx+234\]  
    00438FB9     50                   push eax  
    00438FBA     8B4424 18            mov eax,dword ptr ss:\[esp+18\]  
    00438FBE     50                   push eax  
    00438FBF     E8 7A290D00          call <jmp.&MFC42.#4224>      ; 从 bp MessageBoxA 断点返回到这里程序领空，出现提示窗口!  
    00438FC4     C74424 34 FFFFFFFF   mov dword ptr ss:\[esp+34\],-1  
    00438FCC     8D4C24 10            lea ecx,dword ptr ss:\[esp+10\]  
    00438FD0     E9 37010000          jmp LiveBadu.0043910C  
    00438FD5     8DBE E8060000        lea edi,dword ptr ds:\[esi+6E8\]    ；这里，飞向光明！  
    00438FDB     8BCF                 mov ecx,edi  
    00438FDD     E8 EE89FDFF          call LiveBadu.004119D0  
    00438FE2     8D4C24 14            lea ecx,dword ptr ss:\[esp+14\]  
    00438FE6     8D5424 18            lea edx,dword ptr ss:\[esp+18\]  
    00438FEA     85C0                 test eax,eax  
    00438FEC     51                   push ecx  
    00438FED     8D4424 20            lea eax,dword ptr ss:\[esp+20\]  
    00438FF1     52                   push edx  
    00438FF2     8D4C24 28            lea ecx,dword ptr ss:\[esp+28\]  
     .......  
       
    来到了程序领空，出现了出错窗口！我们往上看，怎样才能避开这里呢，我们发现了 00438F81  jnz short LiveBadu.00438FD5，取消所有  
  
断点，下断点在 00438F7F，重复点击试下窗口的“形势判断”按钮，中断后，我们发现走到这里不跳，我们试着改为 jmp，然后 F9，“形势判  
  
断”出现了！爆破他吧！  
    
  再来看第二个，此时需要进入一个禁用“形势判断”的房间，如果没有，只好自己开房了！  
  进入房间后同样重复上面的工作，取消所有断点，下断点 bp MessageBoxA，点击“形势判断”按钮，假设房主的协议是 “本局不能使用形势  
  
分析功能”，此时会中断，出现提示窗口的，中断点同上，只是堆栈窗口换了下面：  
    
    0012CE64   001E04DE  |hOwner = 001E04DE (class='AfxWnd42',parent=00130698)  
    0012CE68   04618710  |Text = "根据双方协议，本局不能使用形势分析功能。"  
    0012CE6C   00D04F50  |Title = "Tom 对弈"  
    0012CE70   00000030  \\Style = MB\_OK|MB\_ICONEXCLAMATION|MB\_APPLMODAL  
    0012CE74   0465DCC8  
    0012CE78   004F5420  返回到 LiveBadu.004F5420 来自 <jmp.&MFC42.#4224>  
      
  F8 一路返回到 LiveBadu 程序领空，如下  
    004F539D     90                   nop  
    004F539E     90                   nop  
    004F539F     90                   nop  
    004F53A0     64:A1 00000000       mov eax,dword ptr fs:\[0\]  
    004F53A6     6A FF                push -1  
    004F53A8     68 A25C5200          push LiveBadu.00525CA2  
    004F53AD     50                   push eax  
    004F53AE     A1 08065600          mov eax,dword ptr ds:\[560608\]  
    004F53B3     64:8925 00000000     mov dword ptr fs:\[0\],esp  
    004F53BA     83EC 40              sub esp,40  
    004F53BD     55                   push ebp  
    004F53BE     56                   push esi  
    004F53BF     8BF1                 mov esi,ecx  
    004F53C1     57                   push edi  
    004F53C2     8B88 E0060000        mov ecx,dword ptr ds:\[eax+6E0\]  
    004F53C8     85C9                 test ecx,ecx  
    004F53CA     75 65                jnz short LiveBadu.004F5431    ; jmp，这里就是爆破点  
    004F53CC     8B86 9C000000        mov eax,dword ptr ds:\[esi+9C\]  
    004F53D2     85C0                 test eax,eax  
    004F53D4     75 5B                jnz short LiveBadu.004F5431   ; jmp，这里就是爆破点  
    004F53D6     8B0D 00065600        mov ecx,dword ptr ds:\[560600\]                    
    004F53DC     8B81 4C040000        mov eax,dword ptr ds:\[ecx+44C\]  
    004F53E2     85C0                 test eax,eax  
    004F53E4     75 4B                jnz short LiveBadu.004F5431     ; jmp，这里就是爆破点  
    004F53E6     8D4C24 10            lea ecx,dword ptr ss:\[esp+10\]  
    004F53EA     E8 A9620100          call <jmp.&MFC42.#540>  
    004F53EF     68 26030000          push 326  
    004F53F4     8D4C24 14            lea ecx,dword ptr ss:\[esp+14\]  
    004F53F8     C74424 58 00000000   mov dword ptr ss:\[esp+58\],0  
    004F5400     E8 49640100          call <jmp.&MFC42.#4160>  
    004F5405     8B15 00065600        mov edx,dword ptr ds:\[560600\]                  
    004F540B     6A 30                push 30  
    004F540D     8BCE                 mov ecx,esi  
    004F540F     8B82 34020000        mov eax,dword ptr ds:\[edx+234\]  
    004F5415     50                   push eax  
    004F5416     8B4424 18            mov eax,dword ptr ss:\[esp+18\]  
    004F541A     50                   push eax  
    004F541B     E8 1E650100          call <jmp.&MFC42.#4224>   ; 从 bp MessageBoxA 断点返回到这里程序领空，出现提示窗口!  
    004F5420     C74424 54 FFFFFFFF   mov dword ptr ss:\[esp+54\],-1  
    004F5428     8D4C24 10            lea ecx,dword ptr ss:\[esp+10\]  
    004F542C     E9 3A020000          jmp LiveBadu.004F566B  
    004F5431     8D4C24 28            lea ecx,dword ptr ss:\[esp+28\]   ；这里，飞向光明！  
    004F5435     8DAE AC040000        lea ebp,dword ptr ds:\[esi+4AC\]  
    004F543B     51                   push ecx  
    004F543C     8BCD                 mov ecx,ebp  
    004F543E     E8 BDB5F1FF          call LiveBadu.00410A00  
    004F5443     8B86 C0000000        mov eax,dword ptr ds:\[esi+C0\]  
    004F5449     85C0                 test eax,eax  
    004F544B     74 52                je short LiveBadu.004F549F  
    004F544D     8B86 24020000        mov eax,dword ptr ds:\[esi+224\]  
    004F5453     85C0                 test eax,eax  
    004F5455     74 48                je short LiveBadu.004F549F  
    004F5457     8B15 00065600        mov edx,dword ptr ds:\[560600\]       ; LiveBadu.0055FB78  
    004F545D     8B86 98040000        mov eax,dword ptr ds:\[esi+498\]  
    004F5463     8B3D A88A5200        mov edi,dword ptr ds:\[<&MSVCRT.\_mbscmp>\]   ; MSVCRT.\_mbscmp  
      
   来到了程序领空，出现了出错窗口！我们往上看，怎样才能避开这里呢，我们发现了 004F53CA  jnz short LiveBadu.004F5431，取消所有  
  
断点，下断点在 004F53C8，重复点击“形势判断”按钮，中断后，我们发现走到这里不跳，我们试着改为 jmp，然后 F9，“形势判断”出现了  
  
！爆破他吧！   
    
\--------------------------------------------------------------------------------  
【总结】  
  没什么可总结的，找好 api 函数断点，跟踪，爆破！主要是给大家一个过程，一个思路！  
他是经常升级的，每次要改有些麻烦的！  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

 4dd466d0e1e0cc123afbe504da2462ca