

# PasteItInV1.2 注册算法分析--AES128+Base128

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-72.htm`](https://www.pediy.com/kssd/pediy07/pediy7-72.htm)

PasteItIn 注册算法分析--AES128+Base128  
  
URL:http://www.wavget.com/  
  
DESCRiPTiON:  
PasteItIn continually monitors the Windows Clipboard and                
automatically stores any text information you copy from any             
application. As you copy items, PasteItIn fills buttons with the        
information. You can then paste the items by simply pressing on         
each button as you need it. You can create multiple groups of           
buttons that can automatically pop up when you open a specific         
application or window.     
  
 这个软件是 CORE(Challenge of Reverse Engineering)在 01/08/2005 的 0day 中发布的  
其貌不扬,看了一下其保护算法,竟然是 AES(也即 Rijndael),当即决定拿下它来看看 AES 在软件  
保护中的应用,先说明一下,本文 too long ,需要稍为了解一下 AES,如果您不知道它,那么你可以  
先看 fips-197(Federal Information Processing Standards Publication 197) 即联邦信息处理标准 197 号文档,  
是关于 AES 的权威说明,但是首先要明白的是 128 位的 AES 共需要生成 10 个子密钥,10 轮加密,分组都为 128 位  
  
PeID 查为 Delphi 6-7,kanal 插件分析为 Rijndael 和 Base64,其实是 base128.用 DarKDe 分析为 Delphi 6,导出.map 文件,然后把 map 文件导入 Olly 中     
  
经分析,软件保护验证的地方在这里                   
  
0048D768 <>/.  55                      push ebp                                      ;  <-TTrayForm@MenuRegisterPasteItInClick  
0048D769   |.  8BEC                    mov ebp,esp  
0048D76B   |.  B9 05000000             mov ecx,5  
0048D770   |>  6A 00                   /push 0  
0048D772   |.  6A 00                   |push 0  
0048D774   |.  49                      |dec ecx  
0048D775   |.^ 75 F9                   \\jnz short PasteItI.0048D770  
0048D777   |.  53                      push ebx  
0048D778   |.  56                      push esi  
0048D779   |.  8BD8                    mov ebx,eax  
0048D77B   |.  33C0                    xor eax,eax  
0048D77D   |.  55                      push ebp  
0048D77E   |.  68 4ED94800             push <PasteItI.->System.@HandleFinally;>  
0048D783   |.  64:FF30                 push dword ptr fs:\[eax\]  
0048D786   |.  64:8920                 mov dword ptr fs:\[eax\],esp  
0048D789   |.  8D45 F4                 lea eax,dword ptr ss:\[ebp-C\]  
0048D78C   |.  BA 64D94800             mov edx,PasteItI.0048D964                     ;  ASCII "PasteItIn"  
0048D791 <>|.  E8 2273F7FF             call PasteItI.00404AB8                        ;  ->System.@LStrLAsg(void;void;void;void);  
0048D796   |.  A1 F4EC4900             mov eax,dword ptr ds:\[49ECF4\]  
0048D79B   |.  8038 00                 cmp byte ptr ds:\[eax\],0  
0048D79E   |.  0F85 62010000           jnz PasteItI.0048D906  
0048D7A4   |.  6A 00                   push 0  
0048D7A6   |.  68 78D94800             push PasteItI.0048D978                        ;  ASCII "If you have received your "  
0048D7AB   |.  FF75 F4                 push dword ptr ss:\[ebp-C\]  
0048D7AE   |.  68 9CD94800             push PasteItI.0048D99C  
0048D7B3   |.  68 A8D94800             push PasteItI.0048D9A8                        ;  ASCII "registration information, click Yes to continue."  
0048D7B8   |.  8D45 F0                 lea eax,dword ptr ss:\[ebp-10\]  
0048D7BB   |.  BA 04000000             mov edx,4  
0048D7C0 <>|.  E8 D375F7FF             call PasteItI.00404D98                        ;  ->System.@LStrCatN;  
0048D7C5   |.  8B45 F0                 mov eax,dword ptr ss:\[ebp-10\]                 ; |  
0048D7C8   |.  66:8B0D DCD94800        mov cx,word ptr ds:\[48D9DC\]                   ; |  
0048D7CF   |.  B2 03                   mov dl,3                                      ; |  
0048D7D1   |.  E8 325AFBFF             call PasteItI.00443208                        ; \\PasteItI.00443208  
0048D7D6   |.  83F8 06                 cmp eax,6  
0048D7D9   |.  0F85 54010000           jnz PasteItI.0048D933  
0048D7DF   |.  8D45 FC                 lea eax,dword ptr ss:\[ebp-4\]  
0048D7E2   |.  50                      push eax                                      ; /Arg1  
0048D7E3   |.  33C9                    xor ecx,ecx                                   ; |  
0048D7E5   |.  BA E8D94800             mov edx,PasteItI.0048D9E8                     ; |ASCII "Please enter your name:"  
0048D7EA   |.  B8 08DA4800             mov eax,PasteItI.0048DA08                     ; |ASCII "Registration"  
0048D7EF <>|.  E8 045EFBFF             call PasteItI.004435F8                        ; \\->:TMessageForm.\_PROC\_004435F8()  
0048D7F4   |.  8D45 F8                 lea eax,dword ptr ss:\[ebp-8\]  
0048D7F7   |.  50                      push eax                                      ; /Arg1  
0048D7F8   |.  33C9                    xor ecx,ecx                                   ; |  
0048D7FA   |.  BA 20DA4800             mov edx,PasteItI.0048DA20                     ; |ASCII "Please enter your registration code"  
0048D7FF   |.  B8 08DA4800             mov eax,PasteItI.0048DA08                     ; |ASCII "Registration"  
0048D804 <>|.  E8 EF5DFBFF             call PasteItI.004435F8                        ; \\->:TMessageForm.\_PROC\_004435F8()  
0048D809   |.  8D45 EC                 lea eax,dword ptr ss:\[ebp-14\]  
0048D80C   |.  50                      push eax  
0048D80D   |.  8D4D E8                 lea ecx,dword ptr ss:\[ebp-18\]  
0048D810   |.  8B55 FC                 mov edx,dword ptr ss:\[ebp-4\]                  ;name here is "cnbragon"  
0048D813   |.  8BC3                    mov eax,ebx  
0048D815 <>|.  E8 4AFCFFFF             call <PasteItI.<-TTrayForm@Encrypt>           ;  ->:TTrayForm.Encrypt()  
0048D81A   |.  8B55 E8                 mov edx,dword ptr ss:\[ebp-18\]                 ;很容易看出来上面就是加密的地方  
0048D81D   |.  8B4D F8                 mov ecx,dword ptr ss:\[ebp-8\]  
0048D820   |.  8BC3                    mov eax,ebx  
0048D822 <>|.  E8 BDFDFFFF             call <PasteItI.<-TTrayForm@GetCode>           ;  ->:TTrayForm.GetCode()  
0048D827   |.  8B45 EC                 mov eax,dword ptr ss:\[ebp-14\]  
0048D82A   |.  8B55 F8                 mov edx,dword ptr ss:\[ebp-8\]  
0048D82D <>|.  E8 EA75F7FF             call PasteItI.00404E1C                        ;  ->System.@LStrCmp;  
0048D832   |.  0F85 90000000           jnz PasteItI.0048D8C8  
0048D838   |.  8D55 E0                 lea edx,dword ptr ss:\[ebp-20\]  
\===========================================================================================  
0048D464 <>/$  55                      push ebp                                      ;  <-TTrayForm@Encrypt  
0048D465   |.  8BEC                    mov ebp,esp  
0048D467   |.  83C4 F8                 add esp,-8  
0048D46A   |.  53                      push ebx  
0048D46B   |.  56                      push esi  
0048D46C   |.  33DB                    xor ebx,ebx  
0048D46E   |.  895D F8                 mov dword ptr ss:\[ebp-8\],ebx  
0048D471   |.  8BF1                    mov esi,ecx  
0048D473   |.  8955 FC                 mov dword ptr ss:\[ebp-4\],edx  
0048D476   |.  8BD8                    mov ebx,eax  
0048D478   |.  8B45 FC                 mov eax,dword ptr ss:\[ebp-4\]  
0048D47B <>|.  E8 407AF7FF             call PasteItI.00404EC0                        ;  ->System.@LStrAddRef(void;void):Pointer;  
0048D480   |.  33C0                    xor eax,eax  
0048D482   |.  55                      push ebp  
0048D483   |.  68 EFD44800             push <PasteItI.->System.@HandleFinally;>  
0048D488   |.  64:FF30                 push dword ptr fs:\[eax\]  
0048D48B   |.  64:8920                 mov dword ptr fs:\[eax\],esp  
0048D48E   |.  BA 04D54800             mov edx,PasteItI.0048D504                     ;  ASCII "Init Vector"  
0048D493 <>|.  8B83 20030000           mov eax,dword ptr ds:\[ebx+320\]                ;  \*Rijndael:TRijndael  
0048D499   |.  E8 5A7CFEFF             call PasteItI.004750F8   
\-----------------------------------------------------------------------------------------------------------  
初始化 AES 中的初始向量为"Init Vector",这个值影响 AES 的 state 初始值,所以也影响最后的加密结果  
00475122   |.  B8 01000000             mov eax,1  
00475127   |>  8BC8                    /mov ecx,eax  
00475129   |.  49                      |dec ecx  
0047512A   |.  83E1 0F                 |and ecx,0F  
0047512D   |.  8D4C0B 36               |lea ecx,dword ptr ds:\[ebx+ecx+36\]  
00475131   |.  51                      |push ecx  
00475132   |.  8A4C06 FF               |mov cl,byte ptr ds:\[esi+eax-1\]     ;这儿就是"Init Vector"  
00475136   |.  5F                      |pop edi  
00475137   |.  300F                    |xor byte ptr ds:\[edi\],cl  
00475139   |.  40                      |inc eax  
0047513A   |.  4A                      |dec edx  
0047513B   |.^ 75 EA                   \\jnz short PasteItI.00475127  
0047513D   |>  C643 35 01              mov byte ptr ds:\[ebx+35\],1          ;初始化成功标志  
  
\-----------------------------------------------------------------------------------------------------------                       ;    
0048D49E   |.  BA 18D54800             mov edx,PasteItI.0048D518                     ;  ASCII "PasteItIn" ,这个是 AES 的密钥  
0048D4A3 <>|.  8B83 20030000           mov eax,dword ptr ds:\[ebx+320\]                ;  \*Rijndael:TRijndael 9 个字节  
0048D4A9 <>|.  E8 367BFEFF             call PasteItI.00474FE4                        ;  ->:TRijndael.\_PROC\_00474FE4()  
\-----------------------------------------------------------------------------------------------------------  
00474FE4   /$  53                      push ebx                                      ;这儿是 AES\_Encrypt()  
00474FE5   |.  56                      push esi  
00474FE6   |.  57                      push edi  
00474FE7   |.  55                      push ebp  
00474FE8   |.  83C4 E0                 add esp,-20  
00474FEB   |.  8BFA                    mov edi,edx  
00474FED   |.  8BF0                    mov esi,eax  
00474FEF   |.  8BC6                    mov eax,esi  
00474FF1   |.  E8 1AF9FFFF             call PasteItI.00474910  
00474FF6   |.  8BC7                    mov eax,edi  
00474FF8   |.  E8 DBFCF8FF             call PasteItI.00404CD8  
00474FFD   |.  48                      dec eax  
00474FFE   |.  7C 0E                   jl short PasteItI.0047500E  
00475000   |.  8BC7                    mov eax,edi  
00475002   |.  E8 D1FCF8FF             call PasteItI.00404CD8  
00475007   |.  3D FF000000             cmp eax,0FF                                   ;密钥长度  
0047500C   |.  7E 1B                   jle short PasteItI.00475029  
0047500E   |>  B9 C8504700             mov ecx,PasteItI.004750C8                     ;  ASCII "Rijndael: Key must be between 1 and 255 bytes"  
00475013   |.  B2 01                   mov dl,1                                      ;这儿说的很清楚了  
00475015   |.  A1 38464700             mov eax,dword ptr ds:\[474638\]                   
....................................  
004750A1   |>  55                      push ebp                                   ;上面都是做一些初始化的工作,没什么好看的  
004750A2   |.  8D5424 04               lea edx,dword ptr ss:\[esp+4\]  
004750A6   |.  B9 1F000000             mov ecx,1F  
004750AB   |.  8BC6                    mov eax,esi   
004750AD   |.  E8 02070000             call PasteItI.004757B4                     ;这个 call 最有意思 :)  
\-----------------------------------------------------------------------------------------------------------  
在往下看之前,先看看 AES 的 key schedule 的 C 伪代码,摘自 fips-197  
KeyExpansion(byte key\[4\*Nk\], word w\[Nb\*(Nr+1)\], Nk)  
begin  
word temp  
i = 0  
while (i < Nk)  
w\[i\] = word(key\[4\*i\], key\[4\*i+1\], key\[4\*i+2\], key\[4\*i+3\])  
i = i+1  
end while  
i = Nk  
while (i < Nb \* (Nr+1)\]  
temp = w\[i-1\]  
if (i mod Nk = 0)  
temp = SubWord(RotWord(temp)) xor Rcon\[i/Nk\]  
else if (Nk > 6 and i mod Nk = 4)  
temp = SubWord(temp)  
end if  
w\[i\] = w\[i-Nk\] xor temp  
i = i + 1  
end while  
end  
虽然下面的这一段没有严格照着上面的来写,但大体上的代码功能不难识别出来  
具体的说明请参考 fips-197  
\-------------------------------------------------------------------------------------------------------------  
....................  
00475800    .  EB 0A                   jmp short PasteItI.0047580C  
00475802    >  B9 08000000             mov ecx,8  
00475807    .  BE 0E000000             mov esi,0E  
0047580C    >  8BC6                    mov eax,esi  
0047580E    .  8B55 FC                 mov edx,dword ptr ss:\[ebp-4\]  
00475811    .  8B52 30                 mov edx,dword ptr ds:\[edx+30\]  
00475814    .  8842 04                 mov byte ptr ds:\[edx+4\],al  
00475817    .  33C0                    xor eax,eax  
00475819    .  8945 F8                 mov dword ptr ss:\[ebp-8\],eax  
0047581C    .  33D2                    xor edx,edx  
0047581E    .  33C0                    xor eax,eax  
00475820    .  EB 30                   jmp short PasteItI.00475852  
00475822    .  EB 1B                   jmp short PasteItI.0047583F  
00475824    >  8B5D F8                 mov ebx,dword ptr ss:\[ebp-8\]  
00475827    .  C1E3 02                 shl ebx,2  
0047582A    .  8B7D FC                 mov edi,dword ptr ss:\[ebp-4\]  
0047582D    .  8B7F 30                 mov edi,dword ptr ds:\[edi+30\]  
00475830    .  8D1CDF                  lea ebx,dword ptr ds:\[edi+ebx\*8\]  
00475833    .  8D7C85 D0               lea edi,dword ptr ss:\[ebp+eax\*4-30\]  
00475837    .  8B3F                    mov edi,dword ptr ds:\[edi\]  
00475839    .  897C93 05               mov dword ptr ds:\[ebx+edx\*4+5\],edi       ; word()  
0047583D    .  40                      inc eax  
0047583E    .  42                      inc edx  
0047583F    >  3BC8                    cmp ecx,eax  
00475841    .  7E 05                   jle short PasteItI.00475848  
00475843    .  83FA 04                 cmp edx,4  
00475846    .^ 7C DC                   jl short PasteItI.00475824  
00475848    >  83FA 04                 cmp edx,4  
0047584B    .  75 05                   jnz short PasteItI.00475852  
0047584D    .  33D2                    xor edx,edx  
0047584F    .  FF45 F8                 inc dword ptr ss:\[ebp-8\]  
00475852    >  3BC8                    cmp ecx,eax  
00475854    .  7E 08                   jle short PasteItI.0047585E  
00475856    .  8D5E 01                 lea ebx,dword ptr ds:\[esi+1\]  
00475859    .  3B5D F8                 cmp ebx,dword ptr ss:\[ebp-8\]  
0047585C    .^ 7F E1                   jg short PasteItI.0047583F  
0047585E    >  33C0                    xor eax,eax  
00475860    .  8945 F4                 mov dword ptr ss:\[ebp-C\],eax  
00475863    .  E9 47010000             jmp PasteItI.004759AF  
00475868    >  33C0                    xor eax,eax  
0047586A    .  8A448D CD               mov al,byte ptr ss:\[ebp+ecx\*4-33\]  
0047586E    .  8A80 38C54900           mov al,byte ptr ds:\[eax+49C538\]     ;AES 的 s-box  
\-----------------------------------------------------------------------------------------------  
0049C538  63 7C 77 7B F2 6B 6F C5 30 01 67 2B FE D7 AB 76  c|w{騥 o?g+玽  
0049C548  CA 82 C9 7D FA 59 47 F0 AD D4 A2 AF 9C A4 72 C0  蕚蓕鶼 G 瓠寓瘻  
0049C558  B7 FD 93 26 36 3F F7 CC 34 A5 E5 F1 71 D8 31 15  俘?6?魈 4ュ駋?  
0049C568  04 C7 23 C3 18 96 05 9A 07 12 80 E2 EB 27 B2 75  ????€怆'瞮  
0049C578  09 83 2C 1A 1B 6E 5A A0 52 3B D6 B3 29 E3 2F 84  .?nZ\_R;殖)?  
0049C588  53 D1 00 ED 20 FC B1 5B 6A CB BE 39 4A 4C 58 CF  S??\[j 司 9JLX  
0049C598  D0 EF AA FB 43 4D 33 85 45 F9 02 7F 50 3C 9F A8  酗CM3 匛?P<煥  
0049C5A8  51 A3 40 8F 92 9D 38 F5 BC B6 DA 21 10 FF F3 D2  Q彃?跫囤!�笠  
0049C5B8  CD 0C 13 EC 5F 97 44 17 C4 A7 7E 3D 64 5D 19 73  ?靇桪魔~=d\]s  
0049C5C8  60 81 4F DC 22 2A 90 88 46 EE B8 14 DE 5E 0B DB  \`丱?\*悎 F 罡轣  
0049C5D8  E0 32 3A 0A 49 06 24 5C C2 D3 AC 62 91 95 E4 79  ?:.I$\\掠琤憰鋣  
0049C5E8  E7 C8 37 6D 8D D5 4E A9 6C 56 F4 EA 65 7A AE 08  缛 7m 嵳 N﹍V 絷 ez?  
0049C5F8  BA 78 25 2E 1C A6 B4 C6 E8 DD 74 1F 4B BD 8B 8A  簒%.Υ畦輙 K 綃  
0049C608  70 3E B5 66 48 03 F6 0E 61 35 57 B9 86 C1 1D 9E  p>礷 H?a5W 箚?  
0049C618  E1 F8 98 11 69 D9 8E 94 9B 1E 87 E9 CE 55 28 DF  狲?i 賻敍囬蜺(  
0049C628  8C A1 89 0D BF E6 42 68 41 99 2D 0F B0 54 BB 16  尅?挎 BhA?癟?  
  
\------------------------------------------------------------------------------------------------  
00475874    .  3045 D0                 xor byte ptr ss:\[ebp-30\],al        ;subword()  
00475877    .  33C0                    xor eax,eax  
00475879    .  8A448D CE               mov al,byte ptr ss:\[ebp+ecx\*4-32\]  
0047587D    .  8A80 38C54900           mov al,byte ptr ds:\[eax+49C538\]  
00475883    .  3045 D1                 xor byte ptr ss:\[ebp-2F\],al  
00475886    .  33C0                    xor eax,eax  
00475888    .  8A448D CF               mov al,byte ptr ss:\[ebp+ecx\*4-31\]  
0047588C    .  8A80 38C54900           mov al,byte ptr ds:\[eax+49C538\]  
00475892    .  3045 D2                 xor byte ptr ss:\[ebp-2E\],al  
00475895    .  33C0                    xor eax,eax  
00475897    .  8A448D CC               mov al,byte ptr ss:\[ebp+ecx\*4-34\]  
0047589B    .  8A80 38C54900           mov al,byte ptr ds:\[eax+49C538\]  
004758A1    .  3045 D3                 xor byte ptr ss:\[ebp-2D\],al  
004758A4    .  8B45 F4                 mov eax,dword ptr ss:\[ebp-C\]  
004758A7    .  8A0485 C0C44900         mov al,byte ptr ds:\[eax\*4+49C4C0\] ;这儿就是 Rcon\[i\]  
\------------------------------------------------------------------------------------------------  
在 miracl 中定义如下  
static const MR\_WORD rco\[\]=  
{1,2,4,8,16,32,64,128,27,54,108,216,171,77,154,47};  
在此内存中为  
0049C4C0  01 00 00 00 02 00 00 00 04 00 00 00 08 00 00 00  ............  
0049C4D0  10 00 00 00 20 00 00 00 40 00 00 00 80 00 00 00  ... ...@...€...  
0049C4E0  1B 00 00 00 36 00 00 00 6C 00 00 00 D8 00 00 00  ...6...l...?..  
0049C4F0  AB 00 00 00 4D 00 00 00 9A 00 00 00 2F 00 00 00  ?..M...?../...  
\------------------------------------------------------------------------------------------------  
004758AE    .  3045 D0                 xor byte ptr ss:\[ebp-30\],al  
004758B1    .  FF45 F4                 inc dword ptr ss:\[ebp-C\]  
004758B4    .  83F9 08                 cmp ecx,8  
004758B7    .  74 27                   je short PasteItI.004758E0  
004758B9    .  8BC1                    mov eax,ecx  
004758BB    .  48                      dec eax  
004758BC    .  85C0                    test eax,eax  
004758BE    .  0F8E AB000000           jle PasteItI.0047596F  
004758C4    .  8945 F0                 mov dword ptr ss:\[ebp-10\],eax  
004758C7    .  8D45 D0                 lea eax,dword ptr ss:\[ebp-30\]  
004758CA    >  8BD8                    mov ebx,eax  
004758CC    .  8B1B                    mov ebx,dword ptr ds:\[ebx\]  
004758CE    .  8D78 04                 lea edi,dword ptr ds:\[eax+4\]  
004758D1    .  311F                    xor dword ptr ds:\[edi\],ebx  
004758D3    .  83C0 04                 add eax,4  
004758D6    .  FF4D F0                 dec dword ptr ss:\[ebp-10\]  
004758D9    .^ 75 EF                   jnz short PasteItI.004758CA      ;  
004758DB    .  E9 8F000000             jmp PasteItI.0047596F  
004758E0    >  8BC1                    mov eax,ecx  
004758E2    .  D1F8                    sar eax,1  
004758E4    .  79 03                   jns short PasteItI.004758E9  
004758E6    .  83D0 00                 adc eax,0  
004758E9    >  48                      dec eax  
004758EA    .  85C0                    test eax,eax  
004758EC    .  7E 17                   jle short PasteItI.00475905  
004758EE    .  8945 F0                 mov dword ptr ss:\[ebp-10\],eax  
004758F1    .  8D45 D0                 lea eax,dword ptr ss:\[ebp-30\]  
004758F4    >  8BD8                    mov ebx,eax  
004758F6    .  8B1B                    mov ebx,dword ptr ds:\[ebx\]  
004758F8    .  8D78 04                 lea edi,dword ptr ds:\[eax+4\]  
004758FB    .  311F                    xor dword ptr ds:\[edi\],ebx  
004758FD    .  83C0 04                 add eax,4  
00475900    .  FF4D F0                 dec dword ptr ss:\[ebp-10\]  
00475903    .^ 75 EF                   jnz short PasteItI.004758F4  
00475905    >  8BC1                    mov eax,ecx  
00475907    .  D1F8                    sar eax,1  
00475909    .  79 03                   jns short PasteItI.0047590E  
0047590B    .  83D0 00                 adc eax,0  
0047590E    >  33DB                    xor ebx,ebx  
00475910    .  8A5C85 CC               mov bl,byte ptr ss:\[ebp+eax\*4-34\]  
00475914    .  8A9B 38C54900           mov bl,byte ptr ds:\[ebx+49C538\]  
0047591A    .  305C85 D0               xor byte ptr ss:\[ebp+eax\*4-30\],bl  
0047591E    .  33DB                    xor ebx,ebx  
00475920    .  8A5C85 CD               mov bl,byte ptr ss:\[ebp+eax\*4-33\]  
00475924    .  8A9B 38C54900           mov bl,byte ptr ds:\[ebx+49C538\]  
0047592A    .  305C85 D1               xor byte ptr ss:\[ebp+eax\*4-2F\],bl  
0047592E    .  33DB                    xor ebx,ebx  
00475930    .  8A5C85 CE               mov bl,byte ptr ss:\[ebp+eax\*4-32\]  
00475934    .  8A9B 38C54900           mov bl,byte ptr ds:\[ebx+49C538\]  
0047593A    .  305C85 D2               xor byte ptr ss:\[ebp+eax\*4-2E\],bl  
0047593E    .  33DB                    xor ebx,ebx  
00475940    .  8A5C85 CF               mov bl,byte ptr ss:\[ebp+eax\*4-31\]  
00475944    .  8A9B 38C54900           mov bl,byte ptr ds:\[ebx+49C538\]  
0047594A    .  305C85 D3               xor byte ptr ss:\[ebp+eax\*4-2D\],bl  
0047594E    .  40                      inc eax  
0047594F    .  8BD9                    mov ebx,ecx  
00475951    .  4B                      dec ebx  
00475952    .  2BD8                    sub ebx,eax  
00475954    .  7C 19                   jl short PasteItI.0047596F  
00475956    .  43                      inc ebx  
00475957    .  895D F0                 mov dword ptr ss:\[ebp-10\],ebx  
0047595A    .  8D4485 CC               lea eax,dword ptr ss:\[ebp+eax\*4-34\]  
0047595E    >  8BD8                    mov ebx,eax  
00475960    .  8B1B                    mov ebx,dword ptr ds:\[ebx\]  
00475962    .  8D78 04                 lea edi,dword ptr ds:\[eax+4\]  
00475965    .  311F                    xor dword ptr ds:\[edi\],ebx  
00475967    .  83C0 04                 add eax,4  
0047596A    .  FF4D F0                 dec dword ptr ss:\[ebp-10\]  
0047596D    .^ 75 EF                   jnz short PasteItI.0047595E  
0047596F    >  33C0                    xor eax,eax  
00475971    .  EB 30                   jmp short PasteItI.004759A3  
00475973    .  EB 1B                   jmp short PasteItI.00475990  
00475975    >  8B5D F8                 mov ebx,dword ptr ss:\[ebp-8\]  
00475978    .  C1E3 02                 shl ebx,2  
0047597B    .  8B7D FC                 mov edi,dword ptr ss:\[ebp-4\]  
0047597E    .  8B7F 30                 mov edi,dword ptr ds:\[edi+30\]  
00475981    .  8D1CDF                  lea ebx,dword ptr ds:\[edi+ebx\*8\]  
00475984    .  8D7C85 D0               lea edi,dword ptr ss:\[ebp+eax\*4-30\]  
00475988    .  8B3F                    mov edi,dword ptr ds:\[edi\]  
0047598A    .  897C93 05               mov dword ptr ds:\[ebx+edx\*4+5\],edi  
0047598E    .  40                      inc eax  
0047598F    .  42                      inc edx  
00475990    >  3BC8                    cmp ecx,eax  
00475992    .  7E 05                   jle short PasteItI.00475999  
00475994    .  83FA 04                 cmp edx,4  
00475997    .^ 7C DC                   jl short PasteItI.00475975  
00475999    >  83FA 04                 cmp edx,4  
0047599C    .  75 05                   jnz short PasteItI.004759A3  
0047599E    .  FF45 F8                 inc dword ptr ss:\[ebp-8\]  
004759A1    .  33D2                    xor edx,edx  
004759A3    >  3BC8                    cmp ecx,eax  
004759A5    .  7E 08                   jle short PasteItI.004759AF  
004759A7    .  8D5E 01                 lea ebx,dword ptr ds:\[esi+1\]  
004759AA    .  3B5D F8                 cmp ebx,dword ptr ss:\[ebp-8\]  ;  
004759AD    .^ 7F E1                   jg short PasteItI.00475990  
004759AF    >  8D46 01                 lea eax,dword ptr ds:\[esi+1\]  
004759B2    .  3B45 F8                 cmp eax,dword ptr ss:\[ebp-8\]  
004759B5    .^ 0F8F ADFEFFFF           jg PasteItI.00475868  
004759BB    .  8B45 FC                 mov eax,dword ptr ss:\[ebp-4\]  
004759BE    .  8B40 30                 mov eax,dword ptr ds:\[eax+30\]  
004759C1    .  8D90 E5010000           lea edx,dword ptr ds:\[eax+1E5\]  
004759C7    .  83C0 05                 add eax,5  
004759CA    .  B9 E0010000             mov ecx,1E0  
004759CF    .  E8 A4D0F8FF             call PasteItI.00402A78  
004759D4    .  8B45 FC                 mov eax,dword ptr ss:\[ebp-4\]  ;生成的密钥地址  
004759D7    .  8B40 30                 mov eax,dword ptr ds:\[eax+30\]  
004759DA    .  0FB640 04               movzx eax,byte ptr ds:\[eax+4\]  
004759DE    .  48                      dec eax  
004759DF    .  85C0                    test eax,eax  
004759E1    .  7E 2F                   jle short PasteItI.00475A12  
004759E3    .  8945 F0                 mov dword ptr ss:\[ebp-10\],eax  
004759E6    .  C745 F8 01000000        mov dword ptr ss:\[ebp-8\],1  
004759ED    >  8B45 F8                 mov eax,dword ptr ss:\[ebp-8\]  
004759F0    .  C1E0 02                 shl eax,2  
004759F3    .  8B55 FC                 mov edx,dword ptr ss:\[ebp-4\]  
004759F6    .  8B52 30                 mov edx,dword ptr ds:\[edx+30\]  
004759F9    .  8D94C2 E5010000         lea edx,dword ptr ds:\[edx+eax\*8+1E5\]  
00475A00    .  B1 04                   mov cl,4  
00475A02    .  8B45 FC                 mov eax,dword ptr ss:\[ebp-4\]  
00475A05    .  E8 52FDFFFF             call PasteItI.0047575C         ;here substitution  
\------------------------------------------------------------------------------  
{                                                                      
00475769   |.  33C0                    xor eax,eax  
0047576B   |>  8BC8                    /mov ecx,eax  
0047576D   |.  C1E1 02                 |shl ecx,2  
00475770   |.  0FB6340A                |movzx esi,byte ptr ds:\[edx+ecx\]  
00475774   |.  8D34B5 C0B44900         |lea esi,dword ptr ds:\[esi\*4+49B4C0\] ;关于这几个固定的表在 miracl 中可以找到  
0047577B   |.  8B36                    |mov esi,dword ptr ds:\[esi\]  
0047577D   |.  0FB67C0A 01             |movzx edi,byte ptr ds:\[edx+ecx+1\]  
00475782   |.  8D3CBD C0B84900         |lea edi,dword ptr ds:\[edi\*4+49B8C0\]  
00475789   |.  3337                    |xor esi,dword ptr ds:\[edi\]  
0047578B   |.  0FB67C0A 02             |movzx edi,byte ptr ds:\[edx+ecx+2\]  
00475790   |.  8D3CBD C0BC4900         |lea edi,dword ptr ds:\[edi\*4+49BCC0\]  
00475797   |.  3337                    |xor esi,dword ptr ds:\[edi\]  
00475799   |.  0FB67C0A 03             |movzx edi,byte ptr ds:\[edx+ecx+3\]  
0047579E   |.  8D3CBD C0C04900         |lea edi,dword ptr ds:\[edi\*4+49C0C0\]  
004757A5   |.  3337                    |xor esi,dword ptr ds:\[edi\]  
004757A7   |.  8D0C0A                  |lea ecx,dword ptr ds:\[edx+ecx\]  
004757AA   |.  8931                    |mov dword ptr ds:\[ecx\],esi  
004757AC   |.  40                      |inc eax  
004757AD   |.  4B                      |dec ebx  
004757AE   |.^ 75 BB                   \\jnz short PasteItI.0047576B  
}  
\------------------------------------------------------------------------------------  
00475A0A    .  FF45 F8                 inc dword ptr ss:\[ebp-8\]  
00475A0D    .  FF4D F0                 dec dword ptr ss:\[ebp-10\]    ;0x9  
00475A10    .^ 75 DB                   jnz short PasteItI.004759ED  
\------------------------------------------------------------------------------------  
子密钥在内在中的形式如下  
00E45E4D  4E E6 02 9D AE 4C D1 15 8C 7F 41 FA 8C 7F 41 FA  N?澁 L??A 鷮A  
00E45E5D  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00E45E6D  86 C3 48 82 28 8F 99 97 A4 F0 D8 6D 28 8F 99 97  喢 H?彊棨鹭 m(彊  
00E45E7D  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00E45E8D  E4 90 B6 2E CC 1F 2F B9 68 EF F7 D4 40 60 6E 43  鋹??/筯秣訞\`nC  
00E45E9D  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00E45EAD  28 FC FB FF E4 E3 D4 46 8C 0C 23 92 CC 6C 4D D1  (�溷訤?#捥 lM  
00E45EBD  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
...............................................  
\-----------------------------------------------------------------------------------------------------------  
004750B2   |.  C646 34 01              mov byte ptr ds:\[esi+34\],1 ;子密钥生成成功标志  
\-----------------------------------------------------------------------------------------------------------  
0048D4AE   |.  8D4D F8                 lea ecx,dword ptr ss:\[ebp-8\]  
0048D4B1   |.  8B55 FC                 mov edx,dword ptr ss:\[ebp-4\]                  ;name "cnbragon"  
0048D4B4 <>|.  8B83 20030000           mov eax,dword ptr ds:\[ebx+320\]                ;  \*Rijndael:TRijndael  
0048D4BA <>|.  E8 897CFEFF             call PasteItI.00475148                        ;  ->:TRijndael.\_PROC\_00475148()  
\-----------------------------------------------------------------------------------------------------------  
下面才开始加密,省略了一大段代码,无非是初始化的工作,正点的在这儿  
00475208   |> /8D45 F8                 /lea eax,dword ptr ss:\[ebp-8\]  
0047520B   |. |E8 18FDF8FF             |call PasteItI.00404F28  
00475210   |. |8D4438 FF               |lea eax,dword ptr ds:\[eax+edi-1\]  
00475214   |. |8D56 5A                 |lea edx,dword ptr ds:\[esi+5A\]  
00475217   |. |B9 10000000             |mov ecx,10  
0047521C   |. |E8 57D8F8FF             |call PasteItI.00402A78                       ;  
\-------------------------------------------------------------------------------------  
经过这个 call 后就可以看到了传说中的 AES 的 state matrix 了,在内存中如下  
00E44C1A  63 6E 62 72 61 67 6F 6E 08 08 08 08 08 08 08 08  cnbragon  
  
前面是用户名,后面的 0x08 是用来填充不足字节的, 即 AES 的 block 长度为 16 字节也就是 128 位,不足的这儿用不足的字节数去补  
即假如用户名的长度为 5 Bytes,那么这儿就要用 16-5=11 来填充了!!  
\--------------------------------------------------------------------------------------  
00475221   |. |83C7 10                 |add edi,10                                     
00475224   |. |8D56 5A                 |lea edx,dword ptr ds:\[esi+5A\]  
00475227   |. |8BC6                    |mov eax,esi  
00475229   |. |E8 6E020000             |call PasteItI.0047549C  
\--------------------------------------------------------------------------------------  
004754C0   |> /8A4C03 36               /mov cl,byte ptr ds:\[ebx+eax+36\]    ;这儿是 state 和上面的 Init Vector 进行 xor  
004754C4   |. |300A                    |xor byte ptr ds:\[edx\],cl  
004754C6   |. |40                      |inc eax  
004754C7   |. |42                      |inc edx  
004754C8   |. |83F8 10                 |cmp eax,10  
004754CB   |.^\\75 F3                   \\jnz short PasteItI.004754C0  
  
004754D2   |.  E8 45050000             call PasteItI.00475A1C  
\-------------------------------------------------------------------------------------  
跟进这个 call 以后我们就开始了万里长征 :)  
其实可以用几行经典的伪代码来搞定它!  
for round = 1 step 1 to Nr–1  
SubBytes(state)   
ShiftRows(state)   
MixColumns(state)   
AddRoundKey(state, w\[round\*Nb, (round+1)\*Nb-1\])  
end for  
SubBytes(state)  
ShiftRows(state)  
AddRoundKey(state, w\[Nr\*Nb, (Nr+1)\*Nb-1\])  
这几行是 AES 加密最经典的了,另一个就是解密时同样经典的代码了,subbytes,shiftrows,mixcolumns,addroundkey  
等函数的详细说明请参考 fips-197,下面一段代码就是 AES 的精华所在了,我把它又精了一下,所以兄弟们只可以看到一点,:)  
\-----------------------------------------------------------------------------------  
  
00475A3A   |>  33C9                    /xor ecx,ecx  
00475A3C   |.  8A0C24                  |mov cl,byte ptr ss:\[esp\]  
00475A3F   |.  8BF9                    |mov edi,ecx  
00475A41   |.  C1E7 02                 |shl edi,2  
00475A44   |.  8B4CF8 05               |mov ecx,dword ptr ds:\[eax+edi\*8+5\]  
00475A48   |.  330A                    |xor ecx,dword ptr ds:\[edx\]  
00475A4A   |.  83C2 04                 |add edx,4  
....  
00475A6D   |.  81E7 FF000000           |and edi,0FF  
00475A73   |.  8B3CBD 38C74900         |mov edi,dword ptr ds:\[edi\*4+49C738\]  
.....  
00475AB7   |.  8B3CBD 38C74900         |mov edi,dword ptr ds:\[edi\*4+49C738\]  
00475ABE   |.  8B6C24 04               |mov ebp,dword ptr ss:\[esp+4\]  
00475AC2   |.  C1ED 08                 |shr ebp,8  
00475AC5   |.  81E5 FF000000           |and ebp,0FF  
.....  
00475B6F   |.  330C9D 38D34900         |xor ecx,dword ptr ds:\[ebx\*4+49D338\]  
00475B76   |.  890A                    |mov dword ptr ds:\[edx\],ecx  
00475B78   |.  FE0424                  |inc byte ptr ss:\[esp\]  
00475B7B   |.  FE4C24 08               |dec byte ptr ss:\[esp+8\]  
00475B7F   |.^ 0F85 B5FEFFFF           \\jnz PasteItI.00475A3A                     ;下面是最后一轮  
00475B85   |>  33C9                    xor ecx,ecx  
00475B87   |.  8A48 04                 mov cl,byte ptr ds:\[eax+4\]  
00475B8A   |.  8BF9                    mov edi,ecx  
00475B8C   |.  C1E7 02                 shl edi,2  
.....  
00475D35   |.  8A48 04                 mov cl,byte ptr ds:\[eax+4\]  
00475D38   |.  C1E1 02                 shl ecx,2  
00475D3B   |.  8B44C8 11               mov eax,dword ptr ds:\[eax+ecx\*8+11\]  
00475D3F   |.  3102                    xor dword ptr ds:\[edx\],eax  
..........  
00475D48   \\.  C3                      retn  
  
到这儿之后,用户名就被加密为密文,在内存中的形式如下,(加密后的 state)  
00E44C1A  1E 9C 30 C1 70 92 08 E8 1B BA CB 7C 79 F8 81 E8  ?羛??核|y 鴣  
\-------------------------------------------------------------------------------------  
接着把加密的密文搬来搬去的,到这儿万里长征只走完了一半!!到了这里  
{  
.....  
00474CB5   |.  8BF8                    mov edi,eax  
00474CB7   |.  BB 01000000             mov ebx,1  
00474CBC   |.  3BFB                    cmp edi,ebx  
00474CBE   |.  0F8C 0F010000           jl PasteItI.00474DD3  
00474CC4   |>  3BFB                    /cmp edi,ebx  
00474CC6   |.  7C 0C                   |jl short PasteItI.00474CD4  
00474CC8   |.  8B45 FC                 |mov eax,dword ptr ss:\[ebp-4\]  
00474CCB   |.  8A4418 FF               |mov al,byte ptr ds:\[eax+ebx-1\]  
00474CCF   |.  8845 F3                 |mov byte ptr ss:\[ebp-D\],al  
00474CD2   |.  EB 04                   |jmp short PasteItI.00474CD8  
00474CD4   |>  C645 F3 00              |mov byte ptr ss:\[ebp-D\],0  
00474CD8   |>  8BC6                    |mov eax,esi  
00474CDA   |.  83E8 01                 |sub eax,1                                    ;  Switch (cases 0..2)  
00474CDD   |.  72 0A                   |jb short PasteItI.00474CE9  
00474CDF   |.  74 34                   |je short PasteItI.00474D15  
00474CE1   |.  48                      |dec eax  
00474CE2   |.  74 72                   |je short PasteItI.00474D56  
00474CE4   |.  E9 D9000000             |jmp PasteItI.00474DC2  
00474CE9   |>  8D45 EC                 |lea eax,dword ptr ss:\[ebp-14\]                ;  Case 0 of switch 00474CDA  
00474CEC   |.  33D2                    |xor edx,edx  
00474CEE   |.  8A55 F3                 |mov dl,byte ptr ss:\[ebp-D\]  
00474CF1   |.  C1EA 02                 |shr edx,2  
00474CF4   |.  8A92 40B34900           |mov dl,byte ptr ds:\[edx+49B340\]  
00474CFA   |.  E8 01FFF8FF             |call PasteItI.00404C00  
00474CFF   |.  8B55 EC                 |mov edx,dword ptr ss:\[ebp-14\]  
00474D02   |.  8D45 F4                 |lea eax,dword ptr ss:\[ebp-C\]  
00474D05   |.  E8 D6FFF8FF             |call PasteItI.00404CE0  
00474D0A   |.  8A45 F3                 |mov al,byte ptr ss:\[ebp-D\]  
00474D0D   |.  8845 F2                 |mov byte ptr ss:\[ebp-E\],al  
00474D10   |.  E9 AD000000             |jmp PasteItI.00474DC2  
00474D15   |>  8D45 E8                 |lea eax,dword ptr ss:\[ebp-18\]                ;  Case 1 of switch 00474CDA  
00474D18   |.  8A55 F2                 |mov dl,byte ptr ss:\[ebp-E\]  
00474D1B   |.  80E2 03                 |and dl,3  
00474D1E   |.  81E2 FF000000           |and edx,0FF  
00474D24   |.  C1E2 04                 |shl edx,4  
00474D27   |.  8A4D F3                 |mov cl,byte ptr ss:\[ebp-D\]  
00474D2A   |.  80E1 F0                 |and cl,0F0  
00474D2D   |.  81E1 FF000000           |and ecx,0FF  
00474D33   |.  C1E9 04                 |shr ecx,4  
00474D36   |.  0BD1                    |or edx,ecx  
00474D38   |.  8A92 40B34900           |mov dl,byte ptr ds:\[edx+49B340\]  
00474D3E   |.  E8 BDFEF8FF             |call PasteItI.00404C00  
00474D43   |.  8B55 E8                 |mov edx,dword ptr ss:\[ebp-18\]  
00474D46   |.  8D45 F4                 |lea eax,dword ptr ss:\[ebp-C\]  
00474D49   |.  E8 92FFF8FF             |call PasteItI.00404CE0  
00474D4E   |.  8A45 F3                 |mov al,byte ptr ss:\[ebp-D\]  
00474D51   |.  8845 F2                 |mov byte ptr ss:\[ebp-E\],al  
00474D54   |.  EB 6C                   |jmp short PasteItI.00474DC2  
00474D56   |>  8D45 E4                 |lea eax,dword ptr ss:\[ebp-1C\]                ;  Case 2 of switch 00474CDA  
00474D59   |.  8A55 F2                 |mov dl,byte ptr ss:\[ebp-E\]  
00474D5C   |.  80E2 0F                 |and dl,0F  
00474D5F   |.  81E2 FF000000           |and edx,0FF  
00474D65   |.  C1E2 02                 |shl edx,2  
00474D68   |.  8A4D F3                 |mov cl,byte ptr ss:\[ebp-D\]  
00474D6B   |.  80E1 C0                 |and cl,0C0  
00474D6E   |.  81E1 FF000000           |and ecx,0FF  
00474D74   |.  C1E9 06                 |shr ecx,6  
00474D77   |.  0BD1                    |or edx,ecx  
00474D79   |.  8A92 40B34900           |mov dl,byte ptr ds:\[edx+49B340\]  
00474D7F   |.  E8 7CFEF8FF             |call PasteItI.00404C00  
00474D84   |.  8B55 E4                 |mov edx,dword ptr ss:\[ebp-1C\]  
00474D87   |.  8D45 F4                 |lea eax,dword ptr ss:\[ebp-C\]  
00474D8A   |.  E8 51FFF8FF             |call PasteItI.00404CE0  
00474D8F   |.  807D F1 00              |cmp byte ptr ss:\[ebp-F\],0  
00474D93   |.  74 04                   |je short PasteItI.00474D99  
00474D95   |.  3BFB                    |cmp edi,ebx  
00474D97   |.  74 25                   |je short PasteItI.00474DBE  
00474D99   |>  8D45 E0                 |lea eax,dword ptr ss:\[ebp-20\]  
00474D9C   |.  8A55 F3                 |mov dl,byte ptr ss:\[ebp-D\]  
00474D9F   |.  80E2 3F                 |and dl,3F  
00474DA2   |.  81E2 FF000000           |and edx,0FF  
00474DA8   |.  8A92 40B34900           |mov dl,byte ptr ds:\[edx+49B340\]  
00474DAE   |.  E8 4DFEF8FF             |call PasteItI.00404C00  
00474DB3   |.  8B55 E0                 |mov edx,dword ptr ss:\[ebp-20\]  
00474DB6   |.  8D45 F4                 |lea eax,dword ptr ss:\[ebp-C\]  
00474DB9   |.  E8 22FFF8FF             |call PasteItI.00404CE0  
00474DBE   |>  C645 F2 00              |mov byte ptr ss:\[ebp-E\],0  
00474DC2   |>  46                      |inc esi                                      ;  Default case of switch 00474CDA  
00474DC3   |.  83FE 03                 |cmp esi,3  
00474DC6   |.  75 02                   |jnz short PasteItI.00474DCA  
00474DC8   |.  33F6                    |xor esi,esi  
00474DCA   |>  43                      |inc ebx  
00474DCB   |.  3BFB                    |cmp edi,ebx  
00474DCD   |.^ 0F8D F1FEFFFF           \\jge PasteItI.00474CC4  
00474E08   \\.  C3                      retn  
}  
上面的这个大循环就是传说中的 base128 了,限于篇幅具体的实现方法请参考偶的注册机源码,偶的代码看起来比这个精简多了:P  
把密文用 base128 表示为"5dkkkL0G0CUPighwSTW-u+"  
\-----------------------------------------------------------------------------------------------------------  
最后一步,生成真码并校验,为了生成真码作者又搞了一个算法出来,发可!  
{  
0048D6A2   |.  8945 F0                 mov dword ptr ss:\[ebp-10\],eax  
0048D6A5   |.  BF 01000000             mov edi,1  
0048D6AA   |>  8B45 FC                 /mov eax,dword ptr ss:\[ebp-4\]  
0048D6AD   |.  0FB64438 FF             |movzx eax,byte ptr ds:\[eax+edi-1\]      ;上面还有几处代码偶没有帖,用来取 base128  
0048D6B2   |.  8BD8                    |mov ebx,eax                            ;的前 0xF 个字符  
0048D6B4   |.  85DB                    |test ebx,ebx                           ;这儿就是 base128 的字符  
0048D6B6   |.  7E 12                   |jle short PasteItI.0048D6CA  
0048D6B8   |>  B8 1A000000             |/mov eax,1A  
0048D6BD   |.  E8 865BF7FF             ||call PasteItI.00403248  
\---------------------------------------------------------------------------------------  
00403248   /$  53                      push ebx  
00403249   |.  31DB                    xor ebx,ebx  
0040324B   |.  6993 08A04900 05840808  imul edx,dword ptr ds:\[ebx+49A008\],8088405  
00403255   |.  42                      inc edx  
00403256   |.  8993 08A04900           mov dword ptr ds:\[ebx+49A008\],edx  
0040325C   |.  F7E2                    mul edx  
0040325E   |.  89D0                    mov eax,edx  
00403260   |.  5B                      pop ebx  
00403261   \\.  C3                      retn  
\---------------------------------------------------------------------------------------  
0048D6C2   |.  8BF0                    ||mov esi,eax  
0048D6C4   |.  83C6 41                 ||add esi,41  
0048D6C7   |.  4B                      ||dec ebx  
0048D6C8   |.^ 75 EE                   |\\jnz short PasteItI.0048D6B8           ;这个小循环和中间的靠就是用来生成真码的了  
0048D6CA   |>  8D45 E4                 |lea eax,dword ptr ss:\[ebp-1C\]          ;不难,做注册机时可以直接用 inline asm 来做  
0048D6CD   |.  8BD6                    |mov edx,esi                            ;特方便  
0048D6CF <>|.  E8 2C75F7FF             |call PasteItI.00404C00                       ;  ->System.@LStrFromChar(String;String;Char);<+>  
0048D6D4   |.  8B55 E4                 |mov edx,dword ptr ss:\[ebp-1C\]  
0048D6D7   |.  8D45 F4                 |lea eax,dword ptr ss:\[ebp-C\]  
0048D6DA <>|.  E8 0176F7FF             |call PasteItI.00404CE0                       ;  ->System.@LStrCat;  
0048D6DF   |.  8B45 F4                 |mov eax,dword ptr ss:\[ebp-C\]  
0048D6E2   |.  8A4438 FF               |mov al,byte ptr ds:\[eax+edi-1\]  
0048D6E6   |.  8B55 F8                 |mov edx,dword ptr ss:\[ebp-8\]  
0048D6E9   |.  3A443A FF               |cmp al,byte ptr ds:\[edx+edi-1\]        ;真码和假码就在这儿相遇了,一个一个 comp  
0048D6ED   |.  75 11                   |jnz short PasteItI.0048D700  
0048D6EF   |.  47                      |inc edi  
0048D6F0   |.  FF4D F0                 |dec dword ptr ss:\[ebp-10\]  
0048D6F3   |.^ 75 B5                   \\jnz short PasteItI.0048D6AA  
}  
  
到这儿就结束了,没有什么好总结的了,熟悉了 AES 做出注册机不成问题 Good luck  
  
Regards  
  
附上我的注册机源码,无编译好的 keygen,VC6 下编译通过  
附件:keygen.zip 

 4dd466d0e1e0cc123afbe504da2462ca