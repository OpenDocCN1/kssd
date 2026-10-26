
```
00557CD1  |.  325F 04       |xor     bl, byte ptr [edi+4]

00557CD4  |.  881D 60A15800 |mov     byte ptr [58A160], bl

00557CDA  |.  0FB7D9        |movzx   ebx, cx

00557CDD  |.  8A9B 84755800 |mov     bl, byte ptr [ebx+587584]

00557CE3  |.  325F 1A       |xor     bl, byte ptr [edi+1A]

00557CE6  |.  881D 76A15800 |mov     byte ptr [58A176], bl

00557CEC  |.  0FB7D9        |movzx   ebx, cx

00557CEF  |.  8A9B 84775800 |mov     bl, byte ptr [ebx+587784]

00557CF5  |.  325F 0E       |xor     bl, byte ptr [edi+E]

00557CF8  |.  881D 6AA15800 |mov     byte ptr [58A16A], bl

00557CFE  |.  0FB7C9        |movzx   ecx, cx

00557D01  |.  8A89 84795800 |mov     cl, byte ptr [ecx+587984]

00557D07  |.  324F 14       |xor     cl, byte ptr [edi+14]

00557D0A  |.  880D 70A15800 |mov     byte ptr [58A170], cl

//////////////////////8

00557D10  |.  56            |push    esi

00557D11  |.  57            |push    edi

00557D12  |.  B9 08000000   |mov     ecx, 8

00557D17  |.  F3:A5         |rep     movs dword ptr es:[edi], dword ptr [esi]  //si 复制到DI

00557D19  |.  5F            |pop     edi

00557D1A  |.  5E            |pop     esi

00557D1B  |.  56            |push    esi

00557D1C  |.  57            |push    edi

00557D1D  |.  8BFE          |mov     edi, esi

00557D1F  |.  BE 5CA15800   |mov     esi, 0058A15C //此处属于C[] 为了方便称为e[] (e[] si[]  di[] 大小相同)

00557D24  |.  B9 08000000   |mov     ecx, 8

00557D29  |.  F3:A5         |rep     movs dword ptr es:[edi], dword ptr [esi ]//e[]复制到si[]

00557D2B  |.  5F            |pop     edi

00557D2C  |.  5E            |pop     esi

00557D2D  |.  83C0 30       |add     eax, 30 //把C[]的指针移动48位

00557D30  |.  FECA          |dec     dl

00557D32  |.^ 0F85 74FAFFFF \jnz     005577AC //循环16次
```

前面的代码主要是用输入的名字和己生成的序列生成两个新的序列

下面就是用这两个序列生成序列号

```
00557D38  |.  8A47 07       mov     al, byte ptr [edi+7]

00557D3B  |.  C1E0 07       shl     eax, 7

00557D3E  |.  8A56 07       mov     dl, byte ptr [esi+7]

00557D41  |.  C1E2 06       shl     edx, 6

00557D44  |.  0AC2          or      al, dl

00557D46  |.  8A57 0F       mov     dl, byte ptr [edi+F]

00557D49  |.  C1E2 05       shl     edx, 5

00557D4C  |.  0AC2          or      al, dl

00557D4E  |.  8A56 0F       mov     dl, byte ptr [esi+F]

00557D51  |.  C1E2 04       shl     edx, 4

00557D54  |.  0AC2          or      al, dl

00557D56  |.  8A57 17       mov     dl, byte ptr [edi+17]

00557D59  |.  C1E2 03       shl     edx, 3

00557D5C  |.  0AC2          or      al, dl

00557D5E  |.  8A56 17       mov     dl, byte ptr [esi+17]

00557D61  |.  C1E2 02       shl     edx, 2

00557D64  |.  0AC2          or      al, dl

00557D66  |.  8A57 1F       mov     dl, byte ptr [edi+1F]

00557D69  |.  03D2          add     edx, edx

00557D6B  |.  0AC2          or      al, dl

00557D6D  |.  0A46 1F       or      al, byte ptr [esi+1F]

00557D70  |.  8B5424 08     mov     edx, dword ptr [esp+8]

00557D74  |.  8802          mov     byte ptr [edx], al                          ;  第1个字节

00557D76  |.  8A47 06       mov     al, byte ptr [edi+6]

00557D79  |.  C1E0 07       shl     eax, 7

00557D7C  |.  8A56 06       mov     dl, byte ptr [esi+6]

00557D7F  |.  C1E2 06       shl     edx, 6

00557D82  |.  0AC2          or      al, dl

00557D84  |.  8A57 0E       mov     dl, byte ptr [edi+E]

00557D87  |.  C1E2 05       shl     edx, 5

00557D8A  |.  0AC2          or      al, dl

00557D8C  |.  8A56 0E       mov     dl, byte ptr [esi+E]

00557D8F  |.  C1E2 04       shl     edx, 4

00557D92  |.  0AC2          or      al, dl

00557D94  |.  8A57 16       mov     dl, byte ptr [edi+16]

00557D97  |.  C1E2 03       shl     edx, 3

00557D9A  |.  0AC2          or      al, dl

00557D9C  |.  8A56 16       mov     dl, byte ptr [esi+16]

00557D9F  |.  C1E2 02       shl     edx, 2

00557DA2  |.  0AC2          or      al, dl

00557DA4  |.  8A57 1E       mov     dl, byte ptr [edi+1E]

00557DA7  |.  03D2          add     edx, edx

00557DA9  |.  0AC2          or      al, dl

00557DAB  |.  0A46 1E       or      al, byte ptr [esi+1E]

00557DAE  |.  8B5424 08     mov     edx, dword ptr [esp+8]

00557DB2  |.  8842 01       mov     byte ptr [edx+1], al                        ;  第2个字节

00557DB5  |.  8A47 05       mov     al, byte ptr [edi+5]

00557DB8  |.  C1E0 07       shl     eax, 7

00557DBB  |.  8A56 05       mov     dl, byte ptr [esi+5]

00557DBE  |.  C1E2 06       shl     edx, 6

00557DC1  |.  0AC2          or      al, dl

00557DC3  |.  8A57 0D       mov     dl, byte ptr [edi+D]

00557DC6  |.  C1E2 05       shl     edx, 5

00557DC9  |.  0AC2          or      al, dl

00557DCB  |.  8A56 0D       mov     dl, byte ptr [esi+D]

00557DCE  |.  C1E2 04       shl     edx, 4

00557DD1  |.  0AC2          or      al, dl

00557DD3  |.  8A57 15       mov     dl, byte ptr [edi+15]

00557DD6  |.  C1E2 03       shl     edx, 3

00557DD9  |.  0AC2          or      al, dl

00557DDB  |.  8A56 15       mov     dl, byte ptr [esi+15]

00557DDE  |.  C1E2 02       shl     edx, 2

00557DE1  |.  0AC2          or      al, dl

00557DE3  |.  8A57 1D       mov     dl, byte ptr [edi+1D]

00557DE6  |.  03D2          add     edx, edx

00557DE8  |.  0AC2          or      al, dl

00557DEA  |.  0A46 1D       or      al, byte ptr [esi+1D]

00557DED  |.  8B5424 08     mov     edx, dword ptr [esp+8]

00557DF1  |.  8842 02       mov     byte ptr [edx+2], al                        ;  第3个字节

00557DF4  |.  8A47 04       mov     al, byte ptr [edi+4]

00557DF7  |.  C1E0 07       shl     eax, 7

00557DFA  |.  8A56 04       mov     dl, byte ptr [esi+4]

00557DFD  |.  C1E2 06       shl     edx, 6

00557E00  |.  0AC2          or      al, dl

00557E02  |.  8A57 0C       mov     dl, byte ptr [edi+C]

00557E05  |.  C1E2 05       shl     edx, 5

00557E08  |.  0AC2          or      al, dl

00557E0A  |.  8A56 0C       mov     dl, byte ptr [esi+C]

00557E0D  |.  C1E2 04       shl     edx, 4

00557E10  |.  0AC2          or      al, dl

00557E12  |.  8A57 14       mov     dl, byte ptr [edi+14]

00557E15  |.  C1E2 03       shl     edx, 3

00557E18  |.  0AC2          or      al, dl

00557E1A  |.  8A56 14       mov     dl, byte ptr [esi+14]

00557E1D  |.  C1E2 02       shl     edx, 2

00557E20  |.  0AC2          or      al, dl

00557E22  |.  8A57 1C       mov     dl, byte ptr [edi+1C]

00557E25  |.  03D2          add     edx, edx

00557E27  |.  0AC2          or      al, dl

00557E29  |.  0A46 1C       or      al, byte ptr [esi+1C]

00557E2C  |.  8B5424 08     mov     edx, dword ptr [esp+8]

00557E30  |.  8842 03       mov     byte ptr [edx+3], al                        ;  第4个字节

00557E33  |.  8A47 03       mov     al, byte ptr [edi+3]

00557E36  |.  C1E0 07       shl     eax, 7

00557E39  |.  8A56 03       mov     dl, byte ptr [esi+3]

00557E3C  |.  C1E2 06       shl     edx, 6

00557E3F  |.  0AC2          or      al, dl

00557E41  |.  8A57 0B       mov     dl, byte ptr [edi+B]

00557E44  |.  C1E2 05       shl     edx, 5

00557E47  |.  0AC2          or      al, dl

00557E49  |.  8A56 0B       mov     dl, byte ptr [esi+B]

00557E4C  |.  C1E2 04       shl     edx, 4

00557E4F  |.  0AC2          or      al, dl

00557E51  |.  8A57 13       mov     dl, byte ptr [edi+13]

00557E54  |.  C1E2 03       shl     edx, 3

00557E57  |.  0AC2          or      al, dl

00557E59  |.  8A56 13       mov     dl, byte ptr [esi+13]

00557E5C  |.  C1E2 02       shl     edx, 2

00557E5F  |.  0AC2          or      al, dl

00557E61  |.  8A57 1B       mov     dl, byte ptr [edi+1B]

00557E64  |.  03D2          add     edx, edx

00557E66  |.  0AC2          or      al, dl

00557E68  |.  0A46 1B       or      al, byte ptr [esi+1B]

00557E6B  |.  8B5424 08     mov     edx, dword ptr [esp+8]

00557E6F  |.  8842 04       mov     byte ptr [edx+4], al                        ;  第5个字节

00557E72  |.  8A47 02       mov     al, byte ptr [edi+2]

00557E75  |.  C1E0 07       shl     eax, 7

00557E78  |.  8A56 02       mov     dl, byte ptr [esi+2]

00557E7B  |.  C1E2 06       shl     edx, 6

00557E7E  |.  0AC2          or      al, dl
```
