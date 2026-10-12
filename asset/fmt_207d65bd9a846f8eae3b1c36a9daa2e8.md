
## 2、第 2 个点

```
0087A1B9     8938               mov dword ptr ds:[eax],edi  
0087A1BB     B3 B8              mov bl,0B8  
0087A1BD     83E8 04            sub eax,4  
0087A1C0     68 4D3C7652        push 52763C4D  
0087A1C5     51                 push ecx  
0087A1C6     80CA 05            or dl,5  
0087A1C9     59                 pop ecx  
0087A1CA     5A                 pop edx  
0087A1CB     4E                 dec esi  
0087A1CC     0F85 18000000      jnz 0087A1EA  
0087A1D2     66:81F1 80F3       xor cx,0F380  
0087A1D7     E9 3A000000        jmp 0087A216  
```

// 解码循环 第 2 个点

// `Patch ②`、`E9 8C7E0200` `jmp 008A2068`

### Patch 代码

```
008A2068     C705 D8A18700 3A00> mov dword ptr ds:[87A1D8],3A  
```

// 还原 `0087A1D8` 处修改的代码

```
008A2072     C705 CFA28700 E9B6> mov dword ptr ds:[87A2CF],27DB6E9  
```

// `Patch` 第 3 个点

```
008A207C     66:C705 D3A28700 0> mov word ptr ds:[87A2D3],0FF00  
008A2085     E9 4D81FDFF        jmp 0087A1D7  
```

————————————————————————

## 3、第 3 个点

```
0087A2C4     83EA 04            sub edx,4  
0087A2C7     BB E633514B        mov ebx,4B5133E6  
0087A2CC     83EE 01            sub esi,1  
0087A2CF     0F85 87FFFFFF      jnz 0087A25C  
```

// 解码循环 第 3 个点

// `Patch ③`、`E9 B67D0200` `jmp 008A208A`

### Patch 代码

```
008A208A     0F85 CC81FDFF      jnz 0087A25C  
008A2090     C705 CFA28700 0F85> mov dword ptr ds:[87A2CF],FF87850F  
008A209A     66:C705 D3A28700 F> mov word ptr ds:[87A2D3],0FFFF  
```

// 还原 `0087A2CF` 处修改的代码

```
008A20A3     C705 7FA38700 E937> mov dword ptr ds:[87A37F],27D37E9  
008A20AD     66:C705 83A38700 0> mov word ptr ds:[87A383],0FF00  
```

// `Patch` 第 4 个点

```
008A20B6     E9 1A82FDFF        jmp 0087A2D5  
```

————————————————————————

## 4、第 4 个点

```
0087A34D     81EF F17FF02D      sub edi,2DF07FF1  
0087A353     81F3 B6D34803      xor ebx,348D3B6  
0087A359     66:BF 62DB         mov di,0DB62  
0087A35D     81C3 B7C8470B      add ebx,0B47C8B7  
0087A363     8918               mov dword ptr ds:[eax],ebx  
0087A365     81D7 61475018      adc edi,18504761  
0087A36B     81E8 47B71B45      sub eax,451BB747  
0087A371     B1 80              mov cl,80  
0087A373     81C0 43B71B45      add eax,451BB743  
0087A379     BA 0C25D675        mov edx,75D6250C  
0087A37E     4E                 dec esi  
0087A37F     0F85 B4FFFFFF      jnz 0087A339  
```

// 解码循环 第 4 个点

// `Patch ④`、`E9 377D0200` `jmp 008A20BB`

### Patch 代码

```
008A20BB     0F85 7882FDFF      jnz 0087A339  
```

// `0087A37F` 代码挪这里执行

```
008A20C1     C705 7FA38700 0F85> mov dword ptr ds:[87A37F],FFB4850F  
008A20CB     66:C705 83A38700 F> mov word ptr ds:[87A383],0FFFF  
```

// 还原 `0087A37F` 处修改的代码

```
008A20D4     C705 27A48700 E9C0> mov dword ptr ds:[87A427],27CC0E9  
008A20DE     66:C705 2BA48700 0> mov word ptr ds:[87A42B],0FF00  
```

// `Patch` 第 5 个点

```
008A20E7     E9 9982FDFF        jmp 0087A385  
```

————————————————————————

## 5、第 5 个点

```
0087A3F4     59                 pop ecx  
0087A3F5     5A                 pop edx  
0087A3F6     81E8 9A2CB57B      sub eax,7BB52C9A  
0087A3FC     50                 push eax  
0087A3FD     8F041F             pop dword ptr ds:[edi+ebx]  
0087A400     66:BE D174         mov si,74D1  
0087A404     66:BE 09ED         mov si,0ED09  
0087A408     81EB 2FB9E712      sub ebx,12E7B92F  
0087A40E     B9 7DD3FF76        mov ecx,76FFD37D  
0087A413     81C3 2BB9E712      add ebx,12E7B92B  
0087A419     0F8B 02000000      jpo 0087A421  
0087A41F     B2 72              mov dl,72  
0087A421     81FB D8FAFFFF      cmp ebx,-528  
0087A427     0F85 91FFFFFF      jnz 0087A3BE  
```

// 解码循环 第 5 个点

// `Patch ⑤`、`E9 C07C0200` `jmp 008A20EC`

### Patch 代码

```
008A20EC     0F85 CC82FDFF      jnz 0087A3BE  
```

// `0087A427` 代码挪这里执行

```
008A20F2     C705 27A48700 0F85> mov dword ptr ds:[87A427],FF91850F  
008A20FC     66:C705 2BA48700 F> mov word ptr ds:[87A42B],0FFFF  
```

// 还原 `0087A427` 处修改的代码

```
008A2105     C705 97A58700 E981> mov dword ptr ds:[87A597],27B81E9  
008A210F     66:C705 9BA58700 0> mov word ptr ds:[87A59B],0  
```

// `Patch` 第 6 个点

```
008A2118     E9 1083FDFF        jmp 0087A42D  
```

————————————————————————

## 6、第 6 个点 获取壳代码重定位基址

```
0087A517     FF95 F0030000      call dword ptr ss:[ebp+3F0] ; kernel32.VirtualAlloc  
0087A51D     8985 31040000      mov dword ptr ss:[ebp+431],eax  
```

// `[ebp+431]=[0087A869]=01280000` 壳代码基址

```
0087A523     8985 D0010000      mov dword ptr ss:[ebp+1D0],eax  
```

// `[ebp+1D0]=[0087A608]=01280000`

```
0087A529     64:67:A1 0000      mov eax,dword ptr fs:[0]  
0087A52E     8985 2D040000      mov dword ptr ss:[ebp+42D],eax  
0087A534     8B55 5B            mov edx,dword ptr ss:[ebp+5B]  
0087A537     8B85 D0010000      mov eax,dword ptr ss:[ebp+1D0]  
0087A53D     8902               mov dword ptr ds:[edx],eax  
0087A53F     8B85 08040000      mov eax,dword ptr ss:[ebp+408]  
0087A545     8942 04            mov dword ptr ds:[edx+4],eax  
0087A548     8D85 9F030000      lea eax,dword ptr ss:[ebp+39F]  
0087A54E     8B40 55            mov eax,dword ptr ds:[eax+55]  
0087A551     8942 08            mov dword ptr ds:[edx+8],eax  
0087A554     8B85 EC030000      mov eax,dword ptr ss:[ebp+3EC]  
0087A55A     8942 10            mov dword ptr ds:[edx+10],eax  
0087A55D     8B85 E8030000      mov eax,dword ptr ss:[ebp+3E8]  
0087A563     8942 14            mov dword ptr ds:[edx+14],eax  
0087A566     8B95 CC010000      mov edx,dword ptr ss:[ebp+1CC]  
0087A56C     BB F8010000        mov ebx,1F8  
0087A571     8B7C1A 0C          mov edi,dword ptr ds:[edx+ebx+C]  
0087A575     0BFF               or edi,edi  
0087A577     74 1E              je short 0087A597  
0087A579     8B4C1A 10          mov ecx,dword ptr ds:[edx+ebx+10]  
0087A57D     0BC9               or ecx,ecx  
0087A57F     74 11              je short 0087A592  
0087A581     03BD D0010000      add edi,dword ptr ss:[ebp+1D0]  
0087A587     8B741A 14          mov esi,dword ptr ds:[edx+ebx+14]  
0087A58B     03F2               add esi,edx  
0087A58D     C1F9 02            sar ecx,2  
0087A590     F3:A5              rep movs dword ptr es:[edi],dword ptr ds:[esi]  
0087A592     83C3 28            add ebx,28  
0087A595     EB DA              jmp short 0087A571  
0087A597     8B85 CC010000      mov eax,dword ptr ss:[ebp+1CC]  
```

// 解码循环 第 6 个点

// `Patch ⑥`、`E9 817B0200` `jmp 008A211D`

### Patch 代码

```
008A211D     C705 97A58700 8B85> mov dword ptr ds:[87A597],1CC858B  
008A2127     C605 9BA58700 00    mov byte ptr ds:[87A59B],0  
```

// 还原 `0087A597` 处修改的代码

```
008A212E     8B85 31040000      mov eax,dword ptr ss:[ebp+431]  
```

// 取得壳代码基址

```
008A2134     C780 ED100300 684C> mov dword ptr ds:[eax+310ED],8A214C68  
008A213E     66:C780 F1100300 0> mov word ptr ds:[eax+310F1],0C300  
```

// `Patch` 第 7 个点 `012B10ED`

```
008A2147     E9 4B84FDFF        jmp 0087A597  
```

————————————————————————

## 7、第 7 个点

```
012B10C4     FF95 79294400      call dword ptr ss:[ebp+442979]  
012B10CA     8985 75294400      mov dword ptr ss:[ebp+442975],eax  
012B10D0     8D9D 452A4400      lea ebx,dword ptr ss:[ebp+442A45]  
012B10D6     50                 push eax  
012B10D7     53                 push ebx  
012B10D8     E8 74050000        call 012B1651  
012B10DD     8BC8               mov ecx,eax  
012B10DF     8DBD 452A4400      lea edi,dword ptr ss:[ebp+442A45]  
012B10E5     8BB5 75294400      mov esi,dword ptr ss:[ebp+442975]  
012B10EB     F3:A4              rep movs byte ptr es:[edi],byte ptr ds:[esi]  
012B10ED     8B85 75294400      mov eax,dword ptr ss:[ebp+442975]  
```

// 第 7 个点

// `Patch ⑦`、

```
012B10ED     68 4C218A00        push 8A214C  
012B10F2     C3                 retn  
```

### Patch 代码

```
008A214C     A1 69A88700        mov eax,dword ptr ds:[87A869]  
```

// 取得壳代码基址

```
008A2151     C780 ED100300 8B85> mov dword ptr ds:[eax+310ED],2975858B  
008A215B     66:C780 F1100300 4> mov word ptr ds:[eax+310F1],44  
```

// 还原 `012B10ED` 处修改的代码
