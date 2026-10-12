
# 功能强大的公式编辑器简单分析（已做出详细分析）

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-176.htm`](https://www.pediy.com/kssd/pediy07/pediy7-176.htm)

**破文作者**：`xsy3660`

**软件名称**：公式编辑器

**软件介绍**：是一个比 `word` 中自带的公式编辑器功能要强得多的软件。

**破解过程**：以下是简单的分析，算法还未完全搞定，一是本人较菜，二是时间不宽裕，若有人时间多，把算法搞定了别忘了说一声。本人看一下，觉得很好的一软件，所以赶快把它放上来与大家共享呀。

用 `OD` 装入后查找字符串，什么也没找到，后来到网上走了一趟，采用在 `USER32` 模板中找 `F3 A5 8B C8 83 E1 03 F3 A4 E8` 下断后分析得到如下结果。

```
0048436D     83FF>cmp edi,12                           ; 注册码长要>18 位
00484370     0F8C>jl MathType.004844E1
00484376     83FF>cmp edi,14                           ; 同时<20 位，即注册码为 19 位
00484379     0F8F>jg MathType.004844E1
0048437F     8B46>mov eax,dword ptr ds:[esi+8]         ; 假码给 eax
00484382     B3 2>mov bl,2D                            ; "-"给 bl
00484384     03C7 add eax,edi                          ; 假码的第一位加假码长度，即指向假码的最后一位的后一字符
00484386     3858>cmp byte ptr ds:[eax-D],bl            ; 第 7 位是否为"-"
00484389     0F85>jnz MathType.004844E1
0048438F     3858>cmp byte ptr ds:[eax-6],bl            ; 第 14 位是否为"-"
00484392     0F85>jnz MathType.004844E1
00484398     33ED xor ebp,ebp                          ; ebp 清零
0048439A     85FF test edi,edi
0048439C     7E 2>jle short MathType.004843C1
0048439E     8B4E>mov ecx,dword ptr ds:[esi+8]         ; 假码给 ecx
004843A1     8A04>mov al,byte ptr ds:[ecx+ebp]         ; 依次取第假码中的字符
004843A4     3AC3 cmp al,bl                            ; 比较是否为"-"
004843A6     74 1>je short MathType.004843BC            ; 是"-"则跳
004843A8     0FBE>movsx edx,al                         ; edx 的初值为 003E0000，edx<--al
004843AB     52   push edx
004843AC     E8 A>call MathType.0049C45E
004843B1     83C4>add esp,4
004843B4     85C0 test eax,eax
004843B6     0F84>je MathType.004844E1
004843BC     45   inc ebp
004843BD     3BEF cmp ebp,edi                           ; 循环控制
004843BF   ^ 7C D>jl short MathType.0048439E
004843C1     8D4C>lea ecx,dword ptr ss:[esp+10]
004843C5     E8 4>call MathType.0048BC10
004843CA     8D44>lea eax,dword ptr ss:[esp+10]
004843CE     6A 0>push 2
004843D0     50   push eax
004843D1     68 A>push MathType.004D59AC               ; ASCII "RegNumPrefix"
004843D6     B9 4>mov ecx,MathType.004EA740
004843DB     C744>mov dword ptr ss:[esp+60],0
004843E3     E8 D>call MathType.00491AC0
004843E8     8B4C2>mov ecx,dword ptr ss:[esp+14]
004843EC     8B542>mov edx,dword ptr ss:[esp+18]        ; “MTWE50”
004843F0     51    push ecx
004843F1     52    push edx
004843F2     8BCE  mov ecx,esi
004843F4     E8 57>call MathType.0048BF50               ; 关键，比较前 6 位是否为 MTWE50，进入
004843F9     85C0  test eax,eax
004843FB     0F85 >jnz MathType.004844D0                ; 若前 6 位不为 MTWE50，则跳至失败的地方
00484401     8D4C2>lea ecx,dword ptr ss:[esp+40]
00484405     E8 06>call MathType.0048BC10
0048440A     8B46 >mov eax,dword ptr ds:[esi+8]          ; 假码给 EAX
0048440D     8D4F >lea ecx,dword ptr ds:[edi-D]          ; 赋值 6
00484410     BB 01>mov ebx,1
00484415     51    push ecx
00484416     50    push eax
00484417     8D4C2>lea ecx,dword ptr ss:[esp+48]
0048441B     885C2>mov byte ptr ss:[esp+5C],bl
0048441F     E8 8C>call MathType.0048C1B0
00484424     8D4C2>lea ecx,dword ptr ss:[esp+34]
00484428     E8 E3>call MathType.0048BC10
0048442D     8B46 >mov eax,dword ptr ds:[esi+8]
00484430     83C7 >add edi,-0C
00484433     6A 06 push 6
00484435     57    push edi
00484436     50    push eax
00484437     8D4C2>lea ecx,dword ptr ss:[esp+40]
0048443B     C6442>mov byte ptr ss:[esp+60],2
00484440     E8 FB>call MathType.0048C140               ; 取出假码的第二段放到 138B8D0
00484445     8D4C2>lea ecx,dword ptr ss:[esp+28]
00484449     E8 C2>call MathType.0048BC10
0048444E     8B46 >mov eax,dword ptr ds:[esi+8]
00484451     6A 05 push 5
00484453     50    push eax
00484454     8D4C2>lea ecx,dword ptr ss:[esp+30]
00484458     C6442>mov byte ptr ss:[esp+5C],3
0048445D     E8 6E>call MathType.0048C1D0               ; 取出假码的第三段放到 138B958
00484462     8D4C2>lea ecx,dword ptr ss:[esp+1C]
00484466     E8 A5>call MathType.0048BC10
0048446B     8B442>mov eax,dword ptr ss:[esp+3C]        ; 第二段给 eax
0048446F     8B4C2>mov ecx,dword ptr ss:[esp+48]        ; 第一段给 ecx
00484473     8D542>lea edx,dword ptr ss:[esp+1C]
00484477     C6442>mov byte ptr ss:[esp+54],4
0048447C     52    push edx                              ; 压入 0
0048447D     50    push eax                              ; 压入第二段
0048447E     51    push ecx                              ; 压入第一段
0048447F     E8 CC>call MathType.00484250               ; 关键，此 call 是产生一字符串，下面的 call 用，进入
00484484     8B542>mov edx,dword ptr ss:[esp+30]
00484488     83C4 >add esp,0C
0048448B     8D4C2>lea ecx,dword ptr ss:[esp+28]
0048448F     52    push edx
00484490     E8 7B>call MathType.0048BF10               ; 上面的 call 产生的字符串与第三段进行比较，若相同
                                                          ; 则 eax 为 0
00484495     85C0  test eax,eax                         ; 若 eax 为 0，则跳至成功的地方
00484497     74 60 je short MathType.004844F9           ; 关键点，暴破改跳
00484499     C6442>mov byte ptr ss:[esp+54],3
0048449E     8D4C2>lea ecx,dword ptr ss:[esp+1C]
004844A2     E8 49>call MathType.0048BCF0
004844A7     8D4C2>lea ecx,dword ptr ss:[esp+28]
004844AB     C6442>mov byte ptr ss:[esp+54],2
004844B0     E8 3B>call MathType.0048BCF0
004844B5     8D4C2>lea ecx,dword ptr ss:[esp+34]
004844B9     885C2>mov byte ptr ss:[esp+54],bl
004844BD     E8 2E>call MathType.0048BCF0
004844C2     8D4C2>lea ecx,dword ptr ss:[esp+40]
004844C6     C6442>mov byte ptr ss:[esp+54],0
004844CB     E8 20>call MathType.0048BCF0
004844D0     8D4C2>lea ecx,dword ptr ss:[esp+10]
004844D4     C7442>mov dword ptr ss:[esp+54],-1
004844DC     E8 0F>call MathType.0048BCF0
004844E1     5F    pop edi
004844E2     5E    pop esi
004844E3     5D    pop ebp
004844E4     B8 03>mov eax,3
004844E9     5B    pop ebx
004844EA     8B4C2>mov ecx,dword ptr ss:[esp+3C]
004844EE     64:89>mov dword ptr fs:[0],ecx
004844F5     83C4 >add esp,48
004844F8     C3    retn
```

------------------------------------------------------------------------------------------

`0048447F     E8 CC>call MathType.00484250` 进入后：

```
00484250     8B542>mov edx,dword ptr ss:[esp+4]
00484254     83EC >sub esp,8
00484257     83C9 >or ecx,FFFFFFFF
0048425A     33C0  xor eax,eax
0048425C     53    push ebx
0048425D     55    push ebp
0048425E     56    push esi
0048425F     57    push edi
00484260     8BFA  mov edi,edx
00484262     6A 24 push 24                              ; 压入 24
00484264     F2:AE repne scas byte ptr es:[edi]
00484266     F7D1  not ecx
00484268     49    dec ecx
00484269     33FF  xor edi,edi
0048426B     57    push edi
0048426C     8D441>lea eax,dword ptr ds:[ecx+edx-5]
00484270     50    push eax                              ; 第一段压入
00484271     E8 A5>call MathType.0049CA1B               ; 进入，由假码第一段产生一数 A
00484276     8BE8  mov ebp,eax
00484278     8B442>mov eax,dword ptr ss:[esp+2C]        ; 假码的第二段给 eax
0048427C     6A 24 push 24                               ; 压入 24
0048427E     57    push edi
0048427F     50    push eax
00484280     E8 96>call MathType.0049CA1B               ; 进入，产生一数 B
00484285     8BD8  mov ebx,eax
00484287     83C4 >add esp,18
0048428A     3BDF  cmp ebx,edi
0048428C     75 05 jnz short MathType.00484293
0048428E     BB 01>mov ebx,1
00484293     8B742>mov esi,dword ptr ss:[esp+24]
00484297     6A 22 push 22
00484299     6A 28 push 28
0048429B     8BCE  mov ecx,esi
0048429D     E8 6E>call MathType.0048BD10
004842A2     895C2>mov dword ptr ss:[esp+14],ebx
004842A6     897C2>mov dword ptr ss:[esp+18],edi
004842AA     DF6C2>fild qword ptr ss:[esp+14]
004842AE     896C2>mov dword ptr ss:[esp+14],ebp
004842B2     897C2>mov dword ptr ss:[esp+18],edi
```



# 反汇编代码分析

## 1. 主逻辑循环
`004842B6     DF6C2>fild qword ptr ss:[esp+14]`

`004842BA     50    push eax`

`004842BB     DEC9  fmulp st(1),st                 ; 数 A 与 B 变成实数后相乘`

`004842BD     DD05 >fld qword ptr ds:[4B08B0]       ; 乘积再除 44099087（实数）`

`004842C3     E8 02>call MathType.0049E8CA`

`004842C8     DC05 >fadd qword ptr ds:[4B08A8]      ; 再加实数 1336337`

`004842CE     E8 7D>call MathType.0049CD50          ; 实数变成 16 进制数 C`

`004842D3     50    push eax                       ; 把 16 进制数压入`

`004842D4     E8 0A>call MathType.0049CCE3          ; 进入！！！（见后面）`

`004842D9     83C4 >add esp,0C`

`004842DC     8BCE  mov ecx,esi`

`004842DE     E8 5D>call MathType.0048BD40`

`004842E3     397E >cmp dword ptr ds:[esi+4],edi`

`004842E6     7E 3D jle short MathType.00484325`

`004842E8     8B4E >mov ecx,dword ptr ds:[esi+8]`

`004842EB     8A043>mov al,byte ptr ds:[ecx+edi]`

`004842EE     3C 49 cmp al,49`

`004842F0     74 21 je short MathType.00484313`

`004842F2     3C 69 cmp al,69`

`004842F4     74 1D je short MathType.00484313`

`004842F6     3C 4F cmp al,4F`

`004842F8     74 14 je short MathType.0048430E`

`004842FA     3C 6F cmp al,6F`

`004842FC     74 10 je short MathType.0048430E`

`004842FE     0FBED>movsx edx,al`

`00484301     57    push edi`

`00484302     52    push edx`

`00484303     E8 D6>call MathType.0049E2DE`

`00484308     83C4 >add esp,4`

`0048430B     50    push eax`

`0048430C     EB 08 jmp short MathType.00484316`

`0048430E     57    push edi`

`0048430F     6A 5A push 5A`

`00484311     EB 03 jmp short MathType.00484316`

`00484313     57    push edi`

`00484314     6A 59 push 59`

`00484316     8BCE  mov ecx,esi`

`00484318     E8 03>call MathType.0048C120`

`0048431D     8B46 >mov eax,dword ptr ds:[esi+4]`

`00484320     47    inc edi`

`00484321     3BF8  cmp edi,eax`

`00484323    ^ 7C C3 jl short MathType.004842E8`

`00484325     5F    pop edi`

`00484326     5E    pop esi`

`00484327     5D    pop ebp`

`00484328     5B    pop ebx`

`00484329     83C4 >add esp,8`

`0048432C     C3    retn`

---

## 2. 函数 `MathType.0049CA1B`
`00484271     E8 A5>call MathType.0049CA1B 进入后：`

`0049CA1B     6A 01 push 1`

`0049CA1D     FF742>push dword ptr ss:[esp+10]`

`0049CA21     FF742>push dword ptr ss:[esp+10]`

`0049CA25     FF742>push dword ptr ss:[esp+10]`

`0049CA29     E8 E5>call MathType.0049C813          ; 进入`

`0049CA2E     83C4 >add esp,10`

`0049C813     55    push ebp`

`0049C814     8BEC  mov ebp,esp`

`0049C816     83EC >sub esp,0C`

`0049C819     53    push ebx`

`.........`
`...............`

`0049C8D3     46    inc esi`

`0049C8D4     46    inc esi`

`0049C8D5     8975 >mov dword ptr ss:[ebp-4],esi`

`0049C8D8     83C8 >or eax,FFFFFFFF`

`0049C8DB     33D2  xor edx,edx`

`0049C8DD     F775 >div dword ptr ss:[ebp+10]`

`0049C8E0     BF 03>mov edi,103`

`0049C8E5     8945 >mov dword ptr ss:[ebp-C],eax`

`0049C8E8     833D >cmp dword ptr ds:[4D6F5C],1     ; 此循环产生一数`

`0049C8EF     0FB6F>movzx esi,bl                    ；假码中的字符给 esi`

`0049C8F2     7E 0C jle short MathType.0049C900`

`0049C8F4     6A 04 push 4`

`0049C8F6     56    push esi`

`0049C8F7     E8 CB>call MathType.004A0AC7`

`0049C8FC     59    pop ecx`

`0049C8FD     59    pop ecx`

`0049C8FE     EB 0B jmp short MathType.0049C90B`

`0049C900     A1 50>mov eax,dword ptr ds:[4D6D50]`

`0049C905     8A047>mov al,byte ptr ds:[eax+esi*2]`

`0049C908     83E0 >and eax,4`

`0049C90B     85C0  test eax,eax`

`0049C90D     74 08 je short MathType.0049C917`

`0049C90F     0FBEC>movsx ecx,bl`

`0049C912     83E9 >sub ecx,30                    ;  若假码中是数字，则 16 制码减 30`

`0049C915     EB 32 jmp short MathType.0049C949`

`0049C917     833D >cmp dword ptr ds:[4D6F5C],1`

`0049C91E     7E 0B jle short MathType.0049C92B`

`0049C920     57    push edi`

`0049C921     56    push esi`

`0049C922     E8 A0>call MathType.004A0AC7`

`0049C927     59    pop ecx`

`0049C928     59    pop ecx`

`0049C929     EB 0B jmp short MathType.0049C936`

`0049C92B     A1 50>mov eax,dword ptr ds:[4D6D50]`

`0049C930     66:8B>mov ax,word ptr ds:[eax+esi*2]`

`0049C934     23C7  and eax,edi`

`0049C936     85C0  test eax,eax`

`0049C938     74 4A je short MathType.0049C984`

`0049C93A     0FBEC>movsx eax,bl`

`0049C93D     50    push eax`

`0049C93E     E8 9B>call MathType.0049E2DE`

`0049C943     59    pop ecx`

`0049C944     8BC8  mov ecx,eax`

`0049C946     83E9 >sub ecx,37                    ; 若假码中是字母，则 16 制码减 37`

`0049C949     3B4D >cmp ecx,dword ptr ss:[ebp+10]`

`0049C94C     73 36 jnb short MathType.0049C984`

`0049C94E     8B75 >mov esi,dword ptr ss:[ebp-8]    ;[ebp-8]（初值为 0）的值给 esi`

`0049C951     834D >or dword ptr ss:[ebp+14],8`

`0049C955     3B75 >cmp esi,dword ptr ss:[ebp-C]`

`0049C958     72 14 jb short MathType.0049C96E`

`0049C95A     75 0C jnz short MathType.0049C968`

`0049C95C     83C8 >or eax,FFFFFFFF`

`0049C95F     33D2  xor edx,edx`

`0049C961     F775 >div dword ptr ss:[ebp+10]`

`0049C964     3BCA  cmp ecx,edx`

`0049C966     76 06 jbe short MathType.0049C96E`

`0049C968     834D >or dword ptr ss:[ebp+14],4`

`0049C96C     EB 09 jmp short MathType.0049C977`

`0049C96E     0FAF7>imul esi,dword ptr ss:[ebp+10]  ; esi*乘 24`

`0049C972     03F1  add esi,ecx                   ; 上面的差值与上面的乘值相加`

`0049C974     8975 >mov dword ptr ss:[ebp-8],esi   ；相加后给[ebp-8]`

`0049C977     8B45 >mov eax,dword ptr ss:[ebp-4]`

`0049C97A     FF45 >inc dword ptr ss:[ebp-4]      ;假码中的字符依次`

`0049C97D     8A18  mov bl,byte ptr ds:[eax]        ; 给 bl`

`0049C97F    ^ E9 64>jmp MathType.0049C8E8`

`0049C984     8B4D >mov ecx,dword ptr ss:[ebp+14]`

`0049C987     FF4D >dec dword ptr ss:[ebp-4]`

`0049C98A     8B55 >mov edx,dword ptr ss:[ebp+C]`

`0049C98D     F6C1 >test cl,8`

`0049C990     75 10 jnz short MathType.0049C9A2`

`0049C992     85D2  test edx,edx`

`0049C994     74 06 je short MathType.0049C99C`

`0049C996     8B45 >mov eax,dword ptr ss:[ebp+8]`

`0049C999     8945 >mov dword ptr ss:[ebp-4],eax`

---

## 3. 函数 `MathType.0049CCE3`
`004842D4     E8 0A>call MathType.0049CCE3  进入后：`

`0049CCE3     6A 00 push 0`

`0049CCE5     FF742>push dword ptr ss:[esp+10]`

`0049CCE9     FF742>push dword ptr ss:[esp+10]`

`0049CCED     FF742>push dword ptr ss:[esp+10]`

`0049CCF1     E8 67>call MathType.0049CC5D          ; 进入`

`0049CCF6     8B442>mov eax,dword ptr ss:[esp+18]`

`0049CCFA     83C4 >add esp,10`

---

## 4. 函数 `MathType.0049CC5D` 部分逻辑
`0049CC67     53    push ebx`

`0049CC68     56    push esi`

`0049CC69     57    push edi`

`0049CC6A     74 0B je short MathType.0049CC77`

`0049CC6C     8B75 >mov esi,dword ptr ss:[ebp+8]`

`0049CC6F     C601 >mov byte ptr ds:[ecx],2D`

`0049CC72     41    inc ecx`

`0049CC73     F7DE  neg esi`

`0049CC75     EB 03 jmp short MathType.0049CC7A`

`0049CC77     8B75 >mov esi,dword ptr ss:[ebp+8]   ; 数 C 给 esi`

`0049CC7A     8BF9  mov edi,ecx`

`0049CC7C     8BC6  mov eax,esi                   ; 把前次循环的商给 esi`

`0049CC7E     33D2  xor edx,edx`

`0049CC80     F775 >div dword ptr ss:[ebp+10]     ; 除以固定数 22`

`0049CC83     8BC6  mov eax,esi`

`0049CC85     8BDA  mov ebx,edx                  ; 余数给 ebx`

`0049CC87     33D2  xor edx,edx`

`0049CC89     F775 >div dword ptr ss:[ebp+10]`

`0049CC8C     83FB >cmp ebx,9                   ; 余数与 9 比较`

`0049CC8F     8BF0  mov esi,eax                  ; 商给 esi`

`0049CC91     76 05 jbe short MathType.0049CC98`

`0049CC93     80C3 >add bl,57                   ; 加 57`

`0049CC96     EB 03 jmp short MathType.0049CC9B`

`0049CC98     80C3 >add bl,30                   ; 加 30`

`0049CC9B     8819  mov byte ptr ds:[ecx],bl      ; 储存 bl`

`0049CC9D     41    inc ecx`

`0049CC9E     85F6  test esi,esi`

`0049CCA0    ^ 77 DA ja short MathType.0049CC7C  ; 此循环求一字符串（注册码的第三段）`



# 汇编代码片段

```
0049CCA2     8021 >and byte ptr ds:[ecx],0  
0049CCA5     49    dec ecx  
0049CCA6     8A17  mov dl,byte ptr ds:[edi]  
0049CCA8     8A01  mov al,byte ptr ds:[ecx]  
0049CCAA     8811  mov byte ptr ds:[ecx],dl  
0049CCAC     8807  mov byte ptr ds:[edi],al  
0049CCAE     49    dec ecx  
0049CCAF     47    inc edi  
0049CCB0     3BF9  cmp edi,ecx  
0049CCB2   ^ 72 F2 jb short MathType.0049CCA6  
0049CCB4     5F    pop edi  
0049CCB5     5E    pop esi  
0049CCB6     5B    pop ebx  
0049CCB7     5D    pop ebp  
```

本文在好好学习 2 兄的帮助下才完整的把整个算法详细的写出来以菜鸟们共享，以是感谢好好学习 2 兄 2 的帮助。好好学习 2 的算法里好像在第一个数的求算时有点不对，大家自己也可以去看看。本想写一 `VB` 注册机源码的，可时间太晚了，明天还得上早班呀，就算了。

## 算法计算示例

### 第一段计算

*   `(W)` $57-37+0*24=20$
*   `(E)` $45-37+20*24=48E$
*   `(5)` $35-30+48E*24=A3FD$
*   `(2)` $32-30+A3FD*24=170F96$
*   `(8)` $38-30+170F96*24=33E3120$

### 第二段计算（假码第二段 6 位字符：`888888` Hex 为：`38 38 38 38 38 38`）

*   `(8)` $38-30+0*24=8$
*   `(8)` $38-30+8*24=128$
*   `(8)` $38-30+128*24=29A8$
*   `(8)` $38-30+29A8*24=5DBA8$
*   `(8)` $38-30+5DBA8*24=D2E3A8$
*   `(8)` $38-30+D2E3A8*24=1DA803A8$

注：`24`、`37`、`30` 为固定值。（字符为字母减 `37`、为数字减 `30`）

### 后续推导

1.  `"33E3120"` 10 进制转换 = `54407456`
2.  `"1DA803A8"` 10 进制转换 = `497550248`
3.  `54407456 * 497550248 = 27070443225849088`
4.  `27070443225849088 MOD 44099087`（固定值） = `27625742`
5.  `27625742 + 1336337`（固定值） = `28962079`
6.  `"28962079"` 16 进制转换 = `1B9ED1F`

#### 字符转换过程

*   `1B9ED1F / 22 = CFF71`
*   `1B9ED1F MOD 22 = 1D`
*   `1D`（大于 `9`） + `57` = `74` (`t`)
*   `CFF71 / 22 = 61DD`
*   `CFF71 MOD 22 = 17`
*   `17`（大于 `9`） + `57` = `6E` (`n`)
*   `61DD / 22 = 2E0`
*   `61DD MOD 22 = 1D`
*   `1D`（大于 `9`） + `57` = `74` (`t`)
*   `2E0 / 22 = 15`
*   `2E0 MOD 22 = 16`
*   `16`（大于 `9`） + `57` = `6D` (`m`)
*   `15 / 22 = 0`
*   `15 MOD 22 = 15`
*   `15`（大于 `9`） + `57` = `6C` (`l`)

注：`22` 为固定值。

### 结果

最后得出真注册码为：`MTWE528-888888-LMTNT`

`4dd466d0e1e0cc123afbe504da2462ca`
