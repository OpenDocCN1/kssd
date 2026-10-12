
# FolderInfo V2.82

> 原文：[`www.pediy.com/kssd/pediy06/pediy6226.htm`](https://www.pediy.com/kssd/pediy06/pediy6226.htm)

### 重启验证——FolderInfo V2.82

- 下载页面： [http://www.skycn.com/soft/8370.html](http://www.skycn.com/soft/8370.html)
- 软件大小： 805 KB
- 软件语言： 英文
- 软件类别： 国外软件 / 共享版 / 磁盘工具
- 应用平台： `Win9x/NT/2000/XP`
- 加入时间： 2003-09-06 15:56:26
- 下载次数： 3584
- 推荐等级： *****
- 开 发 商： [http://www.gemmico.com/](http://www.gemmico.com/)

#### 软件简介
`FolderInfo` 是一个可以帮你分析你的硬盘机状况，并且可以帮你清除一些 temporary 临时且无用的档案和数据夹，也可以打印出你硬盘档案的树状结构表并可储存成 `text`, `html` or `xml` 的档案格式来供你方便的阅读，他也会将你硬盘中每一数据夹所占用的容量一一显示出来，让你更加了解你的硬盘。

#### 软件限制
45 天试用

#### 作者声明
初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

#### 破解工具
`Ollydbg1.09`、`PEiD`、`FileMon`、`W32Dasm 9.0` 白金版

---

#### 过 程

`FolderInfo.exe` 无壳。 Visual C++ 6.0 编写。

现在不加壳的程序越来越少了。跟踪了一下，发现程序对输入的试炼信息简单检测之后就没动静了，看来是重启验证了，于是用 `FileMon`、`RegMon` 监测其启动过程，发现读 `settings.dat` 文件，打开，里面保存着试炼信息。如果验证不通过，程序会删除其中的试炼信息。找到线索就方便了，用 `W32Dasm` 反汇编，查找 key，很容易就找到核心了。

- 用户名：`fly`
- 试炼码：`135724689012`

---

#### 一、输入试炼信息时的验证：

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0044E2A3(C)

```asm
…… …… 省 略 …… ……

:0044E322 F3                repz
:0044E323 A4                movsb
:0044E324 8BFA              mov edi, edx
:0044E326 83C9FF            or ecx, FFFFFFFF
:0044E329 F2                repnz
:0044E32A AE                scasb
:0044E32B F7D1              not ecx
:0044E32D 49                dec ecx
                          ====>取用户名长度

:0044E32E 83F903            cmp ecx, 00000003
                          ====>和 3 比较

:0044E331 7327              jnb 0044E35A
                          ====>不跳则 OVER！      爆破点 ①

:0044E333 50                push eax
* Possible StringData Ref from Data Obj ->"Error"
                          |
:0044E334 68C44D4800        push 00484DC4
* Possible StringData Ref from Data Obj ->"User Name must be at least 3 characters
                          ->"long"
:0044E339 68248F4800        push 00488F24
                          ====>BAD BOY！ 

…… …… 省 略 …… ……

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0044E39B(C)

|
:0044E3EF 8DBC2414020000    lea edi, dword ptr [esp+00000214]
                          ====>EDI=135724689012      试炼码

:0044E3F6 83C9FF            or ecx, FFFFFFFF
:0044E3F9 33C0              xor eax, eax
:0044E3FB 8D9504020000      lea edx, dword ptr [ebp+00000204]
:0044E401 F2                repnz
:0044E402 AE                scasb
:0044E403 F7D1              not ecx
:0044E405 2BF9              sub edi, ecx
:0044E407 8BC1              mov eax, ecx
:0044E409 8BF7              mov esi, edi
:0044E40B 8BFA              mov edi, edx
:0044E40D C1E902            shr ecx, 02
:0044E410 F3                repz
:0044E411 A5                movsd
:0044E412 8BC8              mov ecx, eax
:0044E414 33C0              xor eax, eax
:0044E416 83E103            and ecx, 00000003
:0044E419 F3                repz
:0044E41A A4                movsb
:0044E41B 8BFA              mov edi, edx
:0044E41D 83C9FF            or ecx, FFFFFFFF
:0044E420 F2                repnz
:0044E421 AE                scasb
:0044E422 F7D1              not ecx
:0044E424 49                dec ecx
                          ====>取试炼码长度

:0044E425 83F90C            cmp ecx, 0000000C
                          ====>和 12 比较

:0044E428 7410              je 0044E43A
                          ====>不跳则 OVER！      爆破点 ②

:0044E42A 50                push eax
* Possible StringData Ref from Data Obj ->"Error"
                          |
:0044E42B 68C44D4800        push 00484DC4
* Possible StringData Ref from Data Obj ->"Incorrect registration number"
                          |
:0044E430 68048F4800        push 00488F04
                          ====>BAD BOY！ 
```

---

#### 二、重启时的验证：

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  |:00437FFD(C), :00438003(C)

|

* Possible StringData Ref from Data Obj ->"key"
                          |

```asm
:0043800F 68EC6A4800        push 00486AEC
                          ====>取 settings.dat 中的试炼信息 

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  |:004380B0(C), :004380B5(C)

|
:004380C8 8B74242C          mov esi, dword ptr [esp+2C]
                          ====>ESI=135724689012 试炼码

:004380CC 85F6              test esi, esi
:004380CE 0F95C0            setne al
                          ====>有注册码则 AL=1

:004380D1 84C0              test al, al
:004380D3 742D              je 00438102       爆破点 ③

…… …… 省 略 …… ……

:00438216 AE                scasb
:00438217 F7D1              not ecx
:00438219 49                dec ecx
                          ====>取试炼码长度

:0043821A 83F90C            cmp ecx, 0000000C
:0043821D 0F85D6020000      jne 004384F9
                          ====>跳则 OVER！      爆破点 ④

…… …… 省 略 …… ……

:0043825E 49                dec ecx
                          ====>取试炼码长度

:0043825F 83F90C            cmp ecx, 0000000C
                          ====>和 12 比较

:00438262 0F8591020000      jne 004384F9
                          ====>跳则 OVER！      爆破点 ⑤

…… …… 省 略 …… ……

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:00438276(C)

|
:00438290 8D7C2454          lea edi, dword ptr [esp+54]
:00438294 83C9FF            or ecx, FFFFFFFF
:00438297 33C0              xor eax, eax
:00438299 BB41000000        mov ebx, 00000041
:0043829E F2                repnz
:0043829F AE                scasb
:004382A0 F7D1              not ecx
:004382A2 49                dec ecx
                          ====>取用户名长度

:004382A3 83F905            cmp ecx, 00000005
:004382A6 7359              jnb 00438301
                          ====>不少于 5 位则直接跳下去，否则补数

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:004382F9(C)

|
:004382A8 8D8C2454010000    lea ecx, dword ptr [esp+00000154]
:004382AF 8BC3              mov eax, ebx
:004382B1 6A0A              push 0000000A
:004382B3 51                push ecx
:004382B4 50                push eax
:004382B5 43                inc ebx
* Reference To: MSVCRT._itoa, Ord:0134h
                          |
:004382B6 FF1580C54700      Call dword ptr [0047C580]
:004382BC 8BF8              mov edi, eax
                          ====>EDI=65

:004382BE 83C9FF            or ecx, FFFFFFFF
:004382C1 33C0              xor eax, eax
:004382C3 83C40C            add esp, 0000000C
:004382C6 F2                repnz
:004382C7 AE                scasb
:004382C8 F7D1              not ecx
:004382CA 2BF9              sub edi, ecx
:004382CC 8D542454          lea edx, dword ptr [esp+54]
                          ====>EDX=fly

:004382D0 8BF7              mov esi, edi
:004382D2 8BFA              mov edi, edx
```



```
:004382D4 8BD1             mov edx, ecx

:004382D6 83C9FF           or ecx, FFFFFFFF

:004382D9 F2               repnz

:004382DA AE               scasb

:004382DB 8BCA             mov ecx, edx

:004382DD 4F               dec edi

:004382DE C1E902           shr ecx, 02

:004382E1 F3               repz

:004382E2 A5               movsd

:004382E3 8BCA             mov ecx, edx

:004382E5 83E103           and ecx, 00000003

:004382E8 F3               repz

:004382E9 A4               movsb

                             ====>补足 5 位

:004382EA 8D7C2454         lea edi, dword ptr [esp+54]

                             ====>EDI=fly65

…… …… 省 略 …… ……

:0043839E 8D5A02           lea ebx, dword ptr [edx+02]

                             ====>EBX=5724689012  取试炼码后 10 位

…… …… 省 略 …… ……

:00438401 BFC82F4800       mov edi, 00482FC8

                             ====>EDI=VER21       程序给的固定值

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

00482FC8 的值：

00482FC8  56 45 52 32 31                               VER21

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

:00438406 83C9FF           or ecx, FFFFFFFF

:00438409 33C0             xor eax, eax

:0043840B 8D942454020000   lea edx, dword ptr [esp+00000254]

:00438412 F2               repnz

:00438413 AE               scasb

:00438414 F7D1             not ecx

:00438416 2BF9             sub edi, ecx

:00438418 8BC1             mov eax, ecx

:0043841A 8BF7             mov esi, edi

:0043841C 8BFA             mov edi, edx

:0043841E C1E902           shr ecx, 02

:00438421 F3               repz

:00438422 A5               movsd

:00438423 8BC8             mov ecx, eax

:00438425 83E103           and ecx, 00000003

:00438428 33C0             xor eax, eax

:0043842A F3               repz

:0043842B A4               movsb

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00438441(C)

|

:0043842C 8A8C0454010000   mov cl, byte ptr [esp+eax+00000154]

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

[esp+eax+00000154]：试炼码后 10 位

006EC6E8  57 24 68 90 12                               W$h..

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

:00438433 8A5C0454         mov bl, byte ptr [esp+eax+54]

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

[esp+eax+54]：   补足 5 位的用户名

006EC5E8  66 6C 79 36 35                               fly65

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

:00438437 32D9             xor bl, cl

                             ====>上面内存中的值依次异或

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

异或的结果：

006EC5E8  31 48 11 A6 27                               1H..'

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

:00438439 885C0454         mov byte ptr [esp+eax+54], bl

:0043843D 40               inc eax

:0043843E 83F805           cmp eax, 00000005

:00438441 7CE9             jl 0043842C

:00438443 8D942454020000   lea edx, dword ptr [esp+00000254]爆破点 ⑥

                             ====>EDX=VER21=56 45 52 32 31    固定值

:0043844A 8D442454         lea eax, dword ptr [esp+54]

                             ====>EDI=31 48 11 A6 27      上面运算的结果

:0043844E 52               push edx

:0043844F 50               push eax

* Reference To: MSVCRT._mbscmp, Ord:0159h

                             |

:00438450 FF15C4C54700     Call dword ptr [0047C5C4]

                             ====>比较 CALL！

:00438456 83C408           add esp, 00000008

:00438459 F7D8             neg eax

                             ====>正确则 EAX=0

:0043845B 1BC0             sbb eax, eax

:0043845D 40               inc eax

:0043845E E98D000000       jmp 004384F0

—————————————————————————————————

【算 法  总 结】：

1、注册名至少 3 位，不足 5 位的后补字符：6、5   超过 5 位的只取前 5 位运算

2、注册码 12 位，取后 10 位分成 5 组（依次 2 个 1 组）参与运算

3、注册名前 5 位字符的 HEX 值和注册码的 5 组值异或运算，结果须＝56、45、52、32、31

简单求逆：

1、fly 补足 5 位+65  ==>fly65＝66 6C 79 36 35

2、66 6C 79 36 35 分别异或 56 45 52 32 31＝30 29 2B 04 04

3、得出 30292B0404   前 2 位补上任意字符即可得到注册码：++30292B0404

————————————————————————————————— 

【完 美  爆 破】：

1、0044E331 7327              jnb 0044E35A

     改为：EB27              jmp 0044E35A

2、0044E428 7410              je  0044E43A

     改为：EB10              jmp 0044E43A

3、004380D3 742D              je 00438102

     改为：9090

4、0043821D 0F85D6020000      jne 004384F9

     改为：909090909090

5、00438262 0F8591020000      jne 004384F9

     改为：909090909090

6、00438443 8D942454020000    lea edx, dword ptr [esp+00000254]

     改为：8D542454909090     lea edx, dword ptr [esp+54]

————————————————————————————————— 

【注册信息保存】：

同目录下的 `settings.dat` 文件中：

      <node name="user">
            <attr name="name">fly</attr>
            <attr name="key">++30292B0404</attr>
      </node>

————————————————————————————————— 

【整        理】：

Registartion Name：fly

Registartion Code：++30292B0404

————————————————————————————————— 

          ,      \_/ 

        /| \_.-~/            \_     ,        青春都一饷

       ( /~   /               ~-\_ |
       \`\\  \_/                 ~ )             忍把浮名 

   \_-~~~-.)  )\_\_/;;,.          \_  //'    
  /'\_,   --~     ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂

 \`~ \_( ,\_..-- (     ,;'' /    ~--   /.\_\` 

  /~~//'   /' \`~           ) /--..\_, )\_  \`~    
  "  \`~"  "       \`"       /~'\`    \`\\~~   

                    "     "   "~'  ""

                Cracked By 巢水工作坊——fly [OCN][FCG]

                      2003-10-15  22:15

4dd466d0e1e0cc123afbe504da2462ca
```
