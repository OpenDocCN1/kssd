
# `lockdown2000` v7.0.0.6

> 原文：[`www.pediy.com/kssd/pediy03/forum485.htm`](https://www.pediy.com/kssd/pediy03/forum485.htm)

如何破解 `lockdown2000` v7.0.0.6

-----------------------------

雪椰

2000,12,25am

Email: `wocy@263.net`

`Http://wocy.yeah.net`(有本例中的补丁）

------------------------------

1，检查文件被压缩过了，因为懒得脱壳，所以决定内存补丁

2，以前有一点资料（看雪精华 2）

标题: `Lockdown2000_7.0.0.1` 破解手记 (3 千字)

- 作 者：十夜

- 时 间：2000-5-26 12:02:17

详细信息:

`Lockdown2000_7.0.0.1` 破解手记

本人属懒派狂人，自下载 `Lockdown2000_7.0.0.1` 后一直等待破解的出现，但是始终不见其踪影。

本着你不烦我我不破你的原则平静的度过了十天，终于，讨厌的东西出现了，看来只有按原则办事了，不想这东西加了壳，又臭又硬，还好在吴朝相的主页找到了脱壳后的汉化版本，省了一道工序，可以开始工作了。

用 `W32Dasm` 反编译 `Lockdown2000.exe`，找那个让我讨厌的东西(过期提示)，如下所示：

```
:004C313D E8321CFEFF call 004A4D74
:004C3142 84C0 test al, al
:004C3144 7546 jne 004C318C
:004C3146 E8B516FEFF call 004A4800
:004C314B 48 dec eax
:004C314C 7D27 jge 004C3175
:004C314E A14C9E4D00 mov eax, dword ptr [004D9E4C]
:004C3153 8B00 mov eax, dword ptr [eax]
:004C3155 E82AC5F8FF call 0044F684
* Possible StringData Ref from Code Obj ->"The trial period on this demo "
                                  ->"of LockDown 2000 has expired."
```

往上看看 `004C3144` 那一行能跳过过期提示，改为 `jmp 004C318C` 就不会过期了，但这也太那个……

注意一下 `call 004A4D74`，看起来好像很有意思，到 `004A4D74` 去看看：

```
* Referenced by a CALL at Addresses:
:004B5127 , :004B524E , :004B56FC , :004B5C0F , :004BD36D
:004BE723 , :004C313D , :004C6AF4 , :004CBA42 , :004CCFAB
:004CF4A3 , :004CF5FF

:004A4D74 55 push ebp
:004A4D75 8BEC mov ebp, esp
:004A4D77 B905000000 mov ecx, 00000005
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004A4D81(C)

:004A4D7C 6A00 push 00000000
:004A4D7E 6A00 push 00000000
:004A4D80 49 dec ecx
:004A4D81 75F9 jne 004A4D7C
:004A4D83 53 push ebx
:004A4D84 56 push esi
:004A4D85 33C0 xor eax, eax
:004A4D87 55 push ebp
:004A4D88 68E0514A00 push 004A51E0
:004A4D8D 64FF30 push dword ptr fs:[eax]          <<--- (1)
:004A4D90 648920 mov dword ptr fs:[eax], esp

* Possible StringData Ref from Code Obj ->"LD2KReg.ini"

:004A4D93 B9F8514A00 mov ecx, 004A51F8           <<--- (2)
:004A4D98 B201 mov dl, 01
:004A4D9A A1C8A14700 mov eax, dword ptr [0047A1C8]
:004A4D9F E8CC54FDFF call 0047A270
:004A4DA4 8BD8 mov ebx, eax
:004A4DA6 6A00 push 00000000
:004A4DA8 8D45E0 lea eax, dword ptr [ebp-20]
:004A4DAB 50 push eax
* Possible StringData Ref from Code Obj ->"Register"

:004A4DAC B90C524A00 mov ecx, 004A520C
* Possible StringData Ref from Code Obj ->"Register"

:004A4DB1 BA0C524A00 mov edx, 004A520C
:004A4DB6 8BC3 mov eax, ebx
:004A4DB8 8B30 mov esi, dword ptr [eax]
:004A4DBA FF16 call dword ptr [esi]
:004A4DBC 8D45DC lea eax, dword ptr [ebp-24]
:004A4DBF E8B4F9FFFF call 004A4778
:004A4DC4 8B55DC mov edx, dword ptr [ebp-24]
:004A4DC7 8B45E0 mov eax, dword ptr [ebp-20]
:004A4DCA E8BDF2F5FF call 0040408C
:004A4DCF 7537 jne 004A4E08
……………………………………………………
```

好像很受欢迎的样子，而且它还提到了 `LD2KReg.ini`，再看看后面的代码，应该是判断注册的地方，程序很简单，将 `004A4DCF` 的 `jne 004A4E08` 改为 `je 004A4E08` 即可成为注册版。

3，估计 7006 版跟 7001 版差不多，关键是找到这个检验主函数

上面这段函数中（1）-（2）

这段函数估计不会变。代码是 `64`, `89`, `20`, `b9`

4, 搜索 `15f:400000 l fffffff 64`, `89`, `20`, `b9`
   `find date at 15f:xxxxxx`
   `bpx 15f:xxxxxx`

5, 重启程式，中断后跟踪，发现这个主函数在 `15f:4a4a64`
   `date:55,8b,ec,b9,05,00`
   改为: `b8,01,00,00,00,c3`( `mov eax,1 ret` )

6, `rpp`, 这个世界清净了.

标题: 呵呵，我只是知道而已，我好懒的，我觉得这个软体很讨厌，他能自动去 `server` 校验，就有可能把你的个人信息偷走，不过我想断它的 `socket api` 有可能发现那里跳出的 `nag` 窗。呵呵！

内容:

# 如何破解 `lockdown2000` v7.0.0.6(补)

-----------------------------

雪椰

2001,01,03pm

Email: `wocy@263.net`

`Http://wocy.yeah.net`(有本例中的补丁）

------------------------------

这个补丁的不完全不解，在我主页上，感觉不好。刚好今天有空，搞定它。

1,

软件用 `aspack` 加壳，先脱壳

2，

前面的 7006 补丁会出现连上服务器验证。失败后提示信息->

`"LockDown 2000 has detected that you are running an unauthorized copy..."`

(也可根据楼上那位根据 `http://www.lockdown2000.com/order_now.html`)

`you will go to ...`

-------------------------------------

```
* Possible StringData Ref from Code Obj ->"LockDown 2000 has detected that "
                                  ->"you are running an unauthorized "
                                  ->"copy.
Licenses are purchased "
                                  ->"on a per computer basis.
If you "
                                  ->"believe you are seeing this message "
                                  ->"in error or would like to purchase "
                                  ->"a license for this computer, please "
                                  ->"visit:
http://www.lockdown2000.com/
or "
                                  ->"call:
(800)422-5130 or (603)740-4590"

:004D739C B85C754D00 mov eax, 004D755C
:004D73A1 E83EF6F7FF call 004569E4
:004D73A6 C6059481610001 mov byte ptr [00618194], 01
:004D73AD 8B45FC mov eax, dword ptr [ebp-04]
:004D73B0 E8D351F7FF call 0044C588
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004D7356(C) -----------------> (c)

:004D73B5 E8F2C3F2FF call 004037AC
```

---------------------------------------------

Retrance (C) it to.....

```
:004D72C8 E883BDF2FF call 00403050
:004D72CD 84DB test bl, bl
:004D72CF 0F85EC000000 jne 004D73C1 ---------->point (1)
:004D72D5 803DA481610000 cmp byte ptr [006181A4], 00
:004D72DC 740D je 004D72EB
:004D72DE 803D08CB4D0000 cmp byte ptr [004DCB08], 00
:004D72E5 0F84D6000000 je 004D73C1 ---------->jmp exit point.
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004D72DC(C)

:004D72EB 803D08CB4D0000 cmp byte ptr [004DCB08], 00
:004D72F2 7510 jne 004D7304
:004D72F4 8B45FC mov eax, dword ptr [ebp-04]
:004D72F7 E8C4030000 call 004D76C0
:004D72FC 84C0 test al, al
:004D72FE 0F84BD000000 je 004D73C1
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004D72F2(C)

:004D7304 803D0CCB4D0000 cmp byte ptr [004DCB0C], 00
:004D730B 0F85B0000000 jne 004D73C1
:004D7311 C6050CCB4D0001 mov byte ptr [004DCB0C], 01
```



```
:004D7318 33C0                     xor eax, eax   
:004D731A 55                       push ebp   
:004D731B 683E734D00               push 004D733E   
:004D7320 64FF30                   push dword ptr fs:[eax]   
:004D7323 648920                   mov dword ptr fs:[eax], esp   
:004D7326 8B45FC                   mov eax, dword ptr [ebp-04]   
:004D7329 8B80D0050000             mov eax, dword ptr [eax+000005D0]   
:004D732F E8B0C3FDFF               call 004B36E4   
:004D7334 33C0                     xor eax, eax   
:004D7336 5A                       pop edx   
:004D7337 59                       pop ecx   
:004D7338 59                       pop ecx   
:004D7339 648910                   mov dword ptr fs:[eax], edx   
:004D733C EB7C                     jmp 004D73BA   
:004D733E E9B9C1F2FF               jmp 004034FC   
:004D7343 8B45FC                   mov eax, dword ptr [ebp-04]   
:004D7346 8B80D0050000             mov eax, dword ptr [eax+000005D0]   
:004D734C 81B8A040000091010000     cmp dword ptr [eax+000040A0], 00000191   
:004D7356 755D                     jne 004D73B5                 ------->not equ ,jump (c)   
```

\----------------------------------------------------------------------------   

`3, Modify point (1) : jmp xxxxx(exit point)`   

```
0f 85 ec
e9 ed 00
```

`4, 还有下面几个点有校验`   

`4d868c`   

`4d8c2d`   

`4d91ca`   

`4d9769`   

同上   

`4d9cb3`   

`4da1de`   

这两个不太一样   

```
80 3d a4 81 61
e9 e7 00 00 00
```

`5, all clear. 4dd466d0e1e0cc123afbe504da2462ca`
