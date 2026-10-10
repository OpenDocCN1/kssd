

# labwindows 8.1 明码寻找补充

> 原文：[`www.pediy.com/kssd/pediy10/76340.html`](https://www.pediy.com/kssd/pediy10/76340.html)

坛子里有一篇精华 LabWindows CVI 8.0 的文章，看完试着调试了一下，经验与大家共享，写给自己看的，有点老，  
1. od 载入 cvi.dll  
2.右键 search for ->All referenced text strings, 右键 search for 输入 lm\_ckout.c,  
 找到两处 lm\_ckout.c,一个是两个连着的，一个 7 个连着的。记住 cvi.dll 是 flexnet 10.1.0 版本的。  
两个连着的是正确的，地址是 1061ef33，7 个连着的地址是 1061f029，这两个连着的字串 lm\_ckout.c 属于大名鼎鼎的 lc\_checkout 函数。lc\_checkout 函数比较短，它调用\_l\_checkout 函数，而那七个字串 lm\_ckout.c 属于\_lc\_check\_key 函数，扯远了。  
  
用 od 载入 cvi.dll，设下断点，bp 1061ef38 或 F2，或直接双击第二栏代码栏就可以下断点。然后退出。  
od 载入 cvi.exe，F9 运行，中间按几次 shift+F9，应该可以断下来，看堆栈  
  
0023debc | 0023e928 | ASCII"CVI\_FDS"  
0023deC0 | 0023e95C | ASCII"8.0100"  
可以知道  
            FEATURE 的版本为 8.01  
            FEATURE 的名字是 CVI\_FDS  
还缺少 vendor name，也就是我们常说的守护进程的程序名。  
 od 载入 cvi.dll  
右键 search for-> constant, 右键 search for 输入 87654321,一般只有两个，形式如下的是正确的。  
  
10617A76  |. E8 F4B00000    CALL <cvi.\_l\_sg>                         ; \\cvi.10622B6F  
10617A7B  |. 83C4 0C        ADD ESP,0C  
10617A7E  |. 81BD 84FDFFFF >CMP DWORD PTR SS:\[EBP-27C\],87654321      ;找到这儿，！！！    
10617A88  |. 74 0C          JE SHORT cvi.10617A96  
10617A8A  |. 81BD 88FDFFFF >CMP DWORD PTR SS:\[EBP-278\],12345678  
10617A94  |. 75 5D          JNZ SHORT cvi.10617AF3  
就是前一句是 add esp，0c  后一句是 je 跳转。同时这个 87654321 也能确定大名鼎鼎\_l\_sg 函数。断在\_l\_sg 入口处就能得到 vendor 名字了。本例是 nilm，不是 nilm.exe。  
87654321 这个数属于函数\_l\_init。这个几个常用的调用关系。最好记住。  
  
a. lc\_init -> l\_init  
b. l\_init -> l\_sg(1)  
  
a. lc\_checkout -> l\_checkout  
b. l\_checkout -> lm\_start\_real(2)  
c. lm\_start\_real -> l\_good\_lic\_key(3)  
d. l\_good\_lic\_key -> l\_sg(2)  
  
  
找到三样东西以后，就是找明码了，定位如下  
右键 search for-> constant, 右键 search for 输入 66D8B337，  
找到后前面有连续五个 mov 指令，其中第一个 mov 指令的第一操作数减 1 就是明码的地址。比如下面的例子。  
10626CCE  |. A2 49749310    MOV BYTE PTR DS:\[10937449\],AL  
10626CD3  |. C605 4F749310 >MOV BYTE PTR DS:\[1093744F\],0  
10626CDA  |. 8A15 4F749310  MOV DL,BYTE PTR DS:\[1093744F\]  
10626CE0  |. 8815 4E749310  MOV BYTE PTR DS:\[1093744E\],DL  
10626CE6  |> C785 78FEFFFF >MOV DWORD PTR SS:\[EBP-188\],8  
10626CF0  |. 817D 18 37B3D8>CMP DWORD PTR SS:\[EBP+18\],66D8B337  
  
断在 10626cf0，然后 od 中命令行  db 10937448 即可。找到第一个 sign 后，修改 license，关闭 od，重新载入，下断，就可以找到第二个 sign 了。  
  
另外 license 的格式如下。  
\-------------------------------------------------------------------------------------------  
SERVER THIS\_HOST ANY 27000  
VENDOR vendor\_name     
PACKAGE package\_name vendor\_name feature\_ver COMPONENTS="feature\_name " SIGN=123456789012  
INCREMENT package\_name vendor\_name feature\_ver permanent uncounted VENDOR\_STRING="hello world" HOSTID=\*\*\*\*\*\*\*\*\*\*\*\* vendor\_info="cracked by hhh" TS\_OK SIGN=878787878787  
\---------------------------------------------------------------------------------------------  
解释如下           
vendor\_name 守护程序名，有时称 daemon，比如 pro/E 就是 ptc\_d，由前所述在\_l\_sg 入口处 edx 内容就是 vendor\_name。而\_l\_sg 由 87654321 及前面的 add esp，0c 确定。  
package\_name 随便确定，但是注意影响 sign 的值。  
feature\_ver 和 feature\_name 由属于大名鼎鼎的 lc\_checkout 函数的 lm\_ckout.c 确定，断下，在堆栈中可以看见。  
permanent 表示无时间限制  
uncounted 表示无用户控制。  
vendor\_info 的内容随便写，不影响 sign，  
hostid=\*\*\*\*\*\*\*\*\*\*\*\*表示 12 位的网卡号，或者  
HOSTID=DISK\_SERIAL\_NUM=\*\*\*\*\*\*\*\*   表示 8 位的硬盘号，两者都可以用，当然影响 sign。  
VENDOR\_STRING="hello world"  随便写，但是影响 sign，有无引号无所谓，有无 VENDOR\_STRING 也无所谓。  
  
另外我发现有 seed1 和 seed2 算出的 sign 和直接明码找出的 sign 是不同的，但是都可以通过，也就是全部一样，就是 sign 不一样，但是都可以。  
用 lmcryptgui.exe 输入不同版本对 sign 都没有影响。  
  
  
  
直接找 seed 的方法是：  
正如胖子所说，在 xor al，dl 或者 xor al，bl 指令处，dl 或 bl 内容（1 个字节）就是 seed，这条指令会执行 8 次，就会获得 seed1 和 seed2。比如正确的 seed1 和 seed2 如下：  
seed1   a81b372e      
seed2   4d394f25  
那么在 dl 或 bl 内容将依次出现 2e371ba8  254f394d。就是先出现 seed1，后 seed2，先低字节，后高字节。  
至于定位，xor al，dl 属于\_l\_string\_key 函数，与之类似函数还有\_l\_string\_key1 ~ \_l\_string\_key4 共四个函数。  
这条指令大约在\_l\_string\_key 函数偏移量 0e74 处，也就是约 3700 字节处，或 1051 行处。  
这是对应 flexnet 10.1.0 版本。  
flexlm v8~9  xor al，bl  
flexnet v10.1 xor al，dl  
flexnet v11.1 xor cl, dl  
  
  
或者特征码  
9983E20303C2C1F8028B4D148B，好像通用，不过会有 5 个地址，和常数 66D8B337，前面连续五个 mov 指令联合判定吧。  
\_l\_ckout\_string\_key+103D call    \_atox           // 通过\_atox  确定  
 或者常数 66D8B337，前面连续五个 mov 指令。     
 或者常数 3D4DA1D6 是随机数，有的程序没有改，也可以用来定位。  
  
\_l\_string\_key 既是找 seed 的函数，也是找明码的函数。内容多多。常读常新。  
找明码的好处就是简单，比较 cool，缺点很明显，指对 FEATURE 很少的软件适用，如果像 ansys 那样两百多个 feature，非疯掉不可。  
  
  
直接获得两个 seed 的方法：  
搜索  
250000100085C00F84  
将 84 改为 85，即  
250000100085C00F85  
会出现两个，第一个是的。这个特征码在直接明码或 xor al，dl 的前面，都属于\_l\_string\_key 函数。    
然后在其后的第一个 call 前断下，eax 的内容就是 seed1，再次断下，就是 seed2。  
另外非常奇怪的是在 labwindows8.01 的 cvi.dll 里还发现了 seed1 和 seed2 的明码，简直是搞笑。  
ni 的另一个产品 labview8.2 的程序 labview.exe 也是发现了 seed1 和 seed2 的明码，并且和 labwindosw 的一样，无语中。。。。。  
  
250000100085C00F84           flexlm 10.1.0  
  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*以下代码出现 seed1 和 seed2 的明码\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
10600990  /$  83EC 14            sub     esp, 14  
10600993  |.  53                 push    ebx  
10600994  |.  8BD9               mov     ebx, ecx  
10600996  |.  56                 push    esi  
10600997  |.  57                 push    edi  
10600998  |.  8D7B 08            lea     edi, dword ptr \[ebx+8\]  
1060099B  |.  B9 9F000000        mov     ecx, 9F  
106009A0  |.  BE 40E67C10        mov     esi, 107CE640  
106009A5  |.  F3:A5              rep     movs dword ptr es:\[edi\], dword p>  
106009A7  |.  8B0D 802B9310      mov     ecx, dword ptr \[10932B80\]  
106009AD  |.  85C9               test    ecx, ecx  
106009AF  |.  895C24 1C          mov     dword ptr \[esp+1C\], ebx  
106009B3  |.  C743 04 00000000   mov     dword ptr \[ebx+4\], 0  
106009BA  |.  C703 78E17210      mov     dword ptr \[ebx\], 1072E178  
106009C0  |.  C743 0C 2E371BA8   mov     dword ptr \[ebx+C\], A81B372E         ;seed1!!!!  
106009C7  |.  C743 10 254F394D   mov     dword ptr \[ebx+10\], 4D394F25       ;seed2!!!! 竟然直接出现代码中，    
106009CE  |.  0F8C 97000000      jl      10600A6B  
106009D4  |.  8D71 01            lea     esi, dword ptr \[ecx+1\]  
106009D7  |.  55                 push    ebp  
106009D8  |.  B8 842B9310        mov     eax, 10932B84  
106009DD  |.  BA B82B9310        mov     edx, 10932BB8  
106009E2  |.  81C3 C8000000      add     ebx, 0C8  
106009E8  |.  897424 18          mov     dword ptr \[esp+18\], esi  
106009EC  |.  EB 04              jmp     short 106009F2  
106009EE  |>  8B4C24 1C          /mov     ecx, dword ptr \[esp+1C\]  
106009F2  |>  8BF8                mov     edi, eax  
106009F4  |.  8B2F               |mov     ebp, dword ptr \[edi\]  
106009F6  |.  8DB3 7CFFFFFF      |lea     esi, dword ptr \[ebx-84\]  
106009FC  |.  892E               |mov     dword ptr \[esi\], ebp  
106009FE  |.  8B6F 04            |mov     ebp, dword ptr \[edi+4\]  
10600A01  |.  896E 04            |mov     dword ptr \[esi+4\], ebp  
10600A04  |.  8B7F 08            |mov     edi, dword ptr \[edi+8\]  
10600A07  |.  897E 08            |mov     dword ptr \[esi+8\], edi  
10600A0A  |.  894B 08            |mov     dword ptr \[ebx+8\], ecx  
10600A0D  |.  49                 |dec     ecx  
10600A0E  |.  8D7B 88            |lea     edi, dword ptr \[ebx-78\]  
10600A11  |.  C703 32946110      |mov     dword ptr \[ebx\], 10619432  
10600A17  |.  894C24 1C          |mov     dword ptr \[esp+1C\], ecx  
10600A1B  |.  897C24 10          |mov     dword ptr \[esp+10\], edi  
10600A1F  |.  C74424 14 03000000 |mov     dword ptr \[esp+14\], 3  
10600A27  |>  8B08               |/mov     ecx, dword ptr \[eax\]  
10600A29  |.  8BE9               ||mov     ebp, ecx  
10600A2B  |.  C1E9 02            ||shr     ecx, 2  
10600A2E  |.  8BF2               ||mov     esi, edx  
10600A30  |.  F3:A5              ||rep     movs dword ptr es:\[edi\], dword>  
10600A32  |.  8BCD               ||mov     ecx, ebp  
10600A34  |.  83E1 03            ||and     ecx, 3  
10600A37  |.  F3:A4              ||rep     movs byte ptr es:\[edi\], byte p>  
10600A39  |.  8B7C24 10          ||mov     edi, dword ptr \[esp+10\]  
10600A3D  |.  8B4C24 14          ||mov     ecx, dword ptr \[esp+14\]  
10600A41  |.  83C7 28            ||add     edi, 28  
10600A44  |.  83C0 04            ||add     eax, 4  
10600A47  |.  83C2 28            ||add     edx, 28  
10600A4A  |.  49                 ||dec     ecx  
10600A4B  |.  897C24 10          ||mov     dword ptr \[esp+10\], edi  
10600A4F  |.  894C24 14          ||mov     dword ptr \[esp+14\], ecx  
10600A53  |.^ 75 D2              |\\jnz     short 10600A27  
10600A55  |.  8B4C24 18          |mov     ecx, dword ptr \[esp+18\]  
10600A59  |.  81C3 90000000      |add     ebx, 90  
10600A5F  |.  49                 |dec     ecx  
10600A60  |.  894C24 18          |mov     dword ptr \[esp+18\], ecx  
10600A64  |.^ 75 88              \\jnz     short 106009EE  
10600A66  |.  8B5C24 20          mov     ebx, dword ptr \[esp+20\]  
10600A6A  |.  5D                 pop     ebp  
10600A6B  |>  8D43 04            lea     eax, dword ptr \[ebx+4\]  
10600A6E  |.  50                 push    eax                              ; /Arg4  
10600A6F  |.  8D43 08            lea     eax, dword ptr \[ebx+8\]           ; |  
10600A72  |.  50                 push    eax                              ; |Arg3  
10600A73  |.  68 1CE07210        push    1072E01C                         ; |Arg2 = 1072E01C ASCII "nilm"  
10600A78  |.  6A 00              push    0                                ; |Arg1 = 00000000  
10600A7A  |.  E8 C0640100        call    10616F3F                         ; \\cvi.10616F3F  
10600A7F  |.  83C4 10            add     esp, 10  
10600A82  |.  5F                 pop     edi  
10600A83  |.  5E                 pop     esi  
10600A84  |.  8BC3               mov     eax, ebx  
10600A86  |.  5B                 pop     ebx  
10600A87  |.  83C4 14            add     esp, 14  
10600A8A  \\.  C3                 retn  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*这段代码属于 cvi.dll，不是 flexlm 的函数，不具有通用性，\*\*\*\*\*\*\*\*\*\*\*\*  
  
  
  
  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
106267A1  |> \\83BD 7CFEFFFF 00   |cmp     dword ptr \[ebp-184\], 0  
106267A8  |.  0F85 50040000      |jnz     10626BFE  
106267AE  |.  83BD 90FEFFFF 00   |cmp     dword ptr \[ebp-170\], 0  
106267B5  |.  0F85 47030000      |jnz     10626B02  
106267BB  |.  837D FC 00         |cmp     dword ptr \[ebp-4\], 0  
106267BF  |.  0F85 3D030000      |jnz     10626B02  
106267C5  |.  8B55 08            |mov     edx, dword ptr \[ebp+8\]  
106267C8  |.  8B82 50030000      |mov     eax, dword ptr \[edx+350\]  
106267CE  |.  25 00001000        |and     eax, 100000          ;特征码从这儿开始  
106267D3  |.  85C0               |test    eax, eax          ;250000100085C00F84  
106267D5  |.  0F84 27030000      |je      10626B02                                          ;修改 jne  ,0f85  
106267DB  |.  C785 4CFEFFFF 4874>|mov     dword ptr \[ebp-1B4\], 10937448  
106267E5  |.  8B4D F8            |mov     ecx, dword ptr \[ebp-8\]  
106267E8  |.  C1E1 04            |shl     ecx, 4  
106267EB  |.  8B940D B4FEFFFF    |mov     edx, dword ptr \[ebp+ecx-14C\]  
106267F2  |.  8B45 08            |mov     eax, dword ptr \[ebp+8\]  
106267F5  |.  33C9               |xor     ecx, ecx  
106267F7  |.  8A4C10 08          |mov     cl, byte ptr \[eax+edx+8\]  
106267FB  |.  8B55 14            |mov     edx, dword ptr \[ebp+14\]  
106267FE  |.  8B42 04            |mov     eax, dword ptr \[edx+4\]  
10626801  |.  33C1               |xor     eax, ecx  
10626803  |.  8B4D F8            |mov     ecx, dword ptr \[ebp-8\]  
10626806  |.  C1E1 04            |shl     ecx, 4  
10626809  |.  8B940D B8FEFFFF    |mov     edx, dword ptr \[ebp+ecx-148\]  
10626810  |.  8B4D 08            |mov     ecx, dword ptr \[ebp+8\]  
10626813  |.  33DB               |xor     ebx, ebx  
10626815  |.  8A5C11 08          |mov     bl, byte ptr \[ecx+edx+8\]  
10626819  |.  C1E3 08            |shl     ebx, 8  
1062681C  |.  33C3               |xor     eax, ebx  
1062681E  |.  8B55 F8            |mov     edx, dword ptr \[ebp-8\]  
10626821  |.  C1E2 04            |shl     edx, 4  
10626824  |.  8B8C15 BCFEFFFF    |mov     ecx, dword ptr \[ebp+edx-144\]  
1062682B  |.  8B55 08            |mov     edx, dword ptr \[ebp+8\]  
1062682E  |.  33DB               |xor     ebx, ebx  
10626830  |.  8A5C0A 08          |mov     bl, byte ptr \[edx+ecx+8\]  
10626834  |.  C1E3 10            |shl     ebx, 10  
10626837  |.  33C3               |xor     eax, ebx  
10626839  |.  8B4D F8            |mov     ecx, dword ptr \[ebp-8\]  
1062683C  |.  C1E1 04            |shl     ecx, 4  
1062683F  |.  8B940D C0FEFFFF    |mov     edx, dword ptr \[ebp+ecx-140\]  
10626846  |.  8B4D 08            |mov     ecx, dword ptr \[ebp+8\]  
10626849  |.  33DB               |xor     ebx, ebx  
1062684B  |.  8A5C11 08          |mov     bl, byte ptr \[ecx+edx+8\]  
1062684F  |.  C1E3 18            |shl     ebx, 18  
10626852  |.  33C3               |xor     eax, ebx                          ;eax 即为 seed1    
10626854  |.  50                 |push    eax                             ; /Arg1 = A81B372E  
10626855  |.  E8 D0060000                 |call    10626F2A                        ; \\cvi.10626F2A  
1062685A  |.  83C4 04                     |add     esp, 4  
1062685D  |.  8985 48FEFFFF               |mov     dword ptr \[ebp-1B8\], eax  
10626863  |.  8B95 48FEFFFF               |mov     edx, dword ptr \[ebp-1B8\]  
10626869  |.  81E2 FF000000               |and     edx, 0FF  
1062686F  |.  8B85 4CFEFFFF               |mov     eax, dword ptr \[ebp-1B4\]  
10626875  |.  8A08                        |mov     cl, byte ptr \[eax\]  
10626877  |.  32CA                        |xor     cl, dl  
10626879  |.  8B95 4CFEFFFF               |mov     edx, dword ptr \[ebp-1B4\]  
1062687F  |.  880A                        |mov     byte ptr \[edx\], cl  
10626881  |.  8B85 4CFEFFFF               |mov     eax, dword ptr \[ebp-1B4\]  
10626887  |.  83C0 01                     |add     eax, 1  
1062688A  |.  8985 4CFEFFFF               |mov     dword ptr \[ebp-1B4\], eax  
10626890  |.  81BD 48FEFFFF FF000000      |cmp     dword ptr \[ebp-1B8\], 0FF  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
  
附件有一个脚本，直接出 seed，欢迎测试，

上传的附件
| ![](img/0f24970173caf34cf302ee22e88648bc.jpg) || Ollydbg\_flexlm7.2\_annother.txt  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || lmcryptgui.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || nilm.rar\[论坛下载该文件：[`bbs.pediy.com/showthread.php?t=76340`](http://bbs.pediy.com/showthread.php?t=76340%20) \]  
 |

没有想到还有人关注，  
nilm.exe 是 ni 公司所有软件的 daemon，呵呵，这个里有 seed1 和 seed2，  
nilm 已经上传了，仅有 cvi.dll 是无法执行的，最好到 ni 的网站上下一个安装文件，免费的，  
调试 nilm 的方法有两种，  
（1）od 打开，在调试参数里输入 nilm -T 主机名 11.4 -1 -c "c:\\flexlm\\LabVIEW.lic"  
就可以里了，  
（2）将 nilm.exe 的 oep 处的 55 修改成 cc，然后运行 lmgrd -c "c:\\flexlm\\LabVIEW.lic"，  
让异常呼出 od，或者将 nilm.exe 的 oep 处的 558b 修改成 ebfe，然后运行 lmgrd -c "c:\\flexlm\\LabVIEW.lic"，打开 od，attach 之，就停在 oep 处了，改回即可。