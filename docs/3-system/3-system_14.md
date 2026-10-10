

# 《可执行文件头的变形技术》电子版正式下载(增加了补充说明)

> 原文：[`www.pediy.com/kssd/pediy10/66180.html`](https://www.pediy.com/kssd/pediy10/66180.html)

去年写的书，是什么东西自己看。  
随书光盘的内容太大，以后慢慢的发。  
  
  
对【可执行文件头的变形技术】一书的补充说明  
1、使用软件的版本说明  
   1.1、IDA     ：5.0.0.879(32bit)  
   1.2、RadASM  ：2.2.0.5；  
        RadASM 中，当建立一个新的工程文件时，会覆盖同名的汇编文件，这个不能算 bug，不过对于我来讲，不太喜欢，所以将这个功能给修改了。具体修改方法如下：  

代码:

```
004259F8 68 BE 62 45 00                push    offset FName          ; lpFileName
004259FD 6A 00                         push    0                     ; 修改为   push    1
004259FF 68 20 98 44 00                push    offset szDebug        ; lpKeyName
00425A04 68 FD 93 44 00                push    offset szProject      ; lpAppName
00425A09 E8 70 E6 01 00                call    GetPrivateProfileIntA
00425A0E A3 B2 61 45 00                mov     ReleaseFlag, eax

```

代码:

```
00426456 C7 05 B2 61 45 00 00 00 00 00 mov     ReleaseFlag, 0        ; 修改为  push    1
00426460 68 C2 63 45 00                push    offset byte\_4563C2    ; lpFileName
00426465 68 80 00 00 00                push    80h                   ; nSize
0042646A 68 BE 61 45 00                push    offset dword\_4561BE   ; lpReturnedString
0042646F 68 F9 8B 44 00                push    offset Default        ; lpDefault
00426474 68 0E 98 44 00                push    offset aAssembler     ; lpKeyName
00426479 68 0E 98 44 00                push    offset aAssembler     ; lpAppName
0042647E E8 07 DC 01 00                call    GetPrivateProfileStringA

```

代码:

```
00432262 0B C0                         or      eax, eax
00432264 0F 84 11 07 00 00             jz      loc\_43297B            ; 用 nop 充填

```

代码:

```
004324B7 6A 00                         push    0                     ; dwCreationDisposition
004324B9 68 80 00 00 00                push    80h                   ; lpSecurityAttributes
004324BE 6A 02                         push    2                     ; 修改为  push  4
004324C0 6A 00                         push    0                     ; dwShareMode
004324C2 6A 01                         push    1                     ; dwDesiredAccess
004324C4 68 00 00 00 40                push    40000000h             ; 修改为  push  0C00000000h
004324C9 68 CE 66 45 00                push    offset dword\_4566CE   ; lpFileName
004324CE E8 FD 1A 01 00                call    CreateFileA

```

代码:

```
0043E23A C7 45 F8 00 00 00 00    mov \[ebp+var\_8\], 0
0043E241 C7 45 FC E9 03 00 00    mov \[ebp+var\_4\], 1001  ; 修改为 mov \[ebp+var\_4\], 1

```

修改了 masm.ini，读者打开本书汇编实例的工程文件时，需要使用这个 masm.ini。  
  
   1.3、MASM 使用了 4 个版本，这里给出的版本号是 Link;  
        V 5.12 版：ML.EXE TimeDateStamp 1999.03.30 01:45:35 ; Link.exe V 5.012.8087.0  
        V 6.00 版：ML.EXE TimeDateStamp 2000.03.16 23:20:46 ; Link.exe V 6.00.8168.0  
        V 7.10 版：ML.exe V 7.10.2179.0  ; Link.exe V 7.10.2179.0  
        V 8.00 版：ML.EXE V 8.00.50727.218  ; Link.exe V 8.00.50727.42  
   这 4 个版本编译的结果是不同的，书中如果没有特别指出，一般都是使用 V 5.12 版，附录 A2.4.3 Delay-Load Import Tables 一节的实例使用了所有的 4 个版本。  
   另外：不管那个版本的 Link 文件，都将在 DOS Header 的提示程序代码后生产一段数据，该数据的大概含义是说明该软件是使用什么版本的 MASM 编译，在什么时候连接生成的。大概面貌如下：  
代码:

```
400080                 dd  5A5E01FFh,  93060BBh,  93060BBh,  93060BBh; 0
400080                 dd   93B7F53h,  93060B8h,  93060BBh,  93060B5h; 4
400080                 dd   9237F35h,  93060C9h,  9224047h,  93060BEh; 8
400080                 dd   936667Ch,  93060BAh; 12
4000B8 Rich            db 'Rich'
4000BC                 dd 93060BBh

```

   将来也许有一天你和老盖打官司，那么这段信息就是你的罪证。  
   这段信息可以通过简单的修改 Link.exe 文件而去除，这样生产的可执行文件头将更小，更精练。对于各个版本的 Link 文件修改如下：  
  
   V5.12 Link.exe  
代码:

```
445111 8B 8D E0 01 00 00                 mov     ecx, \[ebp+1E0h\]
445117 89 44 24 10                       mov     \[esp+328h+var\_318\], eax
44511B 03 C8                             add     ecx, eax        ; 去除 PE 头中垃圾产生，nop 这两个字节
44511D 89 8D E4 01 00 00                 mov     \[ebp+1E4h\], ecx
44511D
445123
445123                   loc\_445123:
445123 FF 15 10 11 40 00                 call    ds:\_tzset
445129 A0 64 0F 48 00                    mov     al, byte\_480F64
44512E 84 C0                             test    al, al
445130 74 09                             jz      short loc\_44513B
445130
445132 C7 45 34 FF FF FF+                mov     dword ptr \[ebp+34h\], 0FFFFFFFFh
445139 EB 0D                             jmp     short loc\_445148

```

  V6.00 Link.exe  
代码:

```
445510 8B 8D E0 01 00 00                 mov     ecx, \[ebp+1E0h\]
445516 89 44 24 10                       mov     \[esp+320h+var\_310\], eax
44551A 03 C8                             add     ecx, eax        ; 去除 PE 头中垃圾产生，nop 这两个字节
44551C 89 8D E4 01 00 00                 mov     \[ebp+1E4h\], ecx
44551C
445522
445522                   loc\_445522:
445522 FF 15 14 11 40 00                 call    ds:\_tzset
445528 A0 6C 0F 48 00                    mov     al, byte\_480F6C
44552D 84 C0                             test    al, al
44552F 74 09                             jz      short loc\_44553A
44552F
445531 C7 45 34 FF FF FF+                mov     dword ptr \[ebp+34h\], 0FFFFFFFFh
445538 EB 0D                             jmp     short loc\_445547

```

   V7.10 Link.exe  
代码:

```
45E569 8B 8D 34 02 00 00                 mov     ecx, \[ebp+234h\]
45E56F 03 C8                             add     ecx, eax        ; 去除 PE 头中垃圾产生，nop 这两个字节
45E571 89 44 24 2C                       mov     \[esp+488h+var\_45C\], eax
45E575 89 8D 38 02 00 00                 mov     \[ebp+238h\], ecx
45E57B FF 15 BC 12 40 00                 call    ds:\_tzset
45E581 A0 14 B1 4A 00                    mov     al, byte\_4AB114
45E586 84 C0                             test    al, al
45E588 74 09                             jz      short loc\_45E593
45E588
45E58A C7 45 3C FF FF FF+                mov     dword ptr \[ebp+3Ch\], 0FFFFFFFFh
45E591 EB 15                             jmp     short loc\_45E5A8

```

  
  V8.00 Link.exe  
代码:

```
469A63 8D 4C 24 1C                       lea     ecx, \[esp+4D8h+var\_4BC\]
469A67 51                                push    ecx
469A68 55                                push    ebp
469A69 E8 12 A5 FF FF                    call    IMAGE::CbBuildProdidBlock(void \* \*)
469A69
469A6E 8B 95 5C 02 00 00                 mov     edx, \[ebp+25Ch\]
469A74 03 D0                             add     edx, eax        ; 去除 PE 头中垃圾产生，nop 这两个字节
469A76 89 44 24 38                       mov     \[esp+4D8h+var\_4A0\], eax
469A7A 89 95 60 02 00 00                 mov     \[ebp+260h\], edx
469A80 FF 15 40 13 40 00                 call    ds:\_\_imp\_\_\_tzset
469A86 80 3D 52 2A 4A 00+                cmp     bool fReproducible, 0
469A8D 74 05                             jz      short loc\_469A94
469A8D
469A8F 89 7D 44                          mov     \[ebp+44h\], edi
469A92 EB 15                             jmp     short loc\_469AA9

```

2、RepairPE.exe  
写这个程序原来的目的是想怎么来修改 PE 文件头的，包括删除一个节，增加一个节，将一个变形文件头修改为一个正常的文件头。后来写了这本书，就拿来作为书中使用，书写完，写程序的热情也随之完了，就成了现在这个样子了，有朋友希望我能完成这个程序，可实在提不起什么兴趣，即无经济效益，又无乐趣，何苦为之。  
这个程序使用的帮助文件为\[Visual Studio, Microsoft Portable Executable and Common Object File Format Specification\] Revision 8.0 - May 16, 2006，当时没有翻译版本，就直接使用原版的文件了，按 F1 键可获得在线帮助。  
  
3、IDA 的脚本  
RepairPE.rar 包中包含了 3 个 IDA 的脚本，因为同名的问题，脚本运行时会出现如下的提示信息：  
“can't rename byte as '××××××' because the name is already used in the program”  
不用管这些提示信息，一路下行。  
其中建立资源的由两个脚本共同完成，第一个脚本用于建立结构，脚本运行完毕后，需要手工修改两个结构，这些书中都有介绍。修改完成后运行第二个脚本，建立真正的资源信息。  
  
  
另：Masm 的 4 个版本原来是包括在随书光碟中的，因为太大，就不上传了。同样 RadASM 也不上传了。  
  
附加：PEHdDistortion.pdf

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || PEHdDistortion.rar  
 |

这个程序没有写完，也懒得继续写了，这里给出程序的功能足够这本书使用了。

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || RepairPE.rar  
 |

太大，所以慢慢发

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 章节和附录.rar \[由于附件过大，请到论坛下载：[点击此处下载打包汇总的](http://bbs.pediy.com/upload/bbs/463621/gzgzlxg.rar) \[解压密码：PEDIY\]   或 [点击此处到原帖下载](http://bbs.pediy.com/showpost.php?p=463621&postcount=3) \]   
 |
|   
 |
|  |