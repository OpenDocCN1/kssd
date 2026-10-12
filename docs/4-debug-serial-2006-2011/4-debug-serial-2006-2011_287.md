
# 对 delphi 中 ImageEn 组件的注册分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-330.htm`](https://www.pediy.com/kssd/pediy09/pediy09-330.htm)
>
> 【文章标题】: 对 delphi 中 ImageEn 组件的注册分析  
> 【文章作者】: `sunsjw`  
> 【下载地址】: `http://www.hicomponents.com/downloads/232/DImageEn2007.msi`  
> 【加壳方式】: 无  
> 【编写语言】: delphi  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

这是一个 Delphi 图像处理的组件，功能强大，用这个可以开发简单的图像处理软件。

我下载的是 FOR delphi2007 版本的，其它版本的算法就不知道了。

## 开始

注程序在安装目录里的 `setup.exe`，先用 `peid` 查，无壳。既然是 delphi 的东西就要 `dede` 出场了。

用 `dede` 分析找到注册按钮的事件，反汇编如下

* Reference to control `Edit2` : `TEdit`
|
```
00447A9A   8B80DC020000           mov     eax, [eax+$02DC]
```

* Reference to: `controls.TControl.GetText(TControl):TCaption;`
|
```
00447AA0   E8C3E2FDFF             call    00425D68
00447AA5   837DE000               cmp     dword ptr [ebp-$20], +$00
00447AA9   746D                   jz      00447B18
00447AAB   6A12                   push    $12
00447AAD   6873010000             push    $00000173
00447AB2   8D55DC                 lea     edx, [ebp-$24]
00447AB5   8B45FC                 mov     eax, [ebp-$04]
```

* Reference to control `Edit2` : `TEdit`
|
```
00447AB8   8B80DC020000           mov     eax, [eax+$02DC]
```

* Reference to: `controls.TControl.GetText(TControl):TCaption;`
|
```
00447ABE   E8A5E2FDFF             call    00425D68
00447AC3   8B45DC                 mov     eax, [ebp-$24]
00447AC6   8D55E5                 lea     edx, [ebp-$1B]
00447AC9   B912000000             mov     ecx, $00000012
```

|
```
00447ACE   E889040000             call    00447F5C     可疑的 Call，不能放过。这里要记下地址
00447AD3   C645F700               mov     byte ptr [ebp-$09], $00
00447AD7   8D55D8                 lea     edx, [ebp-$28]
00447ADA   8B45FC                 mov     eax, [ebp-$04]
```

好了，现在该 `OD` 上场了。用 `OD` 加载 `setup.exe`。`Ctrl+G` 来到 `00447ACE`，`F2` 下断点。

```
00447AAB   .  6A 12           PUSH 12                          (固定值，序列号的长度)
00447AAD   .  68 73010000    PUSH 173                         (固定值，算法里有用到)
00447AB2   .  8D55 DC         LEA EDX,DWORD PTR SS:[EBP-24]
00447AB5   .  8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
00447AB8   .  8B80 DC020000  MOV EAX,DWORD PTR DS:[EAX+2DC]
00447ABE   .  E8 A5E2FDFF    CALL setup.00425D68
00447AC3   .  8B45 DC        MOV EAX,DWORD PTR SS:[EBP-24]            (用户名)
00447AC6   .  8D55 E5        LEA EDX,DWORD PTR SS:[EBP-1B]            (存放序列号的地方,没有算之前，这里是空)
00447AC9   .  B9 12000000    MOV ECX,12                             ; |
00447ACE   .  E8 89040000    CALL setup.00447F5C                      <-----断在这里,注册算法就在这个里面
00447AD3   .  C645 F7 00     MOV BYTE PTR SS:[EBP-9],0
00447AD7   .  8D55 D8        LEA EDX,DWORD PTR SS:[EBP-28]
00447ADA   .  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
00447ADD   .  8B80 E0020000  MOV EAX,DWORD PTR DS:[EAX+2E0]
00447AE3   .  E8 80E2FDFF    CALL setup.00425D68
00447AE8   .  8B45 D8        MOV EAX,DWORD PTR SS:[EBP-28]
00447AEB   .  E8 64C3FBFF    CALL setup.00403E54
00447AF0   .  8BD0           MOV EDX,EAX            (用户输入的序列号)
00447AF2   .  8D45 E5        LEA EAX,DWORD PTR SS:[EBP-1B]  (这一步一执行，在 EAX 里直接就可以看到序列号了)
00447AF5   .  E8 220AFCFF    CALL setup.0040851C       (序列号比较，就是一般的字符串比较)
00447AFA   .  85C0           TEST EAX,EAX
00447AFC   .  74 1A          JE SHORT setup.00447B18    (相等就跳，然后是一些登记操作，我没分析了)
00447AFE   .  6A 00          PUSH 0                       ; /Arg1 = 00000000
00447B00   .  66:8B0D A87C4>MOV CX,WORD PTR DS:[447CA8]                ; |
00447B07   .  33D2          XOR EDX,EDX                        ; |
00447B09   .  B8 B47C4400    MOV EAX,setup.00447CB4                 ; |ASCII "Invalid serial number"
00447B0E   .  E8 C9E4FFFF    CALL setup.00445FDC                  ; \\setup.00445FDC
```

## 分析 CALL setup.00447F5C

好了，到这里我们就要分析 `CALL setup.00447F5C` 里到底进行了些什么操作。

该 `IDA` 上场了，因为 `IDA` 看起来比 `OD` 标的更清楚些。。。

`IDA` 加载 `setup`，跳转到指定地址，输入 `00447F5C`，下面是 `IDA` 的反汇编代码,这里要说下的是 delphi 的参数传递，前两个参数用寄存器 `EAX`,`EDX` 后面两个参数用堆栈传递。

```
sub_447F5C      proc near                   ; CODE XREF: sub_4474CC+1D7p
CODE:00447F5C                             ; sub_4474CC+2C3p ...
CODE:00447F5C
CODE:00447F5C var_c             = dword ptr -0Ch
CODE:00447F5C sn                = dword ptr -8       ;这里我重新命名了
CODE:00447F5C username          = dword ptr -4       ;这里我重新命名了
CODE:00447F5C const_173         = dword ptr  8       ;这里我重新命名了
CODE:00447F5C const_12          = dword ptr  0Ch     ;这里我重新命名了
CODE:00447F5C

CODE:00447F5C                     push    ebp
CODE:00447F5D                     mov     ebp, esp
CODE:00447F5F                     add     esp, 0FFFFFFF4h
CODE:00447F62                     push    ebx
CODE:00447F63                     push    esi
CODE:00447F64                     push    edi
CODE:00447F65                     mov     [ebp+sn], edx    ;保存序列号地址
CODE:00447F68                     mov     [ebp+username], eax  ;保存用名名地址
CODE:00447F6B                     mov     edi, [ebp+const_173]
CODE:00447F6E                     mov     ebx, [ebp+const_12]   ; ebx 固定值 0x12（18）
CODE:00447F71                     dec     ebx
CODE:00447F72                     test    ebx, ebx
CODE:00447F74                     jl      short sum_SN
CODE:00447F76                     inc     ebx
CODE:00447F77                     mov     [ebp+var_c], 0
CODE:00447F7E                     mov     esi, offset init_table
```

`init_table` 是一个整型数组，我已经把它弄出来了，见下面

从 `Init_SN` 开始依据输入的用户名对序列号进行初始化操作：

循环变量 `var_c` 从 0 开始对用户名字符串长度求余得 `rem`，然后取出用户名字符串中的第 `rem` 位字符 `x`，从 `init_table` 数组中取出第 `var_c` 位的数字 `y`，然后把字符 `x` 放入序列号数组的第 `y` 位，循环 18 次。

```
CODE:00447F83
CODE:00447F83 Init_SN:                       ; CODE XREF: sub_447F5C+50
CODE:00447F83                     mov     eax, [ebp+username]
CODE:00447F86                     call    Length
CODE:00447F8B                     push    eax               ; username 的长度
CODE:00447F8C                     mov     eax, [ebp+var_c]    ; eax <- var_c
CODE:00447F8F                     pop     edx
CODE:00447F90                     mov     ecx, edx          ; username 的长度 给 ecx
CODE:00447F92                     cdq
CODE:00447F93                     idiv    ecx               ; var_c / 长度
CODE:00447F95                     inc     edx               ; edx 余数
CODE:00447F96                     mov     eax, [ebp+username]    ; eax 用户名
CODE:00447F99                     mov     al, [eax+edx-1]     ;取用户名的第 edx 个字符
CODE:00447F9D                     mov     edx, [esi]     ;从 init_table 取一个数字 edx
CODE:00447F9F                     mov     ecx, [ebp+sn]     ;取序列号地址
CODE:00447FA2                     mov     [ecx+edx], al     ;把取出的字符入到序列号的第 edx 位
CODE:00447FA5                     inc     [ebp+var_c]     ;var_c+1
CODE:00447FA8                     add     esi, 4
CODE:00447FAB                     dec     ebx
CODE:00447FAC                     jnz     short Init_SN
```

循环结束，在序列号数组里填充了依据用户名字符中计算得的一组字符。

```
CODE:00447FAE
CODE:00447FAE sum_SN:                         ; CODE XREF: sub_447F5C+18
CODE:00447FAE                     mov     ebx, [ebp+const_12]
CODE:00447FB1                     dec     ebx
CODE:00447FB2                     test    ebx, ebx
CODE:00447FB4                     jl      short loc_447FDF
CODE:00447FB6                     inc     ebx
CODE:00447FB7                     mov     eax, [ebp+sn]
```

从这里开始对序列号进行一些逻辑运算。循环变量 `i` 从 0 到 18，从序列号数组中取了第 `i` 个字符 `a`，变量 `di` 的初始值为 173，字符 `a` 和 `di` 右移 8 的值进行异或操作后得到值 `x` 放入序列号的第 `i` 位，然后再对 `di` 进行如下操作 `edi = (x+di)*0CE6Dh + 58BFh`

```
CODE:00447FBA
CODE:00447FBA XXX_SN1:                         ; CODE XREF: sub_447F5C+81
```



# 汇编代码分析

以下汇编代码段实现了一个序列号的处理逻辑。核心操作包括对字符进行异或运算、基于特定常量的乘法累加、以及通过查表和位交换来生成最终序列号。

## 核心循环逻辑

```
> CODE:00447FBA          xor     edx, edx  
> CODE:00447FBC          mov     dl, [eax]    ;从序列号数组中取出一个字符放入 dl  
> CODE:00447FBE          movzx   ecx, di      ;di 的值给 ecx 也就是 173  
> CODE:00447FC1          shr     ecx, 8       ;173 右移 8 位 放入 ecx  
> CODE:00447FC4          xor     edx, ecx     ;刚取出的符和右移后的值进行 xor 操作。  
> CODE:00447FC6          mov     [eax], dl    ;xor 后的值再放回序列号数组中  
> CODE:00447FC8          xor     edx, edx     
> CODE:00447FCA          mov     dl, [eax]    ;计算出的字符   
> CODE:00447FCC          add     di, dx  
> CODE:00447FCF          imul    dx, di, 0CE6Dh  
> CODE:00447FD4          add     dx, 58BFh  
> CODE:00447FD9          mov     edi, edx     ;上面的汇编代码可以认为是：edi = (edx+di)*0CE6Dh + 58BFh  
> CODE:00447FDB          inc     eax    ;数组下标加 1  
> CODE:00447FDC          dec     ebx  
> CODE:00447FDD          jnz     short XXX_SN1  
>   
> CODE:00447FDF  
> CODE:00447FDF loc_447FDF:                           ; CODE XREF: sub_447F5C+58  
> CODE:00447FDF          mov     ebx, [ebp+const_12]  
> CODE:00447FE2          dec     ebx  
> CODE:00447FE3          test    ebx, ebx  
> CODE:00447FE5          jl      short loc_448004  
> CODE:00447FE7          inc     ebx  
> CODE:00447FE8          mov     ecx, [ebp+sn]  
```

### 查表转换明文

下面的操作比较简单了，序列号数组中的每个字符对 `31h` 求余得余数 `rem`，然后从 `ascii_table` 中取出第 `rem` 个字符放入序列号数组中相应的位置。

```
> CODE:00447FEB  
> CODE:00447FEB QUERY_ASCII_TABLE:                       ; CODE XREF: sub_447F5C+A6  
> CODE:00447FEB          xor     eax, eax  
> CODE:00447FED          mov     al, [ecx]    ;取序列号字符  
> CODE:00447FEF          mov     esi, 31h     
> CODE:00447FF4          xor     edx, edx  
> CODE:00447FF6          div     esi      ;模 31h 求余    EDX  
> CODE:00447FF8          mov     al, ds:ascii_table[edx]  ;ascii_table 中第余数 edx 个字符  
> CODE:00447FFE          mov     [ecx], al     ;放入原来的序列号数组中  
> CODE:00448000          inc     ecx  
> CODE:00448001          dec     ebx  
> CODE:00448002          jnz     short QUERY_ASCII_TABLE  
```

通过上面的查表操作，把序列号转换成明文。基本上序列号的字符已经确定。

## 位置交换逻辑

```
> CODE:00448004  
> CODE:00448004 loc_448004:                           ; CODE XREF: sub_447F5C+89  
> CODE:00448004          mov     ebx, [ebp+const_12]  
> CODE:00448007          dec     ebx  
> CODE:00448008          test    ebx, ebx  
> CODE:0044800A          jl      short loc_448030  
> CODE:0044800C          inc     ebx  
> CODE:0044800D          mov     eax, [ebp+sn]  
> CODE:00448010          mov     esi, offset unk_44AB20  ;esi 指向一张交换位置表  
```

对序列号进行一些位位置上的交换。遍历序列号，然后查表找到其要交换的位置，然后进行字符交换。

```
> CODE:00448015  
> CODE:00448015 Exchang_XXX:                           ; CODE XREF: sub_447F5C+D2  
> CODE:00448015          mov     dl, [eax]    ;取出序列号的一个字符  
> CODE:00448017          mov     ecx, [esi]  ;需要交换的位置  
> CODE:00448019          mov     edi, [ebp+sn]   
> CODE:0044801C          mov     cl, [edi+ecx]  ;要交换位置的字符  
> CODE:0044801F          mov     [eax], cl  
> CODE:00448021          mov     ecx, [esi]  
> CODE:00448023          mov     edi, [ebp+sn]  
> CODE:00448026          mov     [edi+ecx], dl  ;上面的操作完成两个位置字符的交换  
> CODE:00448029          add     esi, 4  
> CODE:0044802C          inc     eax  
> CODE:0044802D          dec     ebx  
> CODE:0044802E          jnz     short Exchang_XXX  
> CODE:00448030  
> CODE:00448030 loc_448030:                           ; CODE XREF: sub_447F5C+AE  
> CODE:00448030          pop     edi  
> CODE:00448031          pop     esi  
> CODE:00448032          pop     ebx  
> CODE:00448033          mov     esp, ebp  
> CODE:00448035          pop     ebp  
> CODE:00448036          retn    8  
> CODE:00448036 sub_447F5C      endp  
```

## 数据表定义

终于分析完了，搞了一天，真是有点郁闷。把上面提到的三个表（数组）也附上。

这三张表如何得到的，我没有去分析，但是看地址 `DATA` 段的地址是 `0044A000` 大小为 `1000`，这三个表都在 `DATA` 段内，所以估计这三个表（数组）是已经初始化的静态数组，不会变。

### `init_table`

```
> init_table
>
> 0044AAD8  10 00 00 00 02 00 00 00 00 00 00 00 03 00 00 00  . . . . . . . . . . . . . . . .  
> 0044AAE8  0E 00 00 00 01 00 00 00 05 00 00 00 07 00 00 00  . . . . . . . . . . . . . . . .  
> 0044AAF8  04 00 00 00 0F 00 00 00 09 00 00 00 0B 00 00 00  . . . . . . . . . . . . . . . .  
> 0044AB08  11 00 00 00 08 00 00 00 0C 00 00 00 06 00 00 00  . . . . . . . . . . . . . . . .  
> 0044AB18  0D 00 00 00 0A 00 00 00 11 00 00 00 02 00 00 00  . . . . . . . . . . . . . . . .  
> 0044AB28  03 00 00 00 00 00 00 00 0E 00 00 00 01 00 00 00  . . . . . . . . . . . . . . . .  
> 0044AB38  0C 00 00 00 07 00 00 00 05 00 00 00 04 00 00 00  . . . . . . . . . . . . . . . .  
> 0044AB48  0F 00 00 00 09 00 00 00 10 00 00 00 0B 00 00 00  . . . . . . . . . . . . . . . .  
> 0044AB58  08 00 00 00 0D 00 00 00 0A 00 00 00 06 00 00 00  . . . . . . . . . . . . . . . .
```

### `ascii` 表

```
> ascii 表
>
> 0044AAA4  31 51 32 57 33 45 34 52 35 54 36 59 37 55 38 49  1Q2W3E4R5T6Y7U8I  
> 0044AAB4  39 41 31 53 32 44 33 46 34 47 35 48 36 4A 37 4B  9A1S2D3F4G5H6J7K  
> 0044AAC4  38 4C 39 5A 58 31 43 32 56 33 42 34 4E 35 4D 36  8L9ZX1C2V3B4N5M6  
> 0044AAD4  51 37 57 90 10 00 00 00 02 00 00 00 00 00 00 00  Q7W?..........
```

### `Exchang` 表

```
> Exchang 表
>
> 0044AB20  11 00 00 00 02 00 00 00 03 00 00 00 00 00 00 00  . . . . . . . . . . . . . . . .  
> 0044AB30  0E 00 00 00 01 00 00 00 0C 00 00 00 07 00 00 00  . . . . . . . . . . . . . . . .  
> 0044AB40  05 00 00 00 04 00 00 00 0F 00 00 00 09 00 00 00  . . . . . . . . . . . . . . . .  
> 0044AB50  10 00 00 00 0B 00 00 00 08 00 00 00 0D 00 00 00  . . . . . . . . . . . . . . . .  
> 0044AB60  0A 00 00 00 06 00 00 00 80 6F 40 00 F0 6E 40 00  . . . . . . o@.@.
```

## Delphi 算法实现

附上 `delphi` 的算法实现：算法没有优化，写的比较赖。将就一下了，今天一天都还没吃饭，先闪了。。。

```pascal
  Table_InitSN: array[0..$8F] of Byte = (
        $10,00,00,00,$02,00,00,00,$00,00,00,00,$03,00,00,00,
        $0E,00,00,00,$01,00,00,00,$05,00,00,00,$07,00,00,00,
        $04,00,00,00,$0F,00,00,00,$09,00,00,00,$0B,00,00,00,
        $11,00,00,00,$08,00,00,00,$0C,00,00,00,$06,00,00,00,
        $0D,00,00,00,$0A,00,00,00,$11,00,00,00,$02,00,00,00,
        $03,00,00,00,$00,00,00,00,$0E,00,00,00,$01,00,00,00,
        $0C,00,00,00,$07,00,00,00,$05,00,00,00,$04,00,00,00,
        $0F,00,00,00,$09,00,00,00,$10,00,00,00,$0B,00,00,00,
        $08,00,00,00,$0D,00,00,00,$0A,00,00,00,$06,00,00,00);
  Table_AscII: array [0..$3F] of Byte = (
        $31,$51,$32,$57,$33,$45,$34,$52,$35,$54,$36,$59,$37,$55,$38,$49,
        $39,$41,$31,$53,$32,$44,$33,$46,$34,$47,$35,$48,$36,$4A,$37,$4B,
        $38,$4C,$39,$5A,$58,$31,$43,$32,$56,$33,$42,$34,$4E,$35,$4D,$36,
        $51,$37,$57,$90,$10,$00,$00,$00,$02,$00,$00,$00,$00,$00,$00,$00);
  Table_Exchang: array [0..$4F] of Byte = (
        $11,$00,$00,$00,$02,$00,$00,$00,$03,$00,$00,$00,$00,$00,$00,$00,
        $0E,$00,$00,$00,$01,$00,$00,$00,$0C,$00,$00,$00,$07,$00,$00,$00,
        $05,$00,$00,$00,$04,$00,$00,$00,$0F,$00,$00,$00,$09,$00,$00,$00,
        $10,$00,$00,$00,$0B,$00,$00,$00,$08,$00,$00,$00,$0D,$00,$00,$00,
        $0A,$00,$00,$00,$06,$00,$00,$00,$80,$6F,$40,$00,$F0,$6E,$40,$00);

function GenKey(StrUserName:string;var Sn:array of Char;Const_173:Integer;Const_12:Integer):string;
var
  Username: string;
  Serial: PChar;
  iCount: Integer;
  i,len,rem: Integer;
  a,b: Char;
  PTable: PByte;
  iecx: Integer;
  idi: Word;
begin
  Serial := Sn;
  Username := StrUserName;
  iCount := 0;
  PTable := @Table_InitSN;
  //初始公序列号数组
  for i:=Const_12 downto 0 do
  begin
    len := Length(Username);
    rem := iCount mod len;
    Inc(rem);
    a := UserName[rem];
    (Serial+PTable)^ := a;
    Inc(iCount);
    PTable := pbyte(Integer(PTable) + 4);
```



```pascal
end;

//对序列号进行一些运算操作

idi := word(Const_173);
for i := Const_12 downto 0 do
begin
  a := Serial^;
  iecx := idi shr 8;
  a := Char(Integer(a) xor iecx);
  Serial^ := a;
  idi := (idi + word(a)) * $0CE6D + $58BF;
  Serial := Serial + 1;
end;

//对过查表把序列号变成明文

Serial := Sn;
for i := Const_12 downto 0 do
begin
  a := Serial^;
  rem := Byte(a) mod $31;
  a := Char(Table_AscII[rem]);
  Serial^ := a;
  Serial := Serial + 1;
end;

//查表对序列号位置进行交换操作

Serial := Sn;
PTable := @Table_Exchang;
for i := Const_12 downto 0 do
begin
  a := Serial^;
  iecx := PTable^;
  b := (PChar(Sn + iecx))^;
  Serial^ := b;
  (PChar(Sn + iecx))^ := a;
  PTable := pbyte(Integer(PTable) + 4);
  Serial := Serial + 1;
end;

end;
```

`4dd466d0e1e0cc123afbe504da2462ca`
