
# Multimedia Builder MP3 4.7b

> 原文：[`www.pediy.com/kssd/pediy04/kanxue087.htm`](https://www.pediy.com/kssd/pediy04/kanxue087.htm)

如何破解 `Multimedia Builder MP3 4.7b`

软件简介：
多媒体制作软件，假如你需要制作一些有创作力的多媒体产品，而没有时间学习如何制作，不妨试试 `MMB` 这种互动式的多媒体制作软件，`MMB` 能制作互动式多媒体应用产品，容易使用并且操作简单，能制作的多媒体产品包含卡片，产品介绍发表，光盘 `AutoRun` 选单等。

所需工具：
1. `TRW2000`
2. `W32Dasm`
3. 运气 ^\_^

正文：
废话少说，按照一般的方法，我们可以跟踪到如下信息：

```
:004A19B9 8D755C                   lea esi, dword ptr [ebp+5C]
* Possible StringData Ref from Data Obj ->"@"
|
:004A19BC 681CC45D00               push 005DC41C
:004A19C1 8BCE                      mov ecx, esi
:004A19C3 E866D60A00                call 0054F02E    //判断名字中是否有@符号存在
:004A19C8 83F8FF                    cmp eax, FFFFFFFF
:004A19CB 7521                      jne 004A19EE
:004A19CD 6A00                      push 00000000
:004A19CF 6A00                      push 00000000
* Possible StringData Ref from Data Obj ->"Please enter the Name exactly "
                                     ->"as it is stated
in the registration "
                                     ->"e-mail you received."
|
:004A19D1 68BCE05D00               push 005DE0BC
:004A19D6 E814D90B00                call 0055F2EF
:004A19DB 8B4C2424                  mov ecx, dword ptr [esp+24]
```
名字判断正确后，来到：

```
* Possible StringData Ref from Data Obj ->"1-"
|
:004A1B98 6868C55D00               push 005DC568
:004A1B9D 8D4C2428                  lea ecx, dword ptr [esp+28]
* Possible Ref to Menu: MenuID_0075, Item: "Close"
|
:004A1BA1 C744241C01000000          mov [esp+1C], 00000001
:004A1BA9 E880D40A00                call 0054F02E   //判断注册号起始是否为“1-”
:004A1BAE 85C0                      test eax, eax
:004A1BB0 752D                      jne 004A1BDF
```
继续走，来到：

```
:004A1BDF 55                       push ebp
:004A1BE0 56                       push esi
:004A1BE1 57                       push edi
* Possible StringData Ref from Data Obj ->"-"
|
:004A1BE2 6828B55D00               push 005DB528
:004A1BE7 8D4C2434                  lea ecx, dword ptr [esp+34]
:004A1BEB E83ED40A00                call 0054F02E    //判断注册号后面是否含有“-”
:004A1BF0 8B0D7CCF5E00              mov ecx, dword ptr [005ECF7C]
:004A1BF6 8BF0                      mov esi, eax
:004A1BF8 894C2410                  mov dword ptr [esp+10], ecx
:004A1BFC 83FEFF                    cmp esi, FFFFFFFF
:004A1BFF C644242403                mov [esp+24], 03
:004A1C04 7E59                      jle 004A1C5F
```
名字和注册号格式的判断到此为止，我们可以知道序列号的格式应该是：

`1-xxxxxx-yyyy`

下面来到关键比较：

```
:004A1CBD 8A1438                    mov dl, byte ptr [eax+edi]  //eax+edi 指向我们输入的序列号的
    1-xxxxxx 部分
:004A1CC0 88542418                  mov byte ptr [esp+18], dl
:004A1CC4 8B542418                  mov edx, dword ptr [esp+18]
:004A1CC8 81E2FF000000              and edx, 000000FF
:004A1CCE 03F2                      add esi, edx               //ASCII 码相加
:004A1CD0 40                        inc eax
:004A1CD1 3BC1                      cmp eax, ecx
:004A1CD3 7CE8                      jl 004A1CBD
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A1CBB(C)
|
:004A1CD5 5F                        pop edi
:004A1CD6 3BF5                      cmp esi, ebp     //ebp 中的值就是序列号中的 yyyy，和上面计算
    出来的和进行比较
:004A1CD8 5E                        pop esi
:004A1CD9 5D                        pop ebp
:004A1CDA C644241803                mov [esp+18], 03
:004A1CDF 8D4C2400                  lea ecx, dword ptr [esp]
:004A1CE3 7548                      jne 004A1D2D   //bad guy!
```
跟踪到这里，假设一个序列号：

`Name: luoyi@263.net`
`Code: 1-98765-369`

输入，回车。没有出错提示。信息窗口显示：`Registed to: luoyi@263.net`

成功了？

别急，点菜单：`Project`-->`Run`

看到没有？“`Created with unregisted version of Multimedia Builder`”！

做出来的东西如果带着这么一个标记？是不是很丑？呵呵……

怎么办？`bpm`?`bpr`?

看看未注册信息与众不同的地方：黄色的背景色！熟悉 Windows 编程的马上就会想到：

`SetBkColor`！对！我们就用 `SetBkColor` 设断！

`bpx setbkcolor if (*(esp+8)==0000ffff)` //颜色为黄色:`Red=255,Green=255,Blue=0`

继续点击菜单，boom!被 `TRW2000` 拦下，`pmodule`，来到如下地点：

```
:0055A485 FF74240C                  push [esp+0C]
:0055A489 56                        push esi
:0055A48A FFD7                      call edi
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0055A483(C)
|
:0055A48C 5F                        pop edi
:0055A48D 5E                        pop esi
:0055A48E C20400                    ret 0004
```
`F12` 出去，看到：

```
:004B33AC E883750A00                call 0055A934
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B3329(C)
|
:004B33B1 81BE90030000CA380400      cmp dword ptr [esi+00000390], 000438CA
:004B33BB 0F848B010000              je 004B354C
:004B33C1 8B157CCF5E00              mov edx, dword ptr [005ECF7C]
:004B33C7 89542410                  mov dword ptr [esp+10], edx
* Possible StringData Ref from Data Obj ->"!thhqjwE$fjfhqnunxQ%gq#rtjuui{!fhvjuulkjspx$mu"
                                     ->"kz$ifvdiwD""
|
:004B33CB 68D4305E00               push 005E30D4
:004B33D0 8D4C2414                  lea ecx, dword ptr [esp+14]
:004B33D4 C684249000000002          mov byte ptr [esp+00000090], 02
:004B33DC E887390A00                call 00556D68
:004B33E1 68FFFF0000               push 0000FFFF
:004B33E6 8D4C2448                  lea ecx, dword ptr [esp+48]
:004B33EA E873700A00                call 0055A462
:004B33EF 53                        push ebx
:004B33F0 8D4C2448                  lea ecx, dword ptr [esp+48]
:004B33F4 E850710A00                call 0055A549
:004B33F9 895C2440                  mov dword ptr [esp+40], ebx
:004B33FD C744243C84275B00          mov [esp+3C], 005B2784
```
那一大堆乱七八糟的字符串参考就是“`Created with unregisted version of Multimedia Builder`”的密文。很明显，比较处在：

```
:004B33B1 81BE90030000CA380400      cmp dword ptr [esi+00000390], 000438CA
```
看看`[esi+390]`里头放了什么东西？`1864fH=99919D`

莫名其妙，看不出和我们输入的序列号有什么联系，你想找到他的来源？好，一层层的往外剥，结果还是一无所获。

现在知道我为什么把“运气”列为第 3 个工具了吧？如果不是今天我运气好，可能这个软件就搞不定了！运气是什么呢？接着往下看：

让我们改变一下序列号，然后看看改变以后`[esi+390]`中存放的值有何变化：

| 序列号 | [esi+390] |
| :--- | :--- |
| `1-98765-369` | `99919` |
| `1-98766-370` | `99920` |
| `1-98775-370` | `99929` |
| `1-98865-370` | `100019` |
| `1-99765-370` | `100919` |
| `1-88765-368` | `89919` |

还需要更详细的说明吗？规律很明显了，至于 `99919` 和 `98765` 中间所差的那 `1154`，相信你也应该能猜到了——对！就是 `luoyi@263.net` 字符串各字符 ASCII 码的和！

凭借着好运气，找到了算法，可是注册机的编写就没什么运气了。老老实实的一句句的写：

文件名：`mmb.c`

编译环境:`turbo c 2.0`

编译方法：直接在集成开发环境里 `Make EXE` 就成功了！

```c
#include <stdio.h>

#include <stdlib.h>

#include <string.h>

main()

{

unsigned long sum;

int i;

char sn[20]={"1-"},temp3[2]={"-"};

static char temp1[10],temp2[10],name[20];

again: puts("Please Input Your Name:\n");

gets(name);

if (!strchr(name,'@'))

    {

    puts("Name Must Include '@'!");

    goto again;

    }

for (i=0,sum=0;i<strlen(name);i++) sum+=name[i];

ultoa(276682-sum,temp1,10);

strcat(sn,temp1);

sum=0;i=0;

while(sn[i])

    {

    sum+=sn[i];

    i++;

    }

ultoa(sum,temp2,10);

strcat(sn,temp3);

strcat(sn,temp2);

printf("Your RegCode Is:%s",sn);

}
```
说真的，真希望以后每次都能有这么好的运气！ `4dd466d0e1e0cc123afbe504da2462ca`
