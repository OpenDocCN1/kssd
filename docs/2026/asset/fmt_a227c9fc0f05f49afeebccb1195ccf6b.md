
```
.text:100A42FF                 call    sub_10003740
.text:100A4304                 jmp     short loc_100A4298
.text:100A4306 ; ---------------------------------------------------------------------------
.text:100A4306
.text:100A4306 loc_100A4306:                           ; CODE XREF: sub_100A4220+D4 j
.text:100A4306                 lea     ecx, [esi+4]
.text:100A4309                 mov     [esp+40h+var_30], ecx
.text:100A430D                 call    sub_100A3660
.text:100A4312                 cmp     eax, [esi+30h]
.text:100A4315                 jz      short loc_100A432A
.text:100A4317                 mov     ecx, [esp+40h+arg_10]
.text:100A431B                 push    offset aInvalidOrTampe ; "Invalid or tampered license file suppli"...
.text:100A4320                 call    sub_10003740
.text:100A4325                 jmp     loc_100A4298
.text:100A432A ; ---------------------------------------------------------------------------
.text:100A432A
.text:100A432A loc_100A432A:                           ; CODE XREF: sub_100A4220+F5 j
.text:100A432A                 mov     eax, [esp+40h+arg_4]
.text:100A432E                 cmp     [esi+10h], eax
.text:100A4331                 jz      short loc_100A4346
.text:100A4333                 mov     ecx, [esp+40h+arg_10]
```

lic文件不匹配这个产品的时候

```
.text:100A4337                 push    offset aLicenseFileDoe ; "License file does not match software pr"...
```

```
.text:100A433C                 call    sub_10003740
.text:100A4341                 jmp     loc_100A4298
.text:100A4346 ; ---------------------------------------------------------------------------
.text:100A4346
.text:100A4346 loc_100A4346:                           ; CODE XREF: sub_100A4220+111 j
.text:100A4346                 mov     eax, [esp+40h+FileSystemFlags]
.text:100A434A                 cmp     [esi+18h], eax
.text:100A434D                 jge     short loc_100A4362
.text:100A434F
.text:100A434F loc_100A434F:                           ; CODE XREF: sub_100A4220+145 j
.text:100A434F                 mov     ecx, [esp+40h+arg_10]
```

lic文件不匹配这个版本的时候

```
.text:100A4353                 push    offset aLicenseFileD_0 ; "License file does not match software ve"...
```

```
.text:100A4358                 call    sub_10003740
.text:100A435D                 jmp     loc_100A4298
.text:100A4362 ; ---------------------------------------------------------------------------
.text:100A4362
.text:100A4362 loc_100A4362:                           ; CODE XREF: sub_100A4220+12D j
.text:100A4362                 cmp     [esi+14h], eax
.text:100A4365                 jg      short loc_100A434F
.text:100A4367                 cmp     dword ptr [esi+20h], 1
.text:100A436B                 mov     edi, [esp+40h+arg_10]
.text:100A436F                 jnz     short loc_100A43BD
.text:100A4371                 lea     ecx, [esp+40h+var_24]
.text:100A4375                 push    ecx
.text:100A4376                 call    sub_1002A8B0
.text:100A437B                 mov     edx, [esi+24h]
.text:100A437E                 mov     eax, [esi+28h]
.text:100A4381                 mov     ecx, [esi+2Ch]
.text:100A4384                 push    0FFFFFFFFh
.text:100A4386                 push    0
.text:100A4388                 push    0
.text:100A438A                 push    6
.text:100A438C                 push    edx
.text:100A438D                 push    eax
.text:100A438E                 push    ecx
.text:100A438F                 lea     ecx, [esp+5Ch+var_2C]
.text:100A4393                 call    sub_100A1EA0
.text:100A4398                 mov     edx, [esp+40h+var_28]
.text:100A439C                 mov     eax, [esp+40h+var_2C]
.text:100A43A0                 push    edx
.text:100A43A1                 push    eax
.text:100A43A2                 lea     ecx, [esp+48h+var_24]
.text:100A43A6                 call    sub_100A3610
.text:100A43AB                 test    al, al
.text:100A43AD                 jz      short loc_100A43BD
```

lic过期

```
.text:100A43AF                 push    offset aLicenseHasBeen ; "License has been expired."
```

```
.text:100A43B4                 mov     ecx, edi
.text:100A43B6                 xor     bl, bl
.text:100A43B8                 call    sub_10003740
.text:100A43BD
.text:100A43BD loc_100A43BD:                           ; CODE XREF: sub_100A4220+14F j
.text:100A43BD                                         ; sub_100A4220+18D j
.text:100A43BD                 cmp     dword ptr [esi+0Ch], 1E77h
.text:100A43C4                 jz      loc_100A4482
.text:100A43CA                 lea     ecx, [esp+40h+FileSystemFlags]
.text:100A43CE                 push    ecx             ; FileSystemFlags
.text:100A43CF                 call    sub_100A3750
.text:100A43D4                 add     esp, 4
.text:100A43D7                 lea     ecx, [esp+40h+arg_10]
.text:100A43DB                 call    sub_1005ED20
.text:100A43E0                 lea     edx, [esp+40h+arg_10]
.text:100A43E4                 push    edx
.text:100A43E5                 mov     ecx, esi
.text:100A43E7                 mov     byte ptr [esp+44h+var_4], 1
.text:100A43EC                 call    sub_100A3980
.text:100A43F1                 test    al, al
.text:100A43F3                 jnz     short loc_100A4416
```

软件未授权在这个计算机

```
.text:100A43F5                 push    offset aSoftwareIsNotA ; "Software is not authorized to run on th"...
```

```
.text:100A43FA                 mov     ecx, edi
.text:100A43FC                 call    sub_10003740
.text:100A4401                 lea     ecx, [esp+40h+arg_10]
.text:100A4405                 mov     byte_101A8CBA, 1
.text:100A440C                 call    sub_1005EDC0
.text:100A4411                 jmp     loc_100A4298
.text:100A4416 ; ---------------------------------------------------------------------------
.text:100A4416
.text:100A4416 loc_100A4416:                           ; CODE XREF: sub_100A4220+1D3 j
.text:100A4416                 lea     ecx, [esp+40h+arg_0]
.text:100A441A                 call    sub_1005ED20
.text:100A441F                 mov     edx, [esp+40h+arg_4]
.text:100A4423                 lea     eax, [esp+40h+arg_0]
.text:100A4427                 push    eax
.text:100A4428                 mov     eax, [esp+44h+FileSystemFlags]
.text:100A442C                 lea     ecx, [esp+44h+arg_10]
.text:100A4430                 push    ecx
.text:100A4431                 push    0
.text:100A4433                 push    edx
.text:100A4434                 push    eax
.text:100A4435                 mov     byte ptr [esp+54h+var_4], 2
.text:100A443A                 call    sub_100A40C0
.text:100A443F                 add     esp, 14h
.text:100A4442                 test    al, al
.text:100A4444                 jnz     short loc_100A4470
```

同上
