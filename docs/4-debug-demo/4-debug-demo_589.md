

# 突破封锁线：第一章--脱壳篇

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-590.htm`](https://www.pediy.com/kssd/pediy08/pediy8-590.htm)

【文章标题】: 突破封锁线：第一章--脱壳篇  
【文章作者】: Austin  
【作者邮箱】: austiny.cn@gmail.com  
【软件名称】: 神州数码网络客户登陆程序  
【软件大小】: 不重要  
【下载地址】: http://www.dcnetworks.com.cn  
【加壳方式】: UltraProtect 1.x  
【保护方式】: 加壳 网络封禁  
【编写语言】: VC6.0  
【使用工具】: PEid OllyDbg LordPE ImportRec  
【操作平台】: Windows2003  
【软件介绍】: 神州数码小区宽带网络登陆客户端，诸多限制  
【作者声明】: 技术交流中。。。  
\--------------------------------------------------------------------------------  
【详细过程】  
  一、（背景）山和云的彼端  
    
  那时候天很蓝，网很宽，资源可以共享，BT 随便可以下。后来有了客户端，一切化为泡影。  
    
  神州数码是一套小区宽带解决方案，使用 802.x 认证等技术，限制了网络的很多使用：  
    
    1. 帐号与机器绑定（换机器不能登陆）  
    2. 限制共享网络连接（发现代理，强制下线）  
    3. 限制 BT 下载（强制下线）  
    4. 无法从路由后连接（拒绝登陆）  
    ...  
    
  诸多不便，一一破之。  
    
  二、（调查）英雄莫问出处  
    
  神州数码（DigitalChina 下简称 DIGI）安装文件夹大概就是一个主 EXE 文件外加一堆 DLL 文件和配置文件。  
  DLL 文件都是系统常用如 MFC42.DLL 和 AMT.DLL 等，故搞定 EXE 一个即可。  
    
   1. 先看看加壳否  
     
   用 PEid（侦壳工具）打开 DIGI.EXE，得到 "UltraProtect 1.x -> RISCO Software Inc."。（看来盔甲穿得挺厚）  
     
   2. 再看仔细些  
    
   用 LordPE（DUMP 和 PE 修改工具）打开，看到程序有 5 个段。  
   出特别指明，文中都使用十六进制  
    
               地址          长度  
   .text       00410000     10000  
   .rdata      00411000      4000  
   .data       00415000    204000  
   .rsrc       00619000      3000  
   .perplex    0061C000     1D000  
    
   前 4 个都很典型（正常编译的程序一般都有这几个段），故不出意外，壳代码都在 perplex 段中。  
    
   3. 战略  
    
   UltraProtect 壳没有脱壳机，只有自己动手脱光它了：  
    
   调试 -> 找 OEP -> dump -> 修复 IAT -> 修复资源 -> 减肥  
    
  三、（行动）骡子还是马，拉出来遛遛先  
     
  用 OllyDbg（调试工具）装载，运行。啪！OllyDbg 退出了。（烈马！）看来壳中有反调试检查。  
    
  难不倒我们：打开 HideOD 插件的隐藏选项，再运行。再啪！OllyDbg 再次退出了。  
  （我看这马挺烈吧）（那是相当烈啊）  
    
  去看雪搜了一下，发现 UltraProtect 有检查加载者并于进程列表比对的习惯，而非常规的检查调试标志。  
  故 HideOD 骗不倒它。  
    
  再来，加载之，停在 0061c000 处：（正是在.perplex 段中）  
    
  0061C000 >  60              pushad  
  0061C001    FC              cld  
  0061C002    4D              dec     ebp  
  0061C003    72 01           jb      short 0061C006  
  0061C005    4E              dec     esi  
  0061C006    50              push    eax  
  0061C007    E8 01000000     call    0061C00D  
  0061C00C    71 58           jno     short 0061C066  
  0061C00E    58              pop     eax  
  0061C00F    0F84 03000000   je      0061C018  
  0061C015    66:13EF         adc     bp, di  
  0061C018    76 03           jbe     short 0061C01D  
  0061C01A    77 01           ja      short 0061C01D  
  0061C01C  - 7E C1           jle     short 0061BFDF  
  0061C01E    DD22            frstor  \[edx\]  
  0061C020    7C 03           jl      short 0061C025  
  0061C022    7D 01           jge     short 0061C025  
  0061C024    9A BA21D624 CBB>call    far BACB:24D621BA  
    
  将断点下至 Process32First 函数（获得进程列表会调用）  
   bp Process32First+2  
    
  F9 运行，断在了 7c85a26e 处：（应该在 Process32FirstA 中）  
    
  7C85A26E >  55              push    ebp  
  7C85A26F    8BEC            mov     ebp, esp  
  7C85A271    81EC 30020000   sub     esp, 230  
  7C85A277    A1 08D1887C     mov     eax, \[7C88D108\]  
  7C85A27C    56              push    esi  
  7C85A27D    8B75 0C         mov     esi, \[ebp+C\]  
  7C85A280    85F6            test    esi, esi  
  7C85A282    8945 FC         mov     \[ebp-4\], eax  
  7C85A285    8B45 08         mov     eax, \[ebp+8\]  
  7C85A288    0F84 9D000000   je      7C85A32B  
  7C85A28E    813E 28010000   cmp     dword ptr \[esi\], 128  
  7C85A294    0F82 91000000   jb      7C85A32B  
  7C85A29A    53              push    ebx  
  7C85A29B    8D8D D0FDFFFF   lea     ecx, \[ebp-230\]  
  7C85A2A1    51              push    ecx  
  7C85A2A2    50              push    eax  
  7C85A2A3    C785 D0FDFFFF 2>mov     dword ptr \[ebp-230\], 22C  
  7C85A2AD    E8 FDFEFFFF     call    Process32FirstW  
    
  Shift+F9 回到用户代码：  
    
  00620B3F > /E9 AD000000     jmp     00620BF1            ;直接跳转到下面  
  00620B44   |60              pushad  
  00620B45   |8DBD E1444000   lea     edi, \[ebp+4044E1\]  
  00620B4B   |B9 26000000     mov     ecx, 26  
  00620B50   |E8 A9EDFFFF     call    0061F8FE  
  00620B55   |AB              stos    dword ptr es:\[edi\]  
  00620B56  ^|E2 F8           loopd   short 00620B50  
  00620B58   |61              popad  
  00620B59   |60              pushad  
  00620B5A   |E8 F2EFFFFF     call    0061FB51  
          .......  
  00620BF1   \\0BC0            or      eax, eax            ;跳转到此，单步往下走  
  00620BF3    0F84 27020000   je      00620E20            ;此处未跳转  
  00620BF9    8B95 45424000   mov     edx, \[ebp+404245\]   ;\[ebp+404245\]的值从 0 开始，依次为各进程 PID  
  00620BFF    3B95 35424000   cmp     edx, \[ebp+404235\]   ;\[ebp+404235\]中为父进程 OllyDbg 的 PID，此句进行比对  
  00620C05    0F84 DC000000   je      00620CE7            ;在比对前将\[ebp+404235\]的值改为 Explorer.exe 的 PID  
  00620C0B    B8 3D424000     mov     eax, 0040423D       ;就可以骗过 DIGI.EXE  
  00620C10    03C5            add     eax, ebp  
  00620C12    50              push    eax  
  00620C13    FFB5 39424000   push    dword ptr \[ebp+404239\]  
  00620C19    50              push    eax  
  00620C1A    8B85 05454000   mov     eax, \[ebp+404505\]  
  00620C20    8038 CC         cmp     byte ptr \[eax\], 0CC  
  00620C23    74 10           je      short 00620C35  
    
  不多说，改之。顺利通过检测。：）  
    
  清除刚才的断点。（重要，除非你愿意一次又一次重打第一关 BOSS）  
    
  四、（继续）万里长征第二步  
    
  通过了第一关，开始进行寻找 OEP 的伟大战役。  
  （注：OEP=Original Entry Point，就是加壳前程序开始的地方，也是我们 dump 的起点）  
    
  常识告诉我们，OEP 应该在.text 段。（当然也有不是的时候，不过大多数的时候还是是的）  
    
  Alt+M 打开内存图，F2 设一个访问断点在.text 段上。（没有用二次断点，因为幸运的是，此时代码段已经解压完毕）  
    
  F9 往下跑，程序停在 0040fb74 处，在.text 段。（简直就是 VC 程序的标准入口，这里应该就是 EOP 没错了）  
    
  0040FB74 > .  68 A82A4100   push    00412AA8  
  0040FB79   .  68 D4FC4000   push    0040FCD4                         ;  jmp 到 msvcrt.\_except\_handler3; SE 处理程序安装  
  0040FB7E   .  64:A1 0000000>mov     eax, fs:\[0\]  
  0040FB84   .  50            push    eax  
  0040FB85   .  64:8925 00000>mov     fs:\[0\], esp  
  0040FB8C   .  83EC 68       sub     esp, 68  
  0040FB8F   .  53            push    ebx  
  0040FB90   .  56            push    esi  
  0040FB91   .  57            push    edi  
  0040FB92   .  8965 E8       mov     \[ebp-18\], esp  
  0040FB95   .  33DB          xor     ebx, ebx  
  0040FB97   .  895D FC       mov     \[ebp-4\], ebx  
  0040FB9A   .  6A 02         push    2  
  0040FB9C   .  FF15 64144100 call    \[411464\]                         ;  msvcrt.\_\_set\_app\_type  
  0040FBA2   .  59            pop     ecx  
  0040FBA3   .  830D 5C8E6100>or      dword ptr \[618E5C\], FFFFFFFF  
  0040FBAA   .  830D 608E6100>or      dword ptr \[618E60\], FFFFFFFF  
  0040FBB1   .  FF15 68144100 call    \[411468\]                         ;  msvcrt.\_\_p\_\_fmode  
  0040FBB7   .  8B0D A06F4100 mov     ecx, \[416FA0\]  
  0040FBBD   .  8908          mov     \[eax\], ecx  
  0040FBBF   .  FF15 6C144100 call    \[41146C\]                         ;  msvcrt.\_\_p\_\_commode  
  0040FBC5   .  8B0D 9C6F4100 mov     ecx, \[416F9C\]  
    
  至此，EOP 已经找到。  
    
   由于可能多次重复，用 OllyMachine 插件完成整个上述过程：  
    
  ;LOAD.TXT  
  ;//////////////////////////////////////////////////////  
  invoke bp, 0x7c85a26e;Process32First+2  
  run  
  runtousercode  
  invoke bp, 0x7c85a26e;Process32First+2  
  run  
  invoke bc, 0x7c85a26e  
  runtousercode  
  invoke writememlong, 0x61f235, 1960, 2  
  invoke bp, 0x0040fb74  
  run  
  ;//////////////////////////////////////////////////////  
    
  五、（出击）衣带渐宽终不悔  
    
  局势很明朗，dump 之。这里用了 OllyDbg 的插件 OllyDump。  
  注意 dump 的时候不要选择 Rebuild Import（术业有专攻，这个留给 ImportRec 来处理），保存为 DIGI\_DUMP.EXE。  
    
  打开 ImportRec（修复 IAT 之王），在它的进程表中找到正在调试的 DIGI。  
  在 OEP 中填入刚刚找到的 0000FB74（偏移量），点击"IAT AutoSearch"，再点击"Get Imports"，得到 Imported Functions 列表。  
    
  表中有两块为解决的项：  
   ? FThunk:00011048 NbFunc:2D (decimal:45) valid:NO        这一项中有 45 个函数不可用  
   ? FThunk:00011504 NbFunc:16 (decimal:22) valid:NO        这一项中有 22 个函数不可用  
    
  注意表中 00011048 为偏移地址，所以实际地址应加上基地址 00400000，为 00411048。  
  在 OllyDbg 的数据窗口中来到 00411048（按地址方式显示）：  
    
  00411048  0061C000  offset DigitalC.<模块入口点>  
  0041104C  0061C00D  DigitalC.0061C00D  
  00411050  0061C01A  DigitalC.0061C01A  
  00411054  0061C027  DigitalC.0061C027  
  00411058  0061C034  DigitalC.0061C034  
  0041105C  0061C041  DigitalC.0061C041  
  00411060  0061C04E  DigitalC.0061C04E  
    
  表中 00411048 指向 0061c000（注意这正是.perplex 段开始的地方），在反汇编窗口中跟踪：  
    
  0061C000 >  68 D6DDEAA8     push    A8EADDD6  
  0061C005    813424 A6AD68D4 xor     dword ptr \[esp\], D468ADA6  
  0061C00C    C3              retn  
  0061C00D    68 2666EDA8     push    A8ED6626  
  0061C012    813424 4AC468D4 xor     dword ptr \[esp\], D468C44A  
  0061C019    C3              retn  
  0061C01A    68 9D6FEDA8     push    A8ED6F9D  
  0061C01F    813424 7ECC68D4 xor     dword ptr \[esp\], D468CC7E  
  0061C026    C3              retn  
  0061C027    68 75D6EAA8     push    A8EAD675  
  0061C02C    813424 42E668D4 xor     dword ptr \[esp\], D468E642  
  0061C033    C3              retn  
    
  注意每隔三句为表中指向的一项。  
    
  如第一项所做的操作相当于 jmp xxxxxxxx (xxxxxxxx = A8EADDD6 xor D468ADA6)，下面的项对应相同。  
  故将算出的 xxxxxxxx 写入 00411048 即为真实的 Imported Function 地址。  
    
  过程繁琐，用 OllyMachine 插件脚本来完成。（两个 invalid 的表一起解决掉）（此脚本来源于看雪论坛）  
    
  ;修复 IAT.TXT  
  ;//////////////////////////////////////////////////////  
  mov reg01,0x411048  
  lp:  
  invoke ReadMemLong,reg01,4;         ---先取出要跳转的指针  
  ;invoke PrintNum,reg00,16;         ----这些是我最初拿来验证数据是否正确的  
  mov reg02,reg00;         --------------存一下，下面还要用，寄存器多就是好  
  inc reg00;         --------------------上面的代码中可以看出来，xor 的前一个数在被指向地址+1 处  
  invoke ReadMemLong,reg00,4;         ---读取  
  ;invoke PrintNum,reg00,16  
  mov reg10,reg00  
  add reg02,8;         ------------------xor 后一个数在+8 处  
  invoke ReadMemLong,reg02,4;         ---读取  
  xor reg00,reg10;         --------------xor，我们要的 api 地址  
  ;invoke PrintNum,reg00,16  
  invoke WriteMemLong,reg01,reg00,4;  ---写入到 IAT 中  
  add reg01,4  
  cmp reg01,0x4110f9;         -----------比较修复是否完成  
  jb lp  
    
  mov reg01,0x411504  
  lp1:  
  invoke ReadMemLong,reg01,4  
  ;invoke PrintNum,reg00,16  
  mov reg02,reg00  
  inc reg00  
  invoke ReadMemLong,reg00,4  
  ;invoke PrintNum,reg00,16  
  mov reg10,reg00  
  add reg02,8  
  invoke ReadMemLong,reg02,4  
  xor reg00,reg10  
  ;invoke PrintNum,reg00,16  
  invoke WriteMemLong,reg01,reg00,4  
  add reg01,4  
  cmp reg01,0x411559  
  jb lp1  
  ;//////////////////////////////////////////////////////  
    
  运行后发现 00411048 处的函数表已经修复，可以正确识别：  
    
  00411048  7C827070  kernel32.SetThreadPriority  
  0041104C  7C85A26C  kernel32.Process32First  
  00411050  7C85A3E3  kernel32.Process32Next  
  00411054  7C823037  kernel32.CreateThread  
  00411058  7C82BC7C  kernel32.MultiByteToWideChar  
  0041105C  7C806165  kernel32.DeleteFileA  
  00411060  7C826919  kernel32.GlobalHandle  
  00411064  7C8270B2  kernel32.SetEvent  
  00411068  7C8266C5  kernel32.GlobalUnlock  
    
  再回到 Import Rec 中，"Get Imports"，形势一片大好，全部 valid。  
    
  将 Add new section 选中，点击"Fix Dump"，选择之前保存的 DIGI\_DUMP.EXE：  
  得到修复了 Import Table 的 DIGI\_DUMP\_.EXE 文件。  
    
  六、（：）脱掉脱掉脱掉  
    
  脱到这里，脑海里已经响起杜德伟的歌声。  
    
  得意的运行 DIGI\_DUMP\_.EXE 文件。（冷水！）却得到一个引用 0x0000000 内存的错误。  
  分析一下，应该是修改了入口地址（EP），壳中有些代码没有运行，所以程序未能正常运行。  
    
  关闭对内存异常的忽略（因为是内存错误），再用 OllyDbg 打开新得到的 DIGI\_DUMP.EXE 文件。  
    
  从入口点 0040fb74 开始单步跟踪：  
    
  0040FC93  |> \\50            push    eax  
  0040FC94  |.  56            push    esi  
  0040FC95  |.  53            push    ebx  
  0040FC96  |.  53            push    ebx                              ; /pModule  
  0040FC97  |.  FF15 CC104100 call    \[<&kernel32.GetModuleHandleA>\]   ; \\GetModuleHandleA  
  0040FC9D  |.  50            push    eax  
  0040FC9E  |.  E8 13010000   call    0040FDB6                         ;单步过此处出现异常  
  0040FCA3  |.  8945 98       mov     \[local.26\], eax  
  0040FCA6  |.  50            push    eax                              ; /status  
  0040FCA7  |.  FF15 C0144100 call    \[<&msvcrt.exit>\]                 ; \\exit  
  0040FCAD  |.  8B45 EC       mov     eax, \[local.5\]  
  0040FCB0  |.  8B08          mov     ecx, \[eax\]  
  0040FCB2  |.  8B09          mov     ecx, \[ecx\]  
    
  程序停在 0061d137：  
    
  0061D131    FF25 C0951500   jmp     \[1595C0\]  
  0061D137 >  FF25 C4951500   jmp     \[1595C4\]       ;程序停在这里  
  0061D13D    FF25 C8951500   jmp     \[1595C8\]  
  0061D143    FF25 CC951500   jmp     \[1595CC\]  
  0061D149    FF25 D0951500   jmp     \[1595D0\]  
  0061D14F    FF25 D4951500   jmp     \[1595D4\]  
  0061D155    FF25 D8951500   jmp     \[1595D8\]  
  0061D15B    FF25 DC951500   jmp     \[1595DC\]  
  0061D161    FF25 E0951500   jmp     \[1595E0\]  
  0061D167    FF25 E4951500   jmp     \[1595E4\]  
    
  这里（0061d137）已经是.perplex 的领域，就是壳的代码段，看在程序运行后还是在调用壳中的代码。  
    
  看一下堆栈：  
    
  0012AEF8   0040FB6D  返回到 digi\_dum.0040FB6D 来自 digi\_dum.0061D137  
  0012AEFC   00000000  
  0012AF00   00000000  
  0012AF04   00000000  
    
  看来就是从 0040fb6d 之前调用了这里，Ctrl+G 跳过去：  
    
  0040FB64  |.  8501          test    \[ecx\], eax  
  0040FB66  |.  8BE1          mov     esp, ecx  
  0040FB68  |.  E8 CAD52000   call    0061D137               ;就是这里调用了壳中代码  
  0040FB6D  |.  50            push    eax  
  0040FB6E  \\.  C3            retn  
  0040FB6F      63            db      63                               ;  CHAR 'c'  
  0040FB70      72            db      72                               ;  CHAR 'r'  
  0040FB71      6A            db      6A                               ;  CHAR 'j'  
  0040FB72      D8            db      D8  
  0040FB73      D4            db      D4  
  0040FB74 >/$  68 A82A4100   push    00412AA8  
  0040FB79  |.  68 D4FC4000   push    <jmp.&msvcrt.\_except\_handler3>   ;  SE 处理程序安装  
  0040FB7E  |.  64:A1 0000000>mov     eax, fs:\[0\]  
    
  果不其然，就是这里调用了壳中的代码。  
    
  分析一下：  
  这是一种典型的壳驻留方式，就是在解压完毕之后，任有部分壳中代码被运行，以防止壳失去控制权（或者说，被脱掉）。  
    
  这里用了一种叫做 Code Splitting 的技术（之后再具体分析）。  
  应该是壳正常调用时，会在 001595c4 写入正常的函数地址以被调用。  
  而现在壳代码没有运行，001595c4 中就无没有正确的地址了。  
    
  问题的关键就是得到 001595c4 中正确的地址。记下 0061d137 这个地方。  
    
  将之前的内存异常忽略选项打开，用 OllyDbg 打开未脱壳的 DIGI 程序，在 0061d137 下断点。  
  用之前的 OllyMachine 脚本将程序跑起来，停在 0040fb7e 处，再跑，停到 0061d137 处：  
    
  0061D137  - FF25 C4951500   jmp     \[1595C4\]  
  0061D13D  - FF25 C8951500   jmp     \[1595C8\]  
    
  在数据窗口（地址模式）看 1595c4 的内容：  
    
  001595BC  0015A56E  
  001595C0  0015A574  
  001595C4  0015A57A   ;这就是将要 jmp 到的地方  
    
  在反汇编窗口跟过去：  
    
  0015A56E    8BCF            mov     ecx, edi  
  0015A570    8957 FC         mov     \[edi-4\], edx  
  0015A573    C3              retn  
    
  这三句就是壳程序在内存中写入的代码，  
  在 0040FB68  |.  E8 CAD52000   call    0061D137 的调用最后就是调用了这三句代码。  
    
  注意到 0040fb68 中的 call 语句占用了 5 个字节，而三句代码中的前两句占用的也是 5 个字节（第三句用于返回）。  
    
  这就是 Code Splitting 的秘密！！！   
  将原程序中的部分语句用壳中的一个 call 来代替，这样脱出来的程序就离不开壳来运行。  
    
  解决方法就是将 0015a56e 开始的 5 个字节代码拷回至 0040fb68 就可以了。  
    
  这么繁琐的工作，让 OllyMachine 来做：  
    
  ;修复代码 1.TXT  
  ;//////////////////////////////////////////////////////  
  mov reg01, 0x00401030  
  lp:  
  add reg01, 5  
  invoke Find, reg01, "e8????2000"  
  cmp reg00, -1  
  je end1  
    
  mov reg01, reg00 //reg01="call 0061????"  
  invoke printbuftodump, reg01  
  ;invoke printbuf, reg01, 5  
    
  ;invoke MsgYn, "Process?"  
  ;or reg00, 0  
  ;je lp  
    
  mov reg02, reg01  
  inc reg02 //reg02="0020????"  
  invoke readmemlong, reg02, 4  
  mov reg03, reg00 //reg03=0020????  
  ;invoke printnum, reg01, 16  
  ;invoke printnum, reg03, 16  
  add reg03, reg01   
  add reg03, 5 //reg03=61????(jmp\[15????\])  
  add reg03, 2 //reg03="15???"  
  invoke printbuftodump, reg03  
    
  invoke readmemlong, reg03, 4  
  mov reg04, reg00 //reg04=15????  
    
  invoke readmemlong, reg04, 4  
  mov reg05, reg00 //reg05=\15????\  
  invoke printbuftodump, reg05  
  ;invoke printbuf, reg05, 5  
    
  ;invoke MsgYn, "Process?"  
  ;or reg00, 0  
  ;je lp  
    
  invoke readmemlong , reg05, 4  
  invoke writememlong, reg01, reg00, 4  
  add reg01, 4  
  add reg05, 4  
  invoke readmemlong, reg05, 1  
  invoke writememlong, reg01, reg00, 1  
  sub reg01, 4  
  invoke printbuftodump, reg01  
    
  ;invoke MsgYn, "Continue?"  
  ;or reg00, 0  
  jne lp  
  end1:  
  ;//////////////////////////////////////////////////////  
    
  ;修复代码 2.TXT  
  ;//////////////////////////////////////////////////////  
  mov reg01, 0x00401030  
  lp:  
  add reg01, 5  
  invoke Find, reg01, "e8????2100"  
  cmp reg00, -1  
  je end1  
    
  mov reg01, reg00 //reg01="call 0061????"  
  invoke printbuftodump, reg01  
  ;invoke printbuf, reg01, 5  
    
  ;invoke MsgYn, "Process?"  
  ;or reg00, 0  
  ;je lp  
    
  mov reg02, reg01  
  inc reg02 //reg02="0020????"  
  invoke readmemlong, reg02, 4  
  mov reg03, reg00 //reg03=0020????  
  ;invoke printnum, reg01, 16  
  ;invoke printnum, reg03, 16  
  add reg03, reg01   
  add reg03, 5 //reg03=61????(jmp\[15????\])  
  add reg03, 2 //reg03="15???"  
  invoke printbuftodump, reg03  
    
  invoke readmemlong, reg03, 4  
  mov reg04, reg00 //reg04=15????  
    
  invoke readmemlong, reg04, 4  
  mov reg05, reg00 //reg05=\15????\  
  invoke printbuftodump, reg05  
  ;invoke printbuf, reg05, 5  
    
  ;invoke MsgYn, "Process?"  
  ;or reg00, 0  
  ;je lp  
    
  invoke readmemlong , reg05, 4  
  invoke writememlong, reg01, reg00, 4  
  add reg01, 4  
  add reg05, 4  
  invoke readmemlong, reg05, 1  
  invoke writememlong, reg01, reg00, 1  
  sub reg01, 4  
  invoke printbuftodump, reg01  
    
  ;invoke MsgYn, "Continue?"  
  ;or reg00, 0  
  jne lp  
  end1:  
  ;//////////////////////////////////////////////////////  
    
  运行上述两个脚本，以及之前的修复 IAT 脚本。 Dump 出来，用 ImportRes 修复。（啤酒拿出来，准备庆祝）  
    
  运行一下新的 DIGI\_DUMP\_.EXE 程序 ~~ 程序界面飘然而至 ：）  
    
  七、（狂欢）I am a super dancing queen!  
    
  为了确定已经脱光了，用 OllyDbg 打开之，清除所有断点，对.perplex 段下访问断点（在 Alt+M 中），运行。  
  试验各项功能，并没有被断下来，说明.perplex 中的壳代码再也没有运行。  
    
  脱壳成功！！！  
    
\--------------------------------------------------------------------------------  
【经验总结】  
  至此，神州数码的 UltraProtect 壳已经被彻底的脱下来了。为后续的工作做好了准备。  
    
  脱壳中注意的关键点有：  
   1. 壳会对进程列表进行比对，关键点是 Process32First 函数   .........  LOAD.TXT 脚本  
   2. 壳中有奇怪形式的 IAT 表，注意修复     ..........................  修复 IAT.TXT 脚本  
   3. 壳使用了 Code Splitting 技术，注意还原     .....................  修复代码 1.TXT 修复代码 2.TXT 脚本  
   4. 脚本真是个好东西  
    
  突破封锁线：第一章--脱壳篇 \[完\]  
    
  谢谢您耐心的看到这里，在下一章--资源篇中，将详细讲述如何修复资源和加入自己的资源到程序中。  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 04 月 28 日 14:49:03 

【文章标题】: 突破封锁线：第二章--资源篇  
【文章作者】: Austin  
【作者邮箱】: austiny.cn@gmail.com  
【软件名称】: 神州数码网络客户登陆程序  
【软件大小】: 不重要  
【下载地址】: 自己搜索下载  
【加壳方式】: UltraProtect 1.x  
【保护方式】: 加壳 网络封禁  
【编写语言】: VC6.0  
【使用工具】: resHacker DT\_FixRes ResFixer LordPE  
【操作平台】: Windows2003  
【软件介绍】: 神州数码小区宽带网络登陆客户端，诸多限制  
【作者声明】: 技术交流中。。。  
\--------------------------------------------------------------------------------  
【详细过程】  
  一、（换台）前情回顾  
    
  上一回讲到，神州数码（DIGI）确实已经脱光了，我们来做点什么。。。  
    
  二、（YY 中）。。。  
    
  DIGI 的主界面长得象这个样子：  
   \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
  |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|  
  |                                                |  
  |                                                |  
  |                                                |  
  |                 L  O  G  O                     |  
  |                                                |  
  |                                                |  
  |                                                |         弹出菜单象这个样子：  
  |                                                |                \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
  |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|               |                |  
  |               \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_    |               |  显示连接状态  |  
  |    用户名：  |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|   |               |                |  
  |               \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_    |               |  断开当前连接  |  
  |    密  码：  |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|   |               |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|  
  |                                                |               |                |  
  |           X 保存密码    自动加载  X 自动连接   |               |  属性...       |  
  |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|               |                |  
  |                                                |               |  关于...       |  
  |      --------      --------      --------      |               |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|  
  |     |  连接  |    |  退出  |    |  属性  |     |  
  |      --------      --------      --------      |  
  |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|  
    
  我们的主要任务是在主界面增加一个按钮，弹出菜单中增加一项，以调用我们自己的设置对话框。  
    
  （所以准备把 DIGI 送去韩国，注意，不是泰国）  
    
  整容后的 DIGI 应该长成这个样子：  
   \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
  |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|  
  |                                                |  
  |                                                |  
  |                                                |  
  |                 L  O  G  O                     |  
  |                                                |  
  |                                                |  
  |                                                |         弹出菜单象这个样子：  
  |                                                |                \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
  |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|               |                |  
  |               \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_    |               |  显示连接状态  |  
  |    用户名：  |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|   |               |                |  
  |               \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_    |               |  断开当前连接  |  
  |    密  码：  |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|   |               |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|  
  |                                                |               |                |  
  |           X 保存密码    自动加载  X 自动连接   |               |  高级...       |  
  |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|               |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|  
  |                                                |               |                |  
  |   --------   --------   --------   --------    |               |  属性...       |  
  |  |  连接  | |  高级  | |  退出  | |  属性  |   |               |                |  
  |   --------   --------   --------   --------    |               |  关于...       |  
  |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|               |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|  
    
  注意我们加入的“高级”字样  
    
  三、（热身）左三圈右三圈  
    
  先用 ResFixer（很厉害的资源修复工具，不过我们只用它来验验货而已）打开上一章得到的 DIGI\_DUMP\_.EXE 文件。  
    
  果然，果然，红色的显示部分资源（主要是几个 Icon 和 String 还有 VersionInfo）被放在了.perplex 段，  
  而大多数资源（Dialog 和 Menu，都是我们喜欢的），被放在了.rsrc 段，  
  果然是武林中最邪恶、最阴险的面目全非掌。这样的跨段资源在修复以前，是没法直接修改的。  
    
  哎，我们只好使出失传已久的绝学--还~我~漂~漂~拳~。  
    
  用 DT\_FixRes 中的 FixResDemo.exe（一代豪杰 dREAMtHEATER 的作品，真是，一个 demo 都这么帅）打开 DIGI\_DUMP\_.EXE。  
  不多说，直接用"Fix Resource"来修复。（显示 Resource was fixed successfully，成功）  
    
  用 LordPE 打开修复后的 DIGI\_DUMP\_.EXE，我们可以看到程序中又多了一个.rsrc 段。  
  不过，别的段的名字都被抹掉了，不好看。  
    
  没关系，我们再补一拳。在 LordPE 中将各段名字一一改回。  
    
                 地址          长度  
     .text       00410000     10000  
     .rdata      00411000      4000  
     .data       00415000    204000  
     .oldrsrc    00619000      3000  
     .perplex    0061C000     1D000  
     .newimt     00639000      1000  
     .rsrc       0063A000      3000  
    
  （秋香，你变漂亮了耶）  
    
  四、（行动）Action!  
    
  用 ResHack 打开修复好的 DIGI\_DUMP\_.EXE，找到 ID 为 102 的对话框，即是我们要找的对话框：  
    
  //////////////////////////////////////////////////////////////////////////  
  102 DIALOGEX 0, 0, 215, 196  
  STYLE DS\_MODALFRAME | WS\_MINIMIZEBOX | WS\_POPUP | WS\_VISIBLE | WS\_CAPTION | WS\_SYSMENU  
  EXSTYLE WS\_EX\_APPWINDOW  
  CAPTION "连接网络--DigitalChina weba client"  
  LANGUAGE LANG\_CHINESE, 0x2  
  FONT 9, "宋体"  
  {  
     CONTROL "", 1042, EDIT, ES\_LEFT | ES\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 51, 113, 151, 12   
     CONTROL "", 1043, EDIT, ES\_LEFT | ES\_PASSWORD | ES\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 51, 130, 151, 12   
     CONTROL "保存密码", 1044, BUTTON, BS\_AUTOCHECKBOX | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 51, 147, 44, 8   
     CONTROL "自动加载", 1045, BUTTON, BS\_AUTOCHECKBOX | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 102, 147, 47, 9   
     CONTROL "自动连接", 1046, BUTTON, BS\_AUTOCHECKBOX | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 156, 147, 45, 8   
     CONTROL "连接", 1001, BUTTON, BS\_DEFPUSHBUTTON | BS\_NOTIFY | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 67, 171, 41, 14   
     CONTROL "退出", 2, BUTTON, BS\_PUSHBUTTON | BS\_NOTIFY | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 117, 171, 41, 14   
     CONTROL "属性", 1000, BUTTON, BS\_PUSHBUTTON | BS\_NOTIFY | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 166, 171, 41, 14   
     CONTROL "", 1026, STATIC, SS\_BITMAP | WS\_CHILD | WS\_VISIBLE, 7, 7, 15, 14   
     CONTROL "", -1, BUTTON, BS\_GROUPBOX | WS\_CHILD | WS\_VISIBLE, 7, 99, 201, 62   
     CONTROL "用户名：", -1, STATIC, SS\_LEFT | WS\_CHILD | WS\_VISIBLE | WS\_GROUP, 14, 117, 35, 11   
     CONTROL "密  码：", -1, STATIC, SS\_LEFT | WS\_CHILD | WS\_VISIBLE | WS\_GROUP, 14, 133, 35, 11   
     CONTROL "", -1, STATIC, SS\_ETCHEDFRAME | WS\_CHILD | WS\_VISIBLE, 7, 7, 201, 87 , 0x00000020  
  }  
  //////////////////////////////////////////////////////////////////////////  
    
  一堆代码，不习惯也没关系，ResHack 有自带的可视化编辑器。最后改成这样：  
    
  //////////////////////////////////////////////////////////////////////////  
  102 DIALOGEX 0, 0, 215, 196  
  STYLE DS\_MODALFRAME | WS\_MINIMIZEBOX | WS\_POPUP | WS\_VISIBLE | WS\_CAPTION | WS\_SYSMENU  
  EXSTYLE WS\_EX\_APPWINDOW  
  CAPTION "连接网络--DigitalChina weba client"  
  LANGUAGE LANG\_CHINESE, 0x2  
  FONT 9, "宋体"  
  {  
     CONTROL "", 1042, EDIT, ES\_LEFT | ES\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 51, 113, 151, 12   
     CONTROL "", 1043, EDIT, ES\_LEFT | ES\_PASSWORD | ES\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 51, 130, 151, 12   
     CONTROL "保存密码", 1044, BUTTON, BS\_AUTOCHECKBOX | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 51, 147, 44, 8   
     CONTROL "自动加载", 1045, BUTTON, BS\_AUTOCHECKBOX | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 102, 147, 47, 9   
     CONTROL "自动连接", 1046, BUTTON, BS\_AUTOCHECKBOX | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 156, 147, 45, 8   
     CONTROL "连接", 1001, BUTTON, BS\_DEFPUSHBUTTON | BS\_NOTIFY | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 12, 171, 41, 14   
     CONTROL "退出", 2, BUTTON, BS\_PUSHBUTTON | BS\_NOTIFY | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 112, 171, 41, 14   
     CONTROL "属性", 1000, BUTTON, BS\_PUSHBUTTON | BS\_NOTIFY | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 162, 171, 41, 14   
     CONTROL "", 1026, STATIC, SS\_BITMAP | WS\_CHILD | WS\_VISIBLE, 7, 7, 15, 14   
     CONTROL "", -1, BUTTON, BS\_GROUPBOX | WS\_CHILD | WS\_VISIBLE, 7, 99, 201, 62   
     CONTROL "用户名：", -1, STATIC, SS\_LEFT | WS\_CHILD | WS\_VISIBLE | WS\_GROUP, 14, 117, 35, 11   
     CONTROL "密  码：", -1, STATIC, SS\_LEFT | WS\_CHILD | WS\_VISIBLE | WS\_GROUP, 14, 133, 35, 11   
     CONTROL "", -1, STATIC, SS\_ETCHEDFRAME | WS\_CHILD | WS\_VISIBLE, 7, 7, 201, 87 , 0x00000020  
     CONTROL "高级", 1005, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 62, 171, 41, 14   
  }  
  //////////////////////////////////////////////////////////////////////////  
    
  调整了各个按钮的位置，加入了"高级"按钮。  
    
  用同样的方法，对 menu 资源进行修改：  
    
  修改前：  
    
  //////////////////////////////////////////////////////////////////////////  
  282 MENU  
  LANGUAGE LANG\_CHINESE, 0x2  
  {  
  POPUP "托盘菜单"  
  {  
    MENUITEM "显示连接状态",  32771  
    MENUITEM "断开当前连接",  32772  
    MENUITEM SEPARATOR  
    MENUITEM "属性...",  32773  
    MENUITEM "关于...",  32774  
  }  
  }  
  //////////////////////////////////////////////////////////////////////////  
    
  修改后：  
    
  //////////////////////////////////////////////////////////////////////////  
  282 MENU  
  LANGUAGE LANG\_CHINESE, 0x2  
  {  
  POPUP "托盘菜单"  
  {  
    MENUITEM "显示连接状态",  32771  
    MENUITEM "断开当前连接",  32772  
    MENUITEM SEPARATOR  
    MENUITEM "高级...",  32779  
    MENUITEM SEPARATOR  
    MENUITEM "属性...",  32773  
    MENUITEM "关于...",  32774  
  }  
  }  
  //////////////////////////////////////////////////////////////////////////  
    
  改完之后，点击"Compile Script"，然后用"Show Dialog"察看无误后保存。  
    
  （正宗人造美女出炉！正是出水芙蓉~ == ~芙蓉？ ~芙蓉？！~芙蓉！！ @#$%^&\*()\_ 。。。） 120，谢谢。  
    
  五、（闲暇）喝杯咖啡先  
    
  再次运行整容好的 DIGI\_DUMP\_.EXE，我们的"高级"按钮跃然“纸”上，菜单中也有了高级选项。（真是高级啊）  
    
  阶段性成果，文件名改为 DIGI\_DUMP\_RES.EXE。  
    
  资源修改成功！！  
    
\--------------------------------------------------------------------------------  
【经验总结】  
  至此，已经成功的修改了脱壳后的神州数码的资源，为后续的破解和改善打开了局面。  
    
  资源修改中的关键点有：  
   1. 有些壳会将资源跨段存放，或者资源没有位于最后一段，为修改带来不便，应先修复再修改。  
   2. 修改资源是很多工作（如汉化、功能增强等 pediy）的基础。  
   3. 资源修改是一件艺术工作多于是一件技术工作。  
    
  突破封锁线：第二章--资源篇 \[完\]  
    
  谢谢您耐心的看到这里，在下一章--网络篇中，将详细讲述如何跟踪和分析网络应用程序的工作原理：发挥我们的长处，弥  
  补它的漏洞。  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 04 月 28 日 22:55:09 

 

*   标 题： 突破封锁线：第三章--网络篇 
*   作 者：austiny 
*   时 间：2006-05-03 20:30    
    
*   链 接：[`bbs.pediy.com/showthread.php?threadid=25054`](http://bbs.pediy.com/showthread.php?threadid=25054) 

【文章标题】: 突破封锁线：第三章--网络篇  
【文章作者】: Austin  
【作者邮箱】: austiny.cn@gmail.com  
【软件名称】: 神州数码网络客户登陆程序  
【软件大小】: 不重要  
【加壳方式】: UltraProtect 1.x  
【保护方式】: 加壳 网络封禁  
【编写语言】: VC6.0  
【使用工具】: T-Sniffer OllyDbg  
【操作平台】: Windows2003  
【软件介绍】: 神州数码小区宽带网络登陆客户端，诸多限制  
【作者声明】: 技术交流中。。。  
\--------------------------------------------------------------------------------  
【详细过程】  
  一、（广告）前情回顾  
    
  上一回讲到，从韩国回来，我们已经给 DIGI 做了一个彻底的整容。这次我们来看看它里面有些什么。。  
    
  二、（废话）里面当然是代码啦  
    
  说得很对，不过，这一章我们先不看代码。前面说过，DIGI 是一个网络应用程序。我们来看看它的执行流程：  
    
  启动程序 --> 填写用户名、密码 --> 点击连接 --> 等待验证 --> 通过验证 --> 网络开通 --> 。。。 --> 下线  
    
  这过程当中 DIGI 要和服务器通讯很多次。我们的任务，就是找出它们到底说了些什么。  
  （007 双手交叉持枪造型中，小花猫友情客串邦女郎背影。。。）  
    
  三、（窃听）其实就是抓包啦  
    
  根据 TCP/IP 协议，数据在网络中会以包为单位发送和接收。（只见一个个包在天空飞来飞去，猪说，好大的棉花糖）  
  我们使用抓包工具（嗅探器）来查看包的内容。  
    
  不多说，打开 T-Sniffer（一个很小很小的嗅探器，到底有多小呢？ --只比倭国稍微大一点点）  
  当然用专业工具 Sniffer Pro，Ethereal 也是可以的，不过杀鸡焉用牛刀，这把就是劫机幢大楼专用水果刀啦。  
    
  先设置嗅探选项：  
  协议：     TCP UDP ICMP        //所有协议都选上先  
  数据大小： 65535  
  显示风格： ASCII 码  
  源端口：   0-65535             //所有端口都选上  
  目的端口： 0-65535             //同上  
  目标地址： 172.16.241.100      //这是服务器的地址  
  源地址：   0.0.0.0             //任意  
  嗅探接口： 192.168.0.177       //这是本机网络地址  
    
  然后开始抓包：  
    
   启动程序  
   填写用户名、密码  
    
   点击连接（抓到第一个包，包 A）  
    
  源地址:192.168.0.177  端口:1741  目标地址:172.16.241.100  端口:3848  TTL:128  PacketSize:98  
  协议:UDP  
  0x80 0x15 0x3C 0xE4 0x42 0x3B 0xBC 0x52 0xB9 0xCA 0x8C 0x6D 0x56 0x7B 0x0D 0x9A   ..<.B;.R...mV{..  
  0x1B 0xA4 0x80 0xA0 0x48 0x48 0xC9 0x48 0x59 0xD8 0xC8 0x01 0x21 0x68 0x58 0x58   ....HH.HY...!hXX  
  0x68 0x49 0xC8 0xD8 0xE8 0x91 0x20 0x00 0x04 0x27 0xFB 0xC6 0x50 0xA0 0xB1 0xC8   hI.... ..'..P...  
  0xD9 0x49 0x39 0xC8 0x59 0x39 0x58 0xC9 0x39 0xC8 0xD8 0x68 0x21 0x21 0xAC 0x3D   .I9.Y9X.9..h!!.=  
    
   此时网络已经开通  
    
   一分钟后（又抓到一个，包 B）  
    
  源地址:192.168.0.177  端口:1741  目标地址:172.16.241.100  端口:3848  TTL:128  PacketSize:87  
  协议:UDP  
  0x81 0xE9 0xAF 0x91 0x83 0x9B 0x69 0xFB 0x48 0x7C 0xC9 0x0C 0xCD 0x0E 0x38 0x6E   ......i.H|....8n  
  0x57 0xAF 0x91 0x20 0x00 0x04 0x27 0xFB 0xC6 0x50 0x20 0x41 0x49 0xC9 0xD9 0x68   W.. ..'..P AI..h  
  0xD8 0x49 0x68 0xD9 0x58 0xE8 0xC9 0x48 0x58 0xC8 0x58 0xC9 0xA0 0xB1 0xC8 0xE8   .Ih.X..HX.X.....  
    
   三十秒后（还是包 B）  
    
  源地址:192.168.0.177  端口:1741  目标地址:172.16.241.100  端口:3848  TTL:128  PacketSize:87  
  协议:UDP  
  0x81 0xE9 0xAF 0x91 0x83 0x9B 0x69 0xFB 0x48 0x7C 0xC9 0x0C 0xCD 0x0E 0x38 0x6E   ......i.H|....8n  
  0x57 0xAF 0x91 0x20 0x00 0x04 0x27 0xFB 0xC6 0x50 0x20 0x41 0x49 0xC9 0xD9 0x68   W.. ..'..P AI..h  
  0xD8 0x49 0x68 0xD9 0x58 0xE8 0xC9 0x48 0x58 0xC8 0x58 0xC9 0xA0 0xB1 0xC8 0xE8   .Ih.X..HX.X.....  
    
   此后每隔三十秒，都会发出同样的数据包。  
    
   若干次后，选择断开网络（发出四个包，包 C）  
    
  源地址:192.168.0.177  端口:3848  目标地址:172.16.241.100  端口:3848  TTL:128  PacketSize:87  
  协议:UDP  
  0x90 0xE9 0xF3 0xBE 0x68 0xDC 0xAC 0xC2 0xC9 0x67 0x68 0x2B 0x62 0xD1 0x4E 0x54   ....h....gh+b.NT  
  0x0E 0x3D 0x91 0x20 0x00 0x04 0x27 0xFB 0xC6 0x50 0x20 0x41 0x49 0xC9 0xD9 0x68   .=. ..'..P AI..h  
  0xD8 0x49 0x68 0xD9 0x58 0xE8 0xC9 0x48 0x58 0xC8 0x58 0xC9 0xA0 0xB1 0xC8 0xE8   .Ih.X..HX.X.....  
    
  源地址:192.168.0.177  端口:3848  目标地址:172.16.241.100  端口:3848  TTL:128  PacketSize:87  
  协议:UDP  
  0x90 0xE9 0xF3 0xBE 0x68 0xDC 0xAC 0xC2 0xC9 0x67 0x68 0x2B 0x62 0xD1 0x4E 0x54   ....h....gh+b.NT  
  0x0E 0x3D 0x91 0x20 0x00 0x04 0x27 0xFB 0xC6 0x50 0x20 0x41 0x49 0xC9 0xD9 0x68   .=. ..'..P AI..h  
  0xD8 0x49 0x68 0xD9 0x58 0xE8 0xC9 0x48 0x58 0xC8 0x58 0xC9 0xA0 0xB1 0xC8 0xE8   .Ih.X..HX.X.....  
    
  源地址:192.168.0.177  端口:3848  目标地址:172.16.241.100  端口:3848  TTL:128  PacketSize:87  
  协议:UDP  
  0x90 0xE9 0xF3 0xBE 0x68 0xDC 0xAC 0xC2 0xC9 0x67 0x68 0x2B 0x62 0xD1 0x4E 0x54   ....h....gh+b.NT  
  0x0E 0x3D 0x91 0x20 0x00 0x04 0x27 0xFB 0xC6 0x50 0x20 0x41 0x49 0xC9 0xD9 0x68   .=. ..'..P AI..h  
  0xD8 0x49 0x68 0xD9 0x58 0xE8 0xC9 0x48 0x58 0xC8 0x58 0xC9 0xA0 0xB1 0xC8 0xE8   .Ih.X..HX.X.....  
    
  源地址:192.168.0.177  端口:3848  目标地址:172.16.241.100  端口:3848  TTL:128  PacketSize:87  
  协议:UDP  
  0x90 0xE9 0xF3 0xBE 0x68 0xDC 0xAC 0xC2 0xC9 0x67 0x68 0x2B 0x62 0xD1 0x4E 0x54   ....h....gh+b.NT  
  0x0E 0x3D 0x91 0x20 0x00 0x04 0x27 0xFB 0xC6 0x50 0x20 0x41 0x49 0xC9 0xD9 0x68   .=. ..'..P AI..h  
  0xD8 0x49 0x68 0xD9 0x58 0xE8 0xC9 0x48 0x58 0xC8 0x58 0xC9 0xA0 0xB1 0xC8 0xE8   .Ih.X..HX.X.....  
    
   此时网络断开。  
    
  四、（分析）不是人类语言，鉴定完毕  
    
  当然不是，以上的数据包中包含地址端口用户名密码等加密后的信息。（能直接看懂的人都去演 Matrix 了）  
    
  好在我们不需要确切的知道每个包的具体内容。下面分析一下：  
    
  连接的时候，DIGI 发出包 A，用于提交用户名密码等信息交给服务器验证，内容当然就是那些东东啦。  
    
  此后每隔一定的时间，DIGI 会发出包 B，内容不变，用于通知服务器保持网络连接。  
  事实证明，如果三分钟后收不到包 B，服务器将切断网络连接。（此招在警匪片中绑匪常用）  
    
  最后断开连接时，DIGI 发出 4 个包 C，用于通知服务器断开连接。  
    
  五、（限制）。。，。。，嗯，确实没有"级"字  
    
  回忆一下 DIGI 对网络使用的限制：  
    
      1. 帐号与机器绑定（换机器不能登陆）  
      2. 限制共享网络连接（发现代理，强制下线）  
      3. 限制 BT 下载（强制下线）  
      4. 无法从路由后连接（拒绝登陆）  
    
  其中 1 和 4 的限制由包 A 来完成：  
  不是合适的机器（错误的用户名密码和 MAC 地址）或者非授权用户（也就是未缴费用户）将无法通过验证，  
  而路由器后的机器（错误的网络地址）也将无法通过验证。  
    
  2 和 3 的限制由包 B 来完成：  
  运行过程中，DIGI 会检测共享和 BT 下载，一旦发现，将不再发出包 B，并发出包 C 强制下线。  
    
  至此，DIGI 的工作流程已经昭然若揭。  
    
      1#. 收集信息（用户名密码 mac 地址网络地址等）打包  
      2#. 将包发至服务器  
      3#. 检测代理和 BT 等  
      4#. 向服务器发送保持连接包  
      5#. 下线  
    
  六、（成果）We are the world, we are the children.  
    
  好了，我们的目标是？ -- （小朋友可爱状）没~有~蛀~牙~！！  
    
  有道理，不过别忘了要突破封锁线才行哦。  
    
  对 DIGI 实施九年制应试教育，把它变成：  
    
      1#. 收集信息（用户名密码 mac 地址网络地址等）打包 --> 伪造数据包  
      2#. 将包发至服务器                              --> 将伪造包发至服务器  
      3#. 检测代理和 BT 等                              --> 禁止检测  
      4#. 向服务器发送保持连接包                      --> 这个当然  
      5#. 下线                                        --> 这个不变  
    
  经过这样一番改造，DIGI 将对我们死心塌地，忠心耿耿，至死不渝。。。（口水流一地）  
    
  （插一句，这也是大多数网游外挂的工作原理）  
    
  （继续流）  
    
\--------------------------------------------------------------------------------  
【经验总结】  
  至此，我们对 DIGI 进行了一次类似“黑箱”分析的体检，搞清楚了它的工作流程，并提出了改造方案。  
    
  网络分析中的关键点有：  
   1. 网络的开放性给我们提供了从外部观察程序的窗口   ................... 嗅探器  
   2. 网络是网络程序的优势也是劣势    .................................. 普遍适用  
    
  当然更多的网络分析会对包的内容进行解密和更细致的分析，此例简单，不再贽述。  
    
  突破封锁线：第三章--网络篇 \[完\]  
    
  谢谢您耐心的看到这里，在下一章--代码篇中，将详细讲述改造应用程序和加入自己 dll 到程序中。  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 05 月 03 日 20:22:25 

【文章标题】: 突破封锁线：第三章--网络篇  
【文章作者】: Austin  
【作者邮箱】: austiny.cn@gmail.com  
【软件名称】: 神州数码网络客户登陆程序  
【软件大小】: 不重要  
【加壳方式】: UltraProtect 1.x  
【保护方式】: 加壳 网络封禁  
【编写语言】: VC6.0  
【使用工具】: T-Sniffer OllyDbg  
【操作平台】: Windows2003  
【软件介绍】: 神州数码小区宽带网络登陆客户端，诸多限制  
【作者声明】: 技术交流中。。。  
\--------------------------------------------------------------------------------  
【详细过程】  
  一、（广告）前情回顾  
    
  上一回讲到，从韩国回来，我们已经给 DIGI 做了一个彻底的整容。这次我们来看看它里面有些什么。。  
    
  二、（废话）里面当然是代码啦  
    
  说得很对，不过，这一章我们先不看代码。前面说过，DIGI 是一个网络应用程序。我们来看看它的执行流程：  
    
  启动程序 --> 填写用户名、密码 --> 点击连接 --> 等待验证 --> 通过验证 --> 网络开通 --> 。。。 --> 下线  
    
  这过程当中 DIGI 要和服务器通讯很多次。我们的任务，就是找出它们到底说了些什么。  
  （007 双手交叉持枪造型中，小花猫友情客串邦女郎背影。。。）  
    
  三、（窃听）其实就是抓包啦  
    
  根据 TCP/IP 协议，数据在网络中会以包为单位发送和接收。（只见一个个包在天空飞来飞去，猪说，好大的棉花糖）  
  我们使用抓包工具（嗅探器）来查看包的内容。  
    
  不多说，打开 T-Sniffer（一个很小很小的嗅探器，到底有多小呢？ --只比倭国稍微大一点点）  
  当然用专业工具 Sniffer Pro，Ethereal 也是可以的，不过杀鸡焉用牛刀，这把就是劫机幢大楼专用水果刀啦。  
    
  先设置嗅探选项：  
  协议：     TCP UDP ICMP        //所有协议都选上先  
  数据大小： 65535  
  显示风格： ASCII 码  
  源端口：   0-65535             //所有端口都选上  
  目的端口： 0-65535             //同上  
  目标地址： 172.16.241.100      //这是服务器的地址  
  源地址：   0.0.0.0             //任意  
  嗅探接口： 192.168.0.177       //这是本机网络地址  
    
  然后开始抓包：  
    
   启动程序  
   填写用户名、密码  
    
   点击连接（抓到第一个包，包 A）  
    
  源地址:192.168.0.177  端口:1741  目标地址:172.16.241.100  端口:3848  TTL:128  PacketSize:98  
  协议:UDP  
  0x80 0x15 0x3C 0xE4 0x42 0x3B 0xBC 0x52 0xB9 0xCA 0x8C 0x6D 0x56 0x7B 0x0D 0x9A   ..<.B;.R...mV{..  
  0x1B 0xA4 0x80 0xA0 0x48 0x48 0xC9 0x48 0x59 0xD8 0xC8 0x01 0x21 0x68 0x58 0x58   ....HH.HY...!hXX  
  0x68 0x49 0xC8 0xD8 0xE8 0x91 0x20 0x00 0x04 0x27 0xFB 0xC6 0x50 0xA0 0xB1 0xC8   hI.... ..'..P...  
  0xD9 0x49 0x39 0xC8 0x59 0x39 0x58 0xC9 0x39 0xC8 0xD8 0x68 0x21 0x21 0xAC 0x3D   .I9.Y9X.9..h!!.=  
    
   此时网络已经开通  
    
   一分钟后（又抓到一个，包 B）  
    
  源地址:192.168.0.177  端口:1741  目标地址:172.16.241.100  端口:3848  TTL:128  PacketSize:87  
  协议:UDP  
  0x81 0xE9 0xAF 0x91 0x83 0x9B 0x69 0xFB 0x48 0x7C 0xC9 0x0C 0xCD 0x0E 0x38 0x6E   ......i.H|....8n  
  0x57 0xAF 0x91 0x20 0x00 0x04 0x27 0xFB 0xC6 0x50 0x20 0x41 0x49 0xC9 0xD9 0x68   W.. ..'..P AI..h  
  0xD8 0x49 0x68 0xD9 0x58 0xE8 0xC9 0x48 0x58 0xC8 0x58 0xC9 0xA0 0xB1 0xC8 0xE8   .Ih.X..HX.X.....  
    
   三十秒后（还是包 B）  
    
  源地址:192.168.0.177  端口:1741  目标地址:172.16.241.100  端口:3848  TTL:128  PacketSize:87  
  协议:UDP  
  0x81 0xE9 0xAF 0x91 0x83 0x9B 0x69 0xFB 0x48 0x7C 0xC9 0x0C 0xCD 0x0E 0x38 0x6E   ......i.H|....8n  
  0x57 0xAF 0x91 0x20 0x00 0x04 0x27 0xFB 0xC6 0x50 0x20 0x41 0x49 0xC9 0xD9 0x68   W.. ..'..P AI..h  
  0xD8 0x49 0x68 0xD9 0x58 0xE8 0xC9 0x48 0x58 0xC8 0x58 0xC9 0xA0 0xB1 0xC8 0xE8   .Ih.X..HX.X.....  
    
   此后每隔三十秒，都会发出同样的数据包。  
    
   若干次后，选择断开网络（发出四个包，包 C）  
    
  源地址:192.168.0.177  端口:3848  目标地址:172.16.241.100  端口:3848  TTL:128  PacketSize:87  
  协议:UDP  
  0x90 0xE9 0xF3 0xBE 0x68 0xDC 0xAC 0xC2 0xC9 0x67 0x68 0x2B 0x62 0xD1 0x4E 0x54   ....h....gh+b.NT  
  0x0E 0x3D 0x91 0x20 0x00 0x04 0x27 0xFB 0xC6 0x50 0x20 0x41 0x49 0xC9 0xD9 0x68   .=. ..'..P AI..h  
  0xD8 0x49 0x68 0xD9 0x58 0xE8 0xC9 0x48 0x58 0xC8 0x58 0xC9 0xA0 0xB1 0xC8 0xE8   .Ih.X..HX.X.....  
    
  源地址:192.168.0.177  端口:3848  目标地址:172.16.241.100  端口:3848  TTL:128  PacketSize:87  
  协议:UDP  
  0x90 0xE9 0xF3 0xBE 0x68 0xDC 0xAC 0xC2 0xC9 0x67 0x68 0x2B 0x62 0xD1 0x4E 0x54   ....h....gh+b.NT  
  0x0E 0x3D 0x91 0x20 0x00 0x04 0x27 0xFB 0xC6 0x50 0x20 0x41 0x49 0xC9 0xD9 0x68   .=. ..'..P AI..h  
  0xD8 0x49 0x68 0xD9 0x58 0xE8 0xC9 0x48 0x58 0xC8 0x58 0xC9 0xA0 0xB1 0xC8 0xE8   .Ih.X..HX.X.....  
    
  源地址:192.168.0.177  端口:3848  目标地址:172.16.241.100  端口:3848  TTL:128  PacketSize:87  
  协议:UDP  
  0x90 0xE9 0xF3 0xBE 0x68 0xDC 0xAC 0xC2 0xC9 0x67 0x68 0x2B 0x62 0xD1 0x4E 0x54   ....h....gh+b.NT  
  0x0E 0x3D 0x91 0x20 0x00 0x04 0x27 0xFB 0xC6 0x50 0x20 0x41 0x49 0xC9 0xD9 0x68   .=. ..'..P AI..h  
  0xD8 0x49 0x68 0xD9 0x58 0xE8 0xC9 0x48 0x58 0xC8 0x58 0xC9 0xA0 0xB1 0xC8 0xE8   .Ih.X..HX.X.....  
    
  源地址:192.168.0.177  端口:3848  目标地址:172.16.241.100  端口:3848  TTL:128  PacketSize:87  
  协议:UDP  
  0x90 0xE9 0xF3 0xBE 0x68 0xDC 0xAC 0xC2 0xC9 0x67 0x68 0x2B 0x62 0xD1 0x4E 0x54   ....h....gh+b.NT  
  0x0E 0x3D 0x91 0x20 0x00 0x04 0x27 0xFB 0xC6 0x50 0x20 0x41 0x49 0xC9 0xD9 0x68   .=. ..'..P AI..h  
  0xD8 0x49 0x68 0xD9 0x58 0xE8 0xC9 0x48 0x58 0xC8 0x58 0xC9 0xA0 0xB1 0xC8 0xE8   .Ih.X..HX.X.....  
    
   此时网络断开。  
    
  四、（分析）不是人类语言，鉴定完毕  
    
  当然不是，以上的数据包中包含地址端口用户名密码等加密后的信息。（能直接看懂的人都去演 Matrix 了）  
    
  好在我们不需要确切的知道每个包的具体内容。下面分析一下：  
    
  连接的时候，DIGI 发出包 A，用于提交用户名密码等信息交给服务器验证，内容当然就是那些东东啦。  
    
  此后每隔一定的时间，DIGI 会发出包 B，内容不变，用于通知服务器保持网络连接。  
  事实证明，如果三分钟后收不到包 B，服务器将切断网络连接。（此招在警匪片中绑匪常用）  
    
  最后断开连接时，DIGI 发出 4 个包 C，用于通知服务器断开连接。  
    
  五、（限制）。。，。。，嗯，确实没有"级"字  
    
  回忆一下 DIGI 对网络使用的限制：  
    
      1. 帐号与机器绑定（换机器不能登陆）  
      2. 限制共享网络连接（发现代理，强制下线）  
      3. 限制 BT 下载（强制下线）  
      4. 无法从路由后连接（拒绝登陆）  
    
  其中 1 和 4 的限制由包 A 来完成：  
  不是合适的机器（错误的用户名密码和 MAC 地址）或者非授权用户（也就是未缴费用户）将无法通过验证，  
  而路由器后的机器（错误的网络地址）也将无法通过验证。  
    
  2 和 3 的限制由包 B 来完成：  
  运行过程中，DIGI 会检测共享和 BT 下载，一旦发现，将不再发出包 B，并发出包 C 强制下线。  
    
  至此，DIGI 的工作流程已经昭然若揭。  
    
      1#. 收集信息（用户名密码 mac 地址网络地址等）打包  
      2#. 将包发至服务器  
      3#. 检测代理和 BT 等  
      4#. 向服务器发送保持连接包  
      5#. 下线  
    
  六、（成果）We are the world, we are the children.  
    
  好了，我们的目标是？ -- （小朋友可爱状）没~有~蛀~牙~！！  
    
  有道理，不过别忘了要突破封锁线才行哦。  
    
  对 DIGI 实施九年制应试教育，把它变成：  
    
      1#. 收集信息（用户名密码 mac 地址网络地址等）打包 --> 伪造数据包  
      2#. 将包发至服务器                              --> 将伪造包发至服务器  
      3#. 检测代理和 BT 等                              --> 禁止检测  
      4#. 向服务器发送保持连接包                      --> 这个当然  
      5#. 下线                                        --> 这个不变  
    
  经过这样一番改造，DIGI 将对我们死心塌地，忠心耿耿，至死不渝。。。（口水流一地）  
    
  （插一句，这也是大多数网游外挂的工作原理）  
    
  （继续流）  
    
\--------------------------------------------------------------------------------  
【经验总结】  
  至此，我们对 DIGI 进行了一次类似“黑箱”分析的体检，搞清楚了它的工作流程，并提出了改造方案。  
    
  网络分析中的关键点有：  
   1. 网络的开放性给我们提供了从外部观察程序的窗口   ................... 嗅探器  
   2. 网络是网络程序的优势也是劣势    .................................. 普遍适用  
    
  当然更多的网络分析会对包的内容进行解密和更细致的分析，此例简单，不再贽述。  
    
  突破封锁线：第三章--网络篇 \[完\]  
    
  谢谢您耐心的看到这里，在下一章--代码篇中，将详细讲述改造应用程序和加入自己 dll 到程序中。  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 05 月 03 日 20:22:25 

【文章标题】: 突破封锁线：第四章--代码篇  
【文章作者】: Austin  
【作者邮箱】: austiny.cn@gmail.com  
【软件名称】: 神州数码网络客户登陆程序  
【软件大小】: 不重要  
【下载地址】: 自己搜索下载  
【加壳方式】: UltraProtect 1.x  
【保护方式】: 加壳 网络封禁  
【编写语言】: VC6.0  
【使用工具】: OllyDbg VC6.0 LordPE  
【操作平台】: Windows2003  
【软件介绍】: 神州数码小区宽带网络登陆客户端，诸多限制  
【作者声明】: 技术交流中。。。  
\--------------------------------------------------------------------------------  
【详细过程】  
  一、（停电）蜡烛 + 收音机 = 前情回顾  
    
  上一回说到，我们找到了 DIGI 的弱点，我们来突破它。。。  
    
  二、（热身）第一次的亲密接触  
    
  我们的计划是，截获 DIGI 发包的函数，用我们自己的发包函数来替代它发出伪造过的包。  
  （据说在欧洲，穿假名牌是要脱光光的。\[口水\]，真是好制度啊）  
    
  用 OllyDbg 打开 DIGI，停在入口处：  
    
  0040FB74 >/$  68 A82A4100   push    00412AA8  
  0040FB79  |.  68 D4FC4000   push    <jmp.&msvcrt.\_except\_handler3>   ;  SE 处理程序安装  
  0040FB7E  |.  64:A1 0000000>mov     eax, fs:\[0\]  
    
  网络应用程序发送数据包，最后都要用到 wsock32.dll 中的 sendto 函数。找到这个函数，就能找到调用它的发包函数。  
    
  在 CPU 窗口中点击 Ctrl+N 查看当前模块中的名称，果然发现了对 sendto 函数的引用：  
    
  ...  
  00411538   .rdata     输入    (    user32.SendMessageA  
  00411598   .rdata     输入    (    wsock32.sendto  
  00411064   .rdata     输入    (    kernel32.SetEvent  
  ...  
    
  光标移到 sendto 项上，回车查看输入函数参考：  
    
  参考位于 digi\_dum:.text 到 wsock32.sendto  
  地址       反汇编                                    注释  
  00403A10   call    <jmp.&wsock32.sendto>  
  00403A31   call    <jmp.&wsock32.sendto>  
  00403A52   call    <jmp.&wsock32.sendto>  
  00403A73   call    <jmp.&wsock32.sendto>  
  00403CC3   call    <jmp.&wsock32.sendto>  
  00403CE8   call    <jmp.&wsock32.sendto>  
  00403D07   call    <jmp.&wsock32.sendto>  
  00403ECF   call    <jmp.&wsock32.sendto>  
  004040F9   call    <jmp.&wsock32.sendto>  
  0040542F   call    <jmp.&wsock32.sendto>  
  004083F4   call    <jmp.&wsock32.sendto>  
  0040FD50   jmp     \[<&wsock32.sendto>\]               WS2\_32.sendto  
    
  出去最后一项是一个跳转，我们来挨个考察到底哪个调用是发包函数：  
    
  在第一项上回车，在 cpu 窗口中来到 00403A10 处：  
    
  004039FF   .  8BF0          mov     esi, eax  
  00403A01   .  8D4424 24     lea     eax, \[esp+24\]  
  00403A05   .  6A 10         push    10                               ; /ToLength = 10 (16.)  
  00403A07   .  50            push    eax                              ; |pTo  
  00403A08   .  55            push    ebp                              ; |Flags  
  00403A09   .  8D4C24 40     lea     ecx, \[esp+40\]                    ; |  
  00403A0D   .  56            push    esi                              ; |DataSize  
  00403A0E   .  51            push    ecx                              ; |Data  
  00403A0F   .  52            push    edx                              ; |Socket => 0  
  00403A10   .  E8 3BC30000   call    <jmp.&wsock32.sendto>            ; \\sendto  
  00403A15   .  68 4A010000   push    14A  
  00403A1A   .  FFD3          call    ebx  
  00403A1C   .  8B15 386C4100 mov     edx, \[416C38\]  
  00403A22   .  8D4424 24     lea     eax, \[esp+24\]  
  00403A26   .  6A 10         push    10                               ; /ToLength = 10 (16.)  
  00403A28   .  50            push    eax                              ; |pTo  
  00403A29   .  55            push    ebp                              ; |Flags  
  00403A2A   .  8D4C24 40     lea     ecx, \[esp+40\]                    ; |  
  00403A2E   .  56            push    esi                              ; |DataSize  
  00403A2F   .  51            push    ecx                              ; |Data  
  00403A30   .  52            push    edx                              ; |Socket => 0  
  00403A31   .  E8 1AC30000   call    <jmp.&wsock32.sendto>            ; \\sendto  
  00403A36   .  68 4A010000   push    14A  
  00403A3B   .  FFD3          call    ebx  
  00403A3D   .  8B15 386C4100 mov     edx, \[416C38\]  
  00403A43   .  8D4424 24     lea     eax, \[esp+24\]  
  00403A47   .  6A 10         push    10                               ; /ToLength = 10 (16.)  
  00403A49   .  50            push    eax                              ; |pTo  
  00403A4A   .  55            push    ebp                              ; |Flags  
  00403A4B   .  8D4C24 40     lea     ecx, \[esp+40\]                    ; |  
  00403A4F   .  56            push    esi                              ; |DataSize  
  00403A50   .  51            push    ecx                              ; |Data  
  00403A51   .  52            push    edx                              ; |Socket => 0  
  00403A52   .  E8 F9C20000   call    <jmp.&wsock32.sendto>            ; \\sendto  
  00403A57   .  68 4A010000   push    14A  
  00403A5C   .  FFD3          call    ebx  
  00403A5E   .  8B15 386C4100 mov     edx, \[416C38\]  
  00403A64   .  8D4424 24     lea     eax, \[esp+24\]  
  00403A68   .  6A 10         push    10                               ; /ToLength = 10 (16.)  
  00403A6A   .  50            push    eax                              ; |pTo  
  00403A6B   .  55            push    ebp                              ; |Flags  
  00403A6C   .  8D4C24 40     lea     ecx, \[esp+40\]                    ; |  
  00403A70   .  56            push    esi                              ; |DataSize  
  00403A71   .  51            push    ecx                              ; |Data  
  00403A72   .  52            push    edx                              ; |Socket => 0  
  00403A73   .  E8 D8C20000   call    <jmp.&wsock32.sendto>            ; \\sendto  
  00403A78   .  A1 386C4100   mov     eax, \[416C38\]  
  00403A7D   .  5F            pop     edi  
  00403A7E   .  3BC5          cmp     eax, ebp  
  00403A80   .  74 0C         je      short 00403A8E  
  00403A82   .  50            push    eax                              ; /Socket => 0  
  00403A83   .  E8 C2C20000   call    <jmp.&wsock32.closesocket>       ; \\closesocket  
    
  一眼扫去，不免多看了几行，发现这里有连续四次对 sendto 的调用，也就是说连续发出了四个数据包。  
  回忆一下上一章的内容，。。。，对了，只有在断开连接的时候才会发出 4 个包的，所以前 4 个调用都可以排除。  
    
  看看第 5 个，00403CC3：  
    
  00403CB5  |.  8BF0          mov     esi, eax  
  00403CB7  |.  6A 10         push    10                               ; /ToLength = 10 (16.)  
  00403CB9  |.  52            push    edx                              ; |pTo  
  00403CBA  |.  6A 00         push    0                                ; |Flags = 0  
  00403CBC  |.  8D4424 34     lea     eax, \[esp+34\]                    ; |  
  00403CC0  |.  56            push    esi                              ; |DataSize  
  00403CC1  |.  50            push    eax                              ; |Data  
  00403CC2  |.  51            push    ecx                              ; |Socket => 0  
  00403CC3  |.  E8 88C00000   call    <jmp.&wsock32.sendto>            ; \\sendto  
  00403CC8  |.  8B3D 7C104100 mov     edi, \[<&kernel32.Sleep>\]         ;  kernel32.Sleep  
  00403CCE  |.  6A 64         push    64                               ; /Timeout = 100. ms  
  00403CD0  |.  FFD7          call    edi                              ; \\Sleep  
  00403CD2  |.  8B0D 386C4100 mov     ecx, \[416C38\]  
  00403CD8  |.  8D5424 18     lea     edx, \[esp+18\]  
  00403CDC  |.  6A 10         push    10                               ; /ToLength = 10 (16.)  
  00403CDE  |.  52            push    edx                              ; |pTo  
  00403CDF  |.  6A 00         push    0                                ; |Flags = 0  
  00403CE1  |.  8D4424 34     lea     eax, \[esp+34\]                    ; |  
  00403CE5  |.  56            push    esi                              ; |DataSize  
  00403CE6  |.  50            push    eax                              ; |Data  
  00403CE7  |.  51            push    ecx                              ; |Socket => 0  
  00403CE8  |.  E8 63C00000   call    <jmp.&wsock32.sendto>            ; \\sendto  
  00403CED  |.  6A 64         push    64                               ; /Timeout = 100. ms  
  00403CEF  |.  FFD7          call    edi                              ; \\Sleep  
  00403CF1  |.  8B0D 386C4100 mov     ecx, \[416C38\]  
  00403CF7  |.  8D5424 18     lea     edx, \[esp+18\]  
  00403CFB  |.  6A 10         push    10                               ; /ToLength = 10 (16.)  
  00403CFD  |.  52            push    edx                              ; |pTo  
  00403CFE  |.  6A 00         push    0                                ; |Flags = 0  
  00403D00  |.  8D4424 34     lea     eax, \[esp+34\]                    ; |  
  00403D04  |.  56            push    esi                              ; |DataSize  
  00403D05  |.  50            push    eax                              ; |Data  
  00403D06  |.  51            push    ecx                              ; |Socket => 0  
  00403D07  |.  E8 44C00000   call    <jmp.&wsock32.sendto>            ; \\sendto  
  00403D0C  |.  5F            pop     edi  
  00403D0D  |.  B8 01000000   mov     eax, 1  
  00403D12  |.  5E            pop     esi  
  00403D13  |.  81C4 20020000 add     esp, 220  
  00403D19  \\.  C3            retn  
    
  习惯性的又多看了几行，这里发了三个包，中间用 Sleep 函数作了 0.1 秒的技术暂停。  
  （理发师：很明显我就是这样的人）  
  （包租公包租婆：很明显，你不是）  
    
  既然不是，又排除三个，我们再往下看，00403ECF：  
    
  00403EB1   .  BF 01000000   mov     edi, 1  
  00403EB6   .  52            push    edx                              ; /ToLength  
  00403EB7   .  8D5424 40     lea     edx, \[esp+40\]                    ; |  
  00403EBB   .  52            push    edx                              ; |pTo  
  00403EBC   .  53            push    ebx                              ; |Flags  
  00403EBD   .  50            push    eax                              ; |DataSize  
  00403EBE   .  8D8424 700100>lea     eax, \[esp+170\]                   ; |  
  00403EC5   .  50            push    eax                              ; |Data  
  00403EC6   .  51            push    ecx                              ; |Socket => 0  
  00403EC7   .  894C24 78     mov     \[esp+78\], ecx                    ; |  
  00403ECB   .  897C24 74     mov     \[esp+74\], edi                    ; |  
  00403ECF   .  E8 7CBE0000   call    <jmp.&wsock32.sendto>            ; \\sendto  
  00403ED4   .  A1 386C4100   mov     eax, \[416C38\]  
  00403ED9   .  8D4C24 34     lea     ecx, \[esp+34\]  
  00403EDD   .  51            push    ecx                              ; /pTimeout  
  00403EDE   .  53            push    ebx                              ; |Exceptfds  
  00403EDF   .  8D5424 64     lea     edx, \[esp+64\]                    ; |  
  00403EE3   .  53            push    ebx                              ; |Writefds  
  00403EE4   .  40            inc     eax                              ; |  
  00403EE5   .  52            push    edx                              ; |Readfds  
  00403EE6   .  50            push    eax                              ; |nfds  
  00403EE7   .  E8 7CBE0000   call    <jmp.&wsock32.select>            ; \\select  
  00403EEC   .  83F8 FF       cmp     eax, -1  
  00403EEF   .  74 65         je      short 00403F56  
  00403EF1   .  8B15 386C4100 mov     edx, \[416C38\]  
  00403EF7   .  8D4C24 5C     lea     ecx, \[esp+5C\]  
  00403EFB   .  51            push    ecx  
  00403EFC   .  52            push    edx  
  00403EFD   .  E8 60BE0000   call    <jmp.&wsock32.\_\_WSAFDIsSet>  
  00403F02   .  85C0          test    eax, eax  
  00403F04   .  74 50         je      short 00403F56  
  00403F06   .  8D4424 14     lea     eax, \[esp+14\]  
  00403F0A   .  8D4C24 4C     lea     ecx, \[esp+4C\]  
  00403F0E   .  50            push    eax                              ; /pFromLen  
  00403F0F   .  A1 386C4100   mov     eax, \[416C38\]                    ; |  
  00403F14   .  51            push    ecx                              ; |pFrom  
  00403F15   .  53            push    ebx                              ; |Flags  
  00403F16   .  8D9424 6C0300>lea     edx, \[esp+36C\]                   ; |  
  00403F1D   .  68 00020000   push    200                              ; |BufSize = 200 (512.)  
  00403F22   .  52            push    edx                              ; |Buffer  
  00403F23   .  50            push    eax                              ; |Socket => 0  
  00403F24   .  E8 33BE0000   call    <jmp.&wsock32.recvfrom>          ; \\recvfrom  
  00403F29   .  8BF0          mov     esi, eax  
    
  这个很完整了，call 完 sendto，又 call 了 select 和 recvfrom，有嫌疑。  
  （靠，call 这么多人来，斧头帮聚会啊）  
    
  先不着急，再看下一个，004040F9：  
    
  004040DA   .  BF 01000000   mov     edi, 1  
  004040DF   .  52            push    edx                              ; /ToLength  
  004040E0   .  8D5424 40     lea     edx, \[esp+40\]                    ; |  
  004040E4   .  52            push    edx                              ; |pTo  
  004040E5   .  6A 00         push    0                                ; |Flags = 0  
  004040E7   .  50            push    eax                              ; |DataSize  
  004040E8   .  8D8424 700100>lea     eax, \[esp+170\]                   ; |  
  004040EF   .  50            push    eax                              ; |Data  
  004040F0   .  51            push    ecx                              ; |Socket => 0  
  004040F1   .  894C24 78     mov     \[esp+78\], ecx                    ; |  
  004040F5   .  897C24 74     mov     \[esp+74\], edi                    ; |  
  004040F9   .  E8 52BC0000   call    <jmp.&wsock32.sendto>            ; \\sendto  
  004040FE   .  A1 386C4100   mov     eax, \[416C38\]  
  00404103   .  8D4C24 2C     lea     ecx, \[esp+2C\]  
  00404107   .  51            push    ecx                              ; /pTimeout  
  00404108   .  6A 00         push    0                                ; |Exceptfds = NULL  
  0040410A   .  8D5424 64     lea     edx, \[esp+64\]                    ; |  
  0040410E   .  6A 00         push    0                                ; |Writefds = NULL  
  00404110   .  40            inc     eax                              ; |  
  00404111   .  52            push    edx                              ; |Readfds  
  00404112   .  50            push    eax                              ; |nfds  
  00404113   .  E8 50BC0000   call    <jmp.&wsock32.select>            ; \\select  
  00404118   .  3BC7          cmp     eax, edi  
  0040411A   .  75 78         jnz     short 00404194  
  0040411C   .  8B15 386C4100 mov     edx, \[416C38\]  
  00404122   .  8D4C24 5C     lea     ecx, \[esp+5C\]  
  00404126   .  51            push    ecx  
  00404127   .  52            push    edx  
  00404128   .  E8 35BC0000   call    <jmp.&wsock32.\_\_WSAFDIsSet>  
  0040412D   .  85C0          test    eax, eax  
  0040412F   .  74 68         je      short 00404199  
  00404131   .  8D4424 14     lea     eax, \[esp+14\]  
  00404135   .  8D4C24 4C     lea     ecx, \[esp+4C\]  
  00404139   .  50            push    eax                              ; /pFromLen  
  0040413A   .  A1 386C4100   mov     eax, \[416C38\]                    ; |  
  0040413F   .  51            push    ecx                              ; |pFrom  
  00404140   .  6A 00         push    0                                ; |Flags = 0  
  00404142   .  8D9424 6C0300>lea     edx, \[esp+36C\]                   ; |  
  00404149   .  68 00020000   push    200                              ; |BufSize = 200 (512.)  
  0040414E   .  52            push    edx                              ; |Buffer  
  0040414F   .  50            push    eax                              ; |Socket => 0  
  00404150   .  E8 07BC0000   call    <jmp.&wsock32.recvfrom>          ; \\recvfrom  
  00404155   .  8BF0          mov     esi, eax  
  00404157   .  85F6          test    esi, esi  
  00404159   .  7E 2C         jle     short 00404187  
  0040415B   .  8B0D A86E4100 mov     ecx, \[416EA8\]  
  00404161   .  51            push    ecx                              ; /pAddr => 00034460  
  00404162   .  E8 EFBB0000   call    <jmp.&wsock32.inet\_addr>         ; \\inet\_addr  
  00404167   .  394424 50     cmp     \[esp+50\], eax  
  0040416B   .  75 1A         jnz     short 00404187  
  0040416D   .  8D9424 600300>lea     edx, \[esp+360\]  
  00404174   .  56            push    esi  
  00404175   .  52            push    edx  
  00404176   .  E8 651A0000   call    00405BE0  
  0040417B   .  8BE8          mov     ebp, eax  
  0040417D   .  83C4 08       add     esp, 8  
  00404180   .  83FD 02       cmp     ebp, 2  
  00404183   .  74 73         je      short 004041F8  
  00404185   .  EB 12         jmp     short 00404199  
  00404187   >  68 88130000   push    1388                             ; /Timeout = 5000. ms  
  0040418C   .  FF15 7C104100 call    \[<&kernel32.Sleep>\]              ; \\Sleep  
  00404192   .  EB 05         jmp     short 00404199  
  00404194   >  BD 21000000   mov     ebp, 21  
  00404199   >  43            inc     ebx  
  0040419A   .  3BEF          cmp     ebp, edi  
  0040419C   .  75 0D         jnz     short 004041AB  
  0040419E   .  68 30750000   push    7530                             ; /Timeout = 30000. ms  
  004041A3   .  33DB          xor     ebx, ebx                         ; |  
  004041A5   .  FF15 7C104100 call    \[<&kernel32.Sleep>\]              ; \\Sleep  
  004041AB   >  393D 6C6E4100 cmp     \[416E6C\], edi  
  004041B1   .^ 0F84 D3FEFFFF je      0040408A  
  004041B7   .  EB 72         jmp     short 0040422B  
  004041B9   >  A1 386C4100   mov     eax, \[416C38\]  
  004041BE   .  33F6          xor     esi, esi  
  004041C0   .  50            push    eax                              ; /Socket => 0  
  004041C1   .  8935 6C6E4100 mov     \[416E6C\], esi                    ; |  
  004041C7   .  E8 7EBB0000   call    <jmp.&wsock32.closesocket>       ; \\closesocket  
  004041CC   .  8935 386C4100 mov     \[416C38\], esi  
  004041D2   .  E8 6DBB0000   call    <jmp.&wsock32.WSACleanup>        ; \[WSACleanup  
  004041D7   .  56            push    esi  
    
  （这个 call 不但完整，而且小小年纪竟然就有一副横练的筋骨，真是百年难得一遇的武学奇才。）  
  注意到有一个 30 秒的 sleep，回忆上一章的内容，没错，每隔三十秒发包，这个就是用来保持连接的。  
    
  考虑到上一个嫌疑调用就在这儿不远处，极有可能就是发包函数。  
    
  在斧头帮聚会的 00403ECF 处，下一个断点，跑起来。  
    
  然后连接，果然，程序乖乖的断在了 00403ECF 处。就是这里啦。（记下来）  
    
  当然，给 sendto 的调用下全局断点，然后运行看停在哪里，也是一样的。只是--  
  （只是，只是我当时没有想到而已。有时觉得自己真是聪明的可以）  
    
  三、（再热）第二次的亲密接触  
    
  （如果把整个太平洋的水倒出来。。。 再热的话，太平洋里就没有水了）  
    
  还记得整容的时候给 DIGI 加的"高级"按钮吗？现在我们来给它加上处理函数。  
    
  由于 DIGI 是一个 MFC 程序，所以突破口在 CCmdTarget::OnCmdMsg（此函数是 mfc 对 windows 消息处理的封装）  
    
  先找到主对话框的消息表：  
    
  重新加载 DIGI，Ctrl+N 打开名称列表，找到 CCmdTarget::OnCmdMsg：  
    
  ...  
  00411374   .rdata     输入         mfc42.#4407\_CWnd::OnChildNotify  
  00411138   .rdata     输入         mfc42.#4424\_CCmdTarget::OnCmdMsg  
  ...  
    
  在数据窗口跟随至 00411138：  
    
  00411138 >6BC4223C  mfc42.#4424\_CCmdTarget::OnCmdMsg  
  0041113C >6BCB87C6  mfc42.#3738\_CWinApp::GetRuntimeClass  
  00411140 >6BC575A1  mfc42.#815\_CWinApp::~CWinApp  
  00411144 >6BC4AFAF  mfc42.#561\_CWinApp::CWinApp  
  00411148 >6BC48D34  mfc42.#641\_CDialog::~CDialog  
  0041114C >6BC5019D  mfc42.#609\_CButton::~CButton  
  00411150 >6BC544E5  mfc42.#795\_CStatic::~CStatic  
  00411154 >6BC56AB1  mfc42.#2514\_CDialog::DoModal  
    
  可知 CCmdTarget::OnCmdMsg 的地址在 6BC4223C，在 6BC4223C 处下断，点击界面中高级按钮，断点如约而至：  
    
  6BC4223C >  55              push    ebp  
  6BC4223D    8BEC            mov     ebp, esp  
  6BC4223F    8B45 0C         mov     eax, \[ebp+C\]  
  6BC42242    53              push    ebx  
  6BC42243    56              push    esi  
  6BC42244    57              push    edi  
  6BC42245    83F8 FE         cmp     eax, -2  
  6BC42248    8BF9            mov     edi, ecx  
  6BC4224A    0F84 21C30500   je      6BC9E571  
  6BC42250    83F8 FD         cmp     eax, -3  
  6BC42253    0F84 39C30500   je      6BC9E592  
  6BC42259    83F8 FF         cmp     eax, -1  
  6BC4225C    75 29           jnz     short 6BC42287  
  6BC4225E    BB 11010000     mov     ebx, 111  
  6BC42263    8B07            mov     eax, \[edi\]  
  6BC42265    8BCF            mov     ecx, edi  
  6BC42267    FF50 30         call    \[eax+30\]  
  6BC4226A    8BF0            mov     esi, eax  
  6BC4226C    85F6            test    esi, esi  
  6BC4226E    74 40           je      short 6BC422B0  
  6BC42270    FF75 08         push    dword ptr \[ebp+8\]  
  6BC42273    FF75 0C         push    dword ptr \[ebp+C\]  
  6BC42276    53              push    ebx  
  6BC42277    FF76 04         push    dword ptr \[esi+4\]  
  6BC4227A    E8 09FEFFFF     call    #1145\_AfxFindMessageEntry  
    
  将光标停至 6BC4227A    E8 09FEFFFF     call    #1145\_AfxFindMessageEntry 这句，  
  F4 运行到此处，前一句 6BC42277    FF76 04         push    dword ptr \[esi+4\] 压入栈中的就指向消息映射表的地址。  
    
  看看堆栈：  
    
  0012EBF8   00411814  digi.00411814  
  0012EBFC   00000111  
  0012EC00   FFFFFFFF  
  0012EC04   0000800B  
  0012EC08   FFFFFFFF  
    
  可知我们要找的消息表地址就在 00411814 里，在数据窗口中跟随（地址显示方式）：  
    
  00411814  00411818  digi.00411818  
    
  注意 00411814 指向的 00411818 即为消息表，内容如下：  
    
  00411818  00000112  <-- 消息号  
  0041181C  00000000  <-- Code  
  00411820  00000000  <-- 控件 ID  
  00411824  00000000  <-- 最后一个空间 ID  
  00411828  00000012  <-- 响应函数类型  
  0041182C  00402730  digi.00402730  <-- 响应函数  
  00411830  0000000F  
  00411834  00000000  
  ...  
  00411A4C  00000000  
  00411A50  0000000D  
  00411A54  00404E90  digi.00404E90  <-- 消息表结束于此  
  00411A58  00000000  
  00411A5C  00000000  
  00411A60  00000000  
  00411A64  00000000  
  00411A68  00000000  
  00411A6C  00000000  
  00411A70  0040F612  <jmp.&mfc42.#3597\_CDialog::GetRuntimeClass>  
    
  要处理我们自己的按钮消息，要在表中加入新项，注意到表结束后紧跟其他数据，故没有足够的空间来加入新项。  
  没关系，我们将消息处理表整个移到别处。  
    
  回忆一下 Digi 整容的成果：  
    
                   地址          长度  
       .text       00410000     10000  
       .rdata      00411000      4000  
       .data       00415000    204000  
       .oldrsrc    00619000      3000  
       .perplex    0061C000     1D000  
       .newimt     00639000      1000  
       .rsrc       0063A000      3000  
    
  对了，.perplex 是壳代码所在的段，现在正好用来放我们要加入的东西。  
    
  在数据窗口中选中从 00411818 到 00411A54 的所有内容，然后选择二进制复制。  
    
  将 00411814 的内容改为 0061C518，这就是新消息表的地址。  
  在数据窗口中右键，选择复制到可执行文件，保存下来。  
    
  在数据窗口中来到 0061C518 处，选择二进制粘贴。  
  来到消息表的末尾 0061C784 处：  
    
  0061C778  00000000  
  0061C77C  00000000  
  0061C780  0000000C  
  0061C784  0061CD06  digi.0061CD06  
    
  从 0061C758 开始填充自己的消息：  
    
  0061C758  00000111 <-- 0x111 消息，WM\_COMMAND  
  0061C75C  00000000  
  0061C760  000003ED <-- 3ED 即我们加入的"高级"按钮的资源 ID  
  0061C764  000003ED  
  0061C768  0000000C  
  0061C76C  00000000 <-- 此处应为响应函数地址，先留空  
  0061C770  00000110 <-- 0x110 消息，WM\_INITDIALOG  
  0061C774  00000000  
  0061C778  00000000  
  0061C77C  00000000  
  0061C780  0000000C  
  0061C784  0061CD06  digi.0061CD06 <-- 我们的窗体初始化消息响应写在此处  
  0061C788  00000111  <-- 0x111 消息，WM\_COMMAND  
  0061C78C  00000000  
  0061C790  0000800B  <-- 800B 为我们加入的"高级"菜单项的资源 ID  
  0061C794  0000800B  
  0061C798  0000000C  
  0061C79C  00000000  <-- 响应函数，留空  
  0061C7A0  00000000  
  0061C7A4  00000000  
  0061C7A8  00000000  
  0061C7AC  00000000  
  0061C7B0  00000000  
  0061C7B4  00000000  
    
  选中从 0061C518 到 0061C7B4，复制到可执行文件，保存。  
    
  四、（脱水）第三类接触  
    
  （热到脱水的时候，轻舞飞扬也会变成外星人。注：美女就是美女，永远也变不成恐龙。反之亦然）  
    
  下面我们来加入一个窗体初始化处理函数，来完成 Digi 主窗口初始化的时候，加载我们自己的 dll 和取得函数地址。  
    
  在代码窗口中来到 0061CD06 处，开始写汇编代码：  
    
  0061CD07    68 10CC6100     push    0061CC10                                   ; ASCII "dycdll.dll"  
  0061CD0C    E8 4F511E7C     call    kernel32.LoadLibraryA  
  0061CD11    50              push    eax  
  0061CD12    68 20CC6100     push    0061CC20                                   ; ASCII "\_Mysendto@24"  
  0061CD17    50              push    eax  
  0061CD18    E8 A4F2207C     call    kernel32.GetProcAddress  
  0061CD1D    BB 40CC6100     mov     ebx, 0061CC40  
  0061CD22    8903            mov     \[ebx\], eax  
  0061CD24    58              pop     eax  
  0061CD25    50              push    eax  
  0061CD26    68 30CC6100     push    0061CC30                                   ; ASCII "Mysetting"  
  0061CD2B    50              push    eax  
  0061CD2C    E8 90F2207C     call    kernel32.GetProcAddress  
  0061CD31    BB 6CC76100     mov     ebx, 0061C76C  
  0061CD36    8903            mov     \[ebx\], eax  
  0061CD38    BB 9CC76100     mov     ebx, 0061C79C  
  0061CD3D    8903            mov     \[ebx\], eax  
  0061CD3F    58              pop     eax  
  0061CD40    50              push    eax  
  0061CD41    68 60CC6100     push    0061CC60                                   ; ASCII "Getnotest"  
  0061CD46    50              push    eax  
  0061CD47    E8 75F2207C     call    kernel32.GetProcAddress  
  0061CD4C    BB 50CC6100     mov     ebx, 0061CC50  
  0061CD51    8903            mov     \[ebx\], eax  
  0061CD53    58              pop     eax  
  0061CD54    90              nop  
  0061CD55    90              nop  
  0061CD56    90              nop  
  0061CD57    61              popad  
  0061CD58  - E9 5354DEFF     jmp     004021B0                              ;跳回到原处理函数  
    
  其中用到的变量有：  
    
  0061CC10 - dll 文件名  
  0061CC20 - 我们的 sendto 函数名  
  0061CC30 - 我们的按钮处理函数名  
  0061CC40 - 保存 sendto 函数地址  
  0061CC50 - 保存 Getnotest 函数地址  
  0061CC60 - 我们的 Getnotest 函数名  
    
  0061CC10  dycdll.dll......\_Mysendto@24....  
  0061CC30  Mysetting.......?.............  
  0061CC50  P.............Getnotest.......  
    
  注意到按钮处理函数的地址已经被写到 0061C76C 和 0061C79C 的消息表项中。  
    
  将汇编和变量的修改复制到可执行文件，保存。  
  （和所有的动作游戏一样，请养成随时存档的习惯）  
    
  五、（晕了）真的晕了  
    
  （欢迎来到代码篇）  
    
  下面我们来修改登陆时调用的 sendto 函数，来到 00403ECF 处：  
  00403ECF   .  E8 7CBE0000   call    <jmp.&wsock32.sendto>            ; \\sendto  
    
  修改为  
  00403ECF    E8 E88D2100     call    0061CCCB   ; 0061CCCB 为我们自己的处理函数  
    
  复制到可执行文件，保存。  
    
  来到 0061CCCB 处，开始汇编：  
    
  0061CCCB    A1 40CC6100     mov     eax, \[61CC40\]                             ;其中有我们的 sendto 函数地址  
  0061CCD0    83F8 00         cmp     eax, 0  
  0061CCD3    74 0B           je      short 0061CCE0  
  0061CCD5    90              nop  
  0061CCD6    90              nop  
  0061CCD7    90              nop  
  0061CCD8  - FF25 40CC6100   jmp     \[61CC40\]                                   ; dycdll.Mysendto  
  0061CCDE    90              nop  
  0061CCDF    90              nop  
  0061CCE0  - E9 D4AA5471     jmp     WS2\_32.sendto  
    
  完成的功能就是如果我们自己的 sendto 函数已经加载，就用我们的 sendto 函数工作，否则用系统的 sendto 函数工作。  
    
  复制到可执行文件，保存。  
    
  （喝口水，成功在即了）  
    
  六、（编程）如果上帝是个程序员  
    
  脏活累活干得差不多了，写点优雅的代码吧。  
    
  打开 VC6.0，新建一个 MFC 的 DLL 工程。  
    
  加入如下变量和函数定义。  
    
  //下列变量保存 sendto 发出的包内容  
  int len\_authorize=0;  
  int flags\_authorize=0;  
  struct sockaddr to\_authorize={0,0,0,0,0,0,0,0,0,0,0,0,0,0,0};  
  char buf\_authorize\[100\]={0,0,0,0,0,0,0,0,0,0,  
  0,0,0,0,0,0,0,0,0,0,  
  0,0,0,0,0,0,0,0,0,0,  
  0,0,0,0,0,0,0,0,0,0,  
  0,0,0,0,0,0,0,0,0,0,  
  0,0,0,0,0,0,0,0,0,0,  
  0,0,0,0,0,0,0,0,0,0,  
  0,0,0,0,0,0,0,0,0,0,  
  0,0,0,0,0,0,0,0,0,0,  
  0,0,0,0,0,0,0,0,0,0  
  };  
  int tolen\_authorize=0;  
    
  //这三个变量保存程序运行方式  
  BOOL NoTest=TRUE;     //是否禁止代理检测等  
  BOOL SAVE=FALSE;      //是否保存登陆包  
  BOOL LOAD=FALSE;      //是否调入保存的登陆包（路由器穿透时用）  
    
  void ReadIni()        //读配置文件  
  {  
    NoTest=GetPrivateProfileInt("Setting","NoTest",1,".\\\\dyc.ini");  
    SAVE=GetPrivateProfileInt("Setting","SAVE",0,".\\\\dyc.ini");  
    LOAD=GetPrivateProfileInt("Setting","LOAD",0,".\\\\dyc.ini");  
  }  
    
  void WriteIni()       //写配置文件  
  {  
    CString tmp;  
    tmp.Format("%d",NoTest);  
    WritePrivateProfileString("Setting","NoTest",tmp,".\\\\dyc.ini");  
    tmp.Format("%d",SAVE);  
    WritePrivateProfileString("Setting","SAVE",tmp,".\\\\dyc.ini");  
    tmp.Format("%d",LOAD);  
    WritePrivateProfileString("Setting","LOAD",tmp,".\\\\dyc.ini");  
  }  
    
  void Load()           //调入保存的包  
  {  
    HANDLE t=CreateFile("auth.dat",GENERIC\_READ,FILE\_SHARE\_READ,NULL,OPEN\_EXISTING,NULL,NULL);  
    if(t != INVALID\_HANDLE\_VALUE)  
    {  
      unsigned long xlen;  
    
      ReadFile(t,&len\_authorize,1,&xlen,NULL);  
      ReadFile(t,&flags\_authorize,1,&xlen,NULL);  
      ReadFile(t,&to\_authorize,15,&xlen,NULL);  
      ReadFile(t,&buf\_authorize,100,&xlen,NULL);  
      ReadFile(t,&tolen\_authorize,1,&xlen,NULL);  
    
      CloseHandle(t);  
    }  
    
  }  
    
  void Save()           //保存包  
  {  
    HANDLE t=CreateFile("auth.dat",GENERIC\_WRITE,FILE\_SHARE\_WRITE,NULL,CREATE\_ALWAYS,NULL,NULL);  
    if(t != INVALID\_HANDLE\_VALUE)  
    {  
      unsigned long xlen;  
      WriteFile(t,&len\_authorize,1,&xlen,NULL);  
      WriteFile(t,&flags\_authorize,1,&xlen,NULL);  
      WriteFile(t,&to\_authorize,15,&xlen,NULL);  
      WriteFile(t,&buf\_authorize,100,&xlen,NULL);  
      WriteFile(t,&tolen\_authorize,1,&xlen,NULL);  
    
      CloseHandle(t);  
      }  
  }  
    
  extern "C" \_\_declspec(dllexport) BOOL Getnotest()    //导出函数，得到是否不检测  
  {  
    return NoTest;  
  }  
    
  extern "C" \_\_declspec(dllexport) void Mysetting()    //导出函数，"高级"按钮处理函数，显示一个对话框  
  {  
    AFX\_MANAGE\_STATE(AfxGetStaticModuleState());  
    CAdForm p;  
    p.DoModal();  
  //  MessageBox(NULL,"Ha","ha",MB\_OK);  
  }  
    
  extern "C" \_\_declspec(dllexport) int WINAPI Mysendto(  //导出函数，我们自己的 sendto 函数  
    SOCKET s,                          
    const char FAR \* buf,              
    int len,                           
    int flags,                         
    const struct sockaddr FAR \* to,    
    int tolen                          
  )  
  {  
    int result;  
    
    for(int tt=0;tt<len;tt++)  
    {  
      \*(buf\_authorize+tt)=\*(buf+tt);  
    }  
    len\_authorize=len;  
    tolen\_authorize=tolen;  
    flags\_authorize=flags;  
    strcpy(to\_authorize.sa\_data,to->sa\_data);  
    to\_authorize.sa\_family=to->sa\_family;  
      
    if(SAVE)Save();  
    if(LOAD)Load();  
    
    result=::sendto(s,buf\_authorize,len\_authorize,flags\_authorize,&to\_authorize,tolen\_authorize);  
    return result;  
  }  
    
  然后加入一个对话框，在其中处理对程序工作的设置等。  
    
  编译得到的 dll 文件就可以使用了。  
  通过自己的 sendto 函数，我们可以截获正常情况下的登陆数据包，然后在路由器后使用，就可以穿透路由器，正常登陆了。  
    
  七、（点歌）我点一首歌，不要对唱的  
    
  下面我们来解决最后一个检测代理和 BT 的问题。  
    
  先看看登陆代码：  
    
  0040402C   .  8B15 4C8D6100 mov     edx, \[618D4C\]  
  00404032   .  893D 6C6E4100 mov     \[416E6C\], edi  
  00404038   .  891D 706E4100 mov     \[416E70\], ebx  
  0040403E   .  53            push    ebx                                     ; /lParam  
  0040403F   .  8B82 70030400 mov     eax, \[edx+40370\]                        ; |  
  00404045   .  66:C705 026F4>mov     word ptr \[416F02\], 3                    ; |  
  0040404E   .  A3 746F4100   mov     \[416F74\], eax                           ; |  
  00404053   .  8B4D 20       mov     ecx, \[ebp+20\]                           ; |  
  00404056   .  57            push    edi                                     ; |wParam  
  00404057   .  68 8C040000   push    48C                                     ; |Message = MSG(48C)  
  0040405C   .  51            push    ecx                                     ; |hWnd  
  0040405D   .  FF15 24154100 call    \[<&user32.PostMessageA>\]                ; \\PostMessageA  
  00404063   >  A1 506C4100   mov     eax, \[416C50\]  
    
  登陆成功后一直执行到 0040405D 出会有一个对 PostMessageA 的调用，发送窗口消息，消息代码是 48C。  
  初步估计这个消息的响应函数中会有检测的代码。（后来我们才知道，要相信男人的直觉）  
    
  来到消息表 0061c518，往下看直到：  
    
  0061C660  0000000A  
  0061C664  00404250  digi\_new.00404250  
  0061C668  0000048C  <-- 就是我们要找的 48C  
  0061C66C  00000000  
  0061C670  00000000  
  0061C674  00000000  
  0061C678  0000000A  
  0061C67C  00403AC0  digi\_new.00403AC0  <-- 这个就是消息响应函数了  
  0061C680  00000485  
    
  不多说，在代码窗口中跟到 00403AC0：  
    
  00403AC0   .  64:A1 0000000>mov     eax, fs:\[0\]  
  00403AC6   .  6A FF         push    -1  
  00403AC8   .  68 30024100   push    00410230  
  00403ACD   .  50            push    eax  
  00403ACE   .  64:8925 00000>mov     fs:\[0\], esp  
  00403AD5   .  83EC 64       sub     esp, 64  
  00403AD8   .  66:C705 026F4>mov     word ptr \[416F02\], 3  
  00403AE1   .  C705 DC6E4100>mov     dword ptr \[416EDC\], 1  
  00403AEB   .  6A 00         push    0  
  00403AED   .  6A 00         push    0  
  00403AEF   .  6A 00         push    0  
  00403AF1   .  6A 00         push    0  
  00403AF3   .  51            push    ecx  
  00403AF4   .  68 109C4000   push    00409C10  
  00403AF9   .  E8 E0BB0000   call    <jmp.&mfc42.#1105\_AfxBeginThread>  
  00403AFE   .  A1 306E4100   mov     eax, \[416E30\]  
  00403B03   .  8B48 F8       mov     ecx, \[eax-8\]  
  00403B06   .  85C9          test    ecx, ecx  
  ...  
  00403B56   .  C2 0800       retn    8  
    
  注意到 00403AF9 处的 AfxBeginThread 函数启动了一个线程，线程函数地址为 00409C10，跟进去。  
    
  00409C10   .  6A FF         push    -1  
  00409C12   .  68 4C094100   push    0041094C                                ;  SE 处理程序安装  
  00409C17   .  64:A1 0000000>mov     eax, fs:\[0\]  
  00409C1D   .  50            push    eax  
  00409C1E   .  64:8925 00000>mov     fs:\[0\], esp  
  ...  
  00409CC6   .  A3 606F4100   mov     \[416F60\], eax  
  00409CCB   .  E8 A0150000   call    0040B270            ;<-- 有 call，跟进去发现很短，不是  
  00409CD0   .  83C4 04       add     esp, 4  
  00409CD3   .  8BF0          mov     esi, eax  
  00409CD5   .  E8 264B0000   call    0040E800            ;<-- 又有，跟进去也不是，再往下看  
  00409CDA   .  2B05 44704100 sub     eax, \[417044\]  
  ...  
  00409D8D   .  8D5424 30     lea     edx, \[esp+30\]  
  00409D91   .  891D 346E4100 mov     \[416E34\], ebx  
  00409D97   .  52            push    edx                                     ; /pThreadId  
  00409D98   .  57            push    edi                                     ; |CreationFlags  
  00409D99   .  68 346E4100   push    00416E34                                ; |pThreadParm = digi\_dum.00416E34  
  00409D9E   .  68 10964000   push    00409610                                ; |ThreadFunction = digi\_dum.00409610  
  00409DA3   .  57            push    edi                                     ; |StackSize  
  00409DA4   .  57            push    edi                                     ; |pSecurity  
  00409DA5   .  891D E06E4100 mov     \[416EE0\], ebx                           ; |  
  00409DAB   .  FF15 54104100 call    \[<&kernel32.CreateThread>\]              ; \\CreateThread  
    
  又建了一个线程，太可疑，线程函数时 00409610，跟进去：  
    
  00409610   .  53            push    ebx  
  00409611   .  56            push    esi  
  00409612   .  57            push    edi  
  00409613   .  E8 38FDFFFF   call    00409350            ;<-- 有 call 了，跟进去  
  00409618   .  83F8 01       cmp     eax, 1  
  0040961B   .  75 17         jnz     short 00409634  
    
  跟到 00409350：  
    
  00409350  /$  6A FF         push    -1  
  00409352  |.  68 8B084100   push    0041088B                                ;  SE 处理程序安装  
  00409357  |.  64:A1 0000000>mov     eax, fs:\[0\]  
  ...  
  00409384  |.  E8 27610000   call    <jmp.&mfc42.#540\_CString::CString>  
  00409389  |.  56            push    esi                                     ; /ProcessID  
  0040938A  |.  6A 02         push    2                                       ; |Flags = TH32CS\_SNAPPROCESS  
  0040938C  |.  89B424 440100>mov     \[esp+144\], esi                          ; |  
  00409393  |.  E8 7C690000   call    <jmp.&kernel32.CreateToolhelp32Snapshot>; \\CreateToolhelp32Snapshot  
  00409398  |.  8BF8          mov     edi, eax  
  ...  
  004093C5  |.  8D4C24 0C     lea     ecx, \[esp+C\]  
  004093C9  |.  E8 DC600000   call    <jmp.&mfc42.#860\_CString::operator=>  
  004093CE  |.  68 F05D4100   push    00415DF0                                ;  ASCII "SyGate.exe"  
  004093D3  |.  8D4C24 0C     lea     ecx, \[esp+C\]  
  004093D7  |.  E8 70640000   call    <jmp.&mfc42.#2764\_CString::Find>  
  004093DC  |.  3BC6          cmp     eax, esi  
  004093DE  |.  0F8F C1010000 jg      004095A5  
  004093E4  |>  68 E45D4100   /push    00415DE4                               ;  ASCII "Sygate.exe"  
  004093E9  |.  8D4C24 0C     |lea     ecx, \[esp+C\]  
  004093ED  |.  E8 5A640000   |call    <jmp.&mfc42.#2764\_CString::Find>  
  004093F2  |.  3BC6          |cmp     eax, esi  
  ...  
    
  这样一看就很清楚了，这里创建了系统进程列表，逐项比对，来确定没有使用代理和 BT 程序的。  
    
  所以没错，之前受怀疑的 00403AC0 就是检测函数，我们来动个小小手术：  
    
  从：  
  00403AEB   .  6A 00         push    0  
  00403AED   .  6A 00         push    0  
  00403AEF   .  6A 00         push    0  
  00403AF1   .  6A 00         push    0  
  00403AF3   .  51            push    ecx  
  00403AF4   .  68 109C4000   push    00409C10  
  00403AF9   .  E8 E0BB0000   call    <jmp.&mfc42.#1105\_AfxBeginThread>  
    
  改为：  
    
  00403AEB   .- E9 10932100   jmp     0061CE00  
  00403AF0      90            nop  
  00403AF1   .  6A 00         push    0  
  00403AF3   .  51            push    ecx  
  00403AF4   .  68 109C4000   push    00409C10  
  00403AF9   .  E8 E0BB0000   call    <jmp.&mfc42.#1105\_AfxBeginThread>  
    
  然后在 0061CE00 处汇编：  
    
  0061CE00   /EB 13           jmp     short 0061CE15  
  0061CE02   |6A 00           push    0  
  0061CE04   |6A 00           push    0  
  0061CE06   |6A 00           push    0  
  0061CE08   |6A 00           push    0  
  0061CE0A   |51              push    ecx  
  0061CE0B   |68 109C4000     push    00409C10  
  0061CE10   |E8 C928DFFF     call    <jmp.&mfc42.#1105\_AfxBeginThread>  
  0061CE15  -\\E9 E46CDEFF     jmp     00403AFE  
  0061CE1A    8B1D 50CC6100   mov     ebx, \[61CC50\]          ;61CC50 就是我们的 Getnotest 函数  
  0061CE20    83FB 00         cmp     ebx, 0                 ;返回为 0 就执行检测，否则不执行  
  0061CE23  ^ 74 DD           je      short 0061CE02  
  0061CE25    FFD3            call    ebx  
  0061CE27    83F8 00         cmp     eax, 0  
  0061CE2A  ^ 74 D6           je      short 0061CE02  
  0061CE2C  ^ EB E2           jmp     short 0061CE10  
  0061CE2E    90              nop  
    
  复制到可执行文件，保存。至此，代理和 BT 检测也被破除。  
    
  八、（R&B）最后的战役  
    
  最后，最后我们还要干嘛呢？ 嗯，发现没，Digi 的体重已经变到了惊人的 2.23MB！  
  （虽然我一再说 Size 不重要，可是拥有姚明的脸蛋和李宇春的身材也不是那么的令人茶饭不思吧）  
    
  减肥一下，用 LordPE 打开修改好的 Digi，点击"Rebuild PE"，对话框选择刚刚修改过的 Digi，小灯从红色变为绿色，成功。  
    
  瘦身后的 Digi 回复完美身材 207KB。至此，封锁线完全突破成功！！  
  （\[手捧小金人\] 感谢 CCTV，MTV，Channel-V，感谢我的家人和长期以来支持我的扇子们，我爱你们！！）  
    
\--------------------------------------------------------------------------------  
【经验总结】  
  至此，我们已经彻底的突破了客户端对我们的封锁，天接着很蓝，网继续很宽，资源可以共享，BT 随便可以下。。。  
    
  代码编写中的关键点有：  
   1. 网络发包离不开 wsock32.sendto 函数  
   2. 窗体消息的突破口在 CCmdTarget::OnCmdMsg 函数  
   3. 注意消息表的结构，那就是 mfc 的 MESSAGE\_MAP 宏做的事情  
   4. 加入自己的 dll 用 kernel32.LoadLibraryA 函数载入文件和用 kernel32.GetProcAddress 函数取得函数地址  
   5. 懂一点汇编  
   6. 再懂一点 C  
   7. Size matters, kinda hard to admit though.  
    
  突破封锁线：第四章--代码篇 \[完\]  
    
  谢谢您耐心的看到这里，在下一章--外围篇中，将讲述如何不攻入程序内部而改变程序行为，以及 dll 注入技术。  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 05 月 13 日 2:42:37 

【文章标题】: 突破封锁线：第五章--外围篇  
【文章作者】: Austin  
【作者邮箱】: austiny.cn@gmail.com  
【软件名称】: 神州数码网络客户登陆程序  
【软件大小】: 不重要  
【下载地址】: 自己搜索下载  
【加壳方式】: UltraProtect 1.x  
【保护方式】: 加壳 网络封禁  
【编写语言】: VC6.0  
【使用工具】: VC6.0  
【操作平台】: Windows2003  
【软件介绍】: 神州数码小区宽带网络登陆客户端，诸多限制  
【作者声明】: 技术交流中。。。  
\--------------------------------------------------------------------------------  
【详细过程】  
  网络采用了神州数码公司的网络管理产品。主要弊端是：1.封禁代理；2.封禁路由器；3.封禁 BT。  
  （以下省略声讨文字若干），总之，有压迫的地方就有反抗。让我们来解决它。  
    
  一、知己知彼  
    
  神州数码采用了 802.1x 的认证方式，即认证流于数据流分开，  
  这样降低了认证服务器的负担，提高了网络通讯效率，也为我们破解提供了方便。  
    
  下面描述一下正常情况下，使用客户端登陆的流程：  
    1.打开客户端（废话）  
    2.选择网卡，申请服务（此时客户端会向认证服务器发送第一个数据包，我们叫它服务包）  
    3.填写正确的用户名和密码，登陆（此时客户端会向服务器发送第二个数据包，认证包）  
    4.此后每隔三十秒，客户端会向服务器发送保持登陆的数据  
     （这就是第三个包，连接包，根据收到的反馈信息，由客户端会随机产生）  
    5.在客户端运行过程中，它会不断检验代理和 BT 软件的运行，一旦发现，主动断开连接。  
    
  有上述分析，可以看到，路由器的封禁是在第 3 步实现的（通过读取网卡信息），而代理和 BT 的封禁在第 5 步实现。  
    
  二、弱点分析  
    
    1.在认证方式的设计中，一旦登陆成功，其连接包将不会改变，  
      所以只要截下此包，以三十秒的周期向服务器发送，便可以强行关闭客户端，其代理和 BT 封禁便迎刃而解。  
    2.对于路由器的封锁稍微麻烦些，关键是正确登陆信息的取得和发出，和保证客户端算出对应的连接信息，  
      所以在此过程中，不可以抛开客户端。  
    
  三、作战部署  
    
  由前面的分析可以看到，破解的核心在于截包（网络嗅探）。  
  通常的编程方法是利用 wincap 等开发库监视底层通讯，或者将网卡置于混杂模式，过滤通过其所有数据。  
  但这些方法太面向底层，复杂且开销过大。杀鸡焉用牛刀？  
    
  对付小小的神州数码，决定采用一种较为轻便优雅的方法----API 拦截（或叫 API 钩子）。  
  因为在 Windows 平台上，程序的功能最终都是通过调用 Windows API 来实现的。  
  如这里的网络通讯，收发都使用了 UDP 协议，典型的由 Windows API 函数 sendto 和 recvfrom 来实现。  
  所以截获了这两个函数，就能轻松完成截包工作。  
  （这样，我们的任务就和大多数网游外挂要完成的任务相仿了）  
    
  不过，要截获 API 函数，也不是一件简单的工作。  
  因为 API 钩子的编写不像键盘钩子（用于偷密码），鼠标钩子（察看\*密码、屏幕取词等）有现成的钩子函数可以使用，  
  它是一种非常规编程技术。说白了，就是要我们自己的代码在别的程序的空间内运行。  
  我们知道，在 Windows 平台下，使用了虚拟地址空间技术将不同进程的代码和数据分离，  
  但 dll 文件在整个内存中是共享的，所以，通过使目标进程加载我们自己编写的 dll 文件，  
  就有可能让自己的代码在目标进程中执行，这就是所谓的 dll 注入技术。  
    
  四、瞒天过海  
    
  我们提到要使用 dll 注入技术来完成我们的工作。  
  幸好，Windows API 中为我们提供了一个美妙的函数 CreateRemoteThread，恰好可以为我们所用。  
  这个函数用来在目标进程中运行一个线程，而如果这个线程所完成的任务就是加载我们的 dll 文件，  
  那注入的任务不就正好搞定了吗？  
  但是还有一个问题，我们可以让目标进程运行 LoadLibraryA 这个函数，因为它在每个进程空间里的地址都是一样的，  
  但我们的 dll 路径如何传给目标呢？  
  不用急，Windows 也为我们准备好了，  
  VirtualAllocEx 和 WriteProcessMemory 函数正好完成了在目标进程里内存分配和写入数据的任务。  
  这样，只要三个函数配合使用，目标进程就乖乖的加载好了我们的 dll。  
    
  BOOL WINAPI InjectLib(DWORD dwProcessId, LPCSTR strLibFile)  
  {  
    BOOL result=FALSE;  
    HANDLE hProcess=NULL, hThread=NULL;  
    LPCSTR strLibFileRemote=NULL;  
    \_\_try  
    {  
      hProcess=OpenProcess(  
        PROCESS\_CREATE\_THREAD|  
        PROCESS\_VM\_OPERATION |  
        PROCESS\_VM\_WRITE,  
        FALSE,dwProcessId);  
      if(hProcess==NULL)\_\_leave;  
    
      int cch=1+strlen(strLibFile);  
      int cb= cch\*sizeof(CHAR);  
    
      strLibFileRemote=(LPCSTR)  
        VirtualAllocEx(hProcess,NULL,cb,MEM\_COMMIT,PAGE\_READWRITE);  
      if(strLibFileRemote==NULL)\_\_leave;  
    
      if(!WriteProcessMemory(hProcess,(PVOID)strLibFileRemote,  
        (PVOID)strLibFile,cb,NULL))\_\_leave;  
    
      PTHREAD\_START\_ROUTINE pfnThreadRtn=(PTHREAD\_START\_ROUTINE)  
        GetProcAddress(GetModuleHandle("Kernel32"),"LoadLibraryA");  
      if(pfnThreadRtn==NULL)\_\_leave;  
    
      hThread=CreateRemoteThread(hProcess,NULL,0,pfnThreadRtn,(PVOID)strLibFileRemote,0,NULL);  
      if(hThread==NULL)\_\_leave;  
    
      WaitForSingleObject(hThread,INFINITE);  
    
      result=TRUE;  
    }  
    \_\_finally  
    {  
      if(strLibFileRemote!=NULL)  
        VirtualFreeEx(hProcess,(PVOID)strLibFileRemote,0,MEM\_RELEASE);  
      if(hThread!=NULL)  
        CloseHandle(hThread);  
      if(hProcess!=NULL)  
        CloseHandle(hProcess);  
    }  
    return result;  
  }  
    
    
  注意，现在只是加载，并没有运行。不过我们知道，每次 dll 被加载时，它的 DllMain 函数都会被调用，  
  是的，聪明的你应该已经想到，我们的劫持代码就是在这儿被调用的。  
    
  BOOL APIENTRY DllMain( HANDLE hModule,   
              DWORD  ul\_reason\_for\_call,   
              LPVOID lpReserved  
              )  
  {  
    switch(ul\_reason\_for\_call)  
    {  
    case DLL\_PROCESS\_ATTACH:  
      {  
        //MessageBox(NULL,"Attached to Process","Caution",MB\_OK);  
        WORD wVersionRequested;  
        WSADATA wsaData;  
        int err;  
         wVersionRequested = MAKEWORD(2,2);  
     
        err = WSAStartup( wVersionRequested, &wsaData );  
        if(err)MessageBox(NULL,"WSAStartup Error","Error",MB\_OK);  
    
        s=socket(AF\_INET,SOCK\_DGRAM,0);  
        MyFunc();     //这里面就是我们的劫持代码  
      }  
    case DLL\_PROCESS\_DETACH:  
      {  
        //MessageBox(NULL,"Detached from Process","Caution",MB\_OK);  
      }  
    case DLL\_THREAD\_ATTACH:  
      {  
        //MessageBox(NULL,"Attached to Thread","Caution",MB\_OK);  
      }  
    case DLL\_THREAD\_DETACH:  
      {  
        //MessageBox(NULL,"Detached from Thread","Caution",MB\_OK);  
      }  
    }  
      return TRUE;  
  }  
    
    
  下面来关注劫持的问题。大家知道，Windows 的每个进程都会维护一张虚拟函数地址表。  
  程序显示链接的函数地址都保存在这张表中。  
  所以，只要修改了这张表，就可以使得对 API 函数的调用编程对我们自己函数的调用（amazing, huh?）。  
  于是话不多说，改！  
  可是，我们会郁闷的发现，由于加壳的缘故，  
  神州数码的客户端并没有使用显示链接到 SendTo 函数（就是我们要劫持的那个，记得吗），  
  而是用 LibraryA 函数动态加载。所以，修改虚拟函数的方法失效了。  
    
  真的没办法了吗？当然不是。不然我还写这篇东西干吗呢？  
  凡走过比留下痕迹。改还是要改的，不过不改 SendTo 了，而是改 GetProcAddress。  
  因为动态加载通过它来返回函数地址，只要我们把对 SendTo 的加载返回成自己函数的地址，瞒天过海之计就成了。  
    
  extern "C" \_\_declspec(dllexport) void MyFunc()  
  {  
    HOOKAPI a;  
    LPHOOKAPI pHookApi=&a;  
    HMODULE hm1=(HMODULE)0x00400000;  
    
    pHookApi->szFunc=(LPCSTR)"GetProcAddress";  
    pHookApi->pNewProc=(PROC)MyGetProcAddress;  
    HookAPIByName(hm1,"kernel32.dll",pHookApi);  
        
    pHookApi->szFunc=(LPCSTR)"LoadLibraryA";  
    pHookApi->pNewProc=(PROC)MyLoadLibrary;  
    HookAPIByName(hm1,"kernel32.dll",pHookApi);  //这里劫持了 LoadLibraryA 函数  
  }  
    
  typedef struct tag\_HOOKAPI   
  {   
    LPCSTR szFunc;//待 HOOK 的 API 函数名   
    PROC pNewProc;//新的函数指针   
    PROC pOldProc;//老的函数指针   
  }HOOKAPI, \*LPHOOKAPI;   
    
  PIMAGE\_IMPORT\_DESCRIPTOR GetNamedImportDescriptor(HMODULE hModule, LPCSTR szImportMod)   
  {   
    //首先是 DOS 头   
    PIMAGE\_DOS\_HEADER pDOSHeader = (PIMAGE\_DOS\_HEADER) hModule;   
    if(pDOSHeader->e\_magic != IMAGE\_DOS\_SIGNATURE) return NULL;   
    PIMAGE\_NT\_HEADERS pNTHeader = (PIMAGE\_NT\_HEADERS)((DWORD)pDOSHeader + (DWORD)(pDOSHeader->e\_lfanew));   
    if(pNTHeader->Signature != IMAGE\_NT\_SIGNATURE) return NULL;   
    //如果没有 Import 部分,返回失败   
    if(pNTHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\].VirtualAddress == 0)   
      return NULL;   
    //取 Import 部分   
    PIMAGE\_IMPORT\_DESCRIPTOR pImportDesc = (PIMAGE\_IMPORT\_DESCRIPTOR)   
      ((DWORD)pDOSHeader + (DWORD)(pNTHeader->OptionalHeader.   
      DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\].VirtualAddress));   
    //寻找与 szImportMod 相配部分   
    while (pImportDesc->Name)   
    {   
      PSTR szCurrMod = (PSTR)((DWORD)pDOSHeader + (DWORD)(pImportDesc->Name));   
      if (stricmp(szCurrMod, szImportMod) == 0)   
        break; //找到   
      pImportDesc++;   
    }   
    if(pImportDesc->Name == NULL) return NULL;   
    return pImportDesc;   
  }   
    
  HookAPIByName(HMODULE hModule/\*被 HOOK 的目标进程 MODULE\*/, LPCSTR szImportMod/\*如 GDI32.DLL\*/,LPHOOKAPI pHookApi/\*指定函数  
  名,如"MessageBoxW"\*/)   
  {   
    PIMAGE\_IMPORT\_DESCRIPTOR pImportDesc =   
      GetNamedImportDescriptor(hModule, szImportMod);   
    if (pImportDesc == NULL)   
      return FALSE; //需要改换的 API 不能取到正确描  
    PIMAGE\_THUNK\_DATA pOrigThunk = (PIMAGE\_THUNK\_DATA)((DWORD)hModule + (DWORD)(pImportDesc->OriginalFirstThunk));   
    PIMAGE\_THUNK\_DATA pRealThunk =   
      (PIMAGE\_THUNK\_DATA)((DWORD)hModule + (DWORD)(pImportDesc->FirstThunk));   
    while(pOrigThunk->u1.Function)   
    {   
      if((pOrigThunk->u1.Ordinal & IMAGE\_ORDINAL\_FLAG) != IMAGE\_ORDINAL\_FLAG)   
      {   
        PIMAGE\_IMPORT\_BY\_NAME pByName = (PIMAGE\_IMPORT\_BY\_NAME)((DWORD)hModule + (DWORD)  
  (pOrigThunk->u1.AddressOfData));   
        if(pByName->Name\[0\] == '\\0')   
          return FALSE; //失败   
        if(strcmpi(pHookApi->szFunc, (char\*)pByName->Name) == 0)   
        {   
          //改变 thunk 保护属性   
          MEMORY\_BASIC\_INFORMATION mbi\_thunk;   
          VirtualQuery(pRealThunk, &mbi\_thunk, sizeof(MEMORY\_BASIC\_INFORMATION));   
          VirtualProtect(mbi\_thunk.BaseAddress,mbi\_thunk.RegionSize, PAGE\_READWRITE,   
  &mbi\_thunk.Protect);   
          //保存原来的 API 函数指针   
          if(pHookApi->pOldProc == NULL)   
            pHookApi->pOldProc = (PROC)pRealThunk->u1.Function;   
          //改变 API 函数指针   
          pRealThunk->u1.Function = (PDWORD)pHookApi->pNewProc;   
          //将 thunk 保护属性改回来   
          DWORD dwOldProtect;   
          VirtualProtect(mbi\_thunk.BaseAddress, mbi\_thunk.RegionSize,   
            mbi\_thunk.Protect, &dwOldProtect);   
        }   
      }   
      pOrigThunk++;   
      pRealThunk++;   
    }   
    SetLastError(ERROR\_SUCCESS);   
    return TRUE;   
  }   
    
    
  五、实战演练  
    
  现在万事俱备，只欠一个我们反复提到的----自己的函数就可以完成全部的任务了。  
  那这个自己的函数要做些什么呢？其实很简单，就是把通过它发送的数据包一一记录下来备用即可。  
  而记录的数据，将要被我们自己的程序所使用，所以存在 dll 的共享数据区里面。  
  这是其一，对于解除代理和 BT 的封禁，这已经足够了。  
  但要解决路由器的问题，还需要再加一点功能。  
  我们会把正确的不带路由器的认证包（记为 A 包）存起来，然后在有路由器环境下，  
  客户端会产生指示使用了路由器的认证包（记为 B 包），用 A 包代替 B 包发送出去，就起到了欺骗服务器的作用，  
  解除了路由器的封禁。  
    
  static int WINAPI Mysendto (  
    SOCKET s,                          
    const char FAR \* buf,              
    int len,                           
    int flags,                         
    const struct sockaddr FAR \* to,    
    int tolen                          
  )  
  {  
    int ii=0;  
    if(len!=26)  
    {  
      if(statue\_client==0)  
      {  
        if(stat\_authorize==0)  
        {  
          for(int tt=0;tt<len;tt++)  
          {  
            \*(buf\_authorize+tt)=\*(buf+tt);  
          }  
          len\_authorize=len;  
          tolen\_authorize=tolen;  
          flags\_authorize=flags;  
          strcpy(to\_authorize.sa\_data,to->sa\_data);  
          to\_authorize.sa\_family=to->sa\_family;  
          stat\_authorize=1;  
          ii=::sendto(s,buf,len,flags,to,tolen);  
        }  
        else  
        {  
          ii=::sendto(s,buf\_authorize,len\_authorize,flags\_authorize,&to\_authorize,tolen\_authorize);  
        }  
        statue\_client=1;  
      }  
      else if(statue\_client==1)  
      {  
        statue\_client=2;  
        for(int tt=0;tt<len;tt++)  
        {  
          \*(buf\_keepconn+tt)=\*(buf+tt);  
        }  
        len\_keepconn=len;  
        tolen\_keepconn=tolen;  
        flags\_keepconn=flags;  
        strcpy(to\_keepconn.sa\_data,to->sa\_data);  
        to\_keepconn.sa\_family=to->sa\_family;  
        stat\_keepconn=1;  
        statue\_client=3;  
        ii=::sendto(s,buf,len,flags,to,tolen);  
      }  
    }  
    else  
    {  
      for(int tt=0;tt<len;tt++)  
      {  
        \*(buf\_getservice+tt)=\*(buf+tt);  
      }  
      len\_getservice=len;  
      tolen\_getservice=tolen;  
      flags\_getservice=flags;  
      strcpy(to\_getservice.sa\_data,to->sa\_data);  
      to\_getservice.sa\_family=to->sa\_family;  
      stat\_getservice=1;  
      ii=::sendto(s,buf,len,flags,to,tolen);  
    }  
    return ii;  
  }  
    
    
  六、屠城三日  
    
  至此，神州数码的三大封禁已基本被我们解决，  
  剩下来的工作，就是愉快的上网，愉快的用代理，愉快的用路由器，愉快的 BT 了。  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 05 月 13 日 3:00:40 

引用:  *最初由** dezone **发布*  
好文！支持  
  
另外，想问问楼主一个关于 dll 注入的问题，有时候一些程序是在启动时调用相关 API 的(如 findwindow)，我们的 dll 还没来的及注入，它就已经调用了该 api 了，本来想 hook 这些 api 的，但是迟了，请问有什么办法可以以最快的速度把 dll 注入其它程序呢？  
另：用 CreateProcess 插入代码应该可以，哪有例子看看啊。  

  
  
这个函数配合 InjectLib 函数可以完成这个功能：  
  
int APIENTRY WinMain(HINSTANCE hInstance,  
                     HINSTANCE hPrevInstance,  
                     LPSTR     lpCmdLine,  
                     int       nCmdShow)  
{  
LPPROCESS\_INFORMATION p=new \_PROCESS\_INFORMATION;  
STARTUPINFO ss=  
{  
  sizeof(STARTUPINFO),0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0};  
  BOOL b=::CreateProcess(EXEPATH,"",NULL,NULL,TRUE,CREATE\_SUSPENDED,NULL,NULL,&ss,p);  
  DWORD err;  
  if(!b)  
  {  
   err=GetLastError();  
   return -1;  
  }  
  InjectLib(p->dwProcessId,DLLPATH);  
  ::ResumeThread(p->hThread);  
  return 0;  
} 

引用:  *最初由** austiny **发布*  
这个函数配合 InjectLib 函数可以完成这个功能：  
  
int APIENTRY WinMain(HINSTANCE hInstance,  
                     HINSTANCE hPrevInstance,  
                     LPSTR     lpCmdLine,  
........  

  
感谢 austiny 兄的回复。（不好意思，才上看雪，刚看到）  
不过这个使用了 CreateRemoteThread，只能在 2k,xp 以上系统下使用，98 下不能使用。  
  
另外，《windows 核心编程》中有提到用 CreateProcess 插入代码的方法来实现 dll 的注入（最后提到的一种方法，这种方法的好处是 98,2k,xp 均可），但却没有给出代码，只给出大概的思路，我做了一下没成功，网上也找不到相关例子，不知道哪位兄弟做过。 

 4dd466d0e1e0cc123afbe504da2462ca