
# MoleBox捆绑文件的解包方法――MoleBox Pro V2.3.3主程序脱壳+破解

下载页面：``b68K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2G2L8r3g2T1L8%4S2Q4x3X3g2U0L8$3#2Q4x3V1j5` ``.

软件大小：929 KB

软件简介：MoleBox is a runtime exe packer for Windows applications. It bundles the executable together with the DLL and data files into a single EXE file, without losing the ability to run the application. MoleBox compresses and encrypts all the application files. With MoleBox you can protect your application's data and media files from viewing and modifications, and your DLLs from usage by third party programs. Moleboxing does not affect the original application's functionality in any way nor requires any additional coding. Unpacking and decryption (if required) are performed automatically and insensibly for application. Packed program runs without extracting files to the disk.

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教

【调试环境】：WinXP、OllyDBG、PEiD、LordPE、ImportREC

【脱壳过程】：

MoleBox Pro V2.3.3的主程序`mbox2w.exe`外面是ASProtect V2.0x Registered加壳，不过里面依旧是MoleBox捆绑壳，所以脱壳时可以不管外面的ASProtect，直接去对付里面的MoleBox就行了。

```
[MoleBox V2.3X -> MoleStudio.com]
signature = E8 00 00 00 00 60 E8 4F 00 00 00
ep_only = true
```

## 一、MoleBox捆绑了哪些文件

设置OllyDBG忽略所有异常选项。用IsDebug插件去掉OllyDBG的调试器标志。

```
00401000    68 01404400     push 444001
//进入OllyDBG后暂停在这
00401005    E8 01000000     call 0040100B
0040100A    C3              retn
0040100B    C3              retn
```

下断：`BP GetFileTime`，Shift+F9，中断后取消断点Alt+F9返回

```
00432061    FF15 C4F44300   call dword ptr ds:[43F4C4]  ; kernel32.GetFileTime
00432067    C745 B8 0000000>mov dword ptr ss:[ebp-48],0
0043206E    EB 09           jmp short 00432079
00432070    8B4D B8         mov ecx,dword ptr ss:[ebp-48]
00432073    83C1 01         add ecx,1
00432076    894D B8         mov dword ptr ss:[ebp-48],ecx
00432079    8B55 B8         mov edx,dword ptr ss:[ebp-48]
0043207C    3B55 AC         cmp edx,dword ptr ss:[ebp-54]
//[ebp-54]=5   这是捆绑文件的数目
0043207F    0F83 E3000000   jnb 00432168
00432085    8B45 B8         mov eax,dword ptr ss:[ebp-48]
00432088    C1E0 04         shl eax,4
0043208B    8B4D E4         mov ecx,dword ptr ss:[ebp-1C]
0043208E    8B51 04         mov edx,dword ptr ds:[ecx+4]
00432091    8B4D E0         mov ecx,dword ptr ss:[ebp-20]
//[ebp-20]=[0012FBE0]=00BB2F00  在数据窗口中可以看见捆绑的文件名
00BB2F00  6D 62 6F 78 32 5F 62 6C 61 63 6B 6C 69 73 74 2E  mbox2_blacklist.
00BB2F10  74 78 74 00 6D 62 6F 78 32 5F 62 6F 6F 74 75 70  txt.mbox2_bootup
00BB2F20  44 62 67 4C 74 44 65 6D 6F 00 6D 62 6F 78 32 5F  DbgLtDemo.mbox2_
00BB2F30  62 6F 6F 74 75 70 4C 74 44 65 6D 6F 00 4D 53 6B  bootupLtDemo.MSk
00BB2F40  69 6E 43 6F 72 65 2E 64 6C 6C 00 6D 73 76 63 70  inCore.dll.msvcp
00BB2F50  36 30 2E 64 6C 6C                                60.dll
```

## 二、主程序`mbox2w.exe`的脱壳

Ctrl+S搜索以下命令序列：

```
mov eax,dword ptr ss:[ebp-10]
add eax,dword ptr ds:[edx+8]
mov dword ptr ss:[ebp-8],eax
```

找到在`00432B36`处，下断，Shift+F9中断后取消断点

```
00432B26    E8 65000000     call 00432B90
//解压主程序
00432B2B    E9 4DFFFFFF     jmp 00432A7D
00432B30    8B15 44F44300   mov edx,dword ptr ds:[43F444]
//[0043F444]=00BB1F20
//[00BB1F20]=0041C46E    ★ mbox2w.exe的OEP
00432B36    8B45 F0         mov eax,dword ptr ss:[ebp-10]  ; mbox2w.00400000
//中断在这里
00432B39    0342 08         add eax,dword ptr ds:[edx+8]
//[edx+8]=[00BB1F28]=00022488  ★ mbox2w.exe的Import Table RVA
00432B3C    8945 F8         mov dword ptr ss:[ebp-8],eax
00432B3F    C705 78F64300 0>mov dword ptr ds:[43F678],0
00432B49    6A 00           push 0
00432B4B    68 84EA4300     push 43EA84                    ; ASCII "EXECUTABLE"
00432B50    8B0D 74F64300   mov ecx,dword ptr ds:[43F674]  ; mbox2w.00400100
00432B56    51              push ecx
00432B57    8B55 F0         mov edx,dword ptr ss:[ebp-10]
00432B5A    52              push edx
00432B5B    8B45 F8         mov eax,dword ptr ss:[ebp-8]
00432B5E    50              push eax
00432B5F    E8 4CFCFFFF     call 004327B0
00432B64    83C4 14         add esp,14
00432B67    68 C4EA4300     push 43EAC4                    ; ASCII "imm32.dll"
00432B6C    FF15 D8F44300   call dword ptr ds:[43F4D8]     ; kernel32.GetModuleHandleA
00432B72    8945 EC         mov dword ptr ss:[ebp-14],eax
00432B75    837D EC 00      cmp dword ptr ss:[ebp-14],0
00432B79    74 0E           je short 00432B89              ; 00432B89
00432B7B    68 C4EA4300     push 43EAC4                    ; ASCII "imm32.dll"
00432B80    8B4D EC         mov ecx,dword ptr ss:[ebp-14]
00432B83    51              push ecx
00432B84    E8 F7010000     call 00432D80                  ; 00432D80
00432B89    8BE5            mov esp,ebp
00432B8B    5D              pop ebp
00432B8C    C3              retn
//这里下个断点，处理完毕后要走OEP
```

当我们中断在`00432B39`处时，`mbox2w.exe`代码已经解开，运行LordPE完全Dump这个进程。

用LordPE修正`dumped.exe`的`Import Table RVA=00022488`，会发现输入表中2个非系统文件的Dll：`MSkinCore.dll`和`MSVCP60.dll`，这2个正是捆绑的文件，下面就想办法“拿”出这2个躲藏起来的东东吧。

## 三、出来吧：`MSVCP60.dll`+`MSkinCore.dll`

`BP GetCurrentProcessId`，中断后取消断点，继续下断

`BP CreateFileA`，中断后取消断点，看堆栈

```
0012FB64   004345A6  /CALL 到 CreateFileA 来自 mbox2w.004345A0
0012FB68   00BB3A78  |FileName = "D:\DOCUME~1\FLY\LOCALS~1\TEMP\MBX@89C@BB31A0.###"
0012FB6C   40000000  |Access = GENERIC_WRITE
0012FB70   00000000  |ShareMode = 0
0012FB74   00000000  |pSecurity = NULL
0012FB78   00000002  |Mode = CREATE_ALWAYS
0012FB7C   00000000  |Attributes = 0
0012FB80   00000000  \hTemplateFile = NULL
0012FB84   00BB37E0
0012FB88   0042C83E  mbox2w.0042C83E
0012FB8C   00400000  mbox2w.00400000
0012FB90   0012FBCC
0012FB94   00000000
0012FB98   00000006
0012FB9C   0012FC04
0012FBA0   77D1A1D3  返回到 USER32.77D1A1D3 来自 USER32.77D1A270
0012FBA4   00000000
0012FBA8   00CF38EA  ASCII "MSVCP60.dll"
```

看到`0012FBA8`处的`"MSVCP60.dll"`了？对了，创建的这个文件其实就是`MSVCP60.dll`，MoleBox想捣乱一下，随意改名字，不行，在数据窗口里直接修改文件名，让MoleBox老老实实地创建`MSVCP60.dll`吧。

```
0012FB68   00BB3A78   FileName = "D:\DOCUME~1\FLY\LOCALS~1\TEMP\MSVCP60.dll"
```

```
004345A0    FF15 68F44300   call dword ptr ds:[43F468]     ; kernel32.CreateFileA
004345A6    8945 B4         mov dword ptr ss:[ebp-4C],eax
//返回这里
004345A9    837D B4 FF      cmp dword ptr ss:[ebp-4C],-1
004345AD    75 0A           jnz short 004345B9
004345AF    B9 110000EF     mov ecx,EF000011
004345B4    E8 BE0F0000     call 00435577
004345B9    6A 00           push 0
004345BB    8D85 54FFFFFF   lea eax,dword ptr ss:[ebp-AC]
004345C1    50              push eax
004345C2    FF75 D0         push dword ptr ss:[ebp-30]
004345C5    FF75 E4         push dword ptr ss:[ebp-1C]
004345C8    FF75 B4         push dword ptr ss:[ebp-4C]
004345CB    FF15 7CF54300   call dword ptr ds:[43F57C]     ; kernel32.WriteFile
004345D1    837D D8 00      cmp dword ptr ss:[ebp-28],0
004345D5    74 18           je short 004345EF
004345D7    6A 00           push 0
004345D9    8D85 54FFFFFF   lea eax,dword ptr ss:[ebp-AC]
004345DF    50              push eax
004345E0    FF75 D4         push dword ptr ss:[ebp-2C]
004345E3    FF75 D8         push dword ptr ss:[ebp-28]
004345E6    FF75 B4         push dword ptr ss:[ebp-4C]
004345E9    FF15 7CF54300   call dword ptr ds:[43F57C]     ; kernel32.WriteFile
004345EF    6A 00           push 0
004345F1    8D85 54FFFFFF   lea eax,dword ptr ss:[ebp-AC]
004345F7    50              push eax
004345F8    68 00020000     push 200
004345FD    FF75 B8         push dword ptr ss:[ebp-48]
00434600    FF75 B4         push dword ptr ss:[ebp-4C]
00434603    FF15 7CF54300   call dword ptr ds:[43F57C]     ; kernel32.WriteFile
00434609    FF75 B4         push dword ptr ss:[ebp-4C]
0043460C    FF15 98F44300   call dword ptr ds:[43F498]     ; kernel32.FlushFileBuffers
00434612    FF75 B4         push dword ptr ss:[ebp-4C]
00434615    FF15 64F44300   call dword ptr ds:[43F464]     ; kernel32.CloseHandle
0043461B    C705 8CF64300 0>mov dword ptr ds:[43F68C],1
```
