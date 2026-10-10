

# ASProtect.SKE.2.11 stolen code 解密

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-619.htm`](https://www.pediy.com/kssd/pediy08/pediy8-619.htm)

ASProtect.SKE.2.11 stolen code 解密  
关于 ASProtect.SKE.2.11 stolen code 方面的文章已经很多了，今天我想再详细地说说它的细节，献给喜欢破解的兄弟们。  
  
stolen code 并不可怕！  
  
ASProtect.SKE.2.11 将入口处的一部分代码放到壳里了，看下面：  
未加壳的入口点：  
00401350 >    55            push    ebp  
00401351      8BEC        mov     ebp, esp  
00401353      6A FF         push    -1  
00401355      68 F8504000   push    004050F8  
0040135A      68 1C204000   push    0040201C                           
0040135F      64:A1 0000000>mov     eax, fs:\[0\]  
00401365      50            push    eax  
00401366      64:8925 00000>mov     fs:\[0\], esp  
0040136D      83EC 58       sub     esp, 58  
00401370      53            push    ebx  
00401371      56            push    esi  
00401372      57            push    edi  
00401373      8965 E8       mov     \[ebp-18\], esp  
00401376      FF15 60504000 call    \[<&KERNEL32.GetVersion>\]         ;   
加壳后的入口点：  
00C70270    55              push    ebp  
00C70271    EB 02           jmp     short 00C70275  
00C70273    CD20 13EE83CD   vxdjump CD83EE13  
00C70279    AB              stos    dword ptr es:\[edi\]  
00C7027A    BD 0E954300     mov     ebp, 43950E  
00C7027F    8D6C24 3B       lea     ebp, \[esp+3B\]  
00C70283    F3:             prefix rep:  
00C70284    EB 02           jmp     short 00C70288  
00C70286    CD20 8D6DC56A   vxdcall 6AC56D8D  
00C7028C    FF64FF 35       jmp     \[edi+edi\*8+35\]  
  
你可以看到它的很多地方经过变形已经面目全非，跳转不定。如果只是顺序乱的，就算有很多垃圾代码也是可以忍受的。但你看到的是不停地跳到壳里再跳回来的 call:  
00C702C9   /E9 040F0000      jmp     00C711D2  
00C702CE   |E8 2DFD0500     call    00CD0000              //call  
00C702D3   |8B45 FC          mov     eax, \[ebp-4\]  
………  
00C702FF    68 6307C700     push    0C70763  
00C70304    E8 F7FC0500     call    00CD0000        //call  
00C70309    8D4475 63        lea     eax, \[ebp+esi\*2+63\]      
  
看到 call    00CD0000 了吗？  
在被偷的这些代码中有很多这样的 call，它们与未被偷部分代码里的 call 不同：  
未被偷部分代码里的 call：  
0040113D  |.  51            push    ecx  
0040113E  |.  E8 BDEE8800   call    00C90000  
00401143  |.  EF            out     dx, eax  
这些 call    00C90000 是 Advanced Import protection 保护（对系统函数 call 的加密）  
被偷部分代码里的 call：  
00C702FF    68 6307C700     push    0C70763  
00C70304    E8 F7FC0500     call    00CD0000  
00C70309    8D4475 63        lea     eax, \[ebp+esi\*2+63\]    //call  
这些 call    00CD0000 实际上是代码变形！！！！  
  
进入 call    00CD0000：  
00C7079E    E8 5DF80500     call    00CD0000  
{  
//第一层  
00CD0000    F2:             prefix repne:  
00CD0001    EB 01           jmp     short 00CD0004  
00CD0003    9A 509C3344 242>call    far 2824:44339C50  
00CD000A    334424 08       xor     eax, \[esp+8\]  
00CD000E    83EC 20         sub     esp, 20  
00CD0011    81F0 8C87B250   xor     eax, 50B2878C  
。。。。  
一路 F7,基本是些 pushad,pushfd 的变形  
00CD0151    C1C0 B5         rol     eax, 0B5  
00CD0154    334424 08       xor     eax, \[esp+8\]  
00CD0158    58              pop     eax  
00CD0159    FFD0            call    eax          //  
{  
  //第二层  
  一路 F8 来到一个循环：  
  00A88A87    8B45 F8         mov     eax, \[ebp-8\]  
00A88A8A    0FB600          movzx   eax, byte ptr \[eax\]  
00A88A8D    8B5483 40       mov     edx, \[ebx+eax\*4+40\]  
00A88A91    8BC6            mov     eax, esi  
00A88A93    FFD2            call    edx  
00A88A95    3B45 FC         cmp     eax, \[ebp-4\]  
00A88A98    75 1A           jnz     short 00A88AB4  
00A88A9A    8B45 10         mov     eax, \[ebp+10\]  
00A88A9D    50              push    eax  
00A88A9E    8B45 14         mov     eax, \[ebp+14\]  
00A88AA1    50              push    eax  
00A88AA2    E8 19FAFFFF     call    00A884C0  
00A88AA7    50              push    eax  
00A88AA8    8BCE            mov     ecx, esi  
00A88AAA    8B55 18         mov     edx, \[ebp+18\]  
00A88AAD    8BC3            mov     eax, ebx  
00A88AAF    E8 D4FDFFFF     call    00A88888  
00A88AB4    4F              dec     edi  
00A88AB5    0373 6C         add     esi, \[ebx+6C\]  
00A88AB8    85FF            test    edi, edi  
00A88ABA  ^ 77 CB           ja      short 00A88A87  
在 00A88AAF  call    00A88888 上 F4  
00A88AAF    E8 D4FDFFFF     call    00A88888  //F7  
{  
  00A88888    55              push    ebp  
00A88889    8BEC            mov     ebp, esp  
…….  
一路 F8 来到:  
00A888CC    8A43 01         mov     al, \[ebx+1\]  
00A888CF    8B55 F8         mov     edx, \[ebp-8\]  
00A888D2    8B5482 40       mov     edx, \[edx+eax\*4+40\]  
00A888D6    8BC6            mov     eax, esi  
00A888D8    FFD2            call    edx  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
00A888D8 处 call    edx 返回后有 4 种情况：  
0：  call 变形  
1：  jmp 变形  
2：  jxx 条件跳转变形  
3：  cmp    x , y  
  jxx     z  变形      
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
下面分别介绍：  
0 情况：  
对象 004013AA   E8 160B0000   call    00401EC5 这样的用户调用变形  
  
00A888DA    2C 02           sub     al, 2  
00A888DC    72 12           jb      short 00A888F0  
00A888F0    8B45 F8         mov     eax, \[ebp-8\]  
00A888F3    8B50 68         mov     edx, \[eax+68\]  
00A888F6    8BC2            mov     eax, edx  
00A888F8    03C7            add     eax, edi  
00A888FA    83F8 FF         cmp     eax, -1  
00A888FD    75 10           jnz     short 00A8890F  
//////////  
00A888FA  cmp     eax, -1 处：  
若 eax  ==  -1：没有对 call（call    00401EC5）里的 call 做进一步处理，  
该 call    00CD0000 返回后到达 call    00401EC5 里的第一条指令。  
若 eax  ！=  -1：该 call 里面还有处理了的 call，该 call    00CD0000 返回后到达 call    00401EC5 里的第一条指令，不过第一条指令是在壳里。  
1 情况：  
      Jmp 变形很简单，到达：  
      00A8890F    8B55 F8         mov     edx, \[ebp-8\]  
00A88912    0342 18         add     eax, \[edx+18\]  
00A88915    E9 B5000000     jmp     00A889CF  
在 00A88912 处 eax 可得跳转地址  
2 情况：  
      Jxx 变形  
  
      00A88925    8B5482 40       mov     edx, \[edx+eax\*4+40\]  
00A88929    8BC6            mov     eax, esi  
00A8892B    FFD2            call    edx  
00A8892D    8BD8            mov     ebx, eax  
00A8892F    8B4D 10         mov     ecx, \[ebp+10\]  
00A88932    8BD3            mov     edx, ebx  
00A88934    8B45 F8         mov     eax, \[ebp-8\]  
00A88937    E8 D4FBFFFF     call    00A88510  
00A8893C    84C0            test    al, al  
00A8893E    74 17           je      short 00A88957  
  
到达 00A8892B 处 call    edx 返回后是跳转类型：  
eax = 0,1,2,3,4,5….分别对应机器码 70，71，72，73…..  
在 00A88937   call    00A88510 进行比较  
00A8893E 处如果不跳的话可来到：  
00A88949    0345 F4         add     eax, \[ebp-C\]  
00A8894C    8B55 F8         mov     edx, \[ebp-8\]  
00A8894F    0342 68         add     eax, \[edx+68\]  
00A88952    EB 7B           jmp     short 00A889CF  
00A8894F 处 eax 就是原 jxx 跳后的地址。  
跳的话可来到：  
00A88957    8B45 F8         mov     eax, \[ebp-8\]  
00A8895A    8B40 18         mov     eax, \[eax+18\]  
00A8895D    03C7            add     eax, edi  
00A8895F    8B55 F8         mov     edx, \[ebp-8\]  
00A88962    0342 68         add     eax, \[edx+68\]  
00A88965    EB 68           jmp     short 00A889CF  
00A88962 处 eax 就是原 jxx 不跳后的地址。  
3 情况：  
      cmp    x , y  
      jxx     z  变形  
  
      00A888DA    2C 02           sub     al, 2  
00A888DC    72 12           jb      short 00A888F0  
00A888DE    74 3D           je      short 00A8891D  
00A888E0    FEC8            dec     al  
00A888E2    0F84 82000000   je      00A8896A  
00A8896A    8BCE            mov     ecx, esi  
00A8896C    8B55 0C         mov     edx, \[ebp+C\]  
00A8896F    8B45 F8         mov     eax, \[ebp-8\]  
00A88972    E8 D5FDFFFF     call    00A8874C    //F7  
{  
  00A8874C    53              push    ebx  
00A8874D    56              push    esi  
00A8874E    57              push    edi  
……F8  
00A88768    33C0            xor     eax, eax  
00A8876A    8A46 07         mov     al, \[esi+7\]  
00A8876D    8B5483 40       mov     edx, \[ebx+eax\*4+40\]  
00A88771    8BC7            mov     eax, edi  
00A88773    FFD2            call    edx  
如果 x 为内存操作数（\[00401235\]），该 call 求出 00401235，否则为 0  
00A88775    894424 04       mov     \[esp+4\], eax  
00A88779    33C0            xor     eax, eax  
00A8877B    8A46 05         mov     al, \[esi+5\]  
00A8877E    8B5483 40       mov     edx, \[ebx+eax\*4+40\]  
00A88782    8BC7            mov     eax, edi  
00A88784    FFD2            call    edx  
如果 x 为寄存器，该 call 求出哪个寄存器（EXX），否则为大于 8 的值  
(0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi)  
00A88786    8BD0            mov     edx, eax  
00A88788    80EA 08         sub     dl, 8  
00A8878B    0F92C2          setb    dl  
00A8878E    80FA 01         cmp     dl, 1  
00A88791    75 11           jnz     short 00A887A4  
若 x 不是寄存器跳 00A887A4(该地址接下来处理 y)  
00A88793    8BC8            mov     ecx, eax  
00A88795    8B1424          mov     edx, \[esp\]  
00A88798    8BC3            mov     eax, ebx  
00A8879A    E8 75020000     call    00A88A14           
该 call 把 x  EXX 中值取出  
00A8879F    8BE8            mov     ebp, eax  
00A887A1    EB 01           jmp     short 00A887A4  
00A887A4    33C0            xor     eax, eax  
00A887A6    8A46 08         mov     al, \[esi+8\]  
00A887A9    8B5483 40       mov     edx, \[ebx+eax\*4+40\]  
00A887AD    8BC7            mov     eax, edi  
00A887AF    FFD2            call    edx  
如果 y 为内存操作数（\[00401235\]），该 call 求出 00401235，如果 y 为直接操作数（3D），该 call 求出 3D，否则为 0  
00A887B1    894424 08       mov     \[esp+8\], eax  
00A887B5    33C0            xor     eax, eax  
00A887B7    8A46 06         mov     al, \[esi+6\]  
00A887BA    8B5483 40       mov     edx, \[ebx+eax\*4+40\]  
00A887BE    8BC7            mov     eax, edi  
00A887C0    FFD2            call    edx  
如果 y 为寄存器，该 call 求出哪个寄存器（EXX），否则为大于 8 的值  
00A887C2    8BD0            mov     edx, eax  
00A887C4    80EA 08         sub     dl, 8  
00A887C7    0F92C2          setb    dl  
00A887CA    80FA 01         cmp     dl, 1  
00A887CD    75 13           jnz     short 00A887E2 //  
00A887CF    8BC8            mov     ecx, eax  
00A887D1    8B1424          mov     edx, \[esp\]  
00A887D4    8BC3            mov     eax, ebx  
00A887D6    E8 39020000     call    00A88A14  
该 call 把 y  EXX 中值取出  
00A887DB    894424 0C       mov     \[esp+C\], eax  
00A887DF   /EB 01           jmp     short 00A887E2  
00A887E2    036C24 04       add     ebp, \[esp+4\]  
00A887E2    036C24 04       add     ebp, \[esp+4\]  
00A887E6    8B4424 08       mov     eax, \[esp+8\]  
00A887EA    034424 0C       add     eax, \[esp+C\]  
00A887EE    894424 10       mov     \[esp+10\], eax  
00A887F2    EB 01           jmp     short 00A887F5  
00A887F5    33C0            xor     eax, eax  
00A887F7    8A46 09         mov     al, \[esi+9\]  
00A887FA    8B5483 40       mov     edx, \[ebx+eax\*4+40\]  
00A887FE    8BC7            mov     eax, edi  
00A88800    FFD2            call    edx   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
该 call 很有意思，返回后有 5 种情况：  
若 eax=0：  cmp   dword ptr \[x\], y  
若 eax=1：  cmp   x,\[ y\]  
若 eax=2：  cmp   byte ptr \[x\], y  
若 eax=3：  cmp    x,byte ptr \[ y\]  
若 eax=4：  cmp   x, y  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
……  
00A88862    8B5424 10       mov     edx, \[esp+10\]  
00A88866    8BC5            mov     eax, ebp  
00A88868    E8 CBFEFFFF     call    00A88738  
该 call 把比较 x,y，得到标志位  
00A8886D    83C4 14         add     esp, 14  
00A88870    5D              pop     ebp  
00A88871    5F              pop     edi  
00A88872    5E              pop     esi  
00A88873    5B              pop     ebx  
00A88874    C3              retn  
}  
      00A8897D    33C0            xor     eax, eax  
00A8897F    8A43 04         mov     al, \[ebx+4\]  
00A88982    8B55 F8         mov     edx, \[ebp-8\]  
00A88985    8B5482 40       mov     edx, \[edx+eax\*4+40\]  
00A88989    8BC6            mov     eax, esi  
00A8898B    FFD2            call    edx  
该 call 返回后是跳转类型：  
eax = 0,1,2,3,4,5….分别对应机器码 70，71，72，73…..  
00A8898D    8BD8            mov     ebx, eax  
00A8898F    8B4D 10         mov     ecx, \[ebp+10\]  
00A88992    8BD3            mov     edx, ebx  
00A88994    8B45 F8         mov     eax, \[ebp-8\]  
00A88997    E8 74FBFFFF     call    00A88510  
在 00A88937   call    00A88510 进行比较  
00A8899C    84C0            test    al, al  
00A8899E    74 17           je      short 00A889B7  
00A8893E 处如果不跳的话可来到：  
00A88949    0345 F4         add     eax, \[ebp-C\]  
00A8894C    8B55 F8         mov     edx, \[ebp-8\]  
00A8894F    0342 68         add     eax, \[edx+68\]  
00A88952    EB 7B           jmp     short 00A889CF  
00A8894F 处 eax 就是原 jxx 跳后的地址。  
跳的话可来到：  
00A88957    8B45 F8         mov     eax, \[ebp-8\]  
00A8895A    8B40 18         mov     eax, \[eax+18\]  
00A8895D    03C7            add     eax, edi  
00A8895F    8B55 F8         mov     edx, \[ebp-8\]  
00A88962    0342 68         add     eax, \[edx+68\]  
00A88965    EB 68           jmp     short 00A889CF  
00A88962 处 eax 就是原 jxx 不跳后的地址。  
  }//第二层返回  
}//第一层返回  
  
  
具体修复方法相信大家看了后比我清楚，不多说了。。。。。。  
  
ASProtect.SKE.2.11 到此已基本脱壳成功！  
我的脱壳经历：  
1．  找 OEP  
2．  还原 IAT  
3．  到达 OEP 后修复 Advanced Import protection  
4．  到达 OEP 后修复 stolen code 里的 call 变形  
5．  DUMP  
这其中还有一个要提的是：我脱的那个软件就是个贱！加壳时使用了次数限制，为此花了我两三个小时来解决它。大家遇到时小心。。。。。。  
  
  
Wak  4.21 2006 

 4dd466d0e1e0cc123afbe504da2462ca