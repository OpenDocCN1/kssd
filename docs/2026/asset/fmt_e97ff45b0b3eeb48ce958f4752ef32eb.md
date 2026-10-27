
```  
007069D6     8B85 5037AD0A        mov eax,dword ptr ss:[ebp+0xAAD3750]       ; 用到了ecx存放的偏移  
007069DC     FC                   cld  
007069DD     D1E0                 shl eax,1  
007069DF     0F89 09000000        jns v2_3_5_1.007069EE  
007069E5     60                   pushad  
007069E6     66:8BD8              mov bx,ax  
007069E9     66:BF 6693           mov di,0x9366  
007069ED     61                   popad  
007069EE     0385 FC57C70A        add eax,dword ptr ss:[ebp+0xAC757FC]       ; 这里边存放的是对应模块的基址这个地方应该是定位的对应模块的导出表  
007069F4     0F8B 1B000000        jpo v2_3_5_1.00706A15  
00706A4A     31F0                 xor eax,esi                                ; winmm.76B11AD6  
00706A4C     31C6                 xor esi,eax  
00706A4E     31F0                 xor eax,esi                                ; xchg eax,esi  
00706A50     E9 0A000000          jmp v2_3_5_1.00706A5F  
00706A55     4A                   dec edx  
00706A56     AC                   lods byte ptr ds:[esi]  
00706A57     71 07                jno short v2_3_5_1.00706A60  
00706A59     93                   xchg eax,ebx  
00706A5A   ^ 75 CA                jnz short v2_3_5_1.00706A26  
00706A5C     86D3                 xchg bl,dl  
00706A5E     630F                 arpl word ptr ds:[edi],cx  
00706A60     8311 00              adc dword ptr ds:[ecx],0x0  
00706A63     0000                 add byte ptr ds:[eax],al  
00706A65     0F8F 0B000000        jg v2_3_5_1.00706A76  
00706A6B     E9 06000000          jmp v2_3_5_1.00706A76  
00706A70     3e:be cca462de       mov esi,0xde62a4cc  
00706A76     66:AD                lods word ptr ds:[esi]                     ; 拿到导出函数序号  
00706A90     C1E0 02              shl eax,0x2  
00706A93     60                   pushad  
00706A94     80D2 C7              adc dl,0xC7  
00706A97     B4 FE                mov ah,0xFE  
00706A99     61                   popad  
00706A9A     0385 49F7C70A        add eax,dword ptr ss:[ebp+0xAC7F749]       ; winmm.76B11384  
00706AA0     FC                   cld  
00706AA1     50                   push eax  
00706AA2     81EC 04000000        sub esp,0x4  
00706AA8     893424               mov dword ptr ss:[esp],esi                 ; winmm.76B11518  
00706AAB     58                   pop eax                                    ; v2_3_5_1.006F90F3  
00706AAC     5E                   pop esi                                    ; v2_3_5_1.006F90F3  
00706AAD     0F8F 14000000        jg v2_3_5_1.00706AC7                       ; xchg eax,esi  
00706AC7     AD                   lods dword ptr ds:[esi]                    ; 根据导出序号拿到了API得偏移  
00706AED     81C1 8FD0C722        add ecx,0x22C7D08F  
00706AF3     5F                   pop edi                                    ; v2_3_5_1.006F90F3  
00706AF4     61                   popad  
00706AF5     0F88 01000000        js v2_3_5_1.00706AFC  
00706AFB     F8                   clc  
00706AFC     05 C2237533          add eax,0x337523C2  
00706B01     2D CB28C32D          sub eax,0x2DC328CB  
00706B06     01C8                 add eax,ecx                                ; winmm.76B10000  
00706B08     05 CB28C32D          add eax,0x2DC328CB  
00706B0D     2D C2237533          sub eax,0x337523C2  
00706B12     F9                   stc                                        ; 这一大段的意思是去现在装载的模块中根据导出序号找到偏移然后找到函数  
00706B13     8138 E8000000        cmp dword ptr ds:[eax],0xE8                ; 比较开头是不是E8  
00706C77     8138 6970686C        cmp dword ptr ds:[eax],0x6C687069          ; 比较的硬编码  
```

往下会有这么几句指令，我把它摘出来：

```  
00706DBC     8B9D 2D17AD0A        mov ebx,dword ptr ss:[ebp+0xAAD172D]       ; kernel32.7C800000  
00706DDD     29CB                 sub ebx,ecx                                ; winmm.76B10000  
```

这样的指令应该是检测模块位置是否填充正确，我对这个也不确定，因为这些都不影响脱壳。

```  
00706E6C     8D9D 608ECD0A        lea ebx,dword ptr ss:[ebp+0xACD8E60]  
00706E72     E9 08000000          jmp v2_3_5_1.00706E7F  
00706E77     76 51                jbe short v2_3_5_1.00706ECA  
00706E79     006F 00              add byte ptr ds:[edi],ch  
00706E7C     D216                 rcl byte ptr ds:[esi],cl  
00706E7E     46                   inc esi                                    ; v2_3_5_1.006F90F3  
00706E7F     FFD3                 call ebx                                   ; 通过这个call 确定API的正确性  
00706E81     60                   pushad                                     ; 它是通过判断函数头几个字节  
00706EA1     61                   popad                                      ; 覆盖存放的上一个API地址  
00706EA2     8985 C426AD0A        mov dword ptr ss:[ebp+0xAAD26C4],eax       ; winmm.midiOutPrepareHeader  
```

这时候取得是第二个元素：

```  
00709452     AD                   lods dword ptr ds:[esi]                    ; 取第二个元素  
0070946C     893C24               mov dword ptr ss:[esp],edi  
0070946F     BF 6D4D6663          mov edi,0x63664D6D  
00709474     81E7 EE30706C        and edi,0x6C7030EE  
0070947A     C1E7 08              shl edi,0x8  
0070947D     81C7 C779DE03        add edi,0x3DE79C7  
00709483     81F7 C7E5DE63        xor edi,0x63DEE5C7  
00709489     897E FC              mov dword ptr ds:[esi-0x4],edi             ; 第二个元素清零  
0070948C     5F                   pop edi                                    ; winmm.midiOutPrepareHeader  
0070956B     51                   push ecx                                   ; winmm.midiOutPrepareHeader  
0070956C     B9 9FF0C70D          mov ecx,0xDC7F09F  
00709571     05 E22F547B          add eax,0x7B542FE2  
00709576     01C8                 add eax,ecx                                ; winmm.midiOutPrepareHeader  
00709578     2D E22F547B          sub eax,0x7B542FE2  
0070957D     59                   pop ecx                                    ; 这一段执行完成就会拿到IAT的偏移  
0070957E     0F8A 05000000        jpe v2_3_5_1.00709589  
00709584     60                   pushad  
00709585     F9                   stc  
00709586     60                   pushad  
00709587     61                   popad  
00709588     61                   popad  
00709589     0385 550EAD0A        add eax,dword ptr ss:[ebp+0xAAD0E55]       ; 拿到代码段中IAT的基址+偏移  
0070958F     FC                   cld  
00709590     8B8D C426AD0A        mov ecx,dword ptr ss:[ebp+0xAAD26C4]       ; winmm.midiOutPrepareHeader  
007095BB     83EC 04              sub esp,0x4  
007095BE     891424               mov dword ptr ss:[esp],edx                 ; v2_3_5_1.00487604  
007095C1     BA 00000000          mov edx,0x0  
007095C6     01C2                 add edx,eax                                ; v2_3_5_1.00487604  
007095C8     890A                 mov dword ptr ds:[edx],ecx                 ; winmm.midiOutPrepareHeader  
007095CA     5A                   pop edx                                    ; 第二个元素还是去IAT填充API这里的IAT 的基址是487600  
```

到这里就跟文章头部呼应起来了，大体流程就是这些，这样一份Themida加密`IAT`的流程就分析完成了。当我们拉到下边，找到最后一个`EEEEEEEEDDDDDDD`结构，下一个内存访问断点，一直单步到这个地方：

```  
00706420    /E9 BF380000          jmp v2_3_5_1.00709CE4                      ; 运行到这可以直接代码段下内存访问断点到达OEP  
```

代码段内存访问断点以后，`shift+F9`。

到这，直接就到了`OEP`：

```  
00466C98     55                   push ebp  
00466C99     8BEC                 mov ebp,esp  
00466C9B     6A FF                push -0x1  
00466C9D     68 283A4B00          push v2_3_5_1.004B3A28  
00466CA2     68 749A4600          push v2_3_5_1.00469A74  
00466CA7     64:A1 00000000       mov eax,dword ptr fs:[0]  
00466CAD     50                   push eax  
00466CAE     64:8925 00000000     mov dword ptr fs:[0],esp  
00466CB5     83EC 58              sub esp,0x58  
00466CB8     53                   push ebx                                   ; v2_3_5_1.00708D28  
00466CB9     56                   push esi                                   ; v2_3_5_1.0070ACF3  
00466CBA     57                   push edi  
```

但是这个Sure大可能是没有加虚拟机选项之类的保护，也没有`StolenCode`，所以脱壳还是比较轻松的。

到这里手动拖也可以，脚本脱也行了，就不赘述了。

最后我只放上Sure大的加壳程序，毕竟是从人家那里学的，还是要尊重原创嘛。

因为还不是很会写脚本，写出来烂乎乎的，还是用的Sure大的。

最后就是最近看这三个版本的Themida，感觉对于TMD来说，不去看Anti之类的东西，核心技术好像就是这个`[esi]`表了，掌握了这个表，对于脱壳来说是事半功倍的。

新人贴，如有不对，望指正。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
