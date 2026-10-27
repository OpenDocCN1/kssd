
```
  10011390   .  54            push    esp
  10011391   .  E8 D2FFFFFF   call    10011368
  10011396   .  5C            pop     esp
  10011397   .  61            popad
  10011398   .  9D            popfd
  10011399   .  C3            retn
```

跳来跳去，终于到了关键代码

```
  100112BF  |.  50            push    eax
  100112C0  |.  FF75 08       push    dword ptr [ebp+8]
  100112C3  |.  E8 E5FEFFFF   call    100111AD // 初始化MD5参数
  100112C8  |.  8B85 E8FDFFFF mov     eax, dword ptr [ebp-218]
  100112CE  |.  B9 38CE0610   mov     ecx, 1006CE38
  100112D3  |.  8BF8          mov     edi, eax
  100112D5  |.  2B43 04       sub     eax, dword ptr [ebx+4]
  100112D8  |.  50            push    eax // 调用返回地址的RVA（0040279A-00400000=279A）
  100112D9  |.  E8 DCED0300   call    100500BA // 计算MD5
  100112DE  |.  50            push    eax // eax 是MD5的前4字节，就是关键的key
  100112DF  |.  E8 F1010000   call    100114D5
  100112E4  |.  8BC8          mov     ecx, eax
  100112E6      E8 FC010000   call    100114E7 // 在一个128项的哈希表内查找key，算法有兴趣的自己看下，没有冲突时，顺序查找也是一样。
  100112EB  |.  8BF0          mov     esi, eax
  100112ED  |.  85F6          test    esi, esi
  100112EF  |.  74 3F         je      short 10011330
  100112F1  |.  66:837B 08 01 cmp     word ptr [ebx+8], 1
  100112F6  |.  75 3D         jnz     short 10011335
  100112F8  |.  8D85 30FDFFFF lea     eax, dword ptr [ebp-2D0]
  100112FE  |.  8BCE          mov     ecx, esi
  10011300  |.  50            push    eax
  10011301  |.  E8 E4580100   call    10026BEA  // 这里有模拟执行的代码，
  10011306  |.  8BCB          mov     ecx, ebx
  10011308  |.  E8 8AFEFFFF   call    10011197
  1001130D  |.  83F8 04       cmp     eax, 4    // 比较1
  10011310  |.  72 14         jb      short 10011326
  10011312  |.  8BCE          mov     ecx, esi
  10011314  |.  E8 B7570100   call    10026AD0
  10011319  |.  83F8 04       cmp     eax, 4    // 比较2（1和2有一个调用的次数，另一个不清楚）
  1001131C  |.  72 08         jb      short 10011326
  1001131E  |.  57            push    edi       // 2次比较都不跳，会到这里
  1001131F  |.  8BCE          mov     ecx, esi
  10011321  |.  E8 FB570100   call    10026B21  // 内部有还原代码，如果前面跳走了，就不还原了
```

总结一下解密算法是先对调用返回地址的RVA`4 Bytes`求MD5，以结果的前4字节为Key，在存放还原信息的哈希表中查找。

找到之后模拟运行，再根据调用计数器判断是否还原代码，返回。

快速确定还原数据的方法，SafeDisc2.8中在还原信息前`$1006CEB8`有很大一片`0x0001`，之后`$1006CFB8`紧跟的就是还原信息。

还原信息的结构为

```c
struct sd_info {
    unsigned int size;   // 使用前跟0x9877D4A7异或
    unsigned int id;     // 使用前跟0x6F68D2EB异或
    unsigned int code2;  // 使用前跟0xE2536C94异或
    unsigned int code1;  // 使用前跟0x07AEECEA异或
    unsigned int zero[8];// 占位用，全是零
};
```

最后根据`size`的大小，将`code1`和`code2`拼接成原始数据，例如：

原始数据 `size = 6`; `code1 = 000F8400`; `code2 = 00000175`

拼接前   `00 00 00 00 00 00` 一块空内存

`code1`    `0F 84 00`

`code2`          `75 01 00 00`

拼接后   `0F 84 75 01 00 00`

最后同样写程序对代码段穷举一下，结果很快就出来了，128个刚刚好。

## 三、后记

写到最后反而没什么好写，那么删除`stxt774`和`stxt371`两个区段，重构一下文件吧。

14天说多不多，说少不少。我与`safedisc`的第一次邂逅就这样结束了。

风来西林外传的汉化版还要等多久呢？

--------------------------------------------------------------------------------

2009年05月16日 22:43:15

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
