

# 轻松解被 MoleBox 打包了的程序

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-625.htm`](https://www.pediy.com/kssd/pediy08/pediy8-625.htm)

【文章标题】: 轻松解被 MoleBox 打包了的程序  
【文章作者】: wynney  
【软件名称】: flashfxp 3.3.7(bulid 1129) RC1  
【下载地址】: 可以问 LOVE\[BCG\]\[DFCG\]兄要 ^\_^  
【保护方式】: 由 MoleBox 打包  
【软件介绍】: LOVE\[BCG\]\[DFCG\]干的好事 哈哈  
【作者声明】: 今天很无聊，看雪论坛也似乎没以前热闹了，于是，在电脑里找了下，发现先前做的这么个记录，便重新整理下，大虾飘过  
\--------------------------------------------------------------------------------  
【详细过程】  
  不记得什么时候了，在一蓑烟雨论坛上看到烈火兄的脱壳申请，觉得有点意思，就拿来玩了下，一点经验仅供大家参考了  
    
  忽略所有异常  
  一、先去 OEP 溜达下吧  
    
  程序入口  
   

引用:    
  0072FB53 >  E8 00000000        call flashfxp.0072FB58   ；EP，F8  
  0072FB58    60                 pushad                 
  0072FB59    E8 4F000000        call flashfxp.0072FBAD   ；ESP=0012FFA0，hr 0012FFA0,Shift+F9 4 次  
  

  
  这里为什么 Shift+F9 4 次？  
  自己跟下就知道了，easy  
  

引用:    
  00715B57    83C7 08            add edi,8                ；第 4 次中断在这里，删除硬件断点，向下拉下看看。。  
  00715B5A    FF96 1CCB3200      call dword ptr ds:\[esi+32CB1C\]  
  00715B60    95                 xchg eax,ebp  
  00715B61    8A07               mov al,byte ptr ds:\[edi\]  
  00715B63    47                 inc edi  
  00715B64    08C0               or al,al  
  00715B66  ^ 74 DC              je short flashfxp.00715B44  
  00715B68    89F9               mov ecx,edi  
  00715B6A    57                 push edi  
  00715B6B    48                 dec eax  
  00715B6C    F2:AE              repne scas byte ptr es:\[edi\]  
  00715B6E    55                 push ebp  
  00715B6F    FF96 20CB3200      call dword ptr ds:\[esi+32CB20\]  
  00715B75    09C0               or eax,eax  
  00715B77    74 07              je short flashfxp.00715B80  
  00715B79    8903               mov dword ptr ds:\[ebx\],eax  
  00715B7B    83C3 04            add ebx,4  
  00715B7E  ^ EB E1              jmp short flashfxp.00715B61  
  00715B80    FF96 28CB3200      call dword ptr ds:\[esi+32CB28\]  
  00715B86    8BAE 24CB3200      mov ebp,dword ptr ds:\[esi+32CB24\]  
  00715B8C    8DBE 00F0FFFF      lea edi,dword ptr ds:\[esi-1000\]  
  00715B92    BB 00100000        mov ebx,1000  
  00715B97    50                 push eax  
  00715B98    54                 push esp  
  00715B99    6A 04              push 4  
  00715B9B    53                 push ebx  
  00715B9C    57                 push edi  
  00715B9D    FFD5               call ebp  
  00715B9F    8D87 1F020000      lea eax,dword ptr ds:\[edi+21F\]  
  00715BA5    8020 7F            and byte ptr ds:\[eax\],7F  
  00715BA8    8060 28 7F         and byte ptr ds:\[eax+28\],7F  
  00715BAC    58                 pop eax  
  00715BAD    50                 push eax  
  00715BAE    54                 push esp  
  00715BAF    50                 push eax  
  00715BB0    53                 push ebx  
  00715BB1    57                 push edi  
  00715BB2    FFD5               call ebp  
  00715BB4    58                 pop eax  
  00715BB5    61                 popad  
  00715BB6    8D4424 80          lea eax,dword ptr ss:\[esp-80\]  
  00715BBA    6A 00              push 0  
  00715BBC    39C4               cmp esp,eax  
  00715BBE  ^ 75 FA              jnz short flashfxp.00715BBA  
  00715BC0    83EC 80            sub esp,-80  
  00715BC3  ^ E9 8890F0FF        jmp flashfxp.0061EC50       ；跳向 OEP，直接 F4 下来，F8 下到 OEP  
  

  
  

引用:    
  0061EC50    55                 push ebp                    ；OEP，Delphi 程序的入口  
  0061EC51    8BEC               mov ebp,esp  
  0061EC53    83C4 E0            add esp,-20  
  0061EC56    53                 push ebx  
  

  
  到了 OEP 呢，大家先不要着急脱壳，还是先打开 ImportREC。。看看，你会发现有 35 个无效(在你那有多少个我就不知了)  
  
  
  二、IAT 加密处理  
  
  重新加载程序  
  依旧像第一步那样，单步到 0072FB59，hr 0012FFA0(这样做是方便我们到达 OEP)  
  命令行 bp VirtualProtect，Shift+F9  
  中断 2 次后返回，断点不要取消哦，看看。。。  

引用:    
  00736952    8B15 04267400      mov edx,dword ptr ds:\[742604\]      ；返回到这里  
  00736958    8B45 E8            mov eax,dword ptr ss:\[ebp-18\]  
  0073695B    0342 08            add eax,dword ptr ds:\[edx+8\]  
  0073695E    8945 F4            mov dword ptr ss:\[ebp-C\],eax  
  00736961    C705 94287400 0000>mov dword ptr ds:\[742894\],0  
  0073696B    6A 00              push 0  
  0073696D    68 C41B7400        push flashfxp.00741BC4             ; ASCII "EXECUTABLE"  
  00736972    8B0D 90287400      mov ecx,dword ptr ds:\[742890\]      ; flashfxp.00400100  
  

  
  
  补习下英语 EXECUTABLE，呵呵  
  继续 Shift+F9 中断一次，返回  
  

引用:    
  00736F9C    85C0               test eax,eax                       ；返回到这里，Ctrl+B：89 01  
  00736F9E    75 0A              jnz short flashfxp.00736FAA  
  00736FA0    B9 0B0000EF        mov ecx,EF00000B  
  00736FA5    E8 642B0000        call flashfxp.00739B0E  
  00736FAA    8B4D 08            mov ecx,dword ptr ss:\[ebp+8\]  
  00736FAD    8B55 F8            mov edx,dword ptr ss:\[ebp-8\]  
  00736FB0    8B02               mov eax,dword ptr ds:\[edx\]  
  00736FB2    8901               mov dword ptr ds:\[ecx\],eax         ；就是这里对 IAT 加密处理了，nop 掉  
  00736FB4    8D4D F4            lea ecx,dword ptr ss:\[ebp-C\]  
  00736FB7    51                 push ecx  
  00736FB8    8B55 F0            mov edx,dword ptr ss:\[ebp-10\]  
  00736FBB    52                 push edx  
  00736FBC    6A 04              push 4  
  00736FBE    8B45 08            mov eax,dword ptr ss:\[ebp+8\]  
  00736FC1    50                 push eax  
  00736FC2    FF15 2C277400      call dword ptr ds:\[74272C\]         ; kernel32.VirtualProtect  
  

  
  取消先前下的 VirtualProtect 断点，看看我们开始下的 hr 断点，还在。。呵呵  
  准备到 OEP 了，Shift+F9，4 次。。。  
  
  
  和第一步骤一样。。。  

引用:    
  00715B57    83C7 08            add edi,8                          ；中断在这里，取消硬件断点  
  00715B5A    FF96 1CCB3200      call dword ptr ds:\[esi+32CB1C\]  
  00715B60    95                 xchg eax,ebp  
  00715B61    8A07               mov al,byte ptr ds:\[edi\]  
  00715B63    47                 inc edi  
  00715B64    08C0               or al,al  
  00715B66  ^ 74 DC              je short flashfxp.00715B44  
  00715B68    89F9               mov ecx,edi  
  00715B6A    57                 push edi  
  00715B6B    48                 dec eax  
  00715B6C    F2:AE              repne scas byte ptr es:\[edi\]  
  00715B6E    55                 push ebp  
  00715B6F    FF96 20CB3200      call dword ptr ds:\[esi+32CB20\]  
  00715B75    09C0               or eax,eax  
  00715B77    74 07              je short flashfxp.00715B80  
  00715B79    8903               mov dword ptr ds:\[ebx\],eax  
  00715B7B    83C3 04            add ebx,4  
  00715B7E  ^ EB E1              jmp short flashfxp.00715B61  
  00715B80    FF96 28CB3200      call dword ptr ds:\[esi+32CB28\]  
  00715B86    8BAE 24CB3200      mov ebp,dword ptr ds:\[esi+32CB24\]  
  00715B8C    8DBE 00F0FFFF      lea edi,dword ptr ds:\[esi-1000\]  
  00715B92    BB 00100000        mov ebx,1000  
  00715B97    50                 push eax  
  00715B98    54                 push esp  
  00715B99    6A 04              push 4  
  00715B9B    53                 push ebx  
  00715B9C    57                 push edi  
  00715B9D    FFD5               call ebp  
  00715B9F    8D87 1F020000      lea eax,dword ptr ds:\[edi+21F\]  
  00715BA5    8020 7F            and byte ptr ds:\[eax\],7F  
  00715BA8    8060 28 7F         and byte ptr ds:\[eax+28\],7F  
  00715BAC    58                 pop eax  
  00715BAD    50                 push eax  
  00715BAE    54                 push esp  
  00715BAF    50                 push eax  
  00715BB0    53                 push ebx  
  00715BB1    57                 push edi  
  00715BB2    FFD5               call ebp  
  00715BB4    58                 pop eax  
  00715BB5    61                 popad  
  00715BB6    8D4424 80          lea eax,dword ptr ss:\[esp-80\]  
  00715BBA    6A 00              push 0  
  00715BBC    39C4               cmp esp,eax  
  00715BBE  ^ 75 FA              jnz short flashfxp.00715BBA  
  00715BC0    83EC 80            sub esp,-80  
  00715BC3  ^ E9 8890F0FF        jmp flashfxp.0061EC50                 ；直接 F4 下来，跳向 OEP  
  

  
  LordPE 脱壳，ImportREC 修复，全部有效  
  
  Btw：你如果慢慢跟踪你会发现里面一层是 UPX 的壳，所以，脱壳文件使用 PEID 查得显示  
  UPX 0.89.6 - 1.02 / 1.05 - 1.24 (Delphi) stub -> Markus & Laszlo  
  fi 查得是 Delphi 的，为了方便汉化使用 FixRes 修复下资源，Done！  
  
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 06 月 15 日 14:32:46 

感谢楼主，molebox 的 IAT 加密的地方我以前一直没找到，放在一边 N 久了，看了你的文章解决了  
但是有个地方我想补充一下，其实你脱的这个程序是双层壳,  
hr 0012FFA0,Shift+F9 一次时,  
0072F72B    58              pop eax  
0072F72C    894424 24       mov dword ptr ss:\[esp+24\],eax  
0072F730    61              popad  
0072F731    58              pop eax //停在这一行  
0072F732    58              pop eax  
0072F733    FFD0            call eax  
0072F735    E8 A6C00000     call flashfxp.0073B7E0  
0072F73A    CC              int3  
  
停在 72f731 这里，在 72f733 这个 call 用 F7 跟进去就离开 molebox 这一层壳了。第二层好像是 UPX1.90 的壳 

我最喜欢写脚本了:  
//molebox 2.x ,by skylly  
msg "忽略所有异常"  
sto  
sto  
var cool  
mov cool,esp  
bphws cool,"r" //esp 定律,用来找 OEP  
var VirtualProtect  
gpa "VirtualProtect","kernel32.dll"  
cmp $RESULT,0  
je err  
mov VirtualProtect,$RESULT  
bp VirtualProtect  //下 VP 的断点为了找 IAT 加密的地方  
  
eob check  
eoe check //发生中断则执行 check  
esto  
  
ret  
  
  
check:  
cmp eip,VirtualProtect  //检查是否是断在 VP  
je VirtualProtect  
esto  
ret  
  
  
VirtualProtect:  
rtu  
var a  
mov a,eip  
mov a,\[a\]  
cmp a,A75C085 //看用户代码处是否 test eax,eax;jnz  
je iat //如果是则到了关键地点  
esto  
ret  
  
  
  
cool: //所有要做的都做完了,很快到 OEP 了  
cob  
coe  
run  
bphwc cool  
  
zou: //自动走路的代码  
mov a,eip  
mov a,\[a\]  
shl a,8  
shl a,8  
shl a,8  
cmp a,58000000  //直到指令不是 pop eax 为止  
jne cool2  
sto  
jmp zou  
  
cool2:  
sti //进入 call eax 就到 OEP 了  
jmp oep  
ret  
  
  
  
iat: //对 IAT 加密的代码进行 PATCH  
find eip,#8901#  
cmp $RESULT,0  
je err  
mov \[$RESULT\],#9090#  
msg "绕开输入表加密!"  
bc VirtualProtect  
jmp cool  
ret  
  
  
oep:  
cmt eip,"OEP!!!dump and fix IAT"  
an eip  
ret  
  
err:  
msg "error"  
ret