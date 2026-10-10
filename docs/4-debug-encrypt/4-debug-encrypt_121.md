

# IP-Tools 2.50 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6875.htm`](https://www.pediy.com/kssd/pediy06/pediy6875.htm)

官方主页:[`www.ks-soft.net/`](http://www.ks-soft.net/)  
  
软件介绍：  
    集成了许多 TCP/IP 实用工具于一体，比如本地信息、连接信息、端口扫描、PING、TRACE、WHOIS、  
    FINGER、NSLOOKUP、Telnet 客户端、NETBIOS 信息、IP 监视器等等  
  
IP-Tools 的功能确实非常强大,ks-soft 公司的客户包括 Adobe Systems Incorporated,Intel Corporation,IBM Corporation  
等大公司,不知道是否这些公司也用 IP-Tools,刚拿到这个软件的时候没想到它的算法会很复杂  
随着分析的不断深入才发现自己对它用的算法却似乎没有见到过,这个算法具有良好的雪崩效应,以及扩散  
和混乱,有一个单向陷门函数, 同时又具有一个密钥对,一个是加密密钥，另一个是解密密钥  
它和 RSA 的大数运算很相似,注册码为 640 位,我想这种单向函数应该称为非对称算法的  
单向函数,也就是说是公钥密码学和单向函数的混合加密.如果这个算法是作者自己设计和实现的，那么  
我想他有一定的密码学知识,其注册码的验证很像中途攻击,有点碰撞的意思  
呵呵，说多啦，开始我们的算法分析之旅吧  
       
                             eXt |Cnbragon(Radio):Go go go !  
\----------------------------------------------------------------------------------------  
PeID 查看，无壳，Borland Delphi 4.0 - 5.0,Kanal 插件分析知有 Base64,但可惜的是它并没有用在注册  
算法当中，如果是 Base64，那将会是件很容易的事,可事实并非如此,它只用在最后把用户名和注册码加密  
后放入注册表中,这不是我们要研究的重点，不过你可以看看  
这里:  
HKCU\\Software\\KS-Soft\\IP-Tools\\UserName    
HKCU\\Software\\KS-Soft\\IP-Tools\\UserSNum  
还有这里:  
Software\\Microsoft\\Windows\\CurrentVersion\\Devices\\00102  
"DATA5"="..."  
"DATA6"="..."  
程序在启动的时候会有自校验.当然会从上面的这个地方读出注册码再验证一遍  
  
为了对付它，我动用了 DeDe,IDA 和 Ollydbg (Desert Eagle,AK47/B43,AWP),当然 Ollydbg 是主武器(我是我们  
队的 sniper 嘛,有空切磋切磋哦).通过 DeDe 的分析，可以很容易的断在注册处理的地方,如下  
\----------------------------------------------------------------------------------------  
00509FF0 <>|.  E8 E3C0F2FF        call <IP\_TOOLS.sub\_4360D8>                    ;  ->controls.TControl.GetText(TControl):TCaption;  
00509FF5   |.  8B45 F8            mov eax,dword ptr ss:\[ebp\-8\]  
00509FF8   |.  8D55 FC            lea edx,dword ptr ss:\[ebp\-4\]  
00509FFB <>|.  E8 88F4EFFF        call <IP\_TOOLS.sub\_409488>                    ;  ->sysutils.Trim(AnsiString):AnsiString;  
0050A000   |.  8B55 FC            mov edx,dword ptr ss:\[ebp\-4\]  
0050A003   |.  8BC7               mov eax,edi  
0050A005 <>|.  E8 FEC0F2FF        call <IP\_TOOLS.sub\_436108>                    ;  ->controls.TControl.SetText(TControl;TCaption);  
0050A00A   |.  8D55 F0            lea edx,dword ptr ss:\[ebp\-10\]  
0050A00D   |.  8B06               mov eax,dword ptr ds:\[esi\]  
0050A00F <>|.  8BB8 E8020000      mov edi,dword ptr ds:\[eax+2E8\]                ;  \*RxGIFAnimator1:TRxGIFAnimator  
0050A015   |.  8BC7               mov eax,edi  
0050A017 <>|.  E8 BCC0F2FF        call <IP\_TOOLS.sub\_4360D8>                    ;  ->controls.TControl.GetText(TControl):TCaption;  
0050A01C   |.  8B45 F0            mov eax,dword ptr ss:\[ebp\-10\]  
0050A01F   |.  8D55 F4            lea edx,dword ptr ss:\[ebp\-C\]  
0050A022 <>|.  E8 61F4EFFF        call <IP\_TOOLS.sub\_409488>                    ;  ->sysutils.Trim(AnsiString):AnsiString;  
0050A027   |.  8B55 F4            mov edx,dword ptr ss:\[ebp\-C\]  
0050A02A   |.  8BC7               mov eax,edi  
0050A02C <>|.  E8 D7C0F2FF        call <IP\_TOOLS.sub\_436108>                    ;  ->controls.TControl.SetText(TControl;TCaption);  
0050A031   |.  8D55 EC            lea edx,dword ptr ss:\[ebp\-14\]  
0050A034   |.  8B06               mov eax,dword ptr ds:\[esi\]  
0050A036 <>|.  8B80 E4020000      mov eax,dword ptr ds:\[eax+2E4\]                ;  \*Label1:TLabel  
0050A03C <>|.  E8 97C0F2FF        call <IP\_TOOLS.sub\_4360D8>                    ;  ->controls.TControl.GetText(TControl):TCaption;  
0050A041   |.  8B45 EC            mov eax,dword ptr ss:\[ebp\-14\]  
0050A044   |.  E8 E7A4FCFF        call <IP\_TOOLS.sub\_4D4530>  
0050A049   |.  84C0               test al,al  
0050A04B   |.  74 3C              je short <IP\_TOOLS.loc\_50A089>  
0050A04D   |.  68 E4A25000        push <IP\_TOOLS.aSorryYourRegis>               ;  ASCII "Sorry, your registration name ("  
0050A052   |.  8D55 E4            lea edx,dword ptr ss:\[ebp\-1C\]  
0050A055   |.  8B06               mov eax,dword ptr ds:\[esi\]  
0050A057 <>|.  8B80 E4020000      mov eax,dword ptr ds:\[eax+2E4\]                ;  \*Label1:TLabel  
0050A05D <>|.  E8 76C0F2FF        call <IP\_TOOLS.sub\_4360D8>                    ;  ->controls.TControl.GetText(TControl):TCaption;  
0050A062   |.  FF75 E4            push dword ptr ss:\[ebp\-1C\]  
0050A065   |.  68 0CA35000        push <IP\_TOOLS.aIsFoundOnTheBl>               ;  ASCII ") is found on the "Black List".  
  
"  
0050A06A   |.  68 38A35000        push <IP\_TOOLS.aIfYouHaveAnyQu>               ;  ASCII "If you have any questions, please, contact KS-Soft: line1@ks-soft.net; line2@ks-soft.net"  
0050A06F   |.  8D45 E8            lea eax,dword ptr ss:\[ebp\-18\]  
0050A072   |.  BA 04000000        mov edx,4                                     ;  上面是得到用户名并检查是否在黑名单中，黑名单 N 多:D  
0050A077 <>|.  E8 28A2EFFF        call <IP\_TOOLS.sub\_4042A4>                    ;  ->system.@LStrCatN;  
0050A07C   |.  8B45 E8            mov eax,dword ptr ss:\[ebp\-18\]  
0050A07F <>|.  E8 E448F5FF        call <IP\_TOOLS.sub\_45E968>                    ;  ->dialogs.ShowMessage(AnsiString);  
0050A084   |.  E9 EF010000        jmp <IP\_TOOLS.loc\_50A278>  
0050A089 <>|>  8D55 E0            lea edx,dword ptr ss:\[ebp\-20\]                 ;  loc\_50A089  
0050A08C   |.  8B06               mov eax,dword ptr ds:\[esi\]  
0050A08E <>|.  8B80 E4020000      mov eax,dword ptr ds:\[eax+2E4\]                ;  \*Label1:TLabel  
0050A094 <>|.  E8 3FC0F2FF        call <IP\_TOOLS.sub\_4360D8>                    ;  ->controls.TControl.GetText(TControl):TCaption;  
0050A099   |.  837D E0 00         cmp dword ptr ss:\[ebp\-20\],0  
0050A09D   |.  0F84 CB010000      je <IP\_TOOLS.loc\_50A26E>  
0050A0A3   |.  8D55 DC            lea edx,dword ptr ss:\[ebp\-24\]  
0050A0A6   |.  8B06               mov eax,dword ptr ds:\[esi\]  
0050A0A8 <>|.  8B80 E8020000      mov eax,dword ptr ds:\[eax+2E8\]                ;  \*RxGIFAnimator1:TRxGIFAnimator  
0050A0AE <>|.  E8 25C0F2FF        call <IP\_TOOLS.sub\_4360D8>                    ;  ->controls.TControl.GetText(TControl):TCaption;  
0050A0B3   |.  837D DC 00         cmp dword ptr ss:\[ebp\-24\],0  
0050A0B7   |.  0F84 B1010000      je <IP\_TOOLS.loc\_50A26E>  
0050A0BD   |.  8D55 D8            lea edx,dword ptr ss:\[ebp\-28\]  
0050A0C0   |.  8B06               mov eax,dword ptr ds:\[esi\]  
0050A0C2 <>|.  8B80 E4020000      mov eax,dword ptr ds:\[eax+2E4\]                ;  \*Label1:TLabel  
0050A0C8 <>|.  E8 0BC0F2FF        call <IP\_TOOLS.sub\_4360D8>                    ;  ->controls.TControl.GetText(TControl):TCaption;  
0050A0CD   |.  8B45 D8            mov eax,dword ptr ss:\[ebp\-28\]  
0050A0D0 <>|.  E8 67C0FEFF        call <IP\_TOOLS.sub\_4F613C>                    ;  ->:TDebugForm.\_PROC\_004F613C()  
0050A0D5   |.  8BF8               mov edi,eax                                   ;  上面这个 call 是对用户名进行加密处理  
0050A0D7   |.  8D55 D4            lea edx,dword ptr ss:\[ebp\-2C\]  
0050A0DA   |.  8B06               mov eax,dword ptr ds:\[esi\]  
0050A0DC <>|.  8B80 E8020000      mov eax,dword ptr ds:\[eax+2E8\]                ;  \*RxGIFAnimator1:TRxGIFAnimator  
0050A0E2 <>|.  E8 F1BFF2FF        call <IP\_TOOLS.sub\_4360D8>                    ;  ->controls.TControl.GetText(TControl):TCaption;  
0050A0E7   |.  8B45 D4            mov eax,dword ptr ss:\[ebp\-2C\]  
0050A0EA <>|.  E8 E9C0FEFF        call <IP\_TOOLS.sub\_4F61D8>                    ;  ->:TDebugForm.\_PROC\_004F61D8()  
0050A0EF   |.  66:3BF8            cmp di,ax                                     ;  上面这个 call 是对注册码进行解密处理  
0050A0F2   |.  0F85 76010000      jnz <IP\_TOOLS.loc\_50A26E>  
0050A0F8   |.  A1 9CA95400        mov eax,dword ptr ds:\[54A99C\]  
0050A0FD   |.  BA FF010000        mov edx,1FF  
0050A102   |.  E8 19C0FEFF        call <IP\_TOOLS.HashToInt>                     ;得到的 640 位大数进行处理，这个是用户名的  
0050A107   |.  8BF8               mov edi,eax  
0050A109   |.  A1 20A85400        mov eax,dword ptr ds:\[54A820\]  
0050A10E   |.  BA FF010000        mov edx,1FF  
0050A113   |.  E8 08C0FEFF        call <IP\_TOOLS.HashToInt>                     ;得到的 640 位大数进行处理，这个是注册码的  
0050A118   |.  3BF8               cmp edi,eax                                   ;结果相同吗？不同就出错,相同就继续检验  
0050A11A   |.  0F85 4E010000      jnz <IP\_TOOLS.loc\_50A26E>  
0050A120   |.  A1 9CA95400        mov eax,dword ptr ds:\[54A99C\]  
0050A125   |.  83C0 07            add eax,7  
0050A128   |.  8B00               mov eax,dword ptr ds:\[eax\]  
0050A12A   |.  8B15 20A85400      mov edx,dword ptr ds:\[54A820\]                 ;  IP\_TOOLS.0054CB00  
0050A130   |.  83C2 07            add edx,7  
0050A133   |.  3B02               cmp eax,dword ptr ds:\[edx\]                    ;这里  
0050A135   |.  0F85 33010000      jnz <IP\_TOOLS.loc\_50A26E>  
0050A13B   |.  8D55 D0            lea edx,dword ptr ss:\[ebp\-30\]  
0050A13E   |.  8B06               mov eax,dword ptr ds:\[esi\]  
0050A140 <>|.  8B80 E4020000      mov eax,dword ptr ds:\[eax+2E4\]                ;  \*Label1:TLabel  
0050A146 <>|.  E8 8DBFF2FF        call <IP\_TOOLS.sub\_4360D8>                    ;  ->controls.TControl.GetText(TControl):TCaption;  
0050A14B   |.  8B55 D0            mov edx,dword ptr ss:\[ebp\-30\]  
0050A14E   |.  A1 C4AB5400        mov eax,dword ptr ds:\[54ABC4\]  
0050A153 <>|.  E8 609EEFFF        call <IP\_TOOLS.sub\_403FB8>                    ;  ->system.@LStrAsg;  
0050A158   |.  8D55 CC            lea edx,dword ptr ss:\[ebp\-34\]  
0050A15B   |.  8B06               mov eax,dword ptr ds:\[esi\]  
0050A15D <>|.  8B80 E8020000      mov eax,dword ptr ds:\[eax+2E8\]                ;  \*RxGIFAnimator1:TRxGIFAnimator  
0050A163 <>|.  E8 70BFF2FF        call <IP\_TOOLS.sub\_4360D8>                    ;  ->controls.TControl.GetText(TControl):TCaption;  
0050A168   |.  8B55 CC            mov edx,dword ptr ss:\[ebp\-34\]  
0050A16B   |.  A1 64A85400        mov eax,dword ptr ds:\[54A864\]  
0050A170 <>|.  E8 439EEFFF        call <IP\_TOOLS.sub\_403FB8>                    ;  ->system.@LStrAsg;  
0050A175   |.  8BC3               mov eax,ebx  
0050A177   |.  E8 B0F9FFFF        call <IP\_TOOLS.sub\_509B2C>  
0050A17C   |.  B8 9CA35000        mov eax,<IP\_TOOLS.aThankYouForReg>            ;  ASCII "Thank you for registering IP-Tools.  
Please, restart the program."  
0050A181 <>|.  E8 E247F5FF        call <IP\_TOOLS.sub\_45E968>                    ;  ->dialogs.ShowMessage(AnsiString);  
0050A186   |.  B2 01              mov dl,1  
0050A188   |.  A1 E0634500        mov eax,dword ptr ds:\[<off\_4563E0>\]  
0050A18D <>|.  E8 BAC3F4FF        call <IP\_TOOLS.sub\_45654C>                    ;  ->registry.TRegistry.Create(TRegistry;boolean);overload;  
0050A192   |.  8BD8               mov ebx,eax  
0050A194   |.  B1 01              mov cl,1  
0050A196   |.  BA E8A35000        mov edx,<IP\_TOOLS.aSoftwareKsSo\_3>            ;  ASCII "\\Software\\KS-Soft\\IP-Tools"  
0050A19B   |.  8BC3               mov eax,ebx  
0050A19D <>|.  E8 CAC5F4FF        call <IP\_TOOLS.sub\_45676C>                    ;  ->registry.TRegistry.OpenKey(TRegistry;AnsiString;Boolean):Boolean;  
0050A1A2   |.  A1 C4AB5400        mov eax,dword ptr ds:\[54ABC4\]  
0050A1A7   |.  8B00               mov eax,dword ptr ds:\[eax\]  
0050A1A9 <>|.  E8 36A0EFFF        call <IP\_TOOLS.sub\_4041E4>                    ;  ->system.@LStrLen:Integer;<+>  
0050A1AE   |.  8D55 C8            lea edx,dword ptr ss:\[ebp\-38\]  
0050A1B1   |.  E8 1EA9FCFF        call <IP\_TOOLS.sub\_4D4AD4>  
0050A1B6   |.  8B4D C8            mov ecx,dword ptr ss:\[ebp\-38\]                 ;  下面就是用 Base64 把 UserName 和 UserNum 存入注册表  
0050A1B9   |.  BA 0CA45000        mov edx,<IP\_TOOLS.aUsername\_1>                ;  ASCII "UserName"  
0050A1BE   |.  8BC3               mov eax,ebx  
0050A1C0 <>|.  E8 43CAF4FF        call <IP\_TOOLS.sub\_456C08>                    ;  ->registry.TRegistry.WriteString(TRegistry;AnsiString;AnsiString);  
0050A1C5   |.  A1 64A85400        mov eax,dword ptr ds:\[54A864\]  
0050A1CA   |.  8B00               mov eax,dword ptr ds:\[eax\]  
0050A1CC <>|.  E8 13A0EFFF        call <IP\_TOOLS.sub\_4041E4>                    ;  ->system.@LStrLen:Integer;<+>  
0050A1D1   |.  8D55 C4            lea edx,dword ptr ss:\[ebp\-3C\]  
0050A1D4   |.  E8 FBA8FCFF        call <IP\_TOOLS.sub\_4D4AD4>  
0050A1D9   |.  8B4D C4            mov ecx,dword ptr ss:\[ebp\-3C\]  
0050A1DC   |.  BA 20A45000        mov edx,<IP\_TOOLS.aUsersnum>                  ;  ASCII "UserSNum"  
  
....................................  
\---------------------------------------------------------------------------------------------------------------------------  
0050A0D0 <>|.  E8 67C0FEFF        call <IP\_TOOLS.sub\_4F613C>   
{  
004F613C <>/$  55                 push ebp                                      ;  sub\_4F613C  
004F613D   |.  8BEC               mov ebp,esp  
004F613F   |.  81C4 FCFDFFFF      add esp,-204  
004F6145   |.  53                 push ebx  
004F6146   |.  8945 FC            mov dword ptr ss:\[ebp\-4\],eax  
004F6149   |.  8B45 FC            mov eax,dword ptr ss:\[ebp\-4\]  
004F614C   |.  E8 47E2F0FF        call <IP\_TOOLS.sub\_404398>  
004F6151   |.  33C0               xor eax,eax  
004F6153   |.  55                 push ebp  
004F6154   |.  68 C9614F00        push <IP\_TOOLS.loc\_4F61C9>  
004F6159   |.  64:FF30            push dword ptr fs:\[eax\]  
004F615C   |.  64:8920            mov dword ptr fs:\[eax\],esp  
\---------------------------------------------------------------------  
把用户名的长度送给 ebx,如果长度大于 64 的话就以 64 来计算  
004F615F   |.  8B45 FC            mov eax,dword ptr ss:\[ebp\-4\]  
004F6162   |.  E8 7DE0F0FF        call <IP\_TOOLS.sub\_4041E4>  
004F6167   |.  83F8 40            cmp eax,40  
004F616A   |.  7E 07              jle short <IP\_TOOLS.loc\_4F6173>  
004F616C   |.  BB 40000000        mov ebx,40  
004F6171   |.  EB 0A              jmp short <IP\_TOOLS.loc\_4F617D>  
004F6173 <>|>  8B45 FC            mov eax,dword ptr ss:\[ebp\-4\]                  ;  loc\_4F6173  
004F6176   |.  E8 69E0F0FF        call <IP\_TOOLS.sub\_4041E4>  
004F617B   |.  8BD8               mov ebx,eax  
004F617D <>|>  8D45 FC            lea eax,dword ptr ss:\[ebp\-4\]                  ;  loc\_4F617D  
004F6180   |.  E8 2FE2F0FF        call <IP\_TOOLS.sub\_4043B4>  
004F6185   |.  8D95 FCFDFFFF      lea edx,dword ptr ss:\[ebp\-204\]  
004F618B   |.  8BCB               mov ecx,ebx  
004F618D   |.  E8 62C8F0FF        call <IP\_TOOLS.sub\_4029F4>  
004F6192   |.  8D85 FCFDFFFF      lea eax,dword ptr ss:\[ebp\-204\]  
004F6198   |.  50                 push eax                                      ; /Arg5 pointer to name  
004F6199   |.  68 00C95400        push IP\_TOOLS.0054C900                        ; |Arg4 = 0054C900  
004F619E   |.  53                 push ebx                                      ; |Arg3 namelen  
004F619F   |.  68 02805400        push IP\_TOOLS.00548002                        ; |Arg2 = 00548002 固定参数 1  
004F61A4   |.  68 02815400        push IP\_TOOLS.00548102                        ; |Arg1 = 00548102 公钥,用于加密  
004F61A9   |.  E8 3C060000        call <IP\_TOOLS.sub\_4F67EA>                    ; \\IP\_TOOLS.004F67EA  
004F61AE   |.  0FBEC0             movsx eax,al  
}  
  
00548002 处的固定参数为一个长度为 2 的 WORD 数组,WORD constants={0x0003,0x0008};  
00548102 处的公钥是一个 640 位的大数,在内存中表示如下:  
\--------------------------------------------------------------------------  
00548102  A1 8B 33 A7 FA CE 05 97 99 EA 8F 95 AD 8E F5 7C  3?棛陱暛庻|  
00548112  97 09 D6 3B F3 85 6D CC EA 56 FE 14 FE 22 FB 86  ??髤 m 剃 V??麊  
00548122  F7 A5 2F 00 C5 86 78 3B 05 DA CE 9E 8E 40 10 E5  鳐/.艈 x;谖瀻@  
00548132  77 6F E2 71 80 47 99 C7 60 85 DE 1B 1D E9 EB 9D  wo 鈗€G 櫱\`呣殡  
00548142  79 C2 7E 40 40 91 EB 2A 38 C9 BB C2 EC EC 79 C9  y 聗@@戨\*8 苫蚂靬  
\--------------------------------------------------------------------------  
这个公钥将用于大数运算,是以一个 WORD 为单位  
arg4=0054C900 用于存放对用户名处理的结果  
上面是用户名的，再来看看注册码的  
\-----------------------------------------------------------------------------------------------  
{  
004F61D8 <>/$  55                 push ebp                                      ;  sub\_4F61D8  
004F61D9   |.  8BEC               mov ebp,esp  
004F61DB   |.  81C4 FCFDFFFF      add esp,-204  
004F61E1   |.  53                 push ebx  
004F61E2   |.  8945 FC            mov dword ptr ss:\[ebp\-4\],eax  
004F61E5   |.  8B45 FC            mov eax,dword ptr ss:\[ebp\-4\]  
004F61E8   |.  E8 ABE1F0FF        call <IP\_TOOLS.sub\_404398>  
004F61ED   |.  33C0               xor eax,eax  
004F61EF   |.  55                 push ebp  
004F61F0   |.  68 52624F00        push <IP\_TOOLS.loc\_4F6252>  
004F61F5   |.  64:FF30            push dword ptr fs:\[eax\]  
004F61F8   |.  64:8920            mov dword ptr fs:\[eax\],esp  
004F61FB   |.  8D95 FCFDFFFF      lea edx,dword ptr ss:\[ebp\-204\]  
004F6201   |.  B9 FF010000        mov ecx,1FF  
004F6206   |.  8B45 FC            mov eax,dword ptr ss:\[ebp\-4\]  
004F6209   |.  E8 62FEFFFF        call <IP\_TOOLS.sub\_4F6070>  
004F620E   |.  8D85 FCFDFFFF      lea eax,dword ptr ss:\[ebp\-204\]  
004F6214   |.  50                 push eax                                      ;Arg5 pointer to sn  
004F6215   |.  68 00CB5400        push IP\_TOOLS.0054CB00                        ;Arg4 存放结果  
004F621A   |.  8B45 FC            mov eax,dword ptr ss:\[ebp\-4\]  
004F621D   |.  E8 C2DFF0FF        call <IP\_TOOLS.sub\_4041E4>  
004F6222   |.  D1F8               sar eax,1  
004F6224   |.  79 03              jns short <IP\_TOOLS.loc\_4F6229>  
004F6226   |.  83D0 00            adc eax,0  
004F6229 <>|>  50                 push eax                                      ; |Arg3 snlen  
004F622A   |.  68 06825400        push IP\_TOOLS.00548206                        ; |Arg2 = 00548206 固定参数 2  
004F622F   |.  68 06835400        push IP\_TOOLS.00548306                        ; |Arg1 = 00548306 私钥，用于解密  
004F6234   |.  E8 B1050000        call <IP\_TOOLS.sub\_4F67EA>                    ; \\IP\_TOOLS.004F67EA  
004F6239   |.  0FBED8             movsx ebx,al  
}  
  
00548206 处是另外一组固定参数，但是它是一个 40 个 WORD 数组，在内存中表示如下:  
00548206  CD 4D 02 EB CD B6 98 A2 DF FF 2E C3 42 FD 4B 24  蚆胪稑⑦�.肂齂$  
00548216  6C 65 47 3D 3B 12 58 58 B7 0A 03 E9 78 C2 C3 97  leG=;XX?閤旅  
00548226  52 D4 D9 7E 06 7B A5 71 FA 32 40 3D 84 75 89 C0  R 再~{?@=剈壚  
00548236  35 2E F8 08 B4 8D 26 DB 03 55 EE C1 DE 0D BC F3  5.?磵&?U 盍?俭  
00548246  47 DC A9 D1 9B D2 11 63 DE 34 B2 F5 EB B9 0F 1B  G 堠褯?c?蝉牍  
\----------------------------------------------------------------------  
私钥如下:  
\--------------------------------------------------------------------  
00548306  99 C3 C7 0E E6 7B AE 25 D9 5D 38 3A 43 A3 55 63  櫭??#123;?賋 8:Cc  
00548316  4E 3A 09 34 AE 1F 8E C5 E8 BB D6 CD F1 EA 44 72  N:.4?幣杌滞耜 Dr  
00548326  B5 5F B3 35 1B 06 2C 22 9C 01 5C 8C DC AC 26 CA  礯?,"?\\屲?  
00548336  4B 66 63 7A A7 3D E2 45 3A 10 EB AF 12 B6 ED 6B  Kfcz?釫:氙俄 k  
00548346  75 1F 65 44 FD FF E0 9B F2 B1 2C CA 43 71 5B E7  ueD?鄾虮,蔆 q\[  
\---------------------------------------------------------------------  
虽然得到了两组数据，但是由于不知道具体的算法，所以还是要详细的分析其注册算法,对用户名和注册码的处理最终都调用了同一个 call  
但是却使用了不同的密钥和参数，所以我才称它为非对称算法;  
\--------------------------------------------------------------------------  
{  
004F67EA <>/$  55                 push ebp                                      ;  sub\_4F67EA  
                                     .  
                                     .  
                                     .  
004F681C   |.  83C4 0C            add esp,0C  
004F681F   |.  56                 push esi                                      ; /Arg4 公钥  
004F6820   |.  FF75 0C            push dword ptr ss:\[ebp+C\]                     ; |Arg3 固定参数 1  
004F6823   |.  68 04CD5400        push IP\_TOOLS.0054CD04                        ; |Arg2 = 0054CD04 name  
004F6828   |.  68 84CD5400        push IP\_TOOLS.0054CD84                        ; |Arg1 = 0054CD84 存放结果  
004F682D   |.  E8 1B110000        call <IP\_TOOLS.sub\_4F794D>                    ; \\IP\_TOOLS.004F794D  
004F6832   |.  83C4 10            add esp,10  
                                     .  
                                     .  
                                     .  
         上面这个是关键 call，其它的都无关紧要  
}  
{  
004F7A2F   |.  53                 push ebx                                      ; /Arg1 = 00548102 公钥  
004F7A30   |.  E8 6EF6FFFF        call <IP\_TOOLS.sub\_4F70A3>                    ; \\IP\_TOOLS.004F70A3 生成 16 个子密钥  
\-----------------------------------------------------------------------------------------------------  
004F70A3 <>/$  55                 push ebp                                      ;  sub\_4F70A3  
004F70A4   |.  8BEC               mov ebp,esp  
004F70A6   |.  53                 push ebx  
004F70A7   |.  56                 push esi  
004F70A8   |.  8B45 08            mov eax,dword ptr ss:\[ebp+8\]  
004F70AB   |.  BE 0C8C5400        mov esi,IP\_TOOLS.00548C0C  
004F70B0   |.  8906               mov dword ptr ds:\[esi\],eax  
004F70B2   |.  0FBF15 08845400    movsx edx,word ptr ds:\[548408\]  
004F70B9   |.  03D2               add edx,edx  
004F70BB   |.  03C2               add eax,edx  
004F70BD   |.  83C0 FE            add eax,-2  
004F70C0   |.  66:8B08            mov cx,word ptr ds:\[eax\]  
004F70C3   |.  66:890D 508C5400   mov word ptr ds:\[548C50\],cx  
004F70CA   |.  83E8 02            sub eax,2  
004F70CD   |.  66:8B00            mov ax,word ptr ds:\[eax\]  
004F70D0   |.  66:A3 728C5400     mov word ptr ds:\[548C72\],ax  
004F70D6   |.  66:BB 0100         mov bx,1  
004F70DA <>|>  0FBFC3             /movsx eax,bx                                 ;  loc\_4F70DA  
004F70DD   |.  FF7486 FC          |push dword ptr ds:\[esi+eax\*4-4\]              ; /Arg2  
004F70E1   |.  0FBFD3             |movsx edx,bx                                 ; |  
004F70E4   |.  FF3496             |push dword ptr ds:\[esi+edx\*4\]                ; |Arg1  
004F70E7   |.  E8 8AFAFFFF        |call <IP\_TOOLS.memcpy>                       ; \\IP\_TOOLS.004F6B76  
004F70EC   |.  83C4 08            |add esp,8  
004F70EF   |.  6A 00              |push 0                                       ; /Arg2 = 00000000  
004F70F1   |.  0FBFCB             |movsx ecx,bx                                 ; |  
004F70F4   |.  FF348E             |push dword ptr ds:\[esi+ecx\*4\]                ; |Arg1  
004F70F7   |.  E8 27F9FFFF        |call <IP\_TOOLS.SHL1>                         ; \\IP\_TOOLS.004F6A23  
\----------------------------------------------------------------------------------------------------  
004F6A23 <>/$  55                 push ebp                                      ;  SHL1  
004F6A24   |.  8BEC               mov ebp,esp                                   ;  对密钥们左移也就是变为 2 倍  
004F6A26   |.  53                 push ebx                                      ;  但是这个密钥是 672 位的大数  
004F6A27   |.  56                 push esi                                      ;  我称之为 subKey4name  
004F6A28   |.  8B4D 0C            mov ecx,dword ptr ss:\[ebp+C\]  
004F6A2B   |.  8B45 08            mov eax,dword ptr ss:\[ebp+8\]  
004F6A2E   |.  66:8B35 08845400   mov si,word ptr ds:\[548408\]  
004F6A35   |.  EB 1A              jmp short <IP\_TOOLS.loc\_4F6A51>  
004F6A37 <>|>  66:8338 00         /cmp word ptr ds:\[eax\],0                      ;  loc\_4F6A37  
004F6A3B   |.  0F9CC2             |setl dl  
004F6A3E   |.  83E2 01            |and edx,1  
004F6A41   |.  66:D120            |shl word ptr ds:\[eax\],1  
004F6A44   |.  84C9               |test cl,cl  
004F6A46   |.  74 04              |je short <IP\_TOOLS.loc\_4F6A4C>  
004F6A48   |.  66:8308 01         |or word ptr ds:\[eax\],1  
004F6A4C <>|>  8BCA               |mov ecx,edx                                  ;  loc\_4F6A4C  
004F6A4E   |.  83C0 02            |add eax,2  
004F6A51 <>|>  8BDE                mov ebx,esi                                  ;  loc\_4F6A51  
004F6A53   |.  66:83C6 FF         |add si,0FFFF  
004F6A57   |.  66:85DB            |test bx,bx  
004F6A5A   |.^ 75 DB              \\jnz short <IP\_TOOLS.loc\_4F6A37>  
004F6A5C   |.  8BC2               mov eax,edx  
  
\------------------------------------------------------------------------------------------------------  
004F70FC   |.  83C4 08            |add esp,8  
004F70FF   |.  0FBFC3             |movsx eax,bx  
004F7102   |.  8B0486             |mov eax,dword ptr ds:\[esi+eax\*4\]  
004F7105   |.  0FBF15 08845400    |movsx edx,word ptr ds:\[548408\]  
004F710C   |.  03D2               |add edx,edx  
004F710E   |.  03C2               |add eax,edx  
004F7110   |.  83C0 FE            |add eax,-2  
004F7113   |.  0FBFCB             |movsx ecx,bx  
004F7116   |.  66:8B10            |mov dx,word ptr ds:\[eax\]  
004F7119   |.  66:89144D 508C5400 |mov word ptr ds:\[ecx\*2+548C50\],dx  ;这是把子密钥的最高位放到 548C50 地址处  
004F7121   |.  83E8 02            |sub eax,2  
004F7124   |.  0FBFCB             |movsx ecx,bx  
004F7127   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F712A   |.  66:89044D 728C5400 |mov word ptr ds:\[ecx\*2+548C72\],ax  ;这是把子密钥的次高位放到 548C72 地址处  
004F7132   |.  43                 |inc ebx  
004F7133   |.  66:83FB 11         |cmp bx,11  
004F7137   |.^ 7C A1              \\jl short <IP\_TOOLS.loc\_4F70DA>  
004F7139   |.  33C0               xor eax,eax  
004F713B   |.  5E                 pop esi  
004F713C   |.  5B                 pop ebx  
004F713D   |.  5D                 pop ebp  
004F713E   \\.  C3                 retn  
  
}  
\-----------------------------------------------------------------  
上面就要用到大数运算的知识了，可以参考坛子的"RSA 与大数运算",这里是一个 672 位的大数，我用了一个 42 个字的数组来表示这样一个大数  
如下，比如说公钥:  
#define Len 42  
  
WORD wkey4name1\[Len\]={0x8BA1,0xA733,0xCEFA,0x9705,0xEA99,0x958F,0x8EAD,0x7CF5,  
          0x0997,0x3BD6,0x85F3,0xCC6D,0x56ea,0x14FE,0x22FE,0x86FB,  
          0xA5F7,0x002F,0x86C5,0x3B78,0xDA05,0x9ECE,0x408E,0xE510,  
                0x6F77,0x71E2,0x4780,0xC799,0x8560,0x1BDE,0xE91D,0x9DEB,  
                0xC279,0x407E,0x9140,0x2AEB,0xC938,0xC2BB,0xECEC,0xC979,  
                0x0000,0x0000};  
而左移运算根据对汇编的分析可以写出如下的 C 代码:  
  
void WORDSHL(WORD source\[\],WORD dest\[\],int length)  
{  
  DWORD result=0; //因为 DWORD 的存在所以比较成为可能  
  int carry=0;    //进位标志  
  int d=0;  
  int j;  
  for(j=0;j<length;j++)  
  {  
    result=(source\[j\]<<1);  
    if(result>0xFFFF)  
      carry=1;  
    else   
      carry=0;  
    dest\[j\]=(source\[j\]<<1);  
    if(d==1)  
    {  
      dest\[j\]=(dest\[j\]|1); 加上进位  
    }  
    d=carry;  
  }  
}  
  
关于大数运算，我想那篇"RSA 与大数运算"确实不错,兄弟们一定要看看  
\------------------------------------------------------------------------------------------  
生成子密钥后  
004F7A95   |.  66:D1EB            shr bx,1                                     ;bx 初始化为 0x8000  
004F7A98   |.  66:85DB            test bx,bx  
004F7A9B   |.  75 61              jnz short <IP\_TOOLS.loc\_4F7AFE>  
004F7A9D   |.  66:BB 0080         mov bx,8000  
004F7AA1   |.  83EF 02            sub edi,2  
004F7AA4   |.  EB 58              jmp short <IP\_TOOLS.loc\_4F7AFE>  
004F7AA6 <>|>  56                 /push esi                                     ; /loc\_4F7AA6  
004F7AA7   |.  56                 |push esi                                     ; |Arg2  
004F7AA8   |.  8D85 78FFFFFF      |lea eax,dword ptr ss:\[ebp\-88\]                ; |  
004F7AAE   |.  50                 |push eax                                     ; |Arg1  
004F7AAF   |.  E8 8BF6FFFF        |call <IP\_TOOLS.kernel>                       ; \\IP\_TOOLS.004F713F 核心 Function  
004F7AB4   |.  83C4 0C            |add esp,0C  
004F7AB7   |.  8D95 78FFFFFF      |lea edx,dword ptr ss:\[ebp\-88\]  
004F7ABD   |.  52                 |push edx                                     ; /Arg2  
004F7ABE   |.  56                 |push esi                                     ; |Arg1  
004F7ABF   |.  E8 B2F0FFFF        |call <IP\_TOOLS.memcpy>                       ; \\IP\_TOOLS.004F6B76  
004F7AC4   |.  83C4 08            |add esp,8                                    ; 把上面计算出来的结果存入 esi  
004F7AC7   |.  66:851F            |test word ptr ds:\[edi\],bx                    ;这里 edi 就指向固定参数 1  
004F7ACA   |.  74 23              |je short <IP\_TOOLS.loc\_4F7AEF>               ;和 bx 相与不为 0 的话就再做一次  
004F7ACC   |.  FF75 0C            |push dword ptr ss:\[ebp+C\]                    ; /Arg3  
004F7ACF   |.  56                 |push esi                                     ; |Arg2  
004F7AD0   |.  8D8D 78FFFFFF      |lea ecx,dword ptr ss:\[ebp\-88\]                ; |  
004F7AD6   |.  51                 |push ecx                                     ; |Arg1  
004F7AD7   |.  E8 63F6FFFF        |call <IP\_TOOLS.kernel>                       ; \\IP\_TOOLS.004F713F  
004F7ADC   |.  83C4 0C            |add esp,0C  
004F7ADF   |.  8D85 78FFFFFF      |lea eax,dword ptr ss:\[ebp\-88\]  
004F7AE5   |.  50                 |push eax                                     ; /Arg2  
004F7AE6   |.  56                 |push esi                                     ; |Arg1  
004F7AE7   |.  E8 8AF0FFFF        |call <IP\_TOOLS.memcpy>                       ; \\IP\_TOOLS.004F6B76  
004F7AEC   |.  83C4 08            |add esp,8  
004F7AEF <>|>  66:D1EB            |shr bx,1                                     ;  loc\_4F7AEF  
004F7AF2   |.  66:85DB            |test bx,bx                                   ; 一轮共 16 次循环  
004F7AF5   |.  75 07              |jnz short <IP\_TOOLS.loc\_4F7AFE>   
004F7AF7   |.  66:BB 0080         |mov bx,8000  
004F7AFB   |.  83EF 02            |sub edi,2  
004F7AFE <>|>  8B45 FC             mov eax,dword ptr ss:\[ebp\-4\]                 ;  这里是 0x1F, 这好像也是一个不变的量  
004F7B01   |.  8345 FC FF         |add dword ptr ss:\[ebp\-4\],-1  
004F7B05   |.  85C0               |test eax,eax  
004F7B07   |.^ 75 9D              \\jnz short <IP\_TOOLS.loc\_4F7AA6>  
004F7B09   |.  6A 00              push 0                                        ; /Arg2 = 00000000  
004F7B0B   |.  8D95 78FFFFFF      lea edx,dword ptr ss:\[ebp\-88\]                 ; |  
004F7B11   |.  52                 push edx                                      ; |Arg1  
004F7B12   |.  E8 85F0FFFF        call <IP\_TOOLS.memset>                        ; \\IP\_TOOLS.004F6B9C  
  
}  
\------------------------------------------------------------------------------------------------------  
004F7AAF   |.  E8 8BF6FFFF        |call <IP\_TOOLS.kernel>                       ; \\IP\_TOOLS.004F713F  
{  
004F713F <>/$  55                 push ebp                                      ;  sub\_4F713F  
004F7140   |.  8BEC               mov ebp,esp  
004F7142   |.  83C4 F8            add esp,-8  
004F7145   |.  53                 push ebx  
004F7146   |.  56                 push esi  
004F7147   |.  57                 push edi  
004F7148   |.  8B7D 10            mov edi,dword ptr ss:\[ebp+10\]  
004F714B   |.  8B5D 08            mov ebx,dword ptr ss:\[ebp+8\]  
004F714E   |.  FF75 0C            push dword ptr ss:\[ebp+C\]                     ; /Arg2  
004F7151   |.  68 14945400        push IP\_TOOLS.00549414                        ; |Arg1 = 00549414  
004F7156   |.  E8 07FFFFFF        call <IP\_TOOLS.sub\_4F7062>                    ; \\IP\_TOOLS.004F7062  
\----------------------------------------------------------------------------------------------------  
004F7062 <>/$  55                 push ebp                                      ;  这里是对消息进行处理  
004F7063   |.  8BEC               mov ebp,esp                                   ;  生成 16 个大数我称之为  
004F7065   |.  53                 push ebx                                      ; SubKey4Key  
004F7066   |.  56                 push esi  
004F7067   |.  8B75 08            mov esi,dword ptr ss:\[ebp+8\]  
004F706A   |.  8B45 0C            mov eax,dword ptr ss:\[ebp+C\]  
004F706D   |.  8906               mov dword ptr ds:\[esi\],eax  
004F706F   |.  66:BB 0100         mov bx,1  
004F7073 <>|>  0FBFC3             /movsx eax,bx                                 ;  loc\_4F7073  
004F7076   |.  FF7486 FC          |push dword ptr ds:\[esi+eax\*4-4\]              ; /Arg2  
004F707A   |.  0FBFD3             |movsx edx,bx                                 ; |  
004F707D   |.  FF3496             |push dword ptr ds:\[esi+edx\*4\]                ; |Arg1  
004F7080   |.  E8 F1FAFFFF        |call <IP\_TOOLS.memcpy>                       ; \\IP\_TOOLS.004F6B76  
004F7085   |.  83C4 08            |add esp,8  
004F7088   |.  6A 00              |push 0                                       ; /Arg2 = 00000000  
004F708A   |.  0FBFCB             |movsx ecx,bx                                 ; |  
004F708D   |.  FF348E             |push dword ptr ds:\[esi+ecx\*4\]                ; |Arg1  
004F7090   |.  E8 8EF9FFFF        |call <IP\_TOOLS.SHL1>                         ; \\IP\_TOOLS.004F6A23  
004F7095   |.  83C4 08            |add esp,8  
004F7098   |.  43                 |inc ebx  
004F7099   |.  66:83FB 10         |cmp bx,10  
004F709D   |.^ 7C D4              \\jl short <IP\_TOOLS.loc\_4F7073>  
004F709F   |.  5E                 pop esi  
  
\-------------------------------------------------------------------------------------------------  
004F715B   |.  83C4 08            add esp,8  
004F715E   |.  0FBF05 08845400    movsx eax,word ptr ds:\[548408\]  
004F7165   |.  03C0               add eax,eax  
004F7167   |.  03C3               add eax,ebx  
004F7169   |.  83C0 FE            add eax,-2  
004F716C   |.  8945 FC            mov dword ptr ss:\[ebp\-4\],eax  
004F716F   |.  8B75 FC            mov esi,dword ptr ss:\[ebp\-4\]  
004F7172   |.  83EE 02            sub esi,2  
004F7175   |.  6A 00              push 0                                        ; /Arg2 = 00000000  
004F7177   |.  53                 push ebx                                      ; |Arg1  
004F7178   |.  E8 1FFAFFFF        call <IP\_TOOLS.memset>                        ; \\IP\_TOOLS.004F6B9C  
004F717D   |.  83C4 08            add esp,8                                     ; 把存放结果的地方清零  
004F7180   |.  57                 push edi                                      ; /Arg1  
004F7181   |.  E8 45FAFFFF        call <IP\_TOOLS.sub\_4F6BCB>                    ; \\IP\_TOOLS.004F6BCB  
004F7186   |.  59                 pop ecx  
004F7187   |.  66:8945 FA         mov word ptr ss:\[ebp\-6\],ax  
004F718B   |.  66:837D FA 00      cmp word ptr ss:\[ebp\-6\],0  
004F7190   |.  75 07              jnz short <IP\_TOOLS.loc\_4F7199>  
004F7192   |.  33C0               xor eax,eax  
004F7194   |.  E9 2A070000        jmp <IP\_TOOLS.loc\_4F78C3>  
004F7199 <>|>  0FBF55 FA          movsx edx,word ptr ss:\[ebp\-6\]                 ;  loc\_4F7199  
004F719D   |.  03D2               add edx,edx  
004F719F   |.  03FA               add edi,edx  
004F71A1   |.  83C7 FE            add edi,-2  
004F71A4   |.  E9 06070000        jmp <IP\_TOOLS.loc\_4F78AF>  
004F71A9 <>|>  53                 /push ebx                                     ; /loc\_4F71A9  
004F71AA   |.  E8 7DFEFFFF        |call <IP\_TOOLS.sub\_4F702C>                   ; \\IP\_TOOLS.004F702C  
004F71AF   |.  59                 |pop ecx                                      ; 上面这个 call 起到了把数组元素后移的作用  
\--------------------------------------------------------------------------  
  
void F702C(WORD w\[Len\])  
{  
  WORD bx;  
  for(int i=Len;i>0;--i)  
  {  
    bx\=w\[i-1\];  
    w\[i\]=bx;  
  }  
  w\[i\]=0;  
}  
  
\-----------------------------------------------------------------------------  
004F71B0   |.  F647 01 80         |test byte ptr ds:\[edi+1\],80                  ;消息的最后一个字节和 80 进行与不为 0 就加上  
004F71B4   |.  74 11              |je short <IP\_TOOLS.loc\_4F71C7>               ;subKey4key 的最后一个 key  
004F71B6   |.  6A 00              |push 0                                       ; /Arg3 = 00000000  
004F71B8   |.  FF35 50945400      |push dword ptr ds:\[549450\]                   ; |Arg2 = 00549394  
004F71BE   |.  53                 |push ebx                                     ; |Arg1  
004F71BF   |.  E8 C5F7FFFF        |call <IP\_TOOLS.AddHash>                      ; \\IP\_TOOLS.004F6989  
\--------------------------------------------------------------------------------------------  
004F6989 <>/$  55                 push ebp                                      ; 这是 AddHash 函数  
004F698A   |.  8BEC               mov ebp,esp  
004F698C   |.  53                 push ebx  
004F698D   |.  56                 push esi  
004F698E   |.  57                 push edi  
004F698F   |.  8B45 10            mov eax,dword ptr ss:\[ebp+10\]                 ;Arg3 经过分析发现这个 Arg3 是进位标志  
004F6992   |.  8B4D 0C            mov ecx,dword ptr ss:\[ebp+C\]                  ;Arg2 加数  
004F6995   |.  8B55 08            mov edx,dword ptr ss:\[ebp+8\]                  ;Arg1 0012EA20 被 加数结果也存在这里  
004F6998   |.  66:8B35 08845400   mov si,word ptr ds:\[548408\]                   ;2A  
004F699F   |.  EB 25              jmp short <IP\_TOOLS.loc\_4F69C6>  
004F69A1 <>|>  0FB71A             /movzx ebx,word ptr ds:\[edx\]                  ;ebx\=DWORD(result\[i\])   
004F69A4   |.  0FB739             |movzx edi,word ptr ds:\[ecx\]                  ;edi\=DWORD(key16\[i\])  
004F69A7   |.  03DF               |add ebx,edi                                  ;ebx+=edi  
004F69A9   |.  25 FF000000        |and eax,0FF                                  ;eax&=0xFF,如果 eax=1，则结果为 1，否则为 0  
004F69AE   |.  03D8               |add ebx,eax                                  ;ebx+=eax  
004F69B0   |.  8BC3               |mov eax,ebx                                  ;eax\=ebx  
004F69B2   |.  83C1 02            |add ecx,2                                    ;下一个 WORD  
004F69B5   |.  66:8902            |mov word ptr ds:\[edx\],ax                     ;把结果的清零的地方=WORD(result\[i\])  
004F69B8   |.  83C2 02            |add edx,2                                    ;保存结果的地方也+2  
004F69BB   |.  A9 00000100        |test eax,10000                               ;相加的结果和 10000 相与  
004F69C0   |.  0F95C0             |setne al                                     ;如果不等于 0 的话 al=1  
004F69C3   |.  83E0 01            |and eax,1                                    ;eax\=1 或者 eax=0  
004F69C6 <>|>  8BDE               |mov ebx,esi                                  ;    
004F69C8   |.  66:83C6 FF         |add si,0FFFF  
004F69CC   |.  66:85DB            |test bx,bx  
004F69CF   |.^ 75 D0              \\jnz short <IP\_TOOLS.loc\_4F69A1>  
004F69D1   |.  5F                 pop edi  
004F69D2   |.  5E                 pop esi  
004F69D3   |.  5B                 pop ebx  
004F69D4   |.  5D                 pop ebp  
004F69D5   \\.  C3                 retn  
\-----------------------------------------------------------------------------------------------------------------  
我写的 C 代码，大数加法  
  
void AddHash(WORD result\[\],WORD subkey\[\],int carry)  
{  
  DWORD ebx,edi;  
  for(int i=0;i<Len;i++)  
  {  
    ebx\=DWORD(result\[i\]);  
    edi\=DWORD(subkey\[i\]);  
    ebx+=edi;  
    carry=(carry&0xFF);  
    ebx+=carry;  
    result\[i\]=WORD(ebx);  
    if((ebx&0x10000)!=0)  
      carry=1;  
    else   
      carry=0;  
    carry=(carry&1);  
  }  
}  
  
\---------------------------------------------------------------------------------------------------  
004F71C4   |.  83C4 0C            |add esp,0C  
004F71C7 <>|>  F647 01 40         |test byte ptr ds:\[edi+1\],40                  ;  loc\_4F71C7  
004F71CB   |.  74 11              |je short <IP\_TOOLS.loc\_4F71DE>  
004F71CD   |.  6A 00              |push 0                                       ; /Arg3 = 00000000  
004F71CF   |.  FF35 4C945400      |push dword ptr ds:\[54944C\]                   ; |Arg2 = 00549314  
004F71D5   |.  53                 |push ebx                                     ; |Arg1  
004F71D6   |.  E8 AEF7FFFF        |call <IP\_TOOLS.AddHash>                      ; \\IP\_TOOLS.004F6989  
004F71DB   |.  83C4 0C            |add esp,0C  
004F71DE <>|>  F647 01 20         |test byte ptr ds:\[edi+1\],20                  ;  loc\_4F71DE  
004F71E2   |.  74 11              |je short <IP\_TOOLS.loc\_4F71F5>  
004F71E4   |.  6A 00              |push 0                                       ; /Arg3 = 00000000  
004F71E6   |.  FF35 48945400      |push dword ptr ds:\[549448\]                   ; |Arg2 = 00549294  
004F71EC   |.  53                 |push ebx                                     ; |Arg1  
004F71ED   |.  E8 97F7FFFF        |call <IP\_TOOLS.AddHash>                      ; \\IP\_TOOLS.004F6989  
004F71F2   |.  83C4 0C            |add esp,0C  
004F71F5 <>|>  F647 01 10         |test byte ptr ds:\[edi+1\],10                  ;  loc\_4F71F5  
004F71F9   |.  74 11              |je short <IP\_TOOLS.loc\_4F720C>  
004F71FB   |.  6A 00              |push 0                                       ; /Arg3 = 00000000  
004F71FD   |.  FF35 44945400      |push dword ptr ds:\[549444\]                   ; |Arg2 = 00549214  
004F7203   |.  53                 |push ebx                                     ; |Arg1  
004F7204   |.  E8 80F7FFFF        |call <IP\_TOOLS.AddHash>                      ; \\IP\_TOOLS.004F6989  
004F7209   |.  83C4 0C            |add esp,0C  
004F720C <>|>  F647 01 08         |test byte ptr ds:\[edi+1\],8                   ;  loc\_4F720C  
004F7210   |.  74 11              |je short <IP\_TOOLS.loc\_4F7223>  
004F7212   |.  6A 00              |push 0                                       ; /Arg3 = 00000000  
004F7214   |.  FF35 40945400      |push dword ptr ds:\[549440\]                   ; |Arg2 = 00549194  
004F721A   |.  53                 |push ebx                                     ; |Arg1  
004F721B   |.  E8 69F7FFFF        |call <IP\_TOOLS.AddHash>                      ; \\IP\_TOOLS.004F6989  
004F7220   |.  83C4 0C            |add esp,0C  
004F7223 <>|>  F647 01 04         |test byte ptr ds:\[edi+1\],4                   ;  loc\_4F7223  
004F7227   |.  74 11              |je short <IP\_TOOLS.loc\_4F723A>  
004F7229   |.  6A 00              |push 0                                       ; /Arg3 = 00000000  
004F722B   |.  FF35 3C945400      |push dword ptr ds:\[54943C\]                   ; |Arg2 = 00549114  
004F7231   |.  53                 |push ebx                                     ; |Arg1  
004F7232   |.  E8 52F7FFFF        |call <IP\_TOOLS.AddHash>                      ; \\IP\_TOOLS.004F6989  
004F7237   |.  83C4 0C            |add esp,0C  
004F723A <>|>  F647 01 02         |test byte ptr ds:\[edi+1\],2                   ;  loc\_4F723A  
004F723E   |.  74 11              |je short <IP\_TOOLS.loc\_4F7251>  
004F7240   |.  6A 00              |push 0                                       ; /Arg3 = 00000000  
004F7242   |.  FF35 38945400      |push dword ptr ds:\[549438\]                   ; |Arg2 = 00549094  
004F7248   |.  53                 |push ebx                                     ; |Arg1  
004F7249   |.  E8 3BF7FFFF        |call <IP\_TOOLS.AddHash>                      ; \\IP\_TOOLS.004F6989  
004F724E   |.  83C4 0C            |add esp,0C  
004F7251 <>|>  F647 01 01         |test byte ptr ds:\[edi+1\],1                   ;  loc\_4F7251  
004F7255   |.  74 11              |je short <IP\_TOOLS.loc\_4F7268>  
004F7257   |.  6A 00              |push 0                                       ; /Arg3 = 00000000  
004F7259   |.  FF35 34945400      |push dword ptr ds:\[549434\]                   ; |Arg2 = 00549014  
004F725F   |.  53                 |push ebx                                     ; |Arg1  
004F7260   |.  E8 24F7FFFF        |call <IP\_TOOLS.AddHash>                      ; \\IP\_TOOLS.004F6989  
004F7265   |.  83C4 0C            |add esp,0C  
004F7268 <>|>  F607 80            |test byte ptr ds:\[edi\],80                    ;  loc\_4F7268  
004F726B   |.  74 11              |je short <IP\_TOOLS.loc\_4F727E>  
004F726D   |.  6A 00              |push 0                                       ; /Arg3 = 00000000  
004F726F   |.  FF35 30945400      |push dword ptr ds:\[549430\]                   ; |Arg2 = 00548F94  
004F7275   |.  53                 |push ebx                                     ; |Arg1  
004F7276   |.  E8 0EF7FFFF        |call <IP\_TOOLS.AddHash>                      ; \\IP\_TOOLS.004F6989  
004F727B   |.  83C4 0C            |add esp,0C  
004F727E <>|>  F607 40            |test byte ptr ds:\[edi\],40                    ;  loc\_4F727E  
004F7281   |.  74 11              |je short <IP\_TOOLS.loc\_4F7294>  
004F7283   |.  6A 00              |push 0                                       ; /Arg3 = 00000000  
004F7285   |.  FF35 2C945400      |push dword ptr ds:\[54942C\]                   ; |Arg2 = 00548F14  
004F728B   |.  53                 |push ebx                                     ; |Arg1  
004F728C   |.  E8 F8F6FFFF        |call <IP\_TOOLS.AddHash>                      ; \\IP\_TOOLS.004F6989  
004F7291   |.  83C4 0C            |add esp,0C  
004F7294 <>|>  F607 20            |test byte ptr ds:\[edi\],20                    ;  loc\_4F7294  
004F7297   |.  74 11              |je short <IP\_TOOLS.loc\_4F72AA>  
004F7299   |.  6A 00              |push 0                                       ; /Arg3 = 00000000  
004F729B   |.  FF35 28945400      |push dword ptr ds:\[549428\]                   ; |Arg2 = 00548E94  
004F72A1   |.  53                 |push ebx                                     ; |Arg1  
004F72A2   |.  E8 E2F6FFFF        |call <IP\_TOOLS.AddHash>                      ; \\IP\_TOOLS.004F6989  
004F72A7   |.  83C4 0C            |add esp,0C  
004F72AA <>|>  F607 10            |test byte ptr ds:\[edi\],10                    ;  loc\_4F72AA  
004F72AD   |.  74 11              |je short <IP\_TOOLS.loc\_4F72C0>  
004F72AF   |.  6A 00              |push 0                                       ; /Arg3 = 00000000  
004F72B1   |.  FF35 24945400      |push dword ptr ds:\[549424\]                   ; |Arg2 = 00548E14  
004F72B7   |.  53                 |push ebx                                     ; |Arg1  
004F72B8   |.  E8 CCF6FFFF        |call <IP\_TOOLS.AddHash>                      ; \\IP\_TOOLS.004F6989  
004F72BD   |.  83C4 0C            |add esp,0C  
004F72C0 <>|>  F607 08            |test byte ptr ds:\[edi\],8                     ;  loc\_4F72C0  
004F72C3   |.  74 11              |je short <IP\_TOOLS.loc\_4F72D6>  
004F72C5   |.  6A 00              |push 0                                       ; /Arg3 = 00000000  
004F72C7   |.  FF35 20945400      |push dword ptr ds:\[549420\]                   ; |Arg2 = 00548D94  
004F72CD   |.  53                 |push ebx                                     ; |Arg1  
004F72CE   |.  E8 B6F6FFFF        |call <IP\_TOOLS.AddHash>                      ; \\IP\_TOOLS.004F6989  
004F72D3   |.  83C4 0C            |add esp,0C  
004F72D6 <>|>  F607 04            |test byte ptr ds:\[edi\],4                     ;  loc\_4F72D6  
004F72D9   |.  74 11              |je short <IP\_TOOLS.loc\_4F72EC>  
004F72DB   |.  6A 00              |push 0                                       ; /Arg3 = 00000000  
004F72DD   |.  FF35 1C945400      |push dword ptr ds:\[54941C\]                   ; |Arg2 = 00548D14  
004F72E3   |.  53                 |push ebx                                     ; |Arg1  
004F72E4   |.  E8 A0F6FFFF        |call <IP\_TOOLS.AddHash>                      ; \\IP\_TOOLS.004F6989  
004F72E9   |.  83C4 0C            |add esp,0C  
004F72EC <>|>  F607 02            |test byte ptr ds:\[edi\],2                     ;  loc\_4F72EC  
004F72EF   |.  74 11              |je short <IP\_TOOLS.loc\_4F7302>  
004F72F1   |.  6A 00              |push 0                                       ; /Arg3 = 00000000  
004F72F3   |.  FF35 18945400      |push dword ptr ds:\[549418\]                   ; |Arg2 = 00548C94  
004F72F9   |.  53                 |push ebx                                     ; |Arg1  
004F72FA   |.  E8 8AF6FFFF        |call <IP\_TOOLS.AddHash>                      ; \\IP\_TOOLS.004F6989  
004F72FF   |.  83C4 0C            |add esp,0C  
004F7302 <>|>  F607 01            |test byte ptr ds:\[edi\],1                     ;  loc\_4F7302  
004F7305   |.  74 11              |je short <IP\_TOOLS.loc\_4F7318>  
004F7307   |.  6A 00              |push 0                                       ; /Arg3 = 00000000  
004F7309   |.  FF35 14945400      |push dword ptr ds:\[549414\]                   ; |Arg2 = 0054CD84 ASCII "Cnbragon"  
004F730F   |.  53                 |push ebx                                     ; |Arg1  
004F7310   |.  E8 74F6FFFF        |call <IP\_TOOLS.AddHash>                      ; \\IP\_TOOLS.004F6989  
004F7315   |.  83C4 0C            |add esp,0C  
004F7318 <>|>  8B4D FC            |mov ecx,dword ptr ss:\[ebp\-4\]                 ;  从这里开始是把上面 Add 的结果的最高位和  
004F731B   |.  66:8B01            |mov ax,word ptr ds:\[ecx\]                     ; SubKey4name 的最高位比较,次高位和次高位比较  
004F731E   |.  66:2B05 708C5400   |sub ax,word ptr ds:\[548C70\]                  ;如果比 SubKey4name 的大就把结果减去那个 SubKey4name  
004F7325   |.  66:85C0            |test ax,ax  
004F7328   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F735B>  
004F732A   |.  66:85C0            |test ax,ax  
004F732D   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F736C>  
004F732F   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F7332   |.  66:3B15 928C5400   |cmp dx,word ptr ds:\[548C92\]  
004F7339   |.  77 20              |ja short <IP\_TOOLS.loc\_4F735B>  
004F733B   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F733E   |.  66:3B0D 928C5400   |cmp cx,word ptr ds:\[548C92\]  
004F7345   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F736C>  
004F7347   |.  FF35 4C8C5400      |push dword ptr ds:\[548C4C\]                   ; /Arg2 = 00548B8A  
004F734D   |.  53                 |push ebx                                     ; |Arg1  
004F734E   |.  E8 54F7FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F7353   |.  83C4 08            |add esp,8  
004F7356   |.  66:85C0            |test ax,ax  
004F7359   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F736C>  
004F735B <>|>  6A 00              |push 0                                       ; /loc\_4F735B  
004F735D   |.  FF35 4C8C5400      |push dword ptr ds:\[548C4C\]                   ; |Arg2 = 00548B8A  
004F7363   |.  53                 |push ebx                                     ; |Arg1  
004F7364   |.  E8 6DF6FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
\----------------------------------------------------------------------------------------------------  
004F69D6 <>/$  55                 push ebp                                      ;  SubHash  
004F69D7   |.  8BEC               mov ebp,esp  
004F69D9   |.  53                 push ebx  
004F69DA   |.  56                 push esi  
004F69DB   |.  57                 push edi  
004F69DC   |.  8B45 10            mov eax,dword ptr ss:\[ebp+10\]  
004F69DF   |.  8B4D 0C            mov ecx,dword ptr ss:\[ebp+C\]  
004F69E2   |.  8B55 08            mov edx,dword ptr ss:\[ebp+8\]  
004F69E5   |.  66:8B35 08845400   mov si,word ptr ds:\[548408\]  
004F69EC   |.  EB 25              jmp short <IP\_TOOLS.loc\_4F6A13>  
004F69EE <>|>  0FB71A             /movzx ebx,word ptr ds:\[edx\]                    
004F69F1   |.  0FB739             |movzx edi,word ptr ds:\[ecx\]  
004F69F4   |.  2BDF               |sub ebx,edi                     ;ebx\-=edi;  
004F69F6   |.  25 FF000000        |and eax,0FF                     ;eax&=0xff  
004F69FB   |.  2BD8               |sub ebx,eax                     ;ebx\-=eax;  
004F69FD   |.  8BC3               |mov eax,ebx                     ;eax\=ebx  
004F69FF   |.  83C1 02            |add ecx,2  
004F6A02   |.  66:8902            |mov word ptr ds:\[edx\],ax        ;ax\=WORD(eax)  
004F6A05   |.  83C2 02            |add edx,2  
004F6A08   |.  A9 00000100        |test eax,10000                  ;if(eax&0x10000)  
004F6A0D   |.  0F95C0             |setne al                        ;如果不等于 0 就置 al=1  
004F6A10   |.  83E0 01            |and eax,1                       ;和 1 相与  
004F6A13 <>|>  8BDE                mov ebx,esi                                    
004F6A15   |.  66:83C6 FF         |add si,0FFFF  
004F6A19   |.  66:85DB            |test bx,bx  
004F6A1C   |.^ 75 D0              \\jnz short <IP\_TOOLS.loc\_4F69EE>  
004F6A1E   |.  5F                 pop edi  
004F6A1F   |.  5E                 pop esi  
004F6A20   |.  5B                 pop ebx  
004F6A21   |.  5D                 pop ebp  
004F6A22   \\.  C3                 retn  
\-----------------------------------------------------------------------------------------  
我写的 C 代码,672 位的大数减法  
  
  
void SubHash(WORD result\[\],WORD subkey\[\],int carry)  
{  
  DWORD ebx,edi;  
  for(int i=0;i<Len;i++)  
  {  
    ebx\=DWORD(result\[i\]);  
    edi\=DWORD(subkey\[i\]);  
    ebx\-=edi;  
    carry=(carry&0xFF);  
    ebx\-=carry;  
    result\[i\]=WORD(ebx);  
    if((ebx&0x10000)!=0)  
      carry=1;  
    else   
      carry=0;  
    carry=(carry&1);  
  }  
}  
  
\------------------------------------------------------------------------------------------  
004F7369   |.  83C4 0C            |add esp,0C  
004F736C <>|>  8B45 FC            |mov eax,dword ptr ss:\[ebp\-4\]                 ;  loc\_4F736C  
004F736F   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F7372   |.  66:2B05 6E8C5400   |sub ax,word ptr ds:\[548C6E\]  
004F7379   |.  66:85C0            |test ax,ax  
004F737C   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F73AF>  
004F737E   |.  66:85C0            |test ax,ax  
004F7381   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F73C0>  
004F7383   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F7386   |.  66:3B15 908C5400   |cmp dx,word ptr ds:\[548C90\]  
004F738D   |.  77 20              |ja short <IP\_TOOLS.loc\_4F73AF>  
004F738F   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F7392   |.  66:3B0D 908C5400   |cmp cx,word ptr ds:\[548C90\]  
004F7399   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F73C0>  
004F739B   |.  FF35 488C5400      |push dword ptr ds:\[548C48\]                   ; /Arg2 = 00548B0A  
004F73A1   |.  53                 |push ebx                                     ; |Arg1  
004F73A2   |.  E8 00F7FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F73A7   |.  83C4 08            |add esp,8  
004F73AA   |.  66:85C0            |test ax,ax  
004F73AD   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F73C0>  
004F73AF <>|>  6A 00              |push 0                                       ; /loc\_4F73AF  
004F73B1   |.  FF35 488C5400      |push dword ptr ds:\[548C48\]                   ; |Arg2 = 00548B0A  
004F73B7   |.  53                 |push ebx                                     ; |Arg1  
004F73B8   |.  E8 19F6FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
004F73BD   |.  83C4 0C            |add esp,0C  
004F73C0 <>|>  8B45 FC            |mov eax,dword ptr ss:\[ebp\-4\]                 ;  loc\_4F73C0  
004F73C3   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F73C6   |.  66:2B05 6C8C5400   |sub ax,word ptr ds:\[548C6C\]  
004F73CD   |.  66:85C0            |test ax,ax  
004F73D0   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F7403>  
004F73D2   |.  66:85C0            |test ax,ax  
004F73D5   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F7414>  
004F73D7   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F73DA   |.  66:3B15 8E8C5400   |cmp dx,word ptr ds:\[548C8E\]  
004F73E1   |.  77 20              |ja short <IP\_TOOLS.loc\_4F7403>  
004F73E3   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F73E6   |.  66:3B0D 8E8C5400   |cmp cx,word ptr ds:\[548C8E\]  
004F73ED   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F7414>  
004F73EF   |.  FF35 448C5400      |push dword ptr ds:\[548C44\]                   ; /Arg2 = 00548A8A  
004F73F5   |.  53                 |push ebx                                     ; |Arg1  
004F73F6   |.  E8 ACF6FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F73FB   |.  83C4 08            |add esp,8  
004F73FE   |.  66:85C0            |test ax,ax  
004F7401   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F7414>  
004F7403 <>|>  6A 00              |push 0                                       ; /loc\_4F7403  
004F7405   |.  FF35 448C5400      |push dword ptr ds:\[548C44\]                   ; |Arg2 = 00548A8A  
004F740B   |.  53                 |push ebx                                     ; |Arg1  
004F740C   |.  E8 C5F5FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
004F7411   |.  83C4 0C            |add esp,0C  
004F7414 <>|>  8B45 FC            |mov eax,dword ptr ss:\[ebp\-4\]                 ;  loc\_4F7414  
004F7417   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F741A   |.  66:2B05 6A8C5400   |sub ax,word ptr ds:\[548C6A\]  
004F7421   |.  66:85C0            |test ax,ax  
004F7424   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F7457>  
004F7426   |.  66:85C0            |test ax,ax  
004F7429   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F7468>  
004F742B   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F742E   |.  66:3B15 8C8C5400   |cmp dx,word ptr ds:\[548C8C\]  
004F7435   |.  77 20              |ja short <IP\_TOOLS.loc\_4F7457>  
004F7437   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F743A   |.  66:3B0D 8C8C5400   |cmp cx,word ptr ds:\[548C8C\]  
004F7441   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F7468>  
004F7443   |.  FF35 408C5400      |push dword ptr ds:\[548C40\]                   ; /Arg2 = 00548A0A  
004F7449   |.  53                 |push ebx                                     ; |Arg1  
004F744A   |.  E8 58F6FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F744F   |.  83C4 08            |add esp,8  
004F7452   |.  66:85C0            |test ax,ax  
004F7455   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F7468>  
004F7457 <>|>  6A 00              |push 0                                       ; /loc\_4F7457  
004F7459   |.  FF35 408C5400      |push dword ptr ds:\[548C40\]                   ; |Arg2 = 00548A0A  
004F745F   |.  53                 |push ebx                                     ; |Arg1  
004F7460   |.  E8 71F5FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
004F7465   |.  83C4 0C            |add esp,0C  
004F7468 <>|>  8B45 FC            |mov eax,dword ptr ss:\[ebp\-4\]                 ;  loc\_4F7468  
004F746B   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F746E   |.  66:2B05 688C5400   |sub ax,word ptr ds:\[548C68\]  
004F7475   |.  66:85C0            |test ax,ax  
004F7478   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F74AB>  
004F747A   |.  66:85C0            |test ax,ax  
004F747D   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F74BC>  
004F747F   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F7482   |.  66:3B15 8A8C5400   |cmp dx,word ptr ds:\[548C8A\]  
004F7489   |.  77 20              |ja short <IP\_TOOLS.loc\_4F74AB>  
004F748B   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F748E   |.  66:3B0D 8A8C5400   |cmp cx,word ptr ds:\[548C8A\]  
004F7495   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F74BC>  
004F7497   |.  FF35 3C8C5400      |push dword ptr ds:\[548C3C\]                   ; /Arg2 = 0054898A  
004F749D   |.  53                 |push ebx                                     ; |Arg1  
004F749E   |.  E8 04F6FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F74A3   |.  83C4 08            |add esp,8  
004F74A6   |.  66:85C0            |test ax,ax  
004F74A9   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F74BC>  
004F74AB <>|>  6A 00              |push 0                                       ; /loc\_4F74AB  
004F74AD   |.  FF35 3C8C5400      |push dword ptr ds:\[548C3C\]                   ; |Arg2 = 0054898A  
004F74B3   |.  53                 |push ebx                                     ; |Arg1  
004F74B4   |.  E8 1DF5FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
004F74B9   |.  83C4 0C            |add esp,0C  
004F74BC <>|>  8B45 FC            |mov eax,dword ptr ss:\[ebp\-4\]                 ;  loc\_4F74BC  
004F74BF   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F74C2   |.  66:2B05 668C5400   |sub ax,word ptr ds:\[548C66\]  
004F74C9   |.  66:85C0            |test ax,ax  
004F74CC   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F74FF>  
004F74CE   |.  66:85C0            |test ax,ax  
004F74D1   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F7510>  
004F74D3   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F74D6   |.  66:3B15 888C5400   |cmp dx,word ptr ds:\[548C88\]  
004F74DD   |.  77 20              |ja short <IP\_TOOLS.loc\_4F74FF>  
004F74DF   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F74E2   |.  66:3B0D 888C5400   |cmp cx,word ptr ds:\[548C88\]  
004F74E9   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F7510>  
004F74EB   |.  FF35 388C5400      |push dword ptr ds:\[548C38\]                   ; /Arg2 = 0054890A  
004F74F1   |.  53                 |push ebx                                     ; |Arg1  
004F74F2   |.  E8 B0F5FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F74F7   |.  83C4 08            |add esp,8  
004F74FA   |.  66:85C0            |test ax,ax  
004F74FD   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F7510>  
004F74FF <>|>  6A 00              |push 0                                       ; /loc\_4F74FF  
004F7501   |.  FF35 388C5400      |push dword ptr ds:\[548C38\]                   ; |Arg2 = 0054890A  
004F7507   |.  53                 |push ebx                                     ; |Arg1  
004F7508   |.  E8 C9F4FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
004F750D   |.  83C4 0C            |add esp,0C  
004F7510 <>|>  8B45 FC            |mov eax,dword ptr ss:\[ebp\-4\]                 ;  loc\_4F7510  
004F7513   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F7516   |.  66:2B05 648C5400   |sub ax,word ptr ds:\[548C64\]  
004F751D   |.  66:85C0            |test ax,ax  
004F7520   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F7553>  
004F7522   |.  66:85C0            |test ax,ax  
004F7525   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F7564>  
004F7527   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F752A   |.  66:3B15 868C5400   |cmp dx,word ptr ds:\[548C86\]  
004F7531   |.  77 20              |ja short <IP\_TOOLS.loc\_4F7553>  
004F7533   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F7536   |.  66:3B0D 868C5400   |cmp cx,word ptr ds:\[548C86\]  
004F753D   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F7564>  
004F753F   |.  FF35 348C5400      |push dword ptr ds:\[548C34\]                   ; /Arg2 = 0054888A  
004F7545   |.  53                 |push ebx                                     ; |Arg1  
004F7546   |.  E8 5CF5FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F754B   |.  83C4 08            |add esp,8  
004F754E   |.  66:85C0            |test ax,ax  
004F7551   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F7564>  
004F7553 <>|>  6A 00              |push 0                                       ; /loc\_4F7553  
004F7555   |.  FF35 348C5400      |push dword ptr ds:\[548C34\]                   ; |Arg2 = 0054888A  
004F755B   |.  53                 |push ebx                                     ; |Arg1  
004F755C   |.  E8 75F4FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
004F7561   |.  83C4 0C            |add esp,0C  
004F7564 <>|>  8B45 FC            |mov eax,dword ptr ss:\[ebp\-4\]                 ;  loc\_4F7564  
004F7567   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F756A   |.  66:2B05 628C5400   |sub ax,word ptr ds:\[548C62\]  
004F7571   |.  66:85C0            |test ax,ax  
004F7574   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F75A7>  
004F7576   |.  66:85C0            |test ax,ax  
004F7579   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F75B8>  
004F757B   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F757E   |.  66:3B15 848C5400   |cmp dx,word ptr ds:\[548C84\]  
004F7585   |.  77 20              |ja short <IP\_TOOLS.loc\_4F75A7>  
004F7587   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F758A   |.  66:3B0D 848C5400   |cmp cx,word ptr ds:\[548C84\]  
004F7591   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F75B8>  
004F7593   |.  FF35 308C5400      |push dword ptr ds:\[548C30\]                   ; /Arg2 = 0054880A  
004F7599   |.  53                 |push ebx                                     ; |Arg1  
004F759A   |.  E8 08F5FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F759F   |.  83C4 08            |add esp,8  
004F75A2   |.  66:85C0            |test ax,ax  
004F75A5   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F75B8>  
004F75A7 <>|>  6A 00              |push 0                                       ; /loc\_4F75A7  
004F75A9   |.  FF35 308C5400      |push dword ptr ds:\[548C30\]                   ; |Arg2 = 0054880A  
004F75AF   |.  53                 |push ebx                                     ; |Arg1  
004F75B0   |.  E8 21F4FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
004F75B5   |.  83C4 0C            |add esp,0C  
004F75B8 <>|>  8B45 FC            |mov eax,dword ptr ss:\[ebp\-4\]                 ;  loc\_4F75B8  
004F75BB   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F75BE   |.  66:2B05 608C5400   |sub ax,word ptr ds:\[548C60\]  
004F75C5   |.  66:85C0            |test ax,ax  
004F75C8   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F75FB>  
004F75CA   |.  66:85C0            |test ax,ax  
004F75CD   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F760C>  
004F75CF   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F75D2   |.  66:3B15 828C5400   |cmp dx,word ptr ds:\[548C82\]  
004F75D9   |.  77 20              |ja short <IP\_TOOLS.loc\_4F75FB>  
004F75DB   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F75DE   |.  66:3B0D 828C5400   |cmp cx,word ptr ds:\[548C82\]  
004F75E5   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F760C>  
004F75E7   |.  FF35 2C8C5400      |push dword ptr ds:\[548C2C\]                   ; /Arg2 = 0054878A  
004F75ED   |.  53                 |push ebx                                     ; |Arg1  
004F75EE   |.  E8 B4F4FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F75F3   |.  83C4 08            |add esp,8  
004F75F6   |.  66:85C0            |test ax,ax  
004F75F9   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F760C>  
004F75FB <>|>  6A 00              |push 0                                       ; /loc\_4F75FB  
004F75FD   |.  FF35 2C8C5400      |push dword ptr ds:\[548C2C\]                   ; |Arg2 = 0054878A  
004F7603   |.  53                 |push ebx                                     ; |Arg1  
004F7604   |.  E8 CDF3FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
004F7609   |.  83C4 0C            |add esp,0C  
004F760C <>|>  8B45 FC            |mov eax,dword ptr ss:\[ebp\-4\]                 ;  loc\_4F760C  
004F760F   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F7612   |.  66:2B05 5E8C5400   |sub ax,word ptr ds:\[548C5E\]  
004F7619   |.  66:85C0            |test ax,ax  
004F761C   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F764F>  
004F761E   |.  66:85C0            |test ax,ax  
004F7621   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F7660>  
004F7623   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F7626   |.  66:3B15 808C5400   |cmp dx,word ptr ds:\[548C80\]  
004F762D   |.  77 20              |ja short <IP\_TOOLS.loc\_4F764F>  
004F762F   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F7632   |.  66:3B0D 808C5400   |cmp cx,word ptr ds:\[548C80\]  
004F7639   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F7660>  
004F763B   |.  FF35 288C5400      |push dword ptr ds:\[548C28\]                   ; /Arg2 = 0054870A  
004F7641   |.  53                 |push ebx                                     ; |Arg1  
004F7642   |.  E8 60F4FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F7647   |.  83C4 08            |add esp,8  
004F764A   |.  66:85C0            |test ax,ax  
004F764D   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F7660>  
004F764F <>|>  6A 00              |push 0                                       ; /loc\_4F764F  
004F7651   |.  FF35 288C5400      |push dword ptr ds:\[548C28\]                   ; |Arg2 = 0054870A  
004F7657   |.  53                 |push ebx                                     ; |Arg1  
004F7658   |.  E8 79F3FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
004F765D   |.  83C4 0C            |add esp,0C  
004F7660 <>|>  8B45 FC            |mov eax,dword ptr ss:\[ebp\-4\]                 ;  loc\_4F7660  
004F7663   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F7666   |.  66:2B05 5C8C5400   |sub ax,word ptr ds:\[548C5C\]  
004F766D   |.  66:85C0            |test ax,ax  
004F7670   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F76A3>  
004F7672   |.  66:85C0            |test ax,ax  
004F7675   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F76B4>  
004F7677   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F767A   |.  66:3B15 7E8C5400   |cmp dx,word ptr ds:\[548C7E\]  
004F7681   |.  77 20              |ja short <IP\_TOOLS.loc\_4F76A3>  
004F7683   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F7686   |.  66:3B0D 7E8C5400   |cmp cx,word ptr ds:\[548C7E\]  
004F768D   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F76B4>  
004F768F   |.  FF35 248C5400      |push dword ptr ds:\[548C24\]                   ; /Arg2 = 0054868A  
004F7695   |.  53                 |push ebx                                     ; |Arg1  
004F7696   |.  E8 0CF4FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F769B   |.  83C4 08            |add esp,8  
004F769E   |.  66:85C0            |test ax,ax  
004F76A1   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F76B4>  
004F76A3 <>|>  6A 00              |push 0                                       ; /loc\_4F76A3  
004F76A5   |.  FF35 248C5400      |push dword ptr ds:\[548C24\]                   ; |Arg2 = 0054868A  
004F76AB   |.  53                 |push ebx                                     ; |Arg1  
004F76AC   |.  E8 25F3FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
004F76B1   |.  83C4 0C            |add esp,0C  
004F76B4 <>|>  8B45 FC            |mov eax,dword ptr ss:\[ebp\-4\]                 ;  loc\_4F76B4  
004F76B7   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F76BA   |.  66:2B05 5A8C5400   |sub ax,word ptr ds:\[548C5A\]  
004F76C1   |.  66:85C0            |test ax,ax  
004F76C4   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F76F7>  
004F76C6   |.  66:85C0            |test ax,ax  
004F76C9   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F7708>  
004F76CB   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F76CE   |.  66:3B15 7C8C5400   |cmp dx,word ptr ds:\[548C7C\]  
004F76D5   |.  77 20              |ja short <IP\_TOOLS.loc\_4F76F7>  
004F76D7   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F76DA   |.  66:3B0D 7C8C5400   |cmp cx,word ptr ds:\[548C7C\]  
004F76E1   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F7708>  
004F76E3   |.  FF35 208C5400      |push dword ptr ds:\[548C20\]                   ; /Arg2 = 0054860A  
004F76E9   |.  53                 |push ebx                                     ; |Arg1  
004F76EA   |.  E8 B8F3FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F76EF   |.  83C4 08            |add esp,8  
004F76F2   |.  66:85C0            |test ax,ax  
004F76F5   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F7708>  
004F76F7 <>|>  6A 00              |push 0                                       ; /loc\_4F76F7  
004F76F9   |.  FF35 208C5400      |push dword ptr ds:\[548C20\]                   ; |Arg2 = 0054860A  
004F76FF   |.  53                 |push ebx                                     ; |Arg1  
004F7700   |.  E8 D1F2FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
004F7705   |.  83C4 0C            |add esp,0C  
004F7708 <>|>  8B45 FC            |mov eax,dword ptr ss:\[ebp\-4\]                 ;  loc\_4F7708  
004F770B   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F770E   |.  66:2B05 588C5400   |sub ax,word ptr ds:\[548C58\]  
004F7715   |.  66:85C0            |test ax,ax  
004F7718   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F774B>  
004F771A   |.  66:85C0            |test ax,ax  
004F771D   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F775C>  
004F771F   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F7722   |.  66:3B15 7A8C5400   |cmp dx,word ptr ds:\[548C7A\]  
004F7729   |.  77 20              |ja short <IP\_TOOLS.loc\_4F774B>  
004F772B   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F772E   |.  66:3B0D 7A8C5400   |cmp cx,word ptr ds:\[548C7A\]  
004F7735   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F775C>  
004F7737   |.  FF35 1C8C5400      |push dword ptr ds:\[548C1C\]                   ; /Arg2 = 0054858A  
004F773D   |.  53                 |push ebx                                     ; |Arg1  
004F773E   |.  E8 64F3FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F7743   |.  83C4 08            |add esp,8  
004F7746   |.  66:85C0            |test ax,ax  
004F7749   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F775C>  
004F774B <>|>  6A 00              |push 0                                       ; /loc\_4F774B  
004F774D   |.  FF35 1C8C5400      |push dword ptr ds:\[548C1C\]                   ; |Arg2 = 0054858A  
004F7753   |.  53                 |push ebx                                     ; |Arg1  
004F7754   |.  E8 7DF2FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
004F7759   |.  83C4 0C            |add esp,0C  
004F775C <>|>  8B45 FC            |mov eax,dword ptr ss:\[ebp\-4\]                 ;  loc\_4F775C  
004F775F   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F7762   |.  66:2B05 568C5400   |sub ax,word ptr ds:\[548C56\]  
004F7769   |.  66:85C0            |test ax,ax  
004F776C   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F779F>  
004F776E   |.  66:85C0            |test ax,ax  
004F7771   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F77B0>  
004F7773   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F7776   |.  66:3B15 788C5400   |cmp dx,word ptr ds:\[548C78\]  
004F777D   |.  77 20              |ja short <IP\_TOOLS.loc\_4F779F>  
004F777F   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F7782   |.  66:3B0D 788C5400   |cmp cx,word ptr ds:\[548C78\]  
004F7789   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F77B0>  
004F778B   |.  FF35 188C5400      |push dword ptr ds:\[548C18\]                   ; /Arg2 = 0054850A  
004F7791   |.  53                 |push ebx                                     ; |Arg1  
004F7792   |.  E8 10F3FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F7797   |.  83C4 08            |add esp,8  
004F779A   |.  66:85C0            |test ax,ax  
004F779D   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F77B0>  
004F779F <>|>  6A 00              |push 0                                       ; /loc\_4F779F  
004F77A1   |.  FF35 188C5400      |push dword ptr ds:\[548C18\]                   ; |Arg2 = 0054850A  
004F77A7   |.  53                 |push ebx                                     ; |Arg1  
004F77A8   |.  E8 29F2FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
004F77AD   |.  83C4 0C            |add esp,0C  
004F77B0 <>|>  8B45 FC            |mov eax,dword ptr ss:\[ebp\-4\]                 ;  loc\_4F77B0  
004F77B3   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F77B6   |.  66:2B05 548C5400   |sub ax,word ptr ds:\[548C54\]  
004F77BD   |.  66:85C0            |test ax,ax  
004F77C0   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F77F3>  
004F77C2   |.  66:85C0            |test ax,ax  
004F77C5   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F7804>  
004F77C7   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F77CA   |.  66:3B15 768C5400   |cmp dx,word ptr ds:\[548C76\]  
004F77D1   |.  77 20              |ja short <IP\_TOOLS.loc\_4F77F3>  
004F77D3   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F77D6   |.  66:3B0D 768C5400   |cmp cx,word ptr ds:\[548C76\]  
004F77DD   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F7804>  
004F77DF   |.  FF35 148C5400      |push dword ptr ds:\[548C14\]                   ; /Arg2 = 0054848A  
004F77E5   |.  53                 |push ebx                                     ; |Arg1  
004F77E6   |.  E8 BCF2FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F77EB   |.  83C4 08            |add esp,8  
004F77EE   |.  66:85C0            |test ax,ax  
004F77F1   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F7804>  
004F77F3 <>|>  6A 00              |push 0                                       ; /loc\_4F77F3  
004F77F5   |.  FF35 148C5400      |push dword ptr ds:\[548C14\]                   ; |Arg2 = 0054848A  
004F77FB   |.  53                 |push ebx                                     ; |Arg1  
004F77FC   |.  E8 D5F1FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
004F7801   |.  83C4 0C            |add esp,0C  
004F7804 <>|>  8B45 FC            |mov eax,dword ptr ss:\[ebp\-4\]                 ;  loc\_4F7804  
004F7807   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F780A   |.  66:2B05 528C5400   |sub ax,word ptr ds:\[548C52\]  
004F7811   |.  66:85C0            |test ax,ax  
004F7814   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F7847>  
004F7816   |.  66:85C0            |test ax,ax  
004F7819   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F7858>  
004F781B   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F781E   |.  66:3B15 748C5400   |cmp dx,word ptr ds:\[548C74\]  
004F7825   |.  77 20              |ja short <IP\_TOOLS.loc\_4F7847>  
004F7827   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F782A   |.  66:3B0D 748C5400   |cmp cx,word ptr ds:\[548C74\]  
004F7831   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F7858>  
004F7833   |.  FF35 108C5400      |push dword ptr ds:\[548C10\]                   ; /Arg2 = 0054840A  
004F7839   |.  53                 |push ebx                                     ; |Arg1  
004F783A   |.  E8 68F2FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F783F   |.  83C4 08            |add esp,8  
004F7842   |.  66:85C0            |test ax,ax  
004F7845   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F7858>  
004F7847 <>|>  6A 00              |push 0                                       ; /loc\_4F7847  
004F7849   |.  FF35 108C5400      |push dword ptr ds:\[548C10\]                   ; |Arg2 = 0054840A  
004F784F   |.  53                 |push ebx                                     ; |Arg1  
004F7850   |.  E8 81F1FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
004F7855   |.  83C4 0C            |add esp,0C  
004F7858 <>|>  8B45 FC            |mov eax,dword ptr ss:\[ebp\-4\]                 ;  loc\_4F7858  
004F785B   |.  66:8B00            |mov ax,word ptr ds:\[eax\]  
004F785E   |.  66:2B05 508C5400   |sub ax,word ptr ds:\[548C50\]  
004F7865   |.  66:85C0            |test ax,ax  
004F7868   |.  7F 31              |jg short <IP\_TOOLS.loc\_4F789B>  
004F786A   |.  66:85C0            |test ax,ax  
004F786D   |.  75 3D              |jnz short <IP\_TOOLS.loc\_4F78AC>  
004F786F   |.  66:8B16            |mov dx,word ptr ds:\[esi\]  
004F7872   |.  66:3B15 728C5400   |cmp dx,word ptr ds:\[548C72\]  
004F7879   |.  77 20              |ja short <IP\_TOOLS.loc\_4F789B>  
004F787B   |.  66:8B0E            |mov cx,word ptr ds:\[esi\]  
004F787E   |.  66:3B0D 728C5400   |cmp cx,word ptr ds:\[548C72\]  
004F7885   |.  75 25              |jnz short <IP\_TOOLS.loc\_4F78AC>  
004F7887   |.  FF35 0C8C5400      |push dword ptr ds:\[548C0C\]                   ; /Arg2 = 00548102  
004F788D   |.  53                 |push ebx                                     ; |Arg1  
004F788E   |.  E8 14F2FFFF        |call <IP\_TOOLS.sub\_4F6AA7>                   ; \\IP\_TOOLS.004F6AA7  
004F7893   |.  83C4 08            |add esp,8  
004F7896   |.  66:85C0            |test ax,ax  
004F7899   |.  7C 11              |jl short <IP\_TOOLS.loc\_4F78AC>  
004F789B <>|>  6A 00              |push 0                                       ; /loc\_4F789B  
004F789D   |.  FF35 0C8C5400      |push dword ptr ds:\[548C0C\]                   ; |Arg2 = 00548102  
004F78A3   |.  53                 |push ebx                                     ; |Arg1  
004F78A4   |.  E8 2DF1FFFF        |call <IP\_TOOLS.sub\_4F69D6>                   ; \\IP\_TOOLS.004F69D6  
004F78A9   |.  83C4 0C            |add esp,0C  
004F78AC <>|>  83EF 02            |sub edi,2                                    ;  loc\_4F78AC  
004F78AF <>|>  66:8B45 FA          mov ax,word ptr ss:\[ebp\-6\]                   ;  这里是用户名的字长  
004F78B3   |.  66:8345 FA FF      |add word ptr ss:\[ebp\-6\],0FFFF  
004F78B8   |.  66:85C0            |test ax,ax  
004F78BB   |.^ 0F85 E8F8FFFF      \\jnz <IP\_TOOLS.loc\_4F71A9>  
004F78C1   |.  33C0               xor eax,eax  
}  
  
\-------------------------------------------------------------------------------------------------------  
下面是我用 C 写的核心函数,称之为θ函数 Y=θ(message)  
void theta1(WORD message\[Len\])  
{  
  WORD arg\[8\]={0x80,0x40,0x20,0x10,0x8,0x4,0x2,0x1};  
  WORD w;  
  WORD wltemp;  
  WORD whtemp;  
  MakeTempKey(message,subKey4Key,16);//根据消息生成子密钥这个是变化的  
  for(int i=0;i<Len;i++)  
  {  
    subKey4Key\[0\]\[i\]=message\[i\];  
  }  
  memset(Result,0);  
  for(int l=0;l<Len;l++)  
  {  
    if(message\[l+1\]==0x0000)  
      break;  
  }  
  for(int t=l;t>=0;t--)  
  {  
    F702C(Result);  
    w=message\[t\];  
    whtemp=(w & 0xFF);  
    wltemp=(w >>8);  
    for(int i=0;i<8;i++)  
    {  
      function1(wltemp,arg\[i\],(15-i));  //函数 f1  
    }  
    for(int j=0;j<8;j++)  
    {  
      function1(whtemp,arg\[j\],(7-j));    
    }  
    for(int s=16;s>=0;s--)  
    {  
      function2(s);                     //函数 f2  
    }  
  }  
}  
  
void function1(WORD arg1, WORD arg2, WORD arg3)  
{  
  if(arg1 & arg2)  
  {  
    AddHash(Result,subKey4Key\[arg3\],0);  
  }  
}  
  
void function2(int n)  
{  
  WORD ax;  
  WORD dx;  
  int cx;  
  ax\=Result\[Len-1\];  
  ax\-=wHigh\[n\];      //这个是最高位  
  if(ax\>0)  
  {  
    SubHash(Result,subKey4name\[n\],0);  
  }  
  else if(ax\==0)  
  {  
    dx\=Result\[Len-2\];  
    if(dx\>wLow\[n\])        //次高位  
    {  
      SubHash(Result,subKey4name\[n\],0);  
    }  
    else if(dx\==wLow\[n\])  
    {  
      cx\=F6AA7(subKey4name\[n\],Result);  
      if(cx\>=0)  
      {  
        SubHash(Result,subKey4name\[n\],0);  
      }  
    }  
  }  
}  
  
\---------------------------------------------------------  
  
比如：第一次以我的名子作为消息 message,然后做θ运算,得到一个 Hash,  
然后把这个 Hash 作为下一次θ运算的 message,也就是迭代运算,我的 C 代码如下:  
  
        MakeKey4PK(wkey4name,subKey4name,17);   //由公钥生成子密钥  
  for(int i=0;i<Len;i++)  
  {  
    subKey4name\[0\]\[i\]=wkey4name\[i\];  
  }  
  memcpy(name,tempname,Len);  
  WORD arg\[2\]={0x0003,0x8000};     //固定参数 1  
  int m=1;  
  WORD v;  
  v=arg\[m\];  
  WORD bx\=0x4000;  
  for(int i=0;i<0x1f;i++)  
  {  
    theta1(tempname);  
    memcpy(Result,tempname,Len);  
    if((bx & v)==0)  
    {  
      bx\=(bx\>>1);  
      if(bx\==0x0000)  
      {  
        bx\=0x8000;  
        --m;  
        v=arg\[m\];  
      }  
    }  
    else  
    {  
      theta2(tempname,name);  
      memcpy(Result,tempname,Len);  
    }  
  }  
  
\----------------------------------------------------------------------------------  
  
上面是对用户名的处理，总的来说就是用公钥和固定参数 1 对用户名进行加密 0x1F 轮,得到一个 640 位的大数,  
以 Cnbragon 为例最终生成  
  
0054C900  17 DC 2C 08 4D CE 56 49 E8 FC 21 DE 66 33 32 FF  ?M 蜼 I 椟!辠 32�  
0054C910  2D C9 06 B2 74 CC C7 F9 3F 6A 03 4E BD 70 6D 56  -?瞭糖?jN 絧 mV  
0054C920  3E 44 2A E5 68 26 90 E1 3C 59 78 E1 CD 4C 00 DA  >D\*録&愥<Yx 嵬 L.  
0054C930  48 67 4D 34 A5 F2 03 B1 6F 3B FD 9F AB 9B 50 7A  HgM4ヲ眔;裏珱 Pz  
0054C940  50 82 F9 00 17 9D F2 AC 45 DF 7E FD B7 0E 6C A9  P 債.濖珽邁l  
  
我称之为 Cipher1,即 Cipher1=f(Encryptkey,Arg1,0x1f,name)  
\---------------------------------------------------------------------------------  
  
那么，程序是如何对注册码进行处理的呢？前面说过这是一个非对称算法，其加密和解密算法是一样的，只是参数不同  
用私钥和固定参数 2 对注册码进行 0x27C 次处理，最终也得到一个 640 位的大数,  
我称之为 Cipher2,即 Cipher2=f(Decryptkey,Arg2,0x27C,sn)  
  
  
如果 HashToInt(Cipher1)=HashToInt(Cipher2)那么注册就成功!实际上也就是 Cipher1=Cipher2  
  
但是函数 f 是看起来是一个单向散列函数,把明文和密文混和在一起,并且依赖于密钥,因此其扩散和混乱效果很好,对其进行密码分析  
有一定的难度,不过从对注册码的处理看来可以利用这个单向函数的碰撞来进行生成注册码，而且我们有解密密钥  
  
  
因为我并不清楚这个算法的名称(可能是作者自己设计的),所以我无法一下子就知道是如何生成注册码，所以我对以上的几组参数进行了  
分析:   
   EncryptKey 和 DecryptKey 可以作为一组  
   从分析来看固定参数 1 和 0x1F 一定是不可分开的  
   同样固定参数 2 和 0x27C 也是不可分开的,理由兄弟们可以自己分析一下,很有意思  
  
所以在解密的时候我做了几组尝试,可惜都没有成功,幸运的是我静下心来仔细思考这其中的关系,根据经验我得出了这样一个函数  
 f(DecryptKey,Arg1,0x1F,(f(EncryptKey,Arg1,0x1F,name))),也就是说先对用户名进行加密得到一个 640 位的大数,然后用解密密钥  
和固定参数 1,0x1F 对这个 640 位的大数进行解密，最终我成功的得到了注册码!!  
  
sn=f(DecryptKey,Arg1,0x1F,(f(EncryptKey,Arg1,0x1F,name)))  
  
                       Enemy down!  
  
后记:不难看出保密密钥的重要性，对于未知密码学算法,只要逆向出了算法，有了密钥我们就可以轻而易举的实现解密.在分析这个算法的  
过程中偶始终感到对未知算法的分析充满乐趣,愿把我的乐趣和兄弟们一起分享  
    让我感到不爽的是我在 10 月 28 号的 0day 中发现了 IP-Tools 的 TSZ 的注册机:(  
  
我把我写的注册机的部分源码打了包，我没有放编译好的注册机,在我的源码里只差用户名的输入部分我没有写，很容易的,兄弟们自己写吧  
  
附件:keygen.rar 

 4dd466d0e1e0cc123afbe504da2462ca