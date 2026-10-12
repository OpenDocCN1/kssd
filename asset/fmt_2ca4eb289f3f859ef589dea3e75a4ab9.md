
```text
            ->"DDDDE,D9E92E32,DE3G63GG,GDDWCDEC,2966EDDG,ACDD"  
            ->"GF36,EFC4F2G2,DG9E44G9,D2FDGGGD,6GG29ECD,G36GE"  
            ->"9GC,G3D46GEC,F9W22GDG,YYVN855T,D9EGE96E,G999CG"  
            ->"DA,DGGD9XC9,DD43GCW2,236CG2CD,9DGCEDDD,GC9C2D9"  
            ->"G,E9G643D4,G9GWAWED,2GDEGCG4,EEGXGD22,D9D2GD4F"  
            ->",DD9DGDGD,64322E3C,EDEG"  
            |
```

```asm
:004216B0 BF7C944900               mov edi, 0049947C  
                                   <===edi 指向上面我们看到的一长串固定值
:004216B5 F2                       repnz
:004216B6 AE                       scasb
                                   <===取这一长串固定值的长度
:004216B7 F7D1                     not ecx
                                   <===ecx=798
:004216B9 2BF9                     sub edi, ecx
                                   <===调整指针指向第 1 位
:004216BB 8D9424D8190000          lea edx, dword ptr [esp+000019D8]
:004216C2 8BC1                     mov eax, ecx
:004216C4 8BF7                     mov esi, edi
:004216C6 C1E902                   shr ecx, 02
                                   <===ecx=798 div 04=1E6
:004216C9 8BFA                     mov edi, edx
:004216CB 8D542410                lea edx, dword ptr [esp+10]
:004216CF F3                       repz
:004216D0 A5                       movsd
                                   <===从[esi]复制 486 字节(1E6)的到[edi]中
:004216D1 8BC8                     mov ecx, eax
:004216D3 33C0                     xor eax, eax
:004216D5 83E103                   and ecx, 00000003
:004216D8 F3                       repz
:004216D9 A4                       movsb
```

--------------------------------------------------------------------------------

以下的 4 部分同上，我就不注释了！

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

```text
* Possible StringData Ref from Code Obj ->"EL4ICGE4,GFA4CGGB,E6GLGGAC,DFG6D6LF,E4CGECAA,C"
            ->"ADCGGG4,FDLDGDDG,B6GA5FBB,GAFCLG6A,GFAL6LLC,75"
            ->"T48Y5Y,DFDFDDGL,CEEFD4D4,BCGDGGGL,G64A3G6L,6DG"
            ->"GDCLG,G4LZCZ5F,CDCA666A,6DDADFEG,GDLFEDLA,CFL6"
            ->"GLEG,GLCLD6BG,6C6EG4C6,G6CFGGCF,37T448Q4,GC6FC"
            ->"4D4,G44CCA43,4DDGGBEE,DFDL464D,4GD4GFC3,DAL4A6"
            ->"6C,BECF66DC,A4BB2GAG,66C4CEDD,L4CDGFAL,LL3DCDG"
            ->"A,LF43GCED,FDGAGE6D,LGFAFAGF,F6EGLLDF,GLLGDDC3"
            ->",DDFL6CEC,4CDAFGGA,4DLG"
            |
```

```asm
:004216DA BF9C8C4900               mov edi, 00498C9C
:004216DF 83C9FF                    or ecx, FFFFFFFF
:004216E2 F2                       repnz
:004216E3 AE                       scasb
:004216E4 F7D1                     not ecx
                                   <===ecx=7E0
:004216E6 2BF9                     sub edi, ecx
:004216E8 8BC1                     mov eax, ecx
:004216EA 8BF7                     mov esi, edi
:004216EC 8BFA                     mov edi, edx
:004216EE 8D9424A8080000          lea edx, dword ptr [esp+000008A8]
:004216F5 C1E902                   shr ecx, 02
                                   <===ecx=7E0 div 04=1F8
:004216F8 F3                       repz
:004216F9 A5                       movsd
:004216FA 8BC8                     mov ecx, eax
:004216FC 33C0                     xor eax, eax
:004216FE 83E103                   and ecx, 00000003
:00421701 F3                       repz
:00421702 A4                       movsb
```

```text
* Possible StringData Ref from Code Obj ->"DCKJDGDK,DEKC6JGD,EKK6EBKJ,KKJDDJ6J,JKGAKADJ,K"
            ->"7H6JJKD,AKGHKDKJ,GKD7DK6K,DKJGFKKG,HJD2HKKD,DD"
            ->"DDH6JK,2GK3KADJ,KBCKDDJK,JACGJJHK,66CJCK3H,JDK"
            ->"DJKJK,HC3K37GJ,2FJFCKKC,2HKBBKB6,KJJKBG3C,KGJK"
            ->"BDGD,D6KKJGHC,FKJD2GKK,6DDJKDJH,DG6HH26D,3GG6J"
            ->"FGG,KKKGKF3J,GFKCCGDK,6HBHKK66,HJK67K6G,9JHGA3"
            ->"AK,GCDXLFX5,GF2E3KKK,A6E2CIDA,K3DKGDKF,363HH6K"
            ->"E,KDDKG6CD,J27EJ6JJ,KDCDDDKJ,JE2DBDC7,KKDJ6DAE"
            ->",3KHCKFGK,3K6J6HCD,DHK6"
            |
```

```asm
:00421703 BFC4844900               mov edi, 004984C4
:00421708 83C9FF                    or ecx, FFFFFFFF
:0042170B F2                       repnz
:0042170C AE                       scasb
:0042170D F7D1                     not ecx
                                   <===ecx=7D7
:0042170F 2BF9                     sub edi, ecx
:00421711 8BC1                     mov eax, ecx
:00421713 8BF7                     mov esi, edi
:00421715 8BFA                     mov edi, edx
:00421717 8D942440110000          lea edx, dword ptr [esp+00001140]
:0042171E C1E902                   shr ecx, 02
                                   <===ecx=7D7 div 04=1F5
:00421721 F3                       repz
:00421722 A5                       movsd
:00421723 8BC8                     mov ecx, eax
:00421725 33C0                     xor eax, eax
:00421727 83E103                   and ecx, 00000003
:0042172A F3                       repz
:0042172B A4                       movsb
```

```text
* Possible StringData Ref from Code Obj ->"KK8KKBKE,JEEKDKKD,8FKGEFEB,JKHFK2JK,DGGXBFGK,F"
            ->"K7HK5C6,6JJFGD8K,58B7XKJK,FXKGFKG7,GGXGBJJK,K2"
            ->"JJ8JBJ,9BDK8DKK,85C5JHGJ,BXKKFKJK,JFD2KFXF,KJF"
            ->"5KKXG,K8CXFFKF,8KKJH5DF,CJKJBDKJ,HH8XKKFJ,JCK6"
            ->"X5FH,FCH8F8GK,8KJFKJFF,K59KHRBK,ACKGC8DC,KHDFK"
            ->"E5J,95FCKFGK,KKGFKH5K,J786JF9C,KJJ9F5G5,FFXCKH"
            ->"K8,FB7FECWW,ZFD8I7XK,K5FEJ87K,JKJJHCX9,7KKJDJK"
            ->"X,9BKF6CXE,GCHECA9L,FXHKE65F,89KGFF6X,DGF7JDJK"
            ->",7GXEKDKC,K8GJHF7J,JBJD"
            |
```

```asm
:0042172C BF90814900               mov edi, 00498190
:00421731 83C9FF                    or ecx, FFFFFFFF
:00421734 F2                       repnz
:00421735 AE                       scasb
:00421736 F7D1                     not ecx
                                   <===ecx=333
:00421738 2BF9                     sub edi, ecx
:0042173A 8BC1                     mov eax, ecx
:0042173C 8BF7                     mov esi, edi
:0042173E 8BFA                     mov edi, edx
:00421740 C1E902                   shr ecx, 02
                                   <===ecx=333 div 04=CC
:00421743 F3                       repz
:00421744 A5                       movsd
:00421745 8BC8                     mov ecx, eax
:00421747 83E103                   and ecx, 00000003
:0042174A F3                       repz
:0042174B A4                       movsb
```

```text
* Possible StringData Ref from Code Obj ->"XC2C2FXC,H3XLZE2F,XHH2XFZE,CZ3EYFA3,FZFACAX2,E"
            ->"DG2EG2Y,AZFLHCHE,FDEDECAC,CA9FEI2G,G3XZLZHX,VX"
            ->"2WFLFF,ZFZVLAXG,ZFC2CC3Z,3A9F2GLZ,22YE4ZVG,GXZ"
            ->"EEZFD,GCVYVDFF,ZE2ECHZ9,3WA22HXA,Z33DFFVY,CFGC"
            ->"FEFE,X2HZ2FLC,E22EZF2L,CVKKCK8K,EF2LGZ2Y,XWZLZ"
