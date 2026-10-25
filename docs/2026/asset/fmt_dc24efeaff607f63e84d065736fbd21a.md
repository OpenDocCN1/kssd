
# 反汇编代码片段

```text
.text:10011256                   loc_10011256:                 ; CODE XREF: ISwdAudioEncoder__LicenseNumber_put+7Cj
.text:10011256 8B 16             mov     edx, [esi]    //edx指向许可证号
.text:10011258 B8 05 00 00 00    mov     eax, 5      //eax=5
.text:1001125D 8B 4A F8          mov     ecx, [edx-8]    //ecx=许可证号的长度
.text:10011260 3B C8             cmp     ecx, eax
.text:10011262 7D 02             jge     short loc_10011266
.text:10011264 8B C1             mov     eax, ecx
.text:10011266
.text:10011266                   loc_10011266:                 ; CODE XREF: ISwdAudioEncoder__LicenseNumber_put+C2j
.text:10011266 8B 0D 30 D0 01 10 mov     ecx, off_1001D030
.text:1001126C 6A 00             push    0
.text:1001126E 6A 00             push    0
.text:10011270 8D 55 0C          lea     edx, [ebp+lpString]
.text:10011273 89 4D 0C          mov     [ebp+lpString], ecx
.text:10011276 50                push    eax
.text:10011277 52                push    edx
.text:10011278 8B CE             mov     ecx, esi
.text:1001127A E8 91 14 00 00    call    sub_10012710    //取许可证号前5位
.text:1001127F 8D 45 0C          lea     eax, [ebp+lpString]  //eax指向许可证号前5位
.text:10011282 8D 4D FC          lea     ecx, [ebp+var_4]
.text:10011285 50                push    eax
.text:10011286 E8 85 12 00 00    call    sub_10012510
.text:1001128B 8D 4D 0C          lea     ecx, [ebp+lpString]
.text:1001128E E8 1D BB FF FF    call    sub_1000CDB0
.text:10011293 8B 4D FC          mov     ecx, [ebp+var_4]  //ecx指向许可证号前5位
.text:10011296 68 60 E8 01 10    push    offset aMped          ; "MPED-"  //内定的5位固定字符
.text:1001129B 51                push    ecx
.text:1001129C E8 BB 28 00 00    call    __mbscmp    //进行比较
.text:100112A1 8B 0D 34 D0 01 10 mov     ecx, off_1001D034
.text:100112A7 83 C4 08          add     esp, 8
.text:100112AA 8B 3D A8 A0 01 10 mov     edi, ds:InterlockedDecrement
.text:100112B0 85 C0             test    eax, eax    //测试比较结果
.text:100112B2 8B 45 FC          mov     eax, [ebp+var_4]
.text:100112B5 0F 95 C3          setnz   bl      //置标志为给bl
.text:100112B8 8D 50 F4          lea     edx, [eax-0Ch]
.text:100112BB 3B D1             cmp     edx, ecx
.text:100112BD 74 19             jz      short loc_100112D8
.text:100112BF 83 C0 F4          add     eax, 0FFFFFFF4h
.text:100112C2 50                push    eax                   ; lpAddend
.text:100112C3 FF D7             call    edi ; InterlockedDecrement
.text:100112C5 85 C0             test    eax, eax
.text:100112C7 7F 0F             jg      short loc_100112D8
.text:100112C9 8B 45 FC          mov     eax, [ebp+var_4]
.text:100112CC 83 C0 F4          add     eax, 0FFFFFFF4h
.text:100112CF 50                push    eax
.text:100112D0 E8 5B 1C 00 00    call    sub_10012F30
.text:100112D5 83 C4 04          add     esp, 4
.text:100112D8
.text:100112D8                   loc_100112D8:                 ; CODE XREF: ISwdAudioEncoder__LicenseNumber_put+11Dj
.text:100112D8                                                 ; ISwdAudioEncoder__LicenseNumber_put+127j
.text:100112D8 8B 06             mov     eax, [esi]  //eax指向指向输入的许可证号
.text:100112DA 84 DB             test    bl, bl    //测试bl标志位
.text:100112DC 74 3B             jz      short loc_10011319  //为0跳到10011319，否则位显示下面的
                “License number is wrong.”消息框
.text:100112DE 8B 48 F8          mov     ecx, [eax-8]
.text:100112E1 85 C9             test    ecx, ecx
.text:100112E3 74 1C             jz      short loc_10011301
.text:100112E5 8B 48 F4          mov     ecx, [eax-0Ch]
.text:100112E8 85 C9             test    ecx, ecx
.text:100112EA 7C 09             jl      short loc_100112F5
.text:100112EC 8B CE             mov     ecx, esi
.text:100112EE E8 4D C0 FF FF    call    sub_1000D340
.text:100112F3 EB 0C             jmp     short loc_10011301
.text:100112F5                   ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
.text:100112F5
.text:100112F5                   loc_100112F5:                 ; CODE XREF: ISwdAudioEncoder__LicenseNumber_put+14Aj
.text:100112F5 68 F0 18 02 10    push    offset String
.text:100112FA 8B CE             mov     ecx, esi
.text:100112FC E8 7F 14 00 00    call    sub_10012780
.text:10011301
.text:10011301                   loc_10011301:                 ; CODE XREF: ISwdAudioEncoder__LicenseNumber_put+143j
.text:10011301                                                 ; ISwdAudioEncoder__LicenseNumber_put+153j
.text:10011301 8B 4D 08          mov     ecx, [ebp+arg_0]
.text:10011304 6A 10             push    10h
.text:10011306 68 C8 E1 01 10    push    offset aMp3EncoderCont ; "MP3 Encoder Control"
.text:1001130B 68 68 E8 01 10    push    offset aLicenseNumberI ; "License number is wrong."
.text:10011310 8B 51 50          mov     edx, [ecx+50h]
.text:10011313 52                push    edx
.text:10011314 E9 EE 01 00 00    jmp     loc_10011507
.text:10011319                   ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
.text:10011319
.text:10011319                   loc_10011319:                 ; CODE XREF: ISwdAudioEncoder__LicenseNumber_put+13Cj
.text:10011319 8A 50 09          mov     dl, [eax+9]  //取输入的许可证号的第10位
.text:1001131C B1 2D             mov     cl, 2Dh  //cl='-'
.text:1001131E 3A D1             cmp     dl, cl    //比较许可证号的第10位是否等于'-'
.text:10011320 0F 85 A9 01 00 00 jnz     loc_100114CF  //不等跳走
.text:10011326 38 48 0E          cmp     [eax+0Eh], cl  //相等继续比较许可证号的第15位是否等于'-'
.text:10011329 0F 85 A0 01 00 00 jnz     loc_100114CF  //不等跳走
.text:1001132F 38 48 13          cmp     [eax+13h], cl  //相等继续比较许可证号的第20位是否等于'-'
.text:10011332 0F 85 97 01 00 00 jnz     loc_100114CF   //不等跳走
.text:10011338 8B 0E             mov     ecx, [esi]
.text:1001133A B8 04 00 00 00    mov     eax, 4
.text:1001133F 8B 49 F8          mov     ecx, [ecx-8]
.text:10011342 83 F9 09          cmp     ecx, 9
.text:10011345 7D 03             jge     short loc_1001134A
.text:10011347 8D 41 FB          lea     eax, [ecx-5]
.text:1001134A
.text:1001134A                   loc_1001134A:                 ; CODE XREF: ISwdAudioEncoder__LicenseNumber_put+1A5j
.text:1001134A 83 F9 05          cmp     ecx, 5
.text:1001134D 7D 02             jge     short loc_10011351
.text:1001134F 33 C0             xor     eax, eax
.text:10011351
.text:10011351                   loc_10011351:                 ; CODE XREF: ISwdAudioEncoder__LicenseNumber_put+1ADj
.text:10011351 8B 15 30 D0 01 10 mov     edx, off_1001D030
.text:10011357 6A 00             push    0
.text:10011359 6A 05             push    5
.text:1001135B 50                push    eax
.text:1001135C 8D 45 0C          lea     eax, [ebp+lpString]
.text:1001135F 8B CE             mov     ecx, esi
.text:10011361 50                push    eax
.text:10011362 89 55 0C          mov     [ebp+lpString], edx
.text:10011365 E8 A6 13 00 00    call    sub_10012710  //取输入的许可证号的6至9位
.text:1001136A 8D 4D 0C          lea     ecx, [ebp+lpString]  //ecx指向输入的许可证号的6至9位
.text:1001136D 51                push    ecx
.text:1001136E 8D 4D FC          lea     ecx, [ebp+var_4]
.text:10011371 E8 9A 11 00 00    call    sub_10012510    //将许可证号的6至9位保存到[0012ec78]
.text:10011376 8D 4D 0C          lea     ecx, [ebp+lpString]
.text:10011379 E8 32 BA FF FF    call    sub_1000CDB0
.text:1001137E 8B 55 FC          mov     edx, [ebp+var_4]
.text:10011381 52                push    edx    //edx指向输入的许可证号的6至9位
.text:10011382 E8 99 01 00 00    call    sub_10011520  //判断许可证号的每一位是否在0-9，a-f,A-F之间
.text:10011387 8B 15 34 D0 01 10 mov     edx, off_1001D034
.text:1001138D 66 A3 74 19 02 10 mov     word ptr dword_10021974, ax  //ax=许可证号的6至9位,保存到10021974
.text:10011393 8B 45 FC          mov     eax, [ebp+var_4]
.text:10011396 83 C4 04          add     esp, 4
.text:10011399 8D 48 F4          lea     ecx, [eax-0Ch]
.text:1001139C 3B CA             cmp     ecx, edx
```
