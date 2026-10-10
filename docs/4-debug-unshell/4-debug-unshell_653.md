

# 菜鸟脱 ExeCryptor 的笔记【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-626.htm`](https://www.pediy.com/kssd/pediy08/pediy8-626.htm)

【文章标题】: 菜鸟脱 ExeCryptor 的笔记  
【文章作者】: machenglin  
【软件名称】: pm  
【软件大小】: 1.9MB  
【下载地址】: 自己搜索下载  
【加壳方式】: ExeCryptor  
【保护方式】: ExeCryptor  
【编写语言】: Borland Delphi  
【使用工具】: OllyICE、PETools、ImportREC  
【操作平台】: winxp sp2  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
    
  首先感谢 ExeCryptor 脚本的作者。  
    
  感谢参考学习 ExeCryptor 教程的作者。  
    
  OD 载入。  
    
  7C921231    C3              retn                                 ; 系统断点。  
  7C921232    8BFF            mov     edi, edi  
  7C921234    90              nop  
  7C921235    90              nop  
  7C921236    90              nop  
  7C921237    90              nop  
  7C921238    90              nop  
  7C921239 >  CC              int3  
  7C92123A    C3              retn  
  7C92123B    90              nop  
  7C92123C    8BFF            mov     edi, edi  
  ---------------------------------------------------------------------------------------------------  
  一、找到 OEP、转存。  
    
  运行脚本 Bypass AntiDBG OEP，出现几次异常，Shift+F9 通过。  
    
  0079F7BA    E8 038DEDFF     call    006784C2                     ; 停在伪 OEP。  
  0079F7BF    0D 71C08494     or      eax, 9484C071  
  0079F7C4    07              pop     es  
  0079F7C5    40              inc     eax  
  0079F7C6    63E9            arpl    cx, bp  
  0079F7C8    51              push    ecx  
  0079F7C9    97              xchg    eax, edi  
  0079F7CA    ED              in      eax, dx  
  0079F7CB    FFE8            jmp     far eax                      ; 非法使用寄存器  
    
  脚本结束。  
  ---------------------------------------------------------------------------------------------------  
  查 code 段 00401000+23E000=63F000。  
    
  shoooo 大侠告诉我们：  
  “delphi 直接到初始化表的下一行去看，原 oep 处改成了 jmp XXXX, XXXX 可作 oep”。  
    
  0063E9A0  - E9 150E1600     jmp     <模块入口点>                  ; OEP！！！  
  0063E9A5    81E0 CEF1E9B3   and     eax, B3E9F1CE  
  0063E9AB    8BD0            mov     edx, eax  
  0063E9AD    E8 DEF41100     call    0075DE90  
  0063E9B2    335F 0C         xor     ebx, \[edi+C\]  
  0063E9B5    D95B 0B         fstp    dword ptr \[ebx+B\]  
  0063E9B8    B7 BD           mov     bh, 0BD  
  0063E9BA    0BD8            or      ebx, eax  
  0063E9BC  - E9 3F360200     jmp     00662000  
  0063E9C1    D5 E8           aad     0E8  
  0063E9C3    8AD2            mov     dl, dl  
  0063E9C5    53              push    ebx  
  0063E9C6    EC              in      al, dx  
  0063E9C7    3A7C90 6A       cmp     bh, \[eax+edx\*4+6A\]  
  0063E9CB    8424F5 E5D8176A test    \[esi\*8+6A17D8E5\], ah  
  0063E9D2    006A 00         add     \[edx\], ch  
  0063E9D5    68 A4EA6300     push    0063EAA4                         ; ASCII "AQB2005"  
    
  用 PETools 转存为 dumped.exe。  
    
  在 0063E9A0 新建 EIP。  
  ====================================================================================================  
  二、解密 IAT，修复转存文件。  
    
  用 LordPE 查看，一般 EXECryptor 壳，BBS 区段下面的 0eqizouf 区段就是 IAT 表地址，当然，这个区段名由人随便命名  
  ，但在 BBS 区段下面一个区段没有疑问。  
    
  0eqizouf=0025C000。在 OD 数据窗口 Ctrl+G：65C000。  
    
  注意，EXECryptor 加密的 IAT 函数用：文件名.xxxxxxxx 表示，“x”是数字。  
    
  向下看：  
    
  0065C1DC  00000000  
  0065C1E0  007C731F  prj\_safe.007C731F                            ; 加密的 IAT，IATstart。  
  0065C1E4  007DC743  prj\_safe.007DC743  
  0065C1E8  00678B15  prj\_safe.00678B15  
  ......  
  0065C2C0  00760A9E  prj\_safe.00760A9E  
  0065C2C4  00000000  
  0065C2C8  770F4850  oleaut32.SysFreeString                       ; 未加密的 IAT  
  0065C2CC  7711C99D  oleaut32.SysReAllocStringLen  
  0065C2D0  770F4B59  oleaut32.SysAllocStringLen  
  0065C2D4  00000000  
  0065C2D8  00674239  prj\_safe.00674239  
  ......  
  0065CBB8  71A23E00  WS2\_32.bind  
  0065CBBC  00000000  
  0065CBC0  01FD1F70  sys\_aq.get\_key\_password                      ; IATend。    
    
  ----------------------------------------------------------------------------------------------------  
  打开 ExeCryptor 2.xx IAT Rebuilder v1.1 脚本编辑。  
    
  mov IATstart,0065C1E0  
  mov IATend,0065CBC0                  
    
  保存修改，运行 ExeCryptor 2.xx IAT Rebuilder v1.1 脚本。  
    
  这时可打开脚本运行窗口观察恢复情况。  
    
  在恢复到 0065C258 出现异常，记住这个地址，下次修改用！终止脚本。  
    
  注意，最好关闭文件，清理 UDD 全部。  
    
  重新开始，直到伪 OEP。  
    
  在反汇编窗口 Ctrl+G：0063E9A0(OEP)，在 0063E9A0 新建 EIP。  
  在数据窗口 Ctrl+G：0065C258，修改为 00000000。  
    
  运行 ExeCryptor 2.xx IAT Rebuilder v1.1 脚本。  
    
  脚本结束后，根据 0065C258 的函数地址修改为 kernel32.FreeLibrary。  
    
  再用 ImportREC，OEP=0023E9A0 RVA=0025C1E0，修复转存文件。  
  ====================================================================================================  
  三、修复 OEP。  
    
  参考其他 Borland Delphi 文件修改 OEP。  
    
  这个软件中有一个 Borland Delphi 文件 Project1，没有加密。  
  ----------------------------------------------------------------------------------------------------  
  0048FF74 >  55              push ebp  
  0048FF75    8BEC            mov ebp,esp  
  0048FF77    83C4 F0         add esp,-10  
  0048FF7A    53              push ebx  
  0048FF7B    B8 ACFC4800     mov eax,Project1.0048FCAC  
  0048FF80    E8 F760F7FF     call Project1.0040607C                 ; F7 跟随进入。  
    
  来到这里。  
  0040607C    53              push ebx  
  0040607D    8BD8            mov ebx,eax  
  0040607F    33C0            xor eax,eax  
  00406081    A3 9C104900     mov dword ptr ds:\[49109C\],eax  
  00406086    6A 00           push 0  
  00406088    E8 2BFFFFFF     call <jmp.&kernel32.GetModuleHandleA>   ; 多熟悉的函数！  
  ----------------------------------------------------------------------------------------------------  
  修改 OEP：  
  0063E9A0 >  55              push ebp  
  0063E9A1    8BEC            mov ebp,esp  
  0063E9A3    83C4 F0         add esp,-10  
  0063E9A6    53              push ebx  
  0063E9A7    B8 30E26300     mov eax,AQTB\_01.0063E230              ; 这里修改看下面②！  
  0063E9AC    E8 FB86DCFF     call    004070AC                      ; 这里修改看下面①！  
  0063E9B1    8B1D C0E26300   mov     ebx, \[63E2C0\]                 ; AQTB\_2.0043508C 看后面！  
  0063E9B7    68 A4EA6300     push    0063EAA4                      ; ASCII "AQB2005"  
  0063E9BC    6A 00           push    0  
  0063E9BE    6A 00           push    0  
  0063E9C0    E8 6326DDFF     call    00411028  
  0063E9C5    E8 BE27DDFF     call    00411188  
  0063E9CA    3D B7000000     cmp     eax, 0B7  
  0063E9CF    75 20           jnz     short 0063E9F1  
  0063E9D1    6A 00           push    0  
  0063E9D3    6A 00           push    0  
  0063E9D5    68 A4EA6300     push    0063EAA4                      ; ASCII "AQB2005"  
  0063E9DA    E8 DD95DCFF     call    <jmp.&user32.RegisterClipboar>  
  ---------------------------------------------------------------------------------------------------  
  ①在 dumped\_.exe 的 OEP 附近搜索：  
    
  push ebx  
  mov ebx,eax  
  xor eax,eax  
    
  搜索到这里就是了。  
  004070AC    53              push ebx  
  004070AD    8BD8            mov ebx,eax  
  004070AF    33C0            xor eax,eax  
  004070B1    A3 A4F06300     mov dword ptr ds:\[63F0A4\],eax  
  004070B6    6A 00           push 0  
  004070B8    E8 2BFFFFFF     call <jmp.&kernel32.GetModuleHandleA>  
    
  从 OEP 开始进入第一个 call 的入口地址这样就确定为 004070AC。  
    
  ②确定 0063E9A7    B8 30E26300     mov eax,AQTB\_01.00xxxxxx。  
  确定了 OEP 开始进入第一个 call 的入口地址后，F7 进入跟踪。  
  现在另开 OD 打开用没有修复 OEP 的 dumped\_.exe,在反汇编窗口 Ctrl+G：004070AC。  
  在 004070AC 设内存访问断点，Shift+F9，运行。  
  出现注册窗口，点“试用”，断下。  
    
  看积存器：eax=0063E230  
    
  为确认继续，F8 步进到这里。  
  004070E6    E8 4DD5FFFF     call    00404638                         ; F7 进入。  
  004070EB    5B              pop     ebx  
    
  F8 步进到这里。  
  00404672    E8 61FFFFFF     call    004045D8                         ; F7 进入。  
  00404677    C3              retn  
    
  004045D8    55              push    ebp  
  004045D9    8BEC            mov     ebp, esp  
  004045DB    53              push    ebx                    ; AAAA\_.0063E230 这里入栈了。  
  004045DC    56              push    esi  
  004045DD    57              push    edi  
  004045DE    A1 40966400     mov     eax, \[649640\]  
  004045E3    85C0            test    eax, eax  
  004045E5    74 4B           je      short 00404632  
  004045E7    8B30            mov     esi, \[eax\]             ; 注意\[eax\]地址！  
    
  ds:\[0063E230\]=000000ED  
    
  数据窗口中定位：  
  0063E230  000000ED  
  0063E234  0063E238  AAAA\_.0063E238  
  0063E238  00407120  AAAA\_.00407120  
  0063E23C  004070F0  AAAA\_.004070F0  
  0063E240  00406F14  AAAA\_.00406F14  
  0063E244  00406EC0  AAAA\_.00406EC0  
    
  得到：  
  0063E9A7    B8 30E26300     mov eax,0063E230  
    
  ====================================================================================================  
  四、初始化修改。  
    
  修复 OEP 变形代码，完成初始化过程中，又出现内存异常，只能到启动界面，这个问题很费了一番周折，主要是基  
  础太差的原因造成。  
    
  也好，拿出没有修复 OEP 的 dumped\_.exe，对照在第一个 call 后补的代码行：  
    
  0063E9AC    E8 FB86DCFF     call    004070AC                      ; 这里修改看下面①！  
  0063E9B1    8B1D C0E26300   mov     ebx, \[63E2C0\]                 ; AQTB\_2.0043508C  
    
  经动态跟踪，没有修复 OEP 的 dumped\_.exe 的是：  
    
  00634D90    E8 1723DDFF     call prj\_safe.004070AC  
  00634D95    8B1D C0E26300   mov ebx,dword ptr ds:\[63E2C0\]         ; prj\_safe.0063FC14  
    
  修改：  
  0063E9AC    E8 FB86DCFF     call    004070AC                      ; 这里修改看下面①！  
  00634D95    8B1D C0E26300   mov ebx,dword ptr ds:\[63E2C0\]         ; prj\_safe.0063FC14  
    
  004070AC=prj\_safe.004070AC  
    
  保存，运行。到了“用户登陆”界面，测试正常。  
    
  ====================================================================================================  
  五、文件优化。  
    
  这个软件未脱壳前为 1.95 MB，想当然的把 3 个壳区断删除，重建 PE。  
  运行后问题出来了，键入用户名后，又跳出一个口令窗口，我没有设密码呀！？  
  载入未优化的软件，在壳的 3 个区段 F2，断在一个壳区段，只得将这个区段重新装配进去。  
  启动脱壳软件，运行畅快了。  
  就是文件字节还是有些大，为 3.9MB，还是基础太差，汗！  
    
  ====================================================================================================  
  六、脱壳 ExeCryptor 小结  
    
  这是纯属个人理解，千万别误导你的正确思路。  
    
  1、必须有一个能反反跟踪的 OD，建议用 OllyICE。  
  2、转存时 PETools 比 LordPE 成功率高。  
  3、未加壳前文件的 TLS 在 idata 区段头。  
  4、ExeCryptor 加密的 IAT 区段在 BBS 段下面的一个区段(Borland Delphi)。  
  5、脱壳前准备一个同语言的参考文件，最好是本软件文件内的。  
  6、保留一个 ImportREC 修复的最初文件，修复参考用。  
  7、有些 SDK 是存在壳区段，优化时要注意。  
    
  本人是个菜鸟，不是很懂编程语言，作为脱壳过程中走过的失败，给像我一样的初哥们参考。  
    
   
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 10 月 18 日 22:45:42

 4dd466d0e1e0cc123afbe504da2462ca