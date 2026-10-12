
# 多层次破解某象棋软件

> 原文：[`www.pediy.com/kssd/pediy12/128409.html`](https://www.pediy.com/kssd/pediy12/128409.html)

## 多层次破解某象棋软件（申请邀请码）

### 软件信息

- **软件名称**：中国象棋 V2.0
- **软件大小**：908KB
- **软件语言**：中文
- **软件类别**：国内/共享版
- **运行环境**：`win9x/me/NT/XP/2003`
- **下载方式**：网上搜索

首先说明，本人刚学破解几个月的菜鸟，感觉破解这门学问太深了，也想通过论去继续学习，模访网上的文章写的，个人感觉内容还是很丰富的，希望大家不要见笑。

## 方法一：清除软件使用次数限制（两种途径）

### 1. 修改注册表基准日期

首先查壳，运行 `PEID`，载入中国象棋 V2.0，无壳，如下图：

![](img/bcdf20ec85c48cf003dc205958dcf1e7.jpg)

运行软件，启动时有注册提示窗口和 30 天试用信息，如下图：

![](img/9e48268f22853e1050e759e3c5ff305a.jpg)

点击现在注册，注册软件，发现有错误提示窗口，如下图：

![](img/468e28ad8d758b71e583716176464766.jpg)

点击开始运行，输入 `regedit`，打开注册表，点击编辑查找，输入 `chess`：

![](img/b8fbda90de99f4b2246784002982684e.jpg)

点击查找下一个，`F3` 继续，来到下图处：

![](img/ef6960817a2aa539c73cb609fef5a976.jpg)

观察 `DayOfUsed` 键的数据似乎与时间有关，将时间修改为 `2011.1.9` 试一下，在 `DayOfUsed` 键上单击鼠标右键修改，`2011.1.19` 修改为 `2011.1.9`：

![](img/4cabceaeb78b7f2628c8920350e39ef0.jpg)

重新运行软件，发现软件使用天数变为 11 天：

![](img/803a4de718feacfb4654e8a8d1fd0b90.jpg)

可以肯定，该键值的数据就是计算使用天数的基准日期，将 `2011.1.19` 改为 `3011.1.19`，可以达到永久试用的目的。

### 2. 跳过试用天数计算代码

还有一种思路也可达到永久试用的目的，既然软件有计算试用天数的程序，我们只要跳过这部分不就可以达到永久试用的目的吗？

`OD` 载入软件，既然计算试用天数要用到注册表中的基准时间，可下 `bp RegOpenKeyExA` 断点，按 `F9` 运行，观察堆栈，`F9` 继续运行，直到出现 `GZ_POWER_CHESS` 注册表项，如图：

![](img/2392c7ce0417760f45937462b54f4529.jpg)

`Alt+F9` 执行到用户代码，

```
00408BAA    8B4424 10       mov eax,dword ptr ss:[esp+10]  ，返回到此处
00408BAE    3BC3            cmp eax,ebx
00408BB0    0F84 FB000000   je ChineseC.00408CB1        ，关键跳，修改此处正好跳过计算试用天数
00408BB6    8D4C24 14       lea ecx,dword ptr ss:[esp+14]
00408BBA    51              push ecx
00408BBB    68 3F000F00     push 0F003F
00408BC0    53              push ebx
00408BC1    68 20054400  push ChineseC.00440520          ; set
00408BC6    50              push eax
00408BC7    FFD6            call esi
00408BC9    8B4424 14       mov eax,dword ptr ss:[esp+14]
00408BCD    3BC3            cmp eax,ebx
00408BCF    0F84 DA010000   je ChineseC.00408DAF
00408BD5    8D5424 2C       lea edx,dword ptr ss:[esp+2C]
00408BD9    8D4C24 48       lea ecx,dword ptr ss:[esp+48]
00408BDD    52              push edx
00408BDE    51              push ecx
00408BDF    53              push ebx
00408BE0    53              push ebx
00408BE1    68 14054400  push ChineseC.00440514      ; dayofused
00408BE6    50              push eax
00408BE7    C74424 48 00020>mov dword ptr ss:[esp+48],200
00408BEF    C74424 44 1E000>mov dword ptr ss:[esp+44],1E
00408BF7    FFD5            call ebp
00408BF9    85C0            test eax,eax
00408BFB    75 1A           jnz short ChineseC.00408C17
00408BFD    51              push ecx
00408BFE    8D5424 4C       lea edx,dword ptr ss:[esp+4C]
00408C02    8BCC            mov ecx,esp
00408C04    896424 38       mov dword ptr ss:[esp+38],esp
00408C08    52              push edx
00408C09    E8 9CB30200     call <jmp.&MFC42.#537>
00408C0E    8B4C24 20       mov ecx,dword ptr ss:[esp+20]
00408C12    E8 F9080000     call ChineseC.00409510
00408C17    8B5424 14       mov edx,dword ptr ss:[esp+14]
00408C1B    8D4424 2C       lea eax,dword ptr ss:[esp+2C]
00408C1F    8D4C24 48       lea ecx,dword ptr ss:[esp+48]
00408C23    50              push eax
00408C24    51              push ecx
00408C25    53              push ebx
00408C26    53              push ebx
00408C27    68 08054400  push ChineseC.00440508             ; chesscount
00408C2C    52              push edx
00408C2D    FFD5            call ebp
00408C2F    85C0            test eax,eax
00408C31    75 18           jnz short ChineseC.00408C4B
00408C33    8D4424 48       lea eax,dword ptr ss:[esp+48]
00408C37    50              push eax
00408C38   FF15 2C874300   call dword ptr ds:[<&MSVCRT.atol>]      ; msvcrt.atol
00408C3E    8B4C24 20       mov ecx,dword ptr ss:[esp+20]
00408C42    83C4 04         add esp,4
00408C45    8981 C4000000   mov dword ptr ds:[ecx+C4],eax
00408C4B    8B4C24 14       mov ecx,dword ptr ss:[esp+14]
00408C4F    8D5424 30       lea edx,dword ptr ss:[esp+30]
00408C53    8D8424 68020000 lea eax,dword ptr ss:[esp+268]
00408C5A    52              push edx
00408C5B    50              push eax
00408C5C    53              push ebx
00408C5D    53              push ebx
00408C5E    68 FC044400 push ChineseC.004404FC           ; registernum
00408C63    51              push ecx
00408C64    FFD5            call ebp
00408C66    3BC3            cmp eax,ebx
00408C68    74 14           je short ChineseC.00408C7E
00408C6A    8B5424 10       mov edx,dword ptr ss:[esp+10]
00408C6E  8B35 14804300   mov esi,dword ptr ds:[<&ADVAPI32.RegCloseKey>]         ; ADVAPI32.RegCloseKey
00408C74    52              push edx
00408C75    FFD6            call esi
00408C77    8B4424 14       mov eax,dword ptr ss:[esp+14]
00408C7B    50              push eax
00408C7C    EB 31           jmp short ChineseC.00408CAF
00408C7E    8B4424 14       mov eax,dword ptr ss:[esp+14]
00408C82    8D4C24 28       lea ecx,dword ptr ss:[esp+28]
00408C86    8D9424 68010000 lea edx,dword ptr ss:[esp+168]
00408C8D    51              push ecx
00408C8E    52              push edx
00408C8F    53              push ebx
00408C90    53              push ebx
00408C91    68 F0044400  push ChineseC.004404F0            ; username
00408C96    50              push eax
00408C97    FFD5            call ebp
00408C99    3BC3            cmp eax,ebx
00408C9B    74 2D           je short ChineseC.00408CCA
00408C9D    8B4C24 10       mov ecx,dword ptr ss:[esp+10]
00408CA1 8B35 14804300   mov esi,dword ptr ds:[<&ADVAPI32.RegCloseKey>]         ; ADVAPI32.RegCloseKey
00408CA7    51              push ecx
00408CA8    FFD6            call esi
00408CAA    8B5424 14       mov edx,dword ptr ss:[esp+14]
00408CAE    52              push edx
00408CAF    FFD6            call esi
00408CB1    8D4C24 18       lea ecx,dword ptr ss:[esp+18]
00408CB5    C78424 70040000>mov dword ptr ss:[esp+470],-1
```

在 `00408BB0` 这行单击鼠标右键，选择汇编，将 `je` 修改为 `jmp`，

![](img/425cbd8eea1880a1b8c93429a03e1297.jpg)

在汇编窗口单击鼠标右键，选择复制到可执行文件所有修改，然后选择全部复制，

![](img/9843a5b6759bbd700a99c6bc808feb17.jpg)

单击鼠标右键，选择保存文件，OK！软件可以永久试用，不信的话可以修改注册表 `DayOfUsed` 项的数据，发现试用天数不变。

## 方法二：爆破

既然注册时有错误提示，可以通过查找字符串来查找按钮事件，`OD` 载入软件

![](img/21ad9eb21de564b67b1360b161750477.jpg)

在汇编窗口单击鼠标右键，选择 Ultra 字符串参考 1.查找 ASCII，

![](img/d6a45b4c4eeeb41007682575c1d859b9.jpg)

双击“姓名和注册码不匹配，注册不成功！”这一行，来到主程序窗口，

![](img/3a752fa406fd7eaf9682549477f86ae9.jpg)

往上观察，发现了一些关键东西：

```
00430B67    E8 24E9FFFF     call ChineseC.0042F490      关键 CALL
00430B6C    84C0            test al,al                  关键比较
00430B6E    6A 40           push 40
00430B70    68 90044400  push ChineseC.00440490         ; gzpower chess
00430B75    75 21           jnz short ChineseC.00430B98    关键跳
00430B77    68 9C1A4400 push ChineseC.00441A9C  ; 姓名和注册码不匹配，注册不成功!
00430B7C    8BCE            mov ecx,esi
```

在 `00430B70` 这行单击鼠标右键，选择汇编，将 `jnz` 修改为 `jz`，在汇编窗口单击鼠标右键，选择复制到可执行文件所有修改，然后选择全部复制，

![](img/40b02a24ef87867731ffe590bf99664b.jpg)

单击鼠标右键，选择保存文件。

重启中国象棋 V2.0，仍有注册提示窗口，但可以注册成功，

![](img/2c420dce41034475b151e27e4fafd849.jpg)

重启后仍然是未注册版，因些必须去掉提示注册的 NAG 窗口中。

`OD` 载入软件，用 `F12` 暂停法去掉 NAG 窗口，`F9` 运行软件，然后到反汇编窗口，按 `F12` 键，`Alt+F9` 执行到用户代码，

![](img/9ab44bed03f36640d17ee72e9e12aca9.jpg)

`00408499` 这一行是调用注册提示的 NAG 窗口，往上查找，发现 `00408461` 这一行正好跳过调用注册提示的 NAG 窗口，在 `00408461` 这一行单击鼠标右键，选择汇编，将 `jnz` 改为 `jmp`，

![](img/fbfb3bda7d5b4508662e1cb0a24dd89f.jpg)

在汇编窗口单击鼠标右键，选择复制到可执行文件所有修改，然后选择全部复制，然后单击鼠标右键，选择保存文件。

重启中国象棋 V2.0，没有注册提示窗口，直接进入软件，如下图：

![](img/d0ae039b2627152836ef148d3bfeef4b.jpg)

仍然有未注册字样，但不影响使用，也可将未注册字样去掉，`OD` 重新载入修改后中国象棋 V2.0，在汇编窗口单击鼠标右键，选择 Ultra 字符串参考 1.查找 ASCII，

![](img/734b5416e80ac66035e88a95448cfc4f.jpg)

双击“（未注册）”这一行，来到主程序窗口。
