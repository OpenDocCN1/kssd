
# 文件整容专家 3.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue095.htm`](https://www.pediy.com/kssd/pediy04/kanxue095.htm)

文件整容专家 3.0 暴破过程（转载请保存原文完整）

初次写破解文章，难免有不足之处，望多多包涵。

软件功能：

- 改变文件夹的图标
- 加上悬挂式说明文字
- 在打开文件夹时显示消息，或要求输入密码
- 一进入文件夹直接跳转到某个网址
- 添加文件夹窗口的背景音乐、背景图片
- 在文件夹窗口里加入文字、字屏、超级链接、水平线、图片、动画等
- 改掉关于文件属性说明的文字
- 设定文件列表的前景色、背景色、背景图片
- 在桌面、“发送到”菜单等处自动生成该文件夹的快捷方式

功能限制：说是 30 天试用，好像没有时间限制。

文章作者：`eCool`

破解时间：2001/10/10

## 暴破过程

本来用 `SmartCheck` 破解，已经找出注册码，下面用十六进制表示：

```
32 E4 02 00 37 E4 02 00 39 E4 02 00 35 E4 02 00 3D E4 00
```

可惜我不知道为什么怎么输入 ASCII 码为 `02` 的字符，所以无法注册，没有办法，只好暴破了。

（注册码找错否？望高手指点）

运行软件，输入假注册码 `123456`，用 `bpx msvbvm50!__vbastrcmp` 设断，结果拦下：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004328C3(C)
|
:004328D6 8D45B0                lea eax, dword ptr [ebp-50]
:004328D9 50                    push eax
:004328DA 8D45C0                lea eax, dword ptr [ebp-40]
:004328DD 50                    push eax
:004328DE E88E4DFFFF            call 00427671
:004328E3 8BD0                  mov edx, eax
:004328E5 8D4DDC                lea ecx, dword ptr [ebp-24]
* Reference To: MSVBVM50.__vbaStrMove, Ord:0000h
|
:004328E8 E897F2FCFF            Call 00401B84
:004328ED 50                    push eax              <--这里是真注册码
:004328EE FF75E0                push [ebp-20]         <--输入的假注册码
* Reference To: MSVBVM50.__vbaStrCmp, Ord:0000h
|
:004328F1 E80AF2FCFF            Call 00401B00
:004328F6 8BF0                  mov esi, eax
:004328F8 8D45E0                lea eax, dword ptr [ebp-20]
:004328FB F7DE                  neg esi
:004328FD 50                    push eax
:004328FE 8D45DC                lea eax, dword ptr [ebp-24]
:00432901 1BF6                  sbb esi, esi
:00432903 50                    push eax
:00432904 46                    inc esi
:00432905 6A02                  push 00000002
:00432907 F7DE                  neg esi
* Reference To: MSVBVM50.__vbaFreeStrList, Ord:0000h
|
:00432909 E816F2FCFF            Call 00401B24
:0043290E 83C40C                add esp, 0000000C
:00432911 8D45D0                lea eax, dword ptr [ebp-30]
:00432914 50                    push eax
:00432915 8D45D4                lea eax, dword ptr [ebp-2C]
:00432918 50                    push eax
:00432919 8D45D8                lea eax, dword ptr [ebp-28]
:0043291C 50                    push eax
:0043291D 6A03                  push 00000003
* Reference To: MSVBVM50.__vbaFreeObjList, Ord:0000h
|
:0043291F E83CF2FCFF            Call 00401B60
:00432924 83C410                add esp, 00000010
:00432927 8D45B0                lea eax, dword ptr [ebp-50]
:0043292A 50                    push eax
:0043292B 8D45C0                lea eax, dword ptr [ebp-40]
:0043292E 50                    push eax
:0043292F 6A02                  push 00000002
* Reference To: MSVBVM50.__vbaFreeVarList, Ord:0000h
|
:00432931 E85AF2FCFF            Call 00401B90
:00432936 83C40C                add esp, 0000000C
:00432939 B904000280            mov ecx, 80020004
:0043293E 663BF3                cmp si, bx
:00432941 894D98                mov dword ptr [ebp-68], ecx
:00432944 6A0A                  push 0000000A
:00432946 894DA8                mov dword ptr [ebp-58], ecx
:00432949 58                    pop eax
:0043294A 894DB8                mov dword ptr [ebp-48], ecx
:0043294D 894590                mov dword ptr [ebp-70], eax
:00432950 8945A0                mov dword ptr [ebp-60], eax
:00432953 8945B0                mov dword ptr [ebp-50], eax
:00432956 0F8475010000          je 00432AD1       <-----关键跳跃，要跳。改为 jmp 00432AD1
:0043295C 8D5580                lea edx, dword ptr [ebp-80]
:0043295F 8D4DC0                lea ecx, dword ptr [ebp-40]
* Possible StringData Ref from Code Obj ->"鑜孮 b 烺�a"尐`剉鑜孮 0"
|
:00432962 C745882C034100        mov [ebp-78], 0041032C
:00432969 897D80                mov dword ptr [ebp-80], edi
* Reference To: MSVBVM50.__vbaVarDup, Ord:0000h
|
:0043296C E825F2FCFF            Call 00401B96
:00432971 8D4590                lea eax, dword ptr [ebp-70]
:00432974 50                    push eax
:00432975 8D45A0                lea eax, dword ptr [ebp-60]
:00432978 50                    push eax
:00432979 8D45B0                lea eax, dword ptr [ebp-50]
:0043297C 50                    push eax
:0043297D 8D45C0                lea eax, dword ptr [ebp-40]
:00432980 53                    push ebx
:00432981 50                    push eax
* Reference To: MSVBVM50.rtcMsgBox, Ord:0253h
|
:00432982 E815F2FCFF            Call 00401B9C
:00432987 8D4590                lea eax, dword ptr [ebp-70]
:0043298A 50                    push eax
:0043298B 8D45A0                lea eax, dword ptr [ebp-60]
:0043298E 50                    push eax
:0043298F 8D45B0                lea eax, dword ptr [ebp-50]
:00432992 50                    push eax
:00432993 8D45C0                lea eax, dword ptr [ebp-40]
:00432996 50                    push eax
:00432997 6A04                  push 00000004
```

可惜用 `Hiew` 修改后依然提示没有注册，所以程序运行有判断是否注册的代码，好

先用 `bpx msvbvm50!__vbastrcmp`，运行程序，被拦下，按 `F5`，再次被拦下：

```
* Reference To: MSVBVM50.__vbaStrMove, Ord:0000h
|
:00418CD8 E8A78EFEFF            Call 00401B84
:00418CDD FF75E0                push [ebp-20]     <----真注册码
:00418CE0 FF75D8                push [ebp-28]     <----假的，不过是 1111
* Reference To: MSVBVM50.__vbaStrCmp, Ord:0000h
|
:00418CE3 E8188EFEFF            Call 00401B00
:00418CE8 85C0                  test eax, eax
:00418CEA 0F85A1020000          jne 00418F91      <----关键跳跃，不跳，否则 Over。把它 Nop 掉
:00418CF0 FF7508                push [ebp+08]
:00418CF3 8BB5BCFEFFFF          mov esi, dword ptr [ebp+FFFFFEBC]
:00418CF9 66830D38304300FF      or word ptr [00433038], FFFF
:00418D01 FF96FC020000          call dword ptr [esi+000002FC]
:00418D07 50                    push eax
:00418D08 8D45B8                lea eax, dword ptr [ebp-48]
:00418D0B 50                    push eax
```

用 `Hiew` 修改，重新运行，提示已经注册，而且没有注册按钮了。

总结：只要改 `00418CF0` 处的 `FF7508` 改为 `909090` 就可以了，好困啊，睡觉去喽。 `4dd466d0e1e0cc123afbe504da2462ca`
