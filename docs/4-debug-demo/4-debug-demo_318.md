

# 天网防火墙 2.4.6

> 原文：[`www.pediy.com/kssd/pediy04/kanxue271.htm`](https://www.pediy.com/kssd/pediy04/kanxue271.htm)

改一个字节使天网防火墙 2.4.6 永不过期.   
  
破解者：tieji   
破解时间：2001-11-23   
  
注册码可以免费得到,不用破解,现破解的是一个月后弹出过期提示框.   
找到 004033D3  call dword ptr \[edx+000000D8\]  是弹出过期提示框的地方,具体怎样找到的,   
参见《看雪论坛精华Ⅲ》,这里就不再说了.   
  
\* Referenced by a CALL at Addresses:   
|:00405593  , :004056C9  , :0040E4E7  , :0042022F  , :0042044E    
|:004205F4  , :00420720                          
|                                              <============原来有这么多地方来到这里,究竟是   
                                                  哪一个呢,经测试是 00405593,而其他几个是干什么   
                                                  的,请各位大侠帮助研究一下.下面转到 00405593:   
:00403340 55                      push ebp   
:00403341 8BEC                    mov ebp, esp   
:00403343 83C4C0                  add esp, FFFFFFC0   
:00403346 53                      push ebx   
:00403347 56                      push esi   
:00403348 57                      push edi   
:00403349 894DC4                  mov dword ptr \[ebp\-3C\], ecx   
:0040334C 8955FC                  mov dword ptr \[ebp\-04\], edx   
:0040334F 8945C8                  mov dword ptr \[ebp\-38\], eax   
:00403352 B8789F5100              mov eax, 00519F78   
:00403357 E8406A0F00              call 004F9D9C   
:0040335C C745E801000000          mov \[ebp\-18\], 00000001   
:00403363 8D55FC                  lea edx, dword ptr \[ebp\-04\]   
:00403366 8D45FC                  lea eax, dword ptr \[ebp\-04\]   
:00403369 E87A421100              call 005175E8   
:0040336E FF45E8                  inc \[ebp\-18\]   
:00403371 66C745DC0800            mov \[ebp\-24\], 0008   
:00403377 8B4DC4                  mov ecx, dword ptr \[ebp\-3C\]   
:0040337A B201                    mov dl, 01   
  
\* Possible StringData Ref from Data Obj ->"?L"   
                                  |   
:0040337C A154205200              mov eax, dword ptr \[00522054\]   
:00403381 E8069A0100              call 0041CD8C   
:00403386 8945C0                  mov dword ptr \[ebp\-40\], eax   
:00403389 66C745DC1400            mov \[ebp\-24\], 0014   
:0040338F 66C745DC2000            mov \[ebp\-24\], 0020   
:00403395 8D55FC                  lea edx, dword ptr \[ebp\-04\]   
:00403398 8D45F8                  lea eax, dword ptr \[ebp\-08\]   
:0040339B E848421100              call 005175E8   
:004033A0 FF45E8                  inc \[ebp\-18\]   
:004033A3 8B10                    mov edx, dword ptr \[eax\]   
:004033A5 8B4DC0                  mov ecx, dword ptr \[ebp\-40\]   
:004033A8 8B81DC020000            mov eax, dword ptr \[ecx+000002DC\]   
:004033AE E8A16B0B00              call 004B9F54   
:004033B3 FF4DE8                  dec \[ebp\-18\]   
:004033B6 8D45F8                  lea eax, dword ptr \[ebp\-08\]   
:004033B9 BA02000000              mov edx, 00000002   
:004033BE E84D431100              call 00517710   
:004033C3 8B55C0                  mov edx, dword ptr \[ebp\-40\]   
:004033C6 8B45C8                  mov eax, dword ptr \[ebp\-38\]   
:004033C9 E81E230000              call 004056EC   
:004033CE 8B45C0                  mov eax, dword ptr \[ebp\-40\]   
:004033D1 8B10                    mov edx, dword ptr \[eax\]   
:004033D3 FF92D8000000            call dword ptr \[edx+000000D8\]  <========弹出过期提示框   
                                                                  往上看是哪里跳过来的   
:004033D9 66C745DC0800            mov \[ebp\-24\], 0008   
:004033DF 8B4DC0                  mov ecx, dword ptr \[ebp\-40\]   
:004033E2 894DF0                  mov dword ptr \[ebp\-10\], ecx   
:004033E5 837DF000                cmp dword ptr \[ebp\-10\], 00000000   
:004033E9 7421                    je 0040340C   
:004033EB 8B45F0                  mov eax, dword ptr \[ebp\-10\]   
:004033EE 8B10                    mov edx, dword ptr \[eax\]   
:004033F0 8955F4                  mov dword ptr \[ebp\-0C\], edx   
:004033F3 66C745DC3800            mov \[ebp\-24\], 0038   
  
  
\===========================================================================================   
  
:00405546 8D45C8                  lea eax, dword ptr \[ebp\-38\]   
:00405549 5A                      pop edx   
:0040554A E869000000              call 004055B8        <=============跟进看看   
:0040554F 84C0                    test al, al          <============测试 al 是否为零   
:00405551 7445                    je 00405598          <============al 为零,表示没过期,跳过过期提示框   
                                                        什么时候给 al 赋值,跟进上面的 call 看看   
:00405553 66C745E42000            mov \[ebp\-1C\], 0020   
:00405559 8D45F8                  lea eax, dword ptr \[ebp\-08\]   
:0040555C E8FFC7FFFF              call 00401D60   
:00405561 8BD0                    mov edx, eax   
:00405563 FF45F0                  inc \[ebp\-10\]   
  
\* Possible Reference to String Resource ID=30104: "  ?靹 Q?www.sky.net.cn) }"   
                                  |   
:00405566 B898750000              mov eax, 00007598   
:0040556B E8906A0E00              call 004EC000   
:00405570 8D55F8                  lea edx, dword ptr \[ebp\-08\]   
:00405573 8D45FC                  lea eax, dword ptr \[ebp\-04\]   
:00405576 E8C5211100              call 00517740   
:0040557B FF4DF0                  dec \[ebp\-10\]   
:0040557E 8D45F8                  lea eax, dword ptr \[ebp\-08\]   
:00405581 BA02000000              mov edx, 00000002   
:00405586 E885211100              call 00517710   
:0040558B 33C9                    xor ecx, ecx   
:0040558D 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00405590 8B45D0                  mov eax, dword ptr \[ebp\-30\]   
:00405593 E8A8DDFFFF              call 00403340    <=================这里调用   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00405551(C)   
|   
:00405598 FF4DF0                  dec \[ebp\-10\]   
  
\=====================================================================================   
  
\* Referenced by a CALL at Address:   
|:0040554A    
|   
:004055B8 55                      push ebp   
:004055B9 8BEC                    mov ebp, esp   
:004055BB 83C4F8                  add esp, FFFFFFF8   
:004055BE 8955F8                  mov dword ptr \[ebp\-08\], edx   
:004055C1 8945FC                  mov dword ptr \[ebp\-04\], eax   
:004055C4 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
:004055C7 DD00                    fld qword ptr \[eax\]   
:004055C9 8B55F8                  mov edx, dword ptr \[ebp\-08\]   
:004055CC DC1A                    fcomp qword ptr \[edx\]   
:004055CE DFE0                    fstsw ax   
:004055D0 9E                      sahf   
:004055D1 0F97C0                  seta al   
:004055D4 83E001                  and eax, 00000001    <==========这里 eax 和 00000001 与一下,   
                                                        要 al 为零,将代码改为: and eax, 00000000   
                                                        就可使程序永不过期.即将 83E001 改为 83E000   
:004055D7 59                      pop ecx   
:004055D8 59                      pop ecx   
:004055D9 5D                      pop ebp   
:004055DA C3                      ret   
这样将 1 改为 0,只改一个字节使天网防火墙 2.4.6 永不过期. 

 4dd466d0e1e0cc123afbe504da2462ca