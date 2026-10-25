
```
:00419655 E8CE310A00
call 004BC828
:0041965A 8D45F0
lea eax, dword ptr [ebp-10]
:0041965D
E8AA320A00 call 004BC90C
:00419662
8B55C4 mov edx, dword ptr [ebp-3C]
:00419665 8902
mov dword ptr [edx], eax
:00419667 FF4F1C
dec [edi+1C]
:0041966A
8D45F0 lea eax, dword ptr [ebp-10]
:0041966D BA02000000
mov edx, 00000002
:00419672 E839300A00
call 004BC6B0
:00419677 46
inc esi
:00419678 8345C404
add dword ptr [ebp-3C], 00000004
:0041967C 83FE09
cmp esi, 00000009
:0041967F 0F8E32FFFFFF
jle 004195B7 <===循环9次后，这里就不再跳转了!
:00419685
33F6 xor esi, esi <===ESI被清0了。又要循环了。
:00419687
8D95BCFEFFFF lea edx, dword ptr [ebp+FFFFFEBC]
:0041968D 8D8548FFFFFF lea eax, dword ptr [ebp+FFFFFF48]
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004196A8(C)
|
<===这一行到004196A5，又构成一个小循环
:00419693
8B1A mov ebx, dword ptr [edx]
<===这里出现11次的数字是固定的(4，A，6，8，9，2，1，3，5，7，B)，估计是机器码的变形(后来发现没有用)
:00419695 8B08
mov ecx, dword ptr [eax]
<===这里依次就是我们输入的注册码的了(7，8，7，8，7，8，7，8，7，8，7)
:00419697
83C004 add eax, 00000004
:0041969A 83C204
add edx, 00000004
:0041969D 46
inc esi
:0041969E 898C9D14FFFFFF
mov dword ptr [ebp+4*ebx-000000EC], ecx
:004196A5
83FE0A cmp esi, 0000000A <===由于初始值是0，所以这里也要循环11次，
:004196A8 7EE9
jle 00419693
<===循环11次后，不再跳转了。
:004196AA BE01000000
mov esi, 00000001 <===又初始化了，呵呵，又要循环了
:004196AF
8D55A0 lea edx, dword ptr [ebp-60]
:004196B2 8D8518FFFFFF
lea eax, dword ptr [ebp+FFFFFF18]
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004196C6(C)
<===这一行到004196C6，又构成一个小循环
|
:004196B8 8B08
mov ecx, dword ptr [eax]
****注：这里的ECX的取值如下1、取输入的注册码第7位(7)
2、取输入的注册码第6位(8)
3、取输入的注册码第8位(8)
4、取输入的注册码第1位(7)
5、取输入的注册码第9位(7)
6、取输入的注册码第3位(7)
7、取输入的注册码第10位(8)
8、取输入的注册码第4位(8)
9、取输入的注册码第5位(7)
****注：这里可以推断出输入的注册码第2位和第11位，可以为任意数字!
由于是定位取值，所在用假码78787878787将很定位，所以将假码改为12345678901，重新来。
对应的取值变码是768193045，记住这个对应关系后面用得到!
:004196BA
890A mov dword ptr [edx], ecx
<===取得这些值依次放在EDX步进为4的位置上，ECX的初始地址8AF250正好与最后比较的地址相同，故要仔细观察其变化
:004196BC
46 inc esi
:004196BD 83C204
add edx, 00000004
:004196C0 83C004
add eax, 00000004
:004196C3 83FE09
cmp esi, 00000009
<===初始值是1，所以要循环9次。
:004196C6 7EF0
jle 004196B8 <===循环9次后，不再跳转了。
:004196C8
BB04000000 mov ebx, 00000004
:004196CD
BE01000000 mov esi, 00000001 <===又初始化了，呵呵，又要循环了
:004196D2
3BDE cmp ebx, esi <===不相等
:004196D4 7C2B
jl 00419701
<===不会跳
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004196FF(C)
<===这一行到004196FF，又构成一个小循环，
:004196D6 8D8574FFFFFF
lea eax, dword ptr [ebp+FFFFFF74]
:004196DC 50
push eax
:004196DD
8D559C lea edx, dword ptr [ebp-64]
:004196E0 52
push edx
:004196E1 E8FAFCFFFF
call 004193E0
<===第一个CALL会使我们所关心的值(以8AF250为起始地址的步进为4的位置)有变化，F8进去看得究竟吧
:004196E6
83C408 add esp, 00000008
:004196E9 8D4D9C
lea ecx, dword ptr [ebp-64]
:004196EC 51
push ecx
:004196ED 8D8574FFFFFF
lea eax, dword ptr [ebp+FFFFFF74]
:004196F3
50 push eax
:004196F4 E8E7FCFFFF call 004193E0
<===第二个同样的CALL使我们所关心的值有变化
:004196F9
83C408 add esp, 00000008
:004196FC 46
inc esi <===esi为计数器，每次加1
:004196FD
3BDE cmp ebx, esi <===EBX=4
:004196FF 7DD5
jge 004196D6
<===这里循环4次，但由于有了两个同样的CALL，所以实际上768193045(假码的变形)已经被处理了8次，如何处理的呢？我们再研究一下call 004193E0(功能在下面有说)，
**** 将768193045处理后为：524472195
将524472195处理后为：502225745
将502225745处理后为：055757895
将055757895处理后为：550523445
将550523445处理后为：969148595
将969148595处理后为：815477145
将815477145处理后为：083225295
将083225295处理后为：826757845
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004196D4(C)
|
:00419701
BE01000000 mov esi, 00000001 <===又初始化了，呵呵，又要循环了
:00419706
8D95F0FEFFFF lea edx, dword ptr [ebp+FFFFFEF0]
:0041970C
8D45A0 lea eax, dword ptr [ebp-60]
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041974E(C)
|
<===这一行到0041974E，又构成一个小循环，将注册码处理成826757845后，与机器码依次比较，只要一次错了，就OVER
:0041970F
8B08 mov ecx, dword ptr [eax] <===EAX的初始地址8AF250
:00419711 3B0A
cmp ecx, dword ptr [edx]
<===ECX的初始地址8AF1A0
:00419713 742F
je 00419744 <===这里必须跳走，不然到00419742一行无条件跳转，就OVER了
:00419715
33C0 xor eax, eax
:00419717 BA02000000 mov edx, 00000002
:0041971C 50
push eax
:0041971D 8D4508
lea eax, dword ptr [ebp+08]
:00419720 FF4F1C
dec [edi+1C]
:00419723
E8882F0A00 call 004BC6B0
:00419728
FF4F1C dec [edi+1C]
:0041972B
8D450C lea eax, dword ptr [ebp+0C]
:0041972E BA02000000
mov edx, 00000002
:00419733 E8782F0A00
call 004BC6B0
:00419738 58
pop eax
:00419739 8B17
mov edx, dword ptr [edi]
:0041973B 64891500000000 mov dword ptr fs:[00000000], edx
:00419742 EB39
jmp 0041977D
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00419713(C)
|
:00419744
46 inc esi
:00419745 83C204
add edx, 00000004
:00419748 83C004
add eax, 00000004
:0041974B 83FE09
cmp esi, 00000009
:0041974E
7EBF jle 0041970F <===循环9次后，不再跳转了
:00419750
B001 mov al, 01
:00419752 BA02000000 mov edx, 00000002
:00419757 50
push eax
:00419758 8D4508
lea eax, dword ptr [ebp+08]
:0041975B FF4F1C
dec [edi+1C]
:0041975E
E84D2F0A00 call 004BC6B0
:00419763
FF4F1C dec [edi+1C]
:00419766
8D450C lea eax, dword ptr [ebp+0C]
:00419769 BA02000000
mov edx, 00000002
:0041976E E83D2F0A00
call 004BC6B0
:00419773 58
pop eax
<===这里要求是EAX出来的不能为0
:00419774 8B17
mov edx, dword ptr [edi]
:00419776
64891500000000 mov dword ptr fs:[00000000], edx
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004194C8(U), :00419558(U),
:00419634(U), :00419742(U)
|
:0041977D 5F
pop edi
:0041977E 5E
pop esi
:0041977F
5B pop ebx
:00419780 8BE5
mov esp, ebp
:00419782 5D
pop ebp
:00419783 C3
ret
.......
.......
--------------F8进入的数字变形call
004193E0-------------------------------------------------
* Referenced by a CALL at Addresses:
|:004196E1 , :004196F4
|
:004193E0 55
push ebp
:004193E1 8BEC
mov ebp, esp
:004193E3 53
push ebx
:004193E4
8B450C mov eax, dword ptr [ebp+0C]
:004193E7 8B5508
mov edx, dword ptr [ebp+08]
:004193EA 8B4A24
mov ecx, dword ptr [edx+24]
:004193ED
894824 mov dword ptr [eax+24], ecx
:004193F0 8B4A20
mov ecx, dword ptr [edx+20]
:004193F3 3B4A24
cmp ecx, dword ptr [edx+24]
<===第8位与第9位相比
```
