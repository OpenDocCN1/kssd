
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
