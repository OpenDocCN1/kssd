
# 电子日记本（Ediary V2.53）口令验证算法分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-75.htm`](https://www.pediy.com/kssd/pediy09/pediy09-75.htm)

## 摘要
经过历时一天半的详细的跟踪调试和分析之后，本文向读者全面的揭示了电子日记本 (V2.53)的口令验证算法，并分析了其原理，并给出了一个口令穷举破解程序。经分析我们得知，该验证算法是建立在 `SHS` 杂凑算法（`SHA` 算法的小变形）和 `Blowfish` 加密算法的综合利用上的，强度比较高。但是由于 `Blowfish` 算法密钥的暴露，整个口令验证算法的强度等同于 `SHS` 算法的强度，即使是这样，由于 `SHS` 算法有 160 比特的杂凑结果，还是相当难以破解的。根据本文的分析，我们还具体实现了一个方便实用的口令破解程序，破解采用字典攻击方式，字典由外部提供，字典格式简单，是由回车分割的字符串形式，我们还提供了全部的源代码，大家可以参照我们的程序写出更为高效的破解实用程序，比如核心算法的高效汇编实现。

## 1. 引言
随着个人 `PC` 和网络的普及，越来越多的人都将电脑融入了自己的生活，很多资料都习惯保存在电脑上，很多热爱写日记的人也把日记保存在电脑之中，这样日后阅读，整理和打印都比较方便。各式各样的电子日记本软件也就应运而生了。由于日记里面可能会涉及个人的生活或者工作方面的隐私，因此电子日记的保密性就是一个很重要的方面。出于好奇，也是为了考察这类软件的日记保密功能强度，我们选择了网上下载量较大，使用比较广泛的“电子日记本（V2.53）”来做研究，详细分析了其口令验证算法，发现由于高强度的 `HASH` 函数的使用，以及 `Blowfish` 加密算法的辅助使用，整个口令验证算法强度还是不错的。另外我们并没有对带“问题提示”的口令恢复算法做研究，我们估计是用问题的答案来作为密钥把原始口令加密后保存；验证时，利用恢复后的密钥的 `HASH` 值作为核对结果，这样如果想要通过问题提示来作为突破口也是占不到便宜的。下面我们就顺着口令验证的步骤来讲述整个算法的细节。由于我们主要是讲解算法，所以对于程序是如何被脱壳，如何反汇编，如何调试分析，并最终分析找到核心函数代码部分等，我们在这里就不细述了。

## 2. 算法分析
首先我要向大家交代一下程序中使用的密码算法，它们是 `Blowfish` 分组加密算法和 `SHS`（`Secure Hash Standard`）杂凑算法，关于这两个算法的具体细节我就不介绍了，大家可以查阅相关资料去了解，由于都是著名的公开算法，这方面的资料还是很多的。

我们以用户 `drcool`,日记文件 `1.edf` 为例子来说明整个验证过程，我们假定他的口令是 `studygood`。当用户输入口令之后，验证程序就开始了：

1.  **读入密文，得到解密密钥**
    程序从 `1.edf` 的文件偏移 `957H` 处开始读入 341 字节到内存，其中前 24 字节对我们的研究是有重要意义的，我们在下面会细说的。然后就对这 341 字节进行 `Blowfish` 解密，而解密算法用的密钥是从字符串“`Blowfish`”生成的，这个字符串是以明文形式保存在日记本文件中的。我也是由于看到了这个熟悉的单词才猜想它是用了 `Blowfish` 加密算法的，后面的分析也证实了我的猜想。使用的密钥是：
    ```
    0xf9,0x6c,0xea,0x19,0x8a,0xd1,0xdd,0x56,0x17,0xac,0x08,0x4a,0x3d,0x92,0xc6,0x10
    ```
    具体是如何变换来的，我并没有去深究，毕竟它并不影响我们的分析。

2.  **得到算法用的 IV 并进行反馈模式的解密**
    利用上面得到的密钥对 8 字节的全 `0xff` 进行加密，并把加密结果 4 字节反序(也就是 `0x12345678` 变成 `0x78563412`)做为初始 `IV`，这在后面的带反馈模式的解密中会用到的.下面就是解密了，解密过程用如下伪代码表示：
    ```
    1)  IV’<-IV^Cipher
    2)  Minwen<-Blowfish_Dec(Cipher)
    3)  Minwen<-Minwen^IV
    4)  在下一轮解密中 IV 和 IV’互换角色
    ```
    另外在解密函数的出口和入口处，都要对参数进行 4 字节反序操作。经过三轮的解密，我们得到了全部的 24 字节（程序实际是解密了所有的读入数据，但是对我们有用的就这 24 字节）。它们是：`.KVF1T8VelfAo1O03vEcqQ==`。第一个字符“.”其实并不是“.”，而是用来代替无法显示的 `0x00` 的，大家用 `UltralEdit` 打开文件并用 16 进制阅读是就会发现对无法打印的字符是这样处理的。这是什么东西？看到后面两个等号了吗？还记得 `BASE64` 编码算法吗？呵呵，看到此处是不是豁然开朗了。对了就是经过 `BASE64` 编码后的东东。但是细心的读者马上就发现了问题，`BASE64` 编码后的字符串长度都是 4 的倍数而且每个字符串都是可读的，这样除去第一个 `0x00`，只有 23 个字节啊，这是怎么回事呢？是的我开始时也是有这个疑问的，其实这都要怪程序作者在 `BASE64` 编码程序编写上的一个错误，这我在后面会具体谈到，先暂且放在一边。

3.  **核对口令**
    下面程序就把我们输入的 `studygood` 进行 `SHS` 杂凑，得到结果：
    ```
    2951754FC55E95F480A353B4DEF11CA91375A301
    ```
    共 20 字节，但是程序只是取用了前 16 字节来用。接下来把前 16 字节进行 `BASE64` 编码得到：
    `KVF1T8VelfAo1O03vEcqQ==`，这和我们上面解密后得到的 `BASE64` 编码是一致的，这样程序就认定口令正确了。由于 `SHS` 杂凑函数的密码学特性以极大的概率保证了不同的口令有不同的杂凑值，所以这种验证方法是可以接收的。我们注意到算法只是选取了 20 字节杂凑结果的前 16 字节，这不能不说有些遗憾，这样做无疑加大了杂凑碰撞产生的可能性，虽然这种碰撞不能给解密日记文件带来方便，但是却给程序的处理带来了麻烦，程序将如何面对一个通过验证的错误口令呢？报告日记文件被毁坏需要从备份文件修复吗？这个程序确实是这样做的，因为它对日记文件内容有做 `Checksum` 的。但这无疑是将使程序产生病态的反应。

好了整个验证算法就介绍完了，下面我就要具体来说说他的 `BASE64` 编码实现的错误之处以及对我们的破解产生的恶劣影响。

`BASE64` 编码是在邮件传输中用到的编码算法，它是网络上最常见的用于传输 `8Bit` 字节代码的编码方式之一，大家可以查看 `RFC2045`～`RFC2049`，上面有 `MIME` 的详细规范。`Base64` 要求把每三个 `8Bit` 的字节转换为四个 `6Bit` 的字节（`3*8 = 4*6 = 24`），然后把 `6Bit` 再添两位高位 0，组成四个 `8Bit` 的字节，也就是说，转换后的字符串理论上将要比原来的长 1/3。

这样说会不会太抽象了？不怕，我们来看一个例子：

```
转换前  aaaaaabb  ccccdddd  eeffffff
转换后  00aaaaaa  00bbcccc  00ddddee  00ffffff
```

应该很清楚了吧？上面的三个字节是原文，下面的四个字节是转换后的 `Base64` 编码，其前两位均为 0。转换后，我们用一个码表来得到我们想要的字符串（也就是最终的 `Base64` 编码），这个表是这样的：（摘自 `RFC2045`）

```
    Table 1: The Base64 Alphabet

  Value Encoding  Value Encoding  Value Encoding  Value Encoding
      0 A           17 R           34 i           51 z
      1 B           18 S           35 j           52 0
      2 C           19 T           36 k           53 1
      3 D           20 U           37 l           54 2
      4 E           21 V           38 m           55 3
      5 F           22 W           39 n           56 4
      6 G           23 X           40 o           57 5
      7 H           24 Y           41 p           58 6
      8 I           25 Z           42 q           59 7
      9 J           26 a           43 r           60 8
     10 K           27 b           44 s           61 9
     11 L           28 c           45 t           62 +
     12 M           29 d           46 u           63 /
     13 N           30 e           47 v
     14 O           31 f           48 w   (pad) =
     15 P           32 g           49 x
     16 Q           33 h           50 y
```

让我们再来看一个实际的例子，加深印象！

```
转换前  10101101  10111010  01110110
转换后  00101011  00011011  00101001  00110110
十进制  43  27  41  54
对应码表中的值  r  b  p  2
```

所以上面的 24 位编码，编码后的 `Base64` 值为 `rbp2`。

解码同理，把 `rbq2` 的二进制位连接上再重组得到三个 8 位值，得出原码。

（解码只是编码的逆过程，在此我就不多说了，另外有关 `MIME` 的 `RFC` 还是有很多的，如果需要详细情况请自行查找。）

用更接近于编程的思维来说，编码的过程是这样的：

1.  第一个字符通过右移 2 位获得第一个目标字符的 `Base64` 表位置，根据这个数值取到表上相应的字符，就是第一个目标字符。
2.  然后将第一个字符左移 4 位加上第二个字符右移 4 位，即获得第二个目标字符。
3.  再将第二个字符左移 2 位加上第三个字符右移 6 位，获得第三个目标字符。
4.  最后取第三个字符的右 6 位即获得第四个目标字符。

在以上的每一个步骤之后，再把结果与 `0x3F` 进行 `AND` 位操作，就可以得到编码后的字符了。

程序作者是怎么做的呢？假设我们对 `0XAABBCC` 进行 `BASE64` 编码，如果 `CC` 的低 6 位是 0，则本来要被编码输出的“A”将不会被输出，也就是说 `0XAABBCC` 将被编码为 3 个字节而不是 4 个字节。由于我们是没三个字节进行编码的，这样的情况出现几次，我们就要少几个字符的码字！当然也不排除其他少字节码的情况，在实际操作中确实也碰到过的。而这样做的直接恶果是我们无法从这种错误的编码恢复出原始的明文！据我所知不仅仅是程序作者范了这样的错误，可能是他们用的算法库就有这个错误，或者是为了给我们的破解设置障碍？我们不得而知。但是，虽然编码程序有这个问题，却不影响口令的验证过程，因为它对比的是 `BASE64` 编码过后的东东！大家都是通过同一个错误的程序编码得来得啊！（如果老祖宗把太阳叫成 `SUN`，那么我们今天不知道什么是太阳而只知道 `SUN`）

那么这对我们的破解带来什么麻烦呢？由于我们无法在这种情况下恢复出编码之前得明文，也就是 `HASH` 值得前 16 字节；我们就不得不对每个试验口令得 `HASH` 值做这种错误的 `BASE64` 编码来核对结果，这在时间上还是有损失的，尤其是我们这种穷举字典攻击。

## 3. 破解程序实现
好了，整个算法分析完了，下面就是要写一个破解口令的程序了，全是体力活儿，实现技巧也没有太多，简单介绍一下吧。为了简单起见，整个程序做成了一个命令行程序形式。从命令行得到 `edf` 日记文件和字典文件。字典的形式很简单，就是一行一个口令，回车符分割开来。大家可以用字典生成工具来生成，也可以自己编程实现。

程序内部的工作，就是穷举每一个字典口令来进行验证过程，直到找到口令或者穷举完毕而退出。有一点需要注意得就是，如果 `BASE64` 编码的结果没有出错，那么我们得运算量可以少一点，因为可以直接得到编码前明文；如果运行太差，比如遇到 `drcool` 这样得倒霉用户，就要麻烦一点，需要进行“错误的 `BASE64` 编码”之后进行核对验证。其他的就没有什么好说的了，更加具体的东西还是请看我提供的程序吧。长夜路慢慢，只有汇编代码在一路给我指明方向……

## 4. 总结
通过这次分析研究，我们发现电子日记本这类软件对于日记保护的工作做的还是比较令人满意的，毕竟我们所书写的并不是本拉登的藏身地点，而是今天张三借了李四半瓶酱油，明天王五的漂亮姑娘要出嫁，我的心中又少了一些淡淡的“牵挂”……之类的不咸不淡的凡人琐事……。但是对于大家认为有必要进行保密的东西，比如本拉登的藏身地，建议还是不要记录在这种保密功能相对欠缺的软件里面，或者使用相当长的密码来保护为好，毕竟 `SHS` 杂凑函数的强度还是很强的，再加上密码设定我们再强化一下，破解起来还是很困难的。

另外，通过这次分析研究，我又再一次复习了两种重要的密码算法，更加体会到了密码的重要性和熟悉密码算法的重要性。如果不是对各种密码算法有一定的了解，我的分析之路恐怕更是荆棘密布，步履蹒跚了！以后我们要更加熟悉并研究各种密码算法，这对于我日后的工作是有相当的指导意义的。

通过这次分析我还体会到一点，认真读反汇编代码，认真做算法研究笔记和调试过程笔记是一项基本功，断点不在多，有用则灵；熬夜不在长，值得就行。黑夜给了我黑色的眼睛，我却要用它来寻找光明……

## 相关代码
主程序：

```c
#include <stdio.h>
#include <string.h>
#include <memory.h>
#include "blowfish.h"
#include "shs.h"

//BASE64 的编码表，共 65 字节
char Table[66]="ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=";

//SHS 算法全局变量
SHS_INFO Digit;
int wrong_n;   //比正常的 BASE64 编码少几个字符的计数

//BASE64 反编码子程序
int B_search(char ch)
{
  int index = -1;
  if (ch >= 'A' && ch <= 'Z')
  {
    index = ch - 'A';
  }
  else if (ch >= 'a' && ch <= 'z')
  {
    index = ch - 'a' + 26;
  }
  else if (ch >= '0' && ch <= '9')
  {
    index = ch - '0' + 52;
  }
  else if (ch == '+')
  {
    index = 62;
  }
  else if (ch == '/')
  {
    index = 63;
  }

  return index;
}

//BASE64 解码子函数，4 字符变三字节
int B_de(unsigned char *A,unsigned char *B)
{
   char t;
   t=B_search(A[0]);
   if(t==-1)return -1;
   B[0]=t<<2;
   t=B_search(A[1]);
   if(t==-1)return -1;
   B[1]=t<<4;
   B[0]|=(t>>4);
   t=B_search(A[2]);
   if(t==-1) return -1;
   B[1]|=(t>>2);
   B[2]|=(t<<6);
   t=B_search(A[3]);
   if(t==-1) return -1;
   B[2]|=t;
   return 1;
}

//BASE64 编码的逆
```



## C++ Source Code: Base64 and Blowfish Implementation

The following code contains functions for handling Base64 encoding/decoding and Blowfish encryption/decryption, likely used in a cracking tool context.

```cpp
int Re_Base64(unsigned char *A, int len, unsigned char *out, int out_len)  
{  
    int i, j;  
    char t;  
    if(out_len < len * 3 / 4) return -1;  
    memset(out, 0, out_len);  
     for(i=0, j=0; i < len - 4; i += 4, j += 3)  
        B_de(A + i, out + j);  
    if(A[len - 1] != '=')  
    {  
        B_de(A + i, out + j);  
        return 1;  
    }  
    else  
    {  
        if(A[len - 2] == '=')  
        {  
            t = B_search(A[i]);  
            if(t == -1) return -1;  
            out[j] = t << 2;  
            t = B_search(A[i + 1]);  
            if(t == -1) return -1;  
            out[j] |= (t >> 4);  
        }  
        else  
        {  
            t = B_search(A[i]);  
            if(t == -1) return -1;  
            out[j] = t << 2;  
            t = B_search(A[i + 1]);  
            if(t == -1) return -1;  
            out[j + 1] = t << 4;  
            out[j] |= (t >> 4);  
            t = B_search(A[i + 2]);  
            if(t == -1) return -1;  
            out[j + 1] |= (t >> 2);  
            out[j + 2] |= (t << 6);  
        }  
    }  
    return 1;  
}  

// Base64 encoding algorithm with specific errors. Since it's a dedicated sub-function, it is not generalized, only targeting 16-byte Base64 encoding.
void Bad_Base64Enc(unsigned char *A, unsigned char *out)  
{  
    int i, j;  
   
    for(i = 0, j = 0; i < 15; i += 3)  
    {  
        out[j] = Table[A[i] >> 2];  
        out[j + 1] = Table[((A[i] << 4) | (A[i + 1] >> 4)) & 0x3f];  
        out[j + 2] = Table[((A[i + 1] << 2) | (A[i + 2] >> 6)) & 0x3f];  
        out[j + 3] = Table[A[i + 2] & 0x3f];  
        if(out[j + 3] == 'A') j += 3;  
        else j += 4;  
    }  
    out[j] = Table[A[i] >> 2];  
    out[j + 1] = Table[(A[i] << 4) & 0x3f];  
    if(out[j + 1] == 'A')  
    {  
        out[j + 1] = '=';  
        out[j + 2] = '=';  
        return;  
    }  
    out[j + 2] = '=';  
    out[j + 3] = '=';  
    return;  
}  

void Usage()  
{  
    printf("Usage: Ediary_Crack  diaryfile  dictionaryfile");  
}  

int Blow_decry(unsigned long *AA)  
{  
    unsigned long L = AA[0], R = AA[1];  
    unsigned long L1 = AA[2], R1 = AA[3];  
    unsigned long L2 = AA[4], R2 = AA[5];  
    unsigned long s0 = 0xffffffff, s1 = 0xffffffff;  
    unsigned long s2, s3;  
    BLOWFISH_CTX ctx;  
    int i;  
    unsigned char *p;  
    unsigned char Key[16] = {0xf9, 0x6c, 0xea, 0x19, 0x8a, 0xd1, 0xdd, 0x56, 0x17, 0xac, 0x08, 0x4a, 0x3d, 0x92, 0xc6, 0x10};  
    Blowfish_Init(&ctx, Key, 16);  
    // Encrypt 8 bytes of all f as IV, encryption mode is feedback mode
    Blowfish_Encrypt(&ctx, &s0, &s1);  
    _asm  
    {  
        mov eax, s0  
        bswap eax  
        mov s0, eax  
        mov eax, s1  
        bswap eax  
        mov s1, eax  
    }  
    s2 = L ^ s0; s3 = R ^ s1;  
    _asm  
    {  
        mov eax, L  
        bswap eax  
        mov L, eax  
        mov eax, R  
        bswap eax  
        mov R, eax  
    }  
    Blowfish_Decrypt(&ctx, &L, &R);  
    _asm  
    {  
        mov eax, L  
        bswap eax  
        mov L, eax  
        mov eax, R  
        bswap eax  
        mov R, eax  
    }  
    L ^= s0; R ^= s1;  
   

   

    s0 = L1 ^ s2; s1 = R1 ^ s3;  
    _asm  
    {  
        mov eax, L1  
        bswap eax  
        mov L1, eax  
        mov eax, R1  
        bswap eax  
        mov R1, eax  
    }  
    Blowfish_Decrypt(&ctx, &L1, &R1);  
    _asm  
    {  
        mov eax, L1  
        bswap eax  
        mov L1, eax  
        mov eax, R1  
        bswap eax  
        mov R1, eax  
    }  
    L1 ^= s2; R1 ^= s3;  
   

   

    s2 = L2 ^ s0; s3 = R2 ^ s1;  
    _asm  
    {  
        mov eax, L2  
        bswap eax  
        mov L2, eax  
        mov eax, R2  
        bswap eax  
        mov R2, eax  
    }  
    Blowfish_Decrypt(&ctx, &L2, &R2);  
    _asm  
    {  
        mov eax, L2  
        bswap eax  
        mov L2, eax  
        mov eax, R2  
        bswap eax  
        mov R2, eax  
    }  
    L2 ^= s0; R2 ^= s1;  

    AA[0] = L; AA[1] = R;  
    AA[2] = L1; AA[3] = R1;  
    AA[4] = L2; AA[5] = R2;  
    p = (unsigned char*)AA;  
    for(i = 0; i < 24; i++)  
        if(p[i] != 0)  
            break;  
    return i;  
}  

int Try_Check(unsigned char *try_key, unsigned char *De_base64, int len)  
{  
    unsigned char *p;  
    shsInit(&Digit);  
    shsUpdate(&Digit, try_key, len);  
    shsFinal(&Digit);  
    p = Digit.digest;  
    _asm  
    {  
        mov ebx, p  
        mov eax, dword ptr [ebx]  
        bswap eax  
        mov [ebx], eax  
        mov eax, dword ptr [ebx+4]  
        bswap eax  
        mov [ebx+4], eax  
        mov eax, dword ptr [ebx+8]  
        bswap eax  
        mov [ebx+8], eax  
        mov eax, dword ptr [ebx+12]  
        bswap eax  
        mov [ebx+12], eax  
    }  
    if(!memcmp(Digit.digest, De_base64, 16))  
        return 1;  
    return 0;  
}  

int Try_Check_1(unsigned char *try_key, unsigned char *data2, int len)  
{  
    unsigned char *p;  
   
    unsigned char b_out[32];  
    int i, t;  
    shsInit(&Digit);  
    shsUpdate(&Digit, try_key, len);  
    shsFinal(&Digit);  
    p = Digit.digest;  
   
    _asm  
    {  
        mov ebx, p  
        mov eax, dword ptr [ebx]  
        bswap eax  
        mov [ebx], eax  
        mov eax, dword ptr [ebx+4]  
        bswap eax  
        mov [ebx+4], eax  
        mov eax, dword ptr [ebx+8]  
        bswap eax  
        mov [ebx+8], eax  
        mov eax, dword ptr [ebx+12]  
        bswap eax  
        mov [ebx+12], eax  
    }  

    Bad_Base64Enc(p, b_out);  
    if(!memcmp(b_out, data2, 24 - wrong_n)) return 1;  
    return 0;  
}  

void main(int argc, char **argv)  
{  
    FILE *pe, *pd;  
    int i;  
    unsigned char data1[24];  
    unsigned char try_key[100];  
    unsigned char De_base64[20];  
    int key_len;  
    if(argc != 3)  
    {  
        Usage();  
        return;  
    }  
    pe = fopen(argv[1], "rb");  
    pd = fopen(argv[2], "rb");  
    if((!pe) || (!pd))  
    {  
        printf("error in reading files.");  
        return;  
    }  
    fseek(pe, 0x957, SEEK_SET);  
    fread(data1, 24, 1, pe);  
    fclose(pe);  
    wrong_n = Blow_decry(data1);  
    if(!wrong_n)  
    {  
        Re_Base64(data1, 24, De_base64, 20);  
        while(fgets(try_key, 100, pd))  
        {  
            for(i = 0, key_len = 0; ; i++, key_len++)  
            {  
                if(try_key[i] == 0x0d)  
                    break;  
            }  
            if(Try_Check(try_key, De_base64, key_len))  
            {  
                printf("the key is:%s\n", try_key);  
                fclose(pd);  
                return;  
            }  
        }  
        printf("sorry,we can't find the key. Try another dictionary!");  
        fclose(pd);  
        return;  
    }  
    else  
    {  
        while(fgets(try_key, 100, pd))  
        {  
            for(i = 0, key_len = 0; ; i++, key_len++)  
            {  
                if(try_key[i] == 0x0d)  
                    break;  
            }  
            if(Try_Check_1(try_key, data1 + wrong_n, key_len))  
            {  
                printf("the key is:%s\n", try_key);  
                fclose(pd);  
                return;  
            }  
        }  
        printf("sorry,we can't find the key. Try another dictionary!");  
        fclose(pd);  
        return;  
    }  
}
```

SHS and Blowfish related algorithms are not included here.

`4dd466d0e1e0cc123afbe504da2462ca`
