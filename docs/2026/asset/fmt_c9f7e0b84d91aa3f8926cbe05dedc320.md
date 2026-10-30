
到这里再思考下，现在不管`flag`什么条件下为0，有一点可以肯定，正确流程应该是`flag`先为0，开启另一个进程启动窗体，非窗体进程就是窗体进程的父进程，父进程对子进程进行调试，修改流程。但是由于子进程不能附加，所以是看不了代码的。下一步要做的就是：掌握父进程对子进程的修改或想办法搞到子进程的代码。

考虑到跟踪父进程太累，先使用`dump`的办法。用`petools`对刚运行的程序进行`dump`，没进行任何修复改动，直接进`ida`，发现本来`401740`中是这样的：

``` 
String = 0;

memset(&v7, 0, 0x1FCu);

GetDlgItemTextW(hDlg, 1001, &String, 510);

v4 = sub_401A40(&String, 0x451E8B10u / dword_437A08);
```

后来变成了这样：

``` 
sub_418360(&v8, 0, 508);

v745FB940(a2, 1001, &v7, 510, a1, *(_DWORD *)&v7, v9, v10);

v5 = sub_401A40(&v7, 0x451E8B10 / 0u);
```

`dword_437A08`已经为0。 还是要看父进程，除0异常应该被调试者接收吧。

还有一个办法，如果父进程对子进程进行了更改，然后并未还原呢？抱着试试看的想法，我随便输入并确定，提示错误，然后再`dump`。果然不一样了。这里提醒一下，第二次`dump`出来的文件的PE头的`dosheader`已经被修改了，需要还原下。下面对比下原始与`dump`出来的变化情况。

为了方便看，我直接上汇编了，开始是这样的：

``` 
.text:004017B2                 mov     eax, 451E8B10h 
.text:004017B7                 mov     ecx, dword_437A08
.text:004017BD                 div     ecx
.text:004017BF                 mov     [ebp+hDlg], eax
.text:004017C2                 push    [ebp+hDlg]
.text:004017C5                 lea     eax, [ebp+String]
.text:004017CB                 push    eax
.text:004017CC                 call    sub_401A40
.text:00401A40                 push    ebp
.text:00401A41                 mov     ebp, esp
.text:00401A43                 sub     esp, 0Ch
.text:00401A46                 push    ebx
.text:00401A47                 push    esi
.text:00401A48                 push    edi
.text:00401A49                 mov     edi, [ebp+arg_0]
.text:00401A4C                 xor     edx, edx
.text:00401A4E                 xor     esi, esi
.text:00401A50                 mov     [ebp+var_8], edx
.text:00401A53                 mov     [ebp+var_4], esi
.text:00401A56                 xor     ecx, ecx
.text:00401A58                 lea     ebx, [edi+2]
.text:00401A5B                 jmp     short loc_401A60
```

第一次dump后是这样的:

``` 
.text:004017B2                 mov     eax, 451E8B10h
.text:004017B7                 mov     ecx, ds:dword_437A08
.text:004017BD                 div     ecx
.text:004017BF                 mov     [ebp+arg_0], eax
.text:004017C2                 push    [ebp+arg_0]
.text:004017C5                 lea     eax, [ebp+var_600]
.text:004017CB                 push    eax
.text:004017CC                 call    sub_401A40
.text:00401A40                 push    ebp
.text:00401A41                 mov     ebp, esp
.text:00401A43                 sub     esp, 0Ch
.text:00401A46                 push    ebx
.text:00401A47                 push    esi
.text:00401A48                 push    edi
.text:00401A49                 mov     edi, [ebp+arg_0]
.text:00401A4C                 xor     edx, edx
.text:00401A4E                 xor     esi, esi
.text:00401A50                 mov     [ebp+var_8], edx
.text:00401A53                 mov     [ebp+var_4], esi
.text:00401A56                 xor     ecx, ecx
.text:00401A58                 lea     ebx, [edi+2]
.text:00401A5B                 jmp     short loc_401A60
```

唯一的区别就是`dword_437A08`变成了0。第二次dump是这样的：

``` 
.text:004017B2                 mov     eax, 451E8B10h
.text:004017B7                 mov     ecx, ds:dword_437A08
.text:004017BD                 nop
.text:004017BE                 nop
.text:004017BF                 mov     [ebp+arg_0], eax
.text:004017C2                 push    [ebp+arg_0]
.text:004017C5                 lea     eax, [ebp+var_600]
.text:004017CB                 push    eax
.text:004017CC                 call    sub_401A40
.text:00401A40                 push    ebp
.text:00401A41                 mov     ebp, esp
.text:00401A43                 jmp     loc_402303
.text:00402300                 push    ebp
.text:00402301                 mov     ebp, esp
.text:00402303
.text:00402303 loc_402303:                             ; CODE XREF: sub_401A40+3j
.text:00402303                 sub     esp, 424h
.text:00402309                 mov     ecx, ds:dword_43AAFC
.text:0040230F                 mov     [ebp+var_80], 0
.text:00402316                 mov     [ebp+var_34], 0C9DDE6h
.text:0040231D                 mov     [ebp+var_3C], 19170839h
.text:00402324                 mov     [ebp+var_38], 0E1C01506h
.text:0040232B                 mov     eax, [ecx+3Ch]
.text:0040232E                 add     eax, 78h
.text:00402331                 mov     [ebp+var_84], 0
.text:0040233B                 add     eax, ecx
```

这一次变化比较大，除操作已经被`nop`掉了，`401A40`调用实际上已经被改成了`402300`调用。其它地方有没有更改不太清楚，但是更改的可能性比较小，改动开始没有显式调用的函数反倒可能会显露目标，而且这种手法明显只是为了隐藏真实的检验流程，目标已经达到（另外，我当时也简单对比下`402300`，没看出什么改动）。暂定先尝试手动完成父进程完成的工作，看程序运行情况，及检验流程情况。 直接改成这样：

``` 
004017BD      90            nop
004017BE      90            nop
004017BF  |.  8945 08       mov [arg.1],eax
004017C2  |.  FF75 08       push [arg.1]
004017C5  |.  8D85 00FAFFFF lea eax,[local.384]
004017CB  |.  50            push eax
004017CC      E8 2F0B0000   call 11-ToBeB.00402300
```

## 算法

动态前先看下静态的情况，函数过程比较复杂，代码量比较大。我们从输入输出入手，函数参数只有一个输入的字串，返回是`True`则成功，函数对参数及返回读写的区域在`4029D5`到`402B34`之间。所以这部分是重点跟的。

动态跟了下这个函数开头，此处利用开始壳读取并修改的文件内容(从这里可以看出，开始从堆中`dump`出来的程序是不能完成验证的)，加载库及其它操作。直接跳过，在下断`4029D5`。

这里开始的算法比较简单。就是用输入前8位作为一个hex数`a1`，9-10位为一个hex数`a2`。对一堆区0x5000 byte的数据进行运算，最后与`byte_4340B0`处的数据比较前0x60个字节。运算部分的代码翻译过来如下：

``` 
int j = 0
for (int i = a1; i < a1 +0x5000/4;i++)
{
    output[j] = (a2*0x1010101+i)^output[j];
    j++;
}
```

反解，得到前10字节，`0x75A29C09E1`，注意，在`unhex`的函数中均将输入以大写hex进行转换，反解代码如下：

``` 
m1 = 0x83F08EA7
m2 = 0x3F0FBA29
c1 = 0x1070EC81
c2 = 0x55530000
d = 0x1010101
for i in range(0x100):
    if ((i*d+m1)^c1)&0xffffffff == (((i*d+m2)^c2)&0xffffffff)-1:
        print hex(i)        
        print hex(((i*d+m1)^c1)&0xffffffff)
```

后面`memset`两个栈区后，将第11位开始的输入`unhex`（也要求输入是大写hex）。接着读入`!HelloHaniella!`常量，并将硬编码的`19970907`写入栈区，并调用前面刚运算完的堆区，原来上面是为后面解码代码的。

进去看了看，算法比较明显，DES。原因有2：一是`19970907`应该是作为密钥的，进去后立即被变换成56bit；二是再看下变换表，就是DES的表。

后面还有IP变换表，S-BOX变换表，移位表，P变换表等，我一一比对，没有改动。这里有个细节有注意了下，轮密钥是从后面开始用的，说明这是解密过程。下面的代码就是轮密钥与扩展密文的异或过程，轮密钥从后往前用的：

``` 
0068101A    8BCA            mov ecx,edx
0068101C    8DB424 300C0000 lea esi,dword ptr ss:[esp+0xC30]
00681023    8DBC24 900C0000 lea edi,dword ptr ss:[esp+0xC90]
0068102A    F3:A5           rep movs dword ptr es:[edi],dword ptr ds>
0068102C    33F6            xor esi,esi
0068102E    8B7C24 14       mov edi,dword ptr ss:[esp+0x14]
00681032    8BCE            mov ecx,esi
00681034    2BCF            sub ecx,edi
00681036    8A8C0C 50100000 mov cl,byte ptr ss:[esp+ecx+0x1050]
0068103D    308C34 900C0000 xor byte ptr ss:[esp+esi+0xC90],cl
00681044    46              inc esi
00681045    83FE 30         cmp esi,0x30
00681048  ^ 7C E4           jl short 0068102E
```

DES加密完成后，与`!HelloHaniella!`按byte比较，直到遇到0，结束比较，0也进行了比较。

``` 
00402AF2  |> /8A10          /mov dl,byte ptr ds:[eax]   ;加密串
00402AF4  |. |3A11          |cmp dl,byte ptr ds:[ecx]   ;常量
00402AF6  |. |75 30         |jnz short 11-ToBeB.00402B28
00402AF8  |. |84D2          |test dl,dl
00402AFA  |. |74 12         |je short 11-ToBeB.00402B0E
00402AFC  |. |8A50 01       |mov dl,byte ptr ds:[eax+0x1]
00402AFF  |. |3A51 01       |cmp dl,byte ptr ds:[ecx+0x1]
00402B02  |. |75 24         |jnz short 11-ToBeB.00402B28
00402B04  |. |83C0 02       |add eax,0x2
00402B07  |. |83C1 02       |add ecx,0x2
00402B0A  |. |84D2          |test dl,dl
00402B0C  |.^\75 E4         \jnz short 11-ToBeB.00402AF2
```

到这里，我直接将检验值进行des加密，结果竟然不对。此处我花了太多时间，重新对比变换表，及算法过程。确实没有发现问题。没办法，我直接将改待解密数据为检验值，将轮密钥前后倒置，最后看了结果与我算的还是不一样。因为此，我花了半天时间。

最后直接一步一步跟，并写了des，打出各个过程中值，进行比对。解密完成也没有发现不对。后面再跟，发现最后结果提取作了手脚，结果数组下标为偶数的进行了取反。我的天，这是玩心理学啊。其代码如下：
