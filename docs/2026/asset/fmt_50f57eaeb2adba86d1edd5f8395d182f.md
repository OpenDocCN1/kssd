
``` 
00477C30  00 00 00 00 00 01 00 00  .......
00477C38  00 00 00 00 00 10 00 00  .......
00477C40  00 00 00 00 00 00 01 00  .......
00477C48  00 00 00 00 00 00 10 00  .......
00477C50  00 00 00 00 00 00 00 01  .......
00477C58  00 00 00 00 00 00 00 10  .......
----------------------------------------------
```

``` 
:00470B37 E89042F9FF              call
00404DCC　　　<===一个小变换的CALL

:00470B3C 030424                  add
eax, dword ptr [esp]

:00470B3F 13542404                adc
edx, dword ptr [esp+04]

:00470B43 890424                  mov
dword ptr [esp], eax

:00470B46 89542404                mov
dword ptr [esp+04], edx

\* Referenced by a (U)nconditional
or (C)onditional Jump at Address:
|:00470B20(C)
|
:00470B4A 43
   inc ebx
:00470B4B 83FB11                  cmp
ebx, 00000011
:00470B4E 75C3
 jne 00470B13
       \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*内部循环结束\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
:00470B50 4E
   dec esi
:00470B51 85F6
 test esi, esi
:00470B53 75B9
 jne 00470B0E
       \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*外部循环结束\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
:00470B55 8B0424                  mov
eax, dword ptr [esp]
:00470B58 8B542404                mov
edx, dword ptr [esp+04]
:00470B5C 59
   pop ecx
:00470B5D 5A
   pop edx
:00470B5E 5D
   pop ebp
:00470B5F 5F
   pop edi
:00470B60 5E
   pop esi
:00470B61 5B
   pop ebx
:00470B62 C3
   ret
```

.......

.......

### 3、找到第3个注册码校验点。

按照上面的步骤得到1K3RE6789KAH3UTC789RK2R45HCCi01234oU看起来还真有点像，但很快你会发现，每次退出软件后，这个注册信息就会被删除。呵呵，说明还有问题，想想正确的注册信息总不会这样吧。如何来找到它呢？因为是在退出时删除，所以肯定要对注册表进行操作吧，所用字符串参考找到"NOBODY"（每次退出时就被改成这个），有两个地方，干脆用`trw2000`在这两个地方都设上断点。然后退出HTMLock，很快就会断在00471D97，上下看看，有门了：

.......

.......

--------------------------
``` 
:00471D54 53
   push ebx
:00471D55 DD05907C4700            fld qword
ptr [00477C90]
:00471D5B D81DCC1D4700            fcomp dword
ptr [00471DCC]
:00471D61 DFE0
 fstsw ax
:00471D63 9E
   sahf
:00471D64 7661
 jbe 00471DC7
:00471D66 E8D5FFFFFF              call
00471D40  <===这里一个关键CALL，F8跟进
:00471D6B 84C0
 test al, al  <===要想正确注册，AL必须为1
:00471D6D 7558
 jne 00471DC7 <===这里必须跳走
:00471D6F B201
 mov dl, 01
\* Referenced by a (U)nconditional
or (C)onditional Jump at Address:
|:00471D13(C)
|
:00471D71 A180D64200              mov eax,
dword ptr [0042D680]
:00471D76 E805BAFBFF              call
0042D780
:00471D7B 8BD8
 mov ebx, eax
:00471D7D BA02000080              mov edx,
80000002
:00471D82 8BC3
 mov eax, ebx
:00471D84 E897BAFBFF              call
0042D820
:00471D89 B101
 mov cl, 01
\* Referenced by a (U)nconditional
or (C)onditional Jump at Address:
|:00471D1C(C)            <===呵呵，到这里就意味着注册表的信息要被清空了。向上看
|　　　:00471D1C这里一个跳转，也应该向上看
\* Possible StringData Ref from
Code Obj ->"SOFTWARE\Atrise\HTMLock 1.9.3"
           |
:00471D8B BAD81D4700              mov edx,
00471DD8
:00471D90 8BC3
 mov eax, ebx
:00471D92 E8EDBAFBFF              call
0042D884
\* Possible StringData Ref from
Code Obj ->"NOBODY"
           |
:00471D97 B9001E4700              mov ecx,
00471E00
\* Possible StringData Ref from
Code Obj ->"UserName"
           |
:00471D9C BA101E4700              mov edx,
00471E10
:00471DA1 8BC3
 mov eax, ebx
:00471DA3 E878BCFBFF              call
0042DA20
\* Possible StringData Ref from
Code Obj ->"NOKEY"
           |
:00471DA8 B9241E4700              mov ecx,
00471E24
\* Possible StringData Ref from
Code Obj ->"RegKey"
           |
:00471DAD BA341E4700              mov edx,
00471E34
:00471DB2 8BC3
 mov eax, ebx
:00471DB4 E867BCFBFF              call
0042DA20
:00471DB9 8BC3
 mov eax, ebx
:00471DBB E830BAFBFF              call
0042D7F0
:00471DC0 8BC3
 mov eax, ebx
:00471DC2 E8E114F9FF              call
004032A8
\* Referenced by a (U)nconditional
or (C)onditional Jump at Addresses:
|:00471D64(C), :00471D6D(C)
|
:00471DC7 5B
   pop ebx
:00471DC8 C3
   ret

---------:00471D66 call 00471D40  <===这里一个关键CALL，F8跟进----------

:00471D40 8B15787C4700            mov edx, dword
ptr [00477C78]  <===这里就是我们的注册名newlaos
:00471D46 A17C7C4700              mov eax,
dword ptr [00477C7C]  <===注册码1K3RE6789KAH3UTC789RK2R45HCCi01234oU
:00471D4B E8ECF3FFFF              call
0047113C   <===很明显，F8再跟进
:00471D50 C3
   ret

-----------------------------------------------------------

:0047113C 8078062D                cmp
byte ptr [eax+06], 2D  <===呵呵，注册码的第7位应为-，否则就认为是这个暗桩好深呀
:00471140 0F94C0                  sete
al
:00471143 C3
   ret
```

最后，真正的注册码就是1K3RE6-89KAH3UTC789RK2R45HCCi01234oH(注：最后两位也有所变化)

### 三、无奈的结局

原以为可以收工了，但发现生成的加密的文件不正确，就是说更本没有加密。还是在生成加密网页的地方，用`BPX HMEMCPY`设断点，绕过上面第2个检测点，我又发现了第4个检测点。这也是最让我无奈的地方，因为如果要通过第2个检测点，就无法通过这个检测点。请看：

.......

.......

``` 
:00472267 B930750000              mov ecx,
00007530
\* Possible StringData Ref from
Code Obj ->"//Page Begin"
           |
:0047226C BA6C264700              mov edx,
0047266C
:00472271 8B45F0                  mov
eax, dword ptr [ebp-10]
:00472274 E8BBE5FFFF              call
00470834
:00472279 84C0
 test al, al
:0047227B 754A
 jne 004722C7  <===这里跳走，不跳走，就报告临时文件生成错误
:0047227D 8B45F4                  mov
eax, dword ptr [ebp-0C]
.......

.......

:004722E6 E849E5FFFF              call
00470834
:004722EB 84C0
 test al, al
:004722ED 754A
 jne 00472339  <===这里跳走，不跳走，就报告临时文件生成错误
:004722EF 8B45F4                  mov
eax, dword ptr [ebp-0C]
.......

.......

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004722ED(C)
|
:00472339 8B45F0                  mov
eax, dword ptr [ebp-10]
:0047233C 8B8014080000            mov eax, dword
ptr [eax+00000814]
:00472342 8945D8                  mov
dword ptr [ebp-28], eax
:00472345 8D45EC                  lea
eax, dword ptr [ebp-14]
:00472348 E8AB1CF9FF              call
00403FF8
:0047234D 33DB
 xor ebx, ebx
:0047234F 8B450C                  mov
eax, dword ptr [ebp+0C]  <===我们输入的78787878(网页文件解锁密码)
:00472352 E8611FF9FF              call
004042B8
:00472357 8BF0
 mov esi, eax
:00472359 85F6
 test esi, esi
:0047235B 7E10
 jle 0047236D
```

.......(网页加密的代码段，有兴趣的朋友可以研究一下)

``` 
\* Possible StringData Ref from Code Obj ->"4096"
           |
:004725B1 BA08274700              mov edx,
00472708
:004725B6 A17C7C4700              mov eax,
dword ptr [00477C7C]
:004725BB E86CE6FFFF              call
00470C2C   <===呵呵，第4个检测点CALL，F8跟进
:004725C0 84C0
 test al, al     <===AL必须为1
:004725C2 7516
 jne 004725DA    <===这里必须跳走
:004725C4 8B45F4                  mov
eax, dword ptr [ebp-0C]
:004725C7 E8DC0CF9FF              call
004032A8
:004725CC 8B45F0                  mov
eax, dword ptr [ebp-10]
:004725CF E8D40CF9FF              call
004032A8
:004725D4 C645FB01                mov
[ebp-05], 01
:004725D8 EB25
 jmp 004725FF
\* Referenced by a (U)nconditional
or (C)onditional Jump at Addresses:
|:004725AF(C), :004725C2(C)
|
:004725DA 8B4DD8                  mov
ecx, dword ptr [ebp-28]
.......

.......

-----------004725BB call 00470C2C   <===呵呵，第4个注册码检测点，F8跟进-----------------

:00470C2C 55
   push ebp
:00470C2D 8BEC
 mov ebp, esp
:00470C2F 83C4E4                  add
esp, FFFFFFE4
:00470C32 53
   push ebx
:00470C33 33C9
 xor ecx, ecx
:00470C35 894DFC                  mov
dword ptr [ebp-04], ecx
:00470C38 894DE4                  mov
dword ptr [ebp-1C], ecx
:00470C3B 8BD8
 mov ebx, eax       <===1K3RE6-89KAH3UTC789RK2R45HCCi01234oH
:00470C3D 33C0
 xor eax, eax
:00470C3F 55
   push ebp
:00470C40 683C0E4700              push
00470E3C
:00470C45 64FF30                  push
dword ptr fs:[eax]
:00470C48 648920                  mov
dword ptr fs:[eax], esp
:00470C4B 8D45E4                  lea
eax, dword ptr [ebp-1C]
:00470C4E 8BD3
 mov edx, ebx
:00470C50 E83B34F9FF              call
00404090
:00470C55 8D45E4                  lea
eax, dword ptr [ebp-1C]
:00470C58 BA1E000000              mov edx,
0000001E
:00470C5D E8E239F9FF              call
00404644
:00470C62 E8A9FCFFFF              call
00470910
:00470C67 8D45FC                  lea
```
