
# TxtEdit 4.5.2

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-418.htm`](https://www.pediy.com/kssd/pediy08/pediy8-418.htm)

## 简单的 `TxtEdit 4.5.2` 注册算法

### 破文作者
XXNB

### 作者邮箱
支持 PYG

### 作者主页
[`free.ys168.com/?binbinbin7456`](http://free.ys168.com/?binbinbin7456)

### 破解工具
`OD`

### 破解平台
`xpsp2`

### 软件名称
`TxtEdit 4.5.2`

### 软件大小
924KB

### 原版下载
[`www.onlinedown.net/soft/23539.htm`](http://www.onlinedown.net/soft/23539.htm)

### 保护方式
名+码

### 软件简介
文本编辑器，有拼写检查功能!

- 软件语言：英文
- 软件类别：国外软件/共享版/编辑软件
- 运行环境：`Win9x`/`Me`/`NT`/`2000`/`XP`/`2003`
- 加入时间：2006-11-20 9:22:12
- 下载次数：901

### 破解声明
向大侠们学习！！！只为学习！

------------------------------------------------------------------------

## 破解过程

------------------------------------------------------------------------

1、用 Delphi 的利器，`dede`。我们很容易就找到按钮的断点。“找到关键断点，成功一半”

我们输入信息：`regname`：`binbin`
`regNO`：`123456789`

经过我的分析后，输入的注册码要大于“10000000”。具体看下面的代码分析。

```
004E7080   $  55             push    ebp
004E7081   .  8BEC           mov     ebp, esp
004E7083   .  B9 04000000    mov     ecx, 4
004E7088   >  6A 00          push    0
004E708A   .  6A 00          push    0
004E708C   .  49             dec     ecx
004E708D   .^ 75 F9          jnz     short 004E7088
004E708F   .  51             push    ecx
004E7090   .  53             push    ebx
004E7091   .  56             push    esi
004E7092   .  57             push    edi
004E7093   .  8945 FC        mov     dword ptr [ebp-4], eax
004E7096   .  33C0           xor     eax, eax
004E7098   .  55             push    ebp
004E7099   .  68 A8724E00    push    004E72A8
004E709E   .  64:FF30        push    dword ptr fs:[eax]
004E70A1   .  64:8920        mov     dword ptr fs:[eax], esp
004E70A4   .  C745 F0 00000> mov     dword ptr [ebp-10], 0
004E70AB   .  C745 F4 00000> mov     dword ptr [ebp-C], 0
004E70B2   .  8D55 EC        lea     edx, dword ptr [ebp-14]
004E70B5   .  8B45 FC        mov     eax, dword ptr [ebp-4]
004E70B8   .  8B80 28030000  mov     eax, dword ptr [eax+328]
004E70BE   .  E8 DD80F8FF    call    0046F1A0
004E70C3   .  8B45 EC        mov     eax, dword ptr [ebp-14]          ;  用户名出现了 binbin
004E70C6   .  E8 69D7F1FF    call    00404834                         ;  计算位数
004E70CB   .  8BD8           mov     ebx, eax
004E70CD   .  4B             dec     ebx
004E70CE   .  85DB           test    ebx, ebx
004E70D0   .  7E 34          jle     short 004E7106
004E70D2   .  BE 01000000    mov     esi, 1
004E70D7   >  8D55 E8        lea     edx, dword ptr [ebp-18]
004E70DA   .  8B45 FC        mov     eax, dword ptr [ebp-4]
004E70DD   .  8B80 28030000  mov     eax, dword ptr [eax+328]
004E70E3   .  E8 B880F8FF    call    0046F1A0
004E70E8   .  8B45 E8        mov     eax, dword ptr [ebp-18]          ;  用户名
004E70EB   .  8A4430 FF      mov     al, byte ptr [eax+esi-1]         ;  逐个取用户名
004E70EF   .  25 FF000000    and     eax, 0FF                         ;  得到 ascii 码值
004E70F4   .  33D2           xor     edx, edx
004E70F6   .  0345 F0        add     eax, dword ptr [ebp-10]          ;  加上一次的结果
004E70F9   .  1355 F4        adc     edx, dword ptr [ebp-C]
004E70FC      8945 F0        mov     dword ptr [ebp-10], eax          ;  存储结果
004E70FF   .  8955 F4        mov     dword ptr [ebp-C], edx
004E7102   .  46             inc     esi
004E7103   .  4B             dec     ebx
004E7104   .^ 75 D1          jnz     short 004E70D7                   ;  累加用户名 ascii 码值，存储在 ebp-10。注意了，要去掉最后
004E7106   >  33C0           xor     eax, eax                           ;  204（516）去掉最后一位的累加值
004E7108   .  55             push    ebp
004E7109   .  68 7F714E00    push    004E717F
004E710E   .  64:FF30        push    dword ptr fs:[eax]
004E7111   .  64:8920        mov     dword ptr fs:[eax], esp
004E7114   .  8D55 E4        lea     edx, dword ptr [ebp-1C]
004E7117   .  8B45 FC        mov     eax, dword ptr [ebp-4]
004E711A   .  8B80 2C030000  mov     eax, dword ptr [eax+32C]
004E7120   .  E8 7B80F8FF    call    0046F1A0
004E7125   .  8B45 E4        mov     eax, dword ptr [ebp-1C]          ;  假码出现了，还有位数
004E7128   .  E8 2B25F2FF    call    00409658                         ;  哇，转成 16 进制
004E712D   .  0345 F0        add     eax, dword ptr [ebp-10]          ;  加上用户名的累加值
004E7130   .  1355 F4        adc     edx, dword ptr [ebp-C]
004E7133   .  8945 F0        mov     dword ptr [ebp-10], eax
004E7136   .  8955 F4        mov     dword ptr [ebp-C], edx
004E7139   .  837D F4 00     cmp     dword ptr [ebp-C], 0
004E713D   .  75 0B          jnz     short 004E714A
004E713F      817D F0 80969> cmp     dword ptr [ebp-10], 989680       ;  和“10000000”比较
004E7146   .  72 10          jb      short 004E7158                   ;  小于的话跳。跳就失败，所以一定要大于
004E7148   .  EB 02          jmp     short 004E714C
004E714A   >  7C 0C          jl      short 004E7158
004E714C   >  8B45 FC        mov     eax, dword ptr [ebp-4]
004E714F   .  83B8 3C030000> cmp     dword ptr [eax+33C], 5
004E7156   .  7E 1D          jle     short 004E7175
004E7158   >  8B0D 4CF94F00  mov     ecx, dword ptr [4FF94C]          ;  TxtEdit.005019BC
004E715E   .  8B89 78010000  mov     ecx, dword ptr [ecx+178]
004E7164   .  B2 01          mov     dl, 1
004E7166   .  A1 D4884000    mov     eax, dword ptr [4088D4]
004E716B   .  E8 BC60F2FF    call    0040D22C
004E7170   .  E8 13CEF1FF    call    00403F88
004E7175   >  33C0           xor     eax, eax
004E7177   .  5A             pop     edx
004E7178   .  59             pop     ecx
004E7179   .  59             pop     ecx
004E717A   .  64:8910        mov     dword ptr fs:[eax], edx
004E717D   .  EB 28          jmp     short 004E71A7
004E717F   .^ E9 44CCF1FF    jmp     00403DC8
004E7184      01             db      01
004E7185      00             db      00
004E7186      00             db      00
004E7187      00             db      00
004E7188   .  D4884000      dd      TxtEdit.004088D4
004E718C   .  90714E00      dd      TxtEdit.004E7190
004E7190   .  8B45 FC        mov     eax, dword ptr [ebp-4]
004E7193   .  E8 E0060000    call    004E7878
004E7198   .  E8 67CEF1FF    call    00404004
004E719D   .  E9 EB000000    jmp     004E728D
004E71A2   .  E8 5DCEF1FF    call    00404004
004E71A7   >  6A 00          push    0
004E71A9   .  68 2F220000    push    222F                           ;  注意这个常数 222F
004E71AE   .  8B45 F0        mov     eax, dword ptr [ebp-10]          ;  假码 16 进制到 eax
004E71B1   .  8B55 F4        mov     edx, dword ptr [ebp-C]
004E71B4   .  E8 5BE6F1FF    call    00405814                         ;  这个是算法 call 了，它返回的是注册码＋用户名累加值除以
004E71B9   .  83FA 00        cmp     edx, 0                           ;  222F 后的余数。返回一定要是 0
004E71BC   .  0F85 A8000000  jnz     004E726A                         ;  经过上面的 call。如果要等于 0 的话就是说，注册码＋用户名
004E71C2   .  83F8 00        cmp     eax, 0                           ;  累加值后一定要能整除 222F
004E71C5      0F85 9F000000  jnz     004E726A                         ;  余数为 0 就注册成功
004E71CB   .  C605 A0195000> mov     byte ptr [5019A0], 1             ;  关键跳转
004E71D2   .  8B45 FC        mov     eax, dword ptr [ebp-4]
004E71D5   .  8B80 28030000  mov     eax, dword ptr [eax+328]
004E71DB   .  BA 0F0000FF    mov     edx, FF00000F
004E71E0   .  E8 5B81F8FF    call    0046F340
004E71E5   .  8B45 FC        mov     eax, dword ptr [ebp-4]          ；下面这些是注册成功的信息。收工
004E71E8   .  8B80 2C030000  mov     eax, dword ptr [eax+32C]
004E71EE   .  BA 0F0000FF    mov     edx, FF00000F
004E71F3   .  E8 4881F8FF    call    0046F340
004E71F8   .  8D55 E0        lea     edx, dword ptr [ebp-20]
004E71FB   .  8B45 FC        mov     eax, dword ptr [ebp-4]
004E71FE   .  8B80 28030000  mov     eax, dword ptr [eax+328]
004E7204   .  E8 977FF8FF    call    0046F1A0
```



### 汇编代码分析

```asm
004E7209   .  8B4D E0         mov     ecx, dword ptr [ebp-20]  
004E720C   .  A1 D0F74F00    mov     eax, dword ptr [4FF7D0]  
004E7211   .  8B00           mov     eax, dword ptr [eax]  
004E7213   .  8B80 EC040000  mov     eax, dword ptr [eax+4EC]  
004E7219   .  BA 22000000    mov     edx, 22  
004E721E   .  8B18           mov     ebx, dword ptr [eax]  
004E7220   .  FF53 20        call    dword ptr [ebx+20]  
004E7223   .  8D55 DC        lea     edx, dword ptr [ebp-24]  
004E7226   .  8B45 FC        mov     eax, dword ptr [ebp-4]  
004E7229   .  8B80 2C030000  mov     eax, dword ptr [eax+32C]  
004E722F   .  E8 6C7FF8FF    call    0046F1A0  
004E7234   .  8B4D DC        mov     ecx, dword ptr [ebp-24]  
004E7237   .  A1 D0F74F00    mov     eax, dword ptr [4FF7D0]  
004E723C   .  8B00           mov     eax, dword ptr [eax]  
004E723E   .  8B80 EC040000  mov     eax, dword ptr [eax+4EC]  
004E7244   .  BA 23000000    mov     edx, 23  
004E7249   .  8B18           mov     ebx, dword ptr [eax]  
004E724B   .  FF53 20        call    dword ptr [ebx+20]  
004E724E   .  8B15 4CF94F00  mov     edx, dword ptr [4FF94C]          ;  TxtEdit.005019BC  
004E7254   .  8B92 80010000  mov     edx, dword ptr [edx+180]  
004E725A   .  8B45 FC        mov     eax, dword ptr [ebp-4]  
004E725D   .  8B80 20030000  mov     eax, dword ptr [eax+320]  
004E7263   .  E8 687FF8FF    call    0046F1D0  
004E7268   .  EB 08          jmp     short 004E7272  
004E726A   >  8B45 FC        mov     eax, dword ptr [ebp-4]  
004E726D   .  E8 06060000    call    004E7878  
004E7272   >  8A15 A1195000  mov     dl, byte ptr [5019A1]  
004E7278   .  80F2 01        xor     dl, 1  
004E727B   .  0A15 A0195000  or      dl, byte ptr [5019A0]  
004E7281   .  A1 D0F74F00    mov     eax, dword ptr [4FF7D0]  
004E7286   .  8B00           mov     eax, dword ptr [eax]  
004E7288   .  E8 EF230100    call    004F967C  
004E728D   >  33C0           xor     eax, eax  
004E728F   .  5A             pop     edx  
004E7290   .  59             pop     ecx  
004E7291   .  59             pop     ecx  
004E7292   .  64:8910        mov     dword ptr fs:[eax], edx  
004E7295   .  68 AF724E00    push    004E72AF  
004E729A   >  8D45 DC        lea     eax, dword ptr [ebp-24]  
004E729D   .  BA 05000000    mov     edx, 5  
004E72A2   .  E8 F1D2F1FF    call    00404598  
004E72A7   .  C3             retn  
```

### 易语言伪代码

**版本**：`2`
**程序集**：`窗口程序集 1`

**子程序**：`按钮 1_ 被单击`

**局部变量**：
- `name`，`文本型`
- `len`，`整数型`
- `code`，`长整数型`
- `i`，`整数型`
- `eax`，`整数型`
- `sum`，`整数型`

**代码逻辑**：

`name = 编辑框 1.内容`
`len = 取文本长度(name)`
`len = len - 1`

`计次循环首(len, i)`
    `eax = 取代码(name, i)`
    `sum = sum + eax`
`计次循环尾()`

`code = 8751 * 8751 - sum`
`编辑框 2.内容 = 到文本(code)`

### 破解总结

1. 获得用户名，然后取得用户名除最后一位外的 `ascii` 码累加值。
2. 获得输入的假码，加上上面的用户名累加值。结果如果能整除 `8751`，则注册成功。

`4dd466d0e1e0cc123afbe504da2462ca`
