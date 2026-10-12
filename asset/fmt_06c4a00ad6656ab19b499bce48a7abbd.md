
# 总结一

到这里，我知道了程序读取前面的 `E501` 字节内容写入到 `C:\WINDOWS\uninstall\rundl132.exe` 文件当中，并添加到注册表的启动项目中。

`HKEY_CURRENT_USER\Software\Microsoft\Windows NT\CurrentVersion\Windows load=???`

这个应该也是对应 `rundl132.exe` 的。

同时程序结束掉常见的杀毒软件，关闭了 `Kingsoft AntiVirus Service` 这个服务。

创建的线程 `00408B00`，跟到那发现有 `winmm.waveOutSetVolume` 等函数（我的 MP3 突然变成静音了，是不是这几个函数在作怪？呵呵．．）以及用 `FindWindow` 查找 `avp.exe`，用 `SendMessage` 关闭的代码。。。

继续跟着 od 走，来到这里：

```asm
00407F2F    E8 50CBFFFF     call    00404A84                     ; 打开自身文件
00407F34    8BF0            mov     esi, eax
00407F36    83FE FF         cmp     esi, -1
00407F39    0F84 E1030000   je      00408320                     ; 失败？
00407F3F    6A 00           push    0
00407F41    56              push    esi
00407F42    E8 C5C3FFFF     call    0040430C                     ; jmp 到 kernel32.GetFileSize
00407F47    3B05 A0294100   cmp     eax, dword ptr [4129A0]      ; 文件大小-E501
00407F4D    0F9FC3          setg    bl
00407F50    84DB            test    bl, bl
00407F52    0F84 C2030000   je      0040831A
00407F58    8D95 64FDFFFF   lea     edx, dword ptr [ebp-29C]
00407F5E    B8 64834000     mov     eax, 00408364                ; ASCII "??"
00407F63    E8 58CDFFFF     call    00404CC0
00407F68    8B85 64FDFFFF   mov     eax, dword ptr [ebp-29C]     ; ".EXE"
00407F6E    50              push    eax
00407F6F    8D95 60FDFFFF   lea     edx, dword ptr [ebp-2A0]
00407F75    33C0            xor     eax, eax
00407F77    E8 A0A7FFFF     call    0040271C
00407F7C    8B95 60FDFFFF   mov     edx, dword ptr [ebp-2A0]
00407F82    8D45 EC         lea     eax, dword ptr [ebp-14]
00407F85    59              pop     ecx
00407F86    E8 C9B3FFFF     call    00403354
00407F8B    8B45 EC         mov     eax, dword ptr [ebp-14]
00407F8E    E8 75B5FFFF     call    00403508
00407F93    50              push    eax                            ; "C:\Documents and  Settings\huyuytyt\",D7,"烂
                                                                  ; 孳",B2,"",A1,"",B6,"拒 MdnPacker.exe.exe"
00407F94    E8 E3C2FFFF     call    0040427C                       ; jmp 到 kernel32.DeleteFileA
//先删掉。。要不下面的函数容易出错！
00407F99    8B45 EC         mov     eax, dword ptr [ebp-14]
00407F9C    E8 D3CAFFFF     call    00404A74
00407FA1    84C0            test    al, al
00407FA3    0F85 71030000   jnz     0040831A
00407FA9    8B45 EC         mov     eax, dword ptr [ebp-14]
00407FAC    E8 23C9FFFF     call    004048D4
{
  。。。。。。
  004048D7    6A 00           push    0
  004048D9    68 80000000     push    80
  004048DE    6A 02           push    2
  004048E0    6A 00           push    0
  004048E2    6A 00           push    0
  004048E4    68 000000C0     push    C0000000
  004048E9    8BC3            mov     eax, ebx
  004048EB    E8 18ECFFFF     call    00403508
  004048F0    50              push    eax                            |FileName = "C:\Documents and  Settings\huyuytyt\",D7,"
                                                                  ; 烂孳",B2,"",A1,"",B6,"拒 MdnPacker.exe.exe"
  004048F1    E8 66F9FFFF     call    0040425C                       ; jmp 到 kernel32.CreateFileA
  004048F6    5B              pop     ebx
  004048F7    C3              retn

//创建了一个文件，只是比本身的多了一个.exe
}
00407FB1    8BF8            mov     edi, eax
00407FB3    83FF FF         cmp     edi, -1
00407FB6    0F84 5E030000   je      0040831A
00407FBC    8D45 CC         lea     eax, dword ptr [ebp-34]
00407FBF    50              push    eax
00407FC0    8D45 D4         lea     eax, dword ptr [ebp-2C]
00407FC3    50              push    eax
00407FC4    8D45 DC         lea     eax, dword ptr [ebp-24]
00407FC7    50              push    eax
00407FC8    56              push    esi
00407FC9    E8 46C3FFFF     call    00404314                       ; jmp 到 kernel32.GetFileTime
00407FCE    8D45 E4         lea     eax, dword ptr [ebp-1C]
00407FD1    BA 74834000     mov     edx, 00408374
00407FD6    E8 31B2FFFF     call    0040320C
00407FDB    8D85 58FDFFFF   lea     eax, dword ptr [ebp-2A8]
00407FE1    B9 84834000     mov     ecx, 00408384                  ; ASCII "边??"
00407FE6    8B55 E4         mov     edx, dword ptr [ebp-1C]
00407FE9    E8 66B3FFFF     call    00403354
00407FEE    8B85 58FDFFFF   mov     eax, dword ptr [ebp-2A8]
00407FF4    8D95 5CFDFFFF   lea     edx, dword ptr [ebp-2A4]
00407FFA    E8 C1CCFFFF     call    00404CC0
00407FFF    8B85 5CFDFFFF   mov     eax, dword ptr [ebp-2A4]       ; "Logo1\_.exe"
00408005    50              push    eax
00408006    8D85 54FDFFFF   lea     eax, dword ptr [ebp-2AC]
0040800C    E8 93FEFFFF     call    00407EA4                       ; 取系统目录
00408011    8B95 54FDFFFF   mov     edx, dword ptr [ebp-2AC]       ; 系统目录--EDX
00408017    8D45 F0         lea     eax, dword ptr [ebp-10]
0040801A    59              pop     ecx
0040801B    E8 34B3FFFF     call    00403354                       ; 连接
00408020    8B45 F0         mov     eax, dword ptr [ebp-10]       ; "C:\WINDOWS\Logo1\_.exe"
00408023    E8 ACC8FFFF     call    004048D4                       ；这个 call 是创建文件的一个函数，传进去的 EAX 指向文件名
00408052    50              push    eax
00408053    A1 90294100     mov     eax, dword ptr [412990]
00408058    50              push    eax                            ；EAX=E501，读取 E501 字节
00408059    56              push    esi                            ；hFile=64,可以看出是自身文件
0040805A    E8 65C3FFFF     call    004043C4                       ; jmp 到 kernel32.ReadFile
0040805F    837D F8 00      cmp     dword ptr [ebp-8], 0
00408063    74 1D           je      short 00408082
00408065    8B45 F8         mov     eax, dword ptr [ebp-8]
00408068    2945 F4         sub     dword ptr [ebp-C], eax
0040806B    8B15 90294100   mov     edx, dword ptr [412990]
00408071    8B4D F8         mov     ecx, dword ptr [ebp-8]
00408074    8B45 FC         mov     eax, dword ptr [ebp-4]
00408077    E8 7CC8FFFF     call    004048F8                       ；跟进知道是写文件了
0040807C    837D F4 00      cmp     dword ptr [ebp-C], 0
00408080  ^\\77 BD          ja      short 0040803F
00408082    8B45 FC         mov     eax, dword ptr [ebp-4]
00408085    50              push    eax
00408086    E8 B1C1FFFF     call    0040423C                       ; jmp 到 kernel32.CloseHandle
//关闭文件句柄！
0040808B    C645 CB 01      mov     byte ptr [ebp-35], 1
0040808F    EB 04           jmp     short 00408095
00408091    C645 CB 00      mov     byte ptr [ebp-35], 0
00408095    6A 00           push    0                              ；从文件开始处
00408097    6A 00           push    0
00408099    A1 A0294100     mov     eax, dword ptr [4129A0]       ；E501 到 EAX
0040809E    50              push    eax                            将文件指针移动到 E501
0040809F    56              push    esi
004080A0    E8 3FC3FFFF     call    004043E4                       ; jmp 到 kernel32.SetFilePointer
004080A5    6A 00           push    0
004080A7    8D45 F8         lea     eax, dword ptr [ebp-8]
004080AA    50              push    eax
004080AB    68 00000001     push    1000000                        ；读
004080B0    A1 90294100     mov     eax, dword ptr [412990]       ；文
004080B5    50              push    eax                            ；件
```
