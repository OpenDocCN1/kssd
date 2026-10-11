
# 星空电影院 6.01 简单算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-185.htm`](https://www.pediy.com/kssd/pediy07/pediy7-185.htm)

- 软件下载地址：http://vvvvv.cn/zhencui/xkdy.htm
- 软件大小：`2457 KB`
- 软件语言：中文
- 软件类别：共享版
- 应用平台：`windows`

## 【软件简介】

是娱乐所属在线软件，包罗了全国和海外引进的 `3000` 多部动画片、电影、港台、大陆、日韩、欧美电影，并设为独特的秘密观看区，足以使您一饱眼福。`1000M` 光纤接入，`IBM` 专业服务器组，提供在线观看使您即刻体验极速的全方位视觉感受。关于软件的更多特色介绍，速度快质量高的精彩视频，专人实时跟踪更新升级，本软件具有自动升级功能。通过独特的合作资源，提供超过 `400` 套超高清晰的卫星电视节目，高速 `HBO` 大片，`BBC` 新闻，日本 `MM`、港台凤凰、华娱、星空、`TVB` 系列，国内中央及各省卫星电视节目。

## 【软件限制】

功能限制

## 【作者声明】

初学 `Crack`，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

## 【破解工具】

`Ollydbg`，`W32DASM`

---

## 【过程】

用 `PEID` 查壳，发现是用 `DELPHI` 写的，没有加壳的，呵呵~~作者真是好人，关照像我这样的菜鸟。

运行软件输入试练码看看出错提示，然后用 `W32dasm` 反汇编找到出错提示的地方，汇编代码如下：

```
:004E0E72  741E             je 004E0E92
:004E0E74  8D55EC           lea edx, dword ptr [ebp-14]
:004E0E77  8B45FC           mov eax, dword ptr [ebp-04]
:004E0E7A  8B8038030000     mov eax, dword ptr [eax+00000338]
:004E0E80  E84765F6FF       call 004473CC
:004E0E85  8B45EC           mov eax, dword ptr [ebp-14]
:004E0E88  E80736F2FF       call 00404494
:004E0E8D  83F808           cmp eax, 00000008   //判断输入注册码的注册码位数是否小于等于 8 位，大于就无效
:004E0E90  7E0F             jle 004E0EA1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E0E72(C)
|
* Possible StringData Ref from Code Obj ->" 您输入的注册码无效，请重新输入。"

:004E0E92  B8B8104E00       mov eax, 004E10B8
:004E0E97  E8D0FCF4FF       call 00430B6C
:004E0E9C  E9A4010000       jmp 004E1045

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E0E90(C)
|

:004E0EA1  8D45E4           lea eax, dword ptr [ebp-1C]   //如果不大于 8 位就跳到这来，以下为注册码验证的地方
                                                            //可以用 OD 载入后在这个地方下断点，跟踪下面一段看看
:004E0EA4  50               push eax                     //按热键 CTRL+G，然后输入 4E0EA1 就可以到达这个地方了
:004E0EA5  8D55E0           lea edx, dword ptr [ebp-20]
:004E0EA8  8B45FC           mov eax, dword ptr [ebp-04]
:004E0EAB  8B803C030000     mov eax, dword ptr [eax+0000033C]
:004E0EB1  E81665F6FF       call 004473CC
:004E0EB6  8B45E0           mov eax, dword ptr [ebp-20]   //跟踪时发现机器码在这里出现了，传入 EAX，EAX=
:004E0EB9  B906000000       mov ecx, 00000006             //嘿嘿，这个 6 有用，传用 ECX 准备下一步动作，看下面就明白了
:004E0EBE  BA01000000       mov edx, 00000001
:004E0EC3  E82C38F2FF       call 004046F4                  //此 CALL 的作用是取得机器码的前 6 位，知道上面那个 6 的用处了吧
:004E0EC8  8B4DE4           mov ecx, dword ptr [ebp-1C]    //将取得的机器码前 6 位传入 ECX
:004E0ECB  8D45E8           lea eax, dword ptr [ebp-18]
:004E0ECE  BAE4104E00       mov edx, 004E10E4
:004E0ED3  E80836F2FF       call 004044E0
:004E0ED8  8B45E8           mov eax, dword ptr [ebp-18]
:004E0EDB  E8507EF2FF       call 00408D30
:004E0EE0  8BF0             mov esi, eax                   //将机器码前 6 位的字符串转换成 16 进制后存放在 ESI 中
:004E0EE2  33C0             xor eax, eax                   //清空
:004E0EE4  55               push ebp
:004E0EE5  68FC0F4E00       push 004E0FFC
:004E0EEA  64FF30           push dword ptr fs:[eax]
:004E0EED  648920           mov dword ptr fs:[eax], esp
:004E0EF0  8D55DC           lea edx, dword ptr [ebp-24]
:004E0EF3  8B45FC           mov eax, dword ptr [ebp-04]
:004E0EF6  8B8038030000     mov eax, dword ptr [eax+00000338]
:004E0EFC  E8CB64F6FF       call 004473CC
:004E0F01  8B45DC           mov eax, dword ptr [ebp-24]    //试练码（你输入的假注册码）出现，将其传入 EAX
:004E0F04  E8277EF2FF       call 00408D30                   //此 CALL 的作用是将试练码转换成 16 进制
:004E0F09  8BD8             mov ebx, eax                    //转换后传入 EBX
:004E0F0B  8BC3             mov eax, ebx                    //????又放回 EAX，不知道搞什么
:004E0F0D  2BC6             sub eax, esi                    //EAX=EAX-ESI
:004E0F0F  3B05D4CA5700     cmp eax, dword ptr [0057CAD4]   //比较 EAX 与 dword ptr [0057CAD4]中的值是否一样。
                                                            //dword ptr [0057CAD4]里存放着一个常量，调试时
                                                            //可以看见常量值是 1B669
:004E0F15  7438             je 004E0F4F                     //EAX 是否等于 1B669，不相等就 OVER 了

* Possible StringData Ref from Code Obj ->" 您输入的注册码 "

:004E0F17  68F0104E00       push 004E10F0
:004E0F1C  8D55D4           lea edx, dword ptr [ebp-2C]
:004E0F1F  8B45FC           mov eax, dword ptr [ebp-04]
:004E0F22  8B8038030000     mov eax, dword ptr [eax+00000338]
:004E0F28  E89F64F6FF       call 004473CC
:004E0F2D  FF75D4           push [ebp-2C]

* Possible StringData Ref from Code Obj ->" 不正确，请与作者联系。"

:004E0F30  680C114E00       push 004E110C
:004E0F35  8D45D8           lea eax, dword ptr [ebp-28]
:004E0F38  BA03000000       mov edx, 00000003
:004E0F3D  E81236F2FF       call 00404554
:004E0F42  8B45D8           mov eax, dword ptr [ebp-28]
:004E0F45  E822FCF4FF       call 00430B6C
:004E0F4A  E9A3000000       jmp 004E0FF2
```

## 总结

从上面的分析可以知道这个算法很简单就是：

- 注册码的十六进制 = 机器码前 6 位的十六进制 + `1B669`
- 注册码就是机器码前 6 位的十六进制 + `1B669` 然后化成 10 进制

很久没有玩破解，都不会使用这些调试工具了，呵呵~~不过还好，竟然遇上这个算法简单的软件，以是写成一篇文章以供初学者学习

`4dd466d0e1e0cc123afbe504da2462ca`
