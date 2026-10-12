
# IP Tools v1.07

> 原文：[`www.pediy.com/kssd/pediy01/forum006.htm`](https://www.pediy.com/kssd/pediy01/forum006.htm)

用 `W32DASM` 破解 `IP Tools v1.07`（转载希望保持完整）

作者：丁丁虾 又名：`DDXia` `[CCG]`

- **IP Tools 最新版本**：`1.07`
- **文件大小**：`722KB`
- **软件授权**：共享软件
- **使用平台**：`Win95/98/2000/NT`

**软件简介**：

集成了许多 `TCP/IP` 实用工具于一体，比如本地信息、连接信息、端口扫描、`PING`、`TRACE`、`WHOIS`、`FINGER`、`NSLOOKUP`、`Telnet` 客户端、`NETBIOS` 信息、`IP` 监视器等等

`http://www.newhua.com/down/ip-tools.exe`

上 `W32DASM`！我掩护！把它载入 `W32DASM`，`Search Strings`——也就是注册失败时，跳出对话框中的羊肉串——>`"Sorry, but Name or Registration "`

```
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

:0048F771(C), :0048F78B(C), :0048F7C6(C), :0048F7EE(C)

* Possible StringData Ref from Code Obj ->"Sorry, but Name or Registration "
                                ->"number is wrong !"

:0048F927 B884FA4800                   mov eax, 0048FA84
:0048F92C E80F28FBFF                   call 00442140

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

:0048F709(C), :0048F925(U)

:0048F931 33C0                         xor eax, eax
:0048F933 5A                           pop edx
:0048F934 59                           pop ecx
:0048F935 59                           pop ecx
:0048F936 648910                       mov dword ptr fs:[eax], edx
```

这回可不能按老方法啦！信息的上方可以看到如下四个地址:

`:0048F771(C), :0048F78B(C), :0048F7C6(C), :0048F7EE(C)`

瞧一瞧是什么的家伙！

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:0048F699(C)

:0048F6BB 51                           push ecx
:0048F6BC 51                           push ecx
:0048F6BD 51                           push ecx
:0048F6BE 53                           push ebx
:0048F6BF 56                           push esi
:0048F6C0 57                           push edi
:0048F6C1 8BD8                         mov ebx, eax
:0048F6C3 8B3D1CCB4A00                 mov edi, dword ptr [004ACB1C]
:0048F6C9 33C0                         xor eax, eax
:0048F6CB 55                           push ebp
:0048F6CC 685CF94800                   push 0048F95C
:0048F6D1 64FF30                        push dword ptr fs:[eax]
:0048F6D4 648920                        mov dword ptr fs:[eax], esp
:0048F6D7 8B1524CA4A00                 mov edx, dword ptr [004ACA24]
:0048F6DD 8B12                         mov edx, dword ptr [edx]
:0048F6DF 8B07                         mov eax, dword ptr [edi]
:0048F6E1 8B80F0010000                 mov eax, dword ptr [eax+000001F0]
:0048F6E7 E8C82CF9FF                   call 004223B4
:0048F6EC 8B15CCC84A00                 mov edx, dword ptr [004AC8CC]
:0048F6F2 8B12                         mov edx, dword ptr [edx]
:0048F6F4 8B07                         mov eax, dword ptr [edi]
:0048F6F6 8B80F4010000                 mov eax, dword ptr [eax+000001F4]
:0048F6FC E8B32CF9FF                   call 004223B4
:0048F701 8B07                         mov eax, dword ptr [edi]
:0048F703 E80812FAFF                   call 00430910
:0048F708 48                           dec eax
:0048F709 0F8522020000                 jne 0048F931
:0048F70F 8D55F8                       lea edx, dword ptr [ebp-08]
:0048F712 8B07                         mov eax, dword ptr [edi]
:0048F714 8BB0F0010000                 mov esi, dword ptr [eax+000001F0]
:0048F71A 8BC6                         mov eax, esi
:0048F71C E8632CF9FF                   call 00422384
:0048F721 8B45F8                       mov eax, dword ptr [ebp-08]
:0048F724 8D55FC                       lea edx, dword ptr [ebp-04]
:0048F727 E8D87FF7FF                   call 00407704
:0048F72C 8B55FC                       mov edx, dword ptr [ebp-04]
:0048F72F 8BC6                         mov eax, esi
:0048F731 E87E2CF9FF                   call 004223B4
:0048F736 8D55F8                       lea edx, dword ptr [ebp-08]
:0048F739 8B07                         mov eax, dword ptr [edi]
:0048F73B 8BB0F4010000                 mov esi, dword ptr [eax+000001F4]
:0048F741 8BC6                         mov eax, esi
:0048F743 E83C2CF9FF                   call 00422384
:0048F748 8B45F8                       mov eax, dword ptr [ebp-08]
:0048F74B 8D55FC                       lea edx, dword ptr [ebp-04]
:0048F74E E8B17FF7FF                   call 00407704
:0048F753 8B55FC                       mov edx, dword ptr [ebp-04]
:0048F756 8BC6                         mov eax, esi
:0048F758 E8572CF9FF                   call 004223B4
:0048F75D 8D55F8                       lea edx, dword ptr [ebp-08]
:0048F760 8B07                         mov eax, dword ptr [edi]
:0048F762 8B80F0010000                 mov eax, dword ptr [eax+000001F0]
:0048F768 E8172CF9FF                   call 00422384
:0048F76D 837DF800                     cmp dword ptr [ebp-08], 00000000
:0048F771 0F84B0010000                 je 0048F927
                               ^^^^^^^^^^^----->
:0048F777 8D55F4                       lea edx, dword ptr [ebp-0C]
:0048F77A 8B07                         mov eax, dword ptr [edi]
:0048F77C 8B80F4010000                 mov eax, dword ptr [eax+000001F4]
:0048F782 E8FD2BF9FF                   call 00422384
:0048F787 837DF400                     cmp dword ptr [ebp-0C], 00000000
:0048F78B 0F8496010000                 je 0048F927
                               ^^^^^^^^^^^----->
:0048F791 8D55F0                       lea edx, dword ptr [ebp-10]
:0048F794 8B07                         mov eax, dword ptr [edi]
:0048F796 8B80F0010000                 mov eax, dword ptr [eax+000001F0]
:0048F79C E8E32BF9FF                   call 00422384
:0048F7A1 8B45F0                       mov eax, dword ptr [ebp-10]
:0048F7A4 E80792FFFF                   call 004889B0
:0048F7A9 8BF0                         mov esi, eax
:0048F7AB 8D55F0                       lea edx, dword ptr [ebp-10]
:0048F7AE 8B07                         mov eax, dword ptr [edi]
:0048F7B0 8B80F4010000                 mov eax, dword ptr [eax+000001F4]
:0048F7B6 E8C92BF9FF                   call 00422384
:0048F7BB 8B45F0                       mov eax, dword ptr [ebp-10]
:0048F7BE E87992FFFF                   call 00488A3C
:0048F7C3 663BF0                       cmp si, ax
:0048F7C6 0F855B010000                 jne 0048F927
                               ^^^^^^^^^^^----->
:0048F7CC A148C94A00                   mov eax, dword ptr [004AC948]
:0048F7D1 BAFF010000                   mov edx, 000001FF
:0048F7D6 E8BD91FFFF                   call 00488998
:0048F7DB 8BF0                         mov esi, eax
:0048F7DD A19CC84A00                   mov eax, dword ptr [004AC89C]
:0048F7E2 BAFF010000                   mov edx, 000001FF
:0048F7E7 E8AC91FFFF                   call 00488998
:0048F7EC 3BF0                         cmp esi, eax
:0048F7EE 0F8533010000                 jne 0048F927
                               ^^^^^^^^^^^----->
:0048F7F4 8D55F8                       lea edx, dword ptr [ebp-08]
:0048F7F7 8B07                         mov eax, dword ptr [edi]
:0048F7F9 8B80F0010000                 mov eax, dword ptr [eax+000001F0]
:0048F7FF E8802BF9FF                   call 00422384
:0048F804 8B55F8                       mov edx, dword ptr [ebp-08]
:0048F807 A124CA4A00                   mov eax, dword ptr [004ACA24]
:0048F80C E89744F7FF                   call 00403CA8
:0048F811 8D55F8                       lea edx, dword ptr [ebp-08]
:0048F814 8B07                         mov eax, dword ptr [edi]
:0048F816 8B80F4010000                 mov eax, dword ptr [eax+000001F4]
:0048F81C E8632BF9FF                   call 00422384
:0048F821 8B55F8                       mov edx, dword ptr [ebp-08]
:0048F824 A1CCC84A00                   mov eax, dword ptr [004AC8CC]
:0048F829 E87A44F7FF                   call 00403CA8
:0048F82E 8BC3                         mov eax, ebx
:0048F830 E877FAFFFF                   call 0048F2AC
```

以上的跳转都是没有交钱就有你好看的！

往下看，离它对你说 `"Thank You"` 很近了，加油哦 `^_^`

怎么处理上面四个保镖就看你们的罗：） 问我！我功力比较低刚 `10` 点，只好动粗啦，全部 `nop` 掉。爽.......哈...哈..哈.



## 破解记录

`* Possible StringData Ref from Code Obj ->"Thank You for registering"`

```
:0048F835 B874F94800              mov eax, 0048F974
:0048F83A E80129FBFF              call 00442140
:0048F83F B201                    mov dl, 01
:0048F841 A16C364300              mov eax, dword ptr [0043366C]
:0048F846 E8B53FFAFF              call 00433800
:0048F84B 8BD8                    mov ebx, eax
:0048F84D B101                    mov cl, 01
```

**注释**：
- `Thank You for registering`：好亲热的话语，我中听！
- 操作提示：来来，干一杯水、庆贺一下。刚刚想去解决一下温饱问题，一运行，到没有看见 `Thank you` 和 `Sorry`，反倒看见了 `"Program was corrupted!!"`。没有搞错吧！居然用过去时，明明是正在进行嘛！不管了，我可要去吃一点饼干罗! :))
- (评语：这家伙连一点责任心都没有，唉，一代不如一代啦)

(待续）

> **完成时间**：2000.2.19 中午 16:18  
> **耗时**：暂时不知道 :((

---

## 用 W32DASM 破解 IP Tools 最新版本：1.07

**作者**：丁丁虾 又名：`DDXia[CCG]`  
*(转载希望保持完整)*

回来啦！`come on!` 上回可能是太饿的缘故。

从头瞧一便是什么故障！如果实在不行的化，干脆，把电脑从四楼扔下去，然后再下楼拣回来用，那时一定很理智！`^\_^`

先是出现 `"Self test.."`，然后就出现 `"Program was corrupted !"`，外国人老早就知道你会修改那跳转的地方！说不定他就是 `cracker!`，不过，他还是很好心的告诉你，我正在自检，请勿打扰！看来他早有准备，幸好是没有格式化我的硬盘。否则不是电脑下去了，我就先行一步.....（干嘛！干嘛！）...别打岔，回老家种田去。

其实也容易的，出乎意料的，竟然可以查到 `"Program was corrupted !"`。

### 程序分析 1

`* Possible StringData Ref from Code Obj ->"] Self test .."`

```
:004A6958 68506C4A00              push 004A6C50
:004A695D 8D45F0                  lea eax, dword ptr [ebp-10]
:004A6960 BA03000000              mov edx, 00000003
:004A6965 E826D6F5FF              call 00403F90
:004A696A 8B55F0                  mov edx, dword ptr [ebp-10]
:004A696D 8B03                    mov eax, dword ptr [ebx]
:004A696F 8B80DC010000            mov eax, dword ptr [eax+000001DC]
:004A6975 8B8030010000            mov eax, dword ptr [eax+00000130]
:004A697B 8B08                    mov ecx, dword ptr [eax]
:004A697D FF5134                  call [ecx+34]
:004A6980 E81FD0FDFF              call 004839A4
:004A6985 8B1554C84A00            mov edx, dword ptr [004AC854]
:004A698B 3B82B4000000            cmp eax, dword ptr [edx+000000B4]
:004A6991 740F                    je 004A69A2
```

**注释**：
- 第 `004A6991` 行 `je 004A69A2` 这句很明显的就是让我们 `JMP`。
- 有兴趣的跟踪以上的程序，大大可以提高功力！

### 程序分析 2

`* Possible StringData Ref from Code Obj ->"Program was corrupted !"`

```
:004A6993 B8686C4A00              mov eax, 004A6C68
:004A6998 E8A3B7F9FF              call 00442140
:004A699D E93C020000              jmp 004A6BDE
```

### 跳转目标分析

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`
`:004A6991(C)`

```
:004A69A2 68306C4A00              push 004A6C30
:004A69A7 E89C22F6FF              call 00408C48
:004A69AC 83C4F8                  add esp, FFFFFFF8
:004A69AF DD1C24                  fstp qword ptr [esp]
:004A69B2 9B                      wait
:004A69B3 8D45EC                  lea eax, dword ptr [ebp-14]
:004A69B6 E81D2EF6FF              call 004097D8
:004A69BB FF75EC                  push [ebp-14]
```

### 程序分析 3

`* Possible StringData Ref from Code Obj ->"] App Init .."`

```
:004A69BE 68886C4A00              push 004A6C88
:004A69C3 8D45F0                  lea eax, dword ptr [ebp-10]
:004A69C6 BA03000000              mov edx, 00000003
:004A69CB E8C0D5F5FF              call 00403F90
:004A69D0 8B55F0                  mov edx, dword ptr [ebp-10]
:004A69D3 8B03                    mov eax, dword ptr [ebx]
:004A69D5 8B80DC010000            mov eax, dword ptr [eax+000001DC]
:004A69DB 8B8030010000            mov eax, dword ptr [eax+00000130]
:004A69E1 8B08                    mov ecx, dword ptr [eax]
:004A69E3 FF5134                  call [ecx+34]
```

### 测试与逆向分析

OK! 可以收工了吗？问它吧，现在是它说了算！启动运行，没有事，注册，`Thank you`。再运行一下，又有毛病了！！！有完没完的，真是过分！不和你玩！（即时评语：又在耍小孩子脾气！）

说什么呢？？？哼！我跟你没有完！开始启用另一利器：

`------》》REGMONITOR--Regmonitor`，顿时把它看得个清清楚楚、明明白白。

原来它把 `USER` 和 `PASS` 放在注册表中，而且还加了密！挺有责任心的，是位好同志！

在注册表路径：
`HKCU\Software\Microsoft\Windows\CurrentVersion\Devices\0010\DATA1`
值：`SUCCESS "31323132"`
`^^^^^^^` -> 同下---这些已经是我填好的，原来都是乱码

在注册表路径：
`HKCU\Software\Microsoft\Windows\CurrentVersion\Devices\0010\DATA2`
值：`SUCCESS "3132303435"`
`^^^^^^^^` -> 随便填一些正常数字。

不玩了，不玩了！今天是元宵，又是星期六。睡觉，我的最爱 `^\_^`

> **完成时间**：2000.2.19 中午 17:20  
> **耗时**：2 小时又 34 分钟外带 20 分的中餐时间
