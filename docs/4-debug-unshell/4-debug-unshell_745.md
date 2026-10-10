

# 菜鸟也脱壳:手动脱壳入门 MoleBox 2.x 和 Patch

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-534.htm`](https://www.pediy.com/kssd/pediy07/pediy7-534.htm)

【脱文标题】 手动脱壳入门 MoleBox 2.x 和 Patch IAT 加密  
  
【使用工具】 Peid,Ollydbg,ImportREC1.6f,Loadpe  
  
【脱文作者】 lhg  
  
【脱壳声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：)  
  
【脱壳内容】  
  
前几天,在 FCG 论坛看到 Compare It!3.8 的破解补丁,想看看它是如何 PATCH.  
  
首先 Peid 查壳，为 MoleBox 2.x.x -> Mole Studio，OD 载入运行，无任何异常，判断其为压缩壳。  
  
  
005592EC w>  60               pushad  
005592ED     E8 4F000000      call wincmp3.00559341  
005592F2     25 72A89805      and eax,598A872  
005592F7     A5               movs dword ptr es:\[edi\],dword ptr ds:\[es>  
005592F8     304460 FB        xor byte ptr ds:\[eax-5\],al  
005592FC     D86E 2F          fsubr dword ptr ds:\[esi+2F\]  
005592FF     49               dec ecx  
00559300     AD               lods dword ptr ds:\[esi\]  
00559301     20C9             and cl,cl  
00559303   ^ E2 8A            loopd short wincmp3.0055928F  
00559305     0AD9             or bl,cl  
  
F7 步入几次后变成如下代码：  
  
00559290     E8 DBFBFFFF      call wincmp3.00558E70  
00559295     58               pop eax  
00559296     E8 75030000      call wincmp3.00559610  
0055929B     58               pop eax  
0055929C     894424 1C        mov dword ptr ss:\[esp+1C\],eax  
005592A0     61               popad     //在此下断 eax=0048ED77  
005592A1     FFE0             jmp eax   //飞向光明之巅  
  
  
0048ED77     55               push ebp  
0048ED78     8BEC             mov ebp,esp  
0048ED7A     6A FF            push -1  
0048ED7C     68 B8774E00      push wincmp3.004E77B8  
0048ED81     68 6C374900      push wincmp3.0049376C  
0048ED86     64:A1 00000000   mov eax,dword ptr fs:\[0\]  
0048ED8C     50               push eax  
0048ED8D     64:8925 00000000 mov dword ptr fs:\[0\],esp  
  
脱壳后发现程序不能运行,这时需要用 Imprec 修复引入函数表(Import Table)  
  
在 Oep 处填 8ED77,点 IT 自动搜索,然后点获输入信息,有 23 个指针没有修复。  
  
2. Patch IAT 躲开 IAT 加密  
  
  
加密处理后的 IAT 表(如下)  
  
(选择第一个被加密的函数地址在 004D6240,重启 OD,对\[004d6240\]下内存写入断点  
  
  
0055DBA4     66:8901          mov word ptr ds:\[ecx\],ax   //断下,F9  
0055DBA7     8B45 F8          mov eax,dword ptr ss:\[ebp-8\]  
0055DBAA     40               inc eax  
0055DBAB     40               inc eax  
0055DBAC     8945 F8          mov dword ptr ss:\[ebp-8\],eax  
  
0055DBC2     66:8901          mov word ptr ds:\[ecx\],ax //断下,F9 几次后  
0055DBC5     8B45 F8          mov eax,dword ptr ss:\[ebp-8\]  
0055DBC8     40               inc eax  
0055DBC9     40               inc eax  
  
  
0055F093     8B4D E8          mov ecx,dword ptr ss:\[ebp-18\]  
0055F096     8901             mov dword ptr ds:\[ecx\],eax    //断下 ; KERNEL32.UnmapViewOfFile  
0055F098     EB 2C            jmp short wincmp3.0055F0C6      
  
eax=77E77179 (KERNEL32.UnmapViewOfFile)  
ds:\[004D6240\]=000F9708    
  
//到这里 eax 为正确的 IAT,放入\[004d6240\]  
//在看看它是如何加密,继续 F8,到 0055F0C6  
  
  
0055F0C6     8B55 F4          mov edx,dword ptr ss:\[ebp-C\]  
0055F0C9     81E2 FF000000    and edx,0FF  
0055F0CF     85D2             test edx,edx  
0055F0D1     74 17            je short wincmp3.0055F0EA           //判断是否加密,改为 JMP  
0055F0D3     8B45 E4          mov eax,dword ptr ss:\[ebp-1C\]       //为了看它如何加密,继续走  
0055F0D6     50               push eax  
0055F0D7     8B0D F89A5600    mov ecx,dword ptr ds:\[569AF8\]       ; wincmp3.00569AFC  
0055F0DD     51               push ecx  
0055F0DE     8B55 E8          mov edx,dword ptr ss:\[ebp-18\]       ; wincmp3.004D6240   
0055F0E1     52               push edx  
0055F0E2     E8 C9060000      call wincmp3.0055F7B0               //加密 call,跟进  
0055F0E7     83C4 0C          add esp,0C  
0055F0EA   ^ E9 5BFFFFFF      jmp wincmp3.0055F04A  
0055F0EF     8B45 EC          mov eax,dword ptr ss:\[ebp-14\]  
0055F0F2     A3 F89A5600      mov dword ptr ds:\[569AF8\],eax  
0055F0F7     8B4D FC          mov ecx,dword ptr ss:\[ebp-4\]  
0055F0FA     6BC9 14          imul ecx,ecx,14  
0055F0FD     8B55 08          mov edx,dword ptr ss:\[ebp+8\]  
0055F100     C7440A 04 FFFFFF>mov dword ptr ds:\[edx+ecx+4\],-1  
0055F108   ^ E9 12FEFFFF      jmp wincmp3.0055EF1F  
0055F10D     8BE5             mov esp,ebp  
0055F10F     5D               pop ebp  
0055F110     C3               retn  
  
在 Oep 处填 0008ED77,点 IT 自动搜索,然后点获输入信息,全部有效，修复脱壳文件，正常运行。  
  
0055F7B0     55               push ebp  
0055F7B1     8BEC             mov ebp,esp  
0055F7B3     83EC 10          sub esp,10  
0055F7B6     C745 FC 00000000 mov dword ptr ss:\[ebp-4\],0  
0055F7BD     833D 88F05600 00 cmp dword ptr ds:\[56F088\],0  
0055F7C4     75 0A            jnz short wincmp3.0055F7D0  
0055F7C6     B9 0A0000EF      mov ecx,EF00000A  
0055F7CB     E8 34200000      call wincmp3.00561804  
0055F7D0     8B45 08          mov eax,dword ptr ss:\[ebp+8\]   //壳的地址; wincmp3.004D6240  
0055F7D3     8B08             mov ecx,dword ptr ds:\[eax\]  
0055F7D5     51               push ecx  
0055F7D6     8B0D 88F05600    mov ecx,dword ptr ds:\[56F088\]  
0055F7DC     E8 DA470000      call wincmp3.00563FBB  
0055F7E1     8945 F8          mov dword ptr ss:\[ebp-8\],eax  
0055F7E4     837D F8 00       cmp dword ptr ss:\[ebp-8\],0  
0055F7E8     74 45            je short wincmp3.0055F82F  
0055F7EA     8D55 F0          lea edx,dword ptr ss:\[ebp-10\]  
0055F7ED     52               push edx  
0055F7EE     6A 04            push 4  
0055F7F0     6A 04            push 4  
0055F7F2     8B45 08          mov eax,dword ptr ss:\[ebp+8\]  
0055F7F5     50               push eax  
0055F7F6     FF15 4CA95600    call dword ptr ds:\[56A94C\]               ; KERNEL32.VirtualProtect  
0055F7FC     85C0             test eax,eax  
0055F7FE     75 0A            jnz short wincmp3.0055F80A  
0055F800     B9 0B0000EF      mov ecx,EF00000B  
0055F805     E8 FA1F0000      call wincmp3.00561804  
0055F80A     8B4D 08          mov ecx,dword ptr ss:\[ebp+8\]  
0055F80D     8B55 F8          mov edx,dword ptr ss:\[ebp-8\]  
0055F810     8B02             mov eax,dword ptr ds:\[edx\]  
0055F812     8901             mov dword ptr ds:\[ecx\],eax  //将壳的地址替换 正确的 IAT  
  
//eax=00563492 (wincmp3.00563492)  
//ds:\[004D6240\]=77E77179 (KERNEL32.UnmapViewOfFile)  
  
//我们必需让 EAX 是正确的指针。我们看到 0055F7D0  处 EAX 被赋值，而且当时的 EAX 是正确指针。  
  
//故 0055F7D0 处  
//0055F7D0     8B45 08          mov eax,dword ptr ss:\[ebp+8\]  
//修改如下:  
//0055F7D0     8BCD            mov eax,eax  
//0055F7D2     90              nop  
  
0055F814     8D4D F4          lea ecx,dword ptr ss:\[ebp-C\]  
0055F817     51               push ecx  
0055F818     8B55 F0          mov edx,dword ptr ss:\[ebp-10\]  
0055F81B     52               push edx  
0055F81C     6A 04            push 4  
0055F81E     8B45 08          mov eax,dword ptr ss:\[ebp+8\]  
0055F821     50               push eax  
0055F822     FF15 4CA95600    call dword ptr ds:\[56A94C\]               ; KERNEL32.VirtualProtect  
0055F828     C745 FC 01000000 mov dword ptr ss:\[ebp-4\],1  
0055F82F     8B45 FC          mov eax,dword ptr ss:\[ebp-4\]  
0055F832     8BE5             mov esp,ebp  
0055F834     5D               pop ebp  
0055F835     C3               retn  
  
  
加密处理后的 IAT 表:  
  
004D6240  00563492  wincmp3.00563492  //被加密  
004D6270  005628D4  wincmp3.005628D4  //被加密  
004D62B0  005634B9  wincmp3.005634B9  //被加密  
004D6324  00563DC1  wincmp3.00563DC1  //被加密  
004D635C  00562CDA  wincmp3.00562CDA  //被加密  
004D6364  00562D76  wincmp3.00562D76  //被加密  
004D6368  00562F3F  wincmp3.00562F3F  //被加密  
004D6370  00562FA2  wincmp3.00562FA2  //被加密  
004D63A8  00562B71  wincmp3.00562B71  //被加密  
004D63B0  00563453  wincmp3.00563453  //被加密  
004D63BC  0056301E  wincmp3.0056301E  //被加密  
004D63D4  00563282  wincmp3.00563282  //被加密  
004D63EC  00562FE6  wincmp3.00562FE6  //被加密  
004D6400  00562995  wincmp3.00562995  //被加密  
004D6404  005636C1  wincmp3.005636C1  //被加密  
004D6408  00563715  wincmp3.00563715  //被加密  
004D640C  005636EE  wincmp3.005636EE  //被加密  
004D6450  005635AD  wincmp3.005635AD  //被加密  
004D6458  00563384  wincmp3.00563384  //被加密  
004D645C  00563096  wincmp3.00563096  //被加密  
004D6460  00562F18  wincmp3.00562F18  //被加密  
004D6468  00562B23  wincmp3.00562B23  //被加密  
004D6470  00563D7A  wincmp3.00563D7A  //被加密 

 4dd466d0e1e0cc123afbe504da2462ca