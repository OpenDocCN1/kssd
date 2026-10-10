

# Worm.Parite.Residented 详细分析+专杀工具 src

> 原文：[`www.pediy.com/kssd/pediy11/117976.html`](https://www.pediy.com/kssd/pediy11/117976.html)

【文章标题】: Worm.Parite.Residented 详细分析+专杀工具 src  
【文章作者】: Azure\[LCG\]  
【作者邮箱】: Azure@52pojie.cn  
【作者 QQ 号】: 325002492  
【加壳方式】: UPX  
【编写语言】: BC++  
【使用工具】: IDA、OD  
【操作平台】: Win7+WinXP Sp3(VM)  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  完蛋了……看来我的计算机水平还是菜鸟等级的，装着自己很懂似的，不装杀软，裸奔，导致今天在干伤天害理的事情的时候被病毒感染，全盘 exe 文件除了最核心的系统文件无一幸免。  
  这个专杀只是把文件的病毒区段数据填 0，然后修改入口点，但是文件大小还是没恢复回去……因为感染数量相当巨大……这只是 D 盘的= =郁闷啊……  
  留下一个图片  
  ![](img/3b44fe873edabf7ab4297839cf3ee9ff.jpg)  
  我欠回炉重造了……唉。。。  
    
  下次再也不敢帮别人盗版了……= =  
  ------------------------------------------------------------------------------------------------------------------  
  以上为去年 10 月 1 感染病毒当天的吐嘈  
    
  彻底败掉了啊……败给了一个 2001 年 10 月 7 日写的病毒……- -爆肝，既然经历了这么多年的时光还能在网络上进行传染，就说明很有分析的必要吧……  
![](img/b4c372afcada3e32e4c0f1270155d1a2.jpg)  
  当然现在的杀毒软件都可以干掉的、打破了我两年裸奔不中毒的记录啊……- -唉。  
  好吧……现在开始拆它。  
  以感染的 InternetExplore 为例（版本号 6.0.2900.5512）  
感染后明显多出一段![](img/3981c8fba8a63b94738a9cab40dab737.jpg)  
  对比感染前后 PE 头  
![](img/e637b3202e5cd538e445855c206c8e44.jpg)  
![](img/f701e6c79428bb1f01382a5d8c1c9a41.jpg)  
  我们先来分析被感染程序的 Loader  
  用到的 IDA 里的解码脚本  
  - -其实渣质量的.  
  

代码:

```
    mov     eax, 0EAB8D//decode key
    mov     edx, 0041901C//解码起始位置
    mov     esi, 598//固定大小
  L007:
    xor     dword ptr \[edx+esi\], eax
    dec     esi
    sub     esi, 3
    jnz L007
  
  #include <idc.idc>
  static decrypt(end, key)
  {
    auto i, x;
    end\=end+0x598;
    for (i=0x598;i!=0;i=i-4)
    {
      x=Dword(end);
      x= (x^key);
      PatchDword(end,x);
      end = end - 4;
    }
  }
  

```

  执行 decrypt(0x41901c,0x0EAB8D)即可  
  
  既然是解码,我们就把这段叫作 DecodeProc 吧……解码段最后跳去执行 Loader 的 main 函数了  
代码:

```
  .qnk\_:0041901C                 jnz     short loc\_419013
  .qnk\_:0041901E                 nop
  .qnk\_:0041901F                 nop
  .qnk\_:00419020                 call    main
  .qnk\_:00419020 DecodeProc      endp
  .qnk\_:00419020
  .qnk\_:00419020 ; ---------------------------------------------------------------------------
  

```

Loader 流程  
![](img/ff13140180f7377234f28563b89c85cf.jpg)  
  Loader 数据结构  
代码:

```
  00419020  00017DE8  CALL
  00419024  00000000------------------------------------------------
  00419028  00400000  IEXPLORE.00400000;下面一砣是原程序和病毒体信息
  0041902C  00002451  +0x0C OEP RVA
  00419030  00016C00  +0x10 原大小
  00419034  000171D6  +0x14 加密病毒 Dll 偏移
  00419038  0002AA00  +0x18 病毒 Dll 大小
  0041903C  00000000
  00419040  00401000  +0x20 <&KERNEL32.LoadLibraryA>
  00419044  00402B4C  +0x24 ASCII "UnhandledExceptionFilter"
  00419048  00402AE2  +0x28 ASCII "GetCommandLineA"
  0041904C  00001AEC
  00419050  00002B4A  +0x30 被替换掉的第一个 IAT Thunk
  00419054  00002AE0  +0x34 被替换掉的第二个 IAT Thunk  - -插一句，其实如果我早就注意到这里的话专杀的代码可以直接用这个数据、还能省掉减掉基址那一步……嘛，算了，反正效果一样、懒得去改了
  00419058  00000400
  0041905C  7C863E6A  kernel32.UnhandledExceptionFilter; 被替换掉的 Kernel32 前两个函数
  00419060  7C812FAD  kernel32.GetCommandLineA
  

```

  Loader main 函数的关键点  
代码:

```
  push    ebp
  mov     ebp, esp
  add     esp, -140
  mov     eax, ebp
  add     eax, 4                   ; eax\=DATA after call
  mov     edx, ebp
  push    ebx
  push    esi
  push    edi
  xor     ebx, ebx
  mov     edi, dword ptr \[eax\]     ; edi\=DATA
  lea     ecx, dword ptr \[ebp\-38\]
  sub     edi, 5                   ; CALL addr
  mov     eax, dword ptr \[edi+C\]   ; eax\=oep rva
  lea     esi, dword ptr \[edi+84\]  ; esi\=lpKernel32\_dll
  add     eax, dword ptr \[edi+8\]   ; \[edi+8\]base addr
  add     edx, 4
  mov     dword ptr \[edx\], eax     ; 返回地址=oep
  

```

  在 InitImportTable 函数里回填 IAT 并准备 Loader 之后要用到的 IAT，下面的代码是用来回填 IAT 用的  
代码:

```
  push    dword ptr \[eax+24\]
  push    hKernel32
  call    **GetProcAddress**
  mov     edx, dword ptr \[ebp+C\]
  mov     ecx, dword ptr \[edx+20\]
  mov     dword ptr \[ecx\], eax
  mov     eax, dword ptr \[ebp+C\]
  push    dword ptr \[eax+28\]
  push    hKernel32
  call    **GetProcAddress**
  mov     edx, dword ptr \[ebp+C\]
  mov     ecx, dword ptr \[edx+20\]
  add     ecx, 4
  mov     dword ptr \[ecx\], eax
  

```

  
  剩下的几个关键函数用 C 代码的方式分析  
代码:

```
  bool \_\_stdcall IsInfected(int a1, int a2, int a3)
  {
    if ( !**RegOpenKeyEx**(HKEY\_CURRENT\_USER,
            a3 + 189,                             // Software\\Microsoft\\Windows\\CurrentVersion\\Explorer
            0,
            KEY\_READ,
            &hKey) )
    {
      bInfected = (\*(int (\_\_stdcall \*\*)(HKEY, int, \_DWORD, \_DWORD, int, int \*))(a2 + 48))(hKey, a3 + 240, 0, 0, a1, &v5) == 0;
      (\*(void (\_\_stdcall \*\*)(HKEY))(a2 + 52))(hKey);// RegQueryKeyValueExA(X,PINF...)
                                                  //病毒核心 Dll 的文件位置
    }
    return bInfected;
  }
  
  char ExpandVirus(int a1<ebx\>, int a2<edi\>, int a3<esi\>, int a4, int a5, int a6)
  {
    GetModuleFileNameA(0, &v12, MAX\_PATH, a2, a3, a1);
    hFile = CreateFileA(&v12,GENERIC\_READ,1,0,3,1,0);
    if ( hFile == -1 )
    {
      BYTE3(v16) = 0;
    }
    else
    {
      GetTempPathA(MAX\_PATH, &v12);
      v15 = **GetTickCount**();       //生成随机文件名
      v7 = 0;
      do
      {
        v8 = \*((\_BYTE \*)&v15 + v7);
        \*((\_BYTE \*)&v15 + v7++) = v8 / 10 + 97;
      }
      while ( v7 <= 2 );
      BYTE3(v15) = 0;
      GetTempFileNameA(&v15, v8 % 10, &v12, &v15, 0, a4);
      v14 = CreateFileA(a4,GENERIC\_WRITE|GENERIC\_READ,1,0,2,128,0);
      if ( v14 == -1 )
      {
        BYTE3(v16) = 0;
      }
      else
      {                                           // 每次解码 10KB 病毒 DLL，然后写完了继续解
        v9 = \*(\_DWORD \*)(a6 + 24);
        (\*(void (\_\_stdcall \*\*)(int, \_DWORD, \_DWORD, \_DWORD))(a5 + 28))(hFile, \*(\_DWORD \*)(a6 + 20), 0, 0);// SetFilePoint
        for ( ; v9 > 0x2800; v9 -= 10240 )
        {
          (\*(void (\_\_stdcall \*\*)(int, char \*, signed int, char \*, \_DWORD))(a5 + 20))(hFile, &v11, 10240, &v13, 0);// ReadFile
          Decode2(\*(\_DWORD \*)(a6 + 128), &v11, 10240);// 解码病毒 DLL
          (\*(void (\_\_stdcall \*\*)(int, char \*, signed int, char \*, \_DWORD))(a5 + 24))(v14, &v11, 10240, &v13, 0);
        }
        (\*(void (\_\_stdcall \*\*)(int, char \*, unsigned int, char \*, \_DWORD))(a5 + 20))(hFile, &v11, v9, &v13, 0);// 
                                                  // - -解码最后一块，完事擦屁股
        Decode2(\*(\_DWORD \*)(a6 + 128), &v11, v9);
        (\*(void (\_\_stdcall \*\*)(int, char \*, unsigned int, char \*, \_DWORD))(a5 + 24))(v14, &v11, v9, &v13, 0);
        (\*(void (\_\_stdcall \*\*)(int))(a5 + 32))(v14);
        BYTE3(v16) = 1;
      }
      **CloseHandle**(hFile);
    }
    return BYTE3(v16);
  }
  
  char \_\_stdcall InitVirusDll(int a1, int a2, int a3, int a4)
  {
    v4 = (\*(int (\_\_stdcall \*\*)(int))a2)(a1);      // - -Load 从输入表里得到的病毒路径
    return v4
        && (v5 = (\*(int (\_\_stdcall \*\*)(int, int))(a2 + 4))(v4, a4 + 245)) != 0// **GetProcAddress**(Initiate)
        && (unsigned \_\_int8)((int (\_\_stdcall \*)(int))v5)(a3);// Initiate(Addr)
                                                  // 这里 Addr 是一开始 DecodeProc 执行完以后那个 Call 的地址
                                                  // 00419020    E8 7D010000     call    004191A2
                                                  // Loader 分析告一段落。下面继续 Dll 分析
  }
  

```

  以上代码删除了大量的废物、不过这样看起来流程更清晰。  
  
  Loader 分析到此结束。  
  ------------------------------------------------------------------------------------------------------------------  
  病毒执行的流程是强制替换 Kernel32 前两个函数为 LoadLibraryA 和 GetProcAddress,然后在运行的时候再还原。依据此原理，用 C++编了个 16K 的程序。感染以后 188KB……作者还真是不注意文件大小啊。这么大一砣东西粘到一个 16K 的程序上……蛋疼。  
  - -好吧，不扯淡了，继续……  
  
  下面是对于病毒核心 Dll 的分析  
  ------------------------------------------------------------------------------------------------------------------  
  核心 Dll 中 Initiate 函数  
代码:

```
  char \_\_cdecl Initiate(int a1)
  {
    hMutex = OpenMutexA(MUTEX\_ALL\_ACCESS, 0, "Residented");
    if ( !hMutex || (result = sub\_4019A8(v4, (int)&v3, 262)) != 0 && v3 < 7u && v3 >= 2u )
      result = (unsigned int)SetWindowsHookExA(WH\_CALLWNDPROC, (HOOKPROC)AttachHook, \*(HINSTANCE \*)off\_459938\[0\], 0);// 放钩子，咬人
                                                  // 全局钩子，注入 N 多有 CALLWNDPROC 的进程
                                                  // 感谢气泡熊的解答^\_^
    if ( hMutex )
      result = **CloseHandle**(hMutex);
    if ( !v8 )
    {
      v7 = 8;
      v9 = v4;
      if ( v4 )
      {
        v10 = \*(\_DWORD \*)v9;
        v7 = 56;
        result = (\*(int (\_\_fastcall \*\*)(int, signed int))(\*(\_DWORD \*)v9 - 4))(v9, 3);// call    ika1.008FC65C
                                                  // CleanUp
        v7 = 44;
      }
      if ( !v8 )
        result = v6;
    }
    return result;
  }
  
  0012FCF0   00000004  |HookType = WH\_CALLWNDPROC
  0012FCF4   008F1EBC  |Hookproc = offset ika1.AttachHook
  0012FCF8   008F0000  |hModule = 008F0000 (ika1)
  0012FCFC   00000000  \\ThreadID = 0
  

```

  
  通过以下的流程图可以清楚地看到病毒做坏事的过程  
![](img/245addc08a52f0a07091d1ab85dc27e3.jpg)  
  这里关于 InitInfectStruct 函数初始化的结构体的数据结构有必要说一下……  
代码:

```
  008F4944  008F4A14  +0x00 pFileInfo
  008F4948  008F4AA8  +0x04 p?(Size:0xC)
  008F494C  008F2268  +0x08 pSelf
  008F4950  008F4AD0  +0x0C pCurrentFileInfo(struct)
  008F4954  008F4B28  +0x10 p?(Size:0x4)
  008F4958  008F4B34  +0x14 pDecodeProc(Size:0xC)
  008F495C  00000000  +0x18 BOOL bSuccess;
  008F4960  008F48C8  +0x1C LPSTR lpInftectFile;
  008F4964  008F498C  +0x20 p(size:0x84 - Status:inited by 0)
  008F4968  00001800  +0x24 HANDLE hFile;//(Infecting)
  008F496C  00000820  +0x28 DWORD dwFileAttribute;
  008F4970  6E633000  +0x2C FILETIME CreationTime
  008F4974  01CA9B55
  008F4978  66265278  +0x30 FILETIME LastAccessTime
  008F497C  01CAAD4D
  008F4980  6E633000  +0x34 FILETIME LastWriteTime
  008F4984  01CA9B55
  
  00874863   |.  E8 70420400   call    008B8AD8 new
  
  0087487D   |.  E8 1EDCFFFF   call    008724A0;initSturct
  
  \_\_struct DecodeProc{
  };(Size:0x14)
  008F476C  008F457C pSelf
  008F4770  00000000
  008F4774  008F477C pDecodeCode
  
  
  
  
  \_\_struct FileInfo{
  };(Size:0x14)
  
  008F479C  00000003  +0x00 DWORD dwNumberOfSections;
  008F47A0  000000D8  +0x04 DWORD e\_lfanew;
  008F47A4  008F46CC  +0x08 pSelf
  008F47A8  008F47B4  +0x0C pNtHeader
  008F47AC  008F4910  +0x10 pIMAGE\_SECTION\_HEADER
  
  
  
  \_\_struct pCurrentFileInfo{
  };
  
  008F4858  008F46CC  +0x00 pSelf
  008F485C  008F4870  +0x04 LPSTR lpCurrentFilePath;
  008F4860  000000EC  +0x08 HANDLE hFile;(CurrentFile)
  008F4864  0000004A
  008F4868  00000001
  008F486C  00000038
  

```

  关于多态入口的生成函数、这里分析的比较少。总之就两种指令格式  
代码:

```
  push imm32
  pop reg32
  

```

  这是一种、还有  
代码:

```
  mov reg32,imm32
  

```

  
  因为所使用的寄存器并不是固定的，前一种可以判断是否是 0x68，后一种对应的机器码可以表示为 X&0xB0=0xB0，具体参见专杀的代码。  
代码:

```
  008F477C     90              nop
  008F477D     90              nop
  008F477E     68 90909090     push    90909090
  008F4783     5B              pop     ebx
  008F4784     BF 90909090     mov     edi, 90909090
  008F4789     90              nop
  008F478A     90              nop
  008F478B     68 90909090     push    90909090
  008F4790     5E              pop     esi
  008F4791     90              nop
  008F4792     90              nop
  008F4793     90              nop
  008F4794     90              nop
  008F4795     90              nop
  008F4796     90              nop
  

```

  ASM Table  
代码:

```
  008C4C30    B8 BBB9BABE     mov     eax, BEBAB9BB
  008C4C35    BF E8EBE9EA     mov     edi, EAE9EBE8
  008C4C3A    EE              out     dx, al
  008C4C3B    EF              out     dx, eax
  

```

  下面这段是关于 GetPEFileInfo 函数的一点 OD 笔记、也可以结合着在 IDA 里分析出的函数名一起看  
代码:

```
  00873140   /$  53            push    ebx
  00873141   |.  8BD8          mov     ebx, eax
  00873143   |.  8B43 0C       mov     eax, dword ptr \[ebx+C\]
  00873146   |.  8B4B 0C       mov     ecx, dword ptr \[ebx+C\]
  00873149   |.  0FB750 14     movzx   edx, word ptr \[eax+14\]            ;  SizeOfOptionalHeader;
  0087314D   |.  0FB741 06     movzx   eax, word ptr \[ecx+6\]             ;  NumberOfSections;
  00873151   |.  8BC8          mov     ecx, eax
  00873153   |.  8B43 0C       mov     eax, dword ptr \[ebx+C\]
  00873156   |.  C1E1 03       shl     ecx, 3
  00873159   |.  0353 04       add     edx, dword ptr \[ebx+4\]
  0087315C   |.  8D0C89        lea     ecx, dword ptr \[ecx+ecx\*4\]
  0087315F   |.  03D1          add     edx, ecx
  00873161   |.  8B48 54       mov     ecx, dword ptr \[eax+54\]           ;  SizeOfHeaders;
  00873164   |.  83C2 18       add     edx, 18
  00873167   |.  2BCA          sub     ecx, edx
  00873169   |.  83F9 28       cmp     ecx, 28
  0087316C   |.  73 0E         jnb     short 0087317C
  0087316E   |.  8B53 08       mov     edx, dword ptr \[ebx+8\]
  00873171   |.  33C0          xor     eax, eax
  00873173   |.  C742 18 03000>mov     dword ptr \[edx+18\], 3
  0087317A   |.  5B            pop     ebx
  0087317B   |.  C3            retn
  0087317C   |>  8B53 0C       mov     edx, dword ptr \[ebx+C\]
  0087317F   |.  0FB74A 06     movzx   ecx, word ptr \[edx+6\]             ;  NumberOfSections;
  00873183   |.  41            inc     ecx
  00873184   |.  8BC1          mov     eax, ecx
  00873186   |.  C1E0 03       shl     eax, 3
  00873189   |.  8D0480        lea     eax, dword ptr \[eax+eax\*4\]
  0087318C   |.  50            push    eax
  0087318D   |.  E8 E65A0400   call    008B8C78
  00873192   |.  59            pop     ecx
  00873193   |.  8943 10       mov     dword ptr \[ebx+10\], eax
  00873196   |.  8B53 0C       mov     edx, dword ptr \[ebx+C\]
  00873199   |.  8B43 08       mov     eax, dword ptr \[ebx+8\]
  0087319C   |.  0FB752 14     movzx   edx, word ptr \[edx+14\]            ;  SizeOfOptionalHeader;
  008731A0   |.  0353 04       add     edx, dword ptr \[ebx+4\]
  008731A3   |.  83C2 18       add     edx, 18
  008731A6   |.  E8 39F7FFFF   call    008728E4
  008731AB   |.  8B4B 0C       mov     ecx, dword ptr \[ebx+C\]
  008731AE   |.  8B53 10       mov     edx, dword ptr \[ebx+10\]
  008731B1   |.  0FB741 06     movzx   eax, word ptr \[ecx+6\]
  008731B5   |.  8BC8          mov     ecx, eax
  008731B7   |.  8B43 08       mov     eax, dword ptr \[ebx+8\]
  008731BA   |.  C1E1 03       shl     ecx, 3
  008731BD   |.  8D0C89        lea     ecx, dword ptr \[ecx+ecx\*4\]        ;  读取节表
  008731C0   |.  E8 2FF7FFFF   call    008728F4
  008731C5   |.  8B53 08       mov     edx, dword ptr \[ebx+8\]
  008731C8   |.  B9 28000000   mov     ecx, 28
  008731CD   |.  8B43 08       mov     eax, dword ptr \[ebx+8\]
  008731D0   |.  8B52 20       mov     edx, dword ptr \[edx+20\]
  008731D3   |.  83C2 58       add     edx, 58
  008731D6   |.  E8 19F7FFFF   call    008728F4
  008731DB   |.  8B43 0C       mov     eax, dword ptr \[ebx+C\]
  008731DE   |.  0FB750 06     movzx   edx, word ptr \[eax+6\]
  008731E2   |.  8913          mov     dword ptr \[ebx\], edx
  008731E4   |.  B0 01         mov     al, 1
  008731E6   |.  5B            pop     ebx
  008731E7   \\.  C3            retn
  

```

  
  剩下大量的关于核心 Dll 的分析请参见附件中的 idb 文件。  
  
  下面的是我写的专杀中用于判断是否感染病毒和进行病毒清除的核心代码  
代码:

```
  if (pSH->Name\[0\]=='.'&&\\
    (pSH->Name\[1\]-6==pSH->Name\[2\]-3)&&\\
    (pSH->Name\[2\]-3==pSH->Name\[3\])&&\\
    pSH->Name\[4\]==0x7)
  {
    CString str;
    str.Format("Infected file found on %s",sPath);
    OutputDebugString(str);
    pSH= IMAGE\_FIRST\_SECTION(pNtH);
    DWORD dwKey,dwStartAddr;
    dwStartAddr=rva2raw(pNtH->FileHeader.NumberOfSections, pSH, pNtH->OptionalHeader.AddressOfEntryPoint)+(DWORD)lpFile;
    DWORD dwEnd;
    dwEnd=(DWORD)lpFile+GetFileSize(hFile,0);
    for (dwStartAddr;dwStartAddr<dwEnd;dwStartAddr++)
    {
      if((\*((PBYTE)dwStartAddr)&0xB0)==0xB0||\*(PBYTE)(dwStartAddr)==0x68)
      {
        dwKey=\*(PDWORD)(dwStartAddr+1);
        break;
      }
    }
    for (dwStartAddr+=5;dwStartAddr<dwEnd;dwStartAddr++)
    {
      if((\*((PBYTE)dwStartAddr)&0xB0)==0xB0||\*(PBYTE)(dwStartAddr)==0x68)//mov edx,XXor push..
      {
        dwStartAddr=rva2raw(pNtH->FileHeader.NumberOfSections, pSH, \*(PDWORD)(dwStartAddr+1)-pNtH->OptionalHeader.ImageBase)+(DWORD)lpFile;
        break;
      }
    }
    DWORD oep,imp1,imp2;
    oep=dwStartAddr+0x10;
    imp1=dwStartAddr+0x28;
    imp2=dwStartAddr+0x2C;
    oep=(\*(PDWORD)oep)^dwKey;
    imp1=(\*(PDWORD)imp1)^dwKey;
    imp1=/\*rva2raw(pNtH->FileHeader.NumberOfSections, pSH,\*/ imp1-pNtH->OptionalHeader.ImageBase;//);
    imp2=(\*(PDWORD)imp2)^dwKey;
    imp2=/\*rva2raw(pNtH->FileHeader.NumberOfSections, pSH,\*/ imp2-pNtH->OptionalHeader.ImageBase;//);
    str.Format("OEP RVA found at: 0x%.8X",oep);
    OutputDebugString(str);
    str.Format("First API name swap RVA found at : 0x%.8X\\n",imp1);
    OutputDebugString(str);
    str.Format("Second API name swap RVA found at : 0x%.8X\\n",imp2);
    OutputDebugString(str);
    pImportDescriptor=(PIMAGE\_IMPORT\_DESCRIPTOR)((DWORD)lpFile+ rva2raw(pNtH->FileHeader.NumberOfSections, pSH, pNtH->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\].VirtualAddress));
    while(pImportDescriptor->FirstThunk)
    {
      char\* dllname = (char\*)((DWORD)lpFile + rva2raw(pNtH->FileHeader.NumberOfSections, pSH, pImportDescriptor->Name));
      if (!stricmp(dllname,"kernel32.dll"))
      {
        pThunkData = (PIMAGE\_THUNK\_DATA)((DWORD)lpFile+ rva2raw(pNtH->FileHeader.NumberOfSections, pSH, pImportDescriptor->OriginalFirstThunk));
        pThunkData->u1.Function = imp1-2;
        pThunkData++;
        pThunkData->u1.Function = imp2-2;
        break;
      }
      else
        pImportDescriptor++;
    }
    pNtH->OptionalHeader.AddressOfEntryPoint=oep;
    pSH= IMAGE\_FIRST\_SECTION(pNtH)+pNtH->FileHeader.NumberOfSections-1;
    dwSection=pSH->PointerToRawData;
    memset(pSH,0,sizeof(PIMAGE\_SECTION\_HEADER));
    pNtH->FileHeader.NumberOfSections--;
    pNtH->OptionalHeader.SizeOfImage-=0x1000;
    cntVirus++;
    memcpy(pNewBuf,(PCHAR)lpFile+dwSection-4,4);
  }
  

```

  完全代码请参见附件中的 ResidentKiller 源代码。  
  
\--------------------------------------------------------------------------------  
【经验总结】  
  我给这个病毒起个名字好了，综合网上查到的资料和各大杀毒引擎的结果,给这个病毒命名为 Worm.Parite.Residented  
  VirusTotal 扫描结果  
  http://www.virustotal.com/zh-cn/analisis/47bd590e99bee93bb0b46d7b74d60da3c931b8115122a4b450045591ea366399-1280672787  
![](img/9015343f5b53a1c6f9e6827e167ca4c3.jpg)  
  - -专杀的源码写的太挫了，大家就凑合着看吧、第一次在实战中应用 dfs 和 bfs……应该算是吧……反正是学病毒里遍历磁盘的方法  
![](img/a1aa6d7461138ebd45039b0e0f964f3c.jpg)  
  非常感谢气泡熊的解答、还有师傅。  
附件目录以及 Hash  
![](img/a7d6befab64964f782ecea5f788a8dab.jpg)  
![](img/19196d330d177fd9d92eb9e06d6316b3.jpg)  
  附件外链：http://fs.unpack.cn/?fs=26&u=145  
  病毒的主体思路:首先由感染病毒的文件中的 Loader 释放病毒 Dll 并调用 Initiate 函数,然后注入全局钩子，其中 Explorer 会在装载 Dll 时发动一个新的线程去做坏事、这样即使钩子卸掉了，病毒还是可以存活在 Explorer 中。然后分出来的线程负责感染硬盘 exe 和 src 文件、  
  
  终于做完专杀了……时间 2010.7.31.21:59  
  
  专杀思路：UnHook 掉所有类型为 CALLWND 的钩子，因为如果感染病毒的进程运行的话，很多的进程都会因为它的钩子而插入一个 Dll，这样将导致病毒核心 Dll 无法被清除，然后结束掉两个有 CALLWND 的进程，这两个程序的 Dll 是没有办法用 UnHook 卸掉的、只能干掉。bfsFile 一次搜索 8 个文件,避免 CPU 过高被发现,然后慢慢感染……  
  
  - -花了 1 天而已……开发速度已经比以前有了飞跃式的进展……当然以前也没有一次搞过这么长时间编程。  
  病毒分析告一段落了，反正专杀都做出来了，- -剩下的代码分析的太详细了反而不是很好。  
  第一次在实战中应用 bfs 和 dfs 啊……应该算是吧、  
  拖了这么长时间啊……- -真是太对不起你了，电脑啊……今天才给你复仇啊……  
  附件里有 3 个样本，一个是病毒的核心 Dll，一个是经过原版（- -实际上我后来发现我只留了个解压缩过的 Dll，原版已经找不到了，不过一样分析……哈哈~）Dll 感染过的 DeathStartup，一个是经过改良的（因为原版感染速度实在太糟糕了，bfsFile 的时候 8 个文件中间就要隔 N 长时间。懒得等的同学可以使用第二个附件，No\_Delay 版本……）不这个版本对硬盘不是很友好的说……  
![](img/384a87d5cc34000ec495f138108b62bf.jpg)  
  就这样吧……皆さん、つてください  
  
\--------------------------------------------------------------------------------  
【版权声明】: 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2010 年 08 月 04 日  
                                                           Azure\[LCG\]