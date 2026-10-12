
# 一道很难的有关算法的测试题，写逆算法

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-216.htm`](https://www.pediy.com/kssd/pediy09/pediy09-216.htm)

最近在学习压缩算法，刚学懂了一个压缩算法，把它加以改造，做成了一道测试题．

这道题很难，这个算法很巧．如果不知道答案，我都没有把握能做出来.

如果你自信算法方面很强，可以试一试．如果你做不出来，又想知道答案，联系我．

以下已经给出了完整的`Decode`代码，`Encode`代码有几处空，填入代码，使算法工作．

这个算法可以广泛地应用于注册机，网络传输等方面．

如果你做出来了，我愿高薪聘你![](img/7c79845954f5fbf9e981f570ed213a8d.jpg "扮鬼脸")

```c
#include <Windows.h>
#include <stdio.h>

class CBitEncoder
{
  PBYTE m_pbuf;
  UINT m_len;

  UINT Prob;
  UINT _cacheSize;
  BYTE _cache;

  UINT64 Low;
  UINT uiSom;

  void WriteByte(BYTE b)  {    m_pbuf[m_len++] = b;  }
  void Init()
  {
    Prob = (1 << 10);
    Low = 0;
    uiSom = 0xFFFFFFFF;
    _cacheSize = 0;
    _cache = 0;
  }
  void ShiftLow()
  {
    此处空13行代码;
  }
public:
  void SetOutputBufPtr(PBYTE buf)
  {
    Init();
    m_pbuf = buf;
    m_len = 0;
  }
  int GetLength()  {    return m_len;  }

  void FlushData()
  {
    此处空2行代码;
  }

  void bEncode(UINT symbol)
  {
    此处空17行代码;
  }
};

//--------------------以上是 Encoder 以下是 Decoder

class CBitDecoder
{
  PBYTE m_psrc;
  int m_srclen;

  UINT uiSom;
  UINT uiAny;
  UINT Prob;

  BYTE ReadByte()
  {
    m_srclen--;
    return *m_psrc++;
  }
  void Init()
  {
    Prob = (1 << 10);
    uiAny = 0;
    uiSom = 0xFFFFFFFF;
    for(int i = 0; i < 4; i++)
      uiAny = (uiAny << 8) | this->ReadByte();
  }
public:
  void SetSrcBuf(PBYTE psrc, int srclen)
  {
    m_psrc = psrc;
    m_srclen = srclen;
    Init();
  }
  int GetRemainByte()  {    return m_srclen;  }

  bool bDecode()
  {
    bool b;
    UINT u = (this->uiSom >> 11) * this->Prob;
    if (this->uiAny < u)
    {
      this->uiSom = u;
      this->Prob += ((1 << 11) - this->Prob) >> 5;
      b = false;
    }
    else
    {
      this->uiSom -= u;
      this->uiAny -= u;
      this->Prob -= (this->Prob) >> 5;
      b = true;
    }
    if (this->uiSom < (1 << 24))
    {
      this->uiAny = (this->uiAny << 8) | this->ReadByte();
      this->uiSom <<= 8;
    }
    return b;
  }
};

int Test_Encode(PBYTE psrc, int srclen, PBYTE buf)
{
  CBitEncoder bitEncoder;
  bitEncoder.SetOutputBufPtr(buf);

  for (int i=0;i<srclen;i++)
  {
    BYTE b = psrc[i];
    for (int j=0;j<8;j++)
    {
      bitEncoder.bEncode((b>>j) & 1);
    }
  }
  bitEncoder.FlushData();

  return bitEncoder.GetLength();
}

int Test_Decode(PBYTE psrc, int srclen, PBYTE buf)
{
  CBitDecoder bitDecoder;
  bitDecoder.SetSrcBuf(psrc, srclen);

  int totallen = 0;
  for (;;)
  {
    BYTE b = 0;
    for (int i=0;i<8;i++)
    {
      b += (bitDecoder.bDecode() << i);
      if (bitDecoder.GetRemainByte() < 0)
        return totallen;
    }
    *buf++ = b;
    totallen++;
  }
  return totallen;
}

void main()
{
  char* psrc = "exam by LiuTaoTao 20070728";
  int srclen = strlen(psrc);
  BYTE buf[3000];
  int len = Test_Encode((PBYTE)psrc, srclen, buf);
  printf("Srclen = %d Encode len = %d \n", srclen, len);

  BYTE outbuf[3000];
  int len2 = Test_Decode(buf, len, outbuf);
  printf("Decode len from %d to %d \n", len, len2);
  if (len2 >= srclen && !memcmp(psrc, outbuf, srclen))
    printf("OK\n");
  else
    printf("Error\n");
}
// 这种压缩算法，代码简单，速度快．最差的情况，是增长 4 个字节
// 还有一个缺点,就是不能很有效地判断是否已经解码结束,有时候会多解几个字节出来
```

压缩算法一点没研究过，所以先顺便找一篇压缩的文章学习一下基本原理：《聪明的以色列人(上)：LZ77》

虽然和本题关系不大，也算概念入门了

看看谁能把下面的内嵌汇编转成 C 语言的：



```
int Test_Encode(PBYTE psrc, int srclen, PBYTE buf)
{
        BYTE    _cache;
        DWORD   Low_l;
        DWORD   Low_h;
        DWORD   uiSom;
        int      i;
        int      j;
        BYTE    b;

        __asm
        {
            xor     edi, edi
            xor     edx, edx
            xor     esi, esi
            mov     ebx, 400h
            mov     Low_l, edx
            mov     Low_h, edx
            mov     uiSom, 0FFFFFFFFh
            mov     _cache, dl
            mov     i, edx
            cmp     srclen, 0
            jle     loc_01
        loc_12:     
            mov     eax, psrc
            mov     ecx, i
            mov     j, 0
            mov     dl, [ecx+eax]
            mov     b, dl
        loc_11:     
            mov     al, b
            mov     cl, byte ptr j
            mov     edx, uiSom
            shr     al, cl
            shr     edx, 0Bh
            imul    edx, ebx
            test    al, 1
            jnz     loc_02
            mov     ecx, 800h
            mov     uiSom, edx
            sub     ecx, ebx
            shr     ecx, 5
            add     ebx, ecx
            jmp     loc_03
        loc_02:     
            add     Low_l, edx
            adc     Low_h, 0
            sub     uiSom, edx
            mov     edx, ebx
            shr     edx, 5
            sub     ebx, edx
        loc_03:     
            cmp     uiSom, 1000000h
            jnb     loc_04
            shl     uiSom, 8
            cmp     Low_l, 0FF000000h
            jb      loc_05
            cmp     Low_h, 0
            jz      loc_06
        loc_05:     
            mov     cl, _cache
            test    esi, esi
            jz      loc_07
            mov     al, byte ptr Low_h
        loc_08:     
            mov     dl, al
            add     dl, cl
            mov     ecx, buf
            mov     [edi+ecx], dl
            inc     edi
            or      cl, 0FFh
            dec     esi
            jnz     loc_08
        loc_07:     
            mov     eax, Low_l
            shr     eax, 18h
            mov     _cache, al
            jmp     loc_09
        loc_06:     
            mov     al, _cache
        loc_09:     
            shl     Low_l, 8
            mov     Low_h, 0
            inc     esi
            jmp     loc_10
        loc_04:
            mov     al, _cache
        loc_10:
            inc     j
            cmp     j, 8
            jl      loc_11
            mov     edx, srclen
            inc     i
            cmp     i, edx
            jl      loc_12
        loc_01:     
            mov     srclen, 5
        loc_17:     
            mov     edx, Low_l
            cmp     edx, 0FF000000h
            jb      loc_13
            cmp     Low_h, 0
            jz      loc_14
        loc_13:     
            test    esi, esi
            jz      loc_15
            mov     cl, byte ptr Low_h
        loc_16:     
            mov     bl, cl
            add     bl, al
            mov     eax, buf
            mov     [edi+eax], bl
            inc     edi
            or      al, 0FFh
            dec     esi
            jnz     loc_16
        loc_15:     
            mov     eax, edx
            shr     eax, 18h
        loc_14:     
            inc     esi
            shl     edx, 8
            mov     Low_l, edx
            mov     Low_h, 0
            dec     srclen
            jnz     loc_17
            mov     i, edi
        }

        return i;
}
```

我测试了 `ccfer` 的 `Test_Encode`，恭喜你，答对了！

强啊！这么难的题，都被解了，怪不得卫星都上天了呢！

心情比较差，论文看不进

昨天看了下代码，这儿做个总结

`Decode` 主要就是 `uiAny` 为读入数据，

`prob` 作为调整数据，每次都在 1024 左右

然后按照 `u`、`uiSom` 解码，如果 `uiAny` 落在 `[0-u)` 之间，解码 0，如果 `uiAny` 落在 `[u-uiSom]` 之间，就解码为 1

对 `uiSom`、`prob`、`u` 做调整，

调整在一个 8 位字节中，大概主要是对 `0xffffffff-0x0` 之间，当然，具体有小变化，由 `prob` 调整，每次 `u` 大概为 `uiSom` 的一半左右，看前次输出的值为 0 还是为 1，调整 `prob`，如果输出为 0，每次 `prob` 变大一点,不到 `(1/32)`，否则，每次 `prob` 变小，减少 `1/32`，

调整大概就是对 `0xffffffff-0x0` 之间

`0xffffffff/2`，输出 1,小于，输出 0，------------------第一位

`0xffffffff/2`，`0xffffffff/4` 输出 1,小于，输出 0，-----第二位

`0xffffffff/4`，`0xffffffff/8` 输出 1,小于，输出 0，-----第三位

。。。。。。。。。。

注意上面不是一定的，是大概情况而以

以上所说的就是解码这一块

我想了下，如果要编码的话，应该要构造出符合上述条件的 `uiAny` ,但是从解码过程可以看到，

每次解码大概 8 位以后，是需要读入下一个字节的，然后 `uiAny` 左移 8 位，而每次比较的时候是 `uiAny`,每次为输出为 1 时需要减的也是 `uiAny`，所以 `uiAny` 是后向相关的，也就是说，每次对 `uiAny` 减了一次后，对后面的解码有影响。

所以在编码的时候如何考虑这个后向影响呢？

首先，编码的时候，我想到的结构和解码一样，

`u`、`uiSom`、`prob` 都没有变化

`u=......`

```
if(symble ==0)
{
}else
{
}
```

不过对 `Low` 做变化的时候需要 `ShiftLow`，感觉要在这块处理后向相关，应该是每次逆置，

\------------此处没有搞通----------达人不要谦虚阿，多放上来：））

`FlushData` 是要对 `Low` 中残余的数据输出........:)

给出一个代码;)

代码:

```
    void bEncode(UINT symbol)    {   
        UINT  u = (this->uiSom >> 11) * this->Prob;     // 变小,大概 1/2 左右，看 Prob 是大于 0x400 还是小于 0x400    
        if(symbol == 0){// uiAny 始终比 uiSom 小     
            this->uiSom = u;           
            
            this->Prob += ((1 << 11) - this->Prob) >> 5;   
        }
        else{    // uiAny >= u     
            //uiAny -= u;     
            this->Low += u;    
            this->uiSom -= u;    
            this->Prob -= (this->Prob) >> 5;// prob 变小，减少 1/32，如果一直输入一，则 prob 会变得很小-0x1f
        }    
        if(this->uiSom < (1 << 24)){    
          //如果最高位 8 位全 0,写入一个字节,同时左移 8 位    
            this->ShiftLow();    
            this->uiSom <<= 8;
        }
    }       
    void ShiftLow()
    {   
        if(_cacheSize)
        {
            if(_cache == 0xff && (Low>>32 & 0x000000ff) )   
            {
                int i;
                for (i=1; m_pbuf[m_len-i]==0xff&&(m_len-i)>=0;i++)   
                    m_pbuf[m_len-i]=0;
                if((m_len-i)>=0)m_pbuf[m_len-i]++;    
                else
                {
                    for(i=m_len++; i>=0; i--)
                        m_pbuf[i]=m_pbuf[i-1];
                    m_pbuf[0]=1;
                }
            }
            WriteByte(((Low>>32 & 0x000000ff) + _cache));   
            _cacheSize--;
        }
        _cache = ((Low >>24) & 0x000000ff);    
        _cacheSize++;    
        Low <<= 8;   
        Low &= 0xffffffff;
    }       
    void FlushData()
    {
        for(int i = 0; i<_cacheSize+sizeof(DWORD)/sizeof(BYTE);i++)
            ShiftLow();
    }
```

还是 `ccfer` 基础扎实 学习了 `![（img/9e827a3de2ad55eed0f95082425861b2.jpg "微笑")`

`ps: BCB 中需要修改 Aker 的一行代码`

```
_cache = ((Low >>24) & 0x000000ff);    
_cacheSize++; 
Low = (Low<<8);   
Low &= 0xffffffff;
```
