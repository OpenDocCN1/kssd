
```asm

;

; 修复 IAT

;

pusha



mov     esi, c_start[ebp]

mov     edi, c_end[ebp]



@msg    "* emulating IAT decryptions"



.WHILE  esi<edi



        push    esi;防止漏掉api



        .IF     w [esi] == 15FFh || \   ; JMP D[]
                w [esi] == 25FFh || \   ; CALL D[]
                w [esi] == 058Bh || \   ; MOV EAX, []
                w [esi] == 158Bh || \   ; MOV EDX, []
                w [esi] == 0D8Bh || \   ; MOV ECX, []
                w [esi] == 1D8Bh || \   ; MOV EBX, []
                w [esi] == 358Bh || \   ; MOV ESI, []
                w [esi] == 3D8Bh || \   ; MOV EDI, []
                w [esi] == 2D8Bh        ; MOV EBP, []



                lodsw   ; 跳过2个字节
                jmp     fixiat



        .ELSEIF b [esi] == 0A1h



                lodsb   ; 跳过1个字节



fixiat:                                 lodsd



                pusha
                xchg    esi, eax        ; ESI = IAT地址



                push    4
                push    esi
                callX   IsBadReadPtr
                .IF     eax == 0



                        mov     edi, [esi]



                        push    5*5
                        push    edi
                        callX   IsBadReadPtr
                        .IF     eax == 0



                                ;01150000    68 9F6E2619     PUSH    19266E9F
                                ;01150005    68 816B2919     PUSH    19296B81
                                ;0115000A    68 D61EC786     PUSH    86C71ED6
                                ;0115000F    68 01028819     PUSH    19880201
                                ;01150014    E8 E7FFF9FF     CALL    010F0000



                                .IF b [edi] == 68h && \
                                    b [edi+5] == 68h && \
                                    b [edi+5+5] == 68h && \
                                    b [edi+5+5+5] == 68h && \
                                    b [edi+5+5+5+5] == 0E8h







                                        ; 求最后的CALL, 好像很多



                                        lea     eax, [edi+4*5]
                                        add     eax, [eax+1]
                                        add     eax, 5



;01710000    55              PUSH    EBP
;01710001    8BEC            MOV     EBP, ESP
;01710003    60              PUSHA
;01710004    9C              PUSHF
;01710005    8B85 08000000   MOV     EAX, [EBP+8]
;0171000B    81F0 7277B93B   XOR     EAX, 3BB97772
;01710011    81F8 76753122   CMP     EAX, 22317576
;01710017    0F85 B5000000   JNZ     017100D2
;0171001D    68 E517807C     PUSH    kernel32.GetSystemTimeAsFileTime
;01710022    58              POP     EAX
;01710023    9D              POPF
;01710024    61              POPA
;01710025    C9              LEAVE
;01710026    81C4 14000000   ADD     ESP, 14
;0171002C  - FFA424 C0FFFFFF JMP     [ESP-40]



;01001000 >  54              PUSH    ESP
;01001001    5D              POP     EBP                              ; kernel32.7C816D4F

;



                                        ; 为了加快速度, 把第2行修改为push esp / pop ebp作为标记
                                        ; 不影响运行
                                        .IF     w [eax+1] == 0EC8Bh
                                                mov     w [eax+1], 5d54h



                                                ;@showdw        'new director', eax



                                                ;还要搜索所有的jmp [esp-xx]
                                                ;修改成jmp magicret
                                                lea     edx, magicret[ebp]



                                                ;开始搜索直到一片000
                                                .while  1
                                                        ;d [eax] != 0 && d [eax-2] !=0
                                                        .IF     d [eax]==0 && \
                                                                d [eax+4]==0
                                                                .BREAK
                                                        .ENDIF



                                                        ; add esp, 14
                                                        ; jmp [esp-xx]



                                                        .if     w [eax] == 0c481h && \
                                                                w [eax+6] == 0a4ffh



                                                                add     eax, 6;停在jmp



                                                                call    makejmp
                                                        .endif



                                                        inc     eax
                                                .endw
                                        .ENDIF



                                                ;@bpx



                                        ;已经定向到自己的fixer，调用修复



                                        pusha
                                        push    'FUCK'  ;为了找到自己的stack
                                        call    d [esi]







        ; 模拟iat之后回到这里
        magicret:                       @delta  eax
                                        mov     ecx, [esp-40h]  ;???
                                        mov     d buf[eax], ecx



                                        .while  d [esp-4] !='FUCK'
                                        add     esp, 4
                                        .endw
                                        popa;恢复我们的context



                                        mov     eax, d buf[ebp]





                                        mov     [esi], eax





                                .ENDIF
                        .ENDIF
                .ENDIF
                popa



        .ENDIF
        pop     esi
        inc     esi
.ENDW



popa

```

## 1.4 结论

花指令非常重要。

壳不可出现规律性代码。

脱壳不难，但是写教程真是自虐。

forgot/iPB

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
