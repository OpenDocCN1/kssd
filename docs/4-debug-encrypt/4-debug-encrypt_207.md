

# 英语会话精灵 3.2

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-62.htm`](https://www.pediy.com/kssd/pediy08/pediy8-62.htm)

Blowfish 初探－英语会话精灵 3.2  
  
首先严重的感谢看雪论坛的作者们，特别感谢加密与解密第二版，是它一步步引领我前进。  
本文的加密解析章节比较乱，基本是跟着思路写下来的，比较原始的体现了我破解的思路。  
Blowfish 是根据 OD 载入的文字提示及看雪精华 5－upfeed1 的大作－BlowFish 尝试－代码确定的。   
  
1.脱壳：  
PEBundle 0.2 - 3.x -> Jeremy Collake  
00638000 t>  9C               pushfd  
00638001     60               pushad  
00638002     E8 02000000      call topbar.00638009  
...  
0063836F     61               popad  
00638370     9D               popfd  
00638371     68 01306300      push topbar.00633001      ; 参考 RORDBG 来到这里  
00638376     C3               retn  
  
dump＋peid：ASPack 2.12 -> Alexey Solodovnikov  
00633001     60               pushad  
00633002     E8 03000000      call topbar.0063300A  
00633007   - E9 EB045D45      jmp 45C034F7  
...  
006333AF     61               popad                       ; 直接 go 到这里  
006333B0     75 08            jnz short topbar.006333BA  
006333B2     B8 01000000      mov eax,1  
006333B7     C2 0C00          retn 0C  
006333BA     68 70D35600      push topbar.0056D370  
006333BF     C3               retn  
  
dump＋peid：Borland Delphi 6.0 - 7.0  
不用修复，运行 ok。  
  
2.断点：  
机器码：0604C－5FBB0  
练码：9876543210123456  
确认后提示：注册码无效！  
  
OD 载入脱壳后程序，串参考：  
地址=004CF2F7 反汇编=mov edx,up-2.004CF530 文本字符串=注册成功！请保留好您的注册码，谢谢！  
地址=004CF3EF 反汇编=mov edx,up-2.004CF5AC 文本字符串=注册码无效！  
往上找入口：  
004CF160   /.  55             push ebp  
004CF161   |.  8BEC           mov ebp,esp  
  
3.流程：  
:004CF1A4 8B8520FFFFFF            mov eax, dword ptr \[ebp+FFFFFF20\]  
:004CF1AA E85157F3FF              call 00404900  
:004CF1AF 83F80F                  cmp eax, 0000000F                      ;  注册码长度要大于 F  
:004CF1B2 0F8EEB010000            jle 004CF3A3  
  
\* Possible StringData Ref from Code Obj ->"reg.dll"  
:004CF1B8 68BCF44C00              push 004CF4BC  
  
\* Reference To: KERNEL32.LoadLibraryA, Ord:01E6h  
:004CF1BD E8FA7CF3FF              Call 00406EBC  
:004CF1C2 8BF0                    mov esi, eax  
:004CF1C4 85F6                    test esi, esi  
:004CF1C6 0F8407010000            je 004CF2D3  
:004CF1CC 8D8526FFFFFF            lea eax, dword ptr \[ebp+FFFFFF26\]  
:004CF1D2 33C9                    xor ecx, ecx  
:004CF1D4 BA65000000              mov edx, 00000065  
:004CF1D9 E8DA3EF3FF              call 004030B8  
:004CF1DE 8D458B                  lea eax, dword ptr \[ebp-75\]  
:004CF1E1 33C9                    xor ecx, ecx  
:004CF1E3 BA65000000              mov edx, 00000065  
:004CF1E8 E8CB3EF3FF              call 004030B8  
  
\* Possible StringData Ref from Code Obj ->"BlowFishDecrypt"  
:004CF1ED 68C4F44C00              push 004CF4C4  
:004CF1F2 56                      push esi  
  
\* Reference To: KERNEL32.GetProcAddress, Ord:0158h  
:004CF1F3 E8147CF3FF              Call 00406E0C  
:004CF1F8 8945FC                  mov dword ptr \[ebp-04\], eax  
  
\* Possible StringData Ref from Code Obj ->"EncryptStringFun1"|  
:004CF1FB 68D4F44C00              push 004CF4D4  
:004CF200 56                      push esi  
  
\* Reference To: KERNEL32.GetProcAddress, Ord:0158h|  
:004CF201 E8067CF3FF              Call 00406E0C  
:004CF206 8945F8                  mov dword ptr \[ebp-08\], eax  
:004CF209 837DFC00                cmp dword ptr \[ebp-04\], 00000000  
:004CF20D 0F84C0000000            je 004CF2D3  
:004CF213 837DF800                cmp dword ptr \[ebp-08\], 00000000  
:004CF217 0F84B6000000            je 004CF2D3  
  
\* Possible StringData Ref from Code Obj ->"lxhest-EC3CABAC25C0F96DCC5AE18F874B07DC98F0E67"  
                                        ->"2C26924FD"|  
:004CF21D 68E8F44C00              push 004CF4E8  
:004CF222 8D8526FFFFFF            lea eax, dword ptr \[ebp+FFFFFF26\]  
:004CF228 50                      push eax                                 
  
\* Possible StringData Ref from Code Obj ->"sbipxa"|  
:004CF229 6820F54C00              push 004CF520  
:004CF22E 8D9514FFFFFF            lea edx, dword ptr \[ebp+FFFFFF14\]  
:004CF234 8B8300030000            mov eax, dword ptr \[ebx+00000300\]  
:004CF23A E8453DF7FF              call 00442F84  
:004CF23F 8B8514FFFFFF            mov eax, dword ptr \[ebp+FFFFFF14\]  
:004CF245 8D8D18FFFFFF            lea ecx, dword ptr \[ebp+FFFFFF18\]  
:004CF24B BA10000000              mov edx, 00000010                       ; 不管注册码多长，只取前 16 位  
:004CF250 E84BD7F6FF              call 0043C9A0  
:004CF255 8B8518FFFFFF            mov eax, dword ptr \[ebp+FFFFFF18\]  
:004CF25B E89858F3FF              call 00404AF8  
:004CF260 50                      push eax                                ; 输入是注册码  
:004CF261 FF55FC                  call \[ebp-04\]                           ; 加密  
:004CF264 84C0                    test al, al  
:004CF266 7413                    je 004CF27B  
:004CF268 8D45F4                  lea eax, dword ptr \[ebp-0C\]  
:004CF26B 8D9526FFFFFF            lea edx, dword ptr \[ebp+FFFFFF26\]       ; 结果输出  
:004CF271 B965000000              mov ecx, 00000065  
:004CF276 E83556F3FF              call 004048B0  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004CF266(C)  
\* Possible StringData Ref from Code Obj ->"lxhest-EC3CABAC25C0F96DCC5AE18F874B07DC98F0E67"  
                                        ->"2C26924FD"|  
:004CF27B 68E8F44C00              push 004CF4E8  
:004CF280 8D458B                  lea eax, dword ptr \[ebp-75\]  
:004CF283 50                      push eax                               
  
\* Possible StringData Ref from Code Obj ->"sbipxa"|  
:004CF284 6820F54C00              push 004CF520  
:004CF289 8D9510FFFFFF            lea edx, dword ptr \[ebp+FFFFFF10\]  
:004CF28F 8B83FC020000            mov eax, dword ptr \[ebx+000002FC\]  
:004CF295 E8EA3CF7FF              call 00442F84  
:004CF29A 8B8510FFFFFF            mov eax, dword ptr \[ebp+FFFFFF10\]  
:004CF2A0 E85358F3FF              call 00404AF8  
:004CF2A5 50                      push eax                                ; 机器码  
:004CF2A6 FF55F8                  call \[ebp-08\]                           ; 加密  
:004CF2A9 84C0                    test al, al  
:004CF2AB 7426                    je 004CF2D3  
:004CF2AD 8D850CFFFFFF            lea eax, dword ptr \[ebp+FFFFFF0C\]  
:004CF2B3 8D558B                  lea edx, dword ptr \[ebp-75\]             ; 结果输出  
:004CF2B6 B965000000              mov ecx, 00000065  
:004CF2BB E8F055F3FF              call 004048B0  
:004CF2C0 8B850CFFFFFF            mov eax, dword ptr \[ebp+FFFFFF0C\]  
:004CF2C6 8D4DF0                  lea ecx, dword ptr \[ebp-10\]  
:004CF2C9 BA06000000              mov edx, 00000006  
:004CF2CE E8E9D6F6FF              call 0043C9BC  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:004CF1C6(C), :004CF20D(C), :004CF217(C), :004CF2AB(C)  
:004CF2D3 56                      push esi  
  
\* Reference To: KERNEL32.FreeLibrary, Ord:00C8h|  
:004CF2D4 E8BB7AF3FF              Call 00406D94  
:004CF2D9 8B45F4                  mov eax, dword ptr \[ebp-0C\]  
:004CF2DC 8B55F0                  mov edx, dword ptr \[ebp-10\]  
:004CF2DF E86057F3FF              call 00404A44  
:004CF2E4 0F85B9000000            jne 004CF3A3                             ; 关键比较  
:004CF2EA 8BB3F0020000            mov esi, dword ptr \[ebx+000002F0\]  
:004CF2F0 C6466403                mov \[esi+64\], 03  
:004CF2F4 8D4668                  lea eax, dword ptr \[esi+68\]  
  
\* Possible StringData Ref from Code Obj ->"注册成功！请保留好您的注册码，谢谢！"|  
:004CF2F7 BA30F54C00              mov edx, 004CF530  
:004CF2FC E89B53F3FF              call 0040469C  
  
4.对 reg.dll 的分析：  
流程用到 reg.dll，路径=D:\\WINNT\\system32\\reg.dll，程序带的，好像是个库。  
004CF1B8   |.  68 BCF44C00      push up-2.004CF4BC                   ; /FileName = "reg.dll"  
004CF1BD   |.  E8 FA7CF3FF      call <jmp.&KERNEL32.LoadLibraryA>    ; \\LoadLibraryA  
  
载入后数据窗口 0012C530 显示如下加解密算法：    
Base64Decode.Base64Encode.  
BlowFishDecrypt.BlowFishEncrypt.  
CRC32.CRCFileCheck.DecryptStringFun1.DecryptStringFun2.  
DesDecrypt.DesEncrypt.EncryptStringFun1.EncryptStringFun2.FileDecrypt.FileEncrypt.  
GetDllVersion.GetHardDiskId.GetMainBoardId.  
MD5Encrypt.  
RSADecrypt.RSAEncrypt.  
SHAEncrypt.Secret16Encrypt  
  
调用完 reg.dll 还不忘释放：  
004CF2D3   |> \\56              push esi                                ; /hLibModule  
004CF2D4   |.  E8 BB7AF3FF     call <jmp.&KERNEL32.FreeLibrary>        ; \\FreeLibrary  
  
程序还带了个 dll：D:\\WINNT\\system32\\smartread.dll，可能用来取得机器码。    
  
5.加密解析  
2 次加密，第一次对输入的注册码，第二次对机器码，跟踪过第二次加密的值与输入的注册码无关，是定值，  
说明第二次加密的跟输入的注册码无关。因此从第二次加密入手。  
  
5.1 第二次加密：对机器码的加密  
004CF2A5   |.  50            push eax  
004CF2A6   |.  FF55 F8       call dword ptr ss:\[ebp-8\]       ;  堆栈 ss:\[0012FCB0\]=10010880  
入栈参数：  
0012FB90    00F55E64    ASCII "0604C-5FBB0"  
0012FB94    004CF520    ASCII "sbipxa"  
  
100106C3     E8 78CAFFFF   call 1000D140                         
这是对 sbipxa 下硬件访问断点找到的，跨过该 call 后得到 32 个字符，猜测 MD5 散列，试了下果然是。  
MD5（sbipxa）＝ (ASCII "61146E463A46267B3489D53D8E96BB96")  
  
取前 8 位＝61146E46，如果有 F 就替换为 8  
100109C6     83F8 08         cmp eax,8  
100109C9     7D 0F           jge short 100109DA  
100109CB     807C05 E4 46    cmp byte ptr ss:\[ebp+eax-1C\],46  
100109D0     75 05           jnz short 100109D7  
100109D2     C64405 E4 38    mov byte ptr ss:\[ebp+eax-1C\],38  
100109D7     40              inc eax  
100109D8   ^ EB EC           jmp short 100109C6  
  
对 61146E46 的填充到 12 位：  
由上面返回来到的，填充结果：0012ACA0    01BE0D6C    ASCII "61146E466114"  
10001303     8A0C0E         mov cl,byte ptr ds:\[esi+ecx\]  
10001306     8D4424 24      lea eax,dword ptr ss:\[esp+24\]  
1000130A     884C24 24      mov byte ptr ss:\[esp+24\],cl  
1000130E     50             push eax  
1000130F     6A 01          push 1  
10001311     8D4C24 18      lea ecx,dword ptr ss:\[esp+18\]  
10001315     E8 76080000    call 10001B90  
1000131A     8B4C24 10      mov ecx,dword ptr ss:\[esp+10\]  
1000131E     46             inc esi  
1000131F     8379 F8 0C     cmp dword ptr ds:\[ecx-8\],0C  
10001323   ^ 7C DE          jl short 10001303  
  
逆序：0012FB0C  34 31 31 36 36 34 45 36 34 31 31 36              411664E64116  
1000132F     8B7B 08        mov edi,dword ptr ds:\[ebx+8\]  
10001332     C1E7 08        shl edi,8  
10001335     897B 08        mov dword ptr ds:\[ebx+8\],edi  
10001338     8BD7           mov edx,edi  
1000133A     0FBE48 FC      movsx ecx,byte ptr ds:\[eax-4\]  
1000133E     0BCA           or ecx,edx  
10001340     8B53 0C        mov edx,dword ptr ds:\[ebx+C\]  
10001343     C1E2 08        shl edx,8  
10001346     894B 08        mov dword ptr ds:\[ebx+8\],ecx  
10001349     8953 0C        mov dword ptr ds:\[ebx+C\],edx  
1000134C     8BF2           mov esi,edx  
1000134E     0FBE10         movsx edx,byte ptr ds:\[eax\]  
10001351     0BD6           or edx,esi  
10001353     8B73 10        mov esi,dword ptr ds:\[ebx+10\]  
10001356     C1E6 08        shl esi,8  
10001359     8953 0C        mov dword ptr ds:\[ebx+C\],edx  
1000135C     8973 10        mov dword ptr ds:\[ebx+10\],esi  
1000135F     8BFE           mov edi,esi  
10001361     0FBE70 04      movsx esi,byte ptr ds:\[eax+4\]  
10001365     0BF7           or esi,edi  
10001367     40             inc eax  
10001368     8973 10        mov dword ptr ds:\[ebx+10\],esi  
1000136B     8D3C28         lea edi,dword ptr ds:\[eax+ebp\]  
1000136E     83FF 04        cmp edi,4  
10001371   ^ 7C BC          jl short 1000132F  
  
对 411664E64116 循环计算，结果与机器码 0604C-5FBB0 异或：  
0012FB0C  34 31 31 36 36 34 45 36 34 31 31 36              411664E64116  
          A           B           C    
100014D2     8B5C24 10       mov ebx,dword ptr ss:\[esp+10\]  
100014D6     8B41 08         mov eax,dword ptr ds:\[ecx+8\]     ; 取出 A  
100014D9     A8 01           test al,1  
100014DB     74 42           je short 1000151F  
100014DD     8B79 14         mov edi,dword ptr ds:\[ecx+14\]    ; 80000062  
100014E0     8B59 2C         mov ebx,dword ptr ds:\[ecx+2C\]    ; 80000000  
100014E3     33F8            xor edi,eax                      ; A=80000062 XOR A  
100014E5     8B41 0C         mov eax,dword ptr ds:\[ecx+C\]     ; 取出 B  
100014E8     D1EF            shr edi,1                        ; A/2  
100014EA     0BFB            or edi,ebx                       ; OR 80000000  
100014EC     A8 01           test al,1  
100014EE     8979 08         mov dword ptr ds:\[ecx+8\],edi     ; A 保存回原地址 12FB0C  
100014F1     74 1A           je short 1000150D  
100014F3     8B79 18         mov edi,dword ptr ds:\[ecx+18\]    ; 40000020  
100014F6     BB 01000000     mov ebx,1  
100014FB     33F8            xor edi,eax                      ; B XOR 40000020  
100014FD     8B41 30         mov eax,dword ptr ds:\[ecx+30\]    ; C0000000  
10001500     D1EF            shr edi,1                        ; B/2  
10001502     0BF8            or edi,eax                       ; B OR C0000000  
10001504     895C24 10       mov dword ptr ss:\[esp+10\],ebx    ; 赋值 ebx＝1  
10001508     8979 0C         mov dword ptr ds:\[ecx+C\],edi     ; B 保存回原地址 12FB10  
1000150B     EB 45           jmp short 10001552  
1000150D     8B79 24         mov edi,dword ptr ds:\[ecx+24\]    ; 3FFFFFFF  
10001510     D1E8            shr eax,1                        ; B/2  
10001512     23C7            and eax,edi                      ; AND 3FFFFFFF  
10001514     33DB            xor ebx,ebx                      ; 赋值 ebx＝0  
10001516     8941 0C         mov dword ptr ds:\[ecx+C\],eax     ; 保存回原地址 12FB10  
10001519     895C24 10       mov dword ptr ss:\[esp+10\],ebx  
1000151D     EB 33           jmp short 10001552  
1000151F     8B79 20         mov edi,dword ptr ds:\[ecx+20\]    ; 7FFFFFFF  
10001522     D1E8            shr eax,1                        ; A/2  
10001524     23C7            and eax,edi                      ; AND 7FFFFFFF  
10001526     8941 08         mov dword ptr ds:\[ecx+8\],eax     ; 保存回原地址 12FB0C  
10001529     8B41 10         mov eax,dword ptr ds:\[ecx+10\]    ; 取出 C  
1000152C     A8 01           test al,1  
1000152E     74 16           je short 10001546  
10001530     8B51 1C         mov edx,dword ptr ds:\[ecx+1C\]    ; 10000002  
10001533     33D0            xor edx,eax                      ; C XOR 10000002  
10001535     8B41 34         mov eax,dword ptr ds:\[ecx+34\]    ; F0000000  
10001538     D1EA            shr edx,1                        ; C/2  
1000153A     0BD0            or edx,eax                       ; OR F0000000  
1000153C     8951 10         mov dword ptr ds:\[ecx+10\],edx    ; C 保存回原地址 12FB14  
1000153F     BA 01000000     mov edx,1                        ; 赋值 edx＝1  
10001544     EB 0C           jmp short 10001552  
10001546     8B51 28         mov edx,dword ptr ds:\[ecx+28\]    ; 0FFFFFFF  
10001549     D1E8            shr eax,1                        ; C/2  
1000154B     23C2            and eax,edx                      ; AND 0FFFFFFF  
1000154D     33D2            xor edx,edx                      ; 赋值 edx＝0  
1000154F     8941 10         mov dword ptr ds:\[ecx+10\],eax    ; 保存回原地址 12FB14  
10001552     8AC2            mov al,dl  
10001554     32C3            xor al,bl  
10001556     8A5C24 0F       mov bl,byte ptr ss:\[esp+F\]  
1000155A     D0E3            shl bl,1  
1000155C     0AC3            or al,bl  
1000155E     4E              dec esi                           ; 循环指针  
1000155F     884424 0F       mov byte ptr ss:\[esp+F\],al        ; 最后需要的值  
10001563   ^ 0F85 69FFFFFF   jnz 100014D2  
10001569     8B4C24 18       mov ecx,dword ptr ss:\[esp+18\]  
1000156D     5F              pop edi  
1000156E     5E              pop esi  
1000156F     5B              pop ebx  
10001570     8A11            mov dl,byte ptr ds:\[ecx\]         ; 12ACD0 保存了顺取机器码的一个字符  
10001572     32D0            xor dl,al                        ; 核心运算  
10001574     8811            mov byte ptr ds:\[ecx\],dl         ; 结果保存  
10001576     75 02           jnz short 1000157A  
循环 8 次，ABC 一直循环下去，得到一个最终的 AL 值参与机器码的异或。  
  
返回到：  
100015C4     FF52 14         call dword ptr ds:\[edx+14\]           
...  
10001670     881419          mov byte ptr ds:\[ecx+ebx\],dl          ; 异或结果的保存，原地址  
10001673     43              inc ebx  
10001674     3BD8            cmp ebx,eax                           ; 循环次数＝机器码长度  
10001676   ^ 0F8C 35FFFFFF   jl 100015B1  
  
每次循环进一次 call，得到一个 AL 值，分别为：13 FA CB 03 97 FA D9 7D 3B 2F 30  
分别与机器码异或得到结果：01BE0DBC  23 CC FB 37 D4 D7 EC 3B 79 6D 30                 #帖 7 宰?ym0.!.  
  
然后又返回到：100109E8     E8 930BFFFF     call 10001580       
  
往下走，转换成字符串：23CCFB37D4D7EC3B796D30  
10010A51     E8 EAC6FFFF     call 1000D140  
  
然后一路往下走，狂多动作，最后返回程序：  
004CF2A6   |.  FF55 F8       call dword ptr ss:\[ebp-8\]  
  
取最后的 6 位：796D30  
004CF2C9   |.  BA 06000000   mov edx,6  
004CF2CE   |.  E8 E9D6F6FF   call up-2.0043C9BC  
  
5.2 第一次加密：对注册码的加密   
004CF260   |.  50            push eax  
004CF261   |.  FF55 FC       call dword ptr ss:\[ebp-4\]       ;  堆栈 ss:\[0012FCB4\]=100137C0  
入栈参数：  
0012FB94    004CF520    ASCII "sbipxa"  
0012EAC4    00F74240    ASCII "9876543210123456"  
  
对注册码下硬件断点跟踪：  
从 00F74240 拷贝数据：  
第一次拷贝从 00F74240 到 01BE0E2C，跟飞  
10001AA1     F3:A5            rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]  ; 停在此  
01BE0E2C  39 38 37 36 35 34 33 32 31 30 31 32 33 34 35 36 9876543210123456  
  
第二次拷贝从 00F74240 到 01BE0D70，再次硬件断点    
100139AB     F3:A5            rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]  ; 停在此  
01BE0D70  39 38 37 36 35 34 33 32 31 30 31 32 33 34 35 36 9876543210123456  
  
转换（01BE0D70 硬件断点）：  
1000D0FF     8A043E           mov al,byte ptr ds:\[esi+edi\]          ; 停在此  
1000D102     3C 30            cmp al,30  
1000D104     7C 10            jl short 1000D116  
1000D106     3C 39            cmp al,39  
1000D108     7F 0C            jg short 1000D116  
1000D10A     8A11             mov dl,byte ptr ds:\[ecx\]  
1000D10C     80C2 0D          add dl,0D  
1000D10F     C0E2 04          shl dl,4  
1000D112     02D0             add dl,al  
1000D114     EB 12            jmp short 1000D128  
1000D116     3C 41            cmp al,41  
1000D118     7C 16            jl short 1000D130  
1000D11A     3C 46            cmp al,46  
1000D11C     7F 12            jg short 1000D130  
1000D11E     8A11             mov dl,byte ptr ds:\[ecx\]  
1000D120     C0E2 04          shl dl,4  
1000D123     02D0             add dl,al  
1000D125     80EA 37          sub dl,37  
1000D128     46               inc esi  
1000D129     8811             mov byte ptr ds:\[ecx\],dl           ; 0012EAD8  98             
1000D12B     83FE 02          cmp esi,2  
1000D12E   ^ 7C CF            jl short 1000D0FF  
取出注册码的 2 位.  
  
返回：  
1000D1C6     8D4424 1C        lea eax,dword ptr ss:\[esp+1C\]  
1000D1CA     50               push eax  
1000D1CB     57               push edi  
1000D1CC     E8 1FFFFFFF      call 1000D0F0                      ; 调用，取注册码 2 位  
1000D1D1     8A4C24 24        mov cl,byte ptr ss:\[esp+24\]        ; 返回处，堆栈 ss:\[0012EAD8\]=98  
1000D1D5     83C4 08          add esp,8  
1000D1D8     880C2E           mov byte ptr ds:\[esi+ebp\],cl       ; cl=98，ds:\[01BE0D50\]=00  
1000D1DB     46               inc esi  
1000D1DC     83C7 02          add edi,2  
1000D1DF     3BF3             cmp esi,ebx  
1000D1E1   ^ 7C E3            jl short 1000D1C6  
将 01BE0D70 的注册码转换保存于 01BE0D50:   
01BE0D50  98 76 54 32 10 12 34 56  榲 T24V  
  
返回：100139C4     E8 E797FFFF   call 1000D1B0  
  
再次转换（硬件断点 01BE0D50）:  
10004371     8A51 FF          mov dl,byte ptr ds:\[ecx-1\]         ; 停在此，注册码 98  
返回：10004905     E8 56FAFFFF      call 10004360       
该 call 内容就是重排，得到：0012EAB0  32 54 76 98 56 34 12 10  2Tv 榁 4  
增加硬件访问断点 0012EAB0.  
  
运算得到最后结果（硬件断点 0012EAB0）：  
1000401A     8B7D 00            mov edi,dword ptr ss:\[ebp\]             ; 停在此，987654321  
1000401D     8B5E 58            mov ebx,dword ptr ds:\[esi+58\]          ; 0012EAE8,KEY\_sBOX?  
10004020     33DF               xor ebx,edi  
10004022     8BC3               mov eax,ebx  
10004024     8BCB               mov ecx,ebx  
10004026     C1E8 10            shr eax,10  
10004029     25 FF000000        and eax,0FF  
1000402E     C1E9 18            shr ecx,18  
10004031     8BBC86 5C040000    mov edi,dword ptr ds:\[esi+eax\*4+45C\]  
10004038     8B548E 5C          mov edx,dword ptr ds:\[esi+ecx\*4+5C\]  
....  
10004354     894D 00            mov dword ptr ss:\[ebp\],ecx             ; ecx=22F277A2  
10004357     895D 04            mov dword ptr ss:\[ebp+4\],ebx           ; ebx=AB46F813  
  
然后返回：100139D7     E8 D40CFFFF        call 100046B0      ; 计算得到最后的结果  
最后的结果保存于：0012FBDE  22 F2 77 A2 AB 46 F8 13         
  
好像是 BLOWFISH，对输入的注册码的前 8 位加密；运算取值存放在 ESI（0012EAE8）以后的地址。  
  
接下来跟踪 ESI 的值是否跟输入的注册码有关，是如何来的？  
  
对 sbipxa 下硬件访问断点（004CF520）：  
拷贝（之前断下一次，读取长度，未进一步操作）  
第一次拷贝，从 004CF520 到 01BE0DBC：跟飞  
10001AA1     F3:A5        rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]     ; 停在此  
01BE0DBC  73 62 69 70 78 61        sbipxa  
  
第二次拷贝，从 004CF520 到 0012EA94：  
10003B4A     F3:A5          rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]    ; 停在此  
0012EA94  73 62 69 70 78 61        sbipxa  
  
从第二次拷贝 sbipxa 地址往下发现好像是对 pbox,sbox 的初始化：  
10003B58     B9 12000000          mov ecx,12  
10003B5D     BE E0440210          mov esi,100244E0  
10003B62     8BFA                 mov edi,edx  
10003B64     C74424 58 12000000   mov dword ptr ss:\[esp+58\],12  
10003B6C     F3:A5                rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]    ; pbox?\[18\]  
10003B71     B9 00040000          mov ecx,400  
10003B76     BE 28450210          mov esi,10024528  
10003B7B     F3:A5                rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]    ; sbox?\[4\*256\]  
  
pbox\[18\]:  
0012EAFC  88 6A 3F 24 D3 08 A3 85 2E 8A 19 13 44 73 70 03  坖?$?.?Dsp  
...  
0012EB3C  D9 D5 16 92 1B FB 79 89                          僬?鹹?筿倈  
  
sbox\[4\*256\]:  
0012EB44  A6 0B 31 D1 AC B5 DF 98 DB 72 FD 2F B7 DF 1A D0  ?1 熏颠樭 r?愤  
...  
0012FB34  32 61 4E B7 5B E2 77 CE E3 DF 8F 57 E6 72 C3 3A  2aN 穂鈝毋邚 W 鎟?  
  
pbox 和 sbox 可参考看雪精华 5 的 Blowfish 文章。  
  
填充 pbox 得到 key\_pbox：  
10003B83     33D2                 xor edx,edx  
10003B85     C74424 5C 04000000   mov dword ptr ss:\[esp+5C\],4  
10003B8D     8BFA                 mov edi,edx  
10003B8F     33D2                 xor edx,edx  
10003B91     8A11                 mov dl,byte ptr ds:\[ecx\]  
10003B93     C1E7 08              shl edi,8  
10003B96     0BD7                 or edx,edi  
10003B98     41                   inc ecx  
10003B99     40                   inc eax  
10003B9A     3BC3                 cmp eax,ebx  
10003B9C     75 06                jnz short 10003BA4  
10003B9E     33C0                 xor eax,eax  
10003BA0     8D4C24 18            lea ecx,dword ptr ss:\[esp+18\]  
10003BA4     8B7C24 5C            mov edi,dword ptr ss:\[esp+5C\]  
10003BA8     4F                   dec edi  
10003BA9     897C24 5C            mov dword ptr ss:\[esp+5C\],edi  
10003BAD   ^ 75 DE                jnz short 10003B8D  
10003BAF     8B3E                 mov edi,dword ptr ds:\[esi\]              ; 取出 pbox，首地址 0012EAFC  
10003BB1     83C6 04              add esi,4                               ; pbox 下一个  
10003BB4     33FA                 xor edi,edx                             ; 异或了  
10003BB6     8B5424 58            mov edx,dword ptr ss:\[esp+58\]           ; 12，即 18（十进制），循环次数  
10003BBA     897E FC              mov dword ptr ds:\[esi-4\],edi            ; 异或值保存回去  
10003BBD     4A                   dec edx                                 ; 循环指针  
10003BBE     895424 58            mov dword ptr ss:\[esp+58\],edx  
10003BC2   ^ 75 BF                jnz short 10003B83  
  
取 sbipxa 的 ASCII 码（73 62 69 70 78 61）进行循环使用，每次右移一位取 8 位，  
得到十八组（3×6）数据：（73626970;62697078;69707861;70786173;78617362;61736269）×3  
key 相当于：736269706269707869707861707861737861736261736269  
  
BF\_EN 加密 pbox\[18\]：   
10003BD1     8D4C24 10         lea ecx,dword ptr ss:\[esp+10\]  
10003BD5     51                push ecx  
10003BD6     8BCD              mov ecx,ebp  
10003BD8     E8 93000000       call 10003C70                      ; BF\_EN()  
10003BDD     8B5424 10         mov edx,dword ptr ss:\[esp+10\]      ; BF\_EN()结果－1  
10003BE1     8B4424 14         mov eax,dword ptr ss:\[esp+14\]      ; BF\_EN()结果－2  
10003BE5     8916              mov dword ptr ds:\[esi\],edx         ; 替换 key\_pbox\[i\]  
10003BE7     83C6 04           add esi,4  
10003BEA     47                inc edi  
10003BEB     8906              mov dword ptr ds:\[esi\],eax         ; 替换 key\_pbox\[i＋1\]  
10003BED     47                inc edi  
10003BEE     83C6 04           add esi,4  
10003BF1     83FF 12           cmp edi,12  
10003BF4   ^ 72 DB             jb short 10003BD1  
  
用 BF\_EN 加密一个全 0 的 64 位信息，用输出的结果替换 key\_pbox\[0\]和 key\_pbox\[1\]，i＝0；  
用 BF\_EN 加密替换后的 key\_pbox\[i\]和 key\_pbox\[i+1\]，加密结果替代 key\_pbox\[i+1\]和 key\_pbox\[i+2\]；  
直到 key\_pbox 全部被替换。  
  
然后用类似的方法，替换 key\_sbox 信息加密\[4\*256\]：  
10003C07     8D4C24 10         lea ecx,dword ptr ss:\[esp+10\]  
10003C0B     51                push ecx  
10003C0C     8BCD              mov ecx,ebp  
10003C0E     E8 5D000000       call 10003C70  
10003C13     8B5424 10         mov edx,dword ptr ss:\[esp+10\]  
10003C17     8B4424 14         mov eax,dword ptr ss:\[esp+14\]  
10003C1B     8916              mov dword ptr ds:\[esi\],edx  
10003C1D     83C6 04           add esi,4  
10003C20     47                inc edi  
10003C21     8906              mov dword ptr ds:\[esi\],eax  
10003C23     47                inc edi  
10003C24     83C6 04           add esi,4  
10003C27     81FF 00010000     cmp edi,100  
10003C2D   ^ 7C D8             jl short 10003C07  
10003C2F     8B4424 5C         mov eax,dword ptr ss:\[esp+5C\]  
10003C33     81C3 00040000     add ebx,400  
10003C39     48                dec eax  
10003C3A     894424 5C         mov dword ptr ss:\[esp+5C\],eax  
10003C3E   ^ 75 C3             jnz short 10003C03  
  
这次加密 key\_sbox 后得到的值就是上面跟踪注册码提到的 ESI。  
然后返回：10013931     E8 AA01FFFF       call 10003AE0  
  
再往下就是对注册码的处理（与上述跟踪注册码处理的过程连接上）：  
100139C4     E8 E797FFFF       call 1000D1B0     ; 重排  
01BE0D50  98 76 54 32 10 12 34 56  榲 T24V  
  
10004905     E8 56FAFFFF       call 10004360     ; 重排  
0012EAB0  32 54 76 98 56 34 12 10  2Tv 榁 4  
  
10004914     E8 F7F6FFFF       call 10004010     ; 加密  
  
5.3 比较  
对 2 次加密结果的比较，注册码加密的是 16 字节 64 位，机器码加密取后面 6 个字符是 12 字节 24 位。  
004CF2D9   |.  8B45 F4         mov eax,dword ptr ss:\[ebp-C\]  
004CF2DC   |.  8B55 F0         mov edx,dword ptr ss:\[ebp-10\]  
004CF2DF   |.  E8 6057F3FF     call up-2.00404A44                      ;  BIJIAO  
004CF2E4   |.  0F85 B9000000   jnz up-2.004CF3A3  
EAX:00F55E64  22 F2 77 A2 AB 46 F8 13              "騱F?...  
EDX:00F5185C  37 39 36 44 33 30 00 00               796D30..  
  
6.算法总结  
对注册码的加密：  
采用 Blowfish 加密，key＝736269706269707869707861707861737861736261736269，得到比较值之一。  
  
对机器码的加密：  
MD5(sbipxa)，取前 8 位＋前 4 位扩充到 12 位（有 F 的替换成 8），逆序重排后参与循环计算得到 AL 值（1 个字节），根据机器码长度产生同样数目的 AL，然后与机器码 XOR，得到值，取后 12 位作为比较值之一。  
  
如果两者的加密结果一样的话就注册成功。  
  
7.后记  
很遗憾，功力不够，没能反推出注册码。  
看雪论坛上只有一个计算 Blowfish 的工具 cryptocal.exe，可是加密结果跟这个程序加密的值不一样，不晓得是不是字节跟字符没有转换的原因还是其他的。看雪精华 5 里有 Blowfish 加密解密的源代码，可惜我不会用。  
错误与不足之处请指正！  
如果有提供 Blowfish 计算工具的（最好输入格式为字节），不胜感谢！ 

 4dd466d0e1e0cc123afbe504da2462ca