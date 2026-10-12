
# SNAGIT32 5.10

> 原文：[`www.pediy.com/kssd/pediy03/forum345.htm`](https://www.pediy.com/kssd/pediy03/forum345.htm)

熬夜写了第一篇破解教程，希望能敲开（不是敲破，^-^）`[CCG]`的大门，不足之处还望 `SUN.BIRD[CCG]`及各位大客批评指正。

`snagit 32 v5.10`

`SnagIt` 可以方便快速地截取 `Windows` 桌面上的任何内容，它可以卷动 `Web` 页面供你一次截取，可以捕捉视频图像，还可以抓取文本内容。 --我最欣赏的就是这点。

下载：`URL ftp://263.idp.com.cn/pub/newsoft/graphics/grp-cap/snagit.exe`

工具：`Trw2k 1.22`，`w32dsm`。

先来破解 `snagit32.exe`:

输入任意注册码后，下断点 `hmemcpy`，断住后搜索这个字符串地址为 `xxxxxxx`，下断点 `bpm xxxxxxx`。再断后到处找找，会看到下面的内容：

```
:00447EA6 85C0                        test eax, eax
:00447EA8 8945EC                      mov dword ptr [ebp-14], eax
:00447EAB 0F84DD020000                je 0044818E
:00447EB1 83FF0E                      cmp edi, 0000000E
:00447EB4 0F82C3020000                jb 0044817D          // 检查原始注册码长度，不能小于 0x0E(14)。
:00447EF3 FF1508C44A00                Call dword ptr [004AC408] //过滤掉注册码中的-%@#^&*……)，长度-〉eax
:00447EF9 6A31                        push 00000031
:00447EFB 83F80E                      cmp eax, 0000000E     //看看这个长度是否等于 14
:00447EFE 59                          pop ecx
:00447EFF 0F8598000000                jne 00447F9D          //不相等的话，注册失败
:00447F05 8B55EC                      mov edx, dword ptr [ebp-14]
:00447F08 807A0C38                    cmp byte ptr [edx+0C], 38 //注册码的倒数第二位= '8'?
:00447F0C 8A420D                      mov al, byte ptr [edx+0D]
:00447F0F 0F8588000000                jne 00447F9D          //跳就按新用户处理，不跳则可能是老用户
:00447F15 3C34                        cmp al, 34            //---------------------------------->
:00447F17 740C                        je 00447F25           //倒数第一位等于 2，3，4 则为老用户
:00447F19 3C33                        cmp al, 33
:00447F1B 7408                        je 00447F25
:00447F1D 3C32                        cmp al, 32
:00447F1F 7404                        je 00447F25           //<----------------------------------
:00447F21 3AC1                        cmp al, cl
:00447F23 7578                        jne 00447F9D          //跳就是新用户
```

就按照新用户追下去吧

```
:00448078 E8AAC60000                  call 00454727       <---密码在此验证
:0044807D 8BD8                        mov ebx, eax        <---注意：这里将 eax-->ebx

* Possible Reference to Dialog: DialogID_0098, CONTROL_ID:00FF, ""

                                 |
:0044807F 6AFF                        push FFFFFFFF
:00448081 8D4DF0                      lea ecx, dword ptr [ebp-10]
:00448084 895DE4                      mov dword ptr [ebp-1C], ebx
:00448087 E826FA0300                  call 00487AB2
:0044808C 80FB01                      cmp bl, 01          <---到这里才判断 bl 是否为 1
:0044808F 0F85BF000000                jne 00448154        <---不等就注册失败
```

所以我们要跟进 `":00448078 E8AAC60000 call 00454727"` 这个 `call` 看看哪里改变了 `eax`。

注册码的判断比较繁琐，当我发现下面的 `xor eax,eax` 后，便放弃了找注册码的念头，只要改 2 个字节就可以完工，何必浪费青春呢？

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0044793D(C)

|
:0044794F 33C0                        xor eax, eax          //到这里之前 eax 已经是 0 了，就改它好了
                                        //改为 40(inc eax) 90 (nop)
:00447951 EBF7                        jmp 0044794A
:00447953 B839814A00                  mov eax, 004A8139
:00447958 E8BF570200                  call 0046D11C
:0044795D 51                          push ecx
```

到这里，`snagit32.exe` 已经破掉了：

有下面两个注册方法：

1. 把上面所提到的关键点都修改一下，这样注册码可以随意输入，长度，内容都不限。
2. 只修改 2 个字节：
   `s: 33C0EBF7B839814A00`
   `r: 4090--------------`
   然后在输入注册码的时候输入：`xxxxxxxxxxxxAx`（长度 14，其中 `x` 可以为任意字母或数字，`A` 为不等于 8 的任意字母或数字）

再来破 `studio.exe`。

找到

```
:0041D457 C784242C01000000000000  mov dword ptr [esp+0000012C], 00000000
:0041D462 837AF80E                cmp dword ptr [edx-08], 0000000E  //判断你刚刚在 snagit 主输入的注册码长度>=14
:0041D466 7C6C                    jl 0041D4D4                     //小于就抱歉了     改为 9090
:0041D468 8B542410                mov edx, dword ptr [esp+10]
```

再向下找

```
:0041D4B5 E856000000              call 0041D510               //这个 call 里面判断你的注册码正确与否。
                                              //如果有朋友想找注册码，我想在这边找比较好。
:0041D4BA 3C01                    cmp al, 01
:0041D4BC 7505                    jne 0041D4C3                //跳出就 over，改为 9090
:0041D4BE BB01000000              mov ebx, 00000001
```

到这里，`studio.exe` 已经破掉了：

`s: f80e7c6c8b542410`（如果你的密码本身输的就是 14 位，这里不用改了）
`r: ----9090--------`

`s: 3c017505bb010000`
`r: ----9090--------`

还有一点要说说，在 `snagit32` 中注册成功后会提示输入用户名，它会自动将你登陆的用户名字放在编辑框内。因此一定用到了这样的一个 `win32 api`，在 `w32dsm` 反编译结果中查找 `get`，结果一下子就找到了：`GetUserName`((得到登陆用户名)，和 `GetComputerName`（得到机器名）。昨天那位提问怎样在程序中关联打开文档的`[CCG]`的朋友，你也可以用这样的方法，直接搜索 `Reg`，会找到几乎所有的操作注册表的 `win32 api`，如下：

```
Import Module 009: ADVAPI32.dll

Addr:0017E894 hint(015E) Name: RegCreateKeyA
Addr:0017E8A4 hint(017B) Name: RegQueryValueExA
Addr:0017E92E hint(00AA) Name: GetFileSecurityA
Addr:0017E844 hint(015B) Name: RegCloseKey
Addr:0017E852 hint(0172) Name: RegOpenKeyExA
Addr:0017E862 hint(016A) Name: RegEnumValueA
Addr:0017E872 hint(017A) Name: RegQueryValueA
Addr:0017E884 hint(0185) Name: RegSetValueA
Addr:0017E90C hint(0166) Name: RegEnumKeyA
Addr:0017E91A hint(019B) Name: SetFileSecurityA
Addr:0017E8B8 hint(0186) Name: RegSetValueExA
Addr:0017E8CA hint(015F) Name: RegCreateKeyExA
Addr:0017E8DC hint(0164) Name: RegDeleteValueA
Addr:0017E8EE hint(0162) Name: RegDeleteKeyA
Addr:0017E8FE hint(0171) Name: RegOpenKeyA
```

这样再去查 `msdn`，目的性就强的多。：）

`傲世男儿`

`09/28/2000`   `4dd466d0e1e0cc123afbe504da2462ca`
