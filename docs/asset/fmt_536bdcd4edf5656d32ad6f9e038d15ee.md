
```
00505DCC   .  FF15 241040>call dword ptr ds:[<&MSVBVM60.__vbaLenBstr>]  ;  MSVBVM60.__vbaLenBstr  

00505DD2   .  8BC8         mov ecx,eax  

00505DD4   .  FF15 341140>call dword ptr ds:[<&MSVBVM60.__vbaI2I4>]     ;  MSVBVM60.__vbaI2I4  

00505DDA   >  8BD8         mov ebx,eax  

00505DDC   .  B8 01000000 mov eax,1  

00505DE1   .  66:3BD8      cmp bx,ax  

00505DE4   .  0F8C B30000>jl GtChrdHl.00505E9D  

00505DEA   .  8B4D 08      mov ecx,dword ptr ss:[ebp+8]  

00505DED   .  8945 A4      mov dword ptr ss:[ebp-5C],eax  

00505DF0   .  0FBFC3       movsx eax,bx  

00505DF3   .  8D55 9C      lea edx,dword ptr ss:[ebp-64]  

00505DF6   .  898D 44FFFF>mov dword ptr ss:[ebp-BC],ecx  

00505DFC   .  52           push edx  

00505DFD   .  8D8D 3CFFFF>lea ecx,dword ptr ss:[ebp-C4]  

00505E03   .  50           push eax  

00505E04   .  8D55 8C      lea edx,dword ptr ss:[ebp-74]  

00505E07   .  51           push ecx  

00505E08   .  52           push edx  

00505E09   .  C745 9C 020>mov dword ptr ss:[ebp-64],2  

00505E10   .  C785 3CFFFF>mov dword ptr ss:[ebp-C4],4008                ;  逆序取一次  

00505E1A   .  FF15 0C1140>call dword ptr ds:[<&MSVBVM60.#632>]          ;  MSVBVM60.rtcMidCharVar  

00505E20   .  8B45 D8      mov eax,dword ptr ss:[ebp-28]                ;  mid("机器码",len-i,1)  

00505E23   .  8D4D 8C      lea ecx,dword ptr ss:[ebp-74]  

00505E26   .  50           push eax  

00505E27   .  8D55 C4      lea edx,dword ptr ss:[ebp-3C]  

00505E2A   .  51           push ecx  

00505E2B   .  52           push edx  

00505E2C   .  FF15 C41140>call dword ptr ds:[<&MSVBVM60.__vbaStrVarVal>>;  MSVBVM60.__vbaStrVarVal  

00505E32   .  50           push eax                                      ;  2  

00505E33   .  FF15 481040>call dword ptr ds:[<&MSVBVM60.#516>]          ;  MSVBVM60.rtcAnsiValueBstr  

00505E39   .  8BC8         mov ecx,eax                                   ;  asc("2")->32  

00505E3B   .  FF15 601040>call dword ptr ds:[<&MSVBVM60.__vbaI2Abs>]    ;  MSVBVM60.__vbaI2Abs  

00505E41   .  0FBF4D D0    movsx ecx,word ptr ss:[ebp-30]               ;  26  

00505E45   .  0FBFC0       movsx eax,ax                                  

00505E48   .  0FAFC1       imul eax,ecx                                    ;  0x32*0x26  

00505E4B   .  0F80 040700>jo GtChrdHl.00506555  

00505E51   .  50           push eax  

00505E52   .  FF15 101040>call dword ptr ds:[<&MSVBVM60.__vbaStrI4>]    ;  MSVBVM60.__vbaStrI4  

00505E58   .  8BD0         mov edx,eax                                     ;  1900  

00505E5A   .  8D4D C0      lea ecx,dword ptr ss:[ebp-40]  

00505E5D   .  FFD6         call esi  

00505E5F   .  50           push eax  

00505E60   .  FF15 641040>call dword ptr ds:[<&MSVBVM60.__vbaStrCat>]   ;  MSVBVM60.__vbaStrCat  

00505E66   .  8BD0         mov edx,eax  

00505E68   .  8D4D D8      lea ecx,dword ptr ss:[ebp-28]  

00505E6B   .  FFD6         call esi  

00505E6D   .  8D55 C0      lea edx,dword ptr ss:[ebp-40]  

00505E70   .  8D45 C4      lea eax,dword ptr ss:[ebp-3C]  

00505E73   .  52           push edx  

00505E74   .  50           push eax  

00505E75   .  6A 02        push 2  

00505E77   .  FF15 1C1240>call dword ptr ds:[<&MSVBVM60.__vbaFreeStrLis>;  MSVBVM60.__vbaFreeStrList  

00505E7D   .  8D4D 8C      lea ecx,dword ptr ss:[ebp-74]  
```

第三个循环，逆序取机器码，ascii 码乘以上面的 ascii 右边一位的和 j，然后组成新串 `str2`

```
for i=len(mc) to 1 step -1

str2=  str2 & cstr(asc(mid(mc,i,1)) *j) 

next
```

```
00505E80   .  8D55 9C      lea edx,dword ptr ss:[ebp-64]  

00505E83   .  51           push ecx  

00505E84   .  52           push edx  

00505E85   .  6A 02        push 2  

00505E87   .  FFD7         call edi  

00505E89   .  83C8 FF      or eax,FFFFFFFF  

00505E8C   .  83C4 18      add esp,18  

00505E8F   .  66:03C3      add ax,bx  

00505E92   .  0F80 BD0600>jo GtChrdHl.00506555  

00505E98   .^. E9 3DFFFFFF jmp GtChrdHl.00505DDA  

00505E9D   >  8B45 D8      mov eax,dword ptr ss:[ebp-28]                 ;  "1900307819003002190029264370254619004522"  

00505EA0   .  50           push eax  

00505EA1   .  FF15 241040>call dword ptr ds:[<&MSVBVM60.__vbaLenBstr>]  ;  MSVBVM60.__vbaLenBstr  

00505EA7   .  8B4D 0C      mov ecx,dword ptr ss:[ebp+C]  

00505EAA   .  66:8B11      mov dx,word ptr ds:[ecx]  

00505EAD   .  66:6BD2 02   imul dx,dx,2                                    ;  14*2  

00505EB1   .  0F80 9E0600>jo GtChrdHl.00506555  

00505EB7   .  0FBFCA       movsx ecx,dx  

00505EBA   .  3BC1         cmp eax,ecx  

00505EBC   .^. 0F8C 04FFFF>jl GtChrdHl.00505DC6                           ;  比较获得的串是否够 40，不够继续  

00505EC2   .  8B5D 0C      mov ebx,dword ptr ss:[ebp+C]  

00505EC5   .  8D55 D8      lea edx,dword ptr ss:[ebp-28]  

00505EC8   .  8995 44FFFF>mov dword ptr ss:[ebp-BC],edx  

00505ECE   .  8D95 3CFFFF>lea edx,dword ptr ss:[ebp-C4]  

00505ED4   .  66:8B03      mov ax,word ptr ds:[ebx]  

00505ED7   .  C785 3CFFFF>mov dword ptr ss:[ebp-C4],4008  

00505EE1   .  66:6BC0 02   imul ax,ax,2  

00505EE5   .  0F80 6A0600>jo GtChrdHl.00506555  

00505EEB   .  0FBFC8       movsx ecx,ax  

00505EEE   .  51           push ecx  

00505EEF   .  8D45 9C      lea eax,dword ptr ss:[ebp-64]  

00505EF2   .  52           push edx  

00505EF3   .  50           push eax                                        ;  left(str2,40)  

00505EF4   .  FF15 701240>call dword ptr ds:[<&MSVBVM60.#617>]           ;  MSVBVM60.rtcLeftCharVar
```

取串 `str2` 的左边 40 位：

```
str2=left(str2,40)
```

```
00505EFA   .  8D4D 9C      lea ecx,dword ptr ss:[ebp-64]  

00505EFD   .  51           push ecx  

00505EFE   .  FF15 201040>call dword ptr ds:[<&MSVBVM60.__vbaStrVarMove>;  MSVBVM60.__vbaStrVarMove  

00505F04   .  8BD0         mov edx,eax                                     ;  UNICODE "1900307819003002190029264370254619004522"  

00505F06   .  8D4D D8      lea ecx,dword ptr ss:[ebp-28]  

00505F09   .  FFD6         call esi  

00505F0B   .  8D4D 9C      lea ecx,dword ptr ss:[ebp-64]  

00505F0E   .  FF15 1C1040>call dword ptr ds:[<&MSVBVM60.__vbaFreeVar>]   ;  MSVBVM60.__vbaFreeVar  

00505F14   .  66:8B0B      mov cx,word ptr ds:[ebx]  

00505F17   .  B8 01000000 mov eax,1  

00505F1C   .  66:6BC9 02   imul cx,cx,2  

00505F20   .  0F80 2F0600>jo GtChrdHl.00506555  

00505F26   .  898D C0FEFF>mov dword ptr ss:[ebp-140],ecx  

00505F2C   .  8945 E4      mov dword ptr ss:[ebp-1C],eax  

00505F2F   >  66:3BC1      cmp ax,cx                                       ;  循环，cx＝40  

00505F32   .  0F8F 360100>jg GtChrdHl.0050606E  

00505F38   .  8B55 CC      mov edx,dword ptr ss:[ebp-34]  

00505F3B   .  B9 02000000 mov ecx,2  

00505F40   .  8995 04FFFF>mov dword ptr ss:[ebp-FC],edx  

00505F46   .  894D A4      mov dword ptr ss:[ebp-5C],ecx  

00505F49   .  894D 9C      mov dword ptr ss:[ebp-64],ecx  

00505F4C   .  8D4D D8      lea ecx,dword ptr ss:[ebp-28]  

00505F4F   .  0FBFD8       movsx ebx,ax  

00505F52   .  8D55 9C      lea edx,dword ptr ss:[ebp-64]  

00505F55   .  898D 44FFFF>mov dword ptr ss:[ebp-BC],ecx  

00505F5B   .  52           push edx  

00505F5C   .  8D85 3CFFFF>lea eax,dword ptr ss:[ebp-C4]  

00505F62   .  53           push ebx  

00505F63   .  8D4D 8C      lea ecx,dword ptr ss:[ebp-74]  

00505F66   .  50           push eax  

00505F67   .  51           push ecx  

00505F68   .  C785 FCFEFF>mov dword ptr ss:[ebp-104],8  

00505F72   .  C785 3CFFFF>mov dword ptr ss:[ebp-C4],4008  

00505F7C   .  FF15 0C1140>call dword ptr ds:[<&MSVBVM60.#632>]           ;  MSVBVM60.rtcMidCharVar  

00505F82   .  B8 02000000 mov eax,2                                       ;  mid(str2,i,2)  

00505F87   .  8D55 D4      lea edx,dword ptr ss:[ebp-2C]  

00505F8A   .  8945 84      mov dword ptr ss:[ebp-7C],eax  

00505F8D   .  8985 7CFFFF>mov dword ptr ss:[ebp-84],eax  

00505F93   .  8D85 7CFFFF>lea eax,dword ptr ss:[ebp-84]  

00505F99   .  8995 24FFFF>mov dword ptr ss:[ebp-DC],edx  
```
