
# 撰稿人宝典 1.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50427.htm`](https://www.pediy.com/kssd/pediy05/pediy50427.htm)

算法分析——撰稿人宝典 V1.0

下载地址： [`www.skycn.com/soft/10803.html`](http://www.skycn.com/soft/10803.html)

软件大小： 696 KB

软件语言： 简体中文

软件类别： 国产软件 / 共享版 / 文字处理

应用平台： `Win9x/NT/2000/XP`

加入时间： 2003-02-13 12:08:02

下载次数： 136

推荐等级： ***

开发商： [`www.simplepad.51dream.com/`](http://www.simplepad.51dream.com/)

**【软件简介】：** 一款专为撰稿人设计的字处理软件，除了有常用功能外，还有许多特色功能：1.中文字数统计：您写了多少字，可随时在状态栏察看；2.自动保存：自设保存时间间隔，最小化减少意外损失；3.自动开启输入法：您可以自由选择让软件在启动时就切换出您最熟悉的输入法；4.输入时隐藏鼠标指针；5.自动翻页：读长篇文章时可自动翻页，允许自设翻页时间；6.背景音乐播放：支持“Wave,media,mp3,wma”格式，可循环播放，更加节省系统资源；7.无限次“撤销/恢复”功能；8.三击鼠标可选中一段文字，支持托拽文字操作；9.可插入自定义文本：可自定义“信头”，“联系方法”等常用词，下次即可方便插入，一次定义，一劳永逸！10.独创“随机色彩”功能：撰稿人长时间面对白屏黑字，视觉容易疲劳，受 `Delphi` 的启发，为大家精选五种醒目但却不刺眼的颜色，输入时随机变换色彩，可以有效的缓解视疲劳。

**【软件限制】：** `NAG`

**【作者声明】：** 初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

**【破解工具】：** `TRW2000` 娃娃修改版、`FI2.5`、`UPXWin`、`W32Dasm8.93` 黄金版

---

**【过程】：**

呵呵，对 `MiniPad 2000` 感到头疼，去 天空 看到一个小东西，当然是 `DOWN` 下试试了。

---

## 一、脱壳

`word.exe` 是 `UPX 1.2` 壳，用 `UPXWin` 脱之。423K->1.11M。`Delphi` 编写。

用户名：`fly`

申请码：`103B2C1E`

试炼码：`1357 2468`

---

## 二、反汇编、调试

呵呵，当然是祭出倚天剑、屠龙刀了！

### 一>、首先看看申请码的生成

```
:004E57A8 E8CF36F2FF           call 00408E7C
:004E57AD FF75E4               push [ebp-1C]
:004E57B0 B838DF4E00           mov eax, 004EDF38
:004E57B5 BA04000000           mov edx, 00000004
                               ====>4 入 EDX
:004E57BA E84DF0F1FF           call 0040480C
                               ====>关键 CALL！F8 进入！
:004E57BF 8B1538DF4E00         mov edx, dword ptr [004EDF38]
:004E57C5 8B86FC020000         mov eax, dword ptr [esi+000002FC]
:004E57CB E8182CF6FF           call 004483E8
:004E57D0 33D2                 xor edx, edx
:004E57D2 8B8600030000         mov eax, dword ptr [esi+00000300]
:004E57D8 8B08                 mov ecx, dword ptr [eax]
:004E57DA FF5164               call [ecx+64]
:004E57DD 33D2                 xor edx, edx
:004E57DF 8B86F8020000         mov eax, dword ptr [esi+000002F8]
:004E57E5 8B08                 mov ecx, dword ptr [eax]
:004E57E7 FF5164               call [ecx+64]
:004E57EA 8D55E0               lea edx, dword ptr [ebp-20]
:004E57ED 8B86FC020000         mov eax, dword ptr [esi+000002FC]
:004E57F3 E8C02BF6FF           call 004483B8
:004E57F8 8B4DE0               mov ecx, dword ptr [ebp-20]
* Possible StringData Ref from Code Obj ->"IDcode"
  |
:004E57FB BA88584E00           mov edx, 004E5888
:004E5800 8BC3                 mov eax, ebx
:004E5802 E8B99EF8FF           call 0046F6C0
:004E5807 8D55DC               lea edx, dword ptr [ebp-24]
:004E580A 8B86F8020000         mov eax, dword ptr [esi+000002F8]
:004E5810 E8A32BF6FF           call 004483B8
:004E5815 8B4DDC               mov ecx, dword ptr [ebp-24]
* Possible StringData Ref from Code Obj ->"User"
  |
:004E5818 BA98584E00           mov edx, 004E5898
:004E581D 8BC3                 mov eax, ebx
:004E581F E89C9EF8FF           call 0046F6C0
```

`&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&`

进入 `4E57BA` `call 0040480C`

```
:0040480C 53                   push ebx
:0040480D 56                   push esi
:0040480E 57                   push edi
:0040480F 52                   push edx
:00404810 50                   push eax
:00404811 89D3                 mov ebx, edx
:00404813 31FF                 xor edi, edi
:00404815 8B4C9414             mov ecx, dword ptr [esp+4*edx+14]
                               ====>把 ESP+EDX*4+14 处的内存中值移入 ECX
                               呵呵，不知道怎么表达了。希望大家指点！
                               ====>1、00DA26F0 中为 10
:00404819 85C9                 test ecx, ecx
:0040481B 7406                 je 00404823
:0040481D 3908                 cmp dword ptr [eax], ecx
:0040481F 7502                 jne 00404823
:00404821 89C7                 mov edi, eax
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040481B(C), :0040481F(C)
|
:00404823 31C0                 xor eax, eax
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00404837(C)
|
:00404825 8B4C9414             mov ecx, dword ptr [esp+4*edx+14]
                               ====>把 ESP+EDX*4+14 处的内存中值移入 ECX
                               呵呵，不知道怎么表达了。希望大家指点！
                               ====>1、00DA26F0 中为 10
                               ====>2、00DA2F6C 中为 3B
                               ====>3、00DA2134 中为 2C
                               ====>4、00DA5050 中为 1E
:00404829 85C9                 test ecx, ecx
:0040482B 7409                 je 00404836
:0040482D 0341FC               add eax, dword ptr [ecx-04]
:00404830 39CF                 cmp edi, ecx
:00404832 7502                 jne 00404836
:00404834 31FF                 xor edi, edi
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040482B(C), :00404832(C)
|
:00404836 4A                   dec edx
                               ===>依次减 1
:00404837 75EC                 jne 00404825
                               ===>循环 4 次！
```

呵呵，把上面的值连起来就是程序给我们的申请码！不知道为何申请码的生成具有随机性，既每次的申请码皆不同。与用户名无关！希望大家给我指点迷津！

`——————————————————————————————————`

### 二>、追注册码的算法！

查找关键提示"注册码错误"在 `4E5C05`，由 `004E596C` 和 `004E59B3` 跳至！因此我们很容易就找到关键的断点了。因为想看看算法，所以把断点下在 `4E593E` 处。分别输入上面的试炼码，点“注册”，OK！拦下！

```
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004E596C(C), :004E59B3(C)     < ====  向上追！
|
:004E5C03 6A10                 push 00000010
* Possible StringData Ref from Code Obj ->"注册码错误"
  |
:004E5C05 68185E4E00           push 004E5E18
```

`&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&`

`BPX 4E593E` 拦下！呵呵，又开始我的菜鸟学算法之旅了！

```
:004E593E E8752AF6FF           call 004483B8
                               ====>拦下！
:004E5943 8B45E8               mov eax, dword ptr [ebp-18]
:004E5946 50                   push eax
                               ====>EAX=103B2C1E  申请码
:004E5947 8D55E4               lea edx, dword ptr [ebp-1C]
:004E594A 8B83F8020000         mov eax, dword ptr [ebx+000002F8]
:004E5950 E8632AF6FF           call 004483B8
:004E5955 8B45E4               mov eax, dword ptr [ebp-1C]
                               ====> fly 入 EAX
* Possible StringData Ref from Code Obj ->"EZWord"
                               ====>注意：EZWord
:004E5958 B9085D4E00           mov ecx, 004E5D08
                               ====> EZWord 入 ECX
:004E595D 5A                   pop edx
:004E595E E875F9FFFF           call 004E52D8
                               ====>运算第一部分！进入！
:004E5963 8B55EC               mov edx, dword ptr [ebp-14]
                               ====>结果：9e130943 入 EDX
:004E5966 58                   pop eax
                               ====>EAX=1357 试炼码的前部分
:004E5967 E824EFF1FF           call 00404890
                               ====>比较第一部分！进入！
:004E596C 0F8591020000         jne 004E5C03
                               ====>跳则 OVER！
:004E5972 8D55E0               lea edx, dword ptr [ebp-20]
:004E5975 8B8318030000         mov eax, dword ptr [ebx+00000318]
:004E597B E8382AF6FF           call 004483B8
:004E5980 8B45E0               mov eax, dword ptr [ebp-20]
:004E5983 50                   push eax
:004E5984 68FC5C4E00           push 004E5CFC
```



## 汇编代码分析

```
:004E5989 8D45DC                 lea eax, dword ptr [ebp-24]  
:004E598C 50                     push eax  
:004E598D 8D55D8                 lea edx, dword ptr [ebp-28]  
:004E5990 8B83FC020000           mov eax, dword ptr [ebx+000002FC]  
:004E5996 E81D2AF6FF             call 004483B8  
:004E599B 8B55D8                 mov edx, dword ptr [ebp-28]  
                                ====>103B2C1E 入 EDX  申请码
* Possible StringData Ref from Code Obj ->"EZWord"  
                                |  
:004E599E B9085D4E00             mov ecx, 004E5D08  
                                ===> EZWord 入 ECX
:004E59A3 33C0                   xor eax, eax  
:004E59A5 E82EF9FFFF             call 004E52D8  
                                ===>运算第二部分！进入！
:004E59AA 8B55DC                 mov edx, dword ptr [ebp-24]  
                                ===>结果：e4e8a4fe 入 EDX
:004E59AD 58                     pop eax  
                                ===>EAX=2468 试炼码的后部分
:004E59AE E8DDEEF1FF             call 00404890  
                                ===>比较第二部分！进入！
:004E59B3 0F854A020000           jne 004E5C03  
                                ===>跳则 OVER！
&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&  
```

1. 进入运算部分！`4E595E` `call 004E52D8`
2. 进入运算部分！`4E59A5` `call 004E52D8`

* Referenced by a CALL at Addresses:

```
:004E595E   , :004E59A5   , :004E59F6   , :004E5A14   , :004E5A46   
:004E5A64   , :004E5A96   , :004E5AB4   , :004E5AE6   , :004E5B04   
:004E8DD8   , :004E8E11   , :004E8E4A   , :004E8E83   
```

```
:004E52D8 55                     push ebp  
:004E52D9 8BEC                   mov ebp, esp  
:004E52DB 83C4EC                 add esp, FFFFFFEC  
:004E52DE 53                     push ebx  
:004E52DF 33DB                   xor ebx, ebx  
:004E52E1 895DEC                 mov dword ptr [ebp-14], ebx  
:004E52E4 895DF0                 mov dword ptr [ebp-10], ebx  
:004E52E7 894DF4                 mov dword ptr [ebp-0C], ecx  
:004E52EA 8955F8                 mov dword ptr [ebp-08], edx  
:004E52ED 8945FC                 mov dword ptr [ebp-04], eax  
:004E52F0 8B45FC                 mov eax, dword ptr [ebp-04]  
:004E52F3 E83CF6F1FF             call 00404934  
:004E52F8 8B45F8                 mov eax, dword ptr [ebp-08]  
:004E52FB E834F6F1FF             call 00404934  
:004E5300 8B45F4                 mov eax, dword ptr [ebp-0C]  
:004E5303 E82CF6F1FF             call 00404934  
:004E5308 8B450C                 mov eax, dword ptr [ebp+0C]  
:004E530B E824F6F1FF             call 00404934  
:004E5310 33C0                   xor eax, eax  
:004E5312 55                     push ebp  
:004E5313 687E534E00             push 004E537E  
:004E5318 64FF30                 push dword ptr fs:[eax]  
:004E531B 648920                 mov dword ptr fs:[eax], esp  
:004E531E FF75FC                 push [ebp-04]  
:004E5321 FF75F8                 push [ebp-08]  
:004E5324 FF75F4                 push [ebp-0C]  
:004E5327 FF750C                 push [ebp+0C]  
:004E532A 8B450C                 mov eax, dword ptr [ebp+0C]  
:004E532D 50                     push eax  
:004E532E 8D45EC                 lea eax, dword ptr [ebp-14]  
:004E5331 50                     push eax  
:004E5332 8B4DF4                 mov ecx, dword ptr [ebp-0C]  
:004E5335 8B55F8                 mov edx, dword ptr [ebp-08]  
:004E5338 8B45FC                 mov eax, dword ptr [ebp-04]  
:004E533B E840FDFFFF             call 004E5080
:004E5340 FF75EC                 push [ebp-14]  
    运算第一部分！               ===>U@pg%,p`       程序自给  
    运算第二部分！               ===>npnczwFj       程序自给
:004E5343 8D45F0                 lea eax, dword ptr [ebp-10]  
:004E5346 BA05000000             mov edx, 00000005  
:004E534B E8BCF4F1FF             call 0040480C  
    运算第一部分！     ===>此 CALL 把用户名、申请码以及上面的 U@pg%,p`连接起来  
    运算第二部分！     ===>此 CALL 把申请码以及上面的 npnczwFj 连接起来
:004E5350 8B5508                 mov edx, dword ptr [ebp+08]  
:004E5353 8B45F0                 mov eax, dword ptr [ebp-10]  
    运算第一部分！     ===>fly103B2C1EEZWord1U@pg%,p`   入 EAX  
    运算第二部分！     ===>103B2C1EEZWord1npnczwFj       入 EAX       
:004E5356 E831000000             call 004E538C  
                                ===>运算了！进入！
:004E535B 33C0                   xor eax, eax  
:004E535D 5A                     pop edx  
:004E535E 59                     pop ecx  
:004E535F 59                     pop ecx  
:004E5360 648910                 mov dword ptr fs:[eax], edx  
:004E5363 6885534E00             push 004E5385
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
:004E5383(U)  
:004E5368 8D45EC                 lea eax, dword ptr [ebp-14]  
:004E536B BA05000000             mov edx, 00000005  
:004E5370 E833F1F1FF             call 004044A8  
:004E5375 8D450C                 lea eax, dword ptr [ebp+0C]  
:004E5378 E807F1F1FF             call 00404484  
:004E537D C3                     ret
&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&  
```

进入运算 CALL：`4E5356` `call 004E538C`

* Referenced by a CALL at Address:
`:004E5356`

```
:004E538C 55                     push ebp  
:004E538D 8BEC                   mov ebp, esp  
:004E538F 83C4F4                 add esp, FFFFFFF4  
:004E5392 53                     push ebx  
:004E5393 56                     push esi  
:004E5394 33C9                   xor ecx, ecx  
:004E5396 894DF4                 mov dword ptr [ebp-0C], ecx  
:004E5399 8955F8                 mov dword ptr [ebp-08], edx  
:004E539C 8945FC                 mov dword ptr [ebp-04], eax  
:004E539F 8B45FC                 mov eax, dword ptr [ebp-04]  
:004E53A2 E88DF5F1FF             call 00404934  
:004E53A7 33C0                   xor eax, eax  
:004E53A9 55                     push ebp  
:004E53AA 682B544E00             push 004E542B  
:004E53AF 64FF30                 push dword ptr fs:[eax]  
:004E53B2 648920                 mov dword ptr fs:[eax], esp  
:004E53B5 33DB                   xor ebx, ebx  
:004E53B7 8B45FC                 mov eax, dword ptr [ebp-04]  
    运算第一部分！       ===>fly103B2C1EEZWord1U@pg%,p` 入 EAX  
    运算第二部分！       ===>103B2C1EEZWord1npnczwFj    入 EAX
:004E53BA E88DF3F1FF             call 0040474C  
                         ===>求上面字符串的长度  
:004E53BF 85C0                   test eax, eax  
    运算第一部分！       ===>EAX=1A （即：26 位）  
    运算第二部分！       ===>EAX=17 （即：23 位）
:004E53C1 7E2C                   jle 004E53EF
:004E53C3 BE01000000             mov esi, 00000001
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
:004E53ED(C)  
:004E53C8 8B55FC                 mov edx, dword ptr [ebp-04]  
      运算第一部分！    ===>fly103B2C1EEZWord1U@pg%,p` 入 EDX  
      运算第二部分！    ===>103B2C1EEZWord1npnczwFj    入 EDX
:004E53CB 8A5432FF               mov dl, byte ptr [edx+esi-01]  
    运算第一部分！  ===>逐位从 fly103B2C1EEZWord1U@pg%,p`中取字符的 HEX 值！  
                         ===>1、DL=66  
                ……  …… 省略（共 26 次） …… ……
    %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
    运算第二部分！      ===>逐位从 103B2C1EEZWord1npnczwFj 中取字符的 HEX 值！  
                         ===>1、DL=31  
                ……  …… 省略（共 23 次） …… ……
:004E53CF 32D3                   xor dl, bl  
    运算第一部分！      ===>与 BL 进行异或！ 第一次 BL=0  
                         ===>1、DL=66 XOR 0=66  
                ……  …… 省略（共 26 次） …… ……
    %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
    运算第二部分！      ===>与 BL 进行异或！ 第一次 BL=0  
                         ===>1、DL=31 XOR 0=31  
                ……  …… 省略（共 23 次） …… ……
:004E53D1 81E2FF000000           and edx, 000000FF  
:004E53D7 8B1495A4BC4E00         mov edx, dword ptr [4*edx+004EBCA4]  
```



### 运算流程分析

**运算第一部分**  
`EDX` = `A4D1C46D`  
…… 省略（共 26 次） ……

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

**运算第二部分**  
`EDX` = `51DE003A`  
…… 省略（共 23 次） ……

```
:004E53DE C1EB08           shr ebx, 08  
                           ====>EBX 右移 8 位！  
                           ====>1、EBX=0
```

```
:004E53E1 81E3FFFFFF00     and ebx, 00FFFFFF  
:004E53E7 33D3             xor edx, ebx  
                           ====>1、EDX=A4D1C46D XOR EBX=A4D1C46D  
                           …… 省略（共 26 次） ……

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

                           ====>1、EDX=51DE003A XOR EBX=51DE003A  
                           …… 省略（共 23 次） ……
```

```
:004E53E9 8BDA             mov ebx, edx  
                           ====>EDX 移入 EBX  
                           ====>循环 26 次后，EDX=9E130943  
                           %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
                           ====>EDX 移入 EBX  
                           ====>循环 23 次后，EDX=E4E8A4FE
```

```
:004E53EB 46               inc esi  
                           ====>ESI 增 1  

:004E53EC 48               dec eax  
                           ====>EAX 减 1  

:004E53ED 75D9             jne 004E53C8  
                           ====>继续循环！共 26 次！  
                           ====>继续循环！共 23 次！
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  `:004E53C1(C)`

```
:004E53EF 8BC3             mov eax, ebx  
                           ====>循环 26 次后，9E130943 入 EAX  
                           %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
                           ====>循环 23 次后，E4E8A4FE 入 EAX  
```

```
:004E53F1 33D2             xor edx, edx  
:004E53F3 52               push edx  
:004E53F4 50               push eax  
:004E53F5 8D55F4           lea edx, dword ptr [ebp-0C]  
:004E53F8 B808000000       mov eax, 00000008  
:004E53FD E8B63AF2FF       call 00408EB8  
:004E5402 8B45F4           mov eax, dword ptr [ebp-0C]  
:004E5405 8B55F8           mov edx, dword ptr [ebp-08]
:004E5408 E88735F2FF       call 00408994  
                           ====>将 9E130943 中的大写字母转化为小写！  
                           呵呵，9e130943 就是注册码前半部分！  
                           %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
                           ====>将 E4E8A4FE 中的大写字母转化为小写！  
                           呵呵，e4e8a4fe 就是注册码后半部分！
```

```
:004E540D 33C0             xor eax, eax  
:004E540F 5A               pop edx  
:004E5410 59               pop ecx  
:004E5411 59               pop ecx  
:004E5412 648910           mov dword ptr fs:[eax], edx  
:004E5415 6832544E00       push 004E5432
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  `:004E5430(U)`

```
:004E541A 8D45F4           lea eax, dword ptr [ebp-0C]  
:004E541D E862F0F1FF       call 00404484  
:004E5422 8D45FC           lea eax, dword ptr [ebp-04]  
:004E5425 E85AF0F1FF       call 00404484  
:004E542A C3               ret
```

---

### 真假码比较

1. 进入比较部分！`4E5967` `call 00404890`
2. 进入比较部分！`4E59AE` `call 00404890`

* Referenced by a CALL at Addresses:
  `:0041838F`, `:0041EBD2`, `:004213CF`, `:00427129`, `:00428100`

…… 很多地方 `CALL` 此处！ ……

```
:00404890 53               push ebx  
:00404891 56               push esi  
:00404892 57               push edi  
:00404893 89C6             mov esi, eax  
:00404895 89D7             mov edi, edx  
:00404897 39D0             cmp eax, edx  
                           ====>EAX=1357  
                           ====>EDX=9e130943  
                           %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
                           ====>EAX=2468  
                           ====>EDX=e4e8a4fe
```

```
:00404899 0F848F000000     je 0040492E  
:0040489F 85F6             test esi, esi  
:004048A1 7468             je 0040490B  
:004048A3 85FF             test edi, edi  
:004048A5 746B             je 00404912  
:004048A7 8B46FC           mov eax, dword ptr [esi-04]  
:004048AA 8B57FC           mov edx, dword ptr [edi-04]  
:004048AD 29D0             sub eax, edx  
:004048AF 7702             ja 004048B3
```

---

### KeyMake 之内存注册机

**目标程序**：`fly{43th}`

**中断点 1**：
- 中断地址：`4E5967`
- 中断次数：1
- 第一字节：`E8`
- 指令长度：5
- 保存下列信息为注册码：
  - 内存方式：`EDX`
  - 结尾插入字符 `-`（`-`前的为注册码的前半部分，`-`后的则为后半部分）

**中断点 2**：
- 中断地址：`4E59AE`
- 中断次数：1
- 第一字节：`E8`
- 指令长度：5
- 内存方式：`EDX`

**修改内存**：
- 修改地址：`4E596C`
- 修改长度：6
- 原始指令：`0F8591020000`
- 修改指令：`909090909090`

---

### 注册信息保存

```registry
REGEDIT4

[HKEY_LOCAL_MACHINE\Software\EZWord]  
"IDcode"="103B2C1E"  
"User"="fly"  
"Code1"="9e130943"  
"Code2"="e4e8a4fe"
```

呵呵，把这些信息存为`.reg`文件导入注册表中应该就是注册版了。^-^

---

### 整理

- 用户名：`fly`
- 申请码：`103B2C1E`
- 注册码：`9e130943` `e4e8a4fe`

---

Cracked By 巢水工作坊——fly【OCN】

2003-2-18 23:34  
`4dd466d0e1e0cc123afbe504da2462ca`
