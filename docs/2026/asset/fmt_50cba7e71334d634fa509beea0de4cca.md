
# [原创]逆向NP之注入`npggNT.des`

NP=nProtect GameGuard

---

**标题：**逆向NP之注入`npggNT.des`

**作者：**堕落天才

**时间：**2007-2-1

---

我们知道NP启动后会向所有进程注入模块`npggNT.des`,这个过程包括:向目标进程注入执行代码及数据,创建远程线程,远程线程执行,装入`npggNT.des`。之后的事我们都知道了:`npggNT.des`挂钩关键系统函数。在得到NP注入代码前，这个注入过程我想对于象我这样的菜鸟来说是非常神秘的，于是就产生了种种幻想，装入`npggNT.des`是不是使用更底层的系统函数？还是NP自己构建的`LoadLibrary`?又或是使用COPY来的系统函数代码？但是这两天分析的结果令我大跌眼睛它竟然是使用`LoadLibraryA`装入`npggNT.des`的，这怎么可能？？但事实上真的是这样，下面我们来看看代码就知道了。

### 1, 主要代码（这里是`npggNT.des`装入的主要执行过程，要看完整的二进制代码请看附件`NPRemotThread.asm`，编译后反汇编看）

代码：

```asm
00C10000    59              pop     ecx
00C10001    58              pop     eax
00C10002    51              push    ecx                              ;这里是一般函数都会做的寄存器压栈，好像没什么好说
00C10003    53              push    ebx
00C10004    55              push    ebp
00C10005    56              push    esi
00C10006    57              push    edi
00C10007    2BC9            sub     ecx, ecx                         ; ecx清零
00C10009    E8 00000000     call    00C1000E
00C1000E    5B              pop     ebx                              ; ebx=00C100E ->自定位函数入口地址
00C1000F    64:8B51 30      mov     edx, fs:[ecx+30]                 ; [ecx+30]=[30]
00C10013    83C3 F2         add     ebx, -0E                         ; EBX-E=00C10000 ->线程起始地址
00C10016    85C0            test    eax, eax
00C10018    53              push    ebx                              ; ebx指向线程起始地址
00C10019    50              push    eax                              ; 0
00C1001A    E8 23000000     call    00C10042
00C10042    64:FF31         push    dword ptr fs:[ecx]               ; fs:[ecx]=fs:[0]->SEH安装
00C10045    64:8921         mov     fs:[ecx], esp
00C10048    89A3 93000000   mov     [ebx+93], esp                    ; SMC [ebx+93]=[00C10093]->保存SEH返回的ESP
00C1004E    78 2C           js      short 00C1007C
00C10050    394A 0C         cmp     [edx+C], ecx                     ; [edx+c]=[7FFDC00C]=00241EA0
00C10053    75 30           jnz     short 00C10085
00C10085    68 0000807C     push    7C800000                         ; kernel32.dll 的HANDLE －>这里NP硬编码了kernel32.dll的句柄
00C1008A    53              push    ebx                              ; 线程起始地址
00C1008B    50              push    eax                              ; 0
00C1008C    FC              cld
00C1008D    E8 16010000     call    00C101A8
00C101A8    E8 00000000     call    00C101AD
00C101AD    5F              pop     edi                              ; EDI=00C101AD ->函数入口地址
00C101AE    8B5424 0C       mov     edx, [esp+C]                     ; EDX=kernel32.dll HANDLE
00C101B2    81C7 EF010000   add     edi, 1EF                         ; EDI=00C1039C ->npggNT.des路径
00C101B8    8D77 E4         lea     esi, [edi-1C]                    ; ESI=00C10380
00C101BB    6A 04           push    4
00C101BD    59              pop     ecx                              ; ECX=4
00C101BE    AD              lods    dword ptr [esi]
00C101BF    8B28            mov     ebp, [eax]                       ; 这里非常精彩，后面会详细分析
00C101C1    03EA            add     ebp, edx                         ;
00C101C3    896E FC         mov     [esi-4], ebp                     ;
00C101C6  ^ E2 F6           loopd   short 00C101BE
00C101C8    8B5F FE         mov     ebx, [edi-2]                     ; [edi-2]=[00C1039A]=3A440002
00C101CB    0B5F FA         or      ebx, [edi-6]                     ; [edi-6]=[00C10396]=00090100 EBX=3A4D0102
00C101CE    8B47 F4         mov     eax, [edi-C]                     ; [edi-C]=[00C10390]=00000003
00C101D1    66:894F FA      mov     [edi-6], cx                      ; word ptr [edi-6]=[00C10396]=cx=0
00C101D5    F6C7 08         test    bh, 8                            ; bh=01 and 8 =0001b and 1000b =0
00C101D8    0F85 CA000000   jnz     00C102A8
00C101DE    8B07            mov     eax, [edi]                       ; [edi]=[00C1039C]=475C3A44->npggNT.des路径
00C101E0    F6C7 02         test    bh, 2                            ; bh=01 and 2 =0001b and 0010b=0
00C101E3    75 10           jnz     short 00C101F5
00C101E5    8BF0            mov     esi, eax                         ; esi->npggNT.des路径开始4个字节
00C101E7    85C0            test    eax, eax                         ; 判断路径是不是为空
00C101E9    74 02           je      short 00C101ED
00C101EB    8BF7            mov     esi, edi                         ; esi=00C1039C
00C101ED    84DB            test    bl, bl                           ; bl=02
00C101EF    56              push    esi                              ; \
00C101F0    78 3D           js      short 00C1022F                   ;  eax=GetModuleHandleA("D:\\...\\npggNT.des")
00C101F2    FF57 E4         call    [edi-1C]                         ; /
00C101F5    F6C3 02         test    bl, 2                            ; bl=02
00C101F8    75 4B           jnz     short 00C10245                   ; bl 不等于零 跳转实现
00C10245    F6C7 02         test    bh, 2                            ; bh=01
00C10248    75 2F           jnz     short 00C10279                   ; bh=01 and 2=0001b and 0010b=0 跳转未实现
00C1024A    85F6            test    esi, esi                         ; esi=00C1039C->npggNT.des所在路径
00C1024C    74 2B           je      short 00C10279                   ; 不为零
00C1024E    F6C7 01         test    bh, 1                            ; bh=01 and 1=0001b and 0001b=1
00C10251    74 04           je      short 00C10257
00C10253    85C0            test    eax, eax                         ; eax是执行GetModuleHandleA("D:\\...\\npggNT.des")的结果
00C10255    75 22           jnz     short 00C10279                   ; 确定npggNT.des未被加载
00C10257    FF4F F8         dec     dword ptr [edi-8]                ; [00C10394]=00000001-1=0 jmp回来-1=-1
00C1025A    78 1D           js      short 00C10279                   ; 不为负 为负跳
00C1025C    84DB            test    bl, bl                           ; bl=02
00C1025E    56              push    esi                              ; \
00C1025F    78 05           js      short 00C10266                   ;  eax=LoadLibraryA("D:\\...\\npggNT.des")
00C10261    FF57 E8         call    [edi-18]                         ; /
00C10264  ^ EB F1           jmp     short 00C10257
00C10279    85C0            test    eax, eax                         ; eax为模块npggNT.des的句柄
00C1027B    75 07           jnz     short 00C10284
00C1027D    B8 B6F3C2E1     mov     eax, E1C2F3B6
00C10282    EB 64           jmp     short 00C102E8
00C10284    8B4F F4         mov     ecx, [edi-C]                     ; [edi-C]=[00C10390]=00000003
00C10287    F6C7 04         test    bh, 4                            ; bh=01 and 4=0001b and 0100b=0
00C1028A    74 04           je      short 00C10290
00C1028C    03C1            add     eax, ecx
00C1028E    EB 18           jmp     short 00C102A8
00C10290    85C9            test    ecx, ecx                         ; ecx=3
00C10292    75 0B           jnz     short 00C1029F
00C10294    8D8F 04010000   lea     ecx, [edi+104]
00C1029A    8039 00         cmp     byte ptr [ecx], 0
00C1029D    74 49           je      short 00C102E8
00C1029F    84DB            test    bl, bl                           ; bl=02
00C102A1    51              push    ecx                              ; \
00C102A2    50              push    eax                              ;   eax=GetProcAddress(HandleOfNpggNT.des,03)
00C102A3  ^ 78 CB           js      short 00C10270
00C102A5    FF57 F0         call    [edi-10]                         ; /
00C102A8    85C0            test    eax, eax                         ; eax=458A1830->npggNT.des function #03
00C102AA    75 07           jnz     short 00C102B3
00C102AC    B8 B7F3C2E1     mov     eax, E1C2F3B7
```
