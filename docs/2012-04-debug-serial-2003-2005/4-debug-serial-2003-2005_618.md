
# Easy Desktop Keeper 1.5

> 原文：[`www.pediy.com/kssd/pediy06/pediy6810.htm`](https://www.pediy.com/kssd/pediy06/pediy6810.htm)

【破解作者】 `cracklover`

【作者邮箱】 `cracklover@126.com`

【使用工具】 `W32dasm` `ds2.7`

【破解平台】 `Win9x/NT/2000/XP`

【软件名称】 `Easy Desktop Keeper`

【下载地址】 [`www.softheap.com/desksaver.html`](http://www.softheap.com/desksaver.html) 或 [`www.softheap.com/download/desksaver.zip`](http://www.softheap.com/download/desksaver.zip)

【软件简介】 此软件是英文的，兄弟也看不太懂，意思好象是管理桌面的，几乎囊括了桌面的所有功能设置，并且能按你老大的意思，保存几种不同的桌面！

【软件大小】 `1,161K`

【加壳方式】 无壳

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 破解内容

根据注册错误信息提示，很容易找到如下代码：

```
:0049026C 6A00              push 00000000
:0049026E 6A00              push 00000000
:00490270 49                dec ecx
:00490271 75F9              jne 0049026C
:00490273 51                push ecx
:00490274 53                push ebx
:00490275 56                push esi
:00490276 8BF0              mov esi, eax
:00490278 33C0              xor eax, eax
:0049027A 55                push ebp
:0049027B 68E9034900        push 004903E9
:00490280 64FF30            push dword ptr fs:[eax]
:00490283 648920            mov dword ptr fs:[eax], esp
:00490286 8D55F4            lea edx, dword ptr [ebp-0C]
:00490289 8B8644030000      mov eax, dword ptr [esi+00000344]
:0049028F E8FC4DFCFF        call 00455090
:00490294 8B45F4            mov eax, dword ptr [ebp-0C]
:00490297 8D55F8            lea edx, dword ptr [ebp-08]
:0049029A E8B5CDFEFF        call 0047D054
:0049029F 8B55F8            mov edx, dword ptr [ebp-08]
:004902A2 B824914900        mov eax, 00499124
:004902A7 E8303FF7FF        call 004041DC
:004902AC E8DBFDFFFF        call 0049008C
{
  :0049008C 53               push ebx
  :0049008D 56               push esi
  :0049008E 57               push edi
  :0049008F BF24914900      mov edi, 00499124
  :00490094 33F6             xor esi, esi          //ESI 清零！下面用到！
  :00490096 33DB             xor ebx, ebx
  :00490098 8B07             mov eax, dword ptr [edi]  //EAX 指向注册码！
  :0049009A E8A943F7FF       call 00404448       //取注册码长度
  * Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0049002E(C)
  |
  :0049009F 83F80E          cmp eax, 0000000E     //长度是否为 14，否则 OVER！
  * Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0049002C(C)
  |
  :004900A2 7567            jne 0049010B
  :004900A4 8B07             mov eax, dword ptr [edi]    //EAX 指向注册码！
  :004900A6 803833           cmp byte ptr [eax], 33      //第一位是否为 3
  :004900A9 0F94C0           sete al                   //上面相等则设置 AL 为 1
  :004900AC 83E07F           and eax, 0000007F         //7F 也就是二进制的 1111111
  :004900AF 03F0             add esi, eax              //将得到的结果加到 ESI，表示一个条件满足
  :004900B1 8B07             mov eax, dword ptr [edi]
  :004900B3 80780233         cmp byte ptr [eax+02], 33   //第三位是否为 3
  :004900B7 0F94C0           sete al
  :004900BA 83E07F           and eax, 0000007F
  :004900BD 03F0             add esi, eax
  :004900BF 8B07             mov eax, dword ptr [edi]
  :004900C1 80780339         cmp byte ptr [eax+03], 39   //第四位是否为 9
  :004900C5 0F94C0           sete al
  :004900C8 83E07F           and eax, 0000007F
  :004900CB 03F0             add esi, eax
  :004900CD 8B07             mov eax, dword ptr [edi]
  :004900CF 80780430         cmp byte ptr [eax+04], 30   //第五位是否为 0
  :004900D3 0F94C0           sete al
  :004900D6 83E07F           and eax, 0000007F
  :004900D9 03F0             add esi, eax
  :004900DB 8B07             mov eax, dword ptr [edi]
  :004900DD 80780738         cmp byte ptr [eax+07], 38   //第八位是否为 8
  :004900E1 0F94C0           sete al
  :004900E4 83E07F           and eax, 0000007F
  :004900E7 03F0             add esi, eax
  :004900E9 8B07             mov eax, dword ptr [edi]
  :004900EB 80780838         cmp byte ptr [eax+08], 38   //第九位是否为 8
  :004900EF 0F94C0           sete al
  :004900F2 83E07F           and eax, 0000007F
  :004900F5 03F0             add esi, eax
  :004900F7 8B07             mov eax, dword ptr [edi]
  :004900F9 80780A32         cmp byte ptr [eax+0A], 32   //第十一位是否为 2
  :004900FD 0F94C0           sete al
  :00490100 83E07F           and eax, 0000007F
  :00490103 03F0             add esi, eax
  :00490105 83FE07           cmp esi, 00000007  //以上 7 个条件全满足的话，ESI=7
  :00490108 0F94C3           sete bl            //ESI=7 则设置 BL 为 1，则注册成功！
  * Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:004900A2(C)
  |
  :0049010B 8BC3             mov eax, ebx     //将 EBX 传入 EAX，准备返回！
  :0049010D 5F               pop edi
  :0049010E 5E               pop esi
  :0049010F 5B               pop ebx
  :00490110 C3               ret
}
:004902B1 8BD8              mov ebx, eax
:004902B3 84DB              test bl, bl
:004902B5 0F84DC000000      je 00490397
```

此处跳往出错框！`BL` 为比较旗！那上面的两个 `CALL` 肯定有问题，得追进看看，

经分析，第二个 `CALL` 有问题，将其内容列到上面。

下面的几个字符串把我着实吓了一跳，以为又是什么 `RSA` 防破解的呢，后来一分析，

此处的跳转若不跳的话，后面程序怎么执行都会经过注册成功的地方，所以往下就不用分析了！

```
:004902BB C6866403000001    mov byte ptr [esi+00000364], 01
:004902C2 8D45FC            lea eax, dword ptr [ebp-04]
:004902C5 50                push eax
:004902C6 8D55F0            lea edx, dword ptr [ebp-10]
* Possible StringData Ref from Code Obj ->"AC95829F829D9994ABAABF8A8BBF82838982A895"
                    |
:004902C9 B800044900        mov eax, 00490400
:004902CE E89D44FFFF        call 00484770
:004902D3 8B45F0            mov eax, dword ptr [ebp-10]
:004902D6 50                push eax
:004902D7 8D55EC            lea edx, dword ptr [ebp-14]
* Possible StringData Ref from Code Obj ->"BE828B999A8C9F88B1A0848E9F829E828B99B1A9BFA0BF"
                    ->"BEB5"
                    |
:004902DA B834044900        mov eax, 00490434
:004902DF E88C44FFFF        call 00484770
:004902E4 8B55EC            mov edx, dword ptr [ebp-14]
:004902E7 A12C914900        mov eax, dword ptr [0049912C]
:004902EC 59                pop ecx
:004902ED E8865AFFFF        call 00485D78
:004902F2 8D55E8            lea edx, dword ptr [ebp-18]
:004902F5 A124914900        mov eax, dword ptr [00499124]
:004902FA E8D543FFFF        call 004846D4
:004902FF 8B45E8            mov eax, dword ptr [ebp-18]
:00490302 50                push eax
:00490303 8D55E4            lea edx, dword ptr [ebp-1C]
* Possible StringData Ref from Code Obj ->"BE9E95A99F95AFAFBC8E95BEBEBF"
                    |
:00490306 B870044900        mov eax, 00490470
:0049030B E86044FFFF        call 00484770
:00490310 8B45E4            mov eax, dword ptr [ebp-1C]
:00490313 50                push eax
:00490314 8D45E0            lea eax, dword ptr [ebp-20]
:00490317 50                push eax
* Possible StringData Ref from Code Obj ->"BE828B999A8C9F88B1A0848E9F829E828B99B1A9BFA0BF"
```



```
             ->"BEB5"
             |
:00490318 B834044900           mov eax, 00490434
:0049031D 5A                 pop edx
:0049031E E84D44FFFF           call 00484770
:00490323 8B55E0               mov edx, dword ptr [ebp-20]
:00490326 A12C914900           mov eax, dword ptr [0049912C]
:0049032B 59                 pop ecx
:0049032C E8EB5AFFFF           call 00485E1C
:00490331 837DFC00             cmp dword ptr [ebp-04], 00000000
:00490335 7546               jne 0049037D
:00490337 E808A4F7FF           call 0040A744
:0049033C 83C4F4             add esp, FFFFFFF4
:0049033F DB3C24             fstp tbyte ptr [esp]
:00490342 9B                 wait
:00490343 8D45DC             lea eax, dword ptr [ebp-24]
:00490346 E8B59EF7FF           call 0040A200
:0049034B 8B45DC             mov eax, dword ptr [ebp-24]
:0049034E 50                 push eax
:0049034F 8D55D8             lea edx, dword ptr [ebp-28]
* Possible StringData Ref from Code Obj ->"AC95829F829D9994ABAABF8A8BBF82838982A895"
             |
:00490352 B800044900           mov eax, 00490400
:00490357 E81444FFFF           call 00484770
:0049035C 8B45D8             mov eax, dword ptr [ebp-28]
:0049035F 50                 push eax
:00490360 8D45D4             lea eax, dword ptr [ebp-2C]
:00490363 50                 push eax
* Possible StringData Ref from Code Obj ->"BE828B999A8C9F88B1A0848E9F829E828B99B1A9BFA0BF"
             ->"BEB5"
             |
:00490364 B834044900           mov eax, 00490434
:00490369 5A                 pop edx
:0049036A E80144FFFF           call 00484770
:0049036F 8B55D4             mov edx, dword ptr [ebp-2C]
:00490372 A12C914900           mov eax, dword ptr [0049912C]
:00490377 59                 pop ecx
:00490378 E89F5AFFFF           call 00485E1C
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00490335(C)
|
:0049037D 6A40               push 00000040
* Possible StringData Ref from Code Obj ->"Information"
             |
:0049037F B990044900           mov ecx, 00490490
* Possible StringData Ref from Code Obj ->"Registration has been completed "
             ->"successfully!"
             |
:00490384 BA9C044900           mov edx, 0049049C
:00490389 A1F8744900           mov eax, dword ptr [004974F8]
:0049038E 8B00               mov eax, dword ptr [eax]
:00490390 E81B55FEFF           call 004758B0
:00490395 EB22               jmp 004903B9
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004902B5(C)
|
:00490397 B824914900           mov eax, 00499124
:0049039C E8E73DF7FF           call 00404188
:004903A1 6A10               push 00000010
* Possible StringData Ref from Code Obj ->"Error"       //错误信息框出现！
             |
:004903A3 B9CC044900           mov ecx, 004904CC
* Possible StringData Ref from Code Obj ->"Registration code is invalid!"
             |
:004903A8 BAD4044900           mov edx, 004904D4
:004903AD A1F8744900           mov eax, dword ptr [004974F8]
:004903B2 8B00               mov eax, dword ptr [eax]
:004903B4 E8F754FEFF           call 004758B0
--------------------------------------------------------------------------------
```

## 破解总结

注册码必须为 14 位，其中：

并且第一、三、四、五位分别必须为 3，3，9，0；

第八、九、十一位分别必须为 8，8，2；其余的随便，补齐 14 位就可！

即：`3X3390X88X2XXX`，`X` 可为任一字母或数字。

这应该是我等菜鸟的学习的经典破解软件，注册码比对简单，流程清晰。

## 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
