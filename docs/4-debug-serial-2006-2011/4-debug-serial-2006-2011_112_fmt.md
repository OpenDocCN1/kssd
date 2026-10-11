
# Posbase 1.04

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-142.htm`](https://www.pediy.com/kssd/pediy08/pediy8-142.htm)

万里长征第三步：`Posbase 1.04` 注册算法分析

解密者：冲天剑@pediy.com

工具：`peid 0.94`, `ollyice 1.10`

## 0. 导言

`Posbase 1.04` 是个国际象棋方面的应用程序，据其作者称它可以用来储存国际象棋开局的特定局面，作成数据库以方便查询检索。至于这样做有什么特别好处，却未详加介绍。一个引人注目的缺点是占用磁盘空间过大，譬如一个 1,000,000 盘对局的数据库竟需要 7GB，而相同数量对局的 `chessbase` 格式数据库只要一张 CD 就能装下了。

下载地址：`hxxp://www.wmlsoftware.com/download.html`（请自行改正）

## 1. 解密

(1) `PEID` 查壳，结果显示 `Borland Delphi 4.0 - 5.0 [Overlay]`。

(2) 运行原程序试注册，出现错误信息：`"Invalid code"`。用调试器载入，查找此字串。

```
///////////    以下是代码    ///////////

00591B02   .  8B45 E8       mov     eax, [ebp-18]
00591B05   .  8D55 E4       lea     edx, [ebp-1C]
00591B08   .  E8 1370E7FF   call    00408B20
00591B0D   .  8B45 E4       mov     eax, [ebp-1C]
00591B10   .  5A           pop     edx
00591B11   .  E8 82060000   call    00592198
00591B16   .  84C0         test    al, al
00591B18   .  75 75        jnz     short 00591B8F            ; 关键跳转
00591B1A   .  E8 C5B8F1FF   call    004AD3E4
00591B1F   .  33C0         xor     eax, eax
00591B21   .  55           push    ebp
00591B22   .  68 6C1B5900  push    00591B6C
00591B27   .  64:FF30     push    dword ptr fs:[eax]
00591B2A   .  64:8920     mov     fs:[eax], esp
00591B2D   .  E8 7652E7FF   call    <jmp.&kernel32.GetTickCount>    ; [GetTickCount
00591B32   .  8945 F8     mov     [ebp-8], eax
00591B35   >  E8 6E52E7FF   call    <jmp.&kernel32.GetTickCount>    ; [GetTickCount
00591B3A   .  8945 F4     mov     [ebp-C], eax
00591B3D   .  8B45 F4     mov     eax, [ebp-C]
00591B40   .  3B45 F8     cmp     eax, [ebp-8]
00591B43   .  72 14        jb      short 00591B59
00591B45   .  8B45 F8     mov     eax, [ebp-8]
00591B48   .  05 10270000  add     eax, 2710
00591B4D   .  73 05        jnb     short 00591B54
00591B4F   .  E8 6013E7FF   call    00402EB4
00591B54   >  3B45 F4     cmp     eax, [ebp-C]
00591B57   .^ 73 DC        jnb     short 00591B35
00591B59   >  33C0        xor     eax, eax
00591B5B   .  5A           pop     edx
00591B5C   .  59           pop     ecx
00591B5D   .  59           pop     ecx
00591B5E   .  64:8910     mov     fs:[eax], edx
00591B61   .  68 731B5900  push    00591B73
00591B66   >  E8 35B8F1FF   call    004AD3A0
00591B6B   .  C3           retn
00591B6C   .^ E9 B71AE7FF   jmp     00403628
00591B71   .^ EB F3        jmp     short 00591B66
00591B73   .  33C9        xor     ecx, ecx
00591B75   .  BA 101C5900  mov     edx, 00591C10                ;   ASCII "Invalid code"
00591B7A   .  8B45 FC     mov     eax, [ebp-4]
00591B7D   .  E8 92B4F1FF   call    004AD014
00591B82   .  8B45 FC     mov     eax, [ebp-4]
00591B85   .  33D2        xor     edx, edx
00591B87   .  8990 2C020000 mov    [eax+22C], edx
00591B8D   .  EB 45        jmp     short 00591BD4
00591B8F   >  8D55 E8     lea     edx, [ebp-18]
///////////    以上是代码    ///////////
```

这次的关键跳转不太好找，`00591B75` 以上的几行代码显得很没有逻辑性，不知道从程序的什么地方能够跳到这里，而再往上的几个 `jb`，`jnb` 其跳转范围又局限在几行之内，不象是关键跳转，好不容易再往前找到一个 `jnz` 跳过了错误信息，才敢推测它是关键跳转。

于是在它前面的 `00591B05` 处下断点，步过 `call 00592198` 这一行以后才从堆栈里翻出了真注册码，跟进 `00592198` 一看，这还只是比较真码跟输入码的地方，算法还在里面一层。不管怎样说，还是列出这个过程的一个小片段：

```
///////////    以下是代码    ///////////

005921F0  |.  8B45 FC     mov     eax, [ebp-4]
005921F3  |.  E8 2CFEFFFF   call    00592024                      ;   算密码的 Call
005921F8  |.  8B45 E8     mov     eax, [ebp-18]
005921FB  |.  8D55 EC     lea     edx, [ebp-14]
005921FE  |.  E8 4567E7FF   call    00408948
00592203  |.  8B55 EC     mov     edx, [ebp-14]
00592206  |.  58           pop     eax
00592207  |.  E8 081EE7FF   call    00404014                      ;   比较真假码
0059220C  |.  74 04        je      short 00592212
///////////    以上是代码    ///////////
```

以下是跟进 `00592024` 的结果：

```
///////////    以下是代码    ///////////

00592024  /$  55           push    ebp
00592025  |.  8BEC         mov     ebp, esp
00592027  |.  81C4 28FFFFFF add     esp, -0D8
0059202D  |.  33C9         xor     ecx, ecx
0059202F  |.  898D 28FFFFFF mov    [ebp-D8], ecx
00592035  |.  8955 F8     mov     [ebp-8], edx
00592038  |.  8945 FC     mov     [ebp-4], eax                  ;   入口参数：EAX=用户名指针
0059203B  |.  8B45 FC     mov     eax, [ebp-4]
0059203E  |.  E8 7520E7FF   call    004040B8
00592043  |.  33C0         xor     eax, eax
00592045  |.  55           push    ebp
00592046  |.  68 6C215900  push    0059216C
0059204B  |.  64:FF30     push    dword ptr fs:[eax]
0059204E  |.  64:8920     mov     fs:[eax], esp
00592051  |.  837D FC 00    cmp     dword ptr [ebp-4], 0
00592055  |.  0F84 F0000000 je      0059214B                      ;   确保用户名指针有效
0059205B  |.  EB 0B        jmp     short 00592068
0059205D  |>  8D45 FC     /lea     eax, [ebp-4]
00592060  |.  8B55 FC     |mov     edx, [ebp-4]
00592063  |.  E8 A41EE7FF   |call    00403F0C                      ;   字符串连接
00592068  |>  8B45 FC       mov     eax, [ebp-4]
0059206B  |.  E8 941EE7FF   |call    00403F04                      ;   测串长
00592070  |.  83F8 17       |cmp     eax, 17
00592073  |.^ 7C E8         \jl      short 0059205D                ;   与自身连接，直到串长大于 23
00592075  |.  C745 F4 01000>mov     dword ptr [ebp-C], 1
0059207C  |>  8B45 F4     /mov     eax, [ebp-C]
0059207F  |.  8B55 FC     |mov     edx, [ebp-4]                  ;   连接后的串指针
00592082  |.  48           |dec     eax
00592083  |.  3B42 FC       |cmp     eax, [edx-4]
00592086  |.  72 05        |jb      short 0059208D              ;   未到达串尾，跳
00592088  |.  E8 1F0EE7FF   |call    00402EAC
0059208D  |>  40           |inc     eax
0059208E  |.  0FB64402 FF  |movzx   eax, byte ptr [edx+eax-1]
00592093  |.  6BC0 02       |imul    eax, eax, 2                 ;   取此串中每个字符的 ASCII 码乘 2
00592096  |.  71 05        |jno     short 0059209D
00592098  |.  E8 170EE7FF   |call    00402EB4
0059209D  |>  8B55 F4     |mov     edx, [ebp-C]
005920A0  |.  B9 80215900  |mov     ecx, 00592180                 ;   ASCII "wilhelmusvanoranjenasau"
005920A5  |.  4A           |dec     edx
005920A6  |.  3B51 FC       |cmp     edx, [ecx-4]                  ;   前二行字串的长度为 23
005920A9  |.  72 05        |jb      short 005920B0              ;   未到达串尾，跳
005920AB  |.  E8 FC0DE7FF   |call    00402EAC
005920B0  |>  42           |inc     edx
005920B1  |.  0FB65411 FF  |movzx   edx, byte ptr [ecx+edx-1]
005920B6  |.  33C2         |xor     eax, edx                    ;   与给定字串相应字符 ASCII 码值异或
005920B8  |.  8B55 F4     |mov     edx, [ebp-C]
005920BB  |.  83C2 01       |add     edx, 1                      ;   下一个字符
005920BE  |.  71 05        |jno     short 005920C5
005920C0  |.  E8 EF0DE7FF   |call    00402EB4
005920C5  |>  B9 80215900  |mov     ecx, 00592180                 ;   ASCII "wilhelmusvanoranjenasau"
005920CA  |.  4A           |dec     edx
005920CB  |.  3B51 FC       |cmp     edx, [ecx-4]
005920CE  |.  72 05        |jb      short 005920D5
005920D0  |.  E8 D70DE7FF   |call    00402EAC
005920D5  |>  42           |inc     edx
005920D6  |.  0FB65411 FF  |movzx   edx, byte ptr [ecx+edx-1]
005920DB  |.  33C2         |xor     eax, edx                    ;   再异或
005920DD  |.  8B55 F4     |mov     edx, [ebp-C]
005920E0  |.  4A           |dec     edx
005920E1  |.  83FA 31       |cmp     edx, 31
005920E4  |.  76 05        |jbe     short 005920EB
005920E6  |.  E8 C10DE7FF   |call    00402EAC
005920EB  |>  42           |inc     edx
005920EC  |.  898495 28FFFF>mov     [ebp+edx*4-D8], eax          ;   保存
```



```
005920F3  |.  FF45 F4       |inc     dword ptr [ebp-C]              ;  下一个  

005920F6  |.  837D F4 09    |cmp     dword ptr [ebp-C], 9  

005920FA  |.^ 75 80         \\jnz     short 0059207C                ;  当比较完 8 个字符后另行处理  

005920FC  |.  8B45 F8       |mov     eax, [ebp-8]  

005920FF  |.  E8 841BE7FF   |call    00403C88  

00592104  |.  C745 F4 01000>|mov     dword ptr [ebp-C], 1  

0059210B  |>  8D8D 28FFFFFF |lea     ecx, [ebp-D8]  

00592111  |.  8B45 F4       |mov     eax, [ebp-C]  

00592114  |.  48            |dec     eax  

00592115  |.  83F8 31       |cmp     eax, 31  

00592118  |.  76 05         |jbe     short 0059211F  

0059211A  |.  E8 8D0DE7FF   |call    00402EAC  

0059211F  |>  40            |inc     eax  

00592120  |.  8B8485 28FFFF>|mov     eax, [ebp+eax*4-D8]  

00592127  |.  BA 02000000   |mov     edx, 2  

0059212C  |.  E8 D36BE7FF   |call    00408D04  

00592131  |.  8B95 28FFFFFF |mov     edx, [ebp-D8]                  ;  把上面保存的异或值转换为 ASCII 字串  

00592137  |.  8B45 F8       |mov     eax, [ebp-8]  

0059213A  |.  E8 CD1DE7FF   |call    00403F0C  

0059213F  |.  8B45 F8       |mov     eax, [ebp-8]  

00592142  |.  FF45 F4       |inc     dword ptr [ebp-C]  

00592145  |.  837D F4 09    |cmp     dword ptr [ebp-C], 9  

00592149  |.^ 75 C0         \\jnz     short 0059210B  

0059214B  |>  33C0          |xor     eax, eax  

0059214D  |.  5A            |pop     edx  

0059214E  |.  59            |pop     ecx  

0059214F  |.  59            |pop     ecx  

00592150  |.  64:8910       |mov     fs:[eax], edx  

00592153  |.  68 73215900   |push    00592173  

00592158  |>  8D85 28FFFFFF |lea     eax, [ebp-D8]  

0059215E  |.  E8 251BE7FF   |call    00403C88  

00592163  |.  8D45 FC       |lea     eax, [ebp-4]  

00592166  |.  E8 1D1BE7FF   |call    00403C88  

0059216B  \\ .  C3            |retn  

0059216C    .^ E9 B714E7FF   |jmp     00403628  

00592171    .^ EB E5         |jmp     short 00592158  

00592173    .  8BE5          |mov     esp, ebp  

00592175    .  5D            |pop     ebp  

00592176    .  C3            |retn  
```

在 `0059216C` 等两行处又出现没有逻辑性的代码，不过不管它了。还有一个问题，为什么访问一个字符串指针减 4 的 `dword` 存储单元就总能取到这个字符串的长度，如果这样的话那么 `strlen` 之类的函数岂不是多此一举了吗？好了，废话不多说，现在把算法总结如下：

`c$` = 常字符串 `"wilhelmusvanoranjenasau"`

`u$` = 读入用户名字符串

循环：当 (`u$` 长度小于 23) `u$` = `u$` 与自身连接

`i` = 1

`p$` = 空字符串

循环：当 (`i` < 9)

  `a` = `u$` 第 `i` 个字符 `ASCII` 码 * 2

  `b` = `c$` 第 `i` 个字符 `ASCII` 码

  `a` = `a` xor `b`

  `b` = `c$` 第 `i+1` 个字符 `ASCII` 码

  `a` = `a` xor `b`

  `s$` = 将 `a` 的 16 进制表示转换为长度为 2 的 `ASCII` 字串

  `p$` = `p$` 连接 `s$`

  `i` = `i` + 1

`return p$`——注册码

由此可见，实际上没有必要把 `u$` 自身连接到长度大于 23。并且由于异或运算的结合律，我们可以先将 `c$` 的相邻字符先两两异或，得到一组数据，然后直接引用这组数据。

`w` (119) xor `i` (105) = 30

`i` (105) xor `l` (108) = 5

`l` (108) xor `h` (104) = 4

`h` (104) xor `e` (101) = 13

`e` (101) xor `l` (108) = 9

`l` (108) xor `m` (109) = 1

`m` (109) xor `u` (117) = 24

`u` (117) xor `s` (115) = 6

### (3)注册函数

```
char *szHexToAsc(unsigned char uValue)

/* 实现把一字节的 16 进制数值 uValue 转化为长度为 2 的字符串表示 */ 

{
     
    /* 代码略 */
} 

char *szStrCat(char *szString1, char *szString2)

/* 将字符串 szString2 追加到去掉'\0'的 szString1 末尾 */

/* szStrCat 返回追加后的 szString1 指针 */ 

{
     
    /* 代码略 */
}

unsigned int uStrLen(char *szString)

/* 返回字符串 szString 的长度 */ 

{
     
    /* 代码略 */
}

void KeyGen(char *szUserName, char *szUnlockCode)

/* 注册码计算程序 */ 

/* 入口参数：szUserName——用户名（应在主调程序中留出足够大数组空间） 
           
szUnlockCode——用于存放注册码的缓冲区（初始化为 0） 
 
出口参数：szUnlockCode——注册码

*/ 

{
     
    unsigned char a, i = 0, xordata[] = {30, 5, 4, 13, 9, 1, 24, 6};
     
    if (uStrLen(szUserName) == 0) return;
     
    while(uStrLen(szUserName) < 8) 
  
        szUserName = szStrCat(szUserName, szUserName);
     
    while(i < 8)
  
        {
           
           a = (szUserName[i] << 1) ^ xordata[i];   
  
           szUnlockCode = szStrCat(szUnlockCode, szHexToAsc(a));
  
           i = i + 1;
  
        }     
 
     return;
 
}
```

### 2. 感言

到目前为止鄙人所解的程序都没有壳保护，从某个层面上说颇有专捏软柿子之嫌，其实我也碰到过不少有壳保护的软件，并且也很想学脱壳，但研究电脑毕竟不是我的专业，并且我还正在做自己方向的论文，怕在这方面投入太多精力会影响工作，另外我的个人电脑坏了，目前还没钱买新的，从一方面说也是没条件。

`4dd466d0e1e0cc123afbe504da2462ca`
