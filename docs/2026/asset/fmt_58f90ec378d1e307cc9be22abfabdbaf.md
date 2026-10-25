
```
loc_13148B8B:                                ; CODE XREF: sub_131489E4+154j
                                                        ; DATA XREF: sub_131489E4:off_13148B5Fo
                                                        xor        ecx, ecx        ; jumptable 13148B38 case 30
                                                        mov        dword ptr [ebp+var_14], ecx
                                                        jmp        short loc_13148BE1 ; default
                                                        ; jumptable 13148B38 cases 8-29
                                                        ; ---------------------------------------------------------------------------
loc_13148B92:                                ; CODE XREF: sub_131489E4+154j
                                                        ; DATA XREF: sub_131489E4:off_13148B5Fo
                                                        mov        dword ptr [ebp+var_14], 1        ; jumptable 13148B38 case 31
                                                        jmp        short loc_13148BE1 ; default
                                                        ; jumptable 13148B38 cases 8-29
                                                        ; ---------------------------------------------------------------------------
loc_13148B9B:                                ; CODE XREF: sub_131489E4+154j
                                                        ; DATA XREF: sub_131489E4:off_13148B5Fo
                                                        mov        DWORD ptr [ebp+var_14], 2        ; jumptable 13148B38 case 0
                                                        jmp        short loc_13148BE1 ; default
                                                        ; jumptable 13148B38 cases 8-29
                                                        ; ---------------------------------------------------------------------------
loc_13148BA4:                                ; CODE XREF: sub_131489E4+154j
                                                        ; DATA XREF: sub_131489E4:off_13148B5Fo
                                                        mov        DWORD ptr [ebp+var_14], 3        ; jumptable 13148B38 case 1
                                                        jmp        short loc_13148BE1 ; default
                                                        ; jumptable 13148B38 cases 8-29
                                                        ; ---------------------------------------------------------------------------
loc_13148BAD:                                ; CODE XREF: sub_131489E4+154j
                                                        ; DATA XREF: sub_131489E4:off_13148B5Fo
                                                        mov        dword ptr [ebp+var_14], 4        ; jumptable 13148B38 case 2
                                                        jmp        short loc_13148BE1 ; default
                                                        ; jumptable 13148B38 cases 8-29
                                                        ; ---------------------------------------------------------------------------
loc_13148BB6:                                ; CODE XREF: sub_131489E4+154j
                                                        ; DATA XREF: sub_131489E4:off_13148B5Fo
                                                        mov        dword ptr [ebp+var_14], 5        ; jumptable 13148B38 case 3
                                                        jmp        short loc_13148BE1 ; default
                                                        ; jumptable 13148B38 cases 8-29
                                                        ; ---------------------------------------------------------------------------
loc_13148BBF:                                ; CODE XREF: sub_131489E4+154j
                                                        ; DATA XREF: sub_131489E4:off_13148B5Fo
                                                        mov        dword ptr [ebp+var_14], 6        ; jumptable 13148B38 case 4
                                                        jmp        short loc_13148BE1 ; default
                                                        ; jumptable 13148B38 cases 8-29
                                                        ; ---------------------------------------------------------------------------
loc_13148BC8:                                ; CODE XREF: sub_131489E4+154j
                                                        ; DATA XREF: sub_131489E4:off_13148B5Fo
                                                        mov        dword ptr [ebp+var_14], 7        ; jumptable 13148B38 case 5
                                                        jmp        short loc_13148BE1 ; default
                                                        ; jumptable 13148B38 cases 8-29
                                                        ; ---------------------------------------------------------------------------
loc_13148BD1:                                ; CODE XREF: sub_131489E4+154j
                                                        ; DATA XREF: sub_131489E4:off_13148B5Fo
                                                        mov dword ptr [ebp+var_14], 8        ; jumptable 13148B38 case 6
                                                        jmp        short loc_13148BE1 ; default
                                                        ; jumptable 13148B38 cases 8-29
                                                        ; ---------------------------------------------------------------------------
loc_13148BDA:                                ; CODE XREF: sub_131489E4+154j
                                                        ; DATA XREF: sub_131489E4:off_13148B5Fo
                                                        mov        dword ptr [ebp+var_14], 9        ; jumptable 13148B38 case 7
loc_13148BE1:                                ; CODE XREF: sub_131489E4+147j
                                                        ; sub_131489E4+154j ...
                                                        mov        ecx, edx        ; default
                                                        ; jumptable 13148B38 cases 8-29
                                                        and        cl, 0C0h
                                                        cmp        cl, 0C0h
                                                        jz        short loc_13148C08
                                                        movzx        edx, dl
                                                        shr        edx, 3
                                                        and        edx, 7
                                                        mov        ecx, dword ptr [ebp+var_14]
                                                        add        ecx, ecx
                                                        add        ecx, ecx
                                                        lea        ecx, unk_1314BDB0[ecx*8]
                                                        movzx        ecx, word ptr [ecx+edx*2]
                                                        jmp        short loc_13148C24
                                                        ; ---------------------------------------------------------------------------
loc_13148C08:                                ; CODE XREF: sub_131489E4+205j
                                                        movzx        edx, dl
                                                        shr        edx, 3
                                                        and        edx, 7
                                                        mov        ecx, dword ptr [ebp+var_14]
```
