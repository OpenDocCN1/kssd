

# Syntrillum CoolEditPro2.00

> 原文：[`www.pediy.com/kssd/pediy05/pediy5013.htm`](https://www.pediy.com/kssd/pediy05/pediy5013.htm)

Syntrillum CoolEditPro2.00 21 天试用版时间限制破解   
  
简介：Syntrillum 出品的 CoolEdit 可谓是音频处理方面的精品，在国内有较多的用户，曾先后推出了 CoolEdit1.X 和 CoolEdit2000 等版本。2002 年 4 月我得到了其最新试用版本 CoolEditPro2.00，该版本有 21 天的时间限制。为了能更长一点时间来使用它，我试着来解除其时间限制。   
  
工具：Trw2000、W32dasm9.0、Keymaker1.74。   
  
分析：   
程序过期后，其主要限制有以下几方面   
（一）在启动 Logo 画面上显示 Trial version(expired);   
（二）显示 Nag 窗口，等待 10 个数字倒计时，然后按 Keep Evaluating 按钮继续;   
（三）标题栏上显示 Trial version(expired);   
（四）关于（about cool edit pro）对话框标题栏显示(expired)，其中有 This is a 21-DAY TRIAL version 等信息;   
（五）不能保存文件。   
  
（一）在启动 Logo 画面上显示 Trial version(expired)   
用 W32dasm 反汇编 coolpro2.exe 文件，在其中查找(expired)，能找到好几处地址，在 Trw200 中都设为断点，此功能限制主要代码在下面：   
:004B29A5 FF15D8D16500            Call dword ptr \[0065D1D8\]   
:004B29AB 8B442410                mov eax, dword ptr \[esp+10\]   
:004B29AF 85C0                    test eax, eax   
:004B29B1 0F8409010000            je 004B2AC0（这里如跳转，则可执行到 004B2A5B，关键点一）   
                                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^            
…………   
:004B2A56 03D1                    add edx, ecx   
:004B2A58 83FA04                  cmp edx, 00000004   
:004B2A5B 7525                    jne 004B2A82（此处跳到下面，显示 Licensed to:）   
…………   
:004B2A82 68A0E06700              push 0067E0A0   
:004B2A87 8D4C243C                lea ecx, dword ptr \[esp+3C\]   
  
\* Possible StringData Ref from Data Obj ->"Licensed to: %s"   
                                  |   
:004B2A8B 68B04B6800              push 00684BB0   
…………   
:004B2B1D 8D442438                lea eax, dword ptr \[esp+38\]   
  
\* Possible StringData Ref from Data Obj ->"Trial Version (expired)"   
                                  |   
:004B2B21 68044C6800              push 00684C04   
:004B2B26 50                      push eax   
（二）显示 Nag 窗口，等待 10 个数字倒计时，然后按 Keep Evaluating 按钮继续   
:0046E72F E8DCCEFFFF              call 0046B610   
:0046E734 668B1578E16700          mov dx, word ptr \[0067E178\]   
:0046E73B 663B1576E16700          cmp dx, word ptr \[0067E176\]   
:0046E742 0F84D4000000            je 0046E81C（此处若跳转，则可跳过 Nag 窗口，关键点二）   
                                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   
:0046E748 A1C4466A00              mov eax, dword ptr \[006A46C4\]   
:0046E74D 85C0                    test eax, eax   
:0046E74F 0F84C7000000            je 0046E81C   
\* Possible StringData Ref from Data Obj ->"Cool Waveform Editor - UNREGISTERED"   
                                  |   
:0046E755 6894C36700              push 0067C394   
:0046E75A A1D8476A00              mov eax, dword ptr \[006A47D8\]   
:0046E75F 50                      push eax   
  
\* Reference To: USER32.SetWindowTextA, Ord:025Eh   
                                  |   
:0046E760 FF15A4D46500            Call dword ptr \[0065D4A4\]   
  
\* Possible StringData Ref from Data Obj ->"Getting User Info..."   
                                  |   
:0046E766 68B8C36700              push 0067C3B8   
:0046E76B 8B0D20506A00            mov ecx, dword ptr \[006A5020\]   
:0046E771 E83A450400              call 004B2CB0   
:0046E776 6A00                    push 00000000   
:0046E778 68E08C5A00              push 005A8CE0   
:0046E77D 8B0D84456A00            mov ecx, dword ptr \[006A4584\]   
:0046E783 51                      push ecx   
  
\* Possible StringData Ref from Data Obj ->"ID\_UNREGISTERED2"   
                                  |   
:0046E784 68D0C36700              push 0067C3D0   
:0046E789 8B156C436A00            mov edx, dword ptr \[006A436C\]   
:0046E78F 52                      push edx   
:0046E790 E8DB6B1600              call 005D5370（这里出现 Nag 窗口）   
（三）标题栏上显示 Trial version(expired)   
:00469EFB 8B35D4D16500            mov esi, dword ptr \[0065D1D4\]   
:00469F01 55                      push ebp   
:00469F02 FFD6                    call esi   
:00469F04 85C0                    test eax, eax   
:00469F06 0F8564010000            jne 0046A070   
:00469F0C 66A178E16700            mov ax, word ptr \[0067E178\]   
:00469F12 663B0576E16700          cmp ax, word ptr \[0067E176\]   
:00469F19 7539                    jne 00469F54（此处若不跳，则标题栏显示 Licensed to，关键点三）   
                                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   
:00469F1B A0A0E06700              mov al, byte ptr \[0067E0A0\]   
:00469F20 84C0                    test al, al（这里用 AL 做标志判断注册表中是否有注册用户名，如有则 AL 的值为 1，否则为零，所以这里可改成 cmp al,al，然后下面一行改成 je 00469F2D，这样永远都会跳转）   
:00469F22 7509                    jne 00469F2D（此处若不跳，则启动后 open 等命令为灰色，关键点四）   
                                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   
:00469F24 66C705F40D6B000000      mov word ptr \[006B0DF4\], 0000   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00469F22(C)   
|   
  
\* Possible StringData Ref from Data Obj ->"ssssssssssssssssssssssssssssssssssssssssssssss"   
                                        ->"ssssssssssssssssssssssssssssssssssssssssssssss"   
                                        ->"ss"   
                                  |   
:00469F2D 68A0E06700              push 0067E0A0   
:00469F32 6890456A00              push 006A4590   
  
\* Possible StringData Ref from Data Obj ->"Cool Edit"   
                                  |   
:00469F37 687CAE6700              push 0067AE7C   
:00469F3C 8D4C2428                lea ecx, dword ptr \[esp+28\]   
  
\* Possible StringData Ref from Data Obj ->"%s%s - Licensed to %s"   
                                  |   
:00469F40 6850B56700              push 0067B550   
:00469F45 51                      push ecx   
  
\* Reference To: USER32.wsprintfA, Ord:02ACh   
                                  |   
:00469F46 FF15A4D56500            Call dword ptr \[0065D5A4\]   
:00469F4C 83C414                  add esp, 00000014   
:00469F4F E995000000              jmp 00469FE9   
（四）关于（about cool edit pro）对话框标题栏显示(expired)，其中有 This is a 21-DAY TRIAL version 等信息;   
:004B3982 663B0576E16700          cmp ax, word ptr \[0067E176\]   
:004B3989 56                      push esi   
:004B398A 57                      push edi   
:004B398B 894C2410                mov dword ptr \[esp+10\], ecx   
:004B398F 88542414                mov byte ptr \[esp+14\], dl   
:004B3993 C644241500              mov \[esp+15\], 00   
:004B3998 0F8502020000            jne 004B3BA0（此处若跳则显示 This is a 21-DAY TRIAL version 等信                                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   
息;关键点五）   
^^^^^^^^^^^^   
…………   
\* Possible StringData Ref from Data Obj ->"%s version %s%s"   
                                  |   
:004B3C35 68044D6800              push 00684D04   
:004B3C3A 52                      push edx   
:004B3C3B FFD5                    call ebp   
:004B3C3D 66A178E16700            mov ax, word ptr \[0067E178\]   
:004B3C43 83C414                  add esp, 00000014   
:004B3C46 663B0576E16700          cmp ax, word ptr \[0067E176\]   
:004B3C4D 746B                    je 004B3CBA（此处若不跳则标题栏显示 expired，关键点六）   
                                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   
:004B3C4F A1C42E6B00              mov eax, dword ptr \[006B2EC4\]   
:004B3C54 83F801                  cmp eax, 00000001   
:004B3C57 7520                    jne 004B3C79   
（五）不能保存文件   
文件不能保存的限制有好几处，其中 Save as、Save copy as、Save selection 是同样的地址，Save all 是一处地址，而 Save 命令一直呈现灰色不可用状态，所以未找到其限制地址，但无论怎样保存功能已经有了，用其他几个代替吧，结果是一样的!   
  
1.Save as、Save copy as、Save selection 的限制地址：   
:0048E571 663BC7                  cmp ax, di   
:0048E574 740D                    je 0048E583（这里跳可以绕过一个提示不能保存的 Call，关键点七）   
                                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   
:0048E576 A1C42E6B00              mov eax, dword ptr \[006B2EC4\]   
:0048E57B 85C0                    test eax, eax   
:0048E57D 0F84FDDDFFFF            je 0048C380（选则里也可以）   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0048E574(C)   
|   
:0048E583 8B0D48456A00            mov ecx, dword ptr \[006A4548\]   
:0048E589 85C9                    test ecx, ecx   
:0048E58B 0F84AD3C0000            je 0049223E   
:0048E591 E82A5B0500              call 004E40C0   
:0048E596 8BF0                    mov esi, eax   
:0048E598 85F6                    test esi, esi   
:0048E59A 0F849E3C0000            je 0049223E   
:0048E5A0 6681FB7702              cmp bx, 0277   
:0048E5A5 741B                    je 0048E5C2   
:0048E5A7 8A4634                  mov al, byte ptr \[esi+34\]   
:0048E5AA 84C0                    test al, al   
:0048E5AC 7414                    je 0048E5C2   
:0048E5AE 837E1CFF                cmp dword ptr \[esi+1C\], FFFFFFFF   
:0048E5B2 740E                    je 0048E5C2   
:0048E5B4 39BEE0010000            cmp dword ptr \[esi+000001E0\], edi   
:0048E5BA 7406                    je 0048E5C2   
:0048E5BC 893D284D6B00            mov dword ptr \[006B4D28\], edi   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:0048E5A5(C), :0048E5AC(C), :0048E5B2(C), :0048E5BA(C)   
|   
:0048E5C2 A10C4B6A00              mov eax, dword ptr \[006A4B0C\]   
:0048E5C7 6681EB7702              sub bx, 0277   
:0048E5CC 66F7DB                  neg bx   
:0048E5CF 1BDB                    sbb ebx, ebx   
:0048E5D1 6A00                    push 00000000   
:0048E5D3 83E3FE                  and ebx, FFFFFFFE   
:0048E5D6 6A00                    push 00000000   
:0048E5D8 83C303                  add ebx, 00000003   
:0048E5DB 50                      push eax   
:0048E5DC 53                      push ebx   
:0048E5DD 57                      push edi   
:0048E5DE 56                      push esi   
:0048E5DF E8CC05FFFF              call 0047EBB0（这里出现不能保存提示）   
………   
:0048C380 6800000400              push 00040000   
  
\* Possible Reference to String Resource ID=00566: "Demo Version"   
                                  |   
:0048C385 6836020000              push 00000236   
:0048C38A E861EC1400              call 005DAFF0   
:0048C38F 83C404                  add esp, 00000004   
:0048C392 50                      push eax   
  
\* Possible Reference to String Resource ID=00565: "Sorry, the time limit for saving files has expired in this d"   
                                  |   
:0048C393 6835020000              push 00000235   
:0048C398 E853EC1400              call 005DAFF0   
:0048C39D 8B5508                  mov edx, dword ptr \[ebp+08\]   
:0048C3A0 83C404                  add esp, 00000004   
:0048C3A3 50                      push eax   
:0048C3A4 52                      push edx   
:0048C3A5 E8F68A1400              call 005D4EA0（这里也出现不能保存提示）   
:0048C3AA 83C410                  add esp, 00000010   
:0048C3AD 33C0                    xor eax, eax   
:0048C3AF 5F                      pop edi   
:0048C3B0 5E                      pop esi   
:0048C3B1 5B                      pop ebx   
:0048C3B2 8BE5                    mov esp, ebp   
:0048C3B4 5D                      pop ebp   
:0048C3B5 C3                      ret   
  
在关键点五处改变跳转，在经过:0048E5DF E8CC05FFFF              call 0047EBB0 时仍然会有不能保存的提示，F8 进入这个 Call 看看：   
:0047EBEF E8CC951200              call 005A81C0   
:0047EBF4 83C408                  add esp, 00000008   
:0047EBF7 663D0100                cmp ax, 0001   
:0047EBFB 7447                    je 0047EC44（这里跳可以下面提示不能保存的 Call，关键点八）   
                                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   
:0047EBFD 3935C42E6B00            cmp dword ptr \[006B2EC4\], esi   
:0047EC03 753F                    jne 0047EC44   
:0047EC05 6800000400              push 00040000   
  
\* Possible Reference to String Resource ID=00566: "Demo Version"   
                                  |   
:0047EC0A 6836020000              push 00000236   
:0047EC0F E8DCC31500              call 005DAFF0   
:0047EC14 83C404                  add esp, 00000004   
:0047EC17 50                      push eax   
  
\* Possible Reference to String Resource ID=00565: "Sorry, the time limit for saving files has expired in this d"   
                                  |   
:0047EC18 6835020000              push 00000235   
:0047EC1D E8CEC31500              call 005DAFF0   
:0047EC22 8B0DC8466A00            mov ecx, dword ptr \[006A46C8\]   
:0047EC28 83C404                  add esp, 00000004   
:0047EC2B 50                      push eax   
:0047EC2C 51                      push ecx   
:0047EC2D E86E621500              call 005D4EA0（这里出现不能保存提示）   
  
2.Save all 的限制地址:   
:00489D4C E86FE41100              call 005A81C0   
:00489D51 83C408                  add esp, 00000008   
:00489D54 663D0100                cmp ax, 0001   
:00489D58 743F                    je 00489D99（这里跳可以下面提示不能保存的 Call，关键点九）   
                                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   
:00489D5A A1C42E6B00              mov eax, dword ptr \[006B2EC4\]   
:00489D5F 85C0                    test eax, eax   
:00489D61 7536                    jne 00489D99   
:00489D63 6800000400              push 00040000   
  
\* Possible Reference to String Resource ID=00566: "Demo Version"   
                                  |   
:00489D68 6836020000              push 00000236   
:00489D6D E87E121500              call 005DAFF0   
:00489D72 83C404                  add esp, 00000004   
:00489D75 50                      push eax   
  
\* Possible Reference to String Resource ID=00565: "Sorry, the time limit for saving files has expired in this d"   
                                  |   
:00489D76 6835020000              push 00000235   
:00489D7B E870121500              call 005DAFF0   
:00489D80 83C404                  add esp, 00000004   
:00489D83 50                      push eax   
:00489D84 8B4508                  mov eax, dword ptr \[ebp+08\]   
:00489D87 50                      push eax   
:00489D88 E813B11400              call 005D4EA0（这里出现不能保存提示）   
:00489D8D 83C410                  add esp, 00000010   
:00489D90 33C0                    xor eax, eax   
:00489D92 5F                      pop edi   
:00489D93 5E                      pop esi   
:00489D94 5B                      pop ebx   
:00489D95 8BE5                    mov esp, ebp   
:00489D97 5D                      pop ebp   
:00489D98 C3                      ret   
…………   
关键点七改变跳转后跳到:   
:00489D99 8D8C2494070000          lea ecx, dword ptr \[esp+00000794\]   
一直执行到下面的 Call 又出现不能保存的提示：   
:00489E35 FF5064                  call \[eax+64\]   
F8 进入这个 Call，F10 执行到下面就看出了门道：   
:004EE07B E8300BF9FF              call 0047EBB0   
这里和:0048E5DF E8CC05FFFF              call 0047EBB0 是同样的调用，再进入这个 Call，又有下面的调用   
:0047EBFB 7447                    je 0047EC44（这里跳可以下面提示不能保存的 Call，关键点八）   
                                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   
coolpro2.exe 文件有自校验功能，如果直接修改程序代码会导致不能运行，在这里我没有再破解该限制，而是用 Keymaker 直接制作它的内存补丁，能省就省吧!   
  
运行 Keymaker1.73，选择“其他—制作内存补丁”。   
  
程序名称：coolpro2.exe   
“消息标题”和“启动提示”填入自己喜欢的信息。   
依次将上面的几处关键地址添加上。   
  
修改地址：004B29B1   
修改长度：6   
原始指令：0F8409010000   
修改指令：0F8509010000   
  
修改地址：0046E742   
修改长度：6   
原始指令：0F84D4000000   
修改指令：0F85D4000000   
  
修改地址：00469F19   
修改长度：2   
原始指令：7539   
修改指令：7439   
  
修改地址：00469F20   
修改长度：2   
原始指令：84c0   
修改指令：84DB   
  
修改地址：00469F22   
修改长度：2   
原始指令：7509   
修改指令：7409   
  
修改地址：004B3998   
修改长度：6   
原始指令：0F8502020000   
修改指令：0F8402020000   
  
修改地址：004B3C4D   
修改长度：2   
原始指令：746B   
修改指令：756B   
  
修改地址：0048E574   
修改长度：2   
原始指令：740D   
修改指令：750D   
  
修改地址：0047EBFB   
修改长度：2   
原始指令：7447   
修改指令：7547   
  
修改地址：00489D58   
修改长度：2   
原始指令：743F   
修改指令：753F   
  
经过上面的修改后，所有显示 Licensed to 的后面都是空，要想让程序显示你希望的用户名，只要在注册表中 HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Syntrillium\\CEPro2\\Registration 下建立 Name 字符串并将其值设为你希望的内容即可。   
  
最近接到虎翼网的通知，我的免费主页只能使用到 6 月 5 日（免费的东西真是越来越少了），目前还没找到更好的主页空间，有想和我交流的朋友暂时可以发邮件到 e-soft2001@sohu.com 或用我的免费域名 esoft.7i24.com（我会尽快找一个新的空?  4dd466d0e1e0cc123afbe504da2462ca