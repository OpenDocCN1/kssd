
# WinSnap 3.1 注册算法分析 & 算法还原

> 原文：`www.pediy.com/kssd/pediy10/95308.html` ([链接](https://www.pediy.com/kssd/pediy10/95308.html))

WinSnap 是一款屏幕截图软件，可以方便地对屏幕截图并可进行处理。其下载地址是：`www.ntwind.com/software/winsnap.html` ([链接](http://www.ntwind.com/software/winsnap.html))

其最新版为 3.10，未注册版本只有 30 天使用期。

## 一. 程序分析

首先用 `PEID` 查一下是否加壳，显示“什么也没发现”。

既然不知道有没有壳，那就直接载入 `OD` 分析，使用通用的方法对 `GetDlgItemTextW` 下断即可很容易获取输入的用户名和序列号。然后就是字符转换函数，将用户名和序列号由 Unicode 转换为 ASCII。

再走几步就到了对用户注册合法性校验的函数，地址为 `0x00409A10`，下面用 `IDA` 载入程序对这个函数进行静态分析。

此函数非常庞大，下面分几部分进行说明。

首先，函数开始部分将用户名转换为大写，紧接着就是对用户名进行处理。

我们可以看到首先就是对 `sub_409790` 函数的调用，传递的参数是用于处理用户名缓冲区和 -1。

此函数在用户名处理的过程中被多次调用。

```asm
.text:00409790 sub_409790     proc near             ; CODE XREF: CheckPas+75p
.text:00409790                ; CheckPas+1D2p
.text:00409790
.text:00409790 var_104        = byte ptr -104h
.text:00409790 var_4          = dword ptr -4
.text:00409790 pUserNameBuf   = dword ptr  4
.text:00409790 arg_4          = dword ptr  8
.text:00409790
.text:00409790                sub     esp, 104h
.text:00409796                mov     eax, dword_44F050
.text:0040979B                xor     eax, esp
.text:0040979D                mov     [esp+104h+var_4], eax
.text:004097A4                mov     eax, [esp+104h+arg_4] ; 参数 2：-1
.text:004097AB                push    ebx
.text:004097AC                push    ebp
.text:004097AD                mov     ebp, [esp+10Ch+pUserNameBuf]
.text:004097B4                push    esi
.text:004097B5                push    edi
.text:004097B6                mov     edi, ebp        ; edi 用户名指针
.text:004097B8                test    eax, eax
.text:004097BA                jge     short loc_4097CA
.text:004097BC                mov     eax, ebp
.text:004097BE                lea     edx, [eax+1]
.text:004097C1
.text:004097C1 loc_4097C1:               ; CODE XREF: sub_409790+36j
.text:004097C1                mov     cl, [eax]
.text:004097C3                inc     eax
.text:004097C4                test    cl, cl
.text:004097C6                jnz     short loc_4097C1
.text:004097C8                sub     eax, edx        ; 计算用户名长度
.text:004097CA
.text:004097CA loc_4097CA:               ; CODE XREF: sub_409790+2Aj
.text:004097CA                xor     edx, edx        ; 数组索引变量
.text:004097CC                mov     esi, eax        ; esi 用户名长度
.text:004097CE                mov     edi, edi        ; edi 用户名指针
.text:004097D0
.text:004097D0 loc_4097D0:               ; CODE XREF: sub_409790+70j
.text:004097D0                mov     bl, dl
.text:004097D2                add     bl, 0Dh
.text:004097D5                mov     byte_457B78[edx], dl
.text:004097DB                mov     [esp+edx+114h+var_104], bl
.text:004097DF                test    eax, eax
.text:004097E1                jle     short loc_4097F9
.text:004097E3                mov     cl, [edi]
.text:004097E5                add     cl, al
.text:004097E7                xor     cl, bl
.text:004097E9                sub     esi, 1
.text:004097EC                mov     [esp+edx+114h+var_104], cl ; 存放经过处理的用户名的局部数组
.text:004097F0                jnz     short loc_4097F8
.text:004097F2                mov     edi, ebp
.text:004097F4                mov     esi, eax
.text:004097F6                jmp     short loc_4097F9
.text:004097F8 ; ---------------------------------------------------------------------------
.text:004097F8
.text:004097F8 loc_4097F8:               ; CODE XREF: sub_409790+60j
.text:004097F8                inc     edi
.text:004097F9
.text:004097F9 loc_4097F9:               ; CODE XREF: sub_409790+51j
.text:004097F9                ; sub_409790+66j
.text:004097F9                inc     edx
.text:004097FA                cmp     edx, 100h
.text:00409800                jl      short loc_4097D0
.text:00409802                xor     eax, eax
.text:00409804                xor     ecx, ecx        
.text:00409806                mov     dword_457C78, eax
.text:0040980B                mov     dword_457C7C, eax
.text:00409810
.text:00409810 loc_409810:               ; CODE XREF: sub_409790+B2j
.text:00409810                mov     dl, byte_457B78[eax]
.text:00409816                movzx   esi, [esp+eax+114h+var_104]
.text:0040981B                movzx   edi, dl
.text:0040981E                add     esi, ecx
.text:00409820                add     edi, esi
.text:00409822                and     edi, 0FFh
.text:00409828                mov     ecx, edi
.text:0040982A                mov     bl, byte_457B78[ecx]
.text:00409830                mov     byte_457B78[eax], bl
.text:00409836                inc     eax
.text:00409837                cmp     eax, 100h
.text:0040983C                mov     byte_457B78[ecx], dl
.text:00409842                jl      short loc_409810
.text:00409844                mov     ecx, [esp+114h+var_4]
.text:0040984B                pop     edi
.text:0040984C                pop     esi
.text:0040984D                pop     ebp
.text:0040984E                pop     ebx
.text:0040984F                xor     ecx, esp
.text:00409851                call    sub_4322D2
.text:00409856                add     esp, 104h
.text:0040985C                retn
.text:0040985C sub_409790     endp
```

以上代码将处理后的用户名存放在大小为 `100h` 字节的 `UserNameBuf` 中，

其中，主要运算过程是根据 `UserNameBuf` 的值进行运算后建立一个 `100h` 字节的局部变量，接着根据局部数组的值对 `457B78` 中的元素进行运算，用运算后的结构作为索引 `457B78` 数组的值进行两两置换，也就是矩阵的运算。

现给出它的伪代码。

```c
void sub_409790(void pasbuf, int par2)
{
  BYTE buf[0x100];
  void pUserNameBuf = pasbuf;
  DWORD len = par2;
  if(par2 < 0)
    len = GetLen(pUserNameBuf);
  DWORD temp = len;
  DWORD index = 0;
  while(index < 0x100)
  {
    bl = (BYTE)index + 0xD;
    457B78[index] = (BYTE)index;
    buf[index] = bl;
    if(len > 0)
    {
      // 根据用户名缓冲区运算后对局部数组赋值
      buf[index] = (*pUserNameBuf + (BYTE)len) ^ bl;
      temp--;
      if(temp != 0)
        pUserNameBuf++;
      else {
        pUserNameBuf = par1;
        temp = len;
      }
    }
    index++;
  }
  [457C78] = 0;
  [457C7C] = 0;
  index = 0;
  temp2 = 0;
  while(index < 0x100)
  {
    // 根据 buf 的值对索引进行运算，然后置换索引对应的值
    dl = 457B78[index];
    temp2 = ((DWORD)dl + (DWORD)buf[index] + temp2) ^ 0xFF;
    457B78[index] = temp2;
    457B78[temp2] = dl;
    index++;
  }
}
```

接着就是两个循环，代码如下：

```asm
.text:00409A85              call    sub_409790      ; 对从内存 457B78 开始的 100h 个字节根据用户名进行处理
.text:00409A8A              add     esp, 8
.text:00409A8D              mov     [esp+140h+index], 100h
.text:00409A95
.text:00409A95 loc_409A95:               ; CODE XREF: CheckPas+1DFj
.text:00409A95              mov     eax, dword_457C7C
.text:00409A9A              mov     esi, dword_457C78
.text:00409AA0              xor     edi, edi        ; edi 索引变量
.text:00409AA2
.text:00409AA2 loc_409AA2:               ; CODE XREF: CheckPas+1B7j
.text:00409AA2              inc     eax
.text:00409AA3              and     eax, 0FFh
.text:00409AA8              mov     dl, byte_457B78[eax]
.text:00409AAE              movzx   ecx, dl
.text:00409AB1              add     ecx, esi
.text:00409AB3              and     ecx, 0FFh
.text:00409AB9              movzx   ebx, byte_457B78[ecx]
.text:00409AC0              mov     byte_457B78[eax], bl
.text:00409AC6              mov     byte_457B78[ecx], dl
.text:00409ACC              movzx   esi, byte_457B78[eax]
```



### 伪代码实现
以上代码根据数组 `457B78` 的值进行，其实质也是对索引进行运算，然后根据索引将 `457B78` 中的值填充到 `UserNameBuf` 中。其伪代码如下：

```c
409790(UserNameBuf, -1);
DWORD num = 0x100;
while(num != 0)
{
  var1 = [457C7C];   //eax
  index1 = [457C78];  //esi
  
  num2 = 0;
  while(num2 < 0x100) 
  {
    for(int i=0; i<4; i++)
    {
      var1 = var1++ & 0xFF;
      temp = 457B78[var1];
      index1 = (temp + index1) & 0xFF;
      457B78[var1] = 457B78[index1];
      457B78[index1] = temp;
      index2 = (temp + 457B78[var1]) & 0xFF;
      UserNameBuf[num2 + i] = 457B78[index2];
    }
    
    num2 += 4;   
  }
  [457C7C] = var1;
  [457C78] = index1;
  sub_409790(UserNameBuf, 0x100);
  num--;
}
```

### 用户名处理结果
此时用户名仍没有处理完毕，接着就是一段很长的代码，也不外乎是对用户名缓冲区数组进行置换赋值。为了节省篇幅，代码就不列出来了。当其中代码执行完毕后，经处理的用户名缓冲区的部分数据如下：

```text
0013F988  23 03 8A 00 F7 EA 2C 57 72 E7 FF 2B A0 CB E2 21  #?麝,Wr?+?
0013F998  29 8F 07 88 91 1F DA 4C 00 F7 49 D4 98 11 ED 2B  }?.?
0013F9A8  A5 06 AA B4 20 45 37 17 9B 60 68 64 92 95 A8 D2  ? E7hdㄒ
0013F9B8  04 99 FC 45 AA DB 1D FB 14 A7 5F B4 11 0C 9F 85  E??.
0013F9C8  D5 05 E1 20 A2 6E 0D 9C E8 73 DA E0 B8 7A BB 48  ??.s 卩
0013F9D8  6E 1D 12 F0 DA AE B6 D6 DB 8B 3B 5C B5 55 A4 30  n 疒舟?\?
………………………………
………………………………
```

至此，用户名已经处理完毕，跟着就是对用户密码进行处理。

### 密码处理逻辑
代码如下：

```asm
.text:0040A0D8             mov     edx, [ebp+password] ; 获取用户密码
.text:0040A0DB             mov     cl, [edx]
.text:0040A0DD             lea     esi, [esp+140h+PasBuf] ; 密码缓冲区
.text:0040A0E1             mov     [esp+140h+index], esi
.text:0040A0E5             mov     bl, 1
.text:0040A0E7             test    cl, cl
.text:0040A0E9             jz      short loc_40A169
.text:0040A0EB             jmp     short loc_40A0F0
.text:0040A0EB ; ---------------------------------------------------------------------------
.text:0040A0ED             align 10h

.text:0040A0F0
.text:0040A0F0 loc_40A0F0:                      ; CODE XREF: CheckPas+6DBj
.text:0040A0F0                                 ; CheckPas+757j
.text:0040A0F0             mov     al, ds:LetterTable
.text:0040A0F5             mov     edx, offset LetterTable ; ASCII "23456789ABC……XYZ"
.text:0040A0FA             test    al, al
.text:0040A0FC             jz      short loc_40A15B
.text:0040A0FE             mov     edi, edi

.text:0040A100
.text:0040A100 loc_40A100:                      ; CODE XREF: CheckPas+6FAj
.text:0040A100             cmp     al, cl          ; 判断密码的某个字符是否在"234……XYZ"字符表中
.text:0040A102             jz      short loc_40A125 ; 密码字符在字母表中
.text:0040A104             mov     al, [edx+1]
.text:0040A107             inc     edx
.text:0040A108             test    al, al
.text:0040A10A             jnz     short loc_40A100 ; 判断密码的某个字符是否在"234……XYZ"字符表中
.text:0040A10C             jmp     short loc_40A15B ; 密码的某个字符不在"234……XYZ"字符表中
.text:0040A10E ; ---------------------------------------------------------------------------
.text:0040A10E
.text:0040A10E exit:                            ; CODE XREF: CheckPas+6BBj
.text:0040A10E                                 ; CheckPas+792j
.text:0040A10E             xor     al, al          ; al = 0
.text:0040A110             pop     edi
.text:0040A111             pop     esi
.text:0040A112             pop     ebx
.text:0040A113             mov     ecx, [esp+134h+var_4]
.text:0040A11A             xor     ecx, esp
.text:0040A11C             call    sub_4322D2
.text:0040A121             mov     esp, ebp
.text:0040A123             pop     ebp
.text:0040A124             retn

.text:0040A125 ; ---------------------------------------------------------------------------
.text:0040A125
.text:0040A125 loc_40A125:                      ; CODE XREF: CheckPas+6F2j
.text:0040A125             movzx   ecx, cl
.text:0040A128             mov     edi, dword_457778[ecx*4] ; EDI = 密码某字符在字母表中的位置
.text:0040A12F             mov     eax, 1
.text:0040A134             lea     edx, [eax+4]    ; edx = 5

.text:0040A137
.text:0040A137 loc_40A137:                      ; CODE XREF: CheckPas+749j
.text:0040A137             test    edi, eax        ; edi 用户某个密码字符做索引取得的值
```

### 汇编指令序列
.text:00409AD3             movzx   edx, dl  
.text:00409AD6             add     edx, esi  
.text:00409AD8             and     edx, 0FFh  
.text:00409ADE             movzx   edx, byte_457B78[edx]  
.text:00409AE5             mov     [esp+edi+140h+UserNameBuf], dl  
.text:00409AE9             inc     eax  
.text:00409AEA             and     eax, 0FFh  
.text:00409AEF             mov     bl, byte_457B78[eax]  
.text:00409AF5             movzx   edx, bl  
.text:00409AF8             add     edx, ecx  
.text:00409AFA             and     edx, 0FFh  
.text:00409B00             movzx   ecx, byte_457B78[edx]  
.text:00409B07             mov     byte_457B78[eax], cl  
.text:00409B0D             mov     byte_457B78[edx], bl  
.text:00409B13             movzx   esi, byte_457B78[eax]  
.text:00409B1A             movzx   ecx, bl  
.text:00409B1D             add     ecx, esi  
.text:00409B1F             and     ecx, 0FFh  
.text:00409B25             movzx   ecx, byte_457B78[ecx]  
.text:00409B2C             inc     eax  
.text:00409B2D             and     eax, 0FFh  
.text:00409B32             mov     bl, byte_457B78[eax]  
.text:00409B38             mov     [esp+edi+140h+var_107], cl  
.text:00409B3C             movzx   ecx, bl  
.text:00409B3F             add     ecx, edx  
.text:00409B41             and     ecx, 0FFh  
.text:00409B47             movzx   edx, byte_457B78[ecx]  
.text:00409B4E             mov     byte_457B78[eax], dl  
.text:00409B54             mov     byte_457B78[ecx], bl  
.text:00409B5A             movzx   esi, byte_457B78[eax]  
.text:00409B61             movzx   edx, bl  
.text:00409B64             add     edx, esi  
.text:00409B66             and     edx, 0FFh  
.text:00409B6C             movzx   edx, byte_457B78[edx]  
.text:00409B73             mov     [esp+edi+140h+var_106], dl  
.text:00409B77             inc     eax  
.text:00409B78             and     eax, 0FFh  
.text:00409B7D             mov     dl, byte_457B78[eax]  
.text:00409B83             movzx   esi, dl  
.text:00409B86             add     esi, ecx  
.text:00409B88             and     esi, 0FFh  
.text:00409B8E             movzx   ecx, byte_457B78[esi]  
.text:00409B95             mov     byte_457B78[eax], cl  
.text:00409B9B             mov     byte_457B78[esi], dl  
.text:00409BA1             movzx   ecx, byte_457B78[eax]  
.text:00409BA8             movzx   edx, dl  
.text:00409BAB             add     edx, ecx  
.text:00409BAD             and     edx, 0FFh  
.text:00409BB3             movzx   edx, byte_457B78[edx]  
.text:00409BBA             mov     [esp+edi+140h+var_105], dl  
.text:00409BBE             add     edi, 4  
.text:00409BC1             cmp     edi, 100h  
.text:00409BC7             jl      loc_409AA2  
.text:00409BCD             mov     dword_457C7C, eax  
.text:00409BD2             lea     eax, [esp+140h+UserNameBuf]  
.text:00409BD6             push    100h  
.text:00409BDB             push    eax  
.text:00409BDC             mov     dword_457C78, esi  
.text:00409BE2             call    sub_409790  
.text:00409BE7             add     esp, 8  
.text:00409BEA             sub     [esp+140h+index], 1  
.text:00409BEF             jnz     loc_409A95  



### 逆向工程分析：序列号校验算法

#### 1. 初始化与循环准备

```assembly
.text:0040A139                      jz      short loc_40A13F  
.text:0040A13B                      or      [esi], bl       ; [esi]为 var118  
.text:0040A13D                      jmp     short loc_40A149  
.text:0040A13F ; ------------------------------------------------------------
.text:0040A13F  
.text:0040A13F loc_40A13F:                       ; CODE XREF: CheckPas+729j  
.text:0040A13F                      mov     esi, [esp+140h+index]  
.text:0040A143                      mov     cl, bl  
.text:0040A145                      not     cl  
.text:0040A147                      and     [esi], cl  
.text:0040A149  
.text:0040A149 loc_40A149:                       ; CODE XREF: CheckPas+72Dj  
.text:0040A149                      add     eax, eax  
.text:0040A14B                      dec     edx  
.text:0040A14C                      add     bl, bl       ; 将 bl 左移一位  
.text:0040A14E                      jnz     short loc_40A157  
.text:0040A150                      inc     esi                 ; 指向密码处理缓冲区的下一个位置  
.text:0040A151                      mov     bl, 1  
.text:0040A153                      mov     [esp+140h+index], esi  
.text:0040A157  
.text:0040A157 loc_40A157:                       ; CODE XREF: CheckPas+73Ej  
.text:0040A157                      test    edx, edx  
.text:0040A159                      jg      short loc_40A137  
.text:0040A15B  
.text:0040A15B loc_40A15B:                       ; CODE XREF: CheckPas+6ECj  
.text:0040A15B                             ; CheckPas+6FCj  
.text:0040A15B                      mov     eax, [ebp+password]  
.text:0040A15E                      mov     cl, [eax+1]     ; 获取密码的下一个字符  
.text:0040A161                      inc     eax             ; 存放密码的地址指针+1  
.text:0040A162                      mov     [ebp+password], eax  
.text:0040A165                      test    cl, cl  
.text:0040A167                      jnz     short loc_40A0F0  
```

我们注意到，代码使用到了`00445AA0`中的数据，其中存放着字符表`"23456789ABCDEFGHJKLMNPQRSTUVWXYZ"`，代码中首先用一个大循环依次取出序列号的每一个字符，然后用一个小循环对每一个序列号与字符表中的字符进行比较，如果不在字符表中直接取下一个字符继续循环。由此可见序列号的有效字符正是字符表中存放的`32`个字符。所以如果序列号中的字符是有效字符，则获取其在字符表中的位置，其方法正是使用函数`sub_409790`在地址`457778`所建立的`100h`双字节数组。其位置值为`00h~1Fh`。下面给出其伪代码。

#### 2. 序列号处理伪代码

代码:

```c
index = PasBuf;
ch = *password;
bl = 1;    // 用于移位运算
while (ch != NULL)
{
  al = *LetterTable; // 字符表"23456789ABCDEFGHJKLMNPQRSTUVWXYZ"
  pTable = LetterTable;
  while(al != ch) {
    pTable++;
    al = *pTable;
  }
  if(al == ch) {
    // 获取 ch 在“234 …… XYZ”中的位置
    pos = 457778[cl*4];
    var = 1;
    count = var + 4;
    while (count > 0) {
      if((pos & var) == 0) {
        cl = bl;
        *index &= ~cl;
      }
      else
        *index |= bl;
      var <<= 1;
      bl <<= 1;
      // 已经处理完 8bit
      if(bl == 0) {
        index++; // 处理下一字节
        bl = 1;   // 又从第一位开始处理
      }
      count--;
    }
  }
  ch = *(++password);  // 取序列号的下一个字符
}
```

由上面代码可见，处理后的序列号根据其位置每`5bit`一个循环生成一个密钥，紧接着这个密钥将作为一个参数传递到函数`sub_4098D0`中，我们跟进这个函数看看执行了些什么运算。

#### 3. 函数 `sub_4098D0` 汇编分析

```assembly
.text:004098D0 sub_4098D0      proc near                 ; CODE XREF: CheckPas+762p  
.text:004098D0  
.text:004098D0 var_5D          = byte ptr -5Dh  
.text:004098D0 var_5C          = dword ptr -5Ch  
.text:004098D0 var_58          = dword ptr -58h  
.text:004098D0 var_54          = dword ptr -54h  
.text:004098D0 var_50          = byte ptr -50h  
.text:004098D0 var_4F          = byte ptr -4Fh  
.text:004098D0 var_4E          = byte ptr -4Eh  
.text:004098D0 var_4D          = byte ptr -4Dh  
.text:004098D0 var_4C          = byte ptr -4Ch  
.text:004098D0 var_4           = dword ptr -4  
.text:004098D0 arg_0           = dword ptr  4  
.text:004098D0  
.text:004098D0                      sub     esp, 60h  
.text:004098D3                      mov     eax, dword_44F050  
.text:004098D8                      xor     eax, esp  
.text:004098DA                      mov     [esp+60h+var_4], eax  
.text:004098DE                      mov     eax, [esp+60h+arg_0]  
.text:004098E2                      push    ebx  
.text:004098E3                      push    ebp  
.text:004098E4                      push    esi  
.text:004098E5                      mov     esi, ecx  
.text:004098E7                      mov     [esp+6Ch+var_5C], eax  
.text:004098EB                      push    edi  
.text:004098EC                      mov     cl, 1  
.text:004098EE                      xor     eax, eax  
.text:004098F0  
.text:004098F0 loc_4098F0:                     ; CODE XREF: sub_4098D0+34j  
.text:004098F0                      test    [esi], cl  
.text:004098F2                      setnz   dl  
.text:004098F5                      add     cl, cl  
.text:004098F7                      mov     [esp+eax+70h+var_50], dl  
.text:004098FB                      jnz     short loc_409900  
.text:004098FD                      mov     cl, 1  
.text:004098FF                      inc     esi  
.text:00409900  
.text:00409900 loc_409900:                     ; CODE XREF: sub_4098D0+2Bj  
.text:00409900                      inc     eax  
.text:00409901                      cmp     eax, 4Bh  
.text:00409904                      jl      short loc_4098F0  
.text:00409906                      lea     eax, [esp+70h+var_50]  
.text:0040990A                      mov     cl, 1  
.text:0040990C                      dec     eax  
.text:0040990D                      mov     edi, offset unk_4505E0  
.text:00409912                      mov     [esp+70h+var_5D], cl  
.text:00409916                      mov     [esp+70h+var_54], eax  
.text:0040991A                      mov     [esp+70h+var_58], 41h  
.text:00409922  
.text:00409922 loc_409922:                     ; CODE XREF: sub_4098D0+11Cj  
.text:00409922                      test    [edi], cl  
.text:00409924                      setnz   dl  
.text:00409927                      add     cl, cl  
.text:00409929                      jnz     short loc_40992E  
.text:0040992B                      inc     edi  
.text:0040992C                      mov     cl, 1  
.text:0040992E  
.text:0040992E loc_40992E:                     ; CODE XREF: sub_4098D0+59j  
.text:0040992E                      xor     eax, eax  
.text:00409930  
.text:00409930 loc_409930:                     ; CODE XREF: sub_4098D0+B1j  
.text:00409930                      test    [edi], cl  
.text:00409932                      jz      short loc_409938  
.text:00409934                      add     dl, [esp+eax+70h+var_50]  
.text:00409938  
.text:00409938 loc_409938:                     ; CODE XREF: sub_4098D0+62j  
.text:00409938                      add     cl, cl  
.text:0040993A                      jnz     short loc_40993F  
.text:0040993C                      inc     edi  
.text:0040993D                      mov     cl, 1  
.text:0040993F  
.text:0040993F loc_40993F:                     ; CODE XREF: sub_4098D0+6Aj  
.text:0040993F                      test    [edi], cl  
.text:00409941                      jz      short loc_409947  
.text:00409943                      add     dl, [esp+eax+70h+var_4F]  
.text:00409947  
.text:00409947 loc_409947:                     ; CODE XREF: sub_4098D0+71j  
.text:00409947                      add     cl, cl  
.text:00409949                      jnz     short loc_40994E  
.text:0040994B                      inc     edi  
.text:0040994C                      mov     cl, 1  
.text:0040994E  
.text:0040994E loc_40994E:                     ; CODE XREF: sub_4098D0+79j  
.text:0040994E                      test    [edi], cl  
.text:00409950                      jz      short loc_409956  
.text:00409952                      add     dl, [esp+eax+70h+var_4E]  
.text:00409956  
.text:00409956 loc_409956:                     ; CODE XREF: sub_4098D0+80j  
.text:00409956                      add     cl, cl  
.text:00409958                      jnz     short loc_40995D  
.text:0040995A                      inc     edi  
.text:0040995B                      mov     cl, 1  
.text:0040995D
```



## 汇编代码片段

```
.text:0040995D loc_40995D: ; CODE XREF: sub_4098D0+88j
.text:0040995D test [edi], cl
.text:0040995F jz short loc_409965
.text:00409961 add dl, [esp+eax+70h+var_4D]

.text:00409965 loc_409965: ; CODE XREF: sub_4098D0+8Fj
.text:00409965 add cl, cl
.text:00409967 jnz short loc_40996C
.text:00409969 inc edi
.text:0040996A mov cl, 1

.text:0040996C loc_40996C: ; CODE XREF: sub_4098D0+97j
.text:0040996C test [edi], cl
.text:0040996E jz short loc_409974
.text:00409970 add dl, [esp+eax+70h+var_4C]

.text:00409974 loc_409974: ; CODE XREF: sub_4098D0+9Ej
.text:00409974 add cl, cl
.text:00409976 jnz short loc_40997B
.text:00409978 inc edi
.text:00409979 mov cl, 1

.text:0040997B loc_40997B: ; CODE XREF: sub_4098D0+A6j
.text:0040997B add eax, 5
.text:0040997E cmp eax, 4Bh
.text:00409981 jl short loc_409930
.text:00409983 mov ebp, 1
.text:00409988 lea ebx, [ebp+49h]
.text:0040998B jmp short loc_409990
.text:0040998B ; ---------------------------------------------------------------
.text:0040998D align 10h

.text:00409990 loc_409990: ; CODE XREF: sub_4098D0+BBj
.text:00409990 ; sub_4098D0+E9j
.text:00409990 cmp ebp, 4Bh
.text:00409993 mov esi, ebp
.text:00409995 jge short loc_4099B5

.text:00409997 loc_409997: ; CODE XREF: sub_4098D0+E3j
.text:00409997 test [edi], cl
.text:00409999 jz short loc_4099A8
.text:0040999B mov eax, [esp+70h+var_54]
.text:0040999F mov al, [eax+ebp]
.text:004099A2 imul [esp+esi+70h+var_50]
.text:004099A6 add dl, al

.text:004099A8 loc_4099A8: ; CODE XREF: sub_4098D0+C9j
.text:004099A8 add cl, cl
.text:004099AA jnz short loc_4099AF
.text:004099AC inc edi
.text:004099AD mov cl, 1

.text:004099AF loc_4099AF: ; CODE XREF: sub_4098D0+DAj
.text:004099AF inc esi
.text:004099B0 cmp esi, 4Bh
.text:004099B3 jl short loc_409997

.text:004099B5 loc_4099B5: ; CODE XREF: sub_4098D0+C5j
.text:004099B5 inc ebp
.text:004099B6 sub ebx, 1
.text:004099B9 jnz short loc_409990
.text:004099BB mov eax, [esp+70h+var_5C]
.text:004099BF test dl, 1
.text:004099C2 mov dl, [esp+70h+var_5D]
.text:004099C6 jz short loc_4099CC
.text:004099C8 or [eax], dl ; dl 中哪些位为 1，则将[eax]中哪些位置 0
.text:004099CA jmp short loc_4099D0
.text:004099CC ; ---------------------------------------------------------------

.text:004099CC loc_4099CC: ; CODE XREF: sub_4098D0+F6j
.text:004099CC not dl
.text:004099CE and [eax], dl ; dl 中哪些位为 1，则将[eax]中那一位置 0

.text:004099D0 loc_4099D0: ; CODE XREF: sub_4098D0+FAj
.text:004099D0 shl [esp+70h+var_5D], 1
.text:004099D4 jnz short loc_4099DF ; 是否已经处理了 8bit 数据
.text:004099D6 inc [esp+70h+var_5C] ; 是则指向下一字节
.text:004099DA mov [esp+70h+var_5D], 1

.text:004099DF loc_4099DF: ; CODE XREF: sub_4098D0+104j
.text:004099DF cmp cl, 1
.text:004099E2 jbe short loc_4099E7
.text:004099E4 inc edi
.text:004099E5 mov cl, 1

.text:004099E7 loc_4099E7: ; CODE XREF: sub_4098D0+112j
.text:004099E7 sub [esp+70h+var_58], 1
.text:004099EC jnz loc_409922
.text:004099F2 mov ecx, [esp+70h+var_4]
.text:004099F6 pop edi
.text:004099F7 pop esi
.text:004099F8 pop ebp
.text:004099F9 pop ebx
.text:004099FA xor ecx, esp
.text:004099FC call sub_4322D2
.text:00409A01 add esp, 60h
.text:00409A04 retn
.text:00409A04 sub_4098D0 endp
```

## 函数分析

此函数运算比较复杂，首先将传入的密钥扩充为字节存放到一个 `75bit` 大小的字节数组中，由此可以推断密钥是 `75bit` 的（如果我没猜错的话），而上面的代码每 `5bit` 处理一个序列号位置，所以又可以推断注册码应该是 `15` 个字符组成。函数接着根据 `0x004505E0` 中的数据对扩充为字节的密钥进行一系列运算。

本来想做个注册机，但问题就卡在这个函数上，貌似其算法不可逆，而且更加不可能用穷举法列举出 `75bit` 密钥的所有可能值传到函数中进行运算。其数量级可是 $10$ 的 $22$ 次方，大概要运行 $10$ 的 $13$ 次方秒。现在的双核 `CPU` 以 $10$ 的 $9$ 次方每秒运算速度看，呵呵，谁能长生不老的话可以试一试用穷举法。

![恐怖!](img/e71d9210a215d5b66755d038631e0c83.jpg)

代码就不再解析了，看伪代码可以略知一二。

## 伪代码

```c
// 此函数有两个参数，一个由寄存器 ecx 传入，一个由堆栈传入
void 4098D0 (IN PBYTE pSouBuf, OUT PBYTE pPusBuf)
{
  PBYTE pDistin = pPusBuf;
  PBYTE pSource = pSouBuf; // 75bit 密钥
  DOWRD index = 0;
  BYTE cl = 1; // 用于控制移位运算
  BYTE dl;
  BYTE buf[4B];
  // 将 75bit 密钥的每一位扩充为字节存放到 buf 中
  while( index < 0x4B ) {
    if( *pSource & cl )
      dl = 1;
    else
      dl = 0;
    buf[index] = dl;
    cl <<= 1;
    if(cl == 0) {
      cl = 1;
      pSource++;
    }
    index++;
  }
  
  bitCountrol = cl = 1;//控制移位运算
  PBYTE pAdd = 4505E0;
  index = 0x41;
  while(index) 
  {
    if(*pAdd & cl)
      dl = 1;
    else
      dl = 0;
    cl <<= 1;
    if(cl == 0) {
      pAdd++;
      cl = 1;
    }
    DWORD count = 0;
    while (count < 0x4B) {
      for(int i = 0; i < 5; i++) {
        if(*pAdd & cl)
          dl += (buf+i)[count];
        cl <<= 1;
        if(cl == 0) {
          pAdd++;
          cl = 1;
        }
      }
      count += 5;    
    }
    
    count = 0x4A;
    var1 = 1;
    while(count) {
      var2 = var1;
      while(var2 < 0x4B) {
        if(*pAdd & cl) 
          dl += (buf-1)[var1] * buf[var2];
        cl <<= 1;
        if(cl == 0) {
          pAdd++;
          cl = 1;
        }
        var2++;
      }
      count--;
      var1++;
    }
    
    temp = dl;
    dl = bitControl;
    if(temp == 0)
      // dl 中哪些位为 1，则将*pDistin 中对应的位置 0
      *pDistin &= ~dl;
    else
      // dl 中哪些位为 1，则将*pDistin 中对应的位置 1
      *pDistin |= dl;
    bitControl <<= 1;
    if(bitCountrol == 0) {
      // 指向下一字节
      pDistin++;
      bitCountrol = 1;
    }
    if (cl > 1) {
      pAdd++;
      cl = 1;
    }
    index--;
  }
}
```

## 序列号处理后的数据

以下是对序列号处理完后的数据：

```
0013F968 EE 07 CE DF 16 24 87 15 11 0F 50 80 00 0D DB BA ?芜$?P..酆
```

## 合法性检验

根据上面步骤，用户名和序列号都已经处理完毕，接着就是对上面两个缓冲区的数据合法性检验了。

```
.text:0040A180 loc_40A180: ; CODE XREF: CheckPas+77Ej
.text:0040A180 mov cl, byte ptr [esp+eax+140h+index] ; 关键比较，两个缓冲区的 5 个字节都相等则正确
.text:0040A184 cmp cl, [esp+eax+140h+UserNameBuf]
```



```asm
.text:0040A188             jnz     short loc_40A196 ; 关键跳转  
.text:0040A18A             inc     eax  
.text:0040A18B             cmp     eax, 5  
.text:0040A18E             jl      short loc_40A180  
.text:0040A190             mov     bl, [esp+140h+var_131]  
.text:0040A194             jmp     short loc_40A198  
```

在这个小循环中，分别从经过处理的用户名和序列号缓冲区中取出 5 字节，分别比较。不相等则返回 `FALSE`。当然，偶的序列号是乱填上去的，所以匹配永远也不会成功。但我们可以利用投机取巧的方法，因为在地址 `0040A188` 处的 `jnz short loc_40A196` 是关键跳转，表示匹配不成功的话就跳到函数返回处，暴力破解时可以用 `not` 指令替代它。但只修补这一个地方，运气好的话，下次重启时能正常运行，但大部分情况下会提示：

![](img/d5186d2a9e9d7120820f0429733f35d3.jpg)

因为还有一个关键比较：

```asm
loc_40A198:                            ; CODE XREF: CheckPas+784j  
.text:0040A198             mov     al, [esp+140h+var_102]  
.text:0040A19C             xor     al, byte ptr [esp+140h+var_124+2]  
.text:0040A1A0             test    al, 3       ; al bl 跳转的关键变量  
.text:0040A1A2             jnz     exit          ; 关键跳转  
```

在程序中局部变量 `var102` 的值必须和 `var_124+2` 的值异或后的低 2 位不能为 1，否则还是返回 `FALSE`。这两个值都是在处理用户名的那个长代码中设置的，应该是对用户名缓冲区进行运算处理后的验证，所以还要将 `0040A1A2` 的跳转指令 `not` 掉。

至此，暴力破解就完成了![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")。

## 二. 算法还原

以下是根据前面分析的伪代码还原的注册算法。

代码:

```c
// 根据 password 中每个字符的位置每 5bit 一组设置 pBuf
VOID ConversePassword(PBYTE password, PBYTE pBuf)
{
  char ch = *password, al;
  BYTE bl = 1;    // 用于移位运算
  BYTE var, count, pos;
  PCHAR pTable;
  PBYTE index = pBuf;
  while (ch != NULL)
  {
    al = *LetterTable;
    pTable = LetterTable;
    while(al != ch && al != NULL) {
      pTable++;
      al = *pTable;
    }
    if(al == ch) {
      // 获取 ch 在“234 …… XYZ”中的位置
      pos = (BYTE)LetterPosition[(DWORD)ch];
      var = 1;
      count = var + 4;
      while (count > 0) {
        if((pos & var) == 0) {
          BYTE cl = bl;
          *index &= ~cl;
        }
        else
          *index |= bl;
        var <<= 1;
        bl <<= 1;
        // 已经处理完 8bit
        if(bl == 0) {
          index++; // 处理下一字节
          bl = 1;   // 又从第一位开始处理
        }
        count--;
      }
    }
    ch = *(++password);  // 取序列号的下一个字符
  } 
}

// 对 pSouBuf 的内容进行加密
// 输入参数：用 ConversePassword 转换后的密码内容
VOID EncryptMsg(IN PBYTE pSouBuf, OUT PBYTE pPasBuf)
{
  PBYTE  pSource = pSouBuf;
  PBYTE  pDistin = pPasBuf;
  DWORD  index = 0;
  BYTE  cl = 1, bitControl;
  BYTE  dl, var1, var2, temp;
  BYTE  buf[0x4B];

  while ( index < 0x4B )
  {
    if ( *pSource & cl)
      dl = 1;
    else
      dl = 0;
    buf[index] = dl;
    cl <<= 1;
    if ( !cl )
    {
      cl = 1;
      pSource++;
    }
    index++;
  }

  cl = bitControl = 1;
  PBYTE pAdd = EncryTable;
  index = 0x41;
  while(index) 
  {
    if(*pAdd & cl)
      dl = 1;
    else
      dl = 0;
    cl <<= 1;
    if(cl == 0) {
      pAdd++;
      cl = 1;
    }
    DWORD count  = 0;
    while (count < 0x4B) {
      for(int i = 0; i < 5; i++) {
        if(*pAdd & cl)
          dl += (buf+i)[count];
        cl <<= 1;
        if(cl == 0) {
          pAdd++;
          cl = 1;
        }
      }
      count += 5;    
    }

    count = 0x4A;
    var1 = 1;
    while(count) {
      var2 = var1;
      while(var2 < 0x4B) {
        if(*pAdd & cl) 
          dl += (buf-1)[var1] * buf[var2];
        cl <<= 1;
        if(cl == 0) {
          pAdd++;
          cl = 1;
        }
        var2++;
      }
      count--;
      var1++;
    }

    temp = dl;
    dl = bitControl;
    if((temp & 1) == 0)
      // dl 中哪些位为 1，则将*pDistin 中对应的位置 0
      *pDistin &= ~dl;
    else
      // dl 中哪些位为 1，则将*pDistin 中对应的位置 1
      *pDistin |= dl;
    bitControl <<= 1;
    if(bitControl == 0) {
      // 指向下一字节
      pDistin++;
      bitControl = 1;
    }
    if (cl > 1) {
      pAdd++;
      cl = 1;
    }
    index--;
  }
}
```

### 后记

此软件主要通过对输入的用户名和密码分别进行一系列运算后比较它们的运算结果。这里仅完整逆向 了它的序列号处理算法，用户名的处理方法就是一系列的矩阵运算，这里就没将 `sub_409790` 整 体还原，因为还原兼测试其序列号处理算法已经谋杀了小菜的几吨脑细胞了。

实力有限，既然写不出注册机，附件给出一个文件补丁，直接修补主程序，所用的方法就是 `not` 上 面的两条跳转指令。附件的另一个程序是还原的序列号处理算法的可运行程序，供学习研究，如果 大牛们觉得上面的注册算法可逆的话请不吝赐教。逆向此软件仅为了研究学习其算法， 如果觉得此软件好用的话，请购买正版！！！![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")

author: 小泡

2009.8.8

破解补丁 和 还原的注册注册算法测试代码

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | WinSnap3.1 Crack.rar |
|---|---|
