
# 变脸王

> 原文：[`www.pediy.com/kssd/pediy05/pediy50386.htm`](https://www.pediy.com/kssd/pediy05/pediy50386.htm)

这东东都作了半年了，一直没写文章，这次是帮朋友分析才写的，朋友说还算可以，所以才有脸往上放:)

这是某软件后半部分的注册码的实现过程，变脸王的作者注册码后半部分只用了 DES 一种算法!而且是标准 DES，无任何变化:)脱壳后运行变脸王(不脱壳无法 bpx MessageBoxa)，断点设在 MessageBoxa 上，设置两次鼠标主题就可以断下了，显示"确定使用...?"，继续往下走，到这里:

\* Possible StringData Ref from Data Obj ->"holer@21cn.com"

```
:004CCF4A BAC4D44C00             mov edx, 004CD4C4
:004CCF4F 8B45F4                 mov eax, dword ptr [ebp-0C]
:004CCF52 E84964FFFF             call 004C33A0           //那么这里就是逆推注册码的过程了
:004CCF57 8B55CC                 mov edx, dword ptr [ebp-34]
:004CCF5A 8B45F8                 mov eax, dword ptr [ebp-08]
:004CCF5D E82671F3FF             call 00404088           //这里是用来将逆推后的注册码
:004CCF62 7447                   je 004CCFAB             //与输入的名字作比较
:004CCF64 6A40                   push 00000040
```

跟进 4C33A0:

一直到 4C342E 是将输入的码的后半部分转成数字的部分，然后:

```
:004C3430 8B55F8                 mov edx, dword ptr [ebp-08]
:004C3433 8B45F4                 mov eax, dword ptr [ebp-0C]
:004C3436 E8E1FCFFFF             call 004C311C           //这里跟进去研究研究
```

还有:

```
:004C3450 E8C708F4FF             call 00403D1C           //这里也要进去逛逛
:004C3455 C3                     ret
```

进入 4C311C 后会发现他取"holer@21cn.com"前 8 位"holer@21"用来做密钥，后面要有耐心，如想看出算法就要跟进去好几层才可以，从这里进去:

```
:004C319F E848FBFFFF             call 004C2CEC
```

进去走到这里，再进:

```
:004C2D11 E846FEFFFF             call 004C2B5C
```

然后这里，我们可以发现一些有趣的数据:)

```
:004C2B81 BA07000000             mov edx, 00000007
:004C2B86 E8B100F4FF             call 00402C3C
:004C2B8B 33D2                   xor edx, edx            //设 edx 为 i;
:004C2B8D B800874F00             mov eax, 004F8700       //这里我们得到这样一组 16 进制数据
/*PC1="0x38,0x30,0x28,0x20,0x18,0x10,0x08,     0x00,0x39,0x31,0x29,0x21,0x19,0x11,
   0x09,0x01,0x3A,0x32,0x2A,0x22,0x1A,         0x12,0x0A,0x02,0x3B,0x33,0x2B,0x23,
   0x3E,0x36,0x2E,0x26,0x1E,0x16,0x0E,         0x06,0x3D,0x35,0x2D,0x25,0x1D,0x15,
   0x0D,0x05,0x3C,0x34,0x2C,0x24,0x1C,      0x14,0x0C,0x04,0x1B,0x13,0x0B,0x03";
*/
```
将每个数字都加上 1 后再一看就可以想到是 DES 密钥的置换子密钥的参数，有点感觉了，那就继续往下看

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:004C2BE3(C)

:004C2B92 8A18                   mov bl, byte ptr [eax]     //设 eax 为指向这个数组的指针*p
:004C2B94 8BCB                   mov ecx, ebx              //temp=*p
:004C2B96 80E107                 and cl, 07                //以下的操作实质是 bit 级的操作,较为微观:)
:004C2B99 81E1FF000000           and ecx, 000000FF
:004C2B9F 51                     push ecx              //push temp&0x7
:004C2BA0 B907000000             mov ecx, 00000007     //temp&0x7 和 temp  mod 8
:004C2BA5 5E                     pop esi                 //效果一样:)
:004C2BA6 2BCE                   sub ecx, esi              //7-temp&0x7

* Possible Reference to String Resource ID=00001: "http://www.hd1860.com"

:004C2BA8 BE01000000             mov esi, 00000001

:004C2BAD D3E6                   shl esi, cl               //需要将 0x00000001 向左移(7-temp&7)位
:004C2BAF 33C9                   xor ecx, ecx

:004C2BB1 8ACB                   mov cl, bl                //temp=*p
:004C2BB3 C1E903                 shr ecx, 03               //我们知道向右移位相当于除法,这里是整除以 8
:004C2BB6 8B5DFC                 mov ebx, dword ptr [ebp-04]   //这里是"holer@21"
:004C2BB9 0FB60C0B               movzx ecx, byte ptr [ebx+ecx]     //得到第(*p)bit 的值
:004C2BBD 23F1                   and esi, ecx                //用来判断第(*p)bit  的值是否为 0
:004C2BBF 741D                   je 004C2BDE                 //如果是 0 就不用操作了

:004C2BC1 8BCA                   mov ecx, edx                //不为 0 就将第 i 位置 1
:004C2BC3 83E107                 and ecx, 00000007
:004C2BC6 51                     push ecx
:004C2BC7 B907000000             mov ecx, 00000007
:004C2BCC 5B                     pop ebx
:004C2BCD 2BCB                   sub ecx, ebx
:004C2BCF B301                   mov bl, 01
:004C2BD1 D2E3                   shl bl, cl
:004C2BD3 8BCA                   mov ecx, edx
:004C2BD5 C1E903                 shr ecx, 03
:004C2BD8 8B75F8                 mov esi, dword ptr [ebp-08]   //
:004C2BDB 081C0E                 or byte ptr [esi+ecx], bl         //第 i 位置一!

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C2BBF(C)

:004C2BDE 42                     inc edx                         //i++;hoho
:004C2BDF 40                     inc eax
:004C2BE0 83FA38                 cmp edx, 00000038           //计数器为 56 就停止工作
:004C2BE3 75AD                   jne 004C2B92
/*作者是这样来查表的,假设字符串"12345678",2 进制表示"0011 0001......",相当于 2 进制数组 a[8][8],要判断第 i 位是否为 0,那就
先取第 i/8 组,a[i/8][?],然后再来取这个'?'的值,?就是 i mod 8 了,也可以用 i & 0x7 实现,判断的时候,先取得 a[i/8]这 8bit 数据,然后将
0x1 左移 i&0x7bit,就可以通过和 a[i/8]的这 8bit 数据的第 i&0x7 位相与来判断是否为 0,可能有的人一看就明白,不过我一开始弄的挺迷糊
,sigh....所以说明一下
*/
```

上面这一部分看懂那么下面那些置换也能轻松 pass 了，出来之后来到这里:

```
:004C2D16 8A06                   mov al, byte ptr [esi]

...               //这一部分是将置换后得到的 56bit 密钥分成 2 部分,一部分 28bit

:004C2D70 BF10000000             mov edi, 00000010
:004C2D75 BB68874F00             mov ebx, 004F8768       //这里 d 一下看看,
//turnit[]="0x1,0x1,0x2,0x2,0x2,0x2,0x2,0x2,0x1,0x2,0x2,0x2,0x2,0x2,0x2,0x1"
//正好是 DES 的子密钥产生过程需要的每轮旋转数
:004C2D7A 8B75FC                 mov esi, dword ptr [ebp-04]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C2E06(C)

:004C2D7D 8D45F1                 lea eax, dword ptr [ebp-0F]     //把前 28bit 转一转:)
:004C2D80 8A0B                   mov cl, byte ptr [ebx]          //转 turnit[ebx]下,一下一位

* Possible Reference to String Resource ID=00003: "http://www.hd1860.com"

:004C2D82 BA03000000             mov edx, 00000003
:004C2D87 E800FFFFFF             call 004C2C8C           //这个函数负责转圈的
:004C2D8C 8D45ED                 lea eax, dword ptr [ebp-13]     //后 28 也进去转转
:004C2D8F 8A0B                   mov cl, byte ptr [ebx]          //同样的圈数

* Possible Reference to String Resource ID=00003: "http://www.hd1860.com"

:004C2D91 BA03000000             mov edx, 00000003
:004C2D96 E8F1FEFFFF             call 004C2C8C           //还是这破函数:(
:004C2D9B 8A55F1                 mov dl, byte ptr [ebp-0F]
:004C2D9E C1E204                 shl edx, 04
```

圈圈函数在这儿:

```
:004C2C97 8A08                   mov cl, byte ptr [eax]
:004C2C99 03C9                   add ecx, ecx            //用加法实现左移一位的效果
:004C2C9B 33DB                   xor ebx, ebx
:004C2C9D 8A5801                 mov bl, byte ptr [eax+01]   //取下一个字节
:004C2CA0 C1EB07                 shr ebx, 07             //取最高位
:004C2CA3 0ACB                   or cl, bl               //下一字节最高位放入上一字节最低位
:004C2CA5 8808                   mov byte ptr [eax], cl

...                           //这是后 3 个字节的处理,相同所以省略了

:004C2CE0 884803                 mov byte ptr [eax+03], cl
:004C2CE3 80200F                 and byte ptr [eax], 0F        //最后将前 4 位清 0
:004C2CE6 4A                     dec edx                //这个 edx 就是旋转的轮数,是几就转几下
:004C2CE7 75AE                   jne 004C2C97            //感觉这个函数不是很高效
//用#define   ROL(a,n)   (((a)<<(n))|((a)>>(28-(n))))岂不是更好些???
```

继续，转完出去之后:

```
:004C2DF7 BA06000000             mov edx, 00000006
:004C2DFC E8F3FDFFFF             call 004C2BF4     //这里是从 56 取 48 的过程
:004C2E01 83C606                 add esi, 00000006
:004C2E04 43                     inc ebx
```



:004C2E05 `4F` `dec edi` `//要得到 16 个子密钥...`

:004C2E06 `0F8571FFFFFF` `jne 004C2D7D`

:004C2E0C `8B7DD8` `mov edi, dword ptr [ebp-28]`

:004C2E0F `8B75DC` `mov esi, dword ptr [ebp-24]`

进去那个 `call` 里面,会有:

:004C2C23 `33D2` `xor edx, edx`

:004C2C25 `B838874F00` `mov eax, 004F8738` `//这也是个重要的数组`

```
/*"0D 10 0A 17 00 04 02 1B-0E 05 14 09 16 12 0B 03
   19 07 0F 06 1A 13 0C 01-28 33 1E 24 2E 36 1D 27
   32 2C 20 2F 2B 30 26 37-21 34 2D 29 31 23 1C 1F "*/用来从 56 选 48 的...
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:004C2C7B(C)`

:004C2C2A `8A18` `mov bl, byte ptr [eax]`

:004C2C2C `8BCB` `mov ecx, ebx`

:004C2C2E `80E107` `and cl, 07`

`...` `//略掉,选数过程和初始置换相同`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:004C2C57(C)`

:004C2C76 `42` `inc edx`

:004C2C77 `40` `inc eax`

:004C2C78 `83FA30` `cmp edx, 00000030` `//是否全部选完`

:004C2C7B `75AD` `jne 004C2C2A`

出去之后跳过 16 次循环后到上一层:

会发现软件判断注册码是否为 16 的倍数,不是会导致出错...

然后从这里进去:

:004C31FA `E845FDFFFF` `call 004C2F44`

然后到这里进去:

:004C2F82 `E881F9FFFF` `call 004C2908`

这里面就是将明文也就是输入的注册码后半部分,进行初始置换,过程也都是类似的,省略掉,自己看就好啦

:004C2F87 `84DB` `test bl, bl` `//bl 的值不同,下面的流程也不同,我猜是加解密的两个过程:)`

:004C2F89 `0F85B5000000` `jne 004C3044` `//bl=1 解密,bl=0 加密,由于作者设 bl 为定值 1,所以必跳`

:004C3044 `80FB01` `cmp bl, 01`

:004C3047 `0F85B2000000` `jne 004C30FF`

:004C304D `C745F8F0FFFFFF` `mov [ebp-08], FFFFFFF0` `//猜测的依据在这个数据上,这里-16,那里是+16:)`

:004C3054 `BBCA2A5000` `mov ebx, 00502ACA`

`...`

:004C3089 `8BCB` `mov ecx, ebx`

:004C308B `8B450C` `mov eax, dword ptr [ebp+0C]`

:004C308E `8B5508` `mov edx, dword ptr [ebp+08]`

:004C3091 `E886FDFFFF` `call 004C2E1C` `//从这里进去`

里面是:

:004C2E4C `6A05` `push 00000005`

:004C2E4E `8D4DF6` `lea ecx, dword ptr [ebp-0A]`

:004C2E51 `E8BAFBFFFF` `call 004C2A10` `//这里面是 DES 加解密的迭代过程`

:004C2E56 `BB06000000` `mov ebx, 00000006` `//f 函数中 32 变 48 的扩展过程,同上这里略掉不分析`

:004C2E5B `8B45FC` `mov eax, dword ptr [ebp-04]` `//上面 ebx=6!!是因为 48bit,每次异或一个字节!`

:004C2E5E `8D55F6` `lea edx, dword ptr [ebp-0A]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:004C2E68(C)`

:004C2E61 `8A08` `mov cl, byte ptr [eax]`

:004C2E63 `300A` `xor byte ptr [edx], cl` `//膨胀后与子密钥相异或`

:004C2E65 `42` `inc edx`

:004C2E66 `40` `inc eax`

:004C2E67 `4B` `dec ebx`

:004C2E68 `75F7` `jne 004C2E61`

:004C2E6A `8A45F6` `mov al, byte ptr [ebp-0A]`

`...` `//以下是将异或后得到的数修改,因为得到的数是连贯的 48bit,所以要将他们分开成为 6bit 一字节`

:004C2EE6 `8BC3` `mov eax, ebx`

:004C2EE8 `8A16` `mov dl, byte ptr [esi]`

:004C2EEA `E831FCFFFF` `call 004C2B20` `//这里面是将上面得到那 8 个字节调整用来查表,(每 6 个用来查一个表)`

:004C2EEF `8806` `mov byte ptr [esi], al`

:004C2EF1 `43` `inc ebx`

:004C2EF2 `46` `inc esi`

:004C2EF3 `83FB08` `cmp ebx, 00000008` `//查 8 次...`

:004C2EF6 `75EE` `jne 004C2EE6`

然后再出来,好累啊.......坚持就是胜利,这里快结束了

:004C3096 `B804000000` `mov eax, 00000004`

:004C309B `8D55F4` `lea edx, dword ptr [ebp-0C]`

:004C309E `8D75F0` `lea esi, dword ptr [ebp-10]`

:004C30A1 `8B4D0C` `mov ecx, dword ptr [ebp+0C]`

:004C30A4 `83C104` `add ecx, 00000004`

:004C30A7 `894DE8` `mov dword ptr [ebp-18], ecx`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:004C30B9(C)`

:004C30AA `8A0A` `mov cl, byte ptr [edx]` `//经过 f 函数作用后,左半部分`

:004C30AC `320E` `xor cl, byte ptr [esi]` `//和右半部分异或得到下一轮的右半部分`

:004C30AE `8B7DE8` `mov edi, dword ptr [ebp-18]`

:004C30B1 `880F` `mov byte ptr [edi], cl`

:004C30B3 `FF45E8` `inc [ebp-18]`

:004C30B6 `46` `inc esi`

:004C30B7 `42` `inc edx`

:004C30B8 `48` `dec eax`

:004C30B9 `75EF` `jne 004C30AA` `//每字节异或...`

:004C30BB `83EB06` `sub ebx, 00000006`

:004C30BE `FF45F8` `inc [ebp-08]` `//这是调整子密钥,每轮用的子密钥不同`

:004C30C1 `7596` `jne 004C3059`

下面还有一个函数是用来将得到的最后结果做初始置换的逆置换...

:004C3105 `E882F8FFFF` `call 004C298C` `//不看了,全都一样的取数方式:(`

就这样,这就是一个标准 DES 的运算过程,不知道能不能看懂,我讲的可真是够累,好长好长............睡觉去啦 `4dd466d0e1e0cc123afbe504da2462ca`
