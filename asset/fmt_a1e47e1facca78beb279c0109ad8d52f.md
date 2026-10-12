
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
