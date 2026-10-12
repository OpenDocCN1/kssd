
# `xp` 变脸王的注册算法

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-232.htm`](https://www.pediy.com/kssd/pediy08/pediy8-232.htm)

`xp` 变脸王保护首先第一道屏障是壳，`peid` 显示 `Armadillo 1.xx - 2.xx -> Silicon Realms Toolworks` 双进程调试保护 `CopyMEM2`，`Stolen Code` 远程跳转，`CC`。其中 `CC` 之多是罕见的。

站在 `fly` 和 `fxyang` 的肩膀上，我把壳脱掉了。然后用 `od` 载入后，来到这里：其实修复 `CC` 时可以看到这里的。

```
* Possible StringData Ref from Code Obj ->"Do NOT crack me!Merci!"
                             |
:00675F56 BAE4636700           mov edx, 006763E4
:00675F5B 8B45FC               mov eax, dword ptr [ebp-04]
:00675F5E E8AD2DF2FF           call 00598D10                              这是一个共用的算法 call
:00675F63 8B8538FEFFFF         mov eax, dword ptr [ebp+FFFFFE38]
:00675F69 B905000000           mov ecx, 00000005
:00675F6E BA0E000000           mov edx, 0000000E
:00675F73 E8C4EFD8FF           call 00404F3C                              截取注册码 call
:00675F78 8B953CFEFFFF         mov edx, dword ptr [ebp+FFFFFE3C]
:00675F7E 8B45F8               mov eax, dword ptr [ebp-08]
:00675F81 E8A2EED8FF           call 00404E28                              比较 call
:00675F86 0F8491000000         je 0067601D
:00675F8C 33D2                 xor edx, edx
:00675F8E 8B8360040000         mov eax, dword ptr [ebx+00000460]
:00675F94 E89779E0FF           call 0047D930
:00675F99 8D952CFEFFFF         lea edx, dword ptr [ebp+FFFFFE2C]
```

上面是计算并比较第一段注册码的地方，其中 `call 00598D10` 是所有 5 段注册码共用的算法 call，先把序列号计算成一个字符串，然后截取成 5 位，再明码比较。只是前面的字符串不同，以及截取的位置不同，第一段是 `Do NOT crack me!Merci!`，我用我计算机上唯一的一个正版软体金山词霸 2001 oem 版，知道了 `Merci` 是法语，谢谢的意思。

大致跟一下，发现他的算法是如此的复杂，这么复杂然后明码比较，一个很大的问题，很可能是现成的算法源码，在这个 call 里发现，那个字符串 `Do NOT crack me!Merci!` 会变形成 8 个字节，在里面的另一个关键 call 里，发现了如下循环代码：

```
:0059845A 42                  inc edx
:0059845B 40                  inc eax
:0059845C 83FA38              cmp edx, 00000038      这里是循环次数
:0059845F 75AD                jne 0059840E           不等就循环
```

看了一下算法的简介找到 des 的简介如下：

DES 算法的入口参数有三个：`Key`、`Data`、`Mode`。其中 `Key` 为 8 个字节共 64 位，是 DES 算法的工作密钥；DES 算法中只用到 64 位密钥中的其中 56 位。

另外，精华 4 里有一篇变脸王的文章就是 des。

我找啊找啊，找朋友，找到一个好朋友：`delphi` 的 des 源码，只要更改密钥即可，用看见的密钥试了试，果然，大概同益起名也是用的它吧。

简单了，算法如下：（密钥是变形后的 8 个字节）

第一组密钥：`Do NOT cS(@`

从第 14 位取 5 位

第二组密钥：`Merde!`

从第 17 位取 5 位

第三组密钥：`Va-t-en!S(@`

从第 18 位取 5 位

第四组密钥：`holer@21S(@`

从第 19 位取 5 位

第五组密钥：`www.holeS(@`

从第 20 位取 5 位

临时学习了一下 `delphi` 写了个注册机：

```pascal
procedure TForm1.Button1Click(Sender: TObject);

var
 S1,s2,s3,s4,s5,s6,s7,s8,s9,s10: string;
 s11: char;

begin

s1:= EncryStrhex(Edit1.Text,'Do NOT cS(@');

s2:= EncryStrhex(Edit1.Text,'Merde!');

s3:= EncryStrhex(Edit1.Text,'Va-t-en!S(@');

s4:= EncryStrhex(Edit1.Text,'holer@21S(@');

s5:= EncryStrhex(Edit1.Text,'www.holeS(@');

s6:=copy(s1,14,5);

s7:=copy(s2,17,5);

s8:=copy(s3,18,5);

s9:=copy(s5,20,5);

s10:=copy(s4,19,5);

edit2.Text:=s6+'-'+s7+'-'+s8+'-'+s10+'-'+s9;

end;
```

附件是 des 的源码。用我的代码替换源码中的按钮 1 点击代码就行了。

`4dd466d0e1e0cc123afbe504da2462ca`
