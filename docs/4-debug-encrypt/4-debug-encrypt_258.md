
# Matlab2007b PLP 之不完全分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-286.htm`](https://www.pediy.com/kssd/pediy09/pediy09-286.htm)

> 【破文标题】Matlab2007b PLP 之不完全分析  

> 【破文作者】Ptero  

> 【破解工具】Peid, Ollydbg, IDA, DJ Java Decomplier  

> 【注册方式】序列号  

> 【加壳方式】无  

> 【加密算法】变形 RC5 + 自定义  

> 【软件限制】安装限制  

> 【软件简介】不介绍了，很强大的工具。用过的都知道。想看的话去这里吧：[`www.mathworks.cn/products/new...\_features.html`](http://www.mathworks.cn/products/new_products/latest_features.html)  

> 【破解难度】中等  

>   

> \----------  

>   

> 【破解分析】  

>   

Mathworks 在 9 月份推出了 Matlab 的最新版本 Matlab2007b，在网上找了好久，终于用 BT 拉了回来。可是没有 PLP，安装不了。眼看着最新版本就不能用吗？自己动手，丰衣足食！

下载到的文件是个 rar 压缩包，将其全部解压到 C 盘，安装文件在这里：`C:\Matlab 2007b Full Release (no keygen)\win32\setup.exe`。用 `Peid` 看一下，没有加壳。

软件是用 java 编译的。软件启动的时候把 `C:\Matlab 2007b Full Release (no keygen)\win32\bin\win32\` 目录和 `C:\Matlab 2007b Full Release (no keygen)\win32\java\` 目录复制到 `Temp` 文件夹，然后就开始调用 java 的函数了。

因为是第一次接触 java 编译的东西，还不熟悉，在这里耽搁了很长时间之后，终于找到了启动的文件：

`C:\Matlab 2007b Full Release (no keygen)\win32\java\jre\win32\jre\lib\ext\installer.jar.`

将其解压，反编译其中的 `licenseUtil.class`，找到了关键部分：

```java
static boolean processPLP()
{
    String s = getPLP();
    if(s.length() == 0)
    {
        String s1 = myRes.getString("plpalert.title");
        String s2 = myRes.getString("plpalert.message");
        WIOptionPane.show(Installer.getInstance(), s2, s1, 0, -1);
        return false;
    }
    int ai[] = new int[250];
    for(int j = 0; j < 250; j++)
        ai[j] = 0;

    int i = mwinstall.DecipherPLP(plp, ai);
    String s3 = myRes.getString("plperroralert.title");
    if(i != 0)
    {
        String s4 = myRes.getString("plperroralert.incorrect");
        WIOptionPane.show(Installer.getInstance(), s4, s3, 0, -1);
        return false;
    }
    int k = mwinstall.getCDVolNbr();
    if(k < util.getPasscodeVersionNumber())
    {
        String s5 = myRes.getString("plperroralert.release");
        WIOptionPane.show(Installer.getInstance(), s5, s3, 0, -1);
        return false;
    }
    if(mwinstall.CheckExpDate() != 0)
    {
        String s6 = myRes.getString("plperroralert.expired");
        WIOptionPane.show(Installer.getInstance(), s6, s3, 0, -1);
        return false;
    }
    if(!isDemo())
    {
        int l = ai[0];
        ai[0] = getNextProduct();
        ai[0] = l;
    }
    PLPArray = ai;
    ProductContainer productcontainer = Installer.getProductContainer();
    productcontainer.resetSelection();
    int ai1[] = PLPArray;
    int i1 = ai1.length;
    for(int j1 = 0; j1 < i1; j1++)
    {
        int k1 = ai1[j1];
        ArrayList arraylist = productcontainer.getProductsByProductNumber(k1);
        MWProduct mwproduct;
        for(Iterator iterator = arraylist.iterator(); iterator.hasNext(); mwproduct.setLicenseNumber(getLicenseNumber()))
            mwproduct = (MWProduct)iterator.next();

    }

    if(Installer.isDws() && !isDemo() && !isStudent() && !util.getUpdate())
        DownloadServiceWrapper.checkForUpdates();
    return true;
}
```

代码没有被混淆过，一目了然。

关键的判断 `PLP` 的函数在这里：

`mwinstall.DecipherPLP(plp, ai)`。

当然这里可以爆破的：修改代码，绕过所有的验证，然后再编译回去，这样就可以输入任何 `PLP` 进行安装了。但是这样做，不知道程序安装之后还会不会再次验证 `PLP`。万一再次验证，那样跟踪起来就没现在看着源代码分析起来这么方便了。还是在这里用釜底抽薪的办法，得出 `PLP` 的好。这样就不用对原程序进行任何修改了，而且也不用担心安装以后的再次验证。一劳永逸。

下面反编译 `mwinstall.class`，看到这样一句：

```java
static native int DecipherPLP(String s, int ai[]);
```

这个重要函数被编译成 native 方式了。这样就看不到源代码啦。不过没关系，我们有 OD 呢。

用 OD 载入 `setup.exe`，按 F9 运行。

出现输入 PLP 的对话框之后，在 OD 里面按 Alt+E，然后选择 `mwinstall` 模块。按 Ctrl+N 查找当前模块的所有标签，在输出表里面找到 `Java_com_mathworks_installer_mwinstall_DecipherPLP` 函数，按 F2 下断点，然后回到安装窗口输入 PLP，断下。

找到 `C:\Matlab 2007b Full Release (no keygen)\win32\bin\win32\mwinstall.dll` 文件，发现没有加壳，于是用 IDA 分析。OD+IDA，动静结合。

下面的代码来自 IDA：

```asm
……
……
.text:10008AAF             push    edi
.text:10008AB0             push    0
.text:10008AB2             push    ebx
.text:10008AB3             push    esi
.text:10008AB4             call    ecx                 ; 获得输入的 PLP
.text:10008AB6             lea     edx, [esp+10h]
.text:10008ABA             mov     edi, eax
.text:10008ABC             push    edx
.text:10008ABD             push    edi
.text:10008ABE             call    sub_10008A00        ; 这个就是关键 Call 啦
```

按 F7 跟进：

```asm
……
……
.text:10008A39             sub     eax, edx
.text:10008A3B             push    1                 ; size_t
.text:10008A3D             push    eax                 ; size_t
.text:10008A3E             call    _calloc
.text:10008A3E

.text:10008A43             mov     edi, eax
.text:10008A45             push    edi
.text:10008A46             push    esi
.text:10008A47             call    sub_100094F0        ; 把输入的 PLP 去掉前 2 位，去掉连字符，把每个用连字符连接的数字转换成 16 进制
```

按 F7 跟进：

```asm
……
……
.text:10007F1A             push    ebp
.text:10007F1B             lea     edi, [esp+234h+var_1F8]
.text:10007F1F             call    sub_10009030    ; 初始化 RC5 密钥
```

按 F7 跟进：

```asm
……
……
.text:10009085             push    eax
.text:10009086             push    ebx                 ; 字符串 "Copyright (C) 1990-2003 by The MathWorks, Inc."
.text:10009087             call    sub_10008E60        ; 初始化 RC5 密钥
```

按 F7 跟进，看看密钥的初始化：

```asm
……
……
.text:10008E83             mov     word ptr [esp+18h+var_8], 6Fh    ; 字母 'o'
……
……
.text:10008ED0             lea     eax, [esp+18h+var_8]
.text:10008ED4             mov     ecx, eax
……
……
.text:10008EE3             mov     esi, ecx
……
……
.text:10008EF2             mov     ecx, eax
.text:10008EF4             shr     ecx, 2
.text:10008EF7             rep movsd
```

这些代码的作用就是在原来字符串的末尾加上一个 'o'，这样，就得到了密钥字符串：

`"Copyright (C) 1990-2003 by The MathWorks, Inc.o"`

加上末尾的 0，一共是 48 字节。

```asm
……
……
.text:10008F58             mov     dword ptr [eax], 0B7E15163h    ; RC5 的初始常数之一
……
……
.text:10008F72             mov     esi, [ebp+0]
.text:10008F75             mov     edi, [esi+eax*4-4]
.text:10008F79             lea     esi, [esi+eax*4]
.text:10008F7C             sub     edi, 61C88647h                ; 就是加上 0x9E3779B9，RC5 的另一个初始常数
.text:10008F82             add     eax, 1
.text:10008F85             cmp     eax, ecx
.text:10008F87             mov     [esi], edi
.text:10008F89             jl      short loc_10008F72
```

上面这个循环初始化一个常数数组。

接着，把数组与初始密钥混合：

计算循环次数：

```asm
……
……
.text:10008F8F             cmp     esi, ecx
.text:10008F91             lea     eax, [esi+esi*2]
.text:10008F94             jg      short loc_10008F99
.text:10008F94

.text:10008F96             lea     eax, [ecx+ecx*2]
```

初始化一些值：

```asm
……
……
.text:10008F99             xor     edi, edi        ; B=0
.text:10008F9B             xor     esi, esi        ; A=0
.text:10008F9D             xor     ebx, ebx        ; j=0
```

下面正式进入循环：



### 汇编分析：密钥初始化与循环

```
.text:10008FB0    mov     ecx, [esp+18h+arg_0]
.text:10008FB4    mov     eax, ebx
.text:10008FB6    cdq
.text:10008FB7    idiv    ecx            ; i=(i+1) mod 2(r+1)
.text:10008FB9    mov     ecx, [ebp+0]
.text:10008FBC    add     edx, edx
.text:10008FBE    add     edx, edx
.text:10008FC0    lea     eax, [ecx+edx]
.text:10008FC3    lea     ecx, [edi+esi]
.text:10008FC6    add     ecx, [eax]
.text:10008FC8    rol     ecx, 3         ; A=S[i]=(S[i]+A+B)<<<3
.text:10008FCB    mov     [eax], ecx
.text:10008FCD    mov     eax, [ebp+0]
.text:10008FD0    mov     esi, [edx+eax]
.text:10008FD3    mov     eax, ebx
.text:10008FD5    cdq
.text:10008FD6    idiv    [esp+18h+arg_4]    ; j=(j+1) mod c
.text:10008FDA    mov     ecx, [esp+18h+var_8]
.text:10008FDE    lea     eax, [edi+esi]
.text:10008FE1    add     eax, [ecx+edx*4]
.text:10008FE4    add     ebx, 1
.text:10008FE7    lea     edx, [ecx+edx*4]
.text:10008FEA    lea     ecx, [edi+esi]
.text:10008FED    and     ecx, 1Fh
.text:10008FF0    rol     eax, cl        ; B=L[j]=(L[j]+A+B)<<<((A+B)&0x1F)
.text:10008FF2    cmp     ebx, [esp+18h+arg_8]    ; 循环 3*max(2(r+1),c)次
.text:10008FF6    mov     [edx], eax
.text:10008FF8    mov     edi, eax
.text:10008FFA    jl      short loc_10008FB0
```

至此，密钥初始化完毕。

结合 OD 跟踪，可以得到 `RC5` 所用的参数：

- 分组长度 `2w＝64`
- 叠代层数 `r=10`
- 密钥长度 `b＝48`

因此，所用的算法是 `RC5 - w=32/r=10/b=48`。

下面就是混合后的 48 字节密钥：

`43 6F 70 79 72 69 67 68 74 20 28 43 29 20 31 39 39 30 2D 32 30 30 33 20 62 79 20 54 68 65 20 4D 61 74 68 57 6F 72 6B 73 2C 20 49 6E 63 2E 6F 00`

产生密钥以后，来到这里解密：

### 汇编分析：RC5 解密循环

```
.text:100090E0    movzx   eax, byte ptr [edi]
.text:100090E3    movzx   ecx, byte ptr [esi-2]
.text:100090E7    movzx   edx, byte ptr [esi-1]
.text:100090EB    shl     eax, 8
.text:100090EE    add     eax, ecx
.text:100090F0    movzx   ecx, byte ptr [esi]
.text:100090F3    shl     eax, 8
.text:100090F6    add     eax, edx
.text:100090F8    movzx   edx, byte ptr [esi+3]
.text:100090FC    shl     eax, 8
.text:100090FF    add     eax, ecx
.text:10009101    movzx   ecx, byte ptr [esi+2]
.text:10009105    mov     [esp+18h+var_8], eax
.text:10009109    movzx   eax, byte ptr [esi+1]
.text:1000910D    shl     eax, 8
.text:10009110    add     eax, ecx
.text:10009112    movzx   ecx, byte ptr [esi+4]
.text:10009116    shl     eax, 8
.text:10009119    add     eax, edx
.text:1000911B    push    1
.text:1000911D    lea     edx, [esp+1Ch+var_8]
.text:10009121    shl     eax, 8
.text:10009124    push    edx
.text:10009125    add     eax, ecx
.text:10009127    push    ebp
.text:10009128    mov     [esp+24h+var_4], eax
.text:1000912C    call    sub_10008DD0     ; RC5 解密函数
.text:10009131    mov     eax, [esp+24h+var_8]
.text:10009135    mov     [esi], al
.text:10009137    shr     eax, 8
.text:1000913A    mov     [esi-1], al
.text:1000913D    shr     eax, 8
.text:10009140    mov     [esi-2], al
.text:10009143    shr     eax, 8
.text:10009146    mov     [edi], al
.text:10009148    mov     eax, [esp+24h+var_4]
.text:1000914C    mov     [esi+4], al
.text:1000914F    shr     eax, 8
.text:10009152    mov     [esi+3], al
.text:10009155    shr     eax, 8
.text:10009158    mov     [esi+2], al
.text:1000915B    shr     eax, 8
.text:1000915E    mov     [esi+1], al
.text:10009161    add     esp, 0Ch
.text:10009164    sub     edi, 1
.text:10009167    sub     esi, 1
.text:1000916A    sub     ebx, 1
.text:1000916D    jnz     loc_100090E0
```

注意！这里不是直接用 `RC5` 解密，而是循环从输入字符串的最后一个分组开始，然后逐步往前移动一个字节，再解密，一共循环 `(输入字符串长度-8)` 次。相应的，加密过程就是从第一个分组开始，循环 `(输入字符串长度-8)` 次调用 `RC5` 加密算法。

### 解密与加密逻辑

解密过程：

```
for(i=sizeof(input)-8; i>=0; i--)
{
    A=makedword(input[i+3],input[i+2],input[i+1],input[i]);
    B=makedword(input[i+7],input[i+6],input[i+5],input[i+4]);
    RC5 解密(A, B);
    把 A，B 字节颠倒，然后放回原来的数组;
}
```

相应地，加密过程就是：

```
for(i=0; i<=sizeof(input)-8; i++)
{
    A=makedword(input[i+3],input[i+2],input[i+1],input[i]);
    B=makedword(input[i+7],input[i+6],input[i+5],input[i+4]);
    RC5 加密(A, B);
    把 A，B 字节颠倒，然后放回原来的数组;
}
```

例如，

`E3 5C 99 54 1D D2 2D 32 41 D1 24 EF 59 19 15 45 35 30 71 82 20 54 1A E5 2F B7 A4 03 55 86 EC 07 E1 16 48 D7 E4 2B 5E AE 4F DB E6 93 1C 29 6C 3A 6E 84 39 11 3C 21 BC A5 32 EC 30 05 AA C1 E2 14 A9 62`

解密之后就变成：

`42 09 E2 2D 33 00 FF FE 11 0C 85 31 D0 95 2D 8D 73 E1 19 4E 95 B5 F1 9D 6F 9D F7 C2 21 90 A6 3A 12 A5 B1 AE 7C 23 29 D2 B6 BE 33 AD F3 BE F8 44 32 14 C7 42 54 B6 35 CF 84 65 3A 56 D7 C6 75 BE 77 DF`

用 `RC5` 解密完之后来到这里：

### 汇编分析：版本校验

```
.text:10007FA7    push    edx
.text:10007FA8    push    esi
.text:10007FA9    call    sub_10007A40
```

按 `F7` 跟进：

```
.text:10007A40    sub     esp, 0Ch
.text:10007A43    push    esi
.text:10007A44    push    edi
.text:10007A45    mov     edi, [esp+14h+arg_4]
.text:10007A49    push    edi
.text:10007A4A    call    sub_10008D50     ; 取解密后字串的前两个字节
.text:10007A4F    movzx   eax, ax
.text:10007A52    mov     esi, [esp+18h+arg_0]
.text:10007A56    movzx   ecx, ax
.text:10007A59    mov     [esp+18h+var_C], eax
.text:10007A5D    mov     edx, ecx
.text:10007A5F    and     edx, 0FFh
.text:10007A65    mov     eax, 55555556h
.text:10007A6A    imul    edx
.text:10007A6C    mov     eax, edx
.text:10007A6E    shr     eax, 1Fh
.text:10007A71    lea     eax, [edx+eax+0Bh]
.text:10007A75    add     esp, 4
.text:10007A78    cmp     eax, 0Eh         ; 这个就是版本号啦，版本号小于 0x0E 就失败啦
.text:10007A7B    mov     [esi+8], eax
.text:10007A7E    jge     short loc_10007A8B
```

看看版本号是怎么来的：

解密后字串的第 2 个字节 × `0x55555556` + `0x0B` >= `0x0E`

一看就知道是编译器优化过的代码。原始代码是这样的：

解密后字串的第 2 个字节 × 5 / `0x0F` + `0x0B` >= `0x0E`

解这个不等式，得到：

解密后字串的第 2 个字节 >= `0x09`

我的解密字符串是：

`42 09 E2 2D ……`

第 2 个字节是 `0x09`，满足这个条件的。

接下来，这个 `call` 里面还验证了好多东西。由于时间原因，我没有仔细跟踪算法，所以现在暂时还不能写出注册机。

不过，有一个巧妙的方法可以获得 `PLP`：这里我用了个投机取巧的方法，我借用了 `Matlab2007a` 的 `PLP`，幸运地通过了这里的所有验证。



这样，`mwinstall.DecipherPLP`这个函数就返回`TRUE`了。

问题是，`Matlab2007a`的`PLP`无法用在`Matlab2007b`上面，原因在于`mwinstall.getCDVolNbr()`函数，检测输入`PLP`的版本。版本不对的话，也不能安装。

下面看看`C:\com\mathworks\installer\util.class`里面的`getPasscodeVersionNumber()`函数：

```java
static int getPasscodeVersionNumber()
{
    return Integer.parseInt("18");
}
```

原来是返回一个固定的常数`18`。只要版本号大于`18`就可以了。

那么，只要解这个不等式就可以了：

```
5x/0x0F+0x0B>=0x18
```

解得

```
x >= 0x15
```

这就是版本号的来源。

这样，就顺利通过了所有验证。

顺便说一下获得`PLP`的思路：

旧版`PLP` → 循环`RC5`解密 → 修改版本号 → 循环`RC5`加密 → 新版`PLP`

举例：

**旧版PLP：**

```
14-58204-39252-07634-11570-16849-09455-22809-05445-13616-29058-08276-06885-12215-41987-21894-60423-57622-18647-58411-24238-20443-59027-07209-27706-28292-14609-15393-48293-13036-12293-43713-57876-43362
```

↓

转换成`16`进制：

```
E3 5C 99 54 1D D2 2D 32 41 D1 24 EF 59 19 15 45 35 30 71 82 20 54 1A E5 2F B7 A4 03 55 86 EC 07 E1 16 48 D7 E4 2B 5E AE 4F DB E6 93 1C 29 6C 3A 6E 84 39 11 3C 21 BC A5 32 EC 30 05 AA C1 E2 14 A9 62
```

↓

循环`RC5`解密之后：

```
42 09 E2 2D 33 00 FF FE 11 0C 85 31 D0 95 2D 8D 73 E1 19 4E 95 B5 F1 9D 6F 9D F7 C2 21 90 A6 3A 12 A5 B1 AE 7C 23 29 D2 B6 BE 33 AD F3 BE F8 44 32 14 C7 42 54 B6 35 CF 84 65 3A 56 D7 C6 75 BE 77 DF
```

↓

修改版本号：

```
42 15 E2 2D 33 00 FF FE 11 0C 85 31 D0 95 2D 8D 73 E1 19 4E 95 B5 F1 9D 6F 9D F7 C2 21 90 A6 3A 12 A5 B1 AE 7C 23 29 D2 B6 BE 33 AD F3 BE F8 44 32 14 C7 42 54 B6 35 CF 84 65 3A 56 D7 C6 75 BE 77 DF
```

↓

循环`RC5`加密之后：

```
AE 29 97 73 A0 6F E3 3D D6 EA DF 55 12 DD 48 F1 0C A7 8C AB F0 EA C9 EB B5 ED 0B EE B5 31 CB 75 1B 0E 5A F4 42 1D B7 56 05 FA 4C F0 6B B8 66 C1 32 C8 C9 31 C3 24 96 59 DC 3A B3 1D 79 97 48 1D 26 57
```

↓

**新版PLP：**

```
15-44585-38771-41071-58173-55018-57173-04829-18673-03239-36011-61674-51691-46573-03054-46385-52085-06926-23284-16925-46934-01530-19696-27576-26305-13000-51505-49956-38489-56378-45853-31127-18461-09815
```

这样，借用了旧版`PLP`，就不用细细跟踪`PLP`的验证过程了。这样生成的`PLP`可以顺利通过验证。

**后记：** 由于时间原因，没有仔细跟踪`RC5`解密以后的算法，破解过程也没有详细写。见谅！

其实`RC5`我也是现学的。跟`Rijndael`什么的比起来，`RC5`算是个比较简单的算法了。共享一点`RC5/RC6`的资料，大家共同学习。高手请飘过。呵呵。

**【版权声明】** 本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢！
