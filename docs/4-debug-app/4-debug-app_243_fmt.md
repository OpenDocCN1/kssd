
# 破解网络游戏幸福花园 Online

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-48.htm`](https://www.pediy.com/kssd/pediy07/pediy7-48.htm)

【破解作者】 `shoooo[iPB][pycg]`  
【作者邮箱】 `shoooo314@163.com`  
【使用工具】 `softice` 套装 2.7 `iceext` 插件，`Iris`  
【破解平台】 `win2000` `sp4`  
【破解目标】 账号，密码  
【软件名称】 幸福花园 Online 内测中  
【破解时间】 2005-01-28 午  
【加壳方式】 无壳  
【破解声明】 应一好友要求，帮忙看看幸福花园 Online，隧有此文，不对的地方大家批评

## 一，登录包

截包工具有很多，看个人喜好了~~我 `Iris` 用惯了

以账号 `shoooo` 密码 `12345` 登录，截到的登录包是 `"LVJMV2V0dTIrQ0dWZEoeWVYdKTpmel4DDERF\r"` (注：最后的 `"\r"` 是 `0x0A`)

再登录几次，截到的登录包是相同的

总结：加密算法是上下文无关单包操作的，偶非常稀饭

## 二，从 send 包反推

绝大多数网游的客户端与服务器用的是一套加密，即客户端加密，服务器解密；服务器加密，客户端解密（好像是废话）

我想说的是，客户是端含有解密代码的，只要断 `recv` 后，观察客户端是怎么处理接收的包就能找到解密过程。

但是偶喜欢从 `send` 包反推，研究它的加密过程，自己搞解密。

呼出 `sice`, `bpx send`

以 `shoooo,12345` 登录，断下，`F11` 返回

```
00416759   |.  6A 00       push 0                           ; /Flags = 0
0041675B   |.  53         push ebx                         ; |DataSize
0041675C   |.  50         push eax                         ; |Data    我们关心的是这里的值，很可惜断在正面时 eax 的值被改了
0041675D   |.  51         push ecx                         ; |Socket
0041675E   |.  E8 2F9C1200 call <jmp.&WSOCK32.#19>          ; \send
00416763   |.  8BF0       mov esi,eax                      ; 停在这里
```

`bc *` 取消 `send` 断点

`bpx 41675e`，再次登录断下，这次可以看清 `Data` 的值了是，也就是发送数据的地址是 `617B2C`

`bpmd 617B2C w` 再次登录，断在这儿

```
004168CE   |.  BE D0F19100 mov esi,ST_cb_01.0091F1D0                         ESI= 91F1D0
004168D3   |.  BF 2C7B6100 mov edi,ST_cb_01.00617B2C                         EDI= 617B2C
004168D8   |.  C705 D0F192>mov dword ptr ds:[92F1D0],0
004168E2   |.  C1E9 02     shr ecx,      2                                     这句做完 ECX= 1A
004168E5   |.  F3:A5      rep movs dword ptr es:[edi],dword ptr ds:[esi]     memcpy(EDI, ESI, ECX)
004168E7   |.  8BCA       mov ecx,edx
004168E9   |.  50         push eax
004168EA   |.  83E1 03    and ecx,3
004168ED   |.  68 2C7B6100 push ST_cb_01.00617B2C
004168F2   |.  F3:A4      rep movs byte ptr es:[edi],byte ptr ds:[esi]
```

可见 `617B2C` 中发送的数据是从 `91F1D0` 中 `memcpy` 得来

`bpmd 91F1D0 w` 再次登录，断在这儿

```
0041683B   |.  03F2       add esi,edx                                   ESI= 5C7B2C
0041683D   |>  8BC1       mov eax,ecx
0041683F   |.  8DBD D0F191>lea edi,dword ptr ss:[ebp+91F1D0]             ESI= 91F1D0
00416845   |.  C1E9 02    shr ecx,2
00416848   |.  F3:A5      rep movs dword ptr es:[edi],dword ptr ds:[esi]     memcpy
0041684A   |.  8BC8       mov ecx,eax
0041684C   |.  83E1 03    and ecx,3
0041684F   |.  F3:A4      rep movs byte ptr es:[edi],byte ptr ds:[esi]
```

又 `memcpy`! `91F1D0` 的数据从 `5C7B2C` 处来，没完没了

继续 `bpmd 5C7B2C w`，登录，断在这儿

```
00415C53   |.  85C9       test ecx,ecx
00415C55   |.  0F8E 100100>jle ST_cb_01.00415D6B

.....

00415D75   |.  5F         pop edi
00415D76   |.  5B         pop ebx
00415D77   |.  83C4 14    add esp,14
00415D7A   |\.  C3        retn                                      返回到  416627
```

这里已经在执行加密的过程了，`bc *`，`g 415d7a`，跑出这个加密的 `call`，来到这里

```
004165E7   |.  68 00000100 push 10000

004165EC   |.  68 2C7B5F00 push ST_cb_01.005F7B2C
004165F1   |.  51         push ecx
004165F2   |.  52         push edx
004165F3   |.  E8 58F9FFFF call ST_cb_01.00415F50
004165F8   |.  6A 40      push 40
004165FA   |.  68 E87A5C00 push ST_cb_01.005C7AE8
004165FF   |.  50         push eax
00416600   |.  68 2C7B5F00 push ST_cb_01.005F7B2C
00416605   |.  E8 36000000 call ST_cb_01.00416640
0041660A   |.  83C4 20    add esp,20
0041660D   |.  68 2C7B5C00 push ST_cb_01.005C7B2C
00416612   |.  50         push eax
00416613   |.  68 2C7B5F00 push ST_cb_01.005F7B2C
00416618   |.  56         push esi
00416619   |.  E8 12F9FFFF call ST_cb_01.00415F30
0041661E   |.  83C4 04    add esp,4
00416621   |.  50         push eax
00416622   |.  E8 E9F5FFFF call ST_cb_01.00415C10
00416627   |.  50         push eax                 前面的 ret 跳到了这里，因此前面的 call 都值得注意
00416628   |.  68 2C7B5C00 push ST_cb_01.005C7B2C
0041662D   |.  56         push esi
0041662E   |.  E8 2D000000 call ST_cb_01.00416660
```

总结一下，客户端把数据明文加密后第一时间写在 `5C7B2C`，然后 `memcpy` 到 `91F1D0`，再 `memcpy` 到 `617B2C`，然后 `send` 出去

## 三，加密

从 `EIP：4165EC` 开始看，因为再前面没什么特别的

因此，`bc *`，`bpx 4165ec`，先 `F10`(相当于 `OD` 中的 `F8`)走一遍，看看加密的大致流程

```
004165E7   |.  68 00000100 push 10000
004165EC   |.  68 2C7B5F00 push ST_cb_01.005F7B2C
004165F1   |.  51         push ecx                ECX= 1A 明文长度
004165F2   |.  52         push edx                明文的地址
004165F3   |.  E8 58F9FFFF call ST_cb_01.00415F50
```

这里我们看到了明文中的地址，可以 `d edx` 看看明文是什么

```
61 03 00 00 00 00 00 00  0E 00 00 00 06 73 68 6F
    命令码                     长度 s  h  o

6F 6F 6F 05 31 32 33 34  35 00
    o  o  o 长度 1  2  3  4   5
```

这个 `call` 过后，显然看出，它把 `EAX` 指向的明文 `memcpy` 到了 `5F7B2C` 内存处

```
004165F8   |.  6A 40      push 40                 十进制 64
004165FA   |.  68 E87A5C00 push ST_cb_01.005C7AE8          指向一张 64 字节大小的第一次加密的密钥表
004165FF   |.  50         push eax                明文长度
00416600   |.  68 2C7B5F00 push ST_cb_01.005F7B2C          指向明文，同时密文这里输出
00416605   |.  E8 36000000 call ST_cb_01.00416640          第一次加密，详见 A 部分
```

这个 `CALL` 后，`5F7B2C` 处的明文加密成了

```
2D 52 4C 57 65 74 75 32  2B 43 47 56 64 4A 1E 59
56 1D 29 3A 66 7A 5E 03  0C 44 45
```

```
0041660A   |.  83C4 20    add esp,20
0041660D   |.  68 2C7B5C00 push ST_cb_01.005C7B2C          指向我们期待的最终密文的内存地址
00416612   |.  50         push eax                长度
00416613   |.  68 2C7B5F00 push ST_cb_01.005F7B2C          经过第一次加密后的密文
00416618   |.  56         push esi
00416619   |.  E8 12F9FFFF call ST_cb_01.00415F30          这个 call 过后，上面的数据都无发生变化，不采
0041661E   |.  83C4 04    add esp,4
00416621   |.  50         push eax                eax+100 处是第二次加密的密钥表
00416622   |.  E8 E9F5FFFF call ST_cb_01.00415C10          最终第二次加密，详见 B 部分
```

这个 `call` 过后，我们期待的 `5C7B2C` 处出现了最终的密文。

总结：客户端把明文先第一次加密放在 `5F7B2C` 处，再第二次加密放到 `5C7B2C`，加密完成。

## 四，算法

### A 部分：跟进第一次加密的 CALL 看看做了些什么，很容易找到这一段：

```
0041623C   |> /3BC8       /cmp ecx,eax              加密完了吗?
0041623E   |. |7C 04      |jl short ST_cb_01.00416244      没有就跳下去继续加密
00416240   |. |33C9       |xor ecx,ecx
00416242   |. |EB 07      |jmp short ST_cb_01.0041624B     加密完了就跳走 88
00416244   |> |8A1C39     |mov bl,byte ptr ds:[ecx+edi]    取出一个密钥
00416247   |. |301C32     |xor byte ptr ds:[edx+esi],bl    明文异或密钥
0041624A   |. |41         |inc ecx                   密钥指针加 1
0041624B   |> |42         |inc edx                   明文指针加 1
0041624C   |. |3BD5       |cmp edx,ebp               明文还有要加密的吗?
0041624E   |.\7C EC       \jl short ST_cb_01.0041623C      有的话跳上去继续加密吧
```

总结：依次取出明文与密钥异或后产生密文

这里密钥表是固定的

```
-3OWetu2+MGVdLm1
9rFUcKl08qETbJkz
7pDSaIjy6oCRZHix
5nAQYghw4BsPXfNv
```

因此这段加密也即是解密

```c
   void sg_de0(unsigned char* data, int length)
   {
      int i;
      unsigned char key[64] = "-3OWetu2+MGVdLm19rFUcKl08qETbJkz7pDSaIjy6oCRZHix5nAQYghw4BsPXfNv";
      for (i=0; i<length; i++)
      {
         data[i] ^= key[i%0x40];
      }
   }
```

### B 部分：这里我们暂且把第一次加密后的密文称为明文吧，跟进第二次加密的 call，很容易发现它是每 3 个字节加密成 4 个字节

```
00415C83   |> \8A541E 01  |mov dl,byte ptr ds:[esi+ebx+1]      取出明文的第 2 字节
00415C87   |.  83C1 FE    |add ecx,-2
00415C8A   |.  3BF1       |cmp esi,ecx
```



### 逆向分析：加密逻辑

```asm
00415C8C   |.  8A0C1E       |mov cl,byte ptr ds:[esi+ebx]               取出明文的第 1 字节  
00415C8F   |.  884C24 10    |mov byte ptr ss:[esp+10],cl  
00415C93   |.  885424 14    |mov byte ptr ss:[esp+14],dl  
00415C97   |.  7C 09        |jl short ST_cb_01.00415CA2  
00415C99   |.  32C0         |xor al,al  
00415C9B   |.  BD 03000000  |mov ebp,3  
00415CA0   |.  EB 09        |jmp short ST_cb_01.00415CAB  
00415CA2   |>  8A441E 02    |mov al,byte ptr ds:[esi+ebx+2]             取出明文的第 3 字节  
00415CA6   |.  BD 04000000  |mov ebp,4  
00415CAB   |>  80E2 0F      |and dl,0F                                  第 2 个字节留下后 4 位  
00415CAE   |.  8AD8         |mov bl,al  
00415CB0   |.  C0E2 02      |shl dl,2                                   第 2 个字节留下后 4 位后左移 2 位  
00415CB3   |.  C0EB 06      |shr bl,6                                   第 3 个字节右移 6 位，最高 2 位移到低 2 位  
00415CB6   |.  0AD3         |or dl,bl                                   把上面 2 个或操作，即合并起来，不存在冲突部分，放 dl  
00415CB8   |.  24 3F        |and al,3F                                  第 3 个字节只留下最后 6 位，放 al  
00415CBA   |.  83FD 02      |cmp ebp,2  
00415CBD   |.  885424 1C    |mov byte ptr ss:[esp+1C],dl                dl 内存中存一下 一会儿要取出  
00415CC1   |.  884424 20    |mov byte ptr ss:[esp+20],al                al 内存中存一下 一会儿要取出  
00415CC5   |.  7C 46        |jl short ST_cb_01.00415D0D  
00415CC7   |.  8B5424 10    |mov edx,dword ptr ss:[esp+10]  
00415CCB   |.  8B4424 28    |mov eax,dword ptr ss:[esp+28]  
00415CCF   |.  81E2 FF0000> |and edx,0FF  
00415CD5   |.  80E1 03      |and cl,3                                   第 1 个字节只留下低 2 位  
00415CD8   |.  C1EA 02      |shr edx,2                                  第 1 个字节右移 2 位，即高 6 位移到低位  
00415CDB   |.  C0E1 04      |shl cl,4                                   上句的上句的低 2 位左称 4 位  
00415CDE   |.  8A9402 0001> |mov dl,byte ptr ds:[edx+eax+100]          上句的上句的值查表产生第 1 个密文  
00415CE5   |.  81E1 FF0000> |and ecx,0FF  
00415CEB   |.  8817         |mov byte ptr ds:[edi],dl  
00415CED   |.  8B5424 14    |mov edx,dword ptr ss:[esp+14]  
00415CF1   |.  81E2 FF0000> |and edx,0FF  
00415CF7   |.  47          |inc edi  
00415CF8   |.  C1EA 04      |shr edx,4                                  第 2 个字节右移 4 位  
00415CFB   |.  0BCA         |or ecx,edx                                 与移到高位的第 1 字节的低 2 位合并起来  
00415CFD   |.  47          |inc edi  
00415CFE   |.  8A8C01 0001> |mov cl,byte ptr ds:[ecx+eax+100]          合并后查表产生第 2 个密文  
00415D05   |.  884F FF      |mov byte ptr ds:[edi-1],cl  
00415D08   |.  C607 00      |mov byte ptr ds:[edi],0  
00415D0B   |.  EB 04        |jmp short ST_cb_01.00415D11  
00415D0D   |>  8B4424 28    |mov eax,dword ptr ss:[esp+28]  
00415D11   |>  83FD 03      |cmp ebp,3  
00415D14   |.  7C 18        |jl short ST_cb_01.00415D2E  
00415D16   |.  8B5424 1C    |mov edx,dword ptr ss:[esp+1C]              刚刚的 dl 出来  
00415D1A   |.  81E2 FF0000> |and edx,0FF  
00415D20   |.  47          |inc edi  
00415D21   |.  8A8C02 0001> |mov cl,byte ptr ds:[edx+eax+100]          这个 dl 查表产生第 3 个密文  
00415D28   |.  884F FF      |mov byte ptr ds:[edi-1],cl  
00415D2B   |.  C607 00      |mov byte ptr ds:[edi],0  
00415D2E   |>  83FD 04      |cmp ebp,4  
00415D31   |.  7C 18        |jl short ST_cb_01.00415D4B  
00415D33   |.  8B5424 20    |mov edx,dword ptr ss:[esp+20]              刚刚的 al 出来  
00415D37   |.  81E2 FF0000> |and edx,0FF  
00415D3D   |.  47          |inc edi  
00415D3E   |.  8A8402 0001> |mov al,byte ptr ds:[edx+eax+100]          这个 al 查表产生第 4 个密文
```

这一段加密乱七八糟的，不过看了下面相信你应该完全清楚了。

设原明文 3 字节 24 位为：

`a1 a2 a3 a4 a5 a6 a7 a8 b1 b2 b3 b4 b5 b6 b7 b8 c1 c2 c3 c4 c5 c6 c7 c8`

然后：

- `00 00 a1 a2 a3 a4 a5 a6` 查表的位置后取出第 1 个密文
- `00 00 a7 a8 b1 b2 b3 b4` 查表的位置后取出第 2 个密文
- `00 00 b5 b6 b7 b8 c1 c2` 查表的位置后取出第 3 个密文
- `00 00 c3 c4 c5 c6 c7 c7` 查表的位置后取出第 4 个密文

很无聊吧。

看看密钥表：

```
ABCDEFGHIJKLMNOP
QRSTUVWXYZabcdef
ghijklmnopqrstuv
wxyz0123456789+-
```

解密函数 C 源：

```c
void sg_de4(unsigned char in[4], unsigned char out[3]) {
    unsigned char i;
    for (i=0; i<4; i++) {
        if (in[i]>='A' && in[i]<='Z') {
            in[i] -= 'A';
        } else if (in[i]>='a' && in[i]<='z') {
            in[i] = in[i] - 'a' + 26;
        } else if (in[i]>='0' && in[i]<='9') {
            in[i] = in[i] - '0' + 52;
        } else if (in[i] == '+') {
            in[i] = 0x3E;
        } else {
            in[i] = 0x3F;
        }
    }
    out[0] = in[0]<<2 | (in[1]&0x30)>>4;
    out[1] = in[1]<<4 | (in[2]>>2)&0x0F;
    out[2] = in[2]<<6 | in[3];
}
```

总结：3 字节被加密成 4 字节，4 字节要还原成 3 字节，肯定有冗余，要点多数在位移上。

### 五、后话

文章没什么技术含量，看别人的破解文章比自己动手要困难，有兴趣的动手一试吧。

顺便说一下这游戏画面很烂的 ^_^

### 调试技巧与问答

**Q：楼主，有问题想问你一下！！！ 下 `BPMD XXXX W`（`XXXX` 代表地址）。我按 `CTRL+D` 退不出来啊，自己中断啊…… 发现首先中断在系统领空中，`F5` 继续运行又中断在程序领空，在按 `F5`，又中断在系统领空（跟前面一个地方），在按 `F5` 继续运行又中断在程序领空（不是前面的那个程序领空）…… 也就是说不像你那样“`617B2C` 中发送的数据是从 `91F1D0` 中 `memcpy` 得来”是一个地方 `COPY` 过来，我的中断了好几个地方…… 我怎么知道是那个地方 `COPY` 过来的，指点一下，谢谢！！！ 接下来不知道到那里下 `BPMD` 啊……**

A：嗯那，谢谢你的支持！

我是在 `2000`，`sp4` 下调试的，不同的平台地址可能存在线性出入。

每次登录的时候请用错误的账号登录，如果是正确的账号，会登录成功而继续发包。

如果你机器不慢的话，在一次登录调试后，最好能退出游戏客户端，重进游戏，清楚所有断点，设置下一步的断点，继续调试，这样就不太会有其它环境干扰。

如：

1. 第一次只留一个 `bpmd 617B2C w`，登录断下后，应该会看到那个 `memcpy`。
2. 然后，清掉这个断点，退游戏，重进，下 `bpmd 91F1D0 w`，再继续。

当然，`bpmd XXXX` 的值，应该是你本地实际看到的值。

`4dd466d0e1e0cc123afbe504da2462ca`
