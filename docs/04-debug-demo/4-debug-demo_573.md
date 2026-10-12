
# smart explorer6.00.17

> 原文：[`www.pediy.com/kssd/pediy05/pediy50125.htm`](https://www.pediy.com/kssd/pediy05/pediy50125.htm)

smart explorer6.00.17

1. 一个极好用的网络浏览器，可惜无针对国人的注册方法，此程序的限制在 30 天试用，注册码不难找，但程序注册后要上网检查，如为非法注册则清除注册码，同时试用期归 0，因此重点在于网上验证部分；那应该破哪呢，动态跟踪不太现实，想起以前的印豪兄对“人体生物节律”的破解思路，就从网上验证失效后出现的两个网页入手，一个关键字为“Evaluation Expired”，另一个的关键字为“expired.html”。

2. 程序用 aspack 压缩，用最新的 AspackDieD 解压，对解压后的文件反汇编，找“Evaluation Expired”,可看到其来自两处调用 `004BE88D` 及 `004CF821`,分别过去看看如何跳过去；可知对如下关键点 `004CF810` 和 `004BE86A` 可跳过对“Evaluation Expired”的调用。

```text
* Referenced by a CALL at Addresses:
:004BE88D , :004CF821     ********* 看看如何跳过去 *********

:004BCE40 53  push ebx
:004BCE41 8BD8  mov ebx, eax
:004BCE43 8D83180B0000  lea eax, dword ptr [ebx+00000B18]
* Possible StringData Ref from Code Obj ->" ( Evaluation Expired ) "
:004BCE49 BAE0CE4B00  mov edx, 004BCEE0
:004BCE4E E84D6FF4FF  call 00403DA0
:004BCE53 33D2  xor edx, edx
:004BCE55 8B83C0090000  mov eax, dword ptr [ebx+000009C0]
:004BCE5B 8B08  mov ecx, dword ptr [eax]
:004BCE5D FF515C  call [ecx+5C]
:004BCE60 33D2  xor edx, edx
:004BCE62 8B8344070000  mov eax, dword ptr [ebx+00000744]
:004BCE68 E81348F7FF  call 00431680

-------------------------------1 ----------------1---------------------------------------------

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004CF7EF(C)
:004CF801 33C0  xor eax, eax
:004CF803 8983240A0000  mov dword ptr [ebx+00000A24], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004CF7FF(U)
:004CF809 83BB240A000000  cmp dword ptr [ebx+00000A24], 00000000
:004CF810 7E0D  jle 004CF81F     *** 这里可跳过 004CF821 处的调用,nop 掉 ***
:004CF812 C683210A000001  mov byte ptr [ebx+00000A21], 01
:004CF819 C645DB01  mov [ebp-25], 01
:004CF81D EB07  jmp 004CF826

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004CF810(C)
:004CF81F 8BC3  mov eax, ebx
:004CF821 E81AD6FEFF  call 004BCE40

-------------------------------2 ------------------ 2------------------------------------------

* Possible StringData Ref from Code Obj ->"http://www.digitalcandle.com/php-bin/rc.php"
:004BE85B 8B1568CB4D00  mov edx, dword ptr [004DCB68]
:004BE861 8BC3  mov eax, ebx
:004BE863 E8C8FBFFFF  call 004BE430
:004BE868 84C0  test al, al
:004BE86A 7540  jne 004BE8AC     ***这里可跳过 004BE88D 处的调用,让它 JMP***
:004BE86C C683280A000000  mov byte ptr [ebx+00000A28], 00

* Possible StringData Ref from Code Obj ->"UserName"
:004BE873 BAE4E84B00  mov edx, 004BE8E4
:004BE878 8BC6  mov eax, esi
:004BE87A E8154CF9FF  call 00453494

* Possible StringData Ref from Code Obj ->"SerialNo"
:004BE87F BAF8E84B00  mov edx, 004BE8F8
:004BE884 8BC6  mov eax, esi
:004BE886 E8094CF9FF  call 00453494
:004BE88B 8BC3  mov eax, ebx
:004BE88D E8AEE5FFFF  call 004BCE40
```

3. 接下来查找“expired.html”，发现有如下 4 处调用，向上看看如何跳过它。代码如下。

```text
****************************************
Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004C6D40(C)
:004C6D68 8BC6  mov eax, esi
:004C6D6A E8957BFFFF  call 004BE904
:004C6D6F 84C0  test al, al     *****此处改为 b001
:004C6D71 7438  je 004C6DAB
:004C6D73 33D2  xor edx, edx
:004C6D75 8BC6  mov eax, esi
:004C6D77 E8604AF8FF  call 0044B7DC
:004C6D7C 8B80D0020000  mov eax, dword ptr [eax+000002D0]
:004C6D82 50  push eax
:004C6D83 8B96640A0000  mov edx, dword ptr [esi+00000A64]
:004C6D89 8D45E4  lea eax, dword ptr [ebp-1C]
* Possible StringData Ref from Code Obj ->"/expired.html"
:004C6D8C 8B0D60CB4D00  mov ecx, dword ptr [004DCB60]

****************************************

:004BD259 80BB280A000000  cmp byte ptr [ebx+00000A28], 00
:004BD260 7407  je 004BD269
:004BD262 8BC3  mov eax, ebx
:004BD264 E857150000  call 004BE7C0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004BD282(C)
:004BD296 8BC3  mov eax, ebx
:004BD298 E8EF200000  call 004BF38C
:004BD29D 8BF0  mov esi, eax
:004BD29F 85F6  test esi, esi
:004BD2A1 7441  je 004BD2E4
:004BD2A3 8BC3  mov eax, ebx
:004BD2A5 E85A160000  call 004BE904
:004BD2AA 84C0  test al, al     *****此处改为 b001
:004BD2AC 742F  je 004BD2DD
:004BD2AE 8D45F8  lea eax, dword ptr [ebp-08]
* Possible StringData Ref from Code Obj ->"/expired.html"

****************************************

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004BD06A(C)
:004BD0AD 8BC3  mov eax, ebx
:004BD0AF E850180000  call 004BE904
:004BD0B4 84C0  test al, al     *****此处改为 b001
:004BD0B6 7445  je 004BD0FD
:004BD0B8 A104EA4D00  mov eax, dword ptr [004DEA04]
:004BD0BD E89AE6F8FF  call 0044B75C
:004BD0C2 8B80D0020000  mov eax, dword ptr [eax+000002D0]
:004BD0C8 50  push eax
:004BD0C9 8D85E8FEFFFF  lea eax, dword ptr [ebp+FFFFFEE8]
* Possible StringData Ref from Code Obj ->"/expired.html"

****************************************

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004BCDA5(C)
:004BCDBB 8BC3  mov eax, ebx
:004BCDBD E8421B0000  call 004BE904
:004BCDC2 84C0  test al, al       *****此处改为 b001
:004BCDC4 744E  je 004BCE14
:004BCDC6 8BC3  mov eax, ebx
:004BCDC8 E88FE9F8FF  call 0044B75C
:004BCDCD 8B80D0020000  mov eax, dword ptr [eax+000002D0]
:004BCDD3 50  push eax
:004BCDD4 8D45F8  lea eax, dword ptr [ebp-08]
* Possible StringData Ref from Code Obj ->"/expired.html"
:004BCDD7 8B0D60CB4D00  mov ecx, dword ptr [004DCB60]
:004BCDDD 8B93640A0000  mov edx, dword ptr [ebx+00000A64]
:004BCDE3 E83072F4FF  call 00404018
:004BCDE8 8B55F8  mov edx, dword ptr [ebp-08]
```



## 汇编代码段

```
:004BCDEB 8D45FC          lea eax, dword ptr [ebp-04]
:004BCDEE E88577F4FF      call 00404578
:004BCDF3 8B55FC          mov edx, dword ptr [ebp-04]
:004BCDF6 58              pop eax
:004BCDF7 E85821FCFF      call 0047EF54
:004BCDFC EB16            jmp 004BCE14
```

smart explorer6.00.17

## 破解说明

1. 本次破解的目的是针对那些手头上没有注册码的朋友能正常使用该软件，在破掉所有的限制的同时程序界面变的和注册版差不多。
2. 和前者一样用 `AspackDied` 解压后的 `DASM` 进行反汇编，不过此次是查找 `"Evaluation Days Remaining"`。

------------------------------------------------------------------------------------------------------------------------

### Possible StringData Ref from Code Obj -> `"UserName"`

```
:004A43B2 A1B0CF4D00      mov eax, dword ptr [004DCFB0]
:004A43B7 8B00            mov eax, dword ptr [eax]
:004A43B9 80B8280A000000  cmp byte ptr [eax+00000A28], 00  ; 看看是哪里将其附值
:004A43C0 7522            jne 004A43E4      ; 这里可跳过
:004A43C2 8D45E8          lea eax, dword ptr [ebp-18]
```

*Possible StringData Ref from Code Obj -> `"This software is un-licensed."`*

|

```
:004A43C5 BACC494A00      mov edx, 004A49CC
:004A43CA E815FAF5FF      call 00403DE4
:004A43CF 8B45FC          mov eax, dword ptr [ebp-04]
:004A43D2 8B80E0020000    mov eax, dword ptr [eax+000002E0]
```

*Possible StringData Ref from Code Obj -> `"Evaluation Days Remaining:"`*

找到这里，向上看看哪里可跳过

|

```
:004A43D8 BAF4494A00      mov edx, 004A49F4
:004A43DD E85A4DF9FF      call 0043913C
:004A43E2 EB0D            jmp 004A43F1
```

------------------------------------------------------------------------------------------------------------------------

*Referenced by a (U)nconditional or (C)onditional Jump at Address:*

|:004CB601(C)

|

```
:004CB677 648920          mov dword ptr fs:[eax], esp
:004CB67A 8B45FC          mov eax, dword ptr [ebp-04]
:004CB67D C680A509000000  mov byte ptr [eax+000009A5], 00  ; -->01
:004CB684 8B45FC          mov eax, dword ptr [ebp-04]
:004CB687 C680B109000000  mov byte ptr [eax+000009B1], 00  ; -->01
:004CB68E 8B45FC          mov eax, dword ptr [ebp-04]
:004CB691 C680280A000000  mov byte ptr [eax+00000A28], 00  ; -->01  找到这里，其值被附为 0
```

发现大本营了，将所有的附值语句取反。

```
:004CB698 8B45FC          mov eax, dword ptr [ebp-04]
:004CB69B C680210A000001  mov byte ptr [eax+00000A21], 01  ; -->00
:004CB6A2 8B45FC          mov eax, dword ptr [ebp-04]
:004CB6A5 C680200A000000  mov byte ptr [eax+00000A20], 00  ; -->01
:004A4655 80B8280A000000  cmp byte ptr [eax+00000A28], 00
:004A465C 0F84A7000000    je 004A4709
:004A4662 B201            mov dl, 01
:004A4664 A1F02F4500      mov eax, dword ptr [00452FF0]
:004A4669 E882EAFAFF      call 004530F0
:004A466E 8BD8            mov ebx, eax
:004A4670 BA01000080      mov edx, 80000001
:004A4675 8BC3            mov eax, ebx
:004A4677 E814EBFAFF      call 00453190
:004A467C A1B0CF4D00      mov eax, dword ptr [004DCFB0]
:004A4681 8B00            mov eax, dword ptr [eax]
:004A4683 8B90DC090000    mov edx, dword ptr [eax+000009DC]
:004A4689 B101            mov cl, 01
:004A468B 8BC3            mov eax, ebx
:004A468D E866EBFAFF      call 004531F8
:004A4692 FF75EC          push [ebp-14]
:004A4695 68984A4A00      push 004A4A98
:004A469A 8D8DE4FDFFFF    lea ecx, dword ptr [ebp+FFFFFDE4]
```

重新启动后，一切清净了。为了在 `ABOUT` 中出现自已的名字，可将下列文件保存为 `sm.reg` 并导入注册表（将“你的名字”改为你自已的），一切 OK！

由于并未真正注册（`ABOUT` 菜单中仍有输入注册码的一项），所以不会有网络验证，同时外表变的跟注册版差不多，功能跟注册版一样，可谓不注册时较完美的破解。（注：这样破解后请误用真正的注册信息注册）

------------------------------------------------------------------------------------------------------------------------

```reg
REGEDIT4

[HKEY_CURRENT_USER\Software\Smarteque\Smart Explorer]

"UserName"="你的名字"
```

------------------------------------------------------------------------------------------------------------------------

`4dd466d0e1e0cc123afbe504da2462ca`
