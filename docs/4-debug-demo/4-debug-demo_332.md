

# Audio Converter 3.0

> 原文：[`www.pediy.com/kssd/pediy03/forum516.htm`](https://www.pediy.com/kssd/pediy03/forum516.htm)

  
  
1、该软件在输入注册码时只是检查注册码的格式：注册码 26 位，其中第 6 位为"-"，其它位不检测。输入的注册码存入注册表\[HKEY\_USERS\\.DEFAULT\\Software\\Dennisre.com\\Audio Converter\\Settings\]   
"LicenseeInfo"中。   
  
2、反编译主程序，查找字符串"LicenseeInfo"的位置，我找到的是 00405EEB。   
下断点 bpx 00405EEB，并重新启动 audconv.exe，会被 trw2000 拦载。走不远，会来到：   
:0040633F 8D4C2408                lea ecx, dword ptr \[esp+08\]  ; ecx 指向假的注册码   
:00406343 8D542404                lea edx, dword ptr \[esp+04\]   
:00406347 51                      push ecx   
:00406348 52                      push edx   
:00406349 8D4C2408                lea ecx, dword ptr \[esp+08\]   
:0040634D E81EFAFFFF              call 00405D70                ; 这里是关键，打 F8 进入！   
:00406352 85C0                    test eax, eax   
:00406354 7507                    jne 0040635D   
:00406356 81C438030000            add esp, 00000338   
:0040635C C3                      ret   
  
3、进入 call 00405D70 后，走不远会来到：   
:00405E60 E87BFDFFFF              call 00405BE0                ; 计算核心，你可以看一看。   
:00405E65 3B442414                cmp eax, dword ptr \[esp+14\]  ; 比较 1   
:00405E69 754C                    jne 00405EB7   
:00405E6B 3B542418                cmp edx, dword ptr \[esp+18\]  ; 比较 2   
:00405E6F 7546                    jne 00405EB7   
:00405E71 8D442424                lea eax, dword ptr \[esp+24\]   
:00405E75 50                      push eax   
  
只要上面两处比较相等，即可注册成功。   
  
4、修改方法：用 UltraEdit 打开 audconv.exe。   
查找：3B442414754C3B5424187546   
改为：--------74----------74--   
  
即可成为任意注册版。

:0040264B 83F905                  cmp ecx, 00000005   
:0040264E 7508                    jne 00402658   
:00402650 807DD92D                cmp byte ptr \[ebp\-27\], 2D  ; 是这里  
:00402654 7411                    je 00402667   
:00402656 EB0C                    jmp 00402664   
:00402658 8A440DD4                mov al, byte ptr \[ebp+ecx\-2C\]   
:0040265C 3C30                    cmp al, 30   
:0040265E 7C04                    jl 00402664   
:00402660 3C39                    cmp al, 39   
:00402662 7E03                    jle 00402667   
:00402664 895DF8                  mov dword ptr \[ebp\-08\], ebx   
:00402667 41                      inc ecx   
:00402668 83F91A                  cmp ecx, 0000001A   
:0040266B 7CDE                    jl 0040264B

Audio Converter 的破解   
版本:3.0beta   
工具:TRW2000 和 Wdasm 8.93   
目标说明:一个可以将声音文件或 CD 音轨转换为 WAV、MP3 或 WMA 格式的工具   
下载地址:http://www.cnvnet.com/download/d/audc30b.exe   
难度：中级?   
\===========================================================================   
运行程序，输入 User Name:sUpErbOss  Company Name:Super Co. Number of Licenses:1   
Registration Key:11223-44556677889900197678，点击“完成”按钮。   
这时，程序会再注册表中生成一个名为"LicenseeInfo"的键值，里面保存有刚才输入的注册信息。   
程序启动时，会读取这个键，来判断注册是否成功!   
用 TRW2000 加载 Audio Converter，进入程序起始处后，打 BPX REGQUERYVALUEEXA，会中断好几次!   
按几次 F12 键，回到 Audio Converter 模组下，直到下面所指的重点处：   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00406336(C)   
|   
:0040633F 8D4C2408                lea ecx, dword ptr \[esp+08\]   
:00406343 8D542404                lea edx, dword ptr \[esp+04\]   
:00406347 51                      push ecx   
:00406348 52                      push edx   
:00406349 8D4C2408                lea ecx, dword ptr \[esp+08\]   
:0040634D E81EFAFFFF              call 00405D70  <----关键   
:00406352 85C0                    test eax, eax  <--eax\=1,则注册成功!   
:00406354 7507                    jne 0040635D   
:00406356 81C438030000            add esp, 00000338   
:0040635C C3                      ret   
\==============================================================   
\* Referenced by a CALL at Addresses:   
|:00401157  , :0040634D    
|   
:00405D70 83EC64                  sub esp, 00000064   
:00405D73 8B442468                mov eax, dword ptr \[esp+68\]   
:00405D77 53                      push ebx   
  
\* Reference To: KERNEL32.lstrlenA, Ord:0335h   
                                  |   
:00405D78 8B1D88D04100            mov ebx, dword ptr \[0041D088\]   
:00405D7E 55                      push ebp   
:00405D7F 56                      push esi   
:00405D80 8B742478                mov esi, dword ptr \[esp+78\]   
:00405D84 57                      push edi   
:00405D85 8BF9                    mov edi, ecx   
:00405D87 56                      push esi   
:00405D88 C70000000000            mov dword ptr \[eax\], 00000000   
  
\* Possible Reference to String Resource ID=00002: "(Unknown)"   
                                  |   
:00405D8E C744243802000000        mov \[esp+38\], 00000002   
  
\* Possible Reference to String Resource ID=00042: "Normalization"   
                                  |   
:00405D96 C744243C2A000000        mov \[esp+3C\], 0000002A   
:00405D9E C74424400A000000        mov \[esp+40\], 0000000A   
  
\* Possible Reference to String Resource ID=00014: "Output File"   
                                  |   
:00405DA6 C74424440E000000        mov \[esp+44\], 0000000E   
:00405DAE C74424483A000000        mov \[esp+48\], 0000003A   
  
\* Possible Reference to String Resource ID=00022: "Value"   
                                  |   
:00405DB6 C744244C16000000        mov \[esp+4C\], 00000016   
:00405DBE C744245032000000        mov \[esp+50\], 00000032   
  
\* Possible Reference to String Resource ID=00030: "Unable to get the audio CD."   
                                  |   
:00405DC6 C74424541E000000        mov \[esp+54\], 0000001E   
  
\* Possible Reference to String Resource ID=00034: "The output format of the selected items has no output settin"   
                                  |   
:00405DCE C744245822000000        mov \[esp+58\], 00000022   
  
\* Possible Reference to String Resource ID=00038: "Audio CD"   
                                  |   
:00405DD6 C744245C26000000        mov \[esp+5C\], 00000026   
:00405DDE C744246006000000        mov \[esp+60\], 00000006   
:00405DE6 C74424642E000000        mov \[esp+64\], 0000002E   
  
\* Possible Reference to String Resource ID=00026: "Unlicensed.   
  
For evaluation purposes only, the fully funct"   
                                  |   
:00405DEE C74424681A000000        mov \[esp+68\], 0000001A   
:00405DF6 C744246C36000000        mov \[esp+6C\], 00000036   
  
\* Possible Reference to String Resource ID=00018: "A drive to play audio cds has not been found on your system."   
                                  |   
:00405DFE C744247012000000        mov \[esp+70\], 00000012   
:00405E06 C74424743E000000        mov \[esp+74\], 0000003E   
:00405E0E FFD3                    call ebx   
:00405E10 83F805                  cmp eax, 00000005    <----比对第六位上是否为"-"   
:00405E13 0F859E000000            jne 00405EB7   
:00405E19 8D6E06                  lea ebp, dword ptr \[esi+06\]   
:00405E1C 55                      push ebp   
:00405E1D FFD3                    call ebx   
:00405E1F 83F814                  cmp eax, 00000014    <---比对"-"后的字符数是否为 20   
:00405E22 0F858F000000            jne 00405EB7   
:00405E28 56                      push esi   
:00405E29 E88CE40000              call 004142BA   
:00405E2E 55                      push ebp   
:00405E2F 8BD8                    mov ebx, eax   
:00405E31 E88FE40000              call 004142C5   
:00405E36 83C408                  add esp, 00000008   
:00405E39 8D4C2434                lea ecx, dword ptr \[esp+34\]   
:00405E3D 89542418                mov dword ptr \[esp+18\], edx   
:00405E41 8D542414                lea edx, dword ptr \[esp+14\]   
:00405E45 51                      push ecx   
:00405E46 53                      push ebx   
:00405E47 52                      push edx   
:00405E48 8BCF                    mov ecx, edi   
:00405E4A 89442420                mov dword ptr \[esp+20\], eax   
:00405E4E E87DFEFFFF              call 00405CD0    
:00405E53 83C61C                  add esi, 0000001C   
:00405E56 6814030000              push 00000314   
:00405E5B 56                      push esi   
:00405E5C 8BCF                    mov ecx, edi   
:00405E5E 8BD8                    mov ebx, eax  <----bx 的值   
:00405E60 E87BFDFFFF              call 00405BE0  <----算注册码！！关键 Call！   
:00405E65 3B442414                cmp eax, dword ptr \[esp+14\]  <---核心比对处(1)   
:00405E69 754C                    jne 00405EB7   
:00405E6B 3B542418                cmp edx, dword ptr \[esp+18\]  <---核心比对处(2)   
:00405E6F 7546                    jne 00405EB7   
:00405E71 8D442424                lea eax, dword ptr \[esp+24\]   
:00405E75 50                      push eax   
  
\* Reference To: KERNEL32.GetSystemTime, Ord:0174h   
                                  |   
:00405E76 FF1574D04100            Call dword ptr \[0041D074\]   
:00405E7C 8D4C241C                lea ecx, dword ptr \[esp+1C\]   
:00405E80 8D542424                lea edx, dword ptr \[esp+24\]   
:00405E84 51                      push ecx   
:00405E85 52                      push edx   
  
\* Reference To: KERNEL32.SystemTimeToFileTime, Ord:02C8h   
                                  |   
:00405E86 FF15A8D04100            Call dword ptr \[0041D0A8\]   
:00405E8C 8D442412                lea eax, dword ptr \[esp+12\]   
:00405E90 8D4C247C                lea ecx, dword ptr \[esp+7C\]   
:00405E94 50                      push eax   
:00405E95 8D542420                lea edx, dword ptr \[esp+20\]   
:00405E99 51                      push ecx   
:00405E9A 52                      push edx   
  
\* Reference To: KERNEL32.FileTimeToDosDateTime, Ord:0097h   
                                  |   
:00405E9B FF15B0D04100            Call dword ptr \[0041D0B0\]   
:00405EA1 6685DB                  test bx, bx  <----必须满足 bx=0   
:00405EA4 741D                    je 00405EC3   
:00405EA6 663B5C247C              cmp bx, word ptr \[esp+7C\]  <---或满足 bx≥(esp+75)所指向的值   
:00405EAB 7316                    jnb 00405EC3   
  
\==============================================================   
\* Referenced by a CALL at Address:   
|:00405E60    
|   
:00405BE0 83EC08                  sub esp, 00000008   
:00405BE3 8B4C2410                mov ecx, dword ptr \[esp+10\]   
:00405BE7 53                      push ebx   
:00405BE8 8BC1                    mov eax, ecx   
:00405BEA 55                      push ebp   
:00405BEB 56                      push esi   
:00405BEC 8B742418                mov esi, dword ptr \[esp+18\]   
:00405BF0 C1E803                  shr eax, 03   
:00405BF3 57                      push edi   
:00405BF4 33ED                    xor ebp, ebp   
:00405BF6 33D2                    xor edx, edx   
:00405BF8 8BF8                    mov edi, eax   
:00405BFA 48                      dec eax   
:00405BFB 89542414                mov dword ptr \[esp+14\], edx   
:00405BFF 85FF                    test edi, edi   
:00405C01 7414                    je 00405C17   
:00405C03 40                      inc eax   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00405C11(C)   
|   
:00405C04 8B3E                    mov edi, dword ptr \[esi\]   
:00405C06 83C608                  add esi, 00000008   
:00405C09 33EF                    xor ebp, edi    <---注意 ebp 的值---   
:00405C0B 8B7EFC                  mov edi, dword ptr \[esi\-04\]        |--->根据这两个值对注册表相应的位置的值进行更改!   
:00405C0E 33D7                    xor edx, edi    <---注意 edx 的值---   
:00405C10 48                      dec eax   
:00405C11 75F1                    jne 00405C04   
:00405C13 89542414                mov dword ptr \[esp+14\], edx   
\=============================================================   
下面附上导出的正确的 License:   
  
REGEDIT4   
  
\[HKEY\_CURRENT\_USER\\Software\\DennisRe\\Audio Converter 3.0\\Settings\]   
"LicenseeInfo"=hex:31,31,32,32,33,00,34,34,35,35,36,36,37,37,38,38,39,39,30,30,\\   
  31,39,37,36,37,38,00,00,41,75,64,69,6f,20,43,6f,6e,76,65,72,74,65,72,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,73,55,70,45,72,42,6f,\\   
  73,73,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,53,75,70,65,72,20,43,6f,2e,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\\   
  00,00,00,00,00,14,6e,5f,6b,0e,83,1e,66,01,00,00,00,00,00,00,00   
                  |  |  |  |  |  |  |  |    
                  ----------------------    
                  改动后的数据(原来均为 00)  4dd466d0e1e0cc123afbe504da2462ca