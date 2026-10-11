
`.text:0043FE41`

`.text:0043FE41` `loc_43FE41:` ; CODE XREF: `_TForm1_SpeedButton1Click+13Ej`

```
.text:0043FE41          push    esi
.text:0043FE42          push    edi
.text:0043FE43          xor     eax, eax
.text:0043FE45          mov     edi, edx
.text:0043FE47          or      ecx, 0FFFFFFFFh
.text:0043FE4A          lea     esi, [ebp+s]   ******* 用户名拷贝到 s
.text:0043FE50          repne scasb
.text:0043FE52          not     ecx
.text:0043FE54          sub     edi, ecx
.text:0043FE56          mov     edx, ecx
.text:0043FE58          xchg    esi, edi
.text:0043FE5A          shr     ecx, 2
.text:0043FE5D          mov     eax, edi
.text:0043FE5F          rep movsd
.text:0043FE61          mov     ecx, edx
.text:0043FE63          mov     edx, 2
.text:0043FE68          and     ecx, 3
.text:0043FE6B          lea     eax, [ebp+var_2C]  ******* 释放用户名缓冲区
.text:0043FE6E          rep movsb
.text:0043FE70          pop     edi
.text:0043FE71          pop     esi
.text:0043FE72          dec     [ebp+var_88]
.text:0043FE78          call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)
.text:0043FE7D          lea     ebx, [ebp+esi+s]   ******* 指向用户名
```

`.text:0043FE84`

`.text:0043FE84` `loc_43FE84:` ; CODE XREF: `_TForm1_SpeedButton1Click+1A4j`

```
.text:0043FE84          cmp     byte ptr [ebx], 20h  *******取一个字节，比较是否是空格
.text:0043FE87          jnz     short loc_43FEA2
.text:0043FE89          mov     esi, 1
.text:0043FE8E          lea     eax, [ebp+s+1]
```

`.text:0043FE94`

`.text:0043FE94` `loc_43FE94:` ; CODE XREF: `_TForm1_SpeedButton1Click+1A2j`

```
.text:0043FE94          mov     dl, [eax]
.text:0043FE96          inc     esi
.text:0043FE97          mov     [eax-1], dl  *******如果是空格则整个向前移
.text:0043FE9A          inc     eax
.text:0043FE9B          cmp     esi, 14h     *******取 20 位
.text:0043FE9E          jl      short loc_43FE94
.text:0043FEA0          jmp     short loc_43FE84
```

`.text:0043FEA2` ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

`.text:0043FEA2`

`.text:0043FEA2` `loc_43FEA2:` ; CODE XREF: `_TForm1_SpeedButton1Click+18Bj`

```
.text:0043FEA2          mov     esi, 13h
.text:0043FEA7          lea     ebx, [ebp+var_14D] *******指向第 20 位
```

`.text:0043FEAD`

`.text:0043FEAD` `loc_43FEAD:` ; CODE XREF: `_TForm1_SpeedButton1Click+1C2j`

```
.text:0043FEAD          mov     al, [ebx]  *******取一个字节
.text:0043FEAF          cmp     al, 20h    *******比较是不是空格
.text:0043FEB1          jz      short loc_43FEB7
.text:0043FEB3          test    al, al
.text:0043FEB5          jnz     short loc_43FEC0
```

`.text:0043FEB7`

`.text:0043FEB7` `loc_43FEB7:` ; CODE XREF: `_TForm1_SpeedButton1Click+1B5j`

```
.text:0043FEB7          mov     byte ptr [ebx], 0  *******是空格则清零
.text:0043FEBA          dec     esi
.text:0043FEBB          dec     ebx
.text:0043FEBC          test    esi, esi
.text:0043FEBE          jge     short loc_43FEAD
```

\=========================================================上面这段逆向如下：

```
char var_4[] = "1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijkmnpqrstuvwxyz";

char var_c[100];

char name[20];

memset(var_c,0,100);

memset(name,0,20);

strcpy(name,username);

while(name[0]==' ')

{

    for(i=1;i<20;i++)

        name[i-1] = name[i];

}

i = 19;

do{

    if(name[i] != ' ')

    {

        if(name[i] != 0)break;

    }

        name[i] = 0;

        i--;

}while(i >= 0);

\=========================================================
```

`.text:0043FEC0`

`.text:0043FEC0` `loc_43FEC0:` ; CODE XREF: `_TForm1_SpeedButton1Click+1B9j`

```
.text:0043FEC0          xor     esi, esi
.text:0043FEC2          lea     ebx, [ebp+s]     *******指向用户名
```

`.text:0043FEC8`

`.text:0043FEC8` `loc_43FEC8:` ; CODE XREF: `_TForm1_SpeedButton1Click+1E4j`

```
.text:0043FEC8          xor     eax, eax
.text:0043FECA          lea     ecx, [esi+1]
.text:0043FECD          mov     al, [ebx]        *******取一个字节,设为 x
.text:0043FECF          inc     ebx
.text:0043FED0          mov     edx, eax
.text:0043FED2          inc     esi
.text:0043FED3          imul    edx, eax
.text:0043FED6          add     edx, eax         ******* x*x+x
.text:0043FED8          imul    edx, ecx         ******* 乘以 索引加 1
.text:0043FEDB          add     edi, edx         ******* 累加到 edi
.text:0043FEDD          cmp     esi, 14h
.text:0043FEE0          jl      short loc_43FEC8  *******没到 20 位继续循环
.text:0043FEE2          lea     eax, [edi+5BA0h]  ******* 结果加上 0x5ba0
.text:0043FEE8          xor     edx, edx
.text:0043FEEA          mov     [ebp+var_B4], eax  *******保存结果
.text:0043FEF0          mov     [ebp+var_B0], edx
.text:0043FEF6          cmp     [ebp+var_B0], 0
.text:0043FEFD          jnz     short loc_43FF1B
.text:0043FEFF          cmp     [ebp+var_B4], 5C25Ch  *****结果是否等于 0x5c25c
.text:0043FF09          jnz     short loc_43FF1B
.text:0043FF0B          add     [ebp+var_B4], 78h     *****加上 0x78
.text:0043FF12          adc     [ebp+var_B0], 0
.text:0043FF19          jmp     short loc_43FF3E
```

`.text:0043FF1B` ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

`.text:0043FF1B`

`.text:0043FF1B` `loc_43FF1B:` ; CODE XREF: `_TForm1_SpeedButton1Click+201j`

`.text:0043FF1B` ; `_TForm1_SpeedButton1Click+20Dj`

```
.text:0043FF1B          cmp     [ebp+var_B0], 0
.text:0043FF22          jnz     short loc_43FF3E
.text:0043FF24          cmp     [ebp+var_B4], 8C5BCh  *******结果是否等于 0x8c8bc
.text:0043FF2E          jnz     short loc_43FF3E
.text:0043FF30          add     [ebp+var_B4], 64h     *******加上 0x64
.text:0043FF37          adc     [ebp+var_B0], 0
```

`.text:0043FF3E`

`.text:0043FF3E` `loc_43FF3E:` ; CODE XREF: `_TForm1_SpeedButton1Click+21Dj`

`.text:0043FF3E` ; `_TForm1_SpeedButton1Click+226j` ...

```
.text:0043FF3E          push    [ebp+var_B0]
.text:0043FF44          push    [ebp+var_B4]
.text:0043FF4A          push    offset off_505432     ******* "%ld"
.text:0043FF4F          lea     eax, [ebp+var_10]     *******转换成十进制字符串
.text:0043FF52          push    eax
.text:0043FF53          call    @System@WideString@printf$qpxbe ; System::WideString::printf(wchar_t *,...)
```

\=========================================================上面这段逆向如下：

```
int sum;

sum = 0;

for(i=0;i<20;i++)

    sum += name[i]*(name[i]+1)*(i+1);

sum += 0x5ba0;

if(sum == 0x5c25c)sum += 0x78;

else if(sum == 0x8c5bc)sum += 0x64;

sprintf(var_10,"%ld",sum);

\=========================================================
```

`.text:0043FF58`

```
.text:0043FF58          add     esp, 10h
.text:0043FF5B          lea     edx, [ebp+buffer]
.text:0043FF61          push    14h         ; n
.text:0043FF63          push    0           ; c
```
