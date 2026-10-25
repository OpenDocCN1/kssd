
# Absolute MP3 Splitter Converter v2.5.6 算法分析(RSA)兼 FGInt 库的使用

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-21.htm`](https://www.pediy.com/kssd/pediy09/pediy09-21.htm)

## 基本信息
- 【文章标题】: Absolute MP3 Splitter Converter v2.5.6 算法分析(RSA)兼 FGInt 库的使用
- 【软件名称】: Absolute MP3 Splitter Converter v2.5.6
- 【保护方式】: `RSA-100`

## 详细过程

1. `PEid` 分析是 `Delphi` 程序，`kanal` 插件识别出了几个 `FGInt` 的库函数；
2. 用 `DeDe` 分析出注册对话框中 `OK` 按钮的事件地址是 `0050016C`，同时用 `DeDe` 导出 `map` 文件；
3. `OD` 上场，加载上一步的 `map` 文件；
4. 手动给 `PEiD` 识别出来的那几个函数加上标签。估计是程序作者使用的 `FGInt` 库和我的版本不同，导致 `IDA` 没能识别出其库函数。

```assembly
0050016C  >/. push    ebp                 ;  <-TRegForm@BtnOKClick
0050016D   |. mov     ebp, esp
0050016F   |. push    0
00500171   |. push    0
00500173   |. push    ebx
00500174   |. mov     ebx, eax
00500176   |. xor     eax, eax
00500178   |. push    ebp
00500179   |. push    <->System.@HandleFinally;>
0050017E   |. push    dword ptr fs:[eax]
00500181   |. mov     fs:[eax], esp
00500184   |. lea     edx, [ebp-4]         ;  取用户名
00500187  >|. mov     eax, [ebx+314]       ;  *Edit1:N.A.
0050018D  >|. call    00469D48             ;  ->Controls.TControl.GetText
(TControl):TCaption;
00500192   |. lea     edx, [ebp-8]         ;  取注册码
00500195  >|. mov     eax, [ebx+318]       ;  *Edit2:N.A.
0050019B  >|. call    00469D48             ;  ->Controls.TControl.GetText
(TControl):TCaption;
005001A0   |. mov     eax, [505500]
005001A5   |. mov     eax, [eax]
005001A7   |. mov     ecx, [ebp-8]
005001AA   |. mov     edx, [ebp-4]
005001AD  >|. call    004FF1B4             ;  ->:TMainForm._PROC_004FF1B4()
005001B2   |. test    al, al
005001B4   |. je      short 0050020E       ;  比较注册成功与否
005001B6   |. mov     eax, [505500]
005001BB   |. mov     eax, [eax]
005001BD   |. mov     edx, [ebp-4]
005001C0   |. call    004FF528
005001C5   |. push    40
005001C7   |. mov     ecx, 00500250        ;  ASCII "Congratulations!"
005001CC   |. mov     edx, 00500264        ;  ASCII "Register successfully!  Thank
you for your support!"
005001D1   |. mov     eax, [5057BC]
005001D6   |. mov     eax, [eax]
005001D8  >|. call    0048A080             ;  ->Forms.TApplication.MessageBox
(TApplication;PChar;PChar;Longint):Integer;
```

可以看出，`005001AD` 是关键 `call`，跟进去：

```assembly
004FF1B4   /$ push    ebp
004FF1B5   |. mov     ebp, esp
004FF1B7   |. add     esp, -1C
004FF1BA   |. push    ebx
004FF1BB   |. xor     ebx, ebx
004FF1BD   |. mov     [ebp-C], ebx
004FF1C0   |. mov     [ebp-8], ecx
004FF1C3   |. mov     [ebp-4], edx
004FF1C6   |. mov     eax, [ebp-4]
004FF1C9   |. call    00404C2C
004FF1CE   |. mov     eax, [ebp-8]
004FF1D1   |. call    00404C2C
004FF1D6   |. lea     eax, [ebp-14]
004FF1D9   |. mov     edx, [4BF084]        ;  Absolute.004BF088
004FF1DF   |. call    0040524C
004FF1E4   |. lea     eax, [ebp-1C]
004FF1E7   |. mov     edx, [4BF084]        ;  Absolute.004BF088
004FF1ED   |. call    0040524C
004FF1F2   |. xor     eax, eax
004FF1F4   |. push    ebp
004FF1F5   |. push    004FF276
004FF1FA   |. push    dword ptr fs:[eax]
004FF1FD   |. mov     fs:[eax], esp
004FF200   |. xor     ebx, ebx
004FF202   |. lea     edx, [ebp-14]
004FF205   |. mov     eax, [50533C]        ;  e:65537 = 0x10001
004FF20A   |. call    <Base10StringToFGInt>
004FF20F   |. lea     edx, [ebp-1C]
004FF212   |. mov     eax, [505340]        ;  n
004FF217   |. call    <Base10StringToFGInt>
004FF21C   |. lea     eax, [ebp-4]
004FF21F   |. push    eax
004FF220   |. lea     ecx, [ebp-1C]
004FF223   |. lea     edx, [ebp-14]
004FF226   |. mov     eax, [ebp-4]         ;  name
004FF229   |. call    <RsaEncrypt>
004FF22E   |. lea     edx, [ebp-C]
004FF231   |. mov     eax, [ebp-4]
004FF234   |. call    <ConvertBase256to64>         ;  这个函数 PEid 没能识别出来，是我猜+试出来
004FF239   |. mov     eax, [ebp-8]         ;  sn
004FF23C   |. mov     edx, [ebp-C]         ;  真正注册码
004FF23F   |. call    00404B88             ;  明码比较
004FF244   |. jnz     short 004FF248
004FF246   |. mov     bl, 1
004FF248   |> xor     eax, eax
004FF24A   |. pop     edx
004FF24B   |. pop     ecx
004FF24C   |. pop     ecx
004FF24D   |. mov     fs:[eax], edx
004FF250   |. push    004FF27D
004FF255   |> lea     eax, [ebp-1C]
004FF258   |. mov     edx, [4BF084]        ;  Absolute.004BF088
004FF25E   |. mov     ecx, 2
004FF263   |. call    00405368
004FF268   |. lea     eax, [ebp-C]
004FF26B   |. mov     edx, 3
004FF270   |. call    004047A0
004FF275   \. retn
```

尽管是明码比较，但我们还是尝试着来写个注册机，以熟悉 `FGInt` 库：

```pascal
//为了减小 Delphi 程序的体积，我们没有使用 VCL，而使用 console，这样代码更加清晰明了。
//Coded by [HappyTown]/2007-01-16
Program KeyGen;
{$APPTYPE CONSOLE}

Uses Windows,Messages,FGInt,FGIntRSA;
Var
  n,e: TFGInt;
  name,sn: String;
Begin
  Base10StringToFGInt('874802157929661091407425794781', n);
  Base10StringToFGInt('65537', e);
  Write('User name:');
  ReadLn(name);//name 应该小于 n，但我偷懒了，没有对它作限制
  RSAEncrypt(name, e, n, name);
  ConvertBase256to64(name, sn);
  FGIntdestroy(n);
  FGIntdestroy(e);
  WriteLn('Registration code:', sn);
  MessageBox(0, PChar(sn), 'Registration code:', MB_OK);//同时用 MessageBox 显示出来，方便拷贝
End.
```

给 3 组可用的注册码：

- `name`: `HappyTown`
  `sn`: `aRTyKEge7YjReqyGmq`
- `name`: `pediy`
  `sn`: `bWmLoHqb7PWltKEwla`
- `name`: `看雪学院`
  `sn`: `bIkBPy79c06kZKX77y`

## 经验总结

这个算法没有正确使用 `RSA`，因而没有任何挑战性；但让我们知道了 `RSA` 的明码比较以及几个 `FGInt` 库函数的使用方法，呵呵。

## 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
