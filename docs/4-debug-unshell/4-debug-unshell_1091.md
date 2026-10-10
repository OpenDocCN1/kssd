

# 关于最新 TMD/WL 2120 壳的分析（更新）

> 原文：[`www.pediy.com/kssd/pediy11/116091.html`](https://www.pediy.com/kssd/pediy11/116091.html)

由于 TMD/VM 1.x,2.x 脱壳脚本无法成功经行脱壳，通过对 2120 与 2100 壳的比较发现，脚本问题出在 查询内存 #00063006D1C846# 这里  
0086FF89    0006            add     byte ptr \[esi\], al  
0086FF8B    3006            xor     byte ptr \[esi\], al  
0086FF8D    D1C8            ror     eax, 1  
0086FF8F    46              inc     esi  
（0080F000 内存基址 为 WL/TMD 核心加载段）  
在 TMD 核心段中，老 2120 前的版本 可以找到 2 处，但是新版本没有第一处，  
一下为新版的第一处内存  
008FA5BF    60              pushad  
008FA5C0    E9 10000000     jmp     008FA5D5  
008FA5C5    4A              dec     edx  
008FA5C6    85D9            test    ecx, ebx  
008FA5C8  ^ 76 AA           jbe     short 008FA574  
008FA5CA    43              inc     ebx  
008FA5CB    EA B9687B70 27E>jmp     far EC27:707B68B9  
008FA5D2    94              xchg    eax, esp  
008FA5D3    2878 80         sub     byte ptr \[eax-80\], bh  
008FA5D6    CE              into  
008FA5D7    90              nop  
008FA5D8    B8 01000000     mov     eax, 1  
008FA5DD    6A 00           push    0  
008FA5DF    53              push    ebx  
008FA5E0    E8 03000000     call    008FA5E8  
  
以下为老版本第一处内存  
  
0086FF70   /E9 87000000     jmp     0086FFFC  
0086FF75   |52              push    edx  
0086FF76   |8BD4            mov     edx, esp  
0086FF78   |60              pushad  
0086FF79   |8B72 08         mov     esi, dword ptr \[edx+8\]  
0086FF7C   |8B7A 0C         mov     edi, dword ptr \[edx+C\]  
0086FF7F   |B8 B2CA45A9     mov     eax, A945CAB2  
0086FF84   |E9 07000000     jmp     0086FF90  
0086FF89   |0006       add     byte ptr \[esi\], al  ---》查找此处以后 + F 达到 0086FF98  
0086FF8B   |3006            xor     byte ptr \[esi\], al  
0086FF8D   |D1C8            ror     eax, 1  
0086FF8F   |46              inc     esi  
0086FF90   |3BF7            cmp     esi, edi  
0086FF92  ^|0F82 F1FFFFFF   jb      0086FF89  
0086FF98   |61              popad                      ---》目的就是为了查找此处做硬件断点  
0086FF99   |5A              pop     edx  
0086FF9A   |C2 0800         retn    8  
  
老版本在断下以后 会获取 EDI 寄存器后查找 000a0a0a ，查到后方能继续脱壳，新版本的壳去掉了第一处，就无法继续脱壳了，第二次做硬件断点是无效的。望高手能分析一下。。。时间紧迫，难道这就是更改授权方式的后果？  
  
\----------------------------------  
对 TMD 壳也没多少时间去脱他，就用了偷懒的办法，结果还真脱出来了，惭愧的很。  
由于能力有限只能针对 delphi 程序的 TMD/WL 2120 来了一次另类脱壳，载入 OD 以后 断点 GetModuleHandleA，由于该 API 位于 delphi 程序加载前首要位置，断下他就意味着在壳部分已经全部过完，除了偷去的代码以外其他全部载入代码段中，断此 API 需要一边看 CODE 节，一边按 CTRL+F9 ，CODE 段写完了 delphi 的 GetModuleHandleA 就差不多到了，具体需要看情况而定，无需任何脚本就能让你达到这里，当然确保你的 OD 是 noody 的，不会被壳发现。到了此处以后可以通过堆栈查看返回地址，返回到代码段，你可以暂时返回过来的函数地址作为 OEP 进行脱壳，此时就按照 TMD/WL 的传统脱壳大法了，需要找真的 OEP，其实 DELPHI 的 OEP 很有特点的，对比没有壳的代码就可以造出新 OEP，因为 TMD 通常会偷取 OEP 代码，这里就需要你重新脱一次。  
  
00406FE0  /$  53            PUSH EBX                       《----脱壳位置  
00406FE1  |.  8BD8          MOV EBX,EAX  
00406FE3  |.  33C0          XOR EAX,EAX  
00406FE5  |.  A3 A4F05000   MOV DWORD PTR DS:\[50F0A4\],EAX  
00406FEA  |.  6A 00         PUSH 0                                   ; /pModule = NULL  
00406FEC  |.  E8 13FFFFFF   CALL <JMP.&kernel32.GetModuleHandleA>    ; \\GetModuleHandleA //断点  
00406FF1  |.  A3 68765100   MOV DWORD PTR DS:\[517668\],EAX //堆栈返回  
00406FF6  |.  A1 68765100   MOV EAX,DWORD PTR DS:\[517668\]  
00406FFB  |.  A3 B0F05000   MOV DWORD PTR DS:\[50F0B0\],EAX  
00407000  |.  33C0          XOR EAX,EAX  
00407002  |.  A3 B4F05000   MOV DWORD PTR DS:\[50F0B4\],EAX  
00407007  |.  33C0          XOR EAX,EAX  
00407009  |.  A3 B8F05000   MOV DWORD PTR DS:\[50F0B8\],EAX  
0040700E  |.  E8 C1FFFFFF   CALL login\_du.00406FD4  
00407013  |.  BA ACF05000   MOV EDX,login\_du.0050F0AC  
00407018  |.  8BC3          MOV EAX,EBX  
0040701A  |.  E8 11D9FFFF   CALL login\_du.00404930  
0040701F  |.  5B            POP EBX  
00407020  \\.  C3            RETN //运行到这里以后，会返回到 TMD VMP CALL  
  
返回到 VMP   
0075E33F    68 858FEE0D     PUSH 0DEE8F85  
0075E344  ^ E9 E113F1FF     JMP login\_du.0066F72A  
0075E349    68 CA90EE0D     PUSH 0DEE90CA                  ---》会返回到这样的地方，然后 代码段下内存读取断点 F9 过去，每 F9 一次记录 注解和寄存器 通常 EAX EDX ECX EBX 就够，几次以后就能回到 下一个 CALL 位置  
0075E34E  ^ E9 D713F1FF     JMP login\_du.0066F72A  
0075E353    68 5492EE0D     PUSH 0DEE9254  
0075E358  ^ E9 CD13F1FF     JMP login\_du.0066F72A  
0075E35D    68 6E93EE0D     PUSH 0DEE936E  
  
几次下来所有记录好以后，可以对照无壳的 DELPHI 来进行修复 OEP 了。下面我给大家做了一个 修复偷代码用的记录脚本，很多工作都傻瓜化了。  
  
  
var findbase  
var sizebase  
var tempbase  
var count  
var index  
  
mov findbase, 401000                 //代码段基地址  
mov sizebase, 12d000                //代码段长度  
mov tempbase, findbase              
add tempbase, sizebase            //代码段的下一节基地址  
  
mov index, 0                               //记录  
mov count, 2B                             //VMP CALL 的次数  
  
TMD\_START:  
add index, 1  
log index  
log "vmp call start {"  // 一次 VMP CALL 调用的开始  
  
TMD\_IMAGE:  
bprm findbase, sizebase            // 下代码段读取断点  
esto   
bpmc  
cmp eip, tempbase  
jb STD\_IMAGE  
log "vmp execute start{" // 这里记录着在 VMP 中所执行的过程，记录多次  
log eip  
log eax  
log ecx  
log edx  
log ebx  
log esp  
log ebp  
log esi  
log edi  
log "} vmp execute over"  
jmp TMD\_IMAGE  
  
STD\_IMAGE:  
log "} vmp call over" // 一次 VMP CALL 调用的结束  
log "std call start {" //这里就意味着返回到代码段中 再记录一次  
log eip  
log eax  
log ecx  
log edx  
log ebx  
log esp  
log ebp  
log esi  
log edi  
log "} std call over"  
  
OEP\_IMAGE:  
cmp index, count  
je END                        // stop !!!  
rtr  
find eip, #68??????0DE9??????FF# // VMP CALL  
cmp $RESULT, 0  
je OEP\_IMAGE  
jmp TMD\_START  
  
END:  
bpmc  
ret  
  
执行完以后 OD 脚本记录器里记录下内容就可以去修复了，此脚本一般用于达到 VMP CALL 之前执行使用所有偷代码的修复工作，修复起来还是很辛苦的。  
  
下一节 我会讲一下 TMD/WL VM 1.X 2.X 脚本修复 TMD/WL 2120 API 的另类修复大法