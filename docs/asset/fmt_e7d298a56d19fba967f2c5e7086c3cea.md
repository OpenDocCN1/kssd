
# Advanced Find and Replace V1.2

> 原文：[`www.pediy.com/kssd/pediy01/forum031.htm`](https://www.pediy.com/kssd/pediy01/forum031.htm)

破解 `Advanced Find and Replace V1.2.2`（转载希望保持完整）

级别：原始社会-----前辈们请指点。

作者：丁丁虾 又名：`DDXia` \[CCG\]

目的：非常的天真无‘牙’------别无所求（不以任何商业用途为目的）

软件名称：`Advanced Find and Replace`

整理日期：`2000.3.30`

最新版本：`1.2.2`

文件大小：`959KB`

软件授权：共享软件

使用平台：`Win95/98/NT`

发布公司：`Home Page`

软件简介：

文档内容查找工具，支持通配符，支持普通文本、HTML 和 RTF 文件，您可以插入、替换、移除整个段落。

```
http://www.newhua.com.cn/down/c_afr121.zip
```

好久没有出来混，不知道现在哪位是道上的大哥，知道的告诉我一声哦！呵 。。 呵。。^\_^

终于有时间定下心来，写篇破解文了。 （最近工作比较烦，烦、烦阿、还是觉得写破文比较开心，有福同享嘛！：））

先运行程序，注册，看一看出错的信息是什么？`"Sorry, registration code is not correct"`

启动 `W32DASM` 载入主程序，可以去打一壶凉白开（那可是景德镇小茶壶哦）！查找 `"sorry,registration."` ok 发现目标-----方位在:`00473C92`

```
* Possible StringData Ref from Code Obj ->"Sorry, registration code is not "
                                          ->"correct"
                                              |
:00473C92 B8143D4700                      mov eax, 00473D14
:00473C97 E838F3FDFF                      call 00452FD4
:00473C9C C6831103000000                  mov byte ptr [ebx+00000311], 00
```

往上糗一糗：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00473C60(C)
^^^^^^^^^^^^^^-------->这个有问题。查找它的代码！一定是出错后的跳转

:00473C87 6A00                            push 00000000
:00473C89 668B0DD83C4700                  mov cx, word ptr [00473CD8]
:00473C90 B201                            mov dl, 01

* Possible StringData Ref from Code Obj ->"Sorry, registration code is not "
                                          ->"correct"
                                              |
:00473C92 B8143D4700                      mov eax, 00473D14
:00473C97 E838F3FDFF                      call 00452FD4
:00473C9C C6831103000000                  mov byte ptr [ebx+00000311], 00
========================================================================
:00473C2D 8B83D0020000                    mov eax, dword ptr [ebx+000002D0]
:00473C33 E870BCFBFF                      call 0042F8A8-------->取你打入的公司名--DDXia
:00473C38 8B45FC                          mov eax, dword ptr [ebp-04]
:00473C3B 50                              push eax
:00473C3C 8D55F4                          lea edx, dword ptr [ebp-0C]
:00473C3F 8B83D4020000                    mov eax, dword ptr [ebx+000002D4]
:00473C45 E85EBCFBFF                      call 0042F8A8--------->取你打入的注册码---123456789
:00473C4A 8B55F4                          mov edx, dword ptr [ebp-0C]
:00473C4D 8D4DF8                          lea ecx, dword ptr [ebp-08]
:00473C50 8BC3                            mov eax, ebx
:00473C52 E8A9F4FFFF                      call 00473100--------->通过注册码来计算你公司名
:00473C57 8B55F8                          mov edx, dword ptr [ebp-08]
:00473C5A 58                              pop eax
:00473C5B E81802F9FF                      call 00403E78--------->计算结果和公司名比较
:00473C60 7525                            jne 00473C87 --------->如果不对，就跳到 SORRY：（
:00473C62 8BC3                            mov eax, ebx
:00473C64 E853FAFFFF                      call 004736BC
:00473C69 6A00                            push 00000000
:00473C6B 668B0DD83C4700                  mov cx, word ptr [00473CD8]
:00473C72 B202                            mov dl, 02

* Possible StringData Ref from Code Obj ->"Thank you very much for registration"
                                              |
:00473C74 B8E43C4700                      mov eax, 00473CE4
:00473C79 E856F3FDFF                      call 00452FD4
:00473C7E C6831103000001                  mov byte ptr [ebx+00000311], 01
:00473C85 EB1C                            jmp 00473CA3
```

其中有一个问题是，我们键入的是乱码，所以它产生的也是乱码，而且有一些不是 ASC 码，不能得到正确的公司名。因此，只能是暴力解啦！！---不好意思\*-\*；

把 `:00473C60 7525 jne 00473C87` 改为 `je 00473c87`

再运行程序，还是显示没有注册。看来程序的开始也做了检查（这种注册方式时常有的哦）

```
:00473C5B E81802F9FF                      call 00403E78--------->计算结果和公司名比较
```

用 `W32DASM` 看一下其他地方有没有 `call 00403E78` 的。

```
* Referenced by a CALL at Addresses:
:00411FEB , :00416C2B , :00417C2B , :0041CDE9 , :0041DCD0
:00422617 , :00422635 , :004231D0 , :004281DE , :0042E90A
:0042E995 , :0042F6C5 , :0042F900 , :0043140F , :0043A607
:0043ABF1 , :0043AD61 , :0043AED5 , :0043DE6D , :0043E293
:0043E352 , :0043E76E , :0043E9BF , :0043F23F , :0043FC4D
:0043FE03 , :004409D1 , :00440B88 , :00440F63 , :0044D621
:0044E23F , :0045B773 , :0045BBAE , :0045D820 , :0045D928
:0045E953 , :0045F618 , :0046136F , :0046F81E , :00470155
:004701B5 , :0047124C , :00472728 , :00472927 , :004729FF
:00472B54 , :00473586 , :004739E7 , :00473C5B , :00476244
:004762E7 , :00476347 , :00476356 , :00476749 , :00476854
:00476960 , :0047747D , :0047748F

:00403E78 53                              push ebx
:00403E79 56                              push esi
:00403E7A 57                              push edi
:00403E7B 89C6                            mov esi, eax
```

WHAT？？？？？:-o 我好怕哦！！！不是跟我过不去吗？？看来作者还是挺逗的嘛！

哈哈哈：）） 今晚又是西雅图不眠夜啦！！！

Stop！！！！有新发现！

```
:00473C52 E8A9F4FFFF                      call 00473100--------->通过注册码来计算你公司名
```

用 `W32DASM` 看一下其他地方有没有 `call 00403E78` 的

```
* Referenced by a CALL at Addresses:
:0047357B , :004739DE , :00473C52

:00473100 55                              push ebp
:00473101 8BEC                            mov ebp, esp
:00473103 83C4E0                          add esp, FFFFFFE0
:00473106 53                              push ebx
:00473107 33DB                            xor ebx, ebx
:00473109 895DF8                          mov dword ptr [ebp-08], ebx
```

还好了，只有三个地方，而且 `00473C52` 不用作断点的（为什么！看一看上面的程序，不是启动中的判断）

在 `:0047357B`，中断了，让我们分析一下

```
:0047357B E880FBFFFF                      call 00473100----->计算程序
:00473580 8B45E4                          mov eax, dword ptr [ebp-1C]
:00473583 8B55F8                          mov edx, dword ptr [ebp-08]
:00473586 E8ED08F9FF                      call 00403E78--------->比较程序
:0047358B 7523                            jne 004735B0---------> 改 jne---->je.
```

再试一下，只改 `0047358B jne` 改为 `je` 可不可以呢？？？？ 当然可以！

OK！收工！今夜没有星星！！！

完成时间

`2000.3.31 00:54`
