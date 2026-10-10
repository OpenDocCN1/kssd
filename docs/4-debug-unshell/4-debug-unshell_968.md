

# safeguard1.03 主程序之“游晕惊萝”之浅游（未脱）

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-578.htm`](https://www.pediy.com/kssd/pediy07/pediy7-578.htm)

safeguard1.03 主程序之“游晕惊萝”之浅游（未脱）  
  
注：我懒了，累了，无法组装更耐可看的文章，失误之处还请指正！  
  
EB??EB??  
DF694E58DF5974EEEB01DF  
75E99A599C81C1E2FFFFFFEB01DF9DFFE1  
E851E8EBFFFFFFDF223F9AC08D9D808E4000EB4DEB47DF694E58DF5979EEEB01  
DF78E9DF599C81C1E5FFFFFF9DFFE1EB51E8EEFFFFFFE8BAA3223F  
  
????????9AC0\[code\]EB??EB47  
DF??????????C0\[code\]EB??EB47  
  
上面摘了些 simonzh“花无缺”的一些“花瓣”，有可能摘得不完整，主要对付是最后的两个，用得很多  
  
壳发动 除零, int3, 非法内存访问指令，单步跟踪 等异常展开“捉迷藏”活动！  
前面是带人游“御花园”，如果你晕了，就要补补身子，int3 中有惊无有险，看你能否耐住性子！   
  
这个壳的一大亮点是，几乎所有壳解码运行的每条指令都要包装“花心”，举步为艰，难为了“鼠标”和眼睛，  
要摘取相关指令代码真是头痛，经过一番头晕后，写了点脚本突出“花心”的“花蜜指令”，从头一直来到：  
  
解码是一块解一块，如不了解个中动作，呵呵，请勿乱 Shift  
第一次解码：计算 al，它是解码种子  
/\*4157CD\*/  pop dword ptr fs:\[0\]  
/\*4157D4\*/  pop eax  
/\*4157D5\*/  rol al,cl  
/\*4157D7\*/  clc  
/\*4157D8\*/  xor al,cl  
/\*4157DA\*/  sub al,cl  
/\*4157DC\*/  clc  
/\*4157E0\*/  xor al,cl  
/\*4157E5\*/  sub al,cl  
/\*4157E7\*/  rol al,cl  
/\*4157E9\*/  sub al,cl  
/\*4157EB\*/  add al,cl  
/\*4157ED\*/  ror al,cl  
/\*4157EF\*/  sub al,cl  
/\*4157F1\*/  add al,35  
/\*4157F3\*/  xor al,1C  
/\*4157F5\*/  rol al,0B7  
/\*4157F8\*/  clc  
/\*4157F9\*/  clc  
/\*4157FA\*/  clc  
/\*4157FB\*/  xor al,cl  
/\*4157FD\*/  add al,cl  
/\*4157FF\*/  stc  
/\*415800\*/  sub al,cl  
/\*415805\*/  clc  
/\*415806\*/  sub al,cl  
/\*41580B\*/  xor al,cl  
/\*41580D\*/  xor al,cl  
/\*415811\*/  sub al,61  
/\*415813\*/  stc  
////////////////////////////////////////////  
/\*415814\*/  call 00415819  
/\*415822\*/  pop ebp  
/\*41584F\*/  sub ebp,401819  
/\*415880\*/  pushad  
/\*4158AC\*/  lea esi,dword ptr ss:\[ebp+4019EE\]  
/\*4158DB\*/  mov edi,esi  
/\*415908\*/  mov ecx,0CB2B           ; size  
  
/\*415936\*/  lods byte ptr ds:\[esi\] // 解码：4159EE  
/\*415966\*/  rol al,8F  
/\*41596A\*/  inc al  
/\*41596C\*/  xor al,cl  
/\*41596E\*/  rol al,35  
/\*415971\*/  xor al,83  
/\*415977\*/  sub al,41  
/\*415979\*/  xor al,cl  
/\*41597B\*/  inc al  
/\*415984\*/  add al,6A  
/\*415986\*/  sub al,cl  
/\*415988\*/  ror al,cl  
/\*41598A\*/  xor al,cl  
/\*415999\*/  stos byte ptr es:\[edi\]  
/\*4159C8\*/  loopd short 0041595A   // loop 415936  
/\*4159CA\*/  jmp short 00415A17     //\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
  
  
; 读取 Kernel32.dll 基址，并查找符合要求的 API 名的 XOR 值  
/\*4159F3\*/  mov eax,dword ptr fs:\[30\]  
/\*415A22\*/  add eax,0C  
/\*415A50\*/  mov eax,dword ptr ds:\[eax\]  
/\*415A7D\*/  add eax,1C  
/\*415AAB\*/  mov eax,dword ptr ds:\[eax\]  
/\*415AD6\*/  mov eax,dword ptr ds:\[eax\]  
/\*415B01\*/  add eax,8  
/\*415B2F\*/  mov eax,dword ptr ds:\[eax\]        // 获取 Kernel32.dll 基址  
/\*415B5C\*/  mov edi,eax  
/\*415B87\*/  add eax,dword ptr ds:\[eax+3C\]  
/\*415BB3\*/  mov ecx,dword ptr ds:\[eax+78\]  
/\*415BE1\*/  add ecx,edi  
/\*415C0C\*/  lea esi,dword ptr ss:\[ebp+408FA8\] // 定位到一份 API 数据表入口：41CFA8  
/\*415C3B\*/  call 0041C7C3                      ; 获取所需的 API 地址  
  
计算 API 名的 XOR 值  
/\*41D071\*/  push ebx  
/\*41D072\*/  push edx  
/\*41D073\*/  push esi  
/\*41D074\*/  call 0041D079  
/\*41D079\*/  pop ebx  
/\*41D07A\*/  sub ebx,409079  
/\*41D080\*/  add ebx,4090A7  
/\*41D086\*/  mov esi,eax  
/\*41D088\*/  xor edx,edx  
/\*41D08A\*/  mov eax,-1  
/\*41D08F\*/  jmp short 0041D09C  
/\*41D091\*/  mov dl,byte ptr ds:\[esi\]  
/\*41D093\*/  xor dl,al  
/\*41D095\*/  shr eax,8  
/\*41D098\*/  xor eax,dword ptr ds:\[ebx+edx\*4\]  
/\*41D09B\*/  inc esi  
/\*41D09C\*/  cmp byte ptr ds:\[esi\],0  
/\*41D09F\*/  jnz short 0041D091  
/\*41D0A1\*/  not eax  
/\*41D0A3\*/  pop esi  
/\*41D0A4\*/  pop edx  
/\*41D0A5\*/  pop ebx  
/\*41D0A6\*/  retn  
  
/\*415C6E\*/  lea esi,dword ptr ss:\[ebp+403E78\] // 定位到另一份 API 数据表入口：417E78  
/\*415C9F\*/  call 0041C7C3  
  
简单的 not 解码加密黑名单。。。  
/\*415FEB\*/  call dword ptr ss:\[ebp+408FDC\] ; ss:\[0041CFDC\]kernel32.CreateFileA 驱动检测  
\\\\.\\SICE  
\\\\.\\NTICE  
\\\\.\\REGMON  
\\\\.\\REGMON70  
\\\\.\\FILEMON  
\\\\.\\FILEMON70  
\\\\.\\ICEDUMP  
\\\\.\\IceExt  
\\\\.\\FROGSICE  
  
/\*4161E8\*/  mov eax,dword ptr fs:\[18\]  
/\*416217\*/  mov eax,dword ptr ds:\[eax+34\] // 测试系统类型？: here is: 2  
/\*416245\*/  cmp eax,5  
/\*416273\*/  je 0041C129  
/\*4162A5\*/  dec ebx  
/\*4162D6\*/  jnz 00415D4F  
  
/\*416305\*/  lea esi,dword ptr ss:\[ebp+40E554\] ; 00422554 "Advapi32.dll"  
/\*416336\*/  push esi  
/\*416362\*/  call dword ptr ss:\[ebp+408FC8\]    ; ss:\[0041CFC8\] kernel32.LoadLibraryA  
/\*416393\*/  mov dword ptr ss:\[ebp+408FF0\],eax  
/\*4163C2\*/  lea esi,dword ptr ss:\[ebp+408DFE\] ; 0041CDFE "OpenProcessToken"  
/\*4163F3\*/  push esi  
/\*41641F\*/  push dword ptr ss:\[ebp+408FF0\]    ; ss:\[0041CFF0\]=77DA0000 Advapi32  
/\*416451\*/  call dword ptr ss:\[ebp+408FC4\]    ; ss:\[0041CFC4\] kernel32.GetProcAddress  
/\*416487\*/  mov dword ptr ss:\[ebp+408DFE\],eax ; ss:\[0041CDFE\] 地址放回原来的 API 字符名位置  
...  
/\*4166A7\*/  call dword ptr ss:\[ebp+408FE0\]    ; ss:\[0041CFE0\]  kernel32.OutputDebugStringA  
/\*4166D9\*/  lea eax,dword ptr ss:\[ebp+408D0C\] ; pToken 结构地址\[\]：0041CD0C  
/\*416708\*/  push eax                          ; &hToken  
/\*416739\*/  push 28                           ; TOKEN\_ADJUST\_PRIVILEGES  
/\*416769\*/  call dword ptr ss:\[ebp+408FD0\]    ; ss:\[0041CFD0\] kernel32.GetCurrentProcess  
/\*41679A\*/  push eax  
/\*4167C4\*/  call dword ptr ss:\[ebp+408DFE\]    ; ss:\[0041CDFE\]  Advapi32.OpenProcessToken  
/\*4167F8\*/  lea esi,dword ptr ss:\[ebp+408D10\] ; 0041CD10 //对 TOKEN\_PRIVILEGES 结构进行初始化  
/\*416827\*/  mov dword ptr ds:\[esi\], 1         ; p 结构地址\[\]+4  
/\*416858\*/  mov dword ptr ds:\[esi+C\],2  
/\*416888\*/  lea eax,dword ptr ss:\[ebp+408D14\] ; 0041CD14 -> &tp.Privileges\[0\].Luid  
/\*4168B9\*/  push eax  
/\*4168E6\*/  lea eax,dword ptr ss:\[ebp+408D4A\] ; 0041CD4A   ASCII "SeDebugPrivilege" //一定要是字符标识，不是以数值标识  
/\*416917\*/  push eax                          ; szPrivName  
/\*416943\*/  push 0                            ; NULL  
/\*41696E\*/  call dword ptr ss:\[ebp+408E0F\]    ; ss:\[0041CE0F\]  Advapi32.LookupPrivilegeValueA //将特权名字转换成一个 64 位的等价数值叫做 LUID（本地独有标识）。  
/\*4169A4\*/  push 0                            ; Dummy  
/\*4169D1\*/  push 0                            ; nil  
/\*416A01\*/  push 0                            ; SizeOf(TP)  
/\*416A2E\*/  push esi                          ; TP  
/\*416A58\*/  push 0                            ; False  
/\*416A8A\*/  push dword ptr ss:\[ebp+408D0C\]    ; hToken  
/\*416ABC\*/  call dword ptr ss:\[ebp+408E25\]    ; ss:\[0041CE25\]  Advapi32.AdjustTokenPrivileges //要申请 TOKEN\_ADJUST\_PRIVILEGES 权限， 这时成功调用 AdjustTokenPrivileges 的权限  
/\*416AF2\*/  push dword ptr ss:\[ebp+408D0C\]  
/\*416B23\*/  call dword ptr ss:\[ebp+403EA4\]    ; ss:\[00417EA4\]  kernel32.CloseHandle  
/\*416B52\*/  lea eax,dword ptr ss:\[ebp+408220\] ; 41C220  
/\*416B83\*/  push eax  
/\*416BB2\*/  push dword ptr fs:\[0\]  
/\*416BE2\*/  mov dword ptr fs:\[0\],esp          ; 安装异常  
/\*416C14\*/  mov edi,ebp  
/\*416C3F\*/  mov ebp,4243484B  
/\*416C6F\*/  mov ax,4  
  
/\*41C220\*/  push ebp  
/\*41C221\*/  mov ebp,esp  
/\*41C223\*/  push ebx  
/\*41C22B\*/  mov eax,dword ptr ss:\[ebp+10\]     ; 获取 Context 结构  
/\*41C257\*/  xor ebx,ebx                       ; 经典的硬处理  
/\*41C284\*/  inc dword ptr ds:\[eax+B8\]  
/\*41C2B8\*/  mov dword ptr ds:\[eax+4\],ebx  
/\*41C2E4\*/  mov dword ptr ds:\[eax+8\],ebx  
/\*41C312\*/  mov dword ptr ds:\[eax+C\],ebx  
/\*41C33E\*/  mov dword ptr ds:\[eax+10\],ebx  
/\*41C36C\*/  mov dword ptr ds:\[eax+18\],155  
/\*41C39C\*/  xor eax,eax  
/\*41C3C2\*/  pop ebx  
/\*41C3C3\*/  leave  
/\*41C3C4\*/  retn 10  
  
  
/\*416C97\*/  int3                              ; 发动 int3 异常  
/\*416C98\*/  nop  
/\*416CA2\*/  pop dword ptr fs:\[0\]  
/\*416CD2\*/  add esp,4  
/\*416CFE\*/  mov ebp,edi  
/\*416D2B\*/  cmp al,4  
/\*416D58\*/  jnz 0041C129                      ; end your life  
/\*416D8C\*/  call dword ptr ss:\[ebp+403E7C\]    ; ss:\[00417E7C\]  kernel32.GetTickCount  
/\*416DBB\*/  lea esi,dword ptr ss:\[ebp+403E5C\] ; 载入保存时间地址  
/\*416DEA\*/  mov dword ptr ds:\[esi\],eax        ; 待定 Anti  
/\*416E15\*/  lea esi,dword ptr ss:\[ebp+408D66\] ; 地址=0041CD66, ASCII "ntdll.dll"  
/\*416E46\*/  push esi  
/\*416E72\*/  call dword ptr ss:\[ebp+408FC8\]    ; ss:\[0041CFC8\]=77E605D8 kernel32.LoadLibraryA  
/\*416EA4\*/  mov dword ptr ss:\[ebp+408D66\],eax ;  
/\*416ED5\*/  lea esi,dword ptr ss:\[ebp+408E52\] ; 地址=0041CE52, (ASCII "ZwQueryInformationProcess")  
/\*416F06\*/  push esi  
/\*416F30\*/  push dword ptr ss:\[ebp+408D66\]  
/\*416F66\*/  call dword ptr ss:\[ebp+408FC4\]    ; ss:\[0041CFC4\]=77E5A5FD (kernel32.GetProcAddress)  
/\*416F97\*/  mov dword ptr ss:\[ebp+408E52\],eax  
/\*416FC6\*/  lea eax,dword ptr ss:\[ebp+4083C7\] ; seh: 0041C3C7  
/\*416FF7\*/  push eax  
/\*417023\*/  push dword ptr fs:\[0\]  
/\*417053\*/  mov dword ptr fs:\[0\],esp  
/\*41707E\*/  int3                              ; 发动 int3 异常  
/\*41707F\*/  nop  
/\*417080\*/  js short 00417068  
  
  
/\*41C3C7\*/  push ebp  
/\*41C3C8\*/  mov ebp,esp  
/\*41C3CA\*/  push ebx  
/\*41C3CB\*/  push ebp  
/\*41C3D3\*/  mov eax,dword ptr ss:\[ebp+10\]  
/\*41C3FF\*/  mov ebp,dword ptr ds:\[eax+B4\]  
/\*41C42E\*/  lea ebx,dword ptr ss:\[ebp+40433C\]  ; newContext.0041833C  
/\*41C45F\*/  mov dword ptr ds:\[eax+B8\],ebx  
/\*41C490\*/  xor ebx,ebx  
/\*41C4BD\*/  mov dword ptr ds:\[eax+4\],ebx  
/\*41C4E9\*/  mov dword ptr ds:\[eax+8\],ebx  
/\*41C515\*/  mov dword ptr ds:\[eax+C\],ebx  
/\*41C543\*/  mov dword ptr ds:\[eax+10\],ebx  
/\*41C571\*/  mov dword ptr ds:\[eax+18\],155  
/\*41C5A1\*/  xor eax,eax  
/\*41C5C7\*/  pop ebp  
/\*41C5C8\*/  pop ebx  
/\*41C5C9\*/  leave  
/\*41C5CA\*/  retn 10  
  
  
0041833C    jmp short 0041836A                  ; safeguar.0041836A  
/\*418345\*/  pop dword ptr fs:\[0\]  
/\*418375\*/  add esp,4  
/\*4183A3\*/  lea esi,dword ptr ss:\[ebp+4044E5\]  
/\*4183D2\*/  mov edi,esi  
/\*4183FF\*/  mov ecx,3C44  
  
/\*41842D\*/  lods byte ptr ds:\[esi\] // 解码 4184E5  
/\*41845A\*/  rol al,cl  
/\*41845C\*/  rol al,84  
/\*41845F\*/  sub al,8F  
/\*418461\*/  stc  
/\*418462\*/  clc  
/\*418467\*/  sub al,cl  
/\*41846C\*/  add al,cl  
/\*418471\*/  rol al,cl  
/\*418473\*/  add al,6E  
/\*418478\*/  ror al,cl  
/\*41847A\*/  add al,cl  
/\*41847C\*/  ror al,cl  
/\*41847E\*/  clc  
/\*418482\*/  rol al,cl  
/\*418484\*/  add al,0D5  
/\*418486\*/  stc  
/\*418490\*/  stos byte ptr es:\[edi\]  
/\*4184BF\*/  loopd short 00418451   // loopd 41842D  
/\*4184C1\*/  jmp short 0041850E     // \*\*\*\*\*\*\*\*\*\*\*\*\*  
  
/\*4184EA\*/  mov eax,dword ptr fs:\[30\]  
/\*418519\*/  movzx eax,byte ptr ds:\[eax+2\]  
/\*418548\*/  test eax,eax  
/\*418575\*/  jnz 0041C129 // goto white space  
/\*4185A4\*/  lea esi,dword ptr ss:\[ebp+403E5C\]  
/\*4185D5\*/  mov edi,dword ptr ds:\[esi\]  
/\*418602\*/  call dword ptr ss:\[ebp+403E7C\]   //ss:\[00417E7C\] kernel32.GetTickCount  
/\*418631\*/  mov dword ptr ds:\[esi\],eax  
/\*41865E\*/  sub eax,edi  
/\*418689\*/  cmp eax,7D0  
/\*4186B9\*/  ja 0041C129    // game-> push 0   ->call kernel32.CloseHandle -> push 0  ->call kernel32.ExitProcess  
/\*4186EA\*/  push 0  
/\*418717\*/  push 2  
/\*418744\*/  call dword ptr ss:\[ebp+408FA8\]  ; kernel32.CreateToolhelp32Snapshot  
/\*418773\*/  mov edi,eax  
/\*4187AC\*/  call dword ptr ss:\[41CFE0\]      ; kernel32.OutputDebugStringA  
/\*4187DE\*/  lea ebx,dword ptr ss:\[ebp+408E80\]  
/\*41880F\*/  mov dword ptr ds:\[ebx\],128  
/\*41883E\*/  mov esi,dword ptr ss:\[ebp+408D28\]  
/\*41889B\*/  push edi  
/\*4188C5\*/  call dword ptr ss:\[ebp+408FAC\]  ; ss:\[0041CFAC\] kernel32.Process32First  
/\*418AB4\*/  or eax,eax  
/\*418AB6\*/  jnz 004188F9 // to: 418900  
/\*418ABC\*/  jmp short 00418AE7  // to next part  
  
/\*418900\*/  lea eax,dword ptr ds:\[ebx+24\] // 取算定的指定进程名 winlogon.exe 的 CRC 值  
/\*41892E\*/  call 0041D054  
/\*41895E\*/  call 0041D071                 // 求枚举进程名的 crc  
/\*41898A\*/  cmp eax,esi  
/\*41898C\*/  jnz 00418A26                  // to: 418A2F  
/\*418992\*/  jmp short 004189BF            // to: 4189999  
  
/\*418A2F\*/  push ebx  
/\*418A5B\*/  push edi  
/\*418A8A\*/  call dword ptr ss:\[ebp+408FB0\]  ; ss:\[0041CFB0\] kernel32.Process32Next  
/\*418A90\*/  jmp short 00418AB4  
  
/\*418999\*/  push dword ptr ds:\[ebx+8\]  
/\*4189CC\*/  pop dword ptr ss:\[ebp+408E6C\]  
/\*4189FB\*/  xor eax,eax  
/\*418A21\*/  jmp 00418AB4                   // to: 418AB4  
  
/\*418AC5\*/  push edi  
/\*418AF4\*/  call dword ptr ss:\[ebp+403EA4\] ; ss:\[00417EA4\] kernel32.CloseHandle  
/\*418B23\*/  lea esi,dword ptr ss:\[ebp+403E5C\]  
/\*418B54\*/  mov edi,dword ptr ds:\[esi\]  
/\*418B81\*/  call dword ptr ss:\[ebp+403E7C\] ; ss:\[00417E7C\] kernel32.GetTickCount  
/\*418BB3\*/  mov dword ptr ds:\[esi\],eax  
/\*418BE0\*/  sub eax,edi  
/\*418C0B\*/  cmp eax,7D0  
/\*418C3B\*/  ja 0041C129                   // to: hi   
/\*418C6A\*/  mov eax,dword ptr ss:\[ebp+408E6C\] // target.ID  
/\*418C99\*/  test eax,eax  
/\*418CC6\*/  je 0041C129  
/\*418CF7\*/  push dword ptr ss:\[ebp+408E6C\]  
/\*418D28\*/  push 0  
/\*418D58\*/  push 1F0FFF  
/\*418D88\*/  call dword ptr ss:\[ebp+408FB4\] ; ss:\[0041CFB4\]=77E506B7 (kernel32.OpenProcess)  
/\*418DB7\*/  test eax,eax  
/\*418DE4\*/  je 0041C129  
/\*418E13\*/  mov dword ptr ss:\[ebp+408E70\],eax ; 保存 target 的进程句柄  
/\*418E44\*/  push 104  
/\*418E72\*/  lea eax,dword ptr ss:\[ebp+403F6C\]  
/\*418EA3\*/  push eax  
/\*418ED0\*/  push 0  
/\*418F02\*/  call dword ptr ss:\[ebp+403E78\] ; ss:\[00417E78\] kernel32.GetModuleFileNameA  
/\*418F31\*/  test eax,eax  
/\*418F5E\*/  je 0041C129  
/\*418F8D\*/  lea esi,dword ptr ss:\[ebp+408E3B\] ; 0041CE3B "ZwSetInformationThread"  
/\*418FBC\*/  push esi  
/\*418FED\*/  push dword ptr ss:\[ebp+408D66\] // ss:\[0041CD66\]=77F50000 ntdll  
/\*41901E\*/  call dword ptr ss:\[ebp+408FC4\] ; ss:\[0041CFC4\]=77E5A5FD kernel32.GetProcAddress  
/\*419050\*/  mov dword ptr ss:\[ebp+408E3B\],eax  
  
/\*419081\*/  push 0                         ; 这里应该止步了  
/\*4190AE\*/  push 0  
/\*4190DB\*/  push 11  
/\*419109\*/  call dword ptr ss:\[ebp+408FE4\] ; ss:\[0041CFE4\] kernel32.GetCurrentThread  
/\*41913F\*/  push eax  
/\*41916E\*/  call dword ptr ss:\[ebp+408E3B\] // anti  
  
/\*41919D\*/  lea eax,dword ptr ss:\[ebp+403EA8\]  
/\*4191CE\*/  mov dword ptr ds:\[eax\],44  
/\*4191FD\*/  mov dword ptr ds:\[eax+2C\],1  
/\*41922F\*/  mov word ptr ds:\[eax+30\],1  
/\*41925E\*/  lea eax,dword ptr ss:\[ebp+404070\]  
/\*41928D\*/  mov dword ptr ds:\[eax\],10001  
/\*4192BE\*/  push 40  
/\*4192EB\*/  push 1000  
/\*41931E\*/  push 2000                          // size  
/\*41934E\*/  push 0                            // dynamic  
/\*41937C\*/  push dword ptr ss:\[ebp+408E70\]   // target.handle  
/\*4193B2\*/  call dword ptr ss:\[ebp+408FB8\] ; ss:\[0041CFB8\]  kernel32.VirtualAllocEx //为注入申请空间  
/\*4193E1\*/  test eax,eax  
/\*41940E\*/  je 0041C129  
/\*419440\*/  mov dword ptr ss:\[ebp+403E64\],eax // 保存注入地址: 大小 2000  
/\*419471\*/  push 40  
/\*41949E\*/  push 1000  
/\*4194CE\*/  push 1000  
/\*419501\*/  push 0  
/\*41952E\*/  push dword ptr ss:\[ebp+408E70\]   // target.handle  
/\*419562\*/  call dword ptr ss:\[ebp+408FB8\] ; ss:\[0041CFB8\]  kernel32.VirtualAllocEx  
/\*419591\*/  test eax,eax  
/\*4195BE\*/  je 0041C129  
/\*4195ED\*/  mov dword ptr ss:\[ebp+403E68\],eax // 保存注入地址: 大小 1000  
/\*41961C\*/  lea esi,dword ptr ss:\[ebp+40575E\]  
/\*41964B\*/  mov edi,esi  
/\*419678\*/  mov ecx,29CB  
/\*4196A8\*/  lods byte ptr ds:\[esi\]  
/\*4196CD\*/  add al,cl  
/\*4196CF\*/  inc al  
/\*4196D1\*/  clc  
/\*4196D5\*/  sub al,cl  
/\*4196D7\*/  rol al,94  
/\*4196DA\*/  rol al,0D7  
/\*4196DD\*/  add al,cl  
/\*4196E2\*/  ror al,20  
/\*4196E8\*/  clc  
/\*4196E9\*/  rol al,79  
/\*4196EC\*/  xor al,64  
/\*4196EE\*/  ror al,cl  
/\*4196F0\*/  sub al,0DE  
/\*4196F2\*/  stc  
/\*4196F3\*/  sub al,1D  
/\*4196FB\*/  add al,cl  
/\*419706\*/  stos byte ptr es:\[edi\]  
/\*419733\*/  loopd short 004196C7    ; 解码 41975F  
/\*419735\*/  jmp short 00419784      ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
/\*419765\*/  push eax  
/\*419791\*/  mov eax,esp  
/\*4197BE\*/  push 0  
/\*4197EB\*/  push 4  
/\*419818\*/  push eax  
/\*419844\*/  push 7  
/\*419872\*/  call dword ptr ss:\[ebp+408FD0\]  
/\*4198A8\*/  push eax  
/\*4198D4\*/  call dword ptr ss:\[ebp+408E52\] ; ss:\[0041CE52\]  ntdll.ZwQueryInformationProcess  
/\*419932\*/  or eax,eax  
/\*41995F\*/  jnz 0041C129  
/\*41998E\*/  lea eax,dword ptr ss:\[ebp+408E74\]  
/\*4199BF\*/  push eax  
/\*4199EB\*/  push dword ptr ss:\[ebp+403E58\]    ; BytesToWrite = 1ED8  
/\*419A1D\*/  lea eax,dword ptr ss:\[ebp+406251\]  
/\*419A4E\*/  push eax                          ; Buffer = safeguar.0041A251 注入代码的 OEP  
/\*419A7A\*/  push dword ptr ss:\[ebp+403E64\]    ; Address = 第一次注入代码的地址  
/\*419AAE\*/  push dword ptr ss:\[ebp+408E70\]    ; hProcess  
/\*419ADD\*/  call dword ptr ss:\[ebp+403E9C\]    ; ss:\[00417E9C\]  kernel32.WriteProcessMemory  
/\*419B13\*/  test eax,eax  
/\*419B40\*/  je 0041C129  
/\*419B46\*/  jmp short 00419B93 // to: 00419B6F  
  
  
/\*419B6F\*/  lea esi,dword ptr ss:\[ebp+405CB1\] // 00419CB1  
/\*419B9E\*/  mov edi,esi  
/\*419BCB\*/  mov ecx,5A0  
/\*419BFB\*/  lods byte ptr ds:\[esi\]  
/\*419C20\*/  rol al,cl  
/\*419C22\*/  ror al,0A7  
/\*419C25\*/  rol al,5D  
/\*419C2B\*/  rol al,57  
/\*419C2E\*/  sub al,cl  
/\*419C30\*/  rol al,cl  
/\*419C35\*/  rol al,cl  
/\*419C37\*/  nop  
/\*419C38\*/  clc  
/\*419C39\*/  ror al,0D6  
/\*419C3F\*/  ror al,0D  
/\*419C42\*/  sub al,cl  
/\*419C47\*/  ror al,cl  
/\*419C59\*/  stos byte ptr es:\[edi\]  
/\*419C86\*/  loopd short 00419C1A   // 解码 419CB1  
/\*419C88\*/  jmp short 00419CDA     //   
  
  
/\*419CB6\*/  lea edi,dword ptr ss:\[ebp+406251\]  
/\*419CE7\*/  mov ecx,1ED8  
/\*419D17\*/  rep stos byte ptr es:\[edi\]        ; 清理战场 41A251  
/\*419D42\*/  lea eax,dword ptr ss:\[ebp+408E74\] ; 41CE74  
/\*419D73\*/  push eax  
/\*419D9F\*/  push dword ptr ss:\[ebp+403E54\]    ; Write Size  
/\*419DD1\*/  lea eax,dword ptr ss:\[ebp+403E5C\] ; buffer: 417E5C  
/\*419E00\*/  push eax  
/\*419E31\*/  push dword ptr ss:\[ebp+403E68\]    ; Address  
/\*419E65\*/  push dword ptr ss:\[ebp+408E70\]    ; hProcess  
/\*419E96\*/  call dword ptr ss:\[ebp+403E9C\]    ; ss:\[00417E9C\]  kernel32.WriteProcessMemory  
/\*419EC7\*/  test eax,eax  
/\*419EF4\*/  je 0041C129  
/\*419F25\*/  call dword ptr ss:\[ebp+403E7C\]    ; \[00417E7C\]=77E5751A  kernel32.GetTickCount  
  
/\*419F54\*/  lea esi,dword ptr ss:\[ebp+403E5C\] ; 417E5C  
/\*419F85\*/  sub eax,dword ptr ds:\[esi\]  
/\*419FB2\*/  cmp eax,7D0  
/\*419FE0\*/  ja 0041C129                       ; Anti  
/\*41A014\*/  lea eax,dword ptr ss:\[ebp+403E5C\]  
/\*41A045\*/  push eax                       ; lpThreadId       
/\*41A071\*/  push 0                         ; dwCreationFlags  
/\*41A0A1\*/  push dword ptr ss:\[ebp+403E68\] ; lpParameter  
/\*41A0D2\*/  push dword ptr ss:\[ebp+403E64\] ; lpStartAddress -> 第一次申请注入的空间地址  
/\*41A106\*/  push 0                         ; dwStackSize  
/\*41A131\*/  push 0                         ; lpThreadAttributes  
/\*41A163\*/  push dword ptr ss:\[ebp+408E70\] ; hProcess  
/\*41A194\*/  call dword ptr ss:\[ebp+408FBC\] ; ss:\[0041CFBC\]  kernel32.CreateRemoteThread //启动父进程  
/\*41A1C3\*/  push eax                       ;  呵呵，收工了！  
/\*41A1F4\*/  call dword ptr ss:\[ebp+403EA4\] ; ss:\[00417EA4\]  kernel32.CloseHandle  
  
/\*41C132\*/  push dword ptr ss:\[ebp+408E70\] ; target.handle  
/\*41C166\*/  call dword ptr ss:\[ebp+403EA4\]  
  
/\*41C195\*/  popad  
/\*41C196\*/  jmp short 0041C1E6 // byte byte  
  
/\*41C1C6\*/  push 0  
/\*41C1F3\*/  call dword ptr ss:\[ebp+408FC0\] ; ss:\[0041CFC0\] kernel32.ExitProcess  
  
  
// 当然不让父进程注入，你可以想像，虽然用 AdjustTokenPrivileges 获得调试权限，但一个关键的系统程序怎么可以轻易玩得起调试，我可不想机子当当响。。。  
// 应该可以拦截上面的清场，将新的起源定在： 41A251 堆栈环境要做好，就可以继续跟踪。。。现在要休息了....-\_-  
  
父进程进程片花：  
父进程 \[ 本来应该是 in winlogon.exe 的新线程 \]：  
/\*41A251\*/  jmp short 0041A279           // to :next  
/\*41A258\*/  mov eax,dword ptr ss:\[ebp+C\] // eax 当 ebp 使，取 \[ebp+C\] = 417E5C  
/\*41A286\*/  push eax  
/\*41A2B2\*/  call dword ptr ds:\[eax+20\]   ; kernel32.GetTickCount  
/\*41A2E1\*/  mov edx,eax  
/\*41A30E\*/  mov eax,dword ptr ss:\[esp\]  
/\*41A33A\*/  mov eax,dword ptr ds:\[eax\]  
/\*41A365\*/  sub edx,eax  
/\*41A392\*/  cmp edx,7D0  
/\*41A3C3\*/  ja 0041BE6C                  ; Oh, my God...  
/\*41A3F2\*/  mov eax,dword ptr ss:\[esp\]  
/\*41A41E\*/  add eax,90  
/\*41A44E\*/  push eax  
/\*41A47A\*/  sub eax,44  
/\*41A4A8\*/  push eax  
/\*41A4D7\*/  push 0  
/\*41A504\*/  push 0  
/\*41A534\*/  push 3  
/\*41A561\*/  push 0  
/\*41A58C\*/  push 0  
/\*41A5BE\*/  push 0  
/\*41A5EB\*/  push 0  
/\*41A619\*/  add eax,0C4  
/\*41A649\*/  push eax                       ; 00417F6C   ASCII "C:\\My Documents\\safeguard.exe"  
/\*41A675\*/  call dword ptr ds:\[eax-CC\]     ; ds:\[00417EA0\]=77E41BB8  kernel32.CreateProcessA  
  
/\*41A6A9\*/  mov eax,dword ptr ss:\[esp\]     ; // eax 当 ebp 使  
/\*41A6D7\*/  push 104  
/\*41A705\*/  add eax,110  
/\*41A735\*/  push eax                       ; 00417F6C   ASCII "C:\\My Documents\\safeguard.exe"  
/\*41A764\*/  push 0  
/\*41A78F\*/  call dword ptr ds:\[eax-F4\]     ; ds:\[00417E78\]=77E5A099  kernel32.GetModuleFileNameA  
/\*41A7C5\*/  mov eax,dword ptr ss:\[esp\]     ; 00417E5C  
/\*41A7F1\*/  add eax,110  
/\*41A81F\*/  mov eax,dword ptr ds:\[eax\]  
/\*41A84C\*/  cmp eax,5C3F3F5C               ; 比较文件名第 1 个 Dword 是否有: /??/  
/\*41A87C\*/  jnz 0041BE6C                   ; Game Over! 

 4dd466d0e1e0cc123afbe504da2462ca