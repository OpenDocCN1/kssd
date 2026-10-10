

# TUNet

> 原文：[`www.pediy.com/kssd/pediy06/pediy6620.htm`](https://www.pediy.com/kssd/pediy06/pediy6620.htm)

TUNet 是我学校的一个客户端软件，里面包括两个协议端口认证和网关的认证协议，至于协议部分我就不多说，很多同学反映没有开机自动启动的功能。我今天想说一下我打的一个开机补丁。也是对 MFC 的一次实践吧。  
现在该文件夹下加一个配置文件  
peansen.ini  
内容如下：  
\[patch\]  
AUTORUN=1  
检查输入表知道我们还需要一个 GetPrivateProfileIntA 函数的输入项，于是添加之（具体做法很多，可以用工具）  
首先我想把整个程序的状况说一下：  
  
\->Section Header Table  
   1. item:  
    Name:                  .text  
    VirtualSize:           0x0002E506  
    VirtualAddress:        0x00001000  
    SizeOfRawData:         0x0002F000  
    PointerToRawData:      0x00001000  
    PointerToRelocations:  0x00000000  
    PointerToLinenumbers:  0x00000000  
    NumberOfRelocations:   0x0000  
    NumberOfLinenumbers:   0x0000  
    Characteristics:       0x60000020  
    (CODE, EXECUTE, READ)  
  
   2. item:  
    Name:                  .rdata  
    VirtualSize:           0x0000AA40  
    VirtualAddress:        0x00030000  
    SizeOfRawData:         0x0000B000  
    PointerToRawData:      0x00030000  
    PointerToRelocations:  0x00000000  
    PointerToLinenumbers:  0x00000000  
    NumberOfRelocations:   0x0000  
    NumberOfLinenumbers:   0x0000  
    Characteristics:       0xC0000040  
    (INITIALIZED\_DATA, READ, WRITE)  
  
   3. item:  
    Name:                  .data  
    VirtualSize:           0x0000ED04  
    VirtualAddress:        0x0003B000  
    SizeOfRawData:         0x00007000  
    PointerToRawData:      0x0003B000  
    PointerToRelocations:  0x00000000  
    PointerToLinenumbers:  0x00000000  
    NumberOfRelocations:   0x0000  
    NumberOfLinenumbers:   0x0000  
    Characteristics:       0xC0000040  
    (INITIALIZED\_DATA, READ, WRITE)  
  
   4. item:  
    Name:                  .rsrc  
    VirtualSize:           0x00066C40  
    VirtualAddress:        0x0004A000  
    SizeOfRawData:         0x00067000  
    PointerToRawData:      0x00042000  
    PointerToRelocations:  0x00000000  
    PointerToLinenumbers:  0x00000000  
    NumberOfRelocations:   0x0000  
    NumberOfLinenumbers:   0x0000  
    Characteristics:       0x40000040  
    (INITIALIZED\_DATA, READ)  
  
   5. item:  
    Name:                  .gda  
    VirtualSize:           0x00001000  
    VirtualAddress:        0x000B1000  
    SizeOfRawData:         0x00001000  
    PointerToRawData:      0x000A9000  
    PointerToRelocations:  0xFFF9F2FD  
    PointerToLinenumbers:  0x00000000  
    NumberOfRelocations:   0x0000  
    NumberOfLinenumbers:   0x0000  
    Characteristics:       0xE0000020  
    (CODE, EXECUTE, READ, WRITE)  
先给她加上一个复选框，id=0x5210  
然后找到她的消息处理循环，因为那个对话框中的很多 edit 需要初始化，我于是就借助其中一个在初始化的时候同时也处理我加上去的复选框的初始化过程。根据 section 的结构，我们在程序里找到很多空白。0x2f510~0x30000 都是空白，我们把一些只读字符串放在这边。从 0x2f900 开始放如下字符串  
2E5C5C5045414E53454E2E696E690000706174636800000000000000000000004155544F52554E000000000000000000534F4654574152455C5C4D6963726F736F66745C5C57696E646F77735C5C43757272656E7456657273696F6E5C5C52756E0000000000000054554E455400000000310030000000000000000000000000（这些是 16 进制值）  
分别是：.\\\\PEANSEN.ini  
patch  
SOFTWARE\\\\Microsoft\\\\Windows\\\\CurrentVersion\\\\Run  
TUNET  
1  0，和上面的 16 进制值对号入座就好了  
找到了我们要打补丁的地方：0x425cd（不管是初始化还是消息处理都好经过它）代码如下：  
B8 5C F0 42 00                mov eax, offset loc\_42F05C  
.text:00425BF9 E8 3A 2F FF FF                call sub\_418B38  
.text:00425BFE 83 EC 54                      sub esp, 54h  
.text:00425C01 83 65 F0 00                   and dword ptr \[ebp\-10h\], 0  
.text:00425C05 53                            push ebx  
.text:00425C06 8B 5D 08                      mov ebx, \[ebp+8\]  
.text:00425C09 56                            push esi  
.text:00425C0A 57                            push edi  
.text:00425C0B 81 FB 11 01 00 00             cmp ebx, 111h  
.text:00425C11 8B F9                         mov edi, ecx  
.text:00425C13 75 18                         jnz short loc\_425C2D  
.text:00425C15 FF 75 10                      push dword ptr \[ebp+10h\]控件 handle-------从这边开始折向补丁处  
.text:00425C18 8B 07                         mov eax, \[edi\]  
.text:00425C1A FF 75 0C                      push dword ptr \[ebp+0Ch\]控件 id  
.text:00425C1D FF 50 78                      call dword ptr \[eax+78h\] ; 这是 cmd 命令的开始  
.text:00425C20 85 C0                         test eax, eax  
.text:00425C22 0F 84 55 01 00 00             jz loc\_425D7D  
.text:00425C28 E9 1D 04 00 00                jmp loc\_42604A  
打补丁后的  
00425C13    . /75 18                     jnz short TUNet306.00425C2D  
00425C15    . |E9 06990000               jmp TUNet306.0042F520----跳到我们的处理空间  
00425C1A    > |FF75 0C                   push dword ptr ss:\[ebp+C\]  
00425C1D    . |FF50 78                   call dword ptr ds:\[eax+78\]  
00425C20    . |85C0                      test eax,eax  
补丁的主要代码：  
每一段返回都要经过这边，恢复堆栈，加上原来上面用跳转指令被覆盖的代码  
0042F510    > /61                        popad  
0042F511    . |FF7424 10                 push dword ptr ss:\[esp+10\]  
0042F515    . |8B07                      mov eax,dword ptr ds:\[edi\]  
0042F517    .^|E9 FE66FFFF               jmp TUNet306.00425C1A  
0042F51C      |00                        db 00  
0042F51D      |00                        db 00  
0042F51E      |00                        db 00  
0042F51F      |00                        db 00  
我们是从上面跳到这边的：  
0042F520    > |60                        pushad  
0042F521    . |8B45 0C                   mov eax,dword ptr ss:\[ebp+C\]  
0042F524    . |3D 16520004               cmp eax,4005216---抓住 id=0x5216 的初始化过程来初始化我们自己的复选框  
0042F529    . |74 08                     je short TUNet306.0042F533  
0042F52B    . |66:3D 1052                cmp ax,5210----自己的消息处理过程  
0042F52F    . |74 4F                     je short TUNet306.0042F580  
0042F531    .^ EB DD                     jmp short TUNet306.0042F510  
复选框的初始化过程  
0042F533    > |FF75 10                   push dword ptr ss:\[ebp+10\]                            ; /hWnd  
0042F536    . |FF15 E8034300             call dword ptr ds:\[<&USER32.GetParent>\]               ; \\GetParent  
因为我们知道我们所抓住的 id=0x5216 的控件的句柄，通过它来获得 id=0x5210 的 handle（用 getparent 函数和 GetDlgItem 函数）  
0042F53C    . |68 10520000               push 5210                                             ; /ControlID = 5210 (21008.)  
0042F541    . |50                        push eax                                              ; |hWnd  
0042F542    . |FF15 74034300             call dword ptr ds:\[<&USER32.GetDlgItem>\]              ; \\GetDlgItem  
0042F548    . |50                        push eax  
0042F549    . |68 00F94200               push TUNet306.0042F900                                ; /IniFileName = ".\\\\PEANSEN.ini"  
0042F54E    . |6A 00                     push 0                                                ; |Default = 0  
0042F550    . |68 20F94200               push TUNet306.0042F920                                ; |Key = "AUTORUN"  
0042F555    . |68 10F94200               push TUNet306.0042F910                                ; |Section = "patch"  
0042F55A    . |FF15 25204B00             call dword ptr ds:\[<&KERNEL32.GetPrivateProfileIntA>\] ; \\GetPrivateProfileIntA  
取得是否我们在配置文件中的 AUTORUN 的值，就是看我们的复选框开始时要不要打勾  
0042F560    . |5B                        pop ebx  
0042F561    . |6A 00                     push 0                                                ; /lParam = 0  
0042F563    . |50                        push eax                                              ; |wParam  
0042F564    . |68 F1000000               push 0F1                                              ; |Message = BM\_SETCHECK  
0042F569    . |53                        push ebx                                              ; |hWnd  
0042F56A    . |FF15 28044300             call dword ptr ds:\[<&USER32.SendMessageA>\]            ; \\SendMessageA  
  
0042F570    .^ EB 9E                     jmp short TUNet306.0042F510  
0042F572      |00                        db 00  
0042F573      |00                        db 00  
0042F574      |00                        db 00  
0042F575      |00                        db 00  
0042F576      |00                        db 00  
0042F577      |00                        db 00  
0042F578      |00                        db 00  
0042F579      |00                        db 00  
0042F57A      |00                        db 00  
0042F57B      |00                        db 00  
0042F57C      |00                        db 00  
0042F57D      |00                        db 00  
0042F57E      |00                        db 00  
0042F57F      |00                        db 00  
0042F580    > |83EC 50                   sub esp,50  
0042F583    . |68 00F94200               push TUNet306.0042F900                                ; /IniFileName = ".\\\\PEANSEN.ini"  
0042F588    . |6A 00                     push 0                                                ; |Default = 0  
0042F58A    . |68 20F94200               push TUNet306.0042F920                                ; |Key = "AUTORUN"  
0042F58F    . |68 10F94200               push TUNet306.0042F910                                ; |Section = "patch"  
0042F594    . |FF15 25204B00             call dword ptr ds:\[<&KERNEL32.GetPrivateProfileIntA>\] ; \\GetPrivateProfileIntA  
取得 AUTORUN 的值开看看要不要在注册表的 run 项里添加  
在这边为了简化处理当要开机自启动时我们在 run 下的 TUNET 键值添加文件名  
如果不要开机自启动时我们添加 0 字符串  
0042F59A    . |50                        push eax  
0042F59B    . |8D4424 04                 lea eax,dword ptr ss:\[esp+4\]  
0042F59F    . |50                        push eax                                              ; /pHandle  
0042F5A0    . |68 3F000F00               push 0F003F                                           ; |Access = KEY\_ALL\_ACCESS  
0042F5A5    . |6A 00                     push 0                                                ; |Reserved = 0  
0042F5A7    . |68 30F94200               push TUNet306.0042F930                                ; |Subkey = "SOFTWARE\\\\Microsoft\\\\Windows\\\\CurrentVersion\\\\Run"  
0042F5AC    . |68 02000080               push 80000002                                         ; |hKey = HKEY\_LOCAL\_MACHINE  
0042F5B1    . |FF15 08004300             call dword ptr ds:\[<&ADVAPI32.RegOpenKeyExA>\]         ; \\RegOpenKeyExA  
0042F5B7    . |83F8 00                   cmp eax,0  
0042F5BA    . |0F85 94000000             jnz TUNet306.0042F654  
0042F5C0    . |8D4424 08                 lea eax,dword ptr ss:\[esp+8\]  
0042F5C4    . |6A 40                     push 40                                               ; /BufSize = 40 (64.)  
0042F5C6    . |50                        push eax                                              ; |PathBuffer  
0042F5C7    . |6A 00                     push 0                                                ; |hModule = NULL  
0042F5C9    . |FF15 5C024300             call dword ptr ds:\[<&KERNEL32.GetModuleFileNameA>\]    ; \\GetModuleFileNameA  
0042F5CF    . |58                        pop eax  
0042F5D0    . |83F8 00                   cmp eax,0  
0042F5D3    . |74 3E                     je short TUNet306.0042F613  
0042F5D5    . |8D0424                    lea eax,dword ptr ss:\[esp\]  
0042F5D8    . |90                        nop  
0042F5D9    . |6A 40                     push 40                                               ; /BufSize = 40 (64.)  
0042F5DB    . |68 63F94200               push TUNet306.0042F963                                ; |Buffer = TUNet306.0042F963  
0042F5E0    . |6A 01                     push 1                                                ; |ValueType = REG\_SZ  
0042F5E2    . |6A 00                     push 0                                                ; |Reserved = 0  
0042F5E4    . |68 68F94200               push TUNet306.0042F968                                ; |ValueName = "TUNET"  
0042F5E9    . |FF30                      push dword ptr ds:\[eax\]                               ; |hKey  
0042F5EB    . |FF15 00004300             call dword ptr ds:\[<&ADVAPI32.RegSetValueExA>\]        ; \\RegSetValueExA  
0042F5F1    . |68 00F94200               push TUNet306.0042F900                                ; /FileName = ".\\\\PEANSEN.ini"  
0042F5F6    . |68 73F94200               push TUNet306.0042F973                                ; |String = "0"  
0042F5FB    . |68 20F94200               push TUNet306.0042F920                                ; |Key = "AUTORUN"  
0042F600    . |68 10F94200               push TUNet306.0042F910                                ; |Section = "patch"  
0042F605    . |FF15 64014300             call dword ptr ds:\[<&KERNEL32.WritePrivateProfileStri>; \\WritePrivateProfileStringA  
再把启动结果写会配置文件中  
0042F60B    . |90                        nop  
0042F60C    . |90                        nop  
0042F60D    . |90                        nop  
0042F60E    . |EB 3A                     jmp short TUNet306.0042F64A  
0042F610      |90                        nop  
0042F611      |90                        nop  
0042F612      |90                        nop  
0042F613    > |8D0424                    lea eax,dword ptr ss:\[esp\]  
0042F616    . |90                        nop  
0042F617    . |8D5C24 04                 lea ebx,dword ptr ss:\[esp+4\]  
0042F61B    . |6A 50                     push 50                                               ; /BufSize = 50 (80.)  
0042F61D    . |53                        push ebx                                              ; |Buffer  
0042F61E    . |6A 01                     push 1                                                ; |ValueType = REG\_SZ  
0042F620    . |6A 00                     push 0                                                ; |Reserved = 0  
0042F622    . |68 68F94200               push TUNet306.0042F968                                ; |ValueName = "TUNET"  
0042F627    . |FF30                      push dword ptr ds:\[eax\]                               ; |hKey  
0042F629    . |FF15 00004300             call dword ptr ds:\[<&ADVAPI32.RegSetValueExA>\]        ; \\RegSetValueExA  
0042F62F    . |90                        nop  
0042F630    . |68 00F94200               push TUNet306.0042F900                                ; /FileName = ".\\\\PEANSEN.ini"  
0042F635    . |68 71F94200               push TUNet306.0042F971                                ; |String = "1"  
0042F63A    . |68 20F94200               push TUNet306.0042F920                                ; |Key = "AUTORUN"  
0042F63F    . |68 10F94200               push TUNet306.0042F910                                ; |Section = "patch"  
0042F644    . |FF15 64014300             call dword ptr ds:\[<&KERNEL32.WritePrivateProfileStri>; \\WritePrivateProfileStringA  
0042F64A    > |8B0424                    mov eax,dword ptr ss:\[esp\]  
0042F64D    . |83C4 50                   add esp,50  
0042F650    . |EB 0E                     jmp short TUNet306.0042F660  
0042F652      |90                        nop  
0042F653      |90                        nop  
0042F654    > |83C4 54                   add esp,54  
0042F657    .^ E9 B4FEFFFF               jmp TUNet306.0042F510  
0042F65C      |00                        db 00  
0042F65D      |00                        db 00  
0042F65E      |00                        db 00  
0042F65F      |00                        db 00  
0042F660    > |50                        push eax                                              ; /hKey  
0042F661    . |FF15 10004300             call dword ptr ds:\[<&ADVAPI32.RegCloseKey>\]           ; \\RegCloseKey  
0042F667    .^\\E9 A4FEFFFF               jmp TUNet306.0042F510  
0042F66C       00                        db 00  
0042F66D       00                        db 00  
0042F66E       00                        db 00  
  
好了我想 odbg 也注释的很清楚了。  
  
  
在看雪中也混了一年了，也没有发过什么帖子，今天就发一个吧  
由于我已经大四，考研在即（唉！当初不努力呀）没有更多的时间来学习这里的技术，这篇文章就算是我的一个短暂告别吧。我会回来的，不管我考研的结果怎么样。  
呵呵，谢谢这里的各位大虾，小虾。

 4dd466d0e1e0cc123afbe504da2462ca