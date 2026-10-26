
``` 
; **************************************************
55389477     381429           cmp byte ptr ds:[ecx+ebp],dl            ; 和 LK[ebp+ecx] 处一个字节比较

; 这里也是爆破点,把 dl 换成 ch,永远找不到匹配

; **************************************************
5538947A     75 0C            jnz short pdflib.55389488               ; 不相等,跳转
5538947C     40               inc eax                                 ; 相等,继续下一个字节比较,下一个数据Index
5538947D     25 FF000000      and eax,0FF                             ; mask
55389482     41               inc ecx                                 ; 下一个数据Index
55389483     83F9 15          cmp ecx,15                              ; 一组数据是否比较完?
55389486   ^ 7C E2            jl short pdflib.5538946A                ; 没有,跳转,继续...
55389488     83F9 15          cmp ecx,15                              ; 一组数据是否比较完?
5538948B     75 16            jnz short pdflib.553894A3               ; 没有,跳转
5538948D     6A 00            push 0                                  ; 找到一组匹配的数据,Error
5538948F     6A 00            push 0
55389491     6A 00            push 0
55389493     6A 00            push 0
55389495     68 C2070000      push 7C2
5538949A     53               push ebx
5538949B     E8 60E0F7FF      call pdflib.55307500                    ; Error 处理
553894A0     83C4 18          add esp,18
553894A3     83C6 15          add esi,15                              ; 下一组数据Index
553894A6     81FE 98190000    cmp esi,1998                            ; 循环检查 直到 esi = 0x1998,没找到的话
553894AC   ^ 7C B3            jl short pdflib.55389461                ; 所有组都检查完? 没有,继续...
```

### license key 最后一段外貌检查,并转换成 16 进制整数

`LK[] = "XXXXXXX-XXXXXX-XXXXXX-XXXXXX"`

最后一段 6 字节必须是 '0-9'/'A-F' 字符集中的一个

`Data = 55409B38`

``` 
55409B38  B4 E1 B8 5C A6 E4 D2 78 C3 1E 5A 5C 6A E8 4D A6
55409B48  8E 1E 3C 2E E4 D1 55 9A 69 87 D2 63 8E 5C 39 4D
55409B58  A9 C9 55 27 55 2B B2 A9 39 AC 0F E1 C3 74 E4 65
```

'0-9'/'A-F' 入口在这里:

``` 
55409B68  00 01 02 03 04 05 06 07 08 09 00 00 00 00 00 00
55409B78  00 0A 0B 0C 0D 0E 0F 00 50 44 46 6C 69 62 20 6C
```

把 6 字节字符串序列转换成 16进制 整数,如 `"12AB3C" -> 0x0012AB3C`

``` 
res = 0
loop:
res << 4
"12" --> 查表 --> 0x01 0x02
--> (res + 0x01) 左移4位 + 0x02
--> res
--> loop
```

计算结果在 `edi` 中

``` 
553894AE     BE E9FFFFFF      mov esi,-17
553894B3     33FF             xor edi,edi                             ; edi = 0,用于保存计算结果
553894B5     8D45 17          lea eax,dword ptr ss:[ebp+17]           ; eax = LK[0x17]
553894B8     2BF5             sub esi,ebp
553894BA     8A48 FF          mov cl,byte ptr ds:[eax-1]              ; LK[n-1]
553894BD     8A10             mov dl,byte ptr ds:[eax]                ; LK[n]
553894BF     80F9 30          cmp cl,30                               ; '0'
553894C2     0F8C E4020000    jl pdflib.553897AC                      ; LK[n-1] 小于 '0',Error
553894C8     80F9 46          cmp cl,46                               ; 'F'
553894CB     0F8F DB020000    jg pdflib.553897AC                      ; LK[n-1] 大于 'F',Error
553894D1     80FA 30          cmp dl,30                               ; '0'
553894D4     0F8C D2020000    jl pdflib.553897AC                      ; LK[n] 小于 '0',Error
553894DA     80FA 46          cmp dl,46                               ; 'F'
553894DD     0F8F C9020000    jg pdflib.553897AC                      ; LK[n] 大于 'F',Error
553894E3     0FBEC9           movsx ecx,cl                            ; ecx = cl = LK[n-1]
553894E6     0FBED2           movsx edx,dl                            ; edx = dl = LK[n]
553894E9     0FBE89 389B4055  movsx ecx,byte ptr ds:[ecx+55409B38]    ; ecx = Data[ecx] = Data[ LK[n-1] ]
553894F0     0FBE92 389B4055  movsx edx,byte ptr ds:[edx+55409B38]    ; edx = Data[edx] = Data[ LK[n] ]
553894F7     C1E7 04          shl edi,4                               ; edi 乘以 16
553894FA     03CF             add ecx,edi                             ; ecx = ecx + edi
553894FC     83C0 02          add eax,2                               ; eax = eax + 2,调整数据 Index
553894FF     C1E1 04          shl ecx,4                               ; ecx 乘以 16
55389502     03CA             add ecx,edx                             ; ecx = ecx + edx
55389504     8BF9             mov edi,ecx                             ; 保存结果到 edi
55389506     8D0C06           lea ecx,dword ptr ds:[esi+eax]          ; 获取 已经检查/计算的字符个数
55389509     83F9 06          cmp ecx,6                               ; 是否检查/计算完毕?
5538950C   ^ 7C AC            jl short pdflib.553894BA                ; 没有,继续...
```

### license key 前三段查表计算过程(分隔符'-'不参与计算)

`LK[] = "XXXXXXX-XXXXXX-XXXXXX-XXXXXX"`,前三段长度 21 字节

`Data = 55407ED0 - 55407F24`,范围 `0x54`(84) = 21 x 4

``` 
res = ss:[esp+10] = 0
eax = LK[n],检查 LK[n] 是不是 '-',是,则忽略,继续下一个
edx = Data[ LK[n] * 4 ]
edx = edx + Data[n]
edx = edx AND 0x0000007F
edx = Data[ edx * 4 ]
edx:eax = edx * eax
res = res XOR edx
```

计算结果在 `ss:[esp+10]` 中

``` 
5538950E     8BF5             mov esi,ebp                             ; esi = ebp = LK[]
55389510     B9 D07E4055      mov ecx,pdflib.55407ED0                 ; ecx = Data[]
55389515     8A06             mov al,byte ptr ds:[esi]                ; al = LK[n]
55389517     8AD0             mov dl,al                               ; dl = al
55389519     80E2 7F          and dl,7F                               ; dl and 0x7f
5538951C     3C 2D            cmp al,2D                               ; al = '-' ?
5538951E     885424 44        mov byte ptr ss:[esp+44],dl             ; save dl
55389522     74 23            je short pdflib.55389547                ; al is '-',ingore
55389524     8B4424 44        mov eax,dword ptr ss:[esp+44]           ; eax = LK[n]
55389528     25 FF000000      and eax,0FF                             ; mask
5538952D     8B1485 D07E4055  mov edx,dword ptr ds:[eax*4+55407ED0]   ; edx = Data[ eax * 4 ]
55389534     0311             add edx,dword ptr ds:[ecx]              ; edx = edx + Data[ ecx ]
55389536     83E2 7F          and edx,7F                              ; mask
55389539     8B1495 D07E4055  mov edx,dword ptr ds:[edx*4+55407ED0]   ; edx = Data[ edx * 4 ]
55389540     0FAFD0           imul edx,eax                            ; edx * eax
55389543     315424 10        xor dword ptr ss:[esp+10],edx           ; tmp XOR edx
55389547     83C1 04          add ecx,4                               ; Adjust Data pointer
5538954A     46               inc esi                                 ; Adjust LK pointer
5538954B     81F9 247F4055    cmp ecx,pdflib.55407F24                 ; Is Data end?
55389551   ^ 7C C2            jl short pdflib.55389515                ; No,go on...
```

### license key 前三段查表计算结果 和 最后一段转换结果比较

``` 
55389553     8B4424 10        mov eax,dword ptr ss:[esp+10]           ; 前三段计算结果
55389557     25 FFFFFF00      and eax,0FFFFFF                         ; mask
5538955C     3BF8             cmp edi,eax                             ; 和最后一段转换结果比较
5538955E     74 16            je short pdflib.55389576                ; 相等,go on...
55389560     6A 00            push 0                                  ; 不相等,error
55389562     6A 00            push 0
55389564     6A 00            push 0
55389566     6A 00            push 0
55389568     68 C2070000      push 7C2
5538956D     53               push ebx
5538956E     E8 8DDFF7FF      call pdflib.55307500
55389573     83C4 18          add esp,18
```

### license key 第一段检查
