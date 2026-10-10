

# VB 程序内存注册机

> 原文：[`www.pediy.com/kssd/pediy05/pediy50245.htm`](https://www.pediy.com/kssd/pediy05/pediy50245.htm)

1.用 PEid 检查:软件为 VB 程序   
2.使用 SMARTCHECK 和 kWdsm 载入分析   
3.使用 Keymake 1.73 制作内存注册机   
///////////////////////////////////////////////////////////////////   
用 SMARTCHECK 我们可以看到下面内容   
OFFSET:000688C0      <<<<<<<<<<<<<注意这个偏移量，可以帮助我们找到断点 004688XX  :)   
Len(String:"10311820...") returns LONG:35   
  
Arguments   
\--------------------   
String  string1 = 00163E0C   
      = "10311820................."<<<<<<<<<这可是我们想要的东西   
\-----------------------------------------------------------------   
用 kWdsm 载入跟踪调试   
:004686D7 8D4DAC                  lea ecx, dword ptr \[ebp\-54\]   
  
\* Reference To: MSVBVM60.\_\_vbaFreeVar, Ord:0000h   
                                  |   
:004686DA FF1524104000            Call dword ptr \[00401024\]   
:004686E0 8B45C4                  mov eax, dword ptr \[ebp\-3C\]   
:004686E3 50                      push eax   
  
\* Reference To: MSVBVM60.\_\_vbaLenBstr, Ord:0000h   
                                  |   
:004686E4 FF152C104000            Call dword ptr \[0040102C\]   
:004686EA 8BC8                    mov ecx, eax   
:004686EC 8B45E8                  mov eax, dword ptr \[ebp\-18\]   
:004686EF 668BD0                  mov dx, ax   
:004686F2 C745AC03000000          mov \[ebp\-54\], 00000003   
:004686F9 666BD202                imul dx, 0002   
:004686FD 0F8092030000            jo 00468A95   
:00468703 0FBFD2                  movsx edx, dx   
:00468706 2BCA                    sub ecx, edx   
:00468708 8D55AC                  lea edx, dword ptr \[ebp\-54\]   
:0046870B 0F8084030000            jo 00468A95   
:00468711 666BC002                imul ax, 0002   
:00468715 894DB4                  mov dword ptr \[ebp\-4C\], ecx   
:00468718 8D4DC4                  lea ecx, dword ptr \[ebp\-3C\]   
:0046871B 0F8074030000            jo 00468A95   
:00468721 894D84                  mov dword ptr \[ebp\-7C\], ecx   
:00468724 C7857CFFFFFF08400000    mov dword ptr \[ebp+FFFFFF7C\], 00004008   
:0046872E 52                      push edx   
:0046872F 66050100                add ax, 0001   
:00468733 0F805C030000            jo 00468A95   
:00468739 0FBFC0                  movsx eax, ax   
:0046873C 8D8D7CFFFFFF            lea ecx, dword ptr \[ebp+FFFFFF7C\]   
:00468742 50                      push eax   
:00468743 8D559C                  lea edx, dword ptr \[ebp\-64\]   
:00468746 51                      push ecx   
:00468747 52                      push edx   
:00468748 FFD3                    call ebx   
:0046874A 8D459C                  lea eax, dword ptr \[ebp\-64\]   
:0046874D 50                      push eax   
:0046874E FFD7                    call edi   
:00468750 8BD0                    mov edx, eax   
:00468752 8D4DC0                  lea ecx, dword ptr \[ebp\-40\]   
:00468755 FFD6                    call esi   
:00468757 8D4D9C                  lea ecx, dword ptr \[ebp\-64\]   
:0046875A 8D55AC                  lea edx, dword ptr \[ebp\-54\]   
:0046875D 51                      push ecx   
:0046875E 52                      push edx   
:0046875F 6A02                    push 00000002   
  
\* Reference To: MSVBVM60.\_\_vbaFreeVarList, Ord:0000h   
                                  |   
:00468761 FF1540104000            Call dword ptr \[00401040\]   
:00468767 668B45E8                mov ax, word ptr \[ebp\-18\]   
:0046876B 83C40C                  add esp, 0000000C   
:0046876E 662D0100                sub ax, 0001   
:00468772 0F801D030000            jo 00468A95   
:00468778 33C9                    xor ecx, ecx   
:0046877A 89854CFFFFFF            mov dword ptr \[ebp+FFFFFF4C\], eax   
:00468780 894DE4                  mov dword ptr \[ebp\-1C\], ecx   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0046882F(U)   
|   
:00468783 663BC8                  cmp cx, ax   
:00468786 0F8FA8000000            jg 00468834   
:0046878C 8B45E0                  mov eax, dword ptr \[ebp\-20\]   
:0046878F 8D55CC                  lea edx, dword ptr \[ebp\-34\]   
:00468792 895584                  mov dword ptr \[ebp\-7C\], edx   
:00468795 668B55E8                mov dx, word ptr \[ebp\-18\]   
:00468799 898564FFFFFF            mov dword ptr \[ebp+FFFFFF64\], eax   
:0046879F 8D45AC                  lea eax, dword ptr \[ebp\-54\]   
:004687A2 662BD1                  sub dx, cx   
:004687A5 50                      push eax   
:004687A6 0F80E9020000            jo 00468A95   
:004687AC 0FBFC2                  movsx eax, dx   
:004687AF 8D8D7CFFFFFF            lea ecx, dword ptr \[ebp+FFFFFF7C\]   
:004687B5 50                      push eax   
:004687B6 8D559C                  lea edx, dword ptr \[ebp\-64\]   
:004687B9 51                      push ecx   
:004687BA 52                      push edx   
:004687BB C7855CFFFFFF08000000    mov dword ptr \[ebp+FFFFFF5C\], 00000008   
:004687C5 C745B401000000          mov \[ebp\-4C\], 00000001   
:004687CC C745AC02000000          mov \[ebp\-54\], 00000002   
:004687D3 C7857CFFFFFF08400000    mov dword ptr \[ebp+FFFFFF7C\], 00004008   
:004687DD FFD3                    call ebx   
:004687DF 8D855CFFFFFF            lea eax, dword ptr \[ebp+FFFFFF5C\]   
:004687E5 8D4D9C                  lea ecx, dword ptr \[ebp\-64\]   
:004687E8 50                      push eax   
:004687E9 8D558C                  lea edx, dword ptr \[ebp\-74\]   
:004687EC 51                      push ecx   
:004687ED 52                      push edx   
  
\* Reference To: MSVBVM60.\_\_vbaVarCat, Ord:0000h   
                                  |   
:004687EE FF15AC114000            Call dword ptr \[004011AC\]   
:004687F4 50                      push eax   
:004687F5 FFD7                    call edi   
:004687F7 8BD0                    mov edx, eax   
:004687F9 8D4DE0                  lea ecx, dword ptr \[ebp\-20\]   
:004687FC FFD6                    call esi   
:004687FE 8D458C                  lea eax, dword ptr \[ebp\-74\]   
:00468801 8D4D9C                  lea ecx, dword ptr \[ebp\-64\]   
:00468804 50                      push eax   
:00468805 8D55AC                  lea edx, dword ptr \[ebp\-54\]   
:00468808 51                      push ecx   
:00468809 52                      push edx   
:0046880A 6A03                    push 00000003   
  
\* Reference To: MSVBVM60.\_\_vbaFreeVarList, Ord:0000h   
                                  |   
:0046880C FF1540104000            Call dword ptr \[00401040\]   
:00468812 B801000000              mov eax, 00000001   
:00468817 83C410                  add esp, 00000010   
:0046881A 660345E4                add ax, word ptr \[ebp\-1C\]   
:0046881E 0F8071020000            jo 00468A95   
:00468824 8945E4                  mov dword ptr \[ebp\-1C\], eax   
:00468827 8BC8                    mov ecx, eax   
:00468829 8B854CFFFFFF            mov eax, dword ptr \[ebp+FFFFFF4C\]   
:0046882F E94FFFFFFF              jmp 00468783   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00468786(C)   
|   
:00468834 8B45C4                  mov eax, dword ptr \[ebp\-3C\]   
:00468837 50                      push eax   
  
\* Reference To: MSVBVM60.\_\_vbaLenBstr, Ord:0000h   
                                  |   
:00468838 FF152C104000            Call dword ptr \[0040102C\]   
:0046883E 668B55E8                mov dx, word ptr \[ebp\-18\]   
:00468842 8BC8                    mov ecx, eax   
:00468844 666BD202                imul dx, 0002   
:00468848 0F8047020000            jo 00468A95   
:0046884E 0FBFC2                  movsx eax, dx   
:00468851 2BC8                    sub ecx, eax   
:00468853 0F803C020000            jo 00468A95   
:00468859 83E901                  sub ecx, 00000001   
:0046885C 0F8033020000            jo 00468A95   
  
\* Reference To: MSVBVM60.\_\_vbaI2I4, Ord:0000h   
                                  |   
:00468862 FF1534114000            Call dword ptr \[00401134\]   
:00468868 898544FFFFFF            mov dword ptr \[ebp+FFFFFF44\], eax   
:0046886E C745E400000000          mov \[ebp\-1C\], 00000000   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00468940(U)   
|   
:00468875 668B8D44FFFFFF          mov cx, word ptr \[ebp+FFFFFF44\]   
:0046887C 66394DE4                cmp word ptr \[ebp\-1C\], cx   
:00468880 0F8FBF000000            jg 00468945   
:00468886 8B55E0                  mov edx, dword ptr \[ebp\-20\]   
:00468889 8D4DAC                  lea ecx, dword ptr \[ebp\-54\]   
:0046888C 899564FFFFFF            mov dword ptr \[ebp+FFFFFF64\], edx   
:00468892 8B55C4                  mov edx, dword ptr \[ebp\-3C\]   
:00468895 8D45C0                  lea eax, dword ptr \[ebp\-40\]   
:00468898 51                      push ecx   
:00468899 52                      push edx   
:0046889A C7855CFFFFFF08000000    mov dword ptr \[ebp+FFFFFF5C\], 00000008   
:004688A4 C745B401000000          mov \[ebp\-4C\], 00000001   
:004688AB C745AC02000000          mov \[ebp\-54\], 00000002   
:004688B2 894584                  mov dword ptr \[ebp\-7C\], eax   
:004688B5 C7857CFFFFFF08400000    mov dword ptr \[ebp+FFFFFF7C\], 00004008   
  
\* Reference To: MSVBVM60.\_\_vbaLenBstr, Ord:0000h   
                                  |   
:004688BF FF152C104000            Call dword ptr \[0040102C\]   
:004688C5 668B4DE8                mov cx, word ptr \[ebp\-18\]   
:004688C9 666BC902                imul cx, 0002   
:004688CD 0F80C2010000            jo 00468A95   
:004688D3 0FBFD1                  movsx edx, cx   
:004688D6 0FBF4DE4                movsx ecx, word ptr \[ebp\-1C\]   
:004688DA 2BC2                    sub eax, edx   
:004688DC 8D957CFFFFFF            lea edx, dword ptr \[ebp+FFFFFF7C\]   
:004688E2 0F80AD010000            jo 00468A95   
:004688E8 2BC1                    sub eax, ecx   
:004688EA 0F80A5010000            jo 00468A95   
:004688F0 50                      push eax   
:004688F1 8D459C                  lea eax, dword ptr \[ebp\-64\]   
:004688F4 52                      push edx   
:004688F5 50                      push eax   
:004688F6 FFD3                    call ebx   
:004688F8 8D8D5CFFFFFF            lea ecx, dword ptr \[ebp+FFFFFF5C\]   
:004688FE 8D559C                  lea edx, dword ptr \[ebp\-64\]   
:00468901 51                      push ecx   
:00468902 8D458C                  lea eax, dword ptr \[ebp\-74\]   
:00468905 52                      push edx   
:00468906 50                      push eax   
  
\* Reference To: MSVBVM60.\_\_vbaVarCat, Ord:0000h   
                                  |   
:00468907 FF15AC114000            Call dword ptr \[004011AC\]   
:0046890D 50                      push eax   
:0046890E FFD7                    call edi   
:00468910 8BD0                    mov edx, eax   
:00468912 8D4DE0                  lea ecx, dword ptr \[ebp\-20\]   
:00468915 FFD6                    call esi   
:00468917 8D4D8C                  lea ecx, dword ptr \[ebp\-74\]   
:0046891A 8D559C                  lea edx, dword ptr \[ebp\-64\]   
:0046891D 51                      push ecx   
:0046891E 8D45AC                  lea eax, dword ptr \[ebp\-54\]   
:00468921 52                      push edx   
:00468922 50                      push eax   
:00468923 6A03                    push 00000003   
  
\* Reference To: MSVBVM60.\_\_vbaFreeVarList, Ord:0000h   
                                  |   
:00468925 FF1540104000            Call dword ptr \[00401040\]   
:0046892B B801000000              mov eax, 00000001   
:00468930 83C410                  add esp, 00000010   
:00468933 660345E4                add ax, word ptr \[ebp\-1C\]   
:00468937 0F8058010000            jo 00468A95   
:0046893D 8945E4                  mov dword ptr \[ebp\-1C\], eax   
:00468940 E930FFFFFF              jmp 00468875   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00468880(C)   
|   
:00468945 668B45E8                mov ax, word ptr \[ebp\-18\]   
:00468949 662D0100                sub ax, 0001   
:0046894D 0F8042010000            jo 00468A95   
:00468953 33C9                    xor ecx, ecx   
:00468955 89853CFFFFFF            mov dword ptr \[ebp+FFFFFF3C\], eax   
:0046895B 894DE4                  mov dword ptr \[ebp\-1C\], ecx   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00468A0A(U)   
|   
:0046895E 8B55E0                  mov edx, dword ptr \[ebp\-20\]   
:00468961 663BC8                  cmp cx, ax   
:00468964 0F8FA5000000            jg 00468A0F   
:0046896A 8D45C8                  lea eax, dword ptr \[ebp\-38\]   
:0046896D 899564FFFFFF            mov dword ptr \[ebp+FFFFFF64\], edx   
:00468973 894584                  mov dword ptr \[ebp\-7C\], eax   
:00468976 668B45E8                mov ax, word ptr \[ebp\-18\]   
:0046897A 662BC1                  sub ax, cx   
:0046897D 8D55AC                  lea edx, dword ptr \[ebp\-54\]   
:00468980 0F800F010000            jo 00468A95   
:00468986 0FBFC8                  movsx ecx, ax   
:00468989 52                      push edx   
:0046898A 8D957CFFFFFF            lea edx, dword ptr \[ebp+FFFFFF7C\]   
:00468990 51                      push ecx   
:00468991 8D459C                  lea eax, dword ptr \[ebp\-64\]   
:00468994 52                      push edx   
:00468995 50                      push eax   
:00468996 C7855CFFFFFF08000000    mov dword ptr \[ebp+FFFFFF5C\], 00000008   
:004689A0 C745B401000000          mov \[ebp\-4C\], 00000001   
:004689A7 C745AC02000000          mov \[ebp\-54\], 00000002   
:004689AE C7857CFFFFFF08400000    mov dword ptr \[ebp+FFFFFF7C\], 00004008   
:004689B8 FFD3                    call ebx   
:004689BA 8D8D5CFFFFFF            lea ecx, dword ptr \[ebp+FFFFFF5C\]   
:004689C0 8D559C                  lea edx, dword ptr \[ebp\-64\]   
:004689C3 51                      push ecx   
:004689C4 8D458C                  lea eax, dword ptr \[ebp\-74\]   
:004689C7 52                      push edx   
:004689C8 50                      push eax   
  
\* Reference To: MSVBVM60.\_\_vbaVarCat, Ord:0000h   
                                  |   
:004689C9 FF15AC114000            Call dword ptr \[004011AC\]   
:004689CF 50                      push eax   
:004689D0 FFD7                    call edi   
:004689D2 8BD0                    mov edx, eax   
:004689D4 8D4DE0                  lea ecx, dword ptr \[ebp\-20\]   
:004689D7 FFD6                    call esi   
:004689D9 8D4D8C                  lea ecx, dword ptr \[ebp\-74\]   
:004689DC 8D559C                  lea edx, dword ptr \[ebp\-64\]   
:004689DF 51                      push ecx   
:004689E0 8D45AC                  lea eax, dword ptr \[ebp\-54\]   
:004689E3 52                      push edx   
:004689E4 50                      push eax   
:004689E5 6A03                    push 00000003   
  
\* Reference To: MSVBVM60.\_\_vbaFreeVarList, Ord:0000h   
                                  |   
:004689E7 FF1540104000            Call dword ptr \[00401040\]   
:004689ED B801000000              mov eax, 00000001   
:004689F2 83C410                  add esp, 00000010   
:004689F5 660345E4                add ax, word ptr \[ebp\-1C\]   
:004689F9 0F8096000000            jo 00468A95   
:004689FF 8945E4                  mov dword ptr \[ebp\-1C\], eax   
:00468A02 8BC8                    mov ecx, eax   
:00468A04 8B853CFFFFFF            mov eax, dword ptr \[ebp+FFFFFF3C\]   
:00468A0A E94FFFFFFF              jmp 0046895E   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00468964(C)   
|   
:00468A0F 8D4DD0                  lea ecx, dword ptr \[ebp\-30\]//EDX=宽注册码   
  
\* Reference To: MSVBVM60.\_\_vbaStrCopy, Ord:0000h   
                                  |   
:00468A12 FF15F4114000            Call dword ptr \[004011F4\]   
:00468A18 9B                      wait   
:00468A19 687A8A4600              push 00468A7A   
:00468A1E EB30                    jmp 00468A50   
///////////////////////////////////////////////////////////////////   
中断地址  中断次数  指令    长度   
00468A0F  1          8D      3   
内存方式  寄存器 EDX 宽字符串   
///////////////////////////////////////////////////////////////////    
收工。   
                                  lajiaolz   
                                  2002/06/10  4dd466d0e1e0cc123afbe504da2462ca