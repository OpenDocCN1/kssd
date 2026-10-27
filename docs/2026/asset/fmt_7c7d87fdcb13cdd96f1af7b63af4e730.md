
```
                invoke        BFEncryptStream,addr szKeyTmp2,addr szRegName,addr szKeyTmp1

                invoke        lstrcpy,addr szENStr,addr szKeyTmp2

                invoke        HashLMD,addr szKeyTmp1,addr szKeyTmp2

                dec        edi

                jnz        @b

                lea        edi,szRegNum

                invoke        GetTickCount

                mov        ecx,19h

                xor        edx,edx

        @@:

                and        eax,3fh

                mov        bl,BYTE ptr [szBase+eax]

                mov        BYTE ptr [edi+edx],bl

                add        eax,4

                inc        edx

                dec        ecx

                jnz        @b

                lea        esi,szENStr

                mov        al,BYTE ptr [esi]

                mov        BYTE ptr [edi+1],al

                mov        al,BYTE ptr [esi+1]

                mov        BYTE ptr [edi+6],al

                mov        al,BYTE ptr [esi+2]

                mov        BYTE ptr [edi+8],al

                mov        al,BYTE ptr [esi+3]

                mov        BYTE ptr [edi+0dh],al

                mov        al,BYTE ptr [esi+4]

                mov        BYTE ptr [edi+10h],al

                mov        al,BYTE ptr [esi+5]

                mov        BYTE ptr [edi+12h],al

                mov        al,BYTE ptr [esi+6]

                mov        BYTE ptr [edi+14h],al

                mov        al,BYTE ptr [esi+7]

                mov        BYTE ptr [edi+18h],al

                mov        BYTE ptr [edi+4],2dh

                mov        BYTE ptr [edi+9],2dh

                mov        BYTE ptr [edi+0eh],2dh

                mov        BYTE ptr [edi+13h],2dh

                invoke        CharUpper,edi

                invoke        SetDlgItemText,hDlg,IDC\_REG,edi

        .endif

        popad

        ret

GetRegKey        endp
```

---

【版权声明】   本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

---

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
