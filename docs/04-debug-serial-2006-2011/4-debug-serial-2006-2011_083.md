
# MiniQQ V1.4

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-10.htm`](https://www.pediy.com/kssd/pediy08/pediy8-10.htm)

**题目：** 菜鸟分析 MiniQQ V1.4 免费版 2006.01.01 字符串加密算法

**软件名称：** MiniQQ V1.4 免费版 2006.01.01

**软件来源：** Http://www.PrettySky.Com

**软件平台：** Win9x/Me/NT4/2000

**软件简介：**

请相信自己的眼睛，你没有看错，虽然 70K 的 QQ 客户端确实令人不可思议。迷你 QQ (MiniQQ) 是一个超级小的 QQ 客户端，他采用命令行方式工作，字符界面 + 声音带来的效果的确非常 Cool，用它就可以和自己 QQ 上的好友交流，它使用一种精简版的 `TCP/QQ` 通讯协议，这种协议流量极小，在你网速不快，或者流量需要收费的时候（比如笔记本链接手机用 GPRS 上网）你将会获得更快的速度，和更少的费用（GPRS 接入点 `cwwap` 时请使用第三方 HTTP 代理软件即可）。

迷你 QQ (MiniQQ) 具备了所有常用的功能的功能有，显示好友列表、文字聊天、添加好友、删除好友、查看资料、修改登录状态等，语音和视频聊天功能我们目前不打算加入。迷你 QQ (MiniQQ) 直接连接腾讯官方的 TCP 服务器，密码传送采用 `MD5` 不可逆加密法，请放心您的账号安全。

**破解目的：** 练手

**破解难度：** 简单

**工具：** `ollydbg`，`PEID`，`ida`

在 skycn 闲逛看到此软件觉得好奇就下了，用的的确是 TCP 协议，应该是和 `qq` 的手机接口通讯。软件免费版本第一次和好友通讯会带一些提示信息，用 `peid` 查了一下是 `upx` 的壳，手脱了一下。搜索提示字符串没有结果，用 `od` 载入，搜索引用字符串也没有。看样部分字符串资源是加密了。

怎么找到加密字符串的地方，我的思路是封包一定要发出去，先看看封包。用 `commview` 查看了一次封包，是明码查看发送消息的命令字符串中包含 `"cltmsg"`。

用 `od` 装载程序，查看字符串引用，下断点到所有引用包含 `"cltmsg"` 的地方启动软件，登陆后发送一个消息给任意好友 很快断下在 `00417AFB` 处

```
00417ABF    B8 B8934100     MOV EAX,2\_.004193B8                    ; 请输入信息内容:
00417AC4    E8 F7C5FFFF     CALL 2\_.004140C0
00417AC9    B8 E8C84100     MOV EAX,2\_.0041C8E8
00417ACE    E8 39C1FFFF     CALL 2\_.00413C0C
00417AD3    833D E8C84100 0>CMP DWORD PTR DS:[41C8E8],0
00417ADA    0F84 94010000   JE 2\_.00417C74
00417AE0    833D F0C84100 F>CMP DWORD PTR DS:[41C8F0],-1
00417AE7    0F84 BE000000   JE 2\_.00417BAB
00417AED    807B 0C 00      CMP BYTE PTR DS:[EBX+C],0
00417AF1    0F85 B4000000   JNZ 2\_.00417BAB
00417AF7    C643 0C 01      MOV BYTE PTR DS:[EBX+C],1
00417AFB    68 D0934100     PUSH 2\_.004193D0                         ; ver=1.3&con=1&cmd=cltmsg&seq=
00417B00    8D85 30FFFFFF   LEA EAX,DWORD PTR SS:[EBP-D0]
00417B06    50              PUSH EAX
00417B07    B8 3F420F00     MOV EAX,0F423F
00417B0C    E8 F3B2FEFF     CALL 2\_.00402E04
00417B11    8945 BC         MOV DWORD PTR SS:[EBP-44],EAX
00417B14    C645 C0 00      MOV BYTE PTR SS:[EBP-40],0
00417B18    8D55 BC         LEA EDX,DWORD PTR SS:[EBP-44]
00417B1B    33C9            XOR ECX,ECX
```

经过几次跟踪很快发现了这里

```
00417B48    8D95 24FFFFFF   LEA EDX,DWORD PTR SS:[EBP-DC]
00417B4E    B8 18944100     MOV EAX,2\_.00419418
00417B53    E8 7CBFFFFF     CALL 2\_.00413AD4
00417B58    8B8D 24FFFFFF   MOV ECX,DWORD PTR SS:[EBP-DC]
```

`00413AD4` 是用来完成字符串编码的

`413AD4` 的代码如下

```
00413AD4    55              PUSH EBP
00413AD5    8BEC            MOV EBP,ESP
00413AD7    83C4 F8         ADD ESP,-8
00413ADA    53              PUSH EBX
00413ADB    56              PUSH ESI
00413ADC    57              PUSH EDI
00413ADD    8BFA            MOV EDI,EDX
00413ADF    8945 FC         MOV DWORD PTR SS:[EBP-4],EAX
00413AE2    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
00413AE5    E8 4A0BFFFF     CALL 2\_.00404634
00413AEA    33C0            XOR EAX,EAX
00413AEC    55              PUSH EBP
00413AED    68 793B4100     PUSH 2\_.00413B79
00413AF2    64:FF30         PUSH DWORD PTR FS:[EAX]
00413AF5    64:8920         MOV DWORD PTR FS:[EAX],ESP
00413AF8    8BC7            MOV EAX,EDI
00413AFA    8B55 FC         MOV EDX,DWORD PTR SS:[EBP-4]
00413AFD    E8 FA06FFFF     CALL 2\_.004041FC
00413B02    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
00413B05    E8 3A09FFFF     CALL 2\_.00404444
00413B0A    8BF0            MOV ESI,EAX
00413B0C    D1FE            SAR ESI,1
00413B0E    79 03           JNS SHORT 2\_.00413B13
00413B10    83D6 00         ADC ESI,0
00413B13    85F6            TEST ESI,ESI
00413B15    7E 4C           JLE SHORT 2\_.00413B63
00413B17    BB 01000000     MOV EBX,1
00413B1C    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]            ; 得到密文数据结构指针
00413B1F    E8 2009FFFF     CALL 2\_.00404444                        ; 获得密文数据长度
00413B24    2BC3            SUB EAX,EBX                               ; 从后向前取数据
00413B26    8B55 FC         MOV EDX,DWORD PTR SS:[EBP-4]            ; 得到密文数据结构指针
00413B29    8A0402          MOV AL,BYTE PTR DS:[EDX+EAX]            ; 取得一 BYTE 密文数据
00413B2C    34 01           XOR AL,1
00413B2E    8845 FB         MOV BYTE PTR SS:[EBP-5],AL               ; 保存到临时变量 2
00413B31    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]            ; 得到密文数据结构指针
00413B34    E8 0B09FFFF     CALL 2\_.00404444                        ; 获得密文数据长度
00413B39    2BC3            SUB EAX,EBX
00413B3B    50              PUSH EAX
00413B3C    8BC7            MOV EAX,EDI
00413B3E    E8 510BFFFF     CALL 2\_.00404694
00413B43    5A              POP EDX                                   ; 从前向后取数据
00413B44    8B4D FC         MOV ECX,DWORD PTR SS:[EBP-4]            ; 得到密文数据结构指针
00413B47    8A4C19 FF       MOV CL,BYTE PTR DS:[ECX+EBX-1]          ; 取得一 BYTE 密文数据
00413B4B    80F1 01         XOR CL,1
00413B4E    880C10          MOV BYTE PTR DS:[EAX+EDX],CL
00413B51    8BC7            MOV EAX,EDI
00413B53    E8 3C0BFFFF     CALL 2\_.00404694
00413B58    8A55 FB         MOV DL,BYTE PTR SS:[EBP-5]
00413B5B    885418 FF       MOV BYTE PTR DS:[EAX+EBX-1],DL
00413B5F    43              INC EBX
00413B60    4E              DEC ESI
00413B61   ^ 75 B9          JNZ SHORT 2\_.00413B1C
00413B63    33C0            XOR EAX,EAX
00413B65    5A              POP EDX
00413B66    59              POP ECX
00413B67    59              POP ECX
00413B68    64:8910         MOV DWORD PTR FS:[EAX],EDX
00413B6B    68 803B4100     PUSH 2\_.00413B80
00413B70    8D45 FC         LEA EAX,DWORD PTR SS:[EBP-4]
00413B73    E8 3006FFFF     CALL 2\_.004041A8
00413B78    C3              RETN
```

C 函数

```c
void EnCode(const char* Src, int Len, char* Dest)
{
  BYTE al, cl;
  DWORD ebx, esi;
  esi = Len;
  esi = esi >> 1;
  ebx = 1;
  while(esi > 0)
  {
    al = Src[Len-ebx];
    al = al ^ 1;
    cl = Src[ebx-1];
    cl = cl ^ 1;
    Dest[Len-ebx] = cl;
    Dest[ebx-1] = al;
    ebx++;
    esi--;
  }
  Dest[Len] = 0;
}
```

**后记：**

作者真的很聪明，连手机接口都能用上， 程序用 `delphi` 写的，占用的内存还是不少的 呵呵。用命令行聊 `qq` 满好玩的。第一次写这方面文章，写的不好之处还请大家包含。

2006 年 1 月 2 日

`gameres`:`pk2000` `pediy`:`goodcode`

`4dd466d0e1e0cc123afbe504da2462ca`
