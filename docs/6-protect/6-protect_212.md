
# FLEXLM9.5 该洗洗睡了吧——200 水帖纪念

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-296.htm`](https://www.pediy.com/kssd/pediy08/pediy8-296.htm)

FLEXLM9.5 该洗洗睡了吧——200 水帖纪念

【郑重声明】：本文纯属技术交流，无其他目的。请勿用于商业或其他非法用途，否则后果自负。转载需征得看雪论坛同意。

【软件名称】：Imaris 4.16 版本，官方网站有 DOWN。

【下载页面】：`http://www1.skycn.com/soft/20059.html`

【软件简介】：Flexlm9.5 保护，医学用（汗，我不懂－：））

【调试环境】：XP、OLLYICE、PEID0.93、`calcseed.exe`、`lmcryptgui.exe`、SIG 文件

不会写破文，发了 199 个水帖，一直潜水不敢再发。据说 200 帖就可以成为中级了，我汗——我这么菜水平的都成中级？细想一下，我想看雪上的级别资格更多的是给那些有所奉献的朋友吧，不完全代表水平，于是菜鸟我心释然。

论坛上有很多值得我尊敬的人：看雪、fly、一二三水（哈哈）、laoqian 等，无私奉献技术，指导菜鸟提高。当然，我在学习 FLEXLM 的时候，请教时也曾遇到有人索要 RMB 的问题（至今心里忿然 BS）。学 CRACK 仅仅是我的兴趣而非职业，是想了解别人的思想，因此绝不会靠 CRACK 去谋取一分钱，这也不是我学习的目的。

受大水写了第 500 帖纪念的启发，于是发第 200 水帖纪念，希望大锅们不要扁我哈。论坛上有很多水牛，发了那么多帖子，按说水平也该很高了，希望大家也能在整百帖的时候发帖纪念一下，总结一下自己的经验，共同提高！

转入正题：本文是学习 CRACKZ 的文章《FLEXlm latest information by CrackZ》后总结的。同样的软件，版本不同。

## 一、信息收集

1. `-8`，著名标志，许可证密码错误（`0xFFFFFFF8` 如果爆破的话，有用）
2. `-5`，著名标志，FEATURE 名称错误
3. `-2`，许可证语法错误
4. 验证逻辑：

   如果许可证里面给的种子不对，做出的许可证即使过期了，也会提示 `-8` 错误，表示许可证密码无效。这说明是先进行种子参与的密码运算，接着进行密码验证；而后，才是进行日期检查。

   如果给出的种子对了，而日期过期了，则会提示 `-10`，表示 FEATUER 到期；也说明先进行密码检查，后进行日期检查。哈哈。
5. 使用假的许可证，启动后出现提示 `-8` 提示，表示密码不正确
6. 通过分析，比较容易发现 VENDOR DAEMON 是哪个可执行程序。
7. 对于多进程情况，直接调试主干程序；如果调试创建进程的程序，则可能不中断，因为新进程在附加前已经完成了许可证检查
8. 认真读看雪大锅的书，学习 FLEXLM 的基本知识。另外 CRACKZ 上也有不少好文章。

## 二、制作一个假的许可证 `FAKE.LIC`

FEATURE feature_name vendor_name 1.000 permanent uncounted 123456654321\
 HOSTID=ANY SIGN=123456789123

格式一定要正确。如果用 `lmcryptgui.exe` 检查许可证时提示格式错误，它会自动校正；将前面的错误信息删除，再检查一次，则会成功。

## 三、确定 VEDNDOR NAME

目的：确定 VENDOR NAME

方法：搜索字符串“DEMO”，其前面的 `je` 跳转处会出现 VENDOR NAME 名称。（其实程序中多处出现，该步可略去）

```
006CE285  |.  05 0C030000   add     eax, 30C
006CE28A  |.  85C0          test    eax, eax
006CE28C  |.  74 2B         je      short 006CE2B9                    确定 VENDOR=bitplane
006CE28E  |.  B8A28E00      mov     ecx, 008EA2B8                    ;  ASCII "demo"
006CE293  |.  85C9          test    ecx, ecx
006CE295  |.  74 22         je      short 006CE2B9
006CE297  |.  68 C0A28E00   push    008EA2C0                          ; /s2 = "demo"
006CE29C  |.  8B95 6CFDFFFF mov     edx, [ebp-294]                   ; |
006CE2A2  |.  81C2 0C030000 add     edx, 30C                        ; |
```

## 四、定位 `_l_init()` 函数

函数目的：定位 `_l_init()` 函数可确定 `_l_sg` 函数，确定 VENDOR NAME

方法：查找默认的种子 `0x12345678` & `0x87654321` 可以确定 `_l_init()` 函数

该函数调用时堆栈出现正确的 VENDOR NAME；

```
006CE309  |.  51            push    ecx                               ; |Arg2
006CE30A  |.  8B95 6CFDFFFF mov     edx, [ebp-294]                   ; |
006CE310  |.  52            push    edx                               ; |lc_init 函数
006CE311  |.  E8 453DFFFF   call    006C205B                          ; \调用_l_sg 函数，F7 跟进
006CE316  |.  83C4 0C       add     esp, 0C
006CE319  |.  81BD 84FDFFFF>cmp     dword ptr [ebp-27C], 87654321    ;   默认的种子 1，定位标志 1
006CE323  |.  74 0C         je      short 006CE331
006CE325  |.  81BD 88FDFFFF>cmp     dword ptr [ebp-278], 12345678    ;  默认的种子 2，定位标志 2
006CE32F  |.  75 5D         jnz     short 006CE38E
006CE331  |>  8B85 6CFDFFFF mov     eax, [ebp-294]
```

注意调用 `_l_sg` 函数完毕后不会填入正确的种子，因为在 `_l_sg` 函数中会提前跳转

## 五、定位 `_lc_checkout()` 函数以及对其的调用处

目的：确定 FEATURE 和产品版本号（不是 FLEXLM 的版本号）

方法：在字符串参考中查找 `'lm_ckout.c'`；总共也就出现几次，比较容易确定。一般是挨着的两个才是；另外有四个挨着的不是。

定位 `_lc_checkout()` 函数比较容易，可以执行到其返回，再 F8，就可以确定主程序中对它的调用处。（直接查找可能有多处，运行一次找到）

```
006BE4C0  /$  55            push    ebp
```

### 1、定位 `_lc_checkout` 函数

```
006BE4E8  |.  8990 FC030000 mov     [eax+3FC], edx
006BE4EE  |.  68 95000000   push    95                                ; /Arg3 = 00000095
006BE4F3  |.  68 8C9B8E00   push    008E9B8C                          ; |Arg2 = 008E9B8C ASCII "lm_ckout.c"，确定标志 1，
                                                                      //堆栈和寄存器出现 FEATURE
006BE4F8  |.  8B4D 08       mov     ecx, [ebp+8]                      ; |
```

```
006BE543  |.  E8 B3000000   call    006BE5FB                          ; \堆栈出现产品版本和 FEATUER 名称
```

```
006BE5A6  |.  E8 50000000   call    006BE5FB                          ; \如果许可证密码错误，这里会间接调用_l_sg 函数
```

（跟进发现有对 `l_valid_version` 验证版本；`lc_check_key` 调用 `_l_sg` 函数等函数的调用）

（！！！注意：在这里初次不会调用该 CALL。慢慢来，呵呵。我们在该函数里能得到 FEATURE 和版本号就足够了！）

```
006BE5DE  |.  68 A6000000   push    0A6                               ; /Arg3 = 000000A6
006BE5E3  |.  68 989B8E00   push    008E9B98                          ; |Arg2 = 008E9B98 ASCII "lm_ckout.c" 确定标志 2
006BE5E8  |.  8B55 08       mov     edx, [ebp+8]                      ; |
```

### 2、定位对其的调用

```
0041E785    .  51            push    ecx
0041E786    .  53            push    ebx
0041E787    .  52            push    edx
0041E788    .  E8 33FD2900   call    006BE4C0                          ;  调用_l_checkout
0041E78D    .  8BE8          mov     ebp, eax                          ; |
0041E78F    .  8B46 2C       mov     eax, [esi+2C]                     ; |
0041E792    .  53            push    ebx                                 ; |Arg2
0041E793    .  50            push    eax                                 ; |Arg1
0041E794    .  E8 BDEC2900   call    <_lc_hostid 009E:_l_hostid>        ; \堆栈出现产品版本号与 FEATURE
0041E799    .  8B4E 08       mov     ecx, [esi+8]
0041E79C    .  83C4 24       add     esp, 24
```

记录前面的版本号和 FEATURE（有的软件可能有几个产品版本号）

## 六、修改假的许可证

为什么我们在前面不能再次调用 `_l_sg` 函数？这是因为 FLEXLM 的验证机制，如果 FEATUER 和产品号验证不正确，眼看要煮熟的鸭子 `_l_sg` 就直接飞了，就不会再断下来！

修改许可证，`_l_sg` 函数就可以再次停下来了，这就相当于帅哥有钱，MM 停下来；看来造假还是不好，来接近于真的：

FEATURE ImarisBase bitplane 4.1 permanent uncounted 123456654321\
 HOSTID=ANY SIGN=123456789123

## 七、定位 `_l_sg` 函数及其返回

函数目的：`_l_sg` 的意思是"签名 vendor_key5"，呵呵。其返回处是恢复加密种子的关键！

方法：搜索常量 SEEDVAL，9.2 版本的是 `6F7330B8`；注意，有两个出现，其中一个是混淆函数 `_l_svk()` 里面的。可以在 `?_l_init` 函数里面通过 F7 跟进确定，也结合后面的固定结构

V7 glseed?=?0x788F71D2 seedval?=?0x7648B98E
V8 glseed?=?0x3CDE3EBF seedval?=?0x6F73330B8
V9 glseed?=?0x72346B53 seedval?=?0x6F7330B8
v10 glseed?=?0x5332322F seedval?=?0x6F7330B8

(seedval：用于确定 `_l_sg` 函数；glseed：可以用来确定 vendor name，堆栈出现)

（这些值最早出现于 WOODOOM 网站，laoqian 大锅的总结一文也提及了－惠及众生，爽）

可以执行到返回，然后用 `dd` 命令

### 1、定位函数

```
006C205B  /$  55            push    ebp                                 ;  _l_sg 函数
```

```
006C206B  |.  8845 EF       mov     [ebp-11], al
006C206E  |.  C745 F4 B8307>mov     dword ptr [ebp-C], 6F7330B8       ;  seedval 寻找 L_SG，注意混淆函数
006C2075  |.  C745 FC 00000>mov     dword ptr [ebp-4], 0
```

```
006C21D7  |.  5D            pop     ebp                                 ;  _l_sg 函数返回
```

### 2、定位函数的返回：

```
006C02FF  |.  8B45 08       mov     eax, [ebp+8]                        ; |
006C0302  |.  50            push    eax                                 ; |Arg1
006C0303  |.  E8 531D0000   call    006C205B                          ; \调用_l_sg 函数
006C0308  |.  83C4 0C       add     esp, 0C                           ;  返回到这里，该处下断，用 dd [esp]和 dd [esp+8]命令
006C030B  |.  8B4D 08       mov     ecx, [ebp+8]
```

## 八、追踪加密种子和许可证

方法：在调用 `_l_sg` 函数的返回处（后面一般是一个 `add`…命令，就可以在 `add` 命令处下断，使用 `dd` 命令），用 `dd [esp] dd [esp+8]` 命令查看内存中的内容。

`dd [esp]` 得到的是 `00000066`。。。job\[]结构内容
`dd [esp+8]` 得到的是 `00000004`。。。data\[]内容

如果是正确的许可证，与调用 `_l_sg` 函数的次数关系不大，除了 `_l_init` 函数不填充正确的 job 结构外，每一个 FEATUER 都要调用 `_l_sg` 函数一次；

其返回值将是正确的 job 和 data 结构数据；也就是说，每个 FEATUER 对应的 job 和 data 值不相等，但用它们算出的加密种子却一样。

### 1、追踪加密种子

在 `_l_sg` 函数的返回处，使用 `dd` 命令

```
006C0308  |.  83C4 0C       add     esp, 0C                     ;  返回到这里，该处下断，用 dd [esp]和 dd [esp+8]命令
```

`dd [esp]`

```
03A7C350  00000066
03A7C354  00000000
03A7C358  ********    job+08
03A7C35C  ********    job+0c
03A7C360  ********    job+10
03A7C364  00000000
```

`dd [esp+8]`

```
0012F5CC  00000004
0012F5D0  ********     data[0]
0012F5D4  ********     data[1]
0012F5D8  18A011ED
```



`0012F5DC` `274BE197`

`0012F5E0` `E26A21A8`

`0012F5E4` `52580441`

`0012F5E8` `00020009`

用`calcseed.exe`计算加密种子

`vendor=bitplane`

`enseed1=*******` `enseed2=******` 与`VENDOR NAME`的关系是仅仅与其第一个字母有关

`vendor=Bitplane`

追出的`VENDOR NAME`是小写：`bitplane`，算出的`LIC`正确；

而网上有`DOWN`的许可证，第一个字母是大写的`VENDOR NAME`：`Bitplane`也正确。这里将`VENDOR`换成大、小写都没有关系，但用小写`VENDOR`计算的加密`SEEDS`。

### 2、做出许可证

用`lmcryptgui.exe`。具体方法参见`laoqian`大锅的文章。

用`SDK`也可以制作，只不过在`lm_code.h`中要添加一些信息，麻烦哦。我懒，如果你做好了，别忘了送我一份啊：）

### 后记

该文的方法对于`7.2`、`8.0`、`9.2`的版本都适用。`7`之前的版本应该也可以用该方法，因为`FLEXLM`的某些关键结构（哈哈）一直没有改变！这是通用的方法。

本想做个循序渐进册子，突然发现该方法其实对老版本的`_l_init`，`_l_new_job`等方法都包容了。

可以这样说，对于没有使用`ECC`等技术（需要`PATCH`，`SIGN`超过`12`位则使用了该技术，以后讨论）的情况，`FLEXLM`就像一张纸。

* 关键点：定位对`_l_sg`函数的调用返回处，逐步修改假许可证（格式正确）中的内容，就会在`_l_sg`函数处停下来恢复加密的种子。

### 感谢

`CrackZ`

`Nolan Blender`

`tulipfan[CCG]`

`laoqian`

看雪论坛上的朋友们!

### 附：一些常见问题的处理

希望大家补充：

#### 1、`FLEXLM`追出了正确加密种子，`7.2`等`SDK`做不出正确的许可证？

如果能正确启动`genlic32.exe`，恭喜你找对了加密的种子，试试添加选项`"advanced->Add Compatible License Key"`

#### 2、`_l?_sg`函数函数只调用一次，不来第二次？

`CRACKZ`的文章也曾提及，但没有说明原因和处理办法。常见处理：

* A:`许可证格式`要正确
* B:设置正确的环境变量，可在`"我的电脑->属性"`里面或者`autoexec.bat`里面设置
* C：`许可证`里面逐步填入正确的`VENDOR`，`FEATUER`，产品版本号等

`CRACKZ`的另外一篇文章还提及了另外一种处理办法：终止`DAEMON`进程，在`许可证`里面加入`UER_...`信息，但我没有用它试验成功过。

#### 3、如何让后台程序安息？

运行程序，会发现至少有三样不少：`lmgrd.exe`是一陪，`vendor daemon`是二陪，`Cracker`是“三陪”，哈哈。

将`许可证`做成无任何限制的格式，前面两陪直接上床睡觉（进程不启动），晕吧。如果不知道什么是无任何限制的格式，赶快买看雪大锅的书看看（书店一般脱销，我托人才买到）。

#### 4、有了正确的许可证，怎么找加密种子？

实在恭喜你，省事啊。`VENDOR`,`FEATURE`等等都不用找了。

一般情况了，有了正确的`许可证`，`_l_sg`函数会在每个`FEATURE`的验证后返回正确的值，用通用的`种子恢复技术`就可以了。

#### 5、相互交流，才能共同提高！

仅作技术交流，恳请不要用于非法用途！

`4dd466d0e1e0cc123afbe504da2462ca`
