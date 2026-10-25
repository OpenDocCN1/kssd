
# Crypkey v6.5 不用脱壳，内存补丁搞订 sitekey

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-526.htm`](https://www.pediy.com/kssd/pediy09/pediy09-526.htm)

> `Crypkey v6.5` <http://www.crypkey.com>
>
> 下载地址： 见看雪论坛
>
> 软件大小： 9.61 M
>
> 软件简介： `Crypkey` 是一种加密保护的软件，可以分模块，日期，使用次数分别进行授权，`v6.0` 以下的加壳版本很容易脱壳，`6.0` 以上，脱壳较难。
>
> 【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教，转载请保持完成。
>
> 【调试环境】：`WinXP`、`LordPE`、`W32DAsm`、`KeyMaker`（不用调试就可以搞定）
>
> 【声明】：`Crypkey v6` 以上的版本在近 6 年前我接触过，当时由于脱壳水平很有限，于是只有想一些其他的办法。根据 `v5.7` 的经验，很容易就把 `v6.1` 版的 `sitekey` 生成器搞定了。由于水平有限，且一向不喜欢发文，所以一直就放着。前几天看见大家在讨论，所以就把自己的思路写了一下，很简单，高手可以跳过了。
>
> 【破解过程】：
>
> `Crypkey v6.5` 包括两部分，一个是 `cki.exe`，是用来保护程序和压缩加壳的。另一个文件是 `skw.exe`，可以根据 `sitecode` 生成 `sitekey`。`v6` 以后，加壳保护比较厉害，但是 `skw.exe` 的保护有漏洞，`anti` 做的不够好，用 `trw` 或 `softice` 很容易。换种思路，很简单，不用调试。
>
> 运行 `skw.exe`，点上面的 `license`，出现 `sitecode`，这就是 `skw.exe` 需要授权的信息。拷贝 `sitecode`，回到 `verify sitecode`，出现提示：`crypkey is not licensed to make keys for itself`。这就是提示了。或者是添入你要破解的程序显示的 `sitecode`，提示是：`The Site Code you entered is from another company`。搜这个也可以。
>
> 第一步：运行 `skw.exe`，会在临时目录下生成临时文件，和 `skw.exe` 之间相互通信，不管，用 `lordpe` dump。（用 `OD` 你会很惨）
>
> 第二步：反编译 dump 的文件，查找 `make keys for itself`，就可以找到关键的地方。
>
> 第三步：生成内存补丁，搞定！！！快试一试吧，先把 `skw.exe` 的 `sitekey` 找出来吧！！！！！
>
> 注意，要对其他的被保护程序授权，先要在 `skw.ini` 中进行配置，对 `Name`、`company`、`Number`、`Password`、`option` 等进行正确的赋值。
>
> 关键点：
>
> \---------------------------------------------------------
>
> 下面这一段，全部是错误提示，我们关注两个错误：`from another company`，这个错误是提示你不能对当前的程序授权，我们要解掉的地方。`"is not licensed to make keys for itself."`，我们要用 `skw.exe` 先对自己授权才能对别的软件授权。
>
> 看完后就知道了，只要 `call` 到这里就完蛋，无非是不同的错误而已。有 12 处调用，我们就把这 12 处避开就好了。
>
> ```
> * Referenced by a CALL at Addresses:
> |:00406690   , :004066DA   , :00406941   , :00406A31   , :00406ADC
> |:00406B4D   , :00406BC4   , :00406C35   , :004076DB   , :00407A2D
> |:00407A6D   , :0040809C   《－－－－－－12 处调用，回到调用处，查看
> |
>
> :00401508 55                    push ebp
> :00401509 8BEC                  mov ebp, esp
> :0040150B 51                    push ecx
> :0040150C 8B450C                mov eax, dword ptr [ebp+0C]
> :0040150F 83F8FA                cmp eax, FFFFFFFA
> :00401512 7F35                  jg 00401549<-----跳到后面的都是错误的判断
> :00401514 0F8492000000          je 004015AC<-----一样是错误
> :0040151A 83C00D                add eax, 0000000D
> :0040151D 83F806                cmp eax, 00000006
> :00401520 0F87E0000000          ja 00401606<-------unknown erro
> :00401526 FF24852D154000        jmp dword ptr [4*eax+0040152D]<----看看下面，有 7 个地址，实际上 eax 的值会从 1 到 7，下面的这 7 个地址全部是错误提示。
>
>
> :0040152D EB154000              DWORD 004015EB《－－－－无非是不同的错误提示
> :00401531 E2154000              DWORD 004015E2
> :00401535 D9154000              DWORD 004015D9
> :00401539 D0154000              DWORD 004015D0
> :0040153D C7154000              DWORD 004015C7
> :00401541 BE154000              DWORD 004015BE
> :00401545 B5154000              DWORD 004015B5
>
>
> * Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00401512(C)
> |
>
> :00401549 83F8FE                cmp eax, FFFFFFFE
> :0040154C 7F12                  jg 00401560《－－－－这些都是转向不同的错误，crc 错误，length 错误，无授权错误等等。
> :0040154E 7438                  je 00401588
> :00401550 83E8FB                sub eax, FFFFFFFB
> :00401553 744E                  je 004015A3
> :00401555 48                    dec eax
> :00401556 7442                  je 0040159A
> :00401558 48                    dec eax
> :00401559 7436                  je 00401591
> :0040155B E9A6000000            jmp 00401606
> :00401560 83E8FF                sub eax, FFFFFFFF
> :00401563 7417                  je 0040157C
> :00401565 2DE9030000            sub eax, 000003E9
> :0040156A 0F8484000000          je 004015F4
> :00401570 48                    dec eax
> :00401571 0F8486000000          je 004015FD
> :00401577 E98A000000            jmp 00401606
>
>
> * Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00401563(C)
> |
>
> * Possible StringData Ref from Data Obj ->"The Site Code you have entered "
>                         ->"is not the correct length. Make "
>                         ->"sure you have typed all characters "
>                         ->"of the code."
>
> :0040157C C745FCEF854E00        mov [ebp-04], 004E85EF
> :00401583 E985000000            jmp 0040160D
>
>
> * Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:0040154E(C)
> |
>
> * Possible StringData Ref from Data Obj ->"The Site Code you entered is not "
>                         ->"valid (CRC failure). Make sure "
>                         ->"that you have not mis-typed any "
>                         ->"of the characters in the code."
>
> :00401588 C745FC5E864E00        mov [ebp-04], 004E865E
> :0040158F EB7C                  jmp 0040160D
>
>
> * Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00401559(C)
> |
>
> * Possible StringData Ref from Data Obj ->"The Site Code you entered is from "
>                         ->"a newer version of CrypKey. This "
>                         ->"version of the Site Key Generator "
>                         ->"can not generate unlock keys for "
>                         ->"the application."
>
> :00401591 C745FCDD864E00        mov [ebp-04], 004E86DD
> :00401598 EB73                  jmp 0040160D
>
>
> * Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00401556(C)
> |
>
> * Possible StringData Ref from Data Obj ->"The site Code you entered is from "
>                         ->"an older version of CrypKey. This "
>                         ->"version of the Site Key Generator "
>                         ->"can not generate unlock keys for "
>                         ->"the application."
>
> :0040159A C745FC74874E00        mov [ebp-04], 004E8774
> :004015A1 EB6A                  jmp 0040160D
>
>
> * Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00401553(C)
> |
>
> * Possible StringData Ref from Data Obj ->"The Site Code you entered is from "
>                         ->"another company. You can only "
>                         ->"generate unlock keys for your "
>                         ->"company's applications."
>
> :004015A3 C745FC0C884E00        mov [ebp-04], 004E880C
> :004015AA EB61                  jmp 0040160D
>
>
> * Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00401514(C)
> |
>
> * Possible StringData Ref from Data Obj ->"The kill code is the wrong length "
>                         ->"to be a valid kill code."
>
> :004015AC C745FC82884E00        mov [ebp-04], 004E8882
> :004015B3 EB58                  jmp 0040160D
>
>
> * Possible StringData Ref from Data Obj ->"The kill code is not valid (CRC "
>                         ->"failure). Please make sure it "
>                         ->"is entered correctly."
>
> :004015B5 C745FCBD884E00        mov [ebp-04], 004E88BD
> :004015BC EB4F                  jmp 0040160D
> ```



## 错误代码参考与调试分析

> *Possible StringData Ref from Data Obj ->"The kill code is from a newer "*  
> ->"version of CrypKey that is not "*  
> ->"supported in this version of the "*  
> ->"Site Key Generator."  
> |  
> `:004015BE C745FC11894E00          mov [ebp-04], 004E8911`  
> `:004015C5 EB46                    jmp 0040160D`

> *Possible StringData Ref from Data Obj ->"The kill code is from an older "*  
> ->"version of CrypKey that is not "*  
> ->"supported in this version of the "*  
> ->"Site Key Generator."  
> |  
> `:004015C7 C745FC83894E00          mov [ebp-04], 004E8983`  
> `:004015CE EB3D                    jmp 0040160D`

> *Possible StringData Ref from Data Obj ->"The kill code is from a different "*  
> ->"company. You can only check kill "*  
> ->"codes from products that this "*  
> ->"Site Key Generator is authorized "*  
> ->"to generate keys for."  
> |  
> `:004015D0 C745FCF6894E00          mov [ebp-04], 004E89F6`  
> `:004015D7 EB34                    jmp 0040160D`

> *Possible StringData Ref from Data Obj ->"Easy license can not be used for "*  
> ->"this license, probably because "*  
> ->"there is no hard drive information."  
> |  
> `:004015D9 C745FC8E8A4E00          mov [ebp-04], 004E8A8E`  
> `:004015E0 EB2B                    jmp 0040160D`

> *Possible StringData Ref from Data Obj ->"This Site Key Generator is not "*  
> ->"authorized to generate keys for "*  
> ->"this product."  
> |  
> `:004015E2 C745FCF28A4E00          mov [ebp-04], 004E8AF2`  
> `:004015E9 EB22                    jmp 0040160D`

> *Possible StringData Ref from Data Obj ->"There was an encryption error "*  
> ->"with the site key."  
> |  
> `:004015EB C745FC3F8B4E00          mov [ebp-04], 004E8B3F`  
> `:004015F2 EB19                    jmp 0040160D`

> *Referenced by a (U)nconditional or (C)onditional Jump at Address:*  
> `:0040156A(C)`

> *Possible StringData Ref from Data Obj ->"You have entered a Site Code from "*  
> ->"a copy of the CrypKey Site Key "*  
> ->"Generator. This Site Key Generator "*  
> ->"is not licensed to make keys for "*  
> ->"itself."  
> |  
> `:004015F4 C745FC708B4E00          mov [ebp-04], 004E8B70`  
> `:004015FB EB10                    jmp 0040160D`

> *Referenced by a (U)nconditional or (C)onditional Jump at Address:*  
> `:00401571(C)`

> *Possible StringData Ref from Data Obj ->"This Site Key Generator is not "*  
> ->"authorized to generate keys for "*  
> ->"this product."  
> |  
> `:004015FD C745FCFD8B4E00          mov [ebp-04], 004E8BFD`  
> `:00401604 EB07                    jmp 0040160D`

> *Referenced by a (U)nconditional or (C)onditional Jump at Addresses:*  
> `:00401520(C), :0040155B(U), :00401577(U)`

> *Possible StringData Ref from Data Obj ->"There was an unknown Site Key "*  
> ->"Generator error."  
> |  
> `:00401606 C745FC4A8C4E00          mov [ebp-04], 004E8C4A`

> *Referenced by a (U)nconditional or (C)onditional Jump at Addresses:*  
> `:00401583(U), :0040158F(U), :00401598(U), :004015A1(U), :004015AA(U)`  
> `:004015B3(U), :004015BC(U), :004015C5(U), :004015CE(U), :004015D7(U)`  
> `:004015E0(U), :004015E9(U), :004015F2(U), :004015FB(U), :00401604(U)`

```
:0040160D 6A00                    push 00000000
:0040160F FF75FC                  push [ebp-04]
:00401612 68017F0000              push 00007F01
:00401617 6A00                    push 00000000
```

> *Reference To: USER32.LoadIconA, Ord:0000h*  
> `:00401619 E8945E0E00            Call 004E74B2`  
> `:0040161E 50                    push eax`

> *Possible StringData Ref from Data Obj ->"Error"*  
> `:0040161F 68798C4E00            push 004E8C79`  
> `:00401624 FF7508                push [ebp+08]`  
> `:00401627 E8B0F70000            call 00410DDC`  
> `:0040162C 59                    pop ecx`  
> `:0040162D 5D                    pop ebp`  
> `:0040162E C3                    ret`

以上是这个 call 的完整的内容，全部是错误，死亡深渊。我们可以避开对这个 call 的调用就可以了。

`//////////////////////////////////////////////////////////////////////`  
12 个调用不一一说明哪个是什么错误（od 调试不了，当年用的是 trw），举两个例子（避开非授权 from another company 错误），

> *Referenced by a (U)nconditional or (C)onditional Jump at Address:*  
> `:00406AC5(C)`

```
:00406B18 8B4D08                mov ecx, dword ptr [ebp+08]
:00406B1B 83B91004000000        cmp dword ptr [ecx+00000410], 00000000
:00406B22 0F84D8000000          je 00406C00
:00406B28 8B4508                mov eax, dword ptr [ebp+08]
:00406B2B 8B9010040000          mov edx, dword ptr [eax+00000410]
:00406B31 3B1548655000          cmp edx, dword ptr [00506548]
:00406B37 7450                  je 00406B89 <-------这里跳过 check 中出现的 from another company 错误提示，用内存补丁该为 jmp 就好了
:00406B39 FF7508                push [ebp+08]
:00406B3C E837030000            call 00406E78
:00406B41 59                    pop ecx
:00406B42 6AFB                  push FFFFFFFB
:00406B44 8B4508                mov eax, dword ptr [ebp+08]
:00406B47 E84C900800            call 0048FB98
:00406B4C 50                    push eax
:00406B4D E8B6A9FFFF            call 00401508 《－－－－－－这里出现错误提示，call 00401508
:00406B52 83C408                add esp, 00000008
:00406B55 8B5508                mov edx, dword ptr [ebp+08]
:00406B58 8B82E4020000          mov eax, dword ptr [edx+000002E4]
:00406B5E 8B10                  mov edx, dword ptr [eax]
:00406B60 FF92B0000000          call dword ptr [edx+000000B0]
:00406B66 33C0                  xor eax, eax
:00406B68 50                    push eax
:00406B69 FF4DF4                dec [ebp-0C]
:00406B6C 8D45FC                lea eax, dword ptr [ebp-04]
```

`－－－－－－－－－－－－－－－－－－－－－－－`

> *Referenced by a (U)nconditional or (C)onditional Jump at Address:*  
> `:004079B6(C)`

```
:00407A14 8B1548655000          mov edx, dword ptr [00506548]
:00407A1A 3B9578FFFFFF          cmp edx, dword ptr [ebp+FFFFFF78]
:00407A20 7424                  je 00407A46 <-------这里跳过 check 中出现的 from another company，用内存补丁该为 jmp 就好了
:00407A22 6AFB                  push FFFFFFFB
:00407A24 8B45D4                mov eax, dword ptr [ebp-2C]
:00407A27 E86C810800            call 0048FB98
:00407A2C 50                    push eax
:00407A2D E8D69AFFFF            call 00401508 《－－－－－－这里出现错误提示，call 00401508
:00407A32 83C408                add esp, 00000008
:00407A35 8B55D8                mov edx, dword ptr [ebp-28]
```



```
:00407A38 64891500000000    mov dword ptr fs:[00000000], edx

:00407A3F 8BC2              mov eax, edx

:00407A41 E9DE000000        jmp 00407B24
```

上面这两个地方的改动就可以跳过 `from another company` 错误。

12 个地方我就不一一的写了，自己看去吧。

`keymaker` 制作内存补丁就好了，快用 `skw.exe` 自身的授权来试一下吧。

虽然这次没有调试，其实是几年前用 `trw` 调试，很容易发现问题所在。`crypkey` 的失败之处，授权文件生成器的保护太差了，用 `softice` 和 `trw` 很容易调试。

`v5.7` 版的 `skw` 可以直接破解，感谢 `5.7` 版时 `email123` 的提示。

`4dd466d0e1e0cc123afbe504da2462ca`
