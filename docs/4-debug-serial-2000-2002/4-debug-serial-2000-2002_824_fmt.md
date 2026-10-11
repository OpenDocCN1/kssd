
# IP\_tools2.06

> 原文：[`www.pediy.com/kssd/pediy04/kanxue410.htm`](https://www.pediy.com/kssd/pediy04/kanxue410.htm)

## IP_TOOLS2.06 加密算法揭密

几周前看见坛子上在讨论 `IP_TOOLS2.06`，本菜鸟对 IP 工具较感兴趣，就想拿来试试手，happy 一下。

谁知试了一周，花了 n 个小时（n>40)，晕菜无数次，都没搞定，只好走人咯，郁闷啊！这阵放假，本想再试试，整理些资料就算了，没想到竟还大致搞明白了，晕倒！（由于这个破程序 n 郁闷本菜鸟，本菜不禁多吐了些苦水，各位老兄还请多包涵啊:)）好吧，言归正传，下面我们开始吧:)

这个文件注册失败会弹出消息框，偶就想用 `MessageBox`、`MessageboxEx` 以及生成消息框，对话框的那些 API 来拦截，没想到都栏不到，郁闷偶半天。后来用 VC 的 `SPY++`看（`softice` 中用 `hwnd` 应该也能看得到），才发现 `IP_tools` 中的对话框消息框大多使用他们自己开发的类如 `TRegisterDlg`, `TRegisterDlg2`, `TMessageForm`，而不是 windows 自带的 `Dialog` 类。乖乖，敌人的做工还很细噢，菜鸟很佩服的恨恨的说。（如果有时间，研究一下如何自定义窗口类实现模式对话框消息框的效果倒是很有意思）

用 `CreateWindowExa` 倒是可以拦下，不过好像有点怪，菜鸟没有继续，就改拦截 `HMEMCPY` 了。

windows 9X 下，`HMemcpy` 是在 16bit 保护模式运行，被拦截下时的 `ES` 寄存器是目的地址的段 selector，通过 `softice` 的 GDT `****`查看该段的详细信息，段的初始地址就是 32 位下的目的地址。对这个地址下 `bpm` 跟噢，很快程序就会被这个段点拦截下来，看一下是 32 位模式下的串拷贝，一般使用 `ESI`, `EDI`，那就接着对 `EDI-1`(或 2,4)(其实减不减都差不多啦）下 `bpm`，继续跟啊。折腾了半天，发现了程序进行计算和比较的地方了，算是取得了点阶段性的成果，呵呵。不过菜鸟到这花了 7，8 个小时，菜啊，伤心噢。菜鸟使用用户名是 `happy`, 密码是 `1112222`，

注：

* 本文中的数字都是 16 进制数
* 文中有时指针与指针指向的值混用，不过应该能看出来什么时候是什么

## 下面看程序

```
CODE:004AC629               mov    eax, [ebp+var_10]     // d eax, 看到用户名 happy, 
CODE:004AC62C               call   sub_49D94C             //对用户名进行操作，返回加密后的字串长度
CODE:004AC631               mov    edi, eax
CODE:004AC633               lea    edx, [ebp+var_10]
CODE:004AC636               mov    eax, [esi]
CODE:004AC638               mov    eax, [eax+1F4h]
CODE:004AC63E               call   sub_4236DC     
CODE:004AC643               mov    eax, [ebp+var_10]     //d eax, 看到 11112222
CODE:004AC646               call   sub_49D9E8       //对密码进行操作
CODE:004AC64B               cmp    di, ax       
CODE:004AC64E               jnz    loc_4AC7CA       //到了 4ac7ca 我们就不 happy 了，
CODE:004AC654               mov    eax, ds:off_4E128C   //4e128c 中存放的是地址 4e3434, 是用户名加密后的存放地址
CODE:004AC659               mov    edx, 1FFh
CODE:004AC65E               call   sub_49D930       //对加密后的用户名计算
CODE:004AC663               mov    edi, eax
CODE:004AC665               mov    eax, ds:off_4E1190   //4e1190 中存放的是地址 4e3634, 是密码加密后的存放地址
CODE:004AC66A               mov    edx, 1FFh
CODE:004AC66F               call   sub_49D930       //对加密后的密码计算
CODE:004AC674               cmp    edi, eax       
CODE:004AC676               jnz    loc_4AC7CA
CODE:004AC67C               mov    eax, ds:off_4E128C   //这以下按双字比较 4e3434+7， 4e3634+7 空间中的内容
CODE:004AC681               add    eax, 7
CODE:004AC684               mov    eax, [eax]
CODE:004AC686               mov    edx, ds:off_4E1190
CODE:004AC68C               add    edx, 7
CODE:004AC68F               cmp    eax, [edx]
CODE:004AC691               jnz    loc_4AC7CA
CODE:004AC697               lea    edx, [ebp+var_8]       //三次比较都成功了，注册成功的话框就跳出来了噢
。。。。。
CODE:004AC6D8               mov    eax, offset aThankYouForReg ; "Thank you for registering IP-To"
                              //到这菜鸟才注意到这个字串，要是开始时就查找这个子串也许省偶很多事噢，心痛心痛！
。。。。。。
CODE:004AC7CA               mov    eax, offset aSorryYourReg_0 ; "Sorry, your registration number or "
                              // game  over 的字串！Go to hell!
CODE:004AC7CF               call   sub_444700
```

先看一下 `int 49D930`（`int i, char *pChar`), 该函数对 (`pChar+0B`)开始的字符窜按字求和， 长度为 `i-0B` (`i=1ff`)。

让我们看一下重要的两个 `call 49d94c`，`49d9e8`.

### 49d94c

```
CODE:0049D94C               push    ebp
。。。。。。
CODE:0049D9A8               push    eax
CODE:0049D9A9               push    offset unk_4E3434
CODE:0049D9AE               push    ebx
CODE:0049D9AF               push    offset unk_4DCB62
CODE:0049D9B4               push    offset unk_4DCC62
CODE:0049D9B9               call    sub_49DFFA      //加密的核心函数
CODE:0049D9BE               movsx  eax, al
。。。。。。
CODE:0049D9D8               retn
CODE:0049D9D8 sub_49D94C      endp ; sp = -210h
```

### 49d9e8

```
CODE:0049D9E8               push    ebp
CODE:0049D9E9               mov     ebp, esp
CODE:0049D9EB               add     esp, 0FFFFFDFCh
CODE:0049D9F1               push    ebx
CODE:0049D9F2               mov     [ebp+var_4], eax
CODE:0049D9F5               mov     eax, [ebp+var_4]
CODE:0049D9F8               call    sub_4040FC
CODE:0049D9FD               xor     eax, eax
CODE:0049D9FF               push    ebp
CODE:0049DA00               push    offset loc_49DA62
CODE:0049DA05               push    dword ptr fs:[eax]
CODE:0049DA08               mov     fs:[eax], esp
CODE:0049DA0B               lea     edx, [ebp+var_204]
CODE:0049DA11               mov     ecx, 1FFh
CODE:0049DA16               mov     eax, [ebp+var_4]
CODE:0049DA19               call    sub_49D880       //将输入密码当作 16 进制数解读，如‘11112222’被转换成 4 个字节：11 11 22 22，称之为密码 1
CODE:0049DA1E               lea     eax, [ebp+var_204]
CODE:0049DA24               push    eax
CODE:0049DA25               push    offset unk_4E3634
CODE:0049DA2A               mov     eax, [ebp+var_4]
CODE:0049DA2D               call    sub_403F48       //计算密码 1 的字节长度
CODE:0049DA32               sar     eax, 1           //计算密码 1 的字长度
CODE:0049DA34               jns     short loc_49DA39
CODE:0049DA36               adc     eax, 0
CODE:0049DA39
CODE:0049DA39 loc_49DA39:                  ; CODE XREF: sub_49D9E8+4Cj
CODE:0049DA39               push    eax
CODE:0049DA3A               push    offset unk_4DCD66
CODE:0049DA3F               push    offset unk_4DCE66
CODE:0049DA44               call    sub_49DFFA       //核心加密算法
CODE:0049DA49               movsx  ebx, al
CODE:0049DA4C               xor     eax, eax
CODE:0049DA4E               pop     edx
CODE:0049DA4F               pop     ecx
CODE:0049DA50               pop     ecx
CODE:0049DA51               mov     fs:[eax], edx
CODE:0049DA54               push    offset loc_49DA69
CODE:0049DA59
CODE:0049DA59 loc_49DA59:                  ; CODE XREF: CODE:0049DA67j
CODE:0049DA59               lea     eax, [ebp+var_4]
CODE:0049DA5C               call    sub_403CCC
CODE:0049DA61               retn
CODE:0049DA61 sub_49D9E8      endp ; sp = -210h
```

看到这，菜鸟就发晕了，用户名和密码 1 竟然都用相同算法(`49dffa`)加密，再比较其加密后的值，这不是要搞出逆算法才行吗？过分啊！

## 下面看一下 49dffa 的参数

`sub_49dffa(char * pBuffToCode, char *pDestBuff, int iLenBuffToCode, char *pBuff, char *pLongIntZ)`

当进行用户名加密时是：

`sub_49dffa('happy', 4e3434, 5, 4dcb62,4dcc62)`

当进行密码 1 加密时是：

`sub_49dffa( '11 11 22 22', 4e3634, 4, 4dcd66, 4dce66)`

其中 `4dce66` 空间算装内容与 `4dcc62` 完全相同，长度为 `50h` 个字节。`4dcd66` 与 `4dcb62` 空间内容不同。

下面我们进 `49dffa` 去看：

`sub_49dffa(char * pBuffToCode, char *pDestBuff, int iLenBuffToCode, char *pBuff, char *pLongIntZ)`

```
CODE:0049DFFA               push    ebp
CODE:0049DFFB               mov     ebp, esp
CODE:0049DFFD               push    ebx
CODE:0049DFFE               push    esi
CODE:0049DFFF               mov     esi, [ebp+arg_0]
CODE:0049E002               mov     ebx, [ebp+arg_8]
CODE:0049E005               push    esi
CODE:0049E006               call    sub_49F121   //返回 pLoingIntZ 指向字串的 bit 数，从 pLongIntZ+2a*2 的字的高比特往下找第一个非零 bit 位，这里返回 280
```



```
CODE:0049E00B                 pop     ecx   

CODE:0049E00C                 add     eax, 7   

CODE:0049E00F                 sar     eax, 3      //计算修正后 pLongIntZ 字节数，50   

CODE:0049E012                 movzx   edx, bx   

CODE:0049E015                 cmp     eax, edx      //比较 iLenBuffTocode 与 pLongIntZ 长度，小等于则继续    

                    //从算法角度来说，这个比较很有必要噢   

CODE:0049E017                 jge     short loc\_49E01E   

CODE:0049E019                 or      eax, 0FFFFFFFFh   

CODE:0049E01C                 jmp     short loc\_49E055   

CODE:0049E01E ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   

CODE:0049E01E   

CODE:0049E01E loc\_49E01E:                                ; CODE XREF: sub\_49DFFA+1Dj   

CODE:0049E01E                 push    ebx   

CODE:0049E01F                 push    offset unk\_4E3838   

CODE:0049E024                 push    [ebp+arg\_10]   

CODE:0049E027                 call    sub\_49DF4D   //将 pBufTofCode 拷到 4e3838   

CODE:0049E02C                 add     esp, 0Ch   

CODE:0049E02F                 push    esi   

CODE:0049E030                 push    [ebp+arg\_4]   

CODE:0049E033                 push    offset unk\_4E3838   

CODE:0049E038                 push    offset unk\_4E38B8   

CODE:0049E03D                 call    sub\_49F15D   //重点！   

CODE:0049E042                 add     esp, 10h   

CODE:0049E045                 push    [ebp+arg\_C]   

CODE:0049E048                 push    offset unk\_4E38B8   

CODE:0049E04D                 call    sub\_49DFBA   

CODE:0049E052                 add     esp, 8   

CODE:0049E055   

CODE:0049E055 loc\_49E055:                                ; CODE XREF: sub\_49DFFA+22j   

CODE:0049E055                 pop     esi   

CODE:0049E056                 pop     ebx   

CODE:0049E057                 pop     ebp   

CODE:0049E058                 retn    14h   

CODE:0049E058 sub\_49DFFA       endp   
```

下面跟进 `sub_49f15d(char *pDest, char *pSource, char *pBuff, char *pLongIntZ)`

`pDest==4e38b8`, `pSource==4e3838` ;

`pSource`, `pBuff`, `pLongIntZ` 说指向的数实际是超长正整数，和一般的 32bit 的正整数一样，低位在前，高位在后，

不同是的长度>4，输入参数最大字数只有 28，是 `pLongIntZ` 的长度，从而决定操作字窜长度不超过 28+2=2a，长度

值存在空间 `4dcf6a` 中。`sub_49e386` 用来 copy 字串，copy 字的长度一般是 2a。

```
sub\_49F15D       proc near               ; CODE XREF: sub\_49DFFA+43p   

CODE:0049F15D                                                  ; CODE:0049F3DEp ...   

CODE:0049F15D   

CODE:0049F15D var\_88           = byte ptr -88h   

CODE:0049F15D var\_6            = word ptr -6   

CODE:0049F15D var\_4            = dword ptr -4   

CODE:0049F15D pDest             = dword ptr  8   

CODE:0049F15D pSource           = dword ptr  0Ch   

CODE:0049F15D pBuff             = dword ptr  10h   

CODE:0049F15D pLongIntZ         = dword ptr  14h   

CODE:0049F15D   

CODE:0049F15D                 push    ebp   

CODE:0049F15E                 mov     ebp, esp   

CODE:0049F160                 add     esp, 0FFFFFF78h   

CODE:0049F166                 push    ebx   

CODE:0049F167                 push    esi   

CODE:0049F168                 push    edi   

CODE:0049F169                 mov     ebx, [ebp+pLongIntZ]   

CODE:0049F16C                 mov     edi, [ebp+pBuff]   

CODE:0049F16F                 mov     esi, [ebp+pDest]   

CODE:0049F172                 push    1   

CODE:0049F174                 push    esi   

CODE:0049F175                 call    sub\_49E3AC   

CODE:0049F17A                 add     esp, 8   

CODE:0049F17D                 cmp     word ptr [edi], 0 ;   

CODE:0049F17D                                                  ;   

CODE:0049F17D                                                  ;   

CODE:0049F181                 jnz     short loc\_49F1B7    -->跳   

。。。   

CODE:0049F1B7                 cmp     word ptr [ebx], 0   

CODE:0049F1BB                 jnz     short loc\_49F1D4    -->NZ, 跳   

。。。   

CODE:0049F1D4                 movsx   edx, ds:word\_4DCF68 ;   

CODE:0049F1D4                                                  ;   

CODE:0049F1DB                 cmp     word ptr [ebx+edx*2-2], 0   

CODE:0049F1E1                 jge     short loc\_49F1ED    --〉等 0，跳   

。。。   

CODE:0049F1ED ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   

CODE:0049F1ED   

CODE:0049F1ED loc\_49F1ED:                                ; CODE XREF: sub\_49F15D+84j   

CODE:0049F1ED                 push    ebx   

CODE:0049F1EE                 push    [ebp+pSource]   

CODE:0049F1F1                 call    sub\_49E2B7   //比较大小，pLongIntZ 必须大等于 pSource!   

CODE:0049F1F6                 add     esp, 8   

CODE:0049F1F9                 test    ax, ax   

CODE:0049F1FC                 jl      short loc\_49F208   

CODE:0049F1FE                 mov     eax, 0FFFFFFFDh   

CODE:0049F203                 jmp     loc\_49F33C   

CODE:0049F208 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   

CODE:0049F208   

CODE:0049F208 loc\_49F208:                                ; CODE XREF: sub\_49F15D+9Fj   

CODE:0049F208                 push    ebx   

CODE:0049F209                 push    edi   

CODE:0049F20A                 call    sub\_49E2B7   //比较大小，pLongIntZ 必须大等于 pBuff(why?)   

CODE:0049F20F                 add     esp, 8   

CODE:0049F212                 test    ax, ax   

CODE:0049F215                 jl      short loc\_49F221   

CODE:0049F217                 mov     eax, 0FFFFFFFCh   

CODE:0049F21C                 jmp     loc\_49F33C   

CODE:0049F221 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   

CODE:0049F221   

CODE:0049F221 loc\_49F221:                                ; CODE XREF: sub\_49F15D+B8j   

CODE:0049F221                 mov     dx, ds:word\_4DCF68   

CODE:0049F228                 mov     [ebp+var\_6], dx   

CODE:0049F22C                 push    ebx   

CODE:0049F22D                 call    sub\_49F121   //计算 pLongIntZ 的 bit 数，=280,     
    
CODE:0049F232                 pop     ecx   

CODE:0049F233                 add     eax, 20h      //为运算增加 20bits,   

CODE:0049F236                 sar     eax, 4      //计算需要的字数（280+20）/10   

CODE:0049F239                 mov     ds:word\_4DCF68, ax   //4DCF68=2A   

CODE:0049F23F                 push    ebx   

CODE:0049F240                 call    sub\_49E8B3   //重要初始化！！！！！   

CODE:0049F245                 pop     ecx   

CODE:0049F246                 test    eax, eax   

CODE:0049F248                 jz      short loc\_49F25F    --〉跳   

CODE:0049F24A                 mov     cx, [ebp+var\_6]   

。。。   

CODE:0049F25F ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   

CODE:0049F25F   

CODE:0049F25F loc\_49F25F:                                ; CODE XREF: sub\_49F15D+EBj   

CODE:0049F25F                 push    edi   

CODE:0049F260                 call    sub\_49E3DB   //计算 pBuff 所指向的空间的最高非零字的位置   

CODE:0049F265                 pop     ecx   

CODE:0049F266                 test    ax, ax   

CODE:0049F269                 jnz     short loc\_49F272    --〉nz, 跳   

CODE:0049F26B                 xor     eax, eax   

CODE:0049F26D                 jmp     loc\_49F33C   

CODE:0049F272 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   

CODE:0049F272   

CODE:0049F272 loc\_49F272:                                ; CODE XREF: sub\_49F15D+10Cj   

CODE:0049F272                 movsx   edx, ax   

CODE:0049F275                 shl     edx, 4   

CODE:0049F278                 mov     [ebp+var\_4], edx   ebp+var\_4 初始存放 pBuff 有效 bit 数，   

CODE:0049F27B                 movsx   ecx, ax   

CODE:0049F27E                 add     ecx, ecx   

CODE:0049F280                 add     edi, ecx   

CODE:0049F282                 add     edi, 0FFFFFFFEh   //移 pBuff 高位的第一个字到 edi,   

CODE:0049F285                 mov     bx, 8000h   

CODE:0049F289                 jmp     short loc\_49F291   

CODE:0049F28B ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   

CODE:0049F28B   

CODE:0049F28B loc\_49F28B:                                ; CODE XREF: sub\_49F15D+137j   

CODE:0049F28B                 shr     bx, 1   

CODE:0049F28E                 dec     [ebp+var\_4]   

CODE:0049F291   

CODE:0049F291 loc\_49F291:                                ; CODE XREF: sub\_49F15D+12Cj   

CODE:0049F291                 test    [edi], bx   
```



# 汇编代码分析与注释

## 代码片段 1：sub_49F15D 部分逻辑

```asm
CODE:0049F294              jz      short loc_49F28B

CODE:0049F296              dec     [ebp+var_4]    //找到了第一个非零 bit，当前 bit 位数减一

CODE:0049F299              push    [ebp+pSource]

CODE:0049F29C              push    esi

CODE:0049F29D              call    sub_49E386    //用 pSource 初始化 pDes

CODE:0049F2A2              add     esp, 8

CODE:0049F2A5              shr     bx, 1

CODE:0049F2A8              test    bx, bx

CODE:0049F2AB              jnz     short loc_49F30E --〉跳

CODE:0049F2AD              mov     bx, 8000h

CODE:0049F2B1              sub     edi, 2

CODE:0049F2B4              jmp     short loc_49F30E

CODE:0049F2B6 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

CODE:0049F2B6

CODE:0049F2B6 loc_49F2B6:                     ; CODE XREF: sub_49F15D+1BAj

CODE:0049F2B6              push    esi

CODE:0049F2B7              push    esi

CODE:0049F2B8              lea     eax, [ebp+var_88]

CODE:0049F2BE              push    eax

CODE:0049F2BF              call    sub_49E94F    //call sub_49e94f(tempBuf, pDest,pDest);！！！！！

CODE:0049F2C4              add     esp, 0Ch

CODE:0049F2C7              lea     edx, [ebp+var_88]

CODE:0049F2CD              push    edx

CODE:0049F2CE              push    esi

CODE:0049F2CF              call    sub_49E386    //copy tempBuf 到 pDest

CODE:0049F2D4              add     esp, 8

CODE:0049F2D7              test    [edi], bx        //pBuff 当前 bit 位的值是否为 1？

CODE:0049F2DA              jz      short loc_49F2FF

CODE:0049F2DC              push    [ebp+pSource]    //=1 继续

CODE:0049F2DF              push    esi

CODE:0049F2E0              lea     ecx, [ebp+var_88]

CODE:0049F2E6              push    ecx

CODE:0049F2E7              call    sub_49E94F    //call sub_49e94f(tempBuf, pDest, pSource);！！！！

        //这里竟然还有 pSource 作参数，菜鸟一度晕菜！

CODE:0049F2EC              add     esp, 0Ch

CODE:0049F2EF              lea     eax, [ebp+var_88]

CODE:0049F2F5              push    eax

CODE:0049F2F6              push    esi

CODE:0049F2F7              call    sub_49E386    //copy tempBuf 到 pDest

CODE:0049F2FC              add     esp, 8

CODE:0049F2FF

CODE:0049F2FF loc_49F2FF:                     ; CODE XREF: sub_49F15D+17Dj

CODE:0049F2FF              shr     bx, 1        //bx 移位

CODE:0049F302              test    bx, bx

CODE:0049F305              jnz     short loc_49F30E

CODE:0049F307              mov     bx, 8000h

CODE:0049F30B              sub     edi, 2        //edi 指向 pBuff 的当前字

CODE:0049F30E

CODE:0049F30E loc_49F30E:                     ; CODE XREF: sub_49F15D+14Ej

CODE:0049F30E                             ; sub_49F15D+157j ...

CODE:0049F30E              mov     eax, [ebp+var_4]

CODE:0049F311              add     [ebp+var_4], 0FFFFFFFFh    //当前 bit 位减一

CODE:0049F315              test    eax, eax          //遍历完 pBuff?

CODE:0049F317              jnz     short loc_49F2B6       --〉no 则跳

CODE:0049F319              push    0

CODE:0049F31B              lea     edx, [ebp+var_88]

CODE:0049F321              push    edx

CODE:0049F322              call    sub_49E3AC

CODE:0049F327              add     esp, 8

CODE:0049F32A              call    sub_49F0DA

CODE:0049F32F              mov     cx, [ebp+var_6]

CODE:0049F333              mov     ds:word_4DCF68, cx

CODE:0049F33A              xor     eax, eax

CODE:0049F33C

CODE:0049F33C loc_49F33C:                     ; CODE XREF: sub_49F15D+4Ej

CODE:0049F33C                             ; sub_49F15D+55j ...

CODE:0049F33C              pop     edi

CODE:0049F33D              pop     esi

CODE:0049F33E              pop     ebx

CODE:0049F33F              mov     esp, ebp

CODE:0049F341              pop     ebp

CODE:0049F342              retn

CODE:0049F342 sub_49F15D      endp

CODE:0049F342
```

## 逻辑说明

这里有两个 `call`，`sub_49E8B3` 和 `sub_49e94f`。

先看 `sub_49e8b3`，首先将 `pLongIntZ` 的地址存入 `4DD76C+0`。`4dd76c+4` 开始的空间存放了 16 个地址，那些地址指向的空间用来存放 `pLongIntZ*2^n` (`n=1` 到 `16`)，把 `pLongIntZ*2^n` (`n=0` 到 `16`) 的最高字节存到 `4DD7B0+2*n`，把 `pLongIntZ*2^n` (`n=0` 到 `16`) 的次高字节存到 `4DD7D2+2*n`，便于后面比较大小时使用。

`sub_49e94f`，是最核心的函数了，好长啊，从 `49e94f` 到 `49f0d9`，好在结构还比较简单，呵呵。

来看 `sub_49e94f(char *pTempDest, char *pLongIntX, char *pLongIntY)`

## 代码片段 2：sub_49E94F 函数定义

```asm
CODE:0049E94F sub_49E94F      proc near             ; CODE XREF: sub_49F15D+162p

CODE:0049E94F                             ; sub_49F15D+18Ap

CODE:0049E94F

CODE:0049E94F var_6          = word ptr -6

CODE:0049E94F var_4          = dword ptr -4

CODE:0049E94F arg_0          = dword ptr  8

CODE:0049E94F arg_4          = dword ptr  0Ch

CODE:0049E94F arg_8          = dword ptr  10h

CODE:0049E94F

CODE:0049E94F              push    ebp

CODE:0049E950              mov     ebp, esp

CODE:0049E952              add     esp, 0FFFFFFF8h

CODE:0049E955              push    ebx

CODE:0049E956              push    esi

CODE:0049E957              push    edi

CODE:0049E958              mov     edi, [ebp+arg_8]

CODE:0049E95B              mov     ebx, [ebp+arg_0]

CODE:0049E95E              push    [ebp+arg_4]

CODE:0049E961              push    offset dword_4DDF74

CODE:0049E966              call    sub_49E872    //该函数首先将 pLongIntX 的地址存入 4ddf74。

        4ddf74+4 开始的 15 个 DW 空间存放地址，这些地

        址指向的空间的值为 pLongIntX*2^n,n=1 到 15

CODE:0049E96B              add     esp, 8

CODE:0049E96E              movsx   eax, ds:word_4DCF68

CODE:0049E975              add     eax, eax

CODE:0049E977              add     eax, ebx

CODE:0049E979              add     eax, 0FFFFFFFEh

CODE:0049E97C              mov     [ebp+var_4], eax    //ebp+var4 保存 pTempDest 的最高字地址

CODE:0049E97F              mov     esi, [ebp+var_4]

CODE:0049E982              sub     esi, 2          //esi 保存 pTempDest 次高字地址

CODE:0049E985              push    0

CODE:0049E987              push    ebx

CODE:0049E988              call    sub_49E3AC    //初始化 ptempDest 指向的空间为 0

CODE:0049E98D              add     esp, 8

CODE:0049E990              push    edi

CODE:0049E991              call    sub_49E3DB    //计算 pLongIntY 的字长

CODE:0049E996              pop     ecx

CODE:0049E997              mov     [ebp+var_6], ax    //pLongIntY 字长存入 ebp+bar_6

CODE:0049E99B              cmp     [ebp+var_6], 0

CODE:0049E9A0              jnz     short loc_49E9A9    -->nz 跳

CODE:0049E9A2              xor     eax, eax

CODE:0049E9A4              jmp     loc_49F0D3

CODE:0049E9A9 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

CODE:0049E9A9

CODE:0049E9A9 loc_49E9A9:                     ; CODE XREF: sub_49E94F+51j

CODE:0049E9A9              movsx   edx, [ebp+var_6]

CODE:0049E9AD              add     edx, edx

CODE:0049E9AF              add     edi, edx

CODE:0049E9B1              add     edi, 0FFFFFFFEh    //edi 保存 pLongIntY 的当前字的地址

CODE:0049E9B4              jmp     loc_49F0BF    --〉跳

CODE:0049E9B9 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

CODE:0049E9B9

CODE:0049E9B9 loc_49E9B9:                     ; CODE XREF: sub_49E94F+77Cj

CODE:0049E9B9              push    ebx

CODE:0049E9BA              call    sub_49E83C            //将 pTempDest 移高一个字，也就是

                    将 pTempDest*2¹⁶(这步非常重要噢！）

CODE:0049E9BF              pop     ecx

CODE:0049E9C0              test    byte ptr [edi+1], 80h    //比较 pLongIntY 当前字的第 15bit,
```



## 代码片段

```
CODE:0049E9C4                jz      short loc_49E9D7   

CODE:0049E9C6                push    0   

CODE:0049E9C8                push    ds:off_4DDFB0      //ds:off_4DDFB0 存有 pLongIntX*2¹⁵ 的指针    

CODE:0049E9CE                push    ebx   

CODE:0049E9CF                call    sub_49E199         //两超长正整数相加，存回 pTempDest，    
                                                         //字节间的进位处理还晃了本菜一下，呵    

CODE:0049E9D4                add     esp, 0Ch   

CODE:0049E9D7 

CODE:0049E9D7 loc_49E9D7:    ; CODE XREF: sub_49E94F+75j   

......      

CODE:0049EB12 

CODE:0049EB12 loc_49EB12:    ; CODE XREF: sub_49E94F+1B0j   

CODE:0049EB12                test    byte ptr [edi], 1   

CODE:0049EB15                jz      short loc_49EB28   

CODE:0049EB17                push    0   

CODE:0049EB19                push    ds:dword_4DDF74   

CODE:0049EB1F                push    ebx   

CODE:0049EB20                call    sub_49E199   

CODE:0049EB25                add     esp, 0Ch   

CODE:0049EB28 

//以上对 edi+1，和 edi 的各个非零比特，将 pTempDest 加上相应的 pLongIntX*2^n   

CODE:0049EB28 loc_49EB28:    ; CODE XREF: sub_49E94F+1C6j   

CODE:0049EB28                mov     ecx, [ebp+var_4]   

CODE:0049EB2B                mov     ax, [ecx]          //ax=pTempDest 的最高字的值    

CODE:0049EB2E                sub     ax, ds:word_4DD7D0 //4dd7d0 保存了 pLongIntZ*2¹⁶   

                                                         //的最高字的值   

CODE:0049EB35                test    ax, ax   

CODE:0049EB38                jg      short loc_49EB6B   //pTempDest 大就跳到 49eb6b    

CODE:0049EB3A                test    ax, ax   

CODE:0049EB3D                jnz     short loc_49EB7C   

CODE:0049EB3F                mov     dx, [esi]          //dx=pTempDest 的次高字的值    

CODE:0049EB42                cmp     dx, ds:word_4DD7F2 //4dd7d0 保存了 pLongIntZ*2¹⁶   

                                                         //的次高字的值   

CODE:0049EB49                ja      short loc_49EB6B   //pTempDest 大就跳到 49eb6b    

CODE:0049EB4B                mov     cx, [esi]   

CODE:0049EB4E                cmp     cx, ds:word_4DD7F2   

CODE:0049EB55                jnz     short loc_49EB7C   //小于就跳过 49eb6b   

CODE:0049EB57                push    ds:off_4DD7AC   

CODE:0049EB5D                push    ebx   

CODE:0049EB5E                call    sub_49E2B7         //比较 pTempDest 与 pLongIntZ*2¹⁶ 大小    

CODE:0049EB63                add     esp, 8   

CODE:0049EB66                test    ax, ax   

CODE:0049EB69                jl      short loc_49EB7C   //小于就跳走    

CODE:0049EB6B 

CODE:0049EB6B loc_49EB6B:    ; CODE XREF: sub_49E94F+1E9j   

CODE:0049EB6B                  ; sub_49E94F+1FAj   

CODE:0049EB6B                push    0   

CODE:0049EB6D                push    ds:off_4DD7AC   

CODE:0049EB73                push    ebx   

CODE:0049EB74                call    sub_49E1E6         //将 pTempDest 指向的值减去 pLongIntZ*2¹⁶    

CODE:0049EB79                add     esp, 0Ch   

。。。。。。   

CODE:0049F068 

CODE:0049F068 loc_49F068:    ; CODE XREF: sub_49E94F+6DAj   

CODE:0049F068                  ; sub_49E94F+6F2j ...   

CODE:0049F068                mov     eax, [ebp+var_4]   

CODE:0049F06B                mov     ax, [eax]   

CODE:0049F06E                sub     ax, ds:word_4DD7B0   

CODE:0049F075                test    ax, ax   

CODE:0049F078                jg      short loc_49F0AB   

CODE:0049F07A                test    ax, ax   

CODE:0049F07D                jnz     short loc_49F0BC   

CODE:0049F07F                mov     dx, [esi]   

CODE:0049F082                cmp     dx, ds:word_4DD7D2   

CODE:0049F089                ja      short loc_49F0AB   

CODE:0049F08B                mov     cx, [esi]   

CODE:0049F08E                cmp     cx, ds:word_4DD7D2   

CODE:0049F095                jnz     short loc_49F0BC   

CODE:0049F097                push    ds:dword_4DD76C   

CODE:0049F09D                push    ebx   

CODE:0049F09E                call    sub_49E2B7   

CODE:0049F0A3                add     esp, 8   

CODE:0049F0A6                test    ax, ax   

CODE:0049F0A9                jl      short loc_49F0BC   

CODE:0049F0AB 

CODE:0049F0AB loc_49F0AB:    ; CODE XREF: sub_49E94F+729j   

CODE:0049F0AB                  ; sub_49E94F+73Aj   

CODE:0049F0AB                push    0   

CODE:0049F0AD                push    ds:dword_4DD76C   

CODE:0049F0B3                push    ebx   

CODE:0049F0B4                call    sub_49E1E6   

CODE:0049F0B9                add     esp, 0Ch   

//以上依次将 pTempDest 与 pLongIntZ*2^n（n=16 到 0）比较大小，大等则将 pTempDest 减去 pLongIntZ*2^n   

//看出这实值是什么了吗？是将 pTempDest mod(pLongIntZ)啊!!!   

CODE:0049F0BC 

CODE:0049F0BC loc_49F0BC:    ; CODE XREF: sub_49E94F+72Ej   

CODE:0049F0BC                  ; sub_49E94F+746j ...   

CODE:0049F0BC                sub     edi, 2             //下移 pLongIntY 的当前字     
    
CODE:0049F0BF 

CODE:0049F0BF loc_49F0BF:    ; CODE XREF: sub_49E94F+65j   

CODE:0049F0BF                mov     ax, [ebp+var_6]   

CODE:0049F0C3                add     [ebp+var_6], 0FFFFh   

CODE:0049F0C8                test    ax, ax             //pLongIntY 的字都读完了吗？   

CODE:0049F0CB                jnz     loc_49E9B9         --〉nz 跳   

CODE:0049F0D1                xor     eax, eax   

CODE:0049F0D3 

CODE:0049F0D3 loc_49F0D3:    ; CODE XREF: sub_49E94F+55j   

CODE:0049F0D3                pop     edi   

CODE:0049F0D4                pop     esi   

CODE:0049F0D5                pop     ebx   

CODE:0049F0D6                pop     ecx   

CODE:0049F0D7                pop     ecx   

CODE:0049F0D8                pop     ebp   

CODE:0049F0D9                retn   

CODE:0049F0D9 sub_49E94F      endp   
```

看出来 `sub_49e94f(char *ptempDest, char *pLongIntX, char *pLongIntY)` 做了什么吗？说出来

简单 `s` 了，就是 `(*pTempDest)=((*pLongIntX)*(*pLongIntY))mod (*pLongIntZ)` 啊！具体推理

菜鸟就不写了，手写不动了啊:) 各位应该能推得出来，不然要加油噢。

哇，菜鸟到这快写完了噢，开心啊:) 各位是不是看得也有点累了？呵呵

OK! 快要胜利了噢，现在结合 `sub_49F15D` 伪代码来看看就究竟是怎么回事吧！

### 伪代码分析

```
sub_49f15d(char *pDest, char *pSource, char *pBuff, char *pLongIntZ)

{
  初始化 pLongIntZ*2^n(n=1 到 16，其实可以认为 n=0 到 16）   

  从高位到底位找到 pBuff 的第一个非 0 比特位，初始化（*pDest）=（*pSource);   

  对此后从高到底的 pBuff 的每个比特位   

  {
    sub_49e94f(tempBuf, pDest,pDest);！   

    (*pDest)=(*tempBuf);//这两行实际上是(*pDest)=(*pDest)²  mod (*pLongIntZ);   

    if(当前 bit 位值==1）   

    {
      sub_49e94f(tempBuf, pDest,pSource);！   

      (*pDest)=(*tempBuf);//这两行实际上是(*pDest)=(*pDest)*(*pSource)  mod (*pLongIntZ);   

                            这里的 pSource 前期一度让偶认为无法求出逆算法         

    }
  }

}
```

实质上， 最终 `(*pDest)=(*pSource)^m mod (*pLongIntZ)` , `m` 是由 `pBuff` 决定的一个常数！偶用 `*pSorce=1`

试了一下， `(*pDest)` 果然等于 `1`，哈哈。。。还试了一个 `*pSource=10`, `*pBuff=07`, 这时

`*Dest=00 00 00 10`，perfect！哈！

对于偶的例子，就是要求

当

```
sub_49f15d(char *pDest1, "happy", 4dcb62, 4dcc62)

sub_49f15d(char *pDest2, '11 11 22 22', 4dcb66, 4dce66)

*4dcc62=*4dce66
```

时，要求

1） `*pDest1` 的位数=`*pDest2` 的位数，

2） `*pDest1` 的第 8 位开始的 DW 等于 `*pDest2` 的第 8 位开始的 DW。

3） `*pDest1` 的第 0B 位开始的字节和等于 `*pDest2` 的第 0B 位开始的字节和。

嗯，一切都很明了了吧！可是对于 `x^m mod Z =y`, `m,Z` 常数，知道 `y`，怎么求 `x`? 菜鸟不会啊！应该有什么公式吧？不知道和 RSA 算法有没有什么关系？不过，本菜鸟被这破程序折腾得够呛啊，实在是不想再动了。好了，就到这里吧！`IP_TOOLS`，go to hell！呵呵。。。这是本菜第一次写关于破解的东东，没有什么经验，希望各位没有看得晕菜过去噢，呵

半点星

02/02/13

### 预处理代码片段

```
:004ACB2D 8B45F8                mov eax, dword ptr [ebp-08]   

:004ACB30 8D55FC                lea edx, dword ptr [ebp-04]   

:004ACB33 E850B0F5FF            call 00407B88====>用户名[进行预处理,去掉前后的空格]   

                                                 =======>相当于 Java 中的 Trim   
```



```
:004ACB38 8B55FC                 mov edx, dword ptr [ebp-04]

:004ACB3B 8BC7                   mov eax, edi

:004ACB3D E8E26BF7FF             call 00423724

:004ACB42 8D55F8                 lea edx, dword ptr [ebp-08]

:004ACB45 8B06                   mov eax, dword ptr [esi]

:004ACB47 8BB8F4010000           mov edi, dword ptr [eax+000001F4]

:004ACB4D 8BC7                   mov eax, edi

:004ACB4F E8A06BF7FF             call 004236F4

:004ACB54 8B45F8                 mov eax, dword ptr [ebp-08]

:004ACB57 8D55FC                 lea edx, dword ptr [ebp-04]

:004ACB5A E829B0F5FF             call 00407B88====>注册码[进行预处理，处理同用户名的处理]

:004ACB5F 8B55FC                 mov edx, dword ptr [ebp-04]

:004ACB62 8BC7                   mov eax, edi

:004ACB64 E8BB6BF7FF             call 00423724

:004ACB69 8D55F8                 lea edx, dword ptr [ebp-08]

:004ACB6C 8B06                   mov eax, dword ptr [esi]

:004ACB6E 8B80F0010000           mov eax, dword ptr [eax+000001F0]

:004ACB74 E87B6BF7FF             call 004236F4

:004ACB79 8B45F8                 mov eax, dword ptr [ebp-08]

:004ACB7C E8CF8AFEFF             call 00495650====>用户名黑名单检查

:004ACB81 84C0                   test al, al========>al 必须为 0

:004ACB83 743C                   je 004ACBC1

\==============>Black List[黑名单]<=================

'Asteriks Asteriks'
'Ruslan Raymond Morris'
'Elizabeth Wilcox'
'kathleen pelikan'
'LEUNG KAM FUNG EVA'
'BRYAN MCBRIDE'
'Wravldtrebra CMeafnvnsianwg'
'BRUSZAZ'
'Mfagrftdisne AVtatcyeuc'
'vcivrbgnimneihac shratawg'
'Duayntireelw AHtidlvl'
'Dmanvbivdc XCsadurdtlye'
'GAAESGPCATRHIUNJ TSGPYAHREIDN'
'Esldafignhe'
'Dsatvhivdt sStcnhumdesn'
'Ttihm7odtbhiys AWheclylss'
'Cclrafusdcee JPuoatyvviun'
'Wtavrurdesnt nScaedgltedr'
'Gcefotrgghej kTjhhogmfpdssorn'
'srybfytjusmc tsnyifkthuvm'

\=============>End Of Black List<================

\* Possible StringData Ref from Code Obj ->"Sorry, your registration name "
                                        ->"("
                                        |

:004ACB85 68F4CD4A00             push 004ACDF4

:004ACB8A 8D55F8                 lea edx, dword ptr [ebp-08]

:004ACB8D 8B06                   mov eax, dword ptr [esi]

:004ACB8F 8B80F0010000           mov eax, dword ptr [eax+000001F0]

:004ACB95 E85A6BF7FF             call 004236F4

:004ACB9A FF75F8                 push [ebp-08]

\* Possible StringData Ref from Code Obj ->") is found on the "Black List".

                                        |

:004ACB9D 681CCE4A00             push 004ACE1C

\* Possible StringData Ref from Code Obj ->"If you have any questions, please, "
                                        ->"contact KS-Soft: line1@ks-soft.net; "
                                        ->"line2@ks-soft.net"
                                        |

:004ACBA2 6848CE4A00             push 004ACE48

:004ACBA7 8D45FC                 lea eax, dword ptr [ebp-04]

:004ACBAA BA04000000             mov edx, 00000004

:004ACBAF E85474F5FF             call 00404008

:004ACBB4 8B45FC                 mov eax, dword ptr [ebp-04]

:004ACBB7 E8E480F9FF             call 00444CA0

:004ACBBC E9EF010000             jmp 004ACDB0

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004ACB83(C)

\========>Come Here,Here is right!!!======>

:004ACBC1 8D55F8                 lea edx, dword ptr [ebp-08]

:004ACBC4 8B06                   mov eax, dword ptr [esi]

:004ACBC6 8B80F0010000           mov eax, dword ptr [eax+000001F0]

:004ACBCC E8236BF7FF             call 004236F4

:004ACBD1 837DF800               cmp dword ptr [ebp-08], 00000000

:004ACBD5 0F84CB010000           je 004ACDA6=====>用户名是否存在

:004ACBDB 8D55F4                 lea edx, dword ptr [ebp-0C]

:004ACBDE 8B06                   mov eax, dword ptr [esi]

:004ACBE0 8B80F4010000           mov eax, dword ptr [eax+000001F4]

:004ACBE6 E8096BF7FF             call 004236F4

:004ACBEB 837DF400               cmp dword ptr [ebp-0C], 00000000

:004ACBEF 0F84B1010000           je 004ACDA6=====>注册码是否存在

:004ACBF5 8D55F0                 lea edx, dword ptr [ebp-10]

:004ACBF8 8B06                   mov eax, dword ptr [esi]

:004ACBFA 8B80F0010000           mov eax, dword ptr [eax+000001F0]

:004ACC00 E8EF6AF7FF             call 004236F4

:004ACC05 8B45F0                 mov eax, dword ptr [ebp-10]

:004ACC08 E80713FFFF             call 0049DF14====>用户名[有效长度 0x40]，并产生 Buf2

:004ACC0D 8BF8                   mov edi, eax

\===========================
$50=length(Name)*8;
$00=length(Name) div 32;
\===========================

:004ACC0F 8D55F0                 lea edx, dword ptr [ebp-10]

:004ACC12 8B06                   mov eax, dword ptr [esi]

:004ACC14 8B80F4010000           mov eax, dword ptr [eax+000001F4]

:004ACC1A E8D56AF7FF             call 004236F4

:004ACC1F 8B45F0                 mov eax, dword ptr [ebp-10]

:004ACC22 E88913FFFF             call 0049DFB0====>注册码[必须是十六进制的字符串]
                                        ========>中间不能有空格
                                        ========>并产生 Buf1

:004ACC27 663BF8                 cmp di, ax========>??是不是长度比较呢??

:004ACC2A 0F8576010000           jne 004ACDA6

:004ACC30 A198224E00             mov eax, dword ptr [004E2298]

:004ACC35 BAFF010000             mov edx, 000001FF

:004ACC3A E8B912FFFF             call 0049DEF8=====>累加和 Buf1[0..$1FF]

:004ACC3F 8BF8                   mov edi, eax

:004ACC41 A19C214E00             mov eax, dword ptr [004E219C]

:004ACC46 BAFF010000             mov edx, 000001FF

:004ACC4B E8A812FFFF             call 0049DEF8=====>累加和 Buf2[0..$1FF]

:004ACC50 3BF8                   cmp edi, eax

:004ACC52 0F854E010000           jne 004ACDA6

:004ACC58 A198224E00             mov eax, dword ptr [004E2298]

:004ACC5D 83C007                 add eax, 00000007

:004ACC60 8B00                   mov eax, dword ptr [eax]

:004ACC62 8B159C214E00           mov edx, dword ptr [004E219C]

:004ACC68 83C207                 add edx, 00000007

:004ACC6B 3B02                   cmp eax, dword ptr [edx]====>要求第 7 开始的双字必须相等

:004ACC6D 0F8533010000           jne 004ACDA6======

:004ACC73 8D55F8                 lea edx, dword ptr [ebp-08]

:004ACC76 8B06                   mov eax, dword ptr [esi]
```
