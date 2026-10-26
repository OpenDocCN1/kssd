
```
00597AAB  |.  7C 1F            |jl short NetUSB_e.00597ACC
00597AAD  |.  8D45 E4          |lea eax,[local.7]
00597AB0  |.  33D2             |xor edx,edx
00597AB2  |.  8AD3             |mov dl,bl
00597AB4  |.  83C2 3C          |add edx,3C
00597AB7  |.  E8 ECD0E6FF      |call NetUSB_e.00404BA8
00597ABC  |.  8B55 E4          |mov edx,[local.7]
00597ABF  |.  8D45 E8          |lea eax,[local.6]
00597AC2  |.  E8 D1D1E6FF      |call NetUSB_e.00404C98
00597AC7  |.  FF45 F0          |inc [local.4]
00597ACA  |.  EB 39            |jmp short NetUSB_e.00597B05
00597ACC  |>  8D45 E0          |lea eax,[local.8]
00597ACF  |.  33D2             |xor edx,edx
00597AD1  |.  8AD3             |mov dl,bl
00597AD3  |.  83C2 3C          |add edx,3C
00597AD6  |.  E8 CDD0E6FF      |call NetUSB_e.00404BA8
00597ADB  |.  8B55 E0          |mov edx,[local.8]
00597ADE  |.  8D45 E8          |lea eax,[local.6]
00597AE1  |.  E8 B2D1E6FF      |call NetUSB_e.00404C98
00597AE6  |.  8D45 DC          |lea eax,[local.9]
00597AE9  |.  33D2             |xor edx,edx
00597AEB  |.  8A55 EF          |mov dl,byte ptr ss:[ebp-11]
00597AEE  |.  83C2 3C          |add edx,3C
00597AF1  |.  E8 B2D0E6FF      |call NetUSB_e.00404BA8
00597AF6  |.  8B55 DC          |mov edx,[local.9]
00597AF9  |.  8D45 E8          |lea eax,[local.6]
00597AFC  |.  E8 97D1E6FF      |call NetUSB_e.00404C98
00597B01  |.  8345 F0 02       |add [local.4],2
00597B05  |>  33FF             |xor edi,edi
00597B07  |.  C645 EF 00       |mov byte ptr ss:[ebp-11],0
00597B0B  |.  EB 1D            |jmp short NetUSB_e.00597B2A
00597B0D  |>  8D45 D8          |lea eax,[local.10]
00597B10  |.  33D2             |xor edx,edx
00597B12  |.  8AD3             |mov dl,bl
00597B14  |.  83C2 3C          |add edx,3C
00597B17  |.  E8 8CD0E6FF      |call NetUSB_e.00404BA8
00597B1C  |.  8B55 D8          |mov edx,[local.10]
00597B1F  |.  8D45 E8          |lea eax,[local.6]
00597B22  |.  E8 71D1E6FF      |call NetUSB_e.00404C98
00597B27  |.  FF45 F0          |inc [local.4]
00597B2A  |>  46               |inc esi
00597B2B  |.  3B75 F4          |cmp esi,[local.3]
00597B2E  |.^ 0F8C 2CFFFFFF    \jl NetUSB_e.00597A60
00597B34  |>  85FF             test edi,edi
00597B36  |.  7E 1E            jle short NetUSB_e.00597B56
00597B38  |.  8D45 D4          lea eax,[local.11]
00597B3B  |.  33D2             xor edx,edx
00597B3D  |.  8A55 EF          mov dl,byte ptr ss:[ebp-11]
00597B40  |.  83C2 3C          add edx,3C
00597B43  |.  E8 60D0E6FF      call NetUSB_e.00404BA8
00597B48  |.  8B55 D4          mov edx,[local.11]
00597B4B  |.  8D45 E8          lea eax,[local.6]
00597B4E  |.  E8 45D1E6FF      call NetUSB_e.00404C98
00597B53  |.  FF45 F0          inc [local.4]
00597B56  |>  8D45 E8          lea eax,[local.6]
00597B59  |.  8B55 F0          mov edx,[local.4]
00597B5C  |.  E8 B3D4E6FF      call NetUSB_e.00405014
00597B61  |.  8B45 F8          mov eax,[local.2]
00597B64  |.  8B55 E8          mov edx,[local.6]
00597B67  |.  E8 B0CEE6FF      call NetUSB_e.00404A1C
00597B6C  |.  33C0             xor eax,eax
00597B6E  |.  5A               pop edx
00597B6F  |.  59               pop ecx
00597B70  |.  59               pop ecx
00597B71  |.  64:8910          mov dword ptr fs:[eax],edx
00597B74  |.  68 967B5900      push NetUSB_e.00597B96
00597B79  |>  8D45 D4          lea eax,[local.11]
00597B7C  |.  BA 06000000      mov edx,6
00597B81  |.  E8 66CEE6FF      call NetUSB_e.004049EC
00597B86  |.  8D45 FC          lea eax,[local.1]
00597B89  |.  E8 3ACEE6FF      call NetUSB_e.004049C8
00597B8E  \.  C3               retn
00597B8F   .^ E9 9CC7E6FF      jmp NetUSB_e.00404330
00597B94   .^ EB E3            jmp short NetUSB_e.00597B79
00597B96   .  5F               pop edi
00597B97   .  5E               pop esi
00597B98   .  5B               pop ebx
00597B99   .  8BE5             mov esp,ebp
00597B9B   .  5D               pop ebp
00597B9C   .  C3               retn
```

---

这样一来，我们可以先解密“注册信息”，得到：

`网络**|随机数|用户名|HDSerialNumber|机器码|`

再参考前面所描述的，可以很方便的求的软件的注册码了。

注册机 Delphi 7.0 原代码（包括获取硬盘物理序列号、Base64、注册成功截图）附件：`NetUSB 2.05.rar`

## 【破解总结】

因为本人不懂一些公开算法，调试算法花了很多时间，惭愧ing……，有以下几点我认为作者保护的比好：

1. 软件的提示信息都经过加密，不能直接找到关键断点。
2. 软件采用`F1()=F2()`的注册方式，不出现明码，内存注册机肯定是做不了了。
3. 值得一提的是，软件的产品名称也参加注册计算，手工脱壳 `Import REConstructor` 修复等容易丢失这不份信息，导致注册机无效。

我机器上的注册信息：

```
硬盘物理序列号:CSH305DAJB16RB
HDSerialNumber：9252FF533905195C
注册码:M_<rHpHtH?]>JOA=LOY?IL
```

验证通过！累死我了…………，写这篇破文花了我N小时!各位看客看在我疲劳的份上………………顶一下^\_^  (本文完)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
