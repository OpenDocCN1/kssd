
# GIF Movie Gear 算法分析及注册机编写

> 原文：[`www.pediy.com/kssd/pediy12/139089.html`](https://www.pediy.com/kssd/pediy12/139089.html)

## 基本信息

- **文章标题**: GIF Movie Gear 算法分析及注册机编写
- **文章作者**: 红绡枫叶
- **作者邮箱**: a474528738@163.com
- **作者主页**: ---
- **作者 QQ 号**: 474528738
- **软件名称**: GIF Movie Gear 4.2.3
- **软件大小**: 964 KB
- **下载地址**: 已打包
- **加壳方式**: 无
- **保护方式**: 无
- **编写语言**: Microsoft Visual C++ 7.0
- **使用工具**: `OD`, `PEID 0.94`
- **操作平台**: WIN 7
- **软件介绍**: GIF 动画制作软件，小巧玲珑
- **作者声明**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

## 详细过程

用 `PEID` 查壳，是 Microsoft Visual C++ 7.0 写的，软件非常小。写这个分析是为了照顾新手（本人也是），大侠可以不用看。

废话不多说。`OD` 载入，随便输入，出现错误提示：`The information you have provided is invalid. Please be sure that you typed it exactly as it was given to you.`

查找出错字符串，来到：

```asm
00411EE0  /$  8B4424 04      mov eax,dword ptr ss:[esp+4]            /注意地址
00411EE4  |.  8B0D C4C24A00 mov ecx,dword ptr ds:[4AC2C4]           ;  (Initial CPU selection)
00411EEA  |.  68 00020000   push 200                                  ; /Count = 200 (512.)
00411EEF  |.  68 80BF4A00   push movgear.004ABF80                     ; |The information you have provided is invalid. Please be sure that you typed it exactly as it was given to you.
00411EF4  |.  50             push eax                                  ; |RsrcID
00411EF5  |.  51             push ecx                                  ; |hInst => 00400000
00411EF6  |.  FF15 C0F44700 call dword ptr ds:[<&USER32.LoadStringA>>; \LoadStringA
00411EFC  |.  85C0           test eax,eax
00411EFE  |.  74 0D          je short movgear.00411F0D
00411F00  |.  3D 00020000   cmp eax,200
00411F05  |.  7D 06          jge short movgear.00411F0D
00411F07  |.  B8 80BF4A00   mov eax,movgear.004ABF80                  ;  The information you have provided is invalid. Please be sure that you typed it exactly as it was given to you.
00411F0C  |.  C3             retn
00411F0D  |>  33C0           xor eax,eax
00411F0F  \.  C3             retn
```

一看就知道还没到目的地，往下翻翻看，

```asm
00411F4C  |.  E8 8FFFFFFF   call movgear.00411EE0           /*调用的就是上面的！！
00411F51  |.  83C4 04       add esp,4
00411F54  |.  50             push eax                       ; |Text
00411F55  |.  8B8424 100100> mov eax,dword ptr ss:[esp+110]   ; |
00411F5C  |.  50             push eax                       ; |hOwner
00411F5D  |.  FF15 C4F44700 call dword ptr ds:[<&USER32.MessageBoxA>>; \MessageBoxA  /*这里就是错误提示了
00411F63  |.  81C4 00010000 add esp,100
00411F69  \.  C3             retn
```

继续单步往下，来到：

```asm
00434606   > \6A 30          push 30                       /*这里由上面跳转到来的
00434608   .  68 159D0000   push 9D15
0043460D   .  68 149D0000   push 9D14
00434612   .  57             push edi
00434613   .  E8 F8D8FDFF   call movgear.00411F10         /*上一步的出口
00434618   .  83C4 10       add esp,10
0043461B   .  68 4F040000   push 44F
00434620   .  57             push edi
00434621   .  FFD6          call esi
00434623   .  50             push eax                       ; /hWnd
```

哈哈，来到了关键处:

```asm
0043452D   .  57             push edi                       ; ||hWnd
0043452E   .  FFD6          call esi                       ; |\GetDlgItem
00434530   .  50             push eax                       ; |hWnd
00434531   .  FFD3          call ebx                       ; \GetWindowTextA
00434533   .  8D8C24 C40000> lea ecx,dword ptr ss:[esp+C4]
0043453A   .  51             push ecx
0043453B   .  8D5424 64      lea edx,dword ptr ss:[esp+64]
0043453F   .  52             push edx
00434540   .  E8 EBFBFFFF   call movgear.00434130           /*算法
00434545   .  83C4 08       add esp,8
00434548   .  85C0          test eax,eax
0043454A   .  0F84 B6000000 je movgear.00434606            /*关键跳转
00434550   .  8D4424 10      lea eax,dword ptr ss:[esp+10]
00434554   .  50             push eax                       ; /pDisposition
```

进入算法：

```asm
00434130  /$  53             push ebx
00434131  |.  55             push ebp
00434132  |.  8B6C24 10      mov ebp,dword ptr ss:[esp+10]
00434136  |.  807D 00 6D    cmp byte ptr ss:[ebp],6D    /注册码首字符不是 m 就挂
0043413A  |.  56             push esi                       ;  user32.GetDlgItem
0043413B  |.  57             push edi
0043413C  |.  0F85 AD000000 jnz movgear.004341EF
00434142  |.  807D 01 67    cmp byte ptr ss:[ebp+1],67         /注册码第二个字符不是 g 就挂
00434146  |.  0F85 A3000000 jnz movgear.004341EF
0043414C  |.  807D 02 33    cmp byte ptr ss:[ebp+2],33       /注册码第三个不是 3 就挂
00434150  |.  0F85 99000000 jnz movgear.004341EF
00434156  |.  807D 03 37    cmp byte ptr ss:[ebp+3],37       /注册码第四个不是 7 就挂
0043415A  |.  0F85 8F000000 jnz movgear.004341EF
00434160  |.  33DB          xor ebx,ebx
00434162  |>  8BBB F8F34800 /mov edi,dword ptr ds:[ebx+48F3F8]
00434168  |.  8BC7          |mov eax,edi
0043416A  |.  8D50 01       |lea edx,dword ptr ds:[eax+1]
0043416D  |.  8D49 00       |lea ecx,dword ptr ds:[ecx]
00434170  |>  8A08          |/mov cl,byte ptr ds:[eax]
00434172  |.  40            ||inc eax
00434173  |.  84C9          ||test cl,cl
00434175  |.^ 75 F9          |\jnz short movgear.00434170
00434177  |.  2BC2          |sub eax,edx
00434179  |.  8BC8          |mov ecx,eax
0043417B  |.  8BF5          |mov esi,ebp
0043417D  |.  33C0          |xor eax,eax
0043417F  |.  F3:A6        |repe cmps byte ptr es:[edi],byte ptr ds>
00434181  |.  74 65          |je short movgear.004341E8
00434183  |.  83C3 04       |add ebx,4
00434186  |.  81FB 80000000 |cmp ebx,80
0043418C  |.^ 72 D4          \jb short movgear.00434162
0043418E  |.  807D 04 73    cmp byte ptr ss:[ebp+4],73
00434192  |.  75 01          jnz short movgear.00434195
00434194  |.  45             inc ebp
00434195  |>  8D4D 07       lea ecx,dword ptr ss:[ebp+7]
00434198  |.  51             push ecx
00434199  |.  E8 56BE0300   call movgear.0046FFF4            !! 把注册码从第八位到最后一位整个转换成数字放入 eax
0043419E  |.  8B5C24 18     mov ebx,dword ptr ss:[esp+18]
004341A2  |.  8A13          mov dl,byte ptr ds:[ebx]
004341A4  |.  83C4 04       add esp,4
004341A7  |.  33C9          xor ecx,ecx
004341A9  |.  84D2          test dl,dl
```

分析发现，真正的算法是下面的：

```asm
004341AB  |.  8BFB          mov edi,ebx
004341AD  |.  BE DF0B0000   mov esi,0BDF        注意这里，初始化 ESI=BDF
004341B2  |.  74 26         je short movgear.004341DA
004341B4  |>  0FBED2        /movsx edx,dl      把注册名依次放入 dl
004341B7  |.  41             |inc ecx          控制次数，ECX=ECX+1
004341B8  |.  0FAFD1        |imul edx,ecx      edx=EDX 乘 ECX
004341BB  |.  03F2          |add esi,edx        esi=esi+edx
004341BD  |.  81FE BE170000 |cmp esi,17BE      如果 esi＞17be，则 esi=esi-17BE
004341C3  |.  7E 06          |jle short movgear.004341CB
004341C5  |.  81EE BE170000 |sub esi,17BE
004341CB  |>  83F9 0A       |cmp ecx,0A        如果用户名长度>10，则 esi=esi-17BE
004341CE  |.  7E 02          |jle short movgear.004341D2
004341D0  |.  33C9          |xor ecx,ecx
004341D2  |>  8A57 01       |mov dl,byte ptr ds:[edi+1]
004341D5  |.  47             |inc edi
004341D6  |.  84D2          |test dl,dl
004341D8  |.^ 75 DA          \jnz short movgear.004341B4
```



### 汇编分析

`004341DA | > 3BF0 cmp esi,eax esi=eax 就注册成功`

`004341DC |. 75 15 jnz short movgear.004341F3`

`004341DE |. 5F pop edi`

### 算法

把注册码从第八位到最后一位整个转换成数字放入 `eax`，初始化 `ESI`=`BDF`，把注册名依次放入 `dl`，`edx`=`EDX` 乘循环次数，`esi`=`edx`+`esi`。

如果 `esi`＞`17be`，则 `esi`=`esi`-`17BE`，如果用户名长度>`10`，则 `esi`=`esi`-`17BE`，`esi`=`eax` 就注册成功。

### 注册机源代码（VB）

以下是注册机源代码（VB）：

```vb
dim sname,name,tname,lname，code

sname=3039

name=Text1.text

lname=Len(name)

For i=1 To lname
tname=Asc(Mid(name,i,1))*i
sname=sname+tname

If sname>6078 Then
sname=sname-6078
End If

If lname>10 Then
sname=sname-6078
End if

Next

code="mg37fng"&CStr(sname)

text1.captain=code
```

注：VB 源码并不完善，不能输入中文用户名，如那位写出完整版的源码（哪种语言都可以），小弟就在此感激不尽。

--------------------------------------------------------------------------------

**【版权声明】**:  转载请注明作者并保持文章的完整, 谢谢!

2011 年 08 月 07 日 16:16:22

### 上传的附件

![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) GIF Movie Gear+注册机.rar

### 以前对 V4.1.2 的分析

注册码前 4 位为：`mg37`

第 5 位不为 `s`

`6`,`7` 位任意

由用户名算出的 `esi` 十进制为注册码第 8 位到最后一位

#### C# 注册机部分代码

```csharp
string xk;

xk = this.UserName.Text.Trim();

int ecx = 0;

int esi = 0x0BDF;

int edx = 0;

char[] ch;

ch = xk.ToCharArray();

int len = ch.Length;

for (int i = 0; i <= len - 1; i++)
{
edx = (short)ch[i];
ecx = ecx + 1;
edx = edx * ecx;
esi = esi + edx;

if (esi > 0x17BE) esi = esi - 0x17BE;

if (ecx > 0xA) ecx = 0;
}

string s = "mg37123";

this.SN.Text = s + esi.ToString();
```
