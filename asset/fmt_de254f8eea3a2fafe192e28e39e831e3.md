
```assembly
.text:00406B79 50                 push  eax
.text:00406B7A 6A 09              push  9                  ; 9
.text:00406B7C 8D 8D 60 FF FF FF  lea   ecx, [ebp+var_A0]
.text:00406B82 C6 45 FC 21        mov   byte ptr [ebp+var_4], 21h
.text:00406B86 89 55 D4           mov   [ebp+var_2C], edx
.text:00406B89 E8 4A F3 00 00     call  atr_StringBase__atr_StringBase_int_
.text:00406B8E 83 EC 08           sub   esp, 8
.text:00406B91 8B CC             mov   ecx, esp
.text:00406B93 89 65 EC           mov   [ebp+var_14], esp
.text:00406B96 8D 95 60 FF FF FF  lea   edx, [ebp+var_A0]
.text:00406B9C 52                 push  edx
.text:00406B9D 89 4D 08           mov   [ebp+arg_0], ecx
.text:00406BA0 68 C8 00 00 00      push  0C8h                 ; 200
.text:00406BA5 8D 4D 90           lea   ecx, [ebp+var_70]
.text:00406BA8 C6 45 FC 22        mov   byte ptr [ebp+var_4], 22h
.text:00406BAC E8 27 F3 00 00     call  atr_StringBase__atr_StringBase_int_
.text:00406BB1 83 EC 08           sub   esp, 8
.text:00406BB4 8B F4             mov   esi, esp
.text:00406BB6 89 65 EC           mov   [ebp+var_14], esp
.text:00406BB9 8D 45 90           lea   eax, [ebp+var_70]
.text:00406BBC 50                 push  eax
.text:00406BBD 6A 48              push  48h                  ; 72
.text:00406BBF 8D 4D 88           lea   ecx, [ebp+var_78]
.text:00406BC2 C6 45 FC 23        mov   byte ptr [ebp+var_4], 23h
.text:00406BC6 E8 0D F3 00 00     call  atr_StringBase__atr_StringBase_int_
.text:00406BCB 83 EC 08           sub   esp, 8
.text:00406BCE C6 45 FC 24        mov   byte ptr [ebp+var_4], 24h
.text:00406BD2 8B FC             mov   edi, esp
.text:00406BD4 89 65 EC           mov   [ebp+var_14], esp
.text:00406BD7 8D 4D 88           lea   ecx, [ebp+var_78]
.text:00406BDA 51                 push  ecx
.text:00406BDB 6A 33              push  33h                  ; 51
.text:00406BDD 8D 4D 98           lea   ecx, [ebp+var_68]
.text:00406BE0 E8 F3 F2 00 00     call  atr_StringBase__atr_StringBase_int_
.text:00406BE5 83 EC 08           sub   esp, 8
.text:00406BE8 89 65 EC           mov   [ebp+var_14], esp
.text:00406BEB 8B DC             mov   ebx, esp
.text:00406BED 8D 55 98           lea   edx, [ebp+var_68]
.text:00406BF0 52                 push  edx
.text:00406BF1 83 EC 08           sub   esp, 8
.text:00406BF4 89 65 EC           mov   [ebp+var_14], esp
.text:00406BF7 8B CC             mov   ecx, esp
.text:00406BF9 6A 02              push  2                    ; 2
.text:00406BFB C6 45 FC 25        mov   byte ptr [ebp+var_4], 25h
.text:00406BFF E8 D4 F2 00 00     call  atr_StringBase__atr_StringBase_int_
.text:00406C04 53                 push  ebx
.text:00406C05 E8 C6 E9 FF FF     call  sub_4055D0
.text:00406C0A 83 C4 10           add   esp, 10h
.text:00406C0D 57                 push  edi
.text:00406C0E E8 BD E9 FF FF     call  sub_4055D0
.text:00406C13 83 C4 10           add   esp, 10h
.text:00406C16 56                 push  esi
.text:00406C17 E8 B4 E9 FF FF     call  sub_4055D0
.text:00406C1C 8B 45 08           mov   eax, [ebp+arg_0]
.text:00406C1F 83 C4 10           add   esp, 10h
.text:00406C22 50                 push  eax
.text:00406C23 E8 A8 E9 FF FF     call  sub_4055D0
.text:00406C28 8B 4D D4           mov   ecx, [ebp+var_2C]
.text:00406C2B 83 C4 10           add   esp, 10h
.text:00406C2E 51                 push  ecx
.text:00406C2F E8 9C E9 FF FF     call  sub_4055D0
.text:00406C34 8B 55 A4           mov   edx, [ebp+var_5C]
.text:00406C37 83 C4 10           add   esp, 10h
.text:00406C3A 52                 push  edx
.text:00406C3B E8 90 E9 FF FF     call  sub_4055D0
.text:00406C40 8B 45 A8           mov   eax, [ebp+var_58]
.text:00406C43 83 C4 10           add   esp, 10h
.text:00406C46 50                 push  eax
.text:00406C47 E8 84 E9 FF FF     call  sub_4055D0
.text:00406C4C 8B 4D AC           mov   ecx, [ebp+var_54]
.text:00406C4F 83 C4 10           add   esp, 10h
.text:00406C52 51                 push  ecx
.text:00406C53 E8 78 E9 FF FF     call  sub_4055D0
.text:00406C58 8D 55 B8           lea   edx, [ebp+var_48]
.text:00406C5B 83 C4 10           add   esp, 10h
.text:00406C5E 52                 push  edx
.text:00406C5F E8 6C E9 FF FF     call  sub_4055D0
.text:00406C64 83 C4 10           add   esp, 10h
.text:00406C67 8B 45 9C           mov   eax, [ebp+var_64]
.text:00406C6A 33 FF              xor   edi, edi              ;上面的连接成字串 "2517220091423618851"
.text:00406C6C 3B C7              cmp   eax, edi              ;该字串作为 blowfish 的 key1
.text:00406C6E C6 45 FC 2F        mov   byte ptr [ebp+var_4], 2Fh
.text:00406C72 74 0E              jz    short loc_406C82
.text:00406C74 FF 08             dec   dword ptr [eax]
.text:00406C76 39 38              cmp   [eax], edi
.text:00406C78 75 08              jnz   short loc_406C82
.text:00406C7A 8D 4D 98           lea   ecx, [ebp+var_68]
.text:00406C7D E8 38 F2 00 00     call  atr_StringBase__Delete_void_
.text:00406C82
.text:00406C82                   loc_406C82:                 ; CODE XREF: sub_406580+6F2j
.text:00406C82                                       ; sub_406580+6F8j
.text:00406C82 8B 45 8C           mov   eax, [ebp+var_74]
.text:00406C85 3B C7              cmp   eax, edi
.text:00406C87 C6 45 FC 2E        mov   byte ptr [ebp+var_4], 2Eh
.text:00406C8B 74 0E              jz    short loc_406C9B
.text:00406C8D FF 08             dec   dword ptr [eax]
.text:00406C8F 39 38              cmp   [eax], edi
.text:00406C91 75 08              jnz   short loc_406C9B
.text:00406C93 8D 4D 88           lea   ecx, [ebp+var_78]
.text:00406C96 E8 1F F2 00 00     call  atr_StringBase__Delete_void_
.text:00406C9B
.text:00406C9B                   loc_406C9B:                 ; CODE XREF: sub_406580+70Bj
.text:00406C9B                                       ; sub_406580+711j
.text:00406C9B 8B 45 94           mov   eax, [ebp+var_6C]
.text:00406C9E 3B C7              cmp   eax, edi
.text:00406CA0 C6 45 FC 2D        mov   byte ptr [ebp+var_4], 2Dh
.text:00406CA4 74 0E              jz    short loc_406CB4
.text:00406CA6 FF 08             dec   dword ptr [eax]
.text:00406CA8 39 38              cmp   [eax], edi
.text:00406CAA 75 08              jnz   short loc_406CB4
```
