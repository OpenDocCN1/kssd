
```
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
00557DB2  |.  8842 01       mov     byte ptr [edx+1], al                 ;  第 2 个字节  

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
00557DF1  |.  8842 02       mov     byte ptr [edx+2], al                 ;  第 3 个字节  

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
00557E30  |.  8842 03       mov     byte ptr [edx+3], al                 ;  第 4 个字节  

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
00557E6F  |.  8842 04       mov     byte ptr [edx+4], al                 ;  第 5 个字节  

00557E72  |.  8A47 02       mov     al, byte ptr [edi+2]  
00557E75  |.  C1E0 07       shl     eax, 7  
00557E78  |.  8A56 02       mov     dl, byte ptr [esi+2]  
00557E7B  |.  C1E2 06       shl     edx, 6  
00557E7E  |.  0AC2          or      al, dl  
00557E80  |.  8A57 0A       mov     dl, byte ptr [edi+A]  
00557E83  |.  C1E2 05       shl     edx, 5  
00557E86  |.  0AC2          or      al, dl  
00557E88  |.  8A56 0A       mov     dl, byte ptr [esi+A]  
00557E8B  |.  C1E2 04       shl     edx, 4  
00557E8E  |.  0AC2          or      al, dl  
00557E90  |.  8A57 12       mov     dl, byte ptr [edi+12]  
00557E93  |.  C1E2 03       shl     edx, 3  
00557E96  |.  0AC2          or      al, dl  
00557E98  |.  8A56 12       mov     dl, byte ptr [esi+12]  
00557E9B  |.  C1E2 02       shl     edx, 2  
00557E9E  |.  0AC2          or      al, dl  
00557EA0  |.  8A57 1A       mov     dl, byte ptr [edi+1A]  
00557EA3  |.  03D2          add     edx, edx  
00557EA5  |.  0AC2          or      al, dl  
00557EA7  |.  0A46 1A       or      al, byte ptr [esi+1A]  
00557EAA  |.  8B5424 08     mov     edx, dword ptr [esp+8]  
00557EAE  |.  8842 05       mov     byte ptr [edx+5], al                 ;  第 6 个字节  

00557EB1  |.  8A47 01       mov     al, byte ptr [edi+1]  
00557EB4  |.  C1E0 07       shl     eax, 7  
00557EB7  |.  8A56 01       mov     dl, byte ptr [esi+1]  
00557EBA  |.  C1E2 06       shl     edx, 6  
00557EBD  |.  0AC2          or      al, dl  
00557EBF  |.  8A57 09       mov     dl, byte ptr [edi+9]  
00557EC2  |.  C1E2 05       shl     edx, 5  
00557EC5  |.  0AC2          or      al, dl  
00557EC7  |.  8A56 09       mov     dl, byte ptr [esi+9]  
00557ECA  |.  C1E2 04       shl     edx, 4  
00557ECD  |.  0AC2          or      al, dl  
00557ECF  |.  8A57 11       mov     dl, byte ptr [edi+11]  
00557ED2  |.  C1E2 03       shl     edx, 3  
00557ED5  |.  0AC2          or      al, dl  
00557ED7  |.  8A56 11       mov     dl, byte ptr [esi+11]  
00557EDA  |.  C1E2 02       shl     edx, 2  
00557EDD  |.  0AC2          or      al, dl  
00557EDF  |.  8A57 19       mov     dl, byte ptr [edi+19]  
00557EE2  |.  03D2          add     edx, edx  
00557EE4  |.  0AC2          or      al, dl  
00557EE6  |.  0A46 19       or      al, byte ptr [esi+19]  
00557EE9  |.  8B5424 08     mov     edx, dword ptr [esp+8]  
00557EED  |.  8842 06       mov     byte ptr [edx+6], al                 ;  第 7 个字节  

00557EF0  |.  8A07          mov     al, byte ptr [edi]  
00557EF2  |.  C1E0 07       shl     eax, 7  
00557EF5  |.  8A16          mov     dl, byte ptr [esi]  
00557EF7  |.  C1E2 06       shl     edx, 6  
00557EFA  |.  0AC2          or      al, dl  
00557EFC  |.  8A57 08       mov     dl, byte ptr [edi+8]  
00557EFF  |.  C1E2 05       shl     edx, 5  
00557F02  |.  0AC2          or      al, dl  
00557F04  |.  8A56 08       mov     dl, byte ptr [esi+8]  
00557F07  |.  C1E2 04       shl     edx, 4  
00557F0A  |.  0AC2          or      al, dl  
00557F0C  |.  8A57 10       mov     dl, byte ptr [edi+10]  
```
