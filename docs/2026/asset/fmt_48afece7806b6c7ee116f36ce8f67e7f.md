
```
00411238    46              inc esi
00411239    3B75 18         cmp esi,dword ptr ss:[ebp+18]
0041123C  ^ 7C EB           jl short umd.00411229
0041123E    53              push ebx
0041123F    8B55 14         mov edx,dword ptr ss:[ebp+14]
00411242    52              push edx
00411243    53              push ebx
00411244    E8 8FF5FFFF    call umd.004107D8 ; 关键call，对32位数据进行运算，步入
00411249    83C4 0C         add esp,0C
0041124C    5F              pop edi
0041124D    5E              pop esi
0041124E    5B              pop ebx
0041124F    5D              pop ebp
00411250    C3              retn
```

32位数据是用来参与核心运算的，其运算结果要与`D3BD9115AFD78174`进行比较。跟进之后：

```
00410704    55              push ebp
00410705    8BEC            mov ebp,esp
00410707    8B45 0C         mov eax,dword ptr ss:[ebp+C]
0041070A    33D2            xor edx,edx
0041070C    8950 14         mov dword ptr ds:[eax+14],edx
0041070F    8950 10         mov dword ptr ds:[eax+10],edx
00410712    C700 01234567   mov dword ptr ds:[eax],67452301             ; 0012F158堆栈地址填充67452301
00410718    C740 04 89ABCDE>mov dword ptr ds:[eax+4],EFCDAB89        ; 0012F15C堆栈地址填充EFCDAB89
0041071F    C740 08 FEDCBA9>mov dword ptr ds:[eax+8],98BADCFE        ; 0012F160堆栈地址填充98BADCFE
00410726    C740 0C 7654321>mov dword ptr ds:[eax+C],10325476           ; 0012F164堆栈地址填充10325476
0041072D    5D              pop ebp
0041072E    C3              retn
```

实际上就是在`0012F158—0012F164`填充`67452301EFCDAB8998BADCFE10325476`

![](https://bbs.kanxue.com/upload/attach/201312/573536_7ecshqlof2t2e9u.png)

`00411244`对应的调用步入之后，对32数据的运算是在`00410786`的调用里：

```
00410786    E8 D1000000     call umd.0041085C            ; 二次注册码产生的地方
```

一跟进去直接吓了一跳，洋洋洒洒940行，丹都碎了。鉴于自己能力有限，就放弃治疗，不打算分析这个算法了，那位大神如果有兴趣可以分析一下，这里我就把它的头和尾贴出来了事。

```
0041085C    55              push ebp
0041085D    8BEC            mov ebp,esp
0041085F    83C4 BC         add esp,-44
00410862    53              push ebx
00410863    56              push esi
00410864    57              push edi
00410865    8B45 0C         mov eax,dword ptr ss:[ebp+C]
...............
...............
...............
...............
004111FB    E8 E8CC0A00     call umd.004BDEE8
00411200    83C4 0C         add esp,0C
00411203    5F              pop edi
00411204    5E              pop esi
00411205    5B              pop ebx
00411206    8BE5            mov esp,ebp
00411208    5D              pop ebp
00411209    C3              retn
```

另一很大的问题出现了，这个数据跟输入的假码没有太大关系，随便用几个假码一试，发现这个32位假码`67452301EFCDAB8998BADCFE10325476`是不变的。那什么变了呢？与`D3BD9115AFD78174`进行比较的运算出来的16位数据变了。也就是说我们输入的假码决定的是最终算出的16位注册码是个什么样子，跟踪一下也没发现集体操作在哪里，鉴于算法部分太长（940行！），不太容易分析，就爆破一下，浅尝则止吧（哪位大神如果分析出来，一定要教教小弟）。

最简单的方法莫过于爆破了：

```
0040E447   /0F84 D3000000   je umd.0040E520      ; 关键跳转，如果跳转则注册码错误
```

这里，`je`改成`jne`就行了，或者手动pach一下也可以。就写这么多吧，新手小白，没分析出来算法，希望大家多多批评指正。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`图片1.png`](javascript:void(0);)（22.69kb，6次下载）
- [`图片2.png`](javascript:void(0);)（18.02kb，3次下载）
- [`图片3.png`](javascript:void(0);)（25.74kb，5次下载）
- [`图片4.png`](javascript:void(0);)（17.77kb，4次下载）
- [`图片5.png`](javascript:void(0);)（32.44kb，3次下载）
- [`图片6.png`](javascript:void(0);)（18.06kb，4次下载）
- [`图片7.png`](javascript:void(0);)（5.75kb，3次下载）
- [`图片8.png`](javascript:void(0);)（37.16kb，3次下载）
- [`图片9.png`](javascript:void(0);)（44.24kb，5次下载）
- [`图片10.png`](javascript:void(0);)（46.22kb，5次下载）
- [`图片11.png`](javascript:void(0);)（7.42kb，3次下载）
