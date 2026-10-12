
# FileRecoveryAngel 算法分析+注册机

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-23.htm`](https://www.pediy.com/kssd/pediy07/pediy7-23.htm)

破文作者：ZabShow
破解平台：WinXP
软件名称：FileRecoveryAngel
软件大小：777 KB
软件语言：英文
软件类别：国外软件 / 共享版 / 数据备份
应用平台：Win9x/NT/2000/XP
界面预览：无
加入时间：2005-01-22 16:47:00
下载次数：715
推荐等级：
开发商：http://www.filerecoveryangel.com/

**软件介绍：**

`Filerecoveryangle` 是一款文件恢复工具，它能够帮助你从格式化成 `FAT12`、`FAT16`、`FAT32`、`NTFS` 文件系统的磁盘中恢复被删除的文件。它不仅仅可以针对硬盘进行文件恢复，它还很好地适用于软盘、数码相机、`USB` 驱动器、`ZIP` 盘、`CompactFlash` 卡、`SmartMedia`，以及索尼记忆棒。

**保护方式：** 用户名+ 序列号

**破解目的：** 学习中。。。

**破解工具：** `Peid`, `OD`

**破解声明：** 我乃小菜鸟一只，偶得一点心得，愿与大家分享：）

**破解过程:**

用 `peid` 查看为：`Borland Delphi 4.0 - 5.0` 没壳。呵呵~~~

注册失败后会显示“`Register False！`”，用 `od` 载入后用字符串插件找到“`Register False！`”双击后来到 `004808EC`：

```
004808E0   .|.  BA B8094800       mov edx,FileReco.004809B8             ;  ASCII "IsRegister"
004808E5   .|.  8BC7               mov eax,edi
004808E7   .|.  E8 549AFEFF       call FileReco.0046A340
004808EC   .|.  B8 700A4800       mov eax,FileReco.00480A70             ;  ASCII "Register\`False!"  <--------------来到这里
004808F1   .|.  E8 A6FAFCFF       call FileReco.0045039C
004808F6   .|.  8BC7               mov eax,edi
004808F8   .|.  E8 9325F8FF       call FileReco.00402E90
```

往上找到：

```
004807B6   .|./ 75 0F                 jnz short FileReco.004807C7             ；比较用户名是否为空
004807B8   .|| B8 4C094800            mov eax,FileReco.0048094C             ;  ASCII "Name can not been empty!"
004807BD   .|| E8 DA FB FC FF       call FileReco.0045039C
004807C2   .|| E9 36010000          jmp FileReco.004808FD
004807C7   |> \8B45 FC              mov eax,dword ptr ss:[ebp-4]      <----------- 在这里下断
```

我们在 `004807C7` 处下断运行程序，在注册栏输入：

`Name`: `ZabShow`
`Key`: `123456`

点击 `Register` 后被 `od` 断下，一路往下分析：

```
004807C7   |> \8B45 FC              mov eax,dword ptr ss:[ebp-4]          ;  用户名:ZabShow          
004807CA   .|  E8 0936F8FF          call FileReco.00403DD8              ;  取长度:6 位
004807CF   .|  8BF8                mov edi,eax
004807D1   .|  03FF                add edi,edi
004807D3   .|  8D55 EC             lea edx,dword ptr ss:[ebp-14]
004807D6   .|  8B86 E4020000       mov eax,dword ptr ds:[esi+2E4]
004807DC   .|  E8 639AFAFF         call FileReco.0042A244              ;  注册码:123456
004807E1   .|  8B45 EC             mov eax,dword ptr ss:[ebp-14]
004807E4   .|  8D55 F8             lea edx,dword ptr ss:[ebp-8]
004807E7   .|  E8 407DF8FF         call FileReco.0040852C
004807EC   .|  8D45 F8             lea eax,dword ptr ss:[ebp-8]
004807EF   .|  50                  push eax
004807F0   .|  8BCF                mov ecx,edi
004807F2   .|  BA 02000000         mov edx,2
004807F7   .|  8B45 F8             mov eax,dword ptr ss:[ebp-8]          ;  注册码:123456
004807FA   .|  E8 E137F8FF         call FileReco.00403FE0
004807FF   .|  8D55 F4             lea edx,dword ptr ss:[ebp-C]
00480802   .|  8B45 FC             mov eax,dword ptr ss:[ebp-4]          ;  用户名:ZabShow
00480805   .|  E8 7EA0FEFF         call FileReco.0046A888              ; 关键！！！我们等会进去
0048080A   .|  8B45 F4             mov eax,dword ptr ss:[ebp-C]          ;  真注册码
0048080D   .|  8B55 F8             mov edx,dword ptr ss:[ebp-8]          ;  假注册码
00480810   .|  E8 D336F8FF         call FileReco.00403EE8
00480815   .|  75 02               jnz short FileReco.00480819
```

到我们到达 `0048080A` 处，便可以看到真的注册码: `DFF9D6A9898IDC`，好！我们重新来过。

`Name`: `ZabShow`
`Key`: `DFF9D6A9898IDC`

点击 `Register` 后，传说中的对话框又出来了，竟然提示错误！怎么回事呢？咱们稍后分解！

现在进入按 `F7` 跟进 `00480805` `call FileReco.0046A888`，也就是算注册码的地方：

```
0046A888   /$  55                  push ebp
0046A889   .|  8BEC                mov ebp,esp
0046A88B   .|  33C9                xor ecx,ecx
0046A88D   .|  51                  push ecx
0046A88E   .|  51                  push ecx
0046A88F   .|  51                  push ecx
0046A890   .|  51                  push ecx
0046A891   .|  51                  push ecx
0046A892   .|  51                  push ecx
0046A893   .|  51                  push ecx
0046A894   .|  51                  push ecx
0046A895   .|  53                  push ebx
0046A896   .|  56                  push esi
0046A897   .|  57                  push edi
0046A898   .|  8955 F8             mov dword ptr ss:[ebp-8],edx
0046A89B   .|  8945 FC             mov dword ptr ss:[ebp-4],eax          ;  用户名
0046A89E   .|  8B45 FC             mov eax,dword ptr ss:[ebp-4]
0046A8A1   .|  E8 E696F9FF         call FileReco.00403F8C
0046A8A6   .|  33C0                xor eax,eax
0046A8A8   .|  55                  push ebp
0046A8A9   .|  68 D2A94600         push FileReco.0046A9D2
0046A8AE   .|  64:FF30             push dword ptr fs:[eax]
0046A8B1   .|  64:8920             mov dword ptr fs:[eax],esp
0046A8B4   .|  8D45 F4             lea eax,dword ptr ss:[ebp-C]
0046A8B7   .|  E8 9C92F9FF         call FileReco.00403B58
0046A8BC   .|  8B45 FC             mov eax,dword ptr ss:[ebp-4]          ;  用户名
0046A8BF   .|  E8 1495F9FF         call FileReco.00403DD8              ;  取用户名长度
0046A8C4   .|  8BF0                mov esi,eax                          ;  EAX 内为用户名长度
0046A8C6   .|  85F6                test esi,esi
0046A8C8   .|  7E 29               jle short FileReco.0046A8F3          ;  若长度为 0 则跳
0046A8CA   .|  BB 01000000         mov ebx,1                          ;  设置读取字符标记位
0046A8CF   |>   8D4D EC           /lea ecx,dword ptr ss:[ebp-14]
0046A8D2   .|  8B45 FC            |mov eax,dword ptr ss:[ebp-4]        ;  用户名
0046A8D5   .|  0FB64418 FF        |movzx eax,byte ptr ds:[eax+ebx-1]   ;  逐位取用户名字符，把 ASCII 码存入 EAX
0046A8DA   .|  BA 02000000        |mov edx,2
0046A8DF   .|  E8 2CDDF9FF        |call FileReco.00408610             ;  把 ASCII 码由数值型变为字符型
0046A8E4   .|  8B55 EC            |mov edx,dword ptr ss:[ebp-14]       ;  字符型的 ASCII
0046A8E7   .|  8D45 F4            |lea eax,dword ptr ss:[ebp-C]
0046A8EA   .|  E8 F194F9FF        |call FileReco.00403DE0             ;  存入字符串
0046A8EF   .|  43                 |inc ebx
0046A8F0   .|  4E                 |dec esi                          ;  计数器减 1
0046A8F1   .|.^  75 DC            \jnz short FileReco.0046A8CF        ;  没取完则跳
0046A8F3   |>   8B45 F4            mov eax,dword ptr ss:[ebp-C]          ;  由 ASCII 码变换后的字符串
0046A8F6   .|  E8 DD94F9FF        call FileReco.00403DD8              ;  取长度
0046A8FB   .|  8BD8                mov ebx,eax
0046A8FD   .|  E8 4680F9FF        call FileReco.00402948
0046A902   .|  83FB 14            cmp ebx,14                          ;  长度与 14h 比较
0046A905   .|  7E 1B               jle short FileReco.0046A922          ;  小于则跳
0046A907   .|  8D45 F4             lea eax,dword ptr ss:[ebp-C]
0046A90A   .|  50                  push eax
0046A90B   .|  B9 14000000         mov ecx,14
0046A910   .|  BA 01000000         mov edx,1
0046A915   .|  8B45 F4             mov eax,dword ptr ss:[ebp-C]
0046A918   .|  E8 C396F9FF         call FileReco.00403FE0              ;  把 20 位后的字符全部去掉
0046A91D   .|  BB 14000000         mov ebx,14
0046A922   |>   8BF3               mov esi,ebx                       ;  把长度给 ESI
0046A924   .|  85F6                test esi,esi                     ;  测试长度是否为 0
0046A926   .|  7E 7F               jle short FileReco.0046A9A7       ;  为 0 则跳
0046A928   .|  BB 01000000         mov ebx,1                         ;  设置读取字符标记位
0046A92D   |>   8D45 E8           /lea eax,dword ptr ss:[ebp-18]
0046A930   .|  8B55 F4            |mov edx,dword ptr ss:[ebp-C]      ;  由 ASCII 码变换后的字符串
0046A933   .|  8A541A FF         |mov dl,byte ptr ds:[edx+ebx-1]     ;  逐位取字符。把 ASCII 码存入 dl
0046A937   .|  E8 C493F9FF         |call FileReco.00403D00
0046A93C   .|  8B45 E8            |mov eax,dword ptr ss:[ebp-18]
0046A93F   .|  BA E8A94600        |mov edx,FileReco.0046A9E8           ; 表 1：abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ
```



```
0046A944   |.  E8 7B97F9FF       |call FileReco.004040C4           ;  将字符与表 1 比对找到其所在位置,记下位置数，存在 EAX 内  

0046A949   |.  8BF8             |mov edi,eax                      ;  把 EAX 送给 edi，设为数 1  

0046A94B   |.  85FF             |test edi,edi  

0046A94D   |.  7F 0C            |jg short FileReco.0046A95B  

0046A94F   |.  8B45 F4          |mov eax,dword ptr ss:[ebp-C]  

0046A952   |.  8A4418 FF        |mov al,byte ptr ds:[eax+ebx-1]  

0046A956   |.  8845 F3          |mov byte ptr ss:[ebp-D],al  

0046A959   |.  EB 2F            |jmp short FileReco.0046A98A  

0046A95B   |>  8D45 E4          |lea eax,dword ptr ss:[ebp-1C]  

0046A95E   |.  BA 30AA4600      |mov edx,FileReco.0046AA30        ;  表 2：85987456212365897456  

0046A963   |.  8A541A FF       |mov dl,byte ptr ds:[edx+ebx-1]   ;  取上面的表 2 的各位的 ASCII 码  

0046A967   |.  E8 9493F9FF     |call FileReco.00403D00  

0046A96C   |.  8B45 E4          |mov eax,dword ptr ss:[ebp-1C]  

0046A96F   |.  E8 D8DCF9FF     |call FileReco.0040864C           ;  把从表 2 取的字符转化为数字，设为数 2  

0046A974   |.  03F8            |add edi,eax                      ;  数 1 与数 2 相加  

0046A976   |.  83FF 3E         |cmp edi,3E                       ;  比较是否小于 62  

0046A979   |.  7E 03           |jle short FileReco.0046A97E      ;  小于则跳  

0046A97B   |.  83EF 3E         |sub edi,3E                       ;  如果大于 62 则减 62  

0046A97E   |>  B8 E8A94600    |mov eax,FileReco.0046A9E8        ;  表 1：abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ  

0046A983   |.  8A4438 FF       |mov al,byte ptr ds:[eax+edi-1]   ;  从表中依据 edi 找到找到相应的字符  

0046A987   |.  8845 F3         |mov byte ptr ss:[ebp-D],al       ;  存入内存  

0046A98A   |>  8D45 E0         |lea eax,dword ptr ss:[ebp-20]  

0046A98D   |.  8A55 F3         |mov dl,byte ptr ss:[ebp-D]  

0046A990   |.  E8 6B93F9FF    |call FileReco.00403D00           ;  连接成字符串  

0046A995   |.  8B55 E0         |mov edx,dword ptr ss:[ebp-20]  

0046A998   |.  8B45 F8         |mov eax,dword ptr ss:[ebp-8]  

0046A99B   |.  E8 4094F9FF    |call FileReco.00403DE0  

0046A9A0   |.  8B45 F8         |mov eax,dword ptr ss:[ebp-8]  

0046A9A3   |.  43              |inc ebx  

0046A9A4   |.  4E              |dec esi                            ;  计数器减 1  

0046A9A5   |.^ 75 86           \\jnz short FileReco.0046A92D  

0046A9A7   |>  33C0           |xor eax,eax  

0046A9A9   |.  5A              |pop edx  

0046A9AA   |.  59              |pop ecx  

0046A9AB   |.  59              |pop ecx  

0046A9AC   |.  64:8910        |mov dword ptr fs:[eax],edx  

0046A9AF   |.  68 D9A94600    |push FileReco.0046A9D9  

0046A9B4   |>  8D45 E0        |lea eax,dword ptr ss:[ebp-20]  

0046A9B7   |.  BA 04000000    |mov edx,4  

0046A9BC   |.  E8 BB91F9FF   |call FileReco.00403B7C  

0046A9C1   |.  8D45 F4        |lea eax,dword ptr ss:[ebp-C]  

0046A9C4   |.  E8 8F91F9FF   |call FileReco.00403B58  

0046A9C9   |.  8D45 FC        |lea eax,dword ptr ss:[ebp-4]  

0046A9CC   |.  E8 8791F9FF   |call FileReco.00403B58  

0046A9D1   |\\.  C3            |retn  

0046A9D2    |.^ E9 198CF9FF   |jmp FileReco.004035F0  

0046A9D7    |.^ EB DB         |jmp short FileReco.0046A9B4  

0046A9D9    |.  5F            |pop edi  

0046A9DA    |.  5E            |pop esi  

0046A9DB    |.  5B            |pop ebx  

0046A9DC    |.  8BE5          |mov esp,ebp  

0046A9DE    |.  5D            |pop ebp  

0046A9DF    |.  C3            |retn                       ;返回
```

综上:

本软件为典型的用户名+注册码式的注册方式

其中要用到两个表

`T1`=`abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ`

`T2`=`85987456212365897456`

1. 对用户名的每一位字符取 ASCII 码保存为 16 进制,在把 ASCII 码由数值型转为字符串型并依次存入字符串 `STR1` 中,如果 `STR1` 的长度大于 `14H`(20)则把 20 位后的截掉.

2. 对 `STR1` 中的每一个字符,依次从 `T1` 表中找到与其相同字符,并记下位置号记为 `pos`,再对 `T2` 表中依次取字符,并变为数值型与 `pos` 相加得到 `tmp1`.

3. 将 `tmp1` 与 `3EH`(62)相比较,如果大于则将 `tmp1` 减去 62.

4. 再从 `T1` 表中由 `tmp1` 的值确定相应位置的一个字符设为 `tmpKey`

5. 最后将所有的 `tmpKey` 相连则就是所要的注册码了.

好！

下面来解答我们前面留下的问题，为什么输入正确的注册码后仍然不能注册呢？

原因在这里：

```
0048080A   |.  8B45 F4          mov eax,dword ptr ss:[ebp-C]         ;  真注册码  
0048080D   |.  8B55 F8          mov edx,dword ptr ss:[ebp-8]         ;  假注册码  
```

当在 `0048080A` 时我们能看到程序算出的真注册码，下 `d eax` 可以看到注册码为 `DFF9D6A9898IDC`

而 `0048080D` 是我们自己输入的注册码，下 `d edx` 可以看到注册码为 `FF9D6A9898IDC`。呵呵

看出来了吧,原来程序把我们输入的注册码的第一位给去掉了,也就是说第一位注册码不参与比较,只是用来凑数的,或是让人家做的内存注册机没用(因为别人不知道注册码的第一位被省去啦),所以我们只要在真注册码前面随便添一位就可以了( 我加的是 `D` ).

`Name`: `ZabShow`

`Key`: `DDFF9D6A9898IDC`

另附注册机代码(Delphi):

```
for i:=1 to length(name) do  
    tmp1:=tmp1+inttohex(ord(name[i]),2);  
if length(tmp1)>20 then  
  tmp1:=copy(tmp1,1,20);  
for i:=1 to length(tmp1) do  
begin  
  tmpPos:=pos(tmp1[i],t1);  
  tmp3:=copy(t2,i,1);  
  tmpCode:=tmpPos+strtoint(tmp3);  
  if tmpCode>62 then  
    tmpCode:=tmpCode-62;  
  key:=key+copy(t1,tmpCode,1);  
end;  

key:='D'+key;
```

 4dd466d0e1e0cc123afbe504da2462ca
