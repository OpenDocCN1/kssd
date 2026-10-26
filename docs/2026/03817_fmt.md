
# 网页特效咖啡豆算法分析

> 网页特效咖啡豆 V2.0
>
> - 软件语言：简体中文
> - 软件类别：共享软件／国产软件／主页制作
> - 运行环境：Win9x/Me/NT/2000/XP/2003
> - 软件大小：1.0MB
> - 软件简介：一款小巧的网页特效增强软件，软件内集成了14大类近300种网页制作常用的javascript特效，本软件拥有QQ风格的界面，界面优美大方，支持SKIN，特别使用了XP风格的菜单，使广大网页制作者很容易上手使用，瞬间美化您的网页。
>
> 下载：[110K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8G2N6$3&6J5k6h3N6Q4x3X3g2U0L8$3#2Q4x3V1k6e0L8$3k6@1N6$3q4J5k6g2)9J5c8W2k6A6k6i4N6Q4x3X3c8e0L8$3k6@1N6$3q4J5k6g2)9J5k6o6x3#2y4K6c8Q4x3X3g2Z5N6r3#2D9](<https://bbs.kanxue.com/elink@bddK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8G2N6$3&6J5k6h3N6Q4x3X3g2U0L8$3#2Q4x3V1k6e0L8$3k6@1N6$3q4J5k6g2)9J5c8W2k6A6k6i4N6Q4x3X3c8e0L8$3k6@1N6$3q4J5k6g2)9J5k6o6x3#2y4K6c8Q4x3X3g2Z5N6r3#2D9>)
>
> 在PYG看到的作业，纯属兴趣！！

## 网页特效咖啡豆算法分析

### 查壳与脱壳

首先 `PEID` 查壳是 `ASPack 2.1 -> Alexey Solodovnikov` 的壳。

用 `qunpack` 轻松脱去。

### 注册验证与断点定位

然后运行程序注册：

用户名：`k99992002`，注册码：`123456789`，然后点“注册认证”，软件没有提示！

用 `w32dsm` 载入查找“错误”，向上找到个跳转来到 `0049A123`。

用 `OD` 载入，来到 `0049A123`，下断在 `0049A0FA 55 push ebp`。

然后运行程序注册：

用户名：`k99992002`，注册码：`1234567`，然后点“注册认证”。程序被断下！

### 分析

接下来我们来分析！

```asm
0049A117     8D55 FC          lea edx,dword ptr ss:[ebp-4]  〈==你输入的用户名
0049A11A     E8 6DE7F6FF      call TheEndTx.0040888C
0049A11F     837D FC 00       cmp dword ptr ss:[ebp-4],0   〈==检查有没有输入用户名
0049A123     75 1C            jnz short TheEndTx.0049A141
0049A125     B8 F0A14900      mov eax,TheEndTx.0049A1F0
0049A12A     6A 10            push 10
0049A12C     B9 08A24900      mov ecx,TheEndTx.0049A208
0049A131     8B15 AC5B4A00    mov edx,dword ptr ds:[4A5BAC]             ; TheEndTx.004A6C30
0049A137     8B12             mov edx,dword ptr ds:[edx]
0049A139     92               xchg eax,edx
0049A13A     E8 F5F8FBFF      call TheEndTx.00459A34
0049A13F     EB 57            jmp short TheEndTx.0049A198
0049A141     8D55 F0          lea edx,dword ptr ss:[ebp-10]
0049A144     8B83 10030000    mov eax,dword ptr ds:[ebx+310]
0049A14A     E8 35F6F9FF      call TheEndTx.00439784
0049A14F     8B45 F0          mov eax,dword ptr ss:[ebp-10]  〈==你输入的注册码
0049A152     8D55 F4          lea edx,dword ptr ss:[ebp-C]
0049A155     E8 32E7F6FF      call TheEndTx.0040888C
0049A15A     837D F4 00       cmp dword ptr ss:[ebp-C],0  〈==检查有没有输入注册码
0049A15E     75 1C            jnz short TheEndTx.0049A17C
0049A160     B8 10A24900      mov eax,TheEndTx.0049A210
0049A165     6A 10            push 10
0049A167     B9 08A24900      mov ecx,TheEndTx.0049A208
0049A16C     8B15 AC5B4A00    mov edx,dword ptr ds:[4A5BAC]             ; TheEndTx.004A6C30
0049A172     8B12             mov edx,dword ptr ds:[edx]
0049A174     92               xchg eax,edx
0049A175     E8 BAF8FBFF      call TheEndTx.00459A34
0049A17A     EB 1C            jmp short TheEndTx.0049A198
0049A17C     8BC3             mov eax,ebx
0049A17E     E8 11030000      call TheEndTx.0049A494   〈==关键CALL，算法call
0049A183     84C0             test al,al    〈==是0的话就over
0049A185     74 09            je short TheEndTx.0049A190
0049A187     8BC3             mov eax,ebx
0049A189     E8 96000000      call TheEndTx.0049A224  〈==注册成功
0049A18E     EB 08            jmp short TheEndTx.0049A198
```

在算法 call `F7` 进去：

```asm
0049A49C     33D2             xor edx,edx
0049A49E     8955 E8          mov dword ptr ss:[ebp-18],edx
0049A4A1     8955 EC          mov dword ptr ss:[ebp-14],edx
0049A4A4     8955 F4          mov dword ptr ss:[ebp-C],edx
0049A4A7     8945 FC          mov dword ptr ss:[ebp-4],eax
0049A4AA     33C0             xor eax,eax
0049A4AC     55               push ebp
0049A4AD     68 93A54900      push TheEndTx.0049A593
0049A4B2     64:FF30          push dword ptr fs:[eax]
0049A4B5     64:8920          mov dword ptr fs:[eax],esp
0049A4B8     33DB             xor ebx,ebx
0049A4BA     8D55 F4          lea edx,dword ptr ss:[ebp-C]
0049A4BD     8B45 FC          mov eax,dword ptr ss:[ebp-4]
0049A4C0     8B80 FC020000    mov eax,dword ptr ds:[eax+2FC]
0049A4C6     E8 B9F2F9FF      call TheEndTx.00439784
0049A4CB     8B45 F4          mov eax,dword ptr ss:[ebp-C]   〈==用户名入EAX
0049A4CE     E8 E9A1F6FF      call TheEndTx.004046BC   〈=取用户名长度，我的是9
0049A4D3     8BF0             mov esi,eax
0049A4D5     85F6             test esi,esi
0049A4D7     7E 38            jle short TheEndTx.0049A511
0049A4D9     C745 F0 01000000 mov dword ptr ss:[ebp-10],1
0049A4E0     8D45 EC          lea eax,dword ptr ss:[ebp-14]
0049A4E3     50               push eax
0049A4E4     B9 01000000      mov ecx,1
0049A4E9     8B55 F0          mov edx,dword ptr ss:[ebp-10]
0049A4EC     8B45 F4          mov eax,dword ptr ss:[ebp-C]
0049A4EF     E8 20A4F6FF      call TheEndTx.00404914
0049A4F4     8B45 EC          mov eax,dword ptr ss:[ebp-14]
0049A4F7     E8 B8A3F6FF      call TheEndTx.004048B4
0049A4FC     8A00             mov al,byte ptr ds:[eax]  〈=取出用户名的第一个字节k,进行字节操作
0049A4FE     25 FF000000      and eax,0FF  〈=得到的eax和OFF做与运算
0049A503     03D8             add ebx,eax  〈=把eax加到ebx
0049A505     81F3 05FA0B00    xor ebx,0BFA05 〈=把得到的ebx和0BFA05进行异或运算
0049A50B     FF45 F0          inc dword ptr ss:[ebp-10]
0049A50E     4E               dec esi  〈=取完一个减少一次
0049A50F   ^ 75 CF            jnz short TheEndTx.0049A4E0 〈=依次取用户名,直到取完！
0049A511     A1 386D4A00      mov eax,dword ptr ds:[4A6D38]〈=取出注册申请码
0049A516     8BD0             mov edx,eax 〈=放入edx
0049A518     C1E0 04          shl eax,4   〈=逻辑左移4位
0049A51B     03C2             add eax,edx  〈=把edx加到eax
0049A51D     03D8             add ebx,eax    〈=把eax加到ebx
0049A51F     81C3 D4A31300    add ebx,13A3D4   〈=ebx加上12A3D4
0049A525     81F3 8DED5900    xor ebx,59ED8D    〈=把得到的dex和59ED8D异或
0049A52B     8D55 E8          lea edx,dword ptr ss:[ebp-18]
0049A52E     8B45 FC          mov eax,dword ptr ss:[ebp-4]
0049A531     8B80 10030000    mov eax,dword ptr ds:[eax+310]
0049A537     E8 48F2F9FF      call TheEndTx.00439784
0049A53C     8B45 E8          mov eax,dword ptr ss:[ebp-18] 〈==取出假码，放入eax
0049A53F     E8 ACE5F6FF      call TheEndTx.00408AF0  〈=这里将注册码转化DWORD值
0049A544     8BF3             mov esi,ebx 把得到的ebx放入esi
0049A546     81F6 2473C400    xor esi,0C47324 在把esi和0c47324异或
0049A54C     3BC6             cmp eax,esi 〈===比较两个注册码这里的esi就是真码了
0049A54E     75 19            jnz short TheEndTx.0049A569  〈==不相等就over
0049A550     C645 FB 01       mov byte ptr ss:[ebp-5],1 〈==相等就成功！
0049A554     B8 306D4A00      mov eax,TheEndTx.004A6D30
```

![](http://imgphoto.21cn.com/albums/photos/1/2/5/12591405/1116222119.gif)

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
