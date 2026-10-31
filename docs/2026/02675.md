
# 网吧维护系统1.03 (5千字)

今天在`CNCG`的主页上看到发布`CNCG`的第一个`crackme`，呵呵，我乃菜鸟一个，正好试试我的功力如何，立马拉下来了，一看个头还挺大的，我用小猫:)安装之后随便操作几下后，点注册，发现是靠机器码的，不要用户名，随便输入一个注册码，点确定，弹出一个框，说要重新启动软件来验证注册码是否正确，重新运行软件，没有发现什么异样，注册按钮也不见了，我还以为瞎猫碰上死耗子了，点下一步，弹出一个框，说注册码错误，与作者联系，KAO，原来是这样，又运行了几遍，发现出现这个对话框是不固定的，有时候第一下点下一步按钮对话框就出来了，有时候要点两次下一步才会出现这个对话框，KAO，这么麻烦，先看看有壳没有，`pecompact1.4`或 above的壳，用`unpecomact`脱了它，运行脱壳之后的程序，进去就退出来了，KAO，这里也防了，跟那个`sitman`一样，在`trw`中下`bpx` `exitprocess`，运行脱壳后的程序，被`trw`断下来，通过分析、查找和尝试，招到了那个调用这个函数的`call`，将它`nop`掉，在运行，程序死在那里了，看来整个`call`全`nop`掉不行，那就找跳转，无奈发现里面的`call`实在太多，跳转也多，而且有些`call`实际上是跳转，找了几个跳转试了试，不是非法操作就是死机，可能是我没有找对吧，总之烦死了，加上我的汇编又烂，看来爆破是没有门了，还是老老实实的追注册码吧，但是下什么断点来断下程序呢，想到点下一步时会弹出一个错误的框来，就下与它有关的断点吧，结果我试了一个又一个，KAO，竟然没有一个能断下来的，真是活见鬼了。用黄金加强版反反汇编的`w32dsm`反编译脱壳后的文件，里面竟然什么字符串都没有，天啊，还反反反汇编，我倒。到这里我都快黔驴技穷了，随手又察看了一下脱壳后的文件，发现是用`delphi`写的，哈哈，好说好说，用`dede`反之，哈哈，终于显出原型来了，一番分析后，直达要害部位，下面是要害部位的部分代码：

```
0045CB89  push    dword ptr fs:[eax]
0045CB8C  mov    fs:[eax], esp
0045CB8F  mov    edx, $80000002
0045CB94  mov    eax, [ebp-$18]
0045CB97  call    0044EE20
0045CB9C  mov    cl, $01
* Possible String Reference to: '\Software\New Sun\网吧管理员'
0045CB9E  mov    edx, $0045D170
0045CBA3  mov    eax, [ebp-$18]
0045CBA6  call    0044EE84
0045CBAB  mov    eax, dword ptr [$46088C]
0045CBB0  cmp    eax, [$460884]
0045CBB6  jz      0045CBCB
0045CBB8  mov    eax, [ebp-$04]
* Reference to control TForm1.s3 : TLabel
0045CBBB  mov    eax, [eax+$02F4]
* Reference to field TLabel.OFFS_0047
0045CBC1  cmp    byte ptr [eax+$47], $00
0045CBC5  jnz    0045CBCB
0045CBC7  xor    eax, eax
0045CBC9  jmp    0045CBCD
0045CBCB  mov    al, $01
0045CBCD  cmp    al, $01
0045CBCF  jnz    0045CCBB    //这里如果跳走的话，你就成功了
0045CBD5  lea    ecx, [ebp-$1C]
0045CBD8  mov    dl, $01
0045CBDA  mov    eax, [ebp-$04]
* Reference to : TForm1.luomingfor()
0045CBDD  call    0045E0A4
0045CBE2  mov    eax, [ebp-$1C]
0045CBE5  push    eax
0045CBE6  lea    edx, [ebp-$20]
0045CBE9  mov    eax, [ebp-$04]
* Reference to control TForm1.putregcode : TEdit
0045CBEC  mov    eax, [eax+$033C]
0045CBF2  call    0042A80C
0045CBF7  mov    edx, [ebp-$20]
0045CBFA  mov    eax, [ebp-$04]
0045CBFD  pop    ecx
* Reference to : TForm1.ok()
0045CBFE  call    0045CA54    //哈哈，经典对比组合，想知道怎么对比就跟进去，如果只是要注册码的话，就D
ECX吧，你看到了什么？怎么样，你成功了吧。 :)
0045CC03  test    al, al
0045CC05  jz      0045CC56
0045CC07  mov    eax, [ebp-$04]
* Reference to control TForm1.regestg : TGroupBox
0045CC0A  mov    eax, [eax+$032C]
0045CC10  xor    edx, edx
0045CC12  call    0042A724
0045CC17  mov    eax, [ebp-$04]
* Reference to control TForm1.ireg : TButton
0045CC1A  mov    eax, [eax+$0328]
0045CC20  xor    edx, edx
0045CC22  call    0042A724
0045CC27  lea    edx, [ebp-$24]
0045CC2A  mov    eax, [ebp-$04]
* Reference to control TForm1.putregcode : TEdit
0045CC2D  mov    eax, [eax+$033C]
0045CC33  call    0042A80C
0045CC38  mov    ecx, [ebp-$24]
* Possible String Reference to: 'regcodebak'
0045CC3B  mov    edx, $0045D198
0045CC40  mov    eax, [ebp-$18]
0045CC43  call    0044F020
0045CC48  mov    eax, dword ptr [$460880]
0045CC4D  mov    dword ptr [eax+$0C], $0000000B
0045CC54  jmp    0045CCBB          //在这里跳走的话，才是光明大道
0045CC56  xor    ecx, ecx
* Possible String Reference to: 'chereg'
0045CC58  mov    edx, $0045D1AC
0045CC5D  mov    eax, [ebp-$18]
0045CC60  call    0044F108
0045CC65  mov    eax, dword ptr [$460880]
0045CC6A  mov    dword ptr [eax+$0C], $00000003
0045CC71  mov    eax, [ebp-$04]
* Reference to control TForm1.ireg : TButton
0045CC74  mov    eax, [eax+$0328]
0045CC7A  mov    dl, $01
0045CC7C  call    0042A724
0045CC81  mov    eax, [ebp-$04]
* Reference to control TForm1.putregcode : TEdit
0045CC84  mov    eax, [eax+$033C]
0045CC8A  xor    edx, edx
0045CC8C  call    0042A83C
* Possible String Reference to: '对不起,你输入的注册码有问题！如要注
|
          册请与我联系电话:0732-5571778  0732
|
          -2372543Email:icqlm@sohu.com'
|
0045CC91  mov    eax, $0045D1BC
0045CC96  call    0044CF84
0045CC9B  mov    eax, dword ptr [$460880]
0045CCA0  mov    dword ptr [eax+$0C], $00000001
0045CCA7  mov    eax, dword ptr [$45FDEC]
0045CCAC  mov    eax, [eax]
0045CCAE  call    004484AC
0045CCB3  mov    eax, [ebp-$04]
0045CCB6  call    00445184
0045CCBB  xor    eax, eax
0045CCBD  pop    edx
0045CCBE  pop    ecx
0045CCBF  pop    ecx
0045CCC0  mov    fs:[eax], edx
0045CCC3  push    $0045CCE0
0045CCC8  mov    eax, [ebp-$18]
0045CCCB  call    0044EDF0
0045CCD0  mov    eax, [ebp-$18]
0045CCD3  call    00402EDC
0045CCD8  ret
```

最后整理一下：

我的机器码：`77207838775-7009955`

我的注册码：`67678389907`

如果你还想来一遍的话，在注册表中招到下面这两个键值：

`[HKEY_LOCAL_MACHINE\Software\New Sun\网吧管理员]`

`"ifreg"=""`          将它的值删掉

`[HKEY_LOCAL_MACHINE\Software\New Sun\网吧管理员]`

`"regcodebak"=""`    将它的值删掉

你就又可以注册了！

  `cracker`：Turkey/灭害灵

  2001.12.16@02：33

转载请保持完整性，谢谢！！
