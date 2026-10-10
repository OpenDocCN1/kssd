

# ASProtect v1.00 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6705.htm`](https://www.pediy.com/kssd/pediy06/pediy6705.htm)

用 OllyDbg 1.10 手脱 chap708.exe 之 Mission Impassable?（OS:98/XP）

实例下载：chap708.exe 点击此处下载。  
  
98 环境：  
说明：（失败之处，不解之处还请赐教！）  
无需隐藏 OD，你要隐我也没意见^o^，OD 设置忽略所有异常，平台只要你的平台能运行 OD 就行了;  
异常无特别说明，Shift+F9 通过就行了！看到的 nop 指令一般是垃圾指令，你看到的 4xxxxx 不一定是程序空间（参看内存窗口你就知道）  
跟踪得知在 call dword ptr ds:\[eax\-C\]异常前已经加密处理了 Iat，所以……  
  
入口不贴了，重来，int3 和内存异常不忽略，F9 运行来到 int3 断点命令中中断：  
00467B28     CC                int3                             ; // int3 中断命令  
00467B29     EB 02             jmp short 00467B2D               ; // 中断于此  
00467B2B   - E9 025DEB02       jmp 0331D832  
00467B30   - E9 FF33C05A       jmp 5B06AF34  
00467B35     59                pop ecx  
00467B36     59                pop ecx  
00467B37     64:8910           mov dword ptr fs:\[eax\],edx  
00467B3A     EB 0A             jmp short 00467B46  
00467B3C   ^ E9 1305FFFF       jmp 00458054  
  
Ctrl+B 搜索进二进制串：ABEBDC61，搜到后将“stos dword ptr es:\[edi\]”改为 Nop 指令，是不是很简单……  
  
00477B13     AB                stos dword ptr es:\[edi\] //Iat 加密处理，Nop 去（解压码默认是 98 的 AIP）  
00477B14   ^ EB DC             jmp short 00477AF2  
00477B16     61                popad                   //此处下 F2 断点并 Shift+F9 到达，iat 已经是清白之身，不曾被人沾污！  
00477B17     837D 08 00        cmp dword ptr ss:\[ebp+8\],0  
00477B1B     75 09             jnz short 00477B26  
00477B1D     33C0              xor eax,eax  
  
Shift+F9 运行来到壳的空间的异常处：  
  
0046D280     52                push edx  
0046D281     51                push ecx  
0046D282     53                push ebx  
0046D283     FF50 F4           call dword ptr ds:\[eax\-C\]        ; // 这里发生第 1 次异常：读取\[fffffff4\]  
0046D286     31D2              xor edx,edx  
0046D288     8D4C24 10         lea ecx,dword ptr ss:\[esp+10\]  
0046D28C     64:8B1A           mov ebx,dword ptr fs:\[edx\]  
0046D28F     8919              mov dword ptr ds:\[ecx\],ebx  
0046D291     8969 08           mov dword ptr ds:\[ecx+8\],ebp  
0046D294     C741 04 A5D24600  mov dword ptr ds:\[ecx+4\],46D2A5  
0046D29B     8941 0C           mov dword ptr ds:\[ecx+C\],eax  
0046D29E     64:890A           mov dword ptr fs:\[edx\],ecx  
0046D2A1     5B                pop ebx  
0046D2A2     59                pop ecx  
0046D2A3     5A                pop edx  
0046D2A4     C3                retn  
  
0047CB60     55                push ebp  
0047CB61     68 89CB4700       push 47CB89  
0047CB66     64:FF30           push dword ptr fs:\[eax\]  
0047CB69     64:8920           mov dword ptr fs:\[eax\],esp  
0047CB6C     33C9              xor ecx,ecx  
0047CB6E     B2 01             mov dl,1  
0047CB70     B8 20EE4600       mov eax,46EE20  
0047CB75     E8 AE3CFFFF       call 00470828  
0047CB7A     E8 6D0AFFFF       call 0046D5EC  
0047CB7F     33C0              xor eax,eax                      ; // 第 2 次异常:0EEDFACE  
0047CB81     5A                pop edx  
0047CB82     59                pop ecx  
0047CB83     59                pop ecx  
0047CB84     64:8910           mov dword ptr fs:\[eax\],edx  
0047CB87     EB 0A             jmp short 0047CB93  
  
0047CB95     55                push ebp  
0047CB96     68 BECB4700       push 47CBBE  
0047CB9B     64:FF30           push dword ptr fs:\[eax\]  
0047CB9E     64:8920           mov dword ptr fs:\[eax\],esp  
0047CBA1     33C9              xor ecx,ecx  
0047CBA3     B2 01             mov dl,1  
0047CBA5     B8 20EE4600       mov eax,46EE20  
0047CBAA     E8 793CFFFF       call 00470828  
0047CBAF     E8 380AFFFF       call 0046D5EC  
0047CBB4     33C0              xor eax,eax                      ; // 第 3 次异常:0EEDFACE  
0047CBB6     5A                pop edx  
0047CBB7     59                pop ecx  
0047CBB8     59                pop ecx  
0047CBB9     64:8910           mov dword ptr fs:\[eax\],edx  
0047CBBC     EB 0A             jmp short 0047CBC8  
  
0047CBF7     55                push ebp  
0047CBF8     68 20CC4700       push 47CC20      // seh  
0047CBFD     64:FF30           push dword ptr fs:\[eax\]  
0047CC00     64:8920           mov dword ptr fs:\[eax\],esp  
0047CC03     33C9              xor ecx,ecx  
0047CC05     B2 01             mov dl,1  
0047CC07     B8 20EE4600       mov eax,46EE20  
0047CC0C     E8 173CFFFF       call 00470828  
0047CC11     E8 D609FFFF       call 0046D5EC  
0047CC16     33C0              xor eax,eax                      ; // 第 4 次异常:0EEDFACE，可能算是最后一个异常吧（至少我这里是）  
0047CC18     5A                pop edx  
0047CC19     59                pop ecx  
0047CC1A     59                pop ecx  
0047CC1B     64:8910           mov dword ptr fs:\[eax\],edx  
0047CC1E     EB 0A             jmp short 0047CC2A  
  
到达第 4 个异常时向下不远处找 popad 和 jmp 指令，like this:  
  
0048380A     61                popad                            ; // 这里 eax 将会看到 OEP，下 F2 断点，Shift+F9 通过断于此，下面不远处就是返回 OEP!  
0048380B     EB 01             jmp short 0048380E               ; // 以下都是垃圾指令  
0048380D     E8 50EB02E9       call E94B2362  
00483812     17                pop ss  
00483813     E8 02000000       call 0048381A  
00483818     90                nop                              ; // // 为了看到下面的正确指令，NOP 掉两个字节  
00483819     90                nop  
0048381A     58                pop eax                          ; // 垃圾指令结束  
0048381B     803D 00AA4800 00  cmp byte ptr ds:\[48AA00\],0       ; // 比较是否返回 OEP 标志！正确点是是否显示 Nag 提示！  
00483822     74 23             je short 00483847  
00483824     5B                pop ebx  
00483825     6A 10             push 10  
00483827     B8 CE994800       mov eax,4899CE  
0048382C     83C0 0A           add eax,0A  
0048382F     8B00              mov eax,dword ptr ds:\[eax\]  
00483831     50                push eax  
00483832     B8 DE994800       mov eax,4899DE  
00483837     83E8 0A           sub eax,0A  
0048383A     8B00              mov eax,dword ptr ds:\[eax\]  
0048383C     50                push eax  
0048383D     6A 00             push 0  
0048383F     B8 59594700       mov eax,475959  
00483844     48                dec eax  
00483845     53                push ebx  
00483846     50                push eax  
00483847     C3                retn                             ; // 返回 OEP  
  
retn 单步后到了 OEP: 4010cc 还不快拿工具 DUMP，用 ImportREC  "GET Iat" 全部有效，fix it!  
  
Mission Impassable?就此完成，其实写文章比脱壳还要麻烦麻烦麻烦……^\_^ ^\_^  
  
附：  
  
在 call 00467BA8 里单步不知到哪或 call 后断点会令你无法到达正确的 OEP：  
0047716A    /EB 0A             jmp short 00477176  
0047716C   ^|E9 9B07FFFF       jmp 0046790C  
00477171    |E8 320AFFFF       call 00467BA8                   ; // 这里有 trick: 令你无法到达正确的 OEP，小弟不才，请高手解释……  
00477176    \\8B45 F4           mov eax,dword ptr ss:\[ebp\-C\]  
00477179     0145 F8           add dword ptr ss:\[ebp\-8\],eax    ; // 计算出正确的 OEP！  
0047717C     833D 9CE34700 00  cmp dword ptr ds:\[47E39C\],0  
00477183     0F9405 A0E34700   sete byte ptr ds:\[47E3A0\]  
0047718A     EB 01             jmp short 0047718D  
0047718C     90                nop  
0047718D     8B65 FC           mov esp,dword ptr ss:\[ebp\-4\]    ; // 关键地方： \[ebp-4\]一定要等于 esp 或 Nop 掉它，否则跟到这 OEP 没门!  
00477190     EB 01             jmp short 00477193  
00477192     90                nop  
00477193     8B45 F8           mov eax,dword ptr ss:\[ebp\-8\]  
00477196     EB 02             jmp short 0047719A  
00477198     90                nop  
00477199     90                nop  
0047719A     8B1D 90D34700     mov ebx,dword ptr ds:\[47D390\]  
004771A0     EB 01             jmp short 004771A3  
004771A2     90                nop  
004771A3     89041C            mov dword ptr ss:\[esp+ebx\],eax  
004771A6     EB 02             jmp short 004771AA  
004771A8     90                nop  
004771A9     90                nop  
004771AA     61                popad  
004771AB     90                nop  
004771AC     90                nop  
004771AD     90                nop  
004771AE     90                nop  
004771AF     90                nop  
004771B0     90                nop  
004771B1     90                nop  
004771B2     90                nop  
004771B3     90                nop  
004771B4     90                nop  
004771B5     90                nop  
004771B6     90                nop  
004771B7     90                nop  
004771B8     90                nop  
004771B9     90                nop  
004771BA     90                nop  
004771BB     803D A0E34700 00  cmp byte ptr ds:\[47E3A0\],0      ; // 比较是否返回 OEP 标志！正确点是是否显示 Nag 提示！  
  
  
XP 环境：  
  
bp GetModuleHandleA 3 次运行中断点后，取消断点，Alt+F9 返回到用户代码  
  
001A5E3A     FF95 B4314400   call dword ptr ss:\[ebp+4431B4\]  ; kernel32.GetModuleHandleA  
001A5E40     85C0            test eax,eax                    ; 返回到这  
001A5E42     75 07           jnz short 001A5E4B  
001A5E44     53              push ebx  
001A5E45     FF95 B8314400   call dword ptr ss:\[ebp+4431B8\]  
……  
001A5F55     61              popad  
001A5F56     75 08           jnz short 001A5F60  
001A5F58     B8 01000000     mov eax,1  
001A5F5D     C2 0C00         retn 0C  
001A5F60     68 00000000     push 0  
001A5F65     C3              retn    //上面到这是 aspack 的解压，F4 运行到这  
  
我这里上面的 retn 是 00197FCC  
  
00197FCC     55              push ebp     //F7 单步到这  
00197FCD     8BEC            mov ebp,esp  
00197FCF     83C4 F4         add esp,-0C  
00197FD2     E8 6DFBFEFF     call 00187B44  
00197FD7   ^ 0F85 8F09FFFF   jnz 0018896C  
00197FDD     E8 460EFFFF     call 00188E28  
00197FE2     E8 4537FFFF     call 0018B72C  
00197FE7     E8 1C55FFFF     call 0018D508  
00197FEC     E8 A3AAFFFF     call 00192A94  
00197FF1     E8 7609FFFF     call 0018896C  
00197FF6     8BE5            mov esp,ebp  
00197FF8     5D              pop ebp  
00197FF9     C2 0C00         retn 0C  
00197FFC     0D F0ADBA0D     or eax,0DBAADF0   //你看到了吗？从这里到下面一直延伸相同的“垃圾”  
00198001     F0:AD           lock lods dword ptr ds:\[esi\]    ; 锁定前缀是不允许的  
00198003     BA 0DF0ADBA     mov edx,BAADF00D   //就是它  
00198008     0D F0ADBA0D     or eax,0DBAADF0  
  
cpu 窗口往上滚动到顶部，运行脚本“垃圾铲除脚本”--replace code.txt，按说明运行使用。  
小心你的 CPU，我这里弱机要等 10 多秒左右，期间是忙中，失去响应，是正常现象^\_^  
待脚本完成报告时，一看 count == 281D，还真多呀！  
  
我们此时再看：  
00197FCC     55              push ebp  
00197FCD     8BEC            mov ebp,esp  
00197FCF     83C4 F4         add esp,-0C  
00197FD2     E8 6DFBFEFF     call 00187B44  
00197FD7   ^ 0F85 8F09FFFF   jnz 0018896C  
00197FDD     E8 460EFFFF     call 00188E28  
00197FE2     E8 4537FFFF     call 0018B72C  
00197FE7     E8 1C55FFFF     call 0018D508  
00197FEC     E8 A3AAFFFF     call 00192A94  
00197FF1     E8 7609FFFF     call 0018896C  
00197FF6     8BE5            mov esp,ebp  
00197FF8     5D              pop ebp  
00197FF9     C2 0C00         retn 0C  
00197FFC     0000            add byte ptr ds:\[eax\],al  
00197FFE     0000            add byte ptr ds:\[eax\],al  
00198000     0000            add byte ptr ds:\[eax\],al  
00198002     0000            add byte ptr ds:\[eax\],al  
00198004     0000            add byte ptr ds:\[eax\],al  
  
现在正常了，异常没再死敲敲地非法 EIP 了，可以通过异常正常运行！  
iat 修复不可按我 98 的方式进行（抵达入口可以）-- 有点难度，留给大家练练玩玩，只是异常 0EEDFACE 的位置不同而已，另外垃圾来源好像程序未运行前就存在某 region，不知为何存在的，高手指点，ByteBye!  
  
脚本：  
// replace code.txt  
  
var addr  
var count  
  
mov count,0  
  
ask "Please input address for replace:" //输入 cpu 窗口顶部地址  
cmp $RESULT,0  
je exit  
mov addr,$RESULT  
  
loop:  
find addr, #0DF0ADBA# //你机器看到的垃圾代码  
cmp $RESULT,0  
je exit  
mov \[$RESULT\],0  
inc count  
jmp loop  
  
exit:  
msg "Finished to Replace Job!"  
log "清除垃圾代码统计数量在 count："  
log count  
ret  
  
![](img/620448a77d884bd6b2c4a7cfef437501.jpg) ![](img/620448a77d884bd6b2c4a7cfef437501.jpg) ![](img/620448a77d884bd6b2c4a7cfef437501.jpg)

ollydbg load chap708.exe  
bp LocalAlloc  
F9 run  
  
0012FF98   0040D2F7  /CALL to LocalAlloc  
0012FF9C   00000000  |Flags = LMEM\_FIXED  
0012FFA0   0000A091  \\Size = A091 (41105.)  
F9 run  
  
0012FF94   0040D429  /CALL to LocalAlloc  
0012FF98   00000000  |Flags = LMEM\_FIXED  
0012FF9C   00023000  \\Size = 23000 (143360.)  
F9 run  
  
0012FF94   0040D620  /CALL to LocalAlloc  
0012FF98   00000000  |Flags = LMEM\_FIXED  
0012FF9C   00023000  \\Size = 23000 (143360.)  
  
change this in stack : 0012FF98   00000040  |Flags = LPTR  
  
bc LocalAlloc  
F9 run

0040D000    90              nop  
0040D001 >  60              pushad                                   ; 保存寄存器  
0040D002    E8 01000000     call    0040D008                         ; 计算 delta  
0040D007    90              nop  
0040D008    5D              pop     ebp  
0040D009    81ED BFAF4500   sub     ebp, 45AFBF  
0040D00F    BB B8AF4500     mov     ebx, 45AFB8                      ; 计算当前 Imagebase  
0040D014    03DD            add     ebx, ebp  
0040D016    2B9D 91C34500   sub     ebx, \[ebp+45C391\]                ; ebx = Imagebase  
0040D01C    83BD 8CC24500 0>cmp     dword ptr \[ebp+45C28C\], 0        ; 重入标记  
0040D023    899D F5BF4500   mov     \[ebp+45BFF5\], ebx                ; 保存 Imagebase  
0040D029    0F85 CE100000   jnz     0040E0FD                         ; 重入时跳过一些代码  
0040D02F    8D85 94C24500   lea     eax, \[ebp+45C294\]                ; eax -> "kernel32.dll"  
0040D035    50              push    eax  
0040D036    FF95 D0C34500   call    \[ebp+45C3D0\]                     ; GetModuleHandleA  
0040D03C    8985 90C24500   mov     \[ebp+45C290\], eax                ; 保存 kernel32 句柄  
0040D042    8BF8            mov     edi, eax                         ; edi = kernel32 句柄  
0040D044    8D9D A1C24500   lea     ebx, \[ebp+45C2A1\]                ; ebx -> "LocalAlloc"  
0040D04A    53              push    ebx  
0040D04B    50              push    eax  
0040D04C    FF95 CCC34500   call    \[ebp+45C3CC\]                     ; GetProcAddress  
0040D052    8985 99C34500   mov     \[ebp+45C399\], eax                ; 保存 API 地址  
0040D058    8D9D ACC24500   lea     ebx, \[ebp+45C2AC\]                ; ebx -> "LocalFree"  
0040D05E    53              push    ebx  
0040D05F    57              push    edi  
0040D060    FF95 CCC34500   call    \[ebp+45C3CC\]                     ; GetProcAddress  
0040D066    8985 9DC34500   mov     \[ebp+45C39D\], eax                ; 保存 API 地址  
0040D06C    8B85 F5BF4500   mov     eax, \[ebp+45BFF5\]                ; eax -> Imagebase  
0040D072    8985 8CC24500   mov     \[ebp+45C28C\], eax                ; 复制到另一个变量  
0040D078   /EB 4B           jmp     short 0040D0C5                   ; 执行一些代码, 跟过去  
  
0040D0C5    54              push    esp                              ; 保存 ESP  
0040D0C6    8F85 86B04500   pop     dword ptr \[ebp+45B086\]  
0040D0CC    EB 04           jmp     short 0040D0D2                   ; jmp 后紧跟变量, 后面很多  
0040D0CE    0012FFA4                                                 ; 这种写法看起来会比较紧凑, 方便作者找变量  
0040D0D2    FFB5 86B04500   push    dword ptr \[ebp+45B086\]  
0040D0D8    8F85 2BBC4500   pop     dword ptr \[ebp+45BC2B\]  
0040D0DE    FFB5 8CC24500   push    dword ptr \[ebp+45C28C\]  
0040D0E4    8F85 A4B04500   pop     dword ptr \[ebp+45B0A4\]  
  
0040D0D2    FFB5 86B04500   push    dword ptr \[ebp+45B086\]           ; 复制一些变量  
0040D0D8    8F85 2BBC4500   pop     dword ptr \[ebp+45BC2B\]  
0040D0DE    FFB5 8CC24500   push    dword ptr \[ebp+45C28C\]  
0040D0E4    8F85 A4B04500   pop     dword ptr \[ebp+45B0A4\]  
  
0040D0F0    FFB5 A4B04500   push    dword ptr \[ebp+45B0A4\]  
0040D0F6    5B              pop     ebx  
0040D0F7    53              push    ebx  
0040D0F8    8F85 B8B04500   pop     dword ptr \[ebp+45B0B8\]  
  
0040D104    FFB5 B8B04500   push    dword ptr \[ebp+45B0B8\]  
0040D10A    8F85 0FBC4500   pop     dword ptr \[ebp+45BC0F\]  
0040D110    FFB5 90C24500   push    dword ptr \[ebp+45C290\]  
0040D116    8F85 D6B04500   pop     dword ptr \[ebp+45B0D6\]  
  
0040D122    FFB5 D6B04500   push    dword ptr \[ebp+45B0D6\]           ; kernel32.77E40000  
0040D128    58              pop     eax                              ; eax = kernel32 句柄  
0040D129    8D9D E5BB4500   lea     ebx, \[ebp+45BBE5\]                ; ebx -> "GetTickCount"  
  
0040D12F    EB 04           jmp     short 0040D135                   ; 这里开始有一些花指令  
0040D131    E8 EB04E9EB     call    EC29D621  
0040D136    FB              sti  
0040D137    E9 50EB04E8     jmp     E845BC8C  
0040D13C    EB 04           jmp     short 0040D142  
0040D13E    E9 EBFBE9E8     jmp     E92ACD2E  
......  
  
; 杀去比较容易跟踪, 分析一下这些垃圾.  
;  
; 花指令是组合使用的, 一种多一些的是:  
;  
;       jmp     short loc\_01  
;       db      0E8h  
;loc\_02:  
;       jmp     short loc\_03  
;       db      0E9h  
;loc\_01:  
;       jmp     short loc\_02  
;loc\_03:  
;  
;S = EB04??EB04??EBFB??  
;R = 909090909090909090  
;  
; 还有一种用上面的加一些东西  
;  
;       <1>  
;       push    eax  
;       <1>  
;       call    loc\_01  
;       db      0EBh, 01h  
;       db      0E8h  
;loc\_01:  
;       pop     eax  
;       <1>  
;       pop     eax  
;  
;S = EB04??EB04??EBFB??50EB04??EB04??EBFB??E803000000??????58EB04??EB04??EBFB??58  
;R = 9090909090909090909090909090909090909090909090909090909090909090909090909090  
;  
; 有时候把 eax 换成 ebx 的  
;  
;       <1>  
;       push    ebx  
;       <1>  
;       call    loc\_01  
;       db      0EBh, 01h  
;       db      0E8h  
;loc\_01:  
;       pop     ebx  
;       <1>  
;       pop     ebx  
;  
;S = EB04??EB04??EBFB??53EB04??EB04??EBFB??E803000000??????5BEB04??EB04??EBFB??5B  
;R = 9090909090909090909090909090909090909090909090909090909090909090909090909090  
;  
  
0040D12F    90              nop                                      ; 这里清静多了  
0040D130    90              nop  
0040D131    90              nop  
0040D132    90              nop  
0040D133    90              nop  
  
; 现在我们来看一下保护系统如何分配得到随机内存地址, 使每次跟踪地址都会变化  
  
0040D15E    53              push    ebx                              ; chap708.0040DC2D "GetTickCount"  
0040D1BD    50              push    eax                              ; kernel32.77E40000  
0040D1ED    8D85 B3B14500   lea     eax, \[ebp+45B1B3\]                ; 指向后面的 retn  
0040D1F3    40              inc     eax                              ; 地址加 1, 把 retn 后边作为函数返回地址  
0040D1F4    50              push    eax                              ; 把返回地址压栈  
0040D1F5    FFB5 CCC34500   push    dword ptr \[ebp+45C3CC\]           ; GetProcAddress 地址压栈  
0040D1FB    C3              retn                                     ; 模拟 call 指令  
                                                                     ; 一种比较特殊的调用方式  
  
0040D1FC    50              push    eax                              ; kernel32.GetTickCount  
0040D1FD    8F85 BDB14500   pop     dword ptr \[ebp+45B1BD\]  
  
0040D209    FFB5 BDB14500   push    dword ptr \[ebp+45B1BD\]           ; kernel32.GetTickCount  
0040D20F    8F85 F2BB4500   pop     dword ptr \[ebp+45BBF2\]  
0040D215    8D85 DBB14500   lea     eax, \[ebp+45B1DB\]  
0040D21B    40              inc     eax  
0040D21C    50              push    eax  
0040D21D    FFB5 F2BB4500   push    dword ptr \[ebp+45BBF2\]           ; kernel32.GetTickCount  
0040D223    C3              retn  
                                                                     ; 取一个随机数  
0040D224    25 FFFF0100     and     eax, 1FFFF                       ; 去掉高四位  
0040D258    50              push    eax             ; 结果入栈  
  
0040D2B7    6A 00           push    0  
  
0040D2E8    8D85 AEB24500   lea     eax, \[ebp+45B2AE\]  
0040D2EE    40              inc     eax  
0040D2EF    50              push    eax  
0040D2F0    FFB5 99C34500   push    dword ptr \[ebp+45C399\]           ; kernel32.LocalAlloc  
0040D2F6    C3              retn                                     ; 分配内存  
  
;0012FF98   0040D2F7  /CALL to LocalAlloc  
;0012FF9C   00000000  |Flags = LMEM\_FIXED  
;0012FFA0   0000D000  \\Size = D000 (53248.)                          ; 虚拟的大小, 用来占用地址, 使真正分配得的地址随机  
  
0040D326    50              push    eax                              ; 用堆栈保存分配到的内存  
  
0040D385    FFB5 0ABC4500   push    dword ptr \[ebp+45BC0A\]           ; 存放某个内存块的大小, 这才是实际需要的内存大小  
0040D3E9    6A 00           push    0  
  
0040D41A    8D85 E0B34500   lea     eax, \[ebp+45B3E0\]  
0040D420    40              inc     eax  
0040D421    50              push    eax  
0040D422    FFB5 99C34500   push    dword ptr \[ebp+45C399\]           ; kernel32.LocalAlloc  
0040D428    C3              retn                                     ; 再次分配  
  
0040D429    50              push    eax  
0040D42A    8F85 EAB34500   pop     dword ptr \[ebp+45B3EA\]  
  
0040D436    FFB5 EAB34500   push    dword ptr \[ebp+45B3EA\]  
0040D43C    8F85 95C34500   pop     dword ptr \[ebp+45C395\]  
0040D442    FFB5 02BC4500   push    dword ptr \[ebp+45BC02\]  
0040D448    8F85 08B44500   pop     dword ptr \[ebp+45B408\]  
  
0040D454    FFB5 08B44500   push    dword ptr \[ebp+45B408\]           ; 可能是压缩后的一部分外壳  
0040D45A    5B              pop     ebx  
0040D45B    039D 0FBC4500   add     ebx, \[ebp+45BC0F\]         ; +Imagebase  
  
0040D490    50              push    eax  
  
0040D4EF    53              push    ebx                              ; chap708.004114B8  
  
0040D51F    8D85 4BC14500   lea     eax, \[ebp+45C14B\]                ; aPLib 的解压缩函数  
0040D525    50              push    eax  
0040D526    8F85 E6B44500   pop     dword ptr \[ebp+45B4E6\]  
  
0040D532    FFB5 E6B44500   push    dword ptr \[ebp+45B4E6\]  
0040D538    8F85 F6BB4500   pop     dword ptr \[ebp+45BBF6\]  
0040D53E    8D85 04B54500   lea     eax, \[ebp+45B504\]  
0040D544    40              inc     eax  
0040D545    50              push    eax  
0040D546    FFB5 F6BB4500   push    dword ptr \[ebp+45BBF6\]           ; chap708.0040E193  
0040D54C    C3              retn                                     ; 解压缩  
  
0040D57C    FFB5 0ABC4500   push    dword ptr \[ebp+45BC0A\]  
  
0040D5E0    6A 00           push    0  
  
0040D611    8D85 D7B54500   lea     eax, \[ebp+45B5D7\]  
0040D617    40              inc     eax  
0040D618    50              push    eax  
0040D619    FFB5 99C34500   push    dword ptr \[ebp+45C399\]           ; kernel32.LocalAlloc  
0040D61F    C3              retn                                     ; 第三次分配内存  
  
  
0040D620    50              push    eax  
0040D621    8F85 E1B54500   pop     dword ptr \[ebp+45B5E1\]  
  
0040D62D    FFB5 E1B54500   push    dword ptr \[ebp+45B5E1\]  
0040D633    8F85 E1BB4500   pop     dword ptr \[ebp+45BBE1\]  
0040D639    8D85 FFB54500   lea     eax, \[ebp+45B5FF\]  
0040D63F    40              inc     eax  
0040D640    50              push    eax  
0040D641    FFB5 9DC34500   push    dword ptr \[ebp+45C39D\]           ; kernel32.LocalFree  
0040D647    C3              retn                                     ; 释放最初分配的内存  
  
0040D648    FFB5 95C34500   push    dword ptr \[ebp+45C395\]  
0040D64E    8F85 0EB64500   pop     dword ptr \[ebp+45B60E\]  
  
0040D65A    FFB5 0EB64500   push    dword ptr \[ebp+45B60E\]  
0040D660    5A              pop     edx  
0040D661    68 F8010000     push    1F8  
0040D666    8F85 26B64500   pop     dword ptr \[ebp+45B626\]  
  
0040D672    FFB5 26B64500   push    dword ptr \[ebp+45B626\]  
0040D678    5B              pop     ebx  
0040D679    8B7C1A 0C       mov     edi, \[edx+ebx+C\]                 ; 一个表头  
0040D67D    0BFF            or      edi, edi  
0040D67F    74 1E           je      short 0040D69F  
0040D681    8B4C1A 10       mov     ecx, \[edx+ebx+10\]                ; 大小  
0040D685    0BC9            or      ecx, ecx  
0040D687    74 11           je      short 0040D69A  
  
0040D689    03BD E1BB4500   add     edi, \[ebp+45BBE1\]  
0040D68F    8B741A 14       mov     esi, \[edx+ebx+14\]  
0040D693    03F2            add     esi, edx  
0040D695    C1F9 02         sar     ecx, 2  
0040D698    F3:A5           rep     movs dword ptr es:\[edi\], dword ptr \[esi\]  
0040D69A    83C3 28         add     ebx, 28                          ; 下一个段  
0040D69D  ^ EB DA           jmp     short 0040D679  
  
0040D69F    FFB5 95C34500   push    dword ptr \[ebp+45C395\]           ; 用极端无聊的方式把很多东西复制  
0040D6A5    8F85 65B64500   pop     dword ptr \[ebp+45B665\]           ; 从 ebp + 导入 ebx +  
  
0040D6B1    FFB5 65B64500   push    dword ptr \[ebp+45B665\]  
0040D6B7    58              pop     eax  
  
0040D6E7    50              push    eax  
  
0040D717    FFB5 E1BB4500   push    dword ptr \[ebp+45BBE1\]  
0040D71D    8F85 DDB64500   pop     dword ptr \[ebp+45B6DD\]  
  
0040D729    FFB5 DDB64500   push    dword ptr \[ebp+45B6DD\]  
0040D72F    5A              pop     edx  
  
0040D75F    52              push    edx  
  
0040D78F    FF30            push    dword ptr \[eax\]  
0040D791    8F85 51B74500   pop     dword ptr \[ebp+45B751\]  
  
0040D79D    FFB5 51B74500   push    dword ptr \[ebp+45B751\]  
0040D7A3    5B              pop     ebx  
0040D7A4    03DA            add     ebx, edx  
0040D7A6    FFB5 CCC34500   push    dword ptr \[ebp+45C3CC\]  
0040D7AC    8F85 6CB74500   pop     dword ptr \[ebp+45B76C\]  
  
0040D7B8    FFB5 6CB74500   push    dword ptr \[ebp+45B76C\]  
0040D7BE    58              pop     eax  
0040D7BF    50              push    eax  
0040D7C0    8F85 80B74500   pop     dword ptr \[ebp+45B780\]                     ; kernel32.GetProcAddress  
  
0040D7CC    FFB5 80B74500   push    dword ptr \[ebp+45B780\]                     ; kernel32.GetProcAddress  
0040D7D2    8F03            pop     dword ptr \[ebx\]  
0040D7D4    FFB5 D0C34500   push    dword ptr \[ebp+45C3D0\]  
0040D7DA    8F85 9AB74500   pop     dword ptr \[ebp+45B79A\]  
  
0040D7E6    FFB5 9AB74500   push    dword ptr \[ebp+45B79A\]                     ; kernel32.GetModuleHandleA  
0040D7EC    58              pop     eax  
0040D7ED    50              push    eax  
0040D7EE    8F85 AEB74500   pop     dword ptr \[ebp+45B7AE\]  
  
0040D7FA    FFB5 AEB74500   push    dword ptr \[ebp+45B7AE\]                     ; kernel32.GetModuleHandleA  
0040D800    8F43 04         pop     dword ptr \[ebx+4\]  
0040D803    FFB5 D4C34500   push    dword ptr \[ebp+45C3D4\]  
0040D809    8F85 C9B74500   pop     dword ptr \[ebp+45B7C9\]  
  
0040D815    FFB5 C9B74500   push    dword ptr \[ebp+45B7C9\]                     ; kernel32.LoadLibraryA  
0040D81B    58              pop     eax  
0040D81C    50              push    eax  
0040D81D    8F85 DDB74500   pop     dword ptr \[ebp+45B7DD\]  
  
0040D829    FFB5 DDB74500   push    dword ptr \[ebp+45B7DD\]                     ; kernel32.LoadLibraryA  
0040D82F    8F43 08         pop     dword ptr \[ebx+8\]  
0040D832    5F              pop     edi  
0040D833    5E              pop     esi  
0040D834    FF76 04         push    dword ptr \[esi+4\]  
0040D837    8F85 F7B74500   pop     dword ptr \[ebp+45B7F7\]  
  
0040D843    FFB5 F7B74500   push    dword ptr \[ebp+45B7F7\]  
0040D849    58              pop     eax                                        ; kernel32.LoadLibraryA  
0040D84A    03C7            add     eax, edi  
  
0040D87B    50              push    eax  
0040D8AB    8D9D 0FBC4500   lea     ebx, \[ebp+45BC0F\]  
0040D8E0    53              push    ebx                                        ; chap708.0040DC57  
0040D93F    6A 00           push    0  
0040D99F    6A 00           push    0  
0040D9FF    6A 01           push    1  
0040DA5F    57              push    edi  
  
0040DA8F    FF76 08         push    dword ptr \[esi+8\]  
0040DA92    8F85 52BA4500   pop     dword ptr \[ebp+45BA52\]  
  
0040DA9E    FFB5 52BA4500   push    dword ptr \[ebp+45BA52\]  
0040DAA4    5B              pop     ebx  
0040DAA5    03DF            add     ebx, edi  
  
0040DAD6    53              push    ebx  
0040DB35    56              push    esi                                        ; 释放地址  
  
0040DB94    50              push    eax                                        ; 第二段外壳代码, API 结束后返回  
0040DBF3    FFB5 9DC34500   push    dword ptr \[ebp+45C39D\]                     ; kernel32.LocalFree  
0040DC28    C3              retn                                               ; 释放内存  
  
; 现在我们进入了第二段  
00195311    60              pushad  
00195312    E8 44060000     call    0019595B  
  
0019595B    8B2C24          mov     ebp, \[esp\]  
0019595E    81ED 0F294400   sub     ebp, 44290F  
00195964    C3              retn  
  
00195317   /EB 44           jmp     short 0019535D  
0019535D    BB 08294400     mov     ebx, 442908  
00195362    03DD            add     ebx, ebp  
00195364    2B9D 35294400   sub     ebx, \[ebp+442935\]  
0019536A    83BD A0304400 0>cmp     dword ptr \[ebp+4430A0\], 0  
00195371    899D F72D4400   mov     \[ebp+442DF7\], ebx                          ; 重入标记?  
00195377    0F85 42050000   jnz     001958BF  
  
0019537D    8D85 A8304400   lea     eax, \[ebp+4430A8\]  
00195383    50              push    eax  
00195384    FF95 B4314400   call    \[ebp+4431B4\]                               ; kernel32.GetModuleHandleA  
;0012FF60   0019538A  /CALL to GetModuleHandleA from 00195384  
;0012FF64   00195AB0  \\pModule = "kernel32.dll"  
  
0019538A    8985 A4304400   mov     \[ebp+4430A4\], eax                          ; kernel32.77E40000  
00195390    8BF8            mov     edi, eax  
00195392    8D9D B5304400   lea     ebx, \[ebp+4430B5\]  
00195398    53              push    ebx  
00195399    50              push    eax  
0019539A    FF95 B0314400   call    \[ebp+4431B0\]                               ; kernel32.GetProcAddress  
;0012FF5C   001953A0  /CALL to GetProcAddress from 0019539A  
;0012FF60   77E40000  |hModule = 77E40000 (kernel32)  
;0012FF64   00195ABD  \\ProcNameOrOrdinal = "VirtualAlloc"  
;  
  
  
001953A0    8985 3D294400   mov     \[ebp+44293D\], eax  
001953A6    8D9D C2304400   lea     ebx, \[ebp+4430C2\]  
001953AC    53              push    ebx  
001953AD    57              push    edi  
001953AE    FF95 B0314400   call    \[ebp+4431B0\]                               ; kernel32.GetProcAddress  
;0012FF5C   001953B4  /CALL to GetProcAddress from 001953AE  
;0012FF60   77E40000  |hModule = 77E40000 (kernel32)  
;0012FF64   00195ACA  \\ProcNameOrOrdinal = "VirtualFree"  
;  
  
001953B4    8985 41294400   mov     \[ebp+442941\], eax  
001953BA    8B85 F72D4400   mov     eax, \[ebp+442DF7\]  
001953C0    8985 A0304400   mov     \[ebp+4430A0\], eax  
001953C6    6A 04           push    4  
001953C8    68 00100000     push    1000  
001953CD    68 4A050000     push    54A  
001953D2    6A 00           push    0  
001953D4    FF95 3D294400   call    \[ebp+44293D\]                               ; kernel32.VirtualAlloc  
  
001953DA    8985 39294400   mov     \[ebp+442939\], eax  
001953E0    8D9D 092A4400   lea     ebx, \[ebp+442A09\]  
001953E6    50              push    eax  
001953E7    53              push    ebx  
001953E8    E8 78050000     call    00195965                                   ; 解压缩  
001953ED    8BC8            mov     ecx, eax  
001953EF    8DBD 092A4400   lea     edi, \[ebp+442A09\]                          ; 指向后面 call 之后  
001953F5    8BB5 39294400   mov     esi, \[ebp+442939\]  
001953FB    F3:A4           rep     movs byte ptr es:\[edi\], byte ptr \[esi\]     ; 复制后半段外壳解压缩后的代码  
001953FD    8B85 39294400   mov     eax, \[ebp+442939\]  
00195403    68 00800000     push    8000  
00195408    6A 00           push    0  
0019540A    50              push    eax  
0019540B    FF95 41294400   call    \[ebp+442941\]                               ; 释放内存  
  
; 这里揭开之后的样子  
00195411    8D85 152C4400   lea     eax, \[ebp+442C15\]  
00195417    50              push    eax  
00195418    C3              retn                                               ; 跳转  
  
0019561D    8B9D 192A4400   mov     ebx, \[ebp+442A19\]  
00195623    0BDB            or      ebx, ebx  
00195625    74 0A           je      short 00195631                             ; 我这里没有这个选项，走  
  
00195631    8DB5 352A4400   lea     esi, \[ebp+442A35\]                          ; 开始解压段  
00195637    833E 00         cmp     dword ptr \[esi\], 0  
0019563A    0F84 D3000000   je      00195713  
\*  
001956A9   /74 08           je      short 001956B3  
001956AB   |3C E9           cmp     al, 0E9  
001956AD   |74 04           je      short 001956B3  
001956AF   |43              inc     ebx  
001956B0   |49              dec     ecx  
001956B1  ^|EB ED           jmp     short 001956A0  
001956B3   \\291E            sub     \[esi\], ebx  
001956B5    83C3 05         add     ebx, 5  
001956B8    83C6 04         add     esi, 4  
001956BB    83E9 05         sub     ecx, 5  
001956BE  ^ EB E0           jmp     short 001956A0  
001956C0    5B              pop     ebx  
001956C1    5E              pop     esi  
001956C2    59              pop     ecx  
001956C3    58              pop     eax  
001956C4    8BC8            mov     ecx, eax  
001956C6    8B3E            mov     edi, \[esi\]  
001956C8    03BD A0304400   add     edi, \[ebp+4430A0\]  
001956CE    8BB5 39294400   mov     esi, \[ebp+442939\]  
001956D4    C1F9 02         sar     ecx, 2  
001956D7    F3:A5           rep     movs dword ptr es:\[edi\], dword ptr \[esi\]  
001956D9    8BC8            mov     ecx, eax  
001956DB    83E1 03         and     ecx, 3  
001956DE    F3:A4           rep     movs byte ptr es:\[edi\], byte ptr \[esi\]  
001956E0    5E              pop     esi  
001956E1    8B85 39294400   mov     eax, \[ebp+442939\]  
001956E7    68 00800000     push    8000  
001956EC    6A 00           push    0  
001956EE    50              push    eax  
001956EF    FF95 41294400   call    \[ebp+442941\]  
001956F5    83C6 08         add     esi, 8  
001956F8    833E 00         cmp     dword ptr \[esi\], 0  
001956FB  ^ 0F85 45FFFFFF   jnz     00195646  
00195701    8B9D 192A4400   mov     ebx, \[ebp+442A19\]  
00195707    0BDB            or      ebx, ebx  
00195709    74 08           je      short 00195713  
0019570B    8B03            mov     eax, \[ebx\]  
0019570D    8785 1D2A4400   xchg    \[ebp+442A1D\], eax  
  
; 然后处理重定位  
00195713    8B95 A0304400   mov     edx, \[ebp+4430A0\]  
00195719    8B85 152A4400   mov     eax, \[ebp+442A15\]  
0019571F    2BD0            sub     edx, eax  
00195721    74 79           je      short 0019579C  
00195723    8BC2            mov     eax, edx  
00195725    C1E8 10         shr     eax, 10  
00195728    33DB            xor     ebx, ebx  
0019572A    8BB5 212A4400   mov     esi, \[ebp+442A21\]  
00195730    03B5 A0304400   add     esi, \[ebp+4430A0\]  
00195736    833E 00         cmp     dword ptr \[esi\], 0  
00195739    74 61           je      short 0019579C  
0019573B    8B4E 04         mov     ecx, \[esi+4\]  
0019573E    83E9 08         sub     ecx, 8  
00195741    D1E9            shr     ecx, 1  
00195743    8B3E            mov     edi, \[esi\]  
00195745    03BD A0304400   add     edi, \[ebp+4430A0\]  
0019574B    83C6 08         add     esi, 8  
0019574E    66:8B1E         mov     bx, \[esi\]  
00195751    C1EB 0C         shr     ebx, 0C  
00195754    83FB 01         cmp     ebx, 1  
00195757    74 0C           je      short 00195765  
00195759    83FB 02         cmp     ebx, 2  
0019575C    74 16           je      short 00195774  
0019575E    83FB 03         cmp     ebx, 3  
00195761    74 20           je      short 00195783  
00195763    EB 2C           jmp     short 00195791  
00195765    66:8B1E         mov     bx, \[esi\]  
00195768    81E3 FF0F0000   and     ebx, 0FFF  
0019576E    66:01041F       add     \[edi+ebx\], ax  
00195772    EB 1D           jmp     short 00195791  
00195774    66:8B1E         mov     bx, \[esi\]  
00195777    81E3 FF0F0000   and     ebx, 0FFF  
0019577D    66:01141F       add     \[edi+ebx\], dx  
00195781    EB 0E           jmp     short 00195791  
00195783    66:8B1E         mov     bx, \[esi\]  
00195786    81E3 FF0F0000   and     ebx, 0FFF  
0019578C    01141F          add     \[edi+ebx\], edx  
0019578F    EB 00           jmp     short 00195791  
00195791    66:830E FF      or      word ptr \[esi\], 0FFFF  
00195795    83C6 02         add     esi, 2  
00195798  ^ E2 B4           loopd   short 0019574E  
0019579A  ^ EB 9A           jmp     short 00195736  
  
; 接着是 输入表  
  
0019579C    8BB5 252A4400   mov     esi, \[ebp+442A25\]  
001957A2    8B95 A0304400   mov     edx, \[ebp+4430A0\]  
001957A8    03F2            add     esi, edx  
001957AA    8B46 0C         mov     eax, \[esi+C\]  
001957AD    85C0            test    eax, eax  
001957AF    0F84 0A010000   je      001958BF  
  
001957B5    03C2            add     eax, edx  
001957B7    8BD8            mov     ebx, eax  
001957B9    50              push    eax  
001957BA    FF95 B4314400   call    \[ebp+4431B4\]  
001957C0    85C0            test    eax, eax                                   ; kernel32.77E40000  
001957C2    75 07           jnz     short 001957CB  
001957C4    53              push    ebx  
001957C5    FF95 B8314400   call    \[ebp+4431B8\]  
001957CB    8985 11294400   mov     \[ebp+442911\], eax  
001957D1    C785 15294400 0>mov     dword ptr \[ebp+442915\], 0  
001957DB    8B95 A0304400   mov     edx, \[ebp+4430A0\]  
001957E1    8B06            mov     eax, \[esi\]  
001957E3    85C0            test    eax, eax  
001957E5    75 03           jnz     short 001957EA  
001957E7    8B46 10         mov     eax, \[esi+10\]  
001957EA    03C2            add     eax, edx  
001957EC    0385 15294400   add     eax, \[ebp+442915\]  
001957F2    8B18            mov     ebx, \[eax\]  
001957F4    8B7E 10         mov     edi, \[esi+10\]  
001957F7    03FA            add     edi, edx  
001957F9    03BD 15294400   add     edi, \[ebp+442915\]  
001957FF    85DB            test    ebx, ebx  
00195801    0F84 A2000000   je      001958A9  
00195807    F7C3 00000080   test    ebx, 80000000  
0019580D    75 04           jnz     short 00195813  
0019580F    03DA            add     ebx, edx  
00195811    43              inc     ebx  
00195812    43              inc     ebx  
00195813    53              push    ebx  
00195814    81E3 FFFFFF7F   and     ebx, 7FFFFFFF  
0019581A    53              push    ebx  
0019581B    FFB5 11294400   push    dword ptr \[ebp+442911\]  
00195821    FF95 B0314400   call    \[ebp+4431B0\]  
00195827    85C0            test    eax, eax  
00195829    5B              pop     ebx  
0019582A    75 6F           jnz     short 0019589B  
0019582C    F7C3 00000080   test    ebx, 80000000  
00195832    75 19           jnz     short 0019584D  
00195834    57              push    edi  
00195835    8B46 0C         mov     eax, \[esi+C\]  
00195838    0385 A0304400   add     eax, \[ebp+4430A0\]  
0019583E    50              push    eax  
0019583F    53              push    ebx  
00195840    8D85 1B314400   lea     eax, \[ebp+44311B\]  
00195846    50              push    eax  
00195847    57              push    edi  
00195848    E9 99000000     jmp     001958E6  
0019584D    81E3 FFFFFF7F   and     ebx, 7FFFFFFF  
00195853    8B85 A4304400   mov     eax, \[ebp+4430A4\]  
00195859    3985 11294400   cmp     \[ebp+442911\], eax  
0019585F    75 24           jnz     short 00195885  
00195861    57              push    edi  
00195862    8BD3            mov     edx, ebx  
00195864    4A              dec     edx  
00195865    C1E2 02         shl     edx, 2  
00195868    8B9D 11294400   mov     ebx, \[ebp+442911\]  
0019586E    8B7B 3C         mov     edi, \[ebx+3C\]  
00195871    8B7C3B 78       mov     edi, \[ebx+edi+78\]  
00195875    035C3B 1C       add     ebx, \[ebx+edi+1C\]  
00195879    8B0413          mov     eax, \[ebx+edx\]  
0019587C    0385 11294400   add     eax, \[ebp+442911\]  
00195882    5F              pop     edi  
00195883    EB 16           jmp     short 0019589B  
00195885    57              push    edi  
00195886    8B46 0C         mov     eax, \[esi+C\]  
00195889    0385 A0304400   add     eax, \[ebp+4430A0\]  
0019588F    50              push    eax  
00195890    53              push    ebx  
00195891    8D85 6C314400   lea     eax, \[ebp+44316C\]  
00195897    50              push    eax  
00195898    57              push    edi  
00195899    EB 4B           jmp     short 001958E6  
0019589B    8907            mov     \[edi\], eax  
0019589D    8385 15294400 0>add     dword ptr \[ebp+442915\], 4  
001958A4  ^ E9 32FFFFFF     jmp     001957DB  
001958A9    8906            mov     \[esi\], eax  
001958AB    8946 0C         mov     \[esi+C\], eax  
001958AE    8946 10         mov     \[esi+10\], eax  
001958B1    83C6 14         add     esi, 14  
001958B4    8B95 A0304400   mov     edx, \[ebp+4430A0\]  
001958BA  ^ E9 EBFEFFFF     jmp     001957AA  
001958BF    8B85 292A4400   mov     eax, \[ebp+442A29\]  
001958C5    50              push    eax  
001958C6    0385 A0304400   add     eax, \[ebp+4430A0\]  
001958CC    5B              pop     ebx  
001958CD    0BDB            or      ebx, ebx  
001958CF    8985 D92E4400   mov     \[ebp+442ED9\], eax  
  
; 什么东西在招手  
001958D5    61              popad  
001958D6    75 08           jnz     short 001958E0  
001958D8   |B8 01000000     mov     eax, 1  
001958DD   |C2 0C00         retn    0C  
001958E0   \\68 4C791800     push    18794C  
001958E5    C3              retn  
  
; 看起来很可怕  
0018794C    55              push    ebp  
0018794D    8BEC            mov     ebp, esp  
0018794F    83C4 F4         add     esp, -0C  
00187952    E8 6DFBFEFF     call    001774C4  
00187957  ^ 0F85 8F09FFFF   jnz     001782EC  
0018795D    E8 460EFFFF     call    001787A8  
00187962    E8 4537FFFF     call    0017B0AC  
00187967    E8 1C55FFFF     call    0017CE88  
0018796C    E8 A3AAFFFF     call    00182414  
00187971    E8 7609FFFF     call    001782EC  
00187976    8BE5            mov     esp, ebp  
00187978    5D              pop     ebp  
00187979    C2 0C00         retn    0C  
  
; 这里贸然 f4 了一下结果丢了, 后面地址不一样了:((((  
; 重到这里， 第一个 call 就过不去  
  
; 问题是这里  
0016DBD4    58              pop     eax  
0016DBD5    8B15 FC391800   mov     edx, \[1839FC\]  
0016DBDB    85D2            test    edx, edx  
0016DBDD    74 02           je      short 0016DBE1  
0016DBDF    FFD2            call    edx            ; 如果内容初始化了（!=0)就会 call  
0016DBE1    A0 E83D1800     mov     al, \[183DE8\]  
0016DBE6    80F8 02         cmp     al, 2  
0016DBE9    C3              retn  
  
  
;这就是 heXer 说得问题  
  
;剩下就没啥了

Magic debug values  
Magic debug values are specific values written to memory during allocation or deallocation, so that it will later be possible to tell whether or not they have become corrupted and to make it obvious when values taken from uninitialized memory are being used.  
  
Memory is usually viewed in hexadecimal, so common values used are often repeated digits or hexspeak.  
  
Famous and common examples include:  
  
0xBAADF00D   
0xBAADFEED   
0xBADBADBADBAD   
Burroughs B6700 "uninitialized" memory (48-bit words)   
0xC0EDBABE   
0xC001D00D   
0xCCCCCCCC   
Used by Microsoft's C++ compiler to mark uninitialised stack areas in debug mode.   
0xCDCDCDCD   
Used by Microsoft's C++ debugging heap to mark uninitialised heap areas.   
0xDDDDDDDD   
Used by MicroQuill's SmartHeap and Microsoft's C++ debugging heap to mark memory returned to the heap.   
0xDEADBEEF   
Famously used on IBM systems such as the RS/6000, also in OPENSTEP Enterprise and the Commodore Amiga.   
0xEBEBEBEB   
From MicroQuill's SmartHeap.   
OxFACADE   
Used by a number of real-time OS's   
0xFD   
Used by Microsoft's C++ debugging heap to mark guard bytes in the heap.   
0xFEEEFEEE   
Used by Microsoft's C++ compiler to mark the storage area of a deleted class in debug mode.   
Note that most of these are each 8 nybbles (32 bits) long, as most modern computers are designed to manipulate 32 bits at a time.  
  
The prevalence of these values in Microsoft technology is no coincidence; they are discussed in detail in Steve McGuire's well-known book Writing Solid Code from Microsoft Press. He gives a variety of criteria for these values, such as:  
  
They should not be useful; that is, most algorithms that operate on them should be expected to do something unusual. Numbers like zero don't fit this criterion.   
They should be easily recognized by the programmer as invalid values in the debugger.   
On machines that don't have byte alignment, they should be odd, so that dereferencing them as addresses causes an exception.   
They should cause an exception, or perhaps even a debugger break, if executed as code.   
Since they were often used to mark areas of memory that were essentially empty, some of these terms came to be used in phrases meaning "gone, aborted, flushed from memory"; e.g. "Your program is DEADBEEF".

 4dd466d0e1e0cc123afbe504da2462ca