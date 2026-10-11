
# 木马克星

> 原文：[`www.pediy.com/kssd/pediy06/pediy6834.htm`](https://www.pediy.com/kssd/pediy06/pediy6834.htm)

算法以前发过了，再写也没意思，这次主要谈谈如何可以升级。

自上次答应作者不再破解之后，我的 `9.99++` 可升级版就没有再更新了（别以为我和作者有一腿，我向毛主席保证，我是清白的），谁知前两天逛他的网站，他竟在他的论坛说封了我的所有破解版（真是个小人），并且还说他现在的版本加密多么的强，请大家破解，于是我下了个 `0301` 版看了一下，趁着酒性把强奸他的过程写了出来，让高手见笑了。

`0301` 版和他以前的版本不同在于对升级地址加了密，他的升级地址是 `http://www.luosoft.com/cgi-bin/test.pl?name=用户名`，如果用户名不是他服务器里的就显示 `NO OK`，软件就会说你不是注册用户。

如果用户名正确，就会得到一个病毒库文件的地址，具他自己讲病毒库地址 30 分钟换一次（真是变态）。

我们现在所能做的就是将他的升级地址换成我们自己的服务器地址，软件通过我们的服务器认证后获得病毒库文件，但是他的升级地址显示的是 `"Fn2yhGnF7PxJGNVN4g6IinGmjFDbkxlXXTpPy0ZkMN6UvUS9Ipls24II"` 只有在软件运行时通过几千行的变态算法后才还原成 `http://www.luosoft.com/cgi-bin/test.pl?name=`，最后存放在下面的 `[edx]` 中。

原文件:

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| `:004B4B79(C)`
|

```
:004B4B38 8A06                  mov al, byte ptr [esi]
:004B4B3A 8845F7                mov byte ptr [ebp-09], al
:004B4B3D 8B4B34                mov ecx, dword ptr [ebx+34]
:004B4B40 8B5330                mov edx, dword ptr [ebx+30]
:004B4B43 8BC3                  mov eax, ebx
:004B4B45 8B38                  mov edi, dword ptr [eax]
:004B4B47 FF5760                call [edi+60]
:004B4B4A 8B4334                mov eax, dword ptr [ebx+34]
:004B4B4D 8A00                  mov al, byte ptr [eax]
:004B4B4F 3206                  xor al, byte ptr [esi]
:004B4B51 8B55F8                mov edx, dword ptr [ebp-08]
:004B4B54 8802                  mov byte ptr [edx], al------通过不知名的几千行变态算法计算后将最终的升级地址放在[edx]中,我们就改了他.
:004B4B56 8B4B38                mov ecx, dword ptr [ebx+38]
:004B4B59 49                    dec ecx
:004B4B5A 8B5330                mov edx, dword ptr [ebx+30]
:004B4B5D 8B4330                mov eax, dword ptr [ebx+30]
:004B4B60 40                    inc eax
:004B4B61 E862DEF4FF            call 004029C8
:004B4B66 8B4330                mov eax, dword ptr [ebx+30]
:004B4B69 034338                add eax, dword ptr [ebx+38]
:004B4B6C 48                    dec eax
:004B4B6D 8A55F7                mov dl, byte ptr [ebp-09]
:004B4B70 8810                  mov byte ptr [eax], dl
:004B4B72 46                    inc esi
:004B4B73 FF45F8                inc [ebp-08]
:004B4B76 FF4DF0                dec [ebp-10]
:004B4B79 75BD                  jne 004B4B38
:004B4B7B 5F                    pop edi
:004B4B7C 5E                    pop esi
:004B4B7D 5B                    pop ebx
:004B4B7E 8BE5                  mov esp, ebp
:004B4B80 5D                    pop ebp
:004B4B81 C20400                ret 0004
```

修改后的代码:

```
:004B4B0A 807B2400              cmp byte ptr [ebx+24], 00
:004B4B0E 7516                  jne 004B4B26
* Possible StringData Ref from Code Obj ->"Cipher not initialized"
                                 |
:004B4B10 B98C4B4B00            mov ecx, 004B4B8C
:004B4B15 B201                  mov dl, 01
* Possible StringData Ref from Code Obj ->"EDCP_blockcipher 岪"
                                 |
:004B4B17 A1C4404B00            mov eax, dword ptr [004B40C4]
:004B4B1C E83780F5FF            call 0040CB58
:004B4B21 E832EEF4FF            call 00403958
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004B4B0E(C)
|
:004B4B26 B868747470            mov eax, 70747468
:004B4B2B 8902                  mov dword ptr [edx], eax
:004B4B2D B83A2F2F66            mov eax, 662F2F3A
:004B4B32 894204                mov dword ptr [edx+04], eax
:004B4B35 B87265652E            mov eax, 2E656572
:004B4B3A 894208                mov dword ptr [edx+08], eax
:004B4B3D B868626973            mov eax, 73696268
:004B4B42 89420C                mov dword ptr [edx+0C], eax
:004B4B45 B8702E636F            mov eax, 6F632E70
:004B4B4A 894210                mov dword ptr [edx+10], eax
:004B4B4D B86D2F6C65            mov eax, 656C2F6D
:004B4B52 894214                mov dword ptr [edx+14], eax
:004B4B55 B86F7A656D            mov eax, 6D657A6F
:004B4B5A 894218                mov dword ptr [edx+18], eax
:004B4B5D B82F736865            mov eax, 6568732F
:004B4B62 89421C                mov dword ptr [edx+1C], eax
:004B4B65 B86E676A69            mov eax, 696A676E
:004B4B6A 894220                mov dword ptr [edx+20], eax
:004B4B6D B82E617370            mov eax, 7073612E
:004B4B72 894224                mov dword ptr [edx+24], eax
:004B4B75 B83F6D7A3D            mov eax, 3D7A6D3F
:004B4B7A 894228                mov dword ptr [edx+28], eax
:004B4B7D 5B                    pop ebx
:004B4B7E 8BE5                  mov esp, ebp
:004B4B80 5D                    pop ebp
:004B4B81 C20400                ret 0004
```

上面这段代码就是将我的升级地址 `http://free.hbisp.com/leozem/shengji.asp?mz=` 替换进去，`dword ptr [edx]` 也就是他最终的升级地址所存放的位置。

接下来就是讲如何做升级服务器了，打开记事本，打入下列代码:

然后保存为 `shengji.asp` 放到你的服务器上，他的病毒库文件有很多方法可以获得，在这就不讲了。

```
:005434B8 8B83D8020000          mov eax, dword ptr [ebx+000002D8]
:005434BE 8B8010010100          mov eax, dword ptr [eax+00010110]
* Possible StringData Ref from Code Obj ->"no ok"
                                 |
:005434C4 BA80365400            mov edx, 00543680
:005434C9 E8060EECFF            call 004042D4---是否显示"NO OK"
:005434CE 753D                  jne 0054350D-----不是从得到的升级地址下载升级文件.
:005434D0 33D2                  xor edx, edx
:005434D2 A1C8375800            mov eax, dword ptr [005837C8]
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0054348F(C)
|
:005434D7 8B08                  mov ecx, dword ptr [eax]
:005434D9 FF515C                call [ecx+5C]
:005434DC A16CA65600            mov eax, dword ptr [0056A66C]
:005434E1 803800                cmp byte ptr [eax], 00---是否为英文版,是就跳
:005434E4 740C                  je 005434F2
* Possible StringData Ref from Code Obj ->"服务器认证错误!你不是合法用户."
                                 |
:005434E6 B890365400            mov eax, 00543690
:005434EB E89087F1FF            call 0045BC80
:005434F0 EB0A                  jmp 005434FC
```

他以前的版本，在杀毒时需要连接网络，但到了这版却不需要，可能是他顾着对付我，整天只进行加密的研究，疏忽了这点，但软件的代码中还保存有杀毒时的服务器认证，NND 这版连我的大名也进了他的软件，搞什么鬼，我可不怕你，我的硬盘保修期还没过，呵呵。

* Possible StringData Ref from Code Obj ->"loezem"
|

```
:0055E89A B874EE5500            mov eax, 0055EE74
:0055E89F E80C5CEAFF            call 004044B0---启动判断软件中是否含有 loezem,估计是怕我改他的软件,但是可气的是竟把我的 leozem 写成 loezem,晕.如果软件中含有 loezem 只是报错误,还好,没有格我的盘.
:0055E8A4 85C0                  test eax, eax
:0055E8A6 0F8F25050000          jg 0055EDD1
:0055E8AC 8D55E8                lea edx, dword ptr [ebp-18]
:0055E8AF A11C385800            mov eax, dword ptr [0058381C]
:0055E8B4 E8DB79EDFF            call 00436294
:0055E8B9 8B45E8                mov eax, dword ptr [ebp-18]
:0055E8BC 8D55EC                lea edx, dword ptr [ebp-14]
:0055E8BF E8B8A7EAFF            call 0040907C
:0055E8C4 8B55EC                mov edx, dword ptr [ebp-14]
* Possible StringData Ref from Code Obj ->"loezem"
                                 |
:0055E8C7 B874EE5500            mov eax, 0055EE74
:0055E8CC E8DF5BEAFF            call 004044B0
:0055E8D1 85C0                  test eax, eax
:0055E8D3 0F8FF8040000          jg 0055EDD1
:0055E8D9 8D55E0                lea edx, dword ptr [ebp-20]
:0055E8DC A11C385800            mov eax, dword ptr [0058381C]
:0055E8E1 E8AE79EDFF            call 00436294
:0055E8E6 8B45E0                mov eax, dword ptr [ebp-20]
:0055E8E9 8D55E4                lea edx, dword ptr [ebp-1C]
:0055E8EC E88BA7EAFF            call 0040907C
```
