

# Aplus DVD to MP3 Ripper 3.1.3

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-240.htm`](https://www.pediy.com/kssd/pediy08/pediy8-240.htm)

【破解软件】Aplus DVD to MP3 Ripper 3.1.3  
【下载地址】http://www.onlinedown.net/soft/49463.htm  
【软件类别】国外软件/共享版/MP3 制作  
【运行环境】Win9x/Me/NT/2000/XP/2003  
【保护方式】序列号   
【调试工具】Winxp、OllyDBD、PEiD  
【作者声明】初学 Crack，只是感兴趣，消遣业余时间，错误之处敬请诸位前辈不吝赐教。  
【软件信息】是一款易于使用的 MP3 转换软件。该软件可以绕过版权保护将 DVD 音乐转换成可播放的 MP3 格式文件   
  
一、  
PEiD 检查 Microsoft Visual C++ 6.0 无壳正好给我这样的新手练习，：）  
KANAL 分析：CRC16,Miracl mirvar,Miracl powmod,倒还没正规的碰过这东西，看看先（  
输入 Serail：a1234567b1234567c1234567d1234567e1234567f1234567，单击“Register”提示“Serial number error!”  
  
二、算法跟踪  
  
OD 载入程序查找字串参考，来到下面：  
  
00407104  /.  55            push    ebp  
00407105  |.  8BEC          mov     ebp, esp  
00407107  |.  6A FF         push    -1  
00407109  |.  68 60E84700   push    0047E860                         ;  SE 处理程序安装  
0040710E  |.  64:A1 0000000>mov     eax, fs:\[0\]  
00407114  |.  50            push    eax  
00407115  |.  64:8925 00000>mov     fs:\[0\], esp  
0040711C  |.  81EC 9C000000 sub     esp, 9C  
00407122  |.  8965 F0       mov     \[local.4\], esp  
00407125  |.  897D E0       mov     \[local.8\], edi  
00407128  |.  8975 E4       mov     \[local.7\], esi  
0040712B  |.  895D E8       mov     \[local.6\], ebx  
0040712E  |.  894D EC       mov     \[local.5\], ecx  
00407131  |.  A1 80054900   mov     eax, ds:\[490580\]  
00407136  |.  8985 58FFFFFF mov     \[local.42\], eax  
0040713C  |.  C745 FC 00000>mov     \[local.1\], 0  
00407143  |.  8BC1          mov     eax, ecx  
00407145  |.  8D95 58FFFFFF lea     edx, \[local.42\]  
0040714B  |.  52            push    edx                              ;  (initial cpu selection)  
0040714C  |.  8D88 5C040000 lea     ecx, ds:\[eax+45C\]  
00407152  |.  E8 99B30600   call    004724F0                         ;    
00407157  |.  8D8D 58FFFFFF lea     ecx, \[local.42\]  
0040715D  |.  E8 BF870600   call    0046F921  
00407162  |.  8D8D 58FFFFFF lea     ecx, \[local.42\]  
00407168  |.  E8 68870600   call    0046F8D5  
0040716D  |.  8D85 5CFFFFFF lea     eax, \[local.41\]  
00407173  |.  50            push    eax                              ; /pHandle  
00407174  |.  68 9C887300   push    0073889C                         ; |software\\aplus dvd to mp3 ripper  
00407179  |.  68 01000080   push    80000001                         ; |hKey = HKEY\_CURRENT\_USER  
0040717E  |.  FF15 04004800 call    near ds:\[<&ADVAPI32.RegCreateKey>; \\RegCreateKeyA  
00407184  |.  85C0          test    eax, eax  
00407186  |.  75 51         jnz     short 004071D9  
00407188  |.  FFB5 58FFFFFF push    \[local.42\]  
0040718E  |.  68 90887300   push    00738890                         ;  %s  
00407193  |.  8D85 60FFFFFF lea     eax, \[local.40\]  
00407199  |.  50            push    eax  
0040719A  |.  E8 69740500   call    0045E608  
0040719F  |.  83C4 0C       add     esp, 0C  
004071A2  |.  8DBD 60FFFFFF lea     edi, \[local.40\]  
004071A8  |.  33C0          xor     eax, eax  
004071AA  |.  8A37          mov     dh, ds:\[edi\]  
004071AC  |.  8BCF          mov     ecx, edi  
004071AE  |.  84F6          test    dh, dh  
004071B0  |.  74 0C         je      short 004071BE  
004071B2  |>  83C1 01       /add     ecx, 1  
004071B5  |.  83C0 01       |add     eax, 1  
004071B8  |.  8A11          |mov     dl, ds:\[ecx\]  
004071BA  |.  84D2          |test    dl, dl  
004071BC  |.^ 75 F4         \\jnz     short 004071B2  
004071BE  |>  8D95 60FFFFFF lea     edx, \[local.40\]  
004071C4  |.  50            push    eax                              ; /Length  
004071C5  |.  52            push    edx                              ; |Value  
004071C6  |.  6A 01         push    1                                ; |ValueType = REG\_SZ  
004071C8  |.  68 E0957300   push    007395E0                         ; |regcode  
004071CD  |.  FFB5 5CFFFFFF push    \[local.41\]                       ; |hKey  
004071D3  |.  FF15 20004800 call    near ds:\[<&ADVAPI32.RegSetValueA>; \\RegSetValueA  
004071D9  |>  8D8D 58FFFFFF lea     ecx, \[local.42\]         ;   
把 Serail 写入注册表 HKEY\_CURRENT\_USER\\Software\\Aplus DVD to MP3 Ripper\\RegCode  
(不判断直接写入，程序在启动的时候肯定还会再次判断)  
004071DF  |.  68 80000000   push    80  
004071E4  |.  E8 A8D50600   call    00474791  
004071E9  |.  57            push    edi  
004071EA  |.  E8 A1000000   call    00407290                         ;  关键 call  
004071EF  |.  59            pop     ecx  
004071F0  |.  85C0          test    eax, eax           ;   
004071F2      74 35         je      short 00407229         ;跳了就 over  
004071F4  |.  8B0D 1C474900 mov     ecx, ds:\[49471C\]  
004071FA  |.  33C0          xor     eax, eax  
004071FC  |.  50            push    eax  
004071FD  |.  50            push    eax  
004071FE  |.  68 E8957300   push    007395E8                         ;  thank you for registering!  
00407203  |.  E8 F2BE0600   call    004730FA  
00407208  |.  68 D08C7300   push    00738CD0                         ;    aplus dvd to mp3 ripper  
0040720D  |.  A1 1C474900   mov     eax, ds:\[49471C\]  
00407212  |.  8B0D A8E56600 mov     ecx, ds:\[66E5A8\]  
00407218  |.  C780 9C180000>mov     dword ptr ds:\[eax+189C\], 1  
00407222  |.  E8 F7D70600   call    00474A1E  
00407227  |.  EB 36         jmp     short 0040725F  
00407229  |>  8B0D 1C474900 mov     ecx, ds:\[49471C\]  
0040722F  |.  6A 10         push    10  
00407231  |.  68 90767300   push    00737690                         ;  error  
00407236  |.  68 04967300   push    00739604                         ;  serial number error!  
0040723B  |.  E8 BABE0600   call    004730FA  
00407240  |.  68 EC8C7300   push    00738CEC                         ;    aplus dvd to mp3 ripper    trial version  
  
  
  
00407290  /$  57            push    edi  
00407291  |.  56            push    esi  
00407292  |.  55            push    ebp  
00407293  |.  53            push    ebx  
00407294  |.  81EC 10060000 sub     esp, 610  
0040729A  |.  8BD8          mov     ebx, eax           ;可以看到我们输入的伪码在 eax 中  
0040729C  |.  8BFB          mov     edi, ebx  
0040729E  |.  33C0          xor     eax, eax  
004072A0  |.  8A37          mov     dh, ds:\[edi\]                     ;   
004072A2  |.  8BCF          mov     ecx, edi  
004072A4  |.  84F6          test    dh, dh           ;判断 Serail 是否为空  
004072A6  |.  74 0C         je      short 004072B4  
004072A8  |>  83C1 01       /add     ecx, 1  
004072AB  |.  83C0 01       |add     eax, 1  
004072AE  |.  8A11          |mov     dl, ds:\[ecx\]  
004072B0  |.  84D2          |test    dl, dl  
004072B2  |.^ 75 F4         \\jnz     short 004072A8  
004072B4  |>  83F8 30       cmp     eax, 30                          ;    
004072B7  |.  74 0D         je      short 004072C6         ；输入的 Serail 的长度不是 0x30 就 over  
004072B9  |>  33C0          xor     eax, eax  
004072BB  |.  81C4 10060000 add     esp, 610  
004072C1  |.  5B            pop     ebx  
004072C2  |.  5D            pop     ebp  
004072C3  |.  5E            pop     esi  
004072C4  |.  5F            pop     edi  
004072C5  |.  C3            retn  
004072C6  |>  8D7C24 10     lea     edi, ss:\[esp+10\]                 ;    
004072CA  |.  BE 40937300   mov     esi, 00739340  
004072CF  |.  B9 40000000   mov     ecx, 40  
004072D4  |.  F3:A5         rep     movs dword ptr es:\[edi\], dword p>;    
004072D6  |.  8DBC24 100100>lea     edi, ss:\[esp+110\]         ;  清 0x40＊4 长度的内存  
004072DD  |.  BE 40927300   mov     esi, 00739240  
004072E2  |.  B9 40000000   mov     ecx, 40  
004072E7  |.  F3:A5         rep     movs dword ptr es:\[edi\], dword p>  
004072E9  |.  8DBC24 100200>lea     edi, ss:\[esp+210\]        ;  清 0x40＊4 长度的内存  
004072F0  |.  BE 40917300   mov     esi, 00739140  
004072F5  |.  B9 40000000   mov     ecx, 40  
004072FA  |.  F3:A5         rep     movs dword ptr es:\[edi\], dword p>  
004072FC  |.  8D5424 10     lea     edx, ss:\[esp+10\]                 ;  清 0x40＊4 长度的内存  
00407300  |.  6A 10         push    10  
00407302  |.  53            push    ebx  
00407303  |.  52            push    edx  
00407304  |.  E8 E7770500   call    0045EAF0           ;取伪码头 16 位写入刚才清空的地方  
00407309  |.  8D8C24 1C0100>lea     ecx, ss:\[esp+11C\]  
00407310  |.  6A 10         push    10  
00407312  |.  8D53 10       lea     edx, ds:\[ebx+10\]  
00407315  |.  52            push    edx  
00407316  |.  51            push    ecx  
00407317  |.  E8 D4770500   call    0045EAF0                         ;取伪码中间 16 位写入刚才清空的地方  
0040731C  |.  8D9424 280200>lea     edx, ss:\[esp+228\]  
00407323  |.  6A 10         push    10  
00407325  |.  83C3 20       add     ebx, 20  
00407328  |.  53            push    ebx  
00407329  |.  52            push    edx  
0040732A  |.  E8 C1770500   call    0045EAF0           ;取伪码尾部 16 位写入刚才清空的地方  
0040732F  |.  33C0          xor     eax, eax                         ;    
00407331  |.  884424 44     mov     ss:\[esp+44\], al                  ;    
00407335  |.  888424 440100>mov     ss:\[esp+144\], al                 ;  在上面刚分的 3 段尾部各写入 00 结束标记  
0040733C  |.  888424 440200>mov     ss:\[esp+244\], al  
  
00407343  |.  50            push    eax  
00407344  |.  6A 64         push    64  
00407346  |.  E8 E5A50400   call    00451930                         ;  miracl \*mip=mirsys(100,0)  
0040734B  |.  C780 34020000>mov     dword ptr ds:\[eax+234\], 10       ;  mip->IOBASE=16  
00407355  |.  6A 00         push    0  
00407357  |.  E8 04A40400   call    00451760                         ;  m/明文  big m=mirvar(0)  
0040735C  |.  8BE8          mov     ebp, eax                         ;    
0040735E  |.  6A 00         push    0  
00407360  |.  E8 FBA30400   call    00451760                         ;  c/密文  big m=mirvar(0)  
00407365  |.  894424 38     mov     ss:\[esp+38\], eax                 ;    
00407369  |.  6A 00         push    0  
0040736B  |.  E8 F0A30400   call    00451760                         ;  n/模数  big m=mirvar(0)  
00407370  |.  8BD8          mov     ebx, eax                         ;    
00407372  |.  6A 00         push    0  
00407374  |.  E8 E7A30400   call    00451760                         ;  e/公钥  big m=mirvar(0)  
00407379  |.  894424 44     mov     ss:\[esp+44\], eax                 ;    
0040737D  |.  83C4 3C       add     esp, 3C  
00407380  |.  8DBC24 100300>lea     edi, ss:\[esp+310\]  
00407387  |.  BE 40907300   mov     esi, 00739040  
0040738C  |.  B9 40000000   mov     ecx, 40  
00407391  |.  F3:A5         rep     movs dword ptr es:\[edi\], dword p>  
00407393  |.  8D7C24 10     lea     edi, ss:\[esp+10\]  
00407397  |.  33C0          xor     eax, eax  
00407399  |.  8A37          mov     dh, ds:\[edi\]  
0040739B  |.  8BCF          mov     ecx, edi  
0040739D  |.  84F6          test    dh, dh  
0040739F  |.  74 0C         je      short 004073AD  
004073A1  |>  83C1 01       /add     ecx, 1  
004073A4  |.  83C0 01       |add     eax, 1  
004073A7  |.  8A11          |mov     dl, ds:\[ecx\]           ;获取第一段的长度 0x10  
004073A9  |.  84D2          |test    dl, dl  
004073AB  |.^ 75 F4         \\jnz     short 004073A1  
004073AD  |>  8BF8          mov     edi, eax  
004073AF  |.  33F6          xor     esi, esi  
004073B1  |.  85FF          test    edi, edi  
004073B3  |.  7E 17         jle     short 004073CC  
004073B5  |>  0FBE5434 10   /movsx   edx, byte ptr ss:\[esp+esi+10\]  
004073BA  |.  52            |push    edx  
004073BB  |.  E8 2E780500   |call    0045EBEE  
004073C0  |.  59            |pop     ecx  
004073C1  |.  85C0          |test    eax, eax  
004073C3  |.  74 43         |je      short 00407408                  ;  检查第一段的数据  
004073C5  |.  83C6 01       |add     esi, 1  
004073C8  |.  3BF7          |cmp     esi, edi  
004073CA  |.^ 7C E9         \\jl      short 004073B5  
004073CC  |>  85FF          test    edi, edi  
004073CE  |.  74 38         je      short 00407408  
004073D0  |.  8D5424 10     lea     edx, ss:\[esp+10\]  
004073D4  |.  52            push    edx                              ;  伪码的第一段  
004073D5  |.  55            push    ebp                              ;    
004073D6  |.  E8 15B70400   call    00452AF0                         ;  初始化明文,cinstr(m,serail1)  
004073DB  |.  68 F4857300   push    007385F4                         ;  a1c397b69b55b291  
004073E0  |.  53            push    ebx                              ;  模数在我这里是明文  
004073E1  |.  E8 0AB70400   call    00452AF0                         ;  模数初始化，cinstr(n,a1c397b69b55b291)  
004073E6  |.  68 EC857300   push    007385EC                         ;  10001  
004073EB  |.  8B5424 1C     mov     edx, ss:\[esp+1C\]  
004073EF  |.  52            push    edx                              ;    
004073F0  |.  E8 FBB60400   call    00452AF0                         ;  公钥初始化，cinstr(e,10001)  
004073F5  |.  53            push    ebx                              ;    
004073F6  |.  55            push    ebp                              ;    
004073F7  |.  E8 B4B10400   call    004525B0                         ;  比较模数和明文 compare(m,n)  
004073FC  |.  83C4 20       add     esp, 20                          ;    
004073FF  |.  83F8 FF       cmp     eax, -1  
00407402      0F84 D4020000 je      004076DC           ;  m<n 跳走继续运算(这里如果不跳后面参与运算的值就根本没有)  
00407408  |>  8D8424 100300>lea     eax, ss:\[esp+310\]                ;    
0040740F  |.  50            push    eax                              ;    
00407410  |.  E8 D0720500   call    0045E6E5                         ;    
  
第一次参与的数据  
//M(HEX)伪码的第一段  
//N(HEX)a1c397b69b55b291  
//D(HEX)3BD60FDDDF705D29        
//E(HEX)10001  
//Keysize(Bits)=64  
  
后面是重复上面第一次的过程只是几个参数不一样所以略过  
第二次    
//M(HEX)伪码的第一段  
//N(HEX)909ef746744d1931  
//D(HEX)55F020BFB0DE4E81        
//E(HEX)10001  
//Keysize(Bits)=64  
第三次  
//M(HEX)伪码的第一段  
//N(HEX)909ef746744d1931  
//D(HEX)55F020BFB0DE4E81        
//E(HEX)10001  
//Keysize(Bits)=64  
  
  
004075D1  |.  E8 DAAF0400   call    004525B0           ;第 3 次的比较模数和明文 compare(m,n)  
004075D6  |.  83C4 20       add     esp, 20  
004075D9  |.  83F8 FF       cmp     eax, -1  
004075DC  |.  74 40         je      short 0040761E  
004075DE  |>  8D9424 100500>lea     edx, ss:\[esp+510\]                ;    
004075E5  |.  52            push    edx  
004075E6  |.  E8 FA700500   call    0045E6E5                         ; 这个要进去看  
004075EB  |.  59            pop     ecx  
004075EC  |.  85DB          test    ebx, ebx                         ;  第 1 次的结果为 0 就 over  
004075EE  |.^ 0F8E C5FCFFFF jle     004072B9  
004075F4  |.  85ED          test    ebp, ebp                         ;  第 2 次的结果为 0 就 over  
004075F6  |.^ 0F8E BDFCFFFF jle     004072B9  
004075FC  |.  85C0          test    eax, eax                         ;  第 3 次的结果为 0 就 over  
004075FE  |.^ 0F8E B5FCFFFF jle     004072B9  
00407604  |.  03D8          add     ebx, eax  
00407606  |.  3BEB          cmp     ebp, ebx                         ;  ebx+eax=ebp  
00407608  |.^ 0F85 ABFCFFFF jnz     004072B9           ;要求第 1 次＋第 3 次＝第 2 次  
0040760E  |.  B8 01000000   mov     eax, 1  
00407613  |.  81C4 10060000 add     esp, 610  
00407619  |.  5B            pop     ebx  
0040761A  |.  5D            pop     ebp  
0040761B  |.  5E            pop     esi  
0040761C  |.  5F            pop     edi  
0040761D  |.  C3            retn  
0040761E  |>  8B1424        mov     edx, ss:\[esp\]               前面比较 m<n 就是跳到这里了  
00407621  |.  52            push    edx  
00407622  |.  8B4C24 10     mov     ecx, ss:\[esp+10\]  
00407626  |.  51            push    ecx  
00407627  |.  8B7424 0C     mov     esi, ss:\[esp+C\]  
0040762B  |.  56            push    esi  
0040762C  |.  8B7C24 14     mov     edi, ss:\[esp+14\]  
00407630  |.  57            push    edi  
00407631  |.  E8 8ABB0400   call    004531C0      ;powmod(m,e,n,c)//计算 c=m^e mod n  
00407636  |.  8D9424 200500>lea     edx, ss:\[esp+520\]  
0040763D  |.  6A 00         push    0  
0040763F  |.  52            push    edx  
00407640  |.  8B4C24 18     mov     ecx, ss:\[esp+18\]  
00407644  |.  51            push    ecx  
00407645  |.  68 00010000   push    100  
0040764A  |.  E8 A1C20400   call    004538F0      ;big\_to\_bytes(256,c,temp,FALSE);  
0040764F  |.  8BD7          mov     edx, edi      这个 temp 的值就是后面运算需要的  
00407651  |.  52            push    edx  
00407652  |.  E8 B9A90400   call    00452010      ;mirkill(m);//释放内存  
00407657  |.  8B5424 24     mov     edx, ss:\[esp+24\]  
0040765B  |.  52            push    edx  
0040765C  |.  E8 AFA90400   call    00452010      ;mirkill(c);//释放内存  
00407661  |.  8B5424 34     mov     edx, ss:\[esp+34\]  
00407665  |.  52            push    edx  
00407666  |.  E8 A5A90400   call    00452010      ;mirkill(n);//释放内存  
0040766B  |.  8BD6          mov     edx, esi  
0040766D  |.  52            push    edx  
0040766E  |.  E8 9DA90400   call    00452010      ;mirkill(e);//释放内存  
00407673  |.  83C4 30       add     esp, 30  
00407676  |.  E8 B5A90400   call    00452030      ;mirexit();//释放内存  
0040767B  |.^ E9 5EFFFFFF   jmp     004075DE  
00407680  |>  .  
.  
.  
  
0045E6E5  /$  FF7424 04     push    dword ptr ss:\[esp+4\]  
0045E6E9  |.  E8 6CFFFFFF   call    0045E65A  
0045E6EE  |.  59            pop     ecx  
0045E6EF  \\.  C3            retn  
  
0045E65A  /$  53            push    ebx  
0045E65B  |.  55            push    ebp  
0045E65C  |.  56            push    esi  
0045E65D  |.  57            push    edi  
0045E65E  |.  8B7C24 14     mov     edi, ss:\[esp+14\]  
0045E662  |>  833D B41B4900>/cmp     dword ptr ds:\[491BB4\], 1            ;  00491bb4 永远都是 1  
0045E669  |.  7E 0F         |jle     short 0045E67A  
0045E66B  |.  0FB607        |movzx   eax, byte ptr ds:\[edi\]  
0045E66E  |.  6A 08         |push    8  
0045E670  |.  50            |push    eax  
0045E671  |.  E8 CC580000   |call    00463F42  
0045E676  |.  59            |pop     ecx  
0045E677  |.  59            |pop     ecx  
0045E678  |.  EB 0F         |jmp     short 0045E689  
0045E67A  |>  0FB607        |movzx   eax, byte ptr ds:\[edi\]              ;取出前面运算出的 temp 的值(rsa 的值)  
0045E67D  |.  8B0D A8194900 |mov     ecx, ds:\[4919A8\]                    ;  dvdtomp3.004919B2  
0045E683  |.  8A0441        |mov     al, ds:\[ecx+eax\*2\]        
0045E686  |.  83E0 08       |and     eax, 8        ;  
0045E689  |>  85C0          |test    eax, eax  
0045E68B  |.  74 03         |je      short 0045E690  
0045E68D  |.  47            |inc     edi  
0045E68E  |.^ EB D2         \\jmp     short 0045E662  
\[rsa\[i\]\*2+4919B2\]是\*8 之类的就取下一个,继续循环,否则就走  
0045E690  |>  0FB637        movzx   esi, byte ptr ds:\[edi\]  
0045E693  |.  47            inc     edi  
0045E694  |.  83FE 2D       cmp     esi, 2D                              ;  是 2D 在返回前就会把结果 neg eax  
0045E697  |.  8BEE          mov     ebp, esi  
0045E699  |.  74 05         je      short 0045E6A0  
0045E69B  |.  83FE 2B       cmp     esi, 2B                              ;  如果是 2D，2B 就取下一位  
0045E69E  |.  75 04         jnz     short 0045E6A4  
0045E6A0  |>  0FB637        movzx   esi, byte ptr ds:\[edi\]  
0045E6A3  |.  47            inc     edi  
0045E6A4  |>  33DB          xor     ebx, ebx  
0045E6A6  |>  833D B41B4900>/cmp     dword ptr ds:\[491BB4\], 1  
0045E6AD  |.  7E 0C         |jle     short 0045E6BB  
0045E6AF  |.  6A 04         |push    4  
0045E6B1  |.  56            |push    esi  
0045E6B2  |.  E8 8B580000   |call    00463F42  
0045E6B7  |.  59            |pop     ecx  
0045E6B8  |.  59            |pop     ecx  
0045E6B9  |.  EB 0B         |jmp     short 0045E6C6  
0045E6BB  |>  A1 A8194900   |mov     eax, ds:\[4919A8\]  
0045E6C0  |.  8A0470        |mov     al, ds:\[eax+esi\*2\]                  ;  4919A8+Rsa\[i\]\*2  
0045E6C3  |.  83E0 04       |and     eax, 4                              ; 35(5),23(3),ab(b)  
0045E6C6  |>  85C0          |test    eax, eax  
0045E6C8  |.  74 0D         |je      short 0045E6D7  
0045E6CA  |.  8D049B        |lea     eax, ds:\[ebx+ebx\*4\]                 ;  eax=ebx\*5,ebx 初始为 0  
0045E6CD  |.  8D5C46 D0     |lea     ebx, ds:\[esi+eax\*2-30\]              ;  ebx=ebx\*5\*2-30+RSA\[i\]  
0045E6D1  |.  0FB637        |movzx   esi, byte ptr ds:\[edi\]  
0045E6D4  |.  47            |inc     edi  
0045E6D5  |.^ EB CF         \\jmp     short 0045E6A6  
0045E6D7  |>  83FD 2D       cmp     ebp, 2D  
0045E6DA  |.  8BC3          mov     eax, ebx  
0045E6DC  |.  75 02         jnz     short 0045E6E0  
0045E6DE  |.  F7D8          neg     eax  
0045E6E0  |>  5F            pop     edi  
0045E6E1  |.  5E            pop     esi  
0045E6E2  |.  5D            pop     ebp  
0045E6E3  |.  5B            pop     ebx  
0045E6E4  \\.  C3            retn  
\[rsa\[i\]\*2+4919B2\]是\*4 之类的就运算， 取下一个继续循环,否则退出循环  
  
我们假设 rsa\[i\]=FF,4919B2<=\[rsa 的值\*2+4919B2\]<=491BB0,下面我们就去看看这块区域的值  
rsa\[i\]={0x9,0xA,0xB,0xC,0xD,0x20}   \[rsa\[i\]\*2+4919B2\]的结果是\*8  
rsa\[i\]={0x30,0x31,0x32,0x33,0x34,0x35,0x36,0x37,0x38,0x39}\[rsa\[i\]\*2+4919B2\]的结果是\*4  
  
序列号三  
我们利用她的流程来个最简单的:)  
第一次=1,   rsa\[i\]={0x31,0x3A) ,(1:)    185B973541F05CF1   3A 随便选个,只要不上面的表就可以了  
第二次=3,   rsa\[i\]={0x33,0x3A) ,(3:)    1177CA7E07FB44E1  
第三次=2,   rsa\[i\]={0x32,0x3A) ;(2:)用 rsa 运算得出 53945D865D558F7A  
  
serail=185B973541F05CF11177CA7E07FB44E153945D865D558F7A  
  
  
典型的利用 Miracl 大数运算库进行 RSA 算法的实例,那些函数看 Miracl 的参考  
用 Rsatools2 工具输入模数 n，得到下面的几组数据  
//P(HEX)10138CEE3         94E13703          
//Q(HEX)A0FEDEFB    F8AD4EBB  
//N(HEX)a1c397b69b55b291,    909ef746744d1931  
//D(HEX)3BD60FDDDF705D29,  55F020BFB0DE4E81        
//E(HEX)10001  
//Keysize(Bits)=64  
  
    
  
刚好弄完看世界杯![](img/dd27b7eb3e47f4f9a8b69ac3c32485f5.jpg)

 4dd466d0e1e0cc123afbe504da2462ca