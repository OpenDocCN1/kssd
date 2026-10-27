
```
        @4:
        mov        edx,lpTmp2
        mov        edx,DWORD ptr [edx]
        mov        ecx,lpTmp9
        mov        esi,DWORD ptr [ecx]
        mov        ecx,esi
        mov        ebx,lpTmp2
        mov        ebx,DWORD ptr [ebx+4]
        mov        nTmp3,esi
        mov        edi,lpTmp2
        mov        edi,DWORD ptr [edi+8]
        mov        nTmp4,edi
        mov        nTmp5,esi
        mov        edi,lpTmp2
        mov        edi,DWORD ptr [edi+0ch]
        mov        nTmp6,edi
        mov        nTmp7,esi
        mov        esi,8
        @3:
        mov        edi,ecx
        shl        edi,0bh
        xor        edx,edi
        add        nTmp3,edx
        add        ecx,ebx
        mov        edi,ebx
        shr         edi,2
        xor        ecx,edi
        add        nTmp4,ecx
        add        ebx,nTmp3
        mov        edi,nTmp3
        shl        edi,8
        xor        ebx,edi
        add        nTmp5,ebx
        mov        edi,nTmp4
        add        nTmp3,edi
        mov        edi,nTmp4
        shr        edi,10h
        xor        nTmp3,edi
        mov        edi,nTmp3
        add        nTmp6,edi
        mov        edi,nTmp5
        add        nTmp4,edi
        mov        edi,nTmp5
        shl        edi,0ah
        xor        nTmp4,edi
        mov        edi,nTmp4
        add        nTmp7,edi
        mov        edi,nTmp6
        add        nTmp5,edi
        mov        edi,nTmp6
        shr        edi,4
        xor        nTmp5,edi
        add        edx,nTmp5
        mov        edi,nTmp7
        add        nTmp6,edi
        mov        edi,nTmp7
        shl        edi,8
        xor        nTmp6,edi
        add        ecx,nTmp6
        add        nTmp7,edx
        mov        edi,edx
        shr        edi,9
        xor        nTmp7,edi
        add        ebx,nTmp7
        add        edx,ecx
        dec        esi
        jnz        @3
        mov        DWORD ptr [eax],edx
        mov        DWORD ptr [eax+4],ecx
        mov        DWORD ptr [eax+8],ebx
        mov        edx,nTmp3
        mov        DWORD ptr [eax+0ch],edx
        mov        edx,nTmp4
        mov        DWORD ptr [eax+10h],edx
        mov        edx,nTmp5
        mov        DWORD ptr [eax+14h],edx
        mov        edx,nTmp6
        mov        DWORD ptr [eax+18h],edx
        mov        edx,nTmp7
        mov        DWORD ptr [eax+1ch],edx
        add        eax,20h
        add        lpTmp9,4
        dec        nTmp8
        jnz        @4
        popad
        ret

InitEncryptLBC        endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;函数功能：Hash算法运算函数
;函数参数：
;        lpTStr：        指针，初始化数据的地址
;        lpData：         指针，待加密数据的地址（变形MD5算法中的4个变量）
;        nNum:                运算中变换的次数
;返回值：没有
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

EncryptLBC        proc        lpTStr:DWORD,lpData:DWORD,nNum:DWORD
        local        szStr[32]:BYTE,nCon:DWORD
        pushad
        mov        esi,lpData
        mov        ecx,nNum
        mov        nCon,ecx
        @2:
        mov        eax,DWORD ptr [esi]
        mov        DWORD ptr [szStr+0ch],eax
        mov        eax,DWORD ptr [esi+4]
        mov        DWORD ptr [szStr+10h],eax
        mov        eax,DWORD ptr [esi+8]
        mov        DWORD ptr [szStr+14h],eax
        mov        eax,DWORD ptr [esi+0ch]
        mov        DWORD ptr [szStr+18h],eax
        mov        ebx,lpTStr
        mov        eax,DWORD ptr [ebx+4]
        mov        DWORD ptr [szStr+1ch],eax
        add        ebx,8
        push        esi
        @1:
        mov        edx,DWORD ptr [szStr+0ch]
        mov        ecx,DWORD ptr [ebx]
        mov        esi,DWORD ptr [szStr+10h]
        mov        eax,DWORD ptr [ebx+4]
        add        edx,eax
        add        eax,edx
        mov        edi,edx
        shr        edi,7
        xor        edx,edi
        add        ecx,edx
        add        edx,ecx
        mov        edi,ecx
        shl        edi,0dh
        xor        ecx,edi
        add        esi,ecx
        add        ecx,esi
        mov        edi,esi
        shr        edi,11h
        xor        esi,edi
        add        eax,esi
        add        esi,eax
        mov        edi,eax
        shl        edi,9
        xor        eax,edi
        add        edx,eax
        add        eax,edx
        mov        edi,edx
        shr        edi,3
        xor        edx,edi
        add        ecx,edx
        add        edx,ecx
        mov        edi,ecx
        shl        edi,7
        xor        ecx,edi
        add        esi,ecx
        add        ecx,esi
        mov        edi,eax
        shr        edi,0fh
        xor        esi,edi
        add        eax,esi
        add        esi,eax
        mov        edi,eax
        shl        edi,0bh
        xor        eax,edi
        mov        edi,edx
        mov        edx,esi
        mov        esi,edi
        mov        edi,ecx
        mov        ecx,eax
        mov        eax,edi
        add        edx,eax
        add        eax,edx
        mov        edi,edx
        shr        edi,7
        xor        edx,edi
        add        ecx,edx
        add        edx,ecx
        mov        edi,ecx
        shl        edi,0dh
        xor        ecx,edi
        add        esi,ecx
        add        ecx,esi
        mov        edi,esi
        shr        edi,11h
        xor        esi,edi
        add        eax,esi
        add        esi,eax
        mov        edi,eax
        shl        edi,9
        xor        eax,edi
        add        edx,eax
        add        eax,edx
        mov        edi,edx
        shr        edi,3
        xor        edx,edi
        add        ecx,edx
        add        edx,ecx
        mov        edi,ecx
        shl        edi,7
        xor        ecx,edi
        add        esi,ecx
        add        ecx,esi
        mov        edi,eax
        shr        edi,0fh
        xor        esi,edi
        add        eax,esi
        add        esi,eax
        mov        edi,eax
        shl        edi,0bh
        xor        eax,edi
        xor        edx,DWORD ptr [szStr+14h]
        xor        ecx,edx
        mov        DWORD ptr [szStr+4],ecx
```
