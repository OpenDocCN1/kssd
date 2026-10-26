
```text
0044C044  |.  E8 9778FCFF   |call    004138E0                        ;  取上述转换后结果，记为E
0044C049  |.  3945 EC       |cmp     [ebp-14], eax
0044C04C  |.  7D 4B         |jge     short 0044C099
0044C04E  |>  8B45 EC       |/mov     eax, [ebp-14]
0044C051  |.  50            ||push    eax                            ; /Arg1
0044C052  |.  8D4D E4       ||lea     ecx, [ebp-1C]                  ; |
0044C055  |.  E8 C678FCFF   ||call    00413920                       ; \取E
0044C05A  |.  0FBEC8        ||movsx   ecx, al                        ;  E各字符ASCII值逐位至ECX
0044C05D  |.  83F9 41       ||cmp     ecx, 41                        ;  跟41比较
0044C060  |.  7C 14         ||jl      short 0044C076                 ;  小则进入下一处理过程（ASCII值加1B直至大于等于41）
0044C062  |.  8B45 EC       ||mov     eax, [ebp-14]
0044C065  |.  50            ||push    eax                            ; /Arg1
0044C066  |.  8D4D E4       ||lea     ecx, [ebp-1C]                  ; |
0044C069  |.  E8 B278FCFF   ||call    00413920                       ; \取E（加1B处理后的E）
0044C06E  |.  0FBEC8        ||movsx   ecx, al                        ;  E各字符ASCII值逐位至ECX
0044C071  |.  83F9 5A       ||cmp     ecx, 5A                        ;  跟5A比较
0044C074  |.  7E 21         ||jle     short 0044C097
0044C076  |>  8B45 EC       ||mov     eax, [ebp-14]
0044C079  |.  50            ||push    eax                            ; /Arg1
0044C07A  |.  8D4D E4       ||lea     ecx, [ebp-1C]                  ; |
0044C07D  |.  E8 9E78FCFF   ||call    00413920                       ; \取E
0044C082  |.  0FBEC8        ||movsx   ecx, al                        ;  E各字符ASCII值逐位至ECX
0044C085  |.  83C1 1B       ||add     ecx, 1B                        ;  加1B
0044C088  |.  51            ||push    ecx
0044C089  |.  8B55 EC       ||mov     edx, [ebp-14]
0044C08C  |.  52            ||push    edx
0044C08D  |.  8D4D E4       ||lea     ecx, [ebp-1C]
0044C090  |.  E8 D3F90A00   ||call    004FBA68
0044C095  |.^ EB B7         |\jmp     short 0044C04E
0044C097  |>^ EB 9F         \jmp     short 0044C038
0044C099  |>  8D45 E4       lea     eax, [ebp-1C]                    ;  最终结果（真码2）保存于EAX所指向的内存处
0044C09C  |.  50            push    eax
0044C09D  |.  8B4D 08       mov     ecx, [ebp+8]
0044C0A0  |.  E8 86F10A00   call    004FB22B
0044C0A5  |.  8B4D E0       mov     ecx, [ebp-20]
0044C0A8  |.  83C9 01       or      ecx, 1
0044C0AB  |.  894D E0       mov     [ebp-20], ecx
0044C0AE  |.  C645 FC 01    mov     byte ptr [ebp-4], 1
0044C0B2  |.  8D4D E4       lea     ecx, [ebp-1C]
0044C0B5  |.  E8 FCF30A00   call    004FB4B6
0044C0BA  |.  C645 FC 00    mov     byte ptr [ebp-4], 0
0044C0BE  |.  8D4D 0C       lea     ecx, [ebp+C]
0044C0C1  |.  E8 F0F30A00   call    004FB4B6
0044C0C6  |.  8B45 08       mov     eax, [ebp+8]
0044C0C9  |>  8B4D F4       mov     ecx, [ebp-C]
0044C0CC  |.  64:890D 00000>mov     fs:[0], ecx
0044C0D3  |.  5F            pop     edi
0044C0D4  |.  5E            pop     esi
0044C0D5  |.  5B            pop     ebx
0044C0D6  |.  8BE5          mov     esp, ebp
0044C0D8  |.  5D            pop     ebp
0044C0D9  \.  C2 0800       retn    8
```

---

## 破解总结

1. 注册失败提示未以明文直接出现，而以各字符ASCII值出现，增加了破解难度；
2. 存在注册黑名单，如用户名与黑名单中用户名一致，则注册失败；
3. 用户名长度不同，则有不同的注册算法，注册名长度等于小于7，采用注册算法（2），注册名长度大于7，采用注册算法（3）（当注册名长度等于小于7，固定字符串`UXDCOLFGHEWZ`可做通用注册码：），当注册名长度大于7，真码1，真码2均可注册成功，是否有暗桩，未及测试）；
4. 注册信息保存于注册表中 `[HKEY_CURRENT_USER\Software\Photoplorer\Photoplorer\Settings]`

---

## 版权声明

本文系作者原创, 转载请注明作者并保持文章的完整, 谢谢!

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
