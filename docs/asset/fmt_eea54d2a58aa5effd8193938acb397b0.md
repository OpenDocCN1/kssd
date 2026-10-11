
# 速用流水帐财务软件 2.0

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-407.htm`](https://www.pediy.com/kssd/pediy08/pediy8-407.htm)

【破文标题】非常简单的速用流水帐财务软件 2.0 注册算法

【破文作者】XXNB

【作者邮箱】

【作者主页】[`free.ys168.com/?binbinbin7456`](http://free.ys168.com/?binbinbin7456)

【破解工具】OD

【破解平台】xpsp2

【软件名称】速用流水帐财务软件 2.0

【软件大小】2150KB

【原版下载】[`www.onlinedown.net/soft/48569.htm`](http://www.onlinedown.net/soft/48569.htm)

【保护方式】注册码

【软件简介】速用流水帐财务软件可能会成为中国软件市场上最热门的产品，因为所有的企业都需要记帐，与纸管理相比，电脑记录信息的可靠性、准确性、搜索效率都优胜很多，遗憾的是国内众多软件公司在费尽心思做“大”软件的同时却忽略了使用者的习惯和接受程度，并且对于同样的财务数据，如何做出几本帐薄来应付不同情况的需要，也没有拿出什么较好的解决方案。......

【破解声明】向大侠们学习！！！献给学算法的朋友们！

------------------------------------------------------------------------

## 破解过程

------------------------------------------------------------------------

### 1、字符串查找“注册码错误，请重新操作”来到关键代码处：

```
0051EC08  /.  55           push    ebp                 //我们就在这里下断吧，然后 F9 运行。点“注册”按钮后断在这里。
0051EC09  |.  8BEC         mov     ebp, esp
0051EC0B  |.  6A 00        push    0                   //往下执行，F8
0051EC0D  |.  6A 00        push    0
0051EC0F  |.  6A 00        push    0
0051EC11  |.  33C0         xor     eax, eax
0051EC13  |.  55           push    ebp
0051EC14  |.  68 51ED5100  push    0051ED51
0051EC19  |.  64:FF30      push    dword ptr fs:[eax]
0051EC1C  |.  64:8920      mov     fs:[eax], esp
0051EC1F  |.  A1 F01F5200  mov     eax, [521FF0]
0051EC24  |.  8B00         mov     eax, [eax]
0051EC26  |.  50           push    eax                   //ASCII "C:\WINDOWS\mdnsm.ini"
0051EC27  |.  8D45 F8      lea     eax, [ebp-8]
0051EC2A  |.  50           push    eax
0051EC2B  |.  B9 64ED5100  mov     ecx, 0051ED64          ;  urv
0051EC30  |.  BA 70ED5100  mov     edx, 0051ED70          ;  sv
0051EC35  |.  B8 7CED5100  mov     eax, 0051ED7C          ;  supermoney
0051EC3A  |.  E8 CD2AFFFF  call    0051170C
0051EC3F  |.  8B45 F8      mov     eax, [ebp-8]
0051EC42  |.  BA 90ED5100  mov     edx, 0051ED90          ;  regver
0051EC47  |.  E8 D85AEEFF  call    00404724
0051EC4C  |.  75 0F        jnz     short 0051EC5D         //爆破地方到了。
0051EC4E  |.  B8 A0ED5100  mov     eax, 0051EDA0          ;  软件注册已经成功，谢谢您的支持！
0051EC53  |.  E8 ACADF1FF  call    00439A04
0051EC58  |.  E9 D9000000  jmp     0051ED36
0051EC5D  |>  E8 DE2FFFFF  call    00511C40
0051EC62  |.  8D55 FC      lea     edx, [ebp-4]
0051EC65  |.  E8 A2AEEEFF  call    00409B0C
0051EC6A  |.  A1 74205200  mov     eax, [522074]
0051EC6F  |.  8B00         mov     eax, [eax]
0051EC71  |.  05 10030000  add     eax, 310
0051EC76  |.  BA CCED5100  mov     edx, 0051EDCC          ;  软件注册
0051EC7B  |.  E8 EC56EEFF  call    0040436C
0051EC80  |.  68 E0ED5100  push    0051EDE0                 ;  序列号为：（这里就是取硬盘序列号，作者真的好懒。^-^）
0051EC85  |.  FF75 FC      push    dword ptr [ebp-4]
0051EC88  |.  68 F4ED5100  push    0051EDF4                 ;  ，请输入注册码
0051EC8D  |.  A1 74205200  mov     eax, [522074]
0051EC92  |.  8B00         mov     eax, [eax]
0051EC94  |.  05 14030000  add     eax, 314
0051EC99  |.  BA 03000000  mov     edx, 3
0051EC9E  |.  E8 F559EEFF  call    00404698
0051ECA3  |.  A1 74205200  mov     eax, [522074]
0051ECA8  |.  8B00         mov     eax, [eax]
0051ECAA  |.  05 18030000  add     eax, 318
0051ECAF  |.  E8 6456EEFF  call    00404318
0051ECB4  |.  A1 74205200  mov     eax, [522074]
0051ECB9  |.  8B00         mov     eax, [eax]
0051ECBB  |.  8B10         mov     edx, [eax]
0051ECBD  |.  FF92 EC000000 call    [edx+EC]               //运行到这里，跳出注册框。随便输入假码。
```

### 2、继续往下看

```
0051ECC3  |.  48           dec     eax                    ; 按“确定”之后会停在这里
0051ECC4  |.  75 70        jnz     short 0051ED36          ; 没输入的话就...
0051ECC6  |.  8D55 F4      lea     edx, [ebp-C]
0051ECC9  |.  8B45 FC      mov     eax, [ebp-4]            ; 机器码
0051ECCC  |.  E8 072EFFFF  call    00511AD8               ；最重要的算法关键 Call 到了   《－－－－－
0051ECD1  |.  8B55 F4      mov     edx, [ebp-C]            ; 真码
0051ECD4  |.  A1 74205200  mov     eax, [522074]
0051ECD9  |.  8B00         mov     eax, [eax]
0051ECDB  |.  8B80 18030000 mov    eax, [eax+318]          ; 假码
0051ECE1  |.  E8 3E5AEEFF  call    00404724               ; 比较函数
0051ECE6  |.  75 44        jnz     short 0051ED2C          ; 关键跳        《－－－－－－－－－－
0051ECE8  |.  A1 F01F5200  mov     eax, [521FF0]
0051ECED  |.  8B00         mov     eax, [eax]
0051ECEF  |.  50           push    eax
0051ECF0  |.  B9 90ED5100  mov     ecx, 0051ED90          ;  regver
0051ECF5  |.  BA 70ED5100  mov     edx, 0051ED70          ;  sv
0051ECFA  |.  B8 7CED5100  mov     eax, 0051ED7C          ;  supermoney
0051ECFF  |.  E8 1C2BFFFF  call    00511820
0051ED04  |.  A1 F01F5200  mov     eax, [521FF0]
0051ED09  |.  8B00         mov     eax, [eax]
0051ED0B  |.  50           push    eax
0051ED0C  |.  B9 0CEE5100  mov     ecx, 0051EE0C          ;  alive
0051ED11  |.  BA 1CEE5100  mov     edx, 0051EE1C          ;  systemstate
0051ED16  |.  B8 7CED5100  mov     eax, 0051ED7C          ;  supermoney
0051ED1B  |.  E8 002BFFFF  call    00511820
0051ED20  |.  B8 30EE5100  mov     eax, 0051EE30          ;  软件注册成功，谢谢您的支持！
0051ED25  |.  E8 DAACF1FF  call    00439A04
0051ED2A  |.  EB 0A        jmp     short 0051ED36
0051ED2C  |>  B8 58EE5100  mov     eax, 0051EE58          ;  注册码错误，请重新操作
```

### 3、跟进 `call` `00511AD8` 可以看到如下关键代码：

```
00511AE7  |.  8955 F8      mov     [ebp-8], edx
00511AEA  |.  8945 FC      mov     [ebp-4], eax
00511AED  |.  8B45 FC      mov     eax, [ebp-4]
00511AF0  |.  E8 D32CEFFF  call    004047C8
00511AF5  |.  33C0         xor     eax, eax
00511AF7  |.  55           push    ebp
00511AF8  |.  68 031C5100  push    00511C03
00511AFD  |.  64:FF30      push    dword ptr fs:[eax]
00511B00  |.  64:8920      mov     fs:[eax], esp
00511B03  |.  8B45 FC      mov     eax, [ebp-4]            ; 机器码
00511B06  |.  E8 CD2AEFFF  call    004045D8                ; 计算位数
00511B0B  |.  8BF8         mov     edi, eax
00511B0D  |.  85FF         test    edi, edi
00511B0F  |.  7E 2F        jle     short 00511B40
00511B11  |.  BE 01000000  mov     esi, 1                  ; 计数器
00511B16  |>  8B45 FC      /mov    eax, [ebp-4]            ；循环开始
00511B19  |.  8A5C30 FF    |mov    bl, [eax+esi-1]         ; 取第一位
00511B1D  |.  80FB 30      |cmp     bl, 30
00511B20  |.  72 1A        |jb      short 00511B3C         ; 小于 0，跳
00511B22  |.  80FB 39      |cmp     bl, 39
00511B25  |.  77 15        |ja      short 00511B3C         ; 大于 9，跳
00511B27  |.  8D45 F0      |lea     eax, [ebp-10]
00511B2A  |.  8BD3         |mov     edx, ebx
00511B2C  |.  E8 CF29EFFF  |call    00404500
00511B31  |.  8B55 F0      |mov     edx, [ebp-10]
00511B34  |.  8D45 F4      |lea     eax, [ebp-C]
00511B37  |.  E8 A42AEFFF  |call    004045E0
00511B3C  |>  46           |inc     esi
00511B3D  |.  4F           |dec     edi
00511B3E  |.^ 75 D6        \jnz     short 00511B16         ; 循环结束。上面的循环功能是选出序列号里面的数字。
00511B40  |>  837D F4 00   cmp     dword ptr [ebp-C], 0      ; 如果你的硬盘序列号都没有数字的话。嘻嘻～。有的话这里就显示出来了
00511B44  |.  75 0D        jnz     short 00511B53
00511B46  |.  8D45 F4      lea     eax, [ebp-C]
```
