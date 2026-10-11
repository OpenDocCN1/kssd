
# 数据库信息管理开发平台

> 原文：[`www.pediy.com/kssd/pediy05/pediy50666.htm`](https://www.pediy.com/kssd/pediy05/pediy50666.htm)

## 数据库信息管理开发平台 V3.5 算法分析

- **作者**：`wzh123`
- **软件大小**：6851 KB
- **软件语言**：简体中文
- **软件类别**：国产软件 / 共享版 / 数据库类
- **应用平台**：`Win9x/NT/2000/XP`

### 软件介绍

数据库管理系统开发利器，具有如下特色：面向信息管理全过程、支持全方位自定义设计；支持无代码开发，随心设计功能强大；信息流程化处理，处理过程任意控制；开放式数据管理，支持各种数据库格式；支持网络数据开发，轻松设计客户端应用系统；信息分类方便，树形管理简单；信息录入智能化，极大提高录入效率；支持计算公式，让计算器束之高阁；支持字段间运算，计算字段自动求值；记录有效性验证，保证数据准确有效；录入面板可自我设计；所需字段智能导入，鼠标轻点报表呈现；智能判定开发进度，开发方向动态提示；自动生成信息菜单，信息访问快捷方便。

### 工具与申明

- **工具**：`softice`，`W32Dasm8.93` 黄金版，`FI2.5`
- **作者申明**：只是学习，无其他目的。
- **备注**：本人刚刚学破解，错误在所难免，写的也很乱，请各位包涵，也请各位高手指教。

### 分析步骤

1. 软件有 `aspack 2.12` 的壳，脱壳后，使用 `fi` 查出是 `Delphi` 编写的。
2. 这个软件是通过注册码来反推注册名的。
3. 反编译后，很快可以找到关键地方。

- **注册名**：`wzh123`
- **注册码**：`a12345b67890`

### 代码分析

你一定可以来到这里：

```asm
:005CAD79 A190774400           mov eax, dword ptr [00447790]
:005CAD7E E80DCBE7FF          call 00447890
:005CAD83 8945FC             mov dword ptr [ebp-04], eax
:005CAD86 33C0               xor eax, eax
:005CAD88 55                 push ebp
:005CAD89 68A5B05C00         push 005CB0A5
:005CAD8E 64FF30             push dword ptr fs:[eax]
:005CAD91 648920             mov dword ptr fs:[eax], esp
:005CAD94 8D55EC             lea edx, dword ptr [ebp-14]
:005CAD97 8B8308030000       mov eax, dword ptr [ebx+00000308]
:005CAD9D E89E90EBFF         call 00483E40
:005CADA2 8B45EC             mov eax, dword ptr [ebp-14]
:005CADA5 8D55F0             lea edx, dword ptr [ebp-10]
:005CADA8 E80FE5E3FF         call 004092BC
:005CADAD 8B45F0             mov eax, dword ptr [ebp-10]  "a12345b67890"-->eax
:005CADB0 8D4DF4             lea ecx, dword ptr [ebp-0C]

* Possible StringData Ref from Data Obj ->"HDDBIP"

:005CADB3 BA8CB15C00         mov edx, 005CB18C          "HDDBIP"(程序自定)-->edx
:005CADB8 E8EB930300         call 006041A8              算法 call，追入
:005CADBD 8B45F4             mov eax, dword ptr [ebp-0C]
:005CADC0 50                 push eax
:005CADC1 8D55E4             lea edx, dword ptr [ebp-1C]
:005CADC4 8B8304030000       mov eax, dword ptr [ebx+00000304]
:005CADCA E87190EBFF         call 00483E40
:005CADCF 8B45E4             mov eax, dword ptr [ebp-1C]
:005CADD2 8D55E8             lea edx, dword ptr [ebp-18]
:005CADD5 E8E2E4E3FF         call 004092BC
:005CADDA 8B55E8             mov edx, dword ptr [ebp-18]  "wzh123"-->edx
:005CADDD 58                 pop eax                      将算出的注册名-->eax
:005CADDE E8A19CE3FF         call 00404A84                比较
:005CADE3 0F85E9000000       jne 005CAED2                 相等就不跳，注册成功，这里不是爆破点，改了没用
:005CADE9 33C0               xor eax, eax
:005CADEB 55                 push ebp
:005CADEC 68C3AE5C00         push 005CAEC3
:005CADF1 64FF30             push dword ptr fs:[eax]
:005CADF4 648920             mov dword ptr fs:[eax], esp
:005CADF7 BA02000080         mov edx, 80000002
:005CADFC 8B45FC             mov eax, dword ptr [ebp-04]
:005CADFF E82CCBE7FF         call 00447930
:005CAE04 B101               mov cl, 01
```

算法调用 `006041A8` 详细分析：

```asm
; 进去后你一定会来到这里：

:006041A8 55                 push ebp
:006041A9 8BEC               mov ebp, esp
:006041AB 83C4D0             add esp, FFFFFFD0
:006041AE 53                 push ebx
:006041AF 56                 push esi
:006041B0 57                 push edi
:006041B1 33DB               xor ebx, ebx
:006041B3 895DD0             mov dword ptr [ebp-30], ebx
:006041B6 895DD8             mov dword ptr [ebp-28], ebx
:006041B9 895DD4             mov dword ptr [ebp-2C], ebx
:006041BC 895DE0             mov dword ptr [ebp-20], ebx
:006041BF 895DDC             mov dword ptr [ebp-24], ebx
:006041C2 895DEC             mov dword ptr [ebp-14], ebx
:006041C5 894DF4             mov dword ptr [ebp-0C], ecx
:006041C8 8955F8             mov dword ptr [ebp-08], edx
:006041CB 8945FC             mov dword ptr [ebp-04], eax
:006041CE 8B45FC             mov eax, dword ptr [ebp-04]
:006041D1 E85209E0FF         call 00404B28
:006041D6 8B45F8             mov eax, dword ptr [ebp-08]
:006041D9 E84A09E0FF         call 00404B28
:006041DE 33C0               xor eax, eax
:006041E0 55                 push ebp
:006041E1 6803436000         push 00604303
:006041E6 64FF30             push dword ptr fs:[eax]
:006041E9 648920             mov dword ptr fs:[eax], esp
:006041EC 8B45F8             mov eax, dword ptr [ebp-08]      "HDDBIP"(程序自定)-->edx
:006041EF E84407E0FF         call 00404938                    取 HDDBIP 位数，为"6"
:006041F4 8945F0             mov dword ptr [ebp-10], eax      6-->[ebp-10]
:006041F7 837DF000           cmp dword ptr [ebp-10], 00000000
:006041FB 750D               jne 0060420A
:006041FD 8D45F8             lea eax, dword ptr [ebp-08]

* Possible StringData Ref from Data Obj ->"Think Space"

:00604200 BA1C436000         mov edx, 0060431C
:00604205 E80605E0FF         call 00404710

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:006041FB(C)
|
:0060420A 33F6               xor esi, esi
:0060420C 8D45DC             lea eax, dword ptr [ebp-24]
:0060420F 50                 push eax
:00604210 B902000000         mov ecx, 00000002
:00604215 BA01000000         mov edx, 00000001
:0060421A 8B45FC             mov eax, dword ptr [ebp-04]      "a12345b67890"-->eax
:0060421D E87609E0FF         call 00404B98                    取注册码的前两位，如："a1"
:00604222 8B4DDC             mov ecx, dword ptr [ebp-24]      "a1"-->ecx
:00604225 8D45E0             lea eax, dword ptr [ebp-20]

* Possible StringData Ref from Data Obj ->"$"

:00604228 BA30436000         mov edx, 00604330
:0060422D E85207E0FF         call 00404984
:00604232 8B45E0             mov eax, dword ptr [ebp-20]
:00604235 E82655E0FF         call 00409760                    将"a1"->"A1"-->eax
:0060423A 8BF8               mov edi, eax
:0060423C C745E803000000     mov [ebp-18], 00000003           3-->[ebp-18]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:006042C2(C)
|
:00604243 8D45D4             lea eax, dword ptr [ebp-2C]
:00604246 50                 push eax
:00604247 B902000000         mov ecx, 00000002
:0060424C 8B55E8             mov edx, dword ptr [ebp-18]     [ebp-18]初始值为 3-->edx
:0060424F 8B45FC             mov eax, dword ptr [ebp-04]      "a12345b67890"-->eax
:00604252 E84109E0FF         call 00404B98                   从注册码的第三位开始取两位数，
                                                            如: 1、"23"
                                                            2、"45"
                                                            3、"b1"
                                                            4、"b6"
                                                            5、"78"
                                                            6、"90"
:00604257 8B4DD4             mov ecx, dword ptr [ebp-2C]      "23"-->ecx
:0060425A 8D45D8             lea eax, dword ptr [ebp-28]

* Possible StringData Ref from Data Obj ->"$"

:0060425D BA30436000         mov edx, 00604330
:00604262 E81D07E0FF         call 00404984
```



### 汇编代码分析

```
:00604267 8B45D8                 mov eax, dword ptr [ebp-28]

:0060426A E8F154E0FF             call 00409760                  将"23"-- >eax

:0060426F 8945E4                 mov dword ptr [ebp-1C], eax    "23"-->[ebp-1c]

:00604272 3B75F0                 cmp esi, dword ptr [ebp-10]     esi 初始值为 0，与 HDDBIP 位数 6 比较

:00604275 7D03                   jge 0060427A                   如果 esi>=6,则跳到下面置为 1

:00604277 46                     inc esi                         否则 esi+1

:00604278 EB05                   jmp 0060427F                   跳到 0060427F
----------------------------------------->这里的判断是为了循环取"HDDBIP"

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00604275(C)
|
:0060427A BE01000000             mov esi, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00604278(U)
|
:0060427F 8B45F8                 mov eax, dword ptr [ebp-08]      "HDDBIP"-->eax

:00604282 33DB                   xor ebx, ebx                    ebx 清零

:00604284 8A5C30FF               mov bl, byte ptr [eax+esi-01]   依次取"HDDBIP"-- >bl

:00604288 335DE4                 xor ebx, dword ptr [ebp-1C]
                                  --------1、0x48("H")^0x23-->ebx
                                  --------2、0x44("D")^0x45-->ebx
                                  --------3、0x44("D")^0xB6-->ebx
                                  --------4、0x42("B")^0x78-->ebx
                                  --------5、0x49("I")^0x90-->ebx
:0060428B 3BFB                   cmp edi, ebx                    比较
                                  --------1、0xa1 与 ebx 比较
                                  --------2、0x23 与 ebx 比较
                                  --------3、0x45 与 ebx 比较
                                  --------4、0xb6 与 ebx 比较
                                  --------5、0x78 与 ebx 比较
:0060428D 7C0A                   jl 00604299                     小于就跳到 00604299

:0060428F 81C3FF000000           add ebx, 000000FF               否则 ebx+0xff-->ebx

:00604295 2BDF                   sub ebx, edi                    ebx-edi-->ebx(通过循环依次得到第 1、2、......位注册名)

:00604297 EB02                   jmp 0060429B                    跳
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0060428D(C)
|
:00604299 2BDF                   sub ebx, edi
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00604297(U)
|
:0060429B 8D45D0                 lea eax, dword ptr [ebp-30]

:0060429E 8BD3                   mov edx, ebx

:006042A0 E8BB05E0FF             call 00404860

:006042A5 8B55D0                 mov edx, dword ptr [ebp-30]

:006042A8 8D45EC                 lea eax, dword ptr [ebp-14]

:006042AB E89006E0FF             call 00404940

:006042B0 8B7DE4                 mov edi, dword ptr [ebp-1C]
                                  --------1、0x23-->edi
                                  --------2、0x45-->edi
                                  --------3、0xb6-->edi
                                  --------4、0x78-->edi
:006042B3 8345E802               add dword ptr [ebp-18], 00000002 [ebp-18]+2,即取注册码的下两位

:006042B7 8B45FC                 mov eax, dword ptr [ebp-04]      "a12345b67890"-->eax

:006042BA E87906E0FF             call 00404938                   取注册码的位数

:006042BF 3B45E8                 cmp eax, dword ptr [ebp-18]      比较，判断注册码取完了吗

:006042C2 0F8F7BFFFFFF           jg 00604243                     没取完，就向上循环

:006042C8 8B45F4                 mov eax, dword ptr [ebp-0C]

:006042CB 8B55EC                 mov edx, dword ptr [ebp-14]

:006042CE E8F903E0FF             call 004046CC

:006042D3 33C0                   xor eax, eax

:006042D5 5A                     pop edx

:006042D6 59                     pop ecx

:006042D7 59                     pop ecx

:006042D8 648910                 mov dword ptr fs:[eax], edx

:006042DB 680A436000             push 0060430A
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00604308(U)
|
:006042E0 8D45D0                 lea eax, dword ptr [ebp-30]

:006042E3 BA05000000             mov edx, 00000005

:006042E8 E8AF03E0FF             call 0040469C

:006042ED 8D45EC                 lea eax, dword ptr [ebp-14]

:006042F0 E88303E0FF             call 00404678

:006042F5 8D45F8                 lea eax, dword ptr [ebp-08]

:006042F8 BA02000000             mov edx, 00000002

:006042FD E89A03E0FF             call 0040469C

:00604302 C3                     ret
```

### 算法推导

至此，算法分析完了，弄清楚了算法，我们可以推出一组有效的注册信息：

偷个懒，注册名取：`w` (`0x77`)

注册码前两位随便取：`12`，假设后两位为 `X`

根据算法：`(0x48("H")^X)-0x12=0x77("w")`，推导得 `X=(0x77+0x12)^0x48=0xc1`（因为注册码前两位取 `12`，所以 `0x48("H")^X` 会大于 `0x12`，所以不需要减去 `0xff`）

所以注册码为：`12c1`

注册信息：

- 注册名：`w`
- 注册码：`12c1`

回头看看，我考，注册价格： `1000` 元/套，`4dd466d0e1e0cc123afbe504da2462ca`
