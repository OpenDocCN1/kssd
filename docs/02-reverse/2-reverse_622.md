
# 回朔找内存地址

> 原文：[`www.pediy.com/kssd/pediy06/pediy6812.htm`](https://www.pediy.com/kssd/pediy06/pediy6812.htm)

双休日，干什么呢？闲着还不如恶搞一下（心理变态）。

把中国游戏中心的象棋拿来看看，决定破之而后快。

目的，找出表示象棋棋子的内存地址。

象棋棋子的表示和其他游戏里的经验值生命值不同，经验值和生命值是用数值表示的，在内存中用值查找就 OK 了，但是棋子呢，车用什么表示？没人知道。

首先，对软件进行资源分析，发现棋子的在资源位图中的顺序是车马炮帅士象兵，大概猜测知道（红或黑）车~兵是用 `0~6` 表示。但是内存中是 `0~6` 的单元多如牛毛，而且还不确定值的存放长度和确切值，想在内存中找这些数简直是大海捞针。

要怎么办呢，有一点 WINDOWS 图象编程的人都知道，要最终画出图象，多半要用到 `BITBLT` 这个 API，如果知道贴棋子时的 `BITBLT` 参数就可以用回朔法找到棋子信息内存的指针。什么？在 `BITBLT` 上下断点就行了？那是不可能的，因为，`BITBLT` 这个 API 的应用率太高，你的 `SOFTICE` 会一直中断....

于是在 `W32DASM` 中找 `BITBLT` 函数，找到两个：

```
:0041A630 55                     push ebp
:0041A631 8BEC                   mov ebp, esp
:0041A633 51                     push ecx
:0041A634 894DFC                 mov dword ptr [ebp-04], ecx
:0041A637 8B4524                 mov eax, dword ptr [ebp+24]
:0041A63A 50                     push eax
:0041A63B 8B4D20                 mov ecx, dword ptr [ebp+20]
:0041A63E 51                     push ecx
:0041A63F 8B551C                 mov edx, dword ptr [ebp+1C]
:0041A642 52                     push edx
:0041A643 8B4D18                 mov ecx, dword ptr [ebp+18]
:0041A646 E835FFFFFF             call 0041A580
:0041A64B 50                     push eax
:0041A64C 8B4514                 mov eax, dword ptr [ebp+14]
:0041A64F 50                     push eax
:0041A650 8B4D10                 mov ecx, dword ptr [ebp+10]
:0041A653 51                     push ecx
:0041A654 8B550C                 mov edx, dword ptr [ebp+0C]
:0041A657 52                     push edx
:0041A658 8B4508                 mov eax, dword ptr [ebp+08]
:0041A65B 50                     push eax
:0041A65C 8B4DFC                 mov ecx, dword ptr [ebp-04]
:0041A65F 8B5104                 mov edx, dword ptr [ecx+04]
:0041A662 52                     push edx

* Reference To: GDI32.BitBlt, Ord:0011h
                        |
:0041A663 FF15C4304600         Call dword ptr [004630C4]
:0041A669 8BE5                 mov esp, ebp
:0041A66B 5D                   pop ebp
:0041A66C C22000               ret 0020
```

和

```
:00448E78 2B442450             sub eax, dword ptr [esp+50]
:00448E7C 8B4C2450             mov ecx, dword ptr [esp+50]
:00448E80 83E80D               sub eax, 0000000D
:00448E83 99                   cdq
:00448E84 2BC2                 sub eax, edx
:00448E86 C1F801               sar eax, 01
:00448E89 03C8                 add ecx, eax
:00448E8B 8B44244C             mov eax, dword ptr [esp+4C]
:00448E8F 51                   push ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00448D01(U)

|
:00448E90 50                   push eax
:00448E91 56                   push esi

* Reference To: GDI32.BitBlt, Ord:0011h
                        |
:00448E92 FF15C4304600         Call dword ptr [004630C4]
:00448E98 53                   push ebx
:00448E99 57                   push edi
```

我们要的是初始化时绘棋子的那个，是哪个呢？

`BITBLT` 的第 8 个参数是 `RASTER` 运算值，改成 `00000000` 的话就是黑的，在 `HIEW` 中把它们运算值改了，发现改第一个的时候全都黑了，于是断定是第一个。

回到第一个看：

```
:0041A630 55                     push ebp
:0041A631 8BEC                   mov ebp, esp
:0041A633 51                     push ecx
:0041A634 894DFC                 mov dword ptr [ebp-04], ecx
:0041A637 8B4524                 mov eax, dword ptr [ebp+24]
:0041A63A 50                     push eax
:0041A63B 8B4D20                 mov ecx, dword ptr [ebp+20]
:0041A63E 51                     push ecx（8）
:0041A63F 8B551C                 mov edx, dword ptr [ebp+1C]
:0041A642 52                     push edx（7）
:0041A643 8B4D18                 mov ecx, dword ptr [ebp+18]
:0041A646 E835FFFFFF             call 0041A580
:0041A64B 50                     push eax（6）
:0041A64C 8B4514                 mov eax, dword ptr [ebp+14]
:0041A64F 50                     push eax（5）
:0041A650 8B4D10                 mov ecx, dword ptr [ebp+10]
:0041A653 51                     push ecx（4）
:0041A654 8B550C                 mov edx, dword ptr [ebp+0C]
:0041A657 52                     push edx（3）
:0041A658 8B4508                 mov eax, dword ptr [ebp+08]
:0041A65B 50                     push eax（2）
:0041A65C 8B4DFC                 mov ecx, dword ptr [ebp-04]
:0041A65F 8B5104                 mov edx, dword ptr [ecx+04]
:0041A662 52                     push edx（第一个参数）

* Reference To: GDI32.BitBlt, Ord:0011h
                        |
:0041A663 FF15C4304600         Call dword ptr [004630C4]
:0041A669 8BE5                 mov esp, ebp
:0041A66B 5D                   pop ebp
:0041A66C C22000               ret 0020
```

看 CALL 之前的 PUSH，倒数 8 个，八个参数都出来了

于是看出倒数第 3 个正是来源位图的左上角坐标

知道了这个就可以知道是什么棋子了，因为棋子的位图

是通过在一张有所有棋子的位图上截取的。

然后开始回朔，回到上一级调用他的地方

```
* Referenced by a CALL at Addresses:

|:00419FBE   , :0041A13C   , :0041A1C3   , :0041A1EF   , :0041A273

|:0041A29F   , :0041A2C8   , :0041A2F1   , :0041B1AC   , :0041F89C

|:0041FB46   , :00420018   , :0042296B   , :00422A25   , :00422B98

|:00422DEA   , :00422E13   , :00422E38   , :00422E61   , :004230C6

|:00431B57   , :004320F8   , :00432121   , :00432146   , :0043216F
```

这么多！！

不过看看地址可以看出很多地方是同一个函数调用的，其

实只有５～６个函数调用了它，于是回朔看看，到达这里：

```
:00422E3D 6846006600           push 00660046；８（Raster 运算值）
:00422E42 8B5524               mov edx, dword ptr [ebp+24]
:00422E45 52                   push edx；７
:00422E46 8B4520               mov eax, dword ptr [ebp+20]
:00422E49 50                   push eax；６
:00422E4A 8B4D1C               mov ecx, dword ptr [ebp+1C]
:00422E4D 51                   push ecx；５
:00422E4E 8B5518               mov edx, dword ptr [ebp+18]
:00422E51 52                   push edx；４
:00422E52 8B4514               mov eax, dword ptr [ebp+14]
:00422E55 50                   push eax；３
:00422E56 8B4D10               mov ecx, dword ptr [ebp+10]
:00422E59 51                   push ecx；２
:00422E5A 8B550C               mov edx, dword ptr [ebp+0C]
:00422E5D 52                   push edx；第一个参数
:00422E5E 8B4D08               mov ecx, dword ptr [ebp+08]
:00422E61 E8CA77FFFF           call 0041A630
```

看到没这个就是初始化时调用贴棋子的地方！

验证一下，把 Raster 运算值改成０，结果，棋子全变黑了。

然后再回朔，为什么不在这里断掉找棋子的内存地址呢？

因为还没看见坐标变换的运算啊，因为从拿到棋子参数后

到贴图必须经过对棋子逻辑坐标到显示器的位置的坐标变换

才能贴图的。

于是继续回蒴，到了这里：

```
:00422BF9 68D6A54A00           push 004AA5D6
:00422BFE 6A00                 push 00000000
:00422C00 8B550C               mov edx, dword ptr [ebp+0C]
:00422C03 81E2FF000000         and edx, 000000FF
:00422C09 8B45C0               mov eax, dword ptr [ebp-40]
:00422C0C 0FAF9043010000       imul edx, dword ptr [eax+00000143]
:00422C13 52                   push edx
:00422C14 8D4DC8               lea ecx, dword ptr [ebp-38]
:00422C17 51                   push ecx
:00422C18 8B55C0               mov edx, dword ptr [ebp-40]
:00422C1B 8B8247010000         mov eax, dword ptr [edx+00000147]
:00422C21 50                   push eax
:00422C22 8B4DC0               mov ecx, dword ptr [ebp-40]
:00422C25 8B9143010000         mov edx, dword ptr [ecx+00000143]
:00422C2B 52                   push edx
:00422C2C 8B45C0               mov eax, dword ptr [ebp-40]
:00422C2F 8B4D14               mov ecx, dword ptr [ebp+14]
:00422C32 0FAF883F010000       imul ecx, dword ptr [eax+0000013F]
:00422C39 8B55C0               mov edx, dword ptr [ebp-40]
:00422C3C 8B8237010000         mov eax, dword ptr [edx+00000137]
```



```
:00422C42 03C1                   add eax, ecx
:00422C44 50                     push eax
:00422C45 8B4DC0                 mov ecx, dword ptr [ebp-40]
:00422C48 8B5510                 mov edx, dword ptr [ebp+10]
:00422C4B 0FAF913B010000         imul edx, dword ptr [ecx+0000013B]
:00422C52 8B45C0                 mov eax, dword ptr [ebp-40]
:00422C55 8B8833010000           mov ecx, dword ptr [eax+00000133]
:00422C5B 03CA                   add ecx, edx
:00422C5D 51                     push ecx
:00422C5E 8B5508                 mov edx, dword ptr [ebp+08]
:00422C61 52                     push edx
:00422C62 8B4DC0                 mov ecx, dword ptr [ebp-40]
:00422C65 E8D5000000             call 00422D3F
```

看到没，坐标变换运算：

```
:00422C2C 8B45C0                 mov eax, dword ptr [ebp-40]
:00422C2F 8B4D14                 mov ecx, dword ptr [ebp+14]
:00422C32 0FAF883F010000         imul ecx, dword ptr [eax+0000013F]
:00422C39 8B55C0                 mov edx, dword ptr [ebp-40]
:00422C3C 8B8237010000           mov eax, dword ptr [edx+00000137]
:00422C42 03C1                   add eax, ecx
```

`[eax+0000013F]` 是放棋子位图宽的地方， `[[ebp-40]+00000137]` 又是什么呢？是左上角的棋子到棋盘边缘的距离（y 轴）。

找到最重要的 `BITBLT` 第 5 个的寻址：

```
:00422C00 8B550C                 mov edx, dword ptr [ebp+0C]
:00422C03 81E2FF000000           and edx, 000000FF
:00422C09 8B45C0                 mov eax, dword ptr [ebp-40]
:00422C0C 0FAF9043010000         imul edx, dword ptr [eax+00000143]
:00422C13 52                     push edx（第 5 个参数）
```

咦？怎么要先和 `000000FF` `AND` 呢?难道..(后面再说)

不管了,先记下寻址:

```
mov edx, dword ptr [ebp+0C]
```

然后再一路反跳转回到这里：

`* Referenced by a CALL at Addresses:`

`:004229F4 , :00422F78 , :00423193 , :0042323B , :00423FC4`

`:00424337 , :004246E1 , :0042483A , :004249D6 , :004271D8`

`:004272F3 , :00427316 , :00428444 , :0042947E , :004297A2`

`:004297E5 , :00429A6E , :00429A92`

```
:00422A8F 55                     push ebp
:00422A90 8BEC                   mov ebp, esp
:00422A92 6AFF                   push FFFFFFFF
:00422A94 685B094600             push 0046095B
:00422A99 64A100000000           mov eax, dword ptr fs:[00000000]
:00422A9F 50                     push eax
:00422AA0 64892500000000         mov dword ptr fs:[00000000], esp
:00422AA7 83EC34                 sub esp, 00000034
:00422AAA 894DC0                 mov dword ptr [ebp-40], ecx
:00422AAD 837D0800               cmp dword ptr [ebp+08], 00000000
:00422AB1 7505                   jne 00422AB8
:00422AB3 E977020000             jmp 00422D2F
```

还有这么多？回朔哪个？

没关系了，因为如果再回朔一次的话很可能已经可以找到读取棋子参数的内存的指令了。（为什么？因为已经是时候了，该做的都做了，还不是读棋子参数是什么）

于是从第一个回蒴，到达：

```
:004229D2 8B55B8                 mov edx, dword ptr [ebp-48]
:004229D5 52                     push edx
:004229D6 8B45E8                 mov eax, dword ptr [ebp-18]
:004229D9 50                     push eax
:004229DA 8B4DE8                 mov ecx, dword ptr [ebp-18]
:004229DD 6BC90A                 imul ecx, 0000000A
:004229E0 8B55B8                 mov edx, dword ptr [ebp-48]
:004229E3 03D1                   add edx, ecx
:004229E5 8B45B4                 mov eax, dword ptr [ebp-4C]
:004229E8 8A4C1048               mov cl, byte ptr [eax+edx+48]
:004229EC 51                     push ecx
:004229ED 8D55D0                 lea edx, dword ptr [ebp-30]
:004229F0 52                     push edx
:004229F1 8B4DB4                 mov ecx, dword ptr [ebp-4C]
:004229F4 E896000000             call 00422A8F
```

因为上一步已经记下了最重要的寻址:

`EBP + C`

于是只要回数 2 个 `PUSH` 就行了,由于 `CALL` 进去后还 `PUSH` 了一个 `EBP`

```
:004229DA 8B4DE8                 mov ecx, dword ptr [ebp-18]
:004229DD 6BC90A                 imul ecx, 0000000A
:004229E0 8B55B8                 mov edx, dword ptr [ebp-48]
:004229E3 03D1                   add edx, ecx
:004229E5 8B45B4                 mov eax, dword ptr [ebp-4C]
:004229E8 8A4C1048               mov cl, byte ptr [eax+edx+48]
:004229EC 51                     push ecx
```

有点复杂哦,不过 `CRACK` 就必须有耐心,耐心看完后知道:

棋子类型描述单元的寻址是:

`[[ebp - 4c] + [ebp - 48] + [ebp - 18] * 11 + 48]`

这么复杂是正常的,想想看,这么多棋子,肯定要分几步才能找到了

现在怎么办呢,如果能在这里设个短点就可以知道 `ebp` 的值从而找到棋子的单元了.

想一想,只要读不能访问的内存单元就会发生访存错误,程序会被 `BREAK`

呵呵,只要把其中一个语句改成非法的访存语句就 `OK` 了,偶的系统是 `2000`,读写小于 `0x00010000` 的内存是不允许的，会抛出异常:`xxxx` 指令的 `xxxxx` 内存不能为读,于是决定虐待一下它,

用 `HIEW` 把 `mov ecx, dword ptr [ebp-4C]` 改成 `mov ecx, dword ptr [eax-4C]`,运行它!

点开始,生成棋盘棋子---当!`WINDOWS` 的错误框预期出现在面前!

马上对它进行调试,看 `ebp` 的内容,代入 `[[ebp - 4c] + [ebp - 48] + [ebp - 18] * 11 + 48]`

算出地址:`0x016a4f00`(这个值每台电脑可能不同)

于是一切搞顶,现在只要找到初始化棋子时的语句就可以了(可以什么?当然是胡作非为了,想怎么改不行!)

于是在 `0x016a4f00` 上下断点,然后运行象棋...

`SOFTICE` 弹出!

一眼就发现初始化棋子的语句了:

```
:004225AE 8B55FC                 mov edx, dword ptr [ebp-04]
:004225B1 C6424807               mov [edx+48], 07
:004225B5 8B45FC                 mov eax, dword ptr [ebp-04]
:004225B8 C6405208               mov [eax+52], 08
:004225BC 8B4DFC                 mov ecx, dword ptr [ebp-04]
:004225BF C6415C0C               mov [ecx+5C], 0C
:004225C3 8B55FC                 mov edx, dword ptr [ebp-04]
:004225C6 C642660B               mov [edx+66], 0B
:004225CA 8B45FC                 mov eax, dword ptr [ebp-04]
:004225CD C640700A               mov [eax+70], 0A
:004225D1 8B4DFC                 mov ecx, dword ptr [ebp-04]
:004225D4 C6417A0B               mov [ecx+7A], 0B
:004225D8 8B55FC                 mov edx, dword ptr [ebp-04]
:004225DB C682840000000C         mov byte ptr [edx+00000084], 0C
:004225E2 8B45FC                 mov eax, dword ptr [ebp-04]
:004225E5 C6808E00000008         mov byte ptr [eax+0000008E], 08
:004225EC 8B4DFC                 mov ecx, dword ptr [ebp-04]
:004225EF C6819800000007         mov byte ptr [ecx+00000098], 07
:004225F6 8B55FC                 mov edx, dword ptr [ebp-04]
:004225F9 C6425409               mov [edx+54], 09
:004225FD 8B45FC                 mov eax, dword ptr [ebp-04]
:00422600 C6809000000009         mov byte ptr [eax+00000090], 09
:00422607 8B4DFC                 mov ecx, dword ptr [ebp-04]
:0042260A C6414B0D               mov [ecx+4B], 0D
:0042260E 8B55FC                 mov edx, dword ptr [ebp-04]
:00422611 C6425F0D               mov [edx+5F], 0D
:00422615 8B45FC                 mov eax, dword ptr [ebp-04]
:00422618 C640730D               mov [eax+73], 0D
:0042261C 8B4DFC                 mov ecx, dword ptr [ebp-04]
:0042261F C681870000000D         mov byte ptr [ecx+00000087], 0D
:00422626 8B55FC                 mov edx, dword ptr [ebp-04]
:00422629 C6829B0000000D         mov byte ptr [edx+0000009B], 0D
:00422630 8B45FC                 mov eax, dword ptr [ebp-04]
:00422633 C6404E06               mov [eax+4E], 06
:00422637 8B4DFC                 mov ecx, dword ptr [ebp-04]
:0042263A C6416206               mov [ecx+62], 06
:0042263E 8B55FC                 mov edx, dword ptr [ebp-04]
:00422641 C6427500               mov [edx+75], 00
:00422645 8B45FC                 mov eax, dword ptr [ebp-04]
:00422648 C6808A00000006         mov byte ptr [eax+0000008A], 06
:0042264F 8B4DFC                 mov ecx, dword ptr [ebp-04]
:00422652 C6819E00000006         mov byte ptr [ecx+0000009E], 06
:00422659 8B55FC                 mov edx, dword ptr [ebp-04]
:0042265C C6425902               mov [edx+59], 02
:00422660 8B45FC                 mov eax, dword ptr [ebp-04]
:00422663 C6809500000002         mov byte ptr [eax+00000095], 02
:0042266A 8B4DFC                 mov ecx, dword ptr [ebp-04]
```



```
:0042266D C6415100          mov [ecx+51], 00  
:00422671 8B55FC           mov edx, dword ptr [ebp-04]  
:00422674 C6425B01         mov [edx+5B], 01  
:00422678 8B45FC           mov eax, dword ptr [ebp-04]  
:0042267B C6406505         mov [eax+65], 05  
:0042267F 8B4DFC           mov ecx, dword ptr [ebp-04]  
:00422682 C6416F04         mov [ecx+6F], 04  
:00422686 8B55FC           mov edx, dword ptr [ebp-04]  
:00422689 C6427903         mov [edx+79], 03  
:0042268D 8B45FC           mov eax, dword ptr [ebp-04]  
:00422690 C6808300000004  mov byte ptr [eax+00000083], 04  
:00422697 8B4DFC           mov ecx, dword ptr [ebp-04]  
:0042269A C6818D00000005  mov byte ptr [ecx+0000008D], 05  
:004226A1 8B55FC           mov edx, dword ptr [ebp-04]  
:004226A4 C6829700000001  mov byte ptr [edx+00000097], 01  
:004226AB 8B45FC           mov eax, dword ptr [ebp-04]  
:004226AE C680A100000000  mov byte ptr [eax+000000A1], 00
```

呵呵,`0~6` 是黑的车马炮将士象兵 `7~D` 是红的。

现在知道上面做坐标运算的时候为什么要用 `FF` 来 `AND` 了吧,

因为每个棋子的类型只用一个 `byte` 来存放,呵呵,这样如果用值

查找找一辈子也找不到,这里还可以发现存放棋子类型不是以

棋子为对象,是以象棋盘上的交叉点来做对象,点上是 `FF` 为没有

棋子,是 `0~D` 的话就是各种棋子.

现在大功告成,把所有棋子改成帅!呵呵,爽吧.

不过最后告诉大家,由于中国游戏中心服务器只接受棋子移动的信息,

比如跑 `2` 平 `5`,并不接受棋子类型信息,所以如果你把自己的兵改成车的话,在本地机器上是可以发送车 `1` 进 `3` 的命令的,但是到了服务器就会被认为是兵 `1` 进 `3` 了,显然兵是不能进 `3` 的,所以被认为是非法指令,马上被踢!

呵呵,如果能作弊就不敢公开了,被 `XX` 就不好了....

(众怒:`TMD` 骗我们看了这么久..)

啊..大家有话好说嘛....还是先闪了,水平甚菜,有不当之处还请大虾们指出:)

`4dd466d0e1e0cc123afbe504da2462ca`
