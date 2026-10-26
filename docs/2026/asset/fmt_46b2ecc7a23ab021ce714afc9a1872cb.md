
# [原创]010Edit10.0爆破+算法分析+注册机实现

## 11.0版本

### 爆破

假码`pName = xuanci`，`pKey = 00112233445566778899`

![](upload/attach/202010/846211\_EDARTBHQWHNZQRQ.png)

登录出错，复制错误信息：

```sql
---------------------------
010 Editor
---------------------------
Invalid name or password. Please enter your name and password exactly as given when you purchased 010 Editor (make sure no quotes are included).
---------------------------
&OK
---------------------------
```

字符串搜索下，看看能不能搜索到相关信息：

![](upload/attach/202010/846211\_TQR46JDDFUJPA6W.png)

也可以通过`CreateWindowExW`回溯找到，看自己来：

![](upload/attach/202010/846211\_CEV9768EHEHC9G5.jpg)

这里看了下不让他跳，也没见到登录成功的地方，那我们一步一步跟上去看看：

![](upload/attach/202010/846211\_Y9FMY8MDC9SX8KC.jpg)

![](upload/attach/202010/846211\_5K6FA7CQSRSH5K4.jpg)

可以看出这里成功了，那么是不是我们在上面`NOP`掉直接就能成功？奔着这个思想，我们来验证下：

![](upload/attach/202010/846211\_SWZVRNDTQXTXG49.jpg)

好像真的可以了，那我们就来看看到底做了些什么可以让他走过这个地方：

![](upload/attach/202010/846211\_N5VT36DBH9T4HE8.jpg)

```cpp
002A1D30 > \81FB DB000000 cmp ebx, 0xDB ; 根据这条汇编指令我们知道 ebx == 0xDB即可成功
002A1C40 . E8 747BE5FF call 000F97B9 ; eax == 0xDB即可成功
002A1C45 . 8B0D 7C257000 mov ecx, dword ptr [0x70257C]
002A1C4B . 8BD8 mov ebx, eax ; 可以看到ebx == eax
002A1C4D . 8B45 E0 mov eax, dword ptr [ebp-0x20]
002A1C50 . 3D E7000000 cmp eax, 0xE7
```

跳跟踪到上面我们可以找到`ebx`来自于`eax`，`eax == call 000F97B9`，`eax == 0xDB`就能调到成功了吗？我们看看他什么情况下恒与`0xDB`。

进`CALL`最下面，我们可以看到：

```cpp
003D5B99 |> \5F pop edi ; Case 2D of switch 003D5AF3
003D5B9A |. B8 DB000000 mov eax, 0xDB
003D5B9F |. 5E pop esi
003D5BA0 |. 5D pop ebp
003D5BA1 \. C2 0800 retn 0x8
```

`003D5B99`又是`003D5AF6 |. /0F84 9D000000 je 003D5B99`跳转来的，我们只需要让`je`满足条件即可：

```cpp
003D5AEE |. E8 DDC1D1FF call 000F1CD0
003D5AF3 |. 83F8 2D cmp eax, 0x2D ; Switch (cases 2D..E7)
003D5AF6 |. /0F84 9D000000 je 003D5B99
```

`call 000F1CD0`返回值恒与`0x2D`即可了。那么我们进`CALL`，给他：

```cpp
mov eax,0x2D
ret 8
```

![](upload/attach/202010/846211\_QMU8SGPRYHJ5DDV.jpg)

注意这里涉及了重定位，那我们先把随机基址给去除吧。

可以先打开我们刚刚爆破的那个`010Edit`，然后把没修改的拖进去，去修改随机基址：

![](upload/attach/202010/846211\_9HQJADHHCNV4Z49.jpg)

按`Install`安装下模板，安装好我们就去找`dll`标志的字段：

![](upload/attach/202010/846211\_EZJT3VYMWPXWAKX.jpg)

这里我们改成`0`，`Ctrl+s`保存即可。

这时候我们再回去找到原来的地方，给他修改汇编代码即可：

![](upload/attach/202010/846211\_8F55FXC2GCU536J.jpg)

![](upload/attach/202010/846211\_J33M6NYXGVT8NFS.jpg)

提示我这个，其实按正常来说应该是可以`OK`的，但是这里多了个网络验证，我们就得把网络验证去掉。

我们`OD`载入刚修改好的，然后在刚刚修改的地方下段即可，单步跟踪：

![](upload/attach/202010/846211\_RMXZ3WWZK3ZABJB.jpg)

我们继续往下找是否还有错误的地方：

![](upload/attach/202010/846211\_WRBD7K4AM7945DF.jpg)

![](upload/attach/202010/846211\_TP5RE4FN8VDSWZK.jpg)

![](upload/attach/202010/846211\_KN4WWHWAX6NMZN4.jpg)

重新来过：

![](upload/attach/202010/846211\_ESPAMMXZHQ5MCJ5.jpg)

这里给他置`1`，让他的`ZF`标志位为`0`：

![](upload/attach/202010/846211\_GZCVKQKPU7JYZ39.jpg)

成功。这里我们有很多种方法，可以爆破，可以使用：

```cpp
005B1CEA > \75 27 jnz short 005B1D13
改成
005B1CEA > \75 27 jz short 005B1D13
也可以根据上面爆破0x2D一样CALL里返回值让他恒为1
005B3460 B8 01000000 mov eax, 0x1
005B3465 C2 0400 retn 0x4
005B3468 90 nop
005B3469 90 nop
这里随便就看自己了
```

### 算法分析

综上所诉，我们得知那个控制`0x2D`的`CALL`就是我们需要分析的地方，只要他`== 0x2D`我们就可以过掉本地验证。分析出算法，我们就可以写出注册机。

至于网络验证，我们就需要自己去打补丁也好，怎么搞也好，就要看自己了。

那么我们来到返回`0x2D`的地方进行分析吧：

```cpp
005B1C1E . 8B0D 7C25A100 mov ecx, dword ptr [0xA1257C]
005B1C24 . 68 67480000 push 0x4867
005B1C29 . 6A 0C push 0xC
005B1C2B . E8 A000E5FF call 00401CD0
005B1C30 . 8B0D 7C25A100 mov ecx, dword ptr [0xA1257C]
```

首先看下参数，一个`0xC`，`0x4867`，`ecx`我们去看下内存，看看到底是什么：

![](upload/attach/202010/846211\_MZ26ZY38B7BR8SS.jpg)

看着是个地址，那么我们跟进去过去看看到底都是什么：

![](upload/attach/202010/846211\_AXDATPYH7CVBBGB.jpg)

我们在`ECX + 4`的地址哪里看到了用户名：

![](upload/attach/202010/846211\_7VJ29MFTSG4QW5D.jpg)

在`ECX + 8`的地址哪里看到了`KEY`。

那么我们可以猜测这里是不是一个对象，里面存放着账号密码`UNICODE`类型的，还有一些我们不知道的值，具体用到再去分析。

我们进`CALL`，去逐步分析：

```cpp
006E5229 > \8A45 E7 mov al, byte ptr [ebp-0x19] ; al == k[3]
006E522C . 3C 9C cmp al, 0x9C ; 这里判断K[3]是否等于0x9C; Switch (cases 9C..FC)
006E522E . 75 70 jnz short 006E52A0
```

这里`K[3] != 0x9C`就继续判断：

```cpp
006E52A0 > \3C FC cmp al, 0xFC
006E52A2 . 75 1F jnz short 006E52C3
```

判断是否等于`0xFC`，不等于再继续判断：

```cpp
006E52C3 > \3C AC cmp al, 0xAC
006E52C5 . 0F85 94010000 jnz 006E545F
```

这里再判断是否等于`0xAC`，再不行就给`EAX == 0xE7`，那么我们想获取的是`0x2D`，这里获取`E7`肯定不对。

这样我们就可以猜测`K[3] == 0x9C / 0xFC / 0xAC`。

分析一段逻辑，进行代码测试：

```cpp
006E5229 > \8A45 E7 mov al, byte ptr [ebp-0x19] ; al == k[3]
006E522C . 3C 9C cmp al, 0x9C ; 这里判断K[3]是否等于0x9C; Switch (cases 9C..FC)
006E522E . 75 70 jnz short 006E52A0
006E5230 . 8A45 E4 mov al, byte ptr [ebp-0x1C] ; al == K[0]; Case 9C of switch 006E522C
006E5233 . 3245 EA xor al, byte ptr [ebp-0x16] ; al == k[0]^k[6]
006E5236 . 8845 DC mov byte ptr [ebp-0x24], al
006E5239 . 8A45 E5 mov al, byte ptr [ebp-0x1B] ; al == k[1]
006E523C . 3245 EB xor al, byte ptr [ebp-0x15] ; al == k[1]^k[7]
006E523F . FF75 DC push dword ptr [ebp-0x24] ; push al == k[0]^k[6]
006E5242 . 0FB6C8 movzx ecx, al ; ecx == (k[1]^k[7])&0xFF
006E5245 . B8 00010000 mov eax, 0x100 ; eax == 0x100
006E524A . 0FAFC8 imul ecx, eax ; ecx = ((k[1]^k[7])&0xFF)\*0x100
006E524D . 8A45 E6 mov al, byte ptr [ebp-0x1A] ; al == k[2]
006E5250 . 3245 E9 xor al, byte ptr [ebp-0x17] ; al == k[2]^k[5]
006E5253 . 0FB6C0 movzx eax, al ; eax == (k[2]^k[5])&0xFF
006E5256 . 66:03C8 add cx, ax ; cx = (((k[1]^k[7])&0xFF)\*0x100)+((k[2]^k[5])&0xFF)
006E5259 . 0FB7F1 movzx esi, cx ; esi = (((k[1]^k[7])&0xFF)\*0x100)+((k[2]^k[5])&0xFF)&0xFFFF
006E525C . E8 5309D2FF call 00405BB4 ; 处理K[0]和K[6] al == ((k[0]^k[6])^0x18 + 0x3D)^0xA7
006E5261 . 0FB6C0 movzx eax, al ; eax == (((k[0]^k[6])^0x18 + 0x3D)^0xA7)&0xFF
006E5264 . 56 push esi ; push esi = (((k[1]^k[7])&0xFF)\*0x100)+((k[2]^k[5])&0xFF)&0xFFFF
006E5265 . 8943 1C mov dword ptr [ebx+0x1C], eax ; 下面CALL 返回 eax = ((((((k[1]^k[7])&0xFF)\*0x100)+((k[2]^k[5])&0xFF)&0xFFFF)^0x7892 + 0x4D30)&0xFFFF)^0x3421
006E5268 . E8 D84FD2FF call 0040A245 ; 计算的值和0xB做除法 余数不为0则返回0 余数为0则返回商
006E526D . 8B4B 1C mov ecx, dword ptr [ebx+0x1C] ; ecx == (((k[0]^k[6])^0x18 + 0x3D)^0xA7)&0xFF
006E5270 . 83C4 08 add esp, 0x8
006E5273 . 0FB7C0 movzx eax, ax ; eax = ((((((((k[1]^k[7])&0xFF)\*0x100)+((k[2]^k[5])&0xFF)&0xFFFF)^0x7892 + 0x4D30)&0xFFFF)^0x3421) / 0xB) &0xFFFF
006E5276 . 8943 20 mov dword ptr [ebx+0x20], eax
006E5279 . 85C9 test ecx, ecx ; 判断 (((k[0]^k[6])^0x18 + 0x3D)^0xA7)&0xFF != 0
006E527B . 0F84 DE010000 je 006E545F
006E5281 . 85C0 test eax, eax ; 判断 eax = (((((((k[1]^k[7])&0xFF)\*0x100)+((k[2]^k[5])&0xFF)&0xFF)^0x7892 + 0x4D30)&0xFFFF)^0x3421)&0xFFFF != 0
006E5283 . 0F84 D6010000 je 006E545F
006E5289 . 3D E8030000 cmp eax, 0x3E8 ; 且 eax < 0x3E8
006E528E . 0F87 CB010000 ja 006E545F
```

可以看到，最后就是比较：

判断`(((k[0]^k[6])^0x18 + 0x3D)^0xA7)&0xFF != 0`

判断`eax = ((((((((k[1]^k[7])&0xFF)\*0x100)+((k[2]^k[5])&0xFF)&0xFFFF)^0x7892 + 0x4D30)&0xFFFF)^0x3421) / 0xB) &0xFFFF != 0`

判断`eax = ((((((((k[1]^k[7])&0xFF)\*0x100)+((k[2]^k[5])&0xFF)&0xFFFF)^0x7892 + 0x4D30)&0xFFFF)^0x3421) / 0xB) &0xFFFF < 0x3E8`

已知`K[3] == 0x9C / 0xFC / 0xAC`。

那么我们就开始写代码测试吧：

```cpp
#include <stdio.h>
#include <windows.h>
#include <time.h>
int main()
{
/* 
判断 (((k[0]^k[6])^0x18 + 0x3D)^0xA7)&0xFF != 0
判断 eax = ((((((((k[1]^k[7])&0xFF)\*0x100)+((k[2]^k[5])&0xFF)&0xFFFF)^0x7892 + 0x4D30)&0xFFFF)^0x3421) / 0xB) &0xFFFF != 0
判断 eax = ((((((((k[1]^k[7])&0xFF)\*0x100)+((k[2]^k[5])&0xFF)&0xFFFF)^0x7892 + 0x4D30)&0xFFFF)^0x3421) / 0xB) &0xFFFF < 0x3E8
已知K[3] == 0x9C / 0xFC / 0xAC
*/
byte bKey[10] = { 0 };
bKey[3] = 0x9C;
srand(time(NULL));
while (true)
{
byte k0 = rand() % 0xFF;
byte k6 = rand() % 0xFF;
byte s1 = (((k0 ^ k6) ^ 0x18 + 0x3D) ^ 0xA7) & 0xFF;
if (s1 != 0)
{
//获取到了K0 和 K6
bKey[0] = k0;
bKey[6] = k6;
break;
}
}
while (true)
{
byte k1 = rand() % 0xFF;
byte k7 = rand() % 0xFF;
byte k2 = rand() % 0xFF;
byte k5 = rand() % 0xFF;
DWORD s1 = (((((((k1 ^ k7) & 0xFF) * 0x100) + ((k2 ^ k5) & 0xFF) & 0xFFFF) ^ 0x7892) + 0x4D30) ^ 0x3421) & 0xFFFF;
if ((s1 % 0xB) == 0 && (s1 / 0xB) < 0x3E8)
{
bKey[1] = k1;
bKey[7] = k7;
bKey[2] = k2;
bKey[5] = k5;
break;
}
}
for (int i = 0; i < 10; i++)
{
printf("%02X", bKey[i]);
}
getchar();
return 0;
}
```

![](upload/attach/202010/846211\_3TJRNYNCTEE2SAH.jpg)

得出`KEY`：`53BDF79C000D8D1F`

我们输入`KEY`重新来过，看看能不能到达我们想到达的地方。
