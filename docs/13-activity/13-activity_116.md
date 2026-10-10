

# 我写的第一题分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-249.htm`](https://www.pediy.com/kssd/pediy09/pediy09-249.htm)

> 金山杯第一阶段第一题分析  
>   
> 原始文件没有加壳，使用 IDA 加载文件，并手动加载 Sig 文件 Microsoft VisualC 2-8/net runtime。  
> 程序入口点代码为  
> .text:00400522     public start  
> .text:00400522 start proc near  
> .text:00400522     push 0                     ; lpModuleName  
> .text:00400524     call GetModuleHandleA  
> .text:0040052A     push 0                     ; dwInitParam  
> .text:0040052C     push offset DialogFunc  ; lpDialogFunc  
> .text:00400531     push 0                     ; hWndParent  
> .text:00400533     push 65h                  ; lpTemplateName  
> .text:00400535     push eax                  ; hInstance  
> .text:00400536     call DialogBoxParamA    ; Create a modal dialog box from a  
> .text:00400536                                 ; dialog box template resource  
> .text:0040053C     retn  
> .text:0040053C start endp  
> 整个程序只有一个 Dialog，处理函数为 400437 处的 DialogFunc,Dialog 的处理函数原形为  
> INT\_PTR CALLBACK DialogProc(          HWND hwndDlg,  
>     UINT uMsg,  
>     WPARAM wParam,  
>     LPARAM lParam  
> );  
> 对照原形修改 400437 处对应函数参数设定。以下对程序 DialogFunc 的代码进行详细的分析  
> .text:00400437 DialogFunc proc near          ; DATA XREF: start+A o  
> .text:00400437  
> .text:00400437 var\_1018= dword ptr -1018h  
> .text:00400437 String= byte ptr -14h  
> .text:00400437 hwndDlg= dword ptr  8  
> .text:00400437 uMsg= dword ptr  0Ch  
> .text:00400437 wParam= dword ptr  10h  
> .text:00400437 lParam= dword ptr  14h  
> .text:00400437  
> .text:00400437     push ebp  
> .text:00400438     mov ebp, esp  
> .text:0040043A     mov eax, 1018h  
> .text:0040043F     call \_\_alloca\_probe ;在栈上分配局部变量空间  
> .text:00400444     and \[ebp+String\], 0 ；初始化 String 为 0,\[ebp-14h\]  
> .text:00400448     push ebx  
> .text:00400449     push esi  
> .text:0040044A     push edi  
> .text:0040044B     xor eax, eax  
> .text:0040044D     lea edi, \[ebp-13h\]  ;IDA 未能识别出的变量,\[ebp-13h\]  
> .text:00400450     stosd                  ;edi 指向的 4 个字节初始化为 0  
> .text:00400451     stosd                  ;edi 指向的 4 个字节初始化为 0  
> .text:00400452     and byte ptr \[ebp+var\_1018\], 0 ;初始化 var\_1018 第一个字节为 0  
> .text:00400452                                          ;\[ebp-1018h\]  
> .text:00400459     mov ecx, 400h         ;设置 00400476 处 stosd 指令的执行次数  
> .text:0040045E     stosd                  ;edi 指向的 4 个字节初始化为 0  
> .text:0040045F     stosd                  ;edi 指向的 4 个字节初始化为 0  
> .text:00400460     xor eax, eax  
> .text:00400462     cmp \[ebp+uMsg\], 111h;判断消息是否是 WM\_COMMAND  
> .text:00400469     lea edi, \[ebp+var\_1018+1\]  ;edi 指向 var\_1018 第 2 个字节  
> .text:0040046F     mov ebx, 13572468h           ;设置 ebx 初始值  
> .text:00400474     rep stosd                      ;填充 var\_1018+1 开始的 4096 字节为 0  
> .text:00400476     jnz loc\_400519                ;消息不是 WM\_COMMAND  
> 根据这一段分析的变量初始化，可以知道 var\_1018 为一个长度为 4097 字节的字节数组，String 为一个长度为 17 字节的字节数组，并且以上两个数组的内容都被初始化为 0，因此修改 IDA 分析结果中的对应局部变量为指定长度的字节数组。  
> 继续向下进行  
> .text:00400493     mov esi, GetDlgItemTextA  
> .text:00400499     lea eax, \[ebp+String\]  
> .text:0040049C     push 10h                  ; nMaxCount  最大长度  
> .text:0040049E     push eax                  ; lpString   缓存指针  
> .text:0040049F     push 3E9h                 ; nIDDlgItem 输入用户名的 Edit  
> .text:004004A4     push \[ebp+hwndDlg\]        ; hDlg  
> .text:004004A7     call esi ; GetDlgItemTextA  
> .text:004004A9     test eax, eax  
> .text:004004AB     jz  short loc\_400519    ;取得的长度为 0 则跳  
> .text:004004AD     lea eax, \[ebp+var\_1018\]  
> .text:004004B3     push 1000h                ; nMaxCount 最大长度  
> .text:004004B8     push eax                  ; lpString  缓存指针  
> .text:004004B9     push 3EAh                 ; nIDDlgItem 输入注册号的 Edit  
> .text:004004BE     push \[ebp+hwndDlg\]        ; hDlg  
> .text:004004C1     call esi ; GetDlgItemTextA  
> .text:004004C3     test eax, eax  
> .text:004004C5     jz  short loc\_400519    ;取得的长度为 0 则跳  
> 分析过这一段之后可以知道 String 为用户名字符串的数组，var\_1018 为注册号的数组，修改变量的名字为 szName 和 szKey。  
> .text:004004C7     lea edi, \[ebp+szName\]  ;edi 指向 szName  
> .text:004004CA     or  ecx, 0FFFFFFFFh  
> .text:004004CD     xor eax, eax  
> .text:004004CF     xor edx, edx  
> .text:004004D1     repne scasb  
> .text:004004D3     not ecx  
> .text:004004D5     dec ecx                  ;以上代码为计算 szName 长度  
> .text:004004D6     test ecx, ecx  
> .text:004004D8     jle short loc\_4004FD  
> .text:004004DA  
> .text:004004DA loc\_4004DA:                   ; CODE XREF: DialogFunc+C4 j  
> .text:004004DA     movsx eax, byte ptr \[ebp+edx+szName\];保留符号位扩展字节为双字  
> .text:004004DF     add eax, ebx  
> .text:004004E1     imul eax, 3721273h  
> .text:004004E7     add eax, 24681357h  
> .text:004004EC     mov esi, eax  
> .text:004004EE     shl esi, 19h           ;左移 0x19 位  
> .text:004004F1     sar eax, 7             ;算术右移 0x7 位  
> .text:004004F4     or  esi, eax  
> .text:004004F6     inc edx  
> .text:004004F7     cmp edx, ecx  
> .text:004004F9     mov ebx, esi  
> .text:004004FB     jl  short loc\_4004DA ;继续计算下一个字节  
> 这个部分代码的功能是循环计算 szName 的每个字节生成一个 32 位的双字，这部分代码转换为 C 代码为：  
>   //初始值  
>   int iNameKey=0x13572468;  
>   for (i = 0;i < strlen(szName);i++)  
>   {  
>     j=((iNameKey + szName\[i\]) \* 0x3721273) + 0x24681357;  
>     //有符号整数右移，SAR  
>     iNameKey=(j >> 7) | (j << 0x19);  
>   }  
> 继续  
> .text:004004FD loc\_4004FD:                   ; CODE XREF: DialogFunc+A1 j  
> .text:004004FD     lea eax, \[ebp+szKey\]  
> .text:00400503     push eax              ;指向 szKey 的指针  
> .text:00400504     push ebx              ;iNameKey  
> .text:00400505     call sub\_4002CC      ;关键比较调用  
> .text:0040050A     pop ecx  
> .text:0040050B     pop ecx  
> .text:0040050C     jmp short loc\_400519;调到函数结束  
> 这部分没什么可说的，继续进入 sub\_402CC  
> .text:004002CC     push ebp  
> .text:004002CD     mov ebp, esp  
> .text:004002CF     sub esp, 128h  
> .text:004002D5     and byte ptr \[ebp+var\_24\], 0 ;设置 var\_24 的第一个字节为 0,  
> .text:004002D9     push ebx  
> .text:004002DA     push esi  
> .text:004002DB     push edi  
> .text:004002DC     xor eax, eax  
> .text:004002DE     lea edi, \[ebp+var\_24+1\]      ;edi 指向 var24+1 处  
> .text:004002E1     stosd                            ; edi 处 4 个字节为 0,edi=edi+4  
> .text:004002E2     and \[ebp+Text\], 0             ;设置 Text 第一字节为 0,ebp-0x128  
> .text:004002E9     push 40h  
> .text:004002EB     stosd                            ; edi 处 4 个字节为 0,edi=edi+4  
> .text:004002EC     stosb                            ; edi 处 1 个字节为 0,edi=edi+1  
> .text:004002ED     pop ecx                          ;设置 4002FA 处指令 stosd 执行次数  
> .text:004002EE     xor eax, eax                    
> .text:004002F0     lea edi, \[ebp-127h\]            ;edi 指向 ebp-0x127,即 Text+1 处  
> .text:004002F6     or  \[ebp+var\_C\], 0FFh          ;设置 var\_c 的第一字节为 0xFF  
> .text:004002FA     rep stosd                        ;设置 edi 指向的 256 字节为 0  
> .text:004002FC     or  \[ebp+var\_18\], 0FFh         ;设置 var\_18 的第一字节为 0xFF  
> .text:00400300     or  ecx, 0FFFFFFFFh  
> .text:00400303     stosw                   ; 设置 edi 指向的 2 字节为 0,edi=edi+2  
> .text:00400305     stosb                   ; 设置 edi 指向的 1 字节为 0,edi=edi+1  
> .text:00400306     mov edi, \[ebp+lpszKey\];edi 指向输入的注册号  
> .text:00400309     xor eax, eax  
> .text:0040030B     repne scasb  
> .text:0040030D     not ecx  
> .text:0040030F     push 1  
> .text:00400311     dec ecx                  ;取得注册号长度保存在 ecx 中  
> .text:00400312     pop ebx                  ;设置 ebx 为 1  
> .text:00400313     mov byte ptr \[ebp-0Bh\], 63h;IDA 未正确分析出的变量,数组第二个字节  
> .text:00400317     mov \[ebp+var\_A\], 0FBh  
> .text:0040031B     mov \[ebp+var\_9\], 9Ah  
> .text:0040031F     mov \[ebp+var\_8\], 3  
> .text:00400323     mov \[ebp+var\_7\], 0A3h  
> .text:00400327     mov \[ebp+var\_6\], 0DAh  
> .text:0040032B     mov \[ebp+var\_5\], 72h  
> .text:0040032F     mov \[ebp+var\_4\], 0FEh  
> .text:00400333     mov \[ebp+var\_3\], 0C9h  
> .text:00400337     mov \[ebp+var\_2\], 0B7h  
> .text:0040033B     mov byte ptr \[ebp-17h\], 6Ah;IDA 未正确分析出的变量,数组第二个字节  
> .text:0040033F     mov \[ebp+var\_16\], 0D1h  
> .text:00400343     mov \[ebp+var\_15\], 0D2h  
> .text:00400347     mov \[ebp+var\_14\], 4Eh  
> .text:0040034B     mov \[ebp+var\_13\], 82h  
> .text:0040034F     mov \[ebp+var\_12\], 0DAh  
> .text:00400353     mov \[ebp+var\_11\], 72h  
> .text:00400357     mov \[ebp+var\_10\], 0FEh  
> .text:0040035B     mov \[ebp+var\_F\], 0C9h  
> .text:0040035F     mov \[ebp+var\_E\], 0B7h  
> 这部分代码也是完成局部变量的初始化过程，分析后可以知道 Text 为 260 字节的字节数组，var\_24 为 10 字节的字节数组，var\_C 和 var\_18 是两个 11 字节的字节数组，第一个字节为 0xFF，结合后面对这两个变量的引用，可以分析出这两个数组中保存的是编码后的注册成功和注册失败字符串，引用位置和解码函数如下：  
> .text:004003FC     lea eax, \[ebp+MsgText\]  
> .text:00400402     push eax  
> .text:00400403     lea eax, \[ebp+szOK\]   ;注册成功字符串  
> .text:00400406  
> .text:00400406 loc\_400406:                   ; CODE XREF: TestKey+169 j  
> .text:00400406     push eax  
> .text:00400407     call sub\_400240           ;字符串解码函数，简单的 XOR  
> .text:0040040C     pop ecx  
> .text:0040040D     lea eax, \[ebp+MsgText\]  
> .text:00400413     pop ecx  
> .text:00400414     push 0                    ; uType  
> .text:00400416     push offset Caption       ; lpCaption  
> .text:0040041B     push eax                  ; lpText  
> .text:0040041C     push 0                    ; hWnd  
> .text:0040041E     call MessageBoxA        ;显示提示  
> .text:00400424     pop edi  
> .text:00400425     mov eax, ebx  
> .text:00400427     pop esi  
> .text:00400428     pop ebx  
> .text:00400429     leave  
> .text:0040042A     retn  
> .text:0040042B ; ---------------------------------------------------------------------------  
> .text:0040042B  
> .text:0040042B loc\_40042B:                   ; CODE XREF: TestKey+C2 j  
> .text:0040042B                               ; TestKey+CA j ...  
> .text:0040042B     lea eax, \[ebp+MsgText\]  
> .text:00400431     push eax  
> .text:00400432     lea eax, \[ebp+szFail\] ;注册失败字符串  
> .text:00400435     jmp short loc\_400406  
> 修改对应的变量名称后继续分析  
> .text:00400363     mov esi, ecx ;ecx 为注册号长度，保存到 esi 中  
> .text:00400365     mov edi, ebx ;ebx 为 1，循环控制变量初始值为 1  
> .text:00400367  
> .text:00400367 loc\_400367:                   ; CODE XREF: TestKey+AC j  
> .text:00400367     mov eax, \[ebp+iNameKey\]  
> .text:0040036A     mov ecx, edi  
> .text:0040036C     shr eax, cl  
> .text:0040036E     and al, bl  
> .text:00400370     mov byte ptr \[ebp+edi+var\_24\], al;设置字节数组的对应位置  
> .text:00400374     inc edi  
> .text:00400375     cmp edi, 9  
> .text:00400378     jl  short loc\_400367 ;循环控制变量小于 9 则继续进行循环  
> .text:0040037A     xor edi, edi ;清空 edi  
> .text:0040037C     mov byte ptr \[ebp+var\_24+9\], bl;设置字节数组的第九位为 1  
> 此部分功能为保存 iNameKey 的二进制 1 至 8 位到 var\_24 数组的 1-8 位，并设置 var\_24 数组的第 9 位为 1，对应 C 语言代码如下  
>   //根据由 Name 计算出的 DWORD，初始化 Bit 数组  
>   
>   abyBits\[0\]=0;  
>   
>   for (i = 1;i < 9;i++)  
>   {  
>     //无符号数右移 shr  
>     abyBits\[i\] = ((DWORD)iNameKey >> (i)) & 1;  
>   }  
>   abyBits\[9\]=1;  
> 继续向下来到最核心的判断部分  
> .text:0040037F     test esi, esi  
> .text:00400381     jle short loc\_4003EE;注册号长度小于等于 0 则跳  
> .text:00400383  
> .text:00400383 loc\_400383:                   ; CODE XREF: TestKey+120 j  
> .text:00400383     mov eax, \[ebp+lpszKey\]  ;eax 指向注册号  
> .text:00400386     mov al, \[edi+eax\]        ;取注册号的第 edi 个字节,edi 初始值为 0  
> .text:00400389     cmp al, 30h  
> .text:0040038B     mov \[ebp+var\_1\], al      ;保存注册号当前要处理的字节到 var\_1  
> .text:0040038E     jl  loc\_40042B            ;小于’0’则跳  
> .text:00400394     cmp al, 39h  
> .text:00400396     jg  loc\_40042B            ;大于’9’则跳  
> .text:0040039C     mov eax, edi  
> .text:0040039E     push 1Fh  
> .text:004003A0     cdq  
> .text:004003A1     pop ecx  
> .text:004003A2     idiv ecx                   ;循环控制变量除 31(0x1F)  
> .text:004003A2                                  ;余数保存在 edx 中  
> .text:004003A4     mov eax, \[ebp+iNameKey\]  
> .text:004003A7     push 0Ah  
> .text:004003A9     mov ecx, edx              ;余数赋给 ecx  
> .text:004003AB     xor edx, edx  
> .text:004003AD     shr eax, cl               ;iNameKey 逻辑右移 cl 位  
> .text:004003AF     pop ecx  
> .text:004003B0     div ecx                    ;右移后的结果除 10(0x0A)  
> .text:004003B0                                  ;余数保存在 edx  
> .text:004003B2     movsx eax, \[ebp+var\_1\]  ;扩展 var1 到 32 位  
> .text:004003B6     lea eax, \[edx+eax-30h\]  ;eax=扩展后的 var1+余数-30  
> .text:004003BA     xor edx, edx  
> .text:004003BC     div ecx                    ;eax 除 10(0x0A),余数可能为 0 到 9  
> .text:004003BE     cmp edx, ebx              ;余数是否为 1  
> .text:004003C0     jnz short loc\_4003C7    ;不是则跳  
> .text:004003C2     xor byte ptr \[ebp+var\_24+1\], bl;edx 为 1 时,var\_24 数组第一字节  
> .text:004003C2                                          ;xor 1,即翻转状态  
> .text:004003C5     jmp short loc\_4003E9  
> .text:004003C7 ; ---------------------------------------------------------------------------  
> .text:004003C7  
> .text:004003C7 loc\_4003C7:                   ; CODE XREF: TestKey+F4 j  
> .text:004003C7     cmp byte ptr \[ebp+edx+MsgText+103h\], bl;当 edx 为 0 时,会越界判  
> .text:004003C7                                                    ;断 MsgText 的最后一个  
> .text:004003C7                                                    ;字节是否为 1,其与情况  
> .text:004003C7                                                    ;判断 var\_24 数组中的前  
> .text:004003C7                                                   ;一个字节(edx-1)是否为  
> .text:004003C7                                                   ;1  
> .text:004003CB     jnz short loc\_40042B;不是 1 则跳到注册失败,MsgText 最后一个字节为 0  
> .text:004003CB                             ; 所以 edx 不能为 0  
> .text:004003CD     lea eax, \[edx-2\]     ;内嵌循环控制变量终止值 eax=edx-2  
> .text:004003D0     mov ecx, ebx          ; 内嵌循环控制变量初始时化为 1  
> .text:004003D2     cmp eax, ebx          ;eax 和 1 比较  
> .text:004003D4     jl  short loc\_4003E1;小于 1 则跳,即如果 edx=2 则跳去直接执行翻转  
> .text:004003D6  
> .text:004003D6 loc\_4003D6:                   ; CODE XREF: TestKey+113 j  
> .text:004003D6     cmp byte ptr \[ebp+ecx+var\_24\], bl;判断数组 var24 的 ecx 位置字节  
> .text:004003DA     jz  short loc\_40042B                ;为 1 则跳去失败  
> .text:004003DC     inc ecx                               ;内嵌循环控制变量加 1  
> .text:004003DD     cmp ecx, eax                         ;是否到了循环终止值  
> .text:004003DF     jle short loc\_4003D6               ;不是则继续  
> .text:004003E1  
> .text:004003E1 loc\_4003E1:                   ; CODE XREF: TestKey+108 j  
> .text:004003E1     xor byte ptr \[ebp+edx+var\_24\], bl;翻转数组 var\_24 的 edx 位置字  
> .text:004003E1                                             ;节  
> .text:004003E5     lea eax, \[ebp+edx+var\_24\]  
> .text:004003E9  
> .text:004003E9 loc\_4003E9:                   ; CODE XREF: TestKey+F9 j  
> .text:004003E9     inc edi;外部循环控制变量加 1  
> .text:004003EA     cmp edi, esi;判断是否到了注册号的长度  
> .text:004003EC     jl  short loc\_400383;没到则继续循环  
> .text:004003EE  
> .text:004003EE loc\_4003EE:                   ; CODE XREF: TestKey+B5 j  
> .text:004003EE     mov eax, ebx;初始化 eax 为 1,eax 为验证循环控制变量  
> .text:004003F0  
> .text:004003F0 loc\_4003F0:                   ; CODE XREF: TestKey+12E j  
> .text:004003F0     cmp byte ptr \[ebp+eax+var\_24\], bl;判断 var\_24 的 eax 字节  
> .text:004003F4     jz  short loc\_40042B;为 1 则跳到注册失败  
> .text:004003F6     inc eax;验证循环控制变量加 1  
> .text:004003F7     cmp eax, 0Ah;是已经判断了 9 个字节  
> .text:004003FA     jl  short loc\_4003F0;不是则继续循环  
> 这部分是整个算法的核心部分，所以代码比较长，逻辑也比较复杂。整理后的描述如下：  
> 1、  保存由用户名计算出的 32 位 iNameKey 的 1 至 8 位到数组 var\_24 的 1-8 位置，并设置 var\_24 的 0 位置为 0,9 位置为 1。  
> 2、  对注册号的每个字符进行变换生成一个位置序号,生成算法为  
> 1)  当前处理的字符在注册号中的位置对 31 取余作为 iNameKey 右移的位数  
> 2)  iNameKey 右移后的结果对 10 取余后加上当前处理字符的 ASCII 码-0x30  
> 3)  第 2 步的结果再对 10 取余作为位置序号  
> 3、  根据位置序号进行判断，如果符合要求就对 var\_24 数组中的对应位置字节进行 xor 1，即翻转原始值，如果不符合要求则跳去失败，判断是否符合要求的条件如下  
> 1)  如果位置序号为 1，直接进行反转。  
> 2)  如果为其他位置，则首先判断此位置的前一位是否为 1，如果是 1 则符合，继续进行条件 3 的判断，否则不符合条件，失败。  
> 3)  位置序号如果等于 2，则成功，进行翻转，如果位置序号大于 2，则还要判断数组的第 1 位到(序号-2)位是否都是 0，如果都是 0 则符合条件，进行翻转，否则，跳去失败。  
> 4、  循环处完注册号的每一个字节后，var\_24 数组中的每个字节必须都是 0 才能注册成功  
>   
> 因此，整个问题变为找到一个位置序列，使用上面的 3 中的规则顺序处理此序列里的每个数字后 var\_24 数组中原来为的 1 的位置都变成 0，此序列必须满足的条件是，当前处理的序号如果是 1，则可以直接进行翻转，如果不是 1 则需要当前序号的前一位是 1 而且第一位到此序号-2 位都是 0 才可以进行翻转，所以设置 var\_24 数组中任一位为 0 的问题就会变成设置它的前一位为 1 的问题。  
> 举例子说，要设置数组 0 0 0 0 0 0 0 0 1 的第 9 位为 1 则需要先设置第 8 位由 0 变为 1，即把数组变为 0 0 0 0 0 0 0 1 1，而要使数组第 8 位由 0 变为 1 则需要第 7 位为 1，这是典型的递归求解问题。  
> 求解的方法是只要从数组的成员 1 开始，对数组的每一字节调用设置为 0 的递归函数，并记录下翻转顺序就可以得到满足条件的位置序列，生成算法如下：  
> 1)  设置数组中某一位为指定值的递归算法，此函数并未判断第 1 位到第 iPos-2 位是否全为 0，因此使用时需要调用者保证第 1 位到第 iPos-2 位已经为 0。  
> void SetBitValue(PBYTE abyBits,int iPos,BYTE byValue,char \*szKey)  
> {  
>     
>   //abyBits 为原始字节数组,iPos 为需要设置的位置,byValue 为要设置成的值  
>   //szKey 用来保存翻转位置序列  
>   
>   char szIndex\[2\]={0,0};  
>   if (abyBits\[iPos\] != byValue)  
>   {  
>     if (iPos == 1)  
>     {  
>   
>       //是第一位则直接设置  
>       abyBits\[iPos\] = byValue;  
>   
>       //添加位置编号到 szKey  
>       itoa(iPos,szIndex,10);  
>   
>       strncat(szKey,szIndex,4096);  
>     }  
>     else  
>     {  
>       //其他位  
>   
>       //设置前一位为 1  
>       SetBitValue(abyBits,iPos-1,1,szKey);  
>   
>       //设置目的位值  
>       abyBits\[iPos\] = byValue;  
>   
>       //添加位置编号到 szKey  
>       itoa(iPos,szIndex,10);  
>       strncat(szKey,szIndex,4096);  
>   
>       //设置前一位为 0  
>       SetBitValue(abyBits,iPos-1,0,szKey);  
>     }  
>   }  
> }  
> 2)  循环处理数组中的每个位置代码  
>   //计算翻转顺序串  
>   for (i = 1;i < 10;i++)  
>   {  
>     if (abyBits\[i\] == 1)  
>     {  
>               //由第一位开始按顺序进行处理，保证处理第 i 位时第 1 位到第 i-2 位为 0  
>               //此处也可以由第二位开始处理  
>       SetBitValue(abyBits,i,0,szKey);  
>     }  
>   }  
> 得到此翻转序列后，需要对此序列中的每个位置编号根据上面的规则 2 进行逆运算得到最终的注册号，算法如下：  
>   //初始化偏移数组  
>   
>   for (i = 0;i < 31;i++)  
>   {  
>     //无符号数右移 shr  
>     abyKeyTable\[i\] = (((DWORD)iNameKey) >> i) % 10;  
>   }  
>                 //由翻转顺序串和偏移数组循环计算，得到最终结果  
>   
>   for (i = 0;i < strlen(szKey);i++)  
>   {  
>     szKey\[i\]=(((szKey\[i\]-0x30) + 10 - abyKeyTable\[i % 31\]) % 10) + 0x30;  
>   }  
> 总结：  
> 1、本题的难度主要在于看懂注册号的验证过程，抽象出问题，并找到解决问题的方法。此题是经典的 9 连环问题，如果之前接触过此类问题，会降低很大难度，如果没接触过，就会在整理算法逻辑上多花一些时间。  
> 2、能完成注册的位置序列并不唯一，所以对应一个用户名的注册号也可以有很多个。  
> 3、以上代码都已经过验证可用但并未进行任何优化。 

 4dd466d0e1e0cc123afbe504da2462ca