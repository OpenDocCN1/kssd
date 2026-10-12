
# 单词跑马灯 1.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue231.htm`](https://www.pediy.com/kssd/pediy04/kanxue231.htm)

软件：单词跑马灯 1.0

原版下载地址: `http://www.csdn.net/soft/openfile.asp?kind=1&id=14164`

目标：找出注册码

工具：`language2000`，`DeDe`，`Ultraedit`

软件作者：`xtzy`

软件主页：`http://xtzy.51.net/soft/english/english.htm`

软件类别：中文相关 单词浏览，单词听写...

最新版本：下载软件(软件名） 软件大小 源程序语言 发布日期

| 单词跑马灯 | 1.0 | 245K | c++ | 2001-10-24 |

初学破解,一点心得,希望与大家交流.(在白菜大哥的站下了 `fish` 大哥的注册版，可惜 `installer CRC invalid` 出错，所以自力更新).

这个软件用 `language2000` 测出用 `ASProtect/ASPack` 压缩/加密过，用 `ASProtect` 与 `Aspack` 的克星 `Caspr.exe` 脱壳后软件变成 576k 大小，再用 `language2000` 测出是 `Dephi` 编的。该软件使用机器码对应注册码，未注册版仅能显示 300 个单词或选择词序重新学习！

用 `DeDe` 反编译脱壳后软件，在窗体中只有模块一个 `TForm1`，选 `TForm1` 可看到

```
object N1: TMenuItem 
    Caption = '注册' 
    object N2: TMenuItem 
      Caption = '说明' 
      OnClick = N2Click 
    end 
    object N3: TMenuItem 
      Caption = '注册' 
      OnClick = N3Click 
    end 
```

跟进 `N3Click`，可看到

```
* Possible String Reference to: '您的机器码是: ' 
| 
00404FE0  B81A184600     mov    eax, $0046181A 
00404FE5  5A             pop    edx 
| 
00404FE6  E8D5AE0500     call   0045FEC0 
00404FEB  8D4DF4         lea    ecx, [ebp-$0C] 
00404FEE  FF31           push   dword ptr [ecx] 
* Possible String Reference to: '请输入您的注册码：' 
| 
00404FF0  BA29184600     mov    edx, $00461829 
00404FF5  8D45F0         lea    eax, [ebp-$10] 
| 
00404FF8  E8DBAB0500     call   0045FBD8 
00404FFD  FF45D8         inc    dword ptr [ebp-$28] 
00405000  8B10           mov    edx, [eax] 
00405002  8D4DFC         lea    ecx, [ebp-$04] 
00405005  58             pop    eax 
* Reference to: dialogs.InputQuery(System.AnsiString;System.AnsiString;System.AnsiString):System.Boolean; 
| 
00405006  E8CD950200     call   0042E5D8 
0040500B  FF4DD8         dec    dword ptr [ebp-$28] 
0040500E  8D45F0         lea    eax, [ebp-$10] 
00405011  BA02000000     mov    edx, $00000002 
| 
00405016  E88DAC0500     call   0045FCA8 
0040501B  FF4DD8         dec    dword ptr [ebp-$28] 
0040501E  8D45F4         lea    eax, [ebp-$0C] 
00405021  BA02000000     mov    edx, $00000002 
| 
00405026  E87DAC0500     call   0045FCA8 
0040502B  FF4DD8         dec    dword ptr [ebp-$28] 
0040502E  8D45F8         lea    eax, [ebp-$08] 
00405031  BA02000000     mov    edx, $00000002 
| 
00405036  E86DAC0500     call   0045FCA8 
0040503B  8B45FC         mov    eax, [ebp-$04] 
* Reference to: sysutils.StrToInt(System.AnsiString):System.Integer; 
| 
0040503E  E8853A0400     call   00448AC8 
00405043  3B0510144600   cmp    eax, [$461410] 
00405049  0F8591000000   jnz    004050E0                  <====等于 0 成功 
0040504F  66C745CC2C00   mov    word ptr [ebp-$34], $002C  不等于 0 跳 注册失败 
* Possible String Reference to: '注册成功！谢谢！' 
| 
00405055  BA3C184600     mov    edx, $0046183C 
0040505A  8D45EC         lea    eax, [ebp-$14] 
| 
0040505D  E876AB0500     call   0045FBD8 
00405062  FF45D8         inc    dword ptr [ebp-$28] 
00405065  8B00           mov    eax, [eax] 
```

好，我爆、爆、爆(用 `trw` & `Ollydbg` 追过不成功，学艺未精!!!)！用 `Ultraedit` 打开脱壳后软件，找 `0F8591000000` 改为 `909090909090`。重新运行改过的软件，随便输入注册码 `1111`，“ok" '注册成功！谢谢！'生成一个 `ini` 文件，打开有

```
[Regist] 
NO=439956643 <====我机器对应的注册码 
```

重新运行原装软件，输入注册码 `439956643`，注册成功。

纯粹交流学习，别无它意，该软件注册费 four 元，请支持国产软件！！！

这是小弟写的第二篇破文,不知此文可不可以做我加入 `BCG` 的第一篇呢？

`zywicbc` 2001/11/11 上午  `4dd466d0e1e0cc123afbe504da2462ca`
