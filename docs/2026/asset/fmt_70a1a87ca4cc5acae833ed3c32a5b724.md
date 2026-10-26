
```
代码:
---
0095784E     3C 01               cmp al,1//AL=1  ?
00957850     0F85 9C000000       jnz 009578F2
00957856     EB 01               jmp short 00957859
00957858     9A 895C2418 8B07    call far 078B:18245C89
0095785F     8B28                mov ebp,dword ptr ds:[eax]
00957861     8307 04             add dword ptr ds:[edi],4
00957864     8BC5                mov eax,ebp
00957866     FF5424 04           call dword ptr ss:[esp+4]
0095786A     8BE8                mov ebp,eax
0095786C     8B07                mov eax,dword ptr ds:[edi]
0095786E     8A00                mov al,byte ptr ds:[eax]
00957870     884424 1D           mov byte ptr ss:[esp+1D],al
00957874     FF07                inc dword ptr ds:[edi]
00957876     807C24 3C 00        cmp byte ptr ss:[esp+3C],0
0095787B     74 11               je short 0095788E
0095787D     A1 64E79500         mov eax,dword ptr ds:[95E764]
00957882     A3 68E79500         mov dword ptr ds:[95E768],eax
00957887     C64424 3C 00        mov byte ptr ss:[esp+3C],0
0095788C     EB 0B               jmp short 00957899
0095788E     8B4424 34           mov eax,dword ptr ss:[esp+34]
00957892     8B00                mov eax,dword ptr ds:[eax]
00957894     A3 68E79500         mov dword ptr ds:[95E768],eax
00957899     8B0D 68E79500       mov ecx,dword ptr ds:[95E768]
0095789F     8A5424 1D           mov dl,byte ptr ss:[esp+1D]
009578A3     8B07                mov eax,dword ptr ds:[edi]
009578A5     E8 1AD5FFFF         call 00954DC4
009578AA     8B4424 24           mov eax,dword ptr ss:[esp+24]
009578AE     E8 91ACFEFF         call 00942544
009578B3     8BF0                mov esi,eax
009578B5     8B07                mov eax,dword ptr ds:[edi]
009578B7     50                  push eax
009578B8     8B4424 34           mov eax,dword ptr ss:[esp+34]
009578BC     50                  push eax
009578BD     A1 1CC69500         mov eax,dword ptr ds:[95C61C]
009578C2     8B00                mov eax,dword ptr ds:[eax]
009578C4     FFD0                call eax
009578C6     894424 38           mov dword ptr ss:[esp+38],eax
009578CA     6A 00               push 0
009578CC     8D5424 38           lea edx,dword ptr ss:[esp+38]
009578D0     8D4424 3C           lea eax,dword ptr ss:[esp+3C]
009578D4     8BCD                mov ecx,ebp
009578D6     E8 EDF8FFFF         call 009571C8//同样的加密CALL！ ★
009578DB     0305 B8C49500       add eax,dword ptr ds:[95C4B8]
009578E1     8906                mov dword ptr ds:[esi],eax
009578E3     8933                mov dword ptr ds:[ebx],esi//加密地址写入程序！NOP掉 ★
009578E5     33C0                xor eax,eax
009578E7     8A4424 1D           mov al,byte ptr ss:[esp+1D]
009578EB     0107                add dword ptr ds:[edi],eax
009578ED     E9 EEFCFFFF         jmp 009575E0//循环
---
```

```
代码:
---
009578F2     3C 04               cmp al,4//AL=4  ?  则不加密函数，下面单独处理
009578F4     75 5C               jnz short 00957952
009578F6     EB 01               jmp short 009578F9
009578F8     E9 8B078B28         jmp 29208088
009578FD     8307 04             add dword ptr ds:[edi],4
00957900     8B07                mov eax,dword ptr ds:[edi]
00957902     8A00                mov al,byte ptr ds:[eax]
00957904     884424 1D           mov byte ptr ss:[esp+1D],al
00957908     FF07                inc dword ptr ds:[edi]
0095790A     8BCD                mov ecx,ebp
0095790C     8A5424 1D           mov dl,byte ptr ss:[esp+1D]
00957910     8B07                mov eax,dword ptr ds:[edi]
00957912     E8 ADD4FFFF         call 00954DC4
00957917     8B4424 24           mov eax,dword ptr ss:[esp+24]
0095791B     E8 24ACFEFF         call 00942544
00957920     8BF0                mov esi,eax
00957922     8B07                mov eax,dword ptr ds:[edi]
00957924     50                  push eax
00957925     8B4424 34           mov eax,dword ptr ss:[esp+34]
00957929     50                  push eax
0095792A     A1 1CC69500         mov eax,dword ptr ds:[95C61C]
0095792F     8B00                mov eax,dword ptr ds:[eax]
00957931     FFD0                call eax
00957933     894424 38           mov dword ptr ss:[esp+38],eax
00957937     8B4424 38           mov eax,dword ptr ss:[esp+38]
0095793B     2B05 B8C49500       sub eax,dword ptr ds:[95C4B8]
00957941     8906                mov dword ptr ds:[esi],eax
00957943     8933                mov dword ptr ds:[ebx],esi//修改：jmp 00964030 ★
00957945     33C0                xor eax,eax
00957947     8A4424 1D           mov al,byte ptr ss:[esp+1D]
0095794B     0107                add dword ptr ds:[edi],eax
0095794D     E9 8EFCFFFF         jmp 009575E0//循环
---
```

```
代码:
---
00957952     3C 03               cmp al,3//AL=3  ?  这个UnPackMe没有AL=3的情况
00957954     75 7D               jnz short 009579D3//AL=不等于上面几个值，这里跳就出错了
00957956     EB 01               jmp short 00957959
00957959     8B07                mov eax,dword ptr ds:[edi]
0095795B     8A00                mov al,byte ptr ds:[eax]
0095795D     FF07                inc dword ptr ds:[edi]
0095795F     FEC8                dec al
00957961     74 09               je short 0095796C
00957963     FEC8                dec al
00957965     74 30               je short 00957997
00957967     E9 74FCFFFF         jmp 009575E0
0095796C     8B07                mov eax,dword ptr ds:[edi]
0095796E     8A00                mov al,byte ptr ds:[eax]
00957970     884424 1C           mov byte ptr ss:[esp+1C],al
00957974     FF07                inc dword ptr ds:[edi]
00957976     8B4424 24           mov eax,dword ptr ss:[esp+24]
0095797A     E8 C5ABFEFF         call 00942544
0095797F     8BF0                mov esi,eax
00957981     8933                mov dword ptr ds:[ebx],esi
00957983     8A4C24 1C           mov cl,byte ptr ss:[esp+1C]
00957987     8B5424 30           mov edx,dword ptr ss:[esp+30]
0095798B     8BC6                mov eax,esi
0095798D     E8 B2F4FFFF         call 00956E44
00957992     E9 49FCFFFF         jmp 009575E0
00957997     8B07                mov eax,dword ptr ds:[edi]
00957999     8B30                mov esi,dword ptr ds:[eax]
0095799B     8307 04             add dword ptr ds:[edi],4
0095799E     8BC6                mov eax,esi
009579A0     E8 9FABFEFF         call 00942544
009579A5     894424 20           mov dword ptr ss:[esp+20],eax
009579A9     8BCE                mov ecx,esi
009579AB     8B17                mov edx,dword ptr ds:[edi]
009579AD     8B4424 20           mov eax,dword ptr ss:[esp+20]
009579B1     E8 76DCFEFF         call 0094562C
009579B6     0137                add dword ptr ds:[edi],esi
009579B8     8B4424 24           mov eax,dword ptr ss:[esp+24]
009579BC     E8 83ABFEFF         call 00942544
009579C1     8BF0                mov esi,eax
009579C3     8B4424 20           mov eax,dword ptr ss:[esp+20]
009579C7     8906                mov dword ptr ds:[esi],eax
009579C9     8933                mov dword ptr ds:[ebx],esi
009579CB     E9 10FCFFFF         jmp 009575E0//循环
---
```

AL=1，AL＝2 这两个分支处理都使用了`call 009571C8`进行加密，所以进去动动手脚
