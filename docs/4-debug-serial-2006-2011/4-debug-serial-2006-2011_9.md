

# Alive MP4 Converter 1.6.2.2

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-490.htm`](https://www.pediy.com/kssd/pediy08/pediy8-490.htm)

【破文标题】菜鸟 maomaoma 的算法练习破文一  
【破文作者】maomaoma  
【作者邮箱】  
【作者主页】无  
【破解工具】OD、PEiD  
【破解平台】winxp  
【软件名称】Alive MP4 Converter 1.6.2.2  
【软件大小】3311KB  
【原版下载】http://nj.onlinedown.net/soft/52416.htm  
【保护方式】无  
【软件简介】Alive MP4 Converter 是一款专业的 MP4 转换器，它可以将各种主流的视频格式转换为 MP4 (MPEG4).   
【破解声明】我是菜鸟，学写破文，还请大侠多多指教：）  
\------------------------------------------------------------------------  
【破解过程】  
  
1、PEiD 查主程序无壳（暗自高兴：）），Borland Delphi 6.0 - 7.0 编译  
2、OD 载入，根据注册错误提示字符串“invalid registration code! \\n\\nplease enter an available registration code.”下断点  
3、F9 运行，输入用户名：maomaoma，注册码：12345678，OD 断下  
4、具体分析过程及代码注释如下：  
  
第一部分  
/\*486EEC\*/  push    ebp  //OD 断在此处  
/\*486EED\*/  mov     ebp, esp  
/\*486EEF\*/  push    0  
/\*486EF1\*/  push    0  
/\*486EF3\*/  push    0  
/\*486EF5\*/  push    0  
/\*486EF7\*/  push    0  
/\*486EF9\*/  push    ebx  
/\*486EFA\*/  push    esi  
/\*486EFB\*/  mov     \[ebp-8\], ecx  
/\*486EFE\*/  mov     \[ebp-4\], edx  
/\*486F01\*/  mov     esi, eax  
/\*486F03\*/  mov     eax, \[ebp-4\]  
/\*486F06\*/  call    00404A74  
/\*486F0B\*/  mov     eax, \[ebp-8\]  
/\*486F0E\*/  call    00404A74  
/\*486F13\*/  xor     eax, eax  
/\*486F15\*/  push    ebp  
/\*486F16\*/  push    00486FE3  
/\*486F1B\*/  push    dword ptr fs:\[eax\]  
/\*486F1E\*/  mov     fs:\[eax\], esp  
/\*486F21\*/  xor     ebx, ebx  
/\*486F23\*/  xor     edx, edx  
/\*486F25\*/  mov     eax, \[ebp-4\]  
/\*486F28\*/  call    00404BC8  
/\*486F2D\*/  test    eax, eax  
/\*486F2F\*/  jle     short 00486F3C  
/\*486F31\*/  lea     eax, \[ebp-8\]  
/\*486F34\*/  mov     edx, \[ebp-4\]  
/\*486F37\*/  call    0040466C  
/\*486F3C\*/  lea     ecx, \[ebp-C\]  
/\*486F3F\*/  mov     edx, \[ebp-4\]  
/\*486F42\*/  mov     eax, esi  
/\*486F44\*/  call    00487078  //算法 call 跟进  
/\*486F49\*/  mov     edx, \[ebp-C\]  //(initial cpu selection)  
/\*486F4C\*/  mov     eax, \[ebp-8\]  //内存注册机位置  
/\*486F4F\*/  call    004087E8  //真假码比较 call  
/\*486F54\*/  test    eax, eax  
/\*486F56\*/  jnz     short 00486F99  //不等则跳  
/\*486F58\*/  mov     edx, \[ebp-4\]  
/\*486F5B\*/  mov     eax, esi  
/\*486F5D\*/  call    00486A88  
/\*486F62\*/  test    al, al  
/\*486F64\*/  je      short 00486FC8  
/\*486F66\*/  mov     bl, 1  
/\*486F68\*/  push    40  
/\*486F6A\*/  lea     edx, \[ebp-10\]  
/\*486F6D\*/  mov     eax, \[4A8E04\]  
/\*486F72\*/  mov     eax, \[eax\]  
/\*486F74\*/  call    004641C4  
/\*486F79\*/  mov     eax, \[ebp-10\]  
/\*486F7C\*/  call    00404A84  
/\*486F81\*/  push    eax  
/\*486F82\*/  push    00486FF4  //registered successfully, thanks for your registration.  
/\*486F87\*/  mov     eax, \[4A8E04\]  
/\*486F8C\*/  mov     eax, \[eax\]  
/\*486F8E\*/  mov     eax, \[eax+30\]  
/\*486F91\*/  push    eax  
/\*486F92\*/  call    <jmp.&user32.MessageBoxA>  
/\*486F97\*/  jmp     short 00486FC8  
/\*486F99\*/  push    10  
/\*486F9B\*/  lea     edx, \[ebp-14\]  
/\*486F9E\*/  mov     eax, \[4A8E04\]  
/\*486FA3\*/  mov     eax, \[eax\]  
/\*486FA5\*/  call    004641C4  
/\*486FAA\*/  mov     eax, \[ebp-14\]  
/\*486FAD\*/  call    00404A84  
/\*486FB2\*/  push    eax  
/\*486FB3\*/  push    0048702C  //invalid registration code! \\n\\nplease enter an available registration code.  
/\*486FB8\*/  mov     eax, \[4A8E04\]  
/\*486FBD\*/  mov     eax, \[eax\]  
/\*486FBF\*/  mov     eax, \[eax+30\]  
/\*486FC2\*/  push    eax  
/\*486FC3\*/  call    <jmp.&user32.MessageBoxA>  
/\*486FC8\*/  xor     eax, eax  
/\*486FCA\*/  pop     edx  
/\*486FCB\*/  pop     ecx  
/\*486FCC\*/  pop     ecx  
/\*486FCD\*/  mov     fs:\[eax\], edx  
/\*486FD0\*/  push    00486FEA  
/\*486FD5\*/  lea     eax, \[ebp-14\]  
/\*486FD8\*/  mov     edx, 5  
/\*486FDD\*/  call    004045F8  
/\*486FE2\*/  retn  
  
第二部分  
跟进/\*486F44\*/  call    00487078  
  
/\*487078\*/  push    ebp  
/\*487079\*/  mov     ebp, esp  
/\*48707B\*/  push    0  
/\*48707D\*/  push    0  
/\*48707F\*/  push    0  
/\*487081\*/  push    0  
/\*487083\*/  push    0  
/\*487085\*/  push    0  
/\*487087\*/  push    0  
/\*487089\*/  push    0  
/\*48708B\*/  push    ebx  
/\*48708C\*/  push    esi  
/\*48708D\*/  push    edi  
/\*48708E\*/  mov     ebx, ecx  
/\*487090\*/  mov     \[ebp-4\], edx  
/\*487093\*/  mov     edi, eax  
/\*487095\*/  mov     eax, \[ebp-4\]  
/\*487098\*/  call    00404A74  
/\*48709D\*/  xor     eax, eax  
/\*48709F\*/  push    ebp  
/\*4870A0\*/  push    004871D3  
/\*4870A5\*/  push    dword ptr fs:\[eax\]  
/\*4870A8\*/  mov     fs:\[eax\], esp  
/\*4870AB\*/  lea     eax, \[ebp-4\]  //用户名入堆栈  
/\*4870AE\*/  mov     edx, 004871EC  //gg9e1x!co  
/\*4870B3\*/  call    00404894  //用户名跟固定字符串相连，记着 N  
/\*4870B8\*/  mov     eax, \[ebp-4\]  
/\*4870BB\*/  call    0040488C  //取得相连后字符串位数  
/\*4870C0\*/  mov     esi, eax  //字符串位数入 esi  
/\*4870C2\*/  sar     esi, 1  //esi 右移 1 位  
/\*4870C4\*/  jns     short 004870C9  
/\*4870C6\*/  adc     esi, 0  
/\*4870C9\*/  lea     eax, \[ebp-10\]  
/\*4870CC\*/  push    eax  
/\*4870CD\*/  mov     ecx, esi  //esi 值赋给 ecx  
/\*4870CF\*/  mov     edx, 1  
/\*4870D4\*/  mov     eax, \[ebp-4\]  
/\*4870D7\*/  call    00404AE4  //取相连后字符串前 ecx 位构成变换后用户名第一部分,记着 A  
/\*4870DC\*/  mov     eax, \[ebp-10\]  
/\*4870DF\*/  push    eax  
/\*4870E0\*/  lea     eax, \[ebp-14\]  
/\*4870E3\*/  push    eax  
/\*4870E4\*/  mov     eax, \[ebp-4\]  
/\*4870E7\*/  call    0040488C  //取 N 位数  
/\*4870EC\*/  mov     ecx, eax  
/\*4870EE\*/  lea     edx, \[esi+1\]  //edx=esi+1  
/\*4870F1\*/  mov     eax, \[ebp-4\]  
/\*4870F4\*/  call    00404AE4  //取余下的字符构成变换后用户名第二部分，记着 B  
/\*4870F9\*/  mov     edx, \[ebp-14\]  
/\*4870FC\*/  lea     eax, \[ebp-4\]  
/\*4870FF\*/  pop     ecx  
/\*487100\*/  call    004048D8  
/\*487105\*/  lea     eax, \[ebp-8\]  
/\*487108\*/  push    eax  
/\*487109\*/  mov     ecx, 0A  
/\*48710E\*/  mov     edx, 1  
/\*487113\*/  mov     eax, \[ebp-4\]  //A，B 位置置换，记着 C  
/\*487116\*/  call    00404AE4  //取前十位字符，记着 D  
/\*48711B\*/  lea     eax, \[ebp-C\]  
/\*48711E\*/  push    eax  
/\*48711F\*/  mov     eax, \[ebp-4\]  
/\*487122\*/  call    0040488C  
/\*487127\*/  mov     ecx, eax  
/\*487129\*/  mov     edx, 6  //edx=6  
/\*48712E\*/  mov     eax, \[ebp-4\]  //eax=C  
/\*487131\*/  call    00404AE4  //取第六位到最末字符，记着 E  
/\*487136\*/  cmp     dword ptr \[ebp-C\], 0  
/\*48713A\*/  jnz     short 0048714C  
/\*48713C\*/  lea     eax, \[ebp-C\]  
/\*48713F\*/  mov     edx, 004871EC  //gg9e1x!co  
/\*487144\*/  mov     ecx, \[ebp-8\]  
/\*487147\*/  call    004048D8  
/\*48714C\*/  push    ebx  
/\*48714D\*/  mov     ecx, \[ebp-C\]  //ecx=E  
/\*487150\*/  mov     edx, \[ebp-8\]  //edx=D  
/\*487153\*/  mov     eax, edi  
/\*487155\*/  call    00486938  //跟进  
/\*48715A\*/  lea     eax, \[ebp-18\]  
/\*48715D\*/  push    eax  
/\*48715E\*/  mov     eax, \[ebx\]  
/\*487160\*/  mov     ecx, 5  
/\*487165\*/  mov     edx, 1  
/\*48716A\*/  call    00404AE4  //取 N2 前五位（注册码第一部分）  
/\*48716F\*/  push    dword ptr \[ebp-18\]  
/\*487172\*/  push    00487200  //-  
/\*487177\*/  lea     eax, \[ebp-1C\]  
/\*48717A\*/  push    eax  
/\*48717B\*/  mov     eax, \[ebx\]  
/\*48717D\*/  mov     ecx, 5  
/\*487182\*/  mov     edx, 6  
/\*487187\*/  call    00404AE4  //取 N2 六到十位（注册码第二部分）  
/\*48718C\*/  push    dword ptr \[ebp-1C\]  
/\*48718F\*/  push    00487200  //-  
/\*487194\*/  lea     eax, \[ebp-20\]  
/\*487197\*/  push    eax  
/\*487198\*/  mov     eax, \[ebx\]  
/\*48719A\*/  mov     ecx, 5  
/\*48719F\*/  mov     edx, 0B  
/\*4871A4\*/  call    00404AE4  //取 N2 十一到十五位（注册码第二部分）  
/\*4871A9\*/  push    dword ptr \[ebp-20\]  
/\*4871AC\*/  mov     eax, ebx  
/\*4871AE\*/  mov     edx, 5  
/\*4871B3\*/  call    0040494C  //注册码三部分相连，得到最终注册码  
/\*4871B8\*/  xor     eax, eax  
/\*4871BA\*/  pop     edx  
/\*4871BB\*/  pop     ecx  
/\*4871BC\*/  pop     ecx  
/\*4871BD\*/  mov     fs:\[eax\], edx  
/\*4871C0\*/  push    004871DA  
/\*4871C5\*/  lea     eax, \[ebp-20\]  
/\*4871C8\*/  mov     edx, 8  
/\*4871CD\*/  call    004045F8  
/\*4871D2\*/  retn  
/\*4871D3\*/  jmp     00403FFC  
/\*4871D8\*/  jmp     short 004871C5  
/\*4871DA\*/  pop     edi  
/\*4871DB\*/  pop     esi  
/\*4871DC\*/  pop     ebx  
/\*4871DD\*/  mov     esp, ebp  
/\*4871DF\*/  pop     ebp  
/\*4871E0\*/  retn  
  
第三部分  
跟进/\*487155\*/  call    00486938  
  
/\*486938\*/  push    ebp  
/\*486939\*/  mov     ebp, esp  
/\*48693B\*/  add     esp, -20  
/\*48693E\*/  push    ebx  
/\*48693F\*/  push    esi  
/\*486940\*/  push    edi  
/\*486941\*/  xor     ebx, ebx  
/\*486943\*/  mov     \[ebp-20\], ebx  
/\*486946\*/  mov     \[ebp-10\], ebx  
/\*486949\*/  mov     \[ebp-8\], ecx  
/\*48694C\*/  mov     \[ebp-4\], edx  
/\*48694F\*/  mov     eax, \[ebp-4\]  
/\*486952\*/  call    00404A74  
/\*486957\*/  mov     eax, \[ebp-8\]  
/\*48695A\*/  call    00404A74  
/\*48695F\*/  xor     eax, eax  
/\*486961\*/  push    ebp  
/\*486962\*/  push    00486A54  
/\*486967\*/  push    dword ptr fs:\[eax\]  
/\*48696A\*/  mov     fs:\[eax\], esp  
/\*48696D\*/  mov     eax, \[ebp-8\]  
/\*486970\*/  call    0040488C  //取 E 位数  
/\*486975\*/  mov     \[ebp-C\], eax  
/\*486978\*/  cmp     dword ptr \[ebp-C\], 0  
/\*48697C\*/  jnz     short 0048698B  
/\*48697E\*/  lea     eax, \[ebp-8\]  
/\*486981\*/  mov     edx, 00486A6C  //think space  
/\*486986\*/  call    0040466C  
/\*48698B\*/  xor     esi, esi  
/\*48698D\*/  mov     ebx, 100  //ebx 储存 D、E 变换后注册码（N1），初始化为 100  
/\*486992\*/  lea     eax, \[ebp-10\]  
/\*486995\*/  push    eax  
/\*486996\*/  mov     dword ptr \[ebp-1C\], 100  
/\*48699D\*/  mov     byte ptr \[ebp-18\], 0  
/\*4869A1\*/  lea     edx, \[ebp-1C\]  
/\*4869A4\*/  xor     ecx, ecx  
/\*4869A6\*/  mov     eax, 00486A80  //%1.2x  
/\*4869AB\*/  call    004099D0  //格式化 ebx 值  
/\*4869B0\*/  mov     eax, \[ebp-4\]  
/\*4869B3\*/  call    0040488C  //取 D 位数  
/\*4869B8\*/  mov     edi, eax  
/\*4869BA\*/  test    edi, edi  
/\*4869BC\*/  jle     short 00486A1E  
/\*4869BE\*/  mov     dword ptr \[ebp-14\], 1  
/\*4869C5\*/  mov     eax, \[ebp-4\]  //D、E 转为 N1 核心算法部分  
/\*4869C8\*/  mov     edx, \[ebp-14\]  
/\*4869CB\*/  movzx   eax, byte ptr \[eax+edx-1\]  //D 依次取字符 ASCII 值  
/\*4869D0\*/  add     eax, ebx  //加 m（第一次加常数 100）  
/\*4869D2\*/  mov     ecx, 0FF  //ecx=0FF  
/\*4869D7\*/  cdq  
/\*4869D8\*/  idiv    ecx  //除 ecx  
/\*4869DA\*/  mov     ebx, edx  //计算结果保留于 ebx  
/\*4869DC\*/  cmp     esi, \[ebp-C\]  
/\*4869DF\*/  jge     short 004869E4  
/\*4869E1\*/  inc     esi  //循环计数  
/\*4869E2\*/  jmp     short 004869E9  
/\*4869E4\*/  mov     esi, 1  
/\*4869E9\*/  mov     eax, \[ebp-8\]  
/\*4869EC\*/  movzx   eax, byte ptr \[eax+esi-1\]  //E 依次取字符 ASCII 值  
/\*4869F1\*/  xor     ebx, eax  //异或第一部分变换结果  
/\*4869F3\*/  lea     eax, \[ebp-20\]  
/\*4869F6\*/  push    eax  
/\*4869F7\*/  mov     \[ebp-1C\], ebx  //ebx 值保留堆栈中  
/\*4869FA\*/  mov     byte ptr \[ebp-18\], 0  
/\*4869FE\*/  lea     edx, \[ebp-1C\]  
/\*486A01\*/  xor     ecx, ecx  
/\*486A03\*/  mov     eax, 00486A80  //%1.2x  
/\*486A08\*/  call    004099D0  //格式化 ebx 值，记着 m  
/\*486A0D\*/  mov     edx, \[ebp-20\]  
/\*486A10\*/  lea     eax, \[ebp-10\]  
/\*486A13\*/  call    00404894  //初始值 100 及每次 m 相连  
/\*486A18\*/  inc     dword ptr \[ebp-14\]  
/\*486A1B\*/  dec     edi  
/\*486A1C\*/  jnz     short 004869C5  
/\*486A1E\*/  mov     eax, \[ebp+8\]  //最终结果，记着 N2  
/\*486A21\*/  mov     edx, \[ebp-10\]  
/\*486A24\*/  call    00404628  //结果入堆栈  
/\*486A29\*/  xor     eax, eax  
/\*486A2B\*/  pop     edx  
/\*486A2C\*/  pop     ecx  
/\*486A2D\*/  pop     ecx  
/\*486A2E\*/  mov     fs:\[eax\], edx  
/\*486A31\*/  push    00486A5B  
/\*486A36\*/  lea     eax, \[ebp-20\]  
/\*486A39\*/  call    004045D4  
/\*486A3E\*/  lea     eax, \[ebp-10\]  
/\*486A41\*/  call    004045D4  
/\*486A46\*/  lea     eax, \[ebp-8\]  
/\*486A49\*/  mov     edx, 2  
/\*486A4E\*/  call    004045F8  
/\*486A53\*/  retn  
  
\------------------------------------------------------------------------  
【破解总结】  
  
1、用户名与固定字符串组合并作变换  
2、由变换后用户名计算出注册码  
3、真假码明码比较：）  
  
可作内存注册机，刚自学编程，算法注册机不会，还要向大侠们学习：）  
  
提供一组可用注册码：  
用户名：maomaoma  
注册码：10010-56CC5-DE35D  
\------------------------------------------------------------------------  
【版权声明】本文系作者原创, 转载请注明作者并保持文章的完整, 谢谢! 

 4dd466d0e1e0cc123afbe504da2462ca