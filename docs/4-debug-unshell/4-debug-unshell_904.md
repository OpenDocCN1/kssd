

# PESpin V0.3

> 原文：[`www.pediy.com/kssd/pediy06/pediy6928.htm`](https://www.pediy.com/kssd/pediy06/pediy6928.htm)

【脱文标题】 PESpin V0.3 怒而脱之  
   
【脱文作者】 weiyi75\[Dfcg\]  
   
【作者邮箱】 weiyi75@sohu.com  
   
【作者主页】 Dfcg 官方大本营  
   
【使用工具】 Peid,olldbg,Loadpe,ImportREC  
   
【破解平台】 Win2000/XP  
   
【软件名称】 PESpin V0.3  
    
【软件下载 】 点击此处本地下载  
   
【软件大小】 19k  
   
【加壳方式】 PESpin 0.3 -> cyberbob  
   
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
   
\--------------------------------------------------------------------------------  
【脱壳内容】  
   
看着 PESpin V0.7-1.1 都被 Fly,loveboom 拿下，我这里 N 久下载的 PESpin V0.3 还没有搞定，主要是搞不定 IAT 加密，IAT 知识还不全，但不愿意卡住，想办法吧。  
  
苦思好久，Magic Jmp 在 PESpin0.1 被 fly 攻下作者就更变态的隐藏住了，或者根本没有 Magic Jmp 代码，可以自己写解密 IAT 代码嘛，我是不会的。但可以利用作者的代码的，看加密选项有个 IAT 重定项，这就好办了，找到它们的区别，PATCH 不就可以了，看清规律，这个可以处理一些变态壳，一般软件作者当然是全部 Anti 选项了，没料到免费的东西我们脱壳者也有的啊。分别用 PESpin 加密记事本(全部 Anti 选项),加密记事本(全部 Anti 选项除了不加密 IAT)两个记事本。  
  
【详细内容】  
  
OD 选项不忽略特权指令，其余全部忽略。  
  
再特权指令 SEH 异常后就开始处理 IAT,然后到 Stolen code  
  
OD 载入全部 Anti 的记事本  
  
0040C087 N> /EB 01               jmp short NOTEPADy.0040C08A //开始就有小花,用花指令插件清除 6 个  
0040C089    |68 60E80000         push 0E860  
0040C08E     0000                add byte ptr ds:\[eax\],al  
0040C090     8B1C24              mov ebx,dword ptr ss:\[esp\]  
0040C093     83C3 12             add ebx,12  
0040C096     812B E8B10600       sub dword ptr ds:\[ebx\],6B1E8  
0040C09C     FE4B FD             dec byte ptr ds:\[ebx\-3\]  
0040C09F     822C24 B7           sub byte ptr ss:\[esp\],-49  
0040C0A3     CD 46               int 46  
0040C0A5     000B                add byte ptr ds:\[ebx\],cl  
0040C0A7     E4 74               in al,74  
0040C0A9     9E                  sahf  
0040C0AA     75 01               jnz short NOTEPADy.0040C0AD  
0040C0AC     C781 7304D77A F72F8>mov dword ptr ds:\[ecx+7AD70473\],738>  
0040C0B6     1977 00             sbb dword ptr ds:\[edi\],esi  
..............................................................  
  
0040C087 N>  90                  nop  
0040C088     90                  nop  
0040C089     90                  nop  
0040C08A     60                  pushad  
0040C08B     E8 00000000         call NOTEPADy.0040C090 //这里 hr esp,方便找 Stolen code，后面的版本无效，再说。  
0040C090     8B1C24              mov ebx,dword ptr ss:\[esp\]  
0040C093     83C3 12             add ebx,12  
0040C096     812B E8B10600       sub dword ptr ds:\[ebx\],6B1E8  
0040C09C     FE4B FD             dec byte ptr ds:\[ebx\-3\]  
0040C09F     822C24 B7           sub byte ptr ss:\[esp\],-49  
0040C0A3     CD 46               int 46  
0040C0A5     000B                add byte ptr ds:\[ebx\],cl  
0040C0A7     E4 74               in al,74  
0040C0A9     9E                  sahf  
0040C0AA     90                  nop  
0040C0AB     90                  nop  
0040C0AC     90                  nop  
0040C0AD     8173 04 D77AF72F    xor dword ptr ds:\[ebx+4\],2FF77AD7  
0040C0B4     8173 19 770043B7    xor dword ptr ds:\[ebx+19\],B7430077  
..............................................................  
  
Shfit+F9 3 此运行到最后一次特权指令 SEH 处  
  
00400181     FFFF                ???                                 ; 未知命令  
00400183     FFFF                ???                                 ; 未知命令  
00400185     FFFF                ???                                 ; 未知命令  
00400187     FFFF                ???                                 ; 未知命令  
00400189     FFFF                ???                                 ; 未知命令  
0040018B     FFFF                ???                                 ; 未知命令  
0040018D     FFFF                ???                                 ; 未知命令  
0040018F     FFFF                ???                                 ; 未知命令  
00400191     FFFF                ???                                 ; 未知命令  
00400193     FFFF                ???                                 ; 未知命令  
00400195     FFFF                ???                                 ; 未知命令  
00400197     FFFF                ???                                 ; 未知命令  
.......................................................  
  
然后你是记事本迷，IAT 当然知道是  
  
4062e0 开始，大小 248  
  
于是命令行  
  
dd 4062e4  
  
004062E4  00006DB4  
004062E8  00006DC8  
004062EC  00006DA2  
004062F0  00006DE6  
004062F4  00006DD6  
004062F8  00000000  
..................  
00406314  00006C1E   //随意取一段下内存写入断点  
00406318  00006C4A  
0040631C  00006C56  
00406320  00006C40  
  
Shift+F9 中断  
   
0040CAA6     830A 00             or dword ptr ds:\[edx\],0 //第一次，这不是加密 IAT  
0040CAA9     0F84 D2000000       je NOTEPADy.0040CB81  
0040CAAF     8B02                mov eax,dword ptr ds:\[edx\]  
0040CAB1     A9 00000080         test eax,80000000  
0040CAB6     74 09               je short NOTEPADy.0040CAC1  
0040CAB8     25 FFFFFF7F         and eax,7FFFFFFF  
0040CABD     2BFF                sub edi,edi  
0040CABF     EB 08               jmp short NOTEPADy.0040CAC9  
0040CAC1     40                  inc eax  
0040CAC2     0385 CB2C4000       add eax,dword ptr ss:\[ebp+402CCB\]  
0040CAC8     97                  xchg eax,edi  
0040CAC9     68 75DFD0F9         push F9D0DF75  
0040CACE     012C24              add dword ptr ss:\[esp\],ebp  
0040CAD1     810424 B4466F06     add dword ptr ss:\[esp\],66F46B4  
0040CAD8     68 7366630F         push 0F636673  
0040CADD     812C24 9643230F     sub dword ptr ss:\[esp\],0F234396  
0040CAE4     012C24              add dword ptr ss:\[esp\],ebp  
0040CAE7     C3                  retn  
  
0040CB73     8902                mov dword ptr ds:\[edx\],eax  //这句就是加密 IAT,正确 IAT 也是这里写入，记住地址。 现在错误地址 00860838 要写入 4062fc，得分析这段来源。  
  
我们就从  
  
0040CAA6     830A 00             or dword ptr ds:\[edx\],0   
  
到  
  
0040CBF7     C3                  retn  
  
清理一下花指令  
  
搜索地址  
  
0040CAA6  
  
大小  
  
151  
  
清除小花 10 个  
  
0040CB75     EB 01               jmp short NOTEPADy.0040CB78  
0040CB77     40                  inc eax  
0040CB78     83C2 04             add edx,4  
0040CB7B   ^ E9 26FFFFFF         jmp NOTEPADy.0040CAA6  
0040CB80     E8 83661000         call 00513208  
0040CB85     74 01               je short NOTEPADy.0040CB88  
0040CB87     C483 C614E987       les eax,fword ptr ds:\[ebx+87E914C6\]  
0040CB8D     FE                  ???                                 ; 未知命令  
0040CB8E     FFFF                ???                                 ; 未知命令  
0040CB90     C9                  leave  
0040CB91     8000 60             add byte ptr ds:\[eax\],60  
0040CB94     0000                add byte ptr ds:\[eax\],al  
0040CB96     898D 3DE92640       mov dword ptr ss:\[ebp+4026E93D\],ecx  
.............................................................................  
  
整理为  
  
0040CAA6     830A 00             or dword ptr ds:\[edx\],0  
0040CAA9     0F84 D2000000       je NOTEPADy.0040CB81  
0040CAAF     8B02                mov eax,dword ptr ds:\[edx\]  
0040CAB1     A9 00000080         test eax,80000000  
0040CAB6     74 09               je short NOTEPADy.0040CAC1  
0040CAB8     25 FFFFFF7F         and eax,7FFFFFFF  
0040CABD     2BFF                sub edi,edi  
0040CABF     EB 08               jmp short NOTEPADy.0040CAC9  
0040CAC1     40                  inc eax  
0040CAC2     0385 CB2C4000       add eax,dword ptr ss:\[ebp+402CCB\]  
0040CAC8     97                  xchg eax,edi  
0040CAC9     68 75DFD0F9         push F9D0DF75  
0040CACE     012C24              add dword ptr ss:\[esp\],ebp  
0040CAD1     810424 B4466F06     add dword ptr ss:\[esp\],66F46B4  
0040CAD8     68 7366630F         push 0F636673  
0040CADD     812C24 9643230F     sub dword ptr ss:\[esp\],0F234396  
0040CAE4     012C24              add dword ptr ss:\[esp\],ebp  
0040CAE7     C3                  retn  
0040CAE8     85C0                test eax,eax  
0040CAEA     EA 0F845B05 0000    jmp far 0000:055B840F  
0040CAF1     90                  nop  
0040CAF2     90                  nop  
0040CAF3     90                  nop  
0040CAF4     0BFF                or edi,edi  
0040CAF6     9C                  pushfd  
0040CAF7     C12C24 06           shr dword ptr ss:\[esp\],6  
0040CAFB     832424 01           and dword ptr ss:\[esp\],1  
0040CAFF     50                  push eax  
0040CB00     52                  push edx  
0040CB01     B8 0ABBDC1B         mov eax,1BDCBB0A  
0040CB06     05 444523E4         add eax,E4234544  
0040CB0B     F76424 08           mul dword ptr ss:\[esp+8\]  
0040CB0F     8D8428 64264000     lea eax,dword ptr ds:\[eax+ebp+40266>  
0040CB16     894424 08           mov dword ptr ss:\[esp+8\],eax  
0040CB1A     5A                  pop edx  
0040CB1B     58                  pop eax  
0040CB1C     8D6424 04           lea esp,dword ptr ss:\[esp+4\]  
0040CB20     FF6424 FC           jmp dword ptr ss:\[esp\-4\]  
0040CB24     FF0F                dec dword ptr ds:\[edi\]  
0040CB26     BA 67FF079C         mov edx,9C07FF67  
0040CB2B     F71424              not dword ptr ss:\[esp\]  
0040CB2E     832424 01           and dword ptr ss:\[esp\],1  
0040CB32     50                  push eax  
0040CB33     52                  push edx  
0040CB34     B8 EA2F6BF7         mov eax,F76B2FEA  
0040CB39     05 31D09408         add eax,894D031  
0040CB3E     F76424 08           mul dword ptr ss:\[esp+8\]  
0040CB42     8D8428 97264000     lea eax,dword ptr ds:\[eax+ebp+40269>  
0040CB49     894424 08           mov dword ptr ss:\[esp+8\],eax  
0040CB4D     5A                  pop edx  
0040CB4E     58                  pop eax  
0040CB4F     8D6424 04           lea esp,dword ptr ss:\[esp+4\]  
0040CB53     FF6424 FC           jmp dword ptr ss:\[esp\-4\]  
0040CB57     FFE8                jmp far eax                         ; 非法使用寄存器  
0040CB59     0300                add eax,dword ptr ds:\[eax\]  
0040CB5B     0000                add byte ptr ds:\[eax\],al  
0040CB5D     90                  nop  
0040CB5E     90                  nop  
0040CB5F     90                  nop  
0040CB60     90                  nop  
0040CB61     90                  nop  
0040CB62     90                  nop  
0040CB63     830424 0C           add dword ptr ss:\[esp\],0C  
0040CB67     C3                  retn  
0040CB68     0D E8E1FAFF         or eax,FFFAE1E8  
0040CB6D     FF0B                dec dword ptr ds:\[ebx\]  
0040CB6F     E4 90               in al,90  
0040CB71     90                  nop  
0040CB72     90                  nop  
0040CB73     8902                mov dword ptr ds:\[edx\],eax          ; Wiat  
0040CB75     90                  nop  
0040CB76     90                  nop  
0040CB77     90                  nop  
0040CB78     83C2 04             add edx,4  
0040CB7B   ^ E9 26FFFFFF         jmp NOTEPADy.0040CAA6  
  
这段里面有一个加密 Call,呵呵，我是对比不加密 IAT 的分析出了的。  
  
单步走一遍这个循环  
  
0040CB4F     8D6424 04           lea esp,dword ptr ss:\[esp+4\]  
0040CB53     FF6424 FC           jmp dword ptr ss:\[esp\-4\]            ; NOTEPADy.0040CB58  
  
0040CB57     FFE8                jmp far eax                         ; 非法使用寄存器  
  
花指令，怎么 loveboom 没加入花指令特征码，我现在还不会扩充花指令特征文件。  
  
0040CB57     90                  nop  //注意现在 EAX=77F45946,是 GDI 库，PESpin 要处理的东西。  
0040CB58     E8 03000000         call NOTEPADy.0040CB60  //加密 Call,要处理掉。  
  
直接到 0040CB73 即可  
  
修改为  
  
0040CB58    /EB 19               jmp short NOTEPADy.0040CB73  
0040CB5A    |90                  nop  
0040CB5B    |90                  nop  
0040CB5C    |90                  nop  
  
0040CB5D     90                  nop  
0040CB5E     90                  nop  
0040CB5F     90                  nop  
0040CB60     90                  nop  
0040CB61     90                  nop  
0040CB62     90                  nop  
0040CB63     830424 0C           add dword ptr ss:\[esp\],0C  
0040CB67     C3                  retn  //如果执行了  
  
0040CB58     E8 03000000         call NOTEPADy.0040CB60  
  
0040CB71     90                  nop  
0040CB72     90                  nop  
0040CB73     8902                mov dword ptr ds:\[edx\],eax   //这里我们下的内存写入断点就会中断，不加密的如 ADVAPI32 指针等是直接是走这个路线。  
0040CB75     90                  nop  
0040CB76     90                  nop  
0040CB77     90                  nop  
0040CB78     83C2 04             add edx,4  
0040CB7B   ^ E9 26FFFFFF         jmp NOTEPADy.0040CAA6  
  
这样 IAT 问题就解决了，二哥还是喜欢爆破，不喜欢理论。  
  
继续 Shift+F9 就到了  
  
0040CC0D     55                  push ebp  
0040CC0E     EB 01               jmp short NOTEPADy.0040CC11  
0040CC10   ^ 79 8B               jns short NOTEPADy.0040CB9D  
0040CC12     EC                  in al,dx  
0040CC13     EB 01               jmp short NOTEPADy.0040CC16  
0040CC15     45                  inc ebp  
0040CC16     83EC 44             sub esp,44  
0040CC19     EB 01               jmp short NOTEPADy.0040CC1C  
0040CC1B     FC                  cld  
0040CC1C     56                  push esi  
0040CC1D     EB 01               jmp short NOTEPADy.0040CC20  
0040CC1F     38FF                cmp bh,bh  
0040CC21     15 E4634000         adc eax,NOTEPADy.004063E4  
0040CC26     EB 01               jmp short NOTEPADy.0040CC29  
0040CC28     95                  xchg eax,ebp  
0040CC29   - E9 AB44FFFF         jmp NOTEPADy.004010D9  
  
去花  
  
0040CC0D     55                  push ebp  //可以直接在这里就脱壳，也可以到 4010D9 处补上 Stolen code  
0040CC0E     90                  nop  
0040CC0F     90                  nop  
0040CC10     90                  nop  
0040CC11     8BEC                mov ebp,esp  
0040CC13     90                  nop  
0040CC14     90                  nop  
0040CC15     90                  nop  
0040CC16     83EC 44             sub esp,44  
0040CC19     90                  nop  
0040CC1A     90                  nop  
0040CC1B     90                  nop  
0040CC1C     56                  push esi  
0040CC1D     90                  nop  
0040CC1E     90                  nop  
0040CC1F     90                  nop  
0040CC20     FF15 E4634000       call dword ptr ds:\[4063E4\]          ; KERNEL32.GetCommandLineA  
0040CC26     90                  nop  
0040CC27     90                  nop  
0040CC28     90                  nop  
0040CC29   - E9 AB44FFFF         jmp NOTEPADy.004010D9  
......................................................................  
  
004010CC     0000                add byte ptr ds:\[eax\],al  
004010CE     0000                add byte ptr ds:\[eax\],al  
004010D0     0000                add byte ptr ds:\[eax\],al  
004010D2     0000                add byte ptr ds:\[eax\],al  
004010D4     0000                add byte ptr ds:\[eax\],al  
004010D6     0000                add byte ptr ds:\[eax\],al  
004010D8     008B F08A003C       add byte ptr ds:\[ebx+3C008AF0\],cl  
004010DE     2275 1B             and dh,byte ptr ss:\[ebp+1B\]  
004010E1     56                  push esi  
004010E2     FF15 F4644000       call dword ptr ds:\[4064F4\]          ; USER32.CharNextA  
004010E8     8BF0                mov esi,eax  
004010EA     8A00                mov al,byte ptr ds:\[eax\]  
004010EC     84C0                test al,al  
004010EE     74 04               je short NOTEPADy.004010F4  
004010F0     3C 22               cmp al,22  
004010F2   ^ 75 ED               jnz short NOTEPADy.004010E1  
004010F4     803E 22             cmp byte ptr ds:\[esi\],22  
004010F7     75 15               jnz short NOTEPADy.0040110E  
004010F9     46                  inc esi  
004010FA     EB 12               jmp short NOTEPADy.0040110E  
004010FC     3C 20               cmp al,20  
004010FE     7E 0E               jle short NOTEPADy.0040110E  
00401100     56                  push esi  
00401101     FF15 F4644000       call dword ptr ds:\[4064F4\]          ; USER32.CharNextA  
00401107     8038 20             cmp byte ptr ds:\[eax\],20  
0040110A     8BF0                mov esi,eax  
0040110C   ^ 7F F2               jg short NOTEPADy.00401100  
0040110E     803E 00             cmp byte ptr ds:\[esi\],0  
00401111     74 13               je short NOTEPADy.00401126  
00401113     803E 20             cmp byte ptr ds:\[esi\],20  
00401116     77 0E               ja short NOTEPADy.00401126  
00401118     56                  push esi  
00401119     FF15 F4644000       call dword ptr ds:\[4064F4\]          ; USER32.CharNextA  
0040111F     8038 00             cmp byte ptr ds:\[eax\],0  
00401122     8BF0                mov esi,eax  
00401124   ^ 75 ED               jnz short NOTEPADy.00401113  
00401126     C745 E8 00000000    mov dword ptr ss:\[ebp\-18\],0  
0040112D     8D4D BC             lea ecx,dword ptr ss:\[ebp\-44\]  
00401130     51                  push ecx  
00401131     FF15 98634000       call dword ptr ds:\[406398\]          ; KERNEL32.GetStartupInfoA  
00401137     F645 E8 01          test byte ptr ss:\[ebp\-18\],1  
0040113B     B8 0A000000         mov eax,0A  
00401140     74 04               je short NOTEPADy.00401146  
00401142     0FB745 EC           movzx eax,word ptr ss:\[ebp\-14\]  
00401146     50                  push eax  
00401147     56                  push esi  
00401148     6A 00               push 0  
0040114A     6A 00               push 0  
0040114C     FF15 9C634000       call dword ptr ds:\[40639C\]          ; KERNEL32.GetModuleHandleA  
00401152     50                  push eax  
00401153     E8 760F0000         call NOTEPADy.004020CE  
00401158     50                  push eax  
00401159     8BF0                mov esi,eax  
0040115B     FF15 A0634000       call dword ptr ds:\[4063A0\]          ; KERNEL32.ExitProcess  
00401161     8BC6                mov eax,esi  
00401163     5E                  pop esi  
00401164     8BE5                mov esp,ebp  
00401166     5D                  pop ebp  
00401167     C3                  retn  
00401168     833D B8574000 00    cmp dword ptr ds:\[4057B8\],0  
0040116F     74 0C               je short NOTEPADy.0040117D  
00401171     A1 B8574000         mov eax,dword ptr ds:\[4057B8\]  
00401176     50                  push eax  
00401177     FF15 D4634000       call dword ptr ds:\[4063D4\]          ; KERNEL32.GlobalFree  
.......................................................................  
  
看到 IAT 都是好的，这次看来我们又胜利了。  
  
【脱壳总结】  
  
0040CB58     E8 03000000         call NOTEPADy.0040CB60  //加密 Call,要处理掉。  
  
直接到 0040CB73 即可  
  
修改为  
  
0040CB58    /EB 19               jmp short NOTEPADy.0040CB73  
0040CB5A    |90                  nop  
0040CB5B    |90                  nop  
0040CB5C    |90                  nop  
  
因为 IAT 都是好的，所以 OD 插件直接在  
  
0040CC0D 处新建起源，直接脱壳，重建输入表方式 1 脱壳修复一步到位，正常运行。  
  
\--------------------------------------------------------------------------------  
  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

// Pespin V0.3 Stolen code and Patch IAT  v0.1  
// 作者: Mr.David  
// 主页: www.chinadfcg.com  
  
msg "请设置 OD 异常设置除了错误的或有特权的指令外全部忽略，然后从菜单处继续运行脚本"  
pause  
  
var addr1  
var addr2  
  
sto  
sto  
mov addr1,esp  //Esp 定律  
  
run  
esto  
esto  
  
gpa "LoadLibraryA","kernel32.dll"  
mov addr2,$RESULT                    //捷径 API 断点 LoadLibraryA  
bp addr2  
esto  
  
bc addr2    //Clear break point  //取消断点  
rtu        //Alt+F9  
  
findop eip,#ff6424fc#    //特征指令  
mov addr2,$RESULT           
bphws addr2,"x"       
run  
BPHWC addr2  
  
findop eip,#ff6424fc#    //特征指令  
mov addr2,$RESULT           
bphws addr2,"x"       
run  
BPHWC addr2  
  
repl eip, #ff??????#, #eb1e9090#, 4      //有病治病，无病强身  
  
bphws addr1,"r"  
run  
BPHWC addr1  
  
Msg "Stolen code 或 OEP，请直接在这里脱壳，IAT 已经修复"  
  
cmt eip,"Stolen code or OEP To Get,Please dumped it,Enjoy!"  
ret

 4dd466d0e1e0cc123afbe504da2462ca